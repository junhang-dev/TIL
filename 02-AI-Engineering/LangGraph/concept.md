### Agents는 Graph이다
- State : Graph를 통해 흐르는 Data
- Node : Graph가 실제로 일을 하는 곳
- Edge : Node간 연결
- 각 Node는 State를 업데이트하고, LangGraph가 모든 Node마다 State를 준다
- 각 Node의 Return값이 다음 Node로 가는건 아니다
- State를 Private, Input, Output으로 나눌 수 있다
