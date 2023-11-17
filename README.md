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
