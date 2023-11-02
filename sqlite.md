# Навигация #

|
[Создание и подключениие к БД](#создание-и-подключениие-к-бд--↑) |
[Создание таблиц](#создание-таблиц--↑) |
[INSERT](#insert--↑) |
[SELECT](#select--↑) |
[UPDATE](#update--↑) |
[DELETE](#delete--↑) |
[Агрегирование](#агрегирование--↑) |
[GROUP BY](#group-by--↑) |
[JOIN](#join--↑) |
[UNION](#union--↑) |
[execute](#execute--↑) |


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

`PRIMARY KEY` - означает, что поле  должно содержать уникальные значения.

`AUTOINCREMENT` -   казывает СУБД автоматически увеличивать значение при добавлении новой записи. 

`NOT NULL` - поле должно обязательно содержать данные.

`DEFAULT 1` - запишется значение по умолчанию 1.

При создании табилц, автоматически создается скрытое поле `rowid`, которое присваивает записи порядковый номер. К этому полю можно обращаться в запросах.

# INSERT # [&#8593;](#навигация)

`INSERT` - вставка данных в таблицу.
 
Структура команды:

`INSERT INTO <table_name> (<column_name1>, <column_name2>, ...) VALUES (<value1>, <value2>, …)`

или:

`INSERT INTO <table_name> VALUES (<value1>, <value2>, …)`.

Например, `INSERT INTO users VALUES('Михаил', 1, 19, 1000)` или `INSERT INTO users (name, old, score) VALUES('Федор', 32, 200)`.

# SELECT # [&#8593;](#навигация)

`SELECT` - выборка данных из таблиц.

Структура команды:

`SELECT col1, col2, … FROM <table_name> `.

Например, `SELECT name, old, score FROM users`.

Можно получать данные из БД, выполнив запрос с услвием: `SELECT * FROM users WHERE score < 1000`.

`BETWEEN` - позволяет выбрать записи, значения которых лежат в заданном промежутке. Например, `SELECT * FROM users WHERE score BETWEEN 500 AND 1000` будут выведены записи у которых значение поля `score` находится в диапозоне от 500 до 1000.

Можно использовать все возможные операторы сравнения. 

Возможно выборка данных с использованием составных условий, для этого используются следующие операторы:


`AND` – условное И: exp1 AND exp2. Истинно, если одновременно истинны exp1 и exp2.

`OR` – условное ИЛИ: exp1 OR exp2. Истинно, если истинно exp1 или exp2 или оба выражения.

`NOT` – условное НЕ: NOT exp. Преобразует ложное условие в истинное и, наоборот, истинное – в ложное.

`IN` – вхождение во множество значений: col IN (val1, val2, …)

`NOT IN` – не вхождение во множество значений: col NOT IN (val1, val2, …) 

Например, `SELECT * FROM users WHERE old > 20 AND score < 1000` или

`SELECT * FROM users WHERE score < 1000 ORDER BY old`.

`ORDER BY old` параметр для сортирвки по возрастанию значения поля `old`. Аналогичный результат получим, если написать `ORDER BY old ASC`   

`ORDER BY old DESC` параметр для сортировки по убывнию начения поля `old`.

`LIMIT 5` - параметр для ограничения вывода количества записей.

`OFFSET n` - пропустить первые `n` элементов таблицы.

Реализация запросов на python:

```
import sqlite3 as sq
 
with sq.connect("saper.db") as con:
    cur = con.cursor()
 
    cur.execute("SELECT * FROM users WHERE score > 100 ORDER BY score DESC LIMIT 5")
    result = cur.fetchall()
    print(result)
```

`fetchall` - метод для получения результатов отбора SQL-запроса.

`fetchmany(size)` – возвращает число записей не более size.

`fetchone()` – возвращает первую запись. 

# UPDATE # [&#8593;](#навигация)


`UPDATE` – изменение данных в записях;

Структура команды - `UPDATE имя_таблицы SET имя_столбца = новое_значение WHERE условие `

`UPDATE users SET score = 0` - записать во все строки поля `score` - нули.

Для команды `UPDATE` применимы все фильтры, рассмотренные ранее.

# DELETE # [&#8593;](#навигация)

`DELETE` – удаление записей из таблицы. 

Структура команды - `DELETE FROM имя_таблицы WHERE условие `.

Например, `DELETE FROM users WHERE rowid IN(2, 5)` - удалит записи в таблице у которых `rowid = 2` и `rowid = 5`.

# Агрегирование # [&#8593;](#навигация)

`count()` – подсчет числа записей;

Например, `SELECT count() as count FROM games WHERE user_id = 1`

`sum()` – подсчет суммы указанного поля по всем записям выборки;

`avg()` – вычисление среднего арифметического указанного поля;

`min()` – нахождение минимального значения для указанного поля;

`max()` – нахождение максимального значения для указанного поля. 

`DISTINCT` - находдение уникальных значений.

# GROUP BY # [&#8593;](#навигация)

Язык SQL позволяет вызывать агрегирующие функции не для всех записей в выборке, а локально для указанных групп. Например, мы хотим на основе нашей таблицы games создать список ТОП лучших игроков. Для этого нужно вызвать функции sum для каждого уникального user_id. 

```
SELECT user_id, sum(score) as sum 
FROM games
WHERE score > 200
GROUP BY user_id
ORDER BY sum DESC
```

# JOIN # [&#8593;](#навигация)

Структура команды - `JOIN <таблица> ON <условие связывания> `. 

Используется для объединения данных из разных таблиц для формирования сводного отчета.

Объединяет таблицы, если есть данные в обоих таблицах.

```
SELECT name, sex, games.score FROM games
JOIN users ON games.user_id = users.rowid
```

`LEFT JOIN` - позволяет получить все данные из основной таблицы и присоеденить данные из второй таблицы, если они там есть.

```
SELECT name, sex, games.score FROM games
LEFT JOIN users ON games.user_id = users.rowid
```

# UNION # [&#8593;](#навигация)

Оператор позволяет объеденить все уникальные записи в один отчет.

```
SELECT score, `from` FROM tab1
UNION SELECT val, type FROM tab2
```

# execute # [&#8593;](#навигация)

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



`fetchall()` – возвращает число записей в виде упорядоченного списка;

`fetchmany(size)` – возвращает число записей не более size;

`fetchone()` – возвращает первую запись. INSERT INTO cars (car_id, model, price)
