Pesan error yang Anda sebutkan terdiri dari dua masalah utama: masalah App Check dan masalah komunikasi CLEARTEXT. Berikut adalah cara untuk mengatasi masing-masing masalah tersebut.

### 1. Mengatasi Error App Check

Error "No AppCheckProvider installed" menunjukkan bahwa Firebase App Check tidak diatur dengan benar. App Check membantu melindungi backend Firebase Anda dari penyalahgunaan oleh memastikan bahwa hanya aplikasi asli yang dapat mengakses layanan Firebase Anda.

#### Langkah-langkah untuk Mengonfigurasi App Check:

1. **Tambahkan dependensi App Check di `build.gradle`:**
   ```groovy
   implementation 'com.google.firebase:firebase-appcheck-ktx:16.0.0'
   ```

2. **Inisialisasi App Check di aplikasi Anda:**
   Tambahkan kode berikut di kelas `Application` atau `MainActivity`:

   ```kotlin
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

3. **Perbarui aturan keamanan Firebase:**
   Pastikan aturan keamanan Firebase Anda mengizinkan akses dari aplikasi yang menggunakan App Check.

### 2. Mengatasi Error CLEARTEXT Communication

Error "CLEARTEXT communication not permitted by network security policy" berarti aplikasi Android Anda mencoba menggunakan komunikasi HTTP (tidak aman) alih-alih HTTPS (aman). Untuk mengizinkan komunikasi HTTP (meskipun tidak disarankan untuk produksi), Anda perlu mengatur kebijakan keamanan jaringan Anda.

#### Langkah-langkah untuk Mengizinkan CLEARTEXT Communication:

1. **Buat atau perbarui file `network_security_config.xml` di `res/xml`:**

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
       <domain-config cleartextTrafficPermitted="true">
           <domain includeSubdomains="true">34.128.99.253</domain>
       </domain-config>
   </network-security-config>
   ```

2. **Referensikan konfigurasi keamanan jaringan di `AndroidManifest.xml`:**

   ```xml
   <application
       ...
       android:networkSecurityConfig="@xml/network_security_config"
       ... >
       ...
   </application>
   ```

Dengan kedua langkah di atas, aplikasi Anda seharusnya bisa mengatasi kedua error tersebut. Berikut adalah langkah lengkap untuk implementasinya:

### Full Implementation

#### 1. Tambahkan Dependensi di `build.gradle`:

```groovy
implementation 'com.google.firebase:firebase-appcheck-ktx:16.0.0'
```

#### 2. Inisialisasi App Check:

Di `MyApplication.kt` atau `MainActivity.kt`:

```kotlin
import android.app.Application
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

Di `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    ...
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
    ...
</application>
```

#### 3. Buat `network_security_config.xml`:

Di `res/xml/network_security_config.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">34.128.99.253</domain>
    </domain-config>
</network-security-config>
```

Dengan langkah-langkah ini, masalah App Check dan CLEARTEXT communication harus dapat diselesaikan. Pastikan juga untuk memeriksa dan menguji aplikasi Anda secara menyeluruh setelah melakukan perubahan ini.
