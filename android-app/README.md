# 📱 AIoT Climate Control App (Mobile & WearOS)

이 디렉토리는 **AIoT-WearOS-Climate-Control** 프로젝트의 **클라이언트(Client) 애플리케이션** 소스 코드를 포함합니다.
WearOS 워치 앱(Sensor)과 Android 폰 앱(Hub)으로 구성된 **Multi-Module 아키텍처**입니다.

## 📂 Module Structure

프로젝트는 역할에 따라 두 개의 핵심 모듈로 나뉩니다.

| Module | Type | Role & Responsibility | Key Tech |
| :--- | :--- | :--- | :--- |
| **`:wear`** | `com.android.application` | **Data Collector (Edge)**<br>- 실시간 심박수(PPG) 측정<br>- 가속도 센서 데이터 전처리 (LPF)<br>- 모바일로 데이터 송신 | Health Services API<br>Data Layer API |
| **`:app`** | `com.android.application` | **Data Hub & Brain**<br>- 워치 데이터 수신<br>- 외부 날씨 API 호출 (OpenWeatherMap)<br>- 사용자 피드백 수집 및 데이터 로깅<br>- ML 모델 추론 결과 알림 | Jetpack Compose<br>Retrofit2<br>Room / File I/O |

---

## 🛠 Tech Stack & Libraries

### Core
* **Language:** Kotlin 1.9+
* **UI Framework:** Jetpack Compose (Material 3 for Mobile, Wear Compose for Watch)
* **Async:** Coroutines & Flow

### Connectivity & Sensors
* **Wearable Data Layer:** `com.google.android.gms:play-services-wearable` (Phone ↔ Watch 통신)
* **Health Services:** `androidx.health:health-services-client` (배터리 효율적인 센서 접근)
* **Network:** Retrofit2 + OkHttp3 (OpenWeatherMap API 연동)

### Signal Processing (Engineering Base)
* **Moving Average Filter:** WearOS 모듈 내 구현. PPG 센서의 고주파 노이즈 제거.
* **Adaptive Sampling:** 배터리 상태 및 활동량에 따른 센싱 주기 가변 제어 로직 포함.

---

## 🏗 System Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Watch as :wear (Galaxy Watch)
    participant Phone as :app (Smartphone)
    participant Cloud as OpenWeatherMap API

    Note over Watch: 1. Bio-Signal Sensing
    Watch->>Watch: LPF (Signal Processing)
    Watch->>Phone: Send HeartRate & Activity (DataClient)
    
    Note over Phone: 2. Data Aggregation
    Phone->>Cloud: Get Temp/Humidity
    Cloud-->>Phone: Response (24°C, 60%)
    
    Phone->>Phone: Merge Data (HR + Weather)
    
    opt Training Mode
        User->>Phone: Input Preference ("I feel Hot")
        Phone->>Phone: Save to .txt (Labeling)
    end

    opt Inference Mode
        Phone->>Phone: Random Forest Inference
        Phone->>User: Notification ("Recommended Temp: 23°C")
    end