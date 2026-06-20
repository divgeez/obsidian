
### 메모리 용량과 대역폭
용량에 더 관심이 큼. 메모리 용량 때문에 큰 모델을 넣지 못해서, 나누어서 생기는 통신 이슈에 민감
최근 트랜드에서 MLA, GQA 때문에 대역폭은 좀 약한듯
LPDDR 로 확장하는 것에 관심 -> cHBM 에 LPDDR 넣는 것은 효과 있을지 미지수
- CXL 관심
- 최근 GPU 개수가 줄어들면서, CPU 가 비율상 늘어나는 경향
	- Multi-socket 으로 인해, CPU 간 데이터 교환 싫어 하는 추세
	- 최대한 하나의 CPU 많이 넣었으면 좋겠음

### 
### 모델은 계속 커질 것임

### 메타의 모델이 더이상 오픈이 아닐 듯
현재 개발 모델은 아주 secret 하게 진행 중
### EP 가 더욱 발전될 것
고정된 parallelism 이 아니라, EP 와 DP 크기를 바꿔가면서 효율을 찾아감
expert 를 추가적으로 메모리에 많이 담아 둠

### CPU 메모리에서 데이터 가져오는 이슈
NVIDIA 에서 얘기하는 IO cache 등의 핸들링 때문에,
오히려 더 늦어지는 이슈들, page transition 등
굳이 NVIDIA 방법을 쓰지 않아도 됨
다, software-managed 되니
싱글 unified memory
NVIDIA 가 얘기하는 page migration 이 언제 어떻게 될지 모르고, 그러다가 더 늘려진다
SW cache 로 들고 있다가, 임베딩 테이블 정도,
생각보다 bw 가 크지 않다
-> cache 를 직접 관리하는게

CXL 을 생각해 볼려고는 하는데.. 모르겠다

### Heterogeneous 구조
여러 랙레벨로의 hetergo 는 HW 에서 manage 어려움으로 받아들이기 쉽지 않음
- 독립된 rack 은 쉽지 않을 듯
- 메모리 Pool 이 가능하다면 intra-rack 에 넣는 것도 필요
가능한 적정 단위 (한 Tray ?) 등에서는, homogeneous 한게 필요
하나의 하드웨어에서 어떤 때는 prefill, 어떤 때는 decode 이런 형태가 좋다
- 같은 모델이라도 중국어나 영어에 따라 크기가 다르다

현재 Meta 는 Recommendation 과 Gen.AI 를 하나의 서버에서 돌림
-> Recomm. 하는 쪽에서 불만. 필요없는 부붑이 많아서

-> 정말 해야 되면, 하기는 할 거다
### Post training 시, inference 많이 필요
일종의 데이터 생성하는 것인데, synthetic data generation
inference 이기는 하나, latency 가 전혀 중요하지 않음
-> 메모리 대역폭 뿐 아니라, computation 도 중요하다는 얘기에서
Precision 을 좋은 것을 써야 할 수 도 있다

### 메모리에서 precision 바꾸는 것 지원
최근 자유롭게 precision 바꾸는 필요가 커짐. 메모리쪽에서 자동으로 바꾼다면, 의미가 있을 수도

### CPU 와 GPU 의 비율이 달라지면서, 전체 메모리가 줄어든다 ?
GPU 성능이 좋아지면서, 1:1 비율까지..
-> 근데, 전체 메모리는 줄어든다 ?
그래서, 메모리나 Flash 를 같이 쓰면 좋을 거 같은데

### GPU 의 스위치가 엄청 비싸고 파워도 많이 먹는다
그렇게까지 높은 대역폭이 필요없다. 

### Meta 도 별로 랙 레벨에서 바꿀 수 있는 것이 없다
NVIDIA 나 AMD 가 그냥 랙 레벨로 가져온다
디자인하거나 수정할 수 있는게 별로 없다

### 있는 것을 가져다 쓰는 것 - vLLM
외부에서 새로 조인한 사람들이 기존 툴을 싫어하고, vLLM 으로 하는 것으로 좋아함
vLLM 에 직접 참여하는게 필요
사람들이 vLLM 을 받아서, 삼성 하드웨어 패치를 하는 것은 그리 원하지는 않음

### PIM 이 간단한 펑션만 해줘도 adopt 하기 좋을 듯


### HBM power 이슈
B200 1000W GPU 를 750W 로 쓸려고 하는데, HBM bandwidth stream benchmark 만 돌려도, 전력때문에 안나옴
- 최근 데이터의 밸류, all-zero 로 한것과 파워 소모 차이가 큼 -> 인코딩 ?

### 블랙웰이 nvlink 대역폭이 좋은데, 별로 차이가 없다
어짜피 사이즈가 작아서, low-latency 로 하는게 좋다


### expert 를 제한을 주지 않음. 좋은데로 가라

캐싱해야 함


### 연구 관점에서 다양한 구조안 검토하는데 협업 하는 것 환영

메모리 대역폭은 커질 것 같으나, 형태가 달라질 것이다.
DP 와 EP 조건에 따라 달라지니깐