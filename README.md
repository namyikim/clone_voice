# clone_voice

Qwen3-TTS Base 모델로 **보이스 클로닝**을 실습하는 한국어 입문 교재입니다.
참조 음성 몇 초만 있으면 그 목소리로 임의의 문장을 읽게 만들 수 있고,
전체 과정을 Google Colab에서 바로 실행할 수 있도록 노트북 하나로 정리했습니다.

## 바로 실행하기

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/namyikim/clone_voice/blob/main/Qwen_TTS_Manual.ipynb)

위 배지를 누르면 `Qwen_TTS_Manual.ipynb`가 Colab에서 열립니다.
**런타임 → 런타임 유형 변경 → GPU(T4 이상)** 로 설정한 뒤 셀을 위에서부터 순서대로 실행하세요.
CPU로도 동작하지만 생성 속도가 매우 느립니다.

## 노트북 구성

| 단계 | 내용 |
|---|---|
| 1 | Qwen-TTS 개요 — 지원 기능과 10개 지원 언어 |
| 2 | 동작 방식 — 파일 이름 규칙과 전체 흐름 |
| 3 | Google Drive 마운트 |
| 4 | 의존성 설치 (`qwen-tts`, `soundfile`, `ffmpeg`) |
| 5 | Base 모델 로드 + CUDA/dtype 자동 선택 |
| 6 | 참조 음성 준비 — 이름으로 오디오·대본 불러오기 |
| 7 | 음성 생성 — 문장 입력 → 재생·저장 |
| 8 | 자주 막히는 지점 |

## 사전 준비

### 파일 이름 규칙

참조 음성과 그 대본을 **같은 이름**으로 짝지어 두면, 노트북에서 이름 하나만 입력해
둘을 함께 불러옵니다.

| 파일 | 역할 |
|---|---|
| `김남이_애국가1절.m4a` | 참조 음성 → `ref_audio` |
| `김남이_애국가1절.txt` | 그 음성의 대본 → `ref_text` |

목소리를 추가하려면 같은 규칙으로 파일 쌍을 폴더에 넣기만 하면 됩니다.
노트북이 폴더를 훑어 짝이 맞는 이름만 목록으로 보여줍니다.

### Google Drive 폴더

```
MyDrive/2026/clone_voice/
├── recored_voice/                 # 참조 음성 + 대본
│   ├── 김남이_애국가1절.m4a
│   ├── 김남이_애국가1절.txt
│   └── 김남이_애국가1절_16k.wav   # 자동 생성 (변환 캐시)
└── output/                        # 생성 결과 (자동 생성)
```

경로가 다르면 **6.1 셀의 `VOICE_DIR` / `OUTPUT_DIR`만** 수정하면 됩니다.

### 참조 음성 요건

- **길이 3초 이상** (노트북이 자동으로 검사하고 경고합니다)
- m4a / mp3 / wav / webm / ogg / flac 지원
- 내부적으로 **16kHz 모노 wav**로 변환되며, 변환 결과는 캐시돼 재실행 시 건너뜁니다
- 배경 소음이 적고 한 사람만 말하는 구간일수록 결과가 좋습니다

### 대본(.txt) 요건

- 참조 음성에 **실제로 들어 있는 문장을 그대로** 받아써야 합니다. 어긋나면 품질이 크게 떨어집니다
- UTF-8 권장. CP949로 저장된 파일도 자동으로 읽습니다

## 사용 모델

| 항목 | 값 |
|---|---|
| 모델 | `Qwen/Qwen3-TTS-12Hz-0.6B-Base` |
| 용도 | 참조 음성 기반 보이스 클로닝 |
| dtype | GPU면 `bfloat16`, CPU면 `float32` |
| 호출 | `model.generate_voice_clone(text, language, ref_audio, ref_text)` |

Base 모델은 참조 음성을 따라 하는 용도입니다.
미리 준비된 화자로 합성하려면 Base가 아닌 다른 체크포인트를 써야 합니다.

품질을 더 올리고 싶으면 `Qwen/Qwen3-TTS-12Hz-1.7B-Base`로 바꿔보세요.
대신 VRAM 요구량이 늘어나므로 Colab 무료 티어에서는 0.6B가 안전합니다.

## 실행 흐름

```
이름 입력 ──┬──▶ 이름.m4a ──ffmpeg──▶ 16kHz mono wav ──┐
            │                                          ├──▶ generate_voice_clone ──▶ 결과 wav
            └──▶ 이름.txt ─────────── ref_text ────────┤
                                                       │
생성할 문장 입력 ──────────── target_text ─────────────┘
```

참조 음성은 6번에서 **한 번만** 불러오면 되고, 문장만 바꿔 가며 7번 셀을 반복 실행하면
됩니다. 모델을 다시 로드하거나 파일을 다시 변환하지 않습니다.

생성 결과는 `output/이름_날짜시각.wav`로 저장됩니다.

## 주의사항

- **타인의 목소리를 클론할 때는 반드시 본인 동의를 받으세요.** 동의 없는 음성 복제는
  국가에 따라 초상권·음성권 침해나 사기에 해당할 수 있습니다.
- 이 저장소는 public입니다. 참조 음성이나 생성 결과 wav를 커밋하면 누구나 내려받을 수
  있으니, 음성 파일은 Drive에만 두고 저장소에는 올리지 않는 것을 권합니다.
- 노트북 출력에도 오디오가 base64로 포함됩니다. 아래 nbstripout 설정을 권합니다.

## 기여자를 위한 설정

노트북 출력이 커밋에 섞여 들어가지 않도록 `nbstripout`을 걸어두세요.

```bash
pip install nbstripout
nbstripout --install
```

`--install`이 저장소의 git filter와 `.gitattributes`를 설정해 주기 때문에,
이후에는 평소처럼 `git add` / `git commit`만 하면 출력이 자동으로 제거된 상태로 커밋됩니다.
작업 중인 노트북 파일 자체는 건드리지 않습니다.

## 라이선스

노트북 내용은 학습용 자료입니다.
Qwen3-TTS 모델과 `qwen-tts` 패키지는 Apache-2.0 라이선스이며,
자세한 내용은 [QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)를 참고하세요.
