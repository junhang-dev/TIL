# LangGraph Core Concepts & Architecture

> **핵심 철학:** "Agents are Graphs."
> LangGraph는 단순한 파이프라인(DAG)이 아니라, 반복(Cycle)과 상태(State)를 관리하는 순환 그래프입니다.

## 1. 핵심 구성 요소 (Components)

### 🏗️ State (상태)
- **정의:** 그래프의 모든 노드가 공유하는 데이터 저장소(Shared Data Schema).
- **특징:**
  - 그래프를 통해 데이터가 흐르는 것이 아니라, **State가 유지되고 노드들이 이를 조회/수정**하는 방식.
  - **구조 분리:** 목적에 따라 `Input` (입력), `Output` (최종 반환), `Private` (내부 처리용)으로 스키마 설계 가능.

### ⚙️ Node (노드)
- **정의:** 실제로 작업을 수행하는 단위 (LLM 호출, 함수 실행, 도구 사용 등).
- **동작 방식:**
  - 현재 **State**를 입력으로 받음.
  - 작업 수행 후, 변경하고 싶은 State의 일부(Partial Dict)만 반환.
  - **주의:** 반환값은 다음 노드로 직접 전달되는 것이 아니라, **State를 업데이트**하는 데 사용됨.

### 🔗 Edge (엣지)
- **정의:** 노드와 노드 사이의 연결 선.
- **Normal Edge:** 항상 다음 지정된 노드로 이동.
- **Conditional Edge:** `decide_path`와 같은 라우팅 함수 로직에 따라 동적으로 다음 경로 결정.

## 2. State 관리 메커니즘 (State Management)

### Reducer (리듀서)
State가 업데이트될 때의 동작 방식을 정의합니다.
- **Default (덮어쓰기):** 기본적으로 새로운 값이 들어오면 기존 값을 덮어씌움 (Overwrite).
- **`operator.add` (추가하기):** 대화 내역(Message History)처럼 데이터를 계속 쌓아야 할 때 사용.
  ```python
  # 예시: 리스트에 메시지를 append 하는 방식
  messages: Annotated[list, operator.add]

## 3. 제어 흐름과 순환 (Control Flow & Cycles)

- **Cycles (순환):** Agent의 핵심. LLM이 문제를 해결할 때까지 `생각 -> 행동 -> 관찰` 과정을 반복(Loop)할 수 있게 함.
- **Special Nodes:**
  - `START`: 그래프 실행 시 사용자 입력을 State로 변환하여 주입하는 진입점.
  - `END`: 그래프 실행을 종료하고 최종 State를 반환하는 지점.

## 4. 고급 기능 및 최적화 (Advanced & Optimization)

### 💾 Checkpointer (Persistence & Memory)
- **Memory:** 그래프의 각 단계(Step)마다 State의 스냅샷을 저장.
- **Human-in-the-loop:** 실행을 일시 정지하고, 사람이 개입하여 승인/수정 후 다시 실행 가능.
- **Time Travel:** 과거의 특정 시점으로 되돌아가서 다른 경로로 실행 가능.

### ⚡ Node Caching
- 토큰 소모가 많거나 시간이 오래 걸리는 노드의 결과를 캐싱하여 비효율 방지.

### 🛠️ Compilation (컴파일)
- `builder.compile()`을 호출하여 그래프 구조를 실행 가능한 `Runnable` 객체로 변환해야 함.
