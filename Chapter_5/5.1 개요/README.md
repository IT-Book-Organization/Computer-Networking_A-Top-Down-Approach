# 5.1 개요

`포워딩 테이블`(목적지 기반 포워딩의 경우)과 `플로우 테이블`(일반화된 포워딩의 경우)이 네트워크 계층의 데이터 평면과 제어 평면을 연결하는 수요 요소였는데,  
이 테이블들이 라우터의 로컬 데이터 평면에서의 포워딩을 지정했다.

바로 이전 장에서의 그림을 상기해보자.

<br/>

<p align="center"><img width="700" alt="포워딩 테이블 1" src="https://user-images.githubusercontent.com/86337233/213144150-84a7748e-3547-47c6-86d5-c454686cb6e6.png">

<br/>
<br/>


<p align="center"><img width="700" alt="포워딩 테이블 2" src="https://user-images.githubusercontent.com/86337233/213144144-816658f9-54f9-4782-afc5-016e19310c12.png">

<br/>
<br/>


일반화된 포워딩의 경우에 라우터가 취하는 행동은 다양한 형태로 나타날 수 있었다.

- 라우터의 출력 포트로 패킷을 전달
- 패킷을 버리거나 복제
- 2, 3, 4계층의 헤더 필드를 재작성

<br/>

이 장에서는 **포워딩 테이블이나 플로우 테이블이 어떻게 만들어지고 유지 및 설치되는지**를 알아볼 것이다.

<br/>

### 라우터별 제어

> 💡 개별 라우팅 알고리즘들이 제어 평면에서 상호작용한다.

<br/>

아래 그림은 라우팅 알고리즘들이 모든 라우터 **각각에서** 동작하는 경우를 나타낸다.

<br/>

<p align="center"><img width="700" alt="라우터별 제어" src="https://user-images.githubusercontent.com/86337233/213144138-0baa9de9-c18c-4349-ad2a-6198a6dce222.png">

<br/>
<br/>

- 포워팅과 라우팅 기능이 모두 개별 라우터에 포함되어 있다.
- 각 라우터는 **다른 라우터의 라우팅 구성요소와 통신하여**
  자신의 포워딩 테이블의값을 계산하는 라우팅 구성요소를 갖고 있다.
- `OSPF`, `BGP` 프로토콜이 이 라우터별 제어 방식을 기반으로 한다.

<br/>

### 논리적 중앙 집중형 제어

> 💡 일반적으로 원격에 위치한 별개의 컨트롤러가 지역의 제어 에이전트(CA)와 상호작용한다.

<br/>

아래 그림은 `논리적 중앙 집중형 컨트롤러`가 포워딩 테이블을 작성하고, 이를 모든 개별 라우터가 사용할 수 있도록 배포한 경우를 나타낸다.

<br/>

<p align="center"><img width="700" alt="논리적 중앙 집중형 제어" src="https://user-images.githubusercontent.com/86337233/213144125-6428c97b-0b2c-4579-8074-dc12785f9361.png">

<br/>
<br/>

일반화된 ‘`매치 플러스 액션(match plus action)`’ 추상화를 통해  
라우터는 기존에는 별도로 장치로 구현되었던 다양한 기능(부하 분산, 방화벽, NAT) 뿐만 아니라 전통적인 IP 포워딩을 수행할 수 있다.

<br/>

컨트롤러는 프로토콜을 통해 각 라우터의 `제어 에이전트(control agent, CA)`와 상호작용하여 라우터의 플로우 테이블을 구성 및 관리한다.

라우터별 제어 방식과는 다르게, CA는 서로 직접 상호작용하지 않으며, 포워딩 테이블을 계산하는 데도 적극적으로 참여하지 않는다.
