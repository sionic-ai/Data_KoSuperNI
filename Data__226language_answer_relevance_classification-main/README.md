# Data__language_answer_relevance_classification

## 데이터 개관       

| 데이터 이름 | 내용 | 범주 | 도메인 | 입력 언어 | 출력 언어 |
| --- | --- | --- | --- | --- | --- |
| task226_english_language_answer_relevance_classification | 주어진 질의응답 쌍에서 어느 응답이 더 수용가능한지 평가 | [수용성 테스트] Answerability Classification | 코드 -> Repo -> Stack Overflow, Linguistics | 영어 | 영어 |


https://github.com/allenai/natural-instructions 에 수록된 데이터 중에 하나   

- 총 데이터 개수 478개     
- 영어 질의에 대한 응답이 참인지 거짓인지를 크라우드소싱 방법으로 확인, 따라서 입력은 질의 응답이고 출력은 정답이 맞으면 yes, 틀리면 no로 출력되도록 함      


## 실험 세팅과 결과 

목적 1 :    크라우드 소싱 방법으로 사람이 구축한 결과와 GPT-4 에게 정답이 맞으면 yes, 틀리면 no로 출력하도록 했을 때의 결과가 일치하는지 확인      
목적 2 :   영어 질의 응답을 한국어로 번역했을 때 번역된 한국어 데이터에 대해서도 동일한 결과가 나오지를 자동으로 평가할 수 있는지 확인       

- 결론적으로 여러번 실험을 반복해도 60% 정도의 정확성을 보임   
- 따라서 특정 언어(여기서는 영어)의 뉘앙스를 물어보는 난이도 있는 질의에서 GPT4의 추론 능력이 사람을 대신할 수 있는 정도라고 보기는 어려움    
- 질문을 한국어로 번역했을 때도 비슷한 결과를 보이기 때문에 번역어를 사용해서 질의응답 데이터를 만드는 것은 크게 문제가 될 것이 없어 보임       

### 프로세스 1:     
- 영어 입력(질의 응답)을 GPT4에게 평가하도록 하여  맞으면 yes, 틀리면 no로 출력하도록 함   
- 사람이 평가한 것과 GPT4가 평가한 것을 비교

| 행 레이블 | 개수 : 영어 원문 지피티 응답의 동일(=True)성 비교 |   |
| --- | --- |--- |
| 원문 영어를 입력한 경우 |일치 개수  | 퍼센트 |
| 사람의 판단과 GPT 판단이 불일치한 경우 | 186 | 38.92 |
| 사람의 판단과 GPT 판단이 일치한 경우 | 292 | 61.08 |
| 총합계 | 478 | 100 |

-  결과와 얼마나 일치하는지 확인하기 위해 위의 과정을 3번 동일하게 실험을 반복하여 작업 일치도를 측정

Inter-annotator Agreement 측정 : Fleiss' Kappa = 0.826 

  일치도 평가를 위한 코드 참조 : https://github.com/Shamya/FleissKappa    
  Landis 외(1977)에서는 Fleiss’ Kappa = 0.8 이상이면 일치도가 매우 높은 편으로 판단    


### 프로세스 2:     
- 한국어 번역에서도 동일한 결과를 보이는지 확인하기 위해 위의 데이터를 한국어로 번역하여 동일한 과정 반복

- 사람이 평가한 것과 GPT4가 평가한 것을 비교

| 행 레이블 | 개수 : 번역문 지피티 응답의 동일(=True)성 비교 |     |
| --- | --- | --- |
| 사람의 판단과 GPT 판단이 불일치한 경우 | 194 | 40.59|
| 사람의 판단과 GPT 판단이 일치한 경우 | 284 |59.41 |
| 총합계 | 478 | 100 |
    
- 번역문과 원문의 정확도에는 큰 차이가 없는 것으로 판단됨    
- Inter-annotator Agreement 측정 : Fleiss' Kappa = 0.828            


# 프롬프트 예시       

