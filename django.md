# Навигация #

|
[Создание приложения](#создание-приложения--↑) |
[Создание представления](#создание-представления--↑) |
[Маршрутизация](#маршрутизация--↑) |
[Динамические URL](#динамические-url--↑) |

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