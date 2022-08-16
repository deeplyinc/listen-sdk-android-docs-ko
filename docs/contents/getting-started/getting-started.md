# 시작하기

이 페이지에서는 Listen을 안드로이드 프로젝트에 추가하고 설정하는 방법을 소개합니다.

## 종속성 추가

아래 라인을 모듈 레벨 `build.gradle` 파일에 추가합니다.

```groovy
dependencies {
    implementation "com.deeplyinc.listen:listen:VERSION"
}
```

## 권한 설정

Listen은 `RECORD_AUDIO` 권한과 `INTERNET` 권한을 사용합니다.
각각의 권한은 아래와 같은 목적으로 필요합니다. 

- `RECORD_AUDIO` - 녹음 기능 구현을 위해 필요합니다. 
- `INTERNET` - Listen SDK의 인증에 필요합니다. 녹음된 오디오 데이터는 전혀 전송되지 않습니다.

`AndroidManifest.xml` 파일에 아래 라인을 추가해줍니다. 

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
```

`RECORD_AUDIO` 권한은 개인정보보호와 밀접하게 관련되어 있어 안드로이드 프레임워크에서 [위험한 권한](https://developer.android.com/guide/topics/permissions/overview#runtime)으로 분류되기 때문에, 추후에 녹음 기능을 구현할 때에 사용자로부터 실시간으로 권한 사용에 대한 허가를 받아야 합니다. 
이에 대한 내용은 [녹음 기능 구현](audio-recording)에서 자세하게 설명하겠습니다. 


## 초기화

Listen을 사용하기 위해서는 Listen 인스턴스를 생성한 후 SDK 키와 `.dpl` 파일을 이용해 초기화를 먼저 해주어야 합니다. 


### 가장 간단한 방법

가장 간단한 형태의 초기화 코드는 아래와 같습니다. 

```kotlin
class MainActivity : AppCompatActivity() {
    
    // Creat a Listen instance
    private val listen = Listen(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Initialize Listen with SDK key and .dpl file
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")
    }
}
```

이 때 `.dpl` 파일은 `assets` 폴더에 넣은 후 그 파일명을 `init()` 함수에 전달하면 됩니다. 
예를 들어 `listen.dpl` 파일을 `app/src/main/assets/model.dpl` 위치에 넣었다면 아래와 같이 입력하면 됩니다.

```kotlin
listen.init("SDK KEY", "listen.dpl")
```

위의 코드는 작동합니다. 
하지만 Listen을 좀 더 안정적으로 활용하기 위해 아래의 몇 가지 사항을 함께 고려하는 것이 좋습니다. 


### 스레드 블락킹 이슈

`init()` 메소드는 내부적으로 SDK 인증, 딥러닝 모델 로딩 등 다양한 작업을 진행하는데, 디바이스의 성능과 네트워크 상태 등에 따라 이 과정에는 수 초의 시간이 걸릴 수 있습니다. 
초기화 작업은 비동기가 아닌 동기 방식으로 진행되기 때문에 이 작업을 처리하는 동안 스레드는 블락킹될 수 있습니다. 
따라서 만약 메인 스레드에서 초기화 작업을 진행할 경우 스레드 블락킹에 의한 [ANR (Application Not Responding)](https://developer.android.com/topic/performance/vitals/anr) 현상이 발생할 수 있습니다. 

이 문제를 해결하기 위해 `init()` 함수를 별도의 스레드에서 작동시키는 것을 추천합니다. 
아래는 코틀린 코루틴을 활용한 예시입니다. 

```kotlin
// Note that the init() takes time and blocks the thread during the initialization
// process because it contains networking and file operations.
// We recommend to call init() in other thread like the following code.
lifecycleScope.launch(Dispatchers.Default) {
    listen.init("SDK KEY", "DPL FILE ASSETS PATH")
}
```


### 예외처리

`init()` 함수는 여러 가지 이유로 예외를 발생시킬 수 있습니다. 
대표적으로 `.dpl` 파일이 존재하지 않는 경우, 인터넷 연결 문제 등으로 인해 인증 서버 접속에 실패한 경우, SDK 키의 사용 기간이 만료된 경우 등이 있습니다. 
이러한 예외를 따로 처리해주지 않을 경우 앱이 강제 종료되는 현상이 발생합니다. 
따라서 앱의 안정적인 실행을 위해서는 예외가 발생하는 상황에 대한 예외처리를 아래와 같이 해주어야 합니다. 

```kotlin
try {
    listen.init("SDK KEY", "DPL FILE ASSETS PATH")
} catch (e: ListenAuthException) {
    // Handle authentication exception
}
```


### 추천 방식

위에서 소개한 고려사항을 반영한 추천 예시 초기화 코드입니다. 

```kotlin
class MainActivity : AppCompatActivity() {
    
    // Creat a Listen instance
    private val listen = Listen(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Note that the init() takes time and blocks the thread during the initialization
        // process because it contains networking and file operations.
        // We recommend to call init() in other thread like the following code.
        lifecycleScope.launch(Dispatchers.Default) {
            try {
                // Initialize Listen with SDK key and .dpl file
                listen.init("SDK KEY", "DPL FILE ASSETS PATH")
            } catch (e: ListenAuthException) {
                // Handle authentication exception
            }
        }
    }
}
```
