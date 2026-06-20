
용어
- DSE : Design Space Exploration (전체 시스템 - SoC, Compiler 까지 포함하는 전체 시스템 탐색)
- Workload : AI 응용에 한정. 모든 AI 응용이라기 보다는, LLM 외 기타. (현재까지는) LLM 의 비중이 가장 절대적.

# 0. Summary
## 과제 목표 (25년 이후)
미래 AI 시스템에서 경쟁 시스템 대비 우위를 위한 아키텍처 제안 (datacenter향으로만 좁힐 필요있나)
다양한 응용/시스템 구성에 따라 빠르게 대응할 수 있는 Simulation Platform 구축
SoC 아키텍처 및 IP 레벨 마이크로 아키텍처 설계 역량 확보

## 팀 구성
Workload, Arch, Platform (Modeling/Simulation) 3가지 group 으로 구성
- Workload : 1) 주요 workload 이해/전파 & trend 전망, 2) HW 위한 input 모델 생성 
  3) Paritition/mapping 알고리즘 (compiler) 역할 
- Arch : 1) 아키텍처 구조 아이디어 발굴 2) 다양한 analytic 모델 개발 
  3) Workload와 Platform 사이 과제 전체적 리딩  
- Platform : 1) (주로 SoC) 디테일 simulator 개발 2) u-arch 이해와 제안 
  3) DSE Arch 탐색시 bottom-up 결과 생성
## 팀의 미션
- Workload 기반 Top-down 분석으로 Arch 그룹의 DSE 탐색.
- 미래 workload 변화 trend 트랙킹, 
- Workload 입력 interface 개발 (huggingface 모델 자동 입력)
- 상위 아키텍처 및 Network collective 방법 제안
- System SW programmability 환경 정의
- SoC 내부 구조 도출을 위한 simulator 및 신규 구조 제안

# 1. AI 응용 System 아키텍처 설계 방향

Static 하고 determinstic 한 AI 응용을 위한 시스템/SoC 설계는, 알고리즘 (workload) 부터 하위의 하드웨어 구조까지의 연결성이 직접적이고 depth 가 짧음. (복잡한 cache memory 구조나 dynamics를 고려하는 SW 모델이 필요 없음)

상위 레벨 분석은, explainable 해야 하고 analyzable 해야 함. 툴이나 분석 모델의 내부가 충분히 이해되지 않은 상태에서 수많은 param 과 config 값들에 의한 변화를 쫓아가기 어려움.
따라서, 이 param/config 의 변화가 어떤 효과를 가져가는지 "대략적으로라도" 이해할 수 있어야 하고, 반대로 그 효과를 위해서는, 이런 HW config 가 필요하다는 것을 추정할 수 있어야.
예를 들면, batch 나 seq length 의 변화가 어떤 HW param 에 impact 가 있는지를 몇개의 HW 조건에 따라 연결지을 수 있음.

시스템 전체적으로 무엇이 가장 우선해야 하는 것인지 상황에 따라 변화시키면서,
Workload와 아키텍처를 함께 "조망 (overview)" 할 수 있어야 함. 

예를 들면, HBM B/W 와 Network B/W 는 각각 입력 모델의 weight B/W 와 activation B/W 를 추상화할 수 있음. 즉, 입력 모델의 weight/activation ratio 를 알면, HW 에서의 "대략적인" 요구되는 HBM/Network B/W 비율을 추정해 볼 수 있음.
또 다른 예로, Power 가 가장 우선시 되는 상황 (실제 A-HBM 상황) 이라면, 한정된 Power 값을 놓고, compute, network, memory 등 세부 component 들이 그 power buget 을 어떻게 "나눠서 가져야" 할지를 고민해서, (최소한) 절대 불가능한 구조안을 뺄 수 있음.

그냥, 툴이 있고, 거기에 param 값을 넣어서, 돌려서 나오는 결과를 확인하고, 다른 param 을 sweeping 하고... 넓은 design space 에서 길을 잃어 버리게 됨.

