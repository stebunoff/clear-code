1. В методе delete у LinkedList из курса по АСД нужно по сути нужно было написать в голове этот код заново, чтобы понять, как он работает. Теперь после рефакторинга можно бегло просмотреть код и понять, что происходит.
Было:
```python
    def delete(self, val, all=False):
        prev = None
        curr = self.head

        while curr is not None:
            if curr.value == val:
                if prev is None:
                    self.head = curr.next
                else:
                    prev.next = curr.next

                if curr is self.tail:
                    self.tail = prev

                if not all:
                    return

                curr = curr.next
            else:
                prev = curr
                curr = curr.next
```

Стало (помню про недопустимость вложенных if и else, но в этом примере их не трогал для более наглядной демонстрации работы с логическими переменными):
```python
def delete(self, val, all=False):
    prev = None
    curr = self.head

    while curr is not None:
        value_found = curr.value == val

        if value_found:
            deleting_head = prev is None
            deleting_tail = curr is self.tail
            delete_only_one = not all

            if deleting_head:
                self.head = curr.next
            else:
                prev.next = curr.next

            if deleting_tail:
                self.tail = prev

            if delete_only_one:
                return

            curr = curr.next
        else:
            prev = curr
            curr = curr.next
```

2. В методе для стека заменил приведение числа к Boolean на сравнение чисел.
Было:
```python
def peek(self):
        if not self.size():
            return None

        return self.stack[-1]
```

Стало:
```python
def peek(self):
        if self.size() == 0:
            return None

        return self.stack[-1]
```

3. В CircularQueue добавил проверку на нулевую ёмкость на этапе создания объекта.
```python
def __init__(self, capacity):
    if capacity <= 0:
        raise ValueError("Capacity must be greater than zero")
```
Чтобы дальше по коду спокойно выполнять операцию по модулю
```python
self.tail = (self.tail + 1) % self.capacity
```

4. При получении индекса середины массива в бинарном поиске использовал целочисленное деление.
```python
middle = len(items) // 2
```

5. Нашёл у себя случай, где возможно переполнение целых чисел: хотелось бы написать на F# аналитику контекстной рекламы. На средний аккаунт у меня выходит 35 млн. показов в год, то есть, 100 подключённых аккаунтов выйдут за ограничения 2.4 млрд. для int.
Лучше подстраховаться:
```fsharp
let totalImpressions stats =
    stats
    |> List.sumBy (fun x -> int64 x.Impressions)
```

6. Нашёл в коде специфические проверки и назвал их. Например, проверка инициализированного счётчика Яндекс.Метрики.
Было: `if (typeof ym === 'function') {`, стало:
```js
const isNewApiAvailable = typeof ym === 'function';

if (isNewApiAvailable) {
  tryNewApi(counterId, goalId);
}
```

7. В решениях курса по АСД нашёл много случаев, где можно назвать составные проверки.
Проверку выхода индекса за допустимый диапазон `if i < 0 or i >= self.count:` заменил на `is_out_of_bounds = i < 0 or i >= self.count`.

8. В проверке порядка скобок через стек из курса по АСД условие ниже имеет понятный смысл — новое значение становится новым минимумом:.
Вместо
```python
minValue = self.minStack.peek()
if minValue is None or value <= minValue:
    self.minStack.push(value)
```

Лучше использовать явное
```python
minValue = self.minStack.peek()
is_new_minimum = minValue is None or value <= minValue

if is_new_minimum:
    self.minStack.push(value)
```

9. Посмотрел, как в F# (на котором хочу писать) работают с кодировками.
Внутри программы строки хранят как System.String, а кодировку учитывают только на границах ввода/вывода.
Когда файл пришёл извне:
```fsharp
open System.IO
open System.Text

Encoding.RegisterProvider(CodePagesEncodingProvider.Instance)

let windows1251 = Encoding.GetEncoding(1251)

let bytes = File.ReadAllBytes("input.txt")
let text = windows1251.GetString(bytes)
```
И дальше работать со string.

При сохранении `File.WriteAllText("output.txt", text, Encoding.UTF8)`.

10. Заметил, что проверка условий нередко используется в нескольких местах по коду, а значит её можно не только назвать явно, но и вынести в отдельный метод.
Например, проверку на пустую очередь в enqueue:
```python
if self.tail is None:
```

```python
def is_empty(self):
    return self._size == 0

# дальше по коду
if self.is_empty():
```

11. Посмотрел, что в F# (на котором хочу писать) интернационализацию реализуют через ресурсные файлы .resx + ResourceManager + CultureInfo. Например, в каждом файле будут одинаковые ключи:
Resources.resx
Resources.ru.resx
Resources.en.resx
Например, в Resources.ru.resx:
```fsharp
UserNotFound = Пользователь не найден
Welcome = Добро пожаловать, {0}!
```
Получают строку по ключу:
```fsharp
open System.Globalization
open System.Resources

let resources =
    ResourceManager(
        "MyProject.Resources",
        typeof<Program>.Assembly
    )

let getText key =
    resources.GetString(key, CultureInfo.CurrentUICulture)
```
И используют `printfn "%s" (getText "UserNotFound")`. Насколько я вижу, в коде пользовательские тексты стараются не хранить.


12. В JS для простого проекта можно предусмотреть интернационализацию через хранение строк в нескольких языках:
```js
const LINKS_NAMES = {
  ru: {
    save: "Сохранить",
    settings: "Настройки",
  },
  en: {
    save: "Save",
    settings: "Settings",
  },
};
```
