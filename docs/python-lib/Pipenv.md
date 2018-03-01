# pipenv安装使用

## 安装

```shell
$ pip install pipenv
...
```

## 为项目安装依赖包

`Pipenv` 管理每个项目的依赖关系。要安装软件包时，请更改到您的项目目录并运行：

```shell
$ cd  myproject
$ pipenv install requests
...
```

`Pipenv` 将在您的项目目录中安装超赞的 `Requests` 库并为您创建一个 `Pipfile`。

## 使用安装好的包

现在安装了 Requests，您可以创建一个简单的 requests_demo.py 文件来使用它：

```python
import requests

response = requests.get('https://httpbin.org/ip')

print('Your IP is {0}'.format(response.json()['origin']))
```

然后您就可以使用 `pipenv run` 运行这段脚本：

```shell
$ pipenv run python main.py
Your IP is 8.8.8.8
```

使用 `pipenv run` 可确保您的安装包可用于您的脚本。我们还可以生成一个新的 shell， 确保所有命令都可以使用 `pipenv shell` 访问已安装的包。

## pipenv用法

在使用pipenv之前，必须彻底的忘记pip这个东西
新建一个准备当环境的文件夹myproject，并cd进入该文件夹：

- pipenv --three : 会使用当前系统的Python3创建环境

- pipenv --python 3.6 : 指定某一Python版本创建环境

- pipenv shell : 激活虚拟环境

Launching subshell in virtual environment. Type 'exit' to return.

进入后可以直接使用诸如 python xxx.py运行(该python解释器为虚拟环境下的python),退出敲入exit即可
如果不激活可以在目录下使用 `pipenv run python`


- pipenv --where ：显示目录信息

/home/xxx/myproject

- pipenv --venv 显示虚拟环境信息

/home/xxx/.local/share/virtualenvs/myproject-xxxxxxxx
或windows上: C:\Users\xxx\.virtualenvs\myproject-xxxxxxxx

- pipenv --py 显示Python解释器信息

/home/xxx/.local/share/virtualenvs/myproject-xxxxxxxx/bin/python

- pipenv install requests 安装相关模块并加入到Pipfile

- pipenv install django==1.11 安装固定版本模块并加入到Pipfile

- pipenv graph : 查看目前安装的库及其依赖

```
requests==2.18.4
  - certifi [required: >=2017.4.17, installed: 2017.11.5]
  - chardet [required: <3.1.0,>=3.0.2, installed: 3.0.4]
  - idna [required: >=2.5,<2.7, installed: 2.6]
  - urllib3 [required: >=1.21.1,<1.23, installed: 1.22]

```

- pipenv check : 检查安全漏洞

- pipenv uninstall --all : 卸载全部包并从Pipfile中移除

- pipenv --rm : 删除虚拟环境

你会发现`C:\Users\xxx\.virtualenvs\`目录下(或者linux下目录)对应的虚拟环境文件夹删除了

- 设置源

可以设置国内源：Pipfile文件中[source]下面url属性，比如修改成：
url = "https://pypi.tuna.tsinghua.edu.cn/simple"