# Panduan Lengkap Migrasi ke REST API
## Aplikasi Inventory Obat Android → Laravel API

---

## FASE 1: SETUP AWAL DAN DEPENDENCY

### Step 1.1: Update build.gradle (Module: app)

Tambahkan dependency berikut di dalam blok `dependencies`:

```gradle
// Retrofit untuk HTTP Client
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'

// OkHttp untuk logging request/response
implementation 'com.squareup.okhttp3:okhttp:4.10.0'
implementation 'com.squareup.okhttp3:logging-interceptor:4.10.0'

// Coroutines untuk async task
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4'

// Lifecycle & LiveData
implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.5.1'
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.5.1'

// Gson untuk JSON parsing
implementation 'com.google.code.gson:gson:2.10.1'
```

Sync Gradle dan tunggu hingga selesai.

### Step 1.2: Tambahkan Internet Permission

Di `AndroidManifest.xml`, tambahkan di bagian `<manifest>` dan sebelum `<application>`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

---

## FASE 2: BUAT LAYER NETWORKING (API SERVICE)

### Step 2.1: Buat Retrofit Client Configuration

**File: `RetrofitClient.java`**

```java
package com.example.inventoryobat.network;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import okhttp3.OkHttpClient;
import okhttp3.logging.HttpLoggingInterceptor;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class RetrofitClient {

    private static final String BASE_URL = "https://apotek.astrantia.site/api/v1/";
    private static Retrofit retrofit;

    public static Retrofit getClient() {
        if (retrofit == null) {
            // Logging Interceptor untuk debugging
            HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
            loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);

            OkHttpClient okHttpClient = new OkHttpClient.Builder()
                    .addInterceptor(loggingInterceptor)
                    .connectTimeout(30, java.util.concurrent.TimeUnit.SECONDS)
                    .readTimeout(30, java.util.concurrent.TimeUnit.SECONDS)
                    .writeTimeout(30, java.util.concurrent.TimeUnit.SECONDS)
                    .build();

            Gson gson = new GsonBuilder()
                    .setLenient()
                    .create();

            retrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .client(okHttpClient)
                    .addConverterFactory(GsonConverterFactory.create(gson))
                    .build();
        }
        return retrofit;
    }
}
```

### Step 2.2: Buat Response Model untuk API

**File: `ApiResponse.java`** (wrapper untuk semua response API)

```java
package com.example.inventoryobat.network;

import java.util.List;

public class ApiResponse<T> {
    public String status;
    public String message;
    public T data;

    public ApiResponse(String status, String message, T data) {
        this.status = status;
        this.message = message;
        this.data = data;
    }
}
```

### Step 2.3: Update Model POJO untuk Parsing JSON

**File: `Obat.java` (refactor)**

```java
package com.example.inventoryobat.model;

import com.google.gson.annotations.SerializedName;

public class Obat {
    @SerializedName("id_obat")
    private int idObat;

    @SerializedName("nama_obat")
    private String namaObat;

    @SerializedName("jenis_obat")
    private String jenisObat;

    private int stock;
    private String gambar;

    @SerializedName("id_supplier")
    private int idSupplier;

    private Supplier supplier;

    @SerializedName("gambar_url")
    private String gambarUrl;

    // Constructor
    public Obat() {}

    public Obat(int idObat, String namaObat, String jenisObat, int stock, 
                String gambar, int idSupplier, Supplier supplier) {
        this.idObat = idObat;
        this.namaObat = namaObat;
        this.jenisObat = jenisObat;
        this.stock = stock;
        this.gambar = gambar;
        this.idSupplier = idSupplier;
        this.supplier = supplier;
    }

    // Getter & Setter
    public int getIdObat() { return idObat; }
    public void setIdObat(int idObat) { this.idObat = idObat; }

    public String getNamaObat() { return namaObat; }
    public void setNamaObat(String namaObat) { this.namaObat = namaObat; }

    public String getJenisObat() { return jenisObat; }
    public void setJenisObat(String jenisObat) { this.jenisObat = jenisObat; }

    public int getStock() { return stock; }
    public void setStock(int stock) { this.stock = stock; }

    public String getGambar() { return gambar; }
    public void setGambar(String gambar) { this.gambar = gambar; }

    public int getIdSupplier() { return idSupplier; }
    public void setIdSupplier(int idSupplier) { this.idSupplier = idSupplier; }

    public Supplier getSupplier() { return supplier; }
    public void setSupplier(Supplier supplier) { this.supplier = supplier; }

    public String getGambarUrl() { return gambarUrl; }
    public void setGambarUrl(String gambarUrl) { this.gambarUrl = gambarUrl; }
}
```

