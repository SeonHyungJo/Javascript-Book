# Call Stack

## Intro

지금 이 글을 보고 있는 우리가 공부할 언어는 Script 언어로 이름은 **자바스크립트**다.

공부할 언어가 어떤 것이 어떻게 실행되는지 알아보기 위해서는 엔진에 대해서 알아야 한다고 생각한다.

## V8 Engine

가장 많이 사용되는 브라우저는 Chrome이다. 이 Chrome에서 사용되는 자바스크립트 엔진은 구글의 **V8 Engine**이다. 

V8 Engine에는 2가지 Main Components가 있다.

1. Memory Heap : 메모리의 할당이 일어나는 곳.
2. Call Stack : Stack Frame이 실행되는 곳. 쉽게 말해서 우리가 작성한 코드가 실행되는 곳.

## Call Stack

Call Stack은 **LIFO** (Last In, First Out) 원리를 사용하여 함수 호출을 임시 저장하고 관리하는 데이터 구조.

> LIFO : Last In, First Out의 데이터 구조 원칙에 따라 Call Stack이 작동한다. Stack으로 push된 마지막 함수가 처음으로 나옴을 의미한다.

자바스크립트에서 Call Stack은 주로 함수 호출에 이용된다. Call Stack이 하나이기 때문에 함수 실행은 위에서 아래로 한 번에 하나씩 수행된다.

기본적으로 자바스크립트는 싱글 스레드 프로그래밍 언어이다. 이 말은 하나의 Call Stack을 가지고 있다는 것을 의미한다. 다른 말로 하자면 한 번에 한 가지 일만 할 수 있다는 것이다.

```js
function multiply(x, y) {
  return x * y;
}

function printSquare(x) {
  const s = multiply(x, x);
  console.log(s);
}

printSquare(5);
```

위와 같은 코드가 있다면, 

> `printSquare(5) ⇒ multiply(x, x) ⇒ console.log(s) ⇒ printSquare(5)`

순서로 쌓이고 실행이 된다는 것을 의미한다.

> 각 한 줄을 `Stack Frame`이라고 한다.

```javascript
function foo() {
  throw new Error('Session Stack will help you resolve crashes :)');
}

function bar() {
  foo();
}

function start() {
  bar();
}

start();
```

위와 같은 코드를 Chrome에서 실행한다면 당연하게 에러가 나면서 Stack 형태를 자세히 볼 수 있다.

다른 경우로는 Call Stack의 사이즈를 넘어서 쌓이는 경우도 발생한다.

```js
let i = 0;

function recurse () {
  i++;
  recurse();
}

try {
  recurse();
} catch (e) {
  alert('maxStackSize = ' + i + '\nerror: ' + e);
}
```

위의 코드를 사용하면 안되지만 위의 같은 경우는 브라우저에서 계속 쌓아가다가 크롬 기준 **약 15,000개가** 넘어가는 순간에 Stack Size가 넘쳤다고 나올 것이다.

