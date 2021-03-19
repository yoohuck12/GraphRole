# GraphRole

[![Build Status](https://travis-ci.com/dkaslovsky/GraphRole.svg?branch=master)](https://travis-ci.com/dkaslovsky/GraphRole)
[![Coverage Status](https://coveralls.io/repos/github/dkaslovsky/GraphRole/badge.svg?branch=master)](https://coveralls.io/github/dkaslovsky/GraphRole?branch=master)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/GraphRole)

그래프 전이 학습을 위한 자동 특색 추출과 노드 역할 할당
ReFeX/RolX 알고리즘을 기반으로 구현하였다.

<p align="center">
<img src="./examples/karate_graph.png" width=600>
</p>

### Overview
그래프 학습에서 가장 근본적인 문제는 의미있는 특색들을 추출해내는 것이다.
GraphRole 은 'RecursiveFeatureExtractor' 클래스를 제공해, 이 과정을 자동으로
할 수 있는 기능을 제공한다. 주어진 그래프에서 지역적인, 그리고 이웃들과의
구조적 속성을 찾아내고 재귀적으로 특색을 추출해낸다.
구체적인 구현은 ReFex 알고리즘[1]을 이용해냈다.
노드의 특색(degree) 과 ego-net 특색 (이웃, 내/외부의 edge 개수)들이 추출되고
추가적인 정보가 더이상 누적되지 않을때 까지 재귀적으로 각 노드의 이웃 특색들과
aggregate 된다. [1] 에서 보인것 처럼, 지역적인 특색은 node classification 에서
활용될 수 있으며, 전이 학습 작업들에서도 잘 동작한다.

GraphOle 은 또한 'RoleExtractor' 클래스를 제공해 노드의 역할 할당을 제공한다.
그래프에서 서로 다른 노드는 서로 다른 구조적인 역할을 하고, 재귀적인 지역
특색을 사용하여 이러한 역할들이 밝혀지고, 노드들의 집합들에 할당된다.
자연적인 구조이기 때문에 노드 커뮤니티에서 주로 사용되는 것과 다르게 좀 더
직관적이다. 세부적으로 역할은 그래프에서 일반화될 수 있지만, 커뮤니티에서는
그렇지 않다.[2] 노드의 역할을 알아내고 할당하는 것은 다양한 그래프 학습 작업들
에서 사용될 수 있다.

기술적인 세부사항은 [1,2] 에서 찾아봐라.

### Installation
이 패키지는 pip 을 사용해서 설치될 수 있다.
```
$ pip install graphrole
```
소스코드로 설치하고자 하면 다음을 사용한다.
```
$ git clone https://github.com/dkaslovsky/GraphRole.git
$ cd GraphRole
$ python setup.py install
```

### Example
GraphRole 의 사용 예제는 'example' 폴더에 있다. 노트북
[example.ipynb](./examples/example.ipynb)
(also available via [nbviewer](https://nbviewer.jupyter.org/github/dkaslovsky/GraphRole/blob/master/examples/example.ipynb))
은 karate_club_graph 을 사용해 특색 추출부터 역할 할당까지 수행한다.
이것은 networkx 에 포함되어 있다.
재귀적인 특색은 추출된 뒤 역할 할당 학습을 위해 사용되며, 그래프에서 각각의 노드에
대해 수행된다. 위의 그래프는 각각의 노드가 역할에 기반되어 색칠된 것을 보여준다.

추출된 역할은 그래프의 노드 레벨에서 구조적인 속성을 반영한다.
노드 '0', '33' (어두운 초록)는 그래프의 중심이며, 다른 많은 노드와 연결되어 있다.
노드 '1','2','3','32'(빨강) 은 비슷한 역할로 할당되었다.
반면에 어두운 파랑, 밝은 파랑, 핑크로 칠해진 노드는 그래프에서 중요하지 않은 것으로 보인다.
주목할만한 것은 같은 역할을 위해서 노드가 서로 연결되어 있을, 주변에 있을 필요가
없다는 것이다. 대신 비슷한 속성을 가진 노드들이 역할 할당에서 비슷한 역할을
할당받게 된다.

이 예제에는 나와있지 않지만, weighted 그리고 directed 그래프도 역시 사용될 수 있으며
여기서 추출된 특색들도 사용될 수 있다.

### Usage
일반적으로는 특색 및 역할 추출을 위한 2개의 클래스를 import 한다.
```
>>> from graphrole import RecursiveFeatureExtractor, RoleExtractor
```
특색은 그래프 'G' 로부터 추출되며, 'pandas.DataFrame' 형태이다.
```
>>> feature_extractor = RecursiveFeatureExtractor(G)
>>> features = feature_extractor.extract_features()
```
다음으로 특색들이 역할을 학습하기 위해서 사용된다.
'n_roles=None' 가 'RoleExtractor' 클래스에 전달되면 역할의 수는 자동으로 모델 선택 과정에서 정해진다.
대신에 'n_roles' 를 인자로 줄 수도 있다.
```
>>> role_extractor = RoleExtractor(n_roles=None)
>>> role_extractor.extract_role_factors(features)
```
각 노드에 대한 역할 할당은 dictionary 형태로 추출된다.
```
>>> role_extractor.roles
```
역할은 소프트 할당으로 보여질 수도 있으며, 노드의 멤버쉽 퍼센트는 'pandas.DataFrame'
형태로 추출될 수 있다.
```
>>> role_extractor.role_percentage
```

### Graph Interfaces
그래프 데이터 구조를 위한 인터페이스는 'graphole.graph.interface' 모듈로서
제공된다. 'networkx', 'igraph' 를 위한 구현이 포함된다.

'igraph' 패키지는 'requirements.txt' 에 들어있지는 않기 때문에 필요하다면
손으로 설치해야 한다. 이것은 단순히 `pip install python-igraph` 외에도 더
설치할게 필요하기 때문이다. 자세한것은
[igraph documentation](https://igraph.org/python/#pyinstall) 를 보자.
만약 igraph 가 설치되어 있지 않다면 igraph 를 위한 테스트는 스킵된다.

다른 그래프 라이브러리나 데이터 구조의 구현들을 넣고 싶다면,
1. 'BaseGraphInterface' ABC 를 subclass 화 하고, 필요한 함수를 구현해라.
  1. graphrole.graph.interface.base.py 를 보자
1. graphrole.graph.interface.__init__.py 의 'INTERFACES' dict 을 업데이트하여
새로운 서브 클래스를 찾을 수 있도록 하자.
1. 'setUpClass()' 클래스 함수를 간단하게 구현하여 테스트를 추가한다.
1. 필요하다면 추가된 인터페이스를 활용해서 특색 추출 테스트를 하라.

### Future Development
Model explanation ("sense making") will be added to the `RoleExtractor` class in a future release.

### Tests
To run tests:
```
$ python -m unittest discover -v
```
As noted above, the tests for the `igraph` interface are skipped when `igraph` is not installed.  Because this package is intentionally not required, the  test coverage reported above is much lower than when `igraph` is installed and its interface tests are not skipped (__97% coverage__ to date).

### References
[1] Henderson, et al. [It’s Who You Know: Graph Mining Using Recursive Structural Features](http://www.cs.cmu.edu/~leili/pubs/henderson-kdd2011.pdf).

[2] Henderson, et al. [RolX: Structural Role Extraction & Mining in Large Graphs](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/46591.pdf).
