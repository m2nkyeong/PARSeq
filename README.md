# PARSeq
> PARSeq : Scene Text Recognition with Permuted Autoregressive Sequence Models Review     
> PARSeq GitHub Link : https://github.com/baudm/parseq.git

### Abstract
Context-aware STR methods는 전통적으로 internalautoregressive( AR ) laguage models ( LM )이다.     
AR의 본질적인 한계는 Two-stage methods가 나오도록 했는데, External LM을 사용하는 것이다.     
External LM의 Conditional Independence는 정확한n예측을 잘못하여 교정하는 현상을 야기할 수 있다.     
저자들의 PARSeq는 Permutation Language Modeling을 사용하여 공유하는 weights를 활용해 Internal AR LMs의 ensemble을 학습한다.     
이는 Context-free non-AR 과 Context-aware AR inference를 단일화한다.     
그리고 Bidirectional Context를 사용해 반복적으로 Refinement한다.     
Synthetic Training Data를 사용해서, PARSeq는 SOTA results를 달성한다.      
PARSeq는 Accuracy vs Parameter count & Latency 에 대해 최적화하는데 이는 simple하고 unified한 structure와 Parallel Token Processing 때문이다.     
Attention의 광범위한 사용 때문에, 임의의 방향의 text에 견고하다.


### Instruction
Natural Scenes에서 Machines가 text를 읽는 방법은 먼저 text regions를 detect하고 text를 해당 regions에서 인지한다.     
recognizing text의 task는 Scene Text Recoginizing Recognition(STR)이라고 불린다.     
STR은 raod signs나 billboard, paper bills, product labels, logos, printed shirts 등을 읽는다.     
이는 자율주행이나 augmented reality, retail, education 등 분야에 쓰인다.     
문서들에 쓰이는 Optical Character Recognition(OCR)은 Text Attributes가 더 uniform한 반면, STR은 다양한 font styles와 방향,모양, 조명, 많은 양의 가려짐, Sensor Condition을 다룬다.     
Natural Environments에서 Captured된 이미지는 Noisy하고 Blurry하게 되어 있고 Distorted 되어 있기도 하다.     
▶️ 본질적으로 STR은 중요하지만 매우 어려운 문제이다.


STR은 vision task인데, Text 부분이 읽는 데 중요한 역할은 한다.     
Image features 단독으로만 정확한 Inferences를 하기 어려운 문제가 있다.     
어떤 경우, Language Semantics가 전통적으로 Recognition Process를 돕기 위해서 사용된다.     
Context-aware STR methods는 semantic priors를 통합하는 데 word representation model이나 dictionary 또는 sequence modeling을 사용한 data로 부터 학습되어지는 priors를 통합한다.     

Squence Modeling은 End-to-End로 학습 가능한 LM이라는 장점을 가진다.      
Internal LMs를 활용한 STR methods는 Image features와 Language Context를 동시에 처리한다.     
이들은 future tokens가 past tokens에 영향을 미치는 Language Context에 대한 제한적인 Autoregressive(AR)를 활용함으로써 학습된다.    
여기서 Model은 $P(y|x)$를 결과로 주는 데 여기서 y는 Image x에서 T-length Text Label이다.     
AR models는 이런 제약조건으로부터 두 가지의 한계를 가진다.    
* 먼저, 이 model은 **한 방향에 대해서만** token dependencies를 학습할 수 있다. 
  보통, Left-to-Right(LTR) 방향이다.     
  이 unidirectionality는 AR models가 한 방향으로 읽도록 편향되게 하는 것을 야기한다.     
  이는 결과적으로 잘못된 접미사를 추가하거나, 방향 의존적인 예측값을 내놓는다.     
* 둘째로, 추론하는 동안에 AR model은 output tokens를 serally하게 같은 방향으로 출력하는데 사용된다.      

▶️ 이를 **next-token**이나 **monotonic AR decoding**이라고 부른다.    

이 문제를 다루기 위해, 기존 연구들은 Left-to-Right와 Right-to-Left AR models를 합쳐왔다.     
또 다른 연구들은 external LM이나 단독적으로 Context-free STR의 ensemble을 사용하는 Two-stage approach를 선택해왔다.    
LTR과 RTL AR model을 합친 model은 여전히 unidirectional context 문제로 골머리를 썩지만 두 방향으로의 stream에 대해서는 좋은 성능을 보인다.     
본질적으로, 이는 decoding time과 complexity를 증가시킨다.     
그러는 반면, Two-stage ensemble approaches는 Figure 1a에 나와있는데 initial predictions를 parallel한 non-AR decoding을 사용해 얻는다.     
<p align="center"><img src="https://media.springernature.com/lw685/springer-static/image/chp%3A10.1007%2F978-3-031-19815-1_11/MediaObjects/539995_1_En_11_Fig1_HTML.png" width="80%" height="80%"></p>  

