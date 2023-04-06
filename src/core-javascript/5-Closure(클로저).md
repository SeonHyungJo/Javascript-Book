# Closure(클로저)

> 모든 내용은 책을 기반으로 작성되었습니다.

클로저라는 개념은 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성이다. 자바스크립트 고유의 개념이 아니라서 ECMAScript 명세에서도 클로저의 정의 다루지 않고 있다.

다양한 서적에서 클로저를 한 문장으로 요약해서 설명하는 부분들을 소개하면 아래와 같다.

```text
- 자신을 내포하는 함수의 컨텍스트에 접근할 수 있는 함수
- 함수가 특정 스코프에 접근할 수 있도록 의도적으로 그 스코프에서 정의하는 것
- 함수를 선언할 때 만들어지는 유효범위가 사라진 후에도 호출할 수 있는 함수
- 이미 생명 주기상 끝난 외부 함수의 변수를 참조하는 함수
- 자유 변수가 있는 함수와 자유 변수를 알 수 있는 환경의 결합
- 로컬 변수를 참조하고 있는 함수 내의 함수
- 자신이 생성될 때의 스코프에서 알 수 있었던 변수들 중 언젠가 자신이 실행될 때 사용할 변수들만을 기억하여 유지하는 함수
```

MDN은 클로저에 대해서 "클로저는 함수와 그 함수가 선언될 당시의 Lexical Environment의 상호관계에 따른 현상"이라고 적혀있다. 

'선언될 당시의 Lexical Environment'는 이전에 2장에서 소개한 실행 컨테스트의 구성 요소 중 하나인 outerEnvironmentReferences에 해당한다.

위에서의 의미만 조합해 본다면, '어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상`이라고 볼 수 있다.

```js
var outer = function() {
  var a = 1;
  var inner = function () {
    console.log(++a);
  };

  inner()
}

outer();
```

위의 예제에서 outer함수에서 변수 a를 선언했고, outer의 내부함수인 inner에서 a의 값을 1만큼 증가시킨 다음 출력한다.

inner 함수 내부에서는 a를 선언하지 않아서 environmenRecord에서 값을 찾지 못하므로 outerEnvironmentReference에 지정된 상위 컨텍스트인 outer의 outerEnvironment에 접근해서 다시 a를 찾는다. 

outer 함수의 실행 컨텍스트가 종료되면 Lexical Environment에 저장된 식별자들에 대한 참조를 지운다. 
그렇게 되면 자신을 참조하는 변수가 하나도 없게 되므로 가비지 컬렉터의 수집대상이 된다.


```js
var outer = function() {
  var a = 1;
  var inner = function () {
    console.log(++a);
  };

  return inner; // 여기가 바뀜
}

var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
```

이번에는 inner 함수를 반환했다.

그러면 outer함수의 실행 컨텍스트가 종료될 때 outer2 변수는 outer의 실행 결과인 innter 함수를 참조하게 된다. 
이후 outer2를 함수로 호출하게 되면 inner가 실행되게 된다.

inner 함수의 실행 컨텍스트의 environmentRecord에는 수집할 정보가 없다. 
outerEnvironmentReference에는 innter 함수가 선언된 위치의 Lexical Environment가 참조 복사된다. 
inner 함수는 outer 함수 내부에서 선언되었으므로, outer 함수의 Lexical Environment가 담기게 된다. 
이제 스코프체이닝에 따라 outer에서 선언한 변수 a에 접근해서 1만큼 증가시킨 후 결과를 찍는다.

그런데 inner함수의 실행 시점에는 outer함수는 이미 실행이 종료된 상태인데 outer함수의 LexicalEnvrionment에 어떻게 접근할 수 있을까? 

이는 가비지 컬렉터의 동작 방식 때문이다. **가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함되지 않는다.** 

위와 같은 이유론 inner함수가 a에 접근이 가능한 것이다.

이때까지 내용을 보게 되면 클로저는 `어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상`이다.

## return 없이도 클로저가 발생하는 다양한 경우

```js
(function () {
  var a = 0;
  var intervalId = null;
  var inner = function () {
    if (+=a >= 10){
      clearInterval(inervaId);
    }
    console.log(a);
  }
  intervalId = setInterval(inner, 1000);
})()
```

