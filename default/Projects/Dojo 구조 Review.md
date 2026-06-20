## 외부 구조
1. LLM 같은 large sized weights 를 handling 할 수 있는가? --> LLM 응용에 적합한가? --> Dojo 같은 구조는 CNN 같은 응용에만 적합한 걸까?


## 내부 구조
1. 내부 RISC-V ISA 가 기반이라면, 얼마나 기존 RISC-V+RVV 와 다른 건가?
	1. Zhi-Gang's comment 
		1. 4-way SMP (not thread!!) : flexible than GPU's SIMT programming model ?
			1. Simpler than the full version ARM CPUs connecting to SME or AMX2
			2. No data cache ?