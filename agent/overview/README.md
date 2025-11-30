---
icon: store
---

# Overview

// 기본적인 사용법

1. AIAgent Profile (ScriptableObject) 생성
2. Unity 씬에 AIAgent 컴포넌트 추가
3. AIAgent 컴포넌트에 Profile 할당

// AIAgent의 종류
AIAgent는 어떤 Chat API를 사용할지에 따라 종류가 나뉜다는 설명.

1. Chatbot (ChatCompletions API)

- 기본 integration. 모든 Provider사용가능.
- 모든 Provider의 API를 섞어서 사용가능 (예: Microsoft STT > OpenAI GPT > ElevenLabs TTS)
- 툴은 Function만 사용가능

2. Assistant Agent (Assistants API)

- 현재 지원되는 Provider는 OpenAI 뿐.
- 툴은 Function, FileSearch, CodeInterpreter 사용가능하나 FileSearch, CodeInterpreter의 세밀한 설정은 불가능.
- 다른 Provider와 섞어서 사용 불가능.

3. Voice Agent (Realtime API)

- 실시간 음성 대화를 위한 특수한 Agent.
- Agent의 성격에 따라 Voice 입력만 가능.
- 현재 지원되는 Provider는 OpenAI 뿐.
- WebSocket을 통한 실시간 음성 스트리밍.
- 툴은 Function, MCP 사용가능.

4. Response Agent (Responses API) / most advanced

- 현재 지원되는 Provider는 OpenAI, xAI (Grok) 뿐.
- 모든 툴과 세밀 옵션 사용가능.
- 다른 Provider와 섞어서 사용 불가능.
