# Strategy Pattern

> **전략 패턴**이란 런타임에 알고리즘 전략을 선택하여 객체 동작을 변경할 수 있게 하는 행동 디자인 패턴

### Problem

지도 앱의 자동 경로 계획 기능 → 주소를 입력하면 해당 목적지까지의 최단 경로를 보여줌
처음에는 도로로 된 경로만 제공. 시간이 지나면서 도보, 대중교통, 자전거 등 다양한 수단을 활용한 기능 제공 필요해짐.

- 새 경로 구축 알고리즘 추가 시 메인 클래스의 크기가 두 배로 늘어남
- 간단한 버그 수정의 영향 범위가 큼
- 한 클래스의 수정이 타 클래스의 수정에도 영향을 줌

### Solution

- 특정 작업을 다양한 방식으로 수행하는 기능을 Strategy Class로 추출한다.
- 작업 실행을 Strategy Class에 위임하기 위한 Context Class를 선언한다.

### Implementation

```ts
class Context {
  private strategy: Strategy;

  setStrategy(strategy: Strategy) {
    this.strategy = strategy;
  }

  executeStrategy() {
    this.strategy.execute();
  }
}

interface Strategy {
  execute(): void;
}

class TransporationStrategy implements Strategy {
  execute(): void {
    console.log("Transporation Strategy Execute");
  }
}
class DriverStrategy implements Strategy {
  execute(): void {
    console.log("Driver Strategy Execute");
  }
}

class PedestrianStrategy implements Strategy {
  execute(): void {
    console.log("Pedestrian Strategy Execute");
  }
}

(() => {
  const context = new Context();
  const strategy = getStrategy();

  switch (strategy) {
    case "transportation":
      context.setStrategy(new TransporationStrategy());
      context.executeStrategy();
      return;
    case "driver":
      context.setStrategy(new DriverStrategy());
      context.executeStrategy();
      return;
    case "pedestrian":
      context.setStrategy(new PedestrianStrategy());
      context.executeStrategy();
      return;
  }
})();

function getStrategy(): "transportation" | "driver" | "pedestrian" {
  const stragey: "transportation" | "driver" | "pedestrian" = "transportation";
  return stragey;
}
```

### When to use

- 런타임 중에서 알고리즘 전환하고 싶은 경우
- 일부 행동을 실행하는 방식에서만 차이가 있는 유사한 클래스가 많은 경우
