---
title: "[javascript] 클로저(closure)에 대해서 알아보자"
date: 2017-04-12 00:20:04
tags:
- javascript
- closure
---
자바스크립트 문법 중에 가장 어려운 부분을 꼽으라면 단연 클로저(closure)일것입니다. 저 또한 클로저 개념은 어느정도는 이해하고 있었지만, 정확한 용도와 개념을 설명하라고 하면 명쾌하게 말하기 쉽지 않습니다. 아마도 많은 사람들이 저 처럼 대충 클로저가 무언인지는 말할 수 있지만, 정확한 의미와 용도에 대해서는 쉽고 명확히 대답하기는 힘들것입니다. 제가 클로저 개념이 헷갈렸던 이유는 의외로 황당한 이유 때문이었습니다.

> ~~closer (가까운, 닫힌)~~ ==> **closure (폐쇄)**

멍청하게도 처음에는 클로저를 "closer"라고 생각했습니다. 정확한 개념은 당연히 몰랐고 단어의 의미로 단순하게 유추해서 "어떤 것을 닫는다" 정도로 느끼고 있었죠. 당시의 "닫는다"는 의미를 지금에 와서 생각해보면 "변수의 범주(스코프)를 닫는다" 정도로 이해하고 있었던거 같습니다. 하나도 모르고 있었다고해도 과언이 아니었죠. 

구글에서 검색하면 알 수 있는 클로저의 의미는 아래와 같이 조금 모호합니다.

> 외부함수의 맥락(context)에 접근 가능한 내부함수
> 좀 더 포괄적으로는 함수 선언시 생성되는 유효 범위

이런 정의만 보고 과연 사람들이 이해를 할 수 있는건지는 잘 모르겠지만, 클로저에 대한 이해가 거의 없는 분들은 아마도 이해하기 어렵울 것입니다. 그러면, 일단 코드를 보도록 하죠.

```
function outFunc(name) {
	var outVar = "my name is ";
	function innerFunc() {
		return outVar + name;
	}
	return innerFunc;
}

var result = outFunc("bono");
console.log("result: " + result());

// result: my name is bono
```

내부함수 `innerFunc()`에서 `outFunc()` 함수의 인자와 지역변수에 접근이 가능합니다. `outFunc()`의 return 값(var result에 할당)은 `innerFunc()`라는 내부 함수입니다. outFunce() 함수가 실행되면, outFunc()의 스코프는 끝이 나기 때문에 outFunc() 인자인 name과 지역변수인 outVar는 메모리에서 정리되어야합니다. 하지만, 실제 console.log에서 result를 호출하면(내부 함수가 호출), 내부함수 innerFunc()가 선언될때 outFunc() 함수의 인자와 outVar() 지역변수를 innerFunc()의 클로저 객체로 남아 실제로 innerFunc()가 호출될 때 클로저 객체를 통해서 outFunc()의 인자와 변수에 접근이 가능한 것입니다. 이게 바로 클로저가 하는 일입니다. 

다른 예제를 살펴보겠습니다.

```
var out = "out value";

function outFunc() {
	var inner = "in value";

	function inFunc(inParam) {
		console.log("out: " + out);
		console.log("inner: " + inner);
		console.log("inParam: " + inParam);
	}

	return inFunc;
}

var param = "this is param";
var outResult = outFunc();
outResult(param);

// out: out value
// inner: in value
// inParam: this is param

```

이 예제에는 크게 세가지 스코프가 존재합니다. 첫번째는 `전역스코프`, 그다음은 `outFunc()` 함수 내 스코프, 마지막으로 `inFunc()` 내 스코프입니다. 가장 위에 out 이라는 변수가 선언되어 있고, outFunc() 함수 및 param과 outFunc() 의 return 값인 outResult까지 총 4개의 변수(or 함수)가 선언되어 있고 마지막에 outResult 함수를 호출하고 있습니다. 

outResult는 outFunc() 함수의 결과값이므로, inFunc() 함수 자체를 참조하고 있습니다. 그 말은 마지막에 호출한 outResult 함수에 인자를 전달하면 실제 내부 함수인 inFunc()의 파라미터에 해당 값이 들어온다는 의미겠죠. 

클로저의 관점에서 생각해보겠습니다. outFunc() 함수가 선언되었지만, 실제로 호출되기전까진 언제 사용될지 모릅니다. 그래서 해당 함수(outFunc())의 클로저로써 유효범위(전역범위)의 변수들이 클로저 객체로 메모리상에 남아 있게 됩니다. 즉, outFunc() 함수가 실행될 때 해당 함수 내부에서 outFunc() 바깥의 전역영역의 변수에 접근할 수 있는거죠. 그리고 outFunc() 내부에 inFunc()가 선언되는 순간 outFunc() 내의 변수(여기서는 inner 변수)가 inFunc() 함수의 클로저 객체 안에 존재하게 되는것이죠. 그러고나면 각각의 outFunc(), inFunc() 함수가 실제로 호출되어 실행되는 순간에 미리 `메모리에 저장되어 있던 클로저`에서 각각의 변수를 가져올수 있게 되는겁니다.

클로저의 정의에 대해서 알아보았으니 클로저로 활용할 수 있는게 뭐가 있는지는 다음에 알아보도록 하겠습니다.

### 결론

> 클로저는 단순히 함수 외부의 변수에 접근 가능한 내부함수가 아니라 함수가 선언되는 순간에 함수가 실행될때 실제 외부변수에 접근하기 위한 객체이다.
> 클로저도 남발하면 위험하다. 가비지컬렉션 대상이 되어야할 객체들이 메모리상에 남아 있게 되므로, 클로저를 남발하면 오버플로우가 발생할수도 있다. 이는 클로저에 대해 정확히 알아야 하는 이유이기도 하다.
