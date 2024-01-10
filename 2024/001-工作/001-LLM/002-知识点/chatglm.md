![[ChatGLM微调实训.pdf]]
## Finetuning
1. Prerequisite [英 /priː'rekwɪzɪt/]
	1. Mixed precision ![[Pasted image 20240109153137.png]]![[Pasted image 20240109153421.png]]
		1. [[prevent underflow]]
	2. ZeRO
		1. Why not [[Data parallel]] (数据冗余) or [[Model parallel]]（通信慢）
			1. DP replicates the entire model states across all data parallel process resulting in redundant memory consumption; while MP partition these states to obtain high memory efficiency, but often result in too finegrained computation and expensive communication that is less scaling efficient.
		2. Where the memory goes
			1. ![[Pasted image 20240110105102.png]]
		3. 
2. [[P-tuning]]
	1. [[P-tuning v2]]
	2.  
3. Full Parameter
4. LoRA
## Deploy with Gradio