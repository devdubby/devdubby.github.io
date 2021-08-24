---
title: 프로그래머스 FE 생존키트 스터디 3주차
date: 2021-08-07 10:08:41
category: 프로그래머스
thumbnail: { './images/thumbnail-fekit.png' }
draft: false
---

![thumbnail](./images/thumbnail-fekit.png)

## 지난 과제 리뷰
display: none, block 보다 visibility 속성을 활용하면 transition 을 걸 수 있다.
async 함수는 항상 실제 return statement 옆에 무엇이 정의되어 있든 간에 무엇이든 간에 Promise를 return 한다.

## Javascript 에서 Error 처리하기

### try { ... } catch { ... } finally { ... }  
try - 항상 실행  
catch - throw 실행 or Promise.reject 실행  
finally - finally 문은 성공 여부와 상관 없이 항상 실행  

### 에러 다시 던지기
catch 구문에는 예상한 에러들만 항상 떨어진다고 보장하기 어렵기때문에, 의도한 에러인지 파악하고 해당 에러만에 대해서만 처리하고 그렇지 않은 에러가 발생한 경우에는 throw문을 다시 작성해주는 것이 좋다.

## Javascript 에서 비동기 처리

- callback
- callback hell
- 프라미스화 (Promisify)
- Promise API (all, allSettled, race)
- Promise 에러 처리
- async, await

ex)  
여러개의 script를 순차적으로 불러오고 싶다면, 익히들 알고 있는 callback hell이 발생한다.  
```javascript
function loadMultipleKakakoSdk() {
  loadScript(KAKAO_SDK_URL, (error, result) => {
    if (error) {
      return;
    }

    alert('kakao sdk#1 is loaded successfully');
    loadScript(KAKAO_SDK_URL, (error2, result2) => {
      if (error2) {
        return;
      }

      alert('kakao sdk#2 is loaded successfully');

      loadScript(KAKAO_SDK_URL, (error3, result3) => {
        if (error3) {
          return;
        }

        alert('kakao sdk#3 is loaded successfully');
      })
    })
  });
}
```
promise로 여러개의 비동기 코드를 순차적으로 처리하려면 아래와 같이 한번의 nesting으로 처리할 수 있다.  
```javascript
function loadMultipleKakakoSdkPromise() {
  loadScriptPromise(KAKAO_SDK_URL)
    .then((script) => {
      alert('kakao sdk#1 is loaded successfully')
      return loadScriptPromise(KAKAO_SDK_URL)
    })
    .then(() => {
      alert('kakao sdk#2 is loaded successfully')
      return loadScriptPromise(KAKAO_SDK_URL)
    })
    .then(() => {
      alert('kakao sdk#3 is loaded successfully')
    });
}
```

async, await를 사용하면 아무런 nesting없이 promise를 동기적으로 실행할 수 있다.  
async함수는 항상 실제 return statement 옆에 무엇이 정의되어 있든 간에 무엇이든 간에 Promise를 return한다.  
순서가 상관없으면 Promise.all을 await하면 된다.  
```javascript
async function runPromiseWithAsyncAndAwait() {
  await loadKakaoSdkPromise();
  await loadKakaoSdkPromise();
  await loadKakaoSdkPromise();

  // await Promise.all([
  //   loadKakaoSdkPromise(),
  //   loadKakaoSdkPromise(),
  //   loadKakaoSdkPromise(),
  // ])
}
```

## API Request 취소하기
stale된 API Request를 취소해야할까
- resource를 아낄 수 있다.
- 일관적인 데이터를 전달할 수 있다.
- 예를 들어서 리스트 검색 UI를 만들 때에 클라이언트에서 항상 나중에 보낸 request가 더 나중에 올 것이라는 가정을 할 수 없다.
  어떤 상황에 따라서는 먼저보낸 request가 나중에 보낸 request보다 늦게 올 수 있고 그럴 경우에 cancel하지 않을 경우 일관적이지 않은 데이터가 유저에게 노출될 수 있다.
- AbortController 라는 Api 를 사용하여 이를 해결할 수 있다. axios의 cancle token과 같은 역할  

ex)
```javascript
// 해당 API를 사용하면 fetch시에 signal을 넘겨주기만 하면,
// 추후에 해당 signal을 가지고 있는 API request를 취소할 수 있다.
const fetchDog1Controller = new AbortController();
const fetchDog2Controller = new AbortController();

function fetchDog1WithSignal() {
  // fetch의 경우 signal을 url 뒤에 두번째 인자로 넘겨주면 된다.
  const { message } = fetchDog({ signal: fetchDog1Controller.signal })
    .then(({ message }) => {
      addDogImage(message, 'image from fetchDog1', 'result2');
    });
}

function fetchDog2WithSignal() {
  // 예시
  // fetchDog1WithSignal이후에 fetchDog2WithSignal을 부른다고 쳤을 때에
  // 아직 실행되고 있는 fetchDog1WithSignal이 존재한다면 abort시키고,
  // fetchDog2WithSignal을 진행하도록 작성
  fetchDog1Controller.abort();
  return fetchDog({ signal: fetchDog2Controller.signal })
    .then(({ message }) => addDogImage(message, 'image from fetchDog2', 'result2'));
}
```

## CORS와 Simple Request 이해하기
CORS와 Simple Request란 무엇일까?

1) 교차 출처 리소스 공유(Cross-Origin Resource Sharing, CORS)
- 한 출처(도메인)에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제

2) Simple Request란?
- 동일한 도메인이면 바로 요청이 가는데, CORS 요청 시에는 OPTIONS request를 한번 보내고 직접 우리가 작성한 request를 실행 시킨다.  
- 그런데 CORS 인데도 불구하고 이 OPTIONS(preflight request)없이 바로 request를 실행시키는 request  
- Simple Request는 GET, POST만 지원하며, 추가할 수 있는 Header가 제한되어 있다. Simple Request로 처리하는 것이 round trip(다른 도메인으로 왔다 갔다)을 한번으로 줄여줄 뿐만 아니라 performance에 꽤 큰 영향을 미칠 수 있는 가능성이 있기 때문에 가능하다면 Simple Request를 지향하는 것이 좋다.


## Browser에 데이터 저장하기
원격의 데이터베이스가 아니라 브라우저에도 데이터를 저장할 수 있습니다.

1) sessionStorage / localStorage
2) indexedDB
- indexedDB는 sessionStorage, localStorage보다 훨씬 더 많은 양의 데이터를 저장할 수 있고 더 기능도 풍부하지만 직접 사용할만한 use-case는 크게 많지는 않다.
- 브라우저마다 지원현황이 다를 수 있기때문에 브라우저별로 가장 적절한 storage를 골라서 사용해주는 local-storage-fallback 과 같은 library를 쓰는 것이 편리하다.
