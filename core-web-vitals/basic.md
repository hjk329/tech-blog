# 코어 웹 바이탈(Core Web Vitals)로 프론트엔드 웹 최적화하기 - 기본편

코어 웹 바이탈(Core Web Vitals)을 바탕으로 웹 최적화를 했던 경험을 공유합니다.    
## 코어 웹 바이탈(Core Web Vitals)이란?
구글은 웹 사이트의 성능을 측정하고 보고하는 다양한 지표와 도구를 제공해 왔습니다.   
2020년에는 우수한 사용자 경험을 제공하기 위한 코어 웹 바이탈(Core Web Vitals)을 발표했습니다.  
코어 웹 바이탈에는 웹 최적화와 관련한 핵심적인 메트릭 3개(LCP, FID, CLS)를 포함합니다.  

> 메트릭이란?

성능은 상대적일 수 있습니다.   
동일한 사이트를 이용하더라도 빠른 네트워크를 사용하는 사용자는 빠르게, 느린 네트워크를 사용하는 사용자에게는 느리게 동작할 수 있기 때문입니다.  

따라서, 구글은 성능 측정에 관한 상대성을 줄이고, 정량적으로 측정 가능한 객관적인 기준에서 성능을 비교하고자 했습니다. 이러한 객관적인 기준을 '메트릭' 이라고 합니다.

2020년을 기준으로 코어 웹 바이탈을 구성하는 메트릭은 ***LCP, FID, CLS*** 입니다.  

각 메트릭을 자세히 알아보겠습니다. <br><br>

## LCP(Largest Contentful Paint, 최대 콘텐츠풀 페인트)란?
로딩 성능을 측정하는 지표로서, 페이지가 처음으로 로드를 시작한 시점을 기준으로 뷰포트 내에 있는 가장 큰 이미지 또는 텍스트 블록의 렌더링 시간을 보고합니다.  

> 뷰포트란 무엇일까요?  
뷰포트(viewport)는 현재 화면에 보여지고 있는 다각형(보통 직사각형)의 영역을 의미합니다.  

우수한 사용자 경험을 제공하기 위해서는 2.5초 이내에 LCP가 발생해야 합니다.  

> 로딩 성능을 측정하기 위해서 가장 큰 콘텐츠의 렌더링 시기를 파악하는 이유가 무엇일까요?

<br>
기본적으로 로딩 성능을 측정하기 위해서 DOMContentLoaded, Load 와 같은 메트릭을 사용하곤 했습니다.  

웹 페이지가 로딩될때 DOMContentLoaded, load 이벤트가 발생하는데 해당 이벤트가 발생하는 시점으로 로딩 성능을 측정하려 했던 것입니다.  

DOMContentLoaded 이벤트는 HTML과 CSS 파싱이 끝나는 시점에 발생합니다.  
load 이벤트는 HTML상에 필요한 모든 리소스가 로드된 시점에 발생합니다.  

따라서, DOMContentLoaded 이벤트가 발생한 후에 load 이벤트가 발생합니다.  

크롬 개발자 도구의 Network 패널 하단에서도 DOMContentLoaded, Load 메트릭을 확인할 수 있습니다.  

![DOMContentLoaded,Load](/assets/images/core-web-vitals/DOMContentLoaded,Load.png)
<br><br>

그러나, SPA(Single Page Application)의 등장으로 DOMContentLoaded, Load 메트릭으로는 제대로된 로딩 성능을 측정하기 어려워졌습니다.   
SPA는 적은 양의 HTML로 DOMContentLoaded, load 이벤트가 일찍이 발생할 수는 있지만, 해당 이벤트가 발생한 이후에 수많은 스크립트가 실행되기 때문입니다.   
예를 들어, CRA로 프로젝트를 생성하면 public 디렉토리 밑에 index.html 이 아래와 같이 구성됩니다.    


```javascript
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <!--
      Notice the use of %PUBLIC_URL% in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  </body>
</html>
```


HTML 파일은 단순히 'root' 라는 아이디를 가진 div를 리턴할 뿐입니다.   
리액트는 자바스크립트 코드를 해당 HTML 파일에 **주입**하여 브라우저가 코드를 실행하도록 합니다.   
따라서, HTML 파일이 빠르게 파싱되고 필요한 리소스를 불러왔다고 하더라도, 아직 스크립트를 실행하느라 사용자가 실제로 보고 있는 화면은 아직 로딩중일 수가 있습니다.   
이에 따라 DOMContentLoaded, Load 메트릭으로 로딩 성능을 측정하기에는 부적절하다고 판단되었습니다.   

