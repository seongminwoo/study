# 개요

[Chris Richardson](http://microservices.io/)이 포스팅하고 있는 "a 7-part series about designing, building, and deploying microservices." 시리즈 글들(총 7개 예정)을 요약해본다.(2016-03-15 기준 총 6개글 연재.)

# 1. Introduction to Microservices
원본 글 : https://www.nginx.com/blog/introduction-to-microservices/

## Building Monolithic Applications
ex) Uber같은 taxi-haling app. [hexagonal architecture](http://www.infoq.com/news/2014/10/exploring-hexagonal-architecture) = Ports and Adapters architecture.
![](https://www.nginx.com/wp-content/uploads/2015/05/Graph-01-e1431978090737.png)
단일체(monolith)이기 때문에 개발, 테스트, 패키징, 배포가 용이하다.

## Marching Towards Monolithic Hell
애플리케이션이 거대해질 경우 심플한 Monolithic 구조에서는 아래의 문제가 발생한다.

* 생산성 - 수천개의 jars와 수백만 라인의 코드로 이루어진 beast 애플리케이션. Monolithic Hell. 복잡도가 증가하여 개발조직의 고통 시작, 배포시간도 오래걸림(수십분), 간단한 기능 추가에도 오래걸리는 배포 및 테스트로 인한 생산성 저하. CI에 역행. agile개발과 빠른 delivery의 어려움.
* scale - CPU-intensive한 모듈과 in-memory 모듈이 함께 사용되는 경우처럼 리소스 요구사항이 상충될 때 확장성에 제약.
* reliability - 앱이 하나의 프로세스로 돌기 때문에 한 모듈의 버그가 전면 장애로 이어짐.
* adopt new tech speed - 새로운 언어, 프레임워크 기술 적용이 극도로 어려움.
* hiring - 오래되고 비생산적인 기술 사용. 능력있는 인재 채용이 어려움.

## Microservices – Tackling the Complexity
* Amazon, eBay, Netflix 등은 위 문제를 [MSA](http://microservices.io/patterns/microservices.html) 패턴을 도입해서 해결. 즉 monolithic 애플리케이션을 잘게 쪼개서 작은 서로 연결되는 서비스들로 구성.
* 각 microservice는 독립적인 mini application.
![](https://www.nginx.com/wp-content/uploads/2015/05/Graph-031-e1431992337817.png)

* MSA 패턴은 Sacle Cube에서 Y축, 즉 different things를 분리해서 scale을 높이는 개념.
![](https://www.nginx.com/wp-content/uploads/2015/05/Graph-05-e1431978489626.png)

* microservice마다 독립적인 database 스키마. data 중복의 단점은 있지만, 커플링은 낮추고 각 서비스에 효율적인 DB 사용이 가능해짐. ex) Driver Management 서비스는 Driver의 위치를 찾는데 효율적인 geo-queries 지원 DB 사용.
![](https://www.nginx.com/wp-content/uploads/2015/05/Graph-04-e1431978528901.png)

* MSA VS SOA.
 * WSDL,SOAP vs REST
 * ESB vs APIGATEWAY
 * heavy vs simple, lightweight

## The Benefits of Microservices
* 분리된 각 서비스는 개발이 빠르고 유지보수 및 이해하기 쉽다.
 * decompose complexity. 알고리즘으로 치면 divide and conquor 느낌. 분리된 서비스를 연결하는 RPC or message-driven API가 중요해짐.
* 각 서비스에 집중하는 팀 구성 가능
 * 각 팀에서 자유롭게 기술을 선택할 수 있고 mini 서비스여서 기술 변경이 쉽다.
 * 요구사항을 구현하기위해 최적화된 언어와 아키텍처의 선택이 가능해진다.
* CI의 현실화, 고속개발.
 * 각 서비스가 독립적으로 테스트, 배포되기 때문에 빠르고 좋은 품질 유지 가능.
* 확장
 * 서비스 별로 독립적으로 확장 가능하고, 리소스 요구사항에 최적화된 장비 선택이 가능하다.

## The Drawbacks of Microservices
은탄환은 없다.

* distributed system. 
 * messaging or RPC. 부분실패(partial failure) 처리의 어려움, 롤백하기가 쉽지 않다.
* partitioned database architecture
 * db 분산 트랜잭션 처리가 일상. NoSQL, messaging brokers에서는 strong consistency 만족시키기 어려움. eventual consistency.
* 테스트
* 한 서비스의 변경이 여러 서비스에 영향을 줌.
 * A -> B -> C depenency.
* 배포
 * Hailo : 160, Netflix : 600 different services, 수많은 서비스 배포하려면 service discovery, 배포 자동화 도구들 필요. DevOps. ex) PaaS 서비스 이용 or 직접 구축(Mesos나 Kubernetes 이용.)

* 팀간의 의사소통 어려움
* 메모리 소비 증가. NxM services runs in its own JVM. own VM.

# 2. Building Microservices: Using an API Gateway
원본 글 : https://www.nginx.com/blog/building-microservices-using-an-api-gateway/

## Introduction
* 클라이언트가 접근하는 endpoint가 Monolithic 구조에서는 하나였다면 MSA 구조에서는 여러개 존재한다.
![](https://www.nginx.com/wp-content/uploads/2015/06/Amazonapp.png)
* single REST call vs many REST call
![](https://www.nginx.com/wp-content/uploads/2015/06/Graph-08.png)

## Direct Client-to-Microservice Communication
MSA 구조에서는 클라이언트가 접근하는 endpoint가 여러개 존재하고 아래와 같은 단점들로 인해 직접 호출하지 않는게 일반적인다.

* public internet, mobile network에서 많은 요청은 비효율적.
* 클라이언트 코드가 복잡해지고 리팩토링이 어려움.
* microservices 에서 Thrift binary RPC, AMQP messaging protocol을 사용할 경우. 둘다 browser, 방화벽에 친화적이지 않고 내부 네트워크에서의 사용에 적합. 방확벽 밖에서는 HTTP나 WebSocket 프로토콜이 적합.

## Using an API Gateway
* single entry point. OOP에서 Facade 패턴과 유사.
* 내부 시스템 아키텍쳐를 캡슐화해서 클라이언트에 적합한 API를제공.
* request routing, composition, and protocol translation. authentication, monitoring, load balancing, caching, request shaping and management, and static response handling.
* Netflix API Gateway. 수십억 요청 처리.
![](https://www.nginx.com/wp-content/uploads/2015/06/Graph-09.png)

## Benefits and Drawbacks of an API Gateway
### Benefits
* single endpoint(수많은 내부 서버 구조를 숨김). 클라이언트와 서버사이의 round trip 횟수 감소. 클라이언트 코드가 단순해짐.

### Drawbacks
* 관리 포인트. API Gateway가 개발 병목이 될 수 있다. API 탑재 프로세스(update)가 가벼워야 한다.

## Implementing an API Gateway
API Gateway 구현시 아래 디자인 이슈를 고려 해야한다.

* Performance and Scalability
 * 성능을 위해 API Gateway는 비동기, 논블로킹 I/O를 지원하는 플랫폼으로 개발 필요. ex) Netty, Vertx, Spring Reactor, or JBoss Undertow, Node.js, NGINX

* Using a Reactive Programming Model
 * simple yet efficient API Gateway code.

* Service Invocation
 * microservice가 사용하는 다양한 통신 메커니즘 지원 필요. ex) asynchronous, messaging-based mechanism. JMS or AMQP, Zeromq. synchronous mechanism such as HTTP or Thrift.

* Service Discovery
 * autoscaling and upgrades에 따른 service 주소가 dynamic하게 바뀌는 환경에서는 필수. Server-Side Discovery or Client-Side Discovery.

* Handling Partial Failures
 * 백엔드 서비스가 실패 하더라도 (잘 바뀌지 않는 데이터라면)캐시 데이터 or 디폴트 데이터로 실패 처리 가능. cf) Netflix Hystrix


# 3. Building Microservices: Inter-Process Communication in a Microservices Architecture
원본 글 : https://www.nginx.com/blog/building-microservices-inter-process-communication/

## Introduction
language method call VS IPC(inter-process communication)
![](https://www.nginx.com/wp-content/uploads/2015/07/Richardson-microservices-part3-monolith-vs-microservices.png)

## Interaction Styles
* One-to-one – Each client request is processed by exactly one service instance.
* One-to-many – Each request is processed by multiple service instances.
* Synchronous – The client expects a timely response from the service and might even block while it waits.
* Asynchronous – The client doesn’t block while waiting for a response, and the response, if any, isn’t necessarily sent immediately.

MSA 각 서비스 컴퍼넌트들은 아래 다양한 IPC 방식의 조합을 사용하게 된다.

| | One-to-One | One-to-Many |
| ------------- |:-------------:| -----:|
| Synchronous | Request/response | — |
| Asynchronous | Notification, Request/async response | Publish/subscribe, Publish/async responses |

ex) taxi-hailing(우버) app.
![](https://www.nginx.com/wp-content/uploads/2015/07/Richardson-microservices-part3-taxi-service-1024x609.png)

## Defining APIs
* 서비스와 클라이언트간의 계약.
* [API-first approach](http://www.programmableweb.com/news/how-to-design-great-apis-api-first-design-and-raml/how-to/2015/07/10)
* IPC 메커니즘에 의존. ex) HTTP : URL, request and response formats.


## Evolving APIs
* [Robustness principle.](https://en.wikipedia.org/wiki/Robustness_principle)
 - 서비스는 누락된 요청 속성에 대한 기본 값을 제공하고, 클라이언트는 별도로 추가된 응답 속성을 무시한다.
* API 변경이 쉬운 IPC 메커니즘 및 메시지 포맷 사용. 
* API의 대격변에 대비해서 versioning 하는게 좋음.

## Handling Partial Failure
![](https://www.nginx.com/wp-content/uploads/2015/07/Richardson-microservices-part3-threads-blocked-1024x383.png)
* [Netflix Hystrix](https://github.com/Netflix/Hystrix)
* [described by Netflix](http://techblog.netflix.com/2012/02/fault-tolerance-in-high-volume.html)
 * Network timeouts – Never block indefinitely.
 * Limiting the number of outstanding requests – Throttling.
 * Circuit breaker pattern.
 * Provide fallbacks.


## IPC Technologies
### Asynchronous, Message-Based Communication
![](https://www.nginx.com/wp-content/uploads/2015/07/Richardson-microservices-part3-pub-sub-channels-1024x639.png)
RabbitMQ, Apache Kafka,Apache ActiveMQ, and NSQ.

#### 장점
* Decouples the client from the service – The client is completely unaware of the service instances.
* Message buffering – a message broker queues up the messages written to a channel until they can be processed by the consumer.
* Flexible client-service interactions – Messaging supports all of the interaction styles described earlier.
* Explicit inter-process communication. vs RPC

#### 단점
* Additional operational complexity.
* Complexity of implementing request/response-based interaction – The client uses the correlation ID to match the response with the request.

### Synchronous, Request/Response IPC
#### HTTP REST
![](https://www.nginx.com/wp-content/uploads/2015/07/Richardson-microservices-part3-rest-1024x397.png)
[4단계의 REST 모델, maturity model for REST.](http://martinfowler.com/articles/richardsonMaturityModel.html)
IDL tools : RAML and Swagger.

##### 장점
* HTTP is simple and familiar.
* You can test an HTTP API from within a browser using an extension such as Postman or from the command line using curl (assuming JSON or some other text format is used).
* It directly supports request/response-style communication.
* HTTP is, of course, firewall-friendly.
* It doesn’t require an intermediate broker, which simplifies the system’s architecture.

##### 단점
* It only directly supports the request/response style of interaction. You can use HTTP for notifications but the server must always send an HTTP response.
* Because the client and service communicate directly (without an intermediary to buffer messages), they must both be running for the duration of the exchange.
* The client must know the location (i.e., the URL) of each service instance.


#### Thrift
* a framework for writing cross-language RPCclients and servers.
* Thrift provides a C-style IDL for defining your APIs. You use the Thrift compiler to generate client-side stubs and server-side skeletons. The compiler generates code for a variety of languages including C++, Java, Python, PHP, Ruby, Erlang, and Node.js.
* supports JSON, binary, and compact binary message formats. transport protocols including raw TCP and HTTP.

### Message Formats
* JSON and XML text message formats VS binary format.
* [comparison of Thrift, Protocol Buffers, and Avro.](http://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)

# 4. Service Discovery in a Microservices Architecture
원본 글 : https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/

## Why Use Service Discovery?
cloud-based microservices application - changes dynamically because of auto-scaling, failures, and upgrades.
![](https://www.nginx.com/wp-content/uploads/2015/10/theproblemofdiscovery-1005x1024.png)

### The Client-Side Discovery Pattern
![](https://www.nginx.com/wp-content/uploads/2015/10/pattern_clientside-1024x967.png)
Netflix OSS( Netflix Eureka,  Netflix Ribbon)

#### 장점
client knows about the available services instances, it can make intelligent, application-specific load-balancing decisions such as using hashing consistently.

#### 단점
couples the client with the service registry.  You must implement client-side service discovery logic for each programming language and framework used by your service clients.

### The Server-Side Discovery Pattern
![](https://www.nginx.com/wp-content/uploads/2015/10/pattern_serverside-1024x631.png)
AWS Elastic Load Balancer (ELB)
* load balance external traffic from the Internet.
* internal to a virtual private cloud (VPC)

NGINX(with Consul, Consul Template)

Kubernetes, Marathon - proxy 모듈

#### 장점
details of discovery are abstracted away from the client

#### 단점
highly available system component that need to set up and manage.

## The Service Registry
서비스 인스턴스의 네트워크 위치 정보를 저장하고있는 데이터베이스.

* Netflix Eureka
* etcd
* consul
* Apache Zookeeper

Kubernetes, Marathon, and AWS 같은 경우 별도의 service registry가 존재하지는 않고 인프라의 일부로 포함되어 있다.


## Service Registration Options
### The Self-Registration Pattern
![](https://www.nginx.com/wp-content/uploads/2015/10/pattern_selfregistration-1024x893.png)
Netflix OSS Eureka client. Spring Cloud project.(@EnableEurekaClient annotation)

#### 장점
relatively simple and doesn’t require any other system components.

#### 단점
couples the service instances to the service registry. You must implement the registration code in each programming language and framework used by your services.

### The Third-Party Registration Pattern
![](https://www.nginx.com/wp-content/uploads/2015/10/pattern_thirdparties-1024x593.png)
Registrator - etcd or Consul with docker

NetflixOSS Prana - Netflix Eureka.

#### 장점
services are decoupled from the service registry. You don’t need to implement service-registration logic for each programming language and framework used by your developers.

#### 단점
it is yet another highly available system component that you need to set up and manage.


# 5. Event-Driven Data Management for Microservices
원본 글 : https://www.nginx.com/blog/event-driven-data-management-microservices/

## Microservices and the Problem of Distributed Data Management
* MSA구조에서는 각 마이크로서비스마다 datastore를 가진다. 이러한 각 datastore에 대한 접근은 API를 통해서 이루어지고 따라서 여러 마이크로서비스간의 비지니스 트랜잭션 처리가 어려워진다.
 -  consistency across multilpe services is not easy
 -  query across multilpe services is not easy

![](https://assets.wp.nginx.com/wp-content/uploads/2015/12/Richardson-microservices-part5-separate-tables-e1449727641793.png)


## Event-Driven Architecture
![](https://assets.wp.nginx.com/wp-content/uploads/2015/12/Richardson-microservices-part5-subscribe-e1449727516992.png)

#### 장점
* 여러 서비스 간에 eventaul consistency 충족시킬 수 있다.

#### 단점
* ACID transaction 사용할 때 보다 프로그래밍 모델이 복잡해진다.
* 애플리케이션에서 fail 발생시 fail에 대한 보상처리를 해줘야 한다.
* isolation이  보장되지 않기 때문에 inconsistent한 데이터가 보일수 있음을 고려해야한다.
* 중복 event 발생을 감지해서 무시할 수 있어야한다.

- https://en.wikipedia.org/wiki/Eventual_consistency
- http://queue.acm.org/detail.cfm?id=1394128

## Achieving Atomicity
### Publishing Events Using Local Transactions
2PC(Phase Commit)가 필요없음.
![](https://assets.wp.nginx.com/wp-content/uploads/2015/12/Richardson-microservices-part5-local-transaction-e1449727484579.png)

### Mining a Database Transaction Log
2PC(Phase Commit)가 필요없음. 이벤트 테이블도 필요없음. The Transaction Log Miner thread가 직접 트랜잭션로그 후킹.
![](https://assets.wp.nginx.com/wp-content/uploads/2015/12/Richardson-microservices-part5-transaction-log-e1449727434678.png)

### Using Event Sourcing
2PC(Phase Commit)가 필요없음. 심지어 DB updates 없이 오직 evnets만을 사용해서 The Transaction Log Miner thread의 단점을 제거할수 있다. 또한 RDBMS와 ORM 사용시 발생하는 패러다임불일치 문제를 피할수 있는 장점이 있다. 다만 프로그래밍 모델이 복잡해서 러닝커브가 있고 CQRS 기법 사용을 강제하는 단점이 있다.

![](https://assets.wp.nginx.com/wp-content/uploads/2015/12/Richardson-microservices-part5-event-sourcing-e1449711558668.png?_ga=1.162065276.1943494022.1449809930)

- https://github.com/cer/event-sourcing-examples/wiki/WhyEventSourcing
- https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch
- https://github.com/cer/event-sourcing-examples/wiki


# 6. Choosing a Microservices Deployment Strategy
원본 글 : https://www.nginx.com/blog/deploying-microservices/

## Motivations
microservices 애플리케이션은 10, 100여개의 서비스들로 구성된다.
각 서비스는 적절한 CPU, memory, I/O 리소스를 제공받아야 하고 배포는 빠르고 안정적 효율적이어야한다.

## Multiple Service Instances per Host Pattern
http://microservices.io/patterns/deployment/multiple-services-per-host.html

![](https://assets.wp.nginx.com/wp-content/uploads/2016/02/Richardson-microservices-part6-host.png)
##### 장점
* 자원 사용이 효율적이다.
* 상대적으로 배포 속도가 빠르다. 서비스 시작속도도 빠르다.

##### 단점
* 서비스 인스턴스가 동일한 프로세스에서 실행 될 경우 리소스 isolation 하기가 어렵다. cpu, 메모리를 한 서비스 인스턴스가 모두 사용하는 문제가 발생가능하다.
* 운영팀(operations team)이 deploy하는 서비스들의 detail(언어, 프레임워크)을 잘 알아야 되고, 이는 배포 장애로 이어지는 위험요소가 된다.


## Service Instance per Host Pattern
http://microservices.io/patterns/deployment/single-service-per-host.html

### Service Instance per Virtual Machine Pattern
http://microservices.io/patterns/deployment/service-per-vm.html
https://aws.amazon.com/ko/ec2/

![](https://assets.wp.nginx.com/wp-content/uploads/2016/02/Richardson-microservices-part6-vm.png)

Netfilx에서는 video streaming 서비스에 VM을 사용하는데 각 서비스를 Aminator를 사용해서 EC2 AMI(VM 이미지)로 패키지해서 EC2 인스턴스로 배포하고 있다.

##### 장점
* 서비스마다 완벽한 isolation이 가능하다. 고정된 CPU와 메모리를 사용하고 다른 서비스에서 steal하지 않는다.
* 성숙한 클라우드 인프라의 다양한 기능들 활용이 가능하다. ex) AWS load balancing and autoscaling.
* 서비스 구현 기술과 관계없이 VM으로 캡슐화(black box) 가능하다. 이는 VM 관리 API로 단순화 추상화 되어서 쉽고 안정적인 배포를 가능하게한다.

##### 단점
* 자원 사용이 비효율적이다. VM에 할당된 자원이 낭비된다.
* 보통 public IaaS들은 VM이 미사용중이더라도 과금한다.
http://techblog.netflix.com/2013/11/scryer-netflixs-predictive-auto-scaling.html
* VM 이미지 빌드 및 실행이 오래 걸린다. 단 Boxfuse 같이 lightweight VM을 통한 해법이 존재한다.
* VM 관리가 어렵다.

### Service Instance per Container Pattern
http://microservices.io/patterns/deployment/service-per-container.html
https://en.wikipedia.org/wiki/Operating-system-level_virtualization

#### container technologies
* docker
* solaris zones

#### cluster manager
* kubernetes
* marathon

![](https://assets.wp.nginx.com/wp-content/uploads/2016/02/Richardson-microservices-part6-container.png)

##### 장점
* VM의 장점과 대부분 유사.
* VM보다 가볍기 때문에 image 빌드 및 배포가 무척 빠름.

##### 단점
* containers 인프라는 VM 인프라만큼 아직 성숙하지 못함.
* 다른 container와 커널을 공유하기 때문에 VM 만큼 안전(secure)하지 못함
* GCE(Google Container Engine)나 ECS(Amazon EC2 Container Service) 같은 솔루션을 사용하지 않을 경우 직접 인프라를 관리해야함.

사실 요즘엔 VM과 container 간의 구분이 흐릿해지고 있다.(Boxfuse VMs are fast to build and start. The Clear Containers project aims to create lightweight VMs. There is also growing interest in unikernels. Docker, Inc recently acquired Unikernel Systems.)


## Serverless Deployment(AWS Lambda)
https://aws.amazon.com/lambda/

* convenient way to deploy microservices.
* automatically runs enough instances of your microservice to handle requests.
* The request-based pricing.

#### 제약사항
* not intended to long-running services
* Requests must complete within 300 seconds.
* stateless
* Services must also start quickly; otherwise, they might be timed out and terminated.

#### Lambda 호출 4가지 방식
* Directly, using a web service request
* Automatically, in response to an event generated by an AWS service such as S3, DynamoDB, Kinesis, or Simple Email Service
* Automatically, via an AWS API Gateway to handle HTTP requests from clients of the application
* Periodically, according to a cron-like schedule


# 6. Refactoring a Monolith into Microservices
원본 글 : https://www.nginx.com/blog/refactoring-a-monolith-into-microservices/

monolithic 애플리케이션에서 microservices로 점진적으로 migration, refactoring 하는 전략 소개.

## Overview of Refactoring to Microservices
* [application modernization](https://en.wikipedia.org/wiki/Software_modernization)
* "Big Bang” rewrite는 위험하니점진적으로 refactoring 하는 전략을 취해야한다.
* [Strangler Application](http://www.martinfowler.com/bliki/StranglerApplication.html)

## Strategy 1 – Stop Digging
신규 기능을 추가해야 할 경우 기존 MONOLITH가 아닌 별도의 신규 SERVICE를 만들어서 연동하라.
![](https://assets.wp.nginx.com/wp-content/uploads/2016/03/Adding_a_secure_microservice_alongside_a_monolithic_application.png)

신규 SERVICE가 MONOLITH 데이터에 접근하는 3가지 전략으로는 다음과 같다.
* remote API 호출
* database 직접 접근
* database copy(동기화) 

glue code = anti-corruption layer(Eric Evans의 Domain Driven Design 책에서 소개된 패턴 용어.)
SERVICE로 하여금 MONOLITH와 독립적인 domain 모델 사용가능하게 한다.

MONOLITH의 문제점을 근본적으로 해결하는 방법은 아니다 MONOLITH를 break up 해야되는데 다음에서 그 전략을 설명한다.

## Strategy 2 – Split Frontend and Backend
![](https://assets.wp.nginx.com/wp-content/uploads/2016/04/Richardson-microservices-part7-refactoring.png)

Frontend와 Backend를 둘로 나눌 경우 아래 2가지 장점이 있지만 역시나 부분적인 솔루션이다.

* 개발, 배포, 확장이 서로 독립적이다. 특히 Frontend 개발자는 빠르게 UI 작업 반복이 가능하고 A/B 테스팅을 쉽게 수행 가능해진다.
* 자연스럽게 remote API 서버(Backend)가 생성된다.


## Strategy 3 – Extract Services
![](https://assets.wp.nginx.com/wp-content/uploads/2016/04/Richardson-microservices-part7-extract-module.png)

추출(extraction) 전략은 아직 microserivce 경험이 없을 초반에는 추출하기 쉬운 모듈 먼저 진행 하고 그 후에는 이득이 되는 모듈에 우선순위를 두면된다. 이득이 되는 모듈이란 변경이 자주 발생하는 모듈이다. 추출하고 나면 개발, 배포가 독립적으로 이루어질 수 있어서 개발을 가속화 시키기 때문이다. 혹은 자원(CPU, 메모리) 요구사항이 특이한 케이스의 모듈을 대상으로 하는것도 좋다.

* a pair of coarse-grained APIs 정의하기(inbound, outbound interfaces)
* 모듈을 독립적인 서비스로 전환(IPC 사용). [Microservice Chassis framework](http://microservices.io/patterns/microservice-chassis.html) 사용.

## Summary
* monolithic 애플리케이션에서 microservices로 전환은 한번에 할수는 없고 점진적으로 진행햐하는데,  3가지 전략이 있다.(new functionality as microservices, split Frontend and Backend, convert modules into services)
* microservices가 증가함에 따라 개발팀의 기민함(agility)과 속도(velocity)가 증가할 것이다.
