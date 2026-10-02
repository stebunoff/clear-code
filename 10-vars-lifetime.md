# Урок: Время жизни переменных

1. В скрипте, который отправляет данные о выполненной цели в Яндекс.Метрику, можно минимизировать область видимости, так как объект счётчика нужен только в момент отправки данных, а изначально жил на протяжении всей функции.
Код:
```js
const tryOldApi = (counterId, goalId) => {
  const counter = window[`yaCounter${counterId}`];

  if (isOldMetrikaCounterInited(counter)) {
    counter.reachGoal(goalId);
  } else {
    document.addEventListener(`yacounter${counterId}inited`, () => {
      counter.reachGoal(goalId);
    });
  }
};
```

После изменений:
```js
const sendGoal = () => {
  const counter = window[`yaCounter${counterId}`];

  if (counter && typeof counter.reachGoal === 'function') {
    counter.reachGoal(goalId);
    // остальной код
  }
};
```

2. В том же примере можно сгруппировать связанные команды получения счётчика, проверки его готовности и вызов метода ReachGoal.
Стало:
```js
const sendGoal = () => {
  const counter = window[`yaCounter${counterId}`];

  if (counter && typeof counter.reachGoal === 'function') {
    counter.reachGoal(...);
    markSent();
    return true;
  }

  return false;
};
```

3. В парсере определение URL для парсинга, счётчик обработанных страниц и т.п. относятся к процессу парсинга, а сейчас находятся на уровне модуля:
```js
const URLsToParse = read(FILEPATH);

let counter = 1;

const storeData = (url, data) => {
  if (data.length > 0) {
    writeCSV(data);
    counter++;
    console.log(`Successfully parsed: ${url}\n`);
  } else {
    const URLLine = `${url}\n`;
    console.error(`No data on page ${URLLine}`);
    writeFile(ERRORS_FILEPATH, URLLine, { flag: 'a+' });
  }
};

const parseUrl = async (url) => {
    const html = await getBody(url);
    const $ = cheerio.load(html);
    const data = [...(await extract($))];
    storeData(url, data);
    sleep(SLEEP_INTERVAL);
};
```

Их можно сгруппировать:
```js
const parse = () => {
  const URLsToParse = read(FILEPATH);
  let counter = 1;

  const storeData = // код функции
  const parseUrl = // код функции
};
```

4. Заметил, что в том же коде встречается группа операций дважды, которая по сути относится к одному действию записи в лог:
```js
const URLLine = `${url}\n`;
writeFile(ERRORS_FILEPATH, URLLine, { flag: 'a+' });
```

Можно сгруппировать:
```js
const writeErrorUrl = (url) => {
  const URLLine = `${url}\n`;
  writeFile(ERRORS_FILEPATH, URLLine, { flag: 'a+' });
};
```

5. Обнаружил, что в некоторых случаях можно не просто минимизировать область видимости переменной, но и избавится от неё за счёт ранных возвратов из функции.
Например, в коде:
```php
private function getURLSuccess()
    {
        $funnelCode = request()->input('funnel_code');
        if (!$funnelCode) {
            $URLSuccess = route('thank-you');
        } else {
            $funnel = Funnel::where('code', $funnelCode)->first();
            if (!$funnel) {
                Log::error('Funnel code {funnelCode} doesn\'t exist.', ['funnelCode' => $funnelCode]);
                $URLSuccess = route('thank-you');
            } else {
                $URLSuccess = route('funnel') . '?funnel_code=' . $funnelCode;
            }
        }

        return $URLSuccess;
    }
```

```php
private function getURLSuccess()
    {
        if (!$funnelCode) {
            return route('thank-you');
        }

        $funnel = Funnel::where('code', $funnelCode)->first();

        if (!$funnel) {
            Log::error(...);
            return route('thank-you');
        }

        return route('funnel') . '?funnel_code=' . $funnelCode;
    }
```

