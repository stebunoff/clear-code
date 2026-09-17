# Урок: Имена функций/методов

1. getURLSuccess() -> resolveSuccessURL()
Комментарий: метод, который собирает URL, на который будет перенаправлен пользователь в случае успешной оплаты.

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

2. getPaymentData() -> buildPaymentPayload()
Комментарий: метод, который собирает в один массив данные для запроса в платёжную систему.

```php
private function getPaymentData(): array
    {
        $offer = $this->getOffer();
        $expirationDatetime = Carbon::now('UTC')->addMinutes($offer->link_expired_in_min)->setTimezone('+3')->format('Y-m-d H:i:s');
        $productVariants = $this->getOfferProducts($offer);
        
        return [
            'order_id' => $this->getClientId(),
            'do' => 'link',
            'urlNotification' => route('success-payment'),
            'urlSuccess' => $this->getURLSuccess(),
            'sys' => self::PRODAMUS_SYS,
            'paid_content' => 'Материалы курса вы найдёте в личном кабинете студента по адресу https://edu.adbulls.ru/lk. Доступ будет отправлен вам на указанный адрес электронной почты.',
            'link_expired' => $expirationDatetime,
            'type' => 'json',
            'products' => $productVariants
        ];
    }
```

3. createPaymentLink() -> createPaymentLink() + requestPaymentLink() + buildPaymentPayload()
Комментарий: у одного метода PaymentService слишком много ответственности. Лучше разделить его на несколько.

```php
public function createPaymentLink()
    {
        $data = $this->getPaymentData();
        $data['signature'] = $this->signatureVerificationService->create($data, env('PRODAMUS_KEY'));
        $link = sprintf('%s?%s', self::PRODAMUS_FORM_LINK, http_build_query($data));
        $response = Http::withHeaders([
            'Content-Type' => 'text/plain',
            'Charset' => 'utf-8'
        ])->get($link);

        return json_decode($response->body(), true)['payment_link'];
    }
```

4. paymentService->registerPurchase() -> purchaseService->createPurchase()
Комментарий: метод находится в PaymentService, но напрашивается его перенос в отдельный PurchaseService.

```php
public function registerPurchase(int $userId, array $productVariants): int
    {
        $purchase = Purchase::create([
            'user_id' => $userId
        ]);
        foreach ($productVariants as $variant) {
            $purchase->items()->create([
                'product_variant_name' => $variant['productVariantName'],
                'product_price' => $variant['price'],
                'quantity' => $variant['quantity'],
                'product_variant_id' => $variant['productVariantId']
            ]);
        }

        return $purchase->id;
    }
```

5. analyticsService->sendPurchaseEvent() -> analyticsService->sendPurchaseEvent() + analyticsService->buildPurchaseEventParams()
Комментарий: метод отправляет данные в систему аналитики. Лучше разделить ответственность на несколько методов.

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

6. funnelService->moveToNextStep() -> funnelService->resolveNextStep() определяет, какой шаг следующий + funnelService->resolveNextRoute() вычисляет следующий route + funnelService->prepareNextStep() подготовить всё необходимое для побочного эффекта + funnelService->moveToNextStep() меняет состояние текущего шага
Комментарий: в автоматической воронке продаж метод вычисляет, куда дальше направить пользователя.

```php
public function moveToNextStep(string $funnelCode, ?string $currentFunnelStep, string $endPath): string
    {
        [$nextRoute, $isLTO] = $this->getNextStep($funnelCode, $currentFunnelStep, $endPath);
        if ($nextRoute && $isLTO) {
            $this->LTOService->createLTO($nextRoute);
        }

        return $nextRoute;
    }
```

7. productVariantService->giveAccess() -> productVariantService->grantAccess() + emailSender->scheduleAccessGrantedEmail()
Комментарий: метод находится в ProductVariantService, но выдаёт доступ, да ещё и отправляет письмо. Эти ответственности лучше разделить.

```php
public function giveAccess(User $user, array $productVariants)
    {
        foreach ($productVariants as $variant) {
            $user->productVariants()->syncWithoutDetaching($variant['productVariantId']);
            PendingMail::create([
                'email' => $user->email,
                'mailable_class' => \App\Mail\ProductAvailable::class,
                'mailable_data' => [$variant['productVariantName']]
            ]);
        }
    }
```

8. checkAccess() -> hasAccess() в сервисе, а уже в контроллере сделать перенаправление
Комментарий: метод проверяет доступ к варианту продукта и делает redirect / abort.

```php
public function checkAccess(ProductVariant $productVariant)
    {
        if (!request()->user()->productVariants()->where('product_variant_id', $productVariant->id)->exists()) {
            return abort(redirect()->route($productVariant->route_name));
        }
    }
```

9. productVariantService->getProductVariantByBillingName() -> productVariantService->findByBillingName()
Комментарий: метод не является геттером, а ищет в БД.

```php
private function getProductVariantByBillingName(string $productVariantBillingName)
{
    return ProductVariant::where('billing_name', $productVariantBillingName)->first();
}
```

10. processEmailsArrays() -> validateEmailChanges()
Комментарий: название метода в EmailValidationService плохо поясняет то, что метод делает.

```php
public function processEmailsArrays(array $add = [], array $remove = []): array
    {
        $invalidAdd = $this->validate($add);
        $invalidRemove = $this->validate($remove);

        $validAdd = array_diff($add, array_column($invalidAdd, 'email'));
        $validRemove = array_diff($remove, array_column($invalidRemove, 'email'));

        return [
            'valid' => [
                'add' => $validAdd,
                'remove' => $validRemove,
            ],
            'invalid' => [
                'add' => $invalidAdd,
                'remove' => $invalidRemove,
            ],
        ];
    }
```

11. enrichMetrikaData() -> attachMetrikaCounters()
Комментарий: лучше дать более информативное название.

```php
private function enrichMetrikaData(Integration $integration): void
{
    $counters = $this->metrikaService->listCounters($integration->metrika->access_token);
    $integration->setRelation(TempRelationKey::AvailableCounters->value, $counters);
}
```

12. getBody() -> decodeBody(), а остальное вынести вовне
Комментарий: в парсере функция не только получает тело запроса, но и валидирует URL, делает HTTP запрос, проверяет код ответа сервера, определяет кодировку, декодирует тело, перехватывает и логирует ошибки, да и возвращает текст, а не body.

```js
export async function getBody(url) {
  try {
    if (!validateURL(url)) {
      throw new Error(`Invalid url: ${url}`);
    }
    const { statusCode, headers, body } = await request(url);
    checkStatusCode(statusCode);

    let text = '';
    if (headers['content-type'] === 'text/html; charset=utf-8') {
      text = body.text();
    } else {
      const buffer = await body.arrayBuffer();
      const bufferNode = Buffer.from(buffer);
      text = iconv.decode(bufferNode, 'win1251');
    }
    return text;
  } catch (err) {
    console.log(err);
  }
}
```