```js
(function () {
  var count = 0;
  var button = document.createElement('button');
  button.innerText = 'click';
  button.addEventListener('click', function() {
    console.log(++count, 'times clicked');
  })

  document.body.appendChild(button)
})()
```

첫 번째는 별도의 외부 객체인 window의 메서드(setTimeout 또는 setInterval)에 전달할 콜백 함수 내부에서 지역변수를 참조한다.

두 번째는 별도의 외부 객체인 DOM의 메서드(addEventListener)에 등록할 handler 함수 내부에서 지역변수를 참조한다. 

**두 경우 모두 지역변수를 참조하는 내부함수를 외부에 전달**했기 때문에 클로저다.

## 클로저와 메모리 관리

메모리 누수의 위험 이유로 클로저 사용을 조심해야 한다거나 심지어 지양해야 한다고 주장하는 사람들도 있지만, 메모리 소모는 클로저의 본질적인 특성일 뿐이다. 

오히려 이러한 특성을 정확히 이해하고 잘 활용하도록 노력해야 한다.

관리 방법은 간단하다. 클로저는 어떤 필요 때문에 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생한다. 그렇다면 그 필요성이 사라진 시점에는 더는 메모리를 소모하지 않게 해주면 된다.

> 식별자에 참조형이 아닌 기본형 데이터(보통 null이나 undefined)를 할당하면 된다.

```js
var outer = (function () {
  var a = a;
  var inner = function () {
    return ++a;
  }
  return inner;
})();

console.log(outer());
console.log(outer());
outer = null
```

```js
(function () {
  var a = 0;
  var intervalId = null;
  var inner = function() {
    if(++a >= 10){
      clearInterval(intervalId);
      inner = null;
    }
    console.log(a);
  };

  intervalId = setInterval(inner, 1000);
})();
```

```js
(function () {
  var count = 0;
  var button = document.createElement('button');
  button.innerText = 'click';

  var clickHandler = function(){
    console.log(++count, 'times clicked')
    if(count >= 10){
      button.removeEventListener('click', clickHandler);
      clickHandler = null;
    }
  };

  button.addEventListener('click', clickHandler);
  document.body.appendChild(button);
})();
```

## 클로저 활용 사례

### 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

```js
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement('ul'); // (공통 코드)

fruits.forEach(function(fruit) {
  // (A)
  var $li = document.createElement('li');
  $li.innerText = fruit;
  $li.addEventListener('click', function() {
    // (B)
    alert('your choice is ' + fruit);
  });
  $ul.appendChild($li);
});
document.body.appendChild($ul);
```

위의 예제에서 B 함수의 쓰임새가 콜백함수에 국한되지 않는 경우라면 반복을 줄이기 위해서 외부로 분리하는 편이 좋다.

```js
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement('ul');

var alertFruit = function(fruit) {
  alert('your choice is ' + fruit);
};
fruits.forEach(function(fruit) {
  var $li = document.createElement('li');
  $li.innerText = fruit;
  $li.addEventListener('click', alertFruit);
  $ul.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);
```

공통함수로 쓰고자 콜백함수를 외부로 꺼내서 alertFruit 변수에 담았다. 이제 함수를 직접 실행할 수 있다. 그런데 각 li를 클릭하면 클릭한 대상의 과일명이 아닌`[object MouseEvent]`라는 값이 출력된다. 콜백 함수의 인자에 대한 제어권을 addEvnetListener가 가진 상태이며, addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문이다. 이를 bind로 해결할 수 있다.

```js
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement('ul');

var alertFruit = function(fruit) {
  alert('your choice is ' + fruit);
};
fruits.forEach(function(fruit) {
  var $li = document.createElement('li');
  $li.innerText = fruit;
  $li.addEventListener('click', alertFruit.bind(null, fruit));
  $ul.appendChild($li);
});
document.body.appendChild($ul);
```

이렇게 하면 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점과 함수 내부에서의 this가 원래의 값과 달라지는 점을 감안해야한다. 
이를 해결하기 위햇 다른방식으로 고차함수를 활용하는 것이다. 
함수형 프로그래밍에서 자주 쓰이는 방식이다.

