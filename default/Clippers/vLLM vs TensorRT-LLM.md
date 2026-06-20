---
created: 2024-11-26T20:47:02 (UTC -08:00)
tags: []
source: https://blog.squeezebits.com/vllm-vs-tensorrtllm-9-parallelism-strategies-36310
author:
---

# [vLLM vs TensorRT-LLM] #9. Parallelism Strategies - SqueezeBits

> ## Excerpt
> This article provides a comparative analysis of different parallelism strategies on vLLM and TensorRT-LLM frameworks. | Tech

---
## Parallelism Strategies for LLMs

As Large Language Models (LLMs) grow increasingly large and complex, they are pushing the boundaries of modern hardware’s computational capacity. These models, with billions or even trillions of parameters, are achieving groundbreaking results across diverse applications. However, this remarkable progress comes with challenges, particularly regarding memory limitations and latency constraints. Many state-of-the-art models have become so massive that no single GPU, regardless of its power, can handle them independently as their size exceeds the memory capacity of a single GPU. At the same time, the computation time required to process these models poses significant latency challenges. To address these issues, parallelism—the ability to distribute computations across multiple devices—has become a key strategy for efficiently serving LLMs.

Parallelism strategies aim to balance workload distribution, reduce latency, and maximize throughput by dividing AI workloads across multiple computational devices. As the number of devices increases, so does the available memory and computational power, enabling powerful scaling. However, parallelism introduces significant overheads, with communication overhead being the most prominent. Communication overhead arises because a higher number of devices necessitates greater communication between them, leading to increased complexity. These overheads often create a trade-off between achieving better serving metrics and the rapidly escalating costs.

