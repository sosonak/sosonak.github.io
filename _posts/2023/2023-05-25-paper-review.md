---
title: \[논문리뷰\] AdaSpeech2
subtitle: "AdaSpeech2: Adaptive Text To Speech for Custom Voice"
categories:
  - Paper review
tags:
  - TTS
  - Microsoft
  - adaptation
  - untranscribed data
use_math: true
---

# [논문리뷰] AdaSpeech2: Adaptive Text To Speech With Untranscribed Data
Yuzi Yan, Xu Tan, Bohan Li, Tao Qin, Sheng Zhao, Yuan Shen, Tie-Yan Liu, [AdaSpeech2: Adaptive Text To Speech With Untranscribed Data](https://arxiv.org/abs/2104.09715), ICASSP 2021

### Abstract
본 논문에서는 untranscribed speech data로 adaptive TTS를 만들 수 있는 AdaSpeech2를 개발했다. 기존에 잘 훈련된 TTS에 speech reconstruction을 위한 mel encoder를 도입하고, mel encoder의 출력이 phoneme encoder의 출력과 가까워지도록 훈련했다. 그리고 adaptation 시에 speech reconstruction을 위해 untranscribed speech data를 사용해서 TTS decoder를 fine-tune 했다. AdaSpeech2 에는 2가지 장점이 있다.  
1. Pluggable : 기존에 있던 TTS 모델을 재훈련 없이 쉽게 적용할 수 있다.  
2. Effective : untranscribed data에 대해 동일한 양의 transcribed data를 사용한 것과 동등한 성능을 달성했으며, 이전 untranscribed adaptation 방법보다 더 좋은 성능을 달성했다.  

------------

### **Introduction**
&emsp;보통 TTS adaptation pipeline은 잘 훈련된 multi-speaker TTS model을 적은 양의 데이터로 fine-tune 해서 높은 음질과 좋은 유사성을 달성한다.  
&emsp;이전의 대부분의 작업은 text와 speech의 paired data를 요구한다. 그러나 paired data는 untranscribed speech data보다 구하기가 어렵다. 따라서 TTS adatation을 위해 untranscribed speech data를 활용하면 응용 시나리오를 크게 확장할 수 있다.  
&emsp;기본적인 방법은 먼저 ASR 시스템을 사용해서 음성을 텍스트로 전사하고 이전과 동일한 작업을 수행하는 것이다. 그러니 아떤 경우에는 ASR 시스템을 사용하지 못할 수 있으며, 인식 정확도가 충분히 높지 않아서 잘못된 전사 데이터를 사용해 적응 성능에 영향을 미친다. 실제로 TTS adaptation을 위해 전사되지 않은 데이터를 사용하는 경우가 있는데, 이 경우는 TTS pipeline과 adaptation module을 공동으로 훈련해야 한다. 이러한 공동 훈련은 plugaable 하지 않고, 다른 일반 TTS 모델로 확장할 수 있는 방법을 제한한다.  
&emsp;본 논문에서는 적응을 위해 untranscribed speech data를 사용하는 AdaSpeech 2를 제안한다. backbone model로 AdaSpeech를 사용하고, untranscribed speech adaptation을 가능하게 하기 위해 잘 훈련된 소스 TTS 모델에 mel-spectrogram decoder를 추가로 도입한다. mel encoder는 L2 loss로 phoneme encoder와 가깝게 훈련하다. mel encoder를 훈련한 후에 untranscribed speech data로 speech reconstruction 방식을 통해 mel decoder를 fine-tune 한다. 단, 이 때 mel encoder와 phoneme encoder는 업데이트 하지 않는다. Inference에서 변경되지 않은 phoneme encoder와 함께 미세 조정된 mel decoder는 target speaker를 위한 custom voice를 합성할 수 있다.  
&emsp;AdaSpeech 2의 효과를 평가하기 위해 LibriTTS로 source TTS model을 훈련하고, VCTK와 LJSpeech데이터로 adaptation을 했다. 테스트 결과 MOS는 VCTK 3.38, LJSpeech 3.42 이다. 평균 SMOS는 3.87인 반면 전사된 데이터를 사용하는 방법은 3.96으로 다른 baseline 보다 높다. 그리고 내부 spontaneous ASR speech 데이터를 adaptation 데이터로 사용하는 것을 시도했다. 실험 결과 AdaSpeech 2가 baseline 보다 더 좋은 성능을 보였으며, upper bound (AdaSpeech)와 가까운 성능을 달성했다.  
### **Method**
&emsp; Fig.1 보이는 것처럼 본 논문에서 제안한 모델은 2개의 main module을 가지고 있다. 하나는 phoneme encoder와 mel-spectrogram decoder로 구성된 backbone model(AdaSpeech)이고, 또 다른 하나는 untranscribed speech data를 도입할 수 있게 해 주는 mel-spectrogram encoder이다.  
<center><img src="/img/in-post/2023/2023-05-25/fig1.png" width="200" height="300"></center>  
&emsp; Fig.2에 보이는 것 처럼 adaptation pipeline는 4가지 step으로 구성된다. 1) Source model training(multi-speaker), 2) Mel encoder aligning, phoneme encoder output과 mel encoder output을 가깝게 만들기 위한 과정, 3) Untrancribed speech adaptation, untranscribed data로 부터 mel encoder output을 사용해서 mel decoder를 adaptation 하는 과정, 4) Inference, phoneme encoder와 mel decoder를 사용해서 target speaker로 합성하는 과정  
<center><img src="/img/in-post/2023/2023-05-25/fig2.png" width="200" height="300"></center>  

