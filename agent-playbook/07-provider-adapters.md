# 07. Provider Adapters

이 장에서는 Hermes 내부의 OpenAI-compatible 메시지/도구 형식이 다른 모델 제공자 형식으로 변환되는 방식을 본다.

핵심 질문:

- Hermes 내부 표준 메시지 형태는 무엇인가?
- OpenAI, Anthropic, Gemini, Bedrock, Codex는 tool call 형식이 어떻게 다른가?
- adapter는 어느 시점에 요청/응답을 변환하는가?
- provider가 달라도 agent loop는 왜 거의 같은 모양으로 유지되는가?

