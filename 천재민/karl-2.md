# Intersection Observer API 알아보기

<br/>

# 들어가며
 요즘 회사에서 컨텐츠의 가시성을 기반으로 작업해야 할 일이 많습니다. 예를 들어, 광고 요금을 청구하기 위해선 광고 노출 통계를 쌓고 이를 기반으로 청구해야 합니다. 이런 경우 `scroll` 이벤트를 통해 콜백을 호출하여 가시성을 확인하는 방법도 있지만 `Intersection Observer API` 를 사용하여 작업하기도 합니다.

<br/>
<br/>

# Intersection Observer API란?
`Intersection Observer API` 는 2016년 4월에 [구글 개발자 페이지](https://developers.google.com/web/updates/2016/04/intersectionobserver) 를 통해 소개되었습니다. 새롭게 API가 만들어진 이유는 기존의 `scroll` 이벤트와 가시성 관찰에 사용되는 `getBoundingClientRect` 의 문제점 때문입니다.

 `scroll` 이벤트는 성능에 악영향을 줄 수 있는데 스크롤시 짧은 시간 내에 수 백, 수 천의 이벤트가 **동기적**으로 실행될 수 있습니다. 그리고 페이지 내에 각 요소가 각기의 목적(광고, 레이지 로딩, 무한 스크롤 등)의 이유로 `scroll` 이벤트를 리스닝하기 때문에 이에 상응하는 콜백이 무수히 실행될 수 있습니다. 이는 **메인 스레드에 큰 부하를 줄 수 있습니다**.

 그리고 `getBoundingClientRect` 은 **`reflow`를 발생시킬 수 있습니다.** 본래 브라우저는 최적화를 위해 필요한 여러 작업을 묶어 큐에 쌓아 대기하다가 한 번의 `reflow` 로 처리하고자 합니다. 그러나 `getBoundingClientRect` 호출시 값(top, right 등)을 정확히 읽어들이기 위해 **큐를 flush하고** 스타일을 적용함으로써 다 수의 `reflow` 를 발생시킬 수 있습니다. 

 사실, 이러한 루틴이 실무에서 매우 많이 쓰이고 있으므로 **신뢰도있는 공식 API의 필요성**도 있었습니다.

  `Intersection Observer API` 는 루트 요소와 타겟 요소의 교차점을 관찰합니다. 그리고 타겟 요소가 루트 요소와 교차하는지 아닌지를 구별하는 기능을 제공하고 있습니다. `scroll` 이벤트와 다르게 교차 시 **비동기적**으로 실행되며 가시성 구분 시 **`reflow` 를 발생시키지 않습니다.** 여러모로 성능 상 유리합니다. 

<br/>
<br/>

# 교차성(가시성)을 계산하는 방법
 좀 더 자세히 알아보기 전에 `Intersection Observer` 가 교차성(가시성)을 계산하는 방법에 대해서 알아봅시다. 앞으로 나올 사용법에서 계산된 교차성(가시성)에 대해 계속 언급이 되니 말입니다.

`Intersection Observer` 는 모든 영역을 사각형(`rectangle`)로 취급합니다. 요소가 사각형이 아니거나, 이외 이상하고 불규칙한 모습으로 렌더링되었다고 하더라도, **요소의 모든 부분을 감싸는 가장 작은 사각형 가정**하고 교차성(가시성)을 계산합니다.

이를 명심하고 글을 계속 읽어주세요.

<br/>
<br/>

# 사용법 및 스펙
일단 `Intersection Observer` 인스턴스를 생성해봅시다.

```jsx
let options = {
	root: document.querySelector('#scrollArea'),
	rootMargin: '0px',
  threshold: 1.0
}

// options에 따라 인스턴스 생성
let observer = new IntersectionObserver(callback, options);

// 타겟 요소 관찰 시작
let target = document.querySelector('#listItem');
observer.observe(target);
```

`new` 키워드를 통해 인스턴스를 생성합니다. `callback` , `options` 2개의 파라미터를 받습니다. `callback` 은 가시성의 변화가 생겼을 때 호출되는 콜백 로직입니다. `options` 는 만들어질 인스턴스에서 콜백이 호출되는 상황을 정의합니다.

<br/>

## Options

우선 `options` 부터 살펴보도록 하겠습니다.

<br/>

### root

타겟 요소의 가시성을 확인할 때 사용되는 루트 요소입니다. 이것은 타겟 요소보다 상위 요소, **즉 요소의 조상 요소이어야 합니다**. 설정하지 않거나 `root` 값을 `null` 로 주었을 때 기본 값으로 브라우저 뷰포트가 설정됩니다.

<br/>

### rootMargin

 `margin` 을 주어 **루트 요소의 범위를 확장할 수 있습니다.** 즉 확장된 영역 안에 타겟 요소가 들어가면 가시성에 변화가 생깁니다. `CSS` 의 `margin` 과 유사하게 `top`, `right`, `bottom`, `left` 의 `margin` 정도롤 각각 설정할 수 있습니다. 기본 값은 0이며 따로 설정 시 **단위를 꼭 입력해야합니다.**

<br/>

### threshold

콜백이 실행될 타겟 요소의 가시성 퍼센티지를 나타내는 단일 숫자 및 숫자 배열이 들어갈 수 있습니다. 즉, 요소의 `top`, `bottom` 이 노출된 순간만 콜백을 실행할 수 있는 것이 아니라 어**느정도 타겟 요소가 보여졌는 지에 따라서도 콜백을 호출할 수 있습니다**. 예를 들어 요소가 50%만큼 보여졌을 때 탐지하고 싶다면 단일 숫자 값 `0.5` 를 설정하면 됩니다. 혹은 25% 단위로 가시성이 변경될 때마다 콜백이 실행되게 하고 싶다면 `[0, 0.25, 0.5, 0.75, 1]` 을 설정하면 됩니다.

```jsx
// 타겟 요소가 50% 가시성이 확인되었을 때
let observer1 = new IntersectionObserver(callback, {
	threshold: 0.5
});

// 타겟 요소가 25% 단위로 가시성이 확인되었을 때
let observer1 = new IntersectionObserver(callback, {
	threshold: [0, 0.25, 0.5, 0.75, 1]
});
```

<br/>

## Callback

타겟 요소의 관찰이 시작되거나, 가시성에 변화가 감지되면(`threshold` 와 만나면) 등록된 `callback` 이 실행됩니다.

```jsx
let callback = (entries, observer) => {
  entries.forEach(entry => {
		// 각 entry는 가시성 변화가 감지될 때마다 발생하고 그 context를 나타냅니다.
    // target element:
    //   entry.boundingClientRect
    //   entry.intersectionRatio
    //   entry.intersectionRect
    //   entry.isIntersecting
    //   entry.rootBounds
    //   entry.target
    //   entry.time
  });
};
```

이 콜백은 메인스레드에서 처리되고 파라미터로 `entries` 와 `observer` 를 받게 됩니다.

<br/>

### Entries

`entries` 는 [`**IntersectionObserverEntry`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) 인스턴스를 담은 배열**입니다. 일반적으로 `callback` 에 파라미터로 전달이 되고 후술할 `Intersection Observer.takeRecords()` 를 통해 반환받을 수도 있습니다.

`IntersectionObserverEntry` 는 루트요소와 타겟요소의 교차(`threshold` 와 만났을 때)의 상황을 묘사합니다. 포함된 프로퍼티들은 모두 **읽기전용**(`read only`) 입니다.

- `IntersectionObserverEntry.boundingClientRect` : 타겟 요소의 사각형 정보([`DOMRectReadOnly`](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly))를 반환합니다. `getBoundingClientRect()` 호출과는 **다르게 `reflow` 를 발생시키진 않습니다.**

    (그림 요소 자리)

- `IntersectionObserverEntry.intersectionRect` : 타겟 요소의 가시성이 감지된 부분의 정보([`DOMRectReadOnly`](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly))를 반환합니다.

    (그림 요소 자리)

- `IntersectionObserverEntry.intersectionRatio` : 타겟 요소의 `intersectionRect` 이 `boundingClientRect` 와 어느정도로 교차(겹치는 지) 비율(0.0 ~ 1.0)을 반환합니다. 바꿔 말하면 타겟 요소가 루트 요소와 얼마나 교차하는지의 정도와 같습니다.

    (그림 요소 자리)

앞서, 타겟 요소의 관찰이 시작되면 콜백 또한 바로 호출된다고 말씀드렸습니다. 타겟 요소와 루트 요소가 전혀 교차하지 않았음에도 불구하고 말입니다. 이는 `**Intersection Observer` 의 기본동작**입니다.  이를 예외처리 하기 위해서 `intersectionRatio` 가 사용됩니다.

```jsx
let callback = (entries, observer) => {
  entries.forEach(entry => {
		// 타겟 요소가 루트 요소와 교차하는 점이 없으면 콜백을 호출했으되, 조기에 탈출한다.
		if (entry.intersectionRatio <= 0) return

		// 혹은 isIntersecting을 사용할 수 있습니다.
		if (!entry.isIntersecting) return

		// ... 콜백 로직
  });
};
```

- `IntersectionObserverEntry.isIntersecting` : 해당 `entry` 에 타겟 요소가 루트 요소와 교차하는 지 여부를 `Boolean` 값으로 반환합니다. 

    (그림 요소 자리)
- `IntersectionObserverEntry.rootBounds` : 루트 요소의 사각형 정보([`DOMRectReadOnly`](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly))를 반환합니다. 이 정보는 `rootMargin` 옵션 설정에 영향을 받습니다.

    (그림 요소 자리)

- `IntersectionObserverEntry.target` : 타겟 요소를 반환합니다.
- `IntersectionObserverEntry.time` : 문서([`Document`](https://developer.mozilla.org/en-US/docs/Web/API/Document))가 만들어진 표준 시간([`time origin`](https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp#the_time_origin))을 기준으로 타겟 요소와 루트 요소의 교차가 발생한 시간([`DOMHighResTimeStamp`](https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp))을 반환합니다.

<br/>

## Methods

- `IntersectionObserver.observe(targetElement)` : 타겟 요소에 대한 관찰을 시작합니다.

- `IntersectionObserver.unobserve(targetElement)` : 타겟 요소에 대한 관찰을 중지합니다. 관찰의 목적이 이루어져 굳이 계속 관찰을 할 필요가 없는 경우 사용합니다.
- `IntersectionObserver.disconnect(targetElement)` : 인스턴스의 타겟 요소들에 대한 모든 관찰을 중지합니다.
- `IntersectionObserver.takerecords(targetElement)` : `IntersectionObserverEntry` 인스턴스들의 배열을 리턴합니다.

<br/>
<br/>

# 용례

[MDN Intersection Observer API 페이지](https://developer.mozilla.org/ko/docs/Web/API/Intersection_Observer_API) 에서는 대표적인 용례를 4개 정도 말하고 있습니다.

- 페이지가 스크롤 되는 도중에 발생하는 이미지나 다른 컨텐츠의 레이지 로딩
- 스크롤 시에, 더 많은 컨텐츠가 로드 및 렌더링되어 사용자가 페이지를 이동하지 않아도 되게 하는 무한스크롤을 구현
- 광고 수익을 계산하기 위한 용도로 광고의 가시성 보고
- 사용자에게 결과가 표시되는 여부에 따라 작업이나 애니메이션을 수행할 지 여부를 결정

이 중 제가 적용한 지연 로딩과 무한스크롤의 예제를 작성해보도록 하겠습니다.

(밑에 코드펜 임베디드하기)
