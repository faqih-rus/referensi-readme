
### 1. **Setup Retrofit for Network Calls**
Pertama, pastikan Anda telah menyiapkan Retrofit untuk melakukan permintaan jaringan ke backend Anda.

#### RetrofitInstance.kt
```kotlin
import okhttp3.OkHttpClient
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitInstance {
    private const val BASE_URL = "http://your-backend-url"

    private val client = OkHttpClient.Builder().build()

    val api: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```

#### ApiService.kt
```kotlin
import retrofit2.Call
import retrofit2.http.*

interface ApiService {
    @POST("auth/register")
    fun register(@Body request: RegisterRequest): Call<AuthResponse>

    @POST("auth/login")
    fun login(@Body request: LoginRequest): Call<AuthResponse>

    @POST("auth/logout")
    fun logout(@Header("Authorization") token: String): Call<BaseResponse>

    @GET("nurse/predictions")
    fun getPredictions(@Header("Authorization") token: String): Call<PredictionsResponse>

    @POST("nurse/predictions")
    @Multipart
    fun createPrediction(
        @Header("Authorization") token: String,
        @Part("babyName") babyName: RequestBody,
        @Part("age") age: RequestBody,
        @Part("weight") weight: RequestBody,
        @Part("id") id: RequestBody,
        @Part image: MultipartBody.Part
    ): Call<PredictionResponse>

    @GET("nurse/predictions/{id}")
    fun getPredictionById(
        @Header("Authorization") token: String,
        @Path("id") id: String
    ): Call<PredictionResponse>

    @PUT("nurse/predictions/{id}")
    fun modifyPrediction(
        @Header("Authorization") token: String,
        @Path("id") id: String,
        @Body request: ModifyPredictionRequest
    ): Call<PredictionResponse>

    @DELETE("nurse/predictions/{id}")
    fun deletePrediction(
        @Header("Authorization") token: String,
        @Path("id") id: String
    ): Call<BaseResponse>

    @PUT("nurse/profile")
    @Multipart
    fun updateNurseProfile(
        @Header("Authorization") token: String,
        @Part("name") name: RequestBody?,
        @Part("password") password: RequestBody?,
        @Part profileImage: MultipartBody.Part?
    ): Call<BaseResponse>
}
```

### 2. **Create Request and Response Data Classes**
Anda perlu membuat data class untuk permintaan dan respons API.

#### RegisterRequest.kt
```kotlin
data class RegisterRequest(
    val email: String,
    val password: String,
    val name: String
)
```

#### LoginRequest.kt
```kotlin
data class LoginRequest(
    val email: String,
    val password: String
)
```

#### AuthResponse.kt
```kotlin
data class AuthResponse(
    val status: Int,
    val message: String,
    val idToken: String? = null
)
```

#### PredictionsResponse.kt
```kotlin
data class PredictionsResponse(
    val status: String,
    val data: List<PredictionData>
)
```

#### PredictionResponse.kt
```kotlin
data class PredictionResponse(
    val status: String,
    val data: PredictionData
)
```

#### PredictionData.kt
```kotlin
data class PredictionData(
    val id: String,
    val babyName: String,
    val age: Int,
    val weight: Float,
    val lingkar_kepala: Float,
    val lingkar_dada: Float,
    val lingkar_lengan: Float,
    val lingkar_perut: Float,
    val lingkar_paha: Float,
    val panjang_badan: Float,
    val prediction: String,
    val confidence: Float,
    val suggestion: String,
    val createdAt: String,
    val updatedAt: String
)
```

#### BaseResponse.kt
```kotlin
data class BaseResponse(
    val status: String,
    val message: String
)
```

### 3. **Implement ViewModels for Network Calls**
Implement ViewModel untuk melakukan permintaan jaringan.

#### AuthViewModel.kt
```kotlin
class AuthViewModel : ViewModel() {

    private val _authState = MutableLiveData<AuthState>()
    val authState: LiveData<AuthState> = _authState

    fun register(email: String, password: String, name: String) {
        _authState.value = AuthState.Loading
        val request = RegisterRequest(email, password, name)
        RetrofitInstance.api.register(request).enqueue(object : Callback<AuthResponse> {
            override fun onResponse(call: Call<AuthResponse>, response: Response<AuthResponse>) {
                if (response.isSuccessful) {
                    _authState.value = AuthState.Success
                } else {
                    _authState.value = AuthState.Error("Registration failed")
                }
            }

            override fun onFailure(call: Call<AuthResponse>, t: Throwable) {
                _authState.value = AuthState.Error("Network error")
            }
        })
    }

    fun login(email: String, password: String) {
        _authState.value = AuthState.Loading
        val request = LoginRequest(email, password)
        RetrofitInstance.api.login(request).enqueue(object : Callback<AuthResponse> {
            override fun onResponse(call: Call<AuthResponse>, response: Response<AuthResponse>) {
                if (response.isSuccessful) {
                    val token = response.body()?.idToken
                    // Save token securely
                    _authState.value = AuthState.Success
                } else {
                    _authState.value = AuthState.Error("Login failed")
                }
            }

            override fun onFailure(call: Call<AuthResponse>, t: Throwable) {
                _authState.value = AuthState.Error("Network error")
            }
        })
    }

    fun logout(token: String) {
        RetrofitInstance.api.logout(token).enqueue(object : Callback<BaseResponse> {
            override fun onResponse(call: Call<BaseResponse>, response: Response<BaseResponse>) {
                if (response.isSuccessful) {
                    // Clear saved token
                    _authState.value = AuthState.Success
                } else {
                    _authState.value = AuthState.Error("Logout failed")
                }
            }

            override fun onFailure(call: Call<BaseResponse>, t: Throwable) {
                _authState.value = AuthState.Error("Network error")
            }
        })
    }
}
```

