# Библиотеки написанные на C

## ujson

Быстрый аналог модуля json

Аналоги: orjson, nujson, simplejson

```python
>>> import ujson
>>> ujson.dumps([{"key": "value"}, 81, True])
'[{"key":"value"},81,true]'
>>> ujson.loads("""[{"key": "value"}, 81, true]""")
[{'key': 'value'}, 81, True]
```

## immutables

Аналог dict, изменяемый через специальные мутации

![](https://github.com/MagicStack/immutables/raw/master/bench.png)

```python
import immutables

map = immutables.Map(a=1, b=2)

print(map['a'])
# will print '1'

print(map.get('z', 100))
# will print '100'

print('z' in map)
# will print 'False'
```

Изменение
```python
map2 = map.set('a', 10)
print(map, map2)
# will print:
#   <immutables.Map({'a': 1, 'b': 2})>
#   <immutables.Map({'a': 10, 'b': 2})>

map3 = map2.delete('b')
print(map, map2, map3)
# will print:
#   <immutables.Map({'a': 1, 'b': 2})>
#   <immutables.Map({'a': 10, 'b': 2})>
#   <immutables.Map({'a': 10})>
```

После изменений нужно вызвать метод finish

```python
with map.mutate() as mm:
    mm['a'] = 100
    del mm['b']
    mm.set('y', 'y')
    map2 = mm.finish()

print(map, map2)
# will print:
#   <immutables.Map({'a': 1, 'b': 2})>
#   <immutables.Map({'a': 100, 'y': 'y'})>
```