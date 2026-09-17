# Урок: ООП и интерфейсы

## 3.1. Сделайте в своём коде три примера наглядных методов-фабрик.
1. Создание предложения с ограничением по времени (limited time offer).
```php
final readonly class LTO
{
    private function __construct(
        public string $routeName,
        public int $validUntil,
    ) {}

    public static function forRoute(string $routeName): self
    {
        return new self(
            routeName: $routeName,
            validUntil: time() + 30 * 60,
        );
    }
}

// Использование
$lto = LTO::forRoute('orders.show');
```

2. Напрашивается пример создания пользователя из адреса эл. почты.
```php
final class User
{
    public static function fromEmail(string $email): self
    {
        // создание объекта
    }
}
```

3. Встречал у себя в коде создание пользователя с логикой (например, создать с временным паролем).
Тогда появляется смысл создания отдельной фабрики:
```php
final class UserFactory
{
    public function create(string $email): CreatedUser
    {
        $password = Str::password();

        $user = User::create([
            'email' => $email,
            'password' => Hash::make($password),
        ]);

        return new CreatedUser(
            user: $user,
            plainPassword: $password,
        );
    }
}
```

## 3.2. Если вы когда-нибудь использовали интерфейсы или абстрактные классы, напишите несколько примеров их правильного именования.
Мне очень нравится, когда интерфейс называется по роли, а реализация - чем она отличается от других реализаций. Например:
1. Интерфейс: PaymentService
Реализация 1: ProdamusService
Рeализация 2: RobokassaService
2. Интерфейс: FileReader
Реализация 1: TxtFileReader
Реализация 2: CsvFileReader
