---
title: \[논문리뷰\] AdaSpeech3
subtitle: "AdaSpeech3: Adaptive Text To Speech for Spontaneous Style"
categories:
  - Paper review
tags:
  - TTS
  - Microsoft
  - adaptation
  - spontaneous style
use_math: true
---

# [논문리뷰] AdaSpeech3: Adaptive Text To Speech for Spontaneous Style
Yuzi Yan, Xu Tan, Bohan Li, Guangyan Zhang, Tao Qin, Sheng Zhao, Yuan Shen,Wei-Qiang Zhang, Tie-Yan Liu, [AdaSpeech3: Adaptive Text To Speech for Spontaneous Style](https://arxiv.org/abs/2107.02530), INTERSPEECH 2021

### Abstract
최근의 TTS는 reading-style 합성은 매우 잘 하지만 자유발화를 합성하는 것은 도전적인 과제이다. 그 이유는 아래와 같다.  
1. 자유발화 데이터가 부족하다. 
2. 자유발화에서 filled pauses(FP:um and uh와 다양한 리듬을 모델링하는 것이 어렵다.  

본 논문에서 잘 훈련된 reading-style 합성기를 자유발화 합성을 위해 fine-tuning 하는 방법인 AdaSpeech3 을 개발했다. 구체적으로 아래 3가지 이다.  
1. 텍스트 시쿼스에 FP를 적절하게 삽입하기 위해 TTS 모델에 FP 예측기를 도입했다.
2. 다양한 리듬을 모델링하기 위해 mixture of experts(MoE) 기반의 duration 예측기를 도입했다. 
3. 다른 음색으로 적응하기 위해 적은 데이터로 디코더 파라메터를 fine-tune 했다.  

자유발화 연구를 위한 데이터 부족 문제를 해결하기 위해 데이터를 직접 수집했다. 실험에 따르면 AdaSpeech3은 자연스러운 FP와 자유발화 스타일의 리듬으로 음성을 합성하고 이전의 적응형 TTS 시스템보다 훨씬 더 나은 MOS 및 SMOS 점수를 달성했다.

------------

### 1. Introduction
&emsp;reading-style 합성 관련해서는 많은 모델들이 좋은 음질을 달성했다. 하지만 대화, 토크쇼, 팟캐스트와 같은 자유발화 합성에 대해서는 잘 연구되지 않았다. 이전 연구에 따르면 자유발화 음성은 reading-style 음성과 2가지 차이점이 있다.
1. filled-pause(FP) 존재 : <i>um</i>과 <i>uh</i>와 같은 pause
2. 다양한 리듬: 자유발화시에 화자는 그들의 억양을 변화시키는데, 한 음절을 길게, 혹은 짧게 발성한다.  

&emsp;자유발화 관련된 이전 연구들이 있지만 대부분 통계적 매개 변수 음성 합성으로 제한된다. 일부 연구에서 자유발화 음성을 합성하기 위해 신경망 기반 TTS 시스템을 적용했지만, FP 및 다양한 리듬과 같은 필수적인 특성은 고려하지 않고 모델링 되었습니다. 즉, 자유발화 합성을 위해 특별히 설계된 신경망 구조는 없다. 게다가, reading-style 음성과 비교해서 자유발화 합성을 위한 훈련데이터는 매우 제한적이다.   
&emsp;본 논문에서는 자유발화 스타일 음성 합성을 위해 잘 훈련된 reading-style의 TTS 모델을 미세 조정하는 adaptive TTS 시스템인 AdaSpeech 3을 개발했다. AdaSpeech3은 이전 adaptive TTS 시스템을 기반으로 하며, 자유발화 음성을 위해 다음과 같은 특정 설계를 추가했다.  
1. FP를 텍스트 시퀀스에 적절하게 삽입하기 위해 FP 예측기를 TTS 모델에 도입한다.
2. 다양한 리듬을 포착하는 어려움을 고려하여 MoE(Mixture of Expert) 기반의 duration 예측기를 설계했고, TTS 모델의 피치 예측기 뿐만 아니라 duration 예측기도 fine-tune 했다.
3. AdaSpeech3의 훈련을 지원하기 위해 인터넷에서 수집된 팟캐스트에서 자유발화 음성 데이터를 수집했다. 여기에는 AdaSpeech3의 핵심 적용 요소와 일치하는 3개의 서브셋을 포함한다.  
3-1. SPON-FP : PF 예측기를 훈련하기 위핸 text-FP data pair  
3-2. SPON-RHYTHM : MoE 기반 duration 예측기와 피치 예측기를 fine-tune 하기 위한 text-pitch/duration data pair  
3-3. SPON-TIMBRE : 화자 음색 적응을 위한 text-speech data pair  

&emsp;AdaSpeech3의 3가지 주요 장점은 아래와 같다.  
1. Data efficiency : 잘 훈련된 reading-style 합성기로 부터 모델을 적응한다. 반면, FP/피치/dutaion 예측기를 적용하기 위해서 쉽게 접근 가능한 text-FP와 text-pitch/duration data pair만 필요하다. 즉, 부족한 고품질의 자유발화 데이터에 대한 요구를 줄인다.
2. Controllability : FP 예측기의 예측 threshold를 조절해서 FP의 발생 정도를 조절할 수 있다.
3. Transferability : 자유발화이든 아니든 상관없이 적은 음성 데이터로 다른 화자의 자유발화 스타일로 변환할 수 있다.   

&emsp;AdaSpeech3의 성능을 평가하기 위해 다양한 실험을 했다. MOS 평가에서 AdaSpeech(단지 SPON-TIMBRE 데이터세으로 적응한 경우) 보다 0.24점 더 높았다. 자유발화 스타일 유사성 평가인 SMOS 평가에서는 AdaSpeech3이 AdaSpeech 보다 0.3점 더 높았다. 추가적인 분석은 AdaSpeech3의 추가된 모듈의 효과를 입증했다.  

### 2. Spontaneous Dataset Mining
&emsp; 공개적으로 사용 가능한 자유발화 음성 데이터 세트가 없어서 본 논문에서는 직접 데이터를 수집했다. 데이터 세트 수집은 아래와 같이 여러 단계로 구성된다.
* Data crawling  
  - "ThinkComputers" 팟캐스트로 부터 untranscribed 자유발화 음성 데이터를 수집했다. 첫 30개의 에피소드에서 약 28시간 데이터를 선택했다.  
* ASR Transcription
  - 내부 ASR 툴을 이용해 FP 표시와 timestamp를 포함한 transcription을 얻었다. 얻어진 timestamp를 이용해서 음성 데이터와 텍스트를 7~10초 길이로 잘랐다. 얻어진 텍스트를 g2p를 이용해 phoneme열로 변환했다.
* SPON-FP dataset construction
  -  FP 예측기를 훈련을 위한 텍스트-FP 데이터 쌍을 얻기 위해 원래 음소 시퀀스에서 FP를 제거하여 FP가 없는 시퀀스를 얻고 각 음소에 대한 FP 태그를 추출했다. FP 태그는 다음과 같이 정의된다. 음소 뒤에 <i>uh</i> 나 <i>um</i>이 있으면 1 또는 2이고, 그렇지 않으면 0이다. 표 1은 음소 시퀀스와 해당 FP 태그로 구성된 텍스트-FP 데이터 쌍의 예를 보여 준다.
  <center><img src="/img/in-post/2023/2023-06-27/table1.png" width="200" height="300"></center>
* SPON-RHYTHM dataset construction  
  - 자유발화 음성 데이터로 부터 피치를 추출하고, forced-alignment tool을 통해서 duration을 구했다. SPON-RHYTHM dataset은 피치와 duration 예측기를 fine-tune 하기 위해 음소단위의 피치와 duration을 포함한다.
* SPON-TIMBRE dataset construction  
  - 화자 적응에 사용할 음소 시퀀스와 자유발화 음성 데이터쌍이다.  

&emsp; 3가지 데이터 셋에 통계치는 table2에 보여준다. SPON-FP의 경우 338개의 <i>um</i>와 2614개의 <i>uh</i>이 있다.
<center><img src="/img/in-post/2023/2023-06-27/table2.png" width="200" height="300"></center>  

### 3. Method

#### 3.1 Model Overview
&emsp;&emsp;AdaSpeech3은 Fig1에 보여진 것처럼 3가지 부분으로 구성된다.  
<center><img src="/img/in-post/2023/2023-06-27/fig1.png" width="200" height="300"></center>  

&emsp;&emsp;1) multi-speaker TTS model : AdaSpeech  
&emsp;&emsp;2) FP 예측기 : 문장에서 적절한 위치에 FP를 삽입한다.  
&emsp;&emsp;3) MoE 기반의 duration 예측기  
자유발화 스타일로 TTS를 적응하기 위한 pipeline은 아래와 같은 단계로 구성된다.
* Source model training  
4개의 fead-forward Transformer 블럭으로 구성된 AdaSpeech 모델을 사용했다.
* FP 예측기 적응  
FP는 자유 발화의 중용한 특성이며, 문장에 적절한 FP를 추가하면 자유발화 스타일을 증가시킬 수 있다. 각 음소에 대해 FP tag(no FP:0, <i>uh</i>: 1, <i>um</i>:2)를 예측하는 FP 예측기를 음소 인코더 다음에 추가했다. 만약 FP tag가 1 혹은 2라면 일치하는 음소 바로 뒤에 <i>uh</i>, <i>um</i> embedding을 추가했다. SPON-FP는 FP 예측기를 적응하는데 사용했다. 자세한 내용은 3.2 부분에 소개한다.  
* 리듬 적응  
FP외에 리듬은 reading-style 음성으로 부터 자유발화 음성을 구분하는 중요한 요소이다. 자유발화 음성과 더 유사하게 만들기 위해 리듬(duration 과 피치)도 적응했다. 피치 예측기는 단지 fine-tuning만 했다. duration은 자유발화의 큰 변화를 다루기 위해 MoE 기반의 duration 예측기를 설계했다. SPON-RHYTHM은 피치와 duration을 적응하는데 사용했다. 자세한 내용은 3.3. 부분에 소개한다.   
* 화자 음색 적응  
음색을 적응하기 위해 적응할 데이터(ex. SPON-TIMBRE dataset or VCTK)를 사용해서 AdaSpeechd의 speaker embedding과 conditional layer normalization을 fine-tune 했다. 이 방식으로 적은 데이터양(ex. 20발화)으로 임의의 custom voice로 자유발화 스타일로 전환할 수 있다. 적응 데이터가 꼭 자유발화 스타일일 필요는 없다.
#### 3.2 FP Predictor Adaptation
&emsp;&emsp; FP 예측기는 input으로 phoneme hidden squence를 취하고, 각 음소에 대해 FP tag(3개 클래스, 0: FP 없음, 1: <i>uh</i>, 2: <i>um</i>)를 예측한다. FP 예측기는 1) ReLU 활성화가 있는 2-layer 1D-convolution 네트워크, 각각 레이어 정귝화 및 드롭아웃 레이어가 뒤따르고 2) 각 태그를 예측하기 위한 추가 linear layer와 3방향 소프트맥스 계층이 있다.  
&emsp;&emsp; FP 예측기 훈련시 문제는 positivie label(즉, 1 또는 2)이 매우 희소하다는 것이다. 즉, FP를 포한하는 문장 비율이 적고, positive FP 태그(1 또는 2)를 포함하는 토큰의 비율이 매우 작습니다.(ex, 20개 이상의 토큰이 있는 문장에는 일반적으로 <i>um</i> 또는 <i>uh</i>가 하나 있다.). 이러한 방식은 모델이 모든 토큰에 대해 FP 없음(tag:0)을 쉽게 예측하게 한다. 데이터 희소 문제를 완하하기 위해 1) Table1에 언급된 SPON-FP 데이터 셋을 사용한다. 이 때, FP가 없는 문장은 폐기해서 positive label의 밀도를 높인다. 2) 그리고 가중 교차 엔트로피 함수를 손실 함수로 사용한다.  
\$\$L=-y_0logs_0-\sigma\sum_{i=1}^{2}y_ilogs_i \$\$
\$\$[s_0, s_1, s_2]: \text{probability of the output belonging to three specific categories} \$\$
\$\$[y_0, y_1, y_2]: \text{one\-hot encoding of the ground-truth label}\$\$  
$\sigma\$ 는 훈련시 데이터의 균현을 보장하기 위해 조절할 수 있다. 게다가 추론시에 FP 예측기의 threshold를 조절해서 FP의 강도(발화 내에 얼마나 많은 FP를 포함할지)를 조절할 수 있다. 자세한 건 4.3에 열거했다. FP 예측 후 figure 1과 같이 phoneme hidden squence의 해당 위치에 FP 임베딩을 추가한다.  
#### 3.3 Rhythm Adaptation 
&emsp;&emsp; TTS 모델의 리듬(피치, duration)을 잘 적응하기 위해 자유발화(SPON-RHYTHM)와 reading-style(LibriTTS, VCTK)의 데이터 사이에 피치와 duration의 분포 차이를 분석했다. 1) 피치는 각 화자에 대해 자유발화이든 reading-style이든 상관없이 유사하게 가우시안 분포를 따른다. 2) 그러나 duration의 경우 앞서 언급한 것처럼 자유발화는 음절을 길게 하거나 짧게 하는 경우가 있어 음절 분포의 차이가 발생한다. LibriTTS, VCTK는 각 음소의 길이가 대부분 0~25 사이에 분포된 반면, SPON-RHYTHM은 25를 넘는 경우가 꽤 있다. 또한 자유발화의 경우 0~40 (LibriTTS, VCTK의 꼬리 분포 범위)범위에 고르게 분포한다. 통계 결과는 duration 예측기가 보다 세밀하게 음성 속도를 조절 하도록 권장한다.  
&emsp;&emsp; 따라서 피치를 적응하기 위해서는 간단한 전략을 채택하고, duration 예측기의 경우 넓은 범위의 duration을 잘 캡처하기 위해 속도 라우터와 3개의 전문가 예측기로 구성된 MoE를 도입했다. MoE 기반 예측기를 구축하는 프로세트는 다음과 같다.
1) 음소의 duration을 저속, 중속, 고속으로 균등하게 분류하며, 이를 속도 태그라고 부른다. 2) phoneme hidden과 해당 음소의 속도 태그를 사용하여 FP 예측기와 동일한 구조를 공유하는 속도 라우터를 훈련한다. 3) 잘 훈련된 소스 TTS 모델의 duration 예측기로 부터 3개의 duration 전문가를 초기화 한다. 그 후에 각 duration 전문가는 phoneme hidden과 duration 쌍을 이용해 각각 저속, 중속, 고속으로 미세 조정한다. (Figure 1 참조) 4) ground-truth duration은 학습시에 phoneme hidden squence를 확장하는데 사용된다. 반면 추론시에는 예측 duration을 사용한다. MoE로 부터 예측 duration을 얻기 위해서 3개 전문가 예측치의 가중 평균을 계산한다. 여기서 가중치는 속도 라우터에서 예측한 확률이다.  

