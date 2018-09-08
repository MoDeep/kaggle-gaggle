# Santander Product Recommendation Competition(산탄데르 제품 추천 대회)

![Santander Product Recommendation Competition - 1](https://kaggle2.blob.core.windows.net/competitions/kaggle/5558/media/santander-banner-ts-660x.png)

## 대회에 관하여

Item | Description
----|-----------
주최자 | Santander Bank
상금 | $ 60,000
문제 유형 | Multi-class Classification
평가 기준 | Mean Average Precision @ 7 (MAP@7)
대회 기간 | 2016.10.27 ~ 2016.12.22
참여 팀 | 1,787 teams

### Description

    Ready to make a downpayment on your first house? Or looking to leverage the equity in the home you have?
    To support needs for a range of financial decisions,
    Santander Bank offers a lending hand to their customers through personalized product recommendations.

    Under their current system,a small number of Santander’s customers receive many recommendations while many others rarely see any resulting in an uneven customer experience. In their second competition, Santander is challenging Kagglers to predict which products their existing customers will use in the next month based on their past behavior and that of similar customers.

    With a more effective recommendation system in place, Santander can better meet the individual needs of all customers and ensure their satisfaction no matter where they are in life.

    Disclaimer: This data set does not include any real Santander Spain's customer, and thus it is not representative of Spain's customer base.

## 이 대회의 목적

산탄데르 은행은 다른 은행과 마찬가지로 예금, 적금, 대출, 신용카드, 자금 관리 및 보험 등 다양한 금융 제품을 고객한테 판매한다. 그렇기에 산탄데르 은행은 자사의 금융 제품을 사용 중인 고객을 대상으로, 고객이 사용하지 않은 다른 금융 제품을 추천해주어 만족도와 매출을 잡고 싶어한다.

고객을 만족시킬 수 있는 방법에는 숙련된 영업사원이 고객이 나이, 연봉, 자산, 결혼 유무, 거주 지역, 성격 등의 정보를 토대로 이상적인 플랜을 세워주어야 하지만, 어마어마한 인력 문제로 실현이 불가능하다.

그래서 산탄데르 은행은 모두에게 이러한 서비스를 제공하기 위해 머신러닝 알고리즘을 사용하기로 했고 캐글에 대회를 주최해서 솔루션을 얻고자 하였다.

## 채점 방법

이 대회는 **고객이 신규로 구매할 제품이 무엇인지** 예측하는 대회이다. 여기서 `신규 제품`이란 지난 달 기준으로 고객이 보유하고 있지 않은 금융 제품 중, 이번 달에 새로 구매한 제품을 의미한다.

그렇기 때문에 캐글에 제출할 값은 '고객이 2016.05.28 시점에 보유하고 있지 않은 금융 제품 중, 2016.06.28에 구매할 것으로 예측되는 제품 상위 7개'이다.

제출할 때는 고객 고유 식별 번호인 ncodpers와 'ind'로 시작하는 24개의 금융 제품 중 고객이 구매할 것으로 예상되느 제품 상위 7개를 공백으로 띄워서 저장하면 된다.

이 대회에서 사용되는 평가 척도는 Mean Average Precision @ 7(이하 MAP @ 7)이다.

Mean Average Precision을 알기 전에 Average Precision을 짚고 가야된다.

Average Precision이란 **예측 정확도의 평균**을 의미한다.

7개의 금융 제품을 예측을 했다고 가정하자.(1은 정답, 0은 오답)

    # Prediction (예측 결과)
    1 0 0 1 1 1 0

예측 결과물에 대한 정확도를 측정하면 다음과 같다.

    # Precision (예측의 정확도)
    1/1 0 0 2/4 3/5 4/6 0

예측 결과물의 Average Precision은 정확도의 합을 정답의 개수만큼 나눈 숫자이다.

    # Average Precision (예측 정확도의 평균)
    (1/1 + 2/4 + 3/5 + 4/6) / 4 = 0.69

Mean Average Precision은 모든 예측 결과물의 Average Precision의 평균값을 의미한다.

여기서 @ 7은 최대 7개의 금융 제품을 예측할 수 있다는 것이다.

MAP @ 7은 예측의 순서에 아주 예민한 평가 척도인데 정답을 압쪽에 예측하는 것이 더 좋은 점수를 받는다.

    # Prediction (예측 결과)
    1 1 1 1 0 0 0

    # Precision (예측의 정확도)
    1/1 2/2 3/3 4/4 0 0 0

    # Average Percision (예측의 정확도의 평균)
    (1/1 + 2/2 + 3/3 + 4/4) / 4 = 1.00

반대로 해서 계산해보면

    # Prediction (예측 결과)
    0 0 0 1 1 1 1

    # Precision (예측의 정확도)
    0 0 0 1/4 2/5 3/6 4/7

    # Average Percision (예측의 정확도의 평균)
    (1/4 + 2/5 + 3/6 + 4/7) / 4 = 0.43

MAP @ 7 평가 척도를 구하기 위해 mapk.py 코드가 사용된다.

mapk()의 입력 값으로 들어가는 actual, predicted는 (고객수 X 7)의 dimension을 가지고 있는 2중 리스트이다.

makp.py (MAP @ 7 평가 척도를 구하는 코드)

    import numpy as np

    def apk(actual, predicted, k=7, default=0.0):
        # MAP @ 7이므로, 최대 7개만 사용한다
        if len(predicted) > k:
            predicted = predicted[:k]

        score = 0.0
        num_hits = 0.0

        for i, p in enumerate(predicted):
            # 점수를 조건하는 다음과 같다:
            # 예측 값이 정답에 있고 ('p in actual')
            # 예측 값이 중복이 아니면 ('p not in predicted[:i]')
            if p in actual and p not in predicted[:i]:
                num_hits += 1.0
                score += num_hits / (i+1.0)

        # 정답 값이 공백일 경우, 무조건 0.0을 반환
        if not actual:
            return default

        # 정답의 개수(len(actual))로 average precision을 구한다
        return score / min(len(actual), k)

    def mapk(actual, predicted, k=7, default=0.0):
        # list of list인 정답 값(actual)과 예측 값(predicted)에서 고객별 Average Precision을 구하고, np.mean()을 통해 평균을 계산한다.
        return np.mean([apk(a, p, k, default) for a, p in zip(actual, predicted)])

## 주요 접근

산탄데르 제품 추천 경진대회에서는 Tabular 형태의 시계열(Time-series)

### 시계열 데이터

A라는 시간에 B라는 고객이 C라는 제품을 구매했다라는 정보를 담은 데이터

데이터에는 고객 B에 대한 다양한 정보가 들어갈 수 있다.(ex. 고객의 나이, 직업 유무, 결혼 유무, 고객 등급, 거주 지역, 국적 등)

이 뿐만 아니라, C라는 제품에 대하여 다양한 정보가 포함될 수 있다.

### 데이터 전처리 작업

캐글 경진대회에서 제공되는 데이터는 전처리가 필요하다.

1. 입력된 값이 없음을 의미하는 결측값(NaN)이 있을 수 있기 때문에
2. 변수 분포를 따르지 않는 이상치가 있을 수 있기 때문에
3. 사람의 실수로 인해 잘못 적어진 데이터가 있을 수 있기 때문에
4. 등등....

### 피처 엔지니어링

좋은 결과를 내기 위해서는 피처 엔지니어링은 필수이다.

날짜/시간 정보가 포함된 데이터의 경우 주중/주말 여부, 공휴일 여부, 아침/낮/밤, 봄/여름/가을/겨울, 학기/시험기간/방학 등 다양한 파생 변수가 생성 가능하다.

시계열 데이터에서는 과거의 데이터를 사용하는 lag 데이터를 파생 변수로 생성할 수 있다.
