# Пример TDD на Pytest

## Как я должен тестировать свой софт?

Три рекомендации, которые помогут вам написать ценные тесты:

1. Тесты должны сообщать вам об ожидаемом поведении тестируемого модуля. Поэтому желательно, чтобы они были короткими и по существу. структура GIVEN, WHEN, THEN может помочь в этом:

   * GIVEN (Дано) - каковы начальные условия для теста?
   * WHEN (Когда)- что происходит, что необходимо проверить?
   * THEN (Тогда)- какова ожидаемая реакция?

   Таким образом, вы должны подготовить свою среду для тестирования, запустить тесты и, в конце, убедиться, что результат соответствует ожиданиям.

2. Каждый элемент поведения должен быть протестирован один раз. Тестирование одного и того же поведения более одного раза не означает, что ваше ПО с большей вероятностью будет работать. Тесты тоже необходимо поддерживать. Если вы вносите небольшие изменения в свою кодовую базу, а затем двадцать тестов прерываются, как вы узнаете, какая функциональность нарушена? Когда только один тест завершается неудачей, гораздо проще найти ошибку.

3. Каждый тест должен быть независимым от других тестов. В противном случае вам будет трудно поддерживать и запускать набор тестов.

## Простой пример

```python
def another_sum(a, b):
    return a + b


def test_another_sum():
    assert another_sum(3, 2) == 5
```

Прежде всего, вы никогда не будете писать тесты внутри своей кодовой базы, поэтому давайте разделим это на два файла и пакета.

```
├── sum
│   ├── __init__.py
│   └── another_sum.py
└── tests
    ├── __init__.py
    └── test_sum
        ├── __init__.py
        └── test_another_sum.py
```

_test_another_sum.py_

```python
from sum.another_sum import another_sum


def test_another_sum():
    assert another_sum(3, 2) == 5
```

