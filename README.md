Great! Based on your input, we'll design the Android application to communicate with the Hapi.js backend deployed at `http://34.128.99.253/`. We'll use Retrofit for network requests and Firebase for authentication management on the client side.

Here is the outline of the application architecture and implementation plan.

= SPEC-001: Kotlin Android Application for Nurse Predictions
:sectnums:
:toc:


== Background

The purpose of this application is to provide an interface for nurses to manage predictions for infant health based on various input metrics. The backend is implemented using Hapi.js and Firebase, and the Android client will interact with this backend to perform user authentication and CRUD operations for predictions.

== Requirements

- **User Authentication**:
  - Must have the ability to register, login, and logout.
  - Must handle and display appropriate error messages.

- **Prediction Management**:
  - Must allow the nurse to create new predictions.
  - Must display a list of predictions for the authenticated user.
  - Must allow modifications to existing predictions.
  - Must support profile updates for the nurse.
  - Must allow retrieval of a specific prediction by ID.
  - Must support deletion of predictions.

== Method

### Architecture

1. **Network Layer**:
   - Use Retrofit for API calls.
   - Use Firebase Authentication for managing user sessions.

2. **UI Layer**:
   - Use Jetpack Compose for building the UI components.
   - Provide screens for registration, login, predictions list, prediction details, and profile management.

3. **Data Layer**:
   - Use Repository pattern to manage data operations.
   - Use ViewModel to handle UI-related data in a lifecycle-conscious way.

### Retrofit Configuration

Define the API endpoints corresponding to the backend routes.

```kotlin
interface ApiService {
    @POST("/auth/register")
    suspend fun register(@Body request: RegisterRequest): Response<AuthResponse>

    @POST("/auth/login")
    suspend fun login(@Body request: LoginRequest): Response<AuthResponse>

    @POST("/auth/logout")
    suspend fun logout(@Header("Authorization") token: String): Response<LogoutResponse>

    @POST("/nurse/predictions")
    suspend fun createPrediction(
        @Header("Authorization") token: String,
        @PartMap data: Map<String, @JvmSuppressWildcards RequestBody>,
        @Part image: MultipartBody.Part
    ): Response<PredictionResponse>

    @GET("/nurse/predictions")
    suspend fun getPredictions(@Header("Authorization") token: String): Response<PredictionsResponse>

    @PUT("/nurse/predictions/{id}")
    suspend fun modifyPrediction(
        @Header("Authorization") token: String,
        @Path("id") id: String,
        @Body request: ModifyPredictionRequest
    ): Response<PredictionResponse>

    @PUT("/nurse/profile")
    suspend fun updateProfile(
        @Header("Authorization") token: String,
        @PartMap data: Map<String, @JvmSuppressWildcards RequestBody>,
        @Part profileImage: MultipartBody.Part?
    ): Response<ProfileResponse>

    @GET("/nurse/predictions/{id}")
    suspend fun getPredictionById(
        @Header("Authorization") token: String,
        @Path("id") id: String
    ): Response<PredictionResponse>

    @DELETE("/nurse/predictions/{id}")
    suspend fun deletePrediction(
        @Header("Authorization") token: String,
        @Path("id") id: String
    ): Response<DeletePredictionResponse>
}
```

### Data Models

Define data models for the API requests and responses.

```kotlin
data class RegisterRequest(val email: String, val password: String, val name: String)
data class LoginRequest(val email: String, val password: String)
data class AuthResponse(val status: Int, val message: String, val idToken: String?)
data class LogoutResponse(val status: Int, val message: String)
data class PredictionResponse(val status: String, val data: PredictionData?)
data class PredictionsResponse(val status: String, val data: List<PredictionData>?)
data class ModifyPredictionRequest(val babyName: String?, val weight: Double?, val id: String?)
data class ProfileResponse(val status: String, val data: UserProfile?)
data class DeletePredictionResponse(val status: String, val message: String)

data class PredictionData(
    val id: String,
    val babyName: String,
    val age: Int,
    val weight: Double,
    val lingkar_kepala: Double,
    val lingkar_dada: Double,
    val lingkar_lengan: Double,
    val lingkar_perut: Double,
    val lingkar_paha: Double,
    val panjang_badan: Double,
    val prediction: String,
    val confidence: Double,
    val suggestion: String,
    val createdAt: String,
    val updatedAt: String
)

data class UserProfile(val name: String, val profileImageUrl: String)
```

### Retrofit Instance

Create a Retrofit instance to handle API requests.