6. Длинную функцию, в которой работа с одной переменной перемежается с другими действиями:
```php
function completeOrder(Order $order): void
{
    $customer = $this->customerRepository->find($order->customerId);

    $subtotal = 0;

    foreach ($order->items as $item) {
        $subtotal += $item->price * $item->quantity;
    }

    $this->logger->info(
        'Starting order processing',
        ['orderId' => $order->id]
    );

    $this->inventoryService->reserve($order->items);

    $this->auditService->record(
        'ORDER_PROCESSING_STARTED',
        $order->id
    );

    $discount = 0;

    if ($customer->isPremium()) {
        $discount = $subtotal * 0.10;
    }

    $this->notificationService->notifyWarehouse($order->id);

    $this->fraudService->check(
        $customer,
        $order
    );

    $discountedAmount = $subtotal - $discount;

    $this->auditService->record(
        'PRICE_CALCULATED',
        $order->id
    );

    $tax = $discountedAmount * 0.20;

    $this->shippingService->prepareShipment($order);

    $this->metrics->increment('orders.processed');

    $finalPrice = $discountedAmount + $tax;

    $order->subtotal = $subtotal;
    $order->discount = $discount;
    $order->tax = $tax;
    $order->finalPrice = $finalPrice;

    $this->orderRepository->save($order);

    $this->notificationService
        ->sendOrderConfirmation(
            $customer,
            $order
        );
}
```

лучше заменить на отдельный метод, который рассчитывает цену:
```php
function completeOrder(Order $order): void
{
    $customer = $this->customerRepository->find($order->customerId);
    $this->logger->info(
        'Starting order processing',
        ['orderId' => $order->id]
    );

    $this->inventoryService->reserve($order->items);
    $this->fraudService->check(
        $customer,
        $order
    );

    $price = $this->calculatePrice(
        $order,
        $customer
    );

    $order->subtotal = $price['subtotal'];
    $order->discount = $price['discount'];
    $order->tax = $price['tax'];
    $order->finalPrice = $price['finalPrice'];
    $this->shippingService->prepareShipment($order);
    $this->orderRepository->save($order);
    $this->notificationService->sendOrderConfirmation(
        $customer,
        $order
    );
}

private function calculatePrice(
    Order $order,
    Customer $customer
): array {
    $subtotal = 0;

    foreach ($order->items as $item) {
        $lineTotal = $item->price * $item->quantity;
        $subtotal += $lineTotal;
    }

    $discount = $customer->isPremium() ? $subtotal * 0.10 : 0;
    $discountedAmount = $subtotal - $discount;
    $tax = $discountedAmount * 0.20;
    $finalPrice = $discountedAmount + $tax;

    return [
        'subtotal' => $subtotal,
        'discount' => $discount,
        'tax' => $tax,
        'finalPrice' => $finalPrice,
    ];
}
```

7. По-возможности буду стараться передавать значение в параметр функции, если переменная используется в нескольких местах.
Было:
```js
// в начале файла
const TAX_RATE = 0.22

// ниже в фукнциях
function calculatePrice(price) {
  return price * (1 + TAX_RATE);
}
```

Стало:
```js
function calculatePrice(price, taxRate) {
  return price * (1 + taxRate);
}

const result = calculatePrice(100, TAX_RATE);
```

8. В обработчике отправки формы лучше выделить операции с состоянием.
Код обработчика:
```js
async function handleSubmit() {
  const button = document.querySelector('#submit');
  button.disabled = true;
  const data = collectFormData();
  showSpinner();
  await save(data);
  hideSpinner();
  showSuccessMessage();
  button.textContent = 'Saved';
  await analytics.track('form_saved');
  button.disabled = false;
}
```

```js
function setSubmitting(button) {
  button.disabled = true;
  button.textContent = 'Saving...';
}

function setSubmitted(button) {
  button.textContent = 'Saved';
  button.disabled = false;
}

async function handleSubmit() {
  const button = document.querySelector('#submit');
  const data = collectFormData();
  setSubmitting(button);
  showSpinner();
  await save(data);
  hideSpinner();
  setSubmitted(button);
  showSuccessMessage();
  await analytics.track('form_saved');
}
```

9. Обратил внимание, что код должен писаться не в порядке появления мыслей в голове, а ближе к его использованию.
Например, slug объявляется слишком рано:
```js
async function publishArticle(article) {
  const slug = createSlug(article.title);
  await validateArticle(article);
  const author = await loadAuthor(article.authorId);
  await checkPermissions(author);
  const existing = await findBySlug(slug);
  if (existing) {
    throw new Error('Slug already exists');
  }

  return saveArticle({
    ...article,
    slug,
  });
}
```

