# Навигация #

|
[Создание и подключениие к БД](#создание-и-подключениие-к-бд--↑) |
[Создание таблиц](#создание-таблиц--↑) |
[executemany](#executemany--↑) |

SQLite — это встраиваемая кроссплатформенная БД, которая поддерживает достаточно полный набор команд SQL. Поэотму синтаксис SQL здесь не рассматривается.

# Создание и подключениие к БД # [&#8593;](#навигация)

Для работы необходимо сделать импорт библиотеки `import sqlite3 as sq`.

`con = sq.connect("saper.db")` - создать подклчюение к БД.

`cur = con.cursor()` - курсос необходим для взаимодействия с БД.

`cur.execute(""" """)` - пустой запрос к БД.

`con.close()` - закрыть подключение к БД.

Использование контекст-менеджера `with sq.connect("saper.db") as con:` предотвращает потерю данных и гарантирует автоматическое закрытие БД.

Создадим файл `main.py` со следующим содержимым:

```
import sqlite3 as sq

with sq.connect("saper.db") as con:
    cur = con.cursor()

    cur.execute("""
                """)
```

# Создание таблиц # [&#8593;](#навигация)

Создадим таблицу `users`:

```
import sqlite3 as sq

with sq.connect("saper.db") as con:
    cur = con.cursor()

    cur.execute("""DROP TABLE IF EXISTS users""")
    cur.execute("""CREATE TABLE IF NOT EXISTS users (
                user_id INTEGER PRIMARY KEY AUTOINCREMENT ,
                name TEXT NOT NULL,
                sex INTEGER NOT NULL DEFAULT 1,
                old INTEGER,
                score INTEGER
    )""")
```

`DROP TABLE IF EXISTS users` - если существует, удалить таблицу `users`.

`CREATE TABLE IF NOT EXISTS users ` - если не существует, создать таблицу `users`.

При создании табилц, автоматически создается скрытое поле `rowid`, которое присваивает записи порядковый номер. К этому полю можно обращаться в запросах.

# executemany # [&#8593;](#навигация)

```
import sqlite3 as sq
 
cars = [
    ('Audi', 52642),
    ('Mercedes', 57127),
    ('Skoda', 9000),
    ('Volvo', 29000),
    ('Bentley', 350000)
]

import sqlite3 as sq
 
with sq.connect("cars.db") as con:
    cur = con.cursor()
 
    cur.execute("""CREATE TABLE IF NOT EXISTS cars (
        car_id INTEGER PRIMARY KEY AUTOINCREMENT,
        model TEXT,
        price INTEGER
    )""")

    cur.executemany("INSERT INTO cars VALUES(NULL, ?, ?)", cars)
```

`cur.executemany("INSERT INTO cars VALUES(NULL, ?, ?)", cars)` - позволяет вставить все записи из коллекции `cars` в таблицу `cars`.

Вставку дданных из коллекции также можно реализовать в цикле:

```
for car in cars:
    cur.execute("INSERT INTO cars VALUES(NULL, ?, ?)", car)
```

Если нужно выполнить несколько отдельных комманд, то можно использовать `executescript`:

```
cur.executescript("""DELETE FROM cars WHERE model LIKE 'A%';
    UPDATE cars SET price = price+1000
""")
```

`cur.fetchall()` – возвращает число записей в виде упорядоченного списка;

`cur.fetchmany(size)` – возвращает число записей не более size;

`cur.fetchone()` – возвращает первую запись. INSERT INTO cars (car_id, model, price)
