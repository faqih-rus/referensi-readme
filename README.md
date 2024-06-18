Dari log yang Anda berikan, terlihat ada dua jenis error yang perlu diatasi:

1. **Error Firebase App Check**
```
Error getting App Check token; using placeholder token instead. Error: com.google.firebase.FirebaseException: No AppCheckProvider installed.
```

Error ini masih muncul, yang menunjukkan bahwa Firebase App Check belum diinisialisasi dengan benar di aplikasi Android Anda.

2. **Error Kebijakan Keamanan Jaringan**
```
Error: CLEARTEXT communication to 34.128.99.253 not permitted by network security policy
```

Walaupun Anda sudah mengonfigurasi kebijakan keamanan jaringan, sepertinya masih ada masalah yang menyebabkan error ini.

Untuk mengatasi masalah ini, saya sarankan Anda melakukan langkah-langkah berikut:

### 1. Inisialisasi Firebase App Check

Pastikan Anda telah menginisialisasi Firebase App Check dengan benar di aplikasi Android Anda. Anda dapat melakukannya dengan menambahkan kode berikut di kelas `Application` Anda (misalnya `MyApplication.kt`):

```kotlin
import com.google.firebase.FirebaseApp
import com.google.firebase.appcheck.FirebaseAppCheck
import com.google.firebase.appcheck.playintegrity.PlayIntegrityAppCheckProviderFactory

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        FirebaseApp.initializeApp(this)
        val firebaseAppCheck = FirebaseAppCheck.getInstance()
        firebaseAppCheck.installAppCheckProviderFactory(
            PlayIntegrityAppCheckProviderFactory.getInstance()
        )
    }
}
```

Jangan lupa untuk menambahkan referensi ke kelas `Application` Anda di `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    ...>
    ...
</application>
```

### 2. Periksa Konfigurasi Kebijakan Keamanan Jaringan

Pastikan Anda telah mengonfigurasi kebijakan keamanan jaringan dengan benar di `res/xml/network_security_config.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">34.128.99.253</domain>
    </domain-config>
</network-security-config>
```

Dan juga pastikan Anda telah menambahkan referensi ke file `network_security_config.xml` di `AndroidManifest.xml`:

```xml
<application
    ...
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
    ...
</application>
```

### 3. Periksa Kode Sumber dan Dependensi

Periksa kode sumber aplikasi Android Anda dan dependensi yang digunakan. Pastikan tidak ada kode atau dependensi lain yang menyebabkan konflik dengan konfigurasi yang Anda lakukan.

### 4. Bersihkan dan Rebuild Proyek

Coba bersihkan dan rebuild proyek Android Anda. Terkadang, kesalahan cache atau build dapat menyebabkan masalah seperti ini.

Jika Anda telah melakukan semua langkah di atas dan error masih berlanjut, saya sarankan Anda memeriksa logcat untuk informasi error tambahan yang dapat membantu Anda mengatasi masalah ini. Anda juga dapat mencoba membuat proyek baru untuk melihat apakah masalah ini spesifik untuk proyek Anda atau tidak.

Jika masalah masih berlanjut setelah melakukan semua langkah di atas, silakan berikan informasi lebih lanjut seperti kode sumber yang terkait atau detail tambahan tentang proyek Anda, sehingga saya dapat membantu dengan lebih baik.
