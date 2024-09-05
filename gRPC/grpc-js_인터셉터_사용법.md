# grpc-js 클라이언트에 인터셉터 적용하는 방법

`grpc-js`에서 클라이언트에 인터셉터를 적용하는 방법은 클라이언트 호출 시 중간에 로직을 삽입하여 요청을 처리하거나 응답을 가로채기 위해 사용됩니다.  
`grpc-js`에서는 미들웨어와 유사하게 인터셉터를 설정할 수 있습니다.  
인터셉터는 일반적으로 비동기 작업, 로깅, 인증, 재시도 등의 목적으로 사용됩니다.

### 1. `grpc.InterceptingCall`을 사용한 간단한 클라이언트 인터셉터 구현

먼저, 인터셉터는 호출 시 특정 로직을 가로채도록 설계됩니다.  
`grpc-js`에서 인터셉터는 함수 형태로 정의되어 클라이언트 메서드 호출 전후에 실행됩니다.

#### 예제

```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

// 프로토콜 버퍼 파일을 로드
const packageDefinition = protoLoader.loadSync('your_proto_file.proto', {});
const yourProto = grpc.loadPackageDefinition(packageDefinition);

// 클라이언트 생성
const client = new yourProto.YourService('localhost:50051', grpc.credentials.createInsecure());

// 인터셉터 함수 정의
function clientInterceptor(options, nextCall) {
  return new grpc.InterceptingCall(nextCall(options), {
    start: function (metadata, listener, next) {
      // 요청 전에 실행할 로직 (예: 로깅, 인증 추가)
      console.log('Request started');
      // 메타데이터 수정 가능 (예: 인증 토큰 추가)
      metadata.add('Authorization', 'Bearer your-token');

      // 다음 인터셉터로 전달
      next(metadata, {
        onReceiveMetadata: function (metadata, next) {
          // 응답 메타데이터 가로채기
          console.log('Response metadata:', metadata);
          next(metadata);
        },
        onReceiveMessage: function (message, next) {
          // 응답 메시지 가로채기
          console.log('Response message:', message);
          next(message);
        },
        onReceiveStatus: function (status, next) {
          // 최종 응답 상태 가로채기
          console.log('Response status:', status);
          next(status);
        },
      });
    },
  });
}

// 인터셉터가 적용된 클라이언트 호출
const callOptions = {
  interceptors: [clientInterceptor], // 인터셉터 추가
};

client.YourMethod(request, callOptions, (error, response) => {
  if (error) {
    console.error('Error:', error);
  } else {
    console.log('Response:', response);
  }
});
```

### 주요 포인트:
1. **`clientInterceptor`**: 클라이언트의 요청과 응답을 가로채서 원하는 로직을 추가할 수 있습니다.
2. **`metadata.add`**: 요청 메타데이터에 값을 추가하는 예시로, 토큰 등을 추가할 때 사용합니다.
3. **`onReceiveMessage`, `onReceiveStatus`**: 응답 메시지와 상태를 가로채서 필요한 작업을 수행할 수 있습니다.

## clientInterceptor가 받는 매개변수들

`grpc-js`에서 클라이언트 인터셉터(`clientInterceptor`)는 요청과 응답 사이에 중간 로직을 추가하거나 가로채는 기능을 제공합니다.  
인터셉터는 여러 매개변수를 통해 클라이언트 호출의 다양한 단계에서 개입할 수 있습니다.  

### 1. **`options`**
- **설명**: 
  클라이언트 호출에 대한 옵션을 나타내며, RPC 호출을 설정하는 데 사용됩니다.  
  여기에는 서비스 메서드 이름, 호출 메타데이터, 콜백 함수 등 호출에 필요한 정보가 포함될 수 있습니다.
- **역할**: 이 매개변수는 호출 메서드에 대한 정보를 담고 있으며, 이 정보를 바탕으로 클라이언트 호출을 조작할 수 있습니다.
- **예시**: 메타데이터를 수정하거나 특정 호출 메서드에 따라 다른 로직을 적용할 수 있습니다.

### 2. **`nextCall`**
- **설명**: 
  `nextCall`은 다음으로 이어질 클라이언트 호출을 의미합니다.  
  즉, 다음 단계로 RPC 요청을 전달하기 위한 메서드입니다.
- **역할**: 이 매개변수를 통해 인터셉터는 호출을 계속 이어가거나, 특정 조건에 따라 중단할 수 있습니다.
- **예시**: 인터셉터에서 메타데이터를 수정한 후 `nextCall`을 호출하여 다음 단계로 요청을 넘깁니다.
   
   ```javascript
   return new grpc.InterceptingCall(nextCall(options), {
     // 추가 로직
   });
   ```

### 3. **`metadata`**

- **설명**: 
  RPC 호출에 사용되는 메타데이터 객체로, 클라이언트가 서버에 요청을 보낼 때 필요한 추가적인 정보(헤더 등)를 담고 있습니다.
- **역할**: 
  이 매개변수는 서버로 보내는 요청에 포함될 메타데이터를 수정하거나 추가하는 데 사용됩니다.  
  예를 들어, 인증 토큰을 추가하거나 특정 클라이언트 식별 정보를 메타데이터에 포함할 수 있습니다.
- **예시**:
     ```javascript
     metadata.add('Authorization', 'Bearer your-token');
     ```

### 4. **`listener`**

- **설명**: 
  RPC 응답을 처리하는 함수들을 담고 있는 객체입니다.  
  응답 메타데이터, 메시지, 상태를 처리하는 이벤트 리스너로 구성되어 있습니다.
- **역할**: 
  응답을 가로채서 원하는 로직을 추가하거나 응답을 수정할 수 있도록 합니다.  
  응답 메시지를 가로채서 로깅하거나, 최종 응답 상태를 확인할 수 있습니다.

- **주요 메서드**:
  - **`onReceiveMetadata`**: 서버에서 보내온 메타데이터를 받을 때 호출되는 함수.
  - **`onReceiveMessage`**: 서버에서 보내온 메시지를 받을 때 호출되는 함수.
  - **`onReceiveStatus`**: 최종적으로 서버 응답 상태를 받을 때 호출되는 함수.

- **예시**:
     ```javascript
     next(metadata, {
       onReceiveMetadata: function (metadata, next) {
         console.log('Response metadata:', metadata);
         next(metadata);
       },
       onReceiveMessage: function (message, next) {
         console.log('Response message:', message);
         next(message);
       },
       onReceiveStatus: function (status, next) {
         console.log('Response status:', status);
         next(status);
       }
     });
     ```

### 5. **`next`**
- **설명**:
  이 매개변수는 다음 단계로 요청을 넘기기 위한 콜백 함수입니다.  
  인터셉터에서 메타데이터, 응답 메시지, 응답 상태 등을 가로챈 후 이를 다음으로 전달하는 역할을 합니다.
- **역할**:
  메타데이터나 응답을 처리한 후 다음으로 넘기기 위해 호출됩니다.  
  호출하지 않으면 RPC 호출이 중단됩니다.
- **예시**:
     ```javascript
     next(metadata);
     ```