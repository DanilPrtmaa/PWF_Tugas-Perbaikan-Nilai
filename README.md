Berikut sudah **aku rapikan agar cocok untuk README GitHub dan mudah dibaca dosen/tutor**.
Struktur sudah memakai **heading yang jelas, bullet point, code block, dan penjelasan ringkas** supaya terlihat profesional dan langsung menonjolkan penerapan fitur Laravel.

---

# 🧼 **PWF – Website Manajemen Laundry**

**Tugas Perbaikan Nilai – Praktikum Web Framework (Laravel 12)**
**Role Pengguna:** Admin / Kasir (Single Role)

---

## 📌 **Penjelasan Penerapan Fitur Laravel**

Website ini menggunakan 4 komponen utama Laravel:

1. **Migration** – Membuat struktur database
2. **Model (Eloquent ORM)** – Interaksi dengan database
3. **View (Blade Template)** – Tampilan halaman web
4. **Routing & Controller** – Logika & proses CRUD

Semua fitur ini sudah diterapkan sesuai ketentuan tugas.

---

## 1️⃣ **Migration**

Terdapat **4 migration** untuk membangun struktur database:

### 🗂 `create_customers_table.php`

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

### 🗂 `create_services_table.php`

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

### 🗂 `create_orders_table.php`

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

### 🗂 `create_order_items_table.php`

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

## 2️⃣ **Model (Eloquent ORM)**

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

---

## 3️⃣ **View (Blade Template)**

### 🧩 Route Langsung ke View (Dashboard)

```php
Route::get('/', function () {
    $totalCustomers = Customer::count();
    $totalOrders = Order::count();
    $totalServices = Service::count();
    $pendingOrders = Order::where('status', 'pending')->count();

    return view('home', compact(
        'totalCustomers', 'totalOrders', 'totalServices', 'pendingOrders'
    ));
})->name('home');
```

### 🧩 View melalui Controller (CRUD)

```php
Route::resource('customers', CustomerController::class);
Route::resource('services', ServiceController::class);
Route::resource('orders', OrderController::class);
```

### 🧩 Blade Extends Layout

```php
@extends('layouts.app')

@section('title', 'Data Pelanggan')

@section('content')
    <!-- isi konten -->
@endsection
```

### 🧩 Blade `@foreach`

```php
@foreach($customers as $customer)
    <tr>
        <td>{{ $customer->name }}</td>
        <td>{{ $customer->phone }}</td>
    </tr>
@endforeach
```

### 🧩 Blade `@if`

```php
@if($customers->isEmpty())
    <p>Belum ada pelanggan.</p>
@endif
```

### 🧩 Blade `@csrf` & Method PUT/DELETE

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

## 4️⃣ **Routing Tambahan**

```php
// Update status order
Route::post('/orders/{order}/update-status', [OrderController::class, 'updateStatus'])
    ->name('orders.update-status');

// Cetak invoice
Route::get('/orders/{order}/invoice', [OrderController::class, 'invoice'])
    ->name('orders.invoice');
```

---

## ✔ **Kesimpulan Penerapan Fitur**

| Fitur Laravel      | Status    |
| ------------------ | --------- |
| Migration          | ✔ 4 Tabel |
| Model + Relasi ORM | ✔         |
| CRUD Controller    | ✔         |
| Blade Template     | ✔         |
| CSRF Protection    | ✔         |
| Loop (@foreach)    | ✔         |
| Logic (@if)        | ✔         |
| Route + Resource   | ✔         |

---

## 📸 **Tampilan Website**

> Semua screenshot sudah diunggah ke repository:
> Dashboard, CRUD Customer, Services, Orders, Detail Order & Invoice.

*(Screenshot akan otomatis tampil di GitHub)*
![Dashboard](https://github.com/user-attachments/assets/0f17a182-b708-4a5d-a465-705ccb446547)

---

Kalau mau **versi PDF laporan**, atau **BAB I–IV siap print**, tinggal bilang:

> minta versi laporan / PDF

Siap bantu sampai ACC 💯🔥
