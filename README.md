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


