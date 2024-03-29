__Нормализация__

Преимущества:

* Лучшая организация базы данных
* Больше таблиц с небольшими строками
* Эффективный доступ к данным
* Большая гибкость для запросов
* Быстрый поиск информации
* Проще реализовать безопасность данных
* Позволяет легко модифицировать
* Сокращение избыточных и дублирующихся данных
* Более компактная база данных
* Обеспечивает согласованность данных после внесения изменений


Типы:

TODO: примеры БД с нормальными формами
* Первая нормальная форма (1NF) — нет повторяющихся групп в строках
* Вторая нормальная форма (2NF) — каждое неключевое (поддерживающее) значение столбца зависит от всего первичного ключа
* Третья нормальная форма (3NF) — каждое неключевое значение зависит только от первичного ключа и не имеет зависимости от другого неключевого значения столбца

__Денормализация__

Денормализация — намеренное снижение или нарушение форм  нормализации базы данных, обычно — чтобы ускорить чтение из базы за счет добавления избыточных данных. В общем, это процесс, обратный к нормализации.
Так происходит потому, что теория нормальных форм не всегда применима на практике.

К примеру, не атомарные значения — не всегда «зло»: иногда даже наоборот. В некоторых случаях необходимо дополнительное объединение при выполнении запросов, особенно при обработке большого массива информации. В итоге это может улучшить производительность. 

Для баз данных, предназначенных для аналитики, часто выполняют денормализацию, чтобы ускорить выполнение запросов.

__Целостность__

Целостность данных определяет точность, а также согласованность данных, хранящихся в базе данных. Она также определяет ограничения целостности для обеспечения соблюдения бизнес-правил для данных, когда они вводятся в приложение или базу данных.