#### PredictionViewModel.kt
```kotlin
class PredictionViewModel : ViewModel() {

    private val _predictions = MutableLiveData<List<PredictionData>>()
    val predictions: LiveData<List<PredictionData>> = _predictions

    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading

    fun loadPredictions(token: String) {
        _loading.value = true
        RetrofitInstance.api.getPredictions(token).enqueue(object : Callback<PredictionsResponse> {
            override fun onResponse(call: Call<PredictionsResponse>, response: Response<PredictionsResponse>) {
                _loading.value = false
                if (response.isSuccessful) {
                    _predictions.value = response.body()?.data ?: emptyList()
                }
            }

            override fun onFailure(call: Call<PredictionsResponse>, t: Throwable) {
                _loading.value = false
            }
        })
    }

    fun createPrediction(token: String, request: CreatePredictionRequest) {
        _loading.value = true
        RetrofitInstance.api.createPrediction(
            token,
            RequestBody.create(MediaType.parse("text/plain"), request.babyName),
            RequestBody.create(MediaType.parse("text/plain"), request.age.toString()),
            RequestBody.create(MediaType.parse("text/plain"), request.weight.toString()),
            RequestBody.create(MediaType.parse("text/plain"), request.id),
            request.image
        ).enqueue(object : Callback<PredictionResponse> {
            override fun onResponse(call: Call<PredictionResponse>, response: Response<PredictionResponse>) {
                _loading.value = false
                if (response.isSuccessful) {
                    // Handle successful creation
                }
            }

            override fun onFailure(call: Call<PredictionResponse>, t: Throwable) {
                _loading.value = false
            }
        })
    }

    fun getPredictionById(token: String, id: String): LiveData<PredictionData?> {
        val prediction = MutableLiveData<PredictionData?>()
        RetrofitInstance.api.getPredictionById(token, id).enqueue(object : Callback<PredictionResponse> {
            override fun onResponse(call: Call<PredictionResponse>, response: Response<PredictionResponse>) {
                if (response.isSuccessful) {
                    prediction.value = response.body()?.data
                }
            }

            override fun onFailure(call: Call<PredictionResponse>, t: Throwable) {
            }
        })
        return prediction
    }

    fun modifyPrediction(token: String, id: String, request: ModifyPredictionRequest) {
        _loading.value = true
        RetrofitInstance.api.modifyPrediction(token, id, request).enqueue(object : Callback<PredictionResponse> {
            override fun onResponse(call: Call<PredictionResponse>, response: Response<PredictionResponse>) {
                _loading.value = false
                if (response.isSuccessful) {
                    // Handle successful modification
                }
            }

            override fun onFailure(call: Call<Prediction

Response>, t: Throwable) {
                _loading.value = false
            }
        })
    }

    fun deletePrediction(token: String, id: String) {
        _loading.value = true
        RetrofitInstance.api.deletePrediction(token, id).enqueue(object : Callback<BaseResponse> {
            override fun onResponse(call: Call<BaseResponse>, response: Response<BaseResponse>) {
                _loading.value = false
                if (response.isSuccessful) {
                    // Handle successful deletion
                }
            }

            override fun onFailure(call: Call<BaseResponse>, t: Throwable) {
                _loading.value = false
            }
        })
    }
}
```

### 4. **Integrate ViewModels with UI**
Gunakan ViewModel dalam Activity atau Fragment Anda untuk mengelola UI dan data.

#### AuthActivity.kt
```kotlin
class AuthActivity : AppCompatActivity() {

    private lateinit var viewModel: AuthViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_auth)

        viewModel = ViewModelProvider(this).get(AuthViewModel::class.java)

        viewModel.authState.observe(this, Observer {
            when (it) {
                is AuthState.Loading -> {
                    // Show loading
                }
                is AuthState.Success -> {
                    // Navigate to next screen
                }
                is AuthState.Error -> {
                    // Show error message
                }
            }
        })

        // Example usage
        // viewModel.register("email", "password", "name")
        // viewModel.login("email", "password")
        // viewModel.logout("token")
    }
}
```

#### PredictionActivity.kt
```kotlin
class PredictionActivity : AppCompatActivity() {

    private lateinit var viewModel: PredictionViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_prediction)

        viewModel = ViewModelProvider(this).get(PredictionViewModel::class.java)

        viewModel.loading.observe(this, Observer { isLoading ->
            // Show/hide loading indicator
        })

        viewModel.predictions.observe(this, Observer { predictions ->
            // Update UI with prediction data
        })

        // Example usage
        // viewModel.loadPredictions("token")
        // viewModel.createPrediction("token", createPredictionRequest)
    }
}
```
