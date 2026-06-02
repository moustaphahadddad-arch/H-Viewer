markdown
# H-Viewer Modernization Guide

## 📚 Overview

This document provides guidance for migrating H-Viewer from the legacy Android Support Library to modern Jetpack Compose and Material Design 3.

## 🚀 What Changed

### Dependency Updates

| Component | Old | New | Notes |
|-----------|-----|-----|-------|
| **Android Gradle Plugin** | 3.0.1 | 8.2.0 | Major toolchain upgrade |
| **Kotlin** | N/A | 1.9.22 | Full Kotlin support |
| **Compile SDK** | 25 | 34 | Android 14 support |
| **Min SDK** | 17 | 24 | Android 7.0+ |
| **Target SDK** | 25 | 34 | Android 14 |
| **Java Version** | 8 | 17 | Modern Java features |
| **Support Library** | 25.2.0 | Removed | Migrated to AndroidX |
| **OkHttp** | 3.7.0 | 4.11.0 | Latest stable |
| **Retrofit** | N/A | 2.10.0 | Modern HTTP client |
| **Jsoup** | 1.9.2 | 1.16.2 | Modern HTML parsing |
| **Dagger** | 2.0 | 2.48.1 | Latest DI |
| **Fresco** | 1.8.1 | Coil 2.5.0 | Modern image loading |

## 🎨 UI Modernization

### Jetpack Compose Integration

The project now supports Jetpack Compose for building modern, declarative UIs:

```gradle
// Jetpack Compose BOM (manages versions)
implementation platform("androidx.compose:compose-bom:2024.02.00")
implementation 'androidx.compose.ui:ui'
implementation 'androidx.compose.material3:material3'
implementation 'androidx.activity:activity-compose:1.8.1'
```

### Material Design 3

Material Design 3 is now the standard for UI components:

```gradle
implementation 'com.google.android.material:material:1.1.2'
implementation 'androidx.compose.material3:material3'
```

## 📋 Migration Checklist

### 1. Import Changes

**Old Support Library imports:**
```kotlin
import android.support.v7.app.AppCompatActivity
import android.support.v4.app.Fragment
```

**New AndroidX imports:**
```kotlin
import androidx.appcompat.app.AppCompatActivity
import androidx.fragment.app.Fragment
import androidx.compose.material3.* // For Compose
```

### 2. Activity Migration to Compose

**Old (XML-based):**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

**New (Compose):**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                MainScreen()
            }
        }
    }
}

