# 01. Agent Loop

## 우리가 본 것

Hermes의 루프는 `agent/conversation_loop.py`의 `run_conversation()` 안에 있다.

핵심 조건:

```python
while (api_call_count < agent.max_iterations and agent.iteration_budget.remaining > 0) or agent._budget_grace_call:
```

이 루프는 모델을 한 번만 부르지 않는다. 모델이 도구 호출을 만들면, Hermes가 도구를 실행하고, 그 결과를 다시 `messages`에 넣은 뒤 모델을 다시 부른다.

## Chatbot과 Agent의 차이

Stateless chatbot:

```text
user -> model -> answer -> end
```

Tool-using agent:

```text
user
-> model
-> tool call
-> tool execution
-> tool result enters messages
-> model sees result
-> final answer
```

## 핵심 분기

```python
if assistant_message.tool_calls:
```

모델 응답에 `tool_calls`가 있으면 Hermes는 바로 답변을 종료하지 않는다. 대신 도구 이름과 JSON arguments를 검증하고, 실제 tool handler를 실행한다.

## messages는 작업 메모리다

`messages`는 단순한 채팅 기록이 아니다. 모델이 다음 호출에서 보는 작업 메모리이자 관찰 로그다.

예시:

```json
[
  {
    "role": "user",
    "content": "지난번 이직 고민이랑 연결해서 이번 달 흐름 봐줘"
  },
  {
    "role": "assistant",
    "tool_calls": [
      {
        "id": "call_abc123",
        "type": "function",
        "function": {
          "name": "session_search",
          "arguments": "{\"query\":\"이직 고민 퇴사 지난번\"}"
        }
      }
    ]
  },
  {
    "role": "tool",
    "tool_call_id": "call_abc123",
    "name": "session_search",
    "content": "{\"success\":true,\"results\":[...]}"
  }
]
```

다음 모델 호출은 이 `tool` 메시지를 보고 최종 답변을 만들 수 있다.

## tool_call_id

`tool_call_id`는 모델이 만든 tool call과 Hermes가 돌려주는 tool result를 연결하는 receipt 번호다.

```text
assistant.tool_calls[n].id == tool message.tool_call_id
```

모델이 한 번에 여러 도구를 부를 수 있기 때문에 이 ID가 필요하다.

## max_iterations vs iteration_budget

`max_iterations`는 현재 루프의 상한이다.

```text
이번 turn에서 API call을 최대 몇 번까지 허용할 것인가
```

`iteration_budget`은 consume/refund가 가능한 예산 객체다.

```text
consume(): 호출 1회 사용
refund(): 특정 상황에서 호출 1회 반환
remaining: 남은 호출 수
```

## grace call

`_budget_grace_call`은 예산이 끝났을 때 마지막 정리 호출을 허용하는 예외 플래그다.

용도:

```text
도구 결과는 받았는데 예산이 끝난 경우,
모델이 그 결과를 바탕으로 최종 답변을 한 번 만들 수 있게 한다.
```

`grace`는 유예, 봐줌, 관대한 예외라는 뜻이다.

## 현재 이해

Hermes의 루프는 이 구조다.

```text
Model says something
-> if tool call exists, Tools act
-> tool result is written into State/messages
-> Model observes the result
-> repeat until final answer or budget stop
```

한 문장으로 정리:

> Agent loop는 모델이 바로 답하는 구조가 아니라, 모델이 행동을 요청하고 결과를 관찰한 뒤 다시 판단하는 구조다.

## 복습 체크포인트

다음 장으로 넘어가기 전에 아래 질문에 스스로 답할 수 있어야 한다.

1. `messages`는 왜 그냥 채팅 기록이 아니라 작업 메모리인가?
2. 모델이 tool을 쓸지 말지는 누가 판단하는가? Hermes 코드인가, 모델인가?
3. `tools` schema는 `messages` 안에 들어가는가, 아니면 API 요청의 별도 필드인가?
4. `tool_call_id`는 왜 필요한가?
5. `max_iterations`와 `iteration_budget.remaining`은 무엇이 다른가?
6. `_budget_grace_call`은 언제 필요한 예외인가?
7. 사주 에이전트에서 birth profile은 tool인가, state인가? 왜 그런가?

## 챕터 종료 기준

이 장은 아래 흐름을 코드와 함께 설명할 수 있으면 충분하다.

```text
system/user/tools schema를 모델에 보냄
-> 모델이 assistant.tool_calls를 내려줌
-> Hermes가 도구를 실행함
-> role: "tool" 결과를 messages에 붙임
-> 모델이 결과를 보고 최종 답변하거나 다음 tool_call을 냄
```

다음 장에서는 `tool_calls`가 실제 Python 함수 실행으로 이어지는 경로를 본다.