```js
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement('ul');

var alertFruitBuilder = function(fruit) {
  return function() {
    alert('your choice is ' + fruit);
  };
};
fruits.forEach(function(fruit) {
  var $li = document.createElement('li');
  $li.innerText = fruit;
  $li.addEventListener('click', alertFruitBuilder(fruit));
  $ul.appendChild($li);
});
document.body.appendChild($ul);
```

alertFruitBuilder라는 함수를 만들었다.

이 함수는 내부에서 익명함수를 반환하는데, 이 익명함수가 바로 기존의 alertFruit 함수다. alertFruitBuilder함수를 실행하면서 fruit값을 인자로 전달하고, 그러면 이 함수의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로써 전달할 것이다.


### 접근 권한 제어(정보 은닉)

정보 은닉이란 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈 간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나이다. 

자바스크립트는 기본적으로 변수 자체에 이러한 접근 권한을 직접 부여하도록 설계되어 있지 않다. 그렇다고 접근 권한 제어가 불가능한 건 아니다. 클로저를 이용하면 된다.

```js
var car = {
  fuel: Math.ceil(Math.random() * 10 + 10), // 연료(L)
  power: Math.ceil(Math.random() * 3 + 2), // 연비(km/L)
  moved: 0, // 총 이동거리
  run: function() {
    var km = Math.ceil(Math.random() * 6);
    var wasteFuel = km / this.power;
    if (this.fuel < wasteFuel) {
      console.log('이동불가');
      return;
    }
    this.fuel -= wasteFuel;
    this.moved += km;
    console.log(km + 'km 이동 (총 ' + this.moved + 'km)');
  },
};
```

이렇게 하면 `run` 메서드만 호출한다는 가정하에는 충분하다. 그러나 자바스크립트는 값을 변경할 수 있다.

```js
car.fuel = 10000;
car.power = 100;
car.moved = 1000;
```

아래와 같이 바꾸지 못하도록 클로저로 방어가 가능하다. 필요한 멤버만 `return` 하는 것이다.

```js
var createCar = function() {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료(L)
  var power = Math.ceil(Math.random() * 3 + 2); // 연비(km / L)
  var moved = 0; // 총 이동거리
  return {
    get moved() {
      return moved;
    },
    run: function() {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log('이동불가');
        return;
      }
      fuel -= wasteFuel;
      moved += km;
      console.log(km + 'km 이동 (총 ' + moved + 'km). 남은 연료: ' + fuel);
    },
  };
};
var car = createCar();
```

위와 같이 바꾸게 될 경우 run 메서드를 다른 내용으로 덮어씌우는 어뷰징은 여전히 가능하지만, 이전의 코드보다는 안전한 코드가 되었다.

어뷰징까지 막기 위해서는 객체를 return하기 전에 미리 변경할 수 없게 조치를 하면 된다.

```js
var createCar = function() {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료(L)
  var power = Math.ceil(Math.random() * 3 + 2); // 연비(km / L)
  var moved = 0; // 총 이동거리
  var publicMembers = {
    get moved() {
      return moved;
    },
    run: function() {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log('이동불가');
        return;
      }
      fuel -= wasteFuel;
      moved += km;
      console.log(km + 'km 이동 (총 ' + moved + 'km). 남은 연료: ' + fuel);
    },
  };
  Object.freeze(publicMembers); // 추가
  return publicMembers;
};
var car = createCar();
```

### 부분 적용 함수

부분 적용 함수란 n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m) 개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게하는 함수다. 
this를 바인딩해야 하는 점을 제외하면 앞서 살펴본 bind 메서드의 실행 결과가 바로 부분 적용 함수다.

```js
var add = function() {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};
var addPartial = add.bind(null, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10)); // 55
```

앞서 말했던 내용과 동일하게 this를 변경할 수밖에 없기 때문에 메서드에서는 사용할 수 없다. this에 관여하지 않는 별도의 부분 적용 함수를 만들어 보자.

