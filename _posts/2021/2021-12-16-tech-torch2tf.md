---
title: pytorch를 tensorflow로 변환시 참고사항
categories:
  - Tech
tags:
  - torch
  - tensorflow
  - dnn
---

# pytorch를 tensorflow로 변환시 참고사항
<br>
* pytorch와 tensorflow의 기본적인 차이<br>
  - weight shape 이 transpose 되어 있음.<br><br>



* 변환시 각 네트웍별 특이사항

|네트웍|pytorch|tensorflow|처리방안|
|------|---|---|---|
|LSTM|bias term 2개|bias term 1개|pytorch bias term 2개를 sum 해서 TF bias에 적용|
|LSTM|batch_first = False default|batch_first = True default|input data에 맞춰<br>pytorh batch_first=True / TF time_major=False<br>pytorh batch_first=False / TF time_major=True|
|LSTM|출력 = many-to-many|출력 = many-to-one|TF return_sequences=True 인 경우 LSTM 출력 many-to-many|
|GRU|gate 순서 = reset, update, output|gate 순서 = update, reset, output|reset, update gate 순서를 바꾸고 transpose 한 후 사용|