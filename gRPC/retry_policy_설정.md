# gRPC RetryPolicy 설정

gRPC 클라이언트에서 연결 실패 시 정해진 횟수만 재시도하고 이후에는 더 이상 재시도하지 않도록 설정하려면, gRPC의 **retry policy**를 설정하여 제어할 수 있습니다.  
gRPC에서는 **retry**를 위한 설정을 클라이언트에 적용할 수 있으며, 이를 통해 재시도 횟수를 제한할 수 있습니다.

다음은 gRPC의 재시도 정책을 설정하는 방법입니다:

### 1. gRPC 클라이언트 설정에서 `service config` 사용
gRPC 클라이언트에서 서비스 구성(`service config`)을 통해 재시도 정책을 설정할 수 있습니다.

```json
{
  "methodConfig": [{
    "name": [{"service": "your_service_name"}],
    "retryPolicy": {
      "maxAttempts": 3,
      "initialBackoff": "0.1s",
      "maxBackoff": "1s",
      "backoffMultiplier": 2,
      "retryableStatusCodes": ["UNAVAILABLE"]
    }
  }]
}
```

위 설정에서 중요한 부분은 `maxAttempts`로, 이는 최대 재시도 횟수를 설정합니다.  
이 값이 초과되면 더 이상 재시도하지 않고 예외를 던지게 됩니다. 예를 들어, `maxAttempts`를 3으로 설정하면 최대 2번 재시도하고 3번째 시도에서는 예외가 발생합니다.

### 2. **Node.js gRPC 클라이언트에서 재시도 설정**

Node.js에서 사용하는 gRPC는 서비스 구성(`service config`)을 통해 재시도 정책을 설정할 수 있습니다.  
Node.js에서 gRPC 클라이언트를 사용할 경우, `grpc` 또는 `@grpc/grpc-js` 패키지를 사용하게 됩니다.  
`grpc-js`를 사용하여 재시도 정책을 설정하는 방법은 다음과 같습니다.

#### 재시도 정책 설정 방법 (Node.js):

```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');
const path = require('path');

// .proto 파일 로드
const PROTO_PATH = path.resolve(__dirname, './your_service.proto');
const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true
});
const your_proto = grpc.loadPackageDefinition(packageDefinition).your_service_name;

// gRPC 클라이언트 생성 시 재시도 설정
const client = new your_proto.YourService(
  'localhost:50051',
  grpc.credentials.createInsecure(),
  {
    'grpc.service_config': JSON.stringify({
      "methodConfig": [{
        "name": [{ "service": "your_service_name.YourService" }],
        "retryPolicy": {
          "maxAttempts": 3,
          "initialBackoff": "0.1s",
          "maxBackoff": "1s",
          "backoffMultiplier": 2,
          "retryableStatusCodes": [grpc.status.UNAVAILABLE]
        }
      }]
    })
  }
);

// gRPC 호출
client.yourMethod({ yourParam: 'value' }, (error, response) => {
  if (error) {
    console.error('Error:', error);
  } else {
    console.log('Response:', response);
  }
});
```

### 3. **설정 설명**
- `maxAttempts`: 최대 시도 횟수를 설정합니다. 예를 들어, `3`으로 설정하면 최초 시도를 포함하여 최대 3번 요청을 시도합니다.
- `initialBackoff`: 재시도 간 대기 시간을 설정합니다. 첫 번째 재시도는 0.1초 뒤에 시도됩니다.
- `maxBackoff`: 최대 대기 시간을 설정합니다. 백오프(backoff) 간격이 점점 늘어날 때 이 값을 넘지 않도록 제한합니다.
- `backoffMultiplier`: 대기 시간이 늘어나는 배수를 설정합니다. 예를 들어, 첫 번째 대기 시간이 0.1초이면 두 번째는 0.2초, 세 번째는 0.4초로 증가합니다.
- `retryableStatusCodes`: 재시도 가능한 gRPC 상태 코드를 설정합니다. 여기서는 `grpc.status.UNAVAILABLE` 상태일 때 재시도하도록 설정했습니다.

