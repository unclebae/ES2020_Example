# ES2020 에서 달라진점.

ES2020 에서도 몇가지 중요한 기능이 추가 되었습니다.

나날이 자바스크립트의 기능은 더 편리하고 확장되어 가는것을 보니 역시 탑을 유지하는 언어중에 하나라고 생각이 듭니다.

이번에는 ES2020에서 새로 추가된 기능들을 살펴보도록 하겠습니다 .

## 동적 잎포트

동적 임포트는 기존의 정적 임포트 방식에서 좀더 함수 스타일의 프로그래밍을 수행할 수 있도록 도와줍니다.

필요없는 모듈을 로드하지 않아도 되는 이점도 함께 제공합니다.

```
<script>
  const greeting_module = "./greeting.js";

// Promise 를 이용한 방식
  import(greeting_module)
    .then(module => {
      module.default();
      module.greeting("kido");
    })
    .catch(error => {
      console.log(error);
    });

// async/await 를 이용한 방식
  (async function () {
    const greeting_module2 = "./greeting.js";
    const module = await import(greeting_module2);
    module.default();
    module.greeting("kido");
  })();
</script>
```

위 내용을 보면 ES2020 은 import 를 함수처럼 이용할 수 있으며, 이용할때 Promise 를 사용한다는 것을 확인할 수 있습니다.

참고로 다음과 같이 사용할 수 없습니다.

`const greeting = import(...)`

이렇게 이용하려면 await 를 이용해야합니다.

정말 멋진것은 다음과 같이 동적으로 필요한 모듈을 프로그램처럼 로드할 수 있습니다.

```
<script>
  async function loadModules(firstValue, fileName) {
    const module = await import(`./${firstValue}_${fileName}.js`);
    module.loading();
  }

  const firstValue = 'prefix';
  const fileName = 'filename';

  loadModules(firstValue, fileName);
</script>
```

## BigInt

BigInt 는 arbitrary-precision 정수형을 제공합니다.

이는 기존 Number.MAX_SAFE_INTEGER; 의 이상동작이 전혀 없이 연산을 할 수 있습니다.

```
<script>
    console.log("Normal Number ----------------")
    var max = Number.MAX_VALUE;
    console.log("Max: ", max)
    console.log("Max + 1: ", max + 1)
    console.log("Max + 2: ", max + 2)

    var max_safe = Number.MAX_SAFE_INTEGER;
    console.log("Max Safe: ", max_safe)
    console.log("Max Safe + 1: ", max_safe + 1)
    console.log("Max Safe + 2: ", max_safe + 2)
</script>

<script>
    console.log("BigInt ----------------")
    var bigInt = BigInt(Number.MAX_SAFE_INTEGER);
    console.log("bigInt: ", bigInt)
    console.log("bigInt + 1n: ", bigInt + 1n)
    console.log("bigInt + 2n: ", bigInt + 2n)
    console.log("bigInt + 2: ", bigInt + 2)
</script>

<script>
    console.log("typeof ----------------")
    console.log("typeof(max_safe) ", typeof (max_safe))
    console.log("typeof(bigInt) ", typeof (bigInt))

    console.log("operation ----------------")
    console.log("Op 01: (BitInt(100) == 100n): ", (BigInt(100) == 100n))
    console.log("Op 02: (100 == 100n): ", (100 == 100n))
    console.log("Op 03: (0n == true): ", (0n == true))
    console.log("Op 04: (10n + 5n + (2n * 5n)): ", (10n + 5n + (2n * 5n)))
    console.log("Op 05: BigInt(5.1): ", BigInt(5.1))
</script>

<script>
    console.log("Op 06: BigInt(Number.MAX_SAFE_INTEGER) + 1.5: ", BigInt(Number.MAX_SAFE_INTEGER) + 1.5)
</script>
```

![bigint](./bigint.png)

BigInt 는 일반적인 Number 타입이 아니기 때문에 함게 연산할 수 없습니다.

## Promise.allSettled

Promise.all 은 여러개의 promise 가 수행되며, response 혹은 reject 결과 한가지만 노출하는 반면, Promise.allSettled 는 각각 처리 결과에 대한 리스트를 반환합니다.

```
<script>
    const pr1 = new Promise(resolve => resolve('first promise'));
    const pr2 = new Promise((_, reject) => reject('reject your request'))
    const pr3 = new Promise(resolve => resolve("Success "))

    console.log("------------ promise all -------------")
    Promise.all([pr1, pr2, pr3])
        .then(response => console.log(response))
        .catch(error => console.log(error));
</script>

<script>
    console.log("------------ promise allSettled ----------")
    Promise.allSettled([pr1, pr2, pr3])
        .then(response => console.log(response));
</script>
```

