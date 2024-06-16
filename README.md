
### 1. **LoginActivity**
#### Layout XML (res/layout/activity_login.xml)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/email"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Email"
        android:inputType="textEmailAddress" />

    <EditText
        android:id="@+id/password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Password"
        android:inputType="textPassword" />

    <Button
        android:id="@+id/loginButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Login" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone" />
</LinearLayout>
```

#### Activity Class (LoginActivity.kt)
```kotlin
class LoginActivity : AppCompatActivity() {

    private lateinit var viewModel: AuthViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        viewModel = ViewModelProvider(this).get(AuthViewModel::class.java)

        val emailEditText = findViewById<EditText>(R.id.email)
        val passwordEditText = findViewById<EditText>(R.id.password)
        val loginButton = findViewById<Button>(R.id.loginButton)
        val progressBar = findViewById<ProgressBar>(R.id.progressBar)

        viewModel.authState.observe(this, { state ->
            when (state) {
                is AuthState.Loading -> progressBar.visibility = View.VISIBLE
                is AuthState.Success -> {
                    progressBar.visibility = View.GONE
                    startActivity(Intent(this, MainActivity::class.java))
                    finish()
                }
                is AuthState.Error -> {
                    progressBar.visibility = View.GONE
                    Toast.makeText(this, state.message, Toast.LENGTH_LONG).show()
                }
            }
        })

        loginButton.setOnClickListener {
            val email = emailEditText.text.toString()
            val password = passwordEditText.text.toString()
            viewModel.login(email, password)
        }
    }
}
```

### 2. **MainActivity**
MainActivity akan menjadi tempat utama untuk navigasi antara berbagai layar seperti daftar prediksi dan profil.

#### Layout XML (res/layout/activity_main.xml)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

#### Activity Class (MainActivity.kt)
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, PredictionsListFragment())
                .commit()
        }
    }
}
```

### 3. **PredictionsListFragment**
#### Layout XML (res/layout/fragment_predictions_list.xml)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <ListView
        android:id="@+id/predictions_list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone" />
</LinearLayout>
```

#### Fragment Class (PredictionsListFragment.kt)
```kotlin
class PredictionsListFragment : Fragment() {

    private lateinit var viewModel: PredictionViewModel
    private lateinit var listView: ListView
    private lateinit var progressBar: ProgressBar

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val view = inflater.inflate(R.layout.fragment_predictions_list, container, false)
        listView = view.findViewById(R.id.predictions_list)
        progressBar = view.findViewById(R.id.progressBar)
        return view
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel = ViewModelProvider(this).get(PredictionViewModel::class.java)

        viewModel.predictions.observe(viewLifecycleOwner, { predictions ->
            progressBar.visibility = View.GONE
            val adapter = ArrayAdapter(requireContext(), android.R.layout.simple_list_item_1, predictions.map { it.babyName })
            listView.adapter = adapter
        })

        viewModel.loading.observe(viewLifecycleOwner, { isLoading ->
            progressBar.visibility = if (isLoading) View.VISIBLE else View.GONE
        })

        listView.setOnItemClickListener { _, _, position, _ ->
            val prediction = viewModel.predictions.value?.get(position)
            val intent = Intent(activity, PredictionDetailActivity::class.java)
            intent.putExtra("PREDICTION_ID", prediction?.id)
            startActivity(intent)
        }

        viewModel.loadPredictions()
    }
}
```

### 4. **PredictionDetailActivity**
#### Layout XML (res/layout/activity_prediction_detail.xml)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/babyName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Baby Name" />

    <TextView
        android:id="@+id/prediction"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Prediction" />

    <!-- Add other prediction details here -->
</LinearLayout>
```

#### Activity Class (PredictionDetailActivity.kt)
```kotlin
class PredictionDetailActivity : AppCompatActivity() {

    private lateinit var viewModel: PredictionViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_prediction_detail)

        viewModel = ViewModelProvider(this).get(PredictionViewModel::class.java)

        val babyNameTextView = findViewById<TextView>(R.id.babyName)
        val predictionTextView = findViewById<TextView>(R.id.prediction)

        val predictionId = intent.getStringExtra("PREDICTION_ID")
        viewModel.getPredictionById(predictionId).observe(this, { prediction ->
            prediction?.let {
                babyNameTextView.text = it.babyName
                predictionTextView.text = it.prediction
                // Set other prediction details
            }
        })
    }
}
```

### 5. **ViewModel Implementation**

#### AuthViewModel (AuthViewModel.kt)
```kotlin
class AuthViewModel : ViewModel() {

    private val _authState = MutableLiveData<AuthState>()
    val authState: LiveData<AuthState> = _authState

    fun login(email: String, password: String) {
        _authState.value = AuthState.Loading
        FirebaseAuthHelper.login(email, password) { result, error ->
            if (error != null) {
                _authState.value = AuthState.Error(error.message ?: "Unknown error")
            } else {
                _authState.value = AuthState.Success
            }
        }
    }
}
```

#### PredictionViewModel (PredictionViewModel.kt)
```kotlin
class PredictionViewModel : ViewModel() {

    private val _predictions = MutableLiveData<List<PredictionData>>()
    val predictions: LiveData<List<PredictionData>> = _predictions

    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading

    fun loadPredictions() {
        _loading.value = true
        RetrofitInstance.api.getPredictions(/* Auth Token */).enqueue(object : Callback<PredictionsResponse> {
            override fun onResponse(call: Call<PredictionsResponse>, response: Response<PredictionsResponse>) {
                _loading.value = false
                if (response.isSuccessful) {
                    _predictions.value = response.body()?.data ?:

 emptyList()
                }
            }

            override fun onFailure(call: Call<PredictionsResponse>, t: Throwable) {
                _loading.value = false
            }
        })
    }

    fun getPredictionById(id: String?): LiveData<PredictionData?> {
        val prediction = MutableLiveData<PredictionData?>()
        id?.let {
            RetrofitInstance.api.getPredictionById(/* Auth Token */, it).enqueue(object : Callback<PredictionResponse> {
                override fun onResponse(call: Call<PredictionResponse>, response: Response<PredictionResponse>) {
                    if (response.isSuccessful) {
                        prediction.value = response.body()?.data
                    }
                }

                override fun onFailure(call: Call<PredictionResponse>, t: Throwable) {
                }
            })
        }
        return prediction
    }
}
```
