# MIDAS_python
Python package for MIDAS, bugs fixed based on midas_pro(https://github.com/sapphire921/midas_pro)
Enabling Multi-MIDAS and AR-MIDAS(on both recursive and rolling basis), high-frequency indicators considered can have different theta parameters, different sampling frequencies and different lag length.

* Assume that a variable $y_1$ can be observed once in the time interval from $t-1$ to $t$ (e.g., quarterly), while another variable $x_t^{(m)}$ can be observed $m$ times in the same interval. We are interested in the dynamic relationship between $y_t$ and $x_t^{(m)}$, or in other words, we want to project the left side variable of the regression equation $y_t$ onto the historical sequence of the right side variable $x_t^{(m)}$ and its lagged observations $x_{t-j / m}^{(m)}$. The superscript $m$ in $x_{t-j / m}^{(m)}$ indicates a higher sampling frequency, with the exact lag time expressed as a fraction of the unit interval from $t-1$ to $t$. A simple MIDAS model can be expressed as follows:
  $$Y_t=\beta_0+\beta_1 B\left(L^{1 / m} ; \theta\right) x_t^{(m)}+\varepsilon_t^{(m)}$$
  where $t=1, \cdots, T$, $B\left(L^{1 / m} ; \theta\right)=\sum_{k=0} B(k ; \theta) L^{k / m}$, $L^{1 / m}x_t^{(m)}=x_{t-1 / m}^{(m)}$
In practice, it is generally assumed that the function form $B$ of $B(k ; \theta)$ is known but the specific values of the parameter $\theta$ need to be estimated. Thus, the coefficients before the lagged terms are transformed into a parameter vector $\theta$.

* There are two common forms for estimating the function $B(k ; \theta)$:

    1. Almon Lag:
     $$B(k ; \theta)=\frac{\exp \left(\theta_1 k+\cdots+\theta_T k^T\right)}{\sum_{k=1} \exp \left(\theta_1 k+\cdots+\theta_T k^T\right)}$$
     
  2. Beta Lag:
     $$B(k;\theta)=\frac{f\left(\theta_1, \theta_2 ; k / K\right)}{\sum_{k=1} f\left(\theta_1, \theta_2 ; k / K\right)}$$
     
     Where,
     $$f\left(\theta_1, \theta_2 ; k / K\right)=\frac{\Gamma\left(\theta_1+\theta_2\right)}{\Gamma\left(\theta_1\right) \Gamma\left(\theta_2\right)}\left(\frac{k}{K}\right)^{\theta_1-1}\left(1-\frac{k}{K}\right)^{\theta_2-1}$$
     $$\Gamma\left(\delta_1\right)=\int_0^{\infty} e^{-k / K}\left(\frac{k}{K}\right)^{\theta_1-1} d\left(\frac{k}{K}\right), \Gamma\left(\delta_2\right)=\int_0^{\infty} e^{-k / K}\left(\frac{k}{K}\right)^{\theta_2-1} d\left(\frac{k}{K}\right)$$

* Further, we can describe the autocorrelation of the dependent variable by adding lagged terms of the dependent variable into the MIDAS model, leading to the AR-MIDAS model:
Consider adding a lagged term of the low-frequency dependent variable $y_{t_m}$, $y_{t_m-3}$, to the basic model at $m=3$ (where $x$ is a monthly variable and $y$ is a quarterly variable):
  $$y_{t_m}=\beta_0+\lambda y_{t_m-3}+\beta_1 B\left(L_m ; \theta\right) x_{t_m+k-3}^{(3)}+\varepsilon_{t_m}$$

  As Clements and Galvão (2009) noted, this approach is generally inappropriate. Note that:
  $$y_{t_m}=\beta_0(1-\lambda)^{-1}+\beta_1\left(1-\lambda L_m^3\right)^{-1} B\left(L_m ; \theta\right) x_{t_m+w-3}^{(3)}+\left(1-\lambda L_m^3\right)^{-1} \varepsilon_{t_m}$$

  The coefficient polynomial of the variable $x_{t-1}^{(3)}$ is a product of a polynomial in $L^{1 / 3}$ and a polynomial in $L$ (which can be derived from the approach of an almost infinitely shrinking geometric sequence). This product causes $y$ to react seasonally to $x^{(3)}$, regardless of whether $x^{(3)}$ exhibits a seasonal pattern. To avoid this, the authors suggest introducing a vector autoregression term as a common factor:
  $$y_{l_m}=\beta_0+\lambda y_{t_m-3}+\beta_1 b\left(L_m ; \theta\right)\left(1-\lambda L_m^3\right) x_{t_m+n-3}^{(3)}+\varepsilon_{t_m}$$

* Besides adding lagged terms of the dependent variable, we can also consider introducing more independent variables, namely Multi-MIDAS:
  $$y_{t_m}=\beta_0+\beta_1 b\left(L_m ; \theta\right) x_{1 ,t_m+w-h_m}^{(m)}+\beta_2 b\left(L_m ; \theta\right) x_{2 , t_m+w-h_m}^{(m)}+\varepsilon_{t_m}$$

* Taking the example of predicting quarterly GDP based on monthly data, we have only determined the relationship between each quarter and the preceding three months. Furthermore, based on rolling windows and recursive methods, we can use data from months $X-2$, $X-1$, and $X$ to predict the GDP for month $X$, essentially extracting a monthly updated quarterly GDP growth rate.
