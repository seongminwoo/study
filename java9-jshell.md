## Java 9 interpreter(REPL)
드디어 자바에도 REPL(레플)이!!
프로젝트 생성없이 빠르게 코드 작성해서 테스트 및 실험 가능한 장점!! 노클래스! 노메소드! 노컴파일!

## java9 설치 및 jenv로 멀티 자바 버전 관리
```
brew, cask 업데이트
$ brew update && brew upgrade brew-cask && brew cleanup && brew cask cleanup          

java 최신버전 설치하기 
$ brew cask install java

1.9-beta 설치하기
$ brew cask install java9-beta

jenv 사용해서 java version 관리하기
http://www.jenv.be/
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
http://www.jenv.be/
http://davidcai.github.io/blog/posts/install-multiple-jdk-on-mac/
https://crazysalaryman.wordpress.com/2015/03/14/jenv-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/


## reference
http://jakubdziworski.github.io/java/2016/07/31/jshell-getting-started-examples.html
https://www.youtube.com/watch?v=8XmYT89fBKg