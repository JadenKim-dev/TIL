axios를 이용할 때 proxy를 거쳐서 요청을 보내고 싶다면 아래의 방법을 사용합니다.

https proxy agent를 사용하여 proxy를 사용하는 http agent를 만들고, axios.create의 인자로 해당 agent를 넘겨줍니다.

```js
const HttpsProxyAgent = require("https-proxy-agent"),
const axios = require("axios");

const httpsAgent = new HttpsProxyAgent({host: "<proxy domain>", port: "<proxy port>", auth: "<username>:<password>"})
const axiosUsingProxy = axios.create({httpsAgent});
```

이제 httpsAgent를 인자로 넘겨서 생성한 axiosUsingProxy를 이용하여 요청을 보내면 해당 프록시를 거쳐서 요청을 보내게 됩니다.

참고자료:
https://stackoverflow.com/questions/55981040/how-to-use-axios-with-a-proxy-server-to-make-an-https-call
