不是完全不能用这个容器。准确说：

**你可以继续用现有 `devbox` 容器跑 Vite。不能做的是：给一个已经创建好的容器临时追加 `-p 127.0.0.1:5173:5173` 端口映射。**

`-p` 是容器创建时写进网络配置里的，类似给一个插线板预留插孔；容器已经创建后，Podman 没有一个简单的 `podman add-port devbox 5173` 命令。

但你不一定要重建。你有几个办法。

---

## 最省事：不用重建，用 Podman 自带网络访问容器 IP

先在现有容器里启动 Vite：

```bash
podman exec -it devbox sh
```

容器内：

```bash
npm run dev -- --host 0.0.0.0
```

然后在 VPS 上查容器 IP：

```bash
podman inspect devbox --format '{{.NetworkSettings.IPAddress}}'
```

如果输出类似：

```text
10.88.0.23
```

在 VPS 上测试：

```bash
curl http://10.88.0.23:5173
```

如果能看到 HTML，就说明 Vite 已经通了。

然后你本机开 SSH 隧道：

```bash
ssh -L 127.0.0.1:5173:10.88.0.23:5173 root@your-vps-ip
```

本机浏览器打开：

```text
http://localhost:5173
```

这条路是：

```text
本机浏览器
→ 本机 127.0.0.1:5173
→ SSH
→ VPS
→ 容器 IP 10.88.0.23:5173
→ Vite
```

这个方式不需要重新创建容器。

---

## 如果上面查不到 IP

试这个：

```bash
podman inspect devbox --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

如果输出一个 IP，例如：

```text
10.89.0.17
```

就用它：

```bash
curl http://10.89.0.17:5173
```

本机：

```bash
ssh -L 127.0.0.1:5173:10.89.0.17:5173 root@your-vps-ip
```

---

## 如果你用的是 rootless Podman，容器 IP 可能不好从 VPS 直接访问

这种情况下，`curl 容器IP:5173` 可能不通。那就只能走下面两条路之一：

### 方案 A：用 `podman exec` 在容器里确认 Vite 跑起来，然后重建带端口映射的测试容器

这是最稳定、最干净的方式：

```bash
podman commit devbox devbox-image
```

```bash
podman run --name devbox-test --rm -it \
  -p 127.0.0.1:5173:5173 \
  devbox-image
```

容器内：

```bash
npm run dev -- --host 0.0.0.0
```

本机：

```bash
ssh -L 127.0.0.1:5173:127.0.0.1:5173 root@your-vps-ip
```

### 方案 B：如果这个容器本来就是开发容器，以后创建它时就固定加端口

以后第一次创建时写：

```bash
podman run --name devbox -it \
  -p 127.0.0.1:5173:5173 \
  your-image-name
```

后面就可以一直：

```bash
podman start -ai devbox
```

或者：

```bash
podman exec -it devbox sh
```

---

## 为什么会这样

因为容器的这些东西是在创建时固定的：

```text
端口映射
挂载目录
网络模式
容器名
部分权限参数
```

后面 `podman exec` 只是“进入已经存在的容器执行命令”，不能改它外面的网络布线。

---

## 你现在先试这个，不重建

在 VPS 上：

```bash
podman exec -it devbox sh
```

容器里：

```bash
npm run dev -- --host 0.0.0.0
```

另开一个 VPS 终端：

```bash
podman inspect devbox --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

假设输出是：

```text
10.88.0.23
```

VPS 上测试：

```bash
curl http://10.88.0.23:5173
```

如果通，本机直接：

```bash
ssh -L 127.0.0.1:5173:10.88.0.23:5173 root@your-vps-ip
```

本机浏览器：

```text
http://localhost:5173
```

测试完关闭：

```text
本机 SSH 窗口：Ctrl + C
容器里的 Vite：Ctrl + C
```

结论：**不是必须重建；先试容器 IP + SSH 隧道。只有容器 IP 从 VPS 访问不通时，才需要 commit 后重建一个带 `-p` 的测试容器。**
