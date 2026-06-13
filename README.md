# 뇌졸중 예측 모델

## 주제

Kaggle의 Stroke Prediction Dataset을 사용하여 뇌졸중 여부를 예측하는 분류 모델을 만들었습니다.

기존 K-NN, 로지스틱 회귀, 결정 트리 모델에서 나아가 Random Forest와 XGBoost를 추가하였고, ROC-AUC 지표를 포함하여 더 종합적인 성능 비교를 진행했습니다.

## 목표

- 환자 정보를 바탕으로 뇌졸중 여부 예측
- K-NN, 로지스틱 회귀, 결정 트리, Random Forest, XGBoost 모델 성능 비교
- SMOTE로 클래스 불균형 문제 보완
- F1 Score, Precision, Recall, ROC-AUC 기반 성능 평가

## 데이터

- 출처: Kaggle Stroke Prediction Dataset
- 원본 데이터: 5,110개
- 전처리 후 데이터: 4,085개
- 타겟 변수: `stroke`
- `stroke = 0`: 뇌졸중 없음
- `stroke = 1`: 뇌졸중 발생
- 뇌졸중 데이터: 231개, 약 4.9%

## 변수

| 변수명 | 설명 |
|---|---|
| `id` | 고유 식별자 |
| `gender` | 성별 |
| `age` | 나이 |
| `hypertension` | 고혈압 여부 |
| `heart_disease` | 심장질환 여부 |
| `ever_married` | 결혼 여부 |
| `work_type` | 직업 유형 |
| `Residence_type` | 거주지 유형 |
| `avg_glucose_level` | 평균 혈당치 |
| `bmi` | 체질량 지수 |
| `smoking_status` | 흡연 상태 |
| `stroke` | 뇌졸중 여부 |

## 전처리

- 18세 초과 데이터만 사용
- `id` 컬럼 제거
- `Residence_type` 컬럼명을 `residence_type`으로 변경
- `avg_glucose_level`, `bmi` 이상치 제거 (Q3 + 3.0 × IQR 기준)
- `bmi` 결측치 평균값 대체 (파이프라인 내부에서 처리)
- 수치형 변수 표준화, 범주형 변수 원-핫 인코딩

## 탐색적 데이터 분석 (EDA)

### 수치형 변수 분포

<img width="1489" height="390" alt="numeric_distributions" src="https://github.com/user-attachments/assets/a2f9afc9-74e1-4517-8ad9-e6b3df0dd222" />

- 혈당은 100 근처에 많이 분포
- BMI는 30 근처에 많이 분포

### 범주형 변수 분포

<img width="1489" height="790" alt="categorical_distributions" src="https://github.com/user-attachments/assets/0ac7b008-3897-482b-a800-135f1f974705" />

### 뇌졸중 여부별 수치 변수 비교

<img width="1489" height="515" alt="numeric_by_stroke" src="https://github.com/user-attachments/assets/04625ab4-1489-48f5-8f71-fcdbb39fb1b4" />

- 뇌졸중 환자 그룹의 나이, 혈당이 정상 그룹보다 높은 경향
- t-검정 결과 평균 나이 차이가 통계적으로 유의미함 (p < 0.05)

### 범주형 변수별 뇌졸중 발생 비율

<img width="1489" height="1025" alt="stroke_rate_by_category" src="https://github.com/user-attachments/assets/c5303e7e-1489-4b51-9b16-fcdbb39fb1b4" />

- 고령, 고혈압, 심장질환이 있는 그룹에서 뇌졸중 발생 비율이 높음

### 수치형 변수 상관관계

<img width="551" height="490" alt="correlation_heatmap" src="https://github.com/user-attachments/assets/6e966047-f5d7-4f79-91d2-e39ff077df33" />

- 나이(`age`)와 뇌졸중 간 상관관계가 가장 높음

## 모델

- Logistic Regression
- K-NN
- Decision Tree
- Random Forest
- XGBoost
- 검증 방법: 5-Fold 교차검증 (StratifiedKFold)
- 평가 지표: F1 Score, Precision, Recall, ROC-AUC
- 데이터 누수 방지를 위해 전처리(스케일링, 인코딩)와 SMOTE를 파이프라인 내부에서 처리

## SMOTE

- 원본 데이터는 `stroke = 1` 클래스가 약 5%로 심각한 불균형 상태
- Accuracy만 보면 모델이 항상 0을 예측해도 95% 정확도가 나옴
- SMOTE로 소수 클래스를 오버샘플링하여 학습 데이터 균형을 맞춤
- SMOTE는 각 Fold의 학습 데이터에만 적용되도록 파이프라인 내부에서 처리

## 결과

<img width="1189" height="490" alt="model_comparison" src="https://github.com/user-attachments/assets/f32c2865-a533-427a-b56c-b0441c5e7e1f" />

| 모델 | F1 Score | Precision | Recall | ROC-AUC |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.2354 | 0.1394 | **0.7574** | **0.8075** |
| KNN | 0.1632 | 0.1065 | 0.3508 | 0.6494 |
| Decision Tree | 0.1737 | 0.1321 | 0.2551 | 0.5772 |
| Random Forest | 0.1309 | 0.1440 | 0.1213 | 0.7699 |
| XGBoost | 0.1165 | 0.1279 | 0.1080 | 0.7612 |

## 결론

- 뇌졸중 데이터는 클래스 불균형(약 5%)이 심해서 Accuracy만으로 평가하면 의미가 없음
- SMOTE 적용 후 Recall이 크게 향상됨 (뇌졸중을 놓치지 않는 것이 중요하므로 Recall이 핵심 지표)
- Logistic Regression이 Recall 0.7574, ROC-AUC 0.8075로 가장 우수한 성능을 보임
- Random Forest와 XGBoost는 ROC-AUC 기준으로 KNN, Decision Tree보다 높지만 Recall은 낮은 편
- 의료 데이터 특성상 뇌졸중을 놓치지 않는 것이 중요하므로 Recall과 ROC-AUC를 함께 고려해야 함
