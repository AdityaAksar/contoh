# Panduan Testing API Inventory Apotek di Postman

## Analisis Kode

Setelah memeriksa semua file Laravel Anda, **tidak ada error logika yang kritical**. Kode Anda sudah mengikuti standar RESTful API yang baik dengan:

✅ Proper HTTP status codes (200, 201, 404, 422)  
✅ Consistent JSON response format  
✅ Validation rules yang tepat  
✅ Relationship handling dengan eager loading  
✅ File storage management untuk gambar  
✅ CORS configuration sudah set up  

---

## Base URL

```
https://apotek.astrantia.site/api/v1
```

---

## Setup di Postman

### 1. Buat Collection Baru
- Klik **"New"** → **"Collection"** → Beri nama **"Inventory Apotek API"**
- Klik **Collections** > pilih collection yang baru dibuat → **Variables** tab
- Tambahkan variable:
  - **Key:** `base_url` | **Value:** `https://apotek.astrantia.site/api/v1`
  - **Key:** `supplier_id` | **Value:** `1` (sesuaikan dengan ID supplier di database)
  - **Key:** `obat_id` | **Value:** `1` (sesuaikan dengan ID obat di database)

---

## API Endpoints & Testing Guide

### **SUPPLIERS ENDPOINTS**

#### 1️⃣ GET All Suppliers
```
GET {{base_url}}/suppliers
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "id_supplier": 1,
      "nama_supplier": "PT Kimia Farma",
      "nomor": "021-1234567",
      "email": "info@kimiafarma.com",
      "created_at": "2025-11-15T10:00:00Z",
      "updated_at": "2025-11-15T10:00:00Z"
    }
  ]
}
```

---

#### 2️⃣ GET Supplier by ID
```
GET {{base_url}}/suppliers/{{supplier_id}}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "id_supplier": 1,
    "nama_supplier": "PT Kimia Farma",
    "nomor": "021-1234567",
    "email": "info@kimiafarma.com",
    "created_at": "2025-11-15T10:00:00Z",
    "updated_at": "2025-11-15T10:00:00Z"
  }
}
```

**Response (404 Not Found):**
```json
{
  "message": "No query results for model [App\\Models\\Supplier].",
  "exception": "ModelNotFoundException"
}
```

---

#### 3️⃣ CREATE Supplier (POST)
```
POST {{base_url}}/suppliers
```

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "nama_supplier": "PT Mitra Farmasi",
  "nomor": "031-5678910",
  "email": "supplier@mitrafarma.com"
}
```

**Response (201 Created):**
```json
{
  "status": "success",
  "message": "Supplier berhasil ditambahkan",
  "data": {
    "nama_supplier": "PT Mitra Farmasi",
    "nomor": "031-5678910",
    "email": "supplier@mitrafarma.com",
    "id_supplier": 2,
    "updated_at": "2025-11-15T10:30:00Z",
    "created_at": "2025-11-15T10:30:00Z"
  }
}
```

**Response (422 Unprocessable Entity) - Validation Error:**
```json
{
  "message": "The given data was invalid.",
  "errors": {
    "nama_supplier": ["The nama_supplier field is required."],
    "email": ["The email field must be a valid email address."]
  }
}
```

---

#### 4️⃣ UPDATE Supplier (PUT)
```
PUT {{base_url}}/suppliers/{{supplier_id}}
```

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "nama_supplier": "PT Kimia Farma Jaya",
  "nomor": "021-9876543",
  "email": "newemail@kimiafarma.com"
}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Supplier berhasil diperbarui",
  "data": {
    "id_supplier": 1,
    "nama_supplier": "PT Kimia Farma Jaya",
    "nomor": "021-9876543",
    "email": "newemail@kimiafarma.com",
    "created_at": "2025-11-15T10:00:00Z",
    "updated_at": "2025-11-15T10:45:00Z"
  }
}
```

---

#### 5️⃣ DELETE Supplier
```
DELETE {{base_url}}/suppliers/{{supplier_id}}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Supplier berhasil dihapus"
}
```

---

### **OBATS (MEDICINES) ENDPOINTS**

#### 1️⃣ GET All Medicines
```
GET {{base_url}}/obats
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "id_obat": 1,
      "nama_obat": "Paracetamol",
      "jenis_obat": "Analgesik",
      "stock": 100,
      "gambar": "obat/paracetamol.jpg",
      "id_supplier": 1,
      "created_at": "2025-11-15T10:00:00Z",
      "updated_at": "2025-11-15T10:00:00Z",
      "supplier": {
        "id_supplier": 1,
        "nama_supplier": "PT Kimia Farma",
        "nomor": "021-1234567",
        "email": "info@kimiafarma.com"
      }
    }
  ]
}
```

