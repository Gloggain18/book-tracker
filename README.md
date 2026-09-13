# Книжковий портал — Frontend (ASP.NET Core MVC, .NET 8)

Два проєкти:

- `src/BookPortal.Web` — сам фронтенд (MVC + Razor). Це те, що здається.
- `src/BookPortal.MockApi` — тестовий бекенд-заглушка, щоб портал було що показати, поки справжній Backend не запущено. На захист не потрібен, але зручний для демонстрації.

## Як запустити

1. Відкрити `BookPortal.sln` у Visual Studio 2022 (.NET 8 SDK).
2. Правою кнопкою на рішенні → **Set Startup Projects** → *Multiple startup projects* → обидва проєкти на **Start**.
3. F5. Портал відкриється на `http://localhost:5000`, заглушка — на `http://localhost:5080`.

Або з консолі, у двох вкладках:

```bash
dotnet run --project src/BookPortal.MockApi
dotnet run --project src/BookPortal.Web
```

Тестові акаунти заглушки:

| Роль          | Email                      | Пароль      |
|---------------|----------------------------|-------------|
| Адміністратор | admin@bookportal.local     | `Admin123!` |
| Користувач    | reader@bookportal.local    | `Reader123!`|

## Підключення до справжнього бекенду

Усі адреси лежать у `src/BookPortal.Web/appsettings.json` — правити код не треба:

```json
"BackendApi": {
  "BaseUrl": "http://localhost:5080/",
  "Endpoints": {
    "Login":       "api/auth/login",
    "Register":    "api/auth/register",
    "CurrentUser": "api/auth/me",
    "Books":       "api/books",
    "BookById":    "api/books/{id}",
    "Genres":      "api/genres"
  }
}
```

Достатньо змінити `BaseUrl` на адресу свого бекенду і, за потреби, шляхи endpoint-ів. Фронтенд розпізнає різні формати відповідей: токен у полі `token`, `accessToken` або `jwt`; `id` числом або GUID; автора рядком, об'єктом чи масивом; список книг як масив або як `{ items, totalCount }`.

Якщо бекенд повертає `{ items, totalCount }` — фільтрація і пагінація виконуються на сервері. Якщо простий масив — фронтенд фільтрує, сортує і розбиває на сторінки самостійно. Тобто каталог працює в обох випадках.

## Що де лежить

| Тема | Файли |
|---|---|
| Робота з JWT | `Services/Auth/JwtTokenReader.cs`, `ITokenStore.cs`, `Services/Api/JwtAuthorizationHandler.cs` |
| Вхід/реєстрація/вихід | `Controllers/AccountController.cs`, `Services/Auth/AuthService.cs`, `Views/Shared/_AuthModal.cshtml` |
| Реакція на втрату сесії | `Infrastructure/Auth/JwtCookieEvents.cs`, `Infrastructure/BackendExceptionFilter.cs`, `wwwroot/js/site.js` |
| Каталог, пошук, фільтри | `Controllers/BooksController.cs`, `Services/Books/BookCatalogService.cs`, `Models/Books/BookQuery.cs` |
| Картка книги | `Views/Shared/_BookCard.cshtml`, `_BookCover.cshtml` |
| Деталі книги | `Views/Books/Details.cshtml` |
| AJAX-оновлення каталогу | `wwwroot/js/catalog.js`, `Views/Books/_CatalogResults.cshtml` |
| Стилі | `wwwroot/css/site.css` |

## Відповідність плану консультацій

**№1 — базова інтеграція.** Головна сторінка, вхід і реєстрація у Popup, підключена бібліотека `System.IdentityModel.Tokens.Jwt`, передача й отримання даних з Backend.

**№2 — авторизація.** Повний цикл вхід/реєстрація/вихід. JWT зберігається у зашифрованому HttpOnly cookie, тому недоступний для JavaScript. Токен автоматично додається до кожного запиту на захищені endpoint-и. Маршрути захищено через `[Authorize]` і політику `AdminOnly`. Прострочення токена або відповідь 401 від бекенду призводять до виходу з поясненням користувачу; на клієнті працює попередження за 2 хвилини до кінця сесії.

**№3 — каталог.** Сторінка `/catalog` отримує книги з Backend, картка книги винесена в окремий компонент, є навігація між розділами і переходи на сторінку книги. Окремо оброблено стан завантаження (скелетон-картки), порожній каталог і недоступний бекенд.

**№4 — деталі й пошук.** Сторінка `/books/{id}` з повною інформацією про видання (показуються лише ті поля, які реально прийшли з бекенду) і блоком схожих книг. Пошук за назвою, автором та ISBN. Фільтри: жанр, автор, діапазон років. Шість варіантів сортування, пагінація, оновлення результатів без перезавантаження зі збереженням історії браузера. Якщо нічого не знайдено — окремий стан із підказкою.

## Примітки

- Збірка перевірена на .NET 8 SDK: 0 помилок, 0 попереджень.
- Bootstrap 5.3.3 і шрифти підключено з CDN, тож для першого запуску потрібен інтернет.
- Без JavaScript каталог теж працює: форма фільтрів надсилає звичайний GET-запит, сторінка рендериться на сервері.
