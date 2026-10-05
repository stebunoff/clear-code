# Урок: Массивы

Для сокращения количества кода заменил реальные функции на print, где было возможно.

1. Я стараюсь использовать индексацию, когда индекс действительно нужен, но при изучении материалов обнаружил, что можно заменить
```python
for i in range(len(arr)):
    print(i, arr[i])
```

На
```python
for i, element in enumerate(arr):
    print(i, element)
```

То есть, меняем индекс на порядковый номер операции, если порядковый номер необходим.

2. В задаче с выбором каждого второго элемента можно отказаться от индексации.
Было:
```python
arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]

for i in range(0, len(arr), 2):
    print(arr[i])
```

Стало:
```python
arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]

isTake = False

for element in arr:
    if isTake:
        print(element)
    isTake = not isTake
```

3. Историю изменений, сделанную через массив, лучше разделить на стек undo + стек redo.
Тогда вместо
```python
history = ["type A", "type B", "delete B", "type C"]

current_index = 2

current = history[current_index]
previous = history[current_index - 1]
next = history[current_index + 1]
```

Получится
```python
undo_stack = ["type A", "type B", "delete B"]
redo_stack = ["type C"]

current = undo_stack[-1]

def undo():
    if undo_stack:
        command = undo_stack.pop()
        redo_stack.append(command)
        return command

def redo():
    if redo_stack:
        command = redo_stack.pop()
        undo_stack.append(command)
        return command
```

4. При хранении только последних 100 действий в истории изменений первое, что приходит на ум - это воспользоваться массивом.
```python
events.append(event)

if len(events) > 100:
    del events[0]
```

Но ведь можно взять ограниченную очередь (ещё и о сдвиге элементов можно теперь не думать):
```python
from collections import deque

events = deque(maxlen=100)
events.append(event)
```

5. Постепенно знакомлюсь с конечными автоматами и накапливаю примеры их применения.
Обнаружил, что поиск по шаблону как раз является такой задачей. Например, из логов понадобилось извлечь случаи Error -> Retry -> Success.

Первое, что пришло на ум:
```python
events = ["INFO", "ERROR", "RETRY", "SUCCESS", "INFO"]

for i in range(len(events) - 2):
if (
    events[i] == "ERROR"
    and events[i + 1] == "RETRY"
    and events[i + 2] == "SUCCESS"
):
    print("Pattern found")
```

Но ведь можно написать:
```python
events = ["INFO", "ERROR", "RETRY", "SUCCESS", "INFO"]

expected = iter(["ERROR", "RETRY", "SUCCESS"])
target = next(expected)

for event in events:
    if event != target:
        continue

    target = next(expected, None)

    if target is None:
        print("Pattern found")
        break
```
