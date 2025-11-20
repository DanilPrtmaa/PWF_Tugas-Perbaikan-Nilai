# PWF_Tugas-Perbaikan-Nilai


# Penjelasan Penerapan Fitur pada Website Manajemen Laundry

1. Migration
   Penerapan:
Website ini menggunakan 4 migration untuk membuat struktur database yang diperlukan:

// File: database/migrations/[timestamp]_create_customers_table.php
Schema::create('customers', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('phone')->unique();
    $table->string('email')->nullable();
    $table->text('address');
    $table->timestamps();
});

// File: database/migrations/[timestamp]_create_services_table.php  
Schema::create('services', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->decimal('price', 10, 2);
    $table->string('unit');
    $table->integer('estimated_days');
    $table->timestamps();
});

// File: database/migrations/[timestamp]_create_orders_table.php
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

// File: database/migrations/[timestamp]_create_order_items_table.php
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

2. Model
Penerapan:
Website menggunakan 4 model Eloquent yang sesuai dengan tabel database:

// File: app/Models/Customer.php
class Customer extends Model
{
    protected $fillable = ['name', 'phone', 'email', 'address'];
    
    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }
}

// File: app/Models/Service.php
class Service extends Model
{
    protected $fillable = ['name', 'description', 'price', 'unit', 'estimated_days'];
    
    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}

// File: app/Models/Order.php
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
    
    protected static function boot()
    {
        parent::boot();
        static::creating(function ($order) {
            $order->order_number = 'ORD-' . date('Ymd') . '-' . str_pad(static::count() + 1, 4, '0', STR_PAD_LEFT);
        });
    }
}

// File: app/Models/OrderItem.php
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
    
    protected static function boot()
    {
        parent::boot();
        static::creating(function ($orderItem) {
            $orderItem->subtotal = $orderItem->quantity * $orderItem->unit_price;
        });
    }
}

Penjelasan: Model Eloquent digunakan untuk berinteraksi dengan database. Setiap model mendefinisikan:
- $fillable untuk mass assignment protection
- Relationship methods (hasMany, belongsTo) untuk relasi database
- Model events untuk auto-generate order number dan calculate subtotal
- Casting untuk tipe data tertentu

Penggunaan Model untuk CRUD:
- Menampilkan Data:
  // Di Controller
public function index(): View
{
    $customers = Customer::latest()->paginate(10);
    return view('customers.index', compact('customers'));
}
- Menambah Data:
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([...]);
    Customer::create($validated);
    return redirect()->route('customers.index')->with('success', 'Customer berhasil ditambahkan.');
}
- Mengedit Data:
  public function update(Request $request, Customer $customer): RedirectResponse
{
    $validated = $request->validate([...]);
    $customer->update($validated);
    return redirect()->route('customers.index')->with('success', 'Customer berhasil diperbarui.');
}
- Menghapus Data:
  public function destroy(Customer $customer): RedirectResponse
{
    $customer->delete();
    return redirect()->route('customers.index')->with('success', 'Customer berhasil dihapus.');
}
  
3. View
   A. Halaman melalui Route Langsung:
   // File: routes/web.php
Route::get('/', function () {
    $totalCustomers = Customer::count();
    $totalOrders = Order::count();
    $totalServices = Service::count();
    $pendingOrders = Order::where('status', 'pending')->count();
    // ... data lainnya
    
    return view('home', compact(
        'totalCustomers', 'totalOrders', 'totalServices', 
        'pendingOrders', 'processingOrders', 'readyOrders', 
        'completedOrders', 'recentOrders'
    ));
})->name('home');
Penjelasan: Route / menggunakan Closure function langsung untuk menampilkan view home dengan data yang diambil langsung dari model.

   B. Halaman melalui Controller:
   // File: routes/web.php
Route::resource('customers', CustomerController::class);
Route::resource('services', ServiceController::class);
Route::resource('orders', OrderController::class);

// File: app/Http/Controllers/CustomerController.php
public function index(): View
{
    $customers = Customer::latest()->paginate(10);
    return view('customers.index', compact('customers'));
}
// File: routes/web.php
Route::resource('customers', CustomerController::class);
Route::resource('services', ServiceController::class);
Route::resource('orders', OrderController::class);

// File: app/Http/Controllers/CustomerController.php
public function index(): View
{
    $customers = Customer::latest()->paginate(10);
    return view('customers.index', compact('customers'));
}

   C. Blade Templates:
