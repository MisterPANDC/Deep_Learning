基本的算法分为两个步骤，Traing与Sampling，其实也就是训练模型与实际生成的两个步骤
# intro of Training 
![[Pasted image 20230416131140.png]]
记住核心思想在于，前向过程forward，是一个加噪声的过程，而事先已经知道噪声分布，所以其实这个过程完全是已知的可以随意操作的
即给定一张图$x_0$可以随时得到其任意阶段的加噪图像$x_t$,不需要从$x_0$迭代到$x_t$

训练的过程就是训练Noise Predictor，给定当前的t以及$x_t$,要预测出这个过程加的噪声是什么
![[Pasted image 20230416131743.png]]
这里就要注意逻辑上的问题了
因为我们前向采样直接得到的是$\epsilon$,而predict的应该是针对当前周期t的一个noise $\epsilon$，这中间又很多系数关系，之后会展开说明，这里关键是要理清楚思路

# intro of Inference
![[Pasted image 20230416132332.png]]
在推理的过程中，我们需要从一个充满噪声的图片$x_t$,逐渐减少噪声变为$x_{t-1}$，最终变为图片$x_0$
这个过程就就是利用之前提到的Noise Predictor，把这个迭代过程的noise predict出来然后减去即可，(但正如前面所说这一步也有系数关系需要具体研究)
然后需要注意的是，我们额外sample了一个z出来，作为一个随机的噪声来提高生成模型的能力。


# Theory of generation
接下来讲这类生成模型的核心思想
生成模型具有多样性、随机性
其实就是让神经网络的输出分布与真实的输出分布一致即可
![[Pasted image 20230416132929.png]]
从极大似然估计的角度来看，从真实的分布中采样，然后这些样本在我们的模型$P_{\theta}$ 中的概率最大

从另一个角度说明极大似然估计，可以证明这里极大似然估计等价于最小化 KL Divergence（更加直观）
![[Pasted image 20230416133221.png]]
现在的核心问题在于，如何计算对于一张图片，在我们模型的输出分布中的概率是多大，也就是计算$P_{\theta}(x)$

对模型$\theta$来说，我们的输入z是一个确定的简单的分布，然后输出x是一个复杂的分布
$P_{\theta}(x)$ 从条件概率的角度理解，等于原分布中采样出z的概率以及在z的条件模型输出x的概率

$$P_{\theta}(x)=\int_{z}p(z)p_{\theta}(x|z) \,dz$$

但对于我们的模型$P_{\theta}(x|z)$如何表示呢
这里的思想是把模型输出$G(z)$作为均值，然后选取一个固定的方差，$P_{\theta}(x|z)$就是一个以$G(z)$为均值，方差为某一定值的高斯分布

![[Pasted image 20230416134655.png]]

![[Pasted image 20230416134941.png]]
在DDPM中其实是完全一样的思想，只不过多轮去操过程中，==每一次的denoise的输入与输出之间都会有设定一个高斯分布作为条件概率==
去噪denoise是输入$x_{t}$，输出$x_{t-1}$
其中特别注意这个链式法则
$$P(x_0)=\int_{x_1} {P_{\theta}(x_0|x_1)P(x_1)}\,dx_1$$
$$=\int_{x_1} {P_{\theta}(x_0|x_1)P_{\theta}(x_1|x_2)\cdot\cdot\cdot P(x_T)}\,dx_1dx_2\cdot\cdot\cdot dx_T$$
其中，$x_T$ 就是一个简单的已知分布z

---
以上解决了 $P(x)$ 如何表示的问题，接下来重新回答最大似然估计
![[Pasted image 20230416140042.png]]
最大似然估计，转换为最大$P(x)$的下界，再转换为最大化一个期望
![[Pasted image 20230416140159.png]]
最后核心还是最小化一个KL divergence，仍然是把两个分布视为高斯分布，只需要改变期望值即可
前一个分布，作为前向过程，可以直接写出其分布均值，而后一个分布作为后向过程，之前已经提到，就将模型输出值作为分布的均值
![[Pasted image 20230416141207.png]]
![[Pasted image 20230416141322.png]]
![[Pasted image 20230416141331.png]]
通过一阵化简，得到denoise过程输出理论上最应该靠近的值
$$\frac{1}{\sqrt{\alpha_t}}(x_t-\frac{1-\alpha_t}{\sqrt{1-\overline{\alpha_t}}}\epsilon)$$
那么照着这个形式，神经网络==只需要去预测$\epsilon$是多少即可==,最终神经网络去噪过程
$$x_{t-1}=\frac{1}{\sqrt{\alpha_t}}(x_t-\frac{1-\alpha_t}{\sqrt{1-\overline{\alpha_t}}}\epsilon_{\theta}(x_t,t))$$

# 参数细节
在前向过程的加噪音的过程中，有一个权重参数
$q(x_1|x_0)$表示从分布中采样$x_0$后，得到$x_1$的条件概率
这个前向的过程，具体来说就是从一个高斯分布中采样一个噪声，然后和上个阶段的图做加权和得到下一个图
$$x_t = \sqrt{1-\beta_t}\,x_t+\sqrt{\beta_t}\,\epsilon$$
其中这个$\epsilon$其实就是从高斯分布中的一个采样噪声
![[Pasted image 20230416142449.png]]
这个每次采样的噪声是独立同分布的，可以等价为一个新的采样过程
![[Pasted image 20230416143028.png]]
![[Pasted image 20230416143146.png]]
最终可得
$$x_t=\sqrt{1-\beta_1}\sqrt{1-\beta_2}\cdot\cdot \cdot \sqrt{1-\beta_t} \,x_0 + \sqrt{1-(1-\beta_1)(1-\beta_2)\cdot\cdot\cdot(1-\beta_t)}\,\epsilon$$
其中，为了表示方便，设$\alpha_t = 1 - \beta_t$ 以及  $\overline{\alpha_t} = \alpha_1\alpha_2\cdot\cdot\cdot \alpha_t$
$$x_t=\sqrt{\overline{\alpha_t}}\,x_0 + \sqrt{1-\overline{\alpha_t}}\,\epsilon$$
也就是之前说的前向过程不需要迭代可以直接得到任意$x_t$
# 学习过程
明白了参数细节之后，学习过程也就非常明了了

![[Pasted image 20230416131140.png]]
noise predictor其实就是给$x_t=\sqrt{\overline{\alpha_t}}\,x_0 + \sqrt{1-\overline{\alpha_t}}\,\epsilon$ 以及 t ，利用梯度下降学习根据这两个值输出 $\epsilon$ 来，这个过程就是完完全全的让神经网络根据 $x_t$ 与 t 学会解出 $\epsilon$ 