#### &emsp; **Source Model Training**
  + backbone = AdaSpeech ( phoneme encoder와 mel-spectrogram decoder를 위해 4 feed-fowrd Transformer blocks로 구성)  
  + MFA aligner를 통해 groundtruth duration 구한다.  

#### &emsp; **Mel-spectrogram Encoder Aligning**
&emsp;&emsp;적응을 위해 reconstruction을 통해 전사되지 않은 음성 데이터를 활용할 수 있도록 설계된 추가 mel-spectrogram encoder를 도입한다. mel decoder의 입장에서는 음소와 mel 인코더의 출력이 같은 공간에 있기 때문에 음성 재구성에 적응된 mel decoder가 원활하게 음성 합성으로 전환될 수 있을 것으로 기대한다.  
&emsp;&emsp;잘 훈련된 TTS 모델을 얻은 후 mel encoder를 추가한다. mel encoder는 source transcribed speech data를 이용해서 mel encoder의 output과 phoneme encoder의 output이 L2 loss에 의해 가까워지도록 훈련하단. 이 때 phoneme hidden sequence의 길이는 phoneme duration에 따라 확장되어 mel encoder sequence 길이와 동일하다. mel encoder는 시스템 대칭성을 고려해 4개의 feed-forward Transformer blocks로 구성된다.  
&emsp;&emsp;본 논문에서는 source TTS model(phoneme encoder와 mel decoder) 파라메터를 고정하고, 단지 mel encoder 파라메터만 업데이트 한다. 이러한 방식은 임의의 잘 훈련된 source TTS model 사용해서 plug-and-play 방식을 형성할 수 있다. 반면에 이전 작업은 soure TTS model 훈련과 adaptation 과정을 동시에 요구하기 때문에 pluggable 하지 않고, 이는 TTS adaptationd의 광범위 사용을 제한한다.
#### &emsp; **Untranscribed Speech Adaptation**
&emsp;&emsp; untranscribed data는 mel encoder와 mel decoder를 통해 speech reconstrution 하는 방식을 통해서 모델을 fine-tune 한다. adaptation 품질을 유지하면서 가능한 적은 파라메터를 fine-tune 하기 위해서 AdaSpeech의 conditional layer normalization과 관련된 파라메터를 업데이트 한다.
#### &emsp; **Inference**  
&emsp;&emsp; 위에 과정을 통해서 unadapted phoneme encoder와 부분적으로 adapte된 mel decoder를 사용해서 특정 화자의 음성을 생성할 수 있게 된다.

### **Experimental And Results**
#### &emsp; **Datasets and Experimental Setup**
+ Dataset :  
  + source trainging data = LibriTTS dataset (2,456 speakers, 586 hours)  
  + adaptation data = VCTK (108 speakers, 44 hours), LJSpeech(single-speaker, 24 hours), internel data(연속발화 일상대화, 도전적인 시나리오 검토를 위해)
