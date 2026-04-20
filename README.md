## Commit 1 Reflection Notes

**Commit Message:** `(1) Handle-connection, check response`

### Reflection

Pada commit pertama ini, saya berhasil mengimplementasikan fungsi `handle_connection` yang bertugas menangani koneksi dari client dan membaca HTTP request yang dikirimkan.

### Apa yang Dilakukan di Kode

- Membuat `TcpListener` yang mendengarkan di alamat `127.0.0.1:7878`.
- Loop melalui `listener.incoming()` untuk menerima setiap koneksi baru.
- Pada fungsi `handle_connection`:
  - Menggunakan `BufReader::new(&mut stream)` untuk membungkus `TcpStream` agar bisa membaca data dengan lebih nyaman dan efisien.
  - Membaca seluruh request menggunakan `.lines()` yang mengembalikan iterator dari setiap baris teks.
  - Menggunakan `.map(|result| result.unwrap())` untuk mengambil nilai `String` dari `Result<String, _>`.
  - Menggunakan `.take_while(|line| !line.is_empty())` untuk berhenti membaca saat menemukan baris kosong. Hal ini sangat penting karena menurut spesifikasi HTTP/1.1, bagian header selalu diakhiri dengan satu baris kosong (`\r\n\r\n`).
  - Mengumpulkan semua baris tersebut menjadi `Vec<String>` dengan `.collect()`.
  - Menampilkan isi request menggunakan `println!("Request: {:#?}", http_request);` agar output lebih mudah dibaca (pretty print).

### Pembelajaran dan Insight

- Saya belajar bahwa HTTP request bukan sekadar stream byte biasa, melainkan memiliki struktur yang jelas: request line, header fields, dan diakhiri dengan baris kosong.
- `BufReader` jauh lebih praktis dibandingkan membaca manual byte-per-byte karena sudah menyediakan metode `.lines()` dan buffering otomatis.
- Teknik iterator chaining di Rust (`map` + `take_while` + `collect`) sangat powerful dan membuat kode terlihat bersih, deklaratif, dan idiomatik.
- Penggunaan `.unwrap()` masih banyak digunakan di sini karena fokus utama adalah memahami alur dasar. Di tahap selanjutnya, saya akan menggantinya dengan penanganan error yang lebih baik menggunakan `Result` atau `?` operator.
- Memahami konsep ownership dan mutable borrow (`mut stream: TcpStream`) juga semakin jelas saat bekerja dengan stream.

### Tantangan yang Dihadapi

- Awalnya agak bingung kenapa harus berhenti pada baris kosong. Setelah membaca dokumentasi Rust dan penjelasan HTTP di The Rust Book, baru saya paham bahwa itu adalah batas antara header dan body.
- Perlu memahami tipe data yang dikembalikan oleh `.lines()` yaitu `Lines<BufReader<&mut TcpStream>>` dan bagaimana cara mengubahnya menjadi `Vec<String>`.

### Kesimpulan

Commit ini berhasil membuat server sederhana yang dapat menerima koneksi dari browser dan menampilkan isi HTTP request-nya di terminal. Ini merupakan langkah awal yang sangat penting dalam membangun web server dari nol menggunakan Rust.

Fungsi ini sudah sesuai dengan materi di chapter "Building a Single-Threaded Web Server" pada The Rust Book.

**Next Steps:**
- Mengirimkan HTTP response kembali ke client (minimal response 200 OK dengan HTML sederhana).
- Memisahkan parsing request line, headers, dan body.
- Meningkatkan error handling.
- Menambahkan kemampuan untuk melayani file statis.

---


# Rust HTTP Server - Hello World

Proyek sederhana HTTP Server menggunakan Rust yang mengembalikan halaman HTML statis.

## Deskripsi Proyek
Proyek ini adalah implementasi web server sederhana menggunakan `std::net::TcpListener` dan `TcpStream`. Server ini merespons setiap request dengan halaman HTML dari file `hello.html`.

## Commit 2 Reflection Notes

### (2) Returning HTML

Pada commit ini, saya telah berhasil mengubah fungsi `handle_connection` agar tidak hanya membaca HTTP request, tetapi juga mengembalikan respons berupa **halaman HTML** yang valid.

#### Perubahan yang dilakukan:
- Menambahkan `use std::fs;` untuk membaca file `hello.html`
- Membuat response HTTP dengan status `200 OK` dan header `Content-Length`
- Mengirimkan isi file HTML menggunakan `stream.write_all()`
- Menggunakan format string yang benar dengan `\r\n` untuk memisahkan header dan body

#### Screenshot Hasil:

![Commit 2 screen capture](assets\images\image.png)

