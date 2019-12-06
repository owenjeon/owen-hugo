---
title: "My First Post"
date: 2019-12-07T02:01:14+09:00
draft: false
---

### Why we need reflection in JavaScript?

reflection은 동일한 시스템 (또는 그 자체)의 다른 코드를 검사 할 수 있는 코드를 설명하는 데 사용됩니다.

reflection은 여러 가지 유스 케이스 (Composition / DI, Run-time Type Assertions, Testing)에 유용합니다.

우리의 javascript 애플리케이션은 점점 더 커지고 있으며, 복잡성 관리를 위해 (컨트롤 컨테이너의 반전과 같은) 몇 가지 도구와 (Run-time Type Assertions) 같은 기능이 필요합니다. 
문제는 JavaScript에서 reflection이 없기 때문에 이러한 도구 및 기능 중 일부를 구현할 수 없거나 적어도 C # 또는 Java와 같은 프로그래밍 언어 에서처럼 강력 할 정도로 구현할 수 없다는 것입니다.

강력한 리플렉션 API를 사용하면 런타임에 알려지지 않은 객체를 검사하고 그것에 대해 모든 것을 찾을 수 있습니다. 
우리는 다음과 같은 것들을 발견 할 수 있어야합니다.

* 엔티티의 이름.
* 엔티티의 type.
* 엔티티에 의해 구현되는 인터페이스.
* 엔티티의 속성의 이름 및 유형.
* 엔티티의 생성자 인수의 이름과 유형.


JavaScript에서는 `Object.getOwnPropertyDescriptor()` 또는 `Object.keys()`와 같은 함수를 사용하여 엔티티에 대한 정보를 찾을 수 있지만, 더 강력한 개발 도구를 구현하려면 리플렉션이 필요합니다.

그러나 TypeScript가 일부 Reflection 기능을 지원하기 시작하기 때문에 상황이 바뀌었습니다. 이 기능을 살펴 보겠습니다.

### The metadata reflection API

metadata reflection API에 대한 기본 JavaScript 지원은 개발 초기 단계입니다. 
온라인으로 제공되는 Decorator Metadata 용 ES7 Reflection API의 프로토 타입과 함께 ES7에 Decorators를 추가하는 제안이 있습니다.

TypeScript 팀의 팀원 중 일부는 Polyfill을 사용하여 ES7 Reflection API 프로토 타입을 작성했으며 
TypeScript 컴파일러는 데코레이터에 대한 일련의 `design-time` 유형 메타 데이터를 내 보낼 수 있게되었습니다.

`reflect-metadata` 패키지를 사용하여이 메타 데이터 리플렉션 API polyfill을 사용할 수 있습니다.

```
npm install reflect-metadata;
```

TypeScript의 tsconfig.json파일의  `emitDecoratorMetadata`를 `true`로 설정해야합니다. 또한 우리는 `metadata.d.ts` 반영에 대한 참조를 포함해야합니다. 

그런 다음 자체 데코레이터를 구현하고 사용 가능한 반영 메타 데이터 디자인 키 중 하나를 사용해야합니다. 현재로서는 세 가지만 사용할 수 있습니다.

A. reflect metadata API를 사용하여 type metadata 얻기

다음과 같이 속성(property) decorator를 선언해봅시다.

```typescript
function logType(target : any, key : string) {
  var t = Reflect.getMetadata("design:type", target, key);
  console.log(`${key} type: ${t.name}`);
}
```

그런 다음 클래스의 속성 중 하나에 적용하여 해당 type을 가져올 수 있습니다.

```typescript
class Demo{ 
  @logType // apply property decorator
  public attr1 : string;
}
```
위 예제를 실행하면 아래의 콘솔 로그가 찍힙니다.

```
attr1 type: String
```

B) Obtaining Parameter type metadata using the reflect metadata API

parameter decorator를 선언해 봅시다.

```typescript
function logParamTypes(target : any, key : string) {
  var types = Reflect.getMetadata("design:paramtypes", target, key);
  var s = types.map(a => a.name).join();
  console.log(`${key} param types: ${s}`);
}
```

그런 다음 클래스의 메서드 중 하나에 이 함수를 적용하여 인수의 형식에 대한 정보를 얻을 수 있습니다.

```typescript
class Foo {}
interface IFoo {}

class Demo{ 
  @logParamTypes // apply parameter decorator
  doSomething(
    param1 : string,
    param2 : number,
    param3 : Foo,
    param4 : { test : string },
    param5 : IFoo,
    param6 : Function,
    param7 : (a : number) => void,
  ) : number { 
      return 1
  }
}
```
위 예제를 실행하면 아래의 콘솔 로그가 찍힙니다.
```
doSomething param types: String, Number, Foo, Object, Object, Function, Function
```

C) Obtaining return type metadata using the reflect metadata API

`design : returntype`메타 데이터 키를 사용하여 메소드의 반환 유형에 대한 정보를 얻을 수도 있습니다.

