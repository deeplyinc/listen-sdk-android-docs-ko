# 오디오 샘플링

오디오가 마이크를 통해 녹음되어 전달되는 방식을 조금 더 구체적으로 알면 Listen 사용과 오디오 관련 기능 구현에 큰 도움이 됩니다. 

## 기본 개념

마이크는 아날로그 신호인 소리를 아주 짧은 주기로 나누어 수집합니다. 
예를 들어 마이크가 1초에 1,000번 (0.001초에 한 번씩) 소리를 수집할 수도 있고, 1초에 10,000번 (0.0001초에 한 번씩) 수집할 수도 있습니다. 
이렇게 마이크에서 소리를 수집하는 과정을 *오디오 샘플링*이라고 하고, 그렇게 소집된 개별 소리값을 *오디오 샘플*, 1초에 몇 번 소리를 수집하는 횟수를 *샘플링 비율* 혹은 *샘플링 속도*라고 부릅니다. 
예를 들어 1초에 1,000개의 오디오 샘플을 수집하면 샘플링 비율은 1,000 Hz 혹은 1 KHz가 됩니다. 
또한 하나의 오디오 샘플을 몇 비트로 표현하는지는 샘플 깊이 혹은 비트 깊이라고 합니다. 
안드로이드 기기에서는 일반적으로 하나의 오디오 샘플을 16 비트(= 2 바이트)로 표현하고 저장합니다. 
즉, 하나의 마이크에서 16 비트 샘플 깊이로 1,000 Hz 샘플링 비율로 1초 동안 녹음을 한다면 이렇게 녹음된 소리 데이터의 용량은 아래와 같이 계산될 수 있습니다. 

```
1 mic * 16 bit * 1 sec * 1,000 samples/sec
= 1 mic * 2 byte * 1 sec * 1,000 samples/sec
= 2,000 bytes = 2 KB
```

이렇게 샘플링 비율은 1초의 소리를 몇 개의 오디오 샘플로 표현하는지에 대한 정보이기 때문에, 다른 조건이 동일하다면 샘플링 비율이 높을 수록 음질이 더 좋아지고, 동일한 시간 동안 소리를 녹음하더라도 샘플링 비율이 높은 쪽이 용량이 더 커집니다. 
대표적으로 유선 전화의 경우 8 KHz 샘플링 비율이 널리 사용되고 WAV, MP3 파일과 같은 음악 파일의 경우 44.1 KHz 혹은 48 KHz 샘플링 비율이 많이 사용되는데, 이는 곧 유선 전화는 8,000개의 오디오 샘플로 1초의 소리를 표현하고, 음악 파일은 44,100개 혹은 48,000개의 오디오 샘플로 1초의 소리를 표현한다는 뜻입니다. 
유선 전화와 오디오 파일의 음질 차이를 생각해보면 이 샘플링 비율의 차이가 음질에 미치는 영향을 알 수 있습니다. 

이렇게 마이크에서 어떻게 소리를 수집하는지 알면 소리를 다루는 기능을 구현할 때 큰 도움이 됩니다. 
아래에서는 이 개념을 활용해 어떻게 더 효율적으로 기능을 구현할 수 있는지 설명합니다.


## 녹음과 최소 버퍼 크기

안드로이드에서 녹음 기능은 마이크를 통해 수집된 오디오 샘플을 버퍼를 이용해 특정한 갯수만큼 읽어오는 방식으로 작동합니다. 

예를 들어, 이론적으로는 동일한 10,000 Hz 샘플링 비율로 녹음을 하더라도 한 번에 한 개의 오디오 샘플을 읽어올 수도 있고, 여러 개의 오디오 샘플을 읽어올 수도 있습니다. 
만약 16,000 Hz 샘플링 비율로 녹음을 하고, 한 번에 1,600 개씩 오디오 샘플을 읽어온다면 우리는 0.1 초에 한 번씩 1,600 개의 새로운 오디오 샘플을 처리할 수 있는 셈입니다. 
이를 구현한다면 아래와 같습니다. 

```kotlin
// DeeplyRecorder
val recorder = DeeplyRecorder(sampleRate = 16000, bufferSize = 1600)
lifecycleOwner.launch {
    recorder.start().collect { audioSamples ->
        runSomething() // called every 0.1 second, buffer contains 1,600 samples
    }
}
```

```kotlin
// AudioRecord
val SAMPLE_RATE = 16000
val BUFFER_SIZE = 1600

val buffer = ByteArray(BUFFER_SIZE)
val record = AudioRecord(MediaRecorder.AudioSource.MIC, SAMPLE_RATE, AudioFormat.CHANNEL_IN_MONO, AudioFormat.ENCODING_PCM_16BIT, buffer.size)
record.startRecording()
while (true) {
    val read = record.read(buffer, 0, buffer.size)
    runSomething() // called every 0.1 second, buffer contains 1,600 samples
}
```

이렇게 하면 위의 `runSomething()` 함수는 1,600 개의 새로운 오디오 샘플이 버퍼에 모일 때마다, 즉 0.1초에 한 번씩 호출됩니다.

그렇다면 항상 최소한의 오디오 샘플을 가져오는 것이 가장 좋지 않을까요?
만약 runSomething() 함수에서 UI를 업데이트 하는 작업을 실행할 경우, 위의 예시처럼 16,000 Hz 샘플링 비율에 1,600 버퍼 크기를 사용하면 0.1 초에 한 번씩 UI가 변화하게 만들 수 있지만, 버퍼 크기를 10 으로 사용하면 0.001 초에 한 번씩 UI가 변화하게 만들 수 있으니까요. 
만약 샘플링 비율을 1,000,000 Hz 로 설정하고, 버퍼 크기는 1로 설정할 수 있다면 소리에 따라 정말 빠르게 반응하는 기능을 만들 수 있겠네요!

