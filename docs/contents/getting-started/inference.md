# 사운드 이벤트 분석

Listen은 기본 분석, 비동기 분석, 배치 분석의 세 가지 방식의 사운드 이벤트 분석 방식을 제공합니다. 
여기서는 이미 [시작하기](getting-started) 및 [녹음 기능 구현](audio-recording)을 완료했다고 가정하고 설명을 진행합니다. 

```kotlin
val listen = Listen(context)
listen.initialze("SDK_KEY", "MODEL_ASSETS_PATH")

val audioSamples = getAudioSamplesForMic() // get audio samples from mic using AudioRecord
```


## 기본 분석

가장 간단하고 직관적인 방법은 기본 분석 방식입니다. 

```kotlin
// audioSamples must have fixed (pre-defined) sample rate and input size, 
// which is available from listen.getAudioParams().sampleRate and 
// listen.getAudioParams().inputSize, respectively.
val result = listen.analyze(audioSamples) 
```

기본 분석 기능을 제공하는 `analyze()` 함수는 미리 정해진 샘플링 속도로 녹음된 고정 길이의 오디오 샘플을 인자로 받습니다. 
각 모델에 따라 샘플링 속도와 오디오 샘플의 길이가 다를 수 있는데, 각각의 값은
`getAudioParams()` 메소드를 이용해 `sampleRate` 속성과 `inputSize` 속성으로 얻을 수 있습니다. 


## 비동기 분석

만약 실시간으로 특정한 사운드 이벤트를 감지해야 할 경우, 언제 그 이벤트가 발생할지 알 수 없기 때문에 비동기 분석 방식을 사용하는 것이 더욱 바람직합니다. 
비동기 분석은 다음과 같은 방식으로 구현됩니다. 

1. `setAsyncInferenceListener()` 메소드를 이용해 리스너를 등록합니다.
2. 실시간으로 녹음된 오디오 샘플 데이터를 `inferenceAsync(audioSamples)` 메소드로 계속 전달합니다.
3. 등록했던 사운드 이벤트가 감지되면 리스너 혹은 코틀린 코루틴이 호출됩니다. 


<!--
### 사운드 이벤트 등록

```kotlin
listen.registerEvent("cough")
// if you want to use custom threshold, specify the threshold value like below:
// listen.registerEvent("cough", threshold = 0.90)
```

분석 가능한 사운드 이벤트 리스트는 공식 문서 혹은 `getEventTypes()` 메소드를 통해 알 수 있습니다.

```kotlin
val eventTypes: List<String> = listen.getEventTypes()
```


### 오디오 샘플 전달

실시간으로 수집된 오디오 샘플 데이터를 `inferenceAsync(audioSamples)` 메소드로 전달합니다.


### 사운드 이벤트 감지

Listen은 일반적인 리스너 기반 방식과 코틀린 코루틴 기반의 방식의 두 가지 비동기 분석 방식을 제공합니다. 
두 방식 중 편리한 방식을 사용하시면 되며, 동시에 사용될 수도 있습니다. 

```kotlin

```

## 배치 분석

실시간 사운드 이벤트 분석이 필요하지 않은 경우, 혹은 대량의 사운드 데이터를 한 번에 분석해야 할 경우 기본 분석 방식보다 한 번에 많은 데이터를 처리하는 배치 분석을 이용하는 것이 더욱 바람직합니다. 
-->

