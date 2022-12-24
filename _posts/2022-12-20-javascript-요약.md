---
title:  "[JavaScript] 바닐라 JS로 크롬 앱 만들기 (1)"
excerpt: "javascript"

categories:
  - JavaScript
tags:
  - [JavaScript]
  
toc: true
toc_sticky: true
 
date: 2022-12-20
last_modified_at: 2022-12-20
---



### 들어가기 전에

> - 이 글은 노마드코더의 '바닐라 JS로 크롬 앱 만들기' 강의를 통해 학습한 내용을 정리한 것입니다.
> - JavaScript의 특정 개념이나 주제에 대한 완전한 설명을 포함하고 있지 않습니다.



## HTML 파일에 CSS, JavaScript 연결하기

- 브라우저는 HTML을 열고, HTML은 CSS와 자바스크립트를 가져온다. HTML은 접착제와 같은 역할을 한다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- css 파일을 만들고 html 문서의 head에 행을 추가한다. -->
    <link rel="stylesheet" href="style.css"> 
    <title>Momentum App</title>
</head>
<body>
    <div class="hello">
        <h1>Grab me 1!</h1>
    </div>
    <!-- js 파일을 만들고 html 문서의 body에 행을 추가한다. -->
    <script src="app.js"></script>
</body>
</html>
```



- 브라우저 개발자 도구의 Elements 탭에서 HTML과 CSS를, Console 탭에서 JavaScript를 확인하거나 다룰 수 있다.
- JavaScript 코드를 수정하면, 연결된 HTML 파일을 실행하고 있는 브라우저를 새로고침하여 변경사항을 확인할 수 있다.



## 자료형과 변수

#### 자료형(data type)

- **숫자** 자료형에는 **정수형(integer)**과 **실수형(float)**이 있다.

  - ex)  1, 2, 3, .... / 1.5, 2.12, ...
  - JavaScript 코드를 통해 숫자의 연산 결과를 얻을 수 있다.
    - 1+1 입력시, 2를 반환

- **문자** 자료형은 큰 따옴표(**""**) 혹은 작은 따옴표(**''**)로 묶어서 표기하며, JavaScript에서 둘 사이에는 차이가 없다.

  - ex) "hello", 'javascript'
  - "hello" + "javascript" => "hello javascript"

- `boolean` 자료형은 `true` 혹은 `false`의 두 가지 값만을 갖는다.

- `undefined`는 '아무 값도 할당받지 않은 상태'를 나타낸다.

  - 변수를 선언했지만 값을 할당하지 않았다면 자동으로 `undefined`가 할당된다.

    ```javascript
    let age;
    console.log(age); // 'undefined'가 출력
    ```

  - `undefined`는 자바스크립트 엔진이 변수를 초기화할 때 사용하는 값이며, 어떤 변수에 값이 없음을 직접 나타내려면 `undefined`가 아닌 `null`을 할당한다.

- `null`은 '변수에 값이 없다는 것을 의도적으로 명시'할 때 사용한다. 

  - `null`은 자연적으로 발생하지 않는다.
  - 변수에 `null`을 할당하는 것은 변수가 이전에 참조하던 값을 더 이상 참조하지 않겠다는 의미이다.

- `typeof`연산자는 특정 변수의 자료형을 반환한다.

  - ex) typeof 정수형 > integer

#### 변수(variable)

- JavaScript에서는 변수를 선언할 때  `var`,  `const`와 `let`이라는 키워드를 사용한다.

  - `var`는 변수를 중복 선언할 수 있어 변수의 값이 예기치 못하게 변경될 수 있다는 등의 문제점이 있었고, 이를 보완하기 위해 ES6부터 `const`와 `let`이 추가되었다.

    ```javascript
    var name = 'Michael';
    
    var name = 'Jackson';
    ```

    

  - `const`는 변수의 재할당(값 변경)이 불가능하며, 선언과 동시에 초기화가 진행되어야 한다. 

    ```javascript
    const name; // 에러 발생: 초기화 필요
    
    const name = 'Michael';
    
    name = 'Jackson'// 에러 발생: 재할당 불가
    ```

    

  - `let`은 변수의 재할당이 가능하다.

    ```javascript
    let name = 'Michael';
    
    let name = 'Bruno'; // 에러 발생: 재선언 불가
    
    name = 'Jackson'; // 재할당 가능
    ```

    

  - 변수명에서 띄어쓰기가 필요한 부분은 영어 대문자로 표기한다(lowerCamelCase).

    ```javascript
    const longVariableName = "hello javascript";
    ```



- `const`를 주로, `let`은 가끔 사용하되, `var`은 사용하지 않는다.

## 배열과 객체

#### 배열(Array)

- 여러 데이터를 묶어서 나열하기 위해 배열을 사용한다.
- JavaScript에서 배열은 **요소(element)**들을 대괄호(**[ ]**)로 묶어서 나타내며, 이를 변수에 할당할 수 있다.
- 배열의 특정 요소는 '**배열의 이름[숫자]**'와 같은 형태로 나타낸다.

```javascript
const daysOfWeek = ["mon", "tue", "wed", ...];
console.log(daysOfWeek); // (7) ["mon", "tue", "wed", ...] 출력