```   
#!pip install openai==0.28
import openai
import pandas as pd
# OpenAI API 설정 및 입출력 정의  필요   

def get_judgment(sentence_a, sentence_b):
    prompt = f"""A에 대한 답으로 B가 맞다고 생각하면 Yes 아니면 No 라고 말해주세요. 예시는 다음과 같습니다.

A: "언제 시간 되?와 언제 시간 돼? 중에 맞는 표현은 무엇인가요?" / B: "'되어'를 넣고 말이 되면 '돼'를, 말이 되지 않으면 '되'를 씁니다. 따라서 시간 언제 돼가 맞는 표현입니다."
판단: Yes. '되어'를 넣고 말이 되면 '돼'를, 말이 되지 않으면 '되'를 씁니다. 따라서 시간 언제 돼가 맞는 표현이라는 올바른 정보를 전달하고 있습니다.

A: "I disliked the implied criticism in his voice.와 I disliked the inferred criticism in his voice. 중에 맞는 표현은 무엇인가요?/ B: ""Implied"는 말하거나 표현하는 사람이 직접적으로 말하지 않고 간접적으로 나타내는 것을 의미합니다. 즉, 이 경우엔 그 사람의 목소리에서 간접적으로 비판의 의미가 느껴지는 것을 말합니다.
"Inferred"는 듣는 사람이 말이나 상황에서 스스로 추론하거나 해석한 것을 의미합니다. 따라서, 비판이 그 사람의 목소리에 내포되어 있다고 표현하고자 한다면 "implied criticism"이 적합합니다."
판단: Yes. "Your"는 소유격 대명사로, 무언가가 누군가에게 속해 있음을 나타냅니다. 반면에 "you're"는 "you are"의 축약형으로, "당신은"이라는 의미를 가집니다.

A: "오늘로서 취업에 성공한지 일년이 지났다와 오늘로써 취업에 성공한지 일년이 지났다 중 맞는 표현은 무엇인가요?" / B: "로서"는 시간이 지났을 때 사용하는 조사이므로 오늘로써 취업에 성공한지 일년이 지났다가 맞는 표현입니다."
판단: No. '-로써'가 시간이 지났을 때 사용하는 조사이므로 "오늘로서 취업에 성공한지 일년이 지났다"가 맞는 표현입니다.

A: "Everyday와 Every day는 다른 의미로 사용하나요?" / B: Everyday 대신 Every day는 큰 의미 차이 없이 바꿔 쓸 수 있습니다."
판단: No. 'Everyday는 “일상의” 또는 “일반적인”이라는 뜻으로 사용하고 Every day은 매일이라는 뜻으로 사용하기 때문에 의미 차이가 없다고 한 설명은 맞지 않습니다.

A: "{sentence_a}" / B: "{sentence_b}"
판단:"""

    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": "당신은 언어 전문가입니다."},
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message['content'].strip()
    except Exception as e:
        print(f"An error occurred: {e}")
        return "Error"

```   

# 데이터 예시    
        "input_question": "What does \"I'm game\" mean and what is its correct usage?",
        "input_answer": "\"I'm game\" could mean \"I'm very smelly.\".",
        "output": "no",
        "GPT4_output": "No",
        "GPT4_explanation": "\"I'm game\" 은 \"나도 참여하겠다\" 또는 \"나는 준비가 되었다\"라는 의미로 사용되며, 이것이 \"나는 매우 냄새난다\"라는 의미는 아닙니다",
        "input_question_Ko": "\"I`m game\"이 무슨 뜻이고 올바른 사용법은 무엇인가요?",
        "input_answer_Ko": "\"I`m game\"은 \"I`m very smelly.(나는 매우 냄새가 난다)\"라는 뜻일 수 있습니다. 이는 일반적으로 쓰이는 의미가 아니며, 상황에 따라 다른 의미를 가질 수 있습니다.",
        "output__Ko_GPT4": "No",
        "Ko_GPT4_explanation": "\"I'm game\"은 \"나는 마음이 내킨다\" 또는 \"나는 분명히 참여할 것이다\"라는 의미로 사용되며, \"나는 매우 냄새가 난다\"라는 의미로는 전혀 사용되지 않습니다"

# 실험에 사용한 원천 데이터와 번역 데이터 및 GPT-4의 응답 결과 공개        
https://github.com/sionic-ai/Data__language_answer_relevance_classification/blob/main/EnglishVsKorean_language_answer_relevance_classification.csv     
https://github.com/sionic-ai/Data__language_answer_relevance_classification/blob/main/EnglishVsKorean_language_answer_relevance_classification.json         

# Todo                     
지피티의 추론 정확성을 높이기 위한 튜닝 방법 개선


# 참고       
Landis, J. Richard, Gary G. Koch(1977), “The Measurement of Observer Agreement for Categorical Data”, Biometrics 33(1), 159.


**한국어 번역 데이터는 NAVER CLOUD PLATFORM의 Benefits를 사용하여 번역되었음**   
(https://guide.ncloud-docs.com/docs/clovastudio-copyright)

