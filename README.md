# Fake_voice_detect_challenge
 - 2024년 7월 dacon에서 개최된 SW중심대학 디지털 경진대회 : AI부문에 참가한 내용을 정리한다.
 - Public score (0.28887, 56등), Private score (0.28995, 55등)
 - 팀원들이 사용한 모델 제외 내가 사용한 모델들과 전처리를 어떻게 했었지는 요약하도록 하겠다.

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
