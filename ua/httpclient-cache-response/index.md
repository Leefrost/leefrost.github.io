# Кешування та HttpClient


Працюючи з системами з високим навантаженням, де час на відповідь дуже обмежений - кеш це перше що приходить на думку. Я розробив невелику бібліотеку для кешування і відтворення відповідей для HttpClient - досить проста та надійна, з підтримкою тонкої конфігурації та налаштуванням кешування за кодом відповіді.

<!--more-->

## HttpClient.Cache

Проста, in-memory, кеш-обгортка для HttpClient.

### Встановлення

```shell
dotnet add package Leefrost.HttpClient.Cache
```

### Плани та ідеї:

- [x] in-memory підтримка
- [ ] підтримка розподіленого кешування

### Використання

Код нижче кешує топ 3 найчастіших відповідей (OK, BadRequest, та InternalServerError) на різний час - 60/10/5 секунд відповідно. HttpClient робить 5 запитів до `https://randomuser.me/api/` та зберігає відповіді у кеш на відведений час.

Кеш відрепортує 1 промах (початковий, адже у кеші нічого немає) і 4 попадання (тому час відповіді дорівнює 0)

```csharp
const string url = "https://randomuser.me/api/";

//Налаштування кешу
var cacheExpiration = new Dictionary<HttpStatusCode, TimeSpan>
{
    {HttpStatusCode.OK, TimeSpan.FromSeconds(60)},
    {HttpStatusCode.BadRequest, TimeSpan.FromSeconds(10)},
    {HttpStatusCode.InternalServerError, TimeSpan.FromSeconds(5)}
};

//Відпрацювання запитів
var requestHandler = new HttpClientHandler();
var cacheHandler = new InMemoryCacheHandler(requestHandler, cacheExpiration);
using (var httpClient = new HttpClient(cacheHandler))
{
    for (int i = 1; i <= 5; ++i)
    {
        Console.Write($"Attempt {i}: {url}");

        var stopwatch = Stopwatch.StartNew();
        var result = await httpClient.GetAsync(url);
        Console.Write($" --> {result.StatusCode} ");
        stopwatch.Stop();
        
        Console.WriteLine($"Done in: {stopwatch.ElapsedMilliseconds} ms");
        await Task.Delay(TimeSpan.FromSeconds(1));
    }
    Console.WriteLine();
}

//Перевірка статусу
var stats = cacheHandler.StatsProvider.GetReport();
Console.WriteLine($"Cache stats - total requests: {stats.Total.TotalRequests}");
Console.WriteLine($"--> Hits: {stats.Total.CacheHit} [{stats.Total.TotalHitsPercent}]");
Console.WriteLine($"--> Misses: {stats.Total.CacheMiss} [{stats.Total.TotalMissPercent}]");
Console.ReadLine();
```

Ось що ми отримаємо в консолі:

```shell
Attempt 1: https://randomuser.me/api/ --> OK Done in: 681 ms
Attempt 2: https://randomuser.me/api/ --> OK Done in: 75 ms
Attempt 3: https://randomuser.me/api/ --> OK Done in: 0 ms
Attempt 4: https://randomuser.me/api/ --> OK Done in: 0 ms
Attempt 5: https://randomuser.me/api/ --> OK Done in: 0 ms

Cache stats - total requests: 5
--> Hits: 4 [0,8]
--> Misses: 1 [0,2]

```