```kotlin
object RetrofitInstance {
    private val retrofit by lazy {
        Retrofit.Builder()
            .baseUrl("http://34.128.99.253/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    val api: ApiService by lazy {
        retrofit.create(ApiService::class.java)
    }
}
```

### Firebase Authentication

Initialize Firebase Authentication and provide methods for login, logout, and registration.

```kotlin
object FirebaseAuthHelper {
    private val auth: FirebaseAuth = Firebase.auth

    fun register(email: String, password: String, callback: (result: AuthResult?, error: Exception?) -> Unit) {
        auth.createUserWithEmailAndPassword(email, password)
            .addOnCompleteListener { task ->
                if (task.isSuccessful) {
                    callback(task.result, null)
                } else {
                    callback(null, task.exception)
                }
            }
    }

    fun login(email: String, password: String, callback: (result: AuthResult?, error: Exception?) -> Unit) {
        auth.signInWithEmailAndPassword(email, password)
            .addOnCompleteListener { task ->
                if (task.isSuccessful) {
                    callback(task.result, null)
                } else {
                    callback(null, task.exception)
                }
            }
    }

    fun logout() {
        auth.signOut()
    }
}
```

### UI Components

Use Jetpack Compose to build the UI components for the application.

```kotlin
@Composable
fun LoginScreen(viewModel: AuthViewModel) {
    val email = remember { mutableStateOf("") }
    val password = remember { mutableStateOf("") }
    val authState by viewModel.authState.collectAsState()

    Column(modifier = Modifier.padding(16.dp)) {
        TextField(value = email.value, onValueChange = { email.value = it }, label = { Text("Email") })
        TextField(value = password.value, onValueChange = { password.value = it }, label = { Text("Password") })
        Button(onClick = { viewModel.login(email.value, password.value) }) {
            Text("Login")
        }

        when (authState) {
            is AuthState.Loading -> CircularProgressIndicator()
            is AuthState.Success -> Text("Login successful")
            is AuthState.Error -> Text("Error: ${(authState as AuthState.Error).message}")
        }
    }
}
```

### ViewModel

Create a ViewModel to handle authentication logic and state management.

```kotlin
class AuthViewModel(private val authHelper: FirebaseAuthHelper) : ViewModel() {

    private val _authState = MutableStateFlow<AuthState>(AuthState.Idle)
    val authState: StateFlow<AuthState> = _authState

    fun login(email: String, password: String) {
        _authState.value = AuthState.Loading
        authHelper.login(email, password) { result, error ->
            if (error != null) {
                _authState.value = AuthState.Error(error.message ?: "Unknown error")
            } else {
                _authState.value = AuthState.Success
            }
        }
    }
}

sealed class AuthState {
    object Idle : AuthState()
    object Loading : AuthState()
    object Success : AuthState()
    data class Error(val message: String) : AuthState()
}
```

== Implementation

### Steps to Implement

1. **Setup Project**:
   - Create a new Kotlin project in Android Studio.
   - Add necessary dependencies (Retrofit, Firebase, Jetpack Compose, etc.) in `build.gradle`.

2. **Firebase Configuration**:
   - Add Firebase to the Android project and configure `google-services.json`.

3. **Network Layer**:
   - Implement Retrofit instance and API service interface.
   - Implement data models for API requests and responses.

4. **Authentication**:
   - Implement Firebase authentication helper.
   - Implement ViewModel for authentication.
   - Create UI screens for registration, login, and logout.

5. **Prediction Management**:
   - Implement ViewModel and Repository for prediction data.
   - Create UI screens for creating, viewing, modifying, and deleting predictions.
   - Implement file upload for image prediction.

6. **Profile Management**:
   - Implement ViewModel and Repository for profile data.
   - Create UI screens for updating profile information.

7. **Testing and Debugging**:
   - Test each feature thoroughly.
   - Debug and fix any issues.

== Milestones

1. **Project Setup and Configuration** - 1 week
2. **Network Layer and Data Models** - 1 week
3. **Authentication Implementation** - 2 weeks
4. **Prediction Management Implementation** - 3 weeks
5. **Profile Management Implementation** - 2 weeks
6. **Testing and Debugging** - 2

 weeks

== Gathering Results

Evaluate the implementation based on the following criteria:

1. **Functionality**:
   - Ensure all user stories are implemented and functional.
   - Test edge cases and error handling.

2. **Performance**:
   - Measure the app's performance, including API response times and UI responsiveness.

3. **User Feedback**:
   - Gather feedback from nurses using the app and make necessary improvements.

With this plan, you can proceed to implement the Kotlin Android application that interacts with the provided Hapi.js backend. If you have any further questions or need specific code examples for any part, feel free to ask!