#### Reflection:
Sekarang server saya sudah bisa mengembalikan halaman web yang proper, bukan hanya teks biasa. Saya belajar pentingnya:
- Format HTTP response yang benar (status line + headers + blank line + body)
- Penggunaan `Content-Length` header
- Cara membaca file eksternal di Rust menggunakan `fs::read_to_string`

Pesan di halaman HTML sudah saya ubah menjadi:

> "Hi from Rust, running from tirta's machine."



---

## Cara Menjalankan

```bash
cargo run



# Rust HTTP Server - Hello World

Proyek sederhana HTTP Server menggunakan Rust.

## Commit 3 Reflection Notes

### (3) Validating the Request and Selectively Responding

Pada milestone ini, saya merefaktor fungsi `handle_connection` agar server dapat memvalidasi request dari client dan memberikan respons yang berbeda sesuai dengan permintaan.

#### Perubahan yang dilakukan:
- Membaca baris pertama HTTP request (`request_line`)
- Menggunakan conditional `if-else` untuk menentukan jenis respons:
  - Jika request adalah **`GET / HTTP/1.1`** → mengembalikan `hello.html` dengan status **200 OK**
  - Jika request selain itu (misalnya `/bad`, `/about`, dll) → mengembalikan `404.html` dengan status **404 NOT FOUND**
- Memisahkan logic antara response sukses dan response error

#### Alasan Refactoring Diperlukan:
Sebelumnya server selalu mengembalikan `hello.html` meskipun URL yang diminta tidak ada. Ini tidak realistis karena web server seharusnya memberikan respons yang sesuai (success atau error). Dengan validasi request, server menjadi lebih pintar dan sesuai dengan perilaku HTTP yang sebenarnya.



#### Reflection:
Dari milestone ini saya memahami pentingnya mem-parsing HTTP request meskipun hanya baris pertama. Saya juga belajar bagaimana memisahkan logic response agar kode lebih mudah dibaca dan dikelola di masa depan. Refactoring ini menjadi dasar untuk menambahkan routing yang lebih kompleks nantinya.

**Commit Message:** `(3) Validating request and selectively responding`


## Commit 4 Reflection Notes

### (4) Simulation of slow request

Pada milestone ini, saya mensimulasikan masalah pada server single-threaded dengan menambahkan route `/sleep` yang sengaja di-delay selama 10 detik menggunakan `thread::sleep`.

#### Perubahan yang dilakukan:
- Menambahkan `thread` dan `time::Duration` ke dependencies
- Menggunakan `match` expression untuk menangani route yang berbeda
- Menambahkan delay 10 detik pada route `GET /sleep HTTP/1.1`

#### Pengamatan:
Ketika saya membuka `http://127.0.0.1:7878/sleep` di satu tab browser, tab lain yang mengakses `http://127.0.0.1:7878` juga menjadi sangat lambat (terblokir selama 10 detik). Ini menunjukkan bahwa server hanya bisa memproses satu request dalam satu waktu.



#### Reflection:
Dari simulasi ini saya mengerti kenapa single-threaded server tidak cocok untuk web server sungguhan. Satu request lambat saja bisa membuat semua user lain menunggu. Ini menjadi alasan kuat kenapa kita perlu menggunakan multithreading di milestone berikutnya agar server bisa melayani banyak request secara bersamaan.

**Commit Message:** `(4) Simulation of slow request`




## Commit 5 Reflection Notes

### (5) Multithreaded server using Threadpool

Pada milestone ini, saya mengubah server dari single-threaded menjadi multithreaded menggunakan `ThreadPool` custom.

#### Perubahan yang dilakukan:
- Membuat module `ThreadPool` di `src/lib.rs` yang menggunakan `mpsc` channel, `Arc<Mutex>`, dan `thread::spawn`
- Mengubah `main.rs` untuk menggunakan `ThreadPool::new(4)`
- Mengganti loop `for stream in listener.incoming()` dengan `pool.execute(|| { handle_connection(stream) })`

#### Pengamatan:
Sekarang ketika saya membuka banyak tab sekaligus (termasuk `/sleep`), server tetap responsif. Request lambat tidak lagi memblokir request lain. Server dapat menangani hingga 4 request secara paralel.



#### Reflection:
Dengan ThreadPool, saya belajar konsep **thread pooling**, **message passing** menggunakan channel, dan **shared ownership** dengan `Arc<Mutex>`. Ini jauh lebih baik dibandingkan single-threaded server. Sekarang server saya lebih siap menghadapi banyak user secara bersamaan.

**Commit Message:** `(5) Multithreaded server using Threadpool`