Затем добавьте пустой conftest.py файл, который используется для хранения [фикстур](https://docs.pytest.org/en/stable/explanation/fixtures.html) pytest, внутри папки tests.

Наконец, добавьте pytest.ini - файл конфигурации pytest - в папку tests, которая также может быть пустой, как и в этот момент.

Полная структура проекта теперь должна выглядеть следующим образом:

```
├── sum
│   ├── __init__.py
│   └── another_sum.py
└── tests
    ├── __init__.py
    ├── conftest.py
    ├── pytest.ini
    └── test_sum
        ├── __init__.py
        └── test_another_sum.py
```

Хранение ваших тестов в одном пакете позволяет вам:

1. Переиспользование конфигурации pytest во всех тестах
2. Переиспользование фикстур во всех тестах
3. Упрощённое выполнения тестов
Вы можете запустить все тесты с помощью этой команды:

```bash
(venv)$ python -m pytest tests
```

Результат проверки:
```bash
============================== test session starts ==============================
platform darwin -- Python 3.10.1, pytest-7.0.1, pluggy-1.0.0
rootdir: /testing_project/tests, configfile: pytest.ini
collected 1 item

tests/test_sum.py/test_another_sum.py .                                 [100%]

=============================== 1 passed in 0.01s ===============================
```

## Реальное приложение

Создадим простое приложения для блога. Flask - web-фреймворк, SQLite - база данных.

Функционал нашего приложения:
* Создание статей
* Отображение списка статей
* Обновление статей

Файловая структура:

```
blog_app
    ├── blog
    │   ├── __init__.py
    │   ├── app.py
    │   └── models.py
    └── tests
        ├── __init__.py
        ├── conftest.py
        └── pytest.ini
```

_models.py_

```python
import os
import sqlite3
import uuid
from typing import List

from pydantic import BaseModel, EmailStr, Field


class NotFound(Exception):
    pass


class Article(BaseModel):
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    author: EmailStr
    title: str
    content: str

    @classmethod
    def get_by_id(cls, article_id: str):
        con = sqlite3.connect(os.getenv("DATABASE_NAME", "database.db"))
        con.row_factory = sqlite3.Row

        cur = con.cursor()
        cur.execute("SELECT * FROM articles WHERE id=?", (article_id,))

        record = cur.fetchone()

        if record is None:
            raise NotFound

        article = cls(**record)  # Row can be unpacked as dict
        con.close()

        return article

    @classmethod
    def get_by_title(cls, title: str):
        con = sqlite3.connect(os.getenv("DATABASE_NAME", "database.db"))
        con.row_factory = sqlite3.Row

        cur = con.cursor()
        cur.execute("SELECT * FROM articles WHERE title = ?", (title,))

        record = cur.fetchone()

        if record is None:
            raise NotFound

        article = cls(**record)  # Row can be unpacked as dict
        con.close()

        return article

    @classmethod
    def list(cls) -> List["Article"]:
        con = sqlite3.connect(os.getenv("DATABASE_NAME", "database.db"))
        con.row_factory = sqlite3.Row

        cur = con.cursor()
        cur.execute("SELECT * FROM articles")

        records = cur.fetchall()
        articles = [cls(**record) for record in records]
        con.close()

        return articles

    def save(self) -> "Article":
        with sqlite3.connect(os.getenv("DATABASE_NAME", "database.db")) as con:
            cur = con.cursor()
            cur.execute(
                "INSERT INTO articles (id,author,title,content) VALUES(?, ?, ?, ?)",
                (self.id, self.author, self.title, self.content)
            )
            con.commit()

        return self

    @classmethod
    def create_table(cls, database_name="database.db"):
        conn = sqlite3.connect(database_name)

        conn.execute(
            "CREATE TABLE IF NOT EXISTS articles (id TEXT, author TEXT, title TEXT, content TEXT)"
        )
        conn.close()
```
[comment]: <> (TODO: Active Record паттерн)

Это  [Active Record](https://en.wikipedia.org/wiki/Active_record_pattern)-style модель, которая предоставляет методы для хранения, извлечения одной статьи и перечисления всех статей.

## Создание новой статьи

Далее давайте рассмотрим нашу бизнес-логику. Мы напишем несколько вспомогательных команд и запросов, чтобы отделить нашу логику от модели и API. Поскольку мы используем pydantic, мы можем легко проверять данные на основе нашей модели.

Создайте пакет _test_article_ в папке _tests_. Затем добавьте файл с именем _test_commands.py_ к нему.

```
blog_app
    ├── blog
    │   ├── __init__.py
    │   ├── app.py
    │   └── models.py
    └── tests
        ├── __init__.py
        ├── conftest.py
        ├── pytest.ini
        └── test_article
            ├── __init__.py
            └── test_commands.py
```

_test_commands.py:_

```python
import pytest

from blog.models import Article
from blog.commands import CreateArticleCommand, AlreadyExists


def test_create_article():
    """
    GIVEN CreateArticleCommand with valid author, title, and content properties
    WHEN the execute method is called
    THEN a new Article must exist in the database with the same attributes
    """
    cmd = CreateArticleCommand(
        author="john@doe.com",
        title="New Article",
        content="Super awesome article"
    )

    article = cmd.execute()

    db_article = Article.get_by_id(article.id)

    assert db_article.id == article.id
    assert db_article.author == article.author
    assert db_article.title == article.title
    assert db_article.content == article.content


def test_create_article_already_exists():
    """
    GIVEN CreateArticleCommand with a title of some article in database
    WHEN the execute method is called
    THEN the AlreadyExists exception must be raised
    """

    Article(
        author="jane@doe.com",
        title="New Article",
        content="Super extra awesome article"
    ).save()

    cmd = CreateArticleCommand(
        author="john@doe.com",
        title="New Article",
        content="Super awesome article"
    )

    with pytest.raises(AlreadyExists):
        cmd.execute()
```

Эти тесты охватывают следующие бизнес-кейсы:

* статьи должны создаваться на основе достоверных данных
* название статьи должно быть уникальным

_blog/commands.py_

```python
from pydantic import BaseModel, EmailStr

from blog.models import Article, NotFound


class AlreadyExists(Exception):
    pass


class CreateArticleCommand(BaseModel):
    author: EmailStr
    title: str
    content: str

    def execute(self) -> Article:
        try:
            Article.get_by_title(self.title)
            raise AlreadyExists
        except NotFound:
            pass

        article = Article(
            author=self.author,
            title=self.title,
            content=self.content
        ).save()

        return article
```

## Тестовые фикстуры

Мы можем использовать pytest-фикстуры для очистки базы данных после каждого теста и создавать новые данные перед каждым тестом. Фикстуры - это функции, обернут
