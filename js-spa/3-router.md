# 라우터 구현
SPA(Single Page Application)의 Router를 구현하는 방법으로는 `hash`, `history API`를 활용하는 방법이 있습니다.  
`hash` 방식은 URL에 `#`을 사용해서 페이지를 구분하는 방식이고, `history API`는 `pushState`를 사용해서 페이지를 구분하는 방식입니다.  
`hash` 방식보다는 `history API`를 사용하면, URL이 깔끔해지고, SEO에도 유리하다고 합니다.  
그래서 `history API`를 사용해서 라우터를 구현했어요.  

## History API
`history API`를 사용하기 위해서는 `window.history` 객체를 사용해야 해요.
```js
// pushState를 사용하면, history에 새로운 state를 추가할 수 있어요.
window.history.pushState({ page: 1 }, 'title 1', '/page/1');

// back을 사용하면, history에서 이전 state로 이동할 수 있어요.
window.history.back();

// forward를 사용하면, history에서 다음 state로 이동할 수 있어요.
window.history.forward();

// go를 사용하면, history에서 특정 state로 이동할 수 있어요.
window.history.go(-1);
```

## Router
SPA를 구현함에 있어서 가장 중요한 부분이라고 생각해요.  
`Router`는 `history API`를 사용해서, URL의 변화를 감지하고, 변화에 맞는 `custom element`를 렌더링하는 역할을 해요.  
자세한 내용은 [프레임워크 없는 프론트엔드 개발](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=260034588) 책을 참고했어요.  
아래 내용들은 라우터 구현 과정에서 중요하다고 생각한 부분들을 정리했어요.

### routes
`routes`는 `path`와 `tagName`를 가지고 있는 객체의 배열입니다.  
`path`는 URL의 path를 의미하고, `tagName`은 `custom element`의 이름을 의미해요.  
`path`에 `:id`와 같이 `:`을 사용하면, `params`로 인식하고, `params`는 `custom element`의 `attribute`로 전달됩니다.
```js
const routes = [
    { path: '/', tagName: 'main-page' },
    { path: '/article/:articleId', tagName: 'article-page' },
    { path: '/404', tagName: 'notfound-page' },
];
```

### REGEXP
`REGEXP`는 `path`를 정규식으로 변환하기 위한 정규식입니다.  
`ROUTE_PARAMETER`는 `path`에서 `:`을 사용해서 정의한 파라미터의 이름을 찾기 위한 정규식이고,  
`URL_FRAGMENT`는 `[^\/]+`와 같이 정규식을 사용해서, `/`를 제외한 모든 문자를 찾을 수 있어요.
```js
const REGEXP = {
    ROUTE_PARAMETER: /:(\w+)/g,
    URL_FRAGMENT: '([^\\/]+)',
};

const parsedPath = route.path
    .replace(REGEXP.ROUTE_PARAMETER, (match, paramName) => {
        params.push(paramName);
        return REGEXP.URL_FRAGMENT;
    }).replace(/\//g, '\\/');

describe('REGEXP를 테스트 합니다.', () => {
    describe('ROUTE_PARAMETER는 :param 형식을 가집니다.', () => {
        test(`${API_MOCK.ARTICLE_DETAIL} 는 :id와 매치 됩니다.`, () => {
            const route = API_MOCK.ARTICLE_DETAIL;
            const match = route.match(REGEXP.ROUTE_PARAMETER);
            expect(match).toEqual([':id']);
        });
    });
    describe('URL_FRAGMENT는 URL 경로에서 params 값을 추출합니다.', () => {
        test(`/api/article/:id는 api, article, :id와 매치됩니다.`, () => {
            const urlFragment = new RegExp(REGEXP.URL_FRAGMENT, 'g');
            const params = '/api/article/:id'.match(urlFragment);
            expect(params).toEqual(['api', 'article', ':id']);
        });
    });
});
```


