Двойное сравнивание

New
```python
if min_price <= price <= max_price:
    ...
```
Old
```python
if price >= min_price and price <= max_price:
    ...
```