Berikut adalah penjelasan dan kode lengkap untuk membuat fitur sejarah prediksi dan menampilkan detail prediksi di aplikasi Anda.

### 1. Membuat Layout Card dan RecyclerView untuk HistoryFragment

Pertama, buat layout card untuk setiap item sejarah di dalam `res/layout/item_history.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <ImageView
        android:id="@+id/iv_history_image"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:scaleType="centerCrop" />

    <TextView
        android:id="@+id/tv_history_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:paddingTop="8dp" />

    <TextView
        android:id="@+id/tv_history_nik"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="14sp"
        android:paddingTop="4dp" />

</LinearLayout>
```

Lalu, buat layout untuk `HistoryFragment` di `res/layout/fragment_history.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv_History"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

### 2. Membuat Model dan Adapter untuk RecyclerView

Buat kelas `HistoryItem`:

```kotlin
package com.capstone.babymeter.history

import java.io.Serializable

data class HistoryItem(
    val name: String,
    val nik: String,
    val imageUrl: String
) : Serializable
```

Buat kelas `HistoryAdapter`:

```kotlin
package com.capstone.babymeter.history

import android.content.Context
import android.content.Intent
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.capstone.babymeter.R

class HistoryAdapter(private val historyList: List<HistoryItem>, private val context: Context) :
    RecyclerView.Adapter<HistoryAdapter.HistoryViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): HistoryViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_history, parent, false)
        return HistoryViewHolder(view)
    }

    override fun onBindViewHolder(holder: HistoryViewHolder, position: Int) {
        val historyItem = historyList[position]
        holder.bind(historyItem)

        holder.itemView.setOnClickListener {
            val intent = Intent(context, DetailsActivity::class.java).apply {
                putExtra("nik", historyItem.nik)
            }
            context.startActivity(intent)
        }
    }

    override fun getItemCount() = historyList.size

    class HistoryViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val imageView: ImageView = itemView.findViewById(R.id.iv_history_image)
        private val nameTextView: TextView = itemView.findViewById(R.id.tv_history_name)
        private val nikTextView: TextView = itemView.findViewById(R.id.tv_history_nik)

        fun bind(historyItem: HistoryItem) {
            nameTextView.text = historyItem.name
            nikTextView.text = historyItem.nik
            Glide.with(itemView.context).load(historyItem.imageUrl).into(imageView)
        }
    }
}
```

### 3. Mengupdate `HistoryFragment` untuk Mengambil Data dari Backend

Update `HistoryFragment` untuk mengambil data dari backend menggunakan Retrofit atau OkHttp dan mengatur RecyclerView:

```kotlin
package com.capstone.babymeter.history

import android.content.Context
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.capstone.babymeter.R
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.firestore.FirebaseFirestore
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.tasks.await

class HistoryFragment : Fragment() {

    private lateinit var rvHistory: RecyclerView
    private lateinit var historyAdapter: HistoryAdapter
    private val historyList = mutableListOf<HistoryItem>()
    private lateinit var auth: FirebaseAuth
    private val db = FirebaseFirestore.getInstance()

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val rootView = inflater.inflate(R.layout.fragment_history, container, false)

        auth = FirebaseAuth.getInstance()
        rvHistory = rootView.findViewById(R.id.rv_History)
        rvHistory.layoutManager = LinearLayoutManager(requireContext())

        historyAdapter = HistoryAdapter(historyList, requireContext())
        rvHistory.adapter = historyAdapter

        fetchHistoryData()

        return rootView
    }

    private fun fetchHistoryData() {
        val user = auth.currentUser
        user?.getIdToken(true)?.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                val idToken = task.result?.token
                idToken?.let { token ->
                    CoroutineScope(Dispatchers.IO).launch {
                        try {
                            val documents = db.collection("predictions").get().await()
                            for (document in documents) {
                                val name = document.getString("babyName") ?: "N/A"
                                val nik = document.getString("nik") ?: "N/A"
                                val imageUrl = document.getString("imageUrl") ?: "N/A"
                                historyList.add(HistoryItem(name, nik, imageUrl))
                            }
                            CoroutineScope(Dispatchers.Main).launch {
                                historyAdapter.notifyDataSetChanged()
                            }
                        } catch (e: Exception) {
                            e.printStackTrace()
                        }
                    }
                }
            }
        }
    }
}
```

### 4. Mengupdate `DetailsActivity` untuk Menampilkan Data dari Backend

Update `DetailsActivity` untuk menampilkan data detail prediksi:

```kotlin
package com.capstone.babymeter.history

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import android.widget.ImageView
import android.widget.TextView
import com.bumptech.glide.Glide
import com.capstone.babymeter.R
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.firestore.FirebaseFirestore
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.tasks.await

class DetailsActivity : AppCompatActivity() {

    private lateinit var auth: FirebaseAuth
    private val db = FirebaseFirestore.getInstance()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_details)

        auth = FirebaseAuth.getInstance()

        val nik = intent.getStringExtra("nik") ?: ""

        fetchDetailData(nik)
    }

    private fun fetchDetailData(nik: String) {
        CoroutineScope(Dispatchers.IO).launch {
            try {
                val document = db.collection("predictions").document(nik).get().await()
                if (document != null && document.exists()) {
                    val babyName = document.getString("babyName") ?: "N/A"
                    val age = document.getLong("age")?.toInt() ?: 0
                    val weight = document.getDouble("weight") ?: Double.NaN
                    val chestSize = document.getDouble("lingkar_dada") ?: Double.NaN
                    val headCircumference = document.getDouble("lingkar_kepala") ?: Double.NaN
                    val armCircumference = document.getDouble("lingkar_lengan") ?: Double.NaN
                    val thighCircumference = document.getDouble("lingkar_paha") ?: Double.NaN
                    val abdominalCircumference = document.getDouble("lingkar_perut") ?: Double.NaN
                    val height = document.getDouble("panjang_badan") ?: Double.NaN
                    val category = document.getString("prediction") ?: "N/A"
                    val suggestion = document.getString("suggestion") ?: "N/A"
                    val imageUrl = document.getString("imageUrl") ?: "N/A"

                    CoroutineScope(Dispatchers.Main).launch {
                        findViewById<TextView>(R.id.tv_detail_name).text = babyName
                        findViewById<TextView>(R.id.NIK).text = nik
                        findViewById<TextView>(R.id.Beratbayi).text = weight.toString()
                        findViewById<TextView>(R.id.lingkarkepala).text = headCircumference.toString()
                        findViewById<TextView>(R.id.lingkarbadan).text = chestSize.toString()
                        findViewById<TextView>(R.id.lingkarkaki).text = thighCircumference.toString()
                        findViewById<TextView>(R.id.ting

gibadan).text = height.toString()
                        findViewById<TextView>(R.id.bmi).text = abdominalCircumference.toString()
                        findViewById<TextView>(R.id.categori).text = category
                        Glide.with(this@DetailsActivity).load(imageUrl).into(findViewById(R.id.iv_detail_photo))
                    }
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
}
```

### Penjelasan:

1. **HistoryFragment**:
   - Mengambil data prediksi dari Firestore dan menampilkannya dalam RecyclerView.
   - Setiap item dalam RecyclerView berupa card yang menampilkan `babyName`, `nik`, dan `imageUrl`.

2. **DetailsActivity**:
   - Menampilkan detail prediksi berdasarkan `nik` yang dikirim dari HistoryFragment.
   - Menggunakan Glide untuk memuat gambar.

Dengan implementasi ini, Anda dapat menampilkan daftar sejarah prediksi dan menampilkan detail dari setiap prediksi saat item dipilih.
