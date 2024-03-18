> 객체들을 새로운 행동들을 포함한 특수 래퍼 객체들 내에 넣어서 위 행동들을 해당 객체들에 연결시키는 구조적 디자인 패턴

## 문제
알림 라이브러리의 요구사항이 계속 변함.
1. 이메일 리스트에 알림을 전송
2. 여러 SNS의 알림을 지원![image](https://github.com/song-ku-hae-hyeon/Dive-into-Design-Pattern/assets/50704533/e4ddabfd-34a8-43a3-8b35-6286349b9005)
3. 사용자마다 대상들을 선택![image](https://github.com/song-ku-hae-hyeon/Dive-into-Design-Pattern/assets/50704533/9842d72f-a5fa-49af-b1c2-d1771fd97158)
조합의 갯수만큼 알림 구상 클래스도 늘어난다.

## 해결책
객체의 동작을 변경해야 할 때 가장 먼저 고려되는 방법은 클래스의 확장
### 상속보다 복합(책에서는 집합)
- 상속은 정적이다.
    - 런타임에 기존 객체의 행동을 변경할 수 없음
- 자식 클래스는 하나의 부모 클래스만 가질 수 있다.
    - 대부분의 언어에서 다중 상속을 허용하지 않음.

- 래퍼는 일부 대상 객체와 연결할 수 있는 객체
    - 대상 객체와 같은 메서드들의 집합이 포함
        - 동일한 인터페이스 -> 클라이언트 관점에서는 모두 같은 객체
    - 자신이 받는 모든 요청을 대상 객체에 위임
        - 위임하기 전/후에 무언가를 수행할 수 있음

![image](https://github.com/song-ku-hae-hyeon/Dive-into-Design-Pattern/assets/50704533/2e28e10a-5ae6-497b-9e26-4837da6ab383)
- 여러 래퍼로 객체를 포장해서 모든 래퍼들의 합성된 행동들을 객체에 추가할 수 있음.
    - 클라이언트 코드는 데코레이터들의 집합으로 래핑해야 함.
    - 모든 데코레이터들이 기초 알림자와 같은 인터페이스를 구현해서 가능한 일.

![image](https://github.com/song-ku-hae-hyeon/Dive-into-Design-Pattern/assets/50704533/832fc5e8-0e6d-4e74-a2a5-d1d3012333de)

## 의사코드

```ts
interface DataSource {
    writeData(data)
    readData(): string
}

class FileDataSource implements DataSource {
    data
    constructor(filename) {}

    writeData(data) {
        this.data = data
        console.log(`write ${data} \n`)
    }
    readData(): string { 
        console.log(`read ${this.data}`)
        return this.data
    }
}

class DataSourceDecorator implements DataSource {
    protected wrappee: DataSource

    constructor(source: DataSource) {
        this.wrappee = source
    }

    writeData(data: any) {
        this.wrappee.writeData(data)
    }

    readData(): string {
        return this.wrappee.readData()
    }
}

class EncryptionDecorator extends DataSourceDecorator {
    writeData(data: any) {
        console.log('do encryption')
        this.wrappee.writeData(data)
    }

    readData(): string {
        const data = this.wrappee.readData()
        console.log('do decryption')
        return data
    }
}

class CompressionDecorator extends DataSourceDecorator {
    writeData(data: any) {
        console.log('do compression')
        this.wrappee.writeData(data)
    }

    readData(): string {
        const data = this.wrappee.readData()
        console.log('do decompression')
        return data
    }
}

class Application {
    dumbUsageExample() {
        let source: DataSource = new FileDataSource('foo.txt')
        console.log('기본 파일 입출력')
        source.writeData('foo bar baz')
        source.readData()
        
        console.log('\n압축 파일 입출력')
        source = new CompressionDecorator(source)
        source.writeData('foo bar baz')
        source.readData()

        console.log('\n암호화/압축 파일 입출력')
        source = new EncryptionDecorator(source)
        source.writeData('foo bar baz')
        source.readData()
    }
}

new Application().dumbUsageExample()
/* 출력 결과

기본 파일 입출력
write foo bar baz 

read foo bar baz

압축 파일 입출력
do compression
write foo bar baz 

read foo bar baz
do decompression

암호화/압축 파일 입출력
do encryption
do compression
write foo bar baz 

read foo bar baz
do decompression
do decryption
*/


// 외부 데이터 소스를 사용하는 클라이언트 코드 예시
// SalaryManager 객체들은 데이터 저장 세부 사항들을 알지도 못하고 신경 쓰지 않아도 된다.
class SalaryManager {
    source: DataSource

    constructor(source: DataSource) { this.source = source }

    load() {
        return this.source.readData()
    }

    save() {
        this.source.writeData('foo bar baz')
    }
}

// 앱 설정 혹은 환경에 따라 다양한 데코레이터 "스택"들을 조합할 수 있다.
class ApplicationConfigurator {
    configurationExample() {
        let source: DataSource = new FileDataSource('foo.txt')
        if (env.enabledEncryption) {
            source = new EncryptionDecorator(source)
        }
        if (env.enabledCompression) {
            source = new CompressionDecorator(source)
        }

        const logger = new SalaryManager(source)
        const salary = logger.load()
    }
}
```

## 적용
- 클라이언트 코드를 훼손하지 않으면서 런타임에 추가 행동들을 객체들에 할당 할 수 있을 때 사용
    - 비즈니스 로직을 계층(스택)으로 구성
- 상속을 사용하여 객체의 행동을 확장하는 것이 어색하거나 불가능할 때 사용
    - 상속을 방지하는 final 키워드가 있음.
## 장점
- 새 자식 클래스를 만들지 않고도 객체의 행동을 확장 가능
- 런타임에 책임들을 추가/제거 가능
- 여러 행동들을 합성 가능
- 모놀리식 클래스를 여러 개의 작은 클래스들로 나눌 수 있음. (단일 책임 원칙)
## 단점
- 래퍼들의 스택에서 특정 래퍼를 제거하기 어려움.
- 데코레이터의 행동이 데코레이터 스택 내의 순서에 의존하지 않는 방식으로 구현하기 어려움.
- 계층들의 초기 설정 코드가 더러울 수 있음.