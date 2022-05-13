---
title: 샘플사이즈 계산기의 공식 이해 및 파이썬 적용
slug: sample-size-formula
category: Experiment
author: "정원혁"

tags: ["중급"]
date: 2022-05-14
---



## 시작하며 

AB 테스팅 스터디 리드를 준비하면서 sample size 계산에 대한 수식적인 내용을 보충해야겠다는 생각이 들었습니다. 책의 Ch02 에서 샘플 사이즈를 계산하기 위해서는 baseline 의 평균과 표준편차를 알아야 한다는 내용이 나옵니다.

## 샘플 사이즈 계산 시작하기

최적의 샘플 사이즈를 구하기 위해서는 귀무가설과 대립가설이 필요합니다. 귀무가설은 처치의 효과가 없다이고, 대립가설은 처치의 효과가 나타나기를 기대하는 최솟값 이상입니다. 최적의 샘플 사이즈를 구하기 위해서는 귀무가설이 참일 때, 이를 잘못 기각하지 않아야 하고 (두 집단 사이에 차이가 없는데, 두 집단 사이에 차이가 있다고 잘못 판단하는 경우), 대립가설이 참일 때, 귀무가설을 잘못 채택해서는 안됩니다 (두 집단 사이에 우리가 기대하는 최솟값이 나타날 때 두 집단 사이에 효과가 없다고 판단하는 경우). 

통제집단의 outcome 분포를 $Y_{i0} \sim N(\mu_0, \sigma_0^2)$이라 하고, 처치집단의 outcome 분포를 $Y_{i1} \sim N(\mu_1, \sigma_1^2)$라 합시다. 아직 실험이 시작하기 전에 이 값들을 구해야하기 때문에, 기존 데이터에서 평균과 분산을 가져 오거나, 파일럿 실험을 통해서 이를 구합니다. 더불어서, 최소한의 처치 집단의 효과를 정합니다 ($\mu_1 - \mu_0 = \delta$). 추가적으로, 처치의 효과가 t-test 를 통해서 결정된다고 가정합니다. 

귀무가설과 대립가설은 다음과 같습니다.
$$
H_0 : \mu_0 = \mu_1
$$
$$
H_1 : \mu_0 \ne \mu_1
$$

두 표본분포의 평균은 다음과 같습니다.
$$
E[\bar{Y_1}-\bar{Y_0}] = \mu_1 - \mu_0
$$
그러면, 
$$
\frac{ \bar{Y_1}-\bar{Y_0} - E[\bar{Y_1}-\bar{Y_0}] }{ std(\bar{Y_1}-\bar{Y_0} ) } = \frac{ \bar{Y_1}-\bar{Y_0} - (\mu_1 - \mu_0) }{ std(\bar{Y_1}-\bar{Y_0} ) } 
$$

귀무가설 $H_0$에서는, $E[\bar{Y_1}-\bar{Y_0}] = \mu_1 - \mu_0 =0$이고, 대립가설 $H_1$ 에서는, $E[\bar{Y_1}-\bar{Y_0}] = \mu_1 - \mu_0 = \delta$ 이며,

$$
Var(\bar{Y_1}-\bar{Y_0}) = Var(\bar{Y_1})+Var(\bar{Y_0}) = \frac{Var(Y_1)}{n_1} + \frac{Var(Y_0)}{n_0} = \frac{\sigma^2_1}{n_1}+\frac{\sigma^2_0}{n_0} 
$$

이므로, 

귀무가설 $H_0$에서는, 
$$
\frac{\bar{Y_1}-\bar{Y_0} }{std(\bar{Y_1}-\bar{Y_0} )} = \frac{\bar{Y_1}-\bar{Y_0} }{ \sqrt{ \frac{\sigma^2_1}{n_1}+\frac{\sigma^2_0}{n_0}  } }  = t_{\frac{\alpha}{2}}
$$

$$
	\Rightarrow \bar{Y_1}-\bar{Y_0}=  t_{\frac{\alpha}{2}} \times \sqrt{ \frac{\sigma^2_1}{n_1}+\frac{\sigma^2_0}{n_0} }
$$
이고, 대립가설 $H_1$ 에서는, 
$$
\frac{\bar{Y_1}-\bar{Y_0} - \delta }{ \sqrt{ \frac{\sigma^2_1}{n_1}+\frac{\sigma^2_0}{n_0}  } }  = -t_{\beta}
$$
$$
	\Rightarrow \bar{Y_1}-\bar{Y_0}=  \delta - t_{\beta} \times \sqrt{ \frac{\sigma^2_1}{n_1}+\frac{\sigma^2_0}{n_0} }
$$


$$
\delta = ( t_{\frac{\alpha}{2}} + t_{\beta} ) \times \sqrt{ \frac{\sigma^2_1}{n_1}+\frac{\sigma^2_0}{n_0} } 
$$

추가적으로 $\sigma_0^2 = \sigma_1^2 = \sigma^2$, and $n_0 = n_1 = n$ 라고 가정하면, 

$$
	n_0^* = n_1^* = n^* = 2 ( t_{ \frac{\alpha}{2}} + t_\beta )^2 \Big( \frac{\sigma}{\delta} \Big)^2
$$
입니다. 

## 확률

이전의 사례와 달리, 구매전환율과 같은 비율의 차이를 구하는 경우에는 조금 다릅니다. 

$$
H_0: p_0 = p_1
$$
$$
H_1: p_0 \ne p_1
$$

분산은 $p(1-p)$ 입니다. 

귀무가설 $H_0$에서는, 

$$
\frac{ \hat{p_0} - \hat{p_1} - 0 }{ \sqrt{ \frac{ \bar{p} (1-\bar{p}) }{ n_0 } + \frac{ \bar{p} (1-\bar{p}) }{ n_1 } }  } = t_{\frac{\alpha}{2}}
$$

이고, 대립가설 $H_1$ 에서는, 
$$
\frac{ \hat{p_0} - \hat{p_1} - \delta }{ \sqrt{ \frac{ p_0 (1- p_0) }{ n_0 } + \frac{ p_1 (1- p_1) }{ n_1 } }  } = -t_{\beta}
$$

추가적으로 $n_0=n_1=n$로 가정하면, 

$$
t_{\frac{\alpha}{2}} \times \sqrt{ \frac{ 2 \bar{p}(1-\bar{p} }{n} } = \delta - t_{\beta} \sqrt{ \frac{ p_0 (1-p_0) + p_1 (1-p_1) }{n} }
$$

$$
n = \Big( t_{\frac{\alpha}{2}} \sqrt{ 2 \bar{p} (1-\bar{p})} + t_\beta \sqrt{ p_0 (1-p_0) + p_1 (1-p_1) }    \Big)^2 \delta^{-2}
$$

```python
import math

p0 = 0.05
mde = 0.10
p1 = p0*(1+mde)
pbar = (p0+p1)/2

alpha=0.95
beta=0.8
t_alpha2=1.96
t_beta = 0.84
delta = p1-p0
```

```
n=(t_alpha2*math.sqrt(2*pbar*(1-pbar))+t_beta*math.sqrt(p0*(1-p0)+p1*(1-p1)))**2*(1/delta)**2
n
```

[ipython](https://colab.research.google.com/drive/1q1vbsst8WsKir8X44r50UqHlnhNdbfoP#scrollTo=F-U4mccBjARj)에 동일한 식을 적어두었습니다.

## 마무리하며

한 문장으로 요약하면, 

위  글은 저의 

## 레퍼런스 