### 4. Experiments and Results
#### 4.1 Experimental Setting
+ Dataset :  
  + source trainging data = LibriTTS dataset   
  + adaptation data = VCTK 에서 여성 2명, 남성 2명 (각 50 utterances)  
+ Model Configurations :  
    + backbone = [AdaSpeech](https://arxiv.org/abs/2103.00993)  
    + hidden dimension = 256
    + attention head = 2
    + feed-forward filter = 1024
    + kernel size = 9
    + mel-spectrogram dim = 80
+ Training, Adaptation and Inference
    + Training
        + source model training = 100,000 step (4 NVIDIA P40 GPU)
        + FP predictor adaptation = 4,000 step
        + 피치 predictor, duration predictor(속도 라우터와 3개의 전문가 포함) adaptation = 4,000 step
        + speaker adaptation = 2,000 step (AdaSpeech 설정에 맞춰서)
        + 모든 adaptation은 1 NVIDIA P40을 사용했다. Adam optimizer, β1= 0.9,β2= 0.98, $\epsilon$=$10^\{-9\}$  

        합성음 평가를 위해서 MOS, SMOS, CMOS 진행했다.
 
#### 4.2 The Quality of AdaSpeech 3  
&emsp;&emsp; AdaSpeech3와 아래 3가지에 대해 비교 평가했다.  
1\) GT : 녹음 데이터  
2\) GT mel + Vocoder(Voc) : ground-truth mel-spectorm으로 음성을 합성하기 위해 MelGAN vocoder 사용  
3\) AdaSpeech : reading-style TTS adaptation 시스템
AdaSpeech3은 AdaSpeech와 동일한 소스 모델을 사용하지만 AdaSpeech3는 FP, 리듬 및 화자 음색 적응을 수행하는 반면 AdaSpeech는 음색 적응만 수행한다는 점에서 다르다. AdaSpeech와의 비교를 통해 AdaSpeech3에서 자유발화를 위해 특별히 고안된 adaptation 모듈의 효과를 입증할 수 있다.  
&emsp;&emsp; nativie English 화자 20명(15문장)에게 유사성과 자연성을 평가했다. SMOS의 경우 피치, 속도, 음량, 억양, 리듬 및 강세 측면에서 평가했고, table3에 결과를 표시했다.  MOS 결과는 table4에 표시했으며, AdaSpeech3가 모든 측면에서 AdaSpeech를 능가하는 것을 알 수 있다.
<center><img src="/img/in-post/2023/2023-06-27/table3.png" width="200" height="300"></center>  
<center><img src="/img/in-post/2023/2023-06-27/table4.png" width="200" height="300"></center>  

