---
layout: post
title: Flask 源码分析-Jinja模板
date:  2017-12-28 21:19:54
tags:
- Flask
- Python
categories: Flask
description: 本文主要分析了 Flask 中 Jinja2 模板的使用。
---
本文主要分析了 Flask 中 Jinja2 模板的使用。  
## 为什么需要模板
Flask 中，视图函数的作用是生成请求的响应，假设我们的视图函数需要返回一个 HTML 页面，也许我们会想到这样做：  

```python
@app.route('/index')
def index():
    user = { 'nickname': 'Miguel' } # fake user
    return '''
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello, ''' + user['nickname'] + '''</h1>
  </body>
</html>
'''
```  
如果我们要返回多个含有大量复杂内容的页面呢，如果按照这种方式，代码将会很复杂，扩展性也很差，把业务逻辑和表现逻辑混在一起会导致代码难以理解和维护。如果能够使得业务逻辑和表现逻辑分离，程序将会更加容易组织，而将表现逻辑移到模板中，可以实现这种分离，提升程序的可维护性。  
模板是一个包含响应文本的文件，其中包含占位变量表示动态部分，其具体值只在请求的上下文才能知道。使用真实值替换变量，再返回最终得到的响应字符串，这一过程称为渲染。为了渲染模板，Flask 使用了名为 Jinja2 的模板引擎。 
## Jinja2 模板引擎  
让我们来编写一个简单的模板文件 `index.html`  
```python
<h1>Hello, {{ name }}!</h1>
```
接下来看看视图函数中是怎样处理模板的：  
```python
from flask import Flask, render_template
@app.route('/index/<name>')
def index(name):
    return render_template('index.html', name=name)
```
Flask 提供的 `render_template` 函数把 Jinja2 模板引擎集成到了程序中，该函数的第一个参数是模板的文件名，随后的参数表示模板中变量对应的真实值，函数返回一个渲染后的模板。在内部，`render_template` 调用了 Jinja2 模板引擎，Jinja2 模板引擎是 Flask 框架的一部分，下面我们看看 Jinja2 的基本用法。

### Jinja2 的基本用法   
为了给应用加载模板，一种简单配置 Jinja2 的方法如下：  
```python
from jinja2 import Environment, PackageLoader, select_autoescape
env = Environment(
    loader=FileSystemLoader('templates'),
)
template = env.get_template('index.html')
print template.render(user='abc')
```  
首先创建一个模板环境对象，该对象含有一个用于查找模板的加载器，本例中，该加载器配置为在`templates` 目录下查找模板。为了加载模板，得调用 `get_template` 函数，传入模板文件名，这个就返回一个被加载的模板，若需要渲染模板中的变量，调用 `render` 函数即可。通过以上几步，就完成了模板的加载和渲染。
`Environment`对象中的加载器主要负责从资源中加载对应的模板，被加载的模板会被保存在内存中。所有的加载器都继承自 `BaseLoader`，并覆写了 `get_source` 函数。`get_source` 函数用于获取模板文件的unicode字符串或ASCII字节串，模板文件名称，然后返回一个元组`(source, filename, uptodate) `。`BaseLoader` 的 `load` 函数通过调用 `get_source` 可加载模板。
接着分析 Flask 中 Jinja2 的用法。
### Flask 中 Jinja2 的使用  
在视图函数中，我们加载、渲染模板时调用了 `render_template` 函数，该函数定义如下：
```python
def render_template(template_name_or_list, **context):
    ctx = _app_ctx_stack.top
    ctx.app.update_template_context(context)
    return _render(ctx.app.jinja_env.get_or_select_template(template_name_or_list),
                   context, ctx.app)
```
`ctx.app.jinja_env.get_or_select_template(template_name_or_list)` 返回一个被加载的模板，该过程的具体的实现经过了多层函数的调用。应用上下文 `ctx.app` 调用 `jinja_env`，该函数返回了一个 `Environment` 对象，然后调用 `get_or_select_template` 返回加载的模板。
```python
    def get_or_select_template(self, template_name_or_list,
                               parent=None, globals=None):
        if isinstance(template_name_or_list, string_types):
            return self.get_template(template_name_or_list, parent, globals)
        elif isinstance(template_name_or_list, Template):
            return template_name_or_list
        return self.select_template(template_name_or_list, parent, globals)
```
在 `_render` 函数中，调用 `render` 方法，完成对模板的渲染。和 Jinja2 库中模板的加载和渲染过程对比发现，Flask 只是对Jinja2 库中的相关函数进行了封装，其本质还是相同的。
```python
def _render(template, context, app):
    """Renders the template and fires the signal"""

    before_render_template.send(app, template=template, context=context)
    rv = template.render(context)
    template_rendered.send(app, template=template, context=context)
    return rv
```

