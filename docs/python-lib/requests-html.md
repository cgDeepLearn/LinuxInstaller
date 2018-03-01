# requets-html安装使用

requests-html库地址: https://github.com/kennethreitz/requests-html
只支持python3的较新版本。

特性：

- Full JavaScript support!
- CSS Selectors (a.k.a jQuery-style, thanks to PyQuery).
- XPath Selectors, for the faint at heart.
- Mocked user-agent (like a real web browser).
- Automatic following of redirects.
- Connection–pooling and cookie persistence.
- The Requests experience you know and love, with magical parsing abilities.

## 目录

- [安装](#安装)
- [使用](#使用)

## 安装

```shell
pipenv install requests-html
```

`pipenv`安装使用请参考[pipenv](Pipenv.md)

## 使用

- 获取首页

```python
>>> from requests_html import HTMLSession
>>> session = HTMLSession()

>>> r = session.get('https://python.org/')
```

- 获取所有链接

```python
>>> r.html.links
{'//docs.python.org/3/tutorial/', '/about/apps/', ...}
>>> type(r.html.links) # 返回的是一个集合set
<class 'set'>
```

- 获取所有绝对地址

```python
>>> r.html.absolute_links
{'https://github.com/python/pythondotorg/issues', 'https://docs.python.org/3/tutorial/', ...}
```

- 使用JQuery选择器

```python
>>> about = r.html.find('#about', first=True) # first为False得到的一个list
>>> print(about.text)
About
Applications
Quotes
Getting Started
Help
Python Brochure
```

- 获取元素属性

```python
>>> about.attrs
{'id': 'about', 'class': ('tier-1', 'element-1'), 'aria-haspopup': 'true'}
```

- 渲染成HTML

```python
>>> about.html
'<li aria-haspopup="true" class="tier-1 element-1 " id="about">\n<a class="" href="/about/" title="">About</a>\n<ul aria-hidden="true" class="subnav menu" role="menu">\n<li class="tier-2 element-1" role="treeitem"><a href="/about/apps/" title="">Applications</a></li>\n<li class="tier-2 element-2" role="treeitem"><a href="/about/quotes/" title="">Quotes</a></li>\n<li class="tier-2 element-3" role="treeitem"><a href="/about/gettingstarted/" title="">Getting Started</a></li>\n<li class="tier-2 element-4" role="treeitem"><a href="/about/help/" title="">Help</a></li>\n<li class="tier-2 element-5" role="treeitem"><a href="http://brochure.getpython.info/" title="">Python Brochure</a></li>\n</ul>\n</li>'
```

- 选择元素

```python
>>> about.find('a')
[<Element 'a' href='/about/' title='' class=''>, <Element 'a' href='/about/apps/' title=''>, <Element 'a' href='/about/quotes/' title=''>, <Element 'a' href='/about/gettingstarted/' title=''>, <Element 'a' href='/about/help/' title=''>, <Element 'a' href='http://brochure.getpython.info/' title=''>]
```

- 单个元素内查找链接

```python
>>> about.absolute_links
{'http://brochure.getpython.info/', 'https://www.python.org/about/gettingstarted/', ...}
```

- 页面文本查找

```python
>>> r.html.search('Python is a {} language')[0]
programming
```

- CSS选择器

```python
>>> r = session.get('https://github.com/')
>>> sel = 'body > div.application-main > div.jumbotron.jumbotron-codelines > div > div > div.col-md-7.text-center.text-md-left > p'

>>> print(r.html.find(sel, first=True).text)
GitHub is a development platform inspired by the way you work. From open source to business, you can host and review code, manage projects, and build software alongside millions of other developers.
```

- xpath解析

```python
>>> r.html.xpath('a')
[<Element 'a' class='btn' href='https://help.github.com/articles/supported-browsers'>]
```

- JavaScript Support

```python
>>> r = session.get('http://python-requests.org')

>>> r.html.render()

>>> r.html.search('Python 2 will retire in only {months} months!')['months']
'<time>25</time>'
```

Note, 首次运行 render() 方法, 会下载 Chromium 到home目录 (e.g. ~/.pyppeteer/). 这只会出现一次

- Using without Requests

```python
>>> from requests_html import HTML
>>> doc = """<a href='https://httpbin.org'>"""

>>> html = HTML(html=doc)
>>> html.links
{'https://httpbin.org'}
```