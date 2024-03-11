## Abstract
1. 以往方法的不足：
	1. Existing solutions such as data and model parallelisms exhibit fundamental limitations to fit these models into limited device memory, while obtaining computation, communication and development efficiency.
		1. 现有的解决方案（如数据和模型并行性）在将这些模型放入有限的设备内存中，同时获得计算、通信和开发效率方面存在根本性局限性。
2. Zero 的成果：
	1. This represents an 8x increase in model size and 10x increase in achievable performance over state-of-the-art.
		1. 这意味着与最先进的设备相比，模型尺寸增加了 8 倍，可实现的性能提高了 10 倍。
	2. researchers have used the system breakthroughs of ZeRO to create the world’s largest language model (17B parameters) with record breaking accuracy.
		1. 研究人员利用 ZeRO 的系统突破，以破纪录的精度创建了世界上最大的语言模型（17B 参数）。


## Extended Introduction
1. 遇到的挑战：
	1. To enable the continuation of model size growth from 10s of billions to trillions of parameters, we experience the challenges of training them - they clearly do not fit within the memory of a single device
		1. 为了实现模型大小从数十亿个参数到数万亿个参数的持续增长，我们遇到了训练它们的挑战——它们显然不适合单个设备的内存
	2. 解决方法：
		1. data parallelism (DP) ： 无效
			1. Basic data parallelism (DP) does not reduce memory per device, and runs out of memory for models with more than 1.4B parameters on current generation of GPUs with 32 GB memory.
				1. 基本数据并行性 （DP） 不会减少每个设备的内存，并且对于具有 32 GB 内存的当前一代 GPU 上参数超过 1.4B 的型号的内存不足。
		2. Pipeline Parallelism (PP)
		3. Model Parallelism (MP) : 最有前途的
			1. 11B T5 model
			2. Megatron-LM 8.3B
			3. 不足：MP cannot scale much further beyond these models sizes
			4. 
		4. CPU-Offloading