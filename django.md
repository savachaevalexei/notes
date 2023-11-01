# Навигация #

|
[Создание приложения](#создание-приложения--↑) |
[Создание представления](#создание-представления--↑) |
[Маршрутизация](#маршрутизация--↑) |
[Динамические URL](#динамические-url--↑) |
[Обработка исключений](#обработка-исключений--↑) |
[redirect](#redirect--↑) |
[Шаблоны](#шаблоны--↑) |
[Передача данных в шаблоны](#передача-данных-в-шаблоны--↑) |
[Стандартные фильтры шаблонов](#стандартные-фильтры-шаблонов--↑) |

|
[for](#тэг-for--↑) |
[if](#тэг-if--↑) |
[url](#тэг-url--↑) |
[extends](#extends--↑) |
[include](#include--↑) |
[simple tags](#simple-tags--↑) |
[inclusion tags](#inclusion-tags--↑) |

`django-admin` - список команд ядра.

`django-admin startproject project_name` - создать новый проект c названием `project_name`.

`python3 manage.py runserver`- запустить тестовый сервер.

По умолчанию сервер запускается на 8000 порту. Чтобы это изменить необходимо использовать команду:

`python3 manage.py runserver 4000`

Каждая самостоятельная часть сайта должна представляться в виде отдельного приложения. Приложения должны быть независимы.

# Создание приложения # [&#8593;](#навигация)

`python3 manage.py startapp app_name` - создать новое приложение с названием `app_name`.

В папке `project_name` есть файл `settings.py` внутри которого есть коллекция с названием `INSTALLED_APPS`. В эту коллекцию необходимо добавить все созданные `app_name`. 

Так как django применяет автоматическое форматирование названия приложения, необходимо скопировать корректное представление названия приложения из файла `apps.py`, который находится в папке `app_name` и произвести вставку в коллекцию.

Например, для приложения с названием `app_name` запись будет выглядеть следующим образом:

```
'app_name.apps.AppNameConfig',
```

# Создание представления # [&#8593;](#навигация)

В папке с приложением `app_name` есть файл с названием `views.py`, необходимый для формирования представления страницы сайта.

Добавим функцию для формирования запроса.

```
from django.http import HttpResponse
from django.shortcuts import render

def index(request):
    return HttpResponse("Page")
```

Параметр функции `request` - ссылка на специальный классс `HttpRequest` и содержит информацию о запросе, сессиях, куках и т.д.

# Маршрутизация # [&#8593;](#навигация)

Созданную функцию с названием `index` необходимо связать с соответствующим url адресом. 

В папке проекта `project_name` имеется файл `urls.py`, в котором описываются различные маршруты.

В коллекции `urlpatterns` уже имеется один маршрут `path('admin/', admin.site.urls),` который предоставляет доступ к админ панели.

Добавим маршрут. Файл будет выглядеть следующим образом:

```
from django.contrib import admin
from django.urls import path
from app_name import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('app_name/', views.index),
]
```

Теперь если запустить тестовый сервер и перейти по адресу `http://127.0.0.1:8000/app_name/` мы увидим созданное ранее представление приложения `app_name`. 

Чтобы получить представление, простой перейдя по ссылке `http://127.0.0.1:8000`, необходимо в `urls.py` прописать маршрут следующим образом:

```
path('', views.index),
```

Если использовать для маршрутизации файл `urls.py` из папки проекта `project_name` - это нарушает конвенцию о независимости приложения Django.

Чтобы исправить это внесем следующие изменения файла `urls.py` из папки `project_name`:

```
from django.contrib import admin
from django.urls import path, include
from app_name import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_name.urls')),
]
```

Затем, в папке с приложением `app_name` создадим новый файл с названием `urls.py` и внесем в него следующее:

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index),
]
```

# Динамические URL # [&#8593;](#навигация)

Пропишем новый маршрут:

```
path('cats/<int:cat_id>/', views.categories)
```

и добавим функцию представления:

```
def categories(request, cat_id):
    return HttpResponse(f"Cat: {cat_id}")   
```

Здесь используется ковертер `int:` для получения числового значения. Также могут использоваться str(строка), slug(набор латинских символов), uuid(цифры, малые латинскте буквы и дефис), path(любая не пустая строка).

Теперь если в браузере перейти по адресу `http://127.0.0.1:8000/cats/1/`, получим вывод `Cat: 1`. Вместо `1` в адрес можно подставить любое чилсо.

# Обработка исключений # [&#8593;](#навигация)

Если перейти на несуществующую страницу, то мы увидим окно отладчика django  браузере. Чтобы это исключить, в файле `settings.py` пакета конфигурации нужно  отредактировать поля `DEBUG`, `ALLOWED_HOSTS` установить следующие значения:

```
DEBUG = False

ALLOWED_HOSTS = ['127.0.0.1']
```

После этого, в браузере будет отоброжаться что был использован неверный запрос к сайту.

Чтобы изменить выводимую информацию при переходе по несуществующей ссылке, необхоимо в файле `urls.py` пакета конфигурации прописать специальный обработчик.

```
from app_name.views import page_not_found

handler404 = page_not_found
```

И далее определить эту функцию в файле `views.py` в папке приложения `app_name`:

```
from django.http import HttpResponse, HttpResponseNotFound

def page_not_found(request, exception):
    return HttpResponseNotFound("СТРАННИЦА НЕ НАЙДЕНА")
```

Таким образом, обработчик `handler404` будет вызываться каждый раз, при возникновении ошибки 404.

# redirect # [&#8593;](#навигация)

`301` - страница перемещена на другой постоянный адрес.

`302` - страница временно перемещена на другой адрес.

Для организации перенаправлений, может использоваться функция `redirect`, например, если передаваемый в url `cat_id` имеет значение больше 3, перенаправим на главную страницу:

```
def categories(request, cat_id):
    if cat_id > 3:
        return redirect('/')
    return HttpResponse(f"Cat: {cat_id}")   
```

В скобках можно указывать как url адрес страницы, так и имя маршута, для этого в urls.py необходимо внести следующие изменения:

```
path('', views.index, name = 'home')
```

А функция redirect будет иметь вид:

```
return redirect('home')
```

# Шаблоны # [&#8593;](#навигация)

Для подключения шаблонов в файле `views.py` необходимо подключить библеотеку `from django.template.loader import render_to_string`. Изменим имеюющуюся функцию `index`:

```
def index(request):
    t = render_to_string('app_name/index.html')
    return HttpResponse(t)
```

В папке приложения `app_name` создадим папку `app_name` а в ней папку`templates` и добавим html файл `index.html`. 

Можно использовать `render` для возвращения представляения, при этом функция `index` будет иметь вид:

```
def index(request):
    return render(request, 'app_name/index.html')
```

Добавим новый маршрут в файле `app_name/urls.py`:

```
path('about/', views.about, name='about')
```

Добавим функцию для возвращения представления в файле `app_name/views.py`:

```
def about(request):
    return render(request, 'app_name/about.html')

```

И добавим соответствующий шаблон для этого предствления в папку `app_name/templates/app_name/about.html`.

В файле `project_name/settings.py` есть коллекция `TEMPLATES`, которая отвечает за настройку шаблонизатора django. Элемент `'APP_DIRS': True,` отвечает за то, чтобы по умолчанию шаблонизатор искал нужный шаблон в папке `templates`.

# Передача данных в шаблоны # [&#8593;](#навигация)

В шаблоны `about.html` и `index.html` внутри тегов `<title></title>` добавим конструкцию `{{ title }}`:

```
<title>{{ title }}</title>
```

Изменим функции представления:

```
def index(request):
    data = {'title' : 'Главная страница'}
    return render(request, 'app_name/index.html', data)

def about(request):
    data = {'title' : 'О сайте'}
    return render(request, 'app_name/about.html', data)
```

Добавим список `dict`:

```
dict = ['menu', 'trash']

def index(request):
    data = {'title' : 'Главная страница', 'dict' : dict}
    return render(request, 'app_name/index.html', data)
```

Внутри шаблона, чтобы обратится к какому то элементу списка, словаря, параметру класса и т.д используется `.` (точка):

```
<h1>{{ dict.1 }}</h1>
```

# Стандартные фильтры шаблонов # [&#8593;](#навигация)

Добавим словарь в `app_name/views.py`:

```
data = {
    'title' : 'главная страница',
    'int' : 28,
    'lst' : [ 1, 2, 'abc', True],
    'set' : {1, 2, 3, 2, 5},
    'dict' : {'key1' : 'value1', 'key2' : 'value2'},          
}
```

И на странице `index.html` испытаем разные фильтры:

`{{ int|add:"50" }}` - прибавить к элементу словаря с ключом `int` значение `50`;

`{{ title|capfirst }}` - сделать первую букву заглавной;

`{{ title|lower }}` - сделать все буквы маленькими;

`{{ title|upper }}` - сделать все буквы большими;

`{{ title|cut:"а"|cut:" " }}` - удалить из строки все символы `a` и пробелы;

`{{ main_title|default:"Без заголовка" }}` - если переменной `main_title` нет, то будет применено значение по умолчанию;

`{{ title|join:" | " }}` - разделить все элементы;

`{{ "Home page"|slugify }}` - преобразовать строку в слаг.

Больше о фильтрах:

https://docs.djangoproject.com/en/4.2/ref/templates/builtins/

# Тэг for # [&#8593;](#навигация)

Реализуем цикл `for`, чтобы вывести данные из data_db. 

В `app_name/views.py` внесем изменения:

```
data_db = [
    {'id' : '1', 'title' : 'Шелдон', 'content' : 'Биография Шелдона', 'is_published' : True},
    {'id' : '2', 'title' : 'Леонард', 'content' : 'Биография Леонарда', 'is_published' : False},  
    {'id' : '3', 'title' : 'Раджеш', 'content' : 'Биография Раджеша', 'is_published' : False},  
    {'id' : '4', 'title' : 'Говард', 'content' : 'Биография Говарда', 'is_published' : True},  
    {'id' : '5', 'title' : 'Пенни', 'content' : 'Биография Пенни', 'is_published' : True},          
]

data = {
    'title' : 'главная страница',
    'main_title' : 'kek',
    'int' : 28,
    'lst' : [ 1, 2, 'abc', True],
    'set' : {1, 2, 3, 2, 5},
    'dict' : {'key1' : 'value1', 'key2' : 'value2'},
    'posts' : data_db,          
}

def index(request):
    return render(request, 'app_name/index.html', data)
```

Изменим шаблон `index.html`:

```
<body>

    <ul>
        {% for p in posts %}
        <li>
            <h2>{{ p.title }}</h2>
            <p>{{ p.content }}</p>
            <hr>
        </li>

        {% endfor %}
    </ul>


</body>
```

# Тэг if # [&#8593;](#навигация)

Реализуем конструкцию `if`, таким образом, чтобы горизотнальная черта после последней записи не проставлялась и выведем только те записи, у которых `is_published = True`:

```
    <ul>
        {% for p in posts %}
        {% if p.is_published %}
        <li>
            <h2>{{ p.title }}</h2>
            <p>{{ p.content }}</p>
            {% if not forloop.last %}
            <hr>
            {% endif %}
        </li>
        {% endif %}
        {% endfor %}
    </ul>
```

# Тэг url # [&#8593;](#навигация)

Добавим новый маршрут в `app_name/urls.py`:

```
path('post/<int:post_id/', views.show_post, name='post'),
```

Добавим функцию представления `app_name/views.py`:

```
def show_post(request, post_id):
    return HttpResponseNotFound(f"Текст статьи с id = {post_id}")
```

Добавим ввывод ссылки в шаблон `index.html`:

```
    <ul>
        {% for p in posts %}
        {% if p.is_published %}
        <li>
            <h2>{{ p.title }}</h2>
            <p>{{ p.content }}</p>
            <p><a href="{% url 'post' p.id %}">Читать</a></p>
            {% if not forloop.last %}
            <hr>
            {% endif %}
        </li>
        {% endif %}
        {% endfor %}
    </ul>
```

Добавим меню для сайта. 

Добавим новые маршуты. Файл `app_name/urls.py` будет:

```
from django.urls import path
from . import views


urlpatterns = [
    path('', views.index, name = 'home'),
    path('about/', views.about, name='about'),
    path('addpage/', views.addpage, name ='add_page'),
    path('contact/', views.contact, name='contact'),
    path('login/', views.login, name='login'),
    path('post/<int:post_id>/', views.show_post, name='post'),
]
```

В файле `views.py` добавим список словарей для меню:

```
menu = [
    {'title' : "О сайте", 'url_name' : 'about'}, 
    {'title' : "Добавить статью", 'url_name' : 'add_page'},
    {'title' : "Обратная связь", 'url_name' : 'contact'},
    {'title' : "Войти", 'url_name' : 'login'},
]
```

и функции представления:

```
def addpage(request):
    return HttpResponseNotFound("add page")

def contact(request):
    return HttpResponseNotFound("contact")

def login(request):
    return HttpResponseNotFound("login")
```

В шабон `index.html` добавим вывод меню:

```
<ul>
<li><a href="{% url 'home' %}">Главная</a></li>
{% for m in menu %}
{% if not forloop.last %}<li>{% else %}<li class="last">{% endif %}
         <a href="{% url m.url_name %}">{{m.title}}</a>
</li>
{% endfor %}
</ul>
```

# extends # [&#8593;](#навигация)

В главном каталоге сайта (каталог, внутри которого есть и `project_name` и `app_name`) создадим каталог  `templates` и внутри него файл `base.html` - это будет базовый шаблон для всех страниц сайта.

Добавим нестандартный маршрут к базовому шаблону в файле `settings.py`

```
 'DIRS': [
            BASE_DIR / 'templates',
        ],
```

Шаблон `base.html` - содержит общую информацию для всех страниц сайта:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>

<ul>
    
<li><a href="{% url 'home' %}">Главная</a></li>
{% for m in menu %}
{% if not forloop.last %}<li>{% else %}<li class="last">{% endif %}
    <a href="{% url m.url_name %}">{{m.title}}</a>
</li>
{% endfor %}
</ul>

    {% block content %} {% endblock %}
</body>
</html>
```

Тогда `index.html` будет содержать:

```
{% extends 'base.html' %}

{% block content %} 
    <ul>
        {% for p in posts %}
        {% if p.is_published %}
        <li>
            <h2>{{ p.title }}</h2>
            <p>{{ p.content }}</p>
            <p><a href="{% url 'post' p.id %}">Читать</a></p>
            {% if not forloop.last %}
            <hr>
            {% endif %}
        </li>
        {% endif %}
        {% endfor %}
    </ul>

    {% endblock %}
```

`about.html`  будет содержать:

```
{% extends 'base.html' %}

{% block content %} 
    <h1>{{ title }}</h1>
    {% endblock %}
```

# include # [&#8593;](#навигация)

Допустим, что мы хотим сделать отображение рубрик статей.

В каталоге `app_name/templates/app_name/` создадим подкаталог `includes` и внутри него файл `nav.html` c содержимым:

```
<nav>
    <a href="">Cat_1</a>
    <a href="">Cat_2</a>
    <a href="">Cat_3</a>
</nav>
```

А внутри файла `index.html` внутри блока `{% block content %} ` добавим `{% include 'app_name/includes/nav.html' %}`.

# Статические файлы # [&#8593;](#навигация)

Создадим в каталоге `app_name` каталог `static` с подкаталогом `app_name` - это путь для хранения статических файлов.

Теперь, в каталоге `app_name/static/app_name` создадим подкаталоги:

`css` - каскадные таблицы стилей

`js` - java script файлы

`images` - изображения

В базовом шаблоне необходимо подключить статические файлы. Для этого нужно добаваить:

```
{% load static %}
```

# simple tags # [&#8593;](#навигация)

Создадим простой тэг.

В каталоге `app_name` создадим подкаталог `templatetags` а в нем, файлы `__init__.py`. и `app_name_tags.py`. 

`app_name_tags.py`
```
from django import template
import app_name.views as views

register = template.Library()

@register.simple_tag()
def get_categories():
    return views.cats_db
```

И в файл `app_name/views.py` добавим новую коллекцию:

```
cats_db = [
    {'id' : 1, 'name' : 'Retro gaming'},
    {'id' : 2, 'name' : 'Modular synth'},
    {'id' : 3, 'name' : 'Books'},
]
```

В базовом шаблоне `base.html` необходимо вначале документа прописать:

```
{% load app_name_tags %}
```

И далее в нужном месте документа прописать:

```
{% get_categories as categories %}
{% for cat in categories %}
<li><a href="{% url 'category' cat.id %}">{{cat.name}}</a></li>
{% endfor %}
```

Добавим новый маршрут в `urls.py`:

```
path('category/<int:cat_id>/', views.show_category, name='category'),
```

Опишем представление для нового маршрута в `views.py`:

```
def show_category(request, cat_id):
    return index(request)

```

# inclusion tags # [&#8593;](#навигация)

Включающий тег позволяет дополнительно формировать свой собственный шаблон на основе некоторых данных и возвращать фрагмент HTML-страницы. 

В файле `app_name_tags.py` определиим новую функцию:

```
@register.inclusion_tag('app_name/list_categories.html')
def show_categories():
    cats = views.cats_db
    return {'cats' : cats}
```

В папке `app_name/templates/app_name` создадим новый файл `list_categories.html`:

```
{% for cat in cats %}
<li><a href="{% url 'category' cat.id %}">{{cat.name}}</a></li>
{% endfor %}
```

А в `base.html` в нужном месте прописать:

```
{% show_categories %}
```