# Fake_voice_detect_challenge
 - 2024년 7월 dacon에서 개최된 SW중심대학 디지털 경진대회 : AI부문에 참가한 내용을 정리한다.
 - Public score (0.28887, 56등), Private score (0.28995, 55등)
 - 팀원들이 사용한 모델 제외 내가 사용한 모델들과 전처리를 어떻게 했었지는 요약하도록 하겠다.

## Data EDA

먼저 제공받은 데이터들의 특성은 다음과 같았다.
![image](https://github.com/user-attachments/assets/eff8bc91-c836-4a84-af71-6244b974debf)
![image](https://github.com/user-attachments/assets/6ab722dc-ff2f-4ab0-9753-0163d729298a)
![image](https://github.com/user-attachments/assets/23715486-ac7c-4032-92ae-6b6d4e84c97f)

우선 sequence data인 음성 데이터를 모델이 입력 받을 수 있는 형태로 가공하는 과정이 필요하였다.
- fake와 real 음성은 mel-spectrogram에서 다른 특성을 보일 것이라고 판단하고 mel-spectrogram을 이용하여 데이터 전처리를 하기로 하였다. 
  - (추가적으로 제공받은 데이터들의 seq_len이 모두 달라 max_len을 기준으로 padding을 0으로 주는 과정을 거쳤다. 대회가 끝나고 좀 더 공부를 하면서 느낀점은 padding을 0으로 주고
    따로 masking을 해주지 않은 것이 성능에서 유의미한 변화가 없었던 원인중 하나일 것 같다.)
- train 데이터의 경우 중첩 음성이 거의 존재하지 않아서 제공 받은 데이터끼리 중첩시켜서 데이터 증강을 진행하였다.
- test 데이터의 경우 중첩 음성 데이터가 거의 대부분이라 모델이 음성 인식을 잘 할 수 있도록 중첩 음성을 분리해주는 과정을 거쳤다. (AV-Sepformer 사용)
- unlabeled data의 경우 활용 방안을 몰라 사용하지 못했다.. (이것도 하나의 문제점인 것 같다.)


## Model
### 1. SimCLR
- Wave 데이터를 전처리(Mel-Spectrogram + padding)하여 모델의 입력으로 넣으면 Contrastive Learning을 통해 입력 데이터의 특징 분포를 학습하는 모델
- info nce loss를 loss function으로 사용
- 데이콘 score : 0.3754
- 모델 구조 자체를 간단하게 짜서인지 아니면 데이터 전처리의 문제인지 예상과는 달리 좋은 성능을 보이지는 못하였다.
### 2. Wav2Vec 
- 비지도 학습 방식을 사용하여 원시 음성 신호로부터 유용한 표현을 학습
- 음성 데이터(raw data)를 입력받아 연속적인 잠재 벡터를 생성하고 이를 통해 음성 인식 및 다양한 음성 처리 작업에 활용할 수 있다.
- Transformer architecture를 도입하고 masked language modeling과 유사한 방식으로 훈련하는 특징이 있다.
- 이를 통해 기존의 라벨링된 데이터가 부족한 상황에서도 높은 성능을 발휘할 수 있다.
   1. pretrained 모델 large와 base를 사용하여 앙상블을 진행하면 성능에서의 개선이 있을 것이라 생각하고 먼저 기존의 raw데이터에 대해서 학습과 예측을 진행
     - score : 0.36 -> SimCLR보다 좋아졌지만 그 차이가 큰 것은 아니다.
   2. 앙상블은 동일하게 하되 증강한 데이터로 학습과 예측을 진행
     - score : 0.3527 -> 조금의 성능이 개선되었고 이를 통해 노이즈가 추가되어 증강된 데이터를 학습에 이용하는 것이 더 효율적이라는 것을 파악하였다. 
   3. 2의 조건과 동일하게 하되 loss function을 CrossEntropy Loss에서 Info NCE Loss로 변경
     - score : 0.3284 -> 손실 함수만 바꿨을 뿐인데 성능이 꽤 많이 개선되었다.
  - 이후 학습을 진행한 모델들의 결과를 보니, 해당 음성이 fake일 확률과 real일 확률이 독립적이지 못하고 서로에게 종속적인(합이 항상 1이 되는) 문제를 발견하였다.
  - 따라서 fake인 train data와 real인 train data에 대해 모델을 독립적으로 학습하고 결과를 예측하여 하나의 csv파일로 합치는 과정을 거쳤다.
  - 이렇게 진행하였을 때 최종적으로 wave2vec의 score은 0.3197로 개선이 되었다.
  
#### 추가적으로 GANomaly모델을 활용해보고 싶었지만 시간 관계상 못하였다. Wav2vec을 좀 더 튜닝을 해서 성능을 더 좋게 할 수 있었지만 시간이 촉박해서 그렇게까지 하지 못한 것이 아쉽다. 그래도 이번 대회를 통해 그동안 접해보지 못한 데이터를 다루어 보고, 여러 모델에 대해서 적용해보는 시간을 가질 수 있어 얻어가는 것이 많은거 같다. 
