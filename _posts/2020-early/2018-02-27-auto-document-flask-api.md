---
title: 自动为Flask写的API生成帮助文档
categories: [Tech]
tags: [python, flask, api-doc]
date: 2018-02-27
---

{% raw %}

Flask 是 Python 一个非常轻量的库，可以让你毫不费力地写一个简单的网站。如果你需要写一些后台 API 或者准备自动化测试数据时，Flask 是一个非常不错的选择。

## 一个 API 例子

举个例子，我们可以这样写几个 API，具体实现暂时略过：

```python
# views/api.py

api = Blueprint('api', __name__)

@api.route('/get_todo', methods=['GET'])
def get_todo():
    """Get all todo tasks."""
    pass


@api.route('/add_todo', methods=['POST'])
def add_todo():
    """
    Add a todo task,  please post data in json format, e.g.

    data = {
              "name":"the title",
              "task":"the detail"
            }
    """
    pass


@api.route('/delete_todo', methods=['GET', 'POST'])
def delete_todo():
    """Delete a todo task."""
    pass

```

一旦你的 API 完成，你可能需要和调用方沟通调用的细节，最好给一些例子。明明你已经在代码里给所有方法都写了注释，难道还要再把这些注释拿出来重新组织排版一下？

我猜你和我一样，听过这么一句话。

> **read the fucking manual**!

可是谁会去翻代码去看你的注释呢，何况你的代码他们还不一定能看到。如果能自动生成一个帮助页面那就好了。

## 自动化 API 帮助文档

假设我们的 API 都是以 `http://127.0.0.1/api/*` 的形式书写的，那么最好把 API 的完整列表就放在根目录下面，比如这样：

![api-demo-home](https://raw.githubusercontent.com/tobyqin/img/master/api-demo-home.png)

view 方法的实现主要依靠 `app.url_map` 来获取 Flask 中所有的 API：

```python
# views/api.py

def get_api_map():
    """Search API from rules, if match the pattern then we said it is API."""
    for rule in get_app().url_map.iter_rules():
        if re.search(r'/api/.+', str(rule)):
            yield str(rule), rule.endpoint


@api.route('/', methods=['GET'])
def index():
    """List all API to this page, api_map contains each api url + endpoint."""
    api_map = sorted(list(get_api_map()))
    index_url = url_for('main.index', _external=True)
    api_map = [(index_url + x[0][1:], x[1]) for x in api_map]
    return render_template('api_index.html', api_map=api_map)
```

模板的实现：

```html
# templates/api_index.html {% extends "./layout.html" %} {% block title %}API
Root{% endblock %} {% block breadcrumb_nav %}
<li><a href="{{ url_for('api.index') }}">Api Root</a></li>
{% endblock %} {% block page_header %}
<h1>Api Root</h1>
{% endblock %} {% block content_area %}
<pre>{
{% for i in api_map %}    "<a href="/docs/{{ i[1] }}">{{ i[0] }}</a>"{{ ",\n" if not loop.last }}{% endfor %}
}</pre>
{% endblock %}
```

接下来我们来文档化每个具体的 API 方法，最终的展示结果会是这样的。

![api-demo-full](https://raw.githubusercontent.com/tobyqin/img/master/api-demo-full.png)

view 方法的实现思路其实也很明确，我们可以通过 `app.view_functions` 这个字典找到每个 API 的 endpoint 所绑定的方法，然后访问方法的名字和文档即可。

```python
# views/main.py

main = Blueprint('main', __name__)


@main.route('/', methods=['GET'])
def index():
    """Redirect home page to docs page."""
    return redirect(url_for('api.index'))


@main.route('/docs/<endpoint>', methods=['GET'])
def docs(endpoint):
    """Document page for an endpoint."""
    api = {
        'endpoint': endpoint,
        'methods': [],
        'doc': '',
        'url': '',
        'name': ''
    }

    try:
        func = get_app().view_functions[endpoint]

        api['name'] = _get_api_name(func)
        api['doc'] = _get_api_doc(func)

        for rule in get_app().url_map.iter_rules():
            if rule.endpoint == endpoint:
                api['methods'] = ','.join(rule.methods)
                api['url'] = str(rule)

    except:
        api['doc'] = 'Invalid api endpoint: "{}"!'.format(endpoint)

    return render_template('api_docs.html', api=api)


def _get_api_name(func):
    """e.g. Convert 'do_work' to 'Do Work'"""
    words = func.__name__.split('_')
    words = [w.capitalize() for w in words]
    return ' '.join(words)


def _get_api_doc(func):
    if func.__doc__:
        return func.__doc__
    else:
        return 'No doc found for this API!'
```

模板的实现：

```html
{% extends "./layout.html" %} {% block title %}API - {{ api['name'] }}{%
endblock %} {% block breadcrumb_nav %}
<li><a href="{{ url_for('api.index') }}">Api Root</a></li>
<li><a href="{{ api['url'] }}">{{ api['name'] }}</a></li>
{% endblock %} {% block page_header %}
<h1>{{ api['name'] | upper }}</h1>
{% endblock %} {% block content_area %}
<pre>
<b>Target:</b><span><a href="{{ api['url'] }}">{{ api['url'] }}</a></span>
<b>Allow :</b> <span>{{ api['methods'] }}</span>
<b>Usage :</b> <span>{{ api['doc'] }}</span>
</pre>
{% endblock %}
```

## GitHub 项目地址

如果你想看完整的例子，可以到我的 GitHub 去拉一份代码。

> https://github.com/tobyqin/flask_api_doc

只需要三步就可以在你的机器上运行 Demo：

```shell
cd /path/to/flask_api/doc
pip install -r requirements.txt
python main.py
```

如果你觉得 Demo 不错，欢迎给个 Star。有建议或者想法也可以拿来讨论。
{% endraw %}
