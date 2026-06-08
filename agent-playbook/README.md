# 헤르메스로 배우는 에이전트 구조

Hermes Agent codebase를 통해 에이전트 구조를 배우기 위한 playbook.

목표는 Hermes 전체를 한 번에 이해하는 것이 아니라, 작은 실행 경로를 따라가며 에이전트의 상위 구조를 익히는 것이다.

## 2026-06-08

에이전트 구조를 볼 때의 기본식:

```text
Agent = Model + State + Tools + Loop + Policy
```

- Model: 말하고 판단하는 뇌
- State: 사용자, 세션, 기억, 현재 상황
- Tools: 직접 할 수 있는 행동들
- Loop: 판단 -> 행동 -> 관찰 -> 재판단의 반복
- Policy: 하면 안 되는 것, 조심해야 하는 것, 톤, 안전 규칙

## 에이전트성의 단계

1. Stateless chatbot
   - 매번 새로 답함
2. Contextual assistant
   - 최근 대화와 프로필을 보고 답함
3. Memory assistant
   - 장기 기억을 검색하고 반영함
4. Tool-using agent
   - 필요한 계산, 검색, 수정 도구를 스스로 고름
5. Autonomous agent
   - 사용자가 안 불러도 목표를 추적하고 먼저 행동함

## 루프

챗봇은 보통 한 번 답하고 끝난다.

```text
사용자 입력 -> 프롬프트 구성 -> LLM 답변 -> 종료
```

에이전트는 상태를 보고, 행동하고, 결과를 관찰하고, 다시 판단한다.

```text
think -> act -> observe -> think -> act -> observe -> answer
```

루프가 생기면 LLM은 단순 답변 생성기가 아니라 상태를 다루는 행위자가 된다.

## 헤르메스에서 볼 것

헤르메스는 범용 에이전트 런타임을 공부하기 좋은 교재다. 전체를 따라 하기보다 각 코드가 아래 부품 중 어디에 속하는지 보면서 읽는다.

- OpenAI API 호출: Model
- 세션 DB, 메시지, 장기 기록: State
- 파일, 터미널, 웹, 검색 도구: Tools
- 모델 호출, tool call 처리, 반복 제한: Loop
- 도구 권한, 설정, 비활성 toolset, guardrail: Policy

읽을 때의 질문:

- 이 코드는 Model, State, Tools, Loop, Policy 중 어디에 속하는가?
- 모델이 답변을 바로 쓰는가, 아니면 도구를 고르고 결과를 다시 보는가?
- 상태는 어디에 저장되고 다음 턴에서 어떻게 복원되는가?
- 도구는 하드코딩되어 있는가, 이름/설명/스키마/실행 함수로 등록되는가?
- 루프가 멈추는 조건은 무엇인가?

## 목차

1. [Agent Loop](./01-agent-loop.md)
   - `while` 루프
   - `tool_calls`
   - `tool_call_id`
   - `messages` as working memory
   - `max_iterations`, `iteration_budget`, grace call
2. [Tool Execution](./02-tool-execution.md)
   - 모델이 낸 `tool_calls`가 실제 Python 함수 실행으로 이어지는 경로를 본다.
   - tool 결과가 어떤 형식으로 다시 `messages`에 들어가고, 다음 모델 호출의 관찰값이 되는지 배운다.
   - `_execute_tool_calls`
   - `handle_function_call`
   - `registry.dispatch`
   - tool result message
3. [Tool Registry and Toolsets](./03-tool-registry-and-toolsets.md)
   - 도구가 이름, 설명, 입력 schema, 실행 handler로 등록되는 구조를 본다.
   - 모델에게 노출되는 도구 목록이 toolset과 availability check로 어떻게 제어되는지 배운다.
   - `registry.register`
   - tool schema
   - enabled/disabled toolsets
   - `check_fn`
4. [State and Session DB](./04-state-and-session-db.md)
   - 에이전트의 대화와 실행 기록이 세션으로 저장되고 복원되는 구조를 본다.
   - 단기 작업 메모리인 `messages`와 장기적으로 검색 가능한 session DB의 차이를 배운다.
   - sessions
   - messages
   - FTS search
   - resume
5. [Memory Layer](./05-memory-layer.md)
   - 사용자의 안정적인 선호, 환경, 반복되는 맥락을 장기 기억으로 다루는 방식을 본다.
   - session search와 durable memory가 각각 어떤 문제를 해결하는지 구분한다.
   - durable memory
   - session search
   - memory provider abstraction
6. [Policy and Guardrails](./06-policy-and-guardrails.md)
   - 에이전트가 할 수 있는 행동과 하면 안 되는 행동을 어디서 제한하는지 본다.
   - prompt-level 규칙과 code-level guardrail, tool permission의 역할 차이를 배운다.
   - tool permissions
   - destructive action checks
   - platform constraints
7. [Provider Adapters](./07-provider-adapters.md)
   - Hermes 내부의 OpenAI-compatible 메시지/도구 형식이 다른 모델 제공자 형식으로 변환되는 과정을 본다.
   - provider가 달라도 agent loop를 같은 구조로 유지하는 adapter의 역할을 배운다.
   - OpenAI-compatible internal shape
   - Anthropic/Gemini/Bedrock/Codex conversion

## 실습 관찰

`~/repo/reboong/hermes-agent`에서 `Write tests for @filename` 같은 요청을 실행해보면 tool-using agent의 루프를 보기 좋다.

관찰할 것:

- 요청을 어떻게 작업 목표로 해석하는지
- 어떤 파일을 먼저 읽는지
- 기존 테스트 구조를 어떻게 찾는지
- 어떤 도구로 파일을 수정하는지
- 테스트 실행 결과를 보고 다시 수정하는지
- 마지막에 어떤 상태와 검증 결과를 보고하는지
