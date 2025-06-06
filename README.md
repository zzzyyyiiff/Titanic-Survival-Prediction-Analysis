# 타이타닉 생존 예측 분석

팀명: G15 (Option A)  

Dataset URL: https://www.kaggle.com/competitions/titanic/data  

Video URL: https://youtu.be/lDs4UTeCMIg?feature=shared

팀원:

      장예아 식품영양학과 zhangruiya1212@163.com (코드 구현, 데이터셋 처리) 
      
      나란 식품영양학과 956738154@qq.com (유튜브 녹화)  
      
      유카 영어영문학과 dyy20030906@gmail.com (그래프 분석)


### 데이터셋 설명

타이타닉 데이터셋은 891명의 승객 정보를 담고 있으며, 각 승객에 대해 객실 등급(Pclass), 성별(Sex), 나이(Age), 요금(Fare), 동반 가족 수(SibSp, Parch), 탑승 항구(Embarked) 등 다양한 특징과 생존 여부(Survived, 0 또는 1)가 포함되어 있습니다. 이 데이터는 1912년 타이타닉 호 침몰 사고 당시 승객들의 생존 여부를 예측하는 데 사용됩니다.

### 동기(Motivation)

이 분석을 수행하는 이유는 타이타닉 사고에서 어떤 요인들이 승객의 생존에 영향을 미쳤는지 이해하고, 이를 바탕으로 생존을 정확히 예측할 수 있는 모델을 만드는 것입니다. 머신러닝 기법을 활용해 데이터를 분석하면서 중요한 특징을 도출하고, 다양한 모델을 비교하여 최적의 예측 방법을 찾고자 합니다.


### 최종 목표

분석의 최종 목표는 타이타닉 승객의 생존 여부를 정확하게 예측할 수 있는 모델을 구축하는 것입니다. 또한, 모델이 식별한 중요한 특징들을 통해 당시 생존에 결정적인 영향을 준 요인들을 확인하고, 이를 통해 데이터에 숨겨진 의미 있는 패턴과 인사이트를 도출하는 것입니다.


## 데이터 전처리

타이타닉 데이터셋은 승객 891명의 기록을 포함하고 있으며, 승객 등급(Pclass), 성별(Sex), 나이(Age), 요금(Fare) 등의 특징과 목표 변수(0 = 사망, 1 = 생존)를 가지고 있습니다. 모델링 전, 다음과 같은 여러 전처리 과정을 수행했습니다:

- **결측값 처리**: 데이터셋에는 Age(177개 결측), Cabin(687개 결측), Embarked(2개 결측) 컬럼에 결측값이 있습니다. Age는 극단값에 영향을 주지 않도록 훈련 데이터의 중앙값(median)으로 채웠습니다. Embarked(승선 항구)는 가장 빈번한 값인 'S'로 채웠습니다. Cabin은 대부분 결측이라 구체적인 값을 보간하기보다 Cabin이 있는지 여부를 나타내는 이진 특성 HasCabin을 새로 만들어 Cabin이 존재하면 1, 없으면 0으로 표시하고 원본 Cabin 컬럼은 제거했습니다.

- **특성 엔지니어링**: 동반 가족 규모를 나타내는 FamilySize를 형제/배우자 수(SibSp)와 부모/자녀 수(Parch)를 합쳐 만들었습니다. 큰 가족은 생존에 영향을 미칠 수 있습니다(가족이 있을 경우 생존에 도움 혹은 장애 요인이 될 수 있음). 승객 등급(Pclass)은 숫자형이지만 범주형으로 처리하여 비선형적인 등급 간 차이를 반영했습니다. 이름(Name), 티켓(Ticket), 승객 ID(PassengerId)는 예측에 직접적 가치가 적어 제거했습니다.

- **범주형 변수 인코딩**: 성별(Sex)은 남성 0, 여성 1로 바이너리 인코딩했고, Embarked(C/Q/S)와 Pclass(1/2/3)는 원-핫 인코딩을 적용했습니다. 중복 정보를 피하기 위해 각각 하나의 더미 변수를 기준으로 삼아 한 범주는 제거했습니다. 예를 들어 Embarked는 C를 기준으로 두 개의 더미 컬럼(Q, S)을 남기고, Pclass는 1을 기준으로 두 개의 더미 컬럼(2, 3)을 남겼습니다.