**File: `Supplier.java` (refactor)**

```java
package com.example.inventoryobat.model;

import com.google.gson.annotations.SerializedName;

public class Supplier {
    @SerializedName("id_supplier")
    private int idSupplier;

    @SerializedName("nama_supplier")
    private String namaSupplier;

    private String nomor;
    private String email;

    // Constructor
    public Supplier() {}

    public Supplier(int idSupplier, String namaSupplier, String nomor, String email) {
        this.idSupplier = idSupplier;
        this.namaSupplier = namaSupplier;
        this.nomor = nomor;
        this.email = email;
    }

    // Getter & Setter
    public int getIdSupplier() { return idSupplier; }
    public void setIdSupplier(int idSupplier) { this.idSupplier = idSupplier; }

    public String getNamaSupplier() { return namaSupplier; }
    public void setNamaSupplier(String namaSupplier) { this.namaSupplier = namaSupplier; }

    public String getNomor() { return nomor; }
    public void setNomor(String nomor) { this.nomor = nomor; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

### Step 2.4: Buat Retrofit API Interface

**File: `InventoryApiService.java`**

```java
package com.example.inventoryobat.network;

import com.example.inventoryobat.model.Obat;
import com.example.inventoryobat.model.Supplier;
import java.util.List;
import okhttp3.MultipartBody;
import okhttp3.RequestBody;
import retrofit2.Call;
import retrofit2.http.*;

public interface InventoryApiService {

    // ============ SUPPLIER ENDPOINTS ============

    @GET("suppliers")
    Call<ApiResponse<List<Supplier>>> getAllSuppliers();

    @GET("suppliers/{id}")
    Call<ApiResponse<Supplier>> getSupplierById(@Path("id") int id);

    @POST("suppliers")
    Call<ApiResponse<Supplier>> createSupplier(@Body Supplier supplier);

    @PUT("suppliers/{id}")
    Call<ApiResponse<Supplier>> updateSupplier(
            @Path("id") int id,
            @Body Supplier supplier
    );

    @DELETE("suppliers/{id}")
    Call<ApiResponse<Object>> deleteSupplier(@Path("id") int id);

    // ============ OBAT ENDPOINTS ============

    @GET("obats")
    Call<ApiResponse<List<Obat>>> getAllObat();

    @GET("obats/{id}")
    Call<ApiResponse<Obat>> getObatById(@Path("id") int id);

    @GET("obats/jenis/{jenis}")
    Call<ApiResponse<List<Obat>>> getObatByJenis(@Path("jenis") String jenis);

    @POST("obats")
    @Multipart
    Call<ApiResponse<Obat>> createObat(
            @Part("nama_obat") RequestBody namaObat,
            @Part("jenis_obat") RequestBody jenisObat,
            @Part("stock") RequestBody stock,
            @Part("id_supplier") RequestBody idSupplier,
            @Part MultipartBody.Part gambar
    );

    @PUT("obats/{id}")
    @Multipart
    Call<ApiResponse<Obat>> updateObat(
            @Path("id") int id,
            @Part("nama_obat") RequestBody namaObat,
            @Part("jenis_obat") RequestBody jenisObat,
            @Part("stock") RequestBody stock,
            @Part("id_supplier") RequestBody idSupplier,
            @Part(required = false) MultipartBody.Part gambar
    );

