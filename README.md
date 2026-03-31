# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. Pada diagram Observer pattern, Subscriber didefinisikan sebagai interface karena pola tersebut dirancang untuk mendukung banyak pengamat yang mungkin memiliki perilaku berbeda saat "mendengar" perubahan dari subject, yang bisa disebut sebagai publisher. Subject cukup mengetahui bahwa setiap observer memiliki method tertentu, misalnya update(), tanpa perlu tahu implementasi detailnya. Namun, pada kasus BambangShop, menurut saya kita tidak lagi membutuhkan interface atau trait untuk Subscriber, dan penggunaan satu model struct saja sudah memadai.

Alasannya adalah karena peran Subscriber di sini lebih bersifat sebagai representasi data daripada entitas dengan banyak variasi perilaku di dalam satu aplikasi. Publisher hanya perlu mengetahui informasi dasar subscriber, seperti url dan name, lalu mengirimkan notifikasi melalui HTTP request ke endpoint subscriber tersebut. Artinya, mekanisme “merespons update” tidak diimplementasikan melalui banyak objek berbeda di dalam proses yang sama, melainkan dipindahkan ke aplikasi receiver yang terpisah. Jadi, variasi perilaku observer yang biasanya menjadi alasan utama penggunaan interface pada Observer pattern tidak benar-benar muncul di level kode publisher ini.

Selain itu, trait akan lebih berguna jika di dalam publisher ada beberapa jenis subscriber dengan logika internal yang berbeda-beda. Pada BambangShop, yang dibutuhkan hanya data subscriber dan alamat tujuan notifikasi. Oleh karena itu, menggunakan struct Subscriber adalah keputusan yang paling sederhana dan sesuai dengan konteks aplikasi.

2. Menurut saya, karena id pada Program dan url pada Subscriber memang dimaksudkan unik, maka penggunaan map seperti DashMap jauh lebih tepat dibandingkan Vec. Jika menggunakan Vec, data hanya disimpan sebagai urutan elemen tanpa jaminan keunikan bawaan. Akibatnya, setiap kali kita ingin menambahkan subscriber baru, kita perlu memeriksa secara manual apakah url tersebut sudah ada. Hal yang sama juga terjadi saat ingin menghapus atau mencari subscriber tertentu, kita harus menelusuri isi vector satu per satu. Pendekatan seperti ini kurang efisien dan membuat logika program menjadi lebih panjang serta lebih mudah menimbulkan bug, sedangkan penggunaan map lebih sederhana dikarenakan sifat map sendiri yang menyimpan key-value sehingga kita bisa langsung menemukan subscriber secara langsung hanya dengan menyimpan id sebagai key pada suatu map.

3. Dalam konteks design pattern, Singleton digunakan untuk memastikan bahwa hanya ada satu instance bersama yang mengelola data repository. Pada BambangShop, ini memang relevan karena kita ingin seluruh bagian aplikasi mengakses sumber data subscriber yang sama, bukan membuat repository yang berbeda-beda untuk tiap request. Namun, menurut saya Singleton saja tidak cukup untuk kasus ini. Alasannya aplikasi web tidak berjalan secara single-threaded sederhana. Dalam Rocket, beberapa request bisa datang hampir bersamaan, sehingga data repository dapat diakses dan dimodifikasi oleh banyak thread pada waktu yang sama. Jika kita hanya membuat Singleton biasa tanpa mekanisme thread safety, maka akan muncul risiko race condition, data tidak konsisten, atau bahkan panic ketika beberapa thread mencoba membaca dan menulis data bersamaan. Di sinilah DashMap menjadi penting. lazy_static memang membantu kita membuat shared global state yang sifatnya mirip Singleton, tetapi lazy_static hanya menyelesaikan persoalan “satu instance global”, bukan persoalan keamanan akses bersamaan. DashMap melengkapi kebutuhan tersebut dengan menyediakan struktur data map yang aman untuk concurrent access. Dengan kata lain, pada implementasi ini, lazy_static berperan sebagai mekanisme singleton-like storage, sedangkan DashMap berperan sebagai thread-safe container.

# (Biar bisa pull request)

#### Reflection Publisher-2

