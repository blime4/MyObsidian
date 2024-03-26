
| 资料                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [http://ext-gitlab.denglin.com:81/software/dl_framework/network_models/LLaMA-7B](http://ext-gitlab.denglin.com:81/software/dl_framework/network_models/LLaMA-7B) |
| https://github.com/huggingface/transformers.git                                                                                                                  |
| https://www.bilibili.com/video/BV1qj411y7kF                                                                                                                      |
| https://www.bilibili.com/video/BV1nK4y1F7x7                                                                                                                      |
| ![[llama_struct_B站良睦路程序员.svg]]                                                                                                                                   |
|                                                                                                                                                                  |

## LLAMA 结构分析
| LLAMA 结构分析 |               |               |
| ---------- | ------------- | ------------- |
| 分词的部分      | [[tokenizer]] | Test->        |
| llama 主干部分 | [[Embedding]] | [[Attention]] |
|            |               | [[MLP]]       |
| 下游任务       | [[自回归Loss]]   |               |
| 下游任务       | [[交叉损失 Loss]] |               |