Blade Extends Layout:
<!-- File: resources/views/customers/index.blade.php -->
@extends('layouts.app')

@section('title', 'Data Pelanggan')

@section('content')
    <!-- Konten halaman spesifik -->
@endsection
Penjelasan: Setiap view extends layout utama layouts.app yang berisi struktur HTML dasar, header, navigation, dan footer.

Blade For-each:
@foreach($customers as $customer)
<tr>
    <td class="px-6 py-4 whitespace-nowrap">
        <div class="text-sm font-medium text-gray-900">{{ $customer->name }}</div>
    </td>
    <td class="px-6 py-4 whitespace-nowrap">
        <div class="text-sm text-gray-900">{{ $customer->phone }}</div>
    </td>
</tr>
@endforeach
Penjelasan: Blade @foreach digunakan untuk iterasi melalui collection data dan menampilkan setiap item dalam tabel.

Blade If:
@if($customers->isEmpty())
<div class="bg-white rounded-2xl shadow-lg p-12 text-center">
    <p class="text-gray-600 mb-6">Belum ada pelanggan. Mulai dengan menambahkan pelanggan pertama Anda.</p>
</div>
@else
<!-- Tampilkan tabel customers -->
@endif

<!-- Blade If untuk status order -->
<span class="px-2 py-1 text-xs rounded-full 
    @if($order->status == 'pending') bg-yellow-100 text-yellow-800
    @elseif($order->status == 'processing') bg-blue-100 text-blue-800
    @else bg-green-100 text-green-800 @endif">
    {{ ucfirst($order->status) }}
</span>
Penjelasan: Blade @if digunakan untuk conditional rendering, seperti menampilkan empty state ketika tidak ada data, atau memberikan style berbeda berdasarkan status.

Blade CSRF untuk Submit Form:
<form action="{{ route('customers.store') }}" method="POST">
    @csrf
    <div class="mb-4">
        <label for="name" class="block text-sm font-medium text-gray-700">Nama Pelanggan</label>
        <input type="text" name="name" id="name" class="...">
    </div>
    <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded-lg">Simpan</button>
</form>

<!-- Untuk form edit dengan method PUT -->
<form action="{{ route('customers.update', $customer) }}" method="POST">
    @csrf
    @method('PUT')
    <!-- Form fields -->
</form>

<!-- Untuk form delete -->
<form action="{{ route('customers.destroy', $customer) }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit" onclick="return confirm('Apakah Anda yakin?')">Hapus</button>
</form>
Penjelasan:
- @csrf menghasilkan token CSRF untuk proteksi terhadap Cross-Site Request Forgery
- @method('PUT') dan @method('DELETE') untuk method spoofing karena HTML form hanya mendukung GET dan POST


4. Routes
   Penerapan Routing:
   // File: routes/web.php

// Route langsung ke view (Dashboard)
Route::get('/', function () {
    // Logic dan return view
})->name('home');

// Route melalui Controller (CRUD operations)
Route::resource('customers', CustomerController::class);
Route::resource('services', ServiceController::class);
Route::resource('orders', OrderController::class);

// Route khusus untuk update status order
Route::post('/orders/{order}/update-status', [OrderController::class, 'updateStatus'])
    ->name('orders.update-status');

// Route untuk cetak invoice
Route::get('/orders/{order}/invoice', [OrderController::class, 'invoice'])
    ->name('orders.invoice');
Penjelasan: Routing mengatur akses ke berbagai fitur aplikasi:
- Route resource otomatis membuat 7 route standar (index, create, store, show, edit, update, destroy)
- Route khusus untuk fungsi tambahan seperti update status dan cetak invoice
- Route naming untuk referensi yang konsisten di seluruh aplikasi
  
5. Kesimpulan Penerapan
Website Manajemen Laundry ini telah mengimplementasikan semua persyaratan dengan lengkap:
- Migration - 4 migration file untuk struktur database yang normalisasi
- Model - 4 model Eloquent dengan relationship, mass assignment protection, dan model events
- View - Menggunakan Blade templates dengan extends layout, conditional rendering, loops, dan CSRF protection
- Controller - 3 controller dengan method lengkap untuk CRUD operations
- Routes - Kombinasi route langsung dan melalui controller dengan resource routing

6. Tampilan Website
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

