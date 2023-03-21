# 개발환경 구성
프로젝트 개발환경 구성에 대해서 알아보겠습니다.  
Vanilla Javascript로 구현을 진행하게 되어서 많은 라이브러리를 활용하진 않았어요.  
API Mocking을 위해서 [MSW](https://mswjs.io/)를 사용했고, API request를 위해서 [Axios](https://axios-http.com/kr/docs/interceptors)를 사용습니다.  
그리고 번들링을 위한 [Webpack](https://webpack.js.org/)을 사용했어요.

## Webpack
Webpack은 모듈 번들러입니다. 모듈 번들러란, 여러개의 모듈을 하나의 파일로 묶어주는 도구입니다.  
모듈 번들러를 사용하면, 여러개의 파일을 하나의 파일로 묶어서 요청을 줄일 수 있어요.

`entry point`를 설정하면 해당 경로 파일을 기준으로 모듈을 찾고, `output`경로로 파일이 번들링 됩니다.  
그리고 `babel-loader`를 사용해서 ES6+ 문법을 ES5로 변환해주었고, `css-loader`와 `style-loader`를 사용해서 css파일을 번들링했어요.
```js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public'),
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: 'babel-loader',
            },
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader'],
            },
        ],
    },
};
```

## MSW (Mock Service Worker)
MSW는 API Mocking을 위한 라이브러리입니다.  
API Mocking을 하면, API를 호출하기 전에 미리 정의한 데이터를 반환하기 때문에, API가 준비되지 않은 상태에서도 개발을 진행할 수 있어요.

브라우저에서 동작해야 하기 때문에, [공식문서](https://mswjs.io/docs/api/setup-worker)를 살펴보면 `setupWorker`를 사용하라고 되어있어요.  
`setupWorker`로 API Mocking을 하고, `worker.start()`를 호출해야 API Mocking이 동작합니다.  
`request Handler`를 등록하는 방식으로 API Mocking을 하는데, `mockServer` 함수를 직접 추상화하여 활용했어요.
```js
import { rest } from 'msw';

// restHandler를 추상화한 함수예요.
const mockServer = ({ method, path, statusCode, responseCallback }) =>
    rest[method.toLowerCase()](path, (req, res, ctx) => {
        const hasParams = isObjectEmpty(req.params);
        const response = hasParams ? responseCallback(req.params) : responseCallback();
        return res(ctx.status(statusCode), ctx.json(response));
    });

// restHandler를 활용하여 API Mocking한 함수예요.
const mockGetArticleList = () =>
    mockServer({
        method: 'GET',
        path: '/api/articles',
        statusCode: 200,
        responseCallback: () => [...요청되는 데이터 목록],
    });

// worker를 초기화하는 함수예요.
const initialWorker = async () => {
    try {
        if (typeof window !== 'undefined') {
            const mockApiHandler = [mockGetArticleList, mockGetArticleDetail].map((handler) => handler());
            const worker = setupWorker(...mockApiHandler);
            await worker.start();
            return true;
        }
        return false;
    } catch (e) {
        console.error(`initMocks Error: ${e}`);
        return false;
    }
};
```


## Axios
Axios는 API request를 위한 라이브러리입니다.  
Promise 기반으로 동작하기 때문에, `async/await`를 사용해서 API request를 진행했어요.

`axios.create`를 사용해서 `instance`를 생성하고,  
`interceptors`를 통해 request, response 요청을 가로채서 에러처리를 진행했어요.
```js
import axios from 'axios';

const instance = axios.create({
    baseURL: '/',
});

instance.interceptors.request.use(
    (config) => {
        return config;
    },
    (error) => {
        return Promise.reject(error);
    },
);

instance.interceptors.response.use(
    (response) => {
        return response;
    },
    (error) => {
        const errorMessage = error.response ? `서버 응답이 올바르지 않습니다. ${error.response.status}: ${error.response.data.message}` : error.message;
        return Promise.reject(new Error(errorMessage));
    },
);

export default instance;
```
