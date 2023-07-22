## 使用容器运行预测模型

修改app.py文件

```python
import numpy as np #导入NumPy
from flask import Flask, request, render_template #导入Flask相关包
import pickle #导入模块序列化包

app = Flask(__name__)
model = pickle.load(open('model.pkl', 'rb'))

@app.route('/')
def home(): # 默认启动页面
    return render_template('index.html') # 启动index.html

@app.route('/predict',methods=['POST'])
def predict(): # 启动预测页面

    features = [int(x) for x in request.form.values()] # 输入特征
    label = [np.array(features)] # 标签
    prediction = model.predict(label) # 预测结果

    output = round(prediction[0], 2) #输出预测结果

    return render_template('index.html', #预测浏览量
                           prediction_text='浏览量 {}'.format(int(output)))

# @app.route('/results',methods=['POST'])
# def results():

#     data = request.get_json(force=True)
#     prediction = model.predict([np.array(list(data.values()))])

#     output = prediction[0]
#     return jsonify(output)

if __name__ == "__main__": # 启动程序,增加host="0.0.0.0"
    app.run(host="0.0.0.0",debug=True)

```

对最后的代码稍作修改，在运行flask服务时加上host="0.0.0.0"，这样容器宿主机可以发布这个服务


创建 `Dockerfile`: 在项目根目录下创建一个名为 `Dockerfile` 的文件(没有文件扩展名)，并添加以下内容

```Dockerfile
# 使用官方 Python 基础镜像
FROM python:3.8-slim

# 设置工作目录
WORKDIR /app

# 将当前目录的内容复制到容器中的 /app 目录
COPY . /app

# 安装任何需要的包
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 运行模型
RUN python model.py

# 设置外部可以访问容器的端口，例如 80
EXPOSE 5000

# 定义环境变量
ENV NAME World

# 运行 app.py 当容器启动时
CMD ["python", "app.py"]

```

创建 `requirements.txt`: 在项目根目录里创建一个名为 `requirements.txt` 的文件，包含所有需要的 Python 包

```
Flask==1.1.2
numpy==1.19.5
pandas==1.1.5
scikit-learn==0.24.1
Jinja2==3.0.1
MarkupSafe==2.0.1
itsdangerous==1.1.0
Werkzeug==1.0.1
```

这个 `requirements.txt` 文件列出了 Python 项目所需的依赖包及其版本。这有助于确保项目在不同环境中正确运行。以下是每个包的简要介绍：

1. Flask (1.1.2) - 是一个轻量级 Python Web 框架，用于构建 web 应用程序。这里有一个简化版的 Flask 1.1.2。    
2. numpy (1.19.5) - 是一个用于支持大型多维数组和矩阵的 Python 库。它还包括大量用于进行数组操作的函数。    
3. pandas (1.1.5) - 是一个用于数据分析和数据处理的 Python 库。例如，它允许创建和处理 DataFrame，分析和操作数据，如排序、过滤、聚合等操作。    
4. scikit-learn (0.24.1) - 是一个用于数据挖掘和数据分析的 Python 库。它包含多种统计模型和机器学习算法，用于分类、回归、聚类、降维、特征工程等任务。    
5. Jinja2 (3.0.1) - 是一个 Python 的模板引擎。它可以与 Flask 一起使用，用于在 web 应用程序中生成动态 HTML 页面。    
6. MarkupSafe (2.0.1) - 是一个用于处理字符串的 Python 库，确保在渲染 HTML 或 XML 时它们是安全的。它与 Jinja2 一起工作，确保模板引擎不会引入安全漏洞。    
7. itsdangerous (1.1.0) - 是一个用于确保数据安全性的 Python 库。例如，将数据签名，以便可以在不受信任的环境中安全发送。它通常与 Flask 一起使用，用于处理诸如 cookie 签名之类的功能。    
8. Werkzeug (1.0.1) - 是一个 WSGI 工具包，可用于开发可复用、可扩展的 Python Web 应用程序。它与 Flask 集成，提供了底层 HTTP 功能和路由。

构建 Docker 镜像: 打开终端，导航到包含 `Dockerfile` 和 `requirements.txt` 的项目根目录。然后运行以下命令以构建 Docker 镜像
```bash
docker build -t lab0715 .
```
注意：命令末尾的点表示当前目录，确保输入这个点。此命令会使用 `Dockerfile` 和 `requirements.txt` 来构建一个 Docker 镜像，并为其指定一个名字，例如 "your_image_name"。
 


运行 Docker 容器: 在终端中运行以下命令启动 Docker 容器
```bash
docker run -d -p 5000:5000 lab0715
```
此命令将容器的端口 5000 映射到主机的端口 5000，现在，您的 Web 应用程序已经在 Docker 容器中运行。您可以通过访问 `http://localhost:5000` 来查看应用程序


也可以使用现成的容器映像
```bash
docker run -d -p 5000:5000 chengzh/mlpredict
```