The nature of these trade-offs varies significantly across different parallelism strategies. Among the various implementations, **Tensor Parallelism (TP)** and **Pipeline Parallelism (PP)** stand out the most. Each approach offers unique advantages and limitations, depending on the hardware, workload, and desired outcomes. In this post, we will dive into the parallelism configurations provided by two leading frameworks, vLLM and TensorRT-LLM, to evaluate their performance under varying conditions. Additionally, we will examine how these strategies impact throughput, latency, and scalability, both with and without high-speed communication interfaces like [NVLink](https://www.nvidia.com/en-us/data-center/nvlink/) for NVIDIA GPUs.

To better understand the benchmarks, let’s first explore how Tensor Parallelism and Pipeline Parallelism work and the specific challenges they address.

## Understanding Tensor Parallelism and Pipeline Parallelism

### Tensor Parallelism: Distributing Work Within Layers

![Figure 1. Tensor Parallelism for a model with 4 layers.](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252F193a527f-7694-42dd-88b4-6779c081a915%252Fimage.png%3Ftable%3Dblock%26id%3D147258ac-0943-8078-8103-f9f3a068b0b3%26cache%3Dv2&w=3840&q=75)

Figure 1. Tensor Parallelism for a model with 4 layers.

Tensor Parallelism divides the computation of a single model layer across multiple GPUs. Rather than executing an entire layer on one GPU, TP splits the layer's tensor operations into smaller chunks (see Figure 1). Each GPU stores its assigned weight parameters locally, processes a portion of the operations in parallel, and synchronizes the results at the end to produce the output.

For instance, in transformer-based models, tasks like dense matrix multiplication and self-attention are both computationally intensive and require significant memory for weight storage. TP enables all GPUs to handle these tasks concurrently with their respective weight parameters, maximizing computational efficiency and eliminating the need to repeatedly load weights onto the devices. This approach is particularly effective for tasks that demand high computational throughput.

However, TP also introduces a significant challenge: communication overhead. Intermediate results produced by each GPU must be gathered on a designated GPU to complete the computation and then broadcasted back to all GPUs to process the next layer. This synchronization step can lead to performance bottlenecks, especially if the GPUs are connected via slower interconnects like PCIe. Additionally, runtime variance can impact overall performance, as all GPUs must wait for the slowest device to finish its computation before moving forward.

Despite these challenges, TP remains a powerful option for AI applications requiring low latency. By utilizing all GPUs simultaneously, it reduces the time needed to complete individual tasks. Furthermore, high-speed interconnects such as NVLink or NVSwitch can significantly mitigate communication overhead, making TP even more effective in demanding setups.

### Pipeline Parallelism: Dividing the Model Across GPUs

![Figure 2. Pipeline Parallelism for a model with 4 layers.](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252F24bba739-0684-44a0-9d85-529e21cbcc83%252Fimage.png%3Ftable%3Dblock%26id%3D147258ac-0943-8088-bdbb-e97765cb6f81%26cache%3Dv2&w=3840&q=75)

Figure 2. Pipeline Parallelism for a model with 4 layers.

Pipeline Parallelism takes a fundamentally different approach to parallelism. Instead of splitting operations within a layer, PP divides the model itself into distinct segments, assigning each segment to a separate GPU (see Figure 2). Inputs are processed sequentially, with each GPU handling its designated segment before passing the output to the next GPU in line. This approach resembles an assembly line, where computations are carried out in a series to produce the final result.

PP is particularly advantageous when high-speed communication methods between devices are unavailable. By assigning a specific portion of the model to each GPU, PP minimizes the need for inter-device communication, as only the input to a segment and the output from that segment are transferred between GPUs.

However, PP also comes with its own set of challenges. Its sequential nature introduces latency, as GPUs must wait for previous stages to finish before beginning their computations. While pipelining can help overlap computations across segments, the dynamic nature of LLM serving makes it challenging to eliminate computation bubbles entirely. Variability in input sizes can lead to uneven workload distribution, further increasing delays—especially if certain model segments are more computationally demanding than others.

PP is most effective in scenarios where high throughput is prioritized over low latency. For instance, it excels in large batch processing scenarios, such as large-scale inference or training, where efficiently handling multiple inputs takes precedence over minimizing response times for individual requests. Additionally, PP can be valuable in multi-node setups where communication must occur without high-speed interconnects.

## Experiment Setup

To evaluate the performance of these parallelism strategies, we conducted benchmarks comparing Tensor Parallelism (TP) and Pipeline Parallelism (PP) under varying request rate workloads. In all experiments, the maximum batch size was fixed at 256, with other configurations left at their default settings for each framework.

To simplify the experimental setup, we restricted all tests to a single-node configuration, which may disadvantage PP. This decision was made for two key reasons: First, widely used open-source LLM models typically cap their largest versions at 70B parameters, which can generally be served without requiring multi-node parallelism. Second, multi-node setups are heavily influenced by network conditions, making it challenging to ensure consistent and fair comparisons. Therefore, it is important to note that the following benchmarks may not fully represent serving scenarios that explicitly depend on multi-node configurations.

Note that, in addition to TP and PP, there is also a parallelism strategy called Data Parallelism (DP). DP involves multiple GPUs processing different input data independently. For a comparison of the effects of DP and TP, you might find [OctoAI's excellent blog post](https://blog.mlc.ai/2024/10/10/optimizing-and-characterizing-high-throughput-low-latency-llm-inference) helpful.

### Benchmark Dataset

For all experiments, we used datasets with random fixed input and output lengths to maintain consistency in the number of processed tokens across both frameworks. Specifically, we set the input length to 256 and the output length to

256

for all tests, as these lengths are similar to the average input and output lengths in the ShareGPT dataset.

### Framework Version

-   **vLLM**: v0.6.3

-   **TensorRT-LLM**: v0.13.0 / **Triton Server**: v2.50.0

### Model and Hardware

-   **Model**: CodeLlama-34B ([link](https://huggingface.co/codellama/CodeLlama-34b-hf))

-   **Hardware**: NVIDIA A6000 48GB (without NVLink) and NVIDIA A100 80GB (with NVLink)

## Results

To evaluate the performance of tensor parallelism and pipeline parallelism, we examined the trade-off between throughput and time-per-output-token (TPOT) across different request rate settings. This trade-off provides insights into both serving cost, which correlates with throughput, and user experience, which depends on TPOT. Typically, increasing the batch size leads to higher throughput but also results in longer TPOT. This means that while the average serving cost decreases, the user experience tends to degrade.

### Evaluation on GPUs without NVLink (A6000 x 4)

![Figure 3. Throughput vs. TPOT on four A6000 GPUs with vLLM (left) and TensorRT-LLM (right).](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252Fb7788c5d-2c12-406b-962d-49478a6d700f%252Fimage.png%3Ftable%3Dblock%26id%3D149258ac-0943-8095-8fd1-c2ad20863251%26cache%3Dv2&w=3840&q=75)

Figure 3. Throughput vs. TPOT on four A6000 GPUs with vLLM (left) and TensorRT-LLM (right).

On A6000 GPUs, Tensor Parallelism consistently outperformed Pipeline Parallelism with both vLLM and TensorRT-LLM, as illustrated in Figure 3. TP configurations delivered either higher throughput at the same TPOT or lower TPOT at the same throughput compared to PP configurations. This aligns with the well-established view that TP is generally more efficient than PP in single-node setups. The findings highlight PP’s inherent drawback—its sequential processing of layers—which results in significantly higher latency compared to TP, regardless of the batch size. Consequently, TP is better suited for latency-sensitive applications, while PP is less effective in scenarios demanding low latency.

**An intriguing pattern emerges as the request rate increases**: the graph does not exhibit a simple monotonic trend but instead shows a reversal. Notably, when comparing infinite request rates to lower rates, such as 16, TPOT decreases significantly at the infinite request rate.

![Figure 4. How requests are scheduled with different request rate on TensorRT-LLM.](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252F2c93ecfd-409c-4970-b17f-560f5f0ac57c%252Fimage.png%3Ftable%3Dblock%26id%3D147258ac-0943-80a9-8c11-fcf4e3b397bb%26cache%3Dv2&w=3840&q=75)

Figure 4. How requests are scheduled with different request rate on TensorRT-LLM.

T

o explore this behavior, we analyzed per-token timestamps for individual requests as in [our previous post](https://blog.squeezebits.com/vllm-vs-tensorrtllm-4-which-scheduler-wins--33083). The findings revealed a straightforward explanation as illustrated in Figure 4:

-   At request rates up to 16, prefill computations frequently outpaced the incoming request rate. As a result, mixed scheduling occurred, where prefill and decode operations overlapped, causing active batch sizes to grow incrementally. This overlap led to uneven GPU utilization and increased TPOT.

-   At an infinite request rate, requests were processed immediately at the maximum batch size. This eliminated scheduling inefficiencies, allowing prefill and decode operations for all requests to proceed simultaneously in a more structured manner. Consequently, TPOT per iteration was significantly reduced.

Our benchmark data, with fixed input and output lengths, further amplified this trend as workloads became increasingly uniform at higher request rates. Interestingly, the highest throughput was achieved with a PP-4 configuration at the infinite request rate. This result reflects the uniformity in workloads under a fixed dataset and infinite request rate, which minimizes pipeline bubbles and enhances PP efficiency.

These findings demonstrate that PP can be a viable option even for single node setups in scenarios with sufficiently high request rates and uniform dataset distributions. Under these conditions, PP may even surpass TP in terms of throughput — making this option attractive when the TPOT limit can be relaxed.

### Evaluation on GPUs with NVLink (A100 x 4)

![Figure 5. Throughput vs. TPOT on four A100 GPUs, vLLM (left) TensorRT-LLM (right).](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252Ffe959c5f-a5b7-4e79-b7df-73ac1d85ec1e%252Fimage.png%3Ftable%3Dblock%26id%3D149258ac-0943-80cd-8ebb-f578eab807d1%26cache%3Dv2&w=3840&q=75)

Figure 5. Throughput vs. TPOT on four A100 GPUs, vLLM (left) TensorRT-LLM (right).

On A100 GPUs, Tensor Parallelism continued to outperform Pipeline Parallelism in both vLLM and TensorRT-LLM, though the results differed slightly from those observed on A6000 GPUs. The inclusion of NVLink significantly reduced communication overhead between GPUs, enabling better scalability for parallelism strategies. Both frameworks demonstrated noticeable improvements compared to the A6000 benchmarks.

T

ensorRT-LLM, in particular, benefited greatly from NVLink. With NVLink, the gap in maximum throughput between the TP-4 and PP-4 configurations narrowed, as the communication overhead associated with frequent tensor gathers and broadcasts in TP-4 was substantially reduced. Consequently, the benchmarks indicated that high-speed interconnects like NVLink favor Tensor Parallelism in single-node setups, as TP can achieve much lower TPOT with similar throughput.

Another notable difference between vLLM and TensorRT-LLM on A100 GPUs was the performance of PP at high request rates, especially as the request rate approached infinity. In this scenario, PP delivered surprisingly strong performance in TensorRT-LLM, but vLLM failed to scale. This is likely due to better optimization of communication overhead in TensorRT-LLM, enabling it to handle PP more effectively under such conditions. To further investigate this phenomenon, we captured the trace of the PP-4 configuration (infinite request rate) using both vLLM and TensorRT-LLM on A100 GPUs. Note that the traces are for a single A100 GPU.

![Figure 6. Computation trace of PP-4 configuration with vLLM and TensorRT-LLM on A100. It focuses on the time between the two consecutive computation steps. Communication overhead is included in the computation step in case of TensorRT-LLM.](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252F1e253166-c274-4473-a7cf-6c9eed7c0ae6%252Fimage.png%3Ftable%3Dblock%26id%3D14a258ac-0943-80c1-940c-fc43bfeba7b9%26cache%3Dv2&w=3840&q=75)

Figure 6. Computation trace of PP-4 configuration with vLLM and TensorRT-LLM on A100. It focuses on the time between the two consecutive computation steps. Communication overhead is included in the computation step in case of TensorRT-LLM.

Figure 6 revealed that the time window between computation steps—comprising both host overhead and communication overhead—was clearly noticeable in vLLM but minimal in TensorRT-LLM. TensorRT-LLM incorporates communication time into its computation steps, leading to longer step durations compared to vLLM. Although the exact communication overhead is not clearly identifiable, the resulting increase in step latency remains relatively shorter than that of vLLM. This suggests that the PP implementation in TensorRT-LLM is substantially more optimized compared to vLLM, in terms of utilizing NVLink for PP.

For the TP-4 configuration, vLLM also demonstrated successful scaling with increasing request rates. However, a performance gap between vLLM and TensorRT-LLM remained. To better understand this gap, we captured traces of the TP-4 configuration for both frameworks on A100 GPUs.

![Figure 7. Computation trace of TP-4 configuration with vLLM and TensorRT-LLM on A100. In both frameworks, a computation step includes communication overhead.](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252Fca38c16a-2f4c-4554-a2dc-6ec9f233f938%252Fimage.png%3Ftable%3Dblock%26id%3D14a258ac-0943-8065-96fd-d1eeb50841a9%26cache%3Dv2&w=3840&q=75)

Figure 7. Computation trace of TP-4 configuration with vLLM and TensorRT-LLM on A100. In both frameworks, a computation step includes communication overhead.

Figure 7 revealed that both vLLM and TensorRT-LLM incorporates communication overhead inside the computation steps, enabling improved scalability. However, its host overhead was still noticeably longer compared to TensorRT-LLM, which ultimately resulted in lower overall performance.

### Comparing vLLM and TensorRT-LLM in the TP-4 Configuration

![Figure 8. TP-4 configuration vLLM vs. TensorRT-LLM (A6000, A100)](https://inblog.ai/_next/image?url=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fprod-files-secure.s3.us-west-2.amazonaws.com%252F23f4b38d-2def-440d-b962-b485f3d7fb97%252F1d818055-5a03-4282-b5dd-5fe2d5c3bbdc%252Fimage.png%3Ftable%3Dblock%26id%3D149258ac-0943-80e1-84ef-dbcc3e179f9b%26cache%3Dv2&w=3840&q=75)

Figure 8. TP-4 configuration vLLM vs. TensorRT-LLM (A6000, A100)

Benchmark results showed that vLLM fails to scale for PP on A100 GPUs. Consequently, the only configuration suitable for a direct comparison between vLLM and TensorRT-LLM is TP=4. Figure 8 illustrates the performance of both frameworks under the TP-4 configuration, offering a clear comparison.

On A6000 GPUs, the performance of vLLM and TensorRT-LLM is nearly identical. At very low request rates, vLLM demonstrates slightly better performance. However, across all other ranges, TensorRT-LLM consistently delivers marginally better results. In this scenario, both frameworks are viable options.

In contrast, A100 GPUs equipped with NVLink reveal significant differences between the two frameworks. TensorRT-LLM outperforms vLLM across all request rates and achieves over 25% higher maximum throughput (i.e., at an infinite request rate). This advantage is likely attributed to TensorRT-LLM’s superior efficiency in utilizing NCCL and NVLink for device-to-device communication. For serving on A100 GPUs, the results indicate that TensorRT-LLM currently provides superior performance.

Ultimately, the benchmark results suggest that the choice between vLLM and TensorRT-LLM depends on specific workload characteristics, optimization goals, and the availability of high-speed interconnects like NVLink.

## Final Thought

Parallelism strategies are crucial in shaping the efficiency of LLM serving. This post highlights that there is no universal solution—each strategy comes with its own strengths and trade-offs:

-   Tensor Parallelism: Ideal for latency-sensitive applications, particularly in environments with high-speed interconnects like NVLink.

-   Pipeline Parallelism: Best suited for batch-heavy tasks prioritizing throughput over latency (or TPOT).

-   Frameworks: TensorRT-LLM surpasses vLLM in optimizing both throughput and latency, particularly in NVLink-enabled setups.

Again, it’s important to note that Pipeline Parallelism may demonstrate even greater potential in multi-node environments. Further experimentation is required to determine optimal strategies for exceptionally large serving scenarios that cannot fit within a single node, especially in large-scale data centers with hundreds or thousands of GPUs. We hope to begin such experiments when suitable use cases and sufficient resources become available for us (fingers crossed).

Stay tuned for more insights in the **vLLM vs TensorRT-LLM** series!