일반적인 general computing 응용에서는 거의 하기 힘든 방법이지만, domain-specific 한 시스템 구성에서는, AI 이든 아니든 이런 상위 아키텍처링 방법이 필수적임.
그리고, 그것이 reasoning 과 합리적인 story line 을 가지고, 다른 stakeholder 들을 설득하고 납득시킬 수 있는 거라고 할 수 있음.
단순히, 이렇게 세팅했더니 숫자가 2배 더 좋았다.. 라는 것은, 대부분 simulation 에 의한 결과를 믿지 않으려는 회의적인 expert 들에게 쉽게 받아들여지지 않음. 하지만, 명확한 reasoning 을 가지고 있으면, 설사 나중에 숫자가 틀려지더라도 (대부분 틀려짐. 심지어 side-effect에 의해 더 안좋게 되는 경우 많음), 아키텍처가 채택되고 함께 진행해 나갈 수 있음. 즉, sim 결과를 buying 하는게 아니라, 그 아키텍처 장점/논리를 buying 하는 것임.

따라서, 아키텍트는, 많은 "정성적인" idea 보따리를 만들수 있어야 함. 그러기 위해서는, 아키텍트의 능력뿐 아니라, Workload 팀과 다양한 통찰과 데이터를 만들어 내면서, 수 많은 시행착오를 거쳐 가야 함. Arch 와 Workload 는 긴밀하게 붙어서 작업해야 하고, workload 의 분석 자료를 바탕으로, 해당 workload 의 기본적인 특성을 하드웨어/시스템에 반영하는 시도를 (concept 이라도) 해야 함. 다양한 analytic model 개발 능력도 필요하고, Compiler 와 SW 이해도 필요. 

이러한 일들이 Arch 그룹과 Workload 그룹이 밀접하게 해야 하는 것.

하지만, 현재 주어진 LLM 응용과 제한된 compute 특성상, 시스템 레벨에서의 변화 폭은 지극히 제한적임. SoC 내부로 들어가지 않으면, 차별화된 부분을 확보하기가 점점 쉽지 않음.
따라서, Platform 그룹에서 SoC 내부의 u-arch 를 위한 simulator 를 구성하면서, 점점 u-arch 에 대한 구성을 하는 것도 자연스러움. SystemC 같은 cycle 모델은 아니더라도, transaction model 로도 SoC u-arch 로 모델링이 가능함.

시장에서는, GPU 대비 PPA 2배도 대부분 시장 진입에서 실패하는 경우들 (AI startup) 이 많기 때문에, 2배 이상이어도 부족하다는 생각을 많이 하지만, 수 많은 AI startup 들이 PPA 2배 향상으로도 계속 실패하는 이유는, 결국 SW 친화성과 현재 심각한 time-to-market 에 의한 조급함 (hyperscaler끼리 경쟁) 때문임. 
때문에, SW 이슈/최적화 어려움 문제를 일부라도 해결해 준다면, 2배 정도의 PPA 우위로도 충분한 경쟁력을 가질수 있다고 생각함.
-> 5배도 부족할 수 있다는 의견의 반론

결론적으로, SoC 와 NPU IP 개발은 계속적으로 혁신해야 함. Memory-centric 의 장점을 가지고 가면서, PE 나 SoC 내부의 아키텍처에서의 차별화 연구를 포기해서는 안됨.
Platform 그룹은 그러한 방향으로의 토대를 만들수 있음.

### 짧은 노트 1 - LLM 의 미래 (5년후에도 가장 중요한 응용 가능성)
Transformer 자체는 "inductive bias" 가 아주 낮은 구조. 즉, 많은 데이터가 요구되지만, 아주 general 하게 다양한 응용을 대응할 수 있다는 점. 그리고, GEMM 기반 구조는 parallelization 에 용이.
즉, (최소 5년은) 주요한 모델로 남아 있을 가능성이 높음.
영상 같은, domain-specific 한 모델 (CNN 기반) 이 다시 득세할 수 있으나, LLM 구조의 하위 call 형태로 남아 있을 가능성이 더 큼. 물론, Diffusion 이나 Mamba 모델들을 잘 살펴보고 있어야.

### 짧은 노트 2 - Memory-BW limit 지속될까
autoregressive 한 생성이라는 관점에서는, long-context 가 필요한 대부분의 응용일 경우 이러한 형태일 것으로 보임. 하지만, 순수한 알고리즘 자체가 지나치게 memory-hungry 함 (분명 optimization 은 지속될 것임). Prefill 정도의 높은 arithmetic density 를 요구하는 경향도 있어 보임. 하지만, 최소한 memory capacity 는 계속적으로 중요해 질 것음. (넓은 context 요구는 capacity 말고 방법이 없을 듯.)

