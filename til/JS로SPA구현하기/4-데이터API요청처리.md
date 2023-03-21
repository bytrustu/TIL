# 데이터 API 요청 처리
toss.tech 사이트에서 더미데이터를 수집하여, API를 구현했어요.  
페이지 전환시에 Mocking API를 호출하고, 데이터를 받아와서 렌더링하는 방식으로 구현했어요.  

## 더미데이터 유틸 함수
API를 요청하는 과정에서, 더미데이터에 필요한 유틸 함수를 만들고 테스트 코드를 작성했어요.  
```js
import { ARTICLE_DETAILS, ARTICLES } from './constant';

export const getArticleList = () => ARTICLES;
export const getArticleDetailById = (id) => ARTICLE_DETAILS.find((article) => article.id === id) ?? null;
```

```js
import { getArticleList, getArticleDetailById } from './articleUtils';
import { ARTICLE_DETAILS, ARTICLES } from './constant';

describe('getArticleList 함수를 테스트합니다.', () => {
  test('Article List를 가져옵니다.', () => {
    const articles = getArticleList();
    expect(articles).toEqual(ARTICLES);
  });

  test('Article List가 비어있지 않은지 확인합니다.', () => {
    const articles = getArticleList();
    expect(articles.length).toBeGreaterThan(0);
  });
});

describe('getArticleDetailById 함수를 테스트합니다.', () => {
  test('id가 1인 Article Detail 데이터를 정상적으로 가져오는지 확인합니다.', () => {
    const id = 1;
    const articleLength = ARTICLE_DETAILS.length;
    const expectedArticle = ARTICLE_DETAILS[articleLength - id];
    const article = getArticleDetailById(id);
    expect(article).toEqual(expectedArticle);
  });

  test('id가 존재하지 않은 데이터 일 경우 null이 반환되는지 확인합니다.', () => {
    const id = 0;
    const article = getArticleDetailById(id);
    expect(article).toBe(null);
  });

  test('id가 음수인 경우 null이 반환되는지 확인합니다.', () => {
    const id = -1;
    const article = getArticleDetailById(id);
    expect(article).toBe(null);
  });

  test('id가 문자열인 경우 null이 반환되는지 확인합니다.', () => {
    const id = '문자예요.';
    const article = getArticleDetailById(id);
    expect(article).toBe(null);
  });

  test('id가 null인 경우 null이 반환되는지 확인합니다.', () => {
    const id = null;
    const article = getArticleDetailById(id);
    expect(article).toBe(null);
  });
});
```
