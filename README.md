# MIDAS_python
Python package for MIDAS, bugs fixed based on midas_pro(https://github.com/sapphire921/midas_pro)
Enabling Multi-MIDAS and AR-MIDAS(on both recursive and rolling basis), high-frequency indicators considered can have different theta parameters, different sampling frequencies and different lag length.


假设变量 $y_1$ 能够在 $t-1$ 到 $t$ 的时间区间 (例如每季度) 内观测到一次, 另外一个变量 $x_t^{(m)}$ 在同样的时间区间内能够观测到 $m$ 次。我们对 $y_t$ 与 $x_t^{(m)}$ 之间的动态关系感兴趣, 或者说, 我们想将回归方程左边的变量 $y_t$ 投射到右边变量 $x_t^{(m)}$ 及其滞后观测值的历史序列 $x_{t-j / m}^{(m)}$ 当中。 $x_{t-j / m}^{(m)}$ 的上标 $m$ 表示较高的采样频率, 其精确的滞后时间表达为单位区间 $t-1$ 到 $t$ 之间一个分数。一个简单的 MIDAS 模型可表达为以下形式:

$$
Y_t=\beta_0+\beta_1 B\left(L^{1 / m} ; \theta\right) x_t^{(m)}+\varepsilon_t^{(m)}
$$	
其中 $t=1, \cdots, T, B\left(L^{1 / m} ; \theta\right)=\sum_{k=0}^K B(k ; \theta) L^{k / m}, \quad L^{1 / m}x_t^{(m)}=x_{t-1 / m}^{(m)}$在实际的操作中我们一般假设 $B(k ; \theta)$ 中的函数形式 $B$ 已知但具体的参数 $\theta$ 的值为我们需要估计的，故此时滞后项之前的系数被转化为一个参数向量 $\theta$ 的取值问题。  
我们有两种较为常见的估计模型形式 $B(k ; \theta)$ 的函数，分别为：
	1.Almon滞后项：
	$$
	  B(k ; \theta)=\frac{\exp \left(\theta_1 k+\cdots+\theta_T k^T\right)}{\sum_{k=1}^K \exp \left(\theta_1 k+\cdots+\theta_T k^T\right)}
	  $$
	2.Beta滞后项：
	$$
	  B(k;\theta)=\frac{f\left(\theta_1, \theta_2 ; k / K\right)}{\sum_{k=1}^K f\left(\theta_1, \theta_2 ; k / K\right)}
	  $$ 	  
	$$
	  \begin{gathered}
	  f\left(\theta_1, \theta_2 ; k / K\right)=\frac{\Gamma\left(\theta_1+\theta_2\right)}{\Gamma\left(\theta_1\right) \Gamma\left(\theta_2\right)}\left(\frac{k}{K}\right)^{\theta_1-1}\left(1-\frac{k}{K}\right)^{\theta_2-1} \\
	  \Gamma\left(\delta_1\right)=\int_0^{\infty} e^{-k / K}\left(\frac{k}{K}\right)^{\theta_1-1} d\left(\frac{k}{K}\right), \Gamma\left(\delta_2\right)=\int_0^{\infty} e^{-k / K}\left(\frac{k}{K}\right)^{\theta_2-1} d\left(\frac{k}{K}\right)
	  \end{gathered}
	  $$ 	 
进一步地，我们可以通过在MIDAS模型中加入因变量的滞后项来进一步描述其自相关关系，即我们可以得到如下的AR-MIDAS模型：
	考虑在 $m=3$ 的基本模型中, 加入低频因变量 $y_{t_m}$ 的一个滞后项 $y_{t_m-3}$ (其中 $x$ 为月度变量, $y$ 为季度变量):
	$$
	  y_{t_m}=\beta_0+\lambda y_{t_m-3}+\beta_1 B\left(L_m ; \theta\right) x_{t_m+k-3}^{(3)}+\varepsilon_{t_m}
	  $$ 
   正如 Clements 和 Galvão (2009) 指出的, 这种方法一般来说是不恰当的, 注意到：  
	$$
	  y_{t_m}=\beta_0(1-\lambda)^{-1}+\beta_1\left(1-\lambda L_m^3\right)^{-1} B\left(L_m ; \theta\right) x_{t_m+w-3}^{(3)}+\left(1-\lambda L_m^3\right)^{-1} \varepsilon_{t_m}
	  $$ 	  
   变量 $x_{t-1}^{(3)}$ 的系数多项式为 $L^{1 / 3}$ 的多项式和 $L$ 的多项式的乘积(可以从接近无穷递缩等比数列的思路求出来这个结果)。这个乘积会造成 $y$ 对 $x^{(3)}$ 的季节性反应，不管 $x^{(3)}$ 有没有呈现季节性模式。为避免这一点, 作者建议引入一个向量自回归项作为共同因子:  
	$$
	  y_{l_m}=\beta_0+\lambda y_{t_m-3}+\beta_1 b\left(L_m ; \theta\right)\left(1-\lambda L_m^3\right) x_{t_m+n-3}^{(3)}+\varepsilon_{t_m}
	  $$ 	  
   这样, $y$ 对 $x^{(3)}$ 的反应就不再是季节性的了。相应地 $h$ -步向前预测模型可以写成:  
	$$
	  y_{t_m}=\beta_0+\lambda y_{t_m-h_m}+\beta_1 b\left(L_m ; \theta\right)\left(1-\lambda L_m^{h_m}\right) x_{t_m+w-h_m}^{(3)}+\varepsilon_{t_m}
	  $$
   除了加入因变量的滞后之外，我们还可以考虑引入更多的自变量，即Multi-MIDAS:
	$$
	  y_{t_m}=\beta_0+\beta_1 b\left(L_m ; \theta\right) x_{1 ,t_m+w-h_m}^{(m)}+\beta_2 b\left(L_m ; \theta\right) x_{2 , t_m+w-h_m}^{(m)}+\varepsilon_{t_m}
	  $$
   以基于月度数据预测季度GDP为例，我们此时只得到了每个季度对应前3个月度的关系是什么。进一步的，基于rolling windows以及recursive方法，我们可以基于 $X-2, X-1, X$ 月的数据得到 $X$ 月的GDP值，即我们相当于提取出了一个每月更新的季度GDP增长率。
