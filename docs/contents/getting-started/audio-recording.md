# 녹음 구현

Listen을 사용하기 위해서는 우선 안드로이드에서 소리를 녹음해야 합니다. 
이를 위해 코틀린 코루틴을 이용해 더욱 쉽게 녹음을 구현할 수 있는 `audioutils` 사용을 추천합니다. 
물론 안드로이드에서 기본적으로 제공하는 `AudioRecord`를 이용해서도 녹음 기능을 구현할 수 있습니다. 

## AudioRecord를 이용한 녹음 구현

안드로이드에서 녹음 기능을 구현하려면 `AudioRecord` 혹은 `MediaRecorder`를 이용할 수 있습니다. Listen 연동을 위해서는 원본 오디오 데이터를 실시간으로 처리하는데 적합한 `AudioRecord`를 이용하는 방식이 더욱 적합합니다. 

```kotlin
val sampleRate = 16000
val audioRecord = AudioRecord()
```

샘플 앱을 통해 실제로 구현된 예제를 참고할 수 있습니다. 
`AudioRecord`와 관련된 더 많은 정보를 원하시면 [공식 문서](https://developer.android.com/reference/android/media/AudioRecord)를 참고하세요.

## AudioUtils를 이용한 녹음 구현 

`audioutils`는 녹음 기능을 포함해 소리 크기 측정, `.wav` 파일 내보내기, 톤 생성 등 다양한 사운드 관련 추가 기능을 제공하는 디플리의 오픈소스 라이브러리입니다.
여기서는 이 `audioutils`를 이용해 녹음 기능을 구현하는 방법을 소개합니다. 
`audioutils`에 대한 더 자세한 내용은 `audioutils` [공식 문서](https://audioutils-android.readthedocs.io)나 [GitHub 링크](https://github.com/deeplyinc/audioutils-android)를 참고해주세요. 

