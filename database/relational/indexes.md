# Индексы


__Кластеризованные индексы__

Различия между кластеризованным и некластеризованным индексами:
* Кластерный индекс используется для простого и быстрого извлечения данных из базы данных, тогда как чтение из некластеризованного индекса происходит относительно медленнее.
* Кластеризованный индекс изменяет способ хранения записей в базе данных — он сортирует строки по столбцу, который установлен как кластеризованный индекс, тогда как в некластеризованном индексе он не меняет способ хранения, но создает отдельный объект внутри таблицы, который указывает на исходные строки таблицы при поиске.
* Одна таблица может иметь только один кластеризованный индекс, тогда как некластеризованных у нее может быть много.