
90% 이상 범위/내용 비슷 - 경훈님 동의
다른게 접근해서, 유지되어야 하는 건가 ? 아니면, 결국 다 통합되는 건가 ?

-> updated at the next day
과제 자체의 내부 동작이
우선, Doug 의 high-level suggestion 을 바탕으로 **성능 평가** 가 중심
한국팀은 새로운 아키텍처 제안은 역할이 아니다. ??
26년정도까지 (아니면 올해) 하고,
팀을 분산 또는 engineering team 으로

-> High-level SA 팀 입장에서의 우려
Former SAL 사람들 장점이라면, 
기존 HPC computing env. 잘 이해하고 있고, 
AI 전용 특성에 맞게, 좀 더 dedicated customization 대한 것은 부족하더라도, (가장 우려되는 것이기는 하나)
뭔가, practical 하게 동작할 수 있는 그림을 그릴 수 있을 듯
반면, cache coherency / programming env. 등의 지나친 관심은
PPA 같은 효율성 면에서 나쁜 방향을 제시하는 경우 많음
최근 test-time scaling 이든 inference 에서도의 변화는 (Deepseek)
단순 

-> 

-> ABCX 과제는 좀 더 A-HBM 등의 사업부 협업에 집중해야
현재 코어 연구의 대부분은 기존 Mobile NPU 때의 scope 와 크게 달라지지 않음
(MAC 개수, granuality 고려, mixed-format 의 효율적 지원, SRAM 최적 활용 위한 세부 구조 등)
NPU data-path 가 CNN 과 달리, 일반적인 MatMul 로 통합되어 가기 때문에,
개발 진입 장벽은 솔직히 더 낮아짐. 
PPA 관점에 집중
Scalability 를 위한 다수 PE 연결 구조 (multi-core 구조) 는
부문내, 특별히 NPU IP 개발 여력을 가지고 있는 곳이 제한 (S.LSI 신규 조직에 대한 정보 부족 - AI PC 향 IP 개발 정도만 알고 있음)


-> 상호 보완적인 그림
실제 SW 동작, 상위 수준의 그림 (SoC 내, 연결성), CPU 와의 연동
