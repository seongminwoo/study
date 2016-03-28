## 개요
nginx cache_lock 기능을 Global API-GATEWAY 프로젝트(java)에 구현하는 과정에서 고민하고 공부한 내용들을 남긴다.

## 고민
lock 동기화 매커니즘 어떤걸 사용해야하나? 아래 몇가지 동기화 매커니즘들 중 readLock, writeLock 패턴이 가장 적합할거라 생각함.
nginx cache lock 구현코드를 보면 lock 관련 코드가 내부에 숨겨져 있어서 매커니즘 확인이 어려웠음.
EHCache에서 제공하는 BlockingCache 구현체 코드를 보면 ReentrantReadWriteLock 사용.

* Guarded Suspension 패턴(or Guarded timed)
* 세마포어
* readLock, writeLock
* CountDownLatch
* pub-sub 패턴

nginx cache lock 구현 코드 : https://github.com/nginx/nginx/blob/645697f111983089fdcee0694d17480e0a05a3a5/src/http/ngx_http_file_cache.c

## 구현체들
* 스프링 cache 추상화.
동기화 요구사항이 계속 있었음. Spring 4.3에서 sync=true 속성으로 지원.
https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3#cache-abstraction-refinements
```
@Cacheable(cacheNames = "foos", sync = true)
public Foo getFoo(String id) { ... }
```
스프링 커밋코드 : https://github.com/spring-projects/spring-framework/commit/19d97c425316801a767cf99178ef30af730b1570

* EHCache
BlockingCache 구현체로 cachelock 기능 제공.
http://www.ehcache.org/documentation/2.8/apis/cache-decorators
코드 : http://grepcode.com/file/repo1.maven.org/maven2/net.sf.ehcache/ehcache/2.10.0/net/sf/ehcache/constructs/blocking/BlockingCache.java#BlockingCache

## 문제점
1. 프로젝트에서 Spring을 사용하지 않는다. cache 미존재시 put하는 로직이 복잡해서 Spring cache 추상화에서 제공하는 인터페이스로 던지기가 애매하다.
2. BlockingCache 구현체는 writeLock에 대한 lock은 get메소드에서 unlock은 put메소드에서 수행하게 되는데 이로 인해 혹여나 애플리케이션에서 put메소드 수행을 하지 않을 경우 lock이 unlock되지 않는 문제가 존재한다.
즉 lock 소유와 해제가 내부적으로 감춰져 있지 않고 애플리케이션으로 침투하게 되어 문제가 된다.
3. BlockingCache는 EHCache에 의존성을 가지고 있어서 redis 등에 cachelock 기능을 사용할수가 없다.

## 해결책
BlockingCache 코드를 기본으로, 아래 스프링 코드처럼 get메소드 내에서 cache 미존재시 put까지 수행하고 writeLock을 해제하도록 코드 작성함.
스프링 참고 코드
```
@SuppressWarnings("unchecked")
@Override
public <T> T get(Object key, Callable<T> valueLoader) {
	Element element = lookup(key);
	if (element != null) {
		return (T) element.getObjectValue();
	}
	else {
		this.cache.acquireWriteLockOnKey(key);
		try {
			element = lookup(key); // One more attempt with the write lock
			if (element != null) {
				return (T) element.getObjectValue();
			}
			else {
				return loadValue(key, valueLoader);
			}
		}
		finally {
			this.cache.releaseWriteLockOnKey(key);
		}
	}

}
```