```js
var partial = function() {
  var originalPartialArgs = arguments;
  var func = originalPartialArgs[0];
  if (typeof func !== 'function') {
    throw new Error('첫 번째 인자가 함수가 아닙니다.');
  }
  return function() {
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
    var restArgs = Array.prototype.slice.call(arguments);
    return func.apply(this, partialArgs.concat(restArgs));
  };
};

var add = function() {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};
var addPartial = partial(add, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10)); // 55

var dog = {
  name: '강아지',
  greet: partial(function(prefix, suffix) {
    return prefix + this.name + suffix;
  }, '왈왈, '),
};
dog.greet('입니다!'); // 왈왈, 강아지입니다.`
```

보통의 경우 부분 적용 함수는 이 정도로 충분하다.

다른 예제로 많이 사용하는 디바운스를 구현해보자. 디바운스는 짧은 시간 동안 동일한 이벤트가 많이 발생할 겨웅 이를 전부 처리하지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한 번만 처리하는 것으로 최적화에 큰 도움을 주는 기능 중 하나이다.

```js
var debounce = function(eventName, func, wait) {
  var timeoutId = null;
  return function(event) {
    var self = this;
    console.log(eventName, 'event 발생');
    clearTimeout(timeoutId);
    timeoutId = setTimeout(func.bind(self, event), wait);
  };
};

var moveHandler = function(e) {
  console.log('move event 처리');
};
var wheelHandler = function(e) {
  console.log('wheel event 처리');
};
document.body.addEventListener('mousemove', debounce('move', moveHandler, 500));
document.body.addEventListener(
  'mousewheel',
  debounce('wheel', wheelHandler, 700)
);
```

### 커링 함수

커링 함수란 여러 개의 인자를 받는 함수를 한의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것을 말한다.

커링은 한 번에 하나의 인자만 전달하는 것을 원칙으로 한다. 또한, 중간 과정상의 함수를 실행한 결과는 그다음 인자를 받기 위해 대기만 할 뿐으로 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다.

```js
var curry3 = function(func) {
  return function(a) {
    return function(b) {
      return func(a, b);
    };
  };
};

var getMaxWith10 = curry3(Math.max)(10);
console.log(getMaxWith10(8)); // 10
console.log(getMaxWith10(25)); // 25

var getMinWith10 = curry3(Math.min)(10);
console.log(getMinWith10(8)); // 8
console.log(getMinWith10(25)); // 10
```

커링함수는 당장 필요한 정보만 받아서 전달하고 또 필요한 정보가 들어오면 전달하는 식으로 하면 결국 마지막 인자가 넘어갈 때까지 함수 실행을 미루는 셈이다. 
이를 함수형 프로그래밍에서는 지연실행이라고 칭한다. 
원하는 시점까지 지연시켰다가 실행하는 것이 용이한 상황이라면 커링을 쓰기에 적합하다.

최근 여러 프레임워크나 라이브러리 등에서 커링을 상당히 광범위하게 사용하고 있다. 

Flux 아키텍처의 구현체 중 하나인 Redux의 미들웨어를 예로 들면 logger와 thunk 둘 다 공통적으로 store, next, action 순서로 인자를 받는다. 
이 중 store는 프로젝트 내에서 한 번 생성된 이후로는 바뀌지 않는 속성이고, dispatch의 의미를 가지는 next 역시 마찬가지이다. 
action의 경우 매번 달라진다. 

결국, store와 next값이 결정되면 Redux 내부에서 logger 또는 thunk에 store, next를 미리 넘겨서 반환된 함수를 저장시켜놓고, 이후에는 action만 받아서 처리할 수 있게 한 것이다.


## 정리

클로저란 어떤 함수에서 선언한 변수를 참조하는 내부함수를 외부로 전달할 경우, 함수의 실행컨텍스트가 종료된 후에도 해당 변수가 사라지지 않는 현상.

내부함수를 외부로 전달하는 방법에는 함수를 return하는 경우만 아니라 콜백으로 전달하는 경우도 포함이다.

클로저는 그 본질이 메모리를 계속 차지하는 개념으로 더이상 사용하지 않게 된 클로저에 대해서는 메모리를 관리해주어야한다.