Лучше перенести ближе к использованию:
```js
async function publishArticle(article) {
  await validateArticle(article);
  const author = await loadAuthor(article.authorId);
  await checkPermissions(author);
  const slug = createSlug(article.title);
  const existing = await findBySlug(slug);
  if (existing) {
    throw new Error('Slug already exists');
  }

  return saveArticle({
    ...article,
    slug,
  });
}
```

10. В процессе поиска примеров для применения рекомендаций обнаружил, что в js важно следить не только за расстоянием в строках кода, но и представлять себе расстояние во времени.
В коде:
```js
let currentPage = 1;

async function parsePage() {
  const html = await fetchPage(currentPage);
  savePage(currentPage, html);
}
```

Лучше зафиксировать значение переменной в начале операции, чтобы пока ждём await другой процесс не изменил значение переменной.
```js
let currentPage = 1;

async function parsePage() {
  const page = currentPage;
  const html = await fetchPage(page);
  savePage(page, html);
}
```

11. Заметил в коде из курса по АСД, что можно сужать не только область видимости переменной, но и ширину её ответственности.
Например, при проверке баланса скобок мне фактически нужен не счётчик изменений changes, просто факт их наличия.
```python
def checkParensBalance(string):
    stack = Stack()

    for item in string:
        stack.push(item)

    balance = 0
    changes = 0

    while stack.size() > 0:
        item = stack.pop()

        if item == ")":
            balance += 1
            changes += 1

        if item == "(":
            balance -= 1
            changes += 1

        if balance < 0:
            return "Sequence unbalanced"

    if changes == 0:
        return "No parenthesis found"

    if balance != 0:
        return "Sequence unbalanced"

    return "Sequence balanced"
```

То есть, можно заменить changes на found_parenthesis = False, то есть, сократить число возможных состояний, в которых может оказаться переменная.

12. Во вращении очереди по кругу можно не просто сузить область видимости переменной, но и вообще избавится от неё.
Тут:
```python
def round(self, n):
        if self.head is None:
            return

        while n > 0:
            self.enqueue(self.dequeue())
            n -= 1
```

Можно сократить до:
```python
for _ in range(n):
    self.enqueue(self.dequeue())
```

13. Задумался над применением {} и нашёл удачный пример: можно переиспользовать удобные имена переменных, если ограничить их блоками кода.
```js
function renderDashboard(data) {
    const userCard = document.querySelector(".user-card");
    const teamCard = document.querySelector(".team-card");

    {
        const title = data.user.name;
        const value = data.user.score;

        userCard.textContent = `${title}: ${value}`;
    }

    {
        const title = data.team.name;
        const value = data.team.score;

        teamCard.textContent = `${title}: ${value}`;
    }
}
```

14. Переменные нужны только для преобразования данных, поэтому код
```js
function updateProfile(user) {
    const firstName = user.firstName.trim();
    const lastName = user.lastName.trim();
    const displayName = `${firstName} ${lastName}`;
    saveProfile({
        displayName
    });
}
```

можно переписать
```js
function updateProfile(user) {
    let displayName;

    {
        const firstName = user.firstName.trim();
        const lastName = user.lastName.trim();

        displayName = `${firstName} ${lastName}`;
    }

    saveProfile({
        displayName
    });
}
```

15. Пример группировки связанных команд для получения результата парсинга:
```js
export async function extract(cheerioObject) {
  const result = {};

  const productName = cheerioObject(Selectors.Name).text();
  result.productName = productName;

  const productSKU = cheerioObject(Selectors.SKU).text();
  result.productSKU = productSKU;

  const productPrice = cheerioObject(Selectors.Price).text().replaceAll(' ', '');
  result.productPrice = productPrice;

  const compatibleModels = extractModels(cheerioObject);
  if (compatibleModels.length === 0) {
    throw new Error('URL without compatible models');
  }
  result.compatibleModels = compatibleModels;

  return [result];
}
```

Группировка + избавление от лишних переменных:
```js
export async function extract(cheerioObject) {
    const result = {
      productName: cheerioObject(Selectors.Name).text(),
      productSKU: cheerioObject(Selectors.SKU).text(),
      productPrice: cheerioObject(Selectors.Price).text().replaceAll(' ', ''),
    };

    const compatibleModels = extractModels(cheerioObject);

    if (compatibleModels.length === 0) {
      throw new Error('URL without compatible models');
    }

    result.compatibleModels = compatibleModels;

    return [result];
}
```

