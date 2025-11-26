# [MLOps] 공정 데이터 이상탐지를 위한 K-Means 최적화: 기하학적 접근과 현장 적용 전략

## 1. 배경 (Problem Context)
공정 데이터를 분석하여 유사 운전 패턴(Operating Mode)을 정의하고, 이를 벗어나는 상황을 이상(Anomaly)으로 탐지하는 프로젝트를 진행함.

* **초기 접근:** K-Means Clustering을 사용하고, 최적의 $k$(군집 수)를 찾기 위해 Inertia의 변화율(Gradient Ratio)을 보는 Elbow Method를 사용.
* **문제점:** 실전 데이터는 교과서처럼 매끄럽게 떨어지지 않음. 국소적인(Local) 변화율에 의존하다 보니 노이즈에 취약하고, 공정마다 임계값(Threshold)을 일일이 튜닝해야 하는 확장성(Scalability) 문제 발생.

> **핵심 질문:** "수많은 공정으로 모델을 확장(Scale-out)해야 하는데, 사람의 개입 없이 **강건하게(Robust)** 최적의 $k$를 찾는 방법은 없을까?"

---

## 2. 기술적 해결: 기하학적 접근 (Kneedle Algorithm)

단편적인 수치(기울기)가 아니라, 데이터의 전체적인 형상(Global Shape)을 보고 꺾임 점을 찾는 **Kneedle Algorithm**을 도입함.

### 2.1. 기존 방식 vs Kneedle 방식
| 구분 | 기존 방식 (Gradient Ratio) | 개선 방식 (Kneedle Algorithm) |
| :--- | :--- | :--- |
| **관점** | **미분(Local)**: "바로 옆 데이터보다 얼마나 변했나?" | **기하학(Global)**: "전체 추세선에서 얼마나 벗어났나?" |
| **장점** | 구현이 매우 단순함 | 노이즈에 강하고, 전체적인 구조를 파악함 |
| **단점** | 데이터가 울퉁불퉁하면 지역 최적해(Local Optima)에 빠짐 | 좌표 정규화 과정이 필요함 |

### 2.2. 알고리즘 동작 원리 (활시위 이론)
1.  **Normalization:** X축($k$)과 Y축(Inertia)의 단위가 다르므로 0~1 사이로 정규화.
2.  **Difference Curve:** 시작점($k_{min}$)과 끝점($k_{max}$)을 잇는 직선(활시위)을 그음.
3.  **Max Distance:** 각 $k$ 지점에서 직선까지의 **수직 거리**가 가장 먼 지점을 Elbow로 선정.

### 2.3. Python 구현 (NumPy)
```python
import numpy as np

def find_optimal_k_kneedle(k_range, inertia_values):
    """
    Kneedle Algorithm: 직선과 곡선 사이의 거리가 최대인 지점(Elbow)을 찾는 함수
    """
    # 1. 데이터 정규화 (Min-Max Scaling)
    k_norm = (k_range - k_range.min()) / (k_range.max() - k_range.min())
    inertia_norm = (inertia_values - inertia_values.min()) / (inertia_values.max() - inertia_values.min())
    
    # 2. 거리 계산 (직선에서 떨어진 거리)
    distances = []
    for i in range(len(k_range)):
        # (Inertia 감소 그래프 기준) 시작점(0,1)과 끝점(1,0)을 잇는 직선과의 거리 계산 수식
        # P1(0, 1), P2(1, 0) -> 직선 방정식: x + y - 1 = 0 (정규화된 좌표계)
        # 단순화된 로직: 기준 직선값 - 실제 곡선값 (Convex Curve 가정)
        distance = inertia_norm[0] - inertia_norm[i] - (inertia_norm[0] - inertia_norm[-1]) * k_norm[i]
        distances.append(distance)
        
    # 3. 최대 거리 지점 반환
    return k_range[np.argmax(distances)]

---

## 3. 전략적 통찰: 왜 K-Means인가? (Business Alignment)

학술적으로는 Auto-Encoder(딥러닝)나 TDA(위상 데이터 분석)가 더 고도화된 방법일 수 있지만, **현장(Field) 적용** 관점에서는 K-Means가 전략적 승리 요인을 가짐.

### 3.1. 설명 가능성 (Explainability)
* **딥러닝(AE):** "재구성 오차가 높아서 이상입니다." (Black-box) → 현장 엔지니어가 이해하기 어렵고 거부감 발생.
* **K-Means:** "현재 운전 상태는 **'패턴 A(고부하 운전)'** 그룹에 속하는데, 평소 패턴 A보다 **압력이 5% 높아서** 알람을 드렸습니다." → 직관적이고 납득 가능함.

### 3.2. 엑셀 필터의 진화 (Tool Evolution)
* 기존에 엔지니어가 엑셀에서 수동으로 필터링(`Temp > 100`)하던 업무를 알고리즘이 대신하여 **"초안(Draft)"**을 잡아주는 도구로 포지셔닝.
* AI가 사람을 대체하는 것이 아니라, **"귀찮은 데이터 탐색을 대신해주는 비서"**로 접근하여 현장 수용성(Acceptance)을 높임.

### 3.3. 확장성 (Scalability)
* **Kneedle 알고리즘** 덕분에 공정마다 데이터 분포가 달라도 **파라미터 튜닝 없이** 표준화된 파이프라인 적용 가능.
* `Docker` + `K-Means` + `Kneedle` 조합으로 전사 확대 적용을 위한 MLOps 기반 마련.

---

## 4. 결론 (Takeaway)
> **"현실적인 문제 해결(Problem Solving)은 최신 알고리즘을 쓰는 것이 아니라, 데이터의 구조(Shape)를 이해하고 사용자의 언어(Explainability)로 번역하는 것에서 온다."**

* **Keywords:** `K-Means`, `Elbow Method`, `Kneedle Algorithm`, `Geometric Approach`, `Explainable AI (XAI)`, `MLOps`
