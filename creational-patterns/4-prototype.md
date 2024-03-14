## 프로토 타입 패턴

- 클래스에 의존시키지 않고 기존 객체들을 복사할 수 있도록 하는 생성 디자인 패턴
- 프로토타입은 복제를 지원하는 객체를 의미함
- 디자인 패턴으로서의 프로토타입은 이미 존재하는 객체를 효율적으로 복사하기 위한 방법론의 일종이다.
- 반면, 자바스크립트의 프로토타입은 상위 객체의 프로퍼티를 하위 객체가 상속받기 위해 사용하는 자바스크립트의 고유 기능이다.

#### 문제

- A 객체의 정확한 복사본을 만들고 싶다면?
  - A 객체 생성에 사용한 클래스로 새로운 B 객체 생성
  - A 기존 객체의 필드들을 살펴본 후 B 객체에 복사
- 항상 가능하지는 않음
  - 객체의 필드 중 일부가 비공개일 수 있기 때문임
  - 객체를 복사하고 싶을 뿐인데 클래스에 의존해야 함
    - 클래스 생성자에 필요한 파라미터를 알아야 함

#### 해결책

- 프로토타입 패턴은 복제되는 객체에 복제 프로세스를 위임함
  - 문제에서 예시로 든 A 객체
- 복제를 지원하는 모든 객체에 대한 공통 인터페이스 선언 → 객체의 클래스에 결합하지 않고 해당 객체 복제할 수 있음
  - 일반적으로 인터페이스에는 단일 `clone` 메서드만 포함됨
- `clone` 메서드는 현재 클래스의 객체를 만든 후 이전 객체의 모든 필드 값을 새 객체로 전달함
- 객체에 수십 개의 필드와 가능한 설정이 존재하는 경우 서브클래싱 대신 프로토타입 패턴을 사용할 수 있음

#### 구조

1. 프로토타입 인터페이스는 복제 메서드를 선언
2. 구상 프로토타입 클래스가 복제 메서드를 구현 : 객체의 데이터 복사 외에도 복제 프로세스와 관련된 일부 예외적인 경우들도 처리할 수 있음 ex) 연결된 객체 복제, 재귀 종속성 풀기
3. 클라이언트는 프로토타입 인터페이스를 따르는 모든 객체의 복사본을 생성할 수 있음

#### 적용

- 복사해야 하는 객체들의 구상 클래스들에 코드가 의존하면 안 될 때 사용
  - 타사 코드에서 전달된 객체들과 함께 작동할 때 발생
    - 접근이 제한적이기 때문에 해당 클래스들에 의존이 어려움
  - `clone` 인터페이스는 클라이언트 코드가 복제하는 객체의 구상 클래스들에서 클라이언트 코드를 독립시킴
- 각각의 객체를 초기화하는 방식만 다른 자식 클래스들(서브 클래싱)의 수를 줄이고 싶을 때 사용

#### 구현 방법

1. 프로토타입 인터페이스 생성 → 그 안에 `clone` 메서드 선언
2. 프로토타입 클래스
   - 클래스 객체를 인수로 받는 대체 생성자 정의. 대체 생성자는 새로 생성된 인스턴스로 복사해야 함.
   - 프로그래밍 언어가 오버로딩을 지원하지 않으면 별도의 '프로토타입' 생성자를 만들 수 없음.
     - 객체의 데이터를 새로 생성된 복제본에 복사하는 작업은 `clone` 메서드 내에서 수행되어 함.
     - 그래도 일반적인 생성자에 두는 것보다는 안전함. new 연산자를 호출한 직후에 생성된 객체는 완전히 설정된 상태이기 때문임
3. 복제 메서드 실행
   - 생성자의 프로토타입 버전으로 new 연산자 실행
   - 복제 메서드를 오버라이딩한 후 new 연산자와 함께 자체 클래스 이름을 사용
4. 프로토타입의 카탈로그를 저장할 중앙 프로토타입 레지스트리 생성할 수 있음

#### 장단점

- 객체가 구상 클래스들에 결합하지 않고 복제할 수 있음
- 복잡한 객체에 대한 사전 설정을 처리할 때 상속 대신 사용할 수 있음
- 복잡한 객체를 쉽게 생성
- 순환 참조가 있는 복잡한 객체를 복제하는 것은 매우 까다로울 수 있음

#### 예제 코드

```ts
export interface Cloneable<T> {
  clone(): T;
}

export class ComponentPrototype implements Cloneable<ComponentPrototype> {
  clone(): ComponentPrototype {
    return { ...this };
  }
}
```

```ts
export class Document extends ComponentPrototype {
  components: ComponentPrototype[] = [];

  clone(): Document {
    const clonedDocument = new Document();
    clonedDocument.components = this.components.map((c) => c.clone());
    return clonedDocument;
  }

  add(component: ComponentPrototype) {
    this.components.push(component);
  }
}

export class Title extends ComponentPrototype {
  constructor(public text: string) {
    super();
  }

  setText(text: string) {
    this.text = text;
  }
}

export class Drawing extends ComponentPrototype {
  constructor(public shape: "circle" | "square" | "line") {
    super();
  }

  setShape(shape: "circle" | "square" | "line") {
    this.shape = shape;
  }
}
```

```ts
/**
 * The client code.
 */
const document = new Document();
const title = new Title("Example Domain");

document.add(title);
document.add(new Drawing("line"));

const clonedDocument = document.clone();
title.setText("New title for the original document");

console.log("document is:");
console.log(document);
console.log("clonedDocument is:");
console.log(clonedDocument);
```

#### Refs

- [message box example](https://velog.io/@ninthsun91/Typescript%EB%A1%9C-%EB%8B%A4%EC%8B%9C-%EC%93%B0%EB%8A%94-GoF-Prototype)
- [nodejs example](https://medium.com/@diegomottadev/exploring-prototype-design-pattern-implementation-with-typescript-and-node-js-b2683fcd29b7)