#### 4.3 Method Analyses    
<b>Effectiveness of FP prediction</b>   
&emsp;&emsp;FP 예측을 문장에 무작위로 같은 양의 FP를 삽입한 경우와 비교해서 FP 삽입의 합리성에 초점을 맞춘 CMOS 평가를 수행했다. 결과 FP 예측이 무작위 삽입 설정보다 CMOS가 0.156 높게 나왔다.  
&emsp;&emsp;3.2에서 설명한 것처럼 예측 임계값을 변경하여 FP 강도를 제어할 수 있다. 세가지 범주에 해당하는 예측 확률은 $s_0$, $s_1$,$s_2$(FP 없음, <i>uh</i>, <i>um</i>)로 표현된다. 만약, $s_0$ 값이 임계치 $T$보다 크다면, FP 없음으로 분류되고, 그렇지 않으면 확률이 더 큰 FP로 분류한다. 리콜, 정밀도, 정확도 비율은 SPON-FP의 40개 테스트 문장에서 측정했다. $T$가 0.10에서 0.99로 증가하면 재현율이 0.800에서 0.950으로 증가한다(정확도는 약간 떨어짐). 이것은 시스템이 FP를 삽입하는 경향이 높다는 것을 의미한다.  

<b>Effectiveness of rhythm adaptation</b>  
&emsp;&emsp;추가된 모듈에 대한 성능을 평가하기 위해 톤과 리듬에 초점을 두고 CMOS 평가를 진행했다. 2가지 설정(FP 삽입이 있는 경우와 없는 경우)으로 평가를 진행했고, 결과는 table5에 표시했다. 아래와 같이 여러가지를 관찰했다.
1\) MoE 기반 예측기를 단일 예측기(w/o MoE로 표기)로 대체하고 미세 조정을 하면 일반적인 폼질 저하가 발생하고, 이는 예측기에서 MoE 설계의 이점을 보여 준다.  
2\) 단일 기간 예측기에서 미세 조정을 추가로 제거하면 (w/o duration adaptation으로 표기) 더 많은 품질 저하가 발생하여 미세 조정의 효과를 증명한다.  
3\) 피치 미세 조정을 제거하면(w/o pitch adaptation) 품질 저하가 발생하지만, duration 적응을 제거하는 것보다는 저하가 작다.  
4\) 피치와 duration 적응 모두 제거하면(w/o pitch/duration adaptation) 품질이 더욱 저하된다.  
5\) 모든 변형의 품질 저하는 FP 설정이 FP 없는 설정보다 더 심각하다. 이는 FP를 삽입할 때 리듬 미세조정이 자유발화 스타일을 향상시킬 수 있음을 나타낸다. 리듬 미세조정과 FP 예측은 자유발화 합성을 보완한다는 사실을 보여준다.
<center><img src="/img/in-post/2023/2023-06-27/table5.png" width="200" height="300"></center>  


