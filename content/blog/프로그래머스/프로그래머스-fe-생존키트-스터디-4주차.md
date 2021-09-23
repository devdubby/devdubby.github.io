---
title: 프로그래머스 FE 생존키트 스터디 4주차
date: 2021-08-14 10:08:15
category: 프로그래머스
thumbnail: { thumbnailSrc }
draft: false
---

![thumbnail](../../assets/thumbnail-fekit.png)

## 지난 과제 리뷰
http status code 는 정말 다양하게 있다.  
성공이 무조건 200 만을 주진 않음  
2xx 로 대응  

## 이미지 최적화 / 지연 로딩
웹 서비스에서 이미지를 사용할 때에 어떤 점들을 고려해야할까?

1) Responsive Image를 사용하자
2) 올바른 format의 image를 사용하자 (WebP)
3) 지연로딩(Lazy loading)을 사용하자

```javascript
<picture>
  <!--
    source tag에서 srcset을 Nx를 기준으로 comma로 분리해서 src를 제공하면 맞는 이미지를 사용할 수 있다. 
    아래 예시는 window.devicePixelRatio를 기준으로 다른 이미지를 부르게된다. source tag와 srcset를 지원하지
    않는 브라우저는 img태그의 src를 바라보게 된다.

    항상 devicePixelRatio로 기준으로 나누는 것이 최선은 아닐 수 있지만, 가장 합리적인 방법이라고 생각하고 있다.
    조금 더 자세한 내용은 달아드린 링크에서 더 깊게 확인하실 수 있다.
  -->
  <source srcset="https://cdn2.thecatapi.com/images/tt01SNoSH.png 1x,
    https://cdn2.thecatapi.com/images/e7-hS3gey.jpg 2x,
    https://cdn2.thecatapi.com/images/Pqtwt4FCq.jpg 3x" />
  <img src="https://cdn2.thecatapi.com/images/tt01SNoSH.png" width="200px" />
</picture>
```
window.devicePixelRatio: 실제 한 픽셀에 보여주는 픽셀을 나타낸 수치

```javascript
// webp라는 새로운 이미지의 포맷은 기존에 많이 사용하던 png, jp(e)g보다 훨씬 더 사이즈도 작고 좋은 퍼포먼스를
// 내는데에 도움이 되기 때문에 가능하다면 webp를 사용하고 지원하지 않는 브라우저를 위해 jpg, png와 같은 포맷을 fallback으로 추가하는 것이 필요하다.
// 위쪽의 devicePixelRatio와 format까지 다 고려해서 source 태그들을 작성하면 훨씬 더 퍼포먼스가 좋은 웹을 구현할 수 있다.

<h4>Image Formats (Webp)</h4>
<picture>
  <source type="image/webp" srcset="/assets/white-cat.webp">
  <source type="image/jpeg" srcset="/assets/white-cat.jpeg">
  <img src="/assets/white-cat.jpeg" width="200px" />
</picture>
```
대부분의 브라우저에서 lazyloading을 대응하기 위해서는 lazysizes와 같은 script를 추가하고  
imgElement들에 data-src라는 dataset값을 넣고, className="lazyload"를 추가해주면 lazysizes가  
직접 image elements들을 보고 viewport에 들어오게되면 해당 image를 로드하기 시작한다.  
직접 위와 같은 내용을 구현하려면 우리가 이전에 배웠던 intersectionObserverAPI를 사용해서 구현할 수도 있겠다.  
하지만 이미 널리 사용되고 있는 라이브러리가 있으니 그냥 해당 라이브러리를 사용하는 것도 좋겠다.  

## 웹폰트 최적화
웹폰트를 사용할 때에 어떤 점들을 주의해야할까

1) 웹폰트가 로드되기전에 fallback 폰트를 사용하자 (font-display)
2) 최적화 체크리스트를 확인하자  
- 꼭 필요한 폰트만 사용하자
- 여러 폰트를 사용할 때에 필요한 unicode-range property를 활용하여 필요한 폰트만 다운로드 받자
- local()과 다양한 브라우저에서 모두 대응할 수 있게 다양한 format을 포함시키자 (woff2, woff, ttf, otf 등)

## Javascript, CSS 최적화
자바스크립트, CSS 파일 사용시에 어떤 최적화 방법들을 적용하면 좋을까?

