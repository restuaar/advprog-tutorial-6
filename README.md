# Tutorial for Advance Programming Course 2023/2024

**Nama** : **Restu Ahmad Ar Ridho** <br/>
**NPM** : **2206028951** <br/>
**Kelas** : **Advance Programming - A**

## Module 6 - Concurrency

### Milestone 1: Single Threaded Web Server

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}
```

Pada method `main` diatas terdapat _instance_ `TcpListener` untuk mendeteksi TCP Connection pada alamat yang telah ditentukan yaitu pada `127.0.0.1` atau localhost dan port `7878` dengan method `bind` yang akan mengembalikan `TcpListener` _instance_. Kemudian dengan method `unwrap()` kode diatas dapat memberhentikan program apabila terdapat _error_ dan mengeluarkan nilai dari `Result`.
Setelah itu, menggunakan loop for untuk menerima koneksi yang masuk. Setiap kali ada koneksi masuk akan menerima dari `TcpStream` untuk melihat apa yang dikirim _client_. Selanjutnya, kita memanggil fungsi `handle_connection` dengan parameter `stream` untuk menangani koneksi tersebut.

```rust
fn handle_connection(mut stream: TcpStream) {
  ...
}
```

Fungsi tersebut menerima parameter `stream` dengan tipe `TcpStream` dengan `mut` yang menandakan bahwa parameter tersebut merupakan nilai mutable.

```rust
fn handle_connection(mut stream: TcpStream) {
  let buf_reader = BufReader::new(&mut stream);
  ...
}
```

Pada variabel `buf_reader` membuat `BufReader` _instance_ untuk membungkus nilai dari parameter `stream` untuk memberikan kemampuan dalam membaca buffer yang dapat meningkatkan _performance_.

```rust
fn handle_connection(mut stream: TcpStream) {
    ...
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
    ...
}
```

Pada bagian ini, membuat variabel baru `http_request` dengan untuk menampung hasil dari pembacaan `buf_reader` menggunakan method `lines()` yang mengembalikan iterator berupa `Result<String, std:io::Error>`.

- `map(|result| result.unwrap())`  
  Akan melakukan iterasi untuk setiap baris dengan method `unwrap()` yang digunakan untuk mengekstrak nilai dari tipe `Result` yang dikembalikan oleh `lines()`. Jika terjadi _error_, maka akan menyebabkan `panic`.
- `take_while(|line| !line.is_empty())`  
  Method ini mengambil baris dari iterator sampai menemukan baris kosong. Iterator akan berhenti segera setelah menemukan baris kosong dan membuang sisanya.
- `collect()`  
  Menggabungkan baris-baris menjadi `Vec<_>` yang merupakan vector String.

```rust
...
println!("Request: {:#?}", http_request);
```

Terakhir, vektor http_request yang telah diproses dicetak ke konsol menggunakan `println!()`. Opsi pemformatan `#?` digunakan untuk mencetak vektor secara bagus, menampilkan setiap elemen pada baris terpisah.

### Milestone 2: Returning HTML

```rust
let status_line = "HTTP/1.1 200 OK";
```

Baris ini mendefinisikan sebuah variabel `status_line` yang merepresentasikan baris status dari respons HTTP. Dalam kasus ini, ini adalah respons `HTTP/1.1` yang berhasil yang ditunjukkan oleh kode status `200 OK`.

```rust
let contents = fs::read_to_string("hello.html").unwrap();
let length = contents.len();
```

Pada kode diatas membaca isi berkas `hello.html` menjadi sebuah string dengan menggunakan method `read_to_string` dari modul `fs`. Metode `unwrap()` digunakan untuk mendapatkan nilai `Result`, dan jika ada _error_ program akan panic. Varibel `length` sebagai panjang dari content berkas html.

```rust
let response = format!("{status_line}\r\nContent-Length:{length}\r\n\r\n{contents}");
```

Baris ini menggunakan `format!` untuk membuat string respons HTTP. `response` mencangkup `status_line`, `Content-Length` header, dan `contents` dari berkas `hello.html`.

> Perhatikan bahwa \r\n digunakan untuk memisahkan header dan isi dalam respons HTTP (CRLF End Line Sequence).

```rust
stream.write_all(response.as_bytes()).unwrap();
```

Terakhir, baris ini menulis respons HTTP ke `stream`. Method `write_all` mencoba menulis seluruh potongan byte ke `stream`. Method `as_bytes` digunakan untuk mengubah string respons menjadi byte.

<img align="center" src="assets\images\commit2.png" alt="Commit 2 screen capture"/>

### Milestone 3: Validating Request And Selectively Responding

```rust
let request_line = buf_reader.lines().next().unwrap().unwrap();
```

