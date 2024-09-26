# FCM 비동기 쓰레드 재정의로 성능 개선


Spring Batch 에서 FireBase 로 알림 서비스를 개선 중에 FireBase 로 Push Message 를 보내는 상황에서 I/O 병목이 발생

이 글은 기존의 Spring Batch 에 대한 기본적인 개념을 숙지하고 참고하는게 좋다.


- 기본적으로 Spring Batch 는 Step 에서 `ItemReader`, `ItemProcessor`, `ItemWriter`로 구성된다.
  - 기존에 `ItemReader`는 JpaPagingItemReader 를 사용하여 FCM 에 보낼수 있는 500 chunk 단위의 데이터를 가져온다.
  - `ItemProcessor`는
 





- Firebase SDK 를 뜯어보자

  - Firebase SDK 를 build.gradle 에 추가하면 `FirebaseMessaing` 이라는
 



~~~java
public class CustomFcmThreadManager extends ThreadManager {

    @Override
    protected ExecutorService getExecutor(FirebaseApp firebaseApp) {
        return Executors.newFixedThreadPool(50);
    }

    @Override
    protected void releaseExecutor(FirebaseApp firebaseApp, ExecutorService executorService) {
        executorService.shutdownNow();
    }

    @Override
    protected ThreadFactory getThreadFactory() {
        return Executors.defaultThreadFactory();
    }
}


~~~


~~~java
FirebaseOptions options = new FirebaseOptions.Builder()
                .setCredentials(GoogleCredentials.fromStream(stream))
                .setThreadManager(new CustomFcmThreadManager())
                .build();

~~~