@Composable
fun MainScreen() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Welcome to H-Viewer")
        Button(onClick = { /* Action */ }) {
            Text("Start")
        }
    }
}
```

### 3. ViewModel Migration

**Old:**
```kotlin
class MyViewModel : ViewModel() {
    val liveData = MutableLiveData<String>()
}
```

**New (with Coroutines):**
```kotlin
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun loadData() {
        viewModelScope.launch {
            try {
                _uiState.value = UiState.Success(data)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

### 4. Networking with Retrofit

**Retrofit Setup:**
```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .client(
        OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))
            .build()
    )
    .build()

interface ApiService {
    @GET("data")
    suspend fun getData(): Response<DataModel>
}
```

### 5. HTML Parsing with Jsoup

**Updated Jsoup Usage:**
```kotlin
viewModelScope.launch(Dispatchers.IO) {
    try {
        val doc = Jsoup.connect("https://example.com")
            .userAgent("Mozilla/5.0")
            .get()
        
        val elements = doc.select("div.item")
        elements.forEach { element ->
            val title = element.selectFirst("h2")?.text() ?: ""
            val link = element.selectFirst("a")?.attr("href") ?: ""
            // Process data
        }
    } catch (e: Exception) {
        Log.e("Jsoup", "Error parsing HTML", e)
    }
}
```

### 6. Image Loading with Coil

**Replace Fresco with Coil:**
```kotlin
// Old (Fresco)
// Not recommended anymore

// New (Coil)
import io.coil.compose.AsyncImage

@Composable
fun ImageDisplay(imageUrl: String) {
    AsyncImage(
        model = imageUrl,
        contentDescription = "Image",
        contentScale = ContentScale.Crop,
        modifier = Modifier
            .width(200.dp)
            .height(200.dp)
            .clip(RoundedCornerShape(8.dp))
    )
}
```

### 7. Data Persistence

**Replace SharedPreferences with DataStore:**
```kotlin
// Create a DataStore
val dataStore: DataStore<Preferences> = context.createDataStore(
    name = "app_preferences"
)

// Write data
suspend fun saveUserPreference(key: String, value: String) {
    dataStore.edit { preferences ->
        preferences[stringPreferencesKey(key)] = value
    }
}

// Read data
val userPreferences: Flow<String> = dataStore.data
    .map { preferences ->
        preferences[stringPreferencesKey("user_name")] ?: ""
    }
```

### 8. Navigation with Compose

**Type-safe Navigation:**
```kotlin
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Detail : Screen("detail/{id}") {
        fun createRoute(id: String) = "detail/$id"
    }
}

@Composable
fun AppNavigation(navController: NavHostController) {
    NavHost(
        navController = navController,
        startDestination = Screen.Home.route
    ) {
        composable(Screen.Home.route) {
            HomeScreen(
                onNavigateToDetail = { id ->
                    navController.navigate(Screen.Detail.createRoute(id))
                }
            )
        }
        composable(
            route = Screen.Detail.route,
            arguments = listOf(navArgument("id") { type = NavType.StringType })
        ) { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id") ?: return@composable
            DetailScreen(id)
        }
    }
}
```

## 🔧 Build Configuration

### Android Studio Settings

Update your `local.properties` or IDE settings:
```properties
android.useAndroidX=true
android.enableJetifier=true
```

### Gradle Configuration

Ensure your `gradle.properties` has:
```properties
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m
android.useAndroidX=true
android.enableJetifier=false
kotlin.code.style=official
```

## 📱 Material Design 3 Theme

### Create Theme.kt

```kotlin
private val DarkColorScheme = darkColorScheme(
    primary = Purple80,
    secondary = PurpleGrey80,
    tertiary = Pink80
)

private val LightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40,
    tertiary = Pink40
)

@Composable
fun HViewerTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

## 🧪 Testing Updates

**Old:**
```gradle
testCompile 'junit:junit:4.12'
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
```

**New:**
```gradle
testImplementation 'junit:junit:4.13.2'
androidTestImplementation 'androidx.test.ext:junit:1.1.5'
androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
androidTestImplementation 'androidx.compose.ui:ui-test-junit4'
```

## 🔐 Security Enhancements

### Biometric Authentication

```kotlin
val biometricPrompt = BiometricPrompt(this, executor,
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(
            result: BiometricPrompt.AuthenticationResult
        ) {
            // Authentication succeeded
        }
    })

val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Authenticate")
    .setNegativeButtonText("Cancel")
    .build()

biometricPrompt.authenticate(promptInfo)
```

## 🐛 Common Migration Issues

### Issue 1: View Binding Conflicts
**Solution:** Use `viewBinding = true` in build.gradle and migrate to View Binding:
```kotlin
val binding = ActivityMainBinding.inflate(layoutInflater)
setContentView(binding.root)
```

### Issue 2: Lifecycle Warnings
**Solution:** Use lifecycle-aware components:
```kotlin
lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
    viewModel.uiState.collect { state ->
        // Update UI
    }
}
```

### Issue 3: ButterKnife Conflicts
**Solution:** Replace with View Binding or migrate to Compose

## 📖 Resources

- [Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)
- [Material Design 3](https://m3.material.io/)
- [AndroidX Migration Guide](https://developer.android.com/jetpack/androidx/migrate)
- [Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
- [Retrofit Documentation](https://square.github.io/retrofit/)
- [Jsoup Documentation](https://jsoup.org/)

## 🎯 Next Steps

1. ✅ Update build.gradle files (DONE)
2. ⬜ Create Material Design 3 theme
3. ⬜ Migrate core Activities to Compose
4. ⬜ Update networking layer with Retrofit/Coroutines
5. ⬜ Replace image loading with Coil
6. ⬜ Migrate to DataStore for preferences
7. ⬜ Update tests for Compose
8. ⬜ Add Biometric authentication

---

**Note:** This is a gradual migration path. You can keep XML layouts alongside Compose using `setContent { ComposeUI() }` in activities.