const everything = [1, 2, "abc", true, null, undefined];
console.log(everything); // (6) [1, 2, "abc", true, null, undefined] 출력

console.log(everything[0]); // 1 출력
console.log(everything[9999]); // undefined 출력
```

- push 메서드를 통해 배열에 새로운 원소를 추가할 수 있다. '**배열의 이름.push(새로운 원소)**'의 형태로 코드를 작성한다.

```javascript
everything.push("new");

console.log(everything); // (7) [1, 2, "abc", true, null, undefined, "new"] 출력
```



#### 객체(Object)

- 여러 데이터를 하나로 묶되 각각의 의미, 성격을 명시하고 싶을 때 객체를 사용한다.
- JavaScript의 **객체**는 키(key)와 값(value)으로 구성된 **프로퍼티(property)**의 집합이다.
  -  키와 값은 콜론(**:**)으로 구분하고, 프로퍼티들은 쉼표(**,**)로 구분한다.
  - 객체의 이름이 A이고 프로퍼티의 키가 B일 때, '**A.B**' 혹은 '**A["B"]**'의 형태로 특정 프로퍼티를 지칭한다.
  - '**A.B = value**'와 같은 형태의 코드를 통해 프로퍼티를 생성하거나 수정할 수 있다.
  - JavaScript 객체는 존재하지 않는 프로퍼티에 접근해도 에러가 발생하지 않고 undefined를 반환한다.

```javascript
const player = {
    name: "Messi",
    backNumber: 10,
    isGOAT: true
};

console.log(player);
// {name: "Messi", backNumber: 10, isGOAT: true} 출력

console.log(player.name);
console.log(player["name"]);
// Messi 출력

player.name = "Lionel Messi";
player.team = "PSG";
console.log(player);
// {name: "Lionel Messi", backNumber: 10, isGOAT: true, team: "PSG"} 출력
```



## 함수(function)

- 함수는 특정 작업을 수행할 때 반복해서 사용할 수 있는 코드 조각이다.
- JavaScript에서 함수의 정의는 `function` 키워드로 시작하며, 함수는 다음과 같은 형태로 정의된다.
  - 매개변수는 정의된 함수의 내부에서만 존재한다.

```javascript
function 함수이름(매개변수1, 매개변수2, ...) {
    함수가 호출되었을 때 실행할 코드
}
```

```javascript
function sayHello() {
    console.log("Hello");
}

sayHello(); // Hello 출력

function sayHello(name) {
    console.log("Hello " + name);
}

sayHello("Michael"); // Hello Michael 출력

console.log(name); // 에러 발생: name이 정의되지 않음
```

- JavaScript의 함수는 객체이며 프로퍼티의 값으로 취급할 수 있다. 
  - 프로퍼티의 값이 함수일 경우, 일반 함수와 구분하기 위해 **메소드**라고 부른다.
  - 메소드는 '**메소드 이름: function(매개변수) {}**'의 형태로 정의한다.

```javascript
const player = {
    name: "Messi",
    backNumber: 10,
    isGOAT: true,
    sayHello: function (anotherName) {
        console.log("Hello " + anotherName)
    }
};

player.sayHello("Ronaldo"); // Hello Ronaldo 출력
```

- 함수의 실행 결과를 반환받기 위해 `return`을 사용한다.
  - 함수를 호출하는 구문을 해당 함수의 결과값으로 대체한다.

```javascript
function plus(a, b) {
    return a + b;
}

const sum = plus(2, 3);
console.log(sum); // 5 출력
```



## 조건문(conditionals)

- JavaScript의 조건문은 다음과 같은 형태로 작성한다.

```javascript
if (조건 A) {
    A가 참일 경우 실행할 코드
} else if (조건 B) {
    B가 참일 경우 실행할 코드
} else {
    조건 A와 B가 거짓일 경우 실행할 코드
}
```

- 또는(OR)은 '**||**', 그리고(AND)'는 '**&&**', '같다(EQUAL)'는 '**===**'으로 표기한다.

```javascript
const age = parseInt("11"); 
// parseInt는 자료형을 숫자로 변경. 변경이 불가하면 NaN을 반환. 
// 아래 isNaN()은 입력받은 인자가 NaN인지 판별하는 함수

if (isNaN(age) || age < 0) {
    console.log("Please write a positive number");
} else if (age < 18) {
    console.log("You are too young.");
} else if (age >= 18 && age < 50) {
    console.log("You can drink.");
} else {
    console.log("You should stop drinking.")
}
```