    @PUT("obats/{id}/stock")
    Call<ApiResponse<Obat>> updateStock(
            @Path("id") int id,
            @Body StockUpdateRequest request
    );

    @DELETE("obats/{id}")
    Call<ApiResponse<Object>> deleteObat(@Path("id") int id);

    @DELETE("obats/{id}/gambar")
    Call<ApiResponse<Obat>> deleteObatGambar(@Path("id") int id);

    // Stock update request model
    class StockUpdateRequest {
        public int stock;

        public StockUpdateRequest(int stock) {
            this.stock = stock;
        }
    }
}
```

---

## FASE 3: BUAT REPOSITORY LAYER

### Step 3.1: Buat ObatRepository

**File: `ObatRepository.java`**

```java
package com.example.inventoryobat.repository;

import android.content.Context;
import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import com.example.inventoryobat.model.Obat;
import com.example.inventoryobat.network.ApiResponse;
import com.example.inventoryobat.network.InventoryApiService;
import com.example.inventoryobat.network.RetrofitClient;
import okhttp3.MediaType;
import okhttp3.MultipartBody;
import okhttp3.RequestBody;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.List;

public class ObatRepository {

    private static final String TAG = "ObatRepository";
    private final InventoryApiService apiService;
    private final MutableLiveData<List<Obat>> obatListLiveData;
    private final MutableLiveData<Obat> obatDetailLiveData;
    private final MutableLiveData<String> errorLiveData;
    private final MutableLiveData<Boolean> loadingLiveData;

    public ObatRepository() {
        this.apiService = RetrofitClient.getClient().create(InventoryApiService.class);
        this.obatListLiveData = new MutableLiveData<>();
        this.obatDetailLiveData = new MutableLiveData<>();
        this.errorLiveData = new MutableLiveData<>();
        this.loadingLiveData = new MutableLiveData<>();
    }

    // ============ GET ALL OBAT ============
    public LiveData<List<Obat>> getAllObat() {
        loadingLiveData.setValue(true);
        Call<ApiResponse<List<Obat>>> call = apiService.getAllObat();

        call.enqueue(new Callback<ApiResponse<List<Obat>>>() {
            @Override
            public void onResponse(Call<ApiResponse<List<Obat>>> call, 
                                   Response<ApiResponse<List<Obat>>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null) {
                    obatListLiveData.setValue(response.body().data);
                } else {
                    errorLiveData.setValue("Gagal memuat data obat");
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<List<Obat>>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
            }
        });

        return obatListLiveData;
    }

    // ============ GET OBAT BY JENIS ============
    public LiveData<List<Obat>> getObatByJenis(String jenisObat) {
        loadingLiveData.setValue(true);
        Call<ApiResponse<List<Obat>>> call = apiService.getObatByJenis(jenisObat);

        call.enqueue(new Callback<ApiResponse<List<Obat>>>() {
            @Override
            public void onResponse(Call<ApiResponse<List<Obat>>> call, 
                                   Response<ApiResponse<List<Obat>>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null) {
                    obatListLiveData.setValue(response.body().data);
                } else {
                    errorLiveData.setValue("Gagal memfilter obat");
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<List<Obat>>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
            }
        });

        return obatListLiveData;
    }

    // ============ GET OBAT BY ID ============
    public LiveData<Obat> getObatById(int id) {
        loadingLiveData.setValue(true);
        Call<ApiResponse<Obat>> call = apiService.getObatById(id);

        call.enqueue(new Callback<ApiResponse<Obat>>() {
            @Override
            public void onResponse(Call<ApiResponse<Obat>> call, 
                                   Response<ApiResponse<Obat>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null) {
                    obatDetailLiveData.setValue(response.body().data);
                } else {
                    errorLiveData.setValue("Gagal memuat detail obat");
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<Obat>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
            }
        });

        return obatDetailLiveData;
    }

