
Extracted Annotations:

Method


分为两个步骤，训练水印提取器以及finetuning


Part1









基于一个基本的watermark框架，名称为：

- > *HiDDeN* [(Fernandez 等, 2023, p. 3)](zotero://open-pdf/library/items/?page=3&annotation=N928PATA) 




- > *jointly optimizes the parameters of watermark encoder WE and extractor network W* [(Fernandez 等, 2023, p. 3)](zotero://open-pdf/library/items/?page=3&annotation=6HV2BNC4) 

![](file://C:%5CUsers%5CAKK87%5CZotero%5Cstorage%5C5MTX7ZMH%5Cimage.png)[ ](zotero://open-pdf/library/items/?page=3&annotation=RMMUVFCX)


首先对于encoder，输入一张图片x以及签名m，输出一个一样的小的带水印图片（实际上是产生residual然后再相加），然后对这个图片进行数据增强，再给extractor进行训练，提取出m’

![](file://C:%5CUsers%5CAKK87%5CZotero%5Cstorage%5CAF23WGHI%5Cimage.png)[ ](zotero://open-pdf/library/items/?page=3&annotation=D638KKFB)


这里在m’上额外套用了一个$\sigma$函数，意思可能是作为sign函数把m’变为标准的二进制01编码


然后要求m与m’尽可能的接近，学习的损失函数如下：

![](file://C:%5CUsers%5CAKK87%5CZotero%5Cstorage%5C8ZGULYCP%5Cimage.png)[ ](zotero://open-pdf/library/items/?page=3&annotation=WCB3V89M)






Part2
接下来是对Diffusion Model进行 fine-tuning，注意这里只是对stable diffusion过程中的decoder进行微调，对整体过程没什么印象，这样即方便又能保证性能


这里的损失函数又分为两个部分

![](file://C:%5CUsers%5CAKK87%5CZotero%5Cstorage%5CQNVMPAZ6%5Cimage.png)[ ](zotero://open-pdf/library/items/?page=3&annotation=YMBLEFWK)




![](file://C:%5CUsers%5CAKK87%5CZotero%5Cstorage%5CK66VREZG%5Cimage.png)[ ](zotero://open-pdf/library/items/?page=3&annotation=VSGSKF6F)


第一个损失函数负责水印，

- > *LDM encoder E that outputs activation map z = E(x)* [(Fernandez 等, 2023, p. 3)](zotero://open-pdf/library/items/?page=3&annotation=DP46HYM9)  
> *图片被autoencoder放入latent space


decoder重新解压这张图片（其实这个过程只涉及autoencoder 和 diffusion过程无关？）


extractor提取这个decoder生成的图中的签名，然后通过$\sigma$函数


损失函数要求这个签名与预先人为对这个模型设计的水印要约接近越好


Decoder完全不需要之前的watermark encoder，只需要对这个损失函数进行微调就可以了！”** 





第二个损失函数负责保证图片质量，其实也就是保证微调后的解码器要与解码器最初的效果接近


这里第二个损失函数就用解码器的输出x’与原始解码器的输出进行比较




![](file://C:%5CUsers%5CAKK87%5CZotero%5Cstorage%5CNB7E3YZI%5Cimage.png)[ ](zotero://open-pdf/library/items/?page=3&annotation=E4UDQQNN)



特别注意，在fine-tuning的阶段没有通过变换来数据增强，但整个检测仍然具有鲁棒性，这是因为在之前的训练中 extractor就已经足够具有鲁棒性了？但如果这里再次把Decoder输出的图进行变换，然后再进入extractor然后计算损失函数，是否可以更有效？





还有个致命问题是，如果攻击者知道了W（白箱），对图片进行对抗攻击，那么W还是没有鲁棒性


会不会是之前的数据增强只是普通的变换而不是更强的对抗训练？

- > *watermark’s resistance to intentional tampering, as opposed to distortions that happen without bad intentions like crops or compression* [(Fernandez 等, 2023, p. 7)](zotero://open-pdf/library/items/?page=7&annotation=TQQ3K6BP) 


作者自己也是把这种恶意的改造图片和无意的剪裁压缩分开，那会不会是之前transformation不够好？





而对于更改模型的攻击，作者讨论了两种

- > *it is difficult to significantly reduce the bit accuracy without compromising the image quality: artifacts start to appear during the purification* [(Fernandez 等, 2023, p. 8)](zotero://open-pdf/library/items/?page=8&annotation=B5LDZQHU) 


前者貌似还不错


但这类文章，是不是都对于攻击只是用实验来简单讨论，似乎都没有更深入的探讨







