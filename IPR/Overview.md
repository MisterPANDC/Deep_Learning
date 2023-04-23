# Intro
>Rethinking Deep Neural Network Ownership Verification: Embedding Passports to Defeat Ambiguity Attacks

一般来说，攻击分为两种 分别是==removal attack== 与 ==ambiguity attack==
removal attack 主要是对模型做各种更改(modifications)，比如
==“model fine-tuning, model pruning and watermark overwriting”==

而一般的watermark基本都能应对以上攻击
#problem

ambiguity attack就是通过伪造水印的方法来攻击验证的过程
对于ambiguity attack 来说，一般的水印方法都无法防御，也就是说攻击者很容易的可以通过逆向工程的方式来伪造水印
甚至说，在不知道原始训练集的基础上，可以花很少的计算量来进行逆向工程
初读这里，有几个疑惑
1. 不知道原始数据集，那是否知道了模型的水印机制
2. 计算量很小，理论上是否用各种方式提高计算量就保证安全

---
传统的水印方法分为两种
第一种是 feature-based watermark
将水印嵌入到模型的参数中，这个是通过在学习的过程中加上额外的正则项来实现的

第二种是 trigger-set based watermark，实际上和后门攻击差不多，就是特定的样本触发特定的标签

第一种也被称为 white-box (verification)
第二种被称为 black-box (verification)
==这里的黑箱白箱不是针对于攻击者，而是对于验证者是否能够访问到被验证模型的参数==


# KEY Attribute
fidelity()
robustness()
invertibility()
# Removal Attacks


# Ambiguity Attacks

## invertible

>Rethinking Deep Neural Network Ownership Verification: Embedding Passports to Defeat Ambiguity Attacks

这篇论文提到，只要推理过程中样本不影响质量就能轻松实现逆向工程
也就是说，只要trigger set 与 signature 完全与一个测试集不相关，就能实现逆向过程

![[Pasted image 20230419183457.png]]
简单来说，这个逆向过程就是==冻住神经网络的参数，然后梯度更新==
注意对于IP保护来说，攻击者自然是知道模型参数的，或许在使用模型服务的情况下攻击者可能不知道模型参数（这就涉及到fingerprint了，也就是从生成的图片中知道是不是我们的模型产生的，和这里的‘验证模型’的场景好像有所不同）

其实思想上优点像对抗攻击，通过梯度来更新输入，伪造出验证集