    // ============ CREATE OBAT (WITH IMAGE) ============
    public LiveData<Boolean> createObat(Obat obat, String imagePath) {
        MutableLiveData<Boolean> resultLiveData = new MutableLiveData<>();
        loadingLiveData.setValue(true);

        // Prepare request body
        RequestBody namaObatBody = RequestBody.create(MediaType.parse("text/plain"), obat.getNamaObat());
        RequestBody jenisObatBody = RequestBody.create(MediaType.parse("text/plain"), obat.getJenisObat());
        RequestBody stockBody = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(obat.getStock()));
        RequestBody idSupplierBody = RequestBody.create(MediaType.parse("text/plain"), 
                                                       String.valueOf(obat.getIdSupplier()));

        // Prepare image file
        MultipartBody.Part imagePart = null;
        if (imagePath != null && !imagePath.isEmpty()) {
            File imageFile = new File(imagePath);
            if (imageFile.exists()) {
                RequestBody fileBody = RequestBody.create(MediaType.parse("image/*"), imageFile);
                imagePart = MultipartBody.Part.createFormData("gambar", imageFile.getName(), fileBody);
            }
        }

        Call<ApiResponse<Obat>> call = apiService.createObat(
                namaObatBody,
                jenisObatBody,
                stockBody,
                idSupplierBody,
                imagePart
        );

        call.enqueue(new Callback<ApiResponse<Obat>>() {
            @Override
            public void onResponse(Call<ApiResponse<Obat>> call, 
                                   Response<ApiResponse<Obat>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null && "success".equals(response.body().status)) {
                    resultLiveData.setValue(true);
                } else {
                    errorLiveData.setValue("Gagal menambah obat");
                    resultLiveData.setValue(false);
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<Obat>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
                resultLiveData.setValue(false);
            }
        });

        return resultLiveData;
    }

    // ============ UPDATE OBAT ============
    public LiveData<Boolean> updateObat(int id, Obat obat, String imagePath) {
        MutableLiveData<Boolean> resultLiveData = new MutableLiveData<>();
        loadingLiveData.setValue(true);

        RequestBody namaObatBody = RequestBody.create(MediaType.parse("text/plain"), obat.getNamaObat());
        RequestBody jenisObatBody = RequestBody.create(MediaType.parse("text/plain"), obat.getJenisObat());
        RequestBody stockBody = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(obat.getStock()));
        RequestBody idSupplierBody = RequestBody.create(MediaType.parse("text/plain"), 
                                                       String.valueOf(obat.getIdSupplier()));

        MultipartBody.Part imagePart = null;
        if (imagePath != null && !imagePath.isEmpty()) {
            File imageFile = new File(imagePath);
            if (imageFile.exists()) {
                RequestBody fileBody = RequestBody.create(MediaType.parse("image/*"), imageFile);
                imagePart = MultipartBody.Part.createFormData("gambar", imageFile.getName(), fileBody);
            }
        }

        Call<ApiResponse<Obat>> call = apiService.updateObat(
                id,
                namaObatBody,
                jenisObatBody,
                stockBody,
                idSupplierBody,
                imagePart
        );

        call.enqueue(new Callback<ApiResponse<Obat>>() {
            @Override
            public void onResponse(Call<ApiResponse<Obat>> call, 
                                   Response<ApiResponse<Obat>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null && "success".equals(response.body().status)) {
                    resultLiveData.setValue(true);
                } else {
                    errorLiveData.setValue("Gagal mengupdate obat");
                    resultLiveData.setValue(false);
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<Obat>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
                resultLiveData.setValue(false);
            }
        });

        return resultLiveData;
    }

    // ============ UPDATE STOCK ============
    public LiveData<Boolean> updateStock(int idObat, int newStock) {
        MutableLiveData<Boolean> resultLiveData = new MutableLiveData<>();
        loadingLiveData.setValue(true);

        InventoryApiService.StockUpdateRequest request = 
            new InventoryApiService.StockUpdateRequest(newStock);

        Call<ApiResponse<Obat>> call = apiService.updateStock(idObat, request);

        call.enqueue(new Callback<ApiResponse<Obat>>() {
            @Override
            public void onResponse(Call<ApiResponse<Obat>> call, 
                                   Response<ApiResponse<Obat>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null && "success".equals(response.body().status)) {
                    resultLiveData.setValue(true);
                } else {
                    errorLiveData.setValue("Gagal mengupdate stock");
                    resultLiveData.setValue(false);
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<Obat>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
                resultLiveData.setValue(false);
            }
        });

        return resultLiveData;
    }

    // ============ DELETE OBAT ============
    public LiveData<Boolean> deleteObat(int id) {
        MutableLiveData<Boolean> resultLiveData = new MutableLiveData<>();
        loadingLiveData.setValue(true);

        Call<ApiResponse<Object>> call = apiService.deleteObat(id);

        call.enqueue(new Callback<ApiResponse<Object>>() {
            @Override
            public void onResponse(Call<ApiResponse<Object>> call, 
                                   Response<ApiResponse<Object>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null && "success".equals(response.body().status)) {
                    resultLiveData.setValue(true);
                } else {
                    errorLiveData.setValue("Gagal menghapus obat");
                    resultLiveData.setValue(false);
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<Object>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
                resultLiveData.setValue(false);
            }
        });

        return resultLiveData;
    }

    // ============ LIVEDATA GETTERS ============
    public LiveData<List<Obat>> getObatList() {
        return obatListLiveData;
    }

    public LiveData<Obat> getObatDetail() {
        return obatDetailLiveData;
    }

    public LiveData<String> getError() {
        return errorLiveData;
    }

    public LiveData<Boolean> getLoading() {
        return loadingLiveData;
    }
}
```

### Step 3.2: Buat SupplierRepository

**File: `SupplierRepository.java`**

```java
package com.example.inventoryobat.repository;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import com.example.inventoryobat.model.Supplier;
import com.example.inventoryobat.network.ApiResponse;
import com.example.inventoryobat.network.InventoryApiService;
import com.example.inventoryobat.network.RetrofitClient;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import java.util.List;