또한, FCP(First Contentful Paint, 최초 콘텐츠풀 페인트)이라는 메트릭은 로딩 경험의 ‘시작’ 부분만을 포착하기 때문에 전반적인 로딩 성능을 측정하기에 부적절하다고 판단되었습니다.   

이외에도 Lighthouse에서 제공하는 FMP(First Meaningful Paint, 유의미한 최초 페인트) 및 SI(Speed Index, 속도 인덱스)와 같은 메트릭이 있지만, 메인 콘텐츠가 로드된 시점을 식별하기에 부적절하다고 판단되었습니다.   


따라서, 구글은 W3C Web Performance Working Group의 토론 및 Google에서 수행한 연구를 바탕으로 ***페이지의 메인 콘텐츠가 로드되는 시기를 측정하는 방법으로 가장 큰 요소가 렌더링된 시기를 확인***하는 것을 고안했습니다.

### 좋은 LCP 점수는 무엇일까요?
LCP가 2.5초 이내에 발생하는 것이 권장됩니다.


### LCP 측정을 위해 고려되는 요소는 무엇인가요?
- `<img>` 요소
- `<svg>` 요소 내부의 `<img>` (`<svg>` 요소는 현재 LCP 후보로 간주되지 않습니다.)
- `<video>` 요소
- url() 함수를 통해 로드된 배경 이미지가 있는 요소
- 텍스트 노드 또는 기타 인라인 수준 텍스트 하위 요소를 포함하는 블록 수준 요소

### 요소의 크기는 어떻게 결정될까요?
LCP에 대해 보고된 요소의 크기는 일반적으로 ‘뷰포트’ 내에서 사용자에게 표시되는 크기입니다.   
만약 요소가 뷰포트 외부로 확장되거나, 잘려서 보이지 않는 부분은 요소의 크기로 포함되지 않습니다.   
만약 기본 크기(Intrinsic size)에서 크기가 조정된 이미지 요소가 있다면 가시적 크기와 기본 크기 중 ‘더 작은 것’이 보고됩니다.   
예를 들어, 기본 크기보다 작게 축소한 이미지라면 축소된 만큼의 크기가 보고되겠고, 기본 크기보다 크게 확장한 이미지라면 확장한 이미지보다 더 작은 기본 크기가 보고됩니다.   
텍스트 요소의 경우 텍스트 노드의 크기만 고려됩니다. 또한, 모든 요소에 대해 CSS를 통해 적용된 여백, 안쪽 여백, 테두리는 고려되지 않는다고 합니다.

### LCP는 언제 보고될까요?
브라우저가 첫 번째 프레임을 그리는 즉시 최대 콘텐츠 요소를 식별하는 PerformanceEntry를 디스패치합니다.   
그리고 이후 프레임이 렌더링된 후에는 최대 콘텐츠풀 요소가 변경될 때마다 또다른 PerformanceEntry를 디스패치합니다.   

우선 PerformanceEntry에 대해 간력히 알아보겠습니다.  

> PerformanceEntry란 무엇인가요?

Web API 에서 제공하는 PerformanceEntry 객체는 performance timline 상의 단일 성능 수치를 캡슐화합니다.   
PerformanceEntry 객체를 생성하는 방법은 아래와 같습니다.   

- 메소드를 통해 ***직접적***으로 생성할 수 있습니다
`performance.mark()` 메소드 혹은 `performance.measure()` 메소드를 사용하여 성능을 측정하고 싶은 특정 포인트에서 직접적으로 해당 객체를 생성해서 확인할 수 있습니다.

- 로딩시 ***간접적***으로 생성되기도 합니다
이미지와 같이 리소스를 로딩할때 PerformanceEntry 객체가 생성됩니다.

<br>
PerformanceEntry 객체의 프로퍼티로는 name, entryType, startTime, duration 이 있습니다.   
즉, 특정 entryType 에 대해 성능 메트릭이 처음으로 시작되는 시기(PerformanceEntry가 생성되는 시기), 성능 이벤트가 지속되는 시기를 파악할 수 있습니다.   

정리하자면, PerformanceEntry 객체를 통해 최대 콘텐츠풀 요소를 감지할 수 있습니다.   
정확한 분석을 위해 가장 최근에 디스패치된 PerformanceEntry를 사용합니다.   

