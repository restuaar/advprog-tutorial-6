# Tutorial for Advance Programming Course 2023/2024

**Nama** : **Restu Ahmad Ar Ridho** <br/>
**NPM** : **2206028951** <br/>
**Kelas** : **Advance Programming - A**

## Module 6 - Concurrency

### Milestone 1: Single Threaded Web Server Refleksi

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

### Milestone 2: Returning HTML Refleksi

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

Kode diatas memeriksa apakah `request_line` sama dengan `"GET / HTTP/1.1"`, yang berarti client meminta halaman utama (`/`). Jika ya, maka `status_line` diatur ke`"HTTP/1.1 200 OK"` dan `path_resource` diatur ke "hello.html". Jika tidak, maka status_line diatur ke `"HTTP/1.1 404 NOT FOUND"` dan `path_resource` diatur ke `"404.html"`. Ini berarti bahwa jika client meminta sumber daya lain selain halaman utama, server akan merespons dengan status 404 dan halaman error 404.

<img align="center" src="assets\images\commit3.png" alt="Commit 3 screen capture"/>
