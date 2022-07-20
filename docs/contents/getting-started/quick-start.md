# 빠른 시작

이 페이지에서는 Listen SDK를 사용하는 방법을 간단하게 소개합니다. 
사운드 이벤트 분석 기능을 사용하려면 분석 기능 이외에도 마이크 권한 요청, 녹음 등 함께 구현해야하는 기능이 많이 있습니다. 
Listen SDK는 사운드 이벤트 분석에 필요한 모든 기능을 손쉽게 사용할 수 있는 다양한 도구를 함께 제공합니다. 
이 문서에서는 그런 도구를 이용해 빠르게 사운드 이벤트 분석 기능을 구현하는 방법을 소개합니다. 

물론 Listen SDK에서 제공하는 도구를 사용하지 않고, 안드로이드 프레임워크에서 제공하는 방식으로 직접 구현하더라도 Listen 사운드 분석 기능을 동일하게 사용할 수 있습니다. 
안드로이드 프레임워크에서 제공하는 방식으로 직접 구현하는 방법은 이후의 문서에 자세하게 설명되어 있습니다. 

여기서는 라이센스를 구매하면 제공되는 SDK 키와 `.dpl` 파일을 이미 보유하고 있다고 가정하겠습니다. 


## 종속성 추가

모듈 레벨 `build.gradle` 파일에 아래와 같은 라인을 추가합니다. 

```groovy
implementation "com.deeplyinc.listen.sdk:listen:VERSION"
```


## 권한 추가

Listen을 사용하기 위해서는 `RECORD_AUDIO` 권한과 `INTERNET` 권한이 필요합니다.
`AndroidManifest.xml` 파일에 아래와 같이 권한을 선언해줍니다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```


## Listen 초기화

SDK 키와 `.dpl` 파일을 이용해 아래와 같이 Listen을 초기화합니다. 
`.dpl` 파일은 앱의 `assets` 폴더에 넣은 후 파일명을 입력하면 됩니다. 

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var listen: Listen
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ... 

        // Initialize Listen with SDK key and .dpl file
        listen = Listen(this)
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")
    }
}

```


## 오디오 녹음

Listen의 사운드 분석 기능을 사용하기 위해서는 녹음 기능을 구현해야 합니다. 
여기서는 Listen SDK와 함께 제공되는 `DeeplyRecorder`를 이용해서 구현하는 방법을 설명합니다. 
만약 안드로이드에서 제공하는 `AudioRecord`를 이용해 직접 구현하는 방법을 확인하시려면 [녹음 기능 구현](audio-recording)을 확인해주세요. 
이미 녹음 기능이 구현되어 있다면 이 부분을 뛰어넘으셔도 됩니다. 

녹음 기능을 구현하기 전, 먼저 사용자에게 녹음 권한을 요청하는 기능을 아래와 같이 구현해야 합니다. 

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var listen: Listen

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...

        listen = Listen(this)
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")

        // request audio recording permission
        DeeplyRecorder.requestAudioPermission() { isGranted ->
            if (isGranted) {
                // RECORD_AUDIO permission is granted
            }
        }
    }
}
```

이제 녹음 기능을 구현합니다. 
역시 Listen SDK에 포함되어 있는 `DeeplyRecorder`를 이용하면 쉽게 구현할 수 있습니다. 

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var listen: Listen

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...

        listen = Listen(this)
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")

        DeeplyRecorder.requestAudioPermission() { isGranted ->
            if (isGranted) {
                // start recording if the user grants the permission
                startRecording()
            }
        }
    }

    private fun startRecording() {
        val sampleRate = listen.getAudioParams().sampleRate
        val recorder = DeeplyRecorder(
            sampleRate = sampleRate,
            bufferSize = sampleRate
        )
        recorder.start().collect { audioSamples ->
            // recording started!
        }
    }
}
```


## 사운드 분석

녹음이 시작되었으니 이제 모든 준비가 완료되었습니다! 
Listen 사운드 이벤트 분석을 시작하려면 녹음을 시작한 후, 녹음된 오디오 샘플 데이터를 Listen 으로 전달하면 됩니다. 
여기서는 가장 간단한 방식인 기본 분석 방식으로 설명하겠습니다. 
Listen 이 제공하는 다양한 분석 방식에 대한 자세한 설명은 [사운드 이벤트 분석](inference) 문서를 참조해주세요.

```kotlin
recorder.start().collect { audioSamples ->
    val result = listen.inference(audioSamples)
    Log.d("Listen", result)
}
```

분석이 완료되었습니다! 
이제 실시간으로 녹음된 데이터가 `audioSamples` 값을 통해 계속 들어오고, 이 값을 `inference()` 함수에 전달하여 그 결과를 `result` 변수를 통해 확인할 수 있습니다!

이제 이 분석 결과로 여러분의 앱을 더욱 훌륭하게 만드는 일만 남았습니다. 

Listen SDK의 좀 더 자세한 사용 방법이 궁금하다면 이어지는 문서에서 그 내용을 확인하실 수 있습니다. 

