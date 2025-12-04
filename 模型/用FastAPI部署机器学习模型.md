# 用FastAPI部署机器学习模型：完整实战指南（基于Iris数据集）
本文基于视频内容，详细归纳了使用FastAPI部署机器学习模型的完整流程，涵盖环境准备、模型训练与保存、FastAPI应用搭建、数据验证、接口测试等核心环节，适合需要快速实现模型服务化的开发者参考。

## 一、核心目标与前置准备
### 1. 核心目标
通过FastAPI构建一个可访问的API接口，实现Iris数据集分类模型的在线部署，支持接收用户输入特征并返回预测结果，同时保障输入数据的合法性校验。

### 2. 前置依赖安装
需提前在系统中安装以下关键包，为模型训练和API开发提供支持：
- scikit-learn：用于加载数据集、训练逻辑回归模型
- pickle：用于模型的序列化与保存、加载
- fastapi：核心Web框架，用于构建API接口
- pydantic：用于输入数据的格式验证
- uvicorn：FastAPI的运行服务器
- typing：用于定义数据类型（如List）

## 二、模型训练与保存（第一步）
### 1. 核心步骤
1. 导入依赖模块：从scikit-learn导入逻辑回归模型（LogisticRegression）、Iris数据集加载工具（load_iris），以及pickle模块。
2. 加载并处理数据：调用load_iris()加载数据集，分离特征（iris.data）和目标变量（iris.target）。
3. 初始化并训练模型：创建逻辑回归模型实例，设置最大迭代次数为200，使用分离后的特征和目标变量训练模型。
4. 序列化保存模型：通过pickle.dump()方法，将训练好的模型保存为`iris_model.pkl`文件，便于后续API加载使用。

### 2. 关键代码执行
将训练逻辑写入`model.py`文件，通过终端执行`python3 model.py`启动训练，执行完成后会在当前目录生成模型文件`iris_model.pkl`。

## 三、FastAPI应用搭建（第二步）
### 1. 导入必要模块
需导入pickle（加载模型）、contextlib（管理应用生命周期）、fastapi（核心框架）、pydantic.BaseModel（数据验证）、typing.List（定义返回类型）。

### 2. 输入数据验证类定义
创建`IrisFeatures`类并继承pydantic.BaseModel，定义Iris数据集的4个特征字段及其类型（均为float）：
- s_length：花萼长度
- s_width：花萼宽度
- petal_length：花瓣长度
- petal_width：花瓣宽度
该类会自动校验输入数据类型，若非浮点型则返回422 Unprocessable Entity错误。

### 3. 应用生命周期管理（加载模型）
通过`@asynccontextmanager`装饰器定义`lifespan`函数，实现模型的启动时加载：
1. 定义全局变量`ML_MODEL`用于存储加载后的模型。
2. 在函数中通过try-except块处理模型加载逻辑：以读字节模式打开`iris_model.pkl`，用pickle.load()加载模型；若文件不存在或加载失败，抛出对应异常提示。
3. 生命周期函数会在FastAPI应用启动时自动执行，确保模型提前加载就绪，避免每次请求时重复加载。

### 4. FastAPI实例初始化
创建FastAPI应用实例，配置核心参数：
- title：应用名称（Iris Classification）
- description：应用描述（API for Iris flower classification）
- version：版本号（1.0.0）
- lifespan：绑定生命周期函数，实现模型启动加载

### 5. 预测接口定义
通过`@app.post("/predict")`装饰器定义POST类型预测接口，核心配置如下：
1. 接收参数：`features: IrisFeatures`，通过`IrisFeatures`类校验输入数据。
2. 返回类型：`response_model=List[int]`，明确接口返回整数类型的预测结果列表。
3. 接口逻辑：
   - 先校验全局变量`ML_MODEL`是否存在，若不存在则返回“Model not found”错误。
   - 将校验后的输入特征转换为模型可接收的格式（列表/数组），包含s_length、s_width、petal_length、petal_width四个字段。
   - 调用`ML_MODEL.predict()`执行预测，返回预测结果。

## 四、应用运行与接口测试
### 1. 启动FastAPI应用
在终端执行命令：`uvicorn app:app --reload`，参数说明：
- `app:app`：前一个`app`为文件名，后一个`app`为FastAPI实例名称。
- `--reload`：开发模式下启用热重载，修改代码后自动重启服务器。

### 2. 接口访问与测试
1. 应用启动后，访问`localhost:8000`即可看到API首页，确认应用正常运行。
2. 输入合法浮点型特征（如s_length=4.3、s_width=3.0、petal_length=1.3、petal_width=0.2），接口会返回对应的分类预测结果（0/1/2，对应Iris的三个品种）。
3. 若输入非浮点型数据（如字符串），接口会返回422错误，验证数据校验功能生效。

## 五、核心优势与关键要点
1. 高效性：FastAPI框架支持异步，性能优异，适合高并发场景下的模型服务。
2. 数据安全性：通过pydantic实现输入数据强制校验，避免非法数据导致模型报错。
3. 易用性：生命周期管理确保模型仅加载一次，提升接口响应速度；uvicorn服务器配置简单，开发调试便捷。
4. 可扩展性：该流程可复用至其他机器学习模型（如回归、分类模型），只需修改数据集、模型类型和特征定义即可。

要不要我帮你整理一份**完整可直接运行的代码文件（含模型训练+API部署）** ？


```
import pickle
import contextlib
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List
class IrisFeatures(BaseModel):
    s_length:float
    s_width:float
    p_length:float
    p_width:float
@contextlib.asynccontextmanager
async def lifespan(app:FastAPI):
    global ml_model
    try:
        with open('iris_model.pkl','rb') as file:
            ml_model=pickle.load(file)
    except FileNotFoundError:
        print('file is not present')
    except Exception as e:
        print('model file is not present')
    yield 
app = FastAPI(
    title=('Iris Prediction API'),
    description=('This is a simple API for predicting the iris species'),
    version=('1.0.0'),
    lifespan=lifespan,
)
@app.post('/predict',response_model=List[int])
def predict(features:IrisFeatures):
    if ml_model is None:
        return {'error':'model not found'}
    data = [[features.s_length,features.s_width,features.p_length,features.p_width]]
    prediction = ml_model.predict(data).tolist()
    return prediction
```
app.py

```
from sklearn.linear_model import LogisticRegreesion
from sklearn.datasets import load_iris
import pickle
iris = load_iris()
X,y = iris.data,iris.target
model  = LogisticRegreesion()
model.fit(X,y)
with open('iris_model.pkl','wb') as file:
    pickle.dump(model,file)
print('Model is saved as iris_model.pkl')

```
model.py