처음에 최대 콘텐츠풀 요소로 간주되었다고 하더라도 이후에 다른 더 큰 요소가 렌더링 완료되는 즉시 다른 PerformanceEntry를 통해 더 큰 요소가 최대 콘텐츠풀 요소로 보고됩니다.   
만약 현재 최대 콘텐츠풀 요소가 뷰포트에서 제거되더라도 이보다 더 큰 요소가 렌더링되지 않는 이상 해당 요소가 최대 콘텐츠풀 요소로 유지됩니다.

### 요소 레이아웃 및 크기 변경은 어떻게 처리될까요?
***오직 뷰포트에서 요소의 처음 크기와 위치만 고려합니다.***  
즉, 렌더링 이후에 요소의 크기나 위치를 변경해도 새 LCP 후보로 생성되지 않습니다.   
반대로 처음에 화면 밖에서 렌더링된 이지미는 보고되지 않을 수 있습니다.  
또한, 처음에 뷰포트에서 렌더링된 요소가 아래로 밀려 뷰에서 벗어나더라도 ‘초기’ 뷰포트 내에서의 크기를 보고합니다.  


## FID(First Input Delay, 최초 입력 지연)란?
사용자와 사이트의 상호작용을 측정하는 메트릭입니다.  
FID는 유저와 페이지가 처음으로 상호작용하는 순간부터 (예를 들어, 링크를 클릭하거나, 버튼을 탭하는 등) 브라우저가 해당 인터렉션에 대한 응답으로 실제로 이벤트 핸들러 처리를 시작하기까지의 시간을 측정합니다.


> FID는 사용자가 응답하지 않는 페이지와 상호 작용할때 느끼는 경험을 수량화하기 때문에 중요한 사용자 중심 메트릭입니다.  

### 좋은 FID 점수는 무엇일까요?
우수한 사용자 경험을 제공하기 위해서는 사이트의 최초 입력 지연 값이 100밀리초 이하가 되어야 합니다.   

### FID는 왜 발생할까요?
일반적으로 입력 지연은 브라우저의 메인 스레드가 다른 작업을 하고 있어서 사용자에게 응답할 수 없기 때문에 발생합니다.  
대개 브라우저가 큰 자바스크립트 파일을 파싱하고 실행하느라 바쁜 경우에 입력 지연이 발생할 수 있습니다.  

예를 들어, CSS, JS 파일을 네트워크 요청으로 다운로드한 이후에 메인 스레드에서 처리중이라면 사용자의 입력에 바로 응답할 수 없습니다.  
여기에서 FID의 특이점을 발견할 수 있습니다.  
`사용자에 따라서` FID 값에 차이가 있을 수 있는 점입니다.  
왜냐하면 메인 스레드가 바쁘게 작업 중일때 입력 이벤트가 발생했다면 브라우저가 바로 응답할 수 없어서 FID 시간이 상대적으로 늘어났을 것입니다.  
하지만, 메인 스레드가 유휴 상태에 있었을때 사용자가 입력 이벤트를 발생시켰다면 브라우저가 보다 신속하게 응답했을 수도 있습니다.  

따라서, FID를 분석할때는 그 분포값을 살펴보는 것이 중요하겠습니다. 

### 상호 작용에 이벤트 리스너가 없으면 FID를 어떻게 측정할까요?
***이벤트 리스너가 등록되지 않은 경우에도 FID는 측정됩니다.***  


FID는 이벤트 리스너를 기준으로 측정되지 않고, 메인 스레드가 유휴 상태인지를 기준으로 측정됩니다.  
브라우저가 사용자의 입력에 응답하기 위해서는 메인 스레드가 반드시 유휴 상태가 되어야 하기 때문입니다.


예를 들어, 사용자가 다음 HTML 요소에 입력한 것에 대해 브라우저가 응답하기 위해서는 메인 스레드에서 진행중이던 작업이 완료되어야 합니다.
- 텍스트 필드, 테크박스, 라디오 버튼 (`<input>` , `<textarea>`)
- 선택 드롭다운(`<select>`)
- 링크 (`<a>`)

