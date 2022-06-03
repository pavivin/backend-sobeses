# SOLID

- [SOLID](#solid)
  - [Single Responsibility Principle^](#single-responsibility-principle)
  - [Open-closed Principle^](#open-closed-principle)
  - [Liskov Substitution Principle^](#liskov-substitution-principle)
  - [Interface Segregation Principle^](#interface-segregation-principle)
  - [Dependency Inversion Principle^](#dependency-inversion-principle)

## Single Responsibility Principle[^](#single-responsibility-principle)
__Принцип единственной обязанности__

    Существует лишь одна причина, приводящая к изменению класса.

Один класс должен решать только какую-то одну задачу. Он может иметь несколько методов, но они должны использоваться лишь для решения общей задачи. Все методы и свойства должны служить одной цели. Если класс имеет несколько назначений, его нужно разделить на отдельные классы.

Пример:
```python
# Возможно убрать лишнее: staticmethod, запрос в ORM
class OrdersReport:
    def get_orders_info(cls, start_date, end_date):
        orders = cls._db_orders_query(start_date, end_date)
        return cls._format(orders)

    @staticmethod
    def _db_orders_query(start_date, end_date):
        return DB.fetch(
            """
                SELECT * FROM ORDERS
                WHERE created_at BETWEEN :start_date AND :end_date
            """,
            {"start_date": start_date, "end_date": end_date},
        )

    @staticmethod
    def _format(orders):
        return f'<h1>{orders}</h1>'
```

Приведённый здесь класс нарушает принцип единственной ответственности. Почему он должен извлекать данные из базы? Это задача для уровня хранения данных, на котором данные сохраняются и извлекаются из хранилища (например, базы данных). Это ответственность не этого класса.

Также данный класс не должен отвечать за формат следующего метода, потому что нам могут понадобиться данные другого формата, например, XML, JSON, HTML и т.д.

Код после рефакторинга будет выглядеть так:


```python
from abc import ABC, abstractmethod


class OrdersReport:
    def __init__(self, repo: OrdersRepository, formatter: OrdersOutPutInterface):
        self.repo = repo
        self.formatter = formatter

    def get_orders_info(self, start_date, end_date):
        orders = repo.get_orders_with_date(start_date, end_date)
        return self.formatter(orders)


class OrdersOutputInterface(ABC):
    @staticmethod
    @abstractmethod
    def output(orders):
        ...


class HtmlOutput(OrdersOutputInterface):
    @staticmethod
    def output(orders):
        return f"<h1>{orders}</h1>"


class OrdersRepository:
    @staticmethod
    def get_orders_with_date(start_date, end_date):
        return DB.fetch(
            """
                SELECT * FROM ORDERS
                WHERE created_at BETWEEN :start_date AND :end_date
            """,
            {"start_date": start_date, "end_date": end_date},
        )
```

## Open-closed Principle[^](#open-closed-principle)
__Принцип открытости/закрытости__

    Программные сущности должны быть открыты для расширения, но закрыты для модификации.

Программные сущности (классы, модули, функции и прочее) должны быть расширяемыми без изменения своего содержимого. Если строго соблюдать этот принцип, то можно регулировать поведение кода без изменения самого исходника.

Пример:

```python
import math


class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height


class Circle:
    def __init__(self, radius):
        self.radius = radius


class AreaCalculator:
    @staticmethod
    def calculate(shape):
        if isinstance(shape, Rectangle):
            area = shape.width * shape.height
        else:
            area = shape.radius * shape.radius * math.pi
        return area

circle = Circle(5)
rect = Rectangle(8, 5)
AreaCalculator.calculate(circle)
```

Если нам нужно вычислить площадь квадрата, то нужно изменить метод вычисления в классе AreaCalculator. Но это нарушит принцип открытости/закрытости, согласно которому мы можем не изменять, а только расширять.

Как же можно решить поставленную задачу?

```python
import math
from abc import ABC, abstractmethod


class AreaInterface(ABC):
    @abstractmethod
    def calculateArea():
        ...

class Rectangle(AreaInterface):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height

class Circle:
    def __init__(self, radius):
        self.radius = radius
    
    def calculate_area(self):
        return shape.radius * shape.radius * math.pi

class AreaCalculator:
    @staticmethod
    def calculate(shape):
        return shape.calculate_area()
```
Теперь можно найти площадь круга, не меняя класс AreaCalculator.

## Liskov Substitution Principle[^](#liskov-substitution-principle)
__Принцип подстановки Барбары Лисков__

В удобочитаемой версии повторяется практически всё, что говорил Бертранд Майер, но здесь в качестве базиса взята система типов:

    Предварительные условия не могут быть усилены в подтипе.
    Постусловия не могут быть ослаблены в подтипе.
    Инварианты супертипа могут быть сохранены в подтипе.

Роберт Мартин в 1996-м дал другое определение, более понятное:

    Функции, использующие указатели ссылок на базовые классы, должны уметь использовать объекты производных классов, даже не зная об этом.

Попросту говоря: подкласс/производный класс должен быть взаимозаменяем с базовым/родительским классом.

Значит, любая реализация абстракции (интерфейса) должна быть взаимозаменяемой в любом месте, в котором принимается эта абстракция. По сути, когда мы используем в коде интерфейсы, то используем контракт не только по входным данным, принимаемым интерфейсом, но и по выходным данным, возвращаемым разными классами, реализующими этот интерфейс. В обоих случаях данные должны быть одного типа.

В этом коде нарушен обсуждаемый принцип и показан способ исправления:

```python
from abc import ABC, abstractmethod


class LessonRepositoryInterface(ABC):
    @staticmethod
    @abstractmethod
    def get_all():
        ...


class FileLessonRepository(LessonRepositoryInterface):
    @staticmethod
    def get_all():
        return []


class DbLessonRepository(LessonRepositoryInterface):
    @staticmethod
    def get_all():
        """
        Нарушает LSP потому что:
        - другой возвращаемый тип
        - экземпляр класса и FileLessonRepository не работают идентично
        return Lesson::all();
        """
        # корректный вариант:
        return list(Lesson.all())
```

## Interface Segregation Principle[^](#interface-segregation-principle)
__Принцип разделения интерфейса__

    Нельзя заставлять клиента реализовать интерфейс, которым он не пользуется.

Это означает, что нужно разбивать интерфейсы на более мелкие, лучше удовлетворяющие конкретным потребностям клиентов.

Как и в случае с принципом единственной ответственности, цель принципа разделения интерфейса заключается в минимизации побочных эффектов и повторов за счёт разделения ПО на независимые части.

Пример:

```python
from abc import ABC, abstractmethod


class WorkerInterface(ABC):
    @staticmethod
    @abstractmethod
    def work():
        ...

    @staticmethod
    @abstractmethod
    def sleep():
        ...


class HumanWorker(WorkerInterface):
    @staticmethod
    def work():
        print("Пьёт кофе")

    @staticmethod
    def sleep():
        print("Дрыхнет")


class RobotWorker(WorkerInterface):
    @staticmethod
    def work():
        print("Работает")

    @staticmethod
    def sleep():
        ...  # Роботу не нужно спать
```

__RobotWorker__’у не нужно спать, но класс должен реализовать метод sleep, потому что все методы в интерфейсе абстрактны. Это нарушает принцип разделения. Вот как это можно исправить:


```python
from abc import ABC, abstractmethod


class WorkAbleInterface(ABC):
    @staticmethod
    @abstractmethod
    def work():
        ...


class SleepAbleInterface(ABC):
    @staticmethod
    @abstractmethod
    def sleep():
        ...


class HumanWorker(WorkAbleInterface, SleepAbleInterface):
    @staticmethod
    def work():
        print("Пьёт кофе")

    @staticmethod
    def sleep():
        print("Дрыхнет")


class RobotWorker(WorkAbleInterface):
    @staticmethod
    def work():
        print("Работает")

```

## Dependency Inversion Principle[^](#dependency-inversion-principle)
__Принцип инверсии зависимостей__

    Высокоуровневые модули не должны зависеть от низкоуровневых. Оба вида модулей должны зависеть от абстракций.

    Абстракции не должны зависеть от подробностей. Подробности должны зависеть от абстракций.

Проще говоря: зависьте от абстракций, а не от чего-то конкретного.

Применяя этот принцип, одни модули можно легко заменять другими, всего лишь меняя модуль зависимости, и тогда никакие перемены в низкоуровневом модуле не повлияют на высокоуровневый.

Пример:

```python
class PostgresConnection:
    def connect():
        print('Postgres connection')

class PasswordReminder:
    def __init__(self, db_connection: PostgresConnection):
        self.db_connection = db_connection
```

Есть распространённое заблуждение, что «инверсия зависимостей» (Dependency Inversion) является синонимом «внедрения зависимостей» ([Dependency Injection](structure-patterns/di.md)). Но это разные вещи.

В приведённом коде, невзирая на то, что класс PostgresConnection был внедрён в класс PasswordReminder, последний зависит от PostgresConnection. Более высокуровневый PasswordReminder не должен зависеть от более низкуровневого модуля PostgresConnection.

Если нам нужно изменить подключение с PostgresConnection на MongoDBConnection, то придётся менять прописанное в коде внедрение конструктора в класс PasswordReminder.

Класс PasswordReminder должен зависеть от абстракций, а не от чего-то конкретного. Но как это сделать? Рассмотрим пример:

```python
from abc import ABC, abstractmethod

class ConnectionInterface(ABC):
    @staticmethod
    @abstractmethod
    def connect():
        ...

class DbConnection(ConnectionInterface):
    @staticmethod
    def connect(self):
        print('Postgres Connection')

class PasswordReminder:
    def __init__(self, db_connection: ConnectionInterface):
        self.db_connection = db_connection
```

Здесь нам нужно изменить подключение с PostgresConnection на MongoDBConnection. Нам не нужно менять внедрение конструктора в класс PasswordReminder, потому что в данном случае класс PasswordReminder зависит только от абстракции.

[Источник](https://habr.com/ru/company/vk/blog/412699/)
