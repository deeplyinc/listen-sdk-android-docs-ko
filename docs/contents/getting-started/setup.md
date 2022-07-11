# 설정

이 페이지에서는 Listen을 안드로이드 프로젝트에 추가하고 설정하는 방법을 소개합니다.

## 종속성 추가

아래 라인을 모듈 레벨 `build.gradle` 파일에 추가합니다.

```groovy
dependencies {
    implementation "com.deeplyinc.listen:listen:VERSION"
    implementation "com.deeplyinc.library.audioutils:audioutils:VERSION" // optional: audioutils
}
```

## 권한 설정

Listen은 `RECORD_AUDIO` 권한과 `INTERNET` 권한을 사용합니다.
`AndroidManifest.xml` 파일에 아래 라인을 추가해줍니다. 

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
```