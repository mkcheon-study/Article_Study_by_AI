


> Written with [StackEdit](https://stackedit.io/).

# 노트북 LM 기반 C-SIDE 공부

# 1 요약

제시된 소스들을 바탕으로 **C-SIDE(Cell type-Specific Inference of Differential Expression)** 논문의 핵심 내용을 요약해 드립니다.

### 1. C-SIDE의 개요 및 목적

**C-SIDE**는 공간 전사체(Spatial Transcriptomics) 데이터에서 **특정 세포 유형 내에서의 차등 발현(Differential Expression, DE)** 유전자를 탐지하기 위해 개발된 일반 모수적 통계 방법입니다. 이 기술의 핵심 목적은 공간적 위치, 병리적 특징과의 인접성, 또는 세포 간 상호작용과 같은 다양한 **조직 맥락(Tissue Context)**에 따라 세포의 유전자 발현이 어떻게 변화하는지 정확하게 추정하는 것입니다.

### 2. 주요 해결 과제: 세포 혼합(Cell Type Mixtures) 문제

기존의 공간 전사체 분석법들이 직면한 가장 큰 문제는 하나의 측정 단위(픽셀 또는 스팟) 내에 **여러 세포 유형이 섞여 있을 수 있다는 점**입니다.

-   **편향 발생:** 세포 유형의 비율 변화를 고려하지 않고 DE를 분석하면, 실제 발현 변화가 아닌 세포 구성의 차이로 인해 잘못된 결과(편향)가 도출될 수 있습니다.
-   **C-SIDE의 해결책:** C-SIDE는 각 픽셀의 세포 유형 비율을 먼저 추정한 후, 이를 모델에 반영하여 특정 세포 유형에서 발생하는 고유한 발현 변화만을 분리해 냅니다.

### 3. 분석 알고리즘 및 작동 원리

C-SIDE의 분석 과정은 크게 세 단계로 나뉩니다:

1.  **세포 유형 비율 추정:** 단일 세포 RNA-seq(scRNA-seq) 참조 데이터를 활용해 각 픽셀에 포함된 세포 유형별 비율을 계산합니다.
2.  **모수적 모델 피팅:** 정의된 공변량(Covariates, 예: 거리, 지역 등)을 사용하여 유전자 발현을 모델링합니다. 이때 로그 선형(Log-linear) 모델을 사용하여 세포 유형별 DE 추정치와 표준 오차를 구합니다.
3.  **통계적 추론:** 샘플 간의 기술적/생물학적 변이를 고려하여 여러 반복 실험(Replicates) 데이터를 통합하고, 최종적으로 통계적으로 유의미한 유전자를 식별합니다.

### 4. 다양한 활용 시나리오

C-SIDE는 매우 유연한 프레임워크를 가지고 있어 다음과 같은 다양한 연구에 적용될 수 있습니다:

-   **해부학적 지역 간 비교:** 조직 내 서로 다른 영역(예: 소뇌의 전엽 vs 결절) 사이의 발현 차이 분석.
-   **거리 기반 분석:** 특정 지표(예: 시상하부 중앙선)로부터의 거리에 따른 변화 분석.
-   **병리적 특징과의 관계:** 알츠하이머병의 **아밀로이드 베타 플라크(Aβ plaque)** 밀도에 따른 면역 세포의 반응 분석.
-   **세포 간 상호작용:** 특정 세포 유형이 인접해 있을 때 발생하는 발현 변화 분석.
-   **비모수적 분석:** 구체적인 가설 없이도 조직 전체의 일반적인 공간 발현 패턴을 탐색 가능.

### 5. 소프트웨어 구현

이 알고리즘은 R 패키지인 **`spacexr`**에 포함되어 배포되고 있으며, 공간 RNA 데이터 처리를 위한 `SpatialRNA` 객체와 참조 데이터를 위한 `Reference` 객체 등을 사용하여 분석을 수행합니다.

요약하자면, C-SIDE는 공간 전사체 데이터에서 **세포 혼합의 한계를 극복**하고, **다양한 생물학적 요인에 따른 세포 유형별 발현 변화를 정밀하게 분석**할 수 있게 해주는 혁신적인 도구입니다.

더 구체적으로 알고 싶은 부분(예: 수학적 모델링, 특정 실험 결과 등)이 있다면 말씀해 주세요!

-------
# 2 강의용 통계 모델 설명 슬라이드


대학원생 강의용으로 적합하도록 **C-SIDE의 통계적 모델링** 핵심 내용을 중심으로 구성한 발표 슬라이드 초안입니다. 각 슬라이드는 핵심 개념, 수학적 정의, 그리고 분석 절차를 포함하고 있습니다.

----------

### **[강의 제목: C-SIDE의 통계적 프레임워크와 공간 차등 발현 분석]**

#### **Slide 1: 공간 전사체 분석의 핵심 과제와 C-SIDE의 목적**

-   **핵심 문제 (Motivation):** 공간 전사체(ST) 데이터의 픽셀(Pixel)은 여러 세포 유형이 섞여 있는 **세포 혼합물(Cell type mixtures)**인 경우가 많습니다.
-   **문제점:** 세포 구성 비율의 변화를 고려하지 않으면, 실제 유전자 발현 변화가 아닌 세포 비율의 차이를 발현 변화로 오인하는 편향(Bias)이 발생합니다.
-   **C-SIDE의 해결책:** 각 픽셀의 세포 유형 비율을 모델에 통합하여, **특정 세포 유형 내에서의 차등 발현(Cell type-specific DE)**을 통계적으로 추정합니다.

#### **Slide 2: 계층적 확률 모델 (Hierarchical Probabilistic Model) - (1)**

-   **관측 데이터 모델링:** 픽셀 $i$, 유전자 $j$, 샘플 $g$에서의 관측된 RNA 카운트 $Y_{i,j,g}$는 포아송 분포를 따른다고 가정합니다.
    -   $Y_{i,j,g} | \lambda_{i,j,g} \sim \text{Poisson}(N_{i,g}\lambda_{i,j,g})$
    -   $N_{i,g}$: 해당 픽셀의 총 전사체 수 (Library size)
    -   $\lambda_{i,j,g}$: 기대되는 유전자 발현율
-   **가산적 혼합 모델 (Additive Mixture Model):** 기대 발현율은 각 세포 유형의 기여도를 합산한 형태로 정의됩니다.
    -   $\log(\lambda_{i,j,g}) = \log(\sum_{k=1}^K \beta_{i,k,g}\mu_{i,k,j,g}) + \gamma_{j,g} + \epsilon_{i,j,g}$
    -   $\beta_{i,k,g}$: 픽셀 $i$ 내 세포 유형 $k$의 비율 (RCTD를 통해 사전 추정)
    -   $\gamma_{j,g}$: 플랫폼 효과 (Platform random effect)
    -   $\epsilon_{i,j,g}$: 유전자별 과분산(Overdispersion)을 고려한 무작위 효과

#### **Slide 3: 차등 발현의 매개변수화 (Parametrization of DE)**

-   **로그 선형 모델 (Log-linear Model):** 특정 세포 유형 $k$의 기대 발현량 $\mu_{i,k,j,g}$를 공변량(Covariates) $x$의 선형 결합으로 모델링합니다.
    -   $\log(\mu_{i,k,j,g}) = \alpha_{0,k,j,g} + \sum_{l=1}^L x_{i,l,g}\alpha_{l,k,j,g}$
-   **주요 변수 정의:**
    -   $x_{i,l,g}$: 연구자가 정의한 공변량 (예: 특정 영역 여부, 특정 지점으로부터의 거리, 병리적 밀도 등)
    -   $\alpha_{l,k,j,g}$: **우리가 구하고자 하는 핵심 계수.** 공변량 $l$에 따른 세포 유형 $k$의 DE 효과 크기(Log fold-change)를 나타냅니다.
    -   $\alpha_{0,k,j,g}$: 절편(Intercept) 항.

#### **Slide 4: 최대 우도 추정 (Maximum Likelihood Estimation, MLE)**

-   **분석 단계:**
    1.  **세포 비율 추정:** RCTD 알고리즘을 사용하여 $\beta_{i,k,g}$와 플랫폼 효과 $\gamma_{j,g}$를 먼저 추정합니다.
    2.  **과분산 추정:** 유전자별 과분산 매개변수 $\sigma_{\epsilon,j,g}$를 추정합니다.
    3.  **DE 계수 추정:** 고정된 $\beta, \gamma, \sigma$ 값을 바탕으로 **2차 신뢰 영역 최적화(Second-order trust-region optimization)** 알고리즘을 사용하여 $\alpha$를 산출합니다.
-   **최적화 특징:** 각 유전자별 우도(Likelihood)가 독립적이므로 대규모 데이터에 대해 병렬 처리가 가능합니다.

#### **Slide 5: 통계적 추론 및 가설 검정 (Statistical Inference)**

-   **점근적 정규성 (Asymptotic Normality):** 픽셀 수 $n$이 충분히 클 때, 추정된 $\hat{\alpha}$는 정규 분포를 따릅니다.
    -   $\sqrt{n}(\hat{\alpha} - \alpha) \approx N(0, I^{-1}_{\alpha})$ ($I_{\alpha}$는 피셔 정보 행렬)
-   **가설 검정:**
    -   표준 오차(Standard Error) $s$를 계산하고, **Z-test**를 통해 유의 확률($P$-value)을 도출합니다.
    -   Benjamini-Hochberg 절차를 사용하여 다중 가설 검정(FDR)을 수행합니다.
    -   기본적으로 FDR 0.01 및 Fold-change 1.5배 이상을 유의미한 유전자로 판단합니다.

#### **Slide 6: 다중 반복 실험 및 샘플 통합 (Inference across Replicates)**

-   **계층적 분산 모델:** 개별 샘플의 추정치 $\hat{\alpha}_g$가 인구 집단 평균 $A$를 중심으로 분포한다고 가정합니다.
    -   $\alpha_g \sim N(A, \tau^2)$ ($\tau$: 샘플 간 생물학적/기술적 변이)
-   **통합 알고리즘:**
    -   **Method of Moments**를 통해 샘플 간 분산 $\tau^2$을 추정합니다.
    -   **가중 평균(Weighted average):** 각 샘플의 표준 오차와 $\tau^2$을 고려하여 인구 집단 수준의 DE($A$)를 산출합니다 (Meta-analysis 방식과 유사).
-   **장점:** 단일 샘플에서 발생할 수 있는 거짓 양성(False discovery)을 줄이고 보다 강건한(Robust) 결과를 도출합니다.

----------

**참고 사항:** 이 슬라이드 구성은 C-SIDE의 수학적 정밀함과 생물학적 해석 가능성을 동시에 강조하고 있습니다. 강의 시에는 **Slide 2의 가산적 모델**이 어떻게 혼합된 시그널을 분리해내는지 시각적으로 설명하면 대학원생들이 더 쉽게 이해할 수 있을 것입니다.

추가로 특정 알고리즘의 R 구현(예: `run.CSIDE.single` 함수 등)에 대한 실습 자료가 필요하시면 말씀해 주세요.

-------------
# 3 통계 모델 설명


네, 가능합니다. 요청하신 대로 **C-SIDE(Cell type-Specific Inference of Differential Expression)** 논문의 핵심 내용과 수식을 포함한 통계 모델의 상세 설명을 Markdown 형식으로 작성해 드립니다.

----------

## C-SIDE: 공간 전사체 데이터를 위한 세포 유형별 차등 발현 분석 모델

## 1. 개요 및 목적

C-SIDE는 공간 전사체(Spatial Transcriptomics) 데이터에서 하나의 픽셀(Pixel) 내에 여러 세포 유형이 섞여 있는 **세포 혼합(Cell type mixtures)** 문제를 해결하기 위해 설계되었습니다. 이 모델의 목적은 특정 세포 유형 내에서 공간적 위치나 병리적 특성(예: 아밀로이드 플라크 밀도)과 같은 **공변량(Covariates)**에 따른 유전자 발현 변화를 통계적으로 정확하게 추정하는 것입니다.

## 2. 계층적 확률 모델 (Hierarchical Probabilistic Model)

C-SIDE는 관측된 RNA 카운트 데이터를 모델링하기 위해 포아송 분포와 로그 선형 모델을 결합한 계층적 구조를 사용합니다.

### 2.1 관측 모델 (Likelihood)

샘플 $g$의 픽셀 $i$에서 유전자 $j$의 관측된 카운트 $Y_{i,j,g}$는 다음과 같이 포아송 분포를 따른다고 가정합니다:

$$Y_{i,j,g} | \lambda_{i,j,g} \sim \text{Poisson}(N_{i,g}\lambda_{i,j,g})$$

-   $N_{i,g}$: 해당 픽셀의 총 전사체 수 (Library size).
-   $\lambda_{i,j,g}$: 기대되는 유전자 발현율.

### 2.2 가산적 혼합 모델 (Additive Mixture Model)

기대 발현율 $\lambda$는 각 세포 유형의 기여도를 합산하여 모델링하며, 로그 공간에서 다음과 같이 정의됩니다:

$$\log(\lambda_{i,j,g}) = \log\left(\sum_{k=1}^K \beta_{i,k,g}\mu_{i,k,j,g}\right) + \gamma_{j,g} + \epsilon_{i,j,g}$$

-   $\beta_{i,k,g}$: 픽셀 $i$ 내 세포 유형 $k$의 비율 (RCTD를 통해 사전 추정됨).
-   $\mu_{i,k,j,g}$: 픽셀 $i$의 세포 유형 $k$에서 유전자 $j$의 기대 발현량.
-   $\gamma_{j,g}$: 유전자별 플랫폼 효과 (Platform random effect).
-   $\epsilon_{i,j,g}$: 기술적/생물학적 변이를 반영하는 유전자별 과분산(Overdispersion) 항.

## 3. 차등 발현의 매개변수화 (Parametrization of DE)

C-SIDE의 핵심은 세포 유형별 발현량 $\mu_{i,k,j,g}$를 공변량 $x$의 함수로 표현하는 것입니다:

$$\log(\mu_{i,k,j,g}) = \alpha_{0,k,j,g} + \sum_{l=1}^L x_{i,l,g}\alpha_{l,k,j,g}$$

-   $x_{i,l,g}$: $l$번째 공변량 (예: 좌표, 특정 영역과의 거리 등).
-   $\alpha_{l,k,j,g}$: **차등 발현 계수.** 공변량 $l$에 따른 세포 유형 $k$의 발현 변화량(Log fold-change)을 나타냅니다.
-   $\alpha_{0,k,j,g}$: 절편(Intercept) 항.

## 4. 모델 추정 및 최적화

### 4.1 단계별 추정 과정

1.  **사전 단계:** RCTD 알고리즘을 사용하여 세포 비율($\beta$)과 플랫폼 효과($\gamma$)를 추정합니다.
2.  **과분산 추정:** 유전자별 과분산 매개변수 $\sigma_{\epsilon,j,g}$를 최대 우도 추정법(MLE)으로 계산합니다.
3.  **핵심 단계:** 고정된 $\beta, \gamma, \sigma$ 값을 바탕으로 DE 계수 $\alpha$를 추정합니다.

### 4.2 신뢰 영역 최적화 (Trust-region Optimization)

우도 함수(Likelihood function)를 최적화하기 위해 C-SIDE는 **2차 신뢰 영역 기반 최적화** 알고리즘을 사용합니다. 이 방식은 로그 우도의 2차 테일러 근사(Quadratic Taylor approximation)를 반복적으로 풀면서 매 단계마다 매개변수의 변화량을 적응적으로 제한하여 안정적인 수렴을 보장합니다.

## 5. 통계적 추론 (Statistical Inference)

### 5.1 점근적 정규성 및 가설 검정

픽셀 수 $n$이 클 때, MLE 추정치 $\hat{\alpha}$는 점근적으로 정규 분포를 따릅니다: $$\sqrt{n}(\hat{\alpha} - \alpha) \sim N(0, I_{\alpha}^{-1})$$ 여기서 $I_{\alpha}$는 피셔 정보 행렬(Fisher Information Matrix)입니다. 이를 통해 표준 오차를 계산하고, **Z-test**를 수행하여 각 유전자의 유의 확률($P$-value)을 도출합니다.

### 5.2 반복 실험 통합 (Inference across Replicates)

여러 샘플 또는 반복 실험 데이터가 있는 경우, 개별 샘플의 추정치 $\hat{\alpha}_g$를 통합하여 인구 집단 수준의 효과 $A$를 산출합니다.

-   **모델:** $\alpha_g \sim N(A, \tau^2)$ ($\tau$: 샘플 간 변이).
-   **통합 방식:** 가중 평균을 사용하는 **가중 최소 제곱법(Weighted Least Squares)** 방식을 적용하여 보다 강건한 통계적 결론을 내립니다.

----------

이 내용은 소스 자료의 수식과 설명을 직접 인용하여 작성되었습니다. 추가적인 세부 수식이나 특정 데이터셋 적용 사례가 필요하시면 알려주세요.

-------------------------------
# 4 계층적 확률 모델

레퍼런스 18(RCTD 논문)과 62(DerSimonian-Laird 메타 분석)가 추가됨에 따라, C-SIDE의 **계층적 확률 모델(Hierarchical Probabilistic Model)**을 가장 낮은 데이터 관측 단계부터 가장 높은 인구 집단 추론 단계까지 통합적으로 설명할 수 있게 되었습니다.

C-SIDE의 모델은 크게 4가지 계층으로 구성됩니다.

### 1단계: 픽셀 단위 관측 모델 (Poisson Sampling)

가장 기본 계층은 공간 전사체 데이터의 특성인 '카운트(Count)' 데이터를 모델링하는 것입니다. 픽셀 $i$, 유전자 $j$, 샘플 $g$에서의 관측된 RNA 카운트 $Y$는 **포아송 분포**를 따른다고 가정합니다.

-   **수식:** $Y_{i,j,g} | \lambda_{i,j,g} \sim \text{Poisson}(N_{i,g}\lambda_{i,j,g})$
-   **의미:** $N$은 픽셀의 총 전사체 수(Library size)이며, $\lambda$는 해당 유전자가 발현될 기대 확률입니다. 이 포아송 모델링을 통해 전사체 수가 적은 데이터(예: Slide-seq)에서 발생하는 샘플링 노이즈를 효과적으로 처리합니다.

### 2단계: 가산적 혼합 모델 (Additive Mixture Model)

두 번째 계층은 픽셀 내의 **세포 혼합물**을 분해하는 단계로, **레퍼런스 18(RCTD)**의 핵심 알고리즘이 적용됩니다.

-   **수식:** $\log(\lambda_{i,j,g}) = \log(\sum_{k=1}^K \beta_{i,k,g}\mu_{i,k,j,g}) + \gamma_{j,g} + \epsilon_{i,j,g}$
-   **주요 변수 (Ref 18 기반):**
    -   $\beta_{i,k,g}$: 픽셀 $i$ 내 세포 유형 $k$의 비율입니다. RCTD는 scRNA-seq 참조 데이터를 사용해 이 비율을 미리 추정합니다.
    -   $\gamma_{j,g}$: **플랫폼 효과(Platform Effect)**입니다. 시퀀싱 기술 간의 체계적 차이를 보정하는 무작위 효과입니다.
    -   $\epsilon_{i,j,g}$: 유전자별 **과분산(Overdispersion)**을 반영하는 항으로, 공간적 변이나 생물학적 변이를 흡수합니다.

### 3단계: 세포 유형별 DE 매개변수화 (Log-linear Model)

세 번째 계층은 C-SIDE만의 핵심 기여분으로, 각 세포 유형의 고유 발현량 $\mu$를 **공변량(Covariates)**의 함수로 정의하는 단계입니다.

-   **수식:** $\log(\mu_{i,k,j,g}) = \alpha_{0,k,j,g} + \sum_{l=1}^L x_{i,l,g}\alpha_{l,k,j,g}$
-   **의미:**
    -   $x$: 연구자가 정의한 공간적 위치, 거리, 병리적 밀도 등의 공변량입니다.
    -   $\alpha$: 우리가 찾고자 하는 **차등 발현 계수**입니다. 특정 세포 유형 $k$ 내에서 공변량 $x$에 의해 유전자 발현이 얼마나 변하는지(Log fold-change)를 나타냅니다.

### 4단계: 다중 반복 실험 통합 (Meta-Analysis Approach)

마지막 계층은 여러 샘플($g$)에서 얻은 결과를 통합하여 인구 집단 수준의 결론을 도출하는 단계입니다. 이 과정에서 **레퍼런스 62(DerSimonian-Laird)**의 메타 분석 기법이 사용됩니다.

-   **계층적 분산 모델:** 각 샘플의 DE 추정치 $\hat{\alpha}_g$가 인구 집단 평균 $A$를 중심으로 정규 분포한다고 가정합니다.
    -   $\alpha_g \sim N(A, \tau^2)$ ($\tau$: 샘플 간 생물학적/기술적 변이)
-   **추론 방식 (Ref 62 기반):** **Method of Moments**를 사용하여 샘플 간 분산 $\tau^2$을 추정하고, 이를 바탕으로 각 샘플 결과에 가중치를 부여해 통합 평균 $A$를 계산합니다. 이 방식은 단일 샘플에서 발생할 수 있는 가짜 양성(False discovery)을 줄이고 결과의 강건성을 보장합니다.

### 요약

이 계층적 모델을 통해 C-SIDE는 **"포아송 샘플링(1단계) → 세포 혼합 해소(2단계, Ref 18) → 세포별 발현 변화 추정(3단계) → 샘플 간 통합(4단계, Ref 62)"**의 과정을 거쳐, 복잡한 조직 환경 속에서 특정 세포 유형이 어떻게 반응하는지를 정밀하게 계산해 낼 수 있습니다.

-------
# 5 Results : C-SIDE Model

C-SIDE 논문의 **Results** 섹션에서 해당 문단은 모델의 전체적인 구조와 수학적 설계를 요약하고 있습니다. 요청하신 문구('Here, we develop C-SIDE...'부터 '... for gene j in cell type k for sample g'까지)를 문장 단위로 상세히 설명해 드립니다.

### **1. C-SIDE의 개발 목적**

-   **원문:** "Here, we develop C-SIDE, a statistical method for determining DE in spatial transcriptomics datasets (Fig. 1a)."
-   **설명:** 이 문장은 C-SIDE가 공간 전사체 데이터셋에서 **차등 발현(Differential Expression, DE)**을 결정하기 위해 개발된 **통계적 방법**임을 정의하며 시작합니다.

### **2. 입력 데이터 정의**

-   **원문:** "C-SIDE inputs one or more experimental samples of spatial transcriptomics data, consisting of $Y_{i,j,g}$ as the observed RNA counts for pixel $i$, gene $j$ and experimental sample $g$."
-   **설명:** C-SIDE는 하나 이상의 실험 샘플을 입력으로 받습니다. 여기서 $Y_{i,j,g}$는 실험 샘플 $g$의 특정 픽셀 $i$에서 관측된 특정 유전자 $j$의 **RNA 카운트(발현량) 데이터**를 의미합니다.

### **3. 관측 모델 (Poisson Sampling)**

-   **원문:** "We then assume Poisson sampling so that $Y_{i,j,g} | \lambda_{i,j,g} \approx \text{Poisson}(N_{i,g}\lambda_{i,j,g})$, with $\lambda_{i,j,g}$ the expected count and $N_{i,g}$ the total transcript count (for example, total unique molecular identifiers, UMIs) for pixel $i$ on experimental sample $g$."
-   **설명:** 관측된 데이터 $Y$가 **포아송 분포**를 따른다고 가정하는 단계입니다. $N$은 해당 픽셀의 **총 전사체 수(Library size)**를 의미하며, $\lambda$는 해당 유전자의 **기대 발현율**을 나타냅니다. 이 모델링은 Slide-seq과 같이 전사체 수가 적은 데이터에서 발생하는 샘플링 노이즈를 처리하는 데 유용합니다.

### **4. 가산적 혼합 모델 (Additive Mixture Model)**

-   **원문:** "Accounting for platform effects and other sources of technical and natural variability, we assume $\lambda_{i,j,g}$ is a mixture of $K$ cell type expression profiles, defined by, $\log(\lambda_{i,j,g}) = \log(\sum_{k=1}^K \beta_{i,k,g}\mu_{i,k,j,g}) + \gamma_{j,g} + \epsilon_{i,j,g}$..."
-   **설명:** 기대 발현율 $\lambda$를 **세포 유형들의 혼합물**로 모델링합니다. 로그 공간에서 정의된 이 수식은 픽셀 내에 섞여 있는 여러 세포 유형($K$)의 기여도를 합산한 형태입니다. 여기에는 기술적 차이를 보정하는 **플랫폼 효과($\gamma$)**와 생물학적/공간적 변이를 반영하는 **과분산 항($\epsilon$)**이 포함됩니다.

### **5. 주요 매개변수 정의**

-   **원문:** "...with $\mu_{i,k,j,g}$ the cell type-specific expected gene expression rate for pixel $i$, gene $j$, experimental sample $g$ and cell type $k$; $\beta_{i,k,g}$ the proportion of cell type $k$ contained in pixel $i$ for experimental sample $g$; $\gamma_{j,g}$ a gene-specific random effect that accounts for platform variability and $\epsilon_{i,j,g}$ a random effect to account for gene-specific overdispersion."
-   **설명:** 위 수식에 사용된 변수들을 정의합니다. **$\mu$**는 특정 세포 유형 내에서의 유전자 발현율, **$\beta$**는 해당 픽셀에서 그 세포 유형이 차지하는 비율, **$\gamma$**는 플랫폼 간 차이를 보정하는 무작위 효과, **$\epsilon$**은 유전자별 과분산을 의미합니다.

### **6. 세포 유형별 차등 발현(DE)의 모델링**

-   **원문:** "To account for cell type-specific DE, we model across pixel locations the log of the cell type-specific profiles $\mu_{i,k,j,g}$ as a linear combination of $L$ covariates used to explain DE."
-   **설명:** C-SIDE의 핵심 기여 부분으로, 세포 유형별 고유 발현량 $\mu$가 픽셀 위치에 따라 어떻게 변하는지 모델링합니다. 이를 위해 유전자 발현 변화를 설명할 수 있는 **$L$개의 공변량(Covariates)**의 **선형 결합**으로 표현합니다.

### **7. 로그 선형 매개변수화 (Log-linear Model)**

-   **원문:** "Specifically, we assume that, $\log(\mu_{i,k,j,g}) = \alpha_{0,k,j,g} + \sum_{\ell=1}^L x_{i,\ell,g}\alpha_{\ell,k,j,g}$."
-   **설명:** 실제 계산에 사용되는 수식(식 3)입니다. 유전자 발현량($\mu$)의 로그값이 절편($\alpha_0$)과 공변량($x$) 및 그 계수($\alpha$)들의 합으로 결정된다는 **로그 선형 모델**을 정의합니다.

### **8. 공변량과 차등 발현 계수의 정의**

-   **원문:** "Here, $\alpha_{0,k,j,g}$ represents the intercept term for gene $j$ and cell type $k$ in sample $g$, and $x_{i,\ell,g}$ represents the $\ell$th covariate, evaluated at pixel $i$ in sample $g$."
-   **설명:** 수식 내의 변수들을 다시 설명합니다. $\alpha_0$는 발현의 기본값을 결정하는 **절편**이며, $x_{i,\ell,g}$는 연구자가 미리 정의한 **공변량**(예: 특정 위치에서의 거리나 영역 여부 등)의 구체적인 값입니다.

### **9. 계수 $\alpha$의 통계적 의미**

-   **원문:** "Similar to linear and generalized linear models, $x$, also called the design matrix, represents predefined covariate(s) that explain DE and the corresponding coefficient(s) $\alpha_{\ell,k,j,g}$ each represent the DE effect size of covariate $\ell$ for gene $j$ in cell type $k$ for sample $g$."
-   **설명:** 일반적인 **선형 회귀 모델**과 유사하게, 설계 행렬($x$)은 차등 발현을 설명하는 요인들을 담고 있습니다. 우리가 최종적으로 구하고자 하는 **계수 $\alpha$**는 특정 공변량($\ell$)이 특정 유전자($j$)와 세포 유형($k$)에 미치는 **효과 크기(Effect size, 즉 발현 변화 정도)**를 나타냅니다.

### **요약**

이 문단은 C-SIDE가 **(1) 포아송 샘플링**을 통해 기초 데이터를 읽고, **(2) 세포 혼합 모델**을 통해 픽셀 내 시그널을 분해하며, **(3) 로그 선형 모델**을 통해 특정 세포 유형이 주변 환경(공변량)에 따라 유전자 발현을 얼마나 바꾸는지($\alpha$)를 통계적으로 계산해 내는 전체적인 논리를 설명하고 있습니다.

-------

# 6 Methods : C-SIDE Model


C-SIDE 논문의 **Methods** 섹션에서 요청하신 두 문단(본문,번 구절)은 모델의 수학적 정의와 설계 철학을 담고 있는 핵심적인 부분입니다. 이를 문장별로 상세히 설명해 드립니다.

### **첫 번째 문단: C-SIDE 모델의 구성 요소 및 변수 정의**

1.  **"Here, we describe C-SIDE, a statistical method for identifying DE in spatial transcriptomics data."**
    
    -   **설명:** C-SIDE가 공간 전사체 데이터에서 **차등 발현(Differential Expression, DE)** 유전자를 식별하기 위한 통계적 방법임을 선언하며 시작합니다.,
2.  **"We define the C-SIDE model in equations (1)–(3)."**
    
    -   **설명:** 모델의 기초가 되는 세 가지 수식을 참조합니다. 이는 **포아송 샘플링(식 1)**, **가산적 혼합 모델(식 2)**, 그리고 **로그 선형 차등 발현 매개변수화(식 3)**를 의미합니다.,,
3.  **"Before fitting C-SIDE, the design matrix is predefined to contain covariates, variables on which gene expression is hypothesized to depend such as spatial position or cellular microenvironment."**
    
    -   **설명:** 분석 전, 연구자는 유전자 발현에 영향을 줄 것으로 예상되는 **공변량(Covariates)**(예: 좌표, 특정 세포와의 거리, 병리적 환경 등)을 포함하는 **설계 행렬(Design Matrix, $x$)**을 미리 정의해야 합니다.,
4.  **"Recall that $x_{i,l,g}$ represents the $l$th covariate, evaluated at pixel $i$ in experimental sample $g$."**
    
    -   **설명:** 변수 $x_{i,l,g}$는 실험 샘플 $g$의 특정 픽셀 $i$에서 측정된 $l$번째 공변량의 구체적인 값(예: 해당 위치의 아밀로이드 플라크 밀도)을 나타냅니다.,
5.  **"For each covariate $x_{\cdot,l,g}$, there is a corresponding coefficient $\alpha_{l,k,j,g}$, representing a gene expression change across pixels per unit change of $x_{\cdot,l,g}$ within cell type $k$ of experimental sample $g$."**
    
    -   **설명:** 각 공변량에는 그에 대응하는 **계수 $\alpha$**가 존재합니다. 이 $\alpha$는 특정 세포 유형 $k$ 내에서 공변량 $x$가 한 단위 변할 때 유전자 $j$의 발현이 얼마나 변하는지(Log fold-change)를 나타내는 핵심 추정치입니다.,
6.  **"Next, recall from equation (2) random effects $\gamma_{j,g}$ and $\epsilon_{i,j,g}$, which we assume both follow normal distributions with mean 0 and standard deviations $\sigma_{\gamma,g}$ and $\sigma_{\epsilon,j,g}$, respectively."**
    
    -   **설명:** 모델에는 두 가지 무작위 효과가 포함됩니다. $\gamma$는 플랫폼 효과(참조 데이터와 공간 데이터 간의 기술적 차이)를, $\epsilon$은 과분산(자연적인 변이)을 반영하며, 둘 다 평균이 0인 정규 분포를 따른다고 가정합니다.,,
7.  **"The overdispersion magnitude, $\sigma_{\epsilon,j,g}$ varies across gene $j$ (Supplementary Fig. 12) and modeling gene-specific overdispersion is necessary for controlling the false-positive rate of C-SIDE."**
    
    -   **설명:** **과분산의 크기($\sigma_{\epsilon}$)**는 유전자마다 다릅니다. 따라서 각 유전자에 최적화된 과분산 값을 모델링하는 것이 가짜 양성(False-positive) 결과를 억제하고 통계적 정확도를 높이는 데 필수적입니다.,

----------

### **두 번째 문단: 세포 유형별 특이성 및 GLM과의 관계**

1.  **"Due to our finding that genes can exhibit DE in some but not all cell types (Fig. 3c), C-SIDE generally does not assume that genes share DE patterns across cell types, allowing for the discovery of cell type-specific DE."**
    
    -   **설명:** 연구 결과, 특정 유전자가 일부 세포 유형에서만 차등 발현될 수 있음이 확인되었습니다(예: _Aldoc_ 유전자는 푸르키니예 세포에서만 변함). 따라서 C-SIDE는 모든 세포가 동일하게 반응한다고 가정하지 않고, **각 세포 유형별로 고유한 발현 변화**를 찾아낼 수 있게 설계되었습니다.,
2.  **"We also developed an option where DE can be assumed to be shared across cell types (Supplementary Notes)."**
    
    -   **설명:** 하지만 모든 세포 유형이 특정 환경에 공통적으로 반응할 것으로 예상되는 경우를 위해, 차등 발현 패턴을 세포 유형 간에 공유하도록 설정하는 옵션도 제공합니다.,
3.  **"C-SIDE can be thought of as a modification of the generalized linear model (GLM) in which each cell type follows a cell type-specific log-linear model before an additive mixture of all cell types is observed."**
    
    -   **설명:** C-SIDE는 통계학의 **일반화 선형 모델(GLM)**을 변형한 형태로 이해할 수 있습니다. 핵심은 각 세포 유형이 각자의 로그 선형 모델을 따르고, 우리가 실제로 관측하는 픽셀 데이터는 이들이 **가산적으로 혼합(Additive mixture)**된 결과물이라는 점입니다.,
4.  **"See Fitting the C-SIDE model and Hypothesis testing for C-SIDE model fitting and hypothesis testing, respectively."**
    
    -   **설명:** 모델을 실제로 데이터에 적합시키는 과정(최대 우도 추정 등)과 통계적 유의성을 검정하는 방법(Z-test 등)에 대한 상세 내용은 뒤이어 나오는 개별 섹션을 참조하라는 안내입니다.,

### **요약**

이 두 문단은 C-SIDE가 **(1) 공변량을 통해 세포의 환경을 정의하고, (2) 각 세포 유형의 고유한 반응($\alpha$)을 개별적으로 추정하며, (3) 유전자별 노이즈를 정밀하게 제어**하여 기존 GLM 방식보다 훨씬 정교하게 공간적 발현 변화를 분석할 수 있음을 이론적으로 설명하고 있습니다.

-----------------------------------------------------
# 7 파라미터 계산 순서

C-SIDE의 실제 분석 과정은 여러 단계의 계층적 추정을 거치며, 각 단계에서 계산되는 주요 파라미터(그릭 문자 중심)와 순서는 다음과 같습니다.

### **1단계: 세포 유형 비율($\beta$) 및 플랫폼 효과($\gamma$) 추정**

C-SIDE 분석의 가장 첫 번째 단계는 **RCTD 알고리즘**을 사용하여 각 픽셀의 구성을 파악하는 것입니다.

-   **$\beta_{i,k,g}$ (Cell type proportions):** 각 픽셀($i$) 내에 존재하는 각 세포 유형($k$)의 비율을 먼저 추정합니다.
-   **$\gamma_{j,g}$ (Platform random effect):** 참조 데이터(scRNA-seq)와 공간 데이터 간의 기술적 차이(플랫폼 효과)를 유전자($j$)별 무작위 효과로 계산하여 보정합니다.
-   이 두 값은 이후 단계에서 **고정된 입력값(Plug-in estimates)**으로 사용됩니다.

### **2단계: 유전자별 과분산($\sigma_\epsilon$) 추정**

세포 비율이 결정된 후, C-SIDE는 유전자별 노이즈 수준을 측정합니다.

-   **$\sigma_{\epsilon,j,g}$ (Overdispersion magnitude):** 포아송 모델만으로 설명되지 않는 생물학적/기술적 변이를 반영하는 과분산 파라미터입니다.
-   가짜 양성(False-positive) 결과를 방지하기 위해 유전자($j$)마다 개별적으로 최대 우도 추정법(MLE)을 통해 계산됩니다.

### **3단계: 차등 발현 계수($\alpha$) 추정**

앞서 계산된 $\beta, \gamma, \sigma_\epsilon$ 값들을 바탕으로, 연구자가 궁금해하는 핵심 계수를 산출합니다.

-   **$\alpha_{0,k,j,g}$ (Intercept):** 특정 세포 유형에서 유전자 발현의 기본값(절편)입니다.
-   **$\alpha_{\ell,k,j,g}$ (DE effect size):** 연구자가 설정한 공변량($\ell$)(예: 특정 영역, 거리, 상호작용 등)에 따른 유전자 발현의 변화량(Log fold-change)입니다.
-   이 값은 **2차 신뢰 영역 최적화(Trust-region optimization)** 알고리즘을 통해 수렴될 때까지 반복 계산됩니다.

### **4단계: 표준 오차($s$) 계산 및 유의성 검정**

추정된 계수의 신뢰도를 평가하는 단계입니다.

-   **$s_{\ell,k,j,g}$ (Standard error):** 피셔 정보 행렬(Fisher Information Matrix)을 통해 계수 $\alpha$의 표준 오차를 계산합니다.
-   이후 $Z$-test를 통해 유의 확률($P$-value)을 도출하고 최종적으로 차등 발현 유전자(DE gene)를 식별합니다.

### **5단계: 다중 반복 실험 통합 ($A$ 및 $\tau$)**

샘플이 여러 개($g > 1$)인 경우, 이를 통합하여 인구 집단 수준의 결론을 내립니다.

-   **$\tau^2_{\ell,k,j}$ (Inter-replicate variance):** 샘플(반복 실험) 간에 발생하는 생물학적/기술적 변이의 크기를 **Method of Moments**로 추정합니다.
-   **$A_{\ell,k,j}$ (Population-level DE):** 각 샘플의 추정치($\hat{\alpha}$)와 변이($\tau^2, s^2$)를 고려한 가중 평균을 통해, 전체 집단을 대표하는 최종 차등 발현 값을 얻게 됩니다.

**요약하자면:** **$\beta, \gamma$ (구성 파악) $\rightarrow$ $\sigma_\epsilon$ (노이즈 측정) $\rightarrow$ $\alpha$ (효과 추정) $\rightarrow$ $s$ (신뢰도 검증) $\rightarrow \tau, A$ (샘플 통합)** 순으로 계산이 진행됩니다.

---------------

C-SIDE의 계층적 모델에서 **뮤($\mu$)**와 **람다($\lambda$)**는 분석의 핵심적인 중간 계산값과 최종 기대값에 해당하며, 각각 다음과 같은 단계에서 정의되고 얻어집니다.

### 1. 뮤($\mu_{i,k,j,g}$): 세포 유형별 기대 발현량

**뮤($\mu$)**는 특정 세포 유형($k$) 내에서 유전자($j$)가 얼마나 발현될지를 나타내는 값으로, **3단계인 차등 발현 계수($\alpha$) 추정 단계**에서 그 구체적인 값이 결정됩니다.

-   **정의 방식:** C-SIDE 모델에서 $\mu$는 우리가 구하고자 하는 계수 $\alpha$와 연구자가 정의한 공변량 $x$의 결합으로 정의됩니다 ($\log(\mu_{i,k,j,g}) = \alpha_{0,k,j,g} + \sum x_{i,\ell,g}\alpha_{\ell,k,j,g}$).
-   **얻는 시점:** 최적화 알고리즘(Trust-region optimization)을 통해 **핵심 파라미터인 $\alpha$ 값이 수렴하여 결정되는 순간**, 각 픽셀 위치($i$)에서의 세포별 기대 발현량인 $\mu$가 산출됩니다.
-   **참고:** 분석 초기 단계에서 RCTD가 참조 데이터(scRNA-seq)로부터 얻는 값은 공간적 변화가 고려되지 않은 '평균 프로필'인 $\hat{\mu}_{k,j}$이며, C-SIDE가 최종적으로 계산해내는 것은 **공간적 맥락이 반영된 $\mu_{i,k,j,g}$**입니다.

### 2. 람다($\lambda_{i,j,g}$): 픽셀 단위 전체 기대 발현율

**람다($\lambda$)**는 여러 세포 유형이 섞여 있는 특정 픽셀($i$) 전체에서 해당 유전자가 관측될 최종적인 기대 확률을 의미합니다.

-   **얻는 단계:** 앞서 계산된 **세포별 발현량($\mu$)**과 **세포 비율($\beta$)**이 모두 구해진 후, 이들을 합산(Additive mixture)하는 과정에서 얻게 됩니다.
-   **수식적 위치:** $\lambda_{i,j,g} = \sum_{k=1}^K \beta_{i,k,g}\mu_{i,k,j,g}$ (로그 공간에서 플랫폼 효과 $\gamma$와 과분산 $\epsilon$이 더해지기 전의 핵심 혼합값).
-   **의미:** 이렇게 얻어진 $\lambda$는 실제 관측 데이터($Y$)가 포아송 분포를 통해 생성되는 기초가 됩니다 ($Y \sim \text{Poisson}(N\lambda)$). 즉, 모델이 예측하는 **'이 픽셀의 총 발현 기대치'**라고 볼 수 있습니다.

### 요약하자면

-   **$\mu$(뮤):** **계수 $\alpha$를 추정하는 단계**에서 공변량과 결합하여 세포 유형별로 먼저 계산됩니다.
-   **$\lambda$(람다):** 계산된 **$\mu$값들을 각 세포의 비율($\beta$)대로 섞어서** 픽셀 단위의 최종 기대치를 구할 때 산출됩니다.

이 두 값은 모델이 실제 데이터($Y$)를 얼마나 잘 설명하는지 확인하기 위한 **예측값(Predicted counts)**을 생성할 때 중요하게 사용됩니다.

-----------------------------------

수식 초반에 등장하는 **뮤($\mu$)**와 **람다($\lambda$)**는 모델의 계층적 구조를 정의하는 **기대값(Expected values)이자 중간 변수**들입니다.

이 파라미터들이 실제 계산 과정에서 어떻게 다뤄지는지 소스 자료를 바탕으로 더 구체적으로 설명해 드리겠습니다.

### 1. 모델의 변수로서의 역할 (구조적 정의)

수식 (1)~(3)에서 이들은 관측된 데이터($Y$)와 우리가 찾고자 하는 핵심 계수($\alpha$)를 연결하는 가교 역할을 합니다.

-   **뮤($\mu_{i,k,j,g}$):** 특정 세포 유형($k$)의 **"이론적인" 기대 발현량**을 정의합니다. 이는 공간적 공변량($x$)과 우리가 추정해야 할 계수($\alpha$)의 함수로 설정되어 있습니다.
-   **람다($\lambda_{i,j,g}$):** 여러 세포가 섞인 **픽셀 전체의 "이론적인" 기대 발현율**입니다. 이는 각 세포의 비율($\beta$)과 위에서 정의한 세포별 발현량($\mu$)을 합친 값으로 정의됩니다.

### 2. 실제 계산에서의 처리 (Optimization 과정)

C-SIDE는 **최대 우도 추정법(MLE)**을 사용하여 실제 계산을 수행하는데, 이때의 논리는 다음과 같습니다.

-   **$\alpha$를 직접 추정:** 알고리즘은 $\mu$나 $\lambda$ 값을 개별적으로 먼저 찾는 것이 아니라, **최종 계수인 $\alpha$를 직접 최적화**합니다.
-   **대입을 통한 단순화:** 수식 (11) 또는 (4)를 보면, $\mu$를 정의하는 로그 선형 수식을 $\lambda$ 수식에 대입하여, 전체 식을 **관측 데이터 $Y$와 미지수 $\alpha$만의 관계로 변환**합니다.
-   **반복 계산:** 신뢰 영역 최적화(Trust-region optimization) 알고리즘이 돌아가는 동안, $\alpha$ 값이 매 단계 업데이트됩니다. 이때 **계산 내부적으로는 업데이트된 $\alpha$에 대응하는 $\mu$와 $\lambda$ 값이 계속 변하게 됩니다**.

### 3. 최종 결과로서의 확정

분석이 끝나고 나면 다음과 같은 방식으로 값이 정해집니다.

-   **결과 도출:** 최적의 $\hat{\alpha}$ 값이 결정되면, 이 값을 수식 (3)에 넣어서 각 위치와 세포별 **최종 뮤($\mu$)**를 얻고, 이를 다시 수식 (2)에 넣어 픽셀별 **최종 람다($\lambda$)**를 얻게 됩니다.
-   **검증에 활용:** 이렇게 얻어진 최종 $\mu$와 $\lambda$는 모델이 실제 데이터($Y$)를 얼마나 잘 설명하는지 확인하기 위한 **예측값(Predicted counts)**으로 사용되어 시각화 등에 활용됩니다.

**요약하자면**, 질문하신 대로 뮤와 람다는 모델의 논리적 구조를 만드는 **변수(함수)**로 기능하며, 실제 수치적 계산은 이들을 포함한 전체 시스템을 가장 잘 설명하는 **최종 계수 $\alpha$를 찾음으로써 한꺼번에 결정**되는 것이라고 이해하시면 완벽합니다.

----------------------------------------

C-SIDE의 실제 계산은 낮은 계층의 데이터 파라미터부터 높은 계층의 통계적 추론 파라미터 순으로 진행됩니다. 각 단계에서 계산되는 주요 파라미터(그릭 문자)와 변수, 그리고 이를 구하는 핵심 수식을 순차적으로 설명해 드립니다.

### 0. 모델의 기본 구조 (기초 수식)

모든 계산은 다음 세 가지 계층적 수식을 기반으로 합니다.

-   **식 (1):** $Y_{i,j,g} | \lambda_{i,j,g} \sim \text{Poisson}(N_{i,g}\lambda_{i,j,g})$ (관측 모델)
-   **식 (2):** $\log(\lambda_{i,j,g}) = \log(\sum_{k=1}^K \beta_{i,k,g}\mu_{i,k,j,g}) + \gamma_{j,g} + \epsilon_{i,j,g}$ (혼합 모델)
-   **식 (3):** $\log(\mu_{i,k,j,g}) = \alpha_{0,k,j,g} + \sum_{\ell=1}^L x_{i,\ell,g}\alpha_{\ell,k,j,g}$ (세포별 DE 모델)

----------

### 1단계: 세포 비율($\beta$) 및 플랫폼 효과($\gamma$) 추정 (RCTD 단계)

C-SIDE 실행 전, RCTD 알고리즘을 통해 기초가 되는 두 값을 먼저 구합니다.

-   **$\beta_{i,k,g}$ (세포 유형 비율):** 각 픽셀 $i$에 포함된 세포 유형 $k$의 비중입니다.
-   **$\gamma_{j,g}$ (플랫폼 효과):** 기술적 차이를 보정하는 유전자별 무작위 효과입니다.
-   **계산 수식:** 공간 데이터를 가상 벌크(Pseudo-bulk) 데이터 $S_j$로 요약하여 추정합니다. $$\log(E[\bar{Y}_j | \lambda]) \approx \gamma_j + \log(\bar{N} \sum_{k=1}^K \mu_{k,j}\beta_k) + \log(\beta_0)$$

### 2단계: 유전자별 과분산($\sigma_\epsilon$) 추정

모델의 노이즈 수준을 결정하기 위해 유전자별 과분산 값을 구합니다.

-   **$\sigma_{\epsilon,j,g}$ (과분산 크기):** 포아송 분포 이상의 변이를 흡수하는 정규 분포의 표준 편차입니다.
-   **계산 수식:** 다음 확률 함수 $Q$를 포함하는 우도(Likelihood)를 최대화하여 MLE로 구합니다. $$Q_l(\bar{\lambda}) \equiv \int_{-\infty}^{\infty} p_{\sigma_\epsilon}(z) e^{-\bar{\lambda}e^z} \frac{(e^z \bar{\lambda})^l}{l!} dz$$ (여기서 $p_{\sigma_\epsilon}$는 표준 편차가 $\sigma_\epsilon$인 정규 분포의 PDF입니다.)

### 3단계: 차등 발현 계수($\alpha$) 추정 (핵심 최적화)

가장 중요한 단계로, 앞의 값들을 고정값(Plugin)으로 넣고 최종 목표인 $\alpha$를 구합니다.

-   **$\alpha_{\ell,k,j,g}$ (DE 계수):** 공변량 $x$에 따른 유전자 발현 변화량(Log fold-change)입니다.
-   **$\mu_{i,k,j,g}$ 및 $\lambda_{i,j,g}$:** 이 시점에서 $\alpha$의 함수로서 정의되며, $\alpha$가 수렴함에 따라 최종값이 정해집니다.
-   **계산 수식 (최적화 대상):** 다음 로그 우도 함수 $L(\alpha)$를 최대화합니다. $$L(\alpha) = \sum_{i=1}^I \log Q_{Y_i}(\bar{\lambda}_i(\alpha))$$ 이때 $\bar{\lambda}_i(\alpha) = N_i \sum_{k=1}^K \hat{\beta}_{i,k} \exp(\alpha_{0,k,j} + \sum x_{i,\ell}\alpha_{\ell,k,j})$ 식을 대입하여 계산합니다.

### 4단계: 표준 오차($s$) 계산 및 유의성 검정

추정된 $\alpha$의 통계적 유의성을 평가합니다.

-   **$s_{\ell,k,j,g}$ (표준 오차):** 계수 $\alpha$의 불확실성 정도입니다.
-   **계산 수식:** 피셔 정보 행렬($I_\alpha$)의 역행렬을 통해 구합니다. $$s_{\ell,k,j,g} = \sqrt{(I_{\alpha_{j,g}}^{-1})_{\ell,k} / n}$$ (여기서 $I_\alpha$는 로그 우도 함수 $L$의 헤시안(Hessian) 행렬인 $A(\alpha)$를 통해 추정됩니다.)

### 5단계: 반복 실험 통합 ($A$ 및 $\tau$) (메타 분석 단계)

여러 샘플의 결과를 하나로 통합하여 인구 집단 수준의 결론을 도출합니다.

-   **$\tau^2_{\ell,k,j}$ (샘플 간 분산):** 반복 실험 간의 생물학적 변이 크기입니다.
-   **$A_{\ell,k,j}$ (인구 집단 수준 DE):** 모든 샘플을 통합한 최종 효과 크기입니다.
-   **계산 수식:**
    1.  $\hat{\tau}^2 = \max { 0, V(\hat{\alpha}_{1 \dots G}) - \frac{1}{G} \sum_{g=1}^G s_g^2 }$ (Method of Moments)
    2.  $\hat{A} = \frac{\sum_{g=1}^G \hat{\alpha}_g / (\hat{\tau}^2 + s_g^2)}{\sum_{g=1}^G 1 / (\hat{\tau}^2 + s_g^2)}$ (가중 평균)

**요약:** C-SIDE는 **$\beta, \gamma$ (데이터 기초)** → **$\sigma_\epsilon$ (노이즈)** → **$\alpha$ (본체)** → **$s$ (신뢰도)** → **$\tau, A$ (통합)** 순으로 수식을 풀어가며 최종 결과를 산출합니다.


-----------------------------------------------------
# 8 RCTD에서 DE

질문에 대한 답변을 소스 자료를 바탕으로 정리해 드립니다.

결론부터 말씀드리면, **RCTD 자체적으로도 특정 세포 유형의 기대 발현량(Expected expression)을 계산하여 차등 발현(DE)을 탐지하는 기능을 갖추고 있습니다.** 하지만 C-SIDE는 이를 한 단계 더 발전시킨 전용 **모수적 통계 모델**입니다.

상세한 설명은 다음과 같습니다.

### 1. RCTD의 DE 구하는 방식 (Decompose 방식)

RCTD 논문에 따르면, RCTD는 각 픽셀의 세포 유형 비율($\beta$)을 추정한 후, 이를 바탕으로 **각 세포 유형에서 유래한 기대 유전자 발현량**을 계산할 수 있습니다.

-   **원리:** 각 픽셀에서 관측된 총 발현량을 해당 픽셀에 섞여 있는 세포들의 비율과 참조 데이터의 프로파일에 비례하여 배분합니다 (conditional expectation).
-   **활용:** 이렇게 계산된 세포 유형별 기대 발현량을 지역 간에 비교하거나 로컬 회귀(Local regression)를 적용하여 공간적으로 변하는 유전자를 찾을 수 있습니다. 소스 자료에서는 이를 **'decompose'** 방법이라고 명명하여 C-SIDE와 비교하기도 합니다.

### 2. C-SIDE가 필요한 이유 (RCTD의 한계)

RCTD의 'decompose' 방식은 픽셀별로 독립적으로 발현량을 할당하기 때문에 몇 가지 한계가 있습니다:

-   **낮은 통계적 검정력:** 세포 유형이 심하게 섞여 있거나 특정 세포 유형이 희귀한 경우, 개별 픽셀 단위의 추정치는 샘플링 노이즈에 취약합니다.
-   **편향 발생 가능성:** 주변 다른 세포 유형의 발현 시그널이 완전히 분리되지 않고 오염(Contamination)되어 실제 DE가 아닌데도 DE로 오인되는 편향이 발생할 수 있습니다.

### 3. C-SIDE의 발전된 접근 방식

C-SIDE는 RCTD가 계산한 세포 유형 비율($\beta$)을 **고정된 입력값**으로 활용하지만, 발현량을 분해하는 방식 대신 **로그 선형 모수 모델(Log-linear parametric model)**을 사용하여 직접 DE 계수($\alpha$)를 추정합니다.

-   **통합 모델링:** 픽셀들을 개별적으로 보지 않고, 전체 데이터를 공변량(위치, 거리 등)의 함수로 한꺼번에 모델링하여 **통계적 검정력을 대폭 높였습니다**.
-   **강건성:** 다른 세포 유형의 비율 변화나 플랫폼 효과 등을 통계적으로 제어하기 때문에, 단순 분해 방식보다 **훨씬 정밀하고 편향 없는 DE 결과를 도출**합니다.

**요약하자면**, RCTD 자체적으로도 기대 발현량 계산을 통해 DE를 찾을 수 있지만, 이는 초기적인 형태의 분석 방식('decompose')에 가깝습니다. **C-SIDE는 RCTD의 결과를 기반으로 하되, 더 복잡한 통계 모델을 적용하여 세포 혼합 문제 속에서도 훨씬 정확하게 특정 세포 유형의 반응을 찾아내는 전문 도구**라고 이해하시면 됩니다.
