---
layout: post
current: post
cover:  ../assets/images/paper_cover1.png
navigation: True
title: Large-Scale Image Retrieval with Attentive Deep Local Features
date: 2018-01-17 10:00:00
tags: [PAPER]
sitemap :
class: post-template
subclass: 'post tag-paper'
author: KEJdev
use_math: true
---  

### Large-Scale Image Retrieval with Attentive Deep Local Features  
Paper URL: https://arxiv.org/pdf/1612.06321.pdf 

이 논문은 CNN 계열의 DELF 라고 부르는 모델을 설명하는 논문입니다.  
CNN( Convolutional Neural Network ) 모델이 무엇인지는 설명을 안해도 아시죠?  
혹시 모르니, 나중에 CNN 모델을 더 자세하게 따로 다루겠습니다. 

CNN은 이미지 연구 분야에서 대단한 성과를 이뤘지만 CNN의 단점이 의외로 많답니다.   
혹시 대규모의 이미지 검색 시스템에서는 성능이 무척이나 떨어진다는 것을 아시나요?

실제로 CNN을 이용하여 방대한 이미지를 검색하는 시스템에서는 거의 사용하기 힘들다고 합니다. 그렇다면 CNN을 이용해서 방대한 이미지를 검색하려면 어떻게 해야댈까요 ?  

----------  

> #### The main purpose of the paper.  

이 논문은 CNN을 활용한 대규모 이미지 검색 시스템을 만드는 것이 목적입니다.  
논문에서는 CNN기반의 DELF 라는 모델을 만들었습니다.  


DELF모델은 아래의 그림과 같은 구조를 가지고 있습니다. 

<center><img src="/../assets/images/delf.png" width="850" height="500"></center>
  
모델의 구조에 대해서는 아래에서 천천히 알아볼께요.  

----------


> #### Dataset   

우선 논문에서 사용된 데이터는 구글에서 공개한 Landmark 데이터입니다.  
이 데이터는 정말 방대한 데이터이면서 여태까지 공개되어 있는 그 어떤 Landmark 데이터보다 좋은 데이터라고  알려져 있습니다.
요즘 캐글이나 Landmark 데이터에 관해 검색을 조금만 하셔도 금방 데이터를 찾아볼 수 있습니다.  

데이터는 12,849개의 명소가 포함된 총 1,060,709개의 이미지와 111,036개의 Query 이미지가 들어있습니다. 
전세계를 대상으로 추출되었고, GPS정보와 연계되어 있습니다.  

아래의 이미지는 데이터의 일부입니다.  

<center><img src="/../assets/images/(a)data.png" width="550" height="300"></center>
<center><img src="/../assets/images/(b)data.png" width="550" height="300"></center>
<center><img src="/../assets/images/(c)data.png" width="550" height="300"></center>

