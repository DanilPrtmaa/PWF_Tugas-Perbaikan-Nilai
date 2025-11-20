
# 🧼 PWF – Tugas Perbaikan Nilai

## **Website Manajemen Laundry**

**Role User:** Admin / Kasir (Single Role)

---

## 📌 Penjelasan Penerapan Fitur Laravel

Website ini telah menerapkan seluruh fitur yang diminta dalam tugas:

✔ Migration
✔ Model Eloquent
✔ View (Blade Template)
✔ Routes & Controller
✔ Blade If, Foreach, CSRF & Extends Layout

---

## 1️⃣ Migration

Website ini menggunakan **4 migration** untuk membentuk struktur database utama:

---

### 📄 `create_customers_table.php`

```php
Schema::create('customers', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('phone')->unique();
    $table->string('email')->nullable();
    $table->text('address');
    $table->timestamps();
});
```

### 📄 `create_services_table.php`

```php
Schema::create('services', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->decimal('price', 10, 2);
    $table->string('unit');
    $table->integer('estimated_days');
    $table->timestamps();
});
```

### 📄 `create_orders_table.php`

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->string('order_number')->unique();
    $table->foreignId('customer_id')->constrained()->onDelete('cascade');
    $table->date('order_date');
    $table->date('delivery_date');
    $table->enum('status', ['pending','processing','ready','completed','cancelled'])->default('pending');
    $table->decimal('total_amount', 10, 2)->default(0);
    $table->text('notes')->nullable();
    $table->timestamps();
});
```

### 📄 `create_order_items_table.php`

```php
Schema::create('order_items', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_id')->constrained()->onDelete('cascade');
    $table->foreignId('service_id')->constrained()->onDelete('cascade');
    $table->integer('quantity');
    $table->decimal('unit_price', 10, 2);
    $table->decimal('subtotal', 10, 2);
    $table->text('description')->nullable();
    $table->timestamps();
});
```

---

## 2️⃣ Model (Eloquent ORM)

### 📁 `Customer.php`

```php
class Customer extends Model
{
    protected $fillable = ['name', 'phone', 'email', 'address'];

    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }
}
```

### 📁 `Service.php`

```php
class Service extends Model
{
    protected $fillable = ['name', 'description', 'price', 'unit', 'estimated_days'];

    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}
```

### 📁 `Order.php`

```php
class Order extends Model
{
    protected $fillable = ['order_number', 'customer_id', 'order_date', 'delivery_date', 'status', 'total_amount', 'notes'];

    public function customer(): BelongsTo
    {
        return $this->belongsTo(Customer::class);
    }

    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }

    // Generate nomor transaksi otomatis
    protected static function boot()
    {
        parent::boot();
        static::creating(function ($order) {
            $order->order_number = 'ORD-' . date('Ymd') . '-' . str_pad(static::count() + 1, 4, '0', STR_PAD_LEFT);
        });
    }
}
```

### 📁 `OrderItem.php`

```php
class OrderItem extends Model
{
    protected $fillable = ['order_id', 'service_id', 'quantity', 'unit_price', 'subtotal', 'description'];

    public function order(): BelongsTo
    {
        return $this->belongsTo(Order::class);
    }

    public function service(): BelongsTo
    {
        return $this->belongsTo(Service::class);
    }

    // Hitung subtotal otomatis
    protected static function boot()
    {
        parent::boot();
        static::creating(function ($orderItem) {
            $orderItem->subtotal = $orderItem->quantity * $orderItem->unit_price;
        });
    }
}
```

✔ **Menerapkan:**

* Mass Assignment (`$fillable`)
* Relasi (`hasMany`, `belongsTo`)
* Auto generate nomor transaksi & subtotal (Model Event)

---

## 3️⃣ View (Blade Template)

### 🔹 Route Langsung ke View

```php
Route::get('/', function () {
    $totalCustomers = Customer::count();
    $totalOrders = Order::count();

    return view('home', compact('totalCustomers', 'totalOrders'));
})->name('home');
```

### 🔹 Route melalui Controller (CRUD)

```php
Route::resource('customers', CustomerController::class);
Route::resource('services', ServiceController::class);
Route::resource('orders', OrderController::class);
```

### 🔹 Blade Extends Layout

```php
@extends('layouts.app')
@section('title', 'Data Pelanggan')
@section('content')
    <!-- isi konten -->