### 최초 입력만 고려하는 이유가 무엇일까요?
FID는 `최초` 입력 지연을 측정하는 메트릭입니다.  
최초 입력 지연은 사용자에게 사이트에 대한 첫인상을 제공하는 중요한 요소입니다.  
또한, 페이지 로드가 늦어져서 FID 점수가 낮은 경우에 FID를 개선함으로써 페이지 로드 중에 발생하는 사용자 경험과 관련된 여러 문제를 자연스럽게 해결할 수 있습니다. 

따라서, 사용자에게 보다 안정적이고 우수한 웹 사이트 경험을 제공하기 위해서 FID는 최초 입력 지연을 측정합니다.  


### 최초 입력으로 간주되는 이벤트는 무엇인가요? 
클릭, 탭 및 키 누름과 같은 이벤트 입니다.  
스크롤이나 확대 및 축소는 최초 입력 이벤트로 간주되지 않습니다.


### 사용자가 사이트와 상호 작용하지 않으면 어떻게 될까요?
모든 사용자가 사이트에 방문해서 상호작용을 하는 것은 아닙니다.  
단순히 사이트를 방문했다가 나갈 수도 있습니다.  
이러한 경우 해당 사용자에게는 FID 값이 없습니다.  
또한, 같은 사이트이더라도 메인 스레드의 유휴 상태 여부에 따라서 FID 값이 높을 수도 있고, 낮을 수도 있습니다.  
메인 스레드가 바쁘게 작업중일때 입력 이벤트를 발생시켰다면 조금 기다려야겠고, 메인 스레드가 유휴 상태일때 입력한 사용자가 있다면 보다 빠르게 응답 받을 수 있기 때문입니다.  
따라서, FID 값은 사용자에 따라서 없을 수도, 높을 수도, 낮을 수도 있습니다.  

### 입력 지연만 고려하는 이유는 무엇일까요?
FID는 `이벤트 처리의 지연`만 측정합니다.  
이벤트를 처리하는 데 소요된 총 시간을 측정하지 않습니다.
이벤트 처리하는 시간 자체를 메트릭으로 선정하면 해당 메트릭은 개선하지만, 실제 응답 지연은 더 늘어나는 사례를 염려했기 때문입니다.  
예를 들어, 이벤트 핸들러 로직을 `setTimeout()` 또는 `requestAnimationFrame()` 등으로 감싸고, 콜백 함수로 이벤트 핸들러를 전달하여 이벤트와 관련된 작업을 분리할 수도 있습니다.  
이러한 경우 이벤트 처리 시간에 대한 메트릭 점수는 높아질 수 있지만, 실제 사용자가 응답 받는 시간은 더욱 느려집니다.  
따라서, FID는 이벤트 처리 시간 자체가 아닌, 입력 지연을 측정합니다.

## CLS(Cumulative Layout Shift, 누적 레이아웃 이동)란 무엇일까요?
CLS는 Cumulative Layout Shift의 약자로 누적 레이아웃 이동을 의미하는 메트릭입니다.  

웹 사이트에 접속한 이후에 웹 폰트가 변경되거나, 보이지 않았던 이미지나 비디오, 광고 등이 화면 위로 불쑥 올라오는 경험이 있으신가요?  
CLS는 바로 위와 같은 예기치 않은 레이아웃 이동에 대한 레이아웃 이동 점수 중에서 가장 큰 버스트를 나타냅니다.  
### 좋은 CLS 점수는 무엇인가요?
좋은 사용자 경험을 제공하기 위한 CLS 점수는 0.1 이하입니다.  

### CLS 점수는 어떻게 계산되나요?
CLS는 `뷰포트 내`의 가시적 요소가 두 프레임 사이에서 시작 위치가 변경될 때마다 보고됩니다.  
레이아웃 이동 점수는 영향분율(impact fraction)과 거리분율(distance fraction)이라는 측정값으로 계산됩니다.  

`layout shift score(레이아웃 이동 점수) = impact fraction(영향분율) * distance fraction(거리분율)`  

- 영향분율

