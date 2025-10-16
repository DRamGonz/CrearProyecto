Perfecto ✅
Aquí tienes el **archivo completo `README.md`** con todos los pasos para crear el proyecto **desde cero**, incluyendo la instalación de Laravel, configuración del entorno, creación de migraciones, modelos, controladores, rutas, autenticación y roles.

Puedes copiarlo directamente como `README.md` en la raíz de tu proyecto.

---

````markdown
# 🚀 API de Gestión de Proyectos y Tareas (Laravel 12)

Este proyecto es una **API REST construida con Laravel 12** que permite gestionar **usuarios, proyectos y tareas** con **roles y permisos básicos** (`admin`, `project_manager`, `developer`).

---

## 🧩 Estructura del proyecto

Entidades principales:
- `users` → usuarios con roles
- `projects` → proyectos gestionados por project managers
- `tasks` → tareas asociadas a proyectos y asignadas a usuarios

---

## 🪜 Pasos para construir el proyecto

---

### 1️⃣ Crear el proyecto Laravel

```bash
composer create-project --prefer-dist laravel/laravel mi-api "12.*"
cd mi-api
````

---

### 2️⃣ Configurar el entorno

```bash
cp .env.example .env
php artisan key:generate
```

Editar el archivo `.env` para conectar la base de datos:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=mi_api
DB_USERNAME=root
DB_PASSWORD=
```

Crear la base de datos en MySQL (vacía):

```sql
CREATE DATABASE mi_api CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

### 3️⃣ Instalar Sanctum para la autenticación

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

En `app/Http/Kernel.php`, activa el middleware de Sanctum en el grupo `api`.

---

### 4️⃣ Crear las migraciones

Laravel ya incluye la de usuarios. Creamos las demás:

```bash
php artisan make:migration create_projects_table
php artisan make:migration create_tasks_table
```

---

### 5️⃣ Definir la estructura de las tablas

#### 🧍‍♂️ Tabla `users`

Editar `database/migrations/xxxx_xx_xx_create_users_table.php` y añadir:

```php
$table->string('role')->default('developer');
```

---

#### 🗂 Tabla `projects`

Archivo: `create_projects_table.php`

```php
Schema::create('projects', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->date('start_date')->nullable();
    $table->date('end_date')->nullable();
    $table->foreignId('user_id')->constrained()->onDelete('cascade'); // project_manager
    $table->timestamps();
});
```

---

#### 🧾 Tabla `tasks`

Archivo: `create_tasks_table.php`

```php
Schema::create('tasks', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('description')->nullable();
    $table->enum('status', ['pendiente', 'en_progreso', 'completada'])->default('pendiente');
    $table->foreignId('project_id')->constrained()->onDelete('cascade');
    $table->foreignId('assigned_to')->nullable()->constrained('users')->onDelete('set null');
    $table->timestamps();
});
```

---

### 6️⃣ Ejecutar migraciones

```bash
php artisan migrate
```

Comprobar con:

```bash
php artisan migrate:status
```

---

### 7️⃣ Crear los modelos

```bash
php artisan make:model Project
php artisan make:model Task
```

---

### 8️⃣ Definir relaciones en los modelos

#### `app/Models/User.php`

```php
public function projects() {
    return $this->hasMany(Project::class);
}

public function tasks() {
    return $this->hasMany(Task::class, 'assigned_to');
}
```

#### `app/Models/Project.php`

```php
class Project extends Model
{
    protected $fillable = ['name', 'description', 'start_date', 'end_date', 'user_id'];

    public function user() {
        return $this->belongsTo(User::class);
    }

    public function tasks() {
        return $this->hasMany(Task::class);
    }
}
```

#### `app/Models/Task.php`

```php
class Task extends Model
{
    protected $fillable = ['title', 'description', 'status', 'project_id', 'assigned_to'];

    public function project() {
        return $this->belongsTo(Project::class);
    }

    public function assignedUser() {
        return $this->belongsTo(User::class, 'assigned_to');
    }
}
```

---

### 9️⃣ Crear controladores

```bash
php artisan make:controller AuthController
php artisan make:controller ProjectController --api
php artisan make:controller TaskController --api
```

---

### 🔟 Rutas API

Archivo: `routes/api.php`

```php
use App\Http\Controllers\AuthController;
use App\Http\Controllers\ProjectController;
use App\Http\Controllers\TaskController;

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('projects', ProjectController::class);
    Route::apiResource('tasks', TaskController::class);
});
```

---

### 11️⃣ Crear middleware para roles

```bash
php artisan make:middleware RoleMiddleware
```

Ejemplo de lógica básica:

```php
public function handle($request, Closure $next, ...$roles)
{
    if (!in_array($request->user()->role, $roles)) {
        return response()->json(['error' => 'Acceso denegado'], 403);
    }
    return $next($request);
}
```

Registrar en `app/Http/Kernel.php`:

```php
'role' => \App\Http\Middleware\RoleMiddleware::class,
```

Usar en rutas:

```php
Route::middleware(['auth:sanctum', 'role:admin,project_manager'])->group(function () {
    Route::apiResource('projects', ProjectController::class);
});
```

---

### 12️⃣ Crear seeders opcionales

```bash
php artisan make:seeder UserSeeder
php artisan make:seeder ProjectSeeder
php artisan make:seeder TaskSeeder
```

Ejecutar:

```bash
php artisan db:seed
```

---

### 13️⃣ Probar la API

Endpoints principales (usando Postman, Insomnia o Thunder Client):

| Método | Endpoint        | Descripción                            |
| ------ | --------------- | -------------------------------------- |
| POST   | `/api/register` | Registrar usuario                      |
| POST   | `/api/login`    | Iniciar sesión (retorna token Sanctum) |
| GET    | `/api/projects` | Listar proyectos                       |
| POST   | `/api/projects` | Crear proyecto (según rol)             |
| GET    | `/api/tasks`    | Listar tareas                          |
| POST   | `/api/tasks`    | Crear tarea                            |

---

### 14️⃣ Mejoras opcionales

* Filtrar tareas por estado:
  `/api/tasks?status=en_progreso`
* Paginación:
  `Project::paginate(10);`
* Pruebas automáticas:

  ```bash
  php artisan test
  ```

---

## 🧠 Roles y permisos

| Rol                 | Permisos principales                                       |
| ------------------- | ---------------------------------------------------------- |
| **Admin**           | Acceso total a todos los recursos                          |
| **Project Manager** | Puede crear, ver y editar sus proyectos y tareas asociadas |
| **Developer**       | Solo puede ver y actualizar tareas asignadas               |

---

## ✅ Fin del setup inicial

Una vez completados todos estos pasos, tendrás la base funcional para tu API de gestión de proyectos y tareas en Laravel 12.

```

---

¿Quieres que te genere este archivo como un `.md` descargable (por ejemplo `README.md`) listo para usar en tu proyecto?
```
