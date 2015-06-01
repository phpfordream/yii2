Обмеження частоти запитів
===============================

Для того, щоб уникнути зловживань, вам слід подумати про додавання обмеження частоти запитів до вашого API. Наприклад,
ви можете обмежити використання вашого API до 100 запитів протягом 10 хвилин для кожного користувача. Якщо від користувача
протягом цього періода часу надходить більша кількість запитів, буде повернута відповідь з кодом 429
("занадто багато запитів").

Для того, щоб увімкнути обмеження частоти запитів, *[[yii\web\User::identityClass|клас user identity]]* повинен реалізовувати
інтерфейс [[yii\filters\RateLimitInterface]]. Цей інтерфейс вимагає реалізації наступних трьох методів:

* `getRateLimit()`: повертає максимальну кількість дозволених запитів та період часу, наприклад `[100, 600]`, що
  означає не більше 100 викликів API прогятом 600 секунд.
* `loadAllowance()`: повертає кількість дозволених запитів, що залишились, та мітку часу *UNIX* останньої перевірки обмеження.
* `saveAllowance()`: зберігає кількість дозволених запитів та поточну мітку часу *UNIX*.

Ви можете використовувати два стовпці в таблиці user для зберігання кількості дозволених запитів та час останньої перевірки.
У методах `loadAllowance()` та `saveAllowance()` можна реалізувати зчитування та зберігання значень цих стовбців відповідно
до даних поточного аутентифікованого користувача. Для підвищення продуктивності можна спробувати зберігати цю
інформацію в кеші чи NoSQL сховищі.

Як тільки відповідний інтерфейс буде реалізований у класі identity, Yii почне автоматично перевіряти обмеження
частоти запитів за допомогою фільтра дій [[yii\filters\RateLimiter]] для [[yii\rest\Controller]]. При перевищенні
обмежень буде викинуто виключення [[yii\web\TooManyRequestsHttpException]].

Ви можете налаштувати обмеження частоти викликів у ваших класах REST-контролерів наступним чином:

```php
public function behaviors()
{
    $behaviors = parent::behaviors();
    $behaviors['rateLimiter']['enableRateLimitHeaders'] = false;
    return $behaviors;
}
```

При увімкненому обмеженні частоти запитів кожна відповідь, за замовчуванням, повертається з наступними HTTP-заголовками,
що містять таку інформацію про поточні обмеження:

* `X-Rate-Limit-Limit`: максимальна кількість запитів, дозволена протягом періоду часу
* `X-Rate-Limit-Remaining`: скільки залишилось дозволених запитів в поточний період часу
* `X-Rate-Limit-Reset`: скільки часу у секундах потрібно почекати до отримання максимальної кількості дозволених запитів

Ви можете відключити ці заголовки, встановивши властивість [[yii\filters\RateLimiter::enableRateLimitHeaders]] у false,
як показано у прикладі вище.