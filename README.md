
1. **Keamanan**:
   Komunikasi melalui protokol HTTP tidak terenkripsi, yang berarti data yang dikirimkan antara klien (aplikasi Android) dan server dapat dibaca oleh pihak ketiga jika mereka berhasil menyadap koneksi tersebut. Ini dapat menyebabkan masalah keamanan seperti kebocoran data sensitif, pencurian sesi, atau serangan man-in-the-middle. Oleh karena itu, menggunakan HTTPS sangat direkomendasikan untuk aplikasi produksi.

2. **Kebijakan Keamanan Jaringan Android**:
   Sejak Android 9 (API level 28), kebijakan keamanan jaringan Android secara default melarang aplikasi Android untuk melakukan komunikasi melalui protokol HTTP yang tidak aman (cleartext). Ini dilakukan untuk meningkatkan keamanan aplikasi. Oleh karena itu, Anda harus mengonfigurasi kebijakan keamanan jaringan di aplikasi Android Anda untuk mengizinkan komunikasi CLEARTEXT ke alamat IP atau domain tertentu.

3. **Persyaratan Aplikasi dan Toko Aplikasi**:
   Beberapa toko aplikasi seperti Google Play Store mungkin memiliki persyaratan keamanan yang lebih ketat dan mungkin menolak aplikasi yang melakukan komunikasi melalui HTTP yang tidak aman. Ini dapat membatasi distribusi aplikasi Anda.

Untuk mengatasi masalah ini, solusi terbaik adalah mengonfigurasi server backend Anda untuk menggunakan HTTPS dengan sertifikat SSL/TLS yang valid. Ini akan menyediakan komunikasi yang aman dan terenkripsi antara aplikasi Android dan server backend.

Namun, jika Anda memang harus menggunakan HTTP untuk sementara waktu (misalnya, untuk tujuan pengembangan atau pengujian), Anda dapat mengonfigurasi kebijakan keamanan jaringan di aplikasi Android Anda untuk mengizinkan komunikasi CLEARTEXT. Lakukan langkah-langkah berikut:

1. Buat file `res/xml/network_security_config.xml` dengan isi:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">IP_ATAU_DOMAIN_SERVER_ANDA</domain>
    </domain-config>
</network-security-config>
```

Ganti `IP_ATAU_DOMAIN_SERVER_ANDA` dengan alamat IP atau domain server backend Anda.

2. Tambahkan referensi ke file `network_security_config.xml` di `AndroidManifest.xml`:

```xml
<application
    ...
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
    ...
</application>
```

Dengan melakukan ini, aplikasi Android Anda akan diizinkan untuk melakukan komunikasi CLEARTEXT (HTTP) ke alamat IP atau domain server backend Anda. Namun, ingat bahwa ini hanya sebagai solusi sementara dan tidak direkomendasikan untuk aplikasi produksi karena alasan keamanan.
