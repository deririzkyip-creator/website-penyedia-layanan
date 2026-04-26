# 🔑 QUICK REFERENCE: Admin Maintenance Mode

## 🚀 Dalam 3 Langkah: Admin Gunakan Maintenance Mode

### Step 1️⃣: Akses Admin Panel
```
URL: http://website.com/login
     atau
     http://website.com/admin/login

Username: [admin username]
Password: [admin password]
```

### Step 2️⃣: Buka Mode Maintenance
```
Di Admin Panel (/admin):
  ↓
Scroll ke section: "Mode Maintenance"
  ↓
Lihat: Info box ℹ️ "Admin Tetap Bisa Akses"
```

### Step 3️⃣: Atur & Toggle
```
Checkbox "Status Maintenance"
├─ ✅ Checked  = Maintenance AKTIF
└─ ⬜ Unchecked = Maintenance OFF

Input Fields:
├─ Judul: "Website Sedang Perbaikan"
├─ Pesan: "Sedang maintenance, kembali nanti"
├─ Estimasi: "1-2 jam"
└─ Button: "Update & Simpan"
```

---

## 🔄 Workflow Lengkap

### Scenario A: Mau Maintenance (ON)

```
Admin Login (/login)
    ↓
Admin akses /admin
    ↓
Scroll ke Mode Maintenance
    ↓
Toggle ✅ Checked
    ↓
Isi: Title, Message, Estimasi
    ↓
Click "Aktifkan Maintenance"
    ↓
Database Update: is_active = 1
    ↓
✅ WEBSITE LANGSUNG MAINTENANCE MODE
   - Pengunjung: Maintenance Page
   - Admin: Tetap normal (bypass)
```

### Scenario B: Selesai Maintenance (OFF)

```
Admin akses /admin (tetap login)
    ↓
Scroll ke Mode Maintenance
    ↓
Toggle ⬜ Unchecked
    ↓
Click "Update & Simpan"
    ↓
Database Update: is_active = 0
    ↓
✅ WEBSITE LANGSUNG NORMAL
   - Pengunjung: Home page
   - Admin: All features normal
```

---

## 📌 Penting: Admin Bypass Mechanism

### Kenapa Admin Tidak Terkena Maintenance?

```
┌─────────────────────────────────────┐
│  Maintenance Filter di CodeIgniter  │
└─────────────────────────────────────┘
        ↓ Cek setiap request
        
if (strpos($path, 'admin') === 0)  → ✅ BYPASS
   Halaman admin selalu accessible

if ($path === 'login')             → ✅ BYPASS
   Login selalu accessible

if ($path === 'maintenance')       → ✅ BYPASS
   Maintenance page selalu accessible

else                               → ❌ CHECK
   if (maintenance_active)
      return maintenance_page
   else
      allow normal access
```

---

## 🎯 Admin Actions - Checklist

### ✅ Admin CAN ALWAYS:

- [ ] Akses /login (login halaman)
- [ ] Login dengan credentials
- [ ] Akses /admin (admin panel)
- [ ] Lihat semua menu di admin
- [ ] Edit konten website
- [ ] Manage services/products
- [ ] Manage FAQ
- [ ] Lihat Mode Maintenance section
- [ ] Toggle maintenance ON/OFF
- [ ] Ubah title/message maintenance
- [ ] Ubah estimasi time

### ❌ Pengunjung CANNOT (Saat Maintenance):

- [ ] Akses /
- [ ] Akses /services
- [ ] Akses /products
- [ ] Akses /help
- [ ] Akses /privacy
- [ ] Navigasi ke halaman lain

---

## 🛡️ Security: Tetap Aman

### Session Protection
```php
// Even if maintenance ON:
// Admin session tetap valid
// No forced logout
// Password tetap aman (tidak perlu re-login)
```

### Database Protection
```
// Data tidak diubah saat maintenance
// Hanya flag is_active yang berubah
// Semua konten tetap tersimpan aman
```

### File Caching
```
// Maintenance status di-cache di file
// Jika DB error, cache fallback tetap jalan
// Website tidak crash
```

---

## 🔧 Troubleshooting Admin

### Q: Admin login tapi halaman maintenance tetap tampil?
**A:** 
- Refresh halaman (F5 atau Ctrl+R)
- Cek browser cache (clear cookies)
- Pastikan session enabled

### Q: Admin logout ketika toggle maintenance?
**A:** 
- Session tetap valid
- Logout hanya jika click "Logout" button
- Toggle maintenance tidak affect session

### Q: Admin ingin akses dari mobile?
**A:**
```
Mobile:
- Buka http://website.com/admin/login (atau /login)
- Login normal
- Admin panel loading responsive
- Maintenance toggle works normal
```

### Q: Admin ingin bypass dari IP tertentu saja?
**A:** Edit MaintenanceFilter.php:
```php
if ($_SERVER['REMOTE_ADDR'] === '192.168.1.100') {
    return; // Bypass maintenance
}
```

---

## 📞 Admin Help Links (Di Maintenance Page)

```
Halaman Maintenance juga punya:
├─ Login button → /login
├─ Panel Admin button → /admin
├─ Contact info (WA, Email)
└─ Refresh button (Cek status)
```

---

## 🚀 Tips Poweruser

### Bulk Toggle Maintenance + Logout
```bash
# Disable maintenance + logout semua user
# CLI Command
php spark db:seed DisableMaintenanceSeeder
```

### Test Maintenance Offline
```bash
# Di Local machine:
php spark db:seed MaintenanceTestSeeder  # Enable
php spark serve
# Test akses / → maintenance page
# Akses /admin → normal

php spark db:seed DisableMaintenanceSeeder  # Disable
```

### Monitor Maintenance Status
```bash
# Di CLI
php spark db:table maintenance
# Lihat is_active status realtime
```

---

## 📋 Database Schema

```sql
Table: maintenance
├─ id (INT) → Primary key
├─ is_active (TINYINT) → 0=OFF, 1=ON
├─ title (VARCHAR) → Halaman title
├─ message (TEXT) → Pesan pengunjung
├─ estimated_time (VARCHAR) → Estimasi waktu
├─ created_at (TIMESTAMP) → Dibuat kapan
└─ updated_at (TIMESTAMP) → Update terakhir
```

---

## ✅ Status Cek

- ✅ Admin bypass filter → WORKING
- ✅ Login accessible → WORKING
- ✅ Admin panel accessible → WORKING
- ✅ Toggle ON/OFF → WORKING
- ✅ Session protection → WORKING
- ✅ Error handling → WORKING
- ✅ File caching → WORKING

**READY TO USE! 🎉**