![promise](./promise.png)

## globalThis

자바스크립트에서 global 값에 접근하기 위해서 각 실행 환경에 따라 다양한 객체에 접근해야 했습니다.

이제는 globalThis 를 이용하여 글로벌 값에 접근할 수 있습니다.

node 에서는

globalThis === global 입니다 .

browser 에서는

globalThis === window 입니다.

이를 실험한 코드는 다음과 같습니다 .

```
// A naive attempt at getting the global `this`. Don’t use this!
const getGlobalThis = () => {
  if (typeof globalThis !== 'undefined') return globalThis;
  if (typeof self !== 'undefined') return self;
  if (typeof window !== 'undefined') return window;
  if (typeof global !== 'undefined') return global;
  // Note: this might still return the wrong result!
  if (typeof this !== 'undefined') return this;
  throw new Error('Unable to locate global `this`');
};
const theGlobalThis = getGlobalThis();
```

from: https://v8.dev/features/globalthis

## String.prototype.matchAll

regular expression 는 효과적인 개발을 위해서 꼭 필요한 도구가 아닐 수 없습니다.

그러나 사용하면서 가끔식은 배열요소들중 매치되는 사항들을 한번에 추출하고 싶은데 장황한 코드가 필요 했었습니다.

이럴때 matchAll 을 이용하여 배열 중 매치되는 요소들만 가져올 수 있습니다.

```
<script>
    console.log("------ 정규식 표현으로 전체가 대문자인 문자만 추출합니다.")
    const string = 'Magic hex numbers: DEADBEEF CAFE';

    //  \b 는 단어의 경계(시작/끝)를 확인합니다. 
    //  \p POXIX 문자 클래스를 이용한다. 이는 US-ASCII 에서만 사용가능  
    // {ASCII_Hex_Digit} 는 아스키, 16진수, 숫자등을 검출한다. [0-9a-fA-F] 내용을 찾을 것입니다. 
    // + 는 1번이상 반복됨을 나타냅니다. 
    //  \b 다음 공백을 찾는다. 
    //  g 는 글로벌의 의미로 대상 문자 모두에 대해 적용합니다. 
    //  u 는 유니코드를 지원합니다. 
    const regex = /\b\p{ASCII_Hex_Digit}+\b/gu;
    for (const match of string.match(regex)) {
        console.log(match);
    }
</script>
<script>
    console.log("------ 매치되는 데이터, 인덱스, 입력된 문자도 노출한다. ")

    const regex2 = /\b\p{ASCII_Hex_Digit}+\b/gu;
    let match;
    while (match = regex2.exec(string)) {
        console.log(match);
    }
</script>
<script>
    console.log("------ 매치되는 데이터, 인덱스, 입력된 문자도 노출한다. 단 matchAll 을 이용한다. ")

    const regex3 = /\b\p{ASCII_Hex_Digit}+\b/gu;
    for (const match of string.matchAll(regex3)) {
        console.log(match);
    }
</script>
<script>
    console.log("------ 매치되는 데이터, 인덱스, 입력된 문자도 노출한다. 단 matchAll 을 이용한다. 그리고 캡쳐를 이용한다.")

    // 단어를 구분하고 \b 이용
    // (?<owner>...) ... 에 매치되는 결과를 owner 이라는 이름으로 캡쳐한다. 
    const regex4 = /\b(?<owner>[a-z0-9]+)\/(?<repo>[a-z0-9\.]+)\b/g;
    for (const match of string.matchAll(regex4)) {
        console.log(`${match[0]} at ${match.index} with '${match.input}'`);
        console.log(`→ owner: ${match.groups.owner}`); //  캡쳐된 단어를 이용한다. 
        console.log(`→ repo: ${match.groups.repo}`); //  캡쳐된 단어를 이용한다. 
    }
</script>
```

from : https://v8.dev/features/string-matchall

에서 소스를 가져 왔으며, 해당 내역마다 주석을 첨부 하였습니다. 

결과를 한번 보겠습니다. 

![regex](./regex.png)

## 결론

지금까지 ES2020 에서 새로 추가된 사항들중에 의미있게 변경된 몇가지를 살펴 보았습니다.

Javascript 를 사용하면서 불편했던 사항들이 개선되면서, 간단한 코드로 훌륭한 작업을 해 낼 수 있는것 같습니다 .