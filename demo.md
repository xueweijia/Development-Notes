# Demo Notes

**先过一遍上周实验课杜同学讲解的基础内容，再实际给大家演示一遍上周的实验内容，最后讲解数据库相关的 Models，和一些 Django 提供给我们的更方便处理表单的 Forms 和通用视图 Generic Views.**  

之后再简单介绍 JavaScript 框架 Vue.js 和 Bootstrap4 的基本用法，鉴于课时有限，Vue.js 或 Bootstrap 都是工程类，不是科研项目，而且具有详尽的英文/中文官方文档可查，Vue.js 的作者甚至是中国人，中文文档非常全（甚至在前些年有吐槽 Vue 开源的代码中有很多中文注释，看不懂）。

这个课程也是给大家提供开发的技术栈的一些**方向选择**，市场上的相关技术也非常多，而且都具有各自明显的优势或劣势。但在某些意义上，比如设计模式、设计理念都是互通的。大家课下如果真的想做 Web（虽然工程类项目难度不大，而且“不必重头造轮子”，在开发初期都是在用人家写好的轮子，但我觉得 Web 这类工程项目还是很有必要的），仔细看看各家技术或框架的介绍，比较下哪个更适合自己。但最开始的时候，懂的知识很少，你看各家文档和优劣势比较觉得完全看不懂，人格分裂了，那就像刚刚说的，既然很多框架持有的思想是共通的，文档也很全，不如跟风找个最火的框架学习，后续转其他框架或技术成本也并不是很高。

另外非常建议回头找一个**真实的项目**去做，我觉得只有通过项目驱动，遇到真实场景和真实问题，才能对各种技术有更深入的理解。

实验二的内容。绕不开。

* settings.py -> 注册 polls
* views.py: HttpResponse()
* urls.py

```python
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index)
]
```

* django_project/urls.py -> 添加 URL

```python
from django.urls import include

urlpatterns = [
    path('', include('polls.urls')),
]
```

* migrate -> 生成 db.sqlite3 数据库文件
* runserver 8080/Debug

**************

* render: 后端向前端传递数据

> 前后端分离：后端向前端传递 JSON 文件供前端读取，纯数据。大型应用开发中极为有用，也是大多企业采用的开发方式。确定好前后端的数据接口后，前端工程师和后端工程师可以左右开弓，分别开发各自逻辑。  
> Django Templates 系统字符插值：简单、直观、方便，但后端对前端的侵入过大，前后端耦合性过强。

```python
from django.shortcuts import render

def index(request):
    retrun render(request, 'polls/index.html', {
        'message': 'Hello Django!',
    })
```

* 切回 PPT，P15，映射关系

**************

* 手写数据库

```python
from django.db import models
from django.utils import timezone


class Questions(models.Model):
    question_text = models.CharField(verbose_name='题目', max_length=100)
    time_created = models.DateTimeField(verbose_name='创建时间', default=timezone.now, editable=False)


class Choices(models.Model):
    question = models.ForeignKey(to=Questions, verbose_name='问题', on_delete=models.CASCADE)
    choice_text = models.CharField(verbose_name='选项', max_length=100)
    votes = models.PositiveIntegerField(verbose_name='投票数量', default=0)
```

* Django database API

`python manage.py shell`/点击 Python Console

```python
from polls.models import *
# Add
Question.objects.create(question_text='中午吃什么?')
# Search
Question.objects.all()
# 发现返回对象名不可读，重写该类的 __str__ 方法，将函数对象转为人类可读的形式
def __str__(self):
    return self.question_text
```

* templates -> views.py -> urls.py

**CSRF_TOKEN!!!**

两种 Templates 语言：

1. 表示值引用的 `{{}}`；
2. 表示*表达式*的 `{% %}`.

```html
<div id="content">
    <form method="post" action="">
        {% csrf_token %}
        <div id="question">
            <label>问题名称
                <input type="text" name="question_text">
            </label>
        </div>
        <div class="choices">
            <label>选项 A
                <input type="text" name="choice_text_a">
            </label>
            <label>选项 B
                <input type="text" name="choice_text_b">
            </label>
            <label>选项 C
                <input type="text" name="choice_text_c">
            </label>
        </div>
        <input type="submit" value="提交">
        <input type="reset" value="重置">
    </form>
</div>
```

```python
# 首先
def create_questions(request):
    return render(request, 'polls/create.html', {})
# URLS.py 设置映射
path('creation', views.create_questions)
# 正常查看后，Debug 写响应逻辑
```

* 查看问题

```python
def view_questions(request):
    questions = Questions.objects.all()
    for question in questions:
    # ForeignKey 可以直接用 一对多关系中的 一 的字段名 + _set 直接引用
    # 在这里存储为 question 的一个属性，直接传给前端
        question.choices = question.choices_set.all()
    return render(request, 'polls/view.html', {'questions': questions})
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{% for question in questions %}
    <p>问题{{ forloop.counter }}：{{ question.question_text }}</p>
    <ul>
    {% for choice in question.choices %}
        <li>选项{{ forloop.counter }} - {{ choice.choice_text }}</li>
    {% endfor %}
    </ul>
{% endfor %}
</body>
</html>
```

**************

* 引入 Forms.py

我们发现，我们在 Models.py 中定义过两种类别的字段：用户通过表单输入的字段和其他说明这些输入信息的字段，如投票题目 Question 表中的题目名称和创建时间。题目名称是需要用户在前端的表单进行输入的，创建时间来记录用户是何时输入了本字段。

但是在前端，我们又定义了一遍题目名称。

随后在 views.py 中，我们又完全响应了所有的前端输入请求，并将他们写入数据库中。

这三步代码间具有某种重复，可以进一步进行封装。具体的重复字段就是我们数据库 models.py 中用户通过表单输入的字段，即全体字段减去其他说明需要用户前端输入的字段。

Django 为我们提供了 Forms 组件，可以选出 models.py 中需要暴露到用户表单中的字段，设置它们的呈现方式（Radio/Checkbox 等 widget 组件），在 Views 中发送到前端，再返回验证即可。Forms 能直接进行一定程度的表单验证，通过后直接存入数据库。

```python
# forms.py
from django import forms
from .models import *

class QuestionForm(forms.ModelForm):
    class Meta:
        model = Questions
        fields = ('question_text', )

# views.py
def create_questions_via_form(request):
    question_form = QuestionForm()
    if request.method == 'POST':
        question_form = QuestionForm(request.POST)
        if question_form.is_valid():
            question = question_form.save()
            return HttpResponse('Question Added Successfully!')
    return render(request, 'polls/form_create.html', {'form': question_form})
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Create Questions</title>
</head>
<body>
<div id="content">
    <form method="post" action="">
        {% csrf_token %}
        <div id="question">
            <label>问题名称
                {{ form }}
            </label>
        </div>
        <input type="submit" value="提交">
        <input type="reset" value="重置">
    </form>
</div>

</body>
</html>
```

**************

* Generic View

```python
from django.views.generic import FormView
class QuestionFormView(FormView):
    form_class = QuestionForm
    template_name = 'polls/form_create.html'

    def form_valid(self, form):
        question = form.save()
        return HttpResponse('Question Added Successfully.')
```
