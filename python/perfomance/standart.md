# Конкатенация строки через join

```python
abc = ''
abc = ''.join((abc, 'abc'))
```

Такой метод работает медленее в ~ 1.5 раза

```python
abc = ''
abc += 'abc'
```


# Разделение строки

Использовать _partition_ вместо split
Когда мы знаем что разделяющий символ в конце, лучше использовать rsplit


```python
>>> from timeit import timeit
>>> timeit('"abcdefghijklmnopqrstuvwxyz,1".split(",", 1)')
0.23717808723449707
# ['abcdefghijklmnopqrstuvwxyz', '1']

>>> timeit('"abcdefghijklmnopqrstuvwxyz,1".rsplit(",", 1)')
0.20203804969787598
# ['abcdefghijklmnopqrstuvwxyz', '1']

>>> timeit('"abcdefghijklmnopqrstuvwxyz,1".partition(",")')
0.11137795448303223
# ('abcdefghijklmnopqrstuvwxyz', ',', '1')

>>> timeit('"abcdefghijklmnopqrstuvwxyz,1".rpartition(",")')
0.10027790069580078
# ('abcdefghijklmnopqrstuvwxyz', ',', '1')
```

Отличие partition от split: разделяющий символ будет в выводе