public class SupplierRepository {

    private final InventoryApiService apiService;
    private final MutableLiveData<List<Supplier>> supplierListLiveData;
    private final MutableLiveData<String> errorLiveData;
    private final MutableLiveData<Boolean> loadingLiveData;

    public SupplierRepository() {
        this.apiService = RetrofitClient.getClient().create(InventoryApiService.class);
        this.supplierListLiveData = new MutableLiveData<>();
        this.errorLiveData = new MutableLiveData<>();
        this.loadingLiveData = new MutableLiveData<>();
    }

    public LiveData<List<Supplier>> getAllSuppliers() {
        loadingLiveData.setValue(true);
        Call<ApiResponse<List<Supplier>>> call = apiService.getAllSuppliers();

        call.enqueue(new Callback<ApiResponse<List<Supplier>>>() {
            @Override
            public void onResponse(Call<ApiResponse<List<Supplier>>> call, 
                                   Response<ApiResponse<List<Supplier>>> response) {
                loadingLiveData.setValue(false);

                if (response.isSuccessful() && response.body() != null) {
                    supplierListLiveData.setValue(response.body().data);
                } else {
                    errorLiveData.setValue("Gagal memuat supplier");
                }
            }

            @Override
            public void onFailure(Call<ApiResponse<List<Supplier>>> call, Throwable t) {
                loadingLiveData.setValue(false);
                errorLiveData.setValue(t.getMessage());
            }
        });

        return supplierListLiveData;
    }

    public LiveData<List<Supplier>> getSupplierList() {
        return supplierListLiveData;
    }

    public LiveData<String> getError() {
        return errorLiveData;
    }

    public LiveData<Boolean> getLoading() {
        return loadingLiveData;
    }
}
```

---

## FASE 4: REFACTOR VIEWMODEL

### Step 4.1: Update MainViewModel untuk API

**File: `MainViewModel.java` (refactor)**

```java
package com.example.inventoryobat.viewmodel;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.ViewModel;
import com.example.inventoryobat.model.Obat;
import com.example.inventoryobat.model.Supplier;
import com.example.inventoryobat.repository.ObatRepository;
import com.example.inventoryobat.repository.SupplierRepository;
import java.util.List;