initial Context-less prediction은 직접 Decoded된다.     
이미지로부터 Context-free model을 사용한다.     
예측값은 $P(y|x)$인데 external LM을 사용하는데, bidirectional Context를 활용한다.     
▶️ 왜냐하면, All Character가 한번에 이용될 수 있기 때문이다. (LM은 spell check로 사용된다.)    

initial prediction을 재조정하고 context-based ouput을 생성한다.     
input image로부터 LM의 Conditional Independence는 정확한 예측을 수정하는 결과를 야기할 수 있다. 
이는 LM의 낮은 accuracy를 가지는 word에 대해서 더욱이 명화가이 두드러진다.     
▶️ 따라서, separate fusion layers는 initial prediction으로 부터의 features와 final output을 얻은 LM prediction을 합치는 데 사용된다.     
ABINet의 LM을 뜯어보면, STR에 대해 inefficient하다는 것을 알 수 있다.     
ABINet model의 전체 compute requirements의 상당 부분을 활용했음에도 불구하고 parameter count를 상대적으로 충분히 활용하지 못하고 dismal word accuracy를 금지한다.     

Sequence Model Literature에서, Sequence generation의 일반화된 models에 관심도가 증가하고 있다.     
AR and Refinement-based non-AR과 같은 다양한 Neural Sequence Models은 generalized Framework에서 특별한 케이스로 보여진다.     
이는, 같은 generalization은 STR Models에서 수행되는데 Context-free와 Context-aware STR을 unifying 함으로써 수행된다고 가정한다.   
이 unification의 장점이 분명하지 않음에도 불구하고, 저자들은 그런 generalized model이 internal LM의 사용이 가능하게 한다는 것을 보여준다.     
물론, external LM의 Refinement Capabilities를 유지하면서 말이다.    

Permutation Language Modeling(PLM)은 원래 large scale language pretraining을 위해 제안되었다.     
그러나 최근 연구들은 다른 decoding schemes의 Transformer-based generalized sequence models의 capable을 학습시키는 데 적용하고 있다.     
이 연구에서, 저자들은 PLM을 STR을 위해 적용한다.     
PLM은 AR modeling의 generalzation으로 고려될 수 있다.     
그리고 PLM-trained model은 shared Architecture와 weights를 가지는 AR models의 ensemble로 여겨질 수 있다.     
Token Dependencies를 dynamically하게 specifying 하기 위한 attention masks를 사용한다.    
Figure 2에서 설명하는 바와 같이 model은 input context의 임의의 subset이 주어질 때, Conditional Character Probabilities를 사용하고 학습할 수 있다.     
monotonic AR decoding, Parallel non-AR decoding 그리고 심지어 반복적인 Refinement를 가능하게 한다.      
<p align="center"><img src="https://media.springernature.com/lw685/springer-static/image/chp%3A10.1007%2F978-3-031-19815-1_11/MediaObjects/539995_1_En_11_Fig2_HTML.png" width="60%" height="60%"></p> 

▶️ 요약하자면, SOTA STR methods는 Two-stage ensemble approach를 선택했고 이는 Bidirectional Language Context를 사용하기 위함이다.      
external LMs의 low word accuracy는 더욱 efficient apporach를 위해 필요성이 강조된다.(물론, training과 runtime requirements가 늘어나더라도)      
결국, 저자들은 Permuted AutoRegressive Sequence(PARSeq)를 제안한다.     
PLM을 가지고 학습된 PARSeq는 unified STR Model이고 Simple Structure를 가지고 있다.     
그렇지만, Context-free와 Context-aware Inference 모두가 가능하다.     
**물론, Bidirectional Context를 사용해서 반복적인 Refinement 또한 가능하다.(PARSeq는 SOTA results를 달성한다.)**      


### Context-free STR
Context-free STR methods는 Characters를 Image Features로 부터 직접 예측한다.    
output Characters는 각각에 대해 Conditionally-Independent하다.    
가장 유명한 approaches는 CTC-based methods이고 features를 Character positions로 pooling하거나 STR을 multi-instance Classification problem으로 casting하기 위해 self-Attention과 같이 다른 approach를 사용한다.     
Ensemble methods는 Attention Mechanism을 사용하는데 initial Context-less predictions를 생성하기 위함이다.     
Context-free methods가 오직 prediction을 위한 image features에 의존하기 때문에, occluded나 incomplete Characters와 같은 corruptions에 취약하다.     
이 한계는 recognition model을 더욱 견고하게 만드는 Language Semantics에 관심을 갖도록 했다.    

### Context-aware STR
