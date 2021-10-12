https://habr.com/ru/post/555488/comments/#comment_23038244

```python
# Вспомогательная функция
rec = lambda f: f(f)

# Определение «числа»
zero = lambda: None
succ = lambda x: lambda: x

# Сложение «чисел»
add_num = rec(lambda r: lambda x, y: x if not y() else r(r)(succ(x), y()))

# Преобразование в «число»
to_num = lambda n: (lambda l: rec(lambda r: lambda a: succ(a) if not next(l) else r(r)(succ(a))))(iter(reversed(range(n))))(zero)

# Преобразование из «числа»
ln = lambda i: next(iter(next(iter(reversed(list(enumerate(i)))))))
from_num = lambda num: ln(rec(lambda r: lambda n, l: l if not n() else r(r)(n(), [*l, []]))(num, [[]]))

# Сложение
add = lambda x, y: from_num(add_num(to_num(x), to_num(y)))

# Проверяем
add(5, 3) # выведет 8
```