public class MainViewModel extends ViewModel {

    private final ObatRepository obatRepository;
    private final SupplierRepository supplierRepository;

    public MainViewModel() {
        this.obatRepository = new ObatRepository();
        this.supplierRepository = new SupplierRepository();
    }

    // ============ OBAT ============
    public LiveData<List<Obat>> getAllObat() {
        return obatRepository.getAllObat();
    }

    public void loadAllObat() {
        obatRepository.getAllObat();
    }

    public void loadObatByJenis(String jenisObat) {
        obatRepository.getObatByJenis(jenisObat);
    }

    public LiveData<Obat> getObatById(int id) {
        return obatRepository.getObatById(id);
    }

    public LiveData<List<Obat>> getObatList() {
        return obatRepository.getObatList();
    }

    public LiveData<Boolean> insertObat(Obat obat, String imagePath) {
        return obatRepository.createObat(obat, imagePath);
    }

    public LiveData<Boolean> updateObat(int id, Obat obat, String imagePath) {
        return obatRepository.updateObat(id, obat, imagePath);
    }

    public LiveData<Boolean> updateStock(int idObat, int newStock) {
        return obatRepository.updateStock(idObat, newStock);
    }

    public LiveData<Boolean> deleteObat(int id) {
        return obatRepository.deleteObat(id);
    }

    // ============ SUPPLIER ============
    public void loadAllSuppliers() {
        supplierRepository.getAllSuppliers();
    }

    public LiveData<List<Supplier>> getSupplierList() {
        return supplierRepository.getSupplierList();
    }

    // ============ LOADING & ERROR ============
    public LiveData<Boolean> getLoading() {
        return obatRepository.getLoading();
    }

    public LiveData<String> getError() {
        return obatRepository.getError();
    }
}
```

---

## FASE 5: REFACTOR ACTIVITIES

### Step 5.1: Update TambahObatActivity

**File: `TambahObatActivity.java` (refactor penting)**

Ubah method `simpanObat()` dan `updateObat()`:

```java
private void simpanObat() {
    String namaObat = binding.edtNamaObat.getText().toString().trim();
    String stockStr = binding.edtStock.getText().toString().trim();
    
    if (namaObat.isEmpty() || stockStr.isEmpty()) {
        Toast.makeText(this, "Isi semua field!", Toast.LENGTH_SHORT).show();
        return;
    }

    int stock = Integer.parseInt(stockStr);
    int jenisPosition = binding.spinnerJenisObat.getSelectedItemPosition();
    String jenisObat = JenisObat.values()[jenisPosition].name();

    Obat obat = new Obat();
    obat.setNamaObat(namaObat);
    obat.setJenisObat(jenisObat);
    obat.setStock(stock);
    obat.setIdSupplier(selectedSupplierId);

    // Get image path from URI (jika ada)
    String imagePath = selectedImageUri != null ? 
                       getFilePathFromURI(selectedImageUri) : null;

    // Call API via ViewModel
    viewModel.insertObat(obat, imagePath).observe(this, success -> {
        if (success) {
            Toast.makeText(TambahObatActivity.this, 
                          "Obat berhasil ditambahkan!", 
                          Toast.LENGTH_SHORT).show();
            finish();
        } else {
            Toast.makeText(TambahObatActivity.this, 
                          "Gagal menambah obat", 
                          Toast.LENGTH_SHORT).show();
        }
    });
}

