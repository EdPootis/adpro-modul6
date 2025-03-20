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

<details>
<summary>Reflection 3</summary>

![Commit 3 screen capture](/assets/images/commit3.png)

Pada kode `handle_connection()` yang baru, pertama-tama terdapat penghapusan penggunaan `Vec<_>` yang sebelumnya dilakukan untuk menyimpan keseluruhan *request* HTTP. Sekarang digunakan variabel `request_line` yang hanya akan menyimpan baris pertama dari *request*.

Lalu pada kode baru respons dari server akan berbeda berdasarkan *request* client, jika request berupa "GET / HTTP/1.1" maka server akan memberikan respons HTTP "HTTP/1.1 200 OK" dan tampilan berupa file `hello.html`. Selain itu, akan diberikan respons HTTP "HTTP/1.1 404 NOT FOUND" dengan tampilan `404.html`.

Pada implementasi `handle_connection()` yang awal, kode dipisahkan di awal dengan sebuah *if-else block* hingga akhir. Dengan *refactoring*, *if-else block* hanya dilakukan di awal untuk menentukan variabel `status_line` dan `filename`. Setelah itu dilakukan pembuatan 3 variabel untuk memberikan respons dan respons diberikan. *Refactoring* yang dilakukan mengurangi duplikasi kode sehingga kedepannya *maintainability* lebih mudah dilakukan.

</details>

<details>
<summary>Reflection 4</summary>

Pada kode yang baru pemilihan *if-else block* diubah menjadi *match* (seperti *switch case*) untuk menambahkan *case* baru jika diakses endpoint `/sleep`. Hal ini juga membuat kode semakin rapih dibandingkan jika digunakan *if-else if-else block*.

Pada endpoint `/sleep`, akan terjadi *sleep* yang akan menunda eksekusi kode selama 10 detik, sehingga client akan menerima respons 10 detik setelah mengakses endpoint. Hal ini mengsimulasikan skenario di mana server memproses banyak permintaan dari berbagai klien sehingga membutuhkan waktu yang lama untuk seorang klien untuk menerima respons. Hal ini akan semakin terasa jika server menggunakan arsitektur *single-threaded* di mana server tidak dapat menangani banyak permintaan secara bersamaan.

</details>

<details>
<summary> Reflection 5</summary>

Pada kode yang baru, server menangani *request* menggunakan `ThreadPool` sehingga tidak lagi merupakan *single-threaded server* yang perlu menunggu untuk setiap *request*, melainkan beberapa *request* dapat dijalankan secara paralel dan tidak menghambat *request* lainnya.

Cara kerja `ThreadPool` dimulai dengan inisialisasinya menggunakan `ThreadPool::new(4)`, yang membuat 4 *worker threads*, jumlah *thread* dibatasi sehingga sistem tidak akan kehabisan resource jika ada DoS. Jika ada *request* yang masuk, maka `pool.execute(|| { handle_connection(stream); })` akan mengirimkan tugas ke antrian tugas (`mpsc::channel`) yang kemudian akan dieksekusi paralel oleh *worker*. Cara kerja *worker* bekerja adalah dengan membuat *thread*, dan melalui loop `receiver.lock().unwrap().recv()`, ia menunggu tugas masuk dan menjalankannya jika ada.

Dengan perubahan kode, server dapat menangani banyak koneksi secara bersamaan (tidak harus menunggu *request* yang lambat) dan server lebih efisien dalam menangani *request* (karena tidak harus menangani *request* secara berurutan)

</details>

<details open>
<summary>Bonus Reflection</summary>

Pada kode yang baru, ditambahkan fungsi `build()` untuk pembuatan `ThreadPool` dengan ErrorHandling. Sebelumnya digunakan `ThreadPool::new(size)` yang akan *panic* jika `size == 0`, tetapi sekarang dengan ditambahkan metode `ThreadPool::build(size) -> Result<ThreadPool, PoolCreationError>`, yang mengembalikan `Result`, jika ukuran lebih dari 0 maka akan dikembalikan
`Ok(ThreadPool)`. Jika ukuran adalah 0, maka dikembalikan `Err(PoolCreationError::ZeroSize)`. Dengan ini `ThreadPool` dapat menangani kesalahan dengan lebih baik tanpa harus *panic* dan menyebabkan crash.

Untuk membuat error yang baru tersebut, juga dibuat `PoolCreationError` yang merupakan enum yang dapat menampilkan pesan error dengan lebih informatif.
</details>