하지만 현실적으로는 그것이 불가능합니다. 
마이크의 특성에 따라 설정할 수 있는 샘플링 비율과 한 번에 가져올 수 있는 최소한의 샘플 갯수, 즉 최소 버퍼 크기가 정해져 있기 때문입니다. 

그래서 특정한 샘플링 비율에서 빠르게 반응하도록 만드려면 버퍼 크기가 가능한 작은 사이즈가 되어야 합니다. 
DeeplyRecorder 에서는 기본 값으로 (2 * 최소 사이즈) 버퍼 크기를 선택하기 때문에 대부분의 경우 별도의 설정을 할 필요가 없습니다. 
AudioRecord 는 객체를 생성할 때 먼저 `AudioRecord.getMinBufferSize()` 메소드를 통해 최소 버퍼 크기를 알아낸 후 이 값을 AudioRecord 생성자를 통해 설정해주어야 합니다.

```kotlin
// DeeplyRecorder
val recorder = DeeplyRecorder(sampleRate = 16000) // the buffer size will be set to the minimum size
val bufferSize = recorder.getBufferSize() // if you want to know the buffer size
```

```kotlin
// AudioRecord
val SAMPLE_RATE = 16000
val minBufferSize = AudioRecord.getMinBufferSize(SAMPLE_RATE, AudioFormat.CHANNEL_IN_MONO, AudioFormat.ENCODING_PCM_16BIT)
val record = AudioRecord(MediaRecorder.AudioSource.MIC, SAMPLE_RATE, AudioFormat.CHANNEL_IN_MONO, AudioFormat.ENCODING_PCM_16BIT, minBufferSize)
```



## 샘플링 비율로 시간 다루기

만약 소리를 10초마다 한 번씩 잘라서 `.wav` 파일로 저장해야 한다면 어떻게 하면 될까요? 
안드로이드에서 특정한 시간 간격으로 기능이 작동해야 하는 경우 `Hanlder` 를 이용한 `postDeplayed()` 메소드를 이용하는 것이 가장 일반적인 방법이겠지요.
하지만 오디오를 처리할 때는 이것은 좋지 않은 방법입니다. 

<!-- 

왜 좋지 않은 방법인지 자세한 예를 통해 알아보고 싶다면 아래 '세부 내용 보기'를 눌러주세요. 

1. AudioRecord 객체를 10초마다 다시 생성하는 방식
2. 하나의 AudioRecord 객체를 사용하지만 startRecording() 함수와 stopRecording() 함수를 번갈아 호출하는 방식
3. 하나의 AudioRecord 객체를 사용하면서 버퍼에 오디오 샘플을 축적하고, 정해진 시간마다 저장되어 있는 모든 데이터를 가져오는 방식

대표적으로 아래와 같은 몇 가지 이유가 있습니다. 
- AudioRecord 객체 생성에 소요되는 시간, 그리고 startRecording() 함수와 `stopRecording()` 함수가 실행되는 시간 사이의 짧은 시간 동안의 오디오 샘플 데이터 일부가 손실됩니다. 
- AudioRecord `startRecording()` 함수와 `stopRecording()` 함수를 빠르게 반복 호출하면 종종 제대로 작동하지 않습니다. 
- 버퍼에 무한히 오디오 샘플이 쌓여서 메모리 에러가 발생하지 않도록 계속 메모리를 비워주는 등의 관리를 해주어야 합니다. 

-->

그러면 어떻게 하는 것이 좋을까요? 
바로 버퍼의 사이즈를 조절하면 됩니다. 
아래와 같이 버퍼의 사이즈를 `10 * 샘플링 비율`로 설정하면, 버퍼를 통해 우리가 얻게 되는 오디오 샘플은 정확히 10초 길이의 오디오가 됩니다. 

```kotlin
val sampleRate = 16000
val recorder = DeeplyRecorder(bufferSize = 10 * sampleRate)
recorder.start().collect { audioSamples ->
    // audioSamples have 10 second length of audio samples
    buildWavFile(audioSamples)
}
```

이렇게 오디오 샘플의 갯수는 곧 시간을 의미한다는 점을 기억하면, 시간과 관련된 오디오 기능을 구현하실 때 큰 도움이 될 수 있습니다. 



## Listen 적용

Listen 사운드 이벤트 AI 분석 모델도 특정한 샘플링 비율 값에 맞추어 만들어졌습니다. 
따라서 Listen 사운드 이벤트 AI 분석을 위해서는 미리 설정되어 있는 AI 모델의 샘플링 비율에 맞추어 녹음 기능도 구현되어야 합니다. 
Listen SDK는 AI 모델의 샘플링 비율을 `getAudioParams()` 메소드를 이용해 제공합니다.
만약 `DeeplyRecorder`를 이용해 녹음 기능을 구현하고 기본 추론 방식으로 사용한다면 아래와 같이 사용할 수 있습니다. 

```kotlin
val listen = Listen(this)
listen.load("SDK_KEY", "DPL FILE ASSETS PATH")

val audioParams = listen.getAudioParams()
val recorder = DeeplyRecorder(
    sampleRate = audioParams.sampleRate,
    bufferSize = audioParams.minInputSize
)
```

<!-- 
주의! 
`.dpl` 파일에 따라 샘플링 비율 값이 달라질 수 있습니다. 
만약 녹음 기능을 Listen 뿐만아니라 다른 목적으로도 동시에 사용한다면 녹음 시의 샘플링 비율이 변경되더라도 문제가 없도록 코드를 작성해야 합니다. 
-->

