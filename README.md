# KB-ALBERT
### ALBERT for Finance Domain from KB국민은행
</br>

# Background

### 1. 자연어 처리 트랜드

- 최근 비정형 텍스트를 처리하는 Task들은 대부분 딥러닝 알고리즘과 대용량 데이터를 이용하여 미리 학습한 언어 모델(PLM : Pre-trained Language Model)을 활용하는 형태로 발전 하였음.
    - 사람을 비교해서 생각해도 기본 언어 능력이 더 뛰어난 사람(책을 많이 읽었던 사람)이 더 쉽게 새로운 언어 이해 관련 Task를 배우고 풀 수 있다는 접근 방법.

    ![](https://yhdosu.github.io/assets/images/plm1.png)
    
- 하지만, 일반 USER가 이러한 언어 모델을 학습 하기에는 몇 가지 제약이 있음.
    - 대용량의 학습 데이터 수집 및 정제가 필요(빅데이터)
    - 학습 알고리즘 구현 필요(알고리즘)
    - 학습을 진행할 GPU 머신 등의 리소스 필요(하드웨어)
- 이러한 이유로 2018년 부터 해외에서는 해당 연구를 선도하는 기업이나 단체에서 PLM을 학습한 후 비영리 목적으로 공개하는 케이스가 많아짐.
    - Google : BERT 시리즈 / Facebook : RoBERTa / OpenAI : GPT 시리즈 등
- 최근에는 HuggingFace 등 각자 최적화 된 형태로 구현한 코드 및 모델을 공개하고 공유하는 형태로 발전


### 2. 한국어를 위한 PLM

- 2019 부터 한국에서도 한국어에 특화된 PLM을 비영리 목적으로 공개하는 케이스가 많이 등장함
    - Etri : KoBERT([http://aiopen.etri.re.kr/service_dataset.php](http://aiopen.etri.re.kr/service_dataset.php))
    - SK T-Brain : SKT-KoBERT, SKT-Ko-GPT... ([https://github.com/SKTBrain](https://github.com/SKTBrain))

### 3. 금융 도메인에 특화된 모델

- 일반적인 자연언어 처리 분야는 처리하고자 하는 대상의 도메인에 많은 디펜던시를 가짐 
(의료 분야, 금융 분야 등)
    - 일반적인 도메인의 언어 능력이 뛰어난 사람도 금융 용어나 의료 용어를 잘 모르기 때문에 해당 관련 Task를 쉽게 배우고 풀 수 없는 것과 같은 이치
    
    ![](https://yhdosu.github.io/assets/images/plm2.png)
    
- 이러한 이유로, KB 금융 그룹에서는 금융 도메인에 특화된 PLM을 학습 / 제공 하고자 함
</br>


# 모델 특징

### 1. 금융 도메인에 특화 되었지만, 일반 도메인에 대해서도 잘 동작

- 금융 도메인과 관련된 문서를 추가적으로 많이 학습 하였고, 해당 도메인으로만 치우치지 않도록 일반 도메인의 학습 데이터도 많은 양을 사용.
- 금융 도메인의 전문 용어들이 vocab에서 버려지지 않도록 vocab size 를 보통 일반적으로 사용하는 32,000개 정도 보다 큰 50,000개 정도로 사용


### 2. "WordpieceModel" 이전 단계에 어근/어미 분리를 진행하도록 추가

- 한글에 "WordpieceModel"을 바로 적용하는 것 보다 형태소 분석 후에 적용하는 것이 일반적으로 성능이 더 좋음
- 하지만 보통 형태소 분석기의 경우 50여개의 형태소 태그를 분류하기 위한 연산의 오버로드와 형태소의 원형을 복원하기 위한 오버로드가 상당한데, 실제로 BPE의 입력에는 해당 부분이 사용 되지 않음.
- 그래서 형태소 분석 말뭉치를 변형하여 간단히 어근과 어미만 분리(실제로는 명사와 동사 어근)하기 위한 말뭉치를 생성 해냈으며, 해당 말뭉치와 CRF를 이용하여 간단한 전처리기(어근 분리기)를 구현하여 WordpieceModel 이전에 적용 함
⇒ 세종 말뭉치에 대해서 99.1%의 정확도 보임.

</br>

# 모델 상세 정보

### 1. Architecture

- max_seq_length=512
- embedding_size=128
- hidden_size=768
- num_hidden_layers = 12
- vocab_size = 50,000

### 2. 학습 데이터 셋

- 일반 도메인 텍스트(위키 + 뉴스 등) : 약 25GB 
- 금융 도메인 텍스트(경제/금융 특화 뉴스 + 리포트 등) : 약 15GB

</br>

# SUB-TASK

|도메인|테스크(데이터셋)|Bert base multi-lingual|KB-ALBERT|
|---|---|---|---|
|일반|감성분류(Naver)|0.888|0.91|
|일반|MRC(KorQuAD 1.0)|0.87|0.90|
|금융|MRC(자체)|0.77|0.89|

</br>

# How-to-Use

### 1. Model Download

- KB-ALBERT를 사용하시고자 하시는 분들은 아래 메일로 소속, 이름, 사용용도를 간단히 작성하셔서 발송해 주세요.
- ai.kbg@kbfg.com


### 2. Source Download and Install

- git clone
- cd KB-AlBert
- pip install transformers


### 3. Using with PyTorch and Transformer(from Hugging Face) 

- Example : Sentence to Vector

```python
import torch
from noun_splitter import NounSplitter
from transformers import *
from transformers import AlBertTokenizer, AlBertModel

# Load noun-splitter 
noun_splitter = NounSplitter("np2.crfsuite")

# Load pre-trained model tokenizer (vocabulary)
tokenizer = AlBertTokenizer.from_pretrained('KB-AlBERT')

# Load pre-trained model
kb_albert = AlBertModel.from_pretrained('KB-AlBERT')

# Tokenize inputs
text = "나는 국민은행에서 오픈한 알버트를 쓴다."
tokenized_text = noun_splitter.do_split(text)
input_ids = tokenizer.encode(tokenized_text)

# Convert inputs to PyTorch tensors
input_tensors = torch.tensor([input_ids])

# Predict hidden states features for each layer
with torch.no_grad():
    outputs = kb_albert(input_tensors)
    last_layer = outputs[0]
```

</br>

# E.T.C Info

### 1. Version History

- v.0.1 : 초기 모델 릴리즈


### 2. License

- KB-ALBERT관련 코드는 Apache-2.0 라이선스 하에 공개되어 있습니다. 코드를 사용할 경우 라이선스 내용을 준수해주세요. 라이선스 전문은 LICENSE 파일에서 확인하실 수 있습니다.
- 해당 모델에 대한 라이센스 범위는 모델을 전달받은 분들에 대해 개별적으로 전달드립니다.