Kode diatas akan membaca baris pertama dari `buf_reader`. `lines()` menghasilkan iterator atas baris dalam input. `next()` mengambil baris pertama dari iterator tersebut. `unwrap()` pertama digunakan untuk mendapatkan Result dari `next()`, dan `unwrap()` kedua digunakan untuk mendapatkan Result dari `lines()`. Jika ada error pada salah satu tahap ini, program akan panic.

```rust
let (status_line, path_resource) = if request_line == "GET / HTTP/1.1" {
    ("HTTP/1.1 200 OK", "hello.html")
} else {
    ("HTTP/1.1 404 NOT FOUND", "404.html")
};
```

Kode diatas memeriksa apakah `request_line` sama dengan `"GET / HTTP/1.1"`, yang berarti _client_ meminta halaman utama (`/`). Jika ya, maka `status_line` diatur ke`"HTTP/1.1 200 OK"` dan `path_resource` diatur ke "hello.html". Jika tidak, maka status*line diatur ke `"HTTP/1.1 404 NOT FOUND"` dan `path_resource` diatur ke `"404.html"`. Ini berarti bahwa jika \_client* meminta sumber daya lain selain halaman utama, server akan merespons dengan status 404 dan halaman error 404.

<img align="center" src="assets\images\commit3.png" alt="Commit 3 screen capture"/>

### Milestone 4: Simulation Slow Response

```rust
let (status_line, path_resource) = match &request_line[..] {__}
```

Baris ini memulai blok `match` pada `request_line`, yang merupakan string yang mewakili baris permintaan HTTP. `&request_line[..]` adalah cara untuk mendapatkan slice yang merujuk ke seluruh string dan mencocokannya dengan pola yang telah didefinisikan.

```rust
"GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
```

Jika `request_line` sama dengan `"GET / HTTP/1.1"`, yang berarti _client_ meminta halaman utama (/), maka `status_line` diatur menjadi `"HTTP/1.1 200 OK"` dan `path_resource` diatur ke `"hello.html"`.

```rust
"GET /sleep HTTP/1.1" => {
    thread::sleep(Duration::from_secs(10));
    ("HTTP/1.1 200 OK", "hello.html")
}
```

Jika `request_line` sama dengan `"GET /sleep HTTP/1.1"`, server akan menunda prosesnya selama 10 detik menggunakan `thread::sleep(Duration::from_secs(10))`. Setelah itu, `status_line` diatur ke `"HTTP/1.1 200 OK"` dan path_resource diatur ke `"hello.html"`.

```rust
_ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
```

Jika `request_line` tidak cocok dengan salah satu dari dua pola di atas, maka `status_line` diatur ke `"HTTP/1.1 404 NOT FOUND"` dan `path_resource` diatur ke `"404.html"`.

### Milestone 5: Multithreaded Server

#### Pada berkas `src/main.rs`

```rust
use advprog_modul6::ThreadPool;
```

Baris ini mengimpor `ThreadPool` dari paket `advprog_modul6`.

```rust
let pool = ThreadPool::new(4);
```

Baris ini membuat `ThreadPool` baru dengan empat thread.

```rust
pool.execute(|| {
  handle_connection(stream);
});
```

Baris ini menjalankan method `handle_connection` dalam thread `pool`. Method ini akan dipanggil dengan `stream` sebagai argumen. `execute` mengambil closure sebagai argumen, yang dalam hal ini adalah `|| { handle_connection(stream); }`. Sehingga eksekusi method tersebut dapat menangani secara bersamaan dengan menggunakan thread yang tersedia pada `pool`.

#### Pada berkas `src/lib.rs`

```rust
pub struct ThreadPool {
    #[allow(dead_code)]
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}
```

Ini adalah definisi dari struktur `ThreadPool`. `ThreadPool` memiliki dua field: `workers` yang merupakan vektor dari `Worker` dan `sender` yang merupakan sender dari channel multiple-producer single-consumer (mpsc).

```rust
type Job = Box<dyn FnOnce() + Send + 'static>;
```

Ini adalah definisi dari tipe Job. Job adalah sebuah closure yang dapat dipanggil sekali (`FnOnce`), dapat dikirim antar thread (`Send`), dan memiliki lifetime `'static` (bisa hidup selama program berjalan).

```rust
pub fn new(size: usize) -> ThreadPool {
    assert!(size > 0);
    let (sender, receiver) = mpsc::channel();
    let receiver = Arc::new(Mutex::new(receiver));
    let mut workers = Vec::with_capacity(size);
    for id in 0..size {
        workers.push(Worker::new(id, Arc::clone(&receiver)));
    }
    ThreadPool { workers, sender }
}
```