![Stack over flow_braze](https://user-images.githubusercontent.com/24274424/97084132-cf3c6000-164f-11eb-8a5c-94355a19fad5.png)

> 당연하겠지만 엔진이 다르면 최대치가 다르다.

싱글 스레드는 멀티 스레드보다 다루기는 쉽다.

> Deadlock(교착상태) 같은 일이 발생하지 않는다.

그러나 역시 제한적이다. 예시로 내가 버튼을 눌러서 서버에서 사진을 가져오려고 할 때. 버튼을 누르고 사진을 가져올 때까지 브라우저는 멈춘 상태가 되어 버린다.

이에 대안으로 **Asynchronous Callbacks** 이다.

## Web API

자바스크립트에 제공되지 않는 것들이 있다. 우리가 비동기를 사용하기 위해서 사용하는 `setTimeOut()`, `setInterval()`와 같은 기능들이 브라우저에서 제공하는 API라고 생각하면 된다. 

지원하는 API는, 

1. DOM Event
2. AJAX(=XMLHttpRequest)
3. setTimeOut
4. 이외

> 브라우저 웹 API : DOM 이벤트, `XMLHttpRequest`, `setTimeout`과 같은 비동기 이벤트를 처리하기 위해 C++로 구현된 **브라우저로 만든 스레드**

## Queue(Message Queue || CallBack Queue)

자바스크립트 런타임에는 처리할 메시지 목록과 콜백 함수인 Message Queue가 있다. 현재 Stack의 용량이 충분하다면 Queue에서 메시지를 가져와서 처리된 CallBack function을 실행한다.

기본적으로 이러한 메시지는 콜백 기능이 제공되면 외부 비동기 이벤트에 대해 응답을 한다. 예를 들어 사용자가 버튼을 클릭하고 콜백 함수가 제공되지 않으면 아무런 메시지도 Queue에 추가되지 않게 되는 것이다.

## Event Loop

네크워크는 느리다. 사진을 불러오는 것이 느린 이유이다. 이에 흔히 사용되는 것이 `AJAX`라 불리는 **비동기 함수**다. 만약 이러한 작업이 동기라면 사진을 불러오는 동안 화면이 멈추는 현상이 일어날 것이다.

가장 쉬운 해결책이 **Asynchronous Callbacks**이다. 

`console.log()`와 다르게 바로 실행되지 않는다. 그렇다면 이런 어떻게 되는가?

응답에서 호출자를 분리하면 비동기 작업이 완료되고 콜백이 시작될 때까지 기다리는 시간 동안 자바스크립트 런타임에서는 다른 작업을 할 수 있다. 

Web API에서 요청한 작업을 완료한 후 Callbacks을 실행해야 한다. 그러나 만약 작업이 완료되고 직접 Web API 쪽에서 Call Stack에 실행 코드를 넣을 수 있다면 끝나는 즉시 Call function이 실행될 것이다. 

그래서 있는 것이 **Queue**다. Web API에서 요청한 작업을 완료한 후에 **Queue**에 넣어 준다. 

**Event Loop는 Call Stack이 비었을 때를 파악하여 Queue에서 들어온 Callback function를 순서에 맞게 수행한다.**

## Execute Context

실행 컨텍스트는 자바스크립트 코드가 평가되고 실행되는 환경의 추상적인 개념이다. 자바스크립트에서 코드를 실행할 때마다 실행 컨텍스트 내에서 실행된다.

자바스크립트 내에는 3가지 타입의 실행컨텍스트가 있다고 한다.

1. Global 
2. Functional
3. ~~Eval Function~~

중요한 것은 **1번, 2번**이다.

### Execution Stack

실행 스택은 위에서 보았다. Call Stack의 개념이다. 실행컨텍스트를 만드는 두 단계가 있다.

1. Creation 단계
2. Execute 단계

#### Creation 단계

한국말로는 만드는 단계이다. 두 가지를 만든다.

1. **LexicalEnvironment**
2. VariableEnvironment

기본 형태는 아래와 같다.

```javascript
ExecutionContext = {
  LexicalEnvironment = <ref. to LexicalEnvironment in memory>,
  VariableEnvironment = <ref. to VariableEnvironment in  memory>,
}
```

#### Lexical Environment

Lexical Environment은 **식별자, 변수 맵핑**을 가지고 있는 구조다. 

> 여기서 식별자는 변수/함수의 이름을 가리키며 변수는 실제 객체 [함수 객체 및 배열 객체 포함] 대한 참조입니다.

```javascript
var a = 20;
var b = 40;

function foo() {
  console.log('bar');
}
```

위와 같은 코드가 있다고 하면 Lexical Environment은

```javascript
lexicalEnvironment = {
  a: 20,
  b: 40,
  foo: <ref. to foo function>
}
```

위와 같이 만들어질 것이다.

여기에는 3가지의 정보를 가지고 있다.

1. **Environment Record** : 변수 및 함수 선언이 Lexical Environment 내에 저장되는 장소
2. **Reference to the outer environment** : 외부 환경에 대한 참조
3. **This binding** : `this`가 결정되거나 설정된다.

#### Environment Record

기본적으로 2가지 정보를 담고 있다.

- **Declarative environment record** (선언적 환경 정보) : 변수 및 함수 선언을 저장합니다.
- **Object environment record** (객체 환경 정보) :  전역 코드의 Lexical Environment에는 객체 환경 레코드가 포함되어 있다. 변수 및 함수 선언 외에 객체 환경 레코드는 전역 바인딩 객체 (브라우저의 창 개체)도 저장합니다. 따라서 각 바인딩 객체의 속성에 대해 새 항목이 레코드에 만들어진다.

함수 코드의 경우 Environment Record에는 함수에 전달된 인덱스와 인수 사이의 매핑과 함수에 전달된 인수의 길이가 포함된 `arguments` 객체도 포함이 된다. 

예를 들어, 아래 함수에 대한 인수 객체는 다음과 같다.

```javascript
function foo(a, b) {
  var c = a + b;
}

foo(2, 3);

// argument object
Arguments: {0: 2, 1: 3, length: 2},
```

#### Reference to the Outer Environment

외부 환경에 대한 참조는 outer environment에 액세스 할 수 있음을 의미한다. 즉, 자바스크립트 엔진은 현재 lexical environment에서 찾을 수 없는 경우 outer environment에서 변수를 찾을 수 있다.

#### This Binding

여기에는 어렵고도 중요한 개념인 `this`가 결정되거나 설정된다.

전역 실행 컨텍스트에서이 값은 전역 개체를 참조합니다.

함수 실행 컨텍스트에서이 값은 함수가 호출되는 방식에 따라 다르게 `this`가 나온다. 객체 참조 때문에 호출되면 `this` 값은 해당 객체로 설정되고, 그렇지 않으면 이 값은 전역 객체로 설정되거나 정의되지 않는다.

```javascript
const person = {
  name: 'peter',
  birthYear: 1994,
  
  calcAge: function() {
    console.log(2018 - this.birthYear);
  }
}

person.calcAge(); 
// 'this' refers to 'person', because 'calcAge' was called with //'person' object reference

const calculateAge = person.calcAge;
calculateAge();
// 'this' refers to the global window object, because no object reference was given
```

#### Variable Environment

이것은 위에서 봤던 LexicalEnvironment와 같다.

ES6에서 LexicalEnvironment 구성 요소와 VariableEnvironment 구성 요소의 차이점 중 하나는 함수 선언과 변수 (`let` 및 `const`)바인딩을 저장하는데 사용되는 반면, 후자는 변수 (`var`)바인딩만 저장하는 데 사용된다.

#### Execute 단계

이 단계에서 모든 변수에 대한 할당이 완료되고 코드가 최종적으로 실행된다.

```javascript
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
  var g = 20;
  return e * f * g;
}

c = multiply(20, 30);
```

```javascript
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
  
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

```javascript
GlobalExectionContext = {
    
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: 20,
      b: 30,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
    
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

```javascript
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

```javascript
FunctionExectionContext = {
  LexicalEnvironment: {
      EnvironmentRecord: {
        Type: "Declarative",
        // Identifier bindings go here
        Arguments: {0: 20, 1: 30, length: 2},
      },
      outer: <GlobalLexicalEnvironment>,
      ThisBinding: <Global Object or undefined>,
    },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: 20
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

함수 실행이 완료된 후 반환 값은 c 안에 저장된다. 그래서 글로벌 Variable Environment에 업데이트된다. 그 후, 전역 코드가 완료되고 프로그램이 완료된다.

`let` 및 `const` 정의 변수는 생성 단계에서 연관된 값이 없지만 `var` 정의 변수는 `undefined`로 설정된다.

이는 생성 단계에서 함수 선언이 환경에 전체적으로 저장되는 동안 변수 및 함수 선언에 대해 코드가 검색되고 변수가 초기에 `undefined`(`var`의 경우)로 설정되거나 초기화되지 않은 상태로 유지되기 때문이다. `let`과 `const`의 경우).

`var` 정의 변수가 선언되기 전에 (정의되지는 않았지만) `var` 정의 변수에 액세스 할 수 있지만 `let` 및 `const` 변수가 선언되기 전에 액세스 할 때 참조 오류가 발생하는 이유다.

실행 단계에서 자바스크립트 엔진이 `let` 변수의 값을 소스 코드에서 선언된 실제 위치에서 찾지 못하면 정의되지 않은 값을 할당한다.