1. Dalam pola MVC klasik, Model memang mencakup data sekaligus logika bisnis. Namun, pada implementasi seperti BambangShop, menurut saya pemisahan antara Model, Service, dan Repository tetap penting karena membuat tanggung jawab tiap komponen menjadi lebih jelas. Model sebaiknya hanya berperan sebagai representasi data, misalnya menyimpan atribut-atribut dari Program, Subscriber, atau Notification. Service bertugas menangani business logic, seperti aturan subscribe, unsubscribe, publish, atau notify. Sementara itu, Repository berfokus pada cara data disimpan, diambil, dan dihapus dari struktur penyimpanan yang digunakan aplikasi.

Pemisahan ini penting karena sesuai dengan prinsip *Single Responsibility Principle*. Jika satu model dipaksa menangani bentuk data, validasi, aturan bisnis, dan akses penyimpanan sekaligus, maka model tersebut akan memiliki terlalu banyak alasan untuk berubah. Misalnya, perubahan pada aturan bisnis subscribe seharusnya hanya memengaruhi service, sedangkan perubahan pada mekanisme penyimpanan data seharusnya hanya memengaruhi repository. Jika semua itu digabung dalam model, maka perubahan kecil pada satu aspek bisa merambat ke seluruh bagian model. Akibatnya, kode menjadi lebih sulit dirawat, lebih sulit diuji, dan lebih rentan menimbulkan efek samping yang tidak diinginkan.

Selain itu, pemisahan ini juga membantu menjaga kode tetap modular. Service dapat diuji sebagai logika bisnis tanpa perlu terlalu bergantung pada detail penyimpanan, sedangkan repository dapat diuji sebagai komponen akses data tanpa harus mencampur aturan bisnis. Menurut saya, inilah alasan utama kenapa pada BambangShop, walaupun secara konsep MVC klasik Model bisa mencakup semuanya, dalam praktik yang lebih maintainable justru lebih baik jika Service dan Repository dipisahkan dari Model. 

2. Jika kita hanya menggunakan Model saja, maka interaksi antar model seperti Program, Subscriber, dan Notification akan membuat kompleksitas kode meningkat cukup tajam. Dalam bayangan saya, setiap model tidak lagi hanya menjadi representasi data, tetapi juga harus mengetahui cara berinteraksi dengan model lain, mengetahui aturan bisnis, sekaligus mengetahui cara menyimpan atau mengambil data. Akibatnya, batas tanggung jawab antar class menjadi kabur. Menurut saya, jika hanya menggunakan Model, kompleksitas tiap model justru akan membesar. Program tidak lagi sekadar model produk, Subscriber tidak lagi sekadar model subscriber, dan Notification tidak lagi sekadar model notifikasi. Semua akan berubah menjadi campuran data structure, business logic, dan data access sekaligus. Desain seperti ini memang mungkin tetap berjalan, tetapi akan jauh lebih sulit dirawat dibandingkan desain yang memisahkan peran model, service, dan repository. 

3. Saya cukup tertarik dengan penggunaan Postman dalam tutorial ini karena tool ini sangat membantu untuk menguji endpoint tanpa harus langsung membuat antarmuka frontend. Pada konteks BambangShop, Postman mempermudah saya untuk mengirim HTTP request ke endpoint yang sedang saya kerjakan, misalnya endpoint subscribe dan unsubscribe, lalu melihat apakah response yang dikembalikan sudah sesuai atau belum. Dengan cara ini, saya bisa fokus memverifikasi apakah route, request body, parameter, dan response status dari backend saya sudah benar. Menurut saya, kelebihan terbesar Postman adalah proses testing menjadi jauh lebih cepat dan lebih terstruktur. Saya bisa mencoba berbagai skenario, misalnya request yang valid, request dengan parameter yang salah, atau request ke endpoint yang belum lengkap implementasinya. Dari sana saya bisa langsung melihat error response dan memperbaiki bagian service atau controller yang bermasalah. Ini sangat membantu karena pada tahap pengembangan backend, sering kali yang ingin dicek pertama bukan tampilan aplikasi, tetapi apakah kontrak API-nya sudah bekerja dengan benar. Jadi, menurut saya Postman sangat membantu dalam proses pengujian kerja saat ini karena ia menjadi alat verifikasi cepat untuk endpoint yang sedang dibuat. 

#### Reflection Publisher-3
