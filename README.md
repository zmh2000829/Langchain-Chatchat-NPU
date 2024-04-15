# Langchain-Chatchat-NPU
基于Langchain-Chatchat进行NPU适配，适配修改的代码待上传



## 本项目基于Langchain-Chatchat 进行NPU适配（待完善）

### 1. 环境配置

+ 首先，确保你的机器安装了 Python 3.8 - 3.10

```
$ python --version
Python 3.10.12
```

接着，创建一个虚拟环境，并在虚拟环境内安装项目的依赖

```shell
# 拉取仓库
$ git clone https://github.com/zmh2000829/Langchain-Chatchat-NPU.git

# 进入目录
$ cd Langchain-Chatchat-NPU

# 安装全部依赖
$ pip install -r requirements.txt 
$ pip install -r requirements_api.txt
$ pip install -r requirements_webui.txt  

# 默认依赖包括基本运行环境（FAISS向量库）。如果要使用 milvus/pg_vector 等向量库，请将 requirements.txt 中相应依赖取消注释再安装。
```

- torch_npu 编译安装（以配套PyTorch2.1.0为例）

```shell
# 克隆torch_npu代码仓
git clone https://gitee.com/ascend/pytorch.git -b v2.1.0-6.0.rc1 --depth 1

# 构建镜像
cd pytorch/ci/docker/{arch} # {arch} for X86 or ARM
docker build -t manylinux-builder:v1 .

# 进入Docker容器,
docker run -it -v /{code_path}/pytorch:/home/pytorch manylinux-builder:v1 bash

### {code_path} is the torch_npu source code path
### 如无法Pull docker镜像，修改 /etc/docker/daemon.json 代码增加如下
# "registry-mirrors": [
#     "quay.io"
# ]
### 如进入docker无法pip install，在DockerFile添加相应代理 
# env http_proxy "http://xxx:8080" 
# env https_proxy "http://xxx:8080" 

# 编译torch_npu, 以python 3.10 为例
cd /home/pytorch
bash ci/build.sh --python=3.10

# pip 安装编译生成的 .whl文件(dist目录下，xxx 2.1.0.post3 xxx)
pip install /home/pytorch/dist/*.whl
```

- 初始化**CANN**环境变量

```shell
# Default path, change it if needed.
source /usr/local/Ascend/ascend-toolkit/set_env.sh
```

- 快速验证NPU

```python
# 方式一 (官方)
import torch
import torch_npu

x = torch.randn(2, 2).npu()
y = torch.randn(2, 2).npu()
z = x.mm(y)

print(z)

# 方式二
import torch, torch_npu
print(torch.npu.is_available()) # True
```



####  PyTorch与Python版本配套表

具体相关配套关系见：https://gitee.com/ascend/pytorch/tree/v2.1.0-6.0.rc1/

| PyTorch版本   | Python版本                                                   |
| ------------- | ------------------------------------------------------------ |
| PyTorch1.11.0 | Python3.7.x(>=3.7.5), Python3.8.x, Python3.9.x, Python3.10.x |
| PyTorch2.1.0  | Python3.8.x, Python3.9.x, Python3.10.x                       |
| PyTorch2.2.0  | Python3.8.x, Python3.9.x, Python3.10.x                       |

### 