# 使用官方 Python 基础镜像
FROM python:3.8-slim

# 设置工作目录
WORKDIR /app

# 将当前目录的内容复制到容器中的 /app 目录
COPY . /app

# 安装任何需要的包
RUN pip install --trusted-host pypi.python.org -r requirements.txt


RUN python model.py

# 设置外部可以访问容器的端口，例如 80
EXPOSE 5000

# 定义环境变量
ENV NAME World

# 运行 app.py 当容器启动时
CMD ["python", "app.py"]
