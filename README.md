## Tanya Jawab Publisher AMQP

**a. Berapa banyak data yang akan dikirim oleh program publisher Anda ke message broker dalam satu kali proses?**

Jumlah data yang dikirim oleh publisher ke message broker dalam satu kali proses akan bergantung pada implementasi spesifik dari program tersebut. Biasanya program publisher akan mengirimkan sejumlah message yang telah ditentukan sebelumnya atau mengirimkan message secara terus-menerus hingga kondisi tertentu terpenuhi. Setiap message akan berisi data yang relevan sesuai dengan tujuan aplikasi. Untuk mengetahui jumlah pastinya, kita perlu merujuk pada kode publisher yang digunakan. Kalau berdasarkan kode yang ada, publisher akan mengirimkan 5 message ke message broker dalam satu kali proses.

**b. URL “amqp://guest:guest@localhost:5672” sama dengan yang ada di program subscriber, apa artinya?**

Kesamaan URL "amqp://guest:guest@localhost:5672" pada publisher dan subscriber memiliki arti bahwa kedua program publisher dan subscriber dikonfigurasi untuk terhubung ke instance message broker RabbitMQ yang sama. `localhost` menunjukkan bahwa message broker berjalan di mesin yang sama (komputer lokal) dengan publisher dan subscriber, `5672` adalah port standar yang digunakan oleh RabbitMQ untuk koneksi AMQP. Dengan kata lain, baik publisher maupun subscriber menggunakan kredensial default untuk mengakses message broker yang berjalan secara lokal pada port standar. Ini memastikan bahwa message yang dikirim oleh publisher dapat diterima oleh subscriber melalui broker yang sama.

### Running RabbitMQ

![RabbitMQ](./running_rabbitmq.png)

### Running Publisher and Subscriber
![Publisher and Subscriber](./running_event.png)

### Spike on RabbitMQ

![Spike on RabbitMQ](./spike.png)

### Analisis Spike pada RabbitMQ

Spike atau lonjakan yang terlihat pada grafik RabbitMQ berkaitan langsung dengan eksekusi program publisher. Ketika program publisher dijalankan, ia mengirimkan 5 pesan ke message broker RabbitMQ dalam waktu yang singkat. Pengiriman pesan secara cepat dan berurutan ini menyebabkan peningkatan tajam dalam jumlah pesan yang masuk dan antri di broker. Interface management RabbitMQ akan menampilkan lonjakan ini pada grafiknya, yang merepresentasikan aktivitas pengiriman pesan yang tiba-tiba meningkat. Lonjakan ini menunjukkan bahwa broker berhasil menerima dan memproses pesan-pesan yang dikirim oleh publisher. Setelah pesan-pesan tersebut dikonsumsi oleh subscriber atau mencapai batas waktu hidupnya, grafik akan kembali normal.