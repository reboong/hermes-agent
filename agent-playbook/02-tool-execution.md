# 02. Tool Execution

이 장에서는 모델이 만든 `tool_calls`가 실제 Python 함수 실행으로 이어지는 경로를 본다.

핵심 질문:

- Hermes는 `assistant.tool_calls`를 어디서 받는가?
- tool 이름과 arguments는 어디서 파싱되는가?
- 실제 tool 함수는 어디서 찾아 실행되는가?
- 실행 결과는 어떤 형식으로 다시 `messages`에 들어가는가?

