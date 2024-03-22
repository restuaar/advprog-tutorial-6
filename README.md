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

Terakhir, vektor http_request yang telah diproses dicetak ke konsol menggunakan `println!()`. Opsi pemformatan `#?` digunakan untuk mencetak vektor secara bagus, menampilkan setiap elemen pada baris terpisah.'
