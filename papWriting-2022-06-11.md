---
title: 매칭 (Subclassification)
slug: matching-subclassification
category: Causal Inference
author: "정원혁"

tags: ["중급"]
date: 2022-06-11
---

## Subclassification
### 예시: 흡연과 사망률

아래 표에서는 흡연여부에 따른 사망률을 나타냅니다. 이 표를 보고 과연 흡연이 사망에 영향을 미치지 않는다고 주장할 수 있을까요? 

| 흡연그룹 | 사망률 | 
| ------ | ------ |
| 비흡연자 | 6.4% |
| 흡연자 | 6.4% |


주장이 타당하기 위해서는 아래 가정을 만족해야 합니다. 

$$E[Y^1 | 비흡연자] = E[Y^1 | 흡연자]$$
$$E[Y^0 | 비흡연자] = E[Y^0 | 흡연자]$$

즉, 흡연자가 담배를 피웠을 때의 사망률과 비흡연자가 담배를 피웠을 때의 사망률이 동일 하고, 흡연자가 담배를 피우지 않았을 떄의 사망률과 비흡연자가 담배를 피우지 않았을 때의 사망률이 동일해야 합니다.

다음 표에서처럼, 비흡연자의 평균 나이가 흡연자의 평균 나이에 비해 높았다고 해봅시다. 

| 흡연그룹 | 30세 | 50세 | 
| ------ | ------ | ------ |
| 비흡연자 | 600명 | 900명 |
| 흡연자 | 400명 | 100명 |

- 비흡연자의 평균 나이: 
$$ 30 \times \frac{600}{600+900} + 50 \times \frac{900}{600+900} = 42 $$

- 흡연자의 평균 나이: 
$$ 30 \times \frac{400}{400+100} + 50 \times \frac{100}{400+100} = 34 $$

비흡연자의 평균 연령이 흡연자의 평균 연령에 비해 높다는 것은 두 집단 모두 담배를 피 우지 않았더라도 사망률이 비흡연자의 경우 더 높았을 것을 의미합니다 (연령이 높아질수록 흡연자와 비흡연자 두 집단 모두에서 사망률이 높아진다고 가정하겠습니다). 다음 표에서처럼, 실재로 실제 사망률은 각 연령 집단에서 흡연자의 경우가 비흡연자의 경우에 비해 더 높았다고 해봅시다. 

| 흡연그룹 | 30세 | 50세 | 
| ------ | ------ | ------ |
| 비흡연자 | 1% | 10% |
| 흡연자 | 5% | 12% |

가장 처음 표의 사망률에 문제가 있었던 이유는 연령이라는 요소를 고려하지 않고 단순 계산 했기 때문입니다. 즉, 아래와 같은 방식으로 각 집단의 전체 명수를 해당 집단 중 사망자 수로 단순 계산하면 두 집단의 사망률이 동일하게 나옵니다.

- 비흡연자의 사망률 단순 계산: 
$$\frac{ 1 \\% \times 600 + 10 \\% \times 900 }{ 600 + 900} = 6.4 \\% $$

- 흡연자의 사망률 단순 계산: 
$$\frac{ 5 \\% \times 400 + 12 \\% \times 100 }{ 400 + 100} = 6.4 \\% $$

### DAG 그래프

사망률을 단순 계산할 경우 흡연자와 비흡연자의 사망률이 동일하게 나오는 이유는 아래 DAG 표와 같이 흡연 여부가 연령과 연관되기 때문입니다. 그리고, 위 표들과 같이, 흡연 여부에 따라서 연령 분포가 다르기 때문에 문제가 발생합니다.

![](./fig1_DAG.png)

### Identifying Assumption

흡연 여부가 사망률에 미치는 인과적 효과를 추정하기 위해서 아래 두 가정을 가정해봅니다.

1. CIA (Conditional Independent Assumption)
    - $(Y^1, Y^0 \perp D | X) $

2. Common Support Assumption
    - $0 < P(D=1 | X) < 1$ with probability one.

두 가정 하에서, 

$$ E[Y^1 - Y^0 | X ] = E [ Y | X, D=1] - E [ Y | X, D=0] $$
$$  = E [ Y^1 | X, D = 1] - E [ Y^0 | X, D = 0] $$
$$ = E [ Y | X, D = 1] - E [ Y | X, D = 0] $$
		
### Calculating Average Treatment Effect

위 예시에서 ATE 를 계산해봅니다. 

$$\hat{\delta}_{ATE}  = \int \Big( E[Y^1 - Y^0 | X ] \Big) d P(X) $$
$$ = \int \Big( E [ Y | X, D=1] -  E [ Y | X, D=0]  \Big) d P(X) $$
$$ = \Big[ E [Y | Age = 30, D= 1] - E [Y | Age = 30, D= 0]  \Big] \times P(Age = 30) $$
$$ + \Big[ E [Y | Age = 50, D= 1] - E [Y | Age = 50, D= 0]  \Big] \times P(Age = 50) $$
$$ = (5  - 1 \\%) \times \frac{600+400}{2000} + (12 \\% - 10 \\%) \times \frac{900+100}{2000} $$
$$ = 3 \\% $$

위의 사례에서는 characteristic 이 연령 하나였습니다. 만약, characteristic 이 많아진다면 어떻게 될까요? 예를 들어, 10개의 binary (0 또는 1)의 변수가 있으면, 210 = 1, 024 개의 subclass 를 만들어야 합니다. 개별 subclass 에 각각 treatment 그룹과 control 그룹이 모두 있으리라 보장하기 어렵습니다.

- 즉, Curse of dimensionality 문제가 발생합니다.

## Propensity Score Matching

### Introduction

PSM 기법은 curse of dimensionality 문제를 해소하고자 treatment 될 확률이라는 (벡터가 아 닌) 스칼라 값 하나로 표현하는 방법입니다. PSM 을 이용해 두 집단의 인과관계를 추론하는 스텝은 다음과 같습니다.

### 예시: 직업훈련

아래 표에서는 데이터 별 단순 이중차분 결과를 나타냅니다. Experimental 데이터는 실제 실험이 이루어진 데이터에 근거한 이중차분 결과로 참값입니다. 이와 달리, PSID 와 CPS 는 매칭없이 단순 이중차분을 하였습니다. Experimental 데이터와 달리, PSID 결과처럼 음수 가 나오거나, CPS 의 결과처럼 약간 양수여도 Experimental 결과값보다 작고, 통계적으로 유의하지 않습니다.

## 마무리하며

위  글은 저의 [개인블로그](https://marvin-ds.tistory.com/15)에서도 읽으실 수 있습니다. 

## Reference

- [Causal Mixtape, "Matching and Subclassification"](https://mixtape.scunning.com/matching-and-subclassification.html)