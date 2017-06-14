# Combine p values

##  前言   
最近NGS Hotpot在考虑一个比较玄乎的问题，即对于同样的一个问题的假设检验，使用不同的检验方法，得到不同的p值，该怎么理性的选择相信哪个？或者有没有办法p值组合起来，最终对一个p值做决定好过对一堆p值做决定。 （感觉从生信角度出发应该是哪个结果好信哪个，此处为自黑。）

针对上述问题，我们考虑实践中的例子：比较简单的gene enrichment分析，对于同一个gene list，其实可以用fisher exact test, 也可以使用binomal test，这样就会有两个p值，相信哪个比较好呢？或者有没有办法把这两个p-values组合起来看？例如GREAT对peaks的target注释然后做gene enrichment 分析，就是会给hypergeometric test 以及binomal test的结果。

带着这样的问题（combining p-values）NGS hotpot找到了参考文献[1]，仔细看完后，感觉文献回答的应该是，对于同样的一个问题（总体），可能由于抽样时间，抽样大小等等不一样，用了同样的检验方法，得到不同的p值，怎么组合比较合理。似乎可以适用于我们原来的问题，但是好像不够完美，希望对于NGS Hotpot原始问题有解决方案的同学留言交流。 

实践操作可用scipy.stats.combine_pvalues。



## 简介  

设对一个问题进行![equation](http://latex.codecogs.com/gif.latex?k)次独立检验，其中一个p值为![equation](http://latex.codecogs.com/gif.latex?P_i)。combining p-values的三种方法如下：



### a. Fisher's combined probability test

![equation](http://latex.codecogs.com/gif.latex?X_F^2=-2\sum_{i=1}^{k}{ln(P_i)})


即例如对于![equation](http://latex.codecogs.com/gif.latex?k)次t-test结果来说，他们的上述结果符合自由度为![equation](http://latex.codecogs.com/gif.latex?2k)的卡方分布，然后就用卡法分布来得到一个p值。这个方法简单的经验解释即是否存在至少一次检验可以拒绝原假设，可能当有一个p值非常显著的时候，组合得到的p值也是显著的，即所谓的非对称性，受小p值的影响很大。例如两个0.01组合起来是0.001, 而一个0.001一个0.999则是0.008。所以对于一个问题，这个方法扑街。



### b. Z-transform test (Stouffer's method)


![equation](http://latex.codecogs.com/gif.latex?Z_s=\frac{\sum_{i=1}^{k}{Z_i}}{\sqrt{k}}) 


其中的![equation](http://latex.codecogs.com/gif.latex?Z_i)即把p值映射回正态分布的Z-score值，python中为scipy.stats.norm.isf

如果原假设为真，则得到的![equation](http://latex.codecogs.com/gif.latex?Z_s)是服从标准正态分布的，到p值的映射即为组合后的p值。映射方法为scipy.stat.norm.sf。

这个方法和上述方法比即不会受到特别小的p值的影响，可以当作一个平均情况。



### c. Weighted Z-method


![equation](http://latex.codecogs.com/gif.latex?Z_w=\frac{{\sum_{i=1}^{k}{w_i}{Z_i}}}{\sqrt{\sum_{i=1}^{k}{w_i^2}}}) 


其中理想的设置是![equation](http://latex.codecogs.com/gif.latex?w_i=std_i)，即第i次抽样／检验中的标准差。该方法和上面的Stouffer's method一脉相承，可以同时考虑不同抽样的样本量。



## 实验证明   

文章中用了一波模拟比较一下了三种方法，结论即weighted Z-method较好，尤其是组合后的p-值和真实p值的相关性来看非常一致。



## 一点迷思   

对于数据符合正太分布的检验来说，直接使用Stouffer's method原理上来说应该是没有问题，所以，为了整合例如hypergeometric和binomal的结果，是否只要相应改变其中的Z-transform的映射就可以了呢？



## Reference  

1.	Whitlock, M.C. Combining probability from independent tests: the weighted Z‐method is superior to Fisher's approach. Journal of evolutionary biology 18, 1368-1373 (2005).
