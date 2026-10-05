# Урок: Комментарии

1. В парсере из-за неудачного нейминга функций им нужен комментарий, чтобы сразу понять, почему их несколько. Например:
```js
// Извлечение моделей из таблицы
const extractModelsTypeOne = (tableHead, tableBody) => {
  // тело функции
}
```

2. Много примеров можно найти в курсе по АСД.
```python
def is_key(self, key):
    idx = self.hash_fun(key)
    idx_start = idx

    # Разрешение коллизий путём линейного пробирования
    while True:
        if self.slots[idx] is None:
            return False

        if self.slots[idx] == key:
            return True

        idx = (idx + 1) % self.size

        if idx == idx_start:
            return False
```

3. В deque со стеком минимумов при балансировке лучше обозначить этапы.
```python
def rebalance_to_left(self):
    values = []

    # Восстанавить порядок элементов дека из правого стека перед использованием
    while self.right.size():
        values.append(self.right.pop())

    values.reverse()

    # Переместить примерно половину в правый дек
    left_size = (len(values) + 1) // 2
    left_values = values[:left_size]
    right_values = values[left_size:]

    # Сделать push в обратном порядке, чтобы начало дека оставалось поверх левого стека
    for value in reversed(left_values):
        self.left.push(value)

    for value in right_values:
        self.right.push(value)
```
4. Добавил комментарии в текущий рабочий проект, где лучше прописать мотивацию добавления проверок из-за особенностей работы браузера.
```js
  // `transitionend` срабатывает для каждого анимируемого свойства и всплывает,
  // поэтому важно дождаться завершения перехода именно для свойства opacity маски hero-секции.
  const isHeroMaskFadeFinished =
    transitionEvent.target === this.heroMask &&
    transitionEvent.propertyName === 'opacity';
```

5. Аналогично предыдущему пункту.
```js
const onPhoneInputPaste = (e) => {
  e.target.setSelectionRange(0, 0);

  // Вставленное значение применяется браузером после события вставки,
  // поэтому обработку полученного ввода нужно выполнять на следующем шаге цикла событий.
  if (!e.target.selectionStart) {
    setTimeout(() => {
      if (e.target.value.startsWith('+7')) {
        return;
      }

      if (e.target.value.startsWith('+8')) {
        e.target.value = `+7 ${e.target.value.slice(3)}`;
        return;
      }

      e.target.value = '';
    });
  }
};
```

6. Полезно комментировать группу инструкций.
```js
const onPhoneInputFocus = ({target}) => {
  if (!target.value) {
    target.value = baseCountryCode;
  }

  // Прикреплять обработчики форматирования только тогда, когда поле ввода номера телефона активно.
  target.addEventListener('input', onPhoneInputInput);
  target.addEventListener('blur', onPhoneInputBlur);
  target.addEventListener('keydown', onPhoneInputKeydown);
  target.addEventListener('paste', onPhoneInputPaste);
  target.addEventListener('click', onPhoneInputClick);
};
```

7. Регулярные выражения без адекватного названия переменных требуют комментария.
```js
const onPhoneInputInput = (e) => {
  const matrix = `${baseCountryCode}${baseMatrix}`;
  const def = matrix.replace(/\D/g, '');
  let i = 0;
  let val = e.target.value.replace(/\D/g, '');
  if (def.length >= val.length) {
    val = def;
  }
  // Заполнить маску слева направо, используя только цифры из текущего ввода.
  e.target.value = matrix.replace(/./g, (a) => {
    if (/[_\d]/.test(a) && i < val.length) {
      return val.charAt(i++);
    } else if (i >= val.length) {
      return '';
    } else {
      return a;
    }
  });
};
```