데이터들은 16khz로 변환했다. mel-spectrogram을 추출하기 위한 hop size는 12.5ms, window size는 50ms이다.  
+ Model Configurations :  
    + backbone = [AdaSpeech](https://arxiv.org/abs/2103.00993)  
    + hidden dimension = 256
    + attention head = 2
    + feed-forward filter = 1024
    + kernel size = 9
    + mel-spectrogram dim = 80
+ Training, Adaptation and Inference
    + Training
        + source model training = 100,000 step
        + mel-spectrogram align = 10,000 step
        + mel decoder adatation = 2,000 step
        + 이렇게 하는 것은 전체 시스템을 re-training 하는 것보다 쉽다.
        + 훈련에 4 NVIDIA P40 GPU, fine-tune 때는 1 NVIDIA P40을 사용했다. Adam optimizer, β1= 0.9,β2= 0.98, $\epsilon$=$10^\{-9\}$  
 
#### &emsp; **The Quality of Adaptation Voice**
&emsp;&emsp; 아래 6가지 경우에 대해 음질을 비교했다. 사용한 데이터는 VCTK에서 무작위로 남자 3명, 여자 3명을 선택했고, LJSpeech를 선택했다. 화자당 adaptation을 위해 50문장, 평가를 위해 15문장을 사용했다. 20명의 native English speaker가 자연성과 유사성을 평가했다.  
&emsp;&emsp;(1) GT : ground-truth recordings  
&emsp;&emsp;(2) GT mel + Vocoder : ground-truth mel-spectrogram을 MelGAN vocoder 로 합성  
&emsp;&emsp;(3) Joint traing : mel encoder와 phoneme encoder를 동시에 훈련  
&emsp;&emsp;(4) PPG-based : TTS를 fine-tune 하기 위해 untranscribed speech로 부터 PPG(phonetic posteriorgram)을 사용한다. PPG는 내부 ASR 모델에 의해서 추출된다. 이 시스템의 경우 mel encoder를 mel encoder와 구조는 동일하지만 input으로 PPG sequence를 받는 PPG encoder로 대체했다. (upper bound : ASR 시스템을 이용해 유사 text/phoneme 정보가 주어기지 때문에)   
&emsp;&emsp;(5) AdaSpeech : text와 speech 쌍을 사용하는 경우(upper bound)  
&emsp;&emsp;(6) AdaSpeech 2 : 본 논문에서 제안한 방법  

&emsp;&emsp;<img src="/img/in-post/2023/2023-05-25/table1.png" width="200" height="300">  
&emsp;&emsp; 위 결과를 보면 제안된 방법이 2가지 upper bound 와 동등한 음질의 성능을 달성했다. SMOS의 경우 upper bound 보다 0.1 정도 차이가 나는 정도로 약간 나쁘다.  
&emsp;&emsp; 그리고 내부 spontaneous speech data에 대해 AdaSpeech(transcribed speech adaptation)와 본 논문에서 제안한 방식에 대해 CMOS test를 했을 때 동등한 음질을 나타냈다. (AdaSpeech가 단지 0.012 정도 높았다.)
#### &emsp; **Analyes on Adaptation Strategy**
&emsp;&emsp;adaptation 전략의 효율성을 증명하기 위해 2가지 실험을 진행했다.
첫번째, mel encoder를 훈련할 때 mel encoder의 output과 phoneme encoder의 output의 L2 loss로 훈련하는 것에 대한 효율성을 검증하기 ablation study를 진행했다. 이것을 검증하기 위해 mel encoder 훈련시 다른 loss는 남겨둔 상태에서 L2 loss만 없앴다. Table 2에 보여진 결과를 보면 L2 loss를 사용하는 것이 필요하다는 것을 알 수 있다. 두번째는 mel encoder와 phoneme encoder의 output space의 일괄성을 유지하기 위해 mel decoder만 adaptation 하는 방식의 유효성을 증명하기 위해, adaptation시 mel decoder와 mel encoder 전체를 fine-tune 하는 실험을 진행했다. Table 2를 보면 mel encoder를 adapting 할 경우 음질이 감소하는 것을 알 수 있다.  
&emsp;&emsp;<img src="/img/in-post/2023/2023-05-25/table2.png" width="200" height="300">
#### &emsp; **Varying Adaptation Data**  
&emsp;&emsp;VCTK와 LJSpeech data에 대해 adaptation 데이터양에 따른 CMOS를 평가했다. Fig.3 에서 보이는 것처럼 adaptation 양이 20개 보다 적은 경우는 데이터양이 증가할수록 성능이 크게 개선되지만, 그 보다 많은 경우에는 성능개선이 뚜렷하게 보이지 않는다.  
&emsp;&emsp;<img src="/img/in-post/2023/2023-05-25/fig3.png" width="200" height="300">
  
### **Conclusions**
&emsp;본 논문에서는 untranscribed speech data를 사용해서 pluggable, effective한 adaptive TTS 시스템을 개발했다. 기존 TTS pipeline 조절 없이 기존 TTS pipeline에 mel-spectrogram encoder를 추가해서, mel-spectrogram encoder의 출력과 phoneme encoder의 출력이 가까워지도록 훈련했다. 전체 시스템은 source model 훈련, mel-spectrogram encoder aligning, untranscribed speech adaptation, 그리고 inference 형태로 pipeline이 구성된다. 우리는 MOS와 SMOS 관련에서 upper bound system에 가깝거나 유사한 성능을 달성했다. 향후에 우리는 음성 품질과 유사성을 개선하기 위해 다양한 적응 방법을 탐색하고, spontaneous speech와 같은 더 도전적인 시나리오로 방법을 확장할 것이다.

### **Sample**
+ https://speechresearch.github.io/adaspeech2/




