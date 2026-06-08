# 03. Tool Registry and Toolsets

이 장에서는 Hermes가 도구를 이름, 설명, schema, handler로 등록하고 모델에게 노출하는 방식을 본다.

핵심 질문:

- `registry.register()`는 무엇을 저장하는가?
- tool schema는 모델에게 어떤 정보로 전달되는가?
- toolset은 왜 필요한가?
- `check_fn`은 언제 도구를 숨기거나 노출하는가?