### 짧은 노트 3 - 추론용으로써의 시스템 크기 이슈
Tray (Board) 레벨 이상의 초대형 추론 응용이 과연 필요할까.
즉, Rack 단위에서의 strong scalability 는 필요없지 않을까. 물론, A-HBM 구성상, 점점 시스템 크기가 줄어드는 경향.

# 3. 필요한 인력/경험
우선, HW architecture 경험자가 거의 없음. 
Compiler 역할 (Partitioning/Mapping) 이 필요.
-> 어떻게 workload 를 distribution 하느냐가 너무 큰 차이를 나오게 함.
Power model 을 만들 수 있는 인력 필요.
최신 알고리즘 trend 을 tracking 하기 위해서, 과제 내부보다는, 산학을 활용하는게 효율적으로 보임.

# 4. 단기적인 팀 구성 변화 (6월 산출물을 위한)
## 현재 이슈들
HighLevel 은 Arch 그룹과 비슷할 수 있으나, Workload 관점에서 top-down 그림을 그리는게 아니라, 주어진 아키텍처의 상위 레벨 모델링을 하는 것임. 그것은, Low-level 팀의 실제 simulator 의 coarse 한 버전에 불가. 즉, arch. 탐색과 분석을 위한 것이 아님. (원래 미션이 주어진 3가지 아키텍처의 모델링이므로)

-> 좀 과한 지적일 수도 있으나, 설명가능한 형태로 리포트가 나오고 있지 않음. 그냥 5배 좋다 정도. 툴에서 나오는 성능 break-down 표를 해석할려고 하지 않음. 심지어, ISO-comparison 통해서, 무엇이 차이를 만들어 내고 있게 세팅하고 있지도 않음. 게다가, topology 결과가 network 팀의 결과와 전혀 매칭되지 않음. LLMCompass 툴의 network 모사의 한계로, multi-level network 부분을 새로 구성하는 중. 
-> 지속적으로 US 팀에 수정 지시하고, analytic model 의 의미와 sweeping 하는 과정을 효율화해서, 가능한 도움이 될 많은 back-data를 산출할 수 있도록 해야 함. 최소한 결과를 설명할 수 있는 reasoning 근거 확보. 한국 HL 팀 수주안에 직접적으로 follow 할 수 있도록 진행 중.

현재 팀내 간단한 workload layer-wise 한 spreadsheet 모델조차도 없음. 
-> 초기 분석할때 필수적이고, 각 layer 의 요구되는 연산량이나 메모리 capacity 확인 부터, 더 나아가서 layer 별 여러 요구사항들을 readable 하게 아키텍트들이 익숙해 지게 할 수 있음.

전반적인 이슈는, 현재 쓰고 있는 모든 툴들이 한번도 경험해 보지 않은 상태라서, 어떤 한계와 이슈가 나올지, 그리고 담당자들이 예상하고 있는 개발 계획이 실제로 가능한지 확신하지 못하고 있는 것.
## 최소한의 변화
일정상 큰 변화는 불가. 최대한 기존 산출물 계획을 그대로 진행하고, 1차 full-cycle 결과를 빠르게 확보하고 수정/보완 하는 것이 유일한 방법임. 
LLMCompass 를 최대한 개선/보완 (새로운 툴을 찾거나 처음부터 다시 개발하기에는 너무 늦음)
US 위주로 진행하되, 한국 HL 팀의 역량 확보를 위해서라도, learn & follow 전략이 필요.
## Compiler (Partition/Mapping)
과제내 가장 어려운 부분은, 돌리는 시뮬레이션 결과가 최적의 mapping 에 의한 결과인지 알 수 없다는 것.
즉, compiler 가 해주는 partitioning / mapping 부분이 없기 때문에, 최적의 mapping 이 아닐 때, 불필요한 data movement 나 불필요한 Op 가 없도록 해야 함.
가장 어려운 부분일 수 있고, 실제 Compiler 팀과의 협력을 고민해 보되, 짧은 남은 일정이 어려운 부분 (뭔가 코드 인터페이스라도 맞춰보기에도 쉽지 않음)
-> Collective Call 부분은 Network 팀에서 대응 가능할 것으로 보이나, (일정상 쉽지 않음) 각 SoC 에 적절한 양의 local data 양을 적절하게 allocation 하는 것까지 하는 것은 일정상 어려울 듯.