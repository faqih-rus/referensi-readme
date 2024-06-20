### ResultFragment
Tambahkan fragment ini ke project Anda, pastikan layout XML sesuai dengan kebutuhan Anda.

```java
package com.capstone.babymeter.fragments;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.fragment.app.Fragment;
import com.capstone.babymeter.R;
import org.json.JSONException;
import org.json.JSONObject;

public class ResultFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_result, container, false);

        // Get the prediction result from arguments
        String predictionResult = getArguments().getString("predictionResult");

        try {
            // Parse the prediction result JSON
            JSONObject jsonObject = new JSONObject(predictionResult);
            JSONObject data = jsonObject.getJSONObject("data");

            String nik = data.getString("nik");
            String babyName = data.getString("babyName");
            int age = data.getInt("age");
            float weight = data.getFloat("weight");
            String imageUrl = data.getString("imageUrl");
            float lingkarDada = (float) data.getDouble("lingkar_dada");
            float lingkarKepala = (float) data.getDouble("lingkar_kepala");
            float lingkarLengan = (float) data.getDouble("lingkar_lengan");
            float lingkarPaha = (float) data.getDouble("lingkar_paha");
            float lingkarPerut = (float) data.getDouble("lingkar_perut");
            float panjangBadan = (float) data.getDouble("panjang_badan");
            String prediction = data.getString("prediction");
            double confidence = data.getDouble("confidence");
            String suggestion = data.getString("suggestion");

            // Set data to TextViews
            ((TextView) view.findViewById(R.id.nikTextView)).setText(nik);
            ((TextView) view.findViewById(R.id.babyNameTextView)).setText(babyName);
            ((TextView) view.findViewById(R.id.ageTextView)).setText(String.valueOf(age));
            ((TextView) view.findViewById(R.id.weightTextView)).setText(String.valueOf(weight));
            ((TextView) view.findViewById(R.id.lingkarDadaTextView)).setText(String.valueOf(lingkarDada));
            ((TextView) view.findViewById(R.id.lingkarKepalaTextView)).setText(String.valueOf(lingkarKepala));
            ((TextView) view.findViewById(R.id.lingkarLenganTextView)).setText(String.valueOf(lingkarLengan));
            ((TextView) view.findViewById(R.id.lingkarPahaTextView)).setText(String.valueOf(lingkarPaha));
            ((TextView) view.findViewById(R.id.lingkarPerutTextView)).setText(String.valueOf(lingkarPerut));
            ((TextView) view.findViewById(R.id.panjangBadanTextView)).setText(String.valueOf(panjangBadan));
            ((TextView) view.findViewById(R.id.predictionTextView)).setText(prediction);
            ((TextView) view.findViewById(R.id.confidenceTextView)).setText(String.valueOf(confidence));
            ((TextView) view.findViewById(R.id.suggestionTextView)).setText(suggestion);

        } catch (JSONException e) {
            e.printStackTrace();
        }

        return view;
    }
}
```

### Layout untuk ResultFragment (res/layout/fragment_result.xml)
Buat layout XML untuk `ResultFragment` yang sesuai dengan tampilan yang Anda inginkan. Berikut contoh dasar layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/nikTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="NIK" />

    <TextView
        android:id="@+id/babyNameTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Baby Name" />

    <TextView
        android:id="@+id/ageTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Age" />

    <TextView
        android:id="@+id/weightTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Weight" />

    <TextView
        android:id="@+id/lingkarDadaTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Lingkar Dada" />

    <TextView
        android:id="@+id/lingkarKepalaTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Lingkar Kepala" />

    <TextView
        android:id="@+id/lingkarLenganTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Lingkar Lengan" />

    <TextView
        android:id="@+id/lingkarPahaTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Lingkar Paha" />

    <TextView
        android:id="@+id/lingkarPerutTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Lingkar Perut" />

    <TextView
        android:id="@+id/panjangBadanTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Panjang Badan" />

    <TextView
        android:id="@+id/predictionTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Prediction" />

    <TextView
        android:id="@+id/confidenceTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Confidence" />

    <TextView
        android:id="@+id/suggestionTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Suggestion" />
</LinearLayout>
```

### Menavigasi ke ResultFragment
Pastikan Anda menambahkan `ResultFragment` ke transaksi fragment seperti yang telah Anda lakukan dalam kode:

```java
// Replace the current fragment with the ResultFragment
parentFragmentManager.beginTransaction().apply {
    replace(R.id.fragmentContainer, resultFragment)
    addToBackStack(null)
    commit()
}
```

Dengan kode di atas, setelah backend memberikan respons sukses, data dari JSON akan diurai dan ditampilkan dalam `ResultFragment`. Pastikan Anda memiliki layout `fragment_result.xml` yang sesuai dengan TextView ID yang digunakan dalam kode ini.
