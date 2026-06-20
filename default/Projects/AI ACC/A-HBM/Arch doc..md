
# 7. A-HBM
## 7.1 Level 1 : Die
This clustering aims to minimize latency and simultaneously maximize the data throughput of HBM by avoiding traditional L2 cache architecture.

7.1.1 Companion mode

NPU
- Tensor engine
- Math
- FP16 mac cycle - 6 cycle
- Total 
	- 40 ~ 45 TFLOPS
	- 90 TFLOPS @ FP16 for standalone mode
- TCM - 2MB

7.1.2 Standalone mode

ARM core with coprocessor interface : custom ISA

Total available area for PE cluster 8 PE : 40 mm2
Power efficiency : 3~4 TFLOPS/W @FP16

DMA Engine : NE doesn't support cache and coherency.
However, a CPU, other PEs, or a DMA can access the PE's local memory (TCM) or registers via the AXI-slave port.
