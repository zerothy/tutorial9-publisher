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

### Spike 2 on RabbitMQ

![Spike 2 on RabbitMQ](./spike2.png)

### Mengapa Jumlah Antrian (Queue) di RabbitMQ Menunjukkan Angka Tertentu?

Jumlah total queue yang ada di RabbitMQ dapat dipengaruhi oleh beberapa faktor. Pertama, queue dapat dibuat oleh aplikasi kita sendiri, baik publisher maupun subscriber. Library seperti `crosstown_bus` sering membuat queue secara otomatis berdasarkan nama event atau topik. Jika kita menjalankan aplikasi beberapa kali atau mengkonfigurasi antrian sebagai durable dan tidak terhapus otomatis, jumlahnya akan bertambah. Kedua, queue bisa berasal dari eksekusi sebelumnya atau aplikasi lain. Jika instance RabbitMQ telah digunakan untuk proyek lain, queue durable dari sesi tersebut mungkin masih ada. Aplikasi lain di sistem yang menggunakan RabbitMQ juga akan memiliki queue mereka sendiri. Ketiga, beberapa library atau framework AMQP mungkin membuat lebih dari satu queue untuk tujuan tertentu, seperti queue untuk dead-letter messages atau queue sementara untuk komunikasi internal. Keempat, RabbitMQ sendiri atau plugin yang terinstall mungkin membuat beberapa queue untuk keperluan internal, meskipun biasanya jumlahnya tidak signifikan. Terakhir, selama pengembangan dan pengujian, seringkali kita membuat berbagai antrian untuk mencoba konfigurasi berbeda, dan jika tidak dibersihkan, antrian ini akan terakumulasi.

### Spike 3 on RabbitMQ

![Spike 3 on RabbitMQ](./spike3.png)

Ketika kita menjalankan beberapa instance subscriber yang listen ke queue yang sama, RabbitMQ akan mendistribusikan message-message dari queue tersebut kepada subscriber yang aktif, biasanya dalam mode round-robin. Ini berarti setiap subscriber akan mengambil dan memproses sebagian dari total message yang ada di qeue secara bersamaan. Akibatnya, keseluruhan proses message consumption menjadi jauh lebih cepat dibandingkan jika hanya ada satu subscriber. Inilah mengapa spike jumlah message dalam queue di RabbitMQ berkurang lebih cepat. Semakin banyak subscriber yang bekerja secara paralel, semakin cepat queue akan dikosongkan karena beban kerja terbagi di antara mereka. Fenomena ini menunjukkan kemampuan skalabilitas horizontal dari sistem antrian pesan seperti RabbitMQ, di mana kita dapat menambah lebih banyak subscriber untuk meningkatkan throughput dan mengurangi latensi dalam pemrosesan message.

### Potensi Peningkatan Kode

1. Error Handling di Publisher: Saat ini, hasil dari `p.publish_event(...)` diabaikan (`_ = ...`). Dalam real case scenario, penting untuk menghandle kemungkinan kegagalan saat publikasi pesan, misalnya dengan mencatat log error.
2.  Idempotensi Subscriber: Jika ada kemungkinan sebuah message bisa diproses lebih dari sekali, misalnya karena kegagalan sementara dan pengiriman ulang oleh broker, logika di sisi subscriber sebaiknya dirancang agar idempoten. Artinya, memproses pesan yang sama beberapa kali tidak akan menimbulkan efek samping yang tidak diinginkan.

