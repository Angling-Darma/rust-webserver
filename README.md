Commit 1 Reflection notes:

Mengembangkan sebuah web server tunggal-utas dengan Rust merupakan langkah awal yang krusial dalam perjalanan membangun sebuah aplikasi server yang handal. Melalui penggunaan std::net::TcpListener, saya mulai dengan mendengarkan koneksi masuk dari jaringan. Fungsi utama mengikat server pada alamat IP dan port tertentu (127.0.0.1:7878), menandakan server siap menerima koneksi TCP. Saat koneksi berhasil terestabilkan, server akan memberikan respons sederhana berupa pesan "Connection established!" yang ditampilkan pada konsol. Meskipun tahap ini masih sangat dasar, namun sangat penting sebagai batu loncatan untuk memahami bagaimana koneksi TCP bekerja serta model pemrograman Rust.

Setelah server dijalankan dan diakses melalui browser pada http://127.0.0.1:7878, saya bisa melihat bahwa implementasi awal saya belum mampu memberikan respons kepada browser. Ini mengajarkan saya pentingnya tidak hanya menerima koneksi, tetapi juga memprosesnya dengan tepat.

Pengenalan fungsi handle_connection merupakan langkah pengembangan berikutnya yang signifikan. Fungsi ini memproses TcpStream yang masuk, mengekstrak dan mencetak permintaan HTTP baris demi baris. Penggunaan BufReader untuk membungkus aliran memungkinkan pembacaan dan manipulasi data secara efisien. Pembaruan ini memberikan pemahaman yang lebih dalam mengenai struktur permintaan HTTP dan protokol komunikasi antara server web dan klien. Server kini dapat membaca dan menampilkan permintaan HTTP penuh dari browser, menawarkan contoh konkret dari interaksi server-klien.

Melalui pencapaian ini, saya belajar tentang pentingnya memproses koneksi dengan efektif dan mengenal konsep-konsep kunci serta modul-modul standar perpustakaan Rust. Peringatan tentang variabel yang tidak digunakan dan saran yang diberikan oleh kompiler Rust menyoroti fokus bahasa terhadap keselamatan dan efisiensi. Dengan mengatasi peringatan ini, pengembang dapat menyempurnakan kode mereka, mengikuti praktik terbaik dan mengoptimalkan kinerja.

Commit 2 Reflection notes:

Pada Milestone kedua ini, saya telah membuat kemajuan signifikan dalam pengembangan server web saya dengan Rust. Modifikasi pada metode handle_connection memungkinkan server tidak hanya menerima koneksi masuk tetapi juga merespons dengan halaman HTML sederhana yang dapat dirender oleh browser. Perubahan ini mengubah server dari hanya menampilkan pesan di konsol menjadi server web yang dapat melayani konten web.

Penggunaan Rust's file system module (fs) untuk membaca konten file HTML dan kemudian mengirimkannya sebagai respons ke browser merupakan langkah penting. Ini memperkenalkan konsep penting dalam pengembangan web, yaitu pelayanan konten statis. Melalui format string yang efisien, server sekarang mampu mengirimkan status line, headers seperti Content-Length, dan konten HTML itu sendiri ke klien. Penggunaan Content-Length di headers adalah contoh bagaimana HTTP headers digunakan untuk memberikan informasi penting tentang respons kepada klien, dalam hal ini ukuran dari konten yang dikirim.

Proses ini menyoroti bagaimana protokol HTTP beroperasi dan bagaimana server web berinteraksi dengan browser menggunakan protokol tersebut. Dengan mengirimkan respons HTTP yang benar, termasuk status line yang menandakan kesuksesan (misalnya, "HTTP/1.1 200 OK") dan konten yang tepat, saya dapat membuat browser menampilkan halaman web yang diinginkan. Ini juga memberikan pengalaman praktis dengan format respons HTTP dan pentingnya mematuhi standar protokol untuk interoperabilitas.

Selain itu, pengalaman ini mengajarkan tentang pentingnya pengujian lokal dan modifikasi konten. Mengedit file hello.html untuk menyesuaikan pesan yang ditampilkan oleh server memperkenalkan konsep pengembangan web iteratif, di mana pengembang secara berkala memodifikasi dan menguji perubahan mereka untuk mencapai hasil yang diinginkan.
![Commit 2 screen capture](/assets/images/commit2.png)

Commit 3 Reflection notes:

Milestone ketiga dalam pengembangan server web kita dengan Rust membawa kita pada tingkat baru dalam hal kefleksibelan dan keamanan: validasi permintaan dan respons selektif. Perubahan ini memperkenalkan logika untuk membedakan antara permintaan yang valid dan tidak valid dari klien. Sebelumnya, server kita akan mengembalikan konten HTML tanpa mempertimbangkan jenis permintaan yang dibuat oleh browser. Dengan penambahan fungsi untuk memeriksa apakah permintaan tersebut adalah untuk jalur /, server kita sekarang dapat merespons secara berbeda tergantung pada permintaan yang diterima.

Ketika server menerima permintaan GET / HTTP/1.1, yang merupakan permintaan untuk halaman utama, ia mengembalikan konten dari file hello.html dengan status 200 OK. Ini menunjukkan bahwa permintaan telah berhasil diproses dan konten yang diminta tersedia. Perubahan ini menjaga fungsi dasar server kita dalam melayani halaman utama.

