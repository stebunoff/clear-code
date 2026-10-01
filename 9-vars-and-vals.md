# Переменные и их значения

На пункт о явном объявлении переменных примеров у себя не нашёл, так как следую этой рекомендации с самых первых программ.

1. Перенёс объявление константы из начала файла непосредственно перед применением.
Было:
```js
import { storage } from "../utils/storage";
import { resetMetrikaFlag } from "./metrika";

const DELAY = 1000;
const EXPIRATION_TIME = 1800000;
const LS_TIME_ON_SITE = "timeOnSite";

// функции

export const isVisitExpired = () => {
  const { lastVisitTime = 0 } = getStoredTime();
  const now = Date.now();
  return now - lastVisitTime > EXPIRATION_TIME;
};
```

Стало:
```js
export const isVisitExpired = () => {
  const { lastVisitTime = 0 } = getStoredTime();
  const now = Date.now();
  const EXPIRATION_TIME = 1800000;
  return now - lastVisitTime > EXPIRATION_TIME;
};
```

2. В парсер добавил проверку на пустой HTML.
```js
const parseUrl = async (url) => {
  const html = await getBody(url);
  assert(
    typeof html === 'string' && html.length > 0,
    `Empty HTML for ${url}`
  );
  // остальной код функции
};
```

3. Добавил проверку при извлечении данных из LocalStorage в коде подсчёта количества просмотренных страниц и времени на сайте.

Было:
```js
export const getStoredTime = () => storage.get(LS_TIME_ON_SITE, {});
```

Стало:
```js
export const getStoredTime = () => {
  const data = storage.get(LS_TIME_ON_SITE, {});

  checkInvariant(
    data.timeOnSite === undefined ||
      (Number.isFinite(data.timeOnSite) && data.timeOnSite >= 0),
    `Invalid stored timeOnSite: ${data.timeOnSite}`
  );

  checkInvariant(
    data.lastVisitTime === undefined ||
      (Number.isFinite(data.lastVisitTime) && data.lastVisitTime >= 0),
    `Invalid stored lastVisitTime: ${data.lastVisitTime}`
  );

  checkInvariant(
    data.isReady === undefined ||
      typeof data.isReady === 'boolean',
    `Invalid stored isReady: ${data.isReady}`
  );

  return data;
};
```

4. Добавил проверку при извлечении refresh токена из БД.

Метод в сервисе для обновления токена (в репозитории проверки не было):
```php
public function refreshToken(): void
    {
        $this->exchangeOrUpdateToken(
            'refresh_token',
            $this->amoTokenRepository->get()->refresh_token,
            'refresh_token'
        );
    }
```

Добавил проверку в репозиторий:
```php
public function get(): AmoTokens
{
    $tokens = AmoTokens::first();

    if ($tokens === null) {
        throw new LogicException('AmoCRM tokens not found.');
    }

    if (empty($tokens->refresh_token)) {
        throw new LogicException('AmoCRM refresh token must not be empty.');
    }

    return $tokens;
}
```

5. Инварианты проще проверять, когда функция возвращает одинаковый результат. В парсере переписал извлечение моделей так, чтобы всегда возвращался массив.

Было:
```js
const extractModels = (cheerioObject) => {
  const modelsTypeOneHead = cheerioObject(Selectors.CompatibleModelsTypeOneHead).children();
  if (modelsTypeOneHead.length !== 0) {
    const modelsTypeOneBody = cheerioObject(Selectors.CompatibleModelsTypeOneBody).children();
    return extractModelsTypeOne(modelsTypeOneHead, modelsTypeOneBody);
  }

  const modelsTypeTwo = cheerioObject(Selectors.CompatibleModelsTypeTwo).children();
  if (modelsTypeTwo.length !== 0) {
    return extractModelsTypeTwo(modelsTypeTwo);
  }

  const modelsTypeThree = cheerioObject(Selectors.CompatibleModelsTypeThree).text();
  if (modelsTypeThree.length !== 0) {
    let compatibleModels = [];
    const data = modelsTypeThree.substring(modelsTypeThree.indexOf(':') + 1)
      .replaceAll(' ', '')
      .split(',');
    if (data[0] !== '') {
      compatibleModels = data;
    }
    return compatibleModels;
  }
};
```

Стало:
```js
const extractModels = (cheerioObject) => {
  // ...

  const modelsTypeThree = cheerioObject(
    Selectors.CompatibleModelsTypeThree
  ).text();

  if (modelsTypeThree.length !== 0) {
    let compatibleModels = [];

    const data = modelsTypeThree
      .substring(modelsTypeThree.indexOf(':') + 1)
      .replaceAll(' ', '')
      .split(',');

    if (data[0] !== '') {
      compatibleModels = data;
    }

    return compatibleModels;
  }

  return [];
};
```

