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

|
[ORM](#orm--↑) |
[Миграции](#миграции--↑) |
[CRUD](#crud--↑) |
[create](#create--↑) |
[read](#read--↑) |
[.filter](#filter--↑) |
[Сортировка](#сортировка--↑) |
[update](#update--↑) |
[delete](#delete--↑) |
[slug](#slug--↑) |
[Пользовательский менеджер модели](#пользовательский-менеджер-модели--↑) |

|
[Типы связей между моделями](#типы-связей-между-моделями--↑) |
[Many-To-One](#many-to-one--↑) |
[ORM команды для Many-To-One](#orm-команды-для-many-to-one--↑) |
[Отображение постов по рубрикам](#отображение-постов-по-рубрикам--↑) |
[Many-To-Many](#many-to-many--↑) |
[Добавление тегов на сайт](#добавление-тегов-на-сайт--↑) |


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

# ORM # [&#8593;](#навигация)

При работе с Django разработчику не нужно беспокоиться о подключении к БД и ее закрытию, когда пользователь покидает сайт. Необходимо лишь через модель взаимодействия выполнять команды API интерфейса, записывать, считывать и обновлять данные.

По умолчанию Django сконфигурирован для работы с БД `SQLite`. Текущую настройку БД можно посмотреть в файле `settings.py` пакета конфигурации: 

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

Чтобы работать с таблицами, нужно обьявить класс с нужными полями. Это делается в файле `app_name/models.py`:

```
class artile(models.Model):
    title = models.CharField(max_length=255)
    content = models.TextField(blank=True)
    time_create = models.DateTimeField(auto_now_add=True)
    time_update = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(default=True)
```

# Миграции # [&#8593;](#навигация)

Чтобы таблица появилась в БД объявить ее модель - не достаточно. Для этого необходимо создать файл миграции. 

При обновлении модели, файл миграции необходимо создать заново. Обновленный файл миграции будет содержать лишь новые изменения. Это можно воспринимать как контроль версий.

Чтобы создать миграцию необходимо выполнить команду: 

`python3 manage.py makemigrations`

После выполнения команды в каталоге `app_name/migrations` появится новый файл с именем `0001_initial.py` - это файл миграции.

Чтобы посмотреть какой sql запрос будет выполнен при использовании файла миграции, необходимо ввести команду:

`python3 manage.py sqlmigrate app_name 0001`

Чтобы выполнить миграцию, нужно выполнить команду:

`python3 manage.py migrate`

Теперь в БД создалось 11 таблиц. Среди которых будет  `app_name_artile`. 

# CRUD # [&#8593;](#навигация)

Класс модели - определяет структуры таблицы БД. Каждый экземпляр класса модели - отдельная запись.

Чтобы просмотреть какие sql запросы были выполнены:

```
from django.db import connection
connection.queries
```

Чтобы просмотреть последний выполненый запрос: `connection.queries[-1]`

Перейдем в консоль django: `python manage.py shell`.

Для работы с объектами класса в консоли, необходимо подключить соответствующий класс:

`from app_name.models import artile`

Для более удобной работы с консолью можно установить дополнительный модуль в виртуальное окружение и расширения для джанго:

```
pip install ipython
pip install django-extensions
```

Далее в файле `settings.py` в коллекцию `installed_apps` добавить запись `django-extensions`.

И выполнить команду:

```
python3 manage.py shell_plus --print-sql
```

Оболочка `shell_plus` автоматически импотирует все классы моделей. Флаг `--print-sql` позволяет автоматически выводить выполненые sql запросы.

# create # [&#8593;](#навигация)


`artile(title='Sheldon Cooper', content='bio')`

Cоздание экземпляра класса не означает фактическое занесение его в таблицу. 

Запишем переменную, которая будет ссылться на созданный экземпляр класса:

`a1= _`

Чтобы произвести запись в таблицу, нужно выполнить команду:

`a1.save()`

Теперь запись была записана в БД. Можно обращаться через консоль джанго к объектам класса, например: `a1.title`.

Добавим записей в таблицу:

```
a2 = artile(title='Melisa Cooper', content='Melisa bio')
a2.save()
```

Атрибут класса `objects` модели ссылается на менеджер записей, соответственно мы можем произвести запись в в таблицу следующей командой:

```
artile.objects.create (title = 'Govard Volovez', content='abracadabra')
```

# read # [&#8593;](#навигация)

`artile.objects.all()` - позволяет получить все записи в формате:

```
<QuerySet [<artile: artile object (1)>, <artile: artile object (2)>, <artile: artile object (3)>]>
```

Чтобы получить в результате выполнения этой команды, например, значение поля `title`, а не идентификатор записи, необходимо в `models.py` нужного приложения добавить метод:

```
def __str__(self):
    return self.title
```

и перезагрузить оболочку shell-plus. Теперь при выполнении команды для вывода всех записей результат будет следующим:

```
<QuerySet [<artile: Sheldon Cooper>, <artile: Melisa Cooper>, <artile: Govard Volovez>]>
```

Чтобы вывести, например первые 3 записи, воспользуемся срезом:

```
r = artile.objects.all()[:3]
```

Так же для вывода записей можно использовать цикл `for`:

```
rs = artile.objects.all()

for r in rs:
    print(r)
```

# .filter # [&#8593;](#навигация)

Так же возможно использование фильтров для вывода записей:

```
artile.objects.filter(title = 'Sheldon Cooper')
```

Для использования в фильтрах операторов сравнения в джанго используется следующий синтаксис (lookups):

`__gte` – сравнение больше или равно (>=);

`__gt` – сравнение больше (>);

`__lte` – сравнение меньше или равно (<=);

`__lt` – сравнение меньше (<). 

`__contains` - проверка на содержание с учетом регистра.

`__icontains` - проверка на содержание без учета регистра.

https://docs.djangoproject.com/en/4.2/ref/models/querysets/#field-lookups

Например, чтобы вывести все записи, у которых id > 2, нужно:

```
artile.objects.filter(pk__gt = 2)
```

Чтобы вывести записи, поле `title` которых содержит определенный фрагмент без учета регистра, необходимо:

```
artile.objects.filter(title__icontains = "pEr")
```

Чтобы вывести список записей с определенным id:

```
artile.objects.filter(pk__in = [1,3])
```

Также используя фильтр, можно передавать несколько параметров:

```
artile.objects.filter(pk__in = [1, 3], title__contains = 'vez')
```

На уровне sql такой запрос будет следующим:

```
Out[19]: SELECT "app_name_artile"."id",
       "app_name_artile"."title",
       "app_name_artile"."content",
       "app_name_artile"."time_create",
       "app_name_artile"."time_update",
       "app_name_artile"."is_published"
  FROM "app_name_artile"
 WHERE ("app_name_artile"."id" IN (1, 3) AND "app_name_artile"."title" LIKE '%vez%' ESCAPE '\')
 LIMIT 21
```

Чтобы исключить записи из вывода по каким-то параметрам необходимо использовать `exclude`. Например, исключим из вывода запись с id = 2:

```
artile.objects.exclude(pk = 2)
```

Метод `.filter` возвращает список в качестве резултата своей работы. Существует метод `.get` который возращает ТОЛЬКО ОДНУ запись.


# Сортировка # [&#8593;](#навигация)

Чтобы выполнить сортировку при выводе записей, необходимо использовать метод `.order_by`. Например,:

```
artile.objects.filter(pk__gte = 2 ).order_by("title")
```

Также есть возможность реализации сортировки по умолчанию, для этого в `models.py` необходимо описать класс `Meta` с необходимыми атрибутами. Например, сделаем по умолчанию сортировку в обратном порядке по значению поля `time_create`.

```
class Meta():
        ordering = ['-time_create']
        indexes = [
            models.Index(fields=['-time_create']),
        ]
```

https://docs.djangoproject.com/en/4.2/ref/models/options/

# update # [&#8593;](#навигация)

Чтобы внести изменения в какую-любо запись, необходимо сперва записать нужную запись с помощью метода `.get` в переменную и обращаясь к нужному полю вносить изменения следующим образом:

```
u = artile.objects.get(pk=2)
u.title = "Missi" 
u.save()
```

Чтобы сделать изменения всех строк конкретного поля нужно:

```
artile.objects.update(is_published=0)
```

Чтобы сделать изменения нескольких строк конкретного поля, нужно использовать метод `filter`:

```
artile.objects.filter(pk__lte=2).update(is_published=1)
```

# delete # [&#8593;](#навигация)

Например, удалим все записи с индексом больше 1.

```
artile.objects.filter(pk__gt = 1).delete()
```

# slug # [&#8593;](#навигация)

Внесем изменения в `views.py` в метод `show_post`:

```
from django.shortcuts import render, redirect, get_object_or_404
from .models import artile

def show_post(request, post_id):
    post = get_object_or_404(artile, pk=post_id)

    data = {
        'title': post.title,
        'menu': menu,
        'post': post,
        'cat_selected': 1,
    }

    return render(request, 'app_name/post.html', context=data)
```

`get_object_or_404` - возвращает запись из модели или генирирует исключение.

Далее сделаем шаблон `post.html` в котором будет отображаться запись из бд:

```
{% extends 'base.html' %}
 
{% block content %}
<h1>{{post.title}}</h1>
 
<!-- ЕСЛИ ЕСТЬ ИЗОБРАЖЕНИЕ - ВСТАВИТЬ -->
{% if post.photo %}
<p ><img class="img-article-left" src="{{post.photo.url}}"></p>
{% endif %}
 
<!-- linebreaks используется для переноса строк при отображении в браузере -->
{{post.content|linebreaks}}
{% endblock %}
```

Теперь перейдя по ссылке `http://127.0.0.1:8000/post/1/` увидем вывод первой записи из таблицы.

Сделаем отображение записей по слагу. Для этого в модели `artile` добавим поле `slug`.

```
slug = models.SlugField(max_length=255, db_index=True, blank=True, default='')
```

После внесения изменений в модель, нужно выполнить и применить миграцию:

```
python3 manage.py makemigrations
python2 manage.py migrate 
```

Поле `slug` добавлено. Теперь перейдем в оболочку `shell-plus` и внесем изменение в это поле:

```
for w in artile.objects.all():
     w.slug = 'slug-'+str(w.pk)
     w.save()
```

Изменим снова модель:

```
slug = models.SlugField(max_length=255, db_index=True, unique=True)
```

Выполним и применим миграцию. 

Теперь нужно сделать отображение статей по слагу, для этого в файле `urls.py` изменим:

```
path('post/<slug:post_slug>/', views.show_post, name='post'),
```

И снова изменим функцию представления:

```
def show_post(request, post_slug):
    post = get_object_or_404(artile, slug=post_slug)

    data = {
        'title': post.title,
        'menu': menu,
        'post': post,
        'cat_selected': 1,
    }

    return render(request, 'app_name/post.html', context=data)
```

Теперь, перейдя по адресу `http://127.0.0.1:8000/post/slug-1/` попадем на отображение первой записи в таблице `artile`.

Пропишем в `models.py` метод, который будет формирвоать слаги по полю `title`:

```
from django.urls import reverse

def get_absolute_url(self):
    return reverse('post', kwargs={'post_slug': self.slug})
```

Теперь изменим вывод ссылок в шаблоне `index.html`:

```
<a href="{{ p.get_absolute_url }}">Читать</a>
```

Также изменим функцию представления `index` в `views.py`:

```
def index(request):
    posts = artile.objects.filter(is_published = 1)
    return render(request, 'app_name/index.html', { 'menu' : menu, 'posts' : posts,})
```

# Пользовательский менеджер модели # [&#8593;](#навигация)

В `models.py` опишем новый класс для менеджера:

```
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_published = 1)
```

 и объвим этот менеджер внутри класса модели `artile`:

 ```
 published = PublishedManager()
 ```

Внесем изменения в функцию представляения `index`: 

```
def index(request):
    posts = artile.published.all()
    return render(request, 'app_name/index.html', { 'menu' : menu, 'posts' : posts,})
```

Теперь используя менеджер `published` на главную страницу выводятся только те записи, которые имеет значение `is_published = 1`.

Определим осмысленные именна вместо 1 и 0. В `models.py` внутри модели `artile` опишем новым класс `status`:

```
class artile(models.Model):
    class Status(models.IntegerChoices):
        DRAFT = 0, 'Черновик'
        PUBLISHED = 1, 'Опубликовано'

    title = models.CharField(max_length=255)
    slug = models.SlugField(max_length=255, db_index=True, unique=True)
    content = models.TextField(blank=True)
    time_create = models.DateTimeField(auto_now_add=True)
    time_update = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(choices=Status.choices, default=Status.DRAFT)

    objects = models.Manager()
    published = PublishedManager()
```

Теперь запросы к модели можно выполнять следующим образом. Например, обновим поле `is_published` для всех записей:

```
w = artile.objects.all()
w.update(is_published=artile.Status.DRAFT)
```

# Типы связей между моделями # [&#8593;](#навигация)

`ForeignKey` – для связей Many to One (многие к одному);

`ManyToManyField` – для связей Many to Many (многие ко многим);

`OneToOneField` – для связей One to One (один к одному). 

# Many-To-One # [&#8593;](#навигация)

Класс `ForeignKey` принимает два обязательных аргумента:

1. `to` – ссылка или строка класса первичной модели, с которой происходит связывание (в нашем случае это класс Category – модели для категорий);

2. `on_delete` – определяет тип ограничения при удалении внешней записи (в нашем примере – это удаление из первичной таблицы Category). 

Значения параметра `on_delete`:

`models.CASCADE` – удаление всех записей из вторичной модели (например, artile), связанных с удаляемой категорией;

`models.PROTECT` – запрещает удаление записи из первичной модели, если она используется во вторичной (выдает исключение);

`models.SET_NULL` – при удалении записи первичной модели (Category) устанавливает значение foreign key в NULL у соответствующих записей вторичной модели (Women);
models.SET_DEFAULT – то же самое, что и SET_NULL, только вместо значения NULL устанавливает значение по умолчанию;

`models.SET() `– то же самое, только устанавливает пользовательское значение;

`models.DO_NOTHING` – удаление записи в первичной модели (Category) не вызывает никаких действий у вторичных.

Реализуем связь многие к одному.

Определим модель для категорий статей (первичная модель):

 ```
 class Category(models.Model):
    name = models.CharField(max_length=100, db_index=True)
    slug = models.CharField(max_length=255, unique=True, db_index=True)

    def __str__(self):
        return self.name
 ```

 Во вторичную модель (`artile`) добавим поле `cat`, которое указывает на то, что оно являяется внешним ключом и ссылается на конкретную запись из модели `Category`:

 ```
 cat = models.ForeignKey('Category', on_delete=models.PROTECT, null=true)
 ```

 Выполним миграцию:

 ```
 python3 manage.py makemigrations
 python3 manage.py migrate
 ```

 Теперь в таблице `artile` добавилось поле `cat_id`, которое является внешним ключом.

 Добавим категории, используя `shell_plus`:

 ```
 Category.objects.create(name='Retro-gaming', slug='retro-gaming')
 Category.objects.create(name='books', slug='books')
 Category.objects.create(name='Modular synth', slug='modular-synth')
 ```

 Присвоим для примера всем записям из таблицы `artile` значение для поля `cat_id` равное 1:

```
w_list = artile.objects.all()
w_list.update(cat_id=1)
```

Уберем значение `null=true` для поля `cat` в модели `artile`.

Выполним снова миграцию (выбрать `Ignore for now`) и применем ее.

# ORM команды для Many-To-One # [&#8593;](#навигация)

Перейдем в шел плюс и выполним команду, которая позволяет вывести все статьи с определнной категорией, например 1:

```
c = Category.objects.get(pk=1)
c.artile_set.all()
```

Также, для большего удобства, мы можем изменить имя атрибута обратного свертывания (`artile_set`) на что-то более осмысленное, для этого в классе `ForeignKey` вторичной модели, необходимо прописать `related_name='posts`:

```
cat = models.ForeignKey('Category',on_delete=models.PROTECT, related_name='posts')
```

Теперь, вывести посты, связанные с какой-либо категорией можно следующим образом:

```
c = Category.objects.get(pk=1)
c.posts.all()
```

С атритом обратного свертывания также можно использовать и фильтры, например, выведем только те статьи, у которых значение поля `is_published = 1`:

```
c.posts.filter(is_published=1)
```

Также, обращаясь к первичной модели, мы можем обратится к полю `slug` вторичной модели, например: 

```
artile.objects.filter(cat__slug='retro-gaming')
```

Т.е. обращение к нужному полю осуществляется аналогично использовонию люкапов (`lookups`), через `__`.

# Отображение постов по рубрикам # [&#8593;](#навигация)

В `urls.py` внесем изменения:

```
path('category/<slug:cat_slug>/', views.show_category, name='category'),
```

В `views.py` изменим функцию `show_category`:

```
def show_category(request, cat_slug):
    category = get_object_or_404(Category, slug=cat_slug)
    posts = artile.published.filter(cat_id=category.pk)

    data = {
        'title': f'Рубрика: {category.name}',
        'menu': menu,
        'posts': posts,
        'cat_selected': category.pk,
    }
 
    return render(request, 'app_name/index.html', context=data)
```

Далее, внесем изменения в файл `templatetags/app_name_tags.py`:

```
@register.inclusion_tag('app_name/list_categories.html')
def show_categories(cat_selected_id=0):
    cats = Category.objects.all()
    return {"cats": cats, "cat_selected": cat_selected_id}
```

В шаблоне `list_categories.html`:

```
{% for cat in cats %}
<li><a href="{{ cat.get_absolute_url }}">{{cat.name}}</a></li>
{% endfor %}
```

`models.py`

```
class Category(models.Model):
    name = models.CharField(max_length=100, db_index=True)
    slug = models.CharField(max_length=255, unique=True, db_index=True)

    def __str__(self):
        return self.name
    
    def get_absolute_url(self):
        return reverse('category', kwargs={'cat_slug':self.slug})
```

Теперь статьи выводятся по категориям.

# Many-To-Many # [&#8593;](#навигация)

Примером связи многие ко многим является тегирование, это значит, что с одним постом можно связать сразу несколько разных тегов и, наоброт, с отдельными тегами – несколько разных постов.

Добавим модель для тегов:

`models.py`

```
class TagPost(models.Model):
    tag = models.CharField(max_length=100, db_index=True)
    slug = models.SlugField(max_length=255, unique=True, db_index=True)

    def __str__(self):
        return self.tag
```

И в модель `artile` пропишем поле `tags` для реализации связи многие ко многим:

```
tags = models.ManyToManyField('TagPost', blank=True, related_name='tags')
```

Выполним и применим миграцию.

Тэги храняться в таблице `app_name_tagpost`.

Перейдем в оболочку джанго и наполним таблицу данными:

```
TagPost.objects.create(tag='geek', slug='geek')
TagPost.objects.create(tag='nerd', slug='nerd')
и т.д
```

Связь многие ко многим будет определяться содержимым вспомогательной таблицы `app_name_artile_tags`.

Добавим связи между тегами и постами:

```
a = artile.objects.get(pk=1) #прочитаем первую запись из таблицы с постами -sheldon cooper

tag_br = TagPost.objects.all()[1] #присвоим переменной значениме тега =  nerd

tag_o, tag_v = TagPost.objects.filter(id__in=[3, 5]) #physic, flash

a.tags.set([tag_br, tag_o, tag_v]) #присвоить список тегов
```

Теперь, в таблице `app_name_artile_tags` появились связи. 

 `a.tags.add(tag_br)` - добавить один тег `tag_br`.

 `a.tags.remove(tag_o)` - удалить тег `tag_o`.

 `a.tags.all()` - получить все теги для текущей записи.

 `tag_br.tags.all()` - получить по тегу все посты, которые с ним связаны.

Теперь, когда реализованы категории статей и теги, добавление новых статей будет выполняться:

```
w = artile.objects.create(title='Пенни Ховстедер', slug='penni-hofsteder', cat_id=2)
w.tags.set([tag_br, tag_v])
```

# Добавление тегов на сайт # [&#8593;](#навигация)
