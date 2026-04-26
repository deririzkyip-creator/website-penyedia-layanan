# 📋 Admin Login & Maintenance Mode Guide

## 🎯 Konsep Utama

Admin **TIDAK TERKENA DAMPAK** maintenance mode karena ada **3 bypass layer**:

### Layer 1: Login Page Bypass
```
Route: /login
Filter: MaintenanceFilter
Status: ✅ BYPASS
Akses: Halaman login selalu accessible
```

### Layer 2: Admin Route Bypass
```
Route: /admin*
Filter: MaintenanceFilter  
Status: ✅ BYPASS
Akses: Panel admin selalu accessible
```

### Layer 3: Maintenance Page Exception
```
Route: /maintenance
Filter: MaintenanceFilter
Status: ✅ BYPASS
Akses: Halaman maintenance sendiri selalu accessible
```

---

## 🔐 Step-by-Step: Admin Login Saat Maintenance Aktif

### Scenario: Website dalam Maintenance Mode (is_active=1)

#### Step 1️⃣: Pengunjung vs Admin

**Pengunjung (Tidak Login):**
```
Akses ke: http://website.com/
↓ (Filter Maintenance)
Melihat: Halaman Maintenance (Status 503)
❌ Tidak bisa akses halaman lain
```

**Admin (Login):**
```
Akses ke: http://website.com/login
↓ (Filter cek route)
✅ Route = "login" → BYPASS FILTER
Melihat: Halaman Login Normal ✅
```

#### Step 2️⃣: Admin Login

```
1. Buka http://website.com/login (atau http://website.com/admin/login)
2. Masukkan username & password admin
3. Klik Submit
4. ✅ Login berhasil
5. Session 'isLoggedIn' = true
6. Redirect ke /admin
```

#### Step 3️⃣: Admin Akses Panel

```
Akses ke: http://website.com/admin
↓ (Filter cek route)
✅ Route starts with "admin" → BYPASS FILTER
Melihat: Admin Panel Normal (Semua fitur loading) ✅
```

#### Step 4️⃣: Kelola Maintenance Mode

**Di Admin Panel:**
```
Section: "Mode Maintenance"
├─ Lihat status: is_active = 1 (AKTIF)
├─ Toggle: ON/OFF
├─ Edit Title: "Website Sedang Maintenance"
├─ Edit Message: Pesan untuk pengunjung
├─ Estimasi: "1-2 jam"
└─ Submit → Database Updated
```

---

## 📝 Code Implementation

### MaintenanceFilter.php - Bypass Logic

```php
public function before(RequestInterface $request, $arguments = null)
{
    $uri = $request->getUri();
    $path = trim($uri->getPath(), '/');

    // ✅ LAYER 1: Allow admin routes
    if (strpos($path, 'admin') === 0) {
        return; // Bypass filter
    }

    // ✅ LAYER 2: Allow login page
    if ($path === 'login') {
        return; // Bypass filter
    }

    // ✅ LAYER 3: Allow maintenance page itself
    if ($path === 'maintenance') {
        return; // Bypass filter
    }

    // Check if maintenance active untuk pengunjung lain
    if ($maintenanceModel->isMaintenanceActive()) {
        return view('maintenance', [...]); // Tampil maintenance
    }
}
```

### Routes.php

```php
// Login routes - Always accessible
$routes->get('login', 'Auth::login');
$routes->post('login', 'Auth::attemptLogin');
$routes->get('admin/login', 'Auth::login');
$routes->post('admin/login', 'Auth::attemptLogin');

// Admin routes - Always accessible (maintenance bypass)
$routes->get('admin', 'Admin::index');
$routes->post('admin/maintenance/toggle', 'Admin::toggleMaintenance');
// ... semua admin routes lainnya
```

---

## 🔄 Full Workflow Saat Maintenance Mode Aktif

```
┌─────────────────────────────────────────────────────────┐
│        WEBSITE DALAM MAINTENANCE MODE (is_active=1)    │
└─────────────────────────────────────────────────────────┘

PENGUNJUNG BIASA:
━━━━━━━━━━━━━━━━━
Akses /          →  [Maintenance Filter]  →  ❌ Maintenance Page (503)
Akses /services  →  [Maintenance Filter]  →  ❌ Maintenance Page (503)
Akses /privacy   →  [Maintenance Filter]  →  ❌ Maintenance Page (503)

ADMIN (Belum Login):
━━━━━━━━━━━━━━━━━━━
Akses /login     →  [Maintenance Filter - BYPASS login] →  ✅ Login Page Normal
Login Credentials →  ✅ Session set → Redirect ke /admin

ADMIN (Sudah Login):
━━━━━━━━━━━━━━━━━━
Akses /admin     →  [Maintenance Filter - BYPASS admin] →  ✅ Admin Panel Normal
Edit Content     →  ✅ Semua fitur berfungsi normal
Lihat Mode Maintenance Form  →  ✅ Bisa toggle maintenance OFF
Toggle OFF       →  is_active=0 → Database updated

WEBSITE NORMAL LAGI:
━━━━━━━━━━━━━━━━━━━
Pengunjung Akses /     →  ✅ Home Page Normal
Pengunjung Akses /services  →  ✅ Services Page Normal
Admin Akses /admin     →  ✅ Admin Panel Normal
```

