# UI 구현
CSS 네이밍으로는 BEM을 사용하였고, 공통된 CSS 속성 값은 css 변수로 관리하였습니다.  
Web Component를 사용하여 UI를 구현하였습니다.

## CSS

### BEM(Block, Element, Modifier)
BEM은 CSS를 구조화하고 모듈화하는 방법론 중 하나입니다. BEM을 사용하면 CSS 코드를 구조적으로 구성하고, 모듈화 할 수 있어요.
```css
.article-author {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin: 1.25rem 0 0 0;
}
.article-author__img {
  width: 3rem;
  height: 3rem;
  border-radius: 9.75rem;
  margin-right: 0.875rem;
}
.article-author__info {
  display: flex;
  flex-direction: column;
}
.article-author__name-job {
  font-size: var(--font-size-h7);
  color: var(--color-h7);
  margin: 0.5rem 0 0.125rem 0;
}
.article-author__date {
  font-size: var(--font-size-span);
  line-height: 1.5;
  color: var(--color-p);        
}
```

### CSS 변수
CSS 변수는 CSS의 전역 변수입니다.  
CSS의 모든 속성에 사용할 수 있고, `var()` 함수를 사용하여 값을 가져올 수 있어요.
```css
:root {
    --border-grey: rgba(0, 27, 55, 0.1);
    --border-radius: 0.875rem;
    --color-h3: #333d4b;
    --color-h7: #4e5968;
    --color-p: #8b95a1;
    --color-blue: #3182f6;
    --color-white: #fff;
    --font-size-h2: 3rem;
    --font-size-h3: 2.25rem;
    --font-size-h7: 1.0625rem;
    --font-size-p: 0.9375rem;
    --font-size-span: 0.875rem;
}
```

## Web Component
Web Component는 HTML 표준 기술 중 하나로, 재사용 가능한 컴포넌트를 만들고 사용할 수 있게 해요.

### Shadow DOM
Shadow DOM은 DOM 트리의 일부로 존재하는 DOM입니다.  
스코프가 분리되어있어, 독립적인 스타일을 적용할 수 있고, 외부에서 접근할 수 없어요.  
```js
class SampleContainer extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        ${style}
      </style>
      <div>Shadow DOM</div>
    `;
  }
}

customElements.define('sample-container', SampleContainer);
```

### Core Component
`Web Component`를 사용하면서 공통적으로 사용되는 기능을 `Core Component`로 만들었어요.

`super()`를 호출할 때, 사용할 속성을 전달하면 해당 속성을 props로 관리하고,  
`connectedCallback()`에서 `render()`를 호출하여, props를 사용하여 HTML을 생성하고, Shadow DOM에 추가해요.

```js
class CoreComponent extends HTMLElement {
  constructor(attributes) {
    super();
    this.props = {};
    for (const attribute of attributes) {
      this.props[attribute] = this.getAttribute(attribute);
    }
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    this.render();
  }

  get styles() {
    return '';
  }

  createHTML(props) {
    return '';
  }

  render() {
    const html = this.createHTML(this.props);
    this.shadowRoot.innerHTML = `
      ${this.styles}
      ${html}
    `;
  }
}

export default CoreComponent;
```

아래 처럼 활용할 수 있어요.  
기존에는 constructor에서 props를 관리하고, attachShadow를 호출하고, render를 호출했어요.  
그 과정을 CoreComponent에서 처리하고, 상속받은 클래스에서는 attributes를 전달하고, createHTML만 구현하면 되게끔 변경했어요.

```js
class SampleContainer extends CoreComponent {
  constructor() {
    super(['title']);
  }

  get styles() {
    return `
      <style>
        .sample-container {
          border: 1px solid var(--border-grey);
          border-radius: var(--border-radius);
          padding: 1.25rem;
        }
      </style>
    `;
  }

  createHTML({ title }) {
    return `
      <div class="sample-container">
        <h3>${title}</h3>
      </div>
    `;
  }
}
```
