# Навигация #

|
[Конструкции внутри шаблона](#конструкции-внутри-шаблона--↑) |
[Экранирование данных](#экранирование-данных--↑) |
[Фильтры](#фильтры--↑) |
[Макросы](#макросы--↑) |
[Загрузчики шаблонов](#загрузчики-шаблонов--↑) |
[include](#include--↑) |
[Наследование](#наследование--↑) |

jinja (дзиндзя) - модуль для обработки шаблонов python (шаблонизатор).

Рассмотрим простой шаблон: 

```
from jinja2 import Template

name = "Alex"

tm = Template("Hello {{ n }}")
msg = tm.render(n = name)

print(msg)

Hello Alex
```

Здесь `tm` - переменная которая хранит внутри шаблон. А переменная `msg` - создает представление на основе шаблона, подставляя нужные значения. 

# Конструкции внутри шаблона # [&#8593;](#навигация)

`{%%}` - спецификатор шаблона. 

`{{}}` - выражение для вставки конструкций python в шаблон.

`{##}` - блок комментариев.

`# ##` - строковый комментарий.

Рассчторим построение шаблона на основе класса:

```
from jinja2 import Template

class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

per = Person('Alex', 27)

tm = Template("Main name is {{ p.name }}. I am {{ p.age }} years old.")
msg = tm.render(p = per)

print(msg)

Main name is Alex. I am 27 years old.
```

Если использовать словарь, как источник данных для шаблона то в общем будет:

```
from jinja2 import Template

per = {'name':'Alex', 
       'age' : 27
       }

tm = Template("Main name is {{ p['name'] }}. I am {{ p['age'] }} years old.")
msg = tm.render(p = per)

print(msg)

Main name is Alex. I am 27 years old.
```

# Экранирование данных # [&#8593;](#навигация)

`{% raw %}` и `{% endrow %}` - используется для экранирования.

```
from jinja2 import Template

per = {'name':'Alex', 
       'age' : 27
       }

tm = Template("{% raw %}Main name is {{ p['name'] }}. I im {{ p['age'] }} old.{% endraw %}")
msg = tm.render(p = per)

print(msg)

Main name is {{ p['name'] }}. I im {{ p['age'] }} old.
```

Реализуем формированние списка ссылок на основе словаря:

```
from jinja2 import Template

pages = [
    {'id' : 1, 'name' : 'home'},
    {'id' : 2, 'name' : 'news'},
    {'id' : 3, 'name' : 'forum'},
    {'id' : 4, 'name' : 'feedback'},
]

link = '''
{% for page in pages -%}
    <a href="page_{{page['id']}}">{{page['name']}}</a>
{% endfor %}
'''

tm = Template(link)
msg = tm.render(pages = pages)

print(msg)
```

# Фильтры # [&#8593;](#навигация)

`sum` - вычисление суммы поля коллекции.

```
from jinja2 import Template

pages = [
    {'id' : 1, 'name' : 'home'},
    {'id' : 2, 'name' : 'news'},
    {'id' : 3, 'name' : 'forum'},
    {'id' : 4, 'name' : 'feedback'},
]

tpl = "Summ id = {{ pages | sum(attribute='id' ) }}"
tm = Template(tpl)
msg = tm.render(pages = pages)

print(msg)

Summ id = 10
```

`max` - поиск набильного значения коллекции.

`min` - поиск наименьшего значения коллекции.

`random` - вернет случайное значение из коллекции.

`replace("o", "O")` - замена символов.

Рассмотрим конструкцию `{% filter %} - {% endfilter %}`:

```
from jinja2 import Template

pages = [
    {'id' : 1, 'name' : 'home'},
    {'id' : 2, 'name' : 'news'},
    {'id' : 3, 'name' : 'forum'},
    {'id' : 4, 'name' : 'feedback'},
]

tpl = '''
{%- for page in pages -%}
{%filter upper %}{{page.name}}{% endfilter %}
{% endfor -%}
'''
tm = Template(tpl)
msg = tm.render(pages = pages)

print(msg)

HOME
NEWS
FORUM
FEEDBACK
```

# Макросы # [&#8593;](#навигация)

`call` - позволяет определять вложенные макросы.

Сформируем макроопределение `list_usres` и передадим ему список `list_of_user`:

```
from jinja2 import Template

persons = [
    {'name' : 'Alex', 'age' : 28, 'weight' : 90.5},
    {'name' : 'Ivan', 'age' : 25, 'weight' : 76.8},
    {'name' : 'Makar', 'age' : 21, 'weight' : 84.2}
]

html = '''
{% macro list_users(list_of_user) -%}
<ul>
{% for u in list_of_user -%}
    <li>{{u.name}}
{%- endfor %}
</ul>
{%- endmacro %}

{{list_users(users)}}

'''

tm = Template(html)
msg = tm.render(users = persons)

print(msg)
```

Добавим в пример выше вложенный макрос:

```
from jinja2 import Template

persons = [
    {'name' : 'Alex', 'age' : 28, 'weight' : 90.5},
    {'name' : 'Ivan', 'age' : 25, 'weight' : 76.8},
    {'name' : 'Makar', 'age' : 21, 'weight' : 84.2}
]

html = '''
{% macro list_users(list_of_user) -%}
<ul>
{% for u in list_of_user -%}
    <li>{{u.name}}{{caller(u)}}
{%- endfor %}
</ul>
{%- endmacro %}
{% call(users) list_users(users) %}
    <ul>
        <li>age: {{users.age}}
        <li> weight: {{users.weight}}
    </ul>
{% endcall -%}

'''

tm = Template(html)
msg = tm.render(users = persons)

print(msg)
```

# Загрузчики шаблонов # [&#8593;](#навигация)

Создадим простой html шаблон:

`main.html`
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
{% for u in list_of_user -%}
    <li>{{u.name}}
{% endfor -%}
</ul>
</body>
</html>
```

Чтобы подставить данные в html шаблон необходимо воспользоваться загрузчиком `FileSystemLoader`, который взаимодействует с файловой системой.

```
from jinja2 import Environment, FileSystemLoader

persons = [
    {'name' : 'Alex', 'age' : 28, 'weight' : 90.5},
    {'name' : 'Ivan', 'age' : 25, 'weight' : 76.8},
    {'name' : 'Makar', 'age' : 21, 'weight' : 84.2}
]

file_loader = FileSystemLoader('templates')
env = Environment(loader = file_loader)

tm = env.get_template('main.html')
msg = tm.render(list_of_user = persons)

print(msg)
```

В `FileSystemLoader` передается каталог в котором хранится нужный шаблон, а в `get_template` передается имя шаблона с расширением.

Типы загрузчиков в  jinja:

`PackageLoader` -  для загрузки шаблонов из пакета.

`DictLoader` - для загрузки шаблонов из словаря.

`FunctionLoader` - для загрузки шаблонов на основе функции.

`PrefixLoader` - загрузчик, использующий словарь для построения подкаталогов.

`ChoiceLoader` - загрузчик, содержащий список других загрузчиков.

`ModuleLoader` - загрузчик для скомпилированных шаблонов.

Реализуем загрузчик на основе `FunctionLoader`:

```
from jinja2 import Environment, FunctionLoader

persons = [
    {'name' : 'Alex', 'age' : 28, 'weight' : 90.5},
    {'name' : 'Ivan', 'age' : 25, 'weight' : 76.8},
    {'name' : 'Makar', 'age' : 21, 'weight' : 84.2}
]
def loadTpl(path):
    if path == "index":
        return '''Имя {{u.name}}, возраст {{u.age}}'''
    else:
        return '''Данные: {{u}}'''
    
file_loader = FunctionLoader(loadTpl)
env = Environment(loader=file_loader)
 
tm = env.get_template('index')
msg = tm.render(u = persons[0])

print(msg)
```

# include # [&#8593;](#навигация)

Часто, шаблоны сайтов разбиваются на отдельные блоки. Например, на footer, main и header.

Изменим шаблон main.html, поделив его на 3 части:

`header.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
```

`footer.html`
```
</body>
</html>
```

`main.html`
```
{% include 'header.html' ignore missing %}

<ul>
{% for u in list_of_user -%}
    <li>{{u.name}}
{% endfor -%}
</ul>

{% include 'footer.html' ignore missing %}
```

В файле `main.html` используется конструкция `{% include 'header.html' ignore missing%}` для подключения других блоков сайта, где `ignore missing` предназначен для игнорирования факта отсутствия подключаемого файла.

Реализуем загрузку шаблона:

```
from jinja2 import Environment, FileSystemLoader

persons = [
    {'name' : 'Alex', 'age' : 28, 'weight' : 90.5},
    {'name' : 'Ivan', 'age' : 25, 'weight' : 76.8},
    {'name' : 'Makar', 'age' : 21, 'weight' : 84.2}
]

file_loader = FileSystemLoader('templates')
env = Environment(loader = file_loader)

tm = env.get_template('main.html')
msg = tm.render(list_of_user = persons)

print(msg)
```

# Наследование # [&#8593;](#навигация)

{% block name %} {% endblock %} - создание именованного блока. Используется для расширения базового шаблона страницы.

Отредактируем файл `main.html`:

```
{% include 'header.html' ignore missing %}

{% block title %}{% endblock %}

{% block content %}{% endblock %}

{% include 'footer.html' ignore missing %}
```

Создадим файл `about.html`, который будет расширять базовый шаблон `main.html`:

```
{% extends 'main.html' %}
 
{% block title%}О сайте{% endblock %}
 
{% block content %}
<h1>О сайте</h1>
<p>Классный сайт, если его доделать.</p>
{% endblock %}
```

Реализуем загрузку шаблона:

```
from jinja2 import Environment, FileSystemLoader

file_loader = FileSystemLoader('templates')
env = Environment(loader = file_loader)

tm = env.get_template('about.html')
msg = tm.render()

print(msg)
```