6. В JS добавил в статическую проверку правило, чтобы всегда использовалась директива "use strict"; если это не ES-модуль и не класс.

7. Заметил, что часто аккумулятор в Python можно вообще убрать.
Было:
```python
total = 0

for order in orders:
    total += order.price

```
Стало:
```python
total = sum(order.price for order in orders)
```

8. В скрипте для проверки ссылок на код ответа 200 лучше добавить инвариант, при котором число обработанных url = числу успешно проверенных + ошибочных.

Было:
```js
let counter = 0;

for (const url of urls) {
  try {
    // ...
    counter++;
  } catch ({ message }) {
    // ...
  }
}
```

Стало:
```js
import assert from 'node:assert/strict';

let passed = 0;
let failed = 0;

for (const url of urls) {
  try {
    validateURL(url);

    const { statusCode } = await request(url);
    checkStatusCode(statusCode);

    passed++;
  } catch ({ message }) {
    failed++;

    const errorMessage = `${url} - ${message}\n`;
    await writeFile(ERRORS_LOG, errorMessage, { flag: 'a+' });
  }

  assert(
    passed + failed <= urls.size,
    'Number of processed URLs exceeds total number of URLs'
  );
}

assert(
  passed + failed === urls.size,
  `Expected ${urls.size} processed URLs, got ${passed + failed}`
);
```

9. Инвариант: цена продукта не может быть меньше нуля.

Для кода:
```php
private function getOfferProducts(Offer $offer)
    {
        return $offer->productVariants->map(function ($variant) {
            return [
                'name' => $variant->billing_name,
                'price' => $variant->pivot->price,
                'quantity' => 1,
            ];
        })->toArray();
    }
```

Проверка:
```php
assert(
    is_numeric($variant->pivot->price) && $variant->pivot->price >= 0,
    "Invalid price for product variant {$variant->id}"
);
```

10. Инвариант: время действия спецпредложения должно быть больше нуля.

Для кода:
```php
$expirationDatetime = Carbon::now('UTC')
    ->addMinutes($offer->link_expired_in_min)
```

```php
assert(
    $offer->link_expired_in_min > 0,
    "Offer {$offer->id} has invalid expiration time: {$offer->link_expired_in_min}"
);
```

11. Инвариант: после генерации подписи платежа она не должна быть пустой.
```php
$data['signature'] = $this->signatureVerificationService->create(
    $data,
    env('PRODAMUS_KEY')
);

assert(
    $data['signature'] !== '',
    'Payment signature must not be empty'
);
```

12. После формирования ссылки $data больше не нужна и можно присвоить ей null.
Вот тут:
```php
public function createPaymentLink()
{
    $data = $this->getPaymentData();

    $data['signature'] = $this->signatureVerificationService->create(
        $data,
        env('PRODAMUS_KEY')
    );

    $link = sprintf(
        '%s?%s',
        self::PRODAMUS_FORM_LINK,
        http_build_query($data)
    );

    $response = Http::get($link);

    return json_decode($response->body(), true)['payment_link'];
}
```

13. В парсере лучше присваивать null крупной строке html (но тогда придётся изменить const на let).
```js
const parseUrl = async (url) => {
  const html = await getBody(url);
  const $ = cheerio.load(html);
  const data = [...(await extract($))];
  storeData(url, data);
  sleep(SLEEP_INTERVAL);
};
```

14. После удаления логически элемент уже не принадлежит деку, но ссылка на него всё ещё остаётся в массиве, поэтому её лучше удалить.
Было:
```python
def removeFront(self):
    if not self.size():
        return None

    element = self.deque[self.head]
    self.head = (self.head + 1) % self.capacity
    self._size -= 1

    return element
```

Стало:
```python
def removeFront(self):
    if not self.size():
        return None

    element = self.deque[self.head]

    self.deque[self.head] = None

    self.head = (self.head + 1) % self.capacity
    self._size -= 1

    return element
```

15. Лучше проверить, что длинна списков после объединения равна сумме длин списков.
```python
def sum_lists(first, second):
    if first.len() == second.len():
        summarized = LinkedList()

        for a, b in zip(first, second):
            summarized.add_in_tail(Node(a + b))

        assert summarized.len() == first.len(), (
            "Result list length must be equal to input lists length"
        )

        return summarized
```

```python
def sum_lists(first, second):
    if first.len() == second.len():
        summarized = LinkedList()

        for a, b in zip(first, second):
            summarized.add_in_tail(Node(a + b))

        assert summarized.len() == first.len(), (
            "Result list length must be equal to input lists length"
        )

        return summarized
```