private void updateObat() {
    String namaObat = binding.edtNamaObat.getText().toString().trim();
    String stockStr = binding.edtStock.getText().toString().trim();
    
    if (namaObat.isEmpty() || stockStr.isEmpty()) {
        Toast.makeText(this, "Isi semua field!", Toast.LENGTH_SHORT).show();
        return;
    }

    int stock = Integer.parseInt(stockStr);
    int jenisPosition = binding.spinnerJenisObat.getSelectedItemPosition();
    String jenisObat = JenisObat.values()[jenisPosition].name();

    Obat obat = new Obat();
    obat.setNamaObat(namaObat);
    obat.setJenisObat(jenisObat);
    obat.setStock(stock);
    obat.setIdSupplier(selectedSupplierId);

    String imagePath = selectedImageUri != null ? 
                       getFilePathFromURI(selectedImageUri) : null;

    viewModel.updateObat(editObatId, obat, imagePath).observe(this, success -> {
        if (success) {
            Toast.makeText(TambahObatActivity.this, 
                          "Obat berhasil diupdate!", 
                          Toast.LENGTH_SHORT).show();
            finish();
        } else {
            Toast.makeText(TambahObatActivity.this, 
                          "Gagal mengupdate obat", 
                          Toast.LENGTH_SHORT).show();
        }
    });
}

// Helper: Convert URI to file path
private String getFilePathFromURI(Uri uri) {
    if (uri.getScheme().equals("content")) {
        String[] projection = { MediaStore.Images.Media.DATA };
        Cursor cursor = getContentResolver().query(uri, projection, null, null, null);
        if (cursor != null && cursor.moveToFirst()) {
            int column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
            String filePath = cursor.getString(column_index);
            cursor.close();
            return filePath;
        }
    } else if (uri.getScheme().equals("file")) {
        return new File(uri.getPath()).getAbsolutePath();
    }
    return null;
}
```

### Step 5.2: Update InfoProdukActivity

**File: `InfoProdukActivity.java` (refactor penting)**

```java
private void loadObatDetail(int obatId) {
    // BARU: Gunakan API
    viewModel.getObatById(obatId).observe(this, obat -> {
        if (obat != null) {
            currentObat = obat;
            binding.tvNamaObatDetail.setText(obat.getNamaObat());
            binding.tvJenisObatDetail.setText("Jenis: " + obat.getJenisObat());
            binding.tvStockDetail.setText("Stock: " + obat.getStock());
            
            if (obat.getSupplier() != null) {
                binding.tvSupplierDetail.setText("Supplier: " + obat.getSupplier().getNamaSupplier());
                binding.tvEmailSupplier.setText("Email: " + obat.getSupplier().getEmail());
                binding.tvNomorSupplier.setText("Nomor: " + obat.getSupplier().getNomor());
            }

            // Load gambar dari URL (bukan local URI)
            if (obat.getGambarUrl() != null && !obat.getGambarUrl().isEmpty()) {
                Glide.with(this)
                    .load(obat.getGambarUrl())
                    .into(binding.imgObatDetail);
            }
        } else {
            Toast.makeText(this, "Gagal memuat data obat", Toast.LENGTH_SHORT).show();
        }
    });
}

private void updateStock() {
    if (currentObat == null) {
        Toast.makeText(this, "Data obat tidak valid", Toast.LENGTH_SHORT).show();
        return;
    }

    String quantityStr = binding.edtQuantity.getText().toString().trim();
    if (quantityStr.isEmpty()) {
        Toast.makeText(this, "Masukkan jumlah quantity", Toast.LENGTH_SHORT).show();
        return;
    }

    int qty = Integer.parseInt(quantityStr);
    int newStock = currentObat.getStock() + qty;

    if (newStock >= 0) {
        viewModel.updateStock(currentObat.getIdObat(), newStock).observe(this, success -> {
            if (success) {
                Toast.makeText(this, "Stock berhasil diupdate", Toast.LENGTH_SHORT).show();
                finish();
            } else {
                Toast.makeText(this, "Gagal mengupdate stock", Toast.LENGTH_SHORT).show();
            }
        });
    } else {
        Toast.makeText(this, "Stock tidak boleh negatif", Toast.LENGTH_SHORT).show();
    }
}