@endsection
```

### 🔹 Blade Foreach

```php
@foreach($customers as $customer)
<tr>
    <td>{{ $customer->name }}</td>
    <td>{{ $customer->phone }}</td>
</tr>
@endforeach
```

### 🔹 Blade If

```php
@if($customers->isEmpty())
    <p>Belum ada pelanggan.</p>
@endif
```

### 🔹 CSRF & Method PUT/DELETE

```php
<form action="{{ route('customers.update', $customer) }}" method="POST">
    @csrf
    @method('PUT')
</form>

<form action="{{ route('customers.destroy', $customer) }}" method="POST">
    @csrf
    @method('DELETE')
</form>
```

---

## 4️⃣ Routing Tambahan

```php
Route::post('/orders/{order}/update-status', [OrderController::class, 'updateStatus'])
    ->name('orders.update-status');

Route::get('/orders/{order}/invoice', [OrderController::class, 'invoice'])
    ->name('orders.invoice');
```

---

## ✔ Kesimpulan Penerapan

| Fitur Laravel        | Status           |
| -------------------- | ---------------- |
| Migration            | ✔ 4 Tabel Dibuat |
| Model + Relasi       | ✔ Lengkap        |
| CRUD Controller      | ✔                |
| Blade Extends Layout | ✔                |
| CSRF                 | ✔                |
| Blade If / Foreach   | ✔                |
| Resource Route       | ✔                |

---

## 📸 Tampilan Website

Semua screenshot fitur pada website:

<img width="1920" height="1521" alt="screencapture-127-0-0-1-8000-2025-11-20-23_00_02" src="https://github.com/user-attachments/assets/0f17a182-b708-4a5d-a465-705ccb446547" />
<img width="1920" height="1255" alt="screencapture-127-0-0-1-8000-customers-2025-11-20-23_00_18" src="https://github.com/user-attachments/assets/00011419-66c1-47e8-bcee-9c9b9da2bb5e" />
<img width="1920" height="1302" alt="screencapture-127-0-0-1-8000-customers-create-2025-11-20-23_01_52" src="https://github.com/user-attachments/assets/858b6948-4d41-46af-859c-445872ed54ab" />
<img width="1920" height="980" alt="screencapture-127-0-0-1-8000-customers-5-edit-2025-11-20-23_03_08" src="https://github.com/user-attachments/assets/4282a937-d4b9-4349-8625-7d848164852f" />
<img width="1920" height="982" alt="screencapture-127-0-0-1-8000-services-2025-11-20-23_00_43" src="https://github.com/user-attachments/assets/da7d8dd5-d44e-40be-8689-c2554d119049" />
<img width="1920" height="1415" alt="screencapture-127-0-0-1-8000-services-create-2025-11-20-23_02_03" src="https://github.com/user-attachments/assets/1ae19ac3-45f3-48a5-a858-b7c445e4f3e6" />
<img width="1920" height="1270" alt="screencapture-127-0-0-1-8000-services-3-edit-2025-11-20-23_03_28" src="https://github.com/user-attachments/assets/c04d0e27-7178-46c1-b746-99dca7bd9d0b" />
<img width="1920" height="1186" alt="screencapture-127-0-0-1-8000-orders-2025-11-20-23_00_56" src="https://github.com/user-attachments/assets/193ee490-4f16-4371-9cc2-469877f607a6" />
<img width="1920" height="1539" alt="screencapture-127-0-0-1-8000-orders-4-2025-11-20-23_03_52" src="https://github.com/user-attachments/assets/60cc1937-a174-4070-8abf-1f8707383f12" />
<img width="1920" height="1304" alt="screencapture-127-0-0-1-8000-orders-4-invoice-2025-11-20-23_04_08" src="https://github.com/user-attachments/assets/9d1a6bd0-c07e-432d-b037-2db953727549" />
<img width="1310" height="868" alt="Screenshot 2025-11-20 230532" src="https://github.com/user-attachments/assets/f4561719-acf3-434d-bfe5-998822d359f9" />

---




