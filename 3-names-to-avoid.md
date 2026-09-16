# Урок: имена, которых следует избегать

## Найдите 12 примеров имён в вашем коде, которые следует избегать, исправьте, и выложите на гитхаб в формате "было - стало" (с учётом контекста).
1. extractModelsTypeOne и extractModelsTypeTwo и не информативны, и похожи. Лучше extractFromTable и extractFromList
Контекст: функции в модуле парсера для извлечения моделей техники из разметки страницы.

2. params - purchaseEventParams
Контекст: функция отправки данных в систему аналитики.
```php
public function sendPurchaseEvent(string $clientId, int $transactionId, array $productVariants)
{
    $params = [
        'tid' => self::METRIKA_COUNTER_ID,
        'cid' => $clientId,
        'ti' => $transactionId,
        'ms' => env('YANDEX_METRIKA_TOKEN'),
        't' => 'event',
        'pa' => 'purchase',
    ];
    $transactionRevenue = 0;
    foreach ($productVariants as $index => $variant) {
        $params['pr' . $index . 'id'] = $index;
        $params['pr' . $index . 'br'] = self::BRAND_NAME;
        $params['pr' . $index . 'nm'] = $variant['productName'];
        $params['pr' . $index . 'ca'] = $variant['category'];
        $params['pr' . $index . 'pr'] = $variant['price'];
        $params['pr' . $index . 'qt'] = $variant['quantity'];
        $params['pr' . $index . 'va'] = $variant['productVariantName'];

        $transactionRevenue = $transactionRevenue + $variant['sum'];
    }
    $params['transactionRevenue'] = $transactionRevenue;

    $link = self::URL . '?' . http_build_query($params);

    $response = Http::withHeaders([
        'Charset' => 'utf-8'
    ])->get($link);
}
```

3. $funnel используется только для проверки существования воронки продаж, поэтому лучше заменить
```php
$funnel = Funnel::where('code', $funnelCode)->first();
```
на
```php
$funnelExists = Funnel::where('code', $funnelCode)->exists();
```

Контекст: в модуле для оплат нужно сформировать ссылку, куда будет отправлен пользователь после успешной оплаты.
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

4. Вместо общего data лучше использовать деструктурирующее присваивание, так как используется не сам объект data, а его содержимое.
```js
export const getStoredPageDepth = () => {
  const { pageDepth, isReady } = storage.get(PAGE_DEPTH_KEY, {});

  return {
    pageDepth: typeof pageDepth === "number" ? pageDepth : 0,
    isReady: !!isReady,
  };
};
```

Контекст: в модуле для работы с глубиной просмотра сайта функция, которая достаёт из хранилища данные о состоянии объекта pageView
```js
export const getStoredPageDepth = () => {
  const data = storage.get(PAGE_DEPTH_KEY, {});
  return {
    pageDepth: typeof data.pageDepth === "number" ? data.pageDepth : 0,
    isReady: !!data.isReady,
  };
};
```

5. target - observedElement
Контекст: функция для работы с браузерным IntersectionObserver для отслеживания появления элемента на экране.
6. data - leadPayload
Контекст: полезная нагрузка для запроса к api AmoCRM по загрузке обращения в систему.
7. Метод exchangeOrUpdateToken лучше разделить на два: exchangeAuthorizationCode и refreshAccessToken.
Контекст: сервис для обмена данными с amoCRM.

8. subdomain лучше переименовать в baseUrl.
Контекст:
```php
private function getUrl(string $path): string
    {
        $subdomain = env('AMOCRM_SUBDOMAIN');
        return $subdomain . $path;
    }
```

9. Названия учебных функций hash1 и hash2 заменить хотя бы на hash17 и hash223 при применении простых констант внутри.
Контекст: две разные функции хэширования в фильтре Блюма.
10. yellow лучше заменить на accent, так как завтра акцентный цвет поменяется и название цвета перестанет подходить.
Контекст: переменная в SCSS, в которой хранится акцентный цвет дизайн-системы.

11. total - orderTotal
Контекст: сумма заказа в корзине интернет-магазина.

12. timeout - timeoutSeconds
Контекст: расчёт времени показа спецпредложения.
