# Penjaminan Mutu SI - Menfess Application

![Laravel](https://img.shields.io/badge/Laravel-10.x-FF2D20?style=flat-square&logo=laravel)
![PHP](https://img.shields.io/badge/PHP-8.1+-777BB4?style=flat-square&logo=php)
![MySQL](https://img.shields.io/badge/MySQL-8.0+-00758F?style=flat-square&logo=mysql)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

Aplikasi anonymous confession/menfess yang dibangun menggunakan **Laravel 10** dengan fitur **Like**, **Comment**, dan **Category Management**.

## 📋 Daftar Isi

- [Fitur Utama](#-fitur-utama)
- [Requirements](#-requirements)
- [Instalasi](#-instalasi)
- [Konfigurasi](#-konfigurasi)
- [Running Aplikasi](#-running-aplikasi)
- [Struktur Database](#-struktur-database)
- [API Endpoints](#-api-endpoints)
- [Testing](#-testing)
- [Troubleshooting](#-troubleshooting)
- [Tim Pengembang](#-tim-pengembang)

---

## ✨ Fitur Utama

- 🔐 **User Authentication** - Register & Login dengan session token (Sanctum)
- 📝 **Create Post** - User dapat membuat post/confession anonim dengan kategori
- ❤️ **Like/Unlike** - User dapat like post orang lain secara realtime
- 💬 **Comment** - User dapat memberikan komentar pada post
- 📂 **Category Management** - Admin dapat mengelola kategori post
- 🔍 **View All Posts** - Melihat semua post dengan real-time updates
- 🎨 **Responsive UI** - Dibangun dengan Tailwind CSS

---

## 📦 Requirements

Sebelum memulai, pastikan sudah install:

- **PHP:** 8.1 atau lebih tinggi
- **Composer:** Latest version
- **MySQL/MariaDB:** 8.0 atau lebih tinggi
- **Node.js & npm:** Untuk asset compilation (optional)
- **XAMPP/WAMP/LAMP:** Untuk development server

**Versi yang Digunakan:**
- Laravel 10.x
- PHP 8.1+
- MySQL 8.0+
- Sanctum (API Token Authentication)
- Tailwind CSS

---

## 🚀 Instalasi

### Step 1: Clone Repository

```bash
git clone https://github.com/YOUR_USERNAME/PenjaminanMutuSI_SI_A_KELOMPOK6.git
cd PenjaminanMutuSI_SI_A_KELOMPOK6
```

### Step 2: Install Dependencies

```bash
composer install
```

### Step 3: Setup Environment File

```bash
# Copy .env.example ke .env
cp .env.example .env

# Generate APP_KEY
php artisan key:generate
```

### Step 4: Database Configuration

Edit file `.env` dan sesuaikan dengan database Anda:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=penjaminan_mutu
DB_USERNAME=root
DB_PASSWORD=
```

### Step 5: Database Migration & Seeding

```bash
# Jalankan migrations
php artisan migrate

# (Optional) Jalankan seeding untuk demo data
php artisan db:seed
```

---

## ⚙️ Konfigurasi

### Setup Middleware Authentication

**File:** `app/Providers/AppServiceProvider.php`

Pastikan middleware `auth.session` sudah terdaftar di method `boot()`:

```php
public function boot(): void
{
    $this->app['router']->aliasMiddleware(
        'auth.session',
        \App\Http\Middleware\AuthFromSessionToken::class
    );
}
```

### Setup Routes

**File:** `routes/web.php`

Pastikan protected routes sudah menggunakan middleware:

```php
Route::middleware(['auth.session'])->group(function () {
    Route::post('/api/likes/{id_post}', [LikeController::class, 'toggleLike']);
    Route::post('/api/comments', [CommentController::class, 'store']);
    // ... routes lainnya
});
```

---

## 🎬 Running Aplikasi

### Development Mode

#### Windows (XAMPP):

```powershell
# Terminal 1: Start Laravel Server
php artisan serve

# Terminal 2: Start XAMPP (jika perlu)
# Buka XAMPP Control Panel → Start Apache & MySQL
```

#### Linux/Mac:

```bash
# Terminal 1: Start Laravel Server
php artisan serve

# Terminal 2: Start MySQL (jika diperlukan)
sudo systemctl start mysql
```

### Akses Aplikasi

1. Buka browser ke: `http://localhost:8000`
2. Register akun baru atau login
3. Mulai membuat & interact dengan posts!

### Compile Assets (jika menggunakan npm)

```bash
# Development
npm run dev

# Production
npm run build
```

---

## 📊 Struktur Database

### Tabel Utama

#### `users`
```
- id_user (PK)
- name
- email (unique)
- password (hashed)
- role (user/admin)
- created_at
- updated_at
```

#### `posts`
```
- id_post (PK)
- id_user (FK → users)
- id_category (FK → categories)
- content (text)
- tanggal_posting
- created_at
- updated_at
```

#### `categories`
```
- id_category (PK)
- nama_category
- created_at
- updated_at
```

#### `likes`
```
- id_like (PK)
- id_post (FK → posts)
- id_user (FK → users)
- tanggal_like
- created_at
```

#### `comments`
```
- id_comment (PK)
- id_post (FK → posts)
- id_user (FK → users)
- comment_content (text)
- tanggal_komentar
- created_at
- updated_at
```

#### `personal_access_tokens` (Sanctum)
```
- id (PK)
- tokenable_type
- tokenable_id
- name
- token (hashed)
- abilities
- last_used_at
- expires_at
- created_at
- updated_at
```

---

## 🔌 API Endpoints

### Public Endpoints

| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| GET | `/api/posts` | Get semua posts |
| GET | `/api/categories` | Get semua categories |
| GET | `/api/posts/{id_post}/comments` | Get comments by post |

### Protected Endpoints (Require Token)

| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| POST | `/api/likes/{id_post}` | Toggle like/unlike |
| GET | `/api/likes/{id_post}/is-liked` | Check if user liked |
| POST | `/api/comments` | Create comment |
| DELETE | `/api/comments/{id}` | Delete comment |

### Admin Endpoints

| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| POST | `/api/categories` | Create category |
| PUT | `/api/categories/{id}` | Update category |
| DELETE | `/api/categories/{id}` | Delete category |

---

## 🧪 Testing

### Manual Testing dengan Postman

1. **Register User:**
   - Method: POST
   - URL: `http://localhost:8000/register`
   - Body: `name`, `email`, `password`, `password_confirmation`

2. **Login:**
   - Method: POST
   - URL: `http://localhost:8000/login`
   - Body: `email`, `password`
   - Response akan berisi token

3. **Get Posts:**
   - Method: GET
   - URL: `http://localhost:8000/api/posts`
   - Header: `Authorization: Bearer {token}`

4. **Like Post:**
   - Method: POST
   - URL: `http://localhost:8000/api/likes/1`
   - Header: `Authorization: Bearer {token}`

### Browser Testing

1. Buka aplikasi di browser
2. Register/Login
3. Click tombol like/comment
4. Buka DevTools (F12) → Network tab untuk lihat API calls

---

## 🐛 Troubleshooting

### Error: "User tidak terautentikasi"

**Penyebab:** Token tidak tersimpan di session

**Solusi:**
1. Pastikan sudah login dengan benar
2. Check DevTools → Console: `console.log("{{ session('api_token') }}")`
3. Jika undefined, cek `routes/web.php` - login handler harus ada `session(['api_token' => $token])`

### Error: "Post tidak ditemukan"

**Penyebab:** Post ID tidak ada di database

**Solusi:**
1. Pastikan sudah membuat post terlebih dahulu
2. Check database apakah ada data di tabel `posts`

### Error: Database Connection

**Penyebab:** .env tidak dikonfigurasi dengan benar

**Solusi:**
```bash
# Edit .env
DB_HOST=127.0.0.1
DB_DATABASE=penjaminan_mutu
DB_USERNAME=root
DB_PASSWORD=
```

### Blank Page / 500 Error

**Penyebab:** Aplikasi key tidak generate

**Solusi:**
```bash
php artisan key:generate
php artisan cache:clear
php artisan config:clear
```

### Session Token Undefined

**Penyebab:** Session tidak tersimpan dengan benar

**Solusi (Windows PowerShell):**
```powershell
php artisan cache:clear
Remove-Item storage\framework\sessions\* -Recurse -Force
```

**Solusi (Linux/Mac):**
```bash
php artisan cache:clear
rm -rf storage/framework/sessions/*
```

---

## 📁 Project Structure

```
PenjaminanMutuSI_SI_A_KELOMPOK6/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── LikeController.php
│   │   │   ├── CommentController.php
│   │   │   ├── PostController.php
│   │   │   └── CategoryController.php
│   │   └── Middleware/
│   │       └── AuthFromSessionToken.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── Post.php
│   │   ├── Like.php
│   │   ├── Comment.php
│   │   └── Category.php
│   └── Providers/
│       └── AppServiceProvider.php
├── routes/
│   ├── web.php
│   └── api.php
├── resources/
│   ├── views/
│   │   ├── home.blade.php
│   │   ├── posts/
│   │   │   └── create.blade.php
│   │   └── auth/
│   │       ├── login.blade.php
│   │       └── register.blade.php
│   └── css/
│       └── app.css
├── storage/
│   ├── framework/
│   │   └── sessions/
│   └── logs/
├── .env
├── composer.json
├── package.json
└── README.md
```

---

## 👥 Tim Pengembang

| Nama | Role | GitHub |
|------|------|--------|
| Kelompok 6 | Development Team | [@username](https://github.com/username) |

---

## 📄 License

Aplikasi ini menggunakan license **MIT**. Lihat file `LICENSE` untuk detail lengkap.

---

## 📝 Catatan Penting

1. **Jangan commit `.env` ke repository** - Selalu gunakan `.env.example`
2. **Backup database secara berkala** - Terutama sebelum production
3. **Use HTTPS di production** - Token akan lebih aman
4. **Setup proper permissions** - Pastikan `storage/` dan `bootstrap/cache/` writable
5. **Enable CORS jika perlu** - Untuk API dari domain berbeda

---

## 🆘 Support & Help

Jika ada masalah, silakan:

1. Baca dokumentasi di atas
2. Check GitHub Issues
3. Buat issue baru dengan detail error
4. Contact tim pengembang

---

## 🎉 Selesai!

Selamat! Aplikasi Menfess sudah siap digunakan. Nikmati fitur-fiturnya dan jangan ragu untuk berkontribusi! 🚀

