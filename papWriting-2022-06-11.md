---
title: 매칭 (Subclassification)
slug: matching-subclassification
category: Causal Inference
author: "정원혁"

tags: ["중급"]
date: 2022-06-11
---

## 시작하며 

아래 표에서는 흡연여부에 따른 사망률을 나타냅니다. 

<center>
| 흡연그룹 | 사망률 | 
| ------ | ------ |
| 비흡연자 | 6.4% |
| 흡연자 | 6.4% |
</center>

주장이 타당하기 위해서는 아래 가정을 만족해야 합니다. 

$$E[Y^1 | 비흡연자] = E[Y^1 | 흡연자]$$
$$E[Y^0 | 비흡연자] = E[Y^0 | 흡연자]$$

즉, 흡연자가 

## 마무리하며

위  글은 저의 [개인블로그](https://marvin-ds.tistory.com/15)에서도 읽으실 수 있습니다. 

## Reference

- [Causal Mixtape, "Matching and Subclassification"](https://mixtape.scunning.com/matching-and-subclassification.html)