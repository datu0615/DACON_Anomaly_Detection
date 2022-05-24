# DACON_Anomaly_Detection
![20220524_125725](https://user-images.githubusercontent.com/84311270/169952625-7899a690-23f6-4e55-af92-2e812f9d214e.png)
HAI 보안데이터셋을 활용하여 AI 기반 제어시스템 보안위협 탐지연구를 활성화하고 기술 보급

## Dataset
HAICon2021 DataSet  
1. 학습 데이터셋 (6개)  
파일명: 'train1.csv', 'train2.csv', 'train3.csv', 'train4.csv', 'train5.csv', 'train6.csv'  
설명: 정상적인 운영 상황에서 수집된 데이터(각 파일별로 시간 연속성을 가짐)  
Column1 ('timestamp'): 관측 시각  
Column2, 3, …, 80 ('C01', 'C02', …, 'C86'): 상태 관측 데이터  
  
2. 검증 데이터셋 (1개)  
파일명: 'validation.csv'  
5가지 공격 상황에서 수집된 데이터(시간 연속성을 가짐)  
Column1 ('timestamp'): 관측 시각  
Column2, 3, …, 80 ('C01', 'C02', …, 'C86'): 상태 관측 데이  
Column88: 공격 라벨 (정상:'0', 공격:'1')  
 
3. 테스트 데이터셋 (3개)  
파일명: 'test1.csv', 'test2.csv', 'test3.csv'  
Column1 ('timestamp'): 관측 시각  
Column2, 3, …, 80 ('C01', 'C02', …, 'C86'): 상태 관측 데이터  

4. sample_submission.csv(제출양식)  
Column1 ('timestamp'): 관측 시각  
Column2 ('attack'): 공격 예측값(정상:'0', 공격:'1')  

5. eTaPR-21.8.2-py3-none-any.whl: 평가산식 도구  

## Model
데이터 전처리의 경우 min-max scaler를 사용하여 0에서 1 사이의 범위로 데이터를 표준화 Feature 별로 값 차이가 존재하여 모델이 조금 편향될 수 있을 것 같아 scaling.    
또한 ewm 함수를 사용하여 noise를 제거하는 방향으로 전처리를 진행.  
데이터 augmentation은 shingle을 사용하였는데 1차원 데이터를 길이 만큼의 차원 데이터로 바꾸어 주어 원래 값에서 작은 규모의 노이즈를 필터링.  
3. 모델 구성 및 학습 방법
모델 같은 경우는 Bidirectional StackedGRU를 사용.  
최종 hyper-parameter는 hidden cells은 100, layer는 3, batch size는 512, dropout은 0으로 설정.   
추가 모델 parameter로는 optimizer에는 AdamW, Metric는 MSE Loss를 사용하였는데 Adamw가 Adam보다 성능이 뛰어났으며 조금더 일반화 성능이 높다고 판단하여 설정.   
Metric는 연속형 데이터이기도 하고 특이치가 많이 존재하지 않기 때문에 mae보단 mse를 사용.  
데이터 후처리 같은 경우는 모델에서 출력된 anomaly score를 후처리하여 noise를 smoothing.   
Smoothing을 하기 위해서 lowpass를 사용하였으며 the order of the filer는 1, the ciritical frequency of the filter는 0.1.   
그 뒤 validation dataset을 이용해 학습된 모델의 threshold를 조정.   
최종 threshold는 0.007599999999999999로 선정하였으며 TaP 대비 TaR이 가장 높은 것으로 선정.  

## Results
Public Score : 0.52495, Private Score : 0.52178로 최종 14등으로 마무리.
![20220524_125747](https://user-images.githubusercontent.com/84311270/169953137-6c90ba15-e996-46ff-acfd-a6554c4e3f92.png)