[Ground Truth](https://en.wikipedia.org/wiki/Ground_truth) 정보를 만들기 위해서 **아래의 두가지 feature 정보를 활용**했습니다. 
사실 정확한 Ground Truth를 만드는 것이 매우 어렵고 힘든 일입니다. 왜냐하면, GPS 정보가 잘못되어 있을 수 있고, Landmark 데이터가 너무 다양한 각도에서 촬영되어 서로 다른 구조물로 보일 수 있기 때문이예요. 그래서 실측 거리 25km 이내로 한정하여 구하면 적당히 좋은 결과를 얻을 수 있습니다.


- Visual feature 
- GPS coordinates


위의 2개의 정보를 활용하여 클러스터를 구축하였고 Query 이미지와 클러스터 거리가 특정 threshold 이내로 들어오면 동일한 이미지라고 판단합니다. 

-----------  

> #### DELF Model   

이제 DELF 모델에 대해 자세하게 설명하겠습니다.  
아래의 그림은 맨 처음에 보여드린 DELF의 전체적인 구조입니다. 
기억나시나요 ?  

<center><img src="/../assets/images/delf.png" width="850" height="500"></center>

DELF 모델의 기본적인 백본은 ResNet50입니다.  
ResNet50을 사용한 이유는 [FCN](https://blog.naver.com/PostView.nhn?blogId=laonple&logNo=220958109081&proxyReferer=https%3A%2F%2Fwww.google.com%2F) 정보를 활용하기위해서 사용했으며, pretrain 된 net에서 conv4_x를 사용했습니다.  

우선 논문에서는 image pyramid를 사용하여 여러개의 feature를 추출하였고, Landmark 데이터로 fine-tunnung 과정을 거칩니다. 
입력 이미지는 모두 center crop 한 뒤에 250x250으로 rescale 되는데, 여기서 또 244x244 크기로 랜덤 crop을 거칩니다.  
이 과정에서 object나 patch 레벨에서의 정보는 사용하지 않는다고 합니다. 

<center><img src="/../assets/images/models.png" width="700" height="500"></center>

추출된 feature를 모두 사용하는 것이 아니라 Attention을 이용해서 keypoint만 추출 한다고 합니다.
keypoint를 추출하는 이유는 시스템 효율화에도 매우 중요하며, 요즘 Attention을 이용해 신경망을 만드는 것이 트렌드이니, 혹시나 Attention을 모르신다면, 이 기회에 알아보는 것도 좋겠습니다.  

---------  

> #### Learning   

논문에서는 weighted sum을 이용한 방식을 사용했으며,  
<center>$$y=W(\sum _{ n } α(f_{ n };θ)⋅f_{ n })$$</center>

여기서 $α(f_{ n };θ)$는 score 함수입니다.  
이때의 파라미터는 
이때의 파라미터는 $θ$가 되며 $W$는 $W∈R^{ Mxd }$ 이고 $M$는 클래스의 갯수입니다. Loss 는 cross entropy loss를 사용했습니다.  
<center>$$L=−y∗⋅log(\frac { exp(y) }{  1^{T}exp(y)})$$</center>  

여기서 $y*$는  ground-truth 이고 1 는 one vector입니다. 
실제 score는 2개의 conv 레이어와 softplus 비선형 함수로 구현되어 있으며, 더 자세한 설명은 논문을 참고하시길 바랍니다.  

제안된 DELF 모델은 정말 간단하게 **요약**하자면, ature 와 attention 이 두가지가 모두 함게 학습되는 구조 입니다. 
그러나 이대로 학습을 한다면, 학습이 잘 안될 수 있어서 여기서는 2-step 학습 방식을 제안했습니다.  

간단합니다.

1. 먼저 descriptor 학습
2. descriptor 를 고정하고 score-function 을 학습.

최대한 성능을 올리기 위해 Attention 을 학습할 때 다양한 scale로 학습한다고 합니다. 
그래서 일단 center-crop을 수행하고 난 뒤 900x900 으로 rescale 합니다. 그리고 마지막으로 랜덤하게 720x720을 크롭한 뒤 r &lt;= 1 범위의 축적으로 최종 rescale 합니다.  

또한 검색 성능을 올리기 위해 최종 dim 을 줄이는데, 먼저 L2 norm을 적용 후 PCA를 돌려 40dim까지 줄입니다. 그리고 다시 L2 norm 하면 끝이 납니다.    

이 논문의 평가방법은 일반적으로 사용하는 mAP( mean average precision ) 에서 약간 바꿔서 사용했습니다. mAP는 모든 쿼리마다 얻어진 결과를 relevance 순으로 정렬한뒤 측정된 ap의 평균값을 의미합니다.   

이 논문에서는 아래의 수식으로 바꾸어 평가하였습니다.   
<center>$$Pre=\frac { ∑_{ q }|R_{ q }^{ TP }| }{ ∑_{ _{ q } }|R_{ q }| } ,Rec=\sum _{ q } |R_{ q }^{ TP }|$$</center>

<br>  
$R_{ q }$는 쿼리 q로 얻어진 검색 결과를 의미하며, $R_{ q }^{ TP }$는 true-positive 를 의미합니다.  
아래의 표는 성능표입니다.  FT는 fine-tuning, ATT는 Attention 이라는 뜻입니다.   

<center><img src="/../assets/images/pred1.png" width="550" height="300"></center>
<center><img src="/../assets/images/pred2.png" width="550" height="300"></center>
<center><img src="/../assets/images/pred3.png"></center>

<br>  

표를 보면 좋은 성능의 모델이 무엇인지 잘 알겠죠 ?  


--------  

기본 CNN을 이용해서 Landmark 데이터 셋을 이용한다면, 좋은 정확도는 절대 뽑지 못할 거라 생각해요.

이 논문을 선택한 이유는 방대한 데이터셋에서 빠르게 이미지를 찾기위해 논문을 찾다가 알게 되었고,
저는 이 논문을 기반으로 짠 코드를 Never AI 해커톤에 나갔으며, 결승까지 가게 되었습니다.

이 논문을 기반으로 짠 코드가 궁금하시다면 [여기](https://github.com/KEJdev/mandoo-model)를 클릭해주세요.  
다음에는 더 좋은 논문과 더 잘 요약된 포스팅글로 찾아뵐께요. 