- 이 과정을 거쳐 최종 특성 집합은 다음과 같습니다: Sex, Age, Fare, FamilySize, HasCabin, Pclass_2, Pclass_3, Embarked_Q, Embarked_S. 일부 모델(예: 신경망)에서는 Age와 Fare 등의 수치형 특성을 표준화하여 학습 안정성을 높였습니다. 이후 데이터를 80% 훈련 세트와 20% 테스트 세트로 분할해 미지의 데이터에 대한 성능을 평가했습니다.
![image](https://github.com/user-attachments/assets/80704c46-bceb-4088-a4a9-d3b47364747e)




## 모델 학습 및 비교

생존 예측을 이진 분류 문제로 다뤘으며, 단순 모델부터 복잡한 모델까지 여러 가지를 훈련해 성능을 비교했습니다:

### 기본 모델:

- **로지스틱 회귀**: 특성의 선형 조합으로 생존 확률의 로그 오즈를 예측하는 선형 모델입니다. L2 규제를 사용했고 수렴을 위해 최대 반복 횟수를 늘렸습니다.

- **의사결정 나무**: 데이터 특성 기준으로 분할하는 간단한 나무 모델입니다. 가지치기 없이 최대 깊이로 학습시켜 훈련 데이터에 최대한 적합하도록 했으며, 비선형 관계 포착에 유리합니다.

### 향상된 모델:

- **랜덤 포레스트**: 다수의 결정 트리를 부트스트랩 샘플과 랜덤한 특성 하위 집합으로 학습한 앙상블 모델입니다. 다수의 트리 예측을 집계해 과적합을 줄이고 일반화 성능을 높입니다.

- **XGBoost (극한 그래디언트 부스팅)**: 이전 트리의 오차를 순차적으로 보완하는 그래디언트 부스팅 모델입니다. 기본 파라미터로 100개의 트리를 사용했고, 정규화와 결측값 처리를 효율적으로 수행합니다.

- **신경망 (MLP)**: 두 개의 은닉층(각각 16, 8 뉴런, ReLU 활성화)으로 구성된 단순 피드포워드 다층 퍼셉트론입니다. 출력층은 시그모이드 활성화로 생존 확률을 출력하며, Adam 최적화 및 이진 교차 엔트로피 손실을 사용했습니다. 데이터가 적어 조기 종료(Early Stopping)를 활용해 과적합을 방지했습니다.

훈련은 훈련 데이터로 수행하고, 별도의 테스트 세트로 성능을 평가했습니다. 하이퍼파라미터 튜닝이나 교차검증은 수행하지 않았으며, 기본 설정으로 실험했습니다. 로지스틱 회귀와 신경망은 특성 스케일링이 필요했고, 트리 기반 모델은 스케일링 없이도 잘 작동했습니다.



## 모델 평가

모든 모델은 동일한 테스트 세트에서 다음 지표로 평가했습니다: 정확도(Accuracy), F1 점수(F1 Score), ROC-AUC. 각각 전체 정확성, 양성 클래스(생존)에서 정밀도와 재현율 균형, 분류 임계값에 무관한 판별 능력을 나타냅니다.

| 모델           | 정확도  | F1 점수 | ROC-AUC |
|---------------|--------|---------|---------|
| 로지스틱 회귀   | 0.81   | 0.74    | 0.84    |
| 의사결정 나무   | 0.76   | 0.69    | 0.75    |
| 랜덤 포레스트   | 0.79   | 0.72    | 0.83    |
| XGBoost       | 0.79   | 0.70    | 0.81    |
| 신경망 (MLP)   | 0.79   | 0.70    | 0.82    |


- 기본 로지스틱 회귀는 테스트에서 약 81.0%의 정확도와 0.74의 F1 점수, 0.84의 AUC를 기록하며 가장 우수한 성능을 보였습니다.

- 의사결정 나무는 정확도 75.98%, F1 0.69, AUC 0.75로 다른 모델에 비해 다소 낮은 성능을 나타냈으며, 생존자 분류에서 덜 세밀한 순위 판별 능력을 보였습니다.

- 향상된 모델 중 랜덤 포레스트는 정확도 78.77%, F1 0.72, AUC 0.83으로 안정적인 성능을 보여 과적합을 줄이고 견고한 패턴을 포착했습니다.

- XGBoost는 랜덤 포레스트와 비슷한 정확도(78.77%)를 기록했으나, F1 점수 0.70과 AUC 0.81로 약간 낮은 성능을 보였으며, 하이퍼파라미터 튜닝 시 개선 여지가 있습니다.

- 신경망(MLP)은 정확도 78.77%, F1 0.70, AUC 0.82로 비교적 준수한 성능을 냈으나, 데이터 양과 모델 복잡성의 한계로 인해 최고의 성능에는 미치지 못했습니다.




## 혼동 행렬 분석

### 모델별 혼동 행렬 해석

각 모델의 테스트 세트 혼동 행렬을 보면, 대부분의 모델이 음성 클래스(사망자)를 비교적 잘 분류하는 경향을 보였습니다.

####  Logistic Regression
- 참 음성(TN): 97  
- 거짓 양성(FP): 13  
- 거짓 음성(FN): 21  
- 참 양성(TP): 48  

####  Decision Tree
- 참 음성(TN): 89  
- 거짓 양성(FP): 21  
- 거짓 음성(FN): 22  
- 참 양성(TP): 47  

####  Random Forest
- 참 음성(TN): 91  
- 거짓 양성(FP): 19  
- 거짓 음성(FN): 19  
- 참 양성(TP): 50  

####  XGBoost
- 참 음성(TN): 96  
- 거짓 양성(FP): 14  
- 거짓 음성(FN): 24  
- 참 양성(TP): 45  

####  MLP (신경망)
- 참 음성(TN): 96  
- 거짓 양성(FP): 14  
- 거짓 음성(FN): 24  
- 참 양성(TP): 45  

---

###  해석 요약

- **Random Forest**는 생존자 분류(True Positive)를 50명으로 가장 많이 예측했으며, 거짓 음성(FN)을 19건으로 줄여 **재현율(Recall)** 측면에서 가장 우수했습니다. 다만, 거짓 양성(FP)은 소폭 증가했습니다.
- **Logistic Regression**과 **Decision Tree**는 각각 생존자를 48명, 47명 예측했으며, 거짓 음성은 21~22건으로 나타났습니다.
- **XGBoost**와 **MLP**는 정확히 같은 성능을 보이며, 생존자 45명을 맞추고 24명을 놓쳤습니다.

>  전체적으로 생존자 탐지가 중요한 상황에서는 **Random Forest**가 더 나은 선택이 될 수 있습니다.


![image](https://github.com/user-attachments/assets/cafc5c76-dc98-45ed-9456-9db532eb453c)


## ROC 곡선

ROC 곡선은 모든 임계값에서의 참 양성 비율(True Positive Rate)과 거짓 양성 비율(False Positive Rate)의 관계를 나타내며, 완벽한 분류기는 좌상단(참 양성률 1, 거짓 양성률 0)에 위치합니다. 검은 점선은 무작위 추정 기준선(AUC = 0.5)을 나타냅니다.

로지스틱 회귀(파란색 선, AUC = 0.84)가 가장 높은 성능을 보였고, 그 뒤를 이어 랜덤 포레스트(빨간색, AUC = 0.83), MLP 신경망(주황색, AUC = 0.82), XGBoost(보라색, AUC = 0.81) 순으로 근접한 성능을 기록했습니다. 반면, 의사결정나무(초록색 선)는 AUC = 0.75로 가장 낮은 성능을 보였습니다.

이 결과는 앙상블 모델들이 전반적으로 우수한 예측력을 보이는 가운데, 로지스틱 회귀 또한 간단하면서도 매우 경쟁력 있는 성능을 낼 수 있음을 시사합니다.

![image](https://github.com/user-attachments/assets/0c044f3d-1334-43d6-a43f-2235f170ac21)
![image](https://github.com/user-attachments/assets/89a0510e-8425-4939-9841-4dd7b6ffc922)
![image](https://github.com/user-attachments/assets/973ded00-e91c-4c56-bed8-0fa251c3202c)
![image](https://github.com/user-attachments/assets/aeb1373b-ad40-4129-a936-d68370f68e81)
![image](https://github.com/user-attachments/assets/33be0eec-bae2-4d7c-8dd7-de1ef04ca18b)


##  특성 중요도 (Random Forest 기준)

트리 기반 모델(Random Forest)의 특성 중요도 분석을 통해 생존 예측에 가장 큰 영향을 준 요인을 확인할 수 있습니다:

- **Fare (요금)**  
  가장 중요한 특성으로 나타났습니다. 높은 요금을 낸 승객일수록 (주로 1등석) 상대적으로 구조 접근성이 높고 생존 확률이 높았을 가능성이 큽니다.

- **Sex_male (성별: 남성 여부)**  
  두 번째로 중요한 변수로, 남성일 경우 생존 가능성이 낮은 경향이 있으며, 이는 역사적으로 여성과 어린이가 구조 우선 대상이었음을 반영합니다.

- **Age (나이)**  
  세 번째로 중요한 요인입니다. 어린 승객은 구조 우선 순위에 있었던 반면, 고령 승객은 생존율이 낮은 편이었습니다.

- **HasCabin (객실 정보 보유 여부)**  
  객실 정보가 있는 경우 상대적으로 상류층일 가능성이 있어 생존 확률에 긍정적 영향을 미쳤습니다.

- **FamilySize (가족 크기)**  
  중간 정도의 중요도를 가지며, 가족 규모가 클수록 모두가 구조되기 어려워 생존에 부정적 영향을 미칠 수 있습니다.

- **Pclass_3 (3등석 여부)**  
  3등석 승객의 생존율은 낮은 편이었으나, Fare와 중복 정보로 인해 상대적인 중요도는 낮아졌습니다.

- **SibSp, Parch, Embarked (탑승 항구 등)**  
  전체적으로 중요도는 낮지만, 일부 변수는 생존 가능성에 간접적 영향을 줄 수 있습니다.

**요약**: 요금, 성별, 나이와 같은 직관적 특성들이 생존 예측에서 핵심 역할을 하며, 기타 변수들은 보조적인 설명력을 제공합니다.

![image](https://github.com/user-attachments/assets/8902f730-221c-4930-99c0-ff190365066b)

## 결론

이번 분석에서는 타이타닉 승객의 생존 여부를 예측하기 위해 다양한 머신러닝 모델을 구축하고 평가했습니다. 결측 나이 처리, 성별 및 승선 항구와 같은 범주형 변수 인코딩, 가족 규모 파생 변수 생성 등 데이터 전처리를 통해 단순한 모델조차도 경쟁력 있는 성능을 낼 수 있는 기반을 마련했습니다.

여러 모델 중 **로지스틱 회귀(Logistic Regression)**가 가장 우수한 성능을 보였으며, **정확도 약 81%**, **AUC 0.84**, **F1 점수 0.74**로 전체 지표에서 최상위를 기록했습니다. 이는 잘 구성된 특성 집합이 주어진다면 단순한 선형 모델도 매우 강력할 수 있음을 보여줍니다. **랜덤 포레스트(Random Forest)**, **XGBoost**, **MLP 신경망**은 각각 **정확도 약 78.8%**를 기록했으며, AUC와 F1 점수는 로지스틱 회귀보다 약간 낮았습니다. 반면, **의사결정 나무(Decision Tree)**는 **정확도 75.9%**, **AUC 0.75**, **F1 점수 0.69**로 전체적으로 가장 낮은 성능을 보였습니다.

중요한 점은, 모든 모델이 생존에 영향을 주는 주요 요인에 대해 **일관된 패턴**을 보였다는 것입니다. **여성일수록**, **어릴수록**, 그리고 **사회경제적 지위가 높을수록** (요금이 높거나 Cabin이 있는 경우 등) 생존 확률이 크게 증가했습니다. 이는 타이타닉 참사의 역사적 사실과도 일치하며, 머신러닝 모델이 도메인 지식과 부합하는 패턴을 효과적으로 학습할 수 있음을 보여줍니다.

결론적으로, 복잡한 모델이 약간의 성능 향상을 가져올 수는 있지만, **특성 중요도 해석**과 **모델의 투명성** 역시 중요하다는 점을 다시금 확인할 수 있습니다.