공통
- preload (현재 페이지에서 필수로 사용될 리소스)
- preconnect (현재 페이지에서 요청할 외부 도메인 리소스 주소)
- prefetch, dns-prefetch (빠른 시일내에 사용될 미래의 리소스 혹은 도메인 주소)

Javscript
- Code Spliting
- Remove unused code (Chrome - Coverage탭 활용)
- Minify Javascript (webpack loader)

CSS
- Extract critical CSS
- Defer non-critical CSS
- Minify CSS (webpack loaders)

ex)
```javascript
<!-- #1 해당 페이지가 작동하기 위해서는 꼭 필요한 리소스를 preload -->
<link rel="preload" href="/assets/lazysizes.min.js" as="script" />

<!-- #2 해당 페이지에서 곧 필요해질 리소트 prefetch -->
<link rel="prefetch" href="/assets/lazyLoadModule.js" />

<!-- #3 해당 페이지에서 곧 요청할 외부 도메인 preconnect -->
<link rel="preconnect" href="https://fonts.google.com/" />
```
초기 로딩에 필요하지 않고 서비스 사용 중에 나중에 필요한 스크립트의 경우  
lazy하게 script를 가져와서 사용할 수 있다. 위와 같은 전략들을 많이 포함시키게 되면  
script를 불러오는 동안의 처리 등은 추가로 해야하지만, 유저가 초기에 받아야하는 javscript의 양은 줄일 수 있기 때문에  
퍼포먼스적으로 좋은 서비스를 만들 수 있따. javascript는 아래와 같이 import할 수 있고,  
react같은 경우에는 `React.lazy`라는 API를 제공하기도 한다.  

## Suspense, Successs, ErrorBoundary
Loading, Success, Error UI 처리를 추상화해보자 웹 서비스를 개발하다보면 비동기코드들을 작성할 수 밖에 없다.  
예를 들어, 비동기적으로 module을 가져온다던가, remote server에서 API를 통해서 데이터를 가져오는 등 많은 일들이    
Javascript에서는 비동기적으로 발생할 수 있다. 위와 같은 코드들을 조금 더 우아하게 어떻게 처리할 수 있을까?  
하나의 method에서 로딩, 성공, 에러를 다 처리하기보다 책임을 나눌 수 있으면 조금 더 깔끔하게 코드를 작성할 수 있을 수 있다.    

ex)
```javascript
resetError() {
  this.setState({ ...this.state, error: null });
  this.componentDidMount();
}

renderOnSuccess() {
  const { images } = this.state;
  this.$target.innerHTML = images.map((url) => `
    <img src=${url} class="child" />
  `).join('');
}
    
renderOnLoading() {
  this.$target.textContent = 'Loading...'
}
    
renderOnError(error) {
  this.$target.innerHTML = `
    <div>
      <div>
        Error occurred: ${error.message}
      </div>
      <div style="margin-top: 16px;">
        <button type="button">RESET ERROR</button>
      </div>
    </div>
  `;
  this.$target.querySelector('button').addEventListener('click', this.resetError.bind(this));
}
```

## 퍼포먼스 향상을 위해 차용할 수 있는 패턴들
퍼포먼스 향상을 위해 추가로 고려해볼 수 있는 패턴들은 무엇이 있을까?

- prefetch API Request (requestIdleCallback)
- refetchOnFocus (데이터 최신화)
- optimistic UI

ex)
```javascript
// window.requestIdleCallback은 browser compatibility가 그렇게 좋지는 않지만
// polyfill해서 사용할 수도 있다. requestIdleCallback을 통해서 등록된 callback들은
// 브라우저가 할 일없이 쉬고 있을 때 미리 데이터들을 준비해두도록 요청할 수 있다.
function addIdleCallbacks(callbacks) {
  callbacks.forEach((callback) => {
    window.requestIdleCallback(callback);
  });
}
```

###  optimistic UI
createUser는 비동기 코드이기때문에 바로 부른다고 해서
바로 user가 만들어지지 않지만, 우선 유저에게 빠른 화면 전환 및 데이터를 보여주기위해서
우선 실행시켜놓고 미리 완료되었을 것이라고 가정하고 추가하는 데이터를 미리 그리고,
이후에 정말로 user가 실제로 추가가 완료되면 에러가 나면 롤백하면서 다시 UI를 다시 그리고
예상했던대로 성공한다면 그냥 그대로 유지하고 유저가 다음작업을 할 수 있도록 한다.
위와 같은 UI 업데이트 접근법을 optimisitc updates (optimistic UI)라고 한다.
