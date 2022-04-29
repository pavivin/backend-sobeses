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
