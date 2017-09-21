## Java 9 interpreter(REPL, Read Evaluate Print Loop)
드디어 자바에도 REPL(레플)이!!
프로젝트 생성없이 빠르게 코드 작성해서 테스트 및 실험 가능한 장점!!(quick feedback loop) 노클래스! 노메소드! 노컴파일!

## java9 설치 및 jenv로 멀티 자바 버전 관리
```
brew, cask 업데이트
$ brew update && brew upgrade brew-cask && brew cleanup && brew cask cleanup          

java 최신버전 설치하기 
$ brew cask install java

1.9-beta 설치하기
$ brew cask install java9-beta

jenv 사용해서 java version 관리하기
$ brew install jenv
$ echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(jenv init -)"' >> ~/.bash_profile
$ /usr/libexec/java_home -V
$ jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home
Or 
$ jenny add $(/usr/libexec/java_home -v1.8)
$ jenv add /Library/Java/JavaVirtualMachines/jdk-9.jdk/Contents/Home
Or
$ jenny add $(/usr/libexec/java_home -v9)
$ jenv versions
$ jenv global oracle64-1.8.0.92
Or
$ jenv local oracle64-1.8.0.92
```

### reference
* http://www.jenv.be/
* http://davidcai.github.io/blog/posts/install-multiple-jdk-on-mac/
* https://crazysalaryman.wordpress.com/2015/03/14/jenv-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/

## 기억해둘만한 내용
[Exploring Java 9 by Venkat Subramaniam](https://www.youtube.com/watch?v=8XmYT89fBKg)
<img width="854" alt="2017-06-14 2 21 20" src="https://user-images.githubusercontent.com/1201462/27116547-e3f1cdca-510c-11e7-93ba-ffc07880bea8.png">

<img width="479" alt="2017-06-14 10 28 28" src="https://user-images.githubusercontent.com/1201462/27111972-dc56b9e0-50ee-11e7-95a7-0043c32884e4.png">

<img width="523" alt="2017-06-14 10 37 46" src="https://user-images.githubusercontent.com/1201462/27111983-e9c7af30-50ee-11e7-8b5f-3845729e9ec3.png">

<img width="405" alt="2017-06-14 10 38 39" src="https://user-images.githubusercontent.com/1201462/27111982-e9c2fdd2-50ee-11e7-98a3-08734e680454.png">

<img width="579" alt="2017-09-21 10 17 52" src="https://user-images.githubusercontent.com/1201462/30674595-4950e894-9eb6-11e7-8e59-9905dff7afb3.png">

<img width="482" alt="2017-06-14 10 42 50" src="https://user-images.githubusercontent.com/1201462/27111989-f29804f2-50ee-11e7-8acd-3010460c8b87.png">

<img width="445" alt="2017-06-14 10 44 37" src="https://user-images.githubusercontent.com/1201462/27111992-f2b9c880-50ee-11e7-8671-aaba8d81a20c.png">

<img width="454" alt="2017-06-14 10 44 52" src="https://user-images.githubusercontent.com/1201462/27111991-f2b8a054-50ee-11e7-9b95-b19180aafb0a.png">


## reference
* http://jakubdziworski.github.io/java/2016/07/31/jshell-getting-started-examples.html
* https://www.youtube.com/watch?v=8XmYT89fBKg
