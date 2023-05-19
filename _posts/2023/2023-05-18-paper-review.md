---
layout:     post
title:      "Paper Review"
subtitle:   "AdaSpeech: Adaptive Text To Speech for Custom Voice"
use_math: true
tags:
  - TTS, Microsoft, CustomVoice, lightweight
---

# [논문리뷰] AdaSpeech: Adaptive Text To Speech for Custom Voice
Mingjian Chen, Xu Tan, Bohan Li, Yanqing Liu, Tao Qin, Sheng Zhao, Tie-Yan Liu, [AdaSpeech: Adaptive Text To Speech for Custom Voice](https://arxiv.org/abs/2103.00993), ICLR 2021

### Abstract
Custom voice를 만들기 위해서는 해결해야 할 두가지 도전적인 과제가 있다.
+ 다양한 고객을 지원하기 위해 source 데이터와 매우 다른 다양한 acoustic condition을 다뤄야 한다.
+ 많은 고객을 지원하기 위해 adaptation 파라메터 수는 충분히 작으면서 높은 음질을 유지해야 한다.  

AdaSpeech는 이러한 문제를 풀기 위해 아래 두가지를 제안한다.
+ 다양한 acoustic condition을 다루기 위해 utterence와 phoneme level의 acoustic vector를 사용한다. pre-training과 fine-tuning 동안에 target speech로 부터 utterence-level vector를 추출하기 위한 encoder와 phoneme-level vector를 추출하기 위한 encoder를 사용한다. 추론시 utterence-level vector는 reference speech로 부터 추출하고, phoneme-level vector는 acoustic predictor를 통해 추출한다.
+ 음성 품질과 적응 매개변수 간의 균형을 잘 맞추기 위해, AdaSpeech는 mel-스펙트럼 디코더에 조건부 레이어 정규화(CLN)를 도입해서 adaptation 과정에서 speaker embedding외에 이 부분을 fine-tuning 한다.  

LibriTTS data로 source TTS model을 pre-training 하고, VCTK와 LJSpeech data(LibriTTS와는 다른 acoustic condition)의 적은 양의 데이터(예를 들면 20문장, 약 1분 데이터)로 fine-tuning 한다. 실험 결과 AdaSpeech는 화자당 5K 정도의 parameter 수로 baseline 보다 좋은 성능을 달성했다.

------------

### **AdaSpeech**
> backbone은 FastSpeech2를 사용한다.
AdaSpeech는 Custom Voice에서 해결해야 할 두가지 문제를 위해 아래 그림에서 보이는 핑크색 부분의 두가지 모듈을 도입했다.   
<img src="/img/in-post/2023/2023-05-18/fig1.png" width="200" height="300">

#### Acoustic Condition Modeling
> 다양한 고객을 지원하기 위해 source 데이터와 매우 다른 다양한 acoustic condition을 다루기 위해 acoustic condition modeling을 사용한다. source speech data는 custom voice의 모든 acoustic condition을 cover 할 수 없다. 그리고 입력 텍스트는 target speech의 acoustic condition을 표현하기에는 부족하다. 또한 model은 training data에 overfit 되는 경향이 있어, adaptation의 일반화 성능을 떨어뜨린다. 이런한 문제를 해결하기 위해 AdaSpeech는 아래 (a)같은 방식으로 acoustic condition을 모델링한다.
<img src="/img/in-post/2023/2023-05-18/fig2.png" width=400 height="200">
+ Speaker level : speaker의 전체적인 특징을 capture하기 위한 coarse-grained acoustic condition
+ **_Utterance level_** : 위 그림의 (b), utterance-level acoustic encoder의 output은 condition은 expand 후에 phoneme hidden sequence와 더해진다.   
+ **_Phoneme level_** : 위 그림의 (c), alignment 결과를 바탕으로 phoneme 단위로 mel-spectrgram을 평균을 취해서 speech frame 길이의 input을 phoneme sequence 길이로 변경한다. inference시에는 위 그림의 (d)의 phoneme-level acoustic predictor를 사용한다.

#### Conditional Layer Normalization
> 적은 파라메터 수만 업데이트 해서 좋은 품질을 얻기 위해 FastSpeech2 모듈을 분석한 결과 layer normalization이 각 self-attention과 decoder에 feed-forward network에 사용되고 있었다. 그리고 layer normalization의 scale vector $\gamma$와 bias vector $\beta$를 학습해서 최종 예측과 hidden acivation에 큰 영향을 줄 수 있었다. 
$$LN(x) = \gamma{x-\mu\over\sigma}+\beta $$
>실제 condition layer normalization은 아래와 같이 구성된다.  
<img src="/img/in-post/2023/2023-05-18/fig3.png" width="250" height="150">

#### Pipeline of AdaSpeech
> <img src="/img/in-post/2023/2023-05-18/adaspeech_alg.png" width="500" height="200">  
$W^{\gamma}_{c}$, $W^{\beta}_{c}$ : conditional layer noramlization의 scale과 bias를 위한 weight vector  
$E_{s}$ : speaker embedding  
$\gamma^{s}_{c}$, $\beta^{s}_{c}$ : $E_{s}$ * $W^{\gamma}_{c}$,  $E_{s}$ * $W^{\beta}_{c}$


### **Experimental Setup**
+ Dataset :  
  + source trainging data = LibriTTS dataset (2,456 speakers, 586 hours)  
  + adaptation data = VCTK, LibriTTS(훈련에 포함되지 않은 화자), LJSpeech(single-speaker) 에서 randomly 하게 남녀 화자, 20문장씩 선택
데이터들은 16khz로 변환했다. mel-spectrogram을 추출하기 위한 hop size는 12.5ms, window size는 50ms이다.  
+ Model Configurations :  
    + backbone = FastSpeech2 ( phoneme encoder와 mel-spectrogram decoder를 위해 4 feed-fowrd Transformer blocks로 구성)  
    + hidden dimension = 256
    + attention head = 2
    + feed-forward filter = 1024
    + kernel size = 9

    phoneme-level acoustic encoder(아래 (c))와 predictor(아래 (d))는 동일한 구조를 가졌다. 2개의 convolutional layer의 커널 사이즈는 각각 256과 3이다. 그리고 linear layer의 hidden dimension는 4(사전 실험을 통해 정해짐)이다. alignment를 위해 **MFA**를 사용했다. utterance-level acoustic encoder(아래 (b))는 2개의 convolutional layer를 가지며, filter size=256, kernel size=5, stride size=3이다. 그리고 single vector를 얻기 위해 pooling layer가 있다.  
<img src="/img/in-post/2023/2023-05-18/fig2.png" width=400 height="200">
+ Training, Adaptation and Inference
    + Training
        + 1차 = phoneme-level acoustic predictor를 제외하고 60,000 step
        + 2차 = phoneme-level acoustic predictor와 jointly를 40,000 step (phoneme-level acoustic encoder는 freeze 한 상태), phoneme-level acoustic predictor는 phoneme-level acoustic encoder의 output과 predictor의 output사이의 mse loss를 통해서 훈련됨.
        + 4 NVIDIA P40 GPU를 사용했으며, GPU당 batch size는 12,500 frames. Adam optimizer, β1= 0.9,β2= 0.98, $\epsilon$=10−9
    + Adaptation
        + 1 NVIDIA P40 GPU 2000 step
        + speaker embedding과 conditional layer-normalization만 업데이트 함.
    + Inference
        + vocoder는 MelGAN 사용
        + utterance-level acoustic condition은 reference speech로 부터 추출
        + phoneme-level acoustic condition은 phoneme-level acoustic predictor 에서 추출

### **Result**
+ The quality of Adaptation Voice  
<img src="/img/in-post/2023/2023-05-18/table1.png" width="500" height="300">  
Baseline (spk_emb) : FastSpeech2에서 speaker embedding만 fine-tune - lower bound  
Baseline (decoder) : FastSpeech2에서 decoder 전체를 fine-tune  
위 표에 3번째 컬럼은 custom voice를 adaptation 하는 동안 추가되는 파라메터 수이며, 괄호안의 숫자는 inference시에 파라메터 수를 나타낸다.  
결과를 보면 AdaSpeech가 Baseline (decoder) 보다 훨씬 적은 수의 파라메터만 fine-tune 했음에도 불구하고 음질과 화자 유사도에서 비슷한 성능을 나타내는 것을 알 수 있다.  
  
+ Method Analysis
    + Ablation Study  
    <img src="/img/in-post/2023/2023-05-18/table2.png" width="300" height="200">  
    VCTK test set에 대해 단지 speaker embedding 만 업데이트 하고, utterance-level, phoneme-level acoustic modeling, conditional layer normalization을 제거했을 때 성능이 떨어지는 것을 확인했다.  
    + Analyses on Acoustic Condition Modeling  
    LibriTTS dataset의 여러명의 화자에 대해 utterance-level acoustic vector를 추출하여 t-SNE 분석 결과를 plot 해 보았다. 갈색 원 부분처럼의 약간의 예외는 있지만 동일한 화자끼리는 가까운 곳에 위치한다는 것을 알 수 있다.  
    <img src="/img/in-post/2023/2023-05-18/fig4-a.png" width="300" height="300">  
    + Analyses on Conditional Layer Normalization  
    conditional layer normalization의 유효성을 검증하기 위해 speaker embedding condition을 제거하고 아래 두가지 경우에 대해서 비교했다. VCTK dataset에 대해서 진행
        + LN + fine-tune scale/bias : speaker embedding 과 layer normalization의 weight와 bias를 fine-tune
        + LN + fine-tune others : decoder의 다른 파라메터를 fine-tune  
    <img src="/img/in-post/2023/2023-05-18/table3.png" width="300" height="100">  
    + Varying Adaptation Data  
    VCTK와 LJSpeech에 대해 adaptation data량에 따른 성능을 평가했다. 10문장 이하에서 성능이 빠르게 감소했다.  
    <img src="/img/in-post/2023/2023-05-18/fig4-b.png" width="200" height="200">    


### **Conclusions**
> AdaSpeech에서 제안한 방법을 통해 source model data와 다른 acoustic condition의 data에 대해서도 적은 메모리 사용량으로 좋은 품질의 합성음을 만들어 낼 수 있었다. future work으로 소음 데이터와 같은 더 다양한 acoustic condition data에 대한 연구를 진행할 것이다. 또한 transcript가 없는 adaptation 데이터에 대한 연구와 더 많은 custom voice를 위한 모델 사이즈를 줄이기 위한 연구를 진행할 것이다.  

### **Sample**
+ https://speechresearch.github.io/adaspeech/




