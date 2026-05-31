# Neon Bricks — Android

Пиксельный Brick Breaker в формате нативного Android-приложения. Игра работает в WebView и не требует интернета.

## Что внутри

```
android_project/
├── app/
│   ├── src/main/
│   │   ├── assets/index.html        ← сама игра (HTML/CSS/JS)
│   │   ├── java/com/neonbricks/game/MainActivity.kt
│   │   ├── res/                     ← иконки, темы, строки
│   │   └── AndroidManifest.xml
│   ├── build.gradle                 ← конфиг модуля app
│   └── proguard-rules.pro
├── build.gradle                     ← корневой Gradle
├── settings.gradle
├── gradle.properties
└── gradle/wrapper/gradle-wrapper.properties
```

## Сборка APK — самый простой путь (Android Studio)

### 1. Установи Android Studio
Скачай с https://developer.android.com/studio (бесплатно).

### 2. Открой проект
- Запусти Android Studio → **Open** → выбери папку `android_project`
- Android Studio сам скачает Gradle и зависимости (займёт 5–10 минут при первом запуске)
- Если он попросит обновить Gradle plugin или AGP — соглашайся

### 3. Собери APK
В меню сверху: **Build → Build Bundle(s) / APK(s) → Build APK(s)**

Когда сборка завершится, в правом нижнем углу появится уведомление со ссылкой **locate** — кликни, откроется папка с готовым APK:
```
app/build/outputs/apk/debug/app-debug.apk
```

### 4. Установка на телефон
- Подключи телефон по USB с включённой **отладкой по USB** (Настройки → Для разработчиков)
- Перетащи APK на телефон или установи через `adb install app-debug.apk`
- На телефоне разреши установку из неизвестных источников
- Запусти приложение **Neon Bricks**

## Сборка APK без Android Studio (CLI)

Нужно установить:
- **JDK 17** (`sudo apt install openjdk-17-jdk` или `brew install openjdk@17`)
- **Android SDK command-line tools** (https://developer.android.com/studio#command-tools)

Затем:
```bash
# создать local.properties с путём к SDK
echo "sdk.dir=/path/to/android-sdk" > local.properties

# сгенерировать gradle wrapper (если не было)
gradle wrapper

# собрать debug APK
./gradlew assembleDebug

# результат: app/build/outputs/apk/debug/app-debug.apk
```

## Релизная (подписанная) сборка для Google Play

1. Сгенерируй ключ:
```bash
keytool -genkey -v -keystore neonbricks-release.keystore \
  -alias neonbricks -keyalg RSA -keysize 2048 -validity 10000
```

2. В `app/build.gradle` добавь блок `signingConfigs` и привяжи к `buildTypes.release`:
```gradle
android {
    signingConfigs {
        release {
            storeFile file('../neonbricks-release.keystore')
            storePassword 'твой_пароль'
            keyAlias 'neonbricks'
            keyPassword 'твой_пароль'
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled false
        }
    }
}
```

3. Собери:
```bash
./gradlew assembleRelease
# результат: app/build/outputs/apk/release/app-release.apk
```

Для Play Store лучше использовать AAB:
```bash
./gradlew bundleRelease
# результат: app/build/outputs/bundle/release/app-release.aab
```

## Технические детали

- **Минимальная версия Android:** 5.0 (API 21)
- **Целевая версия:** Android 14 (API 34)
- **Размер APK:** ~2 MB (debug), ~1.5 MB (release без шринкинга)
- **Используется:** WebView с включённым JS, аппаратное ускорение
- **Ориентация:** заблокирована в портрет
- **Постоянно включённый экран:** через `FLAG_KEEP_SCREEN_ON`
- **Полноэкранный режим:** скрыты статус-бар и навигационная панель

## Что менять при необходимости

- **Игра:** `app/src/main/assets/index.html` — можно править прямо
- **Имя приложения:** `res/values/strings.xml` → `app_name`
- **Package name:** `applicationId` в `app/build.gradle` + папки `java/com/...` + `namespace`
- **Иконка:** PNG в `res/mipmap-*` или векторы в `res/drawable/ic_launcher_*`
- **Ориентация:** `AndroidManifest.xml` → `android:screenOrientation`

## Проблемы и решения

**"SDK location not found"** — создай файл `local.properties` в корне с содержимым:
```
sdk.dir=/Users/имя/Library/Android/sdk
```
(путь зависит от ОС, Android Studio покажет его в Settings → Languages & Frameworks → Android SDK)

**Gradle sync failed** — проверь, что у тебя JDK 17 (не 8, не 11). В Android Studio: File → Project Structure → SDK Location → Gradle Settings → JDK.

**Игра не запускается / белый экран** — проверь логи через `adb logcat | grep -i webview`. Скорее всего проблема с путём к ассетам.
