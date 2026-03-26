```
sudo apt update
sudo apt install -y podman

mkdir -p ~/devbox
cd ~/devbox
```

`~/devbox/Containerfile` 里只写这两行：
```
FROM debian:12
CMD ["/bin/bash"]
```

```
podman build -t mybox .
podman run -it --name devbox mybox
```


第一次 `podman run` 之后，这个容器就已经存在了。
以后你不用重新创建。
先启动它：
```
podman start devbox
```
再进去：
```
podman exec -it devbox bash
```


如果你在容器里输入：

`exit`

你只是退出这个 shell。

如果你想把容器停掉，在宿主机执行：

`podman stop devbox`

https://chatgpt.com/share/69c4b43f-69b0-832f-8557-6bae31b8916c


```
apt-get update && apt-get install -y \
curl \
git

curl -fsSL https://claude.ai/install.sh | bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> .bashrc && source .bashrc
```
