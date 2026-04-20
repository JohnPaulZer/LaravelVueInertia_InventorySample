# Simple Inventory Setup Guide

Easy step-by-step setup for Laravel + Vue + Inertia + PostgreSQL on Windows using Laravel Herd.

## Quick Setup (Fast Path)

If you already installed Herd, Node.js, and PostgreSQL, run this in project root:

```bash
composer install
npm install
copy .env.example .env
php artisan key:generate
php artisan migrate
composer run dev
```

Then open `http://127.0.0.1:8000`.

## Terminal Commands Only (Copy-Paste)

Use these if you want the easiest terminal flow.

### First time setup (run once)

```cmd
cd path\to\Simple_Inventory
composer install
npm install
copy .env.example .env
php artisan key:generate
php artisan migrate
```

### Start project every day

```cmd
cd path\to\Simple_Inventory
composer run dev
```

### Optional: split into separate terminals

Terminal 1:

```cmd
cd path\to\Simple_Inventory
npm run dev
```

Terminal 2:

```cmd
cd path\to\Simple_Inventory
php artisan queue:listen --tries=1
```

## Full Setup (Beginner Friendly)

## 1) Install Required Apps

Install these on your Windows machine:

1. Laravel Herd
2. Node.js LTS (includes npm)
3. PostgreSQL (with command-line tools)
4. Git

After installing, open a new terminal and check:

```bash
php -v
composer -V
node -v
npm -v
psql --version
git --version
```

If one command is missing, install that app first.

## 2) Make Sure Herd Uses Correct PHP Version

Inside Herd, use PHP 8.2 or higher.

Why:

- This project requires modern Laravel/PHP features.

## 3) Open the Project Folder

If you already have the project:

```bash
cd path\to\Simple_Inventory
```

If you still need to clone:

```bash
git clone https://github.com/JohnPaulZer/LaravelVueInertia_InventorySample.git
cd LaravelVueInertia_InventorySample
cd Simple_Inventory
```

## 4) Install Project Dependencies

Run:

```bash
composer install
npm install
```

What these do:

- `composer install`: installs backend (Laravel/PHP) packages
- `npm install`: installs frontend (Vue/Inertia/Vite) packages

## 5) Create PostgreSQL Database and User

### Option A: Using psql

Open PostgreSQL shell:

```bash
psql -U postgres -h 127.0.0.1 -p 5432
```

Run:

```sql
CREATE DATABASE simple_inventory;
CREATE USER inventory_user WITH PASSWORD 'inventory_pass_123';
GRANT ALL PRIVILEGES ON DATABASE simple_inventory TO inventory_user;
\q
```

### Option B: Using pgAdmin

1. Create database named `simple_inventory`
2. Create user `inventory_user`
3. Set password `inventory_pass_123` (or your own)
4. Give this user privileges for `simple_inventory`

## 6) Configure .env File

If `.env` does not exist, create it:

```bash
copy .env.example .env
```

Update database values in `.env`:

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=simple_inventory
DB_USERNAME=inventory_user
DB_PASSWORD=inventory_pass_123
```

Keep these values as they are (project defaults):

```env
QUEUE_CONNECTION=database
SESSION_DRIVER=database
CACHE_STORE=database
```

## 7) Generate Key and Run Migrations

Run:

```bash
php artisan key:generate
php artisan migrate
```

This prepares the app and creates required tables.

## 8) Start the App

Recommended (single command):

```bash
composer run dev
```

This starts:

- Laravel app server
- Queue listener
- Vite frontend server

Open `http://127.0.0.1:8000`.

## 9) How to Know It Works

You are successful when:

1. Browser opens app without error page
2. Terminal has no startup errors
3. Editing a Vue page updates in browser after save (Vite running)

Optional checks:

```bash
php artisan route:list
npm run build
```

## 10) Mini Tutorial: One Complete File That Connects Everything

This section gives one complete controller file sample.
That single file is the bridge between route, PostgreSQL (through model), and Inertia Vue page.

### How the connection works

1. Browser calls route like `GET /products` or `POST /products`
2. Route sends request to `ProductController`
3. Controller uses `Product` model (Eloquent)
4. Eloquent reads/writes PostgreSQL table `products`
5. Controller returns Inertia page `products/Index` with data
6. Vue renders the data and submits forms back to Laravel

### Small supporting setup (required)

Route in `routes/web.php`:

```php
Route::resource('products', ProductController::class);
```

Model in `app/Models/Product.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    protected $fillable = ['name', 'sku', 'quantity', 'price'];
}
```

### One complete connector file: ProductController

Use this as full sample in `app/Http/Controllers/ProductController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Inertia\Inertia;
use Inertia\Response;

class ProductController extends Controller
{
    public function index(): Response
    {
        return Inertia::render('products/Index', [
            'products' => Product::latest()->get(),
        ]);
    }

    public function store(Request $request): RedirectResponse
    {
        $validated = $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'sku' => ['required', 'string', 'max:255', 'unique:products,sku'],
            'quantity' => ['required', 'integer', 'min:0'],
            'price' => ['required', 'numeric', 'min:0'],
        ]);

        Product::create($validated);

        return redirect()->route('products.index')->with('success', 'Product created successfully.');
    }

    public function update(Request $request, Product $product): RedirectResponse
    {
        $validated = $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'sku' => ['required', 'string', 'max:255', 'unique:products,sku,'.$product->id],
            'quantity' => ['required', 'integer', 'min:0'],
            'price' => ['required', 'numeric', 'min:0'],
        ]);

        $product->update($validated);

        return redirect()->route('products.index')->with('success', 'Product updated successfully.');
    }

    public function destroy(Product $product): RedirectResponse
    {
        $product->delete();

        return redirect()->route('products.index')->with('success', 'Product deleted successfully.');
    }
}
```

### PostgreSQL explanation (how this actually saves data)

When you submit the form:

1. Vue sends `POST /products`
2. `store()` validates input
3. `Product::create($validated)` runs an SQL `INSERT` on PostgreSQL
4. Row is saved into `products` table
5. Redirect back to `products.index`
6. `index()` runs SQL `SELECT` through Eloquent and sends fresh data to Vue

Your DB connection for this flow comes from `.env`:

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=simple_inventory
DB_USERNAME=inventory_user
DB_PASSWORD=inventory_pass_123
```

### Quick check in PostgreSQL terminal

Run this after creating products:

```bash
psql -U inventory_user -h 127.0.0.1 -p 5432 -d simple_inventory -c "SELECT id, name, sku, quantity, price FROM products ORDER BY id DESC;"
```

If rows show up, your Laravel -> Controller -> Model -> PostgreSQL connection is working.

## 11) Common Problems (Quick Fix)

### Problem: `could not find driver`

Fix:

- Enable PostgreSQL PHP extension (`pdo_pgsql`) in Herd PHP
- Restart Herd and terminal

### Problem: `SQLSTATE[08006]` or connection refused

Fix:

- Start PostgreSQL service
- Recheck `.env` DB values

### Problem: frontend assets not loading

Fix:

- Run `npm run dev` or `composer run dev`

### Problem: app key missing

Fix:

```bash
php artisan key:generate
```

## Daily Commands You Will Use

```bash
php artisan optimize:clear
php artisan test --compact
npm run lint
npm run format:check
```

## Notes

- This repository already has Vue + Inertia configured.
- You only need correct environment setup to run it.