private void deleteObat() {
    if (currentObat == null) {
        Toast.makeText(this, "Data obat tidak valid", Toast.LENGTH_SHORT).show();
        return;
    }

    viewModel.deleteObat(currentObat.getIdObat()).observe(this, success -> {
        if (success) {
            Toast.makeText(this, "Obat berhasil dihapus", Toast.LENGTH_SHORT).show();
            finish();
        } else {
            Toast.makeText(this, "Gagal menghapus obat", Toast.LENGTH_SHORT).show();
        }
    });
}
```

### Step 5.3: Update ObatFragment

**File: `ObatFragment.java` (refactor)**

```java
@Override
public void onViewCreated(View view, Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    
    viewModel = new MainViewModel();
    adapter = new ObatAdapter(getContext(), new ArrayList<>());
    binding.rvObat.setLayoutManager(new LinearLayoutManager(getContext()));
    binding.rvObat.setAdapter(adapter);

    // Observe data dari API
    if (jenisObat != null && !jenisObat.isEmpty()) {
        viewModel.loadObatByJenis(jenisObat);
    } else {
        viewModel.loadAllObat();
    }

    viewModel.getObatList().observe(getViewLifecycleOwner(), obats -> {
        adapter.setObatList(obats);
        obatFullList = new ArrayList<>(obats);
    });

    viewModel.getError().observe(getViewLifecycleOwner(), error -> {
        if (error != null) {
            Toast.makeText(getContext(), error, Toast.LENGTH_SHORT).show();
        }
    });
}
```

---

## FASE 6: BERSIHKAN KODE LOKAL DATABASE

### Step 6.1: Hapus DatabaseHelper

**Hapus file: `DatabaseHelper.java`**

File ini sudah tidak diperlukan karena semua data sekarang melalui API.

### Step 6.2: Hapus referensi DatabaseHelper di seluruh project

Cari dan hapus semua baris seperti:
```java
DatabaseHelper db = new DatabaseHelper(this);
db.getObatById(...);
```

---

## FASE 7: TESTING & DEBUGGING

### Step 7.1: Test Koneksi API

1. **Buka Android Studio Logcat**
2. **Jalankan aplikasi di emulator/device**
3. **Cek log OkHttp untuk melihat request/response**:
   ```
   I/OkHttp: ---> POST https://apotek.astrantia.site/api/v1/obats
   I/OkHttp: Content-Type: multipart/form-data
   ```

### Step 7.2: Common Issues & Solusi

| Error | Solusi |
|-------|--------|
| `SSL HandshakeException` | Pastikan HTTPS domain valid, atau debug certificate di dev |
| `Connection timeout` | Cek internet connection, ping domain apotek.astrantia.site |
| `Retrofit NullPointerException` | Cek response JSON format, bandingkan dengan API docs |
| `Multipart upload gagal` | Pastikan file image valid, cek path dan permissions |
| `401 Unauthorized` | (Future) Jika backend tambah auth, implementasikan token |

---

## FASE 8: MIGRATION CHECKLIST

Tandai setiap item setelah selesai:

```
✓ Tambah dependency Retrofit & OkHttp
✓ Tambah internet permission
✓ Buat RetrofitClient.java
✓ Buat InventoryApiService.java
✓ Update Obat.java dengan @SerializedName
✓ Update Supplier.java dengan @SerializedName
✓ Buat ObatRepository.java
✓ Buat SupplierRepository.java
✓ Refactor MainViewModel
✓ Update TambahObatActivity
✓ Update InfoProdukActivity
✓ Update ObatFragment
✓ Hapus DatabaseHelper.java
✓ Test semua fitur (CRUD)
✓ Test upload image
✓ Test filter by jenis
✓ Verifikasi log di Logcat
```

---

## FASE 9: FUTURE ENHANCEMENTS

Setelah migrasi berhasil, pertimbangkan:

1. **Authentication**
   - Implementasikan login API (jika backend support)
   - Simpan token di SharedPreferences/DataStore
   - Attach token di setiap request

2. **Offline Support**
   - Implement local caching dengan Room Database
   - Sync data saat online

3. **Error Handling**
   - Better error messages
   - Retry mechanism untuk failed requests

4. **Performance**
   - Image compression sebelum upload
   - Pagination untuk list data
   - Caching dengan interceptor

---

**Selesai! Aplikasi Anda sudah berhasil bermigrasi dari lokal database ke REST API.** 🎉
