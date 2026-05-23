# LLM

## 목차
- token
- context window
- pretraining
- instruction tuning
- inference
- hallucination

## 기초 개념
LLM은 대량의 텍스트를 기반으로 다음 token을 예측하도록 학습된 언어 모델이다. 질의응답, 요약, 코드 작성, 추론 보조 등에 쓰인다.

```text
prompt tokens -> model -> output tokens
```

## 간단한 예시
```text
User prompt: "HTTP와 HTTPS 차이를 설명해줘"
Model output: 설명 텍스트 생성
```

## 반드시 알아야 할 질문
- token은 단어와 무엇이 다른가?
- context window는 어떤 제약을 만드는가?
- hallucination은 왜 발생하는가?
- temperature는 출력에 어떤 영향을 주는가?
