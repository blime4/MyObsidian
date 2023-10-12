
# torch. no_grad () vs. param. requires_grad == False

![[Pasted image 20230705213514.png]]
torch. no_grad 将不保存梯度信息。所以不会爆 OOM。