영향분율은 불안정 요소가 두 프레임 사이 뷰포트 영역에 미치는 영향을 측정합니다.  
![impact fraction](/assets/images/core-web-vitals/cls-impact-fraction.avif)
***이미지 출처: [Cumulative Layout Shift(누적 레이아웃 이동, CLS)](https://web.dev/cls/#%EC%98%81%ED%96%A5%EB%B6%84%EC%9C%A8)***

첫번째 프레임에는 뷰포트의 절반을 차지하는 요소(회색 상자)가 있습니다다.  
두번째 프레임에는 회색 상자가 뷰포트 높이의 25% 만큼 아래로 이동했습니다.  
빨간색 점선 직사각형은 두 프레임 모두에서 회색 상자의 **가시 영역**을 합한 영역입니다.  
이 경우 뷰포트의 75%를 차지하므로 영향분율은 0.75가 됩니다.  
참고로, 영향분율을 계산할때 보이지 않는 영역(뷰포트 밖으로 이동)은 고려되지 않습니다.  

- 거리분율
거리분율은 프레임에서 불안정 요소가 수평 또는 수직으로 이동한 최대 거리의 뷰포트를 가장 큰 치수로 나눈 값입니다.  
이전의 예시 이미지를 다시 살펴봅시다.  
![distance fraction](/assets/images/core-web-vitals/cls-impact-fraction.avif)
***이미지 출처: [Cumulative Layout Shift(누적 레이아웃 이동, CLS)](https://web.dev/cls/#%EC%98%81%ED%96%A5%EB%B6%84%EC%9C%A8)***


위 이미지에서 가장 큰 뷰포트 수치는 너비가 아닌 높이입니다.  
또한, 불안정 요소인 회색 상자가 이동한 거리는 뷰포트 높이의 약 25%입니다.  
따라서, 거리분율은 0.25가 됩니다.  


이에 따라 영향분율은 0.75, 거리분율은 0.25이므로 레이아웃 이동 점수는 `0.75 * 0.25 = 0.1875` 입니다.  


**CLS는 뷰포트 내에서 기존에 보였던 요소의 `시작 위치`가 변경될때 측정됩니다**  

또 다른 예시를 살펴봅시다!


![cls](/assets/images/core-web-vitals/cls-layout-shift.avif)
***이미지 출처: [Cumulative Layout Shift(누적 레이아웃 이동, CLS)](https://web.dev/cls/#%EC%98%81%ED%96%A5%EB%B6%84%EC%9C%A8)***

위와 같이 DOM에 없던 'Click Me!' 라는 주황색 버튼이 생겼을때 불안정 요소는 녹색 상자입니다.  
회색상자와 주황색 버튼 모두 `시작 위치`가 변경되지 않았기 때문입니다. 
시작 위치에 영향을 받은 요소는 녹색상자입니다.  
이에 대한 영향분율은 뷰포트의 50% 에 해당하고, 거리분율은 약 14% 라고 하겠습니다.  
따라서, 레이아웃 이동 점수는 `0.5 * 0.14 = 0.07`이 됩니다.  

### 모든 레이아웃 이동은 나쁜걸까요?
그렇다면 모든 레이아웃 이동을 지양해야 할까요?  
구글은 이에 대해  `사용자가 예상하지 않은 경우에만` 부적절하다고 판단합니다.  
예를 들어서, 사용자가 링크를 클릭해서 어떠한 요소가 뷰포트 위로 올라온다던지, 검색 상자에 입력함으로써 발생하는 레이아웃 이동은 사용자와 페이지의 상호작용으로 인해 발생합니다.  
이러한 경우 레이아웃 이동의 원인이 사용자에게 명확하게 보이기 때문에 부적절하다고 간주되지 않습니다.  

참고로, 사용자와의 상호작용시 레이아웃 이동을 해야 한다면 보다 자연스러운 경험을 제공하기 위해 아래 방법을 사용할 수도 있습니다.  
- `height` 및 `width` 속성을 변경하는 대신에 `transform: scale()`을 사용합니다.
- 리플로우를 야기하는 `top`, `bottom`, `left`, `right` 속성 대신에  `transform: translate()`을 사용하여 점진적인 레이아웃 이동 경험을 제공합니다.



여기까지 코어 웹 바이탈이 무엇이고, 코어 웹 바이탈을 구성하는 3개의 메트릭 LCP, FID, CLS에 대해 알아보았습니다.  
그렇다면 각각의 메트릭을 측정하는 기준과 도구는 무엇일까요?


## 코어 웹 바이탈 측정 기준
코어 웹 바이탈은 위 3개의 메트릭이 각각 ***75번째 백분위수***인지를 기준으로 웹 사이트의 사용자 경험이 양호한지 아닌지를 판단합니다.  
이는 웹 사이트의 방문 횟수가 100회라고 했을때 총 75회의 사용자 경험이 양호하면 관련 메트릭이 좋은 상태라고 판단하고, 그렇지 않으면 해당 메트릭이 열악한 상태로 판단한다는 의미입니다.   

> 왜 75번째 백분위수를 선택했을까요?

구글은 수많은 연구를 통해 75번째 백분위수가 메트릭이 양호한지 아닌지를 가늠할 수 있는 합리적인 숫자라고 판단했다고 합니다.   

만약 95번째 백분위수를 사용한다면 해당 메트릭이 양호한 상태인지를 더욱 확실하게 알 수도 있습니다.  
예를 들어, 75%의 경험보다 95%의 경험이 우수하다면 목표 수준을 도달했는지 더욱 신빙성을 가질 수 있겠습니다.  

하지만, 이렇게 되면 중앙값에서 떨어져 있는 이상값을 놓칠 우려가 있습니다. 총 100개 샘플에 대해 95번째 백분위수를 기준으로 한다면 이상값의 영향을 받는 샘플은 단 5개 뿐이기 때문입니다.   
하지만, 75번째 백분위수를 기준으로 하면 개선할 여지가 있는지 살펴볼 이상값이 25개가 됩니다.   
즉, 중앙값에서 떨어진 이상값들을 적당히 보고하면서도 안정된 수준의 경험을 제공하는지를 알 수 있는 기준점으로 75번째 백분위수가 가장 적절하다고 판단한 것입니다.  

### 코어 웹 바이탈 측정 도구
코어 웹 바이탈을 측정할 수 있는 도구는 다양합니다. 각 도구를 이용하여 우리 프로덕트의 메트릭을 측정해보았습니다.   

#### Lighthouse
LCP, CLS를 측정할 수 있습니다. Lighthouse에서 측정하는 TBT(Total Blocking Time, 총 차단 시간)은 FID와 관련이 깊어서 참고할 수 있습니다.   
![](/assets/images/core-web-vitals/lighthouse.png)
<br><br><br><br>

#### PageSpeed Insights
모바일, 데스크톱 장치 모두에서의 성능을 측정합니다. LCP, FID, CLS 메트릭 모두 측정할 수 있으며, 코어 웹 바이탈 평가 합격 및 불합격 여부도 표시됩니다.   
![](/assets/images/core-web-vitals/page-speed-insights.png)
<br><br><br><br>

#### Chrome DevTools
Performance 패널을 통해 CLS를 파악할 수 있습니다. 특히, CLS 점수가 열악할때 Experience 섹션이 생기기도 합니다. <br> 또한, 패널 하단의 TBT 점수 FID를 하는 용도로 사용할 수도 있습니다.   
![](/assets/images/core-web-vitals/performance-2022-10-20.png)
<br><br><br><br>

#### Google Search Console
코어 웹 바이탈이라는 섹션에서 페이지가 빠른지, 느린지 등의 정보를 파악할 수 있습니다.   
![](/assets/images/core-web-vitals/google-search-console.png) 
<br><br><br><br>

#### Web Vitals 확장 프로그램
크롬 웹 스토어에서 설치하여 사용한다. 코어 웹 바이탈의 모든 메트릭을 실시간으로 측정하여 보여줍니다. <br><br>
![](/assets/images/core-web-vitals/web-vitals-extension.png)
<br><br><br><br>

참고로, [web.dev의 측정 섹션](https://web.dev/measure/)에서도 코어 웹 바이탈 측정을 지원했는데 최근 PageSpeed Insights 로 합쳐졌다고 합니다. <br><br><br><br>  

## 참고한 아티클
[Web Vitals](https://web.dev/vitals/)  
[The business impact of Core Web Vitals](https://web.dev/vitals-business-impact/) 
[Largest Contentful Paint(최대 콘텐츠풀 페인트, LCP)](https://web.dev/lcp/)    
[First Input Delay(최초 입력 지연, FID)](https://web.dev/fid/)  
[Cumulative Layout Shift(누적 레이아웃 이동, CLS)](https://web.dev/cls/)  
[Core Web Vitals 메트릭 및 임계값 정의](https://web.dev/defining-core-web-vitals-thresholds/)  
[Core Web Vitals 측정 도구](https://web.dev/vitals-tools/)  
[성능 최적화](https://ui.toast.com/fe-guide/ko_PERFORMANCE#%EC%84%B1%EB%8A%A5-%EA%B0%9C%EC%84%A0-%EC%A7%80%ED%91%9C)
