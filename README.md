**Tutorial 10 Pemrograman Lanjut (Advanced Programming) 2024/2025 Genap**
* Nama    : Muhammad Almerazka Yocendra
* NPM     : 2306241745
* Kelas   : Pemrograman Lanjut - A

## 🍊 Original Code

![image](https://github.com/user-attachments/assets/7db1aedf-241d-4a24-b84b-de2eb66ab11c)

- Berdasarkan gambar diatas, terlihat implementasi sistem _chat broadcast_ menggunakan `WebSocket` dengan **Tokio** yang mendemonstrasikan komunikasi _real-time_ antara _multiple clients_ dan _server_. Server berjalan di port **2000** dan menerima koneksi dari tiga _client_ berbeda dengan alamat `IP 127.0.0.1` dan port yang berbeda-beda (65097, 65099, 65100). Setiap kali ada client baru yang terhubung, server mencetak `"New connection from [alamat]"` dan mengirimkan pesan welcome `"Welcome to chat! Type a message"` kepada client tersebut.
- Mekanisme _broadcast_ bekerja melalui `tokio::sync::broadcast` _channel_ yang memungkinkan server untuk mendistribusikan pesan dari satu _client_ ke semua _client_ yang terhubung. Ketika _client_ mengetik pesan seperti `"Client pertama!"`, `"Client kedua!"`, atau `"Client ketiga!"`, server menerima pesan tersebut melalui **WebSocket** _connection_, mencetak log `"From client [alamat] [pesan]"`, kemudian mem-_broadcast_ pesan tersebut melalui _broadcast channel_. Semua _client_ yang terhubung akan menerima pesan ini dan menampilkannya dengan format `"From server: [pesan]"`.
- Penggunaan `tokio::select!` pada kedua sisi (_client_ dan _server_) memungkinkan _concurrent handling_ dari _multiple operations_. Pada _client_, `tokio::select!` menangani dua operasi secara bersamaan : membaca input dari keyboard (**stdin**) untuk mengirim pesan ke _server_, dan menerima pesan dari **WebSocket** untuk menampilkan pesan dari _server_. Pada _server_, setiap _connection_ menggunakan `tokio::select!` untuk menangani penerimaan pesan dari _client_ dan pengiriman _broadcast_ _message_ ke _client_ secara _concurrent_. Hal ini memungkinkan _chat application_ untuk _responsive_ terhadap input _user_ dan _broadcast message_ secara _real-time_ tanpa _blocking_, menciptakan pengalaman _chat_ yang _smooth_ di mana semua _participants_ dapat mengirim dan menerima pesan secara _simultaneous_.


## 🍋 Modifying the websocket port

![image](https://github.com/user-attachments/assets/211ebe56-93b2-4752-82e9-f84479648e75)

Untuk mengubah port dari **2000** ke **8080**, diperlukan modifikasi pada kedua file karena B adalah protokol berbasis koneksi yang memerlukan sinkronisasi antara _client_ dan _server_. Pada `server.rs`, kita harus mengubah `TcpListener::bind("127.0.0.1:2000")` menjadi `TcpListener::bind("127.0.0.1:8080")` agar server mendengarkan di port **8080**. Sedangkan pada `client.rs`, **URI** harus diubah dari `ws://127.0.0.1:2000` menjadi `ws://127.0.0.1:8080` agar _client_ terhubung ke port yang benar. Jika hanya salah satu yang diubah, koneksi akan gagal dengan error **"Connection refused"** karena _client_ mencoba _connect_ ke port yang berbeda dari yang di-_listen_ _server_.

## 🥑 Small changes. Add some information to client

![image](https://github.com/user-attachments/assets/7d660e8b-8098-4f49-bc5b-89e5725cb686)

Untuk menambahkan informasi pengirim pada setiap pesan _chat_, dilakukan modifikasi pada `server.rs` dalam fungsi `handle_connection` dengan mengubah format pesan yang di-_broadcast_. Alih-alih hanya mengirim teks pesan asli melalui `bcast_tx.send(text.into())`, format pesan diubah menjadi `bcast_tx.send(format!("{addr:?} : {text}"))` sehingga setiap pesan yang di-_broadcast_ akan menyertakan informasi alamat **IP** dan **port** pengirim. Perubahan ini memungkinkan semua _client_ yang menerima pesan dapat mengetahui siapa pengirimnya berdasarkan alamat koneksi, seperti yang terlihat pada output dimana pesan **"Hello"** ditampilkan sebagai` "From server: 127.0.0.1:49324 : Hello" ` yang menunjukkan bahwa pesan tersebut dikirim oleh _client_ dengan alamat `127.0.0.1:49324`. Modifikasi sederhana ini meningkatkan transparansi komunikasi dalam _chat room_ dengan memberikan konteks tentang identitas pengirim tanpa memerlukan sistem autentikasi yang kompleks.