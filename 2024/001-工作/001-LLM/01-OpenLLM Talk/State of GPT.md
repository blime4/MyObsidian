
地址： https://www.bilibili.com/video/BV1X24y1A7Rz

## GPT Assistant training pipeline
![[Pasted image 20240105171411.png]]

## Pretraining
### Data collection
![[Pasted image 20240105171641.png]]
### Tokenization
![[Pasted image 20240105171733.png]]
### 2 example models
![[Pasted image 20240105172104.png]]
[[LLaMA]] > [[GPT3]] : you shouldn't judge the power of a model just by the number of parameters that it contains
### Pretraining inputs
![[Pasted image 20240105172843.png]]
### Step 1: pretaining Step 2: finetuning
![[Pasted image 20240105173950.png]]
### Base models in the wild
![[Pasted image 20240105180431.png]] 
> Currently the beat available base model probably is the llaMA series from Meta although it is not commercially licensed.

> Base models are NOT 'Assistants'

## Supervised Finetuning
### SFT Dataset
1. Prompt
2. Response
3. Labeling instructions
	1. Helpful
	2. Truthful
	3. Harmless
![[Pasted image 20240108110807.png]]

## Reward Modeling
> In the reward modeling step what we're going to do is we're now going to shift our data collection to be of the form of **comparisons**.
### RM Dataset
![[Pasted image 20240108111358.png]]
### RM Training
![[Pasted image 20240108111514.png]]
![[Pasted image 20240108112021.png]]
![[Pasted image 20240108112037.png]]
![[Pasted image 20240108112056.png]]![[Pasted image 20240108112133.png]]
![[Pasted image 20240108112145.png]]
