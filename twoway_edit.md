데이터 분석가로서 이메일 등록이 고객 참여에 미치는 영향을 분석하는 임무가 주어졌습니다. 어떻게 분석을 해야 할까요? 이메일 등록은 유저마다 다른 시점에서 시작하기 때문에 점진적인 이중차분법 (staggered Difference-in-Difference) 을 적용할 수 있습니다. 오늘 포스팅에서는 staggered difference-in-difference 디자인을 이해하기 위해 필요한 2요인 고정효과 모형 (two-way fixed effects model) 을 다루도록 하겠습니다. 

또 다른 예로, AB 테스팅 툴의 보급이 기업의 퍼포먼스에 미치는 효과를 추정하고 싶다고 합시다. AB 테스팅 툴을 도입하는 시기도 기업마다 제각각입니다. 만약 도입한 기업과 도입하지 않은 기업을 단순 비교하면, 추정치에는 AB 테스팅의 효과 뿐만 아니라 도입한 기업의 내재된 특성으로 인해 발생하는 효과도 함께 포함됩니다. 예를 들어서, 내재된 특성이 좋은 기업이 AB 테스팅도 도입한다면, AB 테스팅 도입 때문이 아니라 퍼포먼스가 좋을 특성들을 가진 기업들이 AB 테스팅을 도입했기 때문에 퍼포먼스가 높게 나올 수 있습니다. 그리고, 시기에 따라서 스타트업의 퍼포먼스가 다르게 나타날 가능성이 존재합니다. 특정 시기에 기업들의 퍼포먼스도 좋아지는데, 그 시기에 맞물려서 기업의 AB 테스팅 도입이 증가할 수 있기 때문입니다. 

오늘 포스팅의 주제인 **2요인 고정효과 모형 (two-way fixed effects model)** 은 기업의 개별 특성과 시기의 특성들을 통제합니다. 

이번 포스팅에서는 데이터가 존재하는 논문을 바탕으로 2요인 고정효과 모형을 추정하는 구체적인 방법을 다룹니다. 그리고 계량경제학적인 수식을 살펴보고, AWS 에서 약 1GB 크기의 약 7백만 개의 관측치를 저장하고, 이를 SQL 을 통해서 추출한 후에 파이썬에서 직접 실습해볼 수 있도록 하는 가이드도 함께 제공합니다. 

### 논문의 사례 예시 (데이터와 함께)

---

