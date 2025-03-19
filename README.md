# Advanced Programming Module 6
**Nama: Edmond Christian**<br>
**NPM: 2306208363**

<details>
<summary>Reflection 1</summary>

Fungsi `handle_connection()` memiliki parameter berupa sebuah `TcpStream`, yang merupakan sebuah koneksi dari seorang klien ke server. Jadi fungsi tersebut bertanggung jawab untuk menanangi koneksi yang masuk melalui TCP.<Br>
Pertama, fungsi ini membuat `BufReader` untuk membaca stream (dengan buffering data) yang diterima. Dari data tersebut, kemudian akan dipisahkan berdasarkan separator baris baru "\n" melalui `.lines()`. Setiap baris yang kemudian terbuat akan memiliki tipe `Result<String, std::io::Error>`, setelah itu digunakan `.map(|result| result.unwrap())` yang akan mengambil nilai stringnya. Jika terjadi error maka `.unwrap()` akan menghentikan program. Hal ini akan dilakukan hingga ditemukan baris-baris kosong yang menandakan akhir header `.take_while(|line| !line.is_empty())`. Setelah itu semua, dengan `.collect()`, hasilnya akan dikumpulkan ke dalam sebuah `Vec<String>` dan diprint ke terminal.


</details>

<details>
<summary>Reflection 2</summary>

![Commit 2 screen capture](/assets/images/commit2.png)

Pada versi baru `handle_connection()`, server tidak hanya membaca *request* dari client, tetapi juga mengirimkan response berupa dalam format HTTP.

Penambahan yang pertama adalah pembuatan `status_line` yang berupa string untuk menunjukkan respon HTTP dengan status 200 OK. Lalu variabel `contents` akan membaca isi file `hello.html` dan diubah menjadi string, dengan `.unwrap()` jika tidak ditemukan file maka program akan berhenti. Variabel `length` akan menyimpan panjang dari isi `contents`. Variabel terakhir, `response` akan menyusun ketiga variabel sebelumnya menjadi sebuah respons HTTP dengan format yang sudah disesuaikan sehingga `status_line` menampilkan status respons HTTP, `Content-Length: {length}` akan menampilkan ukuran dari respons HTTP, dan `{contents}` merupakan respons HTTP server.

```rust
stream.write_all(response.as_bytes()).unwrap();
```
Terakhir, akan dikirimkan respons tersebut ke client melalui jaringan `TcpStream`. Sehingga pada client terdapat halaman `Hello.html`.

</details>

<details open>
<summary>Reflection 3</summary>

![Commit 3 screen capture](/assets/images/commit3.png)

Pada kode `handle_connection()` yang baru, pertama-tama terdapat penghapusan penggunaan `Vec<_>` yang sebelumnya dilakukan untuk menyimpan keseluruhan *request* HTTP. Sekarang digunakan variabel `request_line` yang hanya akan menyimpan baris pertama dari *request*.

Lalu pada kode baru respons dari server akan berbeda berdasarkan *request* client, jika request berupa "GET / HTTP/1.1" maka server akan memberikan respons HTTP "HTTP/1.1 200 OK" dan tampilan berupa file `hello.html`. Selain itu, akan diberikan respons HTTP "HTTP/1.1 404 NOT FOUND" dengan tampilan `404.html`.

Pada implementasi `handle_connection()` yang awal, kode dipisahkan di awal dengan sebuah *if-else block* hingga akhir. Dengan *refactoring*, *if-else block* hanya dilakukan di awal untuk menentukan variabel `status_line` dan `filename`. Setelah itu dilakukan pembuatan 3 variabel untuk memberikan respons dan respons diberikan. *Refactoring* yang dilakukan mengurangi duplikasi kode sehingga kedepannya *maintainability* lebih mudah dilakukan.

</details>

