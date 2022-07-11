# FAQ

자주 묻는 질문에 대한 답변을 정리했습니다.


## 한 가지 유형의 이벤트가 너무 많이 감지될 경우

한 가지 유형의 이벤트가 너무 많이 감지되는 경우, Listen에 잘못된 오디오 샘플 데이터가 인풋으로 전달되고 있을 수 있습니다.
만약 모든 값이 0으로 구성된 인풋 값이 Listen에 계속 인풋으로 들어갈 경우, 이 값은 올바른 자료형을 가지고 있기 때문에 Listen은 정상적으로 처리를 진행합니다. 
하지만 대부분의 경우 이런 인풋 값은 구현 실수 혹은 에러로 인한 잘못된 인풋이기 때문에, Listen은 아래와 같은 경고 메시지를 출력합니다.

```text
All input values are zero. The inference task will continue, but it is usually the result of bad recording.
```