Ini adalah method `new` yang merupakan _constructor_ untuk `ThreadPool`. Method ini menerima ukuran pool (jumlah thread) sebagai argumen, membuat channel `mpsc`, membuat `Worker` sejumlah ukuran pool, dan mengembalikan instance `ThreadPool`.

- `assert!(size > 0);`  
  Baris ini menyatakan bahwa ukuran ThreadPool harus lebih besar dari 0. Jika ukurannya tidak lebih besar dari 0, program akan panik dan berhenti.
- `let (sender, receiver) = mpsc::channel();`  
  Baris ini membuat _channel_ multiple-producer, single-consumer (mpsc) yang baru. _Channel_ ini memungkinkan beberapa thread untuk mengirim data ke satu receiver.
- `let receiver = Arc::new(Mutex::new(receiver));`  
  Baris ini membungkus `receiver` dengan Mutex dan kemudian Arc. Mutex memungkinkan kita untuk memiliki akses yang dapat diubah-ubah ke data dari satu thread pada suatu waktu. Arc adalah _thread-safe reference-counting_.
- `let mut workers = Vec::with_capacity(size);`  
  Baris ini membuat vektor yang dapat diubah dengan kapasitas yang sama dengan ukuran ThreadPool.
- `for loop`  
  Membuat `Worker` _instance_, masing-masing dengan id unik dan _clone_ dari Arc receiver, dan memasukkan kedalam vector Worker.

```rust
pub fn execute<F>(&self, f: F)
where
    F: FnOnce() + Send + 'static,
{
    let job = Box::new(f);
    self.sender.send(job).unwrap();
}
```

Method execute ini adalah method publik yang menerima satu argumen `f`. Argumen `f` ini adalah sebuah closure yang memenuhi trait bounds FnOnce, Send, dan 'static. FnOnce berarti closure ini hanya bisa dipanggil sekali, Send berarti closure ini bisa dikirim ke thread lain, dan 'static berarti closure ini memiliki lifetime 'static (bisa hidup selama program berjalan).

- `let job = Box::new(f);`  
  Closure `f` dibungkus dalam `Box` untuk membuat sebuah `Job`. `Box` digunakan untuk menyimpan data di heap dan memberikan pointer ke data tersebut.
- `self.sender.send(job).unwrap();`
  `Job` yang telah dibuat kemudian dikirim melalui `sender` ke receiver yang ada di thread lain. `unwrap()` digunakan untuk mengambil hasil dari operasi `send`. Jika operasi send gagal, program akan panic dan berhenti.

```rust
struct Worker {
    #[allow(dead_code)]
    id: usize,
    #[allow(dead_code)]
    thread: thread::JoinHandle<()>,
}
```

Ini adalah definisi dari struktur `Worker`. `Worker` memiliki dua field: id yang merupakan identifikasi unik untuk setiap `Worker`, dan thread yang merupakan handle untuk thread yang dijalankan oleh `Worker`.

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {id} got a job; executing.");
            job();
        });
        Worker { id, thread }
    }
}
```

Kode tersebut adalah implementasi dari method `new` pada `struct Worker` dalam Rust. Method ini digunakan untuk membuat instance baru dari `Worker`.

- Fungsi `new` ini adalah fungsi yang menerima dua argumen, `id` dengan tipe `usize` dan `receiver` dengan tipe `Arc<Mutex<mpsc::Receiver<Job>>>`, dan mengembalikan instance `Worker`.
- `let thread = thread::spawn(move || loop {})`  
  Di sini, sebuah thread baru di-spawn dengan closure yang berisi loop tak terbatas
- `let job = receiver.lock().unwrap().recv().unwrap();`  
  `lock` adalah metode yang digunakan untuk mendapatkan akses ke data yang dilindungi oleh Mutex. `recv` adalah metode yang digunakan untuk menerima nilai dari channel. `recv` akan mengembalikan Result yang baik berisi nilai yang diterima atau `RecvError` jika channel pengirim telah ditutup dan tidak ada nilai lagi yang bisa diterima.
- `job()`  
  Ini adalah pemanggilan dari Job yang telah diterima. Job ini adalah closure yang telah dikirim melalui channel.

### Bonus Reflection

Sebenarnya lebih disarankan untuk menggunakan metode `build` yang mengembalikan _instance_ `Result` daripada `new` saat membuat instansiasi ThreadPool. Alasannya adalah karena dengan menggunakan `build`, kita dapat menangani situasi di mana angka yang diberikan untuk ukuran thread pool terlalu kecil yang menyebabkan kesalahan. Oleh karena itu, rekomendasi adalah untuk mengganti metode `new` dengan `build`, yang mengembalikan `Result`, dan kemudian pemanggilan dapat menggunakan `unwrap()` untuk menangani hasilnya.