Namun, jika server menerima permintaan untuk jalur yang tidak dikenali, seperti /bad, maka logika baru yang diperkenalkan akan mengaktifkan blok else. Di sini, server mengembalikan respons dengan status 404 NOT FOUND dan konten dari file 404.html, memberitahu pengguna bahwa halaman yang diminta tidak dapat ditemukan. Hal ini penting untuk memberikan feedback yang jelas kepada pengguna saat mereka mencoba mengakses halaman yang tidak ada atau salah ketik URL.

Penerapan respons 404 ini tidak hanya meningkatkan keamanan server dengan menghindari pengungkapan informasi tentang struktur direktori internal atau file yang tidak seharusnya diakses secara publik, tapi juga meningkatkan pengalaman pengguna dengan menyediakan pesan kesalahan yang lebih informatif dan ramah. ![Commit 3 screen capture](/assets/images/commit3.png)

Commit 4 Reflection notes:

Milestone keempat ini menghadirkan simulasi yang sangat berguna untuk memahami keterbatasan dari server web berbasis single-thread yang kita kembangkan dengan Rust. Dengan menambahkan simulasi respons lambat pada salah satu jalur endpoint, yaitu /sleep, kita dapat melihat secara langsung bagaimana satu permintaan yang memakan waktu lebih lama dapat mempengaruhi kemampuan server untuk merespons permintaan lain.

Pada modifikasi kode yang diperkenalkan, permintaan ke /sleep menyebabkan server menjeda selama 10 detik sebelum merespons. Ini dilakukan melalui fungsi thread::sleep(Duration::from_secs(10));, yang menghentikan eksekusi thread saat ini untuk durasi yang ditentukan. Ketika sebuah permintaan ke /sleep diterima, semua permintaan lain yang datang ke server selama periode tersebut harus menunggu, termasuk permintaan ke / yang seharusnya dapat dilayani hampir seketika.

Melalui eksperimen ini, kita dapat mengamati dan memahami beberapa konsep penting dalam pengembangan aplikasi web server, yaitu:

1. Batasan Single-thread: Server single-thread memproses permintaan satu demi satu dalam urutan yang diterima. Ini berarti bahwa permintaan yang lambat atau berat dapat memblokir eksekusi permintaan berikutnya, menciptakan bottleneck pada kinerja server.
2. Pentingnya Concurrency: Dalam lingkungan produksi, server web harus mampu menangani banyak permintaan secara bersamaan tanpa membiarkan satu permintaan lambat menghambat yang lain. Ini biasanya dicapai melalui model multi-threading atau asynchronous I/O.
3. User Experience: Waktu respons server yang lambat dapat berdampak negatif pada pengalaman pengguna, yang menjadikannya faktor penting dalam desain dan optimisasi server web.

Eksperimen ini mengilustrasikan bagaimana desain arsitektur server yang tidak mempertimbangkan concurrency dapat menghadapi masalah skalabilitas saat traffic meningkat. Meskipun untuk tujuan pengembangan dan pembelajaran, model single-thread ini berguna untuk memahami dasar-dasar pengolahan permintaan HTTP, penting untuk mengeksplorasi model concurrency dan asynchronous processing dalam pembuatan aplikasi web server yang siap produksi.

Commit 5 Reflection notes:


Milestone kelima ini merupakan langkah signifikan dalam evolusi server web yang kita kembangkan dengan Rust, membawa kita pada implementasi multi-threading yang meningkatkan kemampuan server dalam menangani permintaan secara simultan. Dengan memperkenalkan ThreadPool, server kini mampu memproses lebih dari satu permintaan pada satu waktu, mengatasi keterbatasan dari model single-thread yang telah kita eksplor sebelumnya.

Implementasi ThreadPool ini didasarkan pada beberapa konsep penting dalam pemrograman paralel dan konkuren:

mpsc (multiple producer, single consumer) digunakan untuk membuat channel komunikasi antara thread utama (yang menerima permintaan) dan thread pekerja dalam pool. Hal ini memungkinkan server untuk mendelegasikan pekerjaan kepada thread pekerja yang tersedia.
Arc (Atomic Reference Counting) dan Mutex (Mutual Exclusion) digunakan bersama untuk membagi akses ke receiver di antara beberapa thread secara aman. Arc memungkinkan receiver untuk dibagi antara beberapa pemilik (thread pekerja), dan Mutex memberikan akses eksklusif ke receiver, memastikan bahwa hanya satu thread yang dapat membaca dari channel pada satu waktu.
Penggunaan ThreadPool memungkinkan server untuk merespons permintaan /sleep yang memerlukan waktu lebih lama tanpa menghalangi permintaan lain untuk diproses. Sebagai contoh, ketika satu thread pekerja sedang menangani permintaan /sleep, permintaan lain ke / dapat langsung diteruskan ke thread pekerja lain yang tersedia, meningkatkan throughput dan efisiensi server.

Perubahan ini tidak hanya meningkatkan performa dan skalabilitas server tetapi juga memberikan pengalaman pengguna yang lebih baik, dengan mengurangi waktu tunggu untuk pemrosesan permintaan. Selain itu, penerapan ThreadPool memberikan dasar yang kuat untuk pengembangan lebih lanjut, menjadikan server lebih siap untuk lingkungan produksi yang menuntut kemampuan untuk menangani banyak permintaan secara efisien.