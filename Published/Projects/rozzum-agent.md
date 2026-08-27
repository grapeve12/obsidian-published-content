---
title: My AI Agent
date: 2026-08-27

tags: [RVC, PyTorch, ONNX, OpenAIAPI, SSH]
description: "RVC 음성 변환 기술의 핵심 모델 포맷과 최적화 방법론을 다루며, OpenAI 호환 API 연동 및 원격 서버 접속을 위한 SSH 환경 설정까지 포괄적으로 설명합니다."

category: project
thumbnail: ""
draft: false
---

# Concepts

## RVC (Retrieval-based Voice Conversion)

검색 기반 음성 변환이다.

AI 음성 합성 기술으로 기존의 Diff-SVC와 비슷한 형태이지만 Diff-SVC는 Stable Diffusion을 이용해 음파 이미지를 만드는 방식이고 RVC는 기존의 음성데이터를 이용해 변조를 하는 방식이다.
**음성 변조**와 비슷하다고 생각하면 될 듯하다.

내 목소리를 다른 사람의 목소리로 실시간으로 바꿔주는 딥러닝 AI 기술이다.

`.pth` (PyTorch 파이토치 모델)
- 캐릭터의 성대 지문이 통째로 담긴 50~100MB짜리 가장 핵심이 되는 본체 모델 파일.
- 하지만 쌩으로 돌리려면 파이썬 환경이 떡칠된 무거운 프로그램이 필요하기 때문에, Echo 같은 쾌적한 네이티브 앱에 넣으려면 `.onnx` 포맷으로 한 번 변환해 줘야 한다.

`.index` (FAISS 인덱스 파일)
- 본체 모델의 연기력을 200% 부스트 시켜주는 옵션 버프 파일이다.
- Echo 앱에서 실시간으로 마이크를 쏠 때는 이 파일이 굳이 필요 없지만, Audacity 믹싱같은 후반 작업을 할 땐 퀄리티를 확 끌어 올려준다.

`.onnx` (ONNX 최적화 포맷)
- `.pth` 모델 파일의 최적화 버전이다.
- 무겁고 꼬이기 쉬운 파이썬 의존성을 싹 다 날려버려서, 이 포맷을 쓰면 앱 부팅이 2초 만에 끝나고 연산 속도가 빨라진다.

> TTS는 텍스트 → 음성
> RVC는 음성 → 음성

## OpenAI Competible API

실제 OpenAI 서버가 아니어도 OpenAI의 요청 및 응답 규격(`/v1/chat/completions` 등)을 그대로 따르는 인터페이스이다.
코드 수정 없이 OpenAI SDK나 도구를 다른 AI 모델이나 로컬 서버에 그대로 연동할 수 있다.

## SSH & SSHD

**SSH**

내 컴퓨터에서 다른 원격 서버로 "나갈 때" 쓰이는 도구이다.
즉, **네트워크를 통해 다른 컴퓨터의 터미널(Shell)을 안전하게 사용하는 프로토콜**이라고 보면 된다.

기능으로는 다음과 같은 것이 있다.
- SCP/SFCP: 파일 전송 프로토콜(FTP)의 대체이다.
- 호스트와 클라이언트 간의 터널링 구현
- X11 Display 포워딩

**SSHD**

원격 컴퓨터에서 다른 이의 접속 요청을 "받을 때" 쓰이는 백그라운드 프로그램이다.
원자는 SSH Daumon인데, 리눅스에서 백그라운드에서 서비스를 계속 실행하는 프로그램을 흔히 Daemon이라고 부른다.

```
# SSH 서버 시작
> service ssh start
```


> IP -> 어떤 네트워크 목적지
> Port -> 네트워크 목적지의 어떤 서비스

IP와 Port는 **어디로 연결할지**를 결정하는 것이고,
**누가 어디까지 접근할 수 있는지**는 방화벽, 인증, 권한, 네트워크 격리, OS/컨테이너/VM 보안 등이 결정한다.

# Phase

## Phase 0

먼저 기본 뼈대를 만드는 과정이다.

```
src/
├── pyproject.toml   # uv 프로젝트 (ollama, edge-tts 의존성)
├── main.py          # 대화 루프 (입력 → chat() → 화면출력 + speak())
├── llm.py           # Ollama qwen3.5:4b 호출, (한국어, 영어) 튜플 반환
├── tts.py           # edge-tts로 영어 텍스트 → mp3
├── .gitignore       # .venv, output/, .env 제외
└── README.md        # 실행법 3줄
```

한국어 대화 → 영어 번역 → TTS 파이프라인을 구축할 것이고 RVC는 Phase 1에서 연결 할 예정이다

사용할 로컬 LLM은 Qwne3.5 4b이다.
어쩔 수 없다.. 나는 하드웨어 거지다..
물론 OpenAI API를 사용할 수도 있지만 최종 목적은 나만의 LLM을 만드는 것이기도 하고,
API는 후에도 손쉽게 붙일 수 있기 때문에 해당 Phase에서는 파이프라인 구축을 최우선 과제로 삼았다.

돌려본 결과 너무 오래 걸려서 일단 vLLM으로 Qwen만 RunPod에 올려보기로 했다.

