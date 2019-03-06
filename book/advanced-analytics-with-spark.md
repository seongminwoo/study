### Chapter3. 음악 추천과 Audioscrobbler 데이터셋
#### 챕터 목적
음악 추천 모델 만들기 실습을 통해 스파크 MLlib를 적용해보고, 머신러닝 알고리즘의 기본을 이해한다.

#### 3.1 데이터셋
* [오디오스크로블러(Audioscrobbler)](https://ko.wikipedia.org/wiki/%EB%9D%BC%EC%8A%A4%ED%8A%B8_FM)에서 공개한 [데이터셋](https://storage.googleapis.com/aas-data-sets/profiledata_06-May-2005.tar.gz)
* 오디오스크로블러는 인터넷 스트리밍 라디오 서비스를 제공한 [last.fm](https://www.last.fm/)의 첫번째 음악 추천 시스템
* "개똥이가 소녀시대 음악을 들었어"와 같은 단순 재생 정보만을 기록한 데이터셋
* 명시적인 '좋아요', '평점'이 아닌 음악 듣기(재생)를 통해 은연중에 사용자와 아티스트 사이의 관계가 드러나기 때문에 이런 종류의 데이터를 암묵적 피드백(Implicit Feedback)이라고 함
* 데이터셋 구성
  - user_artist_data.txt : 2420만건의 음악 재생정보(사용자ID,아티스트ID,재생횟수)
  - artist_data.txt : 184만건의 아티스트 정보(아티스트ID,아티스트 이름)
  - artist_alias.txt : 19만건의 공식 아티스트ID 매핑정보(아티스트ID,아티스트대표ID)

#### 3.2 교차 최소 제곱 추천 알고리즘(The Alternating Least Squares Recommender Algorithm)
* 암묵적 피드백(Implicit Feedback) 데이터에 적합한 추천 알고리즘은?
* 협업필터링(Collaborative Filtering) - A, B 두 사람이 재생한 노래가 유사하면 A의 노래를 B에게 추천.
* [잠재요인(Latent-factor) 모델](https://en.wikipedia.org/wiki/Factor_analysis)로 분류할 수 있는 많은 알고리즘 중 하나를 사용하고자함.
* 잠재요인 모델 : 다수의 사용자와 아이템 사이에서 관측된 상호작용을 상대적으로 적은 수의 관측되지 않은 숨은 원인으로(잠재특징) 설명하려 할 때 사용. => 특정 음반을 구입한 이유를 (직접 관측할 수 없고 데이터도 주어지지 않은) 음악 장르에 대한 개인 취향으로 설명하는것과 유사.
* recommendation problem => maxtrix completion => matrix factorization
* 행렬 분해(Matrix Factorization) 모델 사용.
  - 사용자가(i)가 아티스트(j)의 음악을 들었다면 A행렬의 i행 j열에 값이 존재한다고 표현
  - 사용자 - 아티스트의 가능한 모든 조합중 극히 일부만 데이터가 존재하기 때문에 A는 희소행렬(Sparse Matrix)
  - 행렬 A를 분해(행렬은 두 개의 낮은 차수로 이루어진 행렬로 분해가 가능하고, 이 분해된 행렬들끼리 다시 곱하면 원래의 행렬과 매우 유사한 단일 행렬이 되는 성질을 이용.)
  - A(i, j) = X(i, k) * Y(k, j)  k는 상호작용하는 데이터를 설명하는 잠재요인(잠재 feature. 행렬의 차수)
  - 사용자-아티스트 행렬 근사치 = 사용자-특징 행렬 * 특징-아티스트 행렬
![책 캡쳐](../images/3-1.jpg)
* 원래의 행렬 A는 매우 희소한 데 비해 행렬 곱 XY는 밀도가 매우 높아서 이 알고리즘을 행렬 채우기(Matrix Completion) 알고리즘이라고도 부름.
* X와 Y를 계산하기 위해 이장에서는 교차 최소 제곱 알고리즘(ALS, Alternating Least Squares)을 사용
  - ["암묵적 피드백 데이터셋에 대한 협업 필터링"](http://yifanhu.net/PUB/cf.pdf), ["넷플릭스 프라이즈를 위한 대규모의 병렬 협업 필터링"](https://dl.acm.org/citation.cfm?id=1424269) 논문에서 아이디어.
* 오차를 최소화하는 공식의 최적값을 구하기 위한 잘 알려진 알고리즘으로 딥러닝에서 많이 들어본 "확률적 경사하강법(SGD)", 교대최소제곱(ALS) 두가지 존재. 이중 ALS가 분산플랫폼에서 병렬화 용이([출처](http://rstatistics.tistory.com/29))
  - 주어진 objective가 pairwise optimization으로 생각하면 non-convex이지만, p나 q 중 하나를 고정하고 나머지에 대해 optimization을 하게 되면 convex, 그것도 closed form으로 계산된다는 점을 이용한 방법
* ALS 알고리즘 장점(스파크 MLlib에 구현된 유일한 추천 알고리즘)
  - 입력 데이터 희소성의 장점 살림
  - 간단하고 최적화된 선형대수 기법 사용
  - 병렬처리가 가능해서 데이터가 많아져도 빠르게 처리 가능
* 넷플릭스는 2006년 ‘넷플릭스 프라이즈’라는 기술 콘테스트를 개최. 넷플릭스의 추천 시스템인 ‘시네매치’의 품질을 10% 개선하는 사람에게 100만 달러(약 12억원) 상금. [출처](https://byline.network/2016/03/1-87/)

#### 실습 예제) 3.3 데이터 준비하기 ~ 3.10 한 걸음 더 나아가기

#### 볼만한 글들
* [추천 엔진에 이용되는 데이터 마이닝 기법](http://rstatistics.tistory.com/29)
* [Recommendation System (Matrix Completion)](http://sanghyukchun.github.io/73/)
* [recommendation system with Implicit Feedback](http://sanghyukchun.github.io/95/)
* [ch3 웹에 정리된 글](http://hyunje.com/data%20analysis/2015/07/13/advanced-analytics-with-spark-ch3-1/)
* [추천 방식 장단 비교 및 추천 알고리즘 구현하기](https://proinlab.com/archives/2103)
* [Spark ml Collaborative Filtering](https://spark.apache.org/docs/2.2.0/ml-collaborative-filtering.html)