---

## 🛡️ Security: Admin Bypass Mechanism

### 1. Path-Based Bypass (Secure)
```php
// ✅ Aman: Hanya route tertentu yang bypass
if (strpos($path, 'admin') === 0 || $path === 'login')
```

**Bypass untuk:**
- ✅ `/login` → Halaman login
- ✅ `/admin` → Admin panel
- ✅ `/admin/services/save` → Save services
- ✅ `/admin/maintenance/toggle` → Toggle maintenance
- ✅ Semua admin sub-routes

**TIDAK bypass untuk:**
- ❌ `/` → Home page (dipaksa maintenance)
- ❌ `/services` → Services page (dipaksa maintenance)
- ❌ `/products` → Products page (dipaksa maintenance)

### 2. Session-Based Protection (Admin Controller)
```php
public function index()
{
    // Additional check di controller
    if (!session()->get('isLoggedIn')) {
        return redirect()->to(site_url('login'));
    }
    
    // Load admin panel
    return view('admin', [...]);
}
```

### 3. Error Handling & Caching
```php
try {
    $isActive = $maintenanceModel->isMaintenanceActive();
} catch (\Exception $e) {
    // Fallback ke cache file jika DB error
    $cached = $this->getCachedMaintenanceStatus();
    $isActive = $cached['is_active'] ?? false;
}
```

---

## 📊 Tabel: Routes & Bypass Status

| Route | Admin? | Filter Bypass? | Maintenance Active (1) | Maintenance Inactive (0) |
|-------|--------|----------------|------------------------|--------------------------|
| `/` | ❌ | ❌ NO | 🔴 Maintenance Page | 🟢 Home Page |
| `/services` | ❌ | ❌ NO | 🔴 Maintenance Page | 🟢 Services Page |
| `/login` | ❌ | ✅ YES | 🟢 Login Page | 🟢 Login Page |
| `/admin` | ✅ | ✅ YES | 🟢 Admin Panel | 🟢 Admin Panel |
| `/admin/maintenance/toggle` | ✅ | ✅ YES | 🟢 Toggle Works | 🟢 Toggle Works |
| `/maintenance` | - | ✅ YES | 🟢 Maintenance Page | 🟢 Maintenance Page |

---

## 🎯 Quick Reference: Admin Actions When Maintenance Active

### ✅ Admin CAN Do:

```
1. Akses http://website.com/login
2. Login dengan username & password
3. Akses http://website.com/admin (Admin Panel)
4. Lihat Mode Maintenance section
5. Ubah Title maintenance
6. Ubah Message maintenance
7. Ubah Estimasi time
8. Toggle maintenance OFF
9. Edit content normal
10. Manage layanan, produk, FAQ, dll
```

### ❌ Pengunjung CANNOT Do (Saat Maintenance Aktif):

```
1. Akses halaman /
2. Akses halaman /services
3. Akses halaman /products
4. Akses halaman /help
5. Akses halaman /privacy
6. Navigasi ke mana saja
→ Semua redirect ke Maintenance Page
```

---

## 🔧 Disable Maintenance Mode

### Via Admin Panel:
```
1. Login ke /admin
2. Scroll ke "Mode Maintenance"
3. Toggle OFF (uncheck checkbox)
4. Click "Update & Simpan"
5. ✅ Website normal lagi
```

### Via Command Line:
```bash
cd ci4
php spark db:seed DisableMaintenanceSeeder
```

### Via Database Query:
```sql
UPDATE maintenance SET is_active = 0 WHERE id = 1;
```

---

## 📋 Troubleshooting

### Q: Admin tidak bisa login saat maintenance aktif?
**A:** Pastikan filter punya bypass untuk `/login` route. Cek MaintenanceFilter.php line 20.

### Q: Admin login tapi maintenance page tetap tampil?
**A:** Cek session `isLoggedIn`. Admin harus login dulu sebelum akses `/admin`.

### Q: Admin akses /admin tapi maintenance filter masih jalan?
**A:** Cek filter config di `Config/Filters.php` - pastikan maintenance filter terdaftar di `$globals['before']`.

### Q: Ingin admin dari IP tertentu bypass maintenance?
**A:** Edit MaintenanceFilter.php tambah:
```php
if ($_SERVER['REMOTE_ADDR'] === '192.168.1.100') {
    return; // Bypass maintenance untuk IP tertentu
}
```

---

## 📌 Kesimpulan

✅ **Admin Login**: Selalu bisa login via `/login` (bypass filter)
✅ **Admin Panel**: Selalu bisa akses `/admin` (bypass filter)
✅ **Manage Maintenance**: Toggle ON/OFF dari admin panel
✅ **Toggle OFF**: Website langsung normal lagi
✅ **Toggle ON**: Website langsung maintenance mode

**Status**: FULLY FUNCTIONAL ✅
