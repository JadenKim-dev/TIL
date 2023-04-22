take를 활용해서 비동기 데이터들을 꺼내오는 코드는 아래와 같다.

```js
go(
  [1, 2, 3],
  L.map((a) => Promise.resolve(a + 10)),
  take(2),
  log
);
```

비동기 데이터를 꺼내올 수 있는 take는 아래와 같이 구현할 수 있다.

```js
const take = curry((l, iter) => {
  let res = [];
  iter = iter[Symbol.iterator]();
  return (function recur() {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      if (a instanceof Promise) {
        return a
          .then((a) => ((res.push(a), res).length == l ? res : recur()))
          .catch((e) => (e == nop ? recur() : Promise.reject(e)));
      }
      res.push(a);
      if (res.length == l) return res;
    }
    return res;
  })();
});
```

즉시 실행 함수인 recur 안에서는 iter를 하나씩 돌면서 Promise인 경우에는 then으로 값을 꺼내고,
그 안에서 recur를 재귀 호출한다.

여기서 헷갈렸던 점은 recur 안에서 Promise를 반환하는 경우가 있다는 것이다.

크기가 다 찬 경우에는 하나씩 데이터를 담은 res를 반환하는데, 어떤 경우는 Promise를 반환하고 있다는 점이 이상하다고 느껴졌다

go 메서드의 경우는 다음과 같이 reduce를 활용해서 구현되어있다.

```js
const go = (...args) => reduce((a, f) => f(a), args);
```

비동기를 처리할 수 있는 reduce 함수는 아래와 같이 구현된다.

```js
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  return go1(acc, function recur(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});
```

위와 같이 acc가 Promise인 경우 acc.then(recur)로 then으로 프로미스 안의 값을 꺼내서 처리하고 있다.

이 때 프로미스의 경우 한 번만 then을 호출해도 그 이전의 비동기 처리가 모두 끝난 상태의 값이 나오기 때문에, take에서 모두 처리가 끝난 상태의 결과를 받을 수 있다.
