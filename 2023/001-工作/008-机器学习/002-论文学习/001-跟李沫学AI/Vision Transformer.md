## 关键词

[[VIT]], [[CV]], [[SWIN Transformer]], [[Transformer]]

## 有趣特性

VIT 能处理比较有趣的图片：遮挡，偏移，对抗 patch, 随机排序
![[Pasted image 20231205144455.png]]
## 难点：
1. 直接作用于图片，输入 transformer 的序列长度太长，计算复杂度太高

之前的尝试：
1. 用特征图作为序列的输入。

VIT 处理的方式：将图片切成 16\*16 的很多的 patch
![[Pasted image 20231205150628.png]]

[[inductive biases]] (归纳偏置)
卷积神经网络特点：
1. locality
2. [[translation equivalence]]