A/B 테스팅 툴의 도입이 스타트업 성과에 미치는 요인에 관한 논문  [(Koning et al. 2022 Management Science)](https://www.hbs.edu/ris/Publication%20Files/AB_Testing_R_R_08b97538-ed3f-413e-bc38-c239b175d868.pdf) 을 중심으로 논의를 진행하겠습니다. 논문에서 사용된 데이터는 Management Science 저널에서 다운로드 받으실 수도 있고, csv 로 변환한 파일을 제 [구글 Colab 폴더](https://drive.google.com/drive/folders/1NDOJ11OsoNKg_qU_xDx_to8kf2ksJpeq?usp=sharing) 에서도 확인하실 수 있습니다. 

__1. 데이터 설명__

저자는 Crunchbase, SimilarWeb, BuiltWith 등으로부터 기업에 관한 정보를 수집합니다. 35,6262개의 기업에 대해서 2015년 4월 5일 부터 2019년 3월 24일까지 208개의 주 (week) 에 대해서 총 7,3334,496개의 기업-주 단위 관측치가 수집되었습니다. 

__2. 변수 설명__

논문의 여러 연구 질문 중에서 이번 포스팅에서는 AB 테스팅이 스타트업의 퍼포먼스에 미치는 영향을 2요인 고정효과 모형 (two-way fixed effect model) 를 이용해서 분석한 내용을 다루겠습니다. AB 테스팅 사용 여부는 해당 기업이 AB Tasty, Adobe Target Standard, Experimentl.ly, Google Optimize, Google Website Optimizer, Omniture Adobe Test and Target, Optimizely, Optimost, Split Optimizer, Visual Website Optimizer 를 사용했는지 여부입니다. 이 때, 고려된 툴은 AB 테스팅에 포커스가 되어 있기 때문에 Mixpanel 과 같은 다른 분석 툴은 대상에서 제외되었습니다. 논문에 따르면, 해당 기간 동안 약 18%의 기업에서 적어도 일주일의 AB 테스팅 경험이 있다고 합니다. 

기술 스택 (Technology Stack) 은 AB 테스팅 이외에 기업에서 채택한 기술의 개수입니다. 예를 들어, 페이스북의 픽셀 트래킹과 같은 기술입니다. 

__3. 회귀식__

2요인 고정효과 모형 (two-way fixed effects model) 을 기반으로한 점진적인 이중차분 모형을 소개하겠습니다. 회귀식의 단위는 기업-주 입니다. 

$$
  y_{it} = \beta ABTesting_{it} + \theta TechStack_{it} + \alpha_i + \gamma_t + \varepsilon_{it}
$$
... (1) 
- $y_{it}$: ($t$ 주에 기업 $i$ 의) 방문자수의 로그값
- $ABTesting_{it}$: ($t$ 주에 기업 $i$ 의) AB 테스팅 도입 여부
- $TechStack_{it}$: ($t$ 주에 기업 $i$ 의) AB 테스팅 이외의 기술 스택의 개수의 로그값
- $\beta$: AB 테스팅 도입이 스타트업 퍼포먼스에 미친 효과
- $\alpha_i$: 기업의 고정효과
- $\gamma_t$: 주 (week) 고정효과

$\gamma_t$ 는 주 (week) 마다 스타트업의 퍼포먼스에 미치는 영향을 통제합니다. 특정 주에 경제 환경이 다르거나 인터넷 사용량이 달랐거나 하는 등의 요인을 통제합니다. $\alpha_i$ 는 변하지 않는 기업 고유의 특징들을 통제합니다. 스타트업의 초반 아이디어의 퀄리티가 다르거나, 특별한 전략의 유무나, 장소적인 이점이 있거나, 파운더의 학력 등의 관측되지 않는 특성들이 AB 테스팅 도입 여부와 스타트업의 퍼포먼스에 모두 관련이 있을 수 있습니다. $\alpha_i$ 는 이를 통제합니다. 이 다음 섹션에서는 이러한 특징들이 demeaning 을 통해서 수리적으로 제거됨을 보입니다.

이와 더불어, 주마다 변화하는 스타트업의 기술 스택 ($TechStack_{it}$) 을 통제합니다. AB 테스팅을 도입할 때 스타트업 퍼포먼스에 영향을 미치는 다른 기술을 함께 도입했다면, 추정치에 편의가 발생하기 때문입니다. 

__4. 평균 빼주기 (Demeaning)__

이번 섹션은 계량경제학적인 내용입니다. 각각의 변수에서 고정효과 ($\alpha_i$ 와 $\gamma_t$) 를 제거하는 방법을 수리적으로 표현합니다. 2요인 고정효과 모형에서는 기업의 고정효과와 시간의 고정효과를 모두 제거하기 위해서 아래와 같이 수식 (1) - (2) - (3) + (4) 를 해서 도출한 수식 (5) 를 생성한 후 수식의 좌우변의 변수들에 대해서 간단한 OLS 회귀분석을 진행합니다. 

이러한 수리적인 테크닉을 소개하는 이유는 고정효과에 관한 라이브러리를 실행할 때 시간이 오래 걸리기 때문에 이를 단축시키기 위해서입니다. 

$$
  \frac{1}{I} \sum_i y_{it} = \beta \frac{1}{I} \sum_i ABTesting_{it} + \theta \frac{1}{I} \sum_i TechStack_{it} + \frac{1}{I} \sum_i \alpha_i + \frac{1}{I} \sum_i \gamma_t + \frac{1}{I} \sum_i \varepsilon_{it}
$$
... (2) 
$$
  \frac{1}{T} \sum_t y_{it} = \beta \frac{1}{T} \sum_t ABTesting_{it} + \theta \frac{1}{T} \sum_t TechStack_{it} + \frac{1}{T} \sum_t \alpha_i + \frac{1}{T} \sum_t \gamma_t + \frac{1}{T} \sum_t \varepsilon_{it}
$$
... (3) 
$$
  \frac{1}{I} \frac{1}{T} \sum_i \sum_t y_{it} = \beta \frac{1}{I} \frac{1}{T} \sum_i \sum_t ABTesting_{it} + \theta \frac{1}{I} \frac{1}{T} \sum_i \sum_t TechStack_{it} + \frac{1}{I}\frac{1}{T} \sum_i \sum_t \alpha_i + \frac{1}{I} \frac{1}{T} \sum_i \sum_t \gamma_t + \frac{1}{I} \frac{1}{T} \sum_i \sum_t \varepsilon_{it}
$$
... (4) 

(1) - (2) - (3) + (4) 를 하면, 

$$
y_{it} - \overline{y_i}  - \overline{y_t}  + \overline{y} = \beta \Big( ABTesting_{it} - \overline{ ABTesting_{i} } - \overline{ ABTesting_{t} } + \overline{ ABTesting }  \Big) + \theta \Big( TechStack_{it} - \overline{ TechStack_{i} } - \overline{ TechStack_{t} } + \overline{TechStack}  \Big) + v_{it}
$$
... (5)

$v_{it} = \varepsilon_{it} - \frac{1}{I} \sum_i \varepsilon_{it} - \frac{1}{T} \sum_t \varepsilon_{it} + \frac{1}{I} \frac{1}{T} \sum_i \sum_t \varepsilon_{it} $ 

$\beta$ 값에 편의가 없기 위해서는 잔차인 $v_{it}$ 와 $\Big( ABTesting_{it} - \overline{ ABTesting_{i} } - \overline{ ABTesting_{t} } + \overline{ ABTesting }  \Big)$ 이 독립적이어야 합니다. 

즉 $v_{it}$ 에 "기술 스택"의 경우처럼 시간에 따라 변화하면서 AB 테스팅과 퍼포먼스에 모두 영향을 미치는 요소가 포함되어 있다면, $\beta$ 값에는 편의가 발생합니다. 이러한 한계점과 해소 방안에 대한 논의는 "한계점과 추가적인 방안" 섹션에서 논의하겠습니다. 이러한 요소가 더 이상 없거나 적다고 가정을 한 상태에서 $\beta$ 값은 인과적인 효과를 추정합니다. 

### 코드를 통해서 실행해보기

---

AWS S3 에 원본 데이터를 저장하고, 이를 SQL 을 통해서 평균값을 빼주는 작업을 미리 해준 후에, Python 에서 간단한 OLS 회귀식을 돌려보도록 하겠습니다. 사전 작업에 대해서 간단히 말씀드리면, AWS S3 에 ab_data.csv (논문의 ab_data.dta 를 csv 파일로 변환한 데이터. 제 [구글 Colab 폴더](https://drive.google.com/drive/folders/1NDOJ11OsoNKg_qU_xDx_to8kf2ksJpeq?usp=sharing) 의 raw 폴더 에 저장해두었습니다) 를 AWS S3 에 저장한 후, 이를 AWS Glue 를 이용해서 크롤링한 후에 AWS Athena 에서 아래와 같은 SQL 코드를 실행하였습니다. AWS S3 에서 Glue 를 거쳐 AWS Athena 를 실행하는 과정에 관한 간략한 흐름은 제 블로그에 작성해둔 [링크](https://marvin-ds.tistory.com/252)를 참고바랍니다. 

만약 AWS 설치가 어려운 분들의 경우에는 [구글 Colab 폴더](https://drive.google.com/drive/folders/1NDOJ11OsoNKg_qU_xDx_to8kf2ksJpeq?usp=sharing) 에서 pap3rdDiDtable2.ipynb 의 파일 참고 바랍니다. SQL 의 demeaning 하는 코드를 Colab 파이썬 코드에서 실행할 수 있도록 해두었습니다. 

__1. SQL 을 이용한 데이터 추출__

AWS S3 에 로드된 데이터를 AWS Glue 를 거쳐서 AWS Athena 에 업로드하셨다면, 아래와 같은 SQL 코드를 실행하여, demean 이 된 변수를 생성할 수 있습니다. 

```
SELECT week_id, firm_id, log1p_pageviews, using_ab_only, log1p_stack2,
    log1p_pageviews - AVG(log1p_pageviews) OVER (PARTITION BY week_id) AS demW_log1p_pageviews,
    using_ab_only - AVG(using_ab_only) OVER (PARTITION BY week_id) AS demW_using_ab_only,
    log1p_stack2 - AVG(log1p_stack2) OVER (PARTITION BY week_id) AS demW_log1p_stack2,
    log1p_pageviews - AVG(log1p_pageviews) OVER (PARTITION BY firm_id) AS demF_log1p_pageviews,
    using_ab_only - AVG(using_ab_only) OVER (PARTITION BY firm_id) AS demF_using_ab_only,
    log1p_stack2 - AVG(log1p_stack2) OVER (PARTITION BY firm_id) AS demF_log1p_stack2,
    log1p_pageviews - AVG(log1p_pageviews) OVER () +
    AVG(log1p_pageviews) OVER () - AVG(log1p_pageviews) OVER (PARTITION BY week_id) -
    AVG(log1p_pageviews) OVER (PARTITION BY firm_id) AS demWF_log1p_pageviews,
    using_ab_only - AVG(using_ab_only) OVER () +
    AVG(using_ab_only) OVER () - AVG(using_ab_only) OVER (PARTITION BY week_id) -
    AVG(using_ab_only) OVER (PARTITION BY firm_id) AS demWF_using_ab_only,
    log1p_stack2 - AVG(log1p_stack2) OVER () +
    AVG(log1p_stack2) OVER () - AVG(log1p_stack2) OVER (PARTITION BY week_id) -
    AVG(log1p_stack2) OVER (PARTITION BY firm_id) AS demWF_log1p_stack2
FROM paperabtest.paperabtest;
```

- demW_log1p_pageviews: 페이지뷰 로그 값의 각 주마다의 평균값입니다. 
  - 수식 (5) 에서 $\overline{y_t}$ 와 대응. 
- demF_log1p_pageviews: 페이지뷰 로그 값의 각 기업마다의 평균값입니다. 
  - 수식 (5) 의 $\overline{y_i}$ 와 대응.  
- demWF_log1p_pageviews: 페이지뷰 로그 값에서 페이지뷰 로그 값의 각 주마다의 평균값과 페이지뷰 로그 값에서 각 기업마다의 평균값을 뺀 후 페이지뷰 로그 값 전체를 더해준 값입니다. 
  - 수식 (5) 의 $(y_{it} - \overline{y_i}  - \overline{y_t}  - \overline{y})$ 과 대응. 

demW_* , demF_* , demWF_* 변수들도 같은 방식으로 구합니다. 
- demWF_using_ab_only:
 수식 (5) 에서 $( ABTesting_{it} - \overline{ ABTesting_{i} } - \overline{ ABTesting_{t} } + \overline{ ABTesting } )$ 에 대응.
- demWF_log1p_stack2: 
 수식 (5) 에서 $( TechStack_{it} - \overline{ TechStack_{i} } - \overline{ TechStack_{t} } + \overline{TechStack}) $ 에 대응.  

__2. SQL 추출 이후 파이썬__

위의 SQL 코드를 실행한 후에 나오는 결과를 ab_dataDeMean.csv 로 저장한 후에 구글 Colab 에서 파이썬으로 아래와 같은 코드를 실행하였습니다. 관련 코드는 [구글 Colab 폴더](https://drive.google.com/drive/folders/1NDOJ11OsoNKg_qU_xDx_to8kf2ksJpeq?usp=sharing) 에서 pap3rdDiDtable2_SQL.ipynb 파일을 참고 바랍니다. 

우선 csv 파일을 읽은 후에 
```
import pandas as pd
data = pd.read_csv('gdrive/My Drive/PAP/PAP3rd/submit1DID/raw/ab_dataDeMean.csv')
```

논문의 테이블 2 의 네번째 열 (column) 을 실행하는 코드를 가져왔습니다. 
```
# Table (2) Column (4)

# Add dummy variables to explanatory variables
X = sm.add_constant(data[['demWF_using_ab_only','demWF_log1p_stack2']])
X = pd.concat([X], axis=1)

# Fit a linear model with clustered standard errors
ols_model = sm.OLS(data['demWF_log1p_pageviews'], X)
ols_results = ols_model.fit(cov_type='cluster', cov_kwds={'groups': data['firm_id']})

# Print the regression summary
print(ols_results.summary())
```
참고로, Table (2) 의 모든 열 (column) 에 대한 회귀식 파이썬 코드와 그 결과는 앞서 말씀드린 pap3rdDiDtable2_SQL.ipynb 파일 에서 확인할 수 있습니다. 

__3. 추정 결과__

Table (2) 에서는 수식 (1) 과 관련된 회귀식의 결과입니다. 

|           | (1) | (2) | (3) | (4) |
|---|---|---|---|---|
| Using AB Tool | 2.957*** | 0.190*** | 0.553*** | 0.131*** |
|  | (0.046) | (0.024) | (0.026) | (0.022) |
| Observation |  7,334,496 | 7,334,496 | 7,334,496 | 7,334,496 |
| Week FE | V | V | V | V |
| Firm FE |  |  | V | V |
| Tech Stack Control |  | V |  | V |

열 (1) 에서는 주 (week) 고정효과만 컨트롤하고, 기술 스택은 통제하지 않았습니다. 2.957 의 추정치는 AB 테스팅 도입 이후에 방문자가 약 300% 증가한다는 의미입니다. 열 (2) 에서는 열 (1) 의 회귀식에 기술 스택을 추가로 통제하는데, AB 테스팅의 도입 효과는 19%로 나타납니다. 열 (1) 과 열 (2) 를 비교할 때, 시간에 따라 함께 변화하는 요소들 중에서 AB 테스팅 도입과 방문자수 변화에 모두 관련이 있는 요소들을 통제하지 않으면 추정치에 편의가 발생함을 알 수 있습니다. 열 (3) 에서는 열 (1) 의 통제변수와 함께 기업의 고정효과를 추가하여 통제합니다. 열 (1) 과 달리, 기업의 고정효과 통제 후 추정치는 55% 로 낮아졌습니다. 즉, 기업들 중에서 AB 테스팅을 도입하는 기업들은 퍼포먼스가 높아지는데 기여하는 요소들을 내재하고 있음을 추론할 수 있습니다. 마지막으로, 열 (4) 에서는 기업의 고정효과와 기술 스택 도입 여부를 모두 통제합니다. 열 (2) 나 열 (3) 에 비해서도 추정치가 감소합니다. 

### 한계점과 추가적인 방안

---

앞의 회귀식에서는 기업의 선택 편의 (selection bias) 의 다양한 형태를 컨트롤하지만, "기술 스택 개수" 처럼 시간에 따라 변화하는 관측되지 않은 특성들 (unobserved time-varying shocks) 을 모두 통제하지 못합니다. 그리고, 기업의 AB 테스팅 도입이 완전히 임의적이지 않다는 문제도 존재합니다. 

해당 논문에서는 구글 옵티마이즈가 2017년 3월 30일에 실행되는 외생적인 변화를 사용하여 추가적인 분석을 수행합니다. 기업들 중에 사전에 Google's Tag Manager (GTM) 를 사용하는 기업들은 구글 옵티마이즈를 도입하는 빈도가 GTM 을 사용하지 않은 기업의 약 두 배 였다고 합니다. 이러한 요인과 고정효과 모형을 결합하여 difference-in-differences instrumental variable 모형을 적용합니다. 이 부분은 다음 포스팅에서 진행하도록 하겠습니다. 추가로, 해당 논문에서는 synthetic control 기법도 사용하고 있는데, 해당 기법에 대해서도 추후 다루어볼 기회가 있었으면 좋겠습니다. 

### 맺음말

---

이번 포스팅에서는 정책의 도입 시점이 유저나 그룹마다 각기 다를 때, 정책 도입의 효과를 추정하는 방법의 기본이 되는 2요인 고정효과 모형 (two-way fixed effects model) 의 기본적인 내용에 대해서 다루어 보았습니다. 관련해서 추가적으로 공부해볼만한 인과추론 관련 기법은 staggered difference in difference 나 difference-in-differences instrumental variable, synthetic control 입니다. 본문에서 다루지는 못했으나, difference-in-difference 관련해서 최근 계량경제학적인 이슈들이 궁금하신 분들은 참고문헌의 논문들 (특히, Athey and Imbens 2022, Baker et al. 2022, Bilinski et al., Goodman-Bacon 2021) 을 참고 바랍니다. 

본문 내용이나 인과추론의 전반적인 기법에 관해 궁금한 점이 있으시면 doctor.marvin.ds@gmail.com 으로 이메일 부탁드립니다. 인과추론이나 실험 등에 관한 주제에 관심이 있으신 분들은 제 [블로그](https://marvin-ds.tistory.com/) 도 참고 바랍니다. 
감사합니다.

### 참고문헌

---

Athey and Imbens, (2022, Journal of Econometrics), "Design-based analysis in Difference-In-Differences settings with staggered adoption"

Baker et al. (2022 Journal of Financial Economics), "How much should we trust staggered difference-in-differences estimates?"

Bilinski et al. (forthcoming, Journal of Econometrics), "What’s Trending in Difference-in-Differences? A Synthesis of the Recent Econometrics Literature"

Goodman-Bacon (2021, Journal of Econometrics), "Difference-in-differences with variation in treatment timing"

Koning et al. (2022, Management Science), "Experimentation and startup performance: Evidence from A/B testing"

Song and Imai (2021, Political Analysis), "On the Use of Two-Way Fixed Effects Regression Models for Causal Inference with Panel Data"
