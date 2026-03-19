`
https://github.com/brbisheng/argecon
`
```python
# 1. 创建虚拟环境 (在 argecon 目录下)
python3 -m venv venv

# 2. 激活虚拟环境
source venv/bin/activate

# 3. 此时你的命令行开头会出现 (venv)，现在可以放心安装了
pip install -r requirements.txt

```

`

uvicorn src.api.app:app --host 0.0.0.0 --port 8000 &
pkill -f uvicorn

`
