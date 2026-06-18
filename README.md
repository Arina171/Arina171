# 📘 Демо-экзамен КОД 09.02.07-5-2026

> **Полный отчёт по выполнению заданий демонстрационного экзамена профильного уровня**
> 
> 📅 Дата: 18 июня 2026 г. | 🎯 Специальность: 09.02.07 Информационные системы и программирование

---

## 📑 Содержание

- [🎯 Модуль 1. Проектирование ER-диаграммы](#-модуль-1-проектирование-er-диаграммы)
- [🗄️ Модуль 2. Разработка базы данных](#️-модуль-2-разработка-базы-данных)
  - [Создание БД через phpMyAdmin](#шаг-1-создание-бд-через-phpmyadmin)
  - [SQL-скрипт таблиц](#шаг-2-sql-скрипт-создания-таблиц)
  - [Импорт из JSON](#шаг-3-импорт-данных-из-json)
- [🔍 Модуль 3. Создание запроса](#-модуль-3-создание-запроса)
- [💻 Модуль 4. Разработка ИС (Laravel)](#-модуль-4-разработка-информационной-системы-laravel)
  - [Создание проекта](#41-создание-проекта)
  - [Настройка .env](#42-настройка-env)
  - [Миграция users](#43-миграция-таблицы-users)
  - [Модель User](#44-модель-user)
  - [Контроллеры](#45-контроллеры)
  - [Маршруты](#46-маршруты-routeswebphp)
  - [Blade-представления](#47-blade-представления)
  - [Тестовый админ](#48-создание-тестового-администратора)
- [📑 Модуль 5. Проектная документация](#-модуль-5-проектная-документация)
- [🔗 Модуль 6. Интеграция модулей](#-модуль-6-интеграция-программных-модулей)
  - [TestCaseController](#61-создание-контроллера)
  - [Python-скрипт для Word](#64-python-скрипт-для-записи-в-закладки-word)
  - [Тест-кейсы](#66-тест-кейсы)
- [📌 Шпаргалка команд](#-шпаргалка-команд-laravel)
- [✅ Чек-лист выполнения](#-итоговый-чек-лист-выполнения)

---

## 🎯 Модуль 1. Проектирование ER-диаграммы

> 📄 Результат: файл `ER_Diagram.pdf`

### Инструкция
1. Открой https://app.diagrams.net/ (draw.io)
2. Выбери шаблон **Entity Relation**
3. Создай 9 сущностей (см. ниже)
4. File → Export as → PDF → сохрани как `ER_Diagram.pdf`

### Структура БД

| Сущность | PK | FK |
|---|---|---|
| **clients** | id | — |
| **nomenclature** | id | — |
| **specifications** | id | product_id → nomenclature |
| **specification_lines** | id | specification_id → specifications, material_id → nomenclature |
| **orders** | id | client_id → clients |
| **order_lines** | id | order_id → orders, product_id → nomenclature |
| **production** | id | — |
| **production_outputs** | id | production_id → production, product_id → nomenclature |
| **production_inputs** | id | production_id → production, material_id → nomenclature |

[⬆ К содержанию](#-содержание)

---

## 🗄️ Модуль 2. Разработка базы данных

### Шаг 1. Создание БД через phpMyAdmin

1. Открой `http://localhost/phpmyadmin`
2. Вкладка **«Базы данных»** → имя: `kod_09_02_07` → сравнение: `utf8mb4_unicode_ci` → **«Создать»**
3. Кликни по БД → вкладка **«SQL»** → вставь скрипт → **«Вперёд»**

### Шаг 2. SQL-скрипт создания таблиц

<details>
<summary><b>📜 Нажми, чтобы раскрыть SQL-скрипт</b></summary>

```sql
CREATE TABLE clients (
    id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    inn VARCHAR(20) DEFAULT NULL,
    address VARCHAR(255) DEFAULT NULL,
    phone VARCHAR(20) DEFAULT NULL,
    salesman BOOLEAN NOT NULL DEFAULT FALSE,
    buyer BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE nomenclature (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(50) DEFAULT NULL,
    name VARCHAR(255) NOT NULL,
    type ENUM('product', 'material') NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    price DECIMAL(10,2) DEFAULT NULL
);

CREATE TABLE specifications (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    product_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL DEFAULT 1,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    FOREIGN KEY (product_id) REFERENCES nomenclature(id)
);

CREATE TABLE specification_lines (
    id INT PRIMARY KEY AUTO_INCREMENT,
    specification_id INT NOT NULL,
    material_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'кг',
    FOREIGN KEY (specification_id) REFERENCES specifications(id),
    FOREIGN KEY (material_id) REFERENCES nomenclature(id)
);

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number INT NOT NULL,
    order_date DATE NOT NULL,
    client_id VARCHAR(20) NOT NULL,
    FOREIGN KEY (client_id) REFERENCES clients(id)
);

CREATE TABLE order_lines (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    price DECIMAL(10,2) NOT NULL,
    total DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price) STORED,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES nomenclature(id)
);

CREATE TABLE production (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number INT NOT NULL,
    production_date DATE NOT NULL
);

CREATE TABLE production_outputs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    production_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'шт',
    FOREIGN KEY (production_id) REFERENCES production(id),
    FOREIGN KEY (product_id) REFERENCES nomenclature(id)
);

CREATE TABLE production_inputs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    production_id INT NOT NULL,
    material_id INT NOT NULL,
    quantity DECIMAL(10,4) NOT NULL,
    unit VARCHAR(20) NOT NULL DEFAULT 'кг',
    FOREIGN KEY (production_id) REFERENCES production(id),
    FOREIGN KEY (material_id) REFERENCES nomenclature(id)
);
```

</details>

### Шаг 3. Импорт данных из JSON

**Команда создания:**
```bash
php artisan make:command ImportClients
```

**Файл:** `app/Console/Commands/ImportClients.php`

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\File;

class ImportClients extends Command
{
    protected $signature = 'import:clients';
    protected $description = 'Import clients from Заказчики.json';

    public function handle()
    {
        $path = base_path('Заказчики.json');

        if (!File::exists($path)) {
            $this->error('Файл Заказчики.json не найден в корне проекта!');
            return;
        }

        $json = File::get($path);
        $data = json_decode($json, true);

        foreach ($data as $item) {
            DB::table('clients')->updateOrInsert(
                ['id' => $item['id']],
                [
                    'name'      => $item['name'],
                    'inn'       => $item['inn'] ?? null,
                    'address'   => $item['address'] ?? null,
                    'phone'     => $item['phone'] ?? null,
                    'salesman'  => $item['salesman'] ?? false,
                    'buyer'     => $item['buyer'] ?? false,
                ]
            );
        }
        $this->info('✅ Данные из Заказчики.json успешно импортированы!');
    }
}
```

**Запуск:**
```bash
php artisan import:clients
```

[⬆ К содержанию](#-содержание)

---

## 🔍 Модуль 3. Создание запроса

> Запрос вычисляет полную стоимость заказа с учётом себестоимости материалов по спецификации.

```sql
SELECT
    o.number AS order_number,
    o.order_date,
    c.name AS client_name,
    prod.name AS product_name,
    ol.quantity AS order_quantity,
    unit_costs.unit_cost AS cost_per_unit,
    ol.quantity * unit_costs.unit_cost AS line_total
FROM orders o
JOIN clients c ON c.id = o.client_id
JOIN order_lines ol ON ol.order_id = o.id
JOIN nomenclature prod ON prod.id = ol.product_id
JOIN (
    SELECT
        s.product_id,
        SUM(sl.quantity * n.price) AS unit_cost
    FROM specifications s
    JOIN specification_lines sl ON sl.specification_id = s.id
    JOIN nomenclature n ON n.id = sl.material_id
    GROUP BY s.product_id
) AS unit_costs ON unit_costs.product_id = ol.product_id;
```

[⬆ К содержанию](#-содержание)

---

## 💻 Модуль 4. Разработка информационной системы (Laravel)

### 4.1. Создание проекта

```bash
composer create-project laravel/laravel kod_project
cd kod_project
```

### 4.2. Настройка .env

Открой `.env` → найди секцию `DATABASE`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=kod_09_02_07
DB_USERNAME=root
DB_PASSWORD=
```

```bash
php artisan config:clear
```

### 4.3. Миграция таблицы users

```bash
php artisan make:migration create_users_table
```

**Файл:** `database/migrations/xxxx_create_users_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->boolean('need_to_change_password')->default(true);
            $table->unsignedTinyInteger('wrong_counter')->default(0);
            $table->boolean('is_admin')->default(false);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

```bash
php artisan migrate
```

### 4.4. Модель User

```bash
php artisan make:model User
```

**Файл:** `app/Models/User.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasFactory;

    protected $fillable = [
        'name', 'email', 'password', 'need_to_change_password',
        'wrong_counter', 'is_admin',
    ];

    protected $hidden = ['password', 'remember_token'];

    protected function casts(): array
    {
        return [
            'password' => 'hashed',
            'need_to_change_password' => 'boolean',
            'is_admin' => 'boolean',
        ];
    }
}
```

### 4.5. Контроллеры

```bash
php artisan make:controller ActionsController
php artisan make:controller ViewsController
```

<details>
<summary><b>📜 ActionsController.php (с капчой и русскими сообщениями)</b></summary>

**Файл:** `app/Http/Controllers/ActionsController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

class ActionsController extends Controller
{
    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required|string|min:6',
            'captcha_solved' => 'required|integer',
        ]);

        if ($request->captcha_solved != 1) {
            return redirect()->route('login')
                ->withErrors(['captcha' => 'Пожалуйста, соберите капчу.']);
        }

        $user = User::firstWhere('email', $request->input('email'));

        if ($user && $user->wrong_counter >= 3) {
            return redirect()->route('login')
                ->withErrors(['email' => 'Вы заблокированы. Обратитесь к администратору']);
        }

        if (Auth::attempt($request->only(['email', 'password']))) {
            $user = Auth::user();
            $user->wrong_counter = 0;
            $user->save();

            if ($user->need_to_change_password) {
                return redirect()->route('login.change-password');
            }

            return redirect()->route('dashboard')
                ->with('status', 'Вы успешно авторизовались');
        }

        if ($user) {
            $user->wrong_counter++;
            $user->save();

            if ($user->wrong_counter >= 3) {
                return redirect()->route('login')
                    ->withErrors(['email' => 'Вы заблокированы. Обратитесь к администратору']);
            }
        }

        return redirect()->route('login')
            ->withErrors(['email' => 'Вы ввели неверный логин или пароль. Пожалуйста проверьте ещё раз введенные данные'])
            ->withInput($request->only('email'));
    }

    public function changePassword(Request $request)
    {
        $request->validate([
            'current_password' => 'required|string|min:6|current_password',
            'new_password' => 'required|string|min:6|confirmed:new_password_repeat',
        ]);

        $user = Auth::user();
        $user->password = Hash::make($request->input('new_password'));
        $user->need_to_change_password = false;
        $user->save();

        return redirect()->route('dashboard')
            ->with('status', 'Пароль успешно изменён.');
    }

    public function editUser(User $user, Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => "required|email|unique:users,email,{$user->id}",
            'password' => 'nullable|string|min:6',
        ], [
            'email.unique' => 'Пользователь с таким логином уже существует',
        ]);

        $user->name = $request->input('name');
        $user->email = $request->input('email');
        if ($request->filled('password')) {
            $user->password = Hash::make($request->input('password'));
        }
        $user->save();

        return redirect()->route('dashboard')
            ->with('status', 'Пользователь обновлён.');
    }

    public function createUser(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|min:6',
        ], [
            'email.unique' => 'Пользователь с таким логином уже существует',
        ]);

        $user = new User();
        $user->name = $request->input('name');
        $user->email = $request->input('email');
        $user->password = Hash::make($request->input('password'));
        $user->save();

        return redirect()->route('dashboard')
            ->with('status', 'Пользователь создан.');
    }

    public function deleteUser(User $user)
    {
        if ($user->id === Auth::id()) {
            return redirect()->route('dashboard')
                ->withErrors(['user' => 'Нельзя удалить свою учётную запись.']);
        }
        $user->delete();
        return redirect()->route('dashboard')
            ->with('status', 'Пользователь удалён.');
    }

    public function unlockUser(User $user)
    {
        if ($user->id === Auth::id()) {
            return redirect()->route('dashboard')
                ->withErrors(['user' => 'Нельзя разблокировать себя.']);
        }
        if ($user->wrong_counter !== 3) {
            $user->wrong_counter = 3;
            $user->save();
            return redirect()->route('dashboard')
                ->with('status', 'Пользователь заблокирован.');
        }
        $user->wrong_counter = 0;
        $user->save();
        return redirect()->route('dashboard')
            ->with('status', 'Пользователь разблокирован.');
    }

    public function logout()
    {
        Auth::logout();
        return redirect()->route('login');
    }
}
```

</details>

<details>
<summary><b>📜 ViewsController.php</b></summary>

**Файл:** `app/Http/Controllers/ViewsController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;

class ViewsController extends Controller
{
    public function index()
    {
        return view('index', ['users' => User::all()]);
    }

    public function login()
    {
        return view('login');
    }

    public function changePassword()
    {
        return view('change_password');
    }

    public function editUser(?User $user = null)
    {
        return view('user_form', ['user' => $user]);
    }
}
```

</details>

### 4.6. Маршруты `routes/web.php`

```php
<?php

use App\Http\Controllers\ViewsController;
use App\Http\Controllers\ActionsController;
use Illuminate\Support\Facades\Route;

Route::get('/', [ViewsController::class, 'index'])
    ->name('dashboard')->middleware(['auth']);

Route::get('/login', [ViewsController::class, 'login'])
    ->name('login')->middleware(['guest']);
Route::post('/login', [ActionsController::class, 'login'])
    ->middleware('guest');

Route::get('/login/change-password', [ViewsController::class, 'changePassword'])
    ->name('login.change-password')->middleware(['auth']);
Route::post('/login/change-password', [ActionsController::class, 'changePassword'])
    ->middleware('auth');

Route::get('/user/{user?}', [ViewsController::class, 'editUser'])
    ->name('user')->middleware(['auth']);
Route::delete('/user/{user}', [ActionsController::class, 'deleteUser'])
    ->name('user.delete')->middleware('auth');
Route::patch('/user/{user?}', [ActionsController::class, 'unlockUser'])
    ->name('user.unlock')->middleware('auth');
Route::post('/user/{user?}', [ActionsController::class, 'createUser'])
    ->name('user.save')->middleware('auth');
Route::put('/user/{user}', [ActionsController::class, 'editUser'])
    ->name('user.update')->middleware('auth');
Route::post('/logout', [ActionsController::class, 'logout'])
    ->name('logout')->middleware('auth');
```

### 4.7. Blade-представления

> 📁 Все файлы создаются вручную в папке `resources/views/`

<details>
<summary><b>📄 resources/views/login.blade.php (с капчой-пазлом)</b></summary>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Авторизация</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        .error { color: red; font-size: 12px; }
        .status { color: green; font-weight: bold; }
        #captcha-container {
            margin: 10px 0; position: relative;
            width: 300px; height: 150px;
            border: 1px solid #ccc; background: #f0f0f0;
        }
        #puzzle-piece {
            position: absolute; cursor: pointer;
            background: rgba(0,0,255,0.5); border: 2px solid #000;
            width: 40px; height: 40px;
        }
        #target-zone {
            position: absolute;
            border: 2px dashed #0a0; background: rgba(0,255,0,0.2);
            width: 40px; height: 40px; left: 220px; top: 80px;
        }
    </style>
</head>
<body>
    <h2>Вход в систему</h2>
    @if(session('status')) <div class="status">{{ session('status') }}</div> @endif

    <form action="{{ route('login') }}" method="POST">
        @csrf
        <div>
            <label>Логин (Email):</label><br>
            <input type="email" name="email" value="{{ old('email') }}" required>
            @error('email') <div class="error">{{ $message }}</div> @enderror
        </div>
        <div>
            <label>Пароль:</label><br>
            <input type="password" name="password" required>
            @error('password') <div class="error">{{ $message }}</div> @enderror
        </div>

        <div>
            <label>Соберите пазл (перетащите синий квадрат в зелёную зону):</label>
            <div id="captcha-container">
                <div id="target-zone"></div>
                <div id="puzzle-piece" style="left: 10px; top: 10px;"></div>
            </div>
            <input type="hidden" name="captcha_solved" id="captcha_solved" value="0">
            @error('captcha') <div class="error">{{ $message }}</div> @enderror
        </div>

        <button type="submit">Войти</button>
    </form>

    <script>
        const piece = document.getElementById('puzzle-piece');
        const target = document.getElementById('target-zone');
        const hiddenInput = document.getElementById('captcha_solved');
        let isDragging = false, offsetX, offsetY;

        piece.addEventListener('mousedown', (e) => {
            isDragging = true;
            offsetX = e.clientX - piece.offsetLeft;
            offsetY = e.clientY - piece.offsetTop;
        });

        document.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            piece.style.left = (e.clientX - offsetX) + 'px';
            piece.style.top = (e.clientY - offsetY) + 'px';
        });

        document.addEventListener('mouseup', () => {
            isDragging = false;
            const pRect = piece.getBoundingClientRect();
            const tRect = target.getBoundingClientRect();
            if (pRect.left >= tRect.left && pRect.right <= tRect.right &&
                pRect.top >= tRect.top && pRect.bottom <= tRect.bottom) {
                piece.style.left = tRect.left + 'px';
                piece.style.top = tRect.top + 'px';
                piece.style.background = 'green';
                hiddenInput.value = '1';
            }
        });
    </script>
</body>
</html>
```

</details>

<details>
<summary><b>📄 resources/views/index.blade.php</b></summary>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
</head>
<body>
    <h1>Панель управления пользователями</h1>
    @if(session('status')) <div style="color:green;">{{ session('status') }}</div> @endif

    <table border="1" cellpadding="5">
        <thead>
            <tr>
                <th>Имя</th><th>Email</th>
                <th>Нужно сменить пароль</th><th>Заблокирован</th><th>Действия</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($users as $user)
                <tr>
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                    <td>{{ $user->need_to_change_password ? 'Да' : 'Нет' }}</td>
                    <td>{{ $user->wrong_counter >= 3 ? 'Да' : 'Нет' }}</td>
                    <td>
                        <a href="{{ route('user', ['user' => $user->id]) }}">Ред.</a>
                        <form action="{{ route('user.delete', ['user' => $user->id]) }}" method="POST" style="display:inline;">
                            @csrf @method('DELETE')
                            <button type="submit">Удал.</button>
                        </form>
                        <form action="{{ route('user.unlock', ['user' => $user->id]) }}" method="POST" style="display:inline;">
                            @csrf @method('PATCH')
                            <button type="submit">{{ $user->wrong_counter >= 3 ? 'Разблок.' : 'Блок.' }}</button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
        <tfoot>
            <tr>
                <td colspan="5">
                    <a href="{{ route('user') }}">Создать пользователя</a>
                    <form action="{{ route('logout') }}" method="POST" style="display:inline;">
                        @csrf <button type="submit">Выйти</button>
                    </form>
                </td>
            </tr>
        </tfoot>
    </table>
</body>
</html>
```

</details>

<details>
<summary><b>📄 resources/views/user_form.blade.php</b></summary>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>{{ $user ? 'Редактирование' : 'Создание' }}</title>
</head>
<body>
    <h1>{{ $user ? 'Редактирование пользователя' : 'Создание пользователя' }}</h1>

    <form action="{{ $user ? route('user.update', $user->id) : route('user.save') }}" method="POST">
        @csrf
        @if ($user) @method('PUT') @endif

        <label>Имя:</label><br>
        <input type="text" name="name" value="{{ old('name', $user->name ?? '') }}" required><br><br>

        <label>Email:</label><br>
        <input type="email" name="email" value="{{ old('email', $user->email ?? '') }}" required><br><br>

        <label>Пароль {{ $user ? '(оставьте пустым, если не меняете)' : '' }}:</label><br>
        <input type="password" name="password" {{ $user ? '' : 'required' }}><br><br>

        <button type="submit">{{ $user ? 'Обновить' : 'Создать' }}</button>
        <a href="{{ route('dashboard') }}">Отмена</a>
    </form>
</body>
</html>
```

</details>

<details>
<summary><b>📄 resources/views/change_password.blade.php</b></summary>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Смена пароля</title>
</head>
<body>
    <h2>Вам необходимо сменить пароль</h2>

    <form action="{{ route('login.change-password') }}" method="POST">
        @csrf
        <label>Текущий пароль:</label><br>
        <input type="password" name="current_password" required><br><br>

        <label>Новый пароль:</label><br>
        <input type="password" name="new_password" required><br><br>

        <label>Повторите новый пароль:</label><br>
        <input type="password" name="new_password_repeat" required><br><br>

        <button type="submit">Сменить пароль</button>
    </form>
</body>
</html>
```

</details>

### 4.8. Создание тестового администратора

```bash
php artisan tinker
```

```php
App\Models\User::create([
    'email' => 'admin@test.ru',
    'password' => '12345678',
    'need_to_change_password' => false,
    'name' => 'Admin',
    'is_admin' => true
]);
exit
```

```bash
php artisan serve
```

Открой: `http://localhost:8000`

[⬆ К содержанию](#-содержание)

---

## 📑 Модуль 5. Проектная документация

> 📄 Создай файл `Модуль_5.docx`

### 1. Функциональное назначение

Информационная система предназначена для авторизации зарегистрированных пользователей с ролями «Администратор» и «Пользователь» и управления их учётными записями. Система обеспечивает защищённый вход по связке логин/пароль, защиту от перебора паролей с интерактивной капчой-пазлом и автоматической блокировкой учётной записи, принудительную смену пароля и административный функционал.

### 2. Структура базы данных (таблица `users`)

| Поле | Тип | Описание |
|---|---|---|
| id | BIGINT (PK) | Уникальный идентификатор |
| name | VARCHAR(255) | Имя пользователя |
| email | VARCHAR(255) | Логин (уникальный) |
| password | VARCHAR(255) | Хэш пароля (bcrypt) |
| need_to_change_password | BOOLEAN | Флаг принудительной смены пароля |
| wrong_counter | TINYINT | Счётчик неверных попыток |
| is_admin | BOOLEAN | Признак роли администратора |
| created_at | TIMESTAMP | Дата создания |
| updated_at | TIMESTAMP | Дата обновления |

### 3. Методы системы

**3.1. Авторизация (login)**
- Маршрут: `POST /login`
- Параметры: `email` (string, req), `password` (string, req), `captcha_solved` (int, req)
- Успех: «Вы успешно авторизовались»
- Ошибка: «Вы ввели неверный логин или пароль. Пожалуйста проверьте ещё раз введенные данные»
- Блокировка: «Вы заблокированы. Обратитесь к администратору»

**3.2. Смена пароля (changePassword)**
- Маршрут: `POST /login/change-password`
- Параметры: `current_password`, `new_password`, `new_password_repeat`

**3.3. Создание пользователя (createUser)**
- Маршрут: `POST /user/{user?}`
- Ошибка при дубликате: «Пользователь с таким логином уже существует»

**3.4. Редактирование пользователя (editUser)**
- Маршрут: `PUT /user/{user}`

**3.5. Удаление пользователя (deleteUser)**
- Маршрут: `DELETE /user/{user}`
- Запрет удаления своей учётной записи

**3.6. Блокировка / разблокировка (unlockUser)**
- Маршрут: `PATCH /user/{user?}`

**3.7. Выход (logout)**
- Маршрут: `POST /logout`

### 4. Безопасность

| Механизм | Описание |
|---|---|
| Капча | Интерактивная пазл-капча на странице входа |
| Хэширование | Алгоритм bcrypt |
| Защита от перебора | Блокировка после 3 неверных попыток |
| CSRF-защита | Все формы защищены CSRF-токеном |
| Валидация | Проверка обязательности и формата на стороне сервера |

[⬆ К содержанию](#-содержание)

---

## 🔗 Модуль 6. Интеграция программных модулей

### 6.1. Создание контроллера

```bash
php artisan make:controller TestCaseController
```

**Файл:** `app/Http/Controllers/TestCaseController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestCaseController extends Controller
{
    public function show(Request $request)
    {
        return view('test.index');
    }

    public function getData()
    {
        // URL строго из задания (api_info.pdf)
        $url = 'http://localhost:4444/TransferSimulator/fullName';

        $context = stream_context_create(['http' => ['timeout' => 5]]);
        $data = @file_get_contents($url, false, $context);

        if ($data) {
            $json = json_decode($data);
            $data = $json->value ?? '';
        } else {
            $data = 'Ошибка получения данных';
        }

        return redirect()->route('test-case')->with('value', $data);
    }

    public function checkData(Request $request)
    {
        $value = $request->input('value');
        $result = '';

        // Критерий 1: наличие цифр
        if (preg_match('/[0-9]/u', $value)) {
            $result = 'Ошибка: ФИО содержит цифры';
        }
        // Критерий 2: наличие спецсимволов
        elseif (preg_match('/[^А-Яа-яЁё\s]/u', $value)) {
            $result = 'Ошибка: ФИО содержит спецсимволы';
        }
        else {
            $result = 'Успешно';
        }

        $this->writeToDocx($result);

        return redirect()->route('test-case')
            ->with('value', $value)
            ->with('message', $result);
    }

    private function writeToDocx($result)
    {
        $docxPath = base_path('ТестКейс.docx');
        $bookmarkName = 'ResultBookmark';
        $pythonScript = base_path('scripts/write_to_docx.py');
        $command = "python \"{$pythonScript}\" \"{$docxPath}\" \"{$bookmarkName}\" \"{$result}\"";
        exec($command, $output, $returnVar);

        if ($returnVar !== 0) {
            \Log::error('Ошибка записи в DOCX: ' . implode("\n", $output));
        }
    }
}
```

**Маршруты (добавь в `routes/web.php`):**

```php
use App\Http\Controllers\TestCaseController;

Route::get('/test', [TestCaseController::class, 'show'])->name('test-case');
Route::get('/test/get', [TestCaseController::class, 'getData'])->name('test-case.get');
Route::post('/test/check', [TestCaseController::class, 'checkData'])->name('test-case.check');
```

<details>
<summary><b>📄 resources/views/test/index.blade.php</b></summary>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация данных</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        .box { border: 1px solid #ccc; padding: 15px; margin: 10px 0; width: 400px; }
        .message { font-weight: bold; color: blue; }
    </style>
</head>
<body>
    <h2>Макет окна приложения валидации данных</h2>

    <form action="{{ route('test-case.get') }}" method="GET">
        <button type="submit">Получить данные</button>
    </form>

    <div class="box">
        <strong>Данные от эмулятора:</strong><br>
        <input type="text" value="{{ session('value', '') }}" readonly style="width: 100%; margin-top: 5px;">
    </div>

    <form method="POST" action="{{ route('test-case.check') }}">
        @csrf
        <input type="hidden" name="value" value="{{ session('value', '') }}">
        <button type="submit">Отправить результат теста</button>
    </form>

    @if(session('message'))
        <div class="message">{{ session('message') }}</div>
    @endif
</body>
</html>
```

</details>

### 6.4. Python-скрипт для записи в закладки Word

```bash
pip install python-docx
```

**Создай папку:** `scripts/` в корне проекта

**Файл:** `scripts/write_to_docx.py`

```python
import sys
from docx import Document
from docx.oxml.ns import qn

def update_bookmark(doc_path, bookmark_name, text):
    doc = Document(doc_path)
    for paragraph in doc.paragraphs:
        bookmark_starts = paragraph._element.findall(qn('w:bookmarkStart'))
        for start in bookmark_starts:
            if start.get(qn('w:name')) == bookmark_name:
                bookmark_id = start.get(qn('w:id'))
                bookmark_ends = paragraph._element.findall(qn('w:bookmarkEnd'))
                for end in bookmark_ends:
                    if end.get(qn('w:id')) == bookmark_id:
                        new_run = paragraph._element.makeelement(qn('w:r'), {})
                        new_text = paragraph._element.makeelement(qn('w:t'), {})
                        new_text.text = text
                        new_run.append(new_text)
                        end.addprevious(new_run)
                        doc.save(doc_path)
                        return "OK"
    return "Bookmark not found"

if __name__ == "__main__":
    if len(sys.argv) == 4:
        update_bookmark(sys.argv[1], sys.argv[2], sys.argv[3])
```

### 6.5. Подготовка ТестКейс.docx

1. Открой `ТестКейс.docx` в Word
2. Заполни столбцы **«Действие»** и **«Ожидаемый результат»**
3. В столбце **«Результат»** для каждой строки поставь закладку `ResultBookmark`:
   - Выдели ячейку → **Вставка → Закладка** → имя `ResultBookmark` → **Добавить**
4. Положи файл в корень проекта Laravel

### 6.6. Тест-кейсы

| № | Действие | Ожидаемый результат |
|---|---|---|
| 1 | Получить данные от эмулятора. ФИО содержит цифры (Иванов1 Иван). Нажать «Отправить результат теста». | Система обнаруживает цифры. Валидация не пройдена. |
| 2 | Получить данные от эмулятора. ФИО содержит спецсимволы (Иванов @Иван). Нажать «Отправить результат теста». | Система обнаруживает спецсимволы. Валидация не пройдена. |
| 3 | Получить данные от эмулятора. ФИО корректное (Иванов Иван Иванович). Нажать «Отправить результат теста». | Валидация пройдена успешно. Данные корректны. |

### 6.7. Запуск эмулятора

```bash
# Вариант 1: EXE
TransferSimulator.exe

# Вариант 2: JAR
java -jar TransferSimulator.jar

# Тестирование через Postman
GET http://localhost:4444/TransferSimulator/fullName
```

[⬆ К содержанию](#-содержание)

---

## 📌 Шпаргалка команд Laravel

| Что создать | Команда | Где появится |
|---|---|---|
| Проект Laravel | `composer create-project laravel/laravel имя` | Новая папка |
| Миграция | `php artisan make:migration create_название_table` | `database/migrations/` |
| Модель | `php artisan make:model Имя` | `app/Models/` |
| Контроллер | `php artisan make:controller ИмяController` | `app/Http/Controllers/` |
| Artisan-команда | `php artisan make:command Имя` | `app/Console/Commands/` |
| View (Blade) | *Вручную* | `resources/views/` |

### Полезные команды

```bash
php artisan serve              # Запуск сервера
php artisan migrate            # Применение миграций
php artisan migrate:refresh    # Откат и повтор миграций
php artisan config:clear       # Очистка кэша конфига
php artisan route:list         # Список маршрутов
php artisan tinker             # Интерактивная консоль
php artisan import:clients     # Импорт клиентов из JSON
```

[⬆ К содержанию](#-содержание)

---

## ✅ Итоговый чек-лист выполнения

<details>
<summary><b>📋 Нажми, чтобы раскрыть чек-лист команд</b></summary>

```bash
# 1. Создание проекта
composer create-project laravel/laravel kod_project
cd kod_project

# 2. Настройка .env (вручную)

# 3. Создание миграции users и запуск
php artisan make:migration create_users_table
php artisan migrate

# 4. Создание модели и контроллеров
php artisan make:model User
php artisan make:controller ActionsController
php artisan make:controller ViewsController
php artisan make:controller TestCaseController

# 5. Создание команды импорта и запуск
php artisan make:command ImportClients
php artisan import:clients

# 6. Создание представлений вручную:
#    resources/views/login.blade.php
#    resources/views/index.blade.php
#    resources/views/user_form.blade.php
#    resources/views/change_password.blade.php
#    resources/views/test/index.blade.php

# 7. Настройка routes/web.php (вручную)

# 8. Создание администратора
php artisan tinker
# App\Models\User::create([...]);

# 9. Установка Python-библиотеки
pip install python-docx

# 10. Создание скрипта scripts/write_to_docx.py

# 11. Запуск сервера
php artisan serve
```

</details>

### 🎯 Что реализовано в проекте

- ✅ ER-диаграмма в PDF (3НФ, PK/FK)
- ✅ База данных MySQL через phpMyAdmin
- ✅ SQL-запрос расчёта стоимости заказа
- ✅ Laravel с миграциями, моделями, контроллерами
- ✅ Капча-пазл на странице входа
- ✅ Блокировка после 3 неверных попыток
- ✅ Принудительная смена пароля
- ✅ CRUD пользователей
- ✅ Все тексты сообщений на русском (точно по заданию)
- ✅ Проверка уникальности логина
- ✅ Интеграция с эмулятором TransferSimulator
- ✅ Валидация ФИО по двум критериям
- ✅ Запись результатов в закладки Word-документа
- ✅ Проектная документация по модулю 5

---

## 🚀 Быстрый старт

```bash
# Клонирование и запуск
git clone <repo-url>
cd kod_project
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan import:clients
php artisan serve
```

---

> 💡 **Совет:** Используй `Ctrl+F` для быстрого поиска по ключевым словам:
> - `login` — авторизация
> - `captcha` — капча
> - `TestCase` — модуль 6
> - `import:clients` — импорт JSON
> - `writeToDocx` — запись в Word

[⬆ В начало](#-демо-экзамен-код-090207-5-2026)
