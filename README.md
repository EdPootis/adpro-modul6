# Advanced Programming Module 6
**Nama: Edmond Christian**<br>
**NPM: 2306208363**

<details open>
<summary>Reflection 1</summary>

Fungsi `handle_connection()` memiliki parameter berupa sebuah `TcpStream`, yang merupakan sebuah koneksi dari seorang klien ke server. Jadi fungsi tersebut bertanggung jawab untuk menanangi koneksi yang masuk melalui TCP.<Br>
Pertama, fungsi ini membuat `BufReader` untuk membaca stream (dengan buffering data) yang diterima. Dari data tersebut, kemudian akan dipisahkan berdasarkan separator baris baru "\n" melalui `.lines()`. Setiap baris yang kemudian terbuat akan memiliki tipe `Result<String, std::io::Error>`, setelah itu digunakan `.map(|result| result.unwrap())` yang akan mengambil nilai stringnya. Jika terjadi error maka `.unwrap()` akan menghentikan program. Hal ini akan dilakukan hingga ditemukan baris-baris kosong yang menandakan akhir header `.take_while(|line| !line.is_empty())`. Setelah itu semua, dengan `.collect()`, hasilnya akan dikumpulkan ke dalam sebuah `Vec<String>` dan diprint ke terminal.


</details>