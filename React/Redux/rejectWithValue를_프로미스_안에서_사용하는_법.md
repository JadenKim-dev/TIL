# rejectWithValue를 프로미스 안에서 사용하는 법

### 1. createAsyncThunk()
redux-toolkit에서는 비동기 요청을 async thunk로 감싸서 처리하는 경우가 많다.  
createAsyncThunk 메서드를 사용할 경우에 호출 예시는 다음과 같다.  
```javascript
const fetchPostById = createAsyncThunk(
  // actionType
  'posts/getByPostId',
  // payloadCreator
  async (postId, thunkAPI) => {
    const response = await axios.get(`/posts/${postId}`)
    return response.data;
  },
);
```

createAsyncThunk에서 두 번째 인자로 넘기는 payloadCreator 안에서는 API 호출 등의 비동기 작업을 통해 데이터를 받아와서, 반환하는 식으로 구성된다.  
이 때 payloadCreator가 두 번째 매개변수로 받는 thunkAPI에는 dispatch, getState 등 redux 관련 메서드들이 담겨있다.

### 2. rejectWithValue()
rejectWithValue() 메서드는 thunkAPI 안에 존재한다.  
payloadCreator 안에서 redux로 에러 정보를 넘겨주기 위해서는 단순히 error를 throw 하는 식으로 하면 안 된다.  
다음과 같이 error에서 받은 정보를 rejectWithValue의 매개변수로 담아서 호출해야 한다.

```javascript
const fetchPostById = createAsyncThunk(
	'posts/getByPostId',
	async (postId, thunkAPI) => {
		try {
			const response = await axios.get(`/posts/${postId}`)
			return response.data;
		} catch (error) {
			return thunkAPI.rejectWithValue(error.response.data);
		}
  	},
);
```

위와 같이 작성할 경우 dispatch의 결과에 payload에 rejectWithValue에 넘겨준 값이 담기게 된다.

예를 들어 위에서 정의한 async thunk를 이용하여 다음과 같이 dispatch 하는 메서드를 작성했다고 하자.
```js
const dispatch = useDispatch();
const postId = '12345'

const result = await dispatch(fetchPostById(postId));
```

result에는 아래와 같은 내용들이 담긴다.
```js
{
    "type": "posts/getByPostId",
    "payload": {
		errorCode: "ERROR_101"
	},
    "error": {
        "message": "Rejected"
    }
}
```
payload에는 rejectWithValue에 매개변수로 넘겨준 데이터가 담겨 있다.  
(위의 예시에서는 error.response.data에 담겨있는 데이터이다.)  
error에는 자동적으로 `{ message: "Rejected" }`가 담긴다.
type에는 action type이 담긴다.(createAsyncThunk 호출 시 첫번째 매개변수로 지정해 줬던 값이다.)

여기서 payload에 담긴 데이터를 이용해서 자유롭게 에러 처리를 할 수 있다.  
내가 참여했던 프로젝트의 경우에는 해당 에러 코드를 이용해서 적절한 메시지를 담은 토스트를 띄어주는 식으로 구현했다.  


### 3. payloadCreator에서 커스텀 프로미스를 반환할 때 rejectWithValue를 사용하는 방법
createAsyncThunk에 2번째 인자로 전달하는 payloadCreator 콜백함수 내에서 async/await 처리를 하는 대신 Promise를 직접 만들어 반환해야 하는 상황이 있을 수 있다.  
만약 이 때 rejectWithValue를 사용하고 싶다면, Promise 매개변수로 넘어온 reject를 사용하지 않고, rejectWithValue 메서드 호출 결과를 resolve 메서드에 담아서 호출해주는 식으로 작성해야 한다.  
예시 코드는 아래와 같다.  

```javascript
const fetchPostById = createAsyncThunk(
	'posts/getByPostId',
	(postId, thunkAPI) => new Promise((resolve, reject) => {
		try {
			const response = await axios.get(`/posts/${postId}`)
			resolve(response.data);
		} catch (error) {
			resolve(thunkAPI.rejectWithValue(error.response.data));
		}
  	})
);
```
프로미스 내에서 reject 메서드는 아예 사용되지 않고 있다.  
에러가 발생한 경우에는 rejectWithValue에 적절한 데이터를 넘겨서 resolve하고 있을 뿐이다.  
이를 통해 일관되게 rejectWithValue를 통해 에러를 핸들링 할 수 있었다.  

### 4. 개인적인 생각
위와 같이 작업을 하면서 rejectWithValue의 인터페이스는 그렇게 바람직한 방식은 아닌 것 같다는 생각을 했다.  
Promise 내에서 에러가 발생하면 reject에 담아서 호출하고, 콜백 함수에서 에러가 발생하면 throw를 하는게 일반적인 에러 처리 방식이다.  
redux-tools의 rejectWithValue는 위 형식을 벗어난 방식으로 에러 처리를 하도록 유도하고 있다.  
일반적인 에러 핸들링 인터페이스를 채택하면서 데이터를 전달할 수 있는 방식을 고안해냈으면 더 좋았을 것 같다.
