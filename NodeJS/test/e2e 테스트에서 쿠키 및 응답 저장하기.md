# e2e 테스트에서 쿠키 및 응답 저장하기

## 1. 문제 상황

e2e 테스트를 작성하다 보면 로그인 API를 통해 사용자 인증을 하고, 해당 인증을 유지하기 위해 쿠키나 토큰 등을 저장해야 하는 상황이 종종 발생한다.

## 2. 사전 요청 보내고, 데이터 추출하기

요청을 보내고 응답을 이용하여 토큰 등의 값을 저장하는 코드는 아래와 같이 작성할 수 있다.

```ts
const loginResponse = await request(app.getHttpServer())
  .post("/api/auth/login")
  .send({
    email: "user@google.com",
    password: "password1234",
  });

const accessToken = loginResponse.body.accessToken;
const cookie = loginResponse.get("Set-Cookie");
```

다른 테스트 요청을 보낼 때와 마찬가지로 `request(app.getHttpServer()).post` 를 통해 서버에 사전 요청을 보낸다.  
await을 이용하면 그 응답 결과를 Response 형태로 받을 수 있다.  
response.body를 이용하면 응답 body에 접근 가능하고, response.get(key)를 통해 key에 해당하는 header 값에 접근할 수 있다.

## 3. 이후 요청에 사용하기

위에서 저장한 값을 아래와 같이 요청에서 사용할 수 있다.  
요청에 쿠키 값을 넣고, 받아온 access token 값을 Authorization 헤더를 추가한다.

```ts
request(app.getHttpServer())
  .post("/api/auth/refresh")
  .set("Cookie", cookie)
  .set("Authorization", `Bearer ${accessToken}`)
  .expect(HttpStatus.CREATED);
```
