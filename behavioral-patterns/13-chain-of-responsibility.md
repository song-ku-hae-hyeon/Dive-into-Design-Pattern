# 책임 연쇄 패턴

> 핸들러들의 체인(사슬)을 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴입니다. 각 핸들러는 요청을 받으면 요청을 처리할지 아니면 체인의 다음 핸들러로 전달할지를 결정합니다.

## Overview

- 쇼핑몰 시스템을 개발한다고 가정해본다.
- 아래와 같은 상황을 생각해볼 수 있다.
  - 사용자가 로그인된 상태여야 주문에 접근 가능
  - 사용자가 관리자 권한을 가졌다면 모든 권한을 가져야 함
- 이때 시간이 지남에 따라 데이터 검증, 캐시 등 같은 요청에 대해 추가적인 작업이 추가될 수 있다.
  - 이 경우 기능을 추가할 때마다 시스템의 복잡도는 배로 증가하게 됨
- 이러한 작업 하나하나를 `핸들러`라는 독립적인 객체로 다루는 것이 `책임 연쇄` 패턴

### When to use

- 프로그램이 다양한 방식으로 여러 종류의 요청을 처리할 것으로 예상되지만, 종류와 순서를 미리 알 수 없는 경우에 사용
- 특정 순서로 여러 핸들러를 실행해야 할 때에 사용
- 핸들러들의 집합과 그들의 순서가 런타임에 변경되어야 할 때 사용

## Terminology

- Handler: 모든 구상 핸들러에 공통적인 인터페이스
  - 일반적으로 요청을 처리하는 단일 메서드만 가지고 있음
- Base Handler(Optional): 모든 핸들러들에 공통적인 부분을 정의함
  - 보통 다음 핸들러에 대한 참조(레퍼런스)를 필드로서 가짐
- Concrete Handler: 요청을 처리하는 실제 코드가 포함
  - 다음 핸들러로 요청을 전달할지 말지를 결정해야함
  - 일반적으로 자기완결적이며 불변성을 가짐
- Client: 앱의 로직에 따라 체인을 동적으로 구성할 수도 있음
  - 요청은 체인의 모든 핸들러에 보낼 수 있음
  - 꼭 첫번째 핸들러에만 요청을 보내야 하는 것은 아님

## Usecase

```ts
// 사용자의 입력값을 검증하는 예제
// 입력값은 3글자 이상, 10글자 미만이어야 하며 `-`, `_`, 알파벳 소문자, 숫자만 허용한다고 가정

// Handler
interface Handler {
  handle(input: string): boolean;
}

// Base Handler
abstract class BaseHandler implements Handler {
  public nextHandler?: Handler;

  public abstract handle(input: string): boolean;

  public chain(handler: Handler): void {
    this.nextHandler = handler;
  }

  public next(input: string): boolean {
    if (!this.nextHandler) return true;
    return this.nextHandler.handle(input);
  }
}

// Concrete Handler
class LargerThanThreeCharacter extends BaseHandler {
  public handle(input: string) {
    if (input.length >= 3) return this.next(input);
    return false;
  }
}

class ShorterThanTenCharacter extends BaseHandler {
  public handle(input: string) {
    if (input.length < 10) return this.next(input);
    return false;
  }
}

class DontHaveForbiddenCharacter extends BaseHandler {
  public handle(input: string) {
    if (input.match(/[^a-zA-z0-9-_]/) === null) return this.next(input);
    return false;
  }
}

// Client
class Main {
  public static main() {
    const handler1 = new LargerThanThreeCharacter();
    const handler2 = new ShorterThanTenCharacter();
    handler1.chain(handler2);
    const handler3 = new DontHaveForbiddenCharacter();
    handler2.chain(handler3);

    const input1 = "abcd";
    const result1 = handler1.handle(input1);
    console.log("result1", result1); // true

    const input2 = "abcd-efg";
    const result2 = handler1.handle(input2);
    console.log("result2", result2); // true

    const input3 = "invalid$input";
    const result3 = handler1.handle(input3);
    console.log("result3", result3); // false
  }
}

Main.main();
```
