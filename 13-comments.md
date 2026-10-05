# Урок: Комментарии

## Задание 1
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

## Задание 2
1. Магические числа позволяют существенно сократить объём кода, но прочесть намерение без комментария потом очень тяжело.
Было:
```js
// Избежать установки курсора внутри префикса кода страны.
const onPhoneInputClick = (e) => {
  if (e.target.selectionStart < 4) {
    e.preventDefault();
    e.target.setSelectionRange(3, 3);
  }
};
```

Стало:
```js
const COUNTRY_CODE_END_POSITION = 3;

const isCaretInsideCountryCode = (input) => input.selectionStart < COUNTRY_CODE_END_POSITION + 1;

const moveCaretAfterCountryCode = (input) => {
  input.setSelectionRange(COUNTRY_CODE_END_POSITION, COUNTRY_CODE_END_POSITION);
};

const onPhoneInputClick = (event) => {
  const input = event.target;

  if (!isCaretInsideCountryCode(input)) {
    return;
  }

  event.preventDefault();
  moveCaretAfterCountryCode(input);
};
```

2. Самый простой случай - когда можно просто поменять имя функции (из примера выше).
До:
```js
// Извлечение моделей из таблицы
const extractModelsTypeOne = (tableHead, tableBody) => {
  // тело функции
}
```

После:
```js
const extractModelsFromTable = (tableHead, tableBody) => {
  // тело функции
}
```

3. Пример выше можно улучшить не добавлением комментария, а рефакторингом.
Было:
```js
const onPhoneInputInput = (e) => {
  const matrix = ${baseCountryCode}${baseMatrix};
  const def = matrix.replace(/\D/g, '');
  let i = 0;
  let val = e.target.value.replace(/\D/g, '');
  if (def.length >= val.length) {
    val = def;
  }
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

```js
const formatPhoneByMask = (value) => {
  const mask = `${baseCountryCode}${baseMatrix}`;
  const defaultDigits = mask.replace(/\D/g, '');
  const inputDigits = value.replace(/\D/g, '');

  const digits = inputDigits.length <= defaultDigits.length
    ? defaultDigits
    : inputDigits;

  let digitIndex = 0;

  const replaceMaskCharacter = (maskCharacter) => {
  const isDigitPlaceholder = /[_\d]/.test(maskCharacter);

  if (isDigitPlaceholder && digitIndex < digits.length) {
    return digits.charAt(digitIndex++);
  }

  if (digitIndex >= digits.length) {
    return '';
  }

  return maskCharacter;
};

return mask.replace(/./g, replaceMaskCharacter);
};

const onPhoneInputInput = (event) => {
event.target.value = formatPhoneByMask(event.target.value);
};
```

4. Вынесение в отдельный метод группы инструкций делает понятным намерение и комментарий не нужен:
Внутри класса:
```js
disablePageScroll() {
  scrollLock.lock();
  this.smoothScroll.stop();
}

init() {
  if (!this.isReady()) return;

  this.disablePageScroll();
  window.addEventListener('wheel', this.onHeroWheel);
}
```

5. Комментарий к regrexp не нужен за счёт корректного имени переменной.
До (нужно вчитываться, чтобы понять, что ищем):
```js
const pattern = document.cookie.match(/(?:^|;\s*)_ym_uid=([^;]+)/)?.[1];
```

После (понятно, что в куки ищем идентификатор Я.Метрики):
```js
const ymUid = document.cookie.match(/(?:^|;\s*)_ym_uid=([^;]+)/)?.[1];
```