---

#### 2️⃣ GET Medicine by ID
```
GET {{base_url}}/obats/{{obat_id}}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "id_obat": 1,
    "nama_obat": "Paracetamol",
    "jenis_obat": "Analgesik",
    "stock": 100,
    "gambar": "obat/paracetamol.jpg",
    "id_supplier": 1,
    "created_at": "2025-11-15T10:00:00Z",
    "updated_at": "2025-11-15T10:00:00Z",
    "supplier": {
      "id_supplier": 1,
      "nama_supplier": "PT Kimia Farma",
      "nomor": "021-1234567",
      "email": "info@kimiafarma.com"
    }
  }
}
```

---

#### 3️⃣ GET Medicines by Type/Jenis
```
GET {{base_url}}/obats/jenis/Analgesik
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": [
    {
      "id_obat": 1,
      "nama_obat": "Paracetamol",
      "jenis_obat": "Analgesik",
      "stock": 100,
      "gambar": "obat/paracetamol.jpg",
      "id_supplier": 1,
      "supplier": {
        "id_supplier": 1,
        "nama_supplier": "PT Kimia Farma"
      }
    },
    {
      "id_obat": 2,
      "nama_obat": "Ibuprofen",
      "jenis_obat": "Analgesik",
      "stock": 50,
      "gambar": null,
      "id_supplier": 1,
      "supplier": {
        "id_supplier": 1,
        "nama_supplier": "PT Kimia Farma"
      }
    }
  ]
}
```

---

#### 4️⃣ CREATE Medicine (POST with File)
```
POST {{base_url}}/obats
```

**Headers:**
```
Content-Type: multipart/form-data
```

**Body (form-data):**
```
nama_obat: Paracetamol
jenis_obat: Analgesik
stock: 100
id_supplier: 1
gambar: [SELECT FILE dari komputer Anda]
```

**Cara di Postman:**
1. Pilih **Body** tab → **form-data**
2. Key pertama: `nama_obat` (type: Text) → Value: `Paracetamol`
3. Key kedua: `jenis_obat` (type: Text) → Value: `Analgesik`
4. Key ketiga: `stock` (type: Text) → Value: `100`
5. Key keempat: `id_supplier` (type: Text) → Value: `1`
6. Key kelima: `gambar` (tukan ke type: File) → Pilih file gambar

**Response (201 Created):**
```json
{
  "status": "success",
  "message": "Obat berhasil ditambahkan",
  "data": {
    "id_obat": 1,
    "nama_obat": "Paracetamol",
    "jenis_obat": "Analgesik",
    "stock": 100,
    "gambar": "obat/ABC123xyz.jpg",
    "id_supplier": 1,
    "created_at": "2025-11-15T10:00:00Z",
    "updated_at": "2025-11-15T10:00:00Z",
    "supplier": {
      "id_supplier": 1,
      "nama_supplier": "PT Kimia Farma"
    }
  }
}
```

---

#### 5️⃣ UPDATE Medicine (PUT with Optional File)
```
PUT {{base_url}}/obats/{{obat_id}}
```

**Headers:**
```
Content-Type: multipart/form-data
```

**Body (form-data):**
```
nama_obat: Paracetamol Extra
jenis_obat: Analgesik Kuat
stock: 150
id_supplier: 1
gambar: [OPTIONAL - Pilih file baru jika ingin update]
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Obat berhasil diperbarui",
  "data": {
    "id_obat": 1,
    "nama_obat": "Paracetamol Extra",
    "jenis_obat": "Analgesik Kuat",
    "stock": 150,
    "gambar": "obat/NEW_FILE_xyz.jpg",
    "id_supplier": 1,
    "updated_at": "2025-11-15T10:30:00Z"
  }
}
```

**Catatan:** Gambar lama akan otomatis dihapus jika Anda upload gambar baru.

---

