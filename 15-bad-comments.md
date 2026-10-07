# Урок: Плохие комментарии

1. К п.1 (неочевидные комментарии).
Было:
```js
// Из-за ограничения внешнего сервиса.
const batchSize = 100;
```

Стало:
```js
// API внешнего сервиса принимает максимум 100 за запрос
const batchSize = 100;
```

2. К п.12 (функции или переменные вместо комментария).
Было:
```js
// Считаем заказ крупным, если сумма больше 100 000
// и клиент не находится на испытательном периоде
if (order.total > 100_000 && customer.createdAt < trialEndDate) {
  requireManagerApproval(order);
}
```

Стало:
```js
if (isManagerApprovalNeeded(order, customer)) {
  requireManagerApproval(order);
}
```

3. К п.3 (недостоверные комментарии).
```js
// Повторяем запрос максимум три раза.
for (let attempt = 0; attempt < 3; attempt++) {
  try {
    return await request();
  } catch {
    // ...
  }
}
```
Попыток всего три, а повторов - максимум два.

4. К п.4 (шум). Ниже - фрагмент opensource-проекта, где комментирую каждую строчку, даже очевидную.
```php
    /**
     * The access token the user is using for the current request.
     *
     * @var TToken
     */
    protected $accessToken;
```

5. К п.12 (функции или переменные вместо комментария).
Было:
```js
// Новый пользователь — тот, кто зарегистрировался меньше недели назад
if (Date.now() - user.createdAt.getTime() < 7 * 24 * 60 * 60 * 1000) {
  showOnboarding(user);
}
```

Стало:
```js
if (isNewUser(user)) {
  showOnboarding(user);
}
```

6. К п.8 (слишком много информации).
Было:
```js
// Заказ проходит стадии draft → confirmed → paid → shipped.
// После shipped его нельзя изменить.
// Отмена возможна только до передачи в логистику.
if (order.status === 'shipped') {
  throw new OrderCannotBeModifiedError();
}
```

Стало:
```js
// Shipped orders are immutable.
if (order.status === 'shipped') {
  throw new OrderCannotBeModifiedError();
}
```

7. К п.7 (избыточные комментарии). Ниже - фрагмент opensource-проекта, поэтому они комментируют каждую строчку кода, но читать это тяжело.
```php
    /**
     * Get the access tokens that belong to model.
     *
     * @return \Illuminate\Database\Eloquent\Relations\MorphMany<TToken, $this>
     */
    public function tokens()
    {
        return $this->morphMany(Sanctum::$personalAccessTokenModel, 'tokenable');
    }

    /**
     * Determine if the current API token has a given scope.
     *
     * @param  string  $ability
     * @return bool
     */
    public function tokenCan(string $ability)
    {
        return $this->accessToken && $this->accessToken->can($ability);
    }

    /**
     * Determine if the current API token does not have a given scope.
     *
     * @param  string  $ability
     * @return bool
     */
    public function tokenCant(string $ability)
    {
        return ! $this->tokenCan($ability);
    }
```

8. К п.8 (слишком много информации). Комментарий лучше было написать при коммите в систему контроля версий, а не в коде.
```ts
// Раньше мы отправляли пустую строку, но в январе 2024
// партнёр поменял API. Потом мы пробовали отправлять null,
// однако это ломало старую версию клиента, поэтому после обсуждения
// решили вообще не передавать поле, если значения нет.
if (email) {
  payload.email = email;
}
```

9. К п.9 (нелокальная информация). Комментарий лучше перенести в документацию к коду.
```ts
// Репликация между регионами асинхронная.
await userRepository.save(user);
```

10. К п.10 (обязательные комментарии). Нашёл у себя в коде 8-летней давности комментарии в стиле фреймворка, которые просто лучше удалить.
```php
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $placements =  Placement::all();
        $exams =  Exam::with('placement')->get();
        return view('exams.index', compact('placements', 'exams'));
    }
```

11. К п.11 (закомментированный код). В гитхаб это не попало, но даже локально лучше вместо комментария не ленится и идти через историю изменений.
```scss
// .video-demo {
//   width: 100%;
//   height: 100vh;
//   max-height: 700px;
//   overflow: hidden;
// }
```

12. К п.12 (функции или переменные вместо комментария).
До:
```js
// Переводим миллисекунды в минуты
const timeout = config.timeout / 1000 / 60;
```

После:
```js
const timeoutInMinutes = config.timeout / 1000 / 60;
```

13. К п.2 (бормотание)
До:
```js
// Временная заплатка
config.useLegacyParser = true;
```

После:
```js
// Обходное решение для ошибки парсера по тикету № 4821.
// Удалить после обновления парсера до версии 6.4 или выше.
config.useLegacyParser = true;
```
 
14. К п.1 (неочевидные комментарии) нашёл пример в репозитории инструмента Nightwatch для Laravel.
Сейчас чтобы понять, почему оставлен комментарий, нужно не просто пройти по ссылке, но и покопаться в исходниках.
```php
/**
 * @see https://github.com/laravel/framework/pull/49730
 * @see https://github.com/laravel/framework/pull/49754
 * @see https://github.com/laravel/framework/pull/49837
 * @see https://github.com/laravel/framework/releases/tag/v11.0.0
 */
self::$contextExists =
self::$queueNameCapturable =
self::$cacheStoreNameCapturable =
    version_compare($version, '11.0.0', '>=');
```

Лучше явно указать причину:
```php
/**
 * Laravel 11 открывает доступ к скрытому контексту и метаданным,
 * необходимым для получения имен очередей и хранилищ кэша.
 *
 * @see https://github.com/laravel/framework/pull/49730
 * @see https://github.com/laravel/framework/releases/tag/v11.0.0
 */
self::$contextExists =
self::$queueNameCapturable =
self::$cacheStoreNameCapturable =
    version_compare($version, '11.0.0', '>=');
```

15. К п.1 (неочевидные комментарии) пример из того же репозитория Nightwatch.
Ну можно же было вместо
```php
/**
* @see https://github.com/symfony/symfony/pull/54347
* @see https://github.com/symfony/console/releases/tag/v7.1.0-BETA1
*/
public static function parseCommand(ArgvInput $input): string
{
   /** @var array<string> */
   $tokens = method_exists($input, 'getRawTokens')
       ? $input->getRawTokens()
       : (new ReflectionProperty(ArgvInput::class, 'tokens'))->getValue($input);
```

Просто написать, что в новой версии Symfony появился публичный getRawTokens(), а в старой приходится лезть через reflection.
```php
/**
 * Symfony 7.1 exposes raw CLI tokens through getRawTokens().
 * Older versions require reading the internal tokens property.
 *
 * @see https://github.com/symfony/symfony/pull/54347
 */
public static function parseCommand(ArgvInput $input): string
```
