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

SDK 키와 `.dpl` 파일을 이용해 아래와 같이 Listen을 초기화합니다. 


```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ... 

        // Initialize Listen with SDK key and .dpl file
        val listen = Listen(this)
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")
    }
}
```

`.dpl` 파일은 `assets` 폴더에 넣은 후 파일명을 입력하면 됩니다. 
예를 들어 `listen.dpl` 파일을 `app/src/main/assets/model.dpl` 위치에 넣었다면 아래와 같이 입력하면 됩니다.

```kotlin
listen.init("SDK KEY", "listen.dpl")
```

`init()` 메소드는 인증, 딥러닝 모델 로딩 등에 시간이 걸릴 수 있으니 앱 최초 구동 시 미리 하는 것을 추천합니다.




