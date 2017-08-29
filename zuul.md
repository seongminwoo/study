## zuul
https://github.com/Netflix/zuul/wiki
https://medium.com/netflix-techblog/announcing-zuul-edge-service-in-the-cloud-ab3af5be08ee

### 개요
netflix cloud gateway. proxy에 중점을 둔 심플 api gateway라고 보면 된다. netfix에는 별도로 api gateway가 안쪽 레이어에 존재한다.
the front door to Netflix’s server infrastructure, handling traffic from all Netflix users around the world.

![image](https://cloud.githubusercontent.com/assets/1201462/26758208/92f9aa0c-4912-11e7-8fe5-648070b2b1f4.png)

### 아키텍쳐
Zuul in Netflix’s Cloud Architecture

![image](https://cdn-images-1.medium.com/max/2000/1*Cv4CCYNTlGnIkQ4VkP4pHg.png)

Zuul Core Architecture

![image](https://cdn-images-1.medium.com/max/2000/1*j9iGkeQ7bPK2nC1a7BgFOw.png)


Request Lifecycle

![image](https://cdn-images-1.medium.com/max/1600/1*9IeEGHSRMGfAnhqM49TLpQ.png)


Netflix OSS libraries in Zuul

![image](https://cdn-images-1.medium.com/max/1600/1*pz6sv69la9ek6yWNTPqymQ.png)

특정 고객 혹은 특정 디바이스에만 발생하는 문제를 디버깅하는 용도로 SurgicalDebugFilter를 사용하면 해당 고객 트래픽은 특정 인스턴스로만 가도록 해서 디버깅하기 편하다. 혹은 특정 페이지의 오류를 확인하거나 성능 튜닝을 할 때도 요긴하다.

## zuul2(비동기 버전)
* https://medium.com/netflix-techblog/zuul-2-the-netflix-journey-to-asynchronous-non-blocking-systems-45947377fb5c
* 한글 : http://chanwookpark.github.io/reactive/netflix/zuul/async/non-blocking/2016/11/21/zuul2-netflixjourney-to-asynchronous-non-blocking-systems/

### 개요
zuul2는 기존 zuul과 달리 Netty를 사용한 비동기, 논블로킹 프레임워크 기반으로 동작한다.

### Differences Between Blocking vs. Non-Blocking Systems

Multithreaded System Architecture

![image](https://cdn-images-1.medium.com/max/1200/0*kPzgZrACokyPJJfy.png)


Asynchronous and Non-blocking System Architecture

![image](https://cdn-images-1.medium.com/max/1200/0*jrG2ldEVRRJcgpkj.png)

백엔드 latency가 느려지거나 retry storms(문제 발생시 재시도 현상) 발생시 커넥션과 이벤트가 큐에 적체가 되지만 이는 기존 쓰레드 기반의 블로킹 시스템에서 쓰레드가 쌓이는 것보다 시스템에 스트레스가 적다.

### Building Non-Blocking Zuul
스레드 로컬 변수는 동일한 스레드에서 여러 요청이 처리되는 비동기-논블로킹에서는 동작하지 않는다. 결과적으로 Zuul2를 구현하는데 따른 복잡성의 상당 부분은 스레드 로컬 변수가 사용되는 로직 수정에 있다.

### Results of Zuul 2 in Production
* 비동기-논블로킹 사용으로 효율성을 크게 향상시킨건 아니지만 커넥션 확장(scaling)에서 이득을 취함.
* 네트워크 커넥션 비용을 확 낮추면서 디바이스와의 push, 양방향 통신이 가능해짐.
* 이를 통해 실시간 사용자 경험을 혁신 할 수 있으며, 푸시 알림을 통해 현재 API 트래픽의 상당 부분을 차지하는 (불필요한)API 요청을 대체함으로써 전체적인 클라우드 비용을 절감 할 수 있음.
* 백엔드 latency 및 retry storms 이슈를 해결하는데도 이점이 있어 resiliency(탄력성) 장점.
* CPU-bound 보다는 IO-bound에서 이점.
* CPU-bound 이더라도 기존 servlet 기반 zuul보다는 netty기반 아키텍쳐를 통해 25%의 throughput 향상(25% CPU 사용 감소).

## 기타 링크
* [배민 API GATEWAY - spring cloud zulu 적용기](http://woowabros.github.io/r&d/2017/06/13/apigateway.html)
* [zuul-netflix-springone-platform](https://www.slideshare.net/MikeyCohen1/zuul-netflix-springone-platform)
* [rethinking-cloud-proxies](https://www.slideshare.net/MikeyCohen1/rethinking-cloud-proxies-54923218)
