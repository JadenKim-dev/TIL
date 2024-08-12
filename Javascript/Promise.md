## 1) 프로미스의 동작 원리

Promise는 비동기 작업을 처리하기 위한 객체로, 주로 비동기 작업의 결과를 처리하는데 사용됩니다.  
내부적으로 `Promise`는 아래와 같은 방식으로 동작합니다

### 1. **Promise 객체 생성**

`Promise` 객체는 생성 시점에 **executor** 함수(즉시 실행 함수)를 받습니다.  
이 함수는 두 개의 매개변수 `resolve`와 `reject`를 받습니다.  
`executor` 함수는 비동기 작업을 수행하고, 작업이 성공하면 `resolve`를, 실패하면 `reject`를 호출합니다.
   
   ```javascript
   const promise = new Promise((resolve, reject) => {
       // 비동기 작업 수행
       if (/* 작업 성공 */) {
           resolve(value);  // 작업 성공, value는 성공 결과
       } else {
           reject(error);  // 작업 실패, error는 오류 정보
       }
   });
   ```

### 2. **상태 관리**

`Promise`는 **pending(대기)**, **fulfilled(성공)**, **rejected(실패)**의 세 가지 상태를 가집니다.  
초기 상태는 `pending`이며, `resolve`가 호출되면 `fulfilled`, `reject`가 호출되면 `rejected` 상태로 전환됩니다.  
상태가 `fulfilled` 또는 `rejected`로 바뀌면, 그 상태는 변하지 않고 고정됩니다(immutable).

### 3. **then과 catch 체인**

`Promise`가 `fulfilled` 상태가 되면, `then` 메서드에 전달된 첫 번째 콜백 함수가 실행됩니다.  
이 함수는 `resolve`로 전달된 값을 인자로 받습니다.

`Promise`가 `rejected` 상태가 되면, `catch` 메서드나 `then` 메서드의 두 번째 콜백 함수가 실행됩니다.  
이 함수는 `reject`로 전달된 오류를 인자로 받습니다.  
`then`이나 `catch`는 새로운 `Promise`를 반환하기 때문에, 체이닝을 통해 연속적인 비동기 작업을 처리할 수 있습니다.
   
   ```javascript
   promise
     .then(result => {
         console.log(result);  // 성공 결과 처리
         return anotherPromise;
     })
     .catch(error => {
         console.error(error);  // 오류 처리
     });
   ```

### 4. **비동기 작업 처리**

`Promise`는 비동기 작업의 결과를 기다리기 때문에, 동기적으로 코드가 실행되지 않습니다.  
비동기 작업이 완료되면, `Promise`가 `fulfilled` 또는 `rejected` 상태로 전환되며, 등록된 콜백이 호출됩니다.  
콜백은 이벤트 루프가 돌아가는 과정에서 다음 작업 큐에 의해 실행됩니다.

### 5. **마이크로태스크 큐**

`Promise`의 콜백은 자바스크립트의 이벤트 루프에서 **마이크로태스크 큐**에 추가됩니다.  
마이크로태스크 큐는 일반 태스크 큐보다 우선순위가 높아, 현재 실행 중인 작업이 완료되면 즉시 실행됩니다.

## 2) async - await의 동작 원리

`async`와 `await`는 JavaScript에서 비동기 코드를 동기 코드처럼 작성할 수 있게 도와주는 문법입니다.  
이들은 `Promise`를 기반으로 동작하며, 내부적으로는 JavaScript 엔진이 비동기 작업을 처리하는 방식에 대한 추상화를 제공합니다.

### 1. **`async` 함수**
`async` 키워드는 함수 앞에 붙어, 해당 함수가 비동기 함수임을 나타냅니다.  

`async` 함수는 항상 `Promise`를 반환합니다.  
명시적으로 `Promise`를 반환하지 않아도, 함수 내에서 반환된 값은 자동으로 `Promise.resolve()`로 감싸집니다.

   ```javascript
   async function example() {
       return "Hello";  // 실제로는 Promise.resolve("Hello")를 반환
   }
   ```

### 2. **`await` 키워드**
`await`는 `Promise`가 처리될 때까지 함수의 실행을 일시적으로 중단(블록킹)하고, `Promise`가 완료되면 그 결과를 반환합니다.  
`await`는 오직 `async` 함수 내에서만 사용할 수 있습니다.  
`await`는 `Promise`가 `fulfilled` 상태가 되면 그 값을 반환하고, `rejected` 상태가 되면 예외를 발생시킵니다.

   ```javascript
   async function example() {
       const result = await someAsyncFunction();
       console.log(result);
   }
   ```

### 3. **내부 동작**

**`async` 함수 호출 시점**:  
`async` 함수가 호출되면 즉시 `Promise`를 반환합니다.  
이때 함수 내부의 코드는 동기적으로 실행되지만, `await`가 등장하는 순간 비동기적으로 처리됩니다.
   
**`await`의 동작**:  
`await`는 `Promise`가 처리될 때까지 해당 줄에서 실행을 멈춥니다.  
이때 JavaScript 엔진은 `await` 표현식을 만났을 때, 함수의 실행을 중단하고, 마이크로태스크 큐에 해당 작업을 추가합니다.  
`Promise`가 처리되면 `await`는 그 결과값을 반환하고, 함수의 나머지 부분이 실행됩니다.
   
**마이크로태스크 큐와 이벤트 루프**:

`await`가 호출된 후, JavaScript 엔진은 다른 작업(예: 이벤트 핸들링, 다른 스크립트 실행)을 계속 처리합니다.  
`Promise`가 완료되면 그 결과는 마이크로태스크 큐에 추가되고, 이벤트 루프는 큐에 있는 다른 작업이 끝난 후 해당 작업을 처리합니다.

### 4. **예외 처리**

`await`는 `Promise`가 `rejected` 상태가 되면 해당 예외를 `throw`합니다.  
이 예외는 `try...catch` 블록을 사용해 처리할 수 있습니다.

   ```javascript
   async function example() {
       try {
           const result = await someAsyncFunction();
           console.log(result);
       } catch (error) {
           console.error('Error:', error);
       }
   }
   ```

### 5. **비동기 흐름 제어**
`async/await`는 `Promise` 체인보다 더 직관적으로 비동기 작업의 흐름을 제어할 수 있게 해줍니다.  
코드를 동기적으로 읽을 수 있게 되므로 가독성이 향상됩니다.
