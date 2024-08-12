# grpc-node의 ChannelOption 조정

### 이슈

다음과 같은 에러메시지가 발생하면서 Node 앱이 죽는 문제

```
2024-08-12T06:40:02.752Z Error: 14 UNAVAILABLE: read ECONNRESET
```

### 해결 방법

grpc의 Channel Option을 조정

```ts
const channelOptions: ChannelOptions = {
  // Send keepalive pings every 10 seconds, default is 2 hours.
  "grpc.keepalive_time_ms": 10 * 1000,
  // Keepalive ping timeout after 5 seconds, default is 20 seconds.
  "grpc.keepalive_timeout_ms": 5 * 1000,
  // Allow keepalive pings when there are no gRPC calls.
  "grpc.keepalive_permit_without_calls": 1,
};
```

다음의 링크를 참고  
https://github.com/grpc/grpc-node/issues/1907#issuecomment-1399670912

## `grpc.keepalive_permit_without_calls`

`grpc.keepalive_permit_without_calls` 옵션은 gRPC에서 **keepalive ping**을 보내는 동작을 제어하는 설정 중 하나입니다.

이 옵션은 클라이언트 측에서 사용되며, 서버와의 연결이 현재 활성 호출 없이도 유지되어야 하는지 여부를 결정합니다.

### **Keepalive Ping이란?**

Keepalive Ping은 클라이언트가 서버에 주기적으로 ‘ping’을 보내서 연결이 여전히 활성 상태인지 확인하는 메커니즘입니다.

이는 네트워크 문제로 인해 연결이 끊어졌는지 확인하고, 필요한 경우 연결을 복구하는 데 유용합니다.

### **grpc.keepalive_permit_without_calls의 역할**

- **true로 설정된 경우**: 클라이언트는 활성 gRPC 호출이 없더라도 서버에 keepalive ping을 보낼 수 있습니다. 즉, 현재 요청이 없더라도 클라이언트는 서버와의 연결을 유지하기 위해 주기적으로 ping을 보내게 됩니다.
- **false로 설정된 경우** (기본값): 클라이언트는 활성 gRPC 호출이 없으면 서버에 keepalive ping을 보내지 않습니다. 즉, 연결을 유지하기 위해 ping을 보내는 작업은 활성 호출이 있을 때만 이루어집니다.

### **언제 사용해야 할까?**

- **true로 설정**: 서버와의 연결 상태를 지속적으로 확인해야 하거나, 빈번한 연결 재설정을 피하고 싶은 경우 유용합니다. 예를 들어, 클라이언트와 서버 간의 연결이 빈번하게 끊어지지 않도록 보장해야 하는 경우에 사용됩니다.
- **false로 설정**: 연결 자원을 절약하거나, 활성 호출이 없을 때 불필요한 네트워크 트래픽을 피하고자 할 때 사용됩니다.
