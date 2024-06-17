
1. **Setup Firebase**:
   - Tambahkan Firebase ke proyek Android Anda dengan mengikuti panduan di [dokumentasi Firebase](https://firebase.google.com/docs/android/setup).

2. **Dependencies**:
   - Pastikan Anda telah menambahkan dependensi Firebase di `build.gradle` Anda:
     ```groovy
     implementation 'com.google.firebase:firebase-auth-ktx:21.0.1'
     implementation 'com.google.firebase:firebase-firestore-ktx:24.0.0'
     implementation 'com.google.firebase:firebase-storage-ktx:20.0.0'
     implementation 'com.squareup.okhttp3:okhttp:4.9.3'
     implementation 'com.squareup.okhttp3:logging-interceptor:4.9.3'
     ```

3. **Upload Image to Firebase Storage**:
   - Berikut adalah cara mengunggah gambar ke Firebase Storage dan mendapatkan URL unduhan.

4. **Save Prediction Data to Firestore**:
   - Setelah Anda mendapatkan URL gambar dari Firebase Storage, simpan URL tersebut bersama dengan data prediksi ke Firestore.

Berikut adalah contoh implementasi Kotlin di Android untuk mengunggah gambar, mendapatkan prediksi dari server Flask, dan menyimpan data ke Firestore:

```kotlin
import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.firestore.FirebaseFirestore
import com.google.firebase.storage.FirebaseStorage
import com.google.firebase.storage.UploadTask
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.tasks.await
import okhttp3.*
import org.json.JSONObject
import java.io.File
import java.util.*

class MainActivity : AppCompatActivity() {

    private val auth: FirebaseAuth by lazy { FirebaseAuth.getInstance() }
    private val firestore: FirebaseFirestore by lazy { FirebaseFirestore.getInstance() }
    private val storage: FirebaseStorage by lazy { FirebaseStorage.getInstance() }
    private val client = OkHttpClient()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Dummy image file Uri
        val imageUri: Uri = Uri.fromFile(File("/path/to/your/image.jpg"))

        // Replace with actual baby details and prediction ID
        val babyName = "Baby Name"
        val age = 1
        val weight = 3.5
        val predictionId = "unique_prediction_id"

        // Start the process
        uploadImageAndGetPrediction(imageUri, babyName, age, weight, predictionId)
    }

    private fun uploadImageAndGetPrediction(imageUri: Uri, babyName: String, age: Int, weight: Double, predictionId: String) {
        CoroutineScope(Dispatchers.IO).launch {
            try {
                val userId = auth.currentUser?.uid ?: throw Exception("User not logged in")

                // Upload image to Firebase Storage
                val imageUrl = uploadImageToFirebaseStorage(imageUri, userId)

                // Send image to Flask server and get prediction
                val predictionData = sendImageToFlaskServer(imageUri)

                // Save prediction data to Firestore
                savePredictionDataToFirestore(userId, imageUrl, predictionData, babyName, age, weight, predictionId)
            } catch (e: Exception) {
                Log.e("MainActivity", "Error: ${e.message}")
            }
        }
    }

    private suspend fun uploadImageToFirebaseStorage(imageUri: Uri, userId: String): String {
        val storageRef = storage.reference.child("predictions/$userId/${UUID.randomUUID()}.jpg")
        val uploadTask = storageRef.putFile(imageUri).await()
        return uploadTask.storage.downloadUrl.await().toString()
    }

    private fun sendImageToFlaskServer(imageUri: Uri): JSONObject {
        val file = File(imageUri.path!!)
        val requestBody = MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("image", file.name, RequestBody.create(MediaType.parse("image/jpeg"), file))
            .build()

        val request = Request.Builder()
            .url("http://34.101.242.192:5000/predict")
            .post(requestBody)
            .build()

        val response = client.newCall(request).execute()
        return JSONObject(response.body()?.string() ?: throw Exception("Invalid response from server"))
    }

    private fun savePredictionDataToFirestore(userId: String, imageUrl: String, predictionData: JSONObject, babyName: String, age: Int, weight: Double, predictionId: String) {
        val prediction = predictionData.getString("prediction")
        val suggestion = predictionData.getString("suggestion")
        val confidence = predictionData.getDouble("confidence")

        val predictionMap = hashMapOf(
            "id" to predictionId,
            "babyName" to babyName,
            "age" to age,
            "weight" to weight,
            "prediction" to prediction,
            "suggestion" to suggestion,
            "confidence" to confidence,
            "imageUrl" to imageUrl,
            "createdAt" to System.currentTimeMillis(),
            "updatedAt" to System.currentTimeMillis()
        )

        firestore.collection("predictions").document(userId).collection("data").document(predictionId).set(predictionMap)
            .addOnSuccessListener {
                Log.d("MainActivity", "Prediction data saved successfully")
            }
            .addOnFailureListener { e ->
                Log.e("MainActivity", "Error saving prediction data: ${e.message}")
            }
    }
}
```

### Penjelasan Kode:

1. **Upload Image**:
   - `uploadImageToFirebaseStorage` mengunggah gambar ke Firebase Storage dan mengembalikan URL gambar yang diunggah.

2. **Send Image to Flask Server**:
   - `sendImageToFlaskServer` mengirim gambar ke server Flask dan mengembalikan hasil prediksi dalam bentuk JSON.

3. **Save Prediction Data to Firestore**:
   - `savePredictionDataToFirestore` menyimpan data prediksi, termasuk URL gambar, ke Firebase Firestore.

### Perlu Diingat:
- Pastikan untuk menyesuaikan `imageUri` dengan jalur gambar yang benar.
- Pastikan Anda telah mengonfigurasi Firebase Authentication, Storage, dan Firestore dengan benar di proyek Anda.
- Pastikan URL server Flask (`http://34.101.242.192:5000/predict`)

telah diatur dan dapat diakses dari aplikasi Android.

Berikut adalah langkah-langkah rinci untuk implementasi:

### 1. Konfigurasi Firebase di Proyek Anda

Pastikan Anda telah mengonfigurasi Firebase di proyek Android Anda dengan mengikuti langkah-langkah di [Firebase Documentation](https://firebase.google.com/docs/android/setup). Tambahkan dependensi berikut ke `build.gradle`:

```groovy
implementation 'com.google.firebase:firebase-auth-ktx:21.0.1'
implementation 'com.google.firebase:firebase-firestore-ktx:24.0.0'
implementation 'com.google.firebase:firebase-storage-ktx:20.0.0'
implementation 'com.squareup.okhttp3:okhttp:4.9.3'
implementation 'com.squareup.okhttp3:logging-interceptor:4.9.3'
```

### 2. Mengunggah Gambar ke Firebase Storage

Untuk mengunggah gambar ke Firebase Storage, gunakan fungsi berikut:

```kotlin
private suspend fun uploadImageToFirebaseStorage(imageUri: Uri, userId: String): String {
    val storageRef = storage.reference.child("predictions/$userId/${UUID.randomUUID()}.jpg")
    val uploadTask = storageRef.putFile(imageUri).await()
    return uploadTask.storage.downloadUrl.await().toString()
}
```

### 3. Mengirim Gambar ke Server Flask dan Mendapatkan Prediksi

Gunakan OkHttp untuk mengirim gambar ke server Flask dan mendapatkan prediksi:

```kotlin
private fun sendImageToFlaskServer(imageUri: Uri): JSONObject {
    val file = File(imageUri.path!!)
    val requestBody = MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("image", file.name, RequestBody.create(MediaType.parse("image/jpeg"), file))
        .build()

    val request = Request.Builder()
        .url("http://34.101.242.192:5000/predict")
        .post(requestBody)
        .build()

    val response = client.newCall(request).execute()
    return JSONObject(response.body()?.string() ?: throw Exception("Invalid response from server"))
}
```

### 4. Menyimpan Data Prediksi ke Firestore

Setelah mendapatkan URL gambar dan data prediksi, simpan data tersebut ke Firestore:

```kotlin
private fun savePredictionDataToFirestore(userId: String, imageUrl: String, predictionData: JSONObject, babyName: String, age: Int, weight: Double, predictionId: String) {
    val prediction = predictionData.getString("prediction")
    val suggestion = predictionData.getString("suggestion")
    val confidence = predictionData.getDouble("confidence")

    val predictionMap = hashMapOf(
        "id" to predictionId,
        "babyName" to babyName,
        "age" to age,
        "weight" to weight,
        "prediction" to prediction,
        "suggestion" to suggestion,
        "confidence" to confidence,
        "imageUrl" to imageUrl,
        "createdAt" to System.currentTimeMillis(),
        "updatedAt" to System.currentTimeMillis()
    )

    firestore.collection("predictions").document(userId).collection("data").document(predictionId).set(predictionMap)
        .addOnSuccessListener {
            Log.d("MainActivity", "Prediction data saved successfully")
        }
        .addOnFailureListener { e ->
            Log.e("MainActivity", "Error saving prediction data: ${e.message}")
        }
}
```

### 5. Memulai Proses dari MainActivity

Gabungkan semua fungsi di atas di `MainActivity`:

```kotlin
class MainActivity : AppCompatActivity() {

    private val auth: FirebaseAuth by lazy { FirebaseAuth.getInstance() }
    private val firestore: FirebaseFirestore by lazy { FirebaseFirestore.getInstance() }
    private val storage: FirebaseStorage by lazy { FirebaseStorage.getInstance() }
    private val client = OkHttpClient()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Dummy image file Uri
        val imageUri: Uri = Uri.fromFile(File("/path/to/your/image.jpg"))

        // Replace with actual baby details and prediction ID
        val babyName = "Baby Name"
        val age = 1
        val weight = 3.5
        val predictionId = "unique_prediction_id"

        // Start the process
        uploadImageAndGetPrediction(imageUri, babyName, age, weight, predictionId)
    }

    private fun uploadImageAndGetPrediction(imageUri: Uri, babyName: String, age: Int, weight: Double, predictionId: String) {
        CoroutineScope(Dispatchers.IO).launch {
            try {
                val userId = auth.currentUser?.uid ?: throw Exception("User not logged in")

                // Upload image to Firebase Storage
                val imageUrl = uploadImageToFirebaseStorage(imageUri, userId)

                // Send image to Flask server and get prediction
                val predictionData = sendImageToFlaskServer(imageUri)

                // Save prediction data to Firestore
                savePredictionDataToFirestore(userId, imageUrl, predictionData, babyName, age, weight, predictionId)
            } catch (e: Exception) {
                Log.e("MainActivity", "Error: ${e.message}")
            }
        }
    }
}
```
