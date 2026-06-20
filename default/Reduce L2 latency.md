
It's due to the micro-architecture of ET ?
- Different voltage domains
- Cross-bars between the private cores and L2 SCB

Need to check other cases
- Intel ? AMD ? 

ChatGPT : 
I don't have real-time or the latest specific information on processor details, as my training only includes knowledge up to January 2022. However, I can provide some general information on L2 cache latency for Intel, AMD, and ARM processors.

    - The L2 cache latency for Intel processors can vary depending on the specific model and architecture. Generally, it's in the range of 10-20 cycles.

https://www.intel.com/content/www/us/en/developer/articles/technical/memory-performance-in-a-nutshell.html

# L1 cache latency ?
## ET case
Looks too long ??
How about other CPUS ??
https://www.7-cpu.com/

보통 4 cycles ???

---> 
재언
[조재언] 2023-11-14 15:27
pointer chasing bench로 측정된 L1 load to use latency가 4cycles인 것 같네요. 순전히 SRAM array access에 걸리는 시간 외에도 pipeline stage에 의한 latency가 더 들어갔을 것 같습니다.

[조재언] 2023-11-14 15:29
그렇다 해도 요즘은 순수 cache latency가 1cycle보다 클지도 모르겠네요

[이세환] 2023-11-14 15:38
위의 7-cpu.com 사이트 정보에 대한 거죠 ?
정확하게 cache latency 를 어디서 측정하는지도 좀 헷갈리네요.
CPU 순수 core 에서 밖으로 나가는 mem request 시기 부터,
데이터가 CPU core 에 들어오는 때 까지인 것 같은데..

[이세환] 2023-11-14 15:39
이게, 
pointer chasing bench 라고 해서, 이 정의가 달라지면 안될 것 같은데..

[이세환] 2023-11-14 15:39
chasing pointer 로 뭔가 한다는 게,
instruction 수행하는 것 부터 시작하는 걸까요?

[조재언] 2023-11-14 15:45
https://www.nexthink.com/blog/smarter-cpu-testing-kaby-lake-haswell-memory
auto last_node = traverse_list(&list[0], num_ops);
l1 hit 발생하는 load instruction을 back-to-back dependency 걸어서 여러번 반복해서, "소요 cycle" / "load 수행 횟수" = "load 1회당 cycle"을 계산합니다.

[조재언] 2023-11-14 15:46
iteration이 충분히 돌아서 branch predictor가 열일하기 시작하면, 반복을 위한 보조 명령어들은 어차피 critical path에서 빠지기 때문에, critical path 총 길이는 load-to-use latency의 합으로 근사가 되겠죠

[이세환] 2023-11-14 15:46
어렵군요. 이게 뭔가 딱 spec. 에 있는게 아니라서 그런지..
이 데이터는 이런 bench 로 얻는 거고,
Esperanto 는 내부 u-arch 기반으로 하는 얘기라서..

[조재언] 2023-11-14 15:49
1. issue instruction to read l1 cache
2. forward the cache data to AGU, which calculate the next address
3. forward the address to scoreboard, to wake up dependent instruction

[조재언] 2023-11-14 15:49
이 3개가 serialize되어서 반복될텐데

[조재언] 2023-11-14 15:50
저 3단계 latency가 약 4 cycles라는 것 말고는 알 수 없겠네요.

[조재언] 2023-11-14 15:50
benchmarking하는 동안 critical path는 대부분 123123123의 시간으로 채워질 것이라 생각합니다.

[이세환] 2023-11-14 15:52
이렇게 무조건 chasing 조건이 있는 benchmark 이고,
재언님 말처럼 3개가 serialize 할 수 밖에 없는데...

그냥 순수하게 data cache access 하는 latency 보다 더 많이 나올 것 같은데요.

[조재언] 2023-11-14 15:52
가정은... loop 구성을 위한 명령어가 OoO superscalar 구동에 의해 critical path에서 완전히 빠진다

[조재언] 2023-11-14 15:52
네 순수한 data cache access latency보다 많이 나올 수 밖에 없는 구조이고, 4 cycles라고 하는 것들은 대게 이렇게 측정된 듯 합니다.

[이세환] 2023-11-14 15:53
그래서, 4 cycles 이 아주 많은 듯

[조재언] 2023-11-14 15:55
AGU exec에서 1cycle, wakeup & issue에서 1cycle이라 가정하면 순수 cache latency는 2 cycles일 수도 있겠다는 생각은 듭니다.

[조재언] 2023-11-14 15:55
low power를 위해 serial tag & data array access를 했다 (2 cycles) 가정하면 아귀가 딱 맞긴 합니다

[Smarter CPU Testing – How to Benchmark Kaby Lake & Haswell Memory Latency | Nexthink](https://www.nexthink.com/blog/smarter-cpu-testing-kaby-lake-haswell-memory)


## ARM Cortex-A57
https://en.wikichip.org/wiki/arm_holdings/microarchitectures/cortex-a76
4 cycles ?

