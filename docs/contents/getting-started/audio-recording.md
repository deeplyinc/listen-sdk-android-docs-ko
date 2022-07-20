# 녹음 기능 구현

Listen 사운드 분석 기능을 사용하기 위해서는 우선 안드로이드에서 소리를 녹음해야 합니다. 
Listen SDK는 이 녹음 기능을 코틀린 코루틴을 이용해 더욱 쉽게 구현할 수 있는 `DeeplyRecorder`를 제공합니다. 
물론 안드로이드에서 기본적으로 제공하는 `AudioRecord`를 이용해서도 녹음 기능을 구현해도 Listen의 모든 사운드 분석 기능을 동일하게 사용할 수 있습니다. 



## DeeplyRecorder

`DeeplyRecorder`는 녹음 구현에 필요한 다양한 기능을 코틀린 코루틴을 이용해 손쉽게 AudioRecord를 사용할 수 있게 도와주는 디플리의 오픈소스 라이브러리로, Listen SDK에 기본적으로 포함되어 있습니다. 
여기서는 이 `DeeplyRecorder`를 이용해 녹음 기능을 구현하는 방법을 소개합니다. 
`DeeplyRecorder`에 대한 더 자세한 내용은 [공식 문서](https://audioutils-android.readthedocs.io)나 [GitHub 링크](https://github.com/deeplyinc/deeply-recorder-android)를 참고해주세요. 


### 녹음 권한 요청

`RECORD_AUDIO` 권한은 사용자의 개인정보와 관련된 민감한 권한으로, 안드로이드 프레임워크에서 위험한 권한으로 분류됩니다. 
이런 위험한 권한의 경우에는 `AndroidManifest.xml` 파일에 권한 선언을 해놓았더라도 추가적으로 앱이 실행되는 동안에 사용자로부터 실시간으로 권한 사용 허가를 받아야 합니다. 
Listen SDK를 사용하면 아래와 같은 방식으로 사용자에게 녹음 권한을 손쉽게 요청, 처리할 수 있습니다. 

```kotlin
DeeplyRecorder.requestAudioPermission() { isGranted ->
    if (isGranted) {
        // RECORD_AUDIO permission is granted
    }
}
```

### 녹음 기능 구현

Listen SDK를 이용하면 녹음 기능을 아래와 같이 손쉽게 구현할 수 있습니다. 

```kotlin
val recorder = DeeplyRecorder()
recorder.start().collect { audioSamples ->
    //
}
```

녹음 기능을 구현할 때는 앱의 기능과 사용 방식에 따라 어디에서 작동하도록 구현할지 잘 결정해야 합니다. 
대표적으로 Activity (ViewModel), Service, ForegroundService 중 하나에서 작동하도록 구현하는 것이 일반적입니다. 
이에 대한 자세한 내용은 [백그라운드 녹음](../advanced-topics/background-recording) 에서 설명합니다. 

이제 Listen의 사운드 분석 기능을 사용할 준비가 되었습니다. [사운드 이벤트 분석](inference) 문서로 넘어갑니다. 



## AudioRecord

### 녹음 권한 요청

녹음 기능을 구현하기 전, 먼저 사용자에게 녹음 권한을 요청하는 기능을 아래와 같이 구현해야 합니다. 

```kotlin
class MainActivity : AppCompatActivity() {

    // for RECORD_AUDIO permission request
    private val requestRecordPermission = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            // Audio recording permission is granted
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...

        val listen = Listen(this)
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")

        // Request RECORD_AUDIO permission
        requestRecordPermission.launch(Manifest.permission.RECORD_AUDIO)
    }
}
```

### 녹음 기능 구현

안드로이드에서 녹음 기능을 구현하려면 `AudioRecord` 혹은 `MediaRecorder`를 이용할 수 있습니다. 
Listen 연동을 위해서는 원본 오디오 데이터를 실시간으로 처리하는데 적합한 `AudioRecord`를 이용하는 방식이 더욱 적합합니다. 
`AudioRecord`와 관련된 더 많은 정보를 원하시면 [공식 문서](https://developer.android.com/reference/android/media/AudioRecord)를 참고하세요.

```kotlin
class MainActivity : AppCompatActivity() {

    private val requestPermission = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            // start recording if permission is granted
            startRecording()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...

        val listen = Listen(this)
        listen.init("SDK KEY", "DPL FILE ASSETS PATH")

        requestPermission.launch(Manifest.permission.RECORD_AUDIO)
    }

    private fun startRecording() {
        val sampleRate = listen.getAudioParams().sampleRate
        val audioRecord = AudioRecord(
            MediaRecorder.AudioSource.MIC,
            sampleRate,
            AudioFormat.CHANNEL_IN_STEREO,
            AudioFormat.ENCODING_PCM_16BIT,
            sampleRate // bufferSizeInBytes
        )


    }
}
```

[샘플 앱](https://github.com/deeplyinc/listen-sdk-android-samples)을 통해 실제로 구현된 예제를 참고할 수 있습니다. 

녹음 기능 구현이 완료되었습니다. 
이제 Listen의 사운드 분석 기능을 사용하기 위해 [사운드 이벤트 분석](inference) 문서로 넘어갑니다. 

