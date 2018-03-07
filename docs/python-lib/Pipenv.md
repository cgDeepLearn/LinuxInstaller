# pipenv安装使用

pipenv库:https://github.com/pypa/pipenv

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

- pipenv install requests : 安装相关模块并加入到Pipfile

- pipenv install django==1.11 : 安装固定版本模块并加入到Pipfile
- pipenv install pytest --dev : 安装开发依赖(Adding pytest to Pipfile's [dev-packages]...)

- pipenv lock

```shell
$ pipenv lock
Assuring all dependencies from Pipfile are installed...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Note: your project now has only default [packages] installed.
To install [dev-packages], run: $ pipenv install --dev
```

- pipenv install --dev  : Install all dev dependencies:

```shell
$ pipenv install --dev
Pipfile found at /Users/xxx/repos/Pipfile. Considering this to be the project home.
Pipfile.lock out of date, updating...
Assuring all dependencies from Pipfile are installed...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
```

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

### Full Usage

```
$ pipenv
Usage: pipenv [OPTIONS] COMMAND [ARGS]...

Options:
  --update         Update Pipenv & pip to latest.
  --where          Output project home information.
  --venv           Output virtualenv information.
  --py             Output Python interpreter information.
  --envs           Output Environment Variable options.
  --rm             Remove the virtualenv.
  --bare           Minimal output.
  --completion     Output completion (to be eval 'd).
  --man            Display manpage.
  --three / --two  Use Python 3/2 when creating virtualenv.
  --python TEXT    Specify which version of Python virtualenv should use.
  --site-packages  Enable site-packages for the virtualenv.
  --version        Show the version and exit.
  -h, --help       Show this message and exit.


Usage Examples:
   Create a new project using Python 3.6, specifically:
   $ pipenv --python 3.6

   Install all dependencies for a project (including dev):
   $ pipenv install --dev

   Create a lockfile containing pre-releases:
   $ pipenv lock --pre

   Show a graph of your installed dependencies:
   $ pipenv graph

   Check your installed dependencies for security vulnerabilities:
   $ pipenv check

   Install a local setup.py into your virtual environment/Pipfile:
   $ pipenv install -e .

   Use a lower-level pip command:
   $ pipenv run pip freeze

Commands:
  check      Checks for security vulnerabilities and against PEP 508 markers
             provided in Pipfile.
  clean      Uninstalls all packages not specified in Pipfile.lock.
  graph      Displays currently–installed dependency graph information.
  install    Installs provided packages and adds them to Pipfile, or (if none
             is given), installs all packages.
  lock       Generates Pipfile.lock.
  open       View a given module in your editor.
  run        Spawns a command installed into the virtualenv.
  shell      Spawns a shell within the virtualenv.
  sync       Installs all packages specified in Pipfile.lock.
  uninstall  Un-installs a provided package and removes it from Pipfile.
```

### issues

如果之前安装过`Annaconda`，`pipenv install`出现了下列问题，请先`conda update python`

```
ERROR: The executable /home/xxx/.virtualenvs/mynews-BcpTHAaR/bin/python is not functioning
ERROR: It thinks sys.prefix is '/home/xxx/program/mynews' (should be '/home/cg/.virtualenvs/mynews-BcpTHAaR')
ERROR: virtualenv is not compatible with this system or executable

File "/home/xxx/anaconda3/lib/python3.6/site-packages/pipenv/core.py", line 343, in ensure_pipfile
    project.create_pipfile(python=python)
  File "/home/xxx/anaconda3/lib/python3.6/site-packages/pipenv/project.py", line 435, in create_pipfile
    data[u'requires'] = {'python_version': python_version(self.which('python'))[:len('2.7')]}
TypeError: 'NoneType' object is not subscriptable

```