### initRouter
`initRouter`는 `routes`를 인자로 받아서, `routes`에 맞는 `custom element`를 렌더링하는 함수예요.  
parsedPath는 `path`를 정규식으로 변환한 값이고, `params`는 `path`에서 `:`을 사용해서 정의한 파라미터의 이름을 담고 있어요.   
App이 실행될 때, `initRouter`를 호출해서 `routes`를 초기화해요.
```js


const initRouter = ({ $target, $element, routes }) => {
  const parsedRoutes = routes.map((route) => {
    const params = [];

    const parsedPath = route.path
      .replace(REGEXP.ROUTE_PARAMETER, (match, paramName) => {
        params.push(paramName);
        return REGEXP.URL_FRAGMENT;
      })
      .replace(/\//g, '\\/');

    const regexp = new RegExp(`^${parsedPath}$`);

    return {
      ...route,
      params,
      regexp,
    };
  });

  window.addEventListener('popstate', handlePopstate.bind(null, parsedRoutes, $element));
  $target.addEventListener('click', handleLinkClick.bind(null, parsedRoutes, $element));
  checkRoutes(parsedRoutes, window.location.pathname, $element);
};

// App.js
initRouter({ $target, $element: $main, routes });
```

### checkRoutes
`checkRoutes`는 `routes`와 `pathname`을 인자로 받아서, `routes`에 맞는 `custom element`를 렌더링하는 함수예요.
```js
const checkRoutes = (routes, pathname, $target) => {
  const currentRoute = routes.find((route) => {
    const { regexp } = route;
    return regexp.test(pathname);
  });

  const notFoundRoute = routes.find((route) => route.path === '/404');
  const targetRoute = currentRoute || notFoundRoute;

  const params = getUrlParams(targetRoute, pathname);
  renderRoute({
    tagName: targetRoute.tagName,
    $target,
    params,
  });
};
```

### getUrlParams
`getUrlParams`는 `route`와 `pathname`을 인자로 받아서, `route`에 맞는 `params`를 반환하는 함수예요.  
`matches`는 `pathname`과 `route.regexp`를 비교해서, `route.params`에 맞는 값을 담고 있어요.
```js
const getUrlParams = (route, pathname) => {
  const params = {};

  if (route.params.length === 0) {
    return params;
  }

  const matches = pathname.match(route.regexp);
  matches.slice(1).forEach((paramValue, index) => {
    const paramName = route.params[index];
    params[paramName] = paramValue;
  });

  return params;
};
```

### renderRoute
`renderRoute`는 `tagName`, `$target`, `params`를 인자로 받아서, `tagName`에 맞는 `custom element`를 렌더링하는 함수예요.
```js
const renderRoute = ({ tagName, $target, params }) => {
  try {
    const $routePage = createElement(tagName, params);
    $target.innerHTML = '';
    $target.appendChild($routePage);
  } catch (e) {
    console.error('renderRoute Error:', e);
  }
};

const createElement = (tagName, props = {}) => {
    const $element = document.createElement(tagName);
    try {
        const hasParams = isObjectEmpty(props);
        if (hasParams) {
            Object.entries(props).forEach(([key, value]) => {
                $element.setAttribute(key, value);
            });
        }
        return $element;
    } catch (e) {
        console.error('createElement Error:', e);
        return $element;
    }
};
```

### handlePopstate
`handlePopstate`는 `routes`와 `$element`를 인자로 받아서, `popstate` 이벤트를 처리하는 함수예요.   
```js
const handlePopstate = (routes, $target) => {
  checkRoutes(routes, window.location.pathname, $target);
};
```

### handleLinkClick
`handleLinkClick`는 `routes`, `$element`, `e`를 인자로 받아서, `click` 이벤트를 처리하는 함수예요.  
`Shadow DOM`을 사용하면, `click` 이벤트가 발생했을 때, `e.target`이 `shadow root`가 되어서, querySelector를 사용할 수 없어요.  
그래서 `e.composedPath()`를 사용해서, `custom element`를 찾아서, `dataset`을 사용해서, `link`를 가져왔어요.  
```js
const handleLinkClick = (routes, $target, e) => {
  const path = e.composedPath();
  const $link = path.find((el) => el.tagName === 'A' && el.dataset.link);
  if ($link) {
    e.preventDefault();
    if (window.location.pathname === $link.dataset.link) {
      return;
    }
    const { link } = $link.dataset;
    window.history.pushState(null, null, link);
    checkRoutes(routes, link, $target);
  }
};
```