#### 6️⃣ UPDATE Stock Only
```
PUT {{base_url}}/obats/{{obat_id}}/stock
```

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "stock": 75
}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Stock berhasil diperbarui",
  "data": {
    "id_obat": 1,
    "nama_obat": "Paracetamol",
    "stock": 75,
    "updated_at": "2025-11-15T10:45:00Z"
  }
}
```

---

#### 7️⃣ DELETE Medicine
```
DELETE {{base_url}}/obats/{{obat_id}}
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Obat berhasil dihapus"
}
```

**Catatan:** File gambar juga akan otomatis dihapus dari storage.

---

#### 8️⃣ DELETE Medicine Image Only
```
DELETE {{base_url}}/obats/{{obat_id}}/gambar
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Gambar berhasil dihapus",
  "data": {
    "id_obat": 1,
    "nama_obat": "Paracetamol",
    "gambar": null
  }
}
```

---

## Testing Checklist

### Supplier Testing
- [ ] GET semua suppliers
- [ ] GET supplier by ID (exist)
- [ ] GET supplier by ID (tidak ada - 404)
- [ ] POST supplier baru (valid)
- [ ] POST supplier (validation error - missing field)
- [ ] POST supplier (invalid email format)
- [ ] PUT supplier (update semua field)
- [ ] PUT supplier (update sebagian field)
- [ ] PUT supplier (invalid data)
- [ ] DELETE supplier

### Obat Testing
- [ ] GET semua obat
- [ ] GET obat by ID (with supplier relationship)
- [ ] GET obat by ID (tidak ada - 404)
- [ ] GET obat by jenis (filter by type)
- [ ] POST obat (dengan gambar)
- [ ] POST obat (tanpa gambar)
- [ ] POST obat (validation error)
- [ ] POST obat (file type tidak sesuai)
- [ ] PUT obat (update semua field + gambar baru)
- [ ] PUT obat (update tanpa gambar)
- [ ] PUT obat stock (update stock saja)
- [ ] DELETE obat (dengan gambar)
- [ ] DELETE obat gambar (hanya hapus file, data tetap)

---

## Response Status Codes

| Status | Arti | Contoh |
|--------|------|--------|
| 200 | OK - Request berhasil | GET, PUT, DELETE sukses |
| 201 | Created - Data berhasil dibuat | POST sukses |
| 400 | Bad Request | Format request salah |
| 404 | Not Found | Resource tidak ditemukan |
| 422 | Unprocessable Entity | Validation error |
| 500 | Internal Server Error | Error di server |

---

## Tips Testing di Postman

1. **Save Response sebagai Example:**
   - Jalankan request → Klik **"Save as Example"** di response panel
   - Berguna untuk referensi response format

2. **Gunakan Environment Variables:**
   - Ganti hardcode ID dengan variabel `{{supplier_id}}`, `{{obat_id}}`

3. **Test Sequential:**
   - Buat supplier dulu sebelum create obat
   - Store ID dari response untuk test berikutnya

4. **Check Headers CORS:**
   - Response harus ada header: `Access-Control-Allow-*`
   - Jika tidak ada, check `Cors.php` middleware

5. **Upload File:**
   - Pastikan format file: `jpeg, png, jpg, gif, webp`
   - Size max: 2MB

---

## Error Troubleshooting

### 404 Not Found pada Update/Delete
**Penyebab:** ID tidak ada di database
**Solusi:** 
1. GET all data dulu untuk lihat ID yang tersedia
2. Update variabel `supplier_id` atau `obat_id` di Postman

### 422 Validation Error
**Penyebab:** Data input tidak sesuai rule
**Solusi:** Check pesan error dan sesuaikan input:
- `nama_obat` harus string max 255 karakter
- `stock` harus integer >= 0
- `id_supplier` harus ada di tabel suppliers
- `email` harus format email valid

### CORS Error di Android/Frontend
**Penyebab:** CORS middleware tidak jalan
**Solusi:** 
1. Check `Cors.php` sudah di register di `app/Http/Kernel.php` sebagai middleware
2. Pastikan `Access-Control-Allow-Origin` set ke `*` atau domain specific

### File Upload Gagal
**Penyebab:** Storage tidak writable atau file besar
**Solusi:**
1. Jalankan: `php artisan storage:link`
2. Check folder `storage/app/public/obat` permissions (755)
3. File harus < 2MB

---

## Generate Postman Collection (Auto-documentation)

Untuk dokumentasi otomatis di Postman:

1. Klik **Postman Documentation** (di web)
2. Share link dokumentasi dengan tim
3. Atau: Klik collection → **"..."** → **"Share"** → **"Postman Web"**

---

## API Response Summary

Semua endpoint mengikuti format yang **konsisten**:

```json
{
  "status": "success|error",
  "message": "Optional message",
  "data": {},
  "errors": {}
}
```

HTTP Status Code mengikuti RESTful standard:
- **2xx** untuk success (GET, POST success, PUT, DELETE)
- **4xx** untuk client error (validation, not found)
- **5xx** untuk server error

---

**End of Documentation**