#### 4.4 Cross-Speaker Spontaneous-Style Transfer    
&emsp;&emsp; AdaSpeech3은 적은 양의 reading-style 데이터를 사용하여 자유발화 데이터가 없는 화자를 자유발화 스타일으로 변환할 수 있다. 화자 음색 적응을 위해 VCTK 데이터 셋에서 남성 2명, 여성 2명을 선택해서 SPON-TIMBRE 테스트 셋의 동일한 텍스트를 기반으로 합성했다. 톤과 리듬에 초점을 맞춘 CMOS 테스트를 통해 자유발화 스타일의 개선 정도를 평가했다. AdaSpeech3은 CMOS 테스트에서 AdaSpeech 보다 남성음에서 0.175, 여성음에서 0.205 만큼 높았다.
  
### 5. Conclusions
&emsp;본 논문에서는 자유발화 음성을 위한 adaptive TTS system인 AdaSpeech3를 개발했다. 본 논문의 연구 작업을 가능하게 하고 이 분야에서 잠재적인 미래 연구를 용이하게 하기 위해 자유발화 스타일 적응을 위한 세가지 하위 집합(SPON-FP, SPON-RHYTHM 및 SPON-TIMBRE)을 포함하는 자유발화 음성 데이터 세트를 수집했다. 문장 내의 적절한 FP를 추가하기 위한 FP 예측기를 설계했고, 자유발화 음성의 다양한 리듬을 포착하기 위해 MoE 기반의 duration 예측기를 도입했다. AdaSpeech3는 AdaSpeech와 비교했을 때 MOS, SMOS, CMOS 관점에서 더 좋은 음질을 달성했다. 향후 연구를 위해, 자유발화 TTS를 개선하기 위해 반복 및 담화 marker와 같은 자유발화 음성에 더 많은 특성을 탐구할 것이다.

### 6. Sample
+ https://speechresearch.github.io/adaspeech3/




