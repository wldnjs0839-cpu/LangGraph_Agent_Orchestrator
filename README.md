# LangGraph_Agent_Orchestrator

> **LangGraph**의 `StateGraph`를 활용해 세 개의 AI 에이전트가 협력하여  
> 사용자의 자연어 질문으로부터 맞춤형 운동을 **개조식**으로 추천하는 멀티 에이전트 파이프라인입니다.

---

## 멀티 에이전트 소개

이 시스템은 하나의 LLM이 모든 작업을 처리하는 대신,  
**역할이 분리된 3개의 전문 에이전트**가 순차적으로 협력하는 구조입니다.

각 에이전트는 독립적인 프롬프트와 역할을 가지며,  
`AgentState`라는 **공유 상태 객체**를 통해 결과를 다음 에이전트에게 전달합니다.

| 에이전트 | 노드명 | 역할 | 입력 → 출력 |
|----------|--------|------|-------------|
| 증상 추출 에이전트 | `extractor` | 사용자의 질문에서 신체적 문제(증상) 키워드를 추출 | `query` → `symptoms` |
| 운동 추천 에이전트 | `recommender` | 추출된 증상을 바탕으로 적합한 운동 5가지를 선별 | `symptoms` → `exercises` |
| 답변 생성 에이전트 | `answer` | 증상과 운동 목록을 받아 개조식 최종 답변을 작성 | `symptoms` + `exercises` → `result` |

### 공유 상태 (AgentState)

에이전트 간 데이터를 주고받는 **공유 그릇** 역할을 합니다.  
각 에이전트는 자신의 결과를 State에 저장하고, 다음 에이전트가 꺼내 씁니다.

```python
class AgentState(TypedDict):
    query: str       # 사용자 원본 질문
    symptoms: str    # 추출된 증상 목록  (extractor가 채움)
    exercises: str   # 추천 운동 리스트  (recommender가 채움)
    result: str      # 최종 개조식 답변  (answer가 채움)
```

### 실행 흐름 요약

```
사용자 질문
    ↓
[extractor]    →  증상 키워드 추출  (예: 체력 저하, 체중 증가)
    ↓
[recommender]  →  운동 5가지 선별  (예: 걷기, 수영, 요가 ...)
    ↓
[answer]       →  개조식 최종 답변 생성
    ↓
출력
```

---

## 기술 스택

| 기술 | 설명 |
|------|------|
| ![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white) | 전체 파이프라인 구현 언어 |
| ![LangGraph](https://img.shields.io/badge/LangGraph-StateGraph-2ea44f) | 에이전트 간 실행 흐름(그래프)을 정의·관리하는 프레임워크. `StateGraph`, `add_node`, `add_edge`로 순서 제어 |
| ![LangChain](https://img.shields.io/badge/LangChain-LCEL-1C3C5E) | `PromptTemplate \| LLM` 형태의 **LCEL 체인**으로 각 에이전트의 프롬프트와 LLM을 연결 |
| ![Ollama](https://img.shields.io/badge/Ollama-Local%20LLM-black) | LLM을 **로컬 환경에서 무료 실행**하는 서버. API 키 불필요 |
| ![EXAONE](https://img.shields.io/badge/EXAONE-3.5:2.4b-9c27b0) | LG AI Research의 **한국어 특화 경량 모델** (2.4B). 한국어 질의에 최적화 |

---

## 그래프 구조

LangGraph는 에이전트 실행 흐름을 **방향성 그래프(DAG)** 로 표현합니다.  
각 노드(Node)는 에이전트 함수이며, 엣지(Edge)는 실행 순서를 나타냅니다.

<img width="145" height="432" alt="output" src="https://github.com/user-attachments/assets/ed7bf518-c9e2-4c14-b6af-133bb5b9c514" />

- **`__start__`** : 그래프 진입점 — 사용자 질문 입력
- **`extractor`** : 증상 추출 노드
- **`recommender`** : 운동 추천 노드
- **`answer`** : 최종 답변 생성 노드
- **`__end__`** : 그래프 종료

> 화살표 옆 `symptoms →` `exercises →` `result →` 는 각 단계에서 State에 추가되는 필드입니다.

---

## 실행 결과

질문 `"체력이 안 좋고, 살이 계속 찌는데 어떤 운동을 할까?"` 를 입력했을 때의 출력 결과입니다.

<img width="960" height="545" alt="스크린샷 2026-05-13 오후 3 56 38" src="https://github.com/user-attachments/assets/0d254678-f0f5-41fe-8d51-6ca7ddeb6680" />

- **[1] 추출된 증상** : extractor 에이전트가 질문에서 핵심 증상을 분리
- **[2] 추천 운동 목록** : recommender 에이전트가 증상에 맞는 운동 선별
- **[3] 최종 답변** : answer 에이전트가 운동별 추천 이유·방법을 구조화 출력

---

## 실행 방법

### 1. 패키지 설치

```bash
pip install langchain langgraph langchain-ollama
```

### 2. Ollama 설치 및 모델 준비

```bash
# Ollama 설치
curl -fsSL https://ollama.com/install.sh | sh

# 한국어 특화 경량 모델 다운로드
ollama pull exaone3.5:2.4b
```

### 3. 노트북 실행

```bash
jupyter notebook LangGraph_Agent_Orchestrator.ipynb
```

---

## 파일 구조

```
.
├── langgraph_exercise.ipynb   # 메인 실습 노트북
├── graph_structure.png        # LangGraph 그래프 구조 이미지
├── result_output.png          # 실행 결과 캡처 이미지
└── README.md                  # 프로젝트 설명
```
