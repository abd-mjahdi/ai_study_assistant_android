# StudyFlow - AI Study Assistant Android Client

StudyFlow is a native Android application designed to transform study materials into active recall tools. By combining on-device Optical Character Recognition (OCR) via Google ML Kit with a dedicated Spring Boot AI backend, the app allows students to convert handwritten notes, printed textbooks, or raw text into structured summaries, interactive quizzes, and 3D animated flashcards.

Part of the AI Study Assistant platform:
* Android Client: [https://github.com/abd-mjahdi/ai_study_assistant_android](https://github.com/abd-mjahdi/ai_study_assistant_android)
* Backend Service: [https://github.com/abd-mjahdi/ai-study-assistant-backend](https://github.com/abd-mjahdi/ai-study-assistant-backend)

---

## Architectural Overview

The application follows clean Android architectural practices, combining modern Android Jetpack components, Material Design 3 guidelines, and on-device machine learning.

```
+-----------------------------------------------------------------------------------+
| StudyFlow Android Client                                                          |
|                                                                                   |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  | Camera / Gallery   |-->| Google ML Kit       |-->| Input Controller         |  |
|  | Image Capture      |   | (On-Device OCR)     |   | (Text / Subject / Depth) |  |
|  +--------------------+   +---------------------+   +--------------------------+  |
|                                                                  |                |
|                                                                  v                |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  | ReviewLaterStore   |   | SharedPreferences   |   | Retrofit 2 + OkHttp      |  |
|  | (Local Review Queue|   | (TokenManager - JWT)|   | (AuthInterceptor)        |  |
|  +--------------------+   +---------------------+   +--------------------------+  |
+------------------------------------------------------------------|----------------+
                                                                   |
                                                            HTTPS / REST (JWT)
                                                                   |
                                                                   v
                                                      +--------------------------+
                                                      |  Spring Boot Backend API |
                                                      |  (PostgreSQL + Gemini)   |
                                                      +--------------------------+
```

### Engineering Highlights

1. **Edge OCR Processing**: Optical Character Recognition runs entirely on the device using Google ML Kit Vision. The application extracts textual content locally from camera captures and photo gallery images, never transmitting raw image files to the server. This guarantees user privacy, reduces network latency, and conserves bandwidth.
2. **Interactive 3D Card Physics**: The flashcard study mode implements card flipping using Android `AnimatorSet` and object animators with custom camera distance calculations to render 3D perspective without external graphics engines.
3. **Automated Knowledge Gap Detection**: Quizzes scored at or below 50% are automatically intercepted by the `ReviewLaterStore` and cataloged into an on-device review queue, allowing students to target weak areas.
4. **Resilient Network Layer**: Network communication is managed through Retrofit 2 and OkHttp, featuring automated JWT Bearer token injection, connection timeout handling, and specific error interception for server-side rate limits (HTTP 429).

---

## Tech Stack and Dependencies

| Layer / Concern | Technology | Version | Purpose |
|---|---|---|---|
| Platform | Android SDK | Min SDK 24, Target SDK 36 | Modern Android runtime compatibility |
| Language | Java | 11 | Application programming language |
| UI Framework | Material Design 3, ViewBinding | 1.13.0 | Modern UI components and view safety |
| Machine Learning | Google ML Kit Text Recognition | 16.0.1 | High-accuracy on-device OCR (Latin script) |
| Networking | Retrofit 2 & OkHttp 3 | 2.11.0 / 4.12.0 | RESTful API client with logging and interceptors |
| JSON Serialization | Google Gson | 2.11.0 | Data serialization and deserialization |
| Navigation | AndroidX Navigation Components | 2.9.0 | Safe fragment transactions and bottom navigation |
| Persistence | SharedPreferences | Jetpack Core | User token storage and local review queue |
| Build System | Gradle with Version Catalogs | 9.1.1 (AGP) | Dependency management via `libs.versions.toml` |

---

## Core Features

### 1. Multi-Modal Content Ingestion
* **Manual Text Entry**: Direct input field supporting up to 20,000 characters with live character counter.
* **Camera Capture**: Integrates with the Android Camera API through `ActivityResultContracts.TakePicture`, writing to a secured `FileProvider`.
* **Gallery Selection**: Supports selecting local device images via `ActivityResultContracts.GetContent`.
* **On-Device OCR**: Instant parsing of printed or handwritten text directly into the input buffer.

### 2. Generative Study Modes
* **Smart Summaries**: Selectable depth profiles:
  * `ESSENTIALS`: Core takeaways and high-level concepts.
  * `BALANCED`: Default structured breakdown with key definitions.
  * `THOROUGH`: Comprehensive study notes with detailed context.
* **Interactive Quizzes**:
  * Configurable question counts from 3 to 10 questions.
  * Multiple-choice questions with real-time visual feedback (green for correct, red for incorrect).
  * In-depth explanation cards revealed upon answer submission.
  * Final score summary and review recommendation.
* **3D Flip Flashcards**:
  * Front-and-back study cards with horizontal flip animation.
  * Progress indicator tracking card index and total count.

### 3. Review and Retention System
* In-app review queue cataloging failed quizzes and flagged summaries.
* Persisted locally per user account in `SharedPreferences` with JSON serialization.
* One-tap restart for targeted re-testing of flagged content.

### 4. Account Management and History
* Token-based user authentication (Register, Login, Logout) with JWT stored in `TokenManager`.
* Profile view with options to update username and password.
* Full study history browser with pull-to-refresh and session deletion.

---

## Application Structure

```
app/src/main/java/com/example/ai_study_assistant_android/
|
+-- model/               # Data transfer objects and entity models
|   +-- AuthRequest.java
|   +-- AuthResponse.java
|   +-- Flashcard.java
|   +-- GenerateRequest.java
|   +-- HistoryItem.java
|   +-- QuizQuestion.java
|   +-- StudySession.java
|
+-- network/             # Network communication layer
|   +-- ApiClient.java          # Retrofit singleton configuration
|   +-- ApiService.java         # REST endpoint contracts
|   +-- AuthInterceptor.java    # Automatic JWT Bearer token attachment
|   +-- TokenManager.java       # SharedPreferences token persistence
|
+-- review/              # Local review and spaced repetition queue
|   +-- ReviewLaterStore.java   # Manages failed quizzes and saved summaries
|
+-- ui/                  # User interface presentation layer
    +-- auth/            # LoginActivity, RegisterActivity
    +-- generate/        # GenerateFragment (Input tabs, OCR, depth chips)
    +-- history/         # HistoryFragment, HistoryAdapter
    +-- home/            # HomeFragment (Quick actions, recent activity)
    +-- main/            # MainActivity (BottomNavigationView host)
    +-- profile/         # ProfileFragment, EditProfileActivity
    +-- result/          # ResultActivity, SummaryFragment, QuizFragment, FlashcardsFragment
    +-- review/          # ReviewFragment, ReviewListAdapter
```

---

## Getting Started

### Prerequisites
* Android Studio Ladybug (2024.2.1) or newer
* Android SDK Platform 36 installed (or compatible SDK 24+)
* Java Development Kit (JDK) 11 or 17
* Physical Android device or Android Emulator running API level 24 or higher
* Running backend instance: [ai-study-assistant-backend](https://github.com/abd-mjahdi/ai-study-assistant-backend)

### Setup Instructions

1. Clone the repository:
   ```bash
   git clone https://github.com/abd-mjahdi/ai_study_assistant_android.git
   cd ai_study_assistant_android
   ```

2. Configure Backend Endpoint:
   Open `app/build.gradle` and configure the `BASE_URL` build config field:

   * When running against an Android Emulator (loopback to host):
     ```groovy
     buildConfigField "String", "BASE_URL", "\"http://10.0.2.2:8080/\""
     ```
   * When running on a physical Android device on the same local Wi-Fi network:
     ```groovy
     buildConfigField "String", "BASE_URL", "\"http://192.168.1.X:8080/\""
     ```

3. Build the Project:
   Using Gradle:
   ```bash
   ./gradlew assembleDebug
   ```
   Or open the root folder in Android Studio and trigger `Sync Project with Gradle Files`.

4. Run the Application:
   Deploy the app to your connected device or emulator directly via Android Studio (`Run 'app'`) or through ADB:
   ```bash
   ./gradlew installDebug
   ```

---

## Testing

Unit and instrumentation tests are executed via Gradle:

```bash
# Run local unit tests
./gradlew test

# Run connected Android tests (requires device or emulator)
./gradlew connectedAndroidTest
```
