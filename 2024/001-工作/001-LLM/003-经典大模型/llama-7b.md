
| 资料                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [http://ext-gitlab.denglin.com:81/software/dl_framework/network_models/LLaMA-7B](http://ext-gitlab.denglin.com:81/software/dl_framework/network_models/LLaMA-7B) |
| https://github.com/huggingface/transformers.git                                                                                                                  |
| [如何看懂模型代码（以 llama 为例）]( https://www.bilibili.com/video/BV1qj411y7kF )                                                                                            |
| [图解llama架构 解读源码实现](https://www.bilibili.com/video/BV1nK4y1F7x7)                                                                                                  |
| [llama模型调试](https://www.bilibili.com/video/BV1Cw411y7gs)                                                                                                         |
| ![[llama_struct_B站良睦路程序员.svg]]                                                                                                                                   |
|                                                                                                                                                                  |

## LLAMA 结构分析
| LLAMA 结构分析   |               |               |          |
| ------------ | ------------- | ------------- | -------- |
| 分词的部分        | [[tokenizer]] |               |          |
| llama 主干部分   | [[Embedding]] | [[Attention]] | [[RoPE]] |
|              |               | [[MLP]]       |          |
| 下游任务 [[CLM]] | [[自回归Loss]]   |               |          |
| 下游任务文本分类     | [[交叉损失 Loss]] |               |          |
