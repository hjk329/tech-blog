## 코어 웹 바이탈 최적화하기
코어 웹 바이탈 개념이 생소하신 분들은 아래에서 자세한 정보를 확인하실 수 있습니다 :)  

지난 포스팅에서 코어 웹 바이탈의 개념과 측정 방식 및 도구에 대해 알아보았습니다.  
이번 포스팅에서는 실제로 코어 웹 바이탈의 메트릭을 최적화했던 사례를 공유합니다.  

(막연히 웹 최적화를 위해 활용했던 방법들이 코어 웹 바이탈에 실질적으로 어떤 영향을 미치는지를 봐주시면 좋겠습니다!)


## LCP 최적화하기
우리 프로덕트의 메인 페이지에 첫 진입했을 때 LCP 요소는 최상단의 이미지입니다.  
LCP  향상을 위해 사용한 전략은 아래와 같습니다.

### 이미지를 압축해서 최적화된 이미지를 전달합니다
### CDN을 이용해서 사용자에게 물리적으로 더 가깝게 캐시합니다


프로덕트를 처음 릴리즈 했을 때는 해당 LCP 요소는 로컬에서 저장된 이미지를 보여주는 방식이었습니다. 
또한, 이미지의 용량이 약 23MB로 꽤나 거대한 파일이었습니다.  

코어 웹 바이탈 도큐먼트에서 안내한 대로 webp 파일로 변환해보았습니다.  
그 크기는 536KB(0.53MB)로 기존의 크기인 23MB의 약 25%로 축소되었습니다.  

사내 프로젝트 정적 리소스는 AWS S3에서 관리하고 있습니다.  
따라서, webp로 변환한 이미지 파일을 S3에 업로드한 후 해당 파일을 CloudFront를 통해 가져오도록 처리했습니다.

개발자 도구의 Performance 패널에서 LCP 소요 시간을 측정해본 결과는 아래와 같습니다.  
캐시 데이터의 영향을 받지 않기 위해서 시크릿 모드에서 테스트했습니다.  

- 기존 이미지 요소의 LCP

![lcp 최적화 전](/assets/images/core-web-vitals/lcp-bad.png)  

- webm으로 변환한 이미지 요소의 LCP
![lcp 최적화 후](/assets/images/core-web-vitals/lcp-improved.png)


결과적으로 LCP 소요시간이 1550ms에서 128ms으로 약 90% 이상 단축되었습니다!  

## FID 최적화하기
TBT(Total Blocking Time, 총 차단 시간) 메트릭을 통해 FID를 개선해보았습니다.  
TBT는 코어 웹 바이탈에 포함되는 메트릭은 아니지만, 메인 스레드가 입력 응답을 막는 시간과 관련이 있기 때문에 해당 메트릭을 개선하면 FID도 자연스레 개선할 수 있습니다.  

TBT는 개발자 도구의 퍼포먼스 패널 하단에서 확인하실 수 있습니다.  
양호한 TBT 점수는 30ms 미만입니다.  

최적화하기 전 TBT는 158ms로 양호한 기준인 300ms 보다는 낮았습니다.  

TBT를 단축하기 위해 최적화된 이미지와 비디오를 제공하려 했습니다.  
메인 페이지에서 보여주는 LCP 요소와 비디오 요소 3개가 TBT에 꽤나 큰 영향을 준다고 생각했기 때문입니다.  

LCP 최적화와 같이 최적화된 이미지를 제공하는 것 이외에도 최적화된 비디오를 제공하는 방법을 시도해보았습니다.  
메인 페이지에서는 3개의 비디오 요소를 보여주는데 모두 mp4 파일로 제공되었고, 그 용량이 25MB가 넘는 파일도 있는 등 용량도 꽤나 큰 편이었습니다.  
해당 비디오 요소들은 다행히도(?) 메인 페이지의 하단쯤에 보여지기 때문에 웹 사이트에 처음 접근할때의 뷰포트에서는 보여지지 않기 때문에 메인 페이지에 처음 접근할 당시의 LCP, CLS 메트릭 평가에서는 벗어날 수 있었습니다.  

또한, 엄밀히 말해서 이미지나 비디오 요소는 자바스크립트나 CSS 파일처럼 렌더링을 블로킹하는 요소는 아닙니다.  
하지만, 메인 스레드는 더 적은 요청량과 더 작은 파일을 더 빨리 다운로드할 수 있기 때문에 최적화된 이미지나 비디오 요소를 제공하면 TBT를 단축할 수 있습니다.  
사용자 경험에서 보더라도 이미지나 비디오가 완전히 로드되고 보여지기까지 오랜 시간이 걸린다면 사용자에게 좋은 경험을 제공할 수 없고, 결과적으로 이탈하는 사용자가 많을 것이기 때문에 반드시 개선해야 한다고 생각했습니다.  

### 모던한 비디오 포맷 제공하기

Web dev 도큐먼트에서는 TBT를 개선하기 위한 방법으로 비디오 요소에는 모던한 포맷인 webm로 제공하는 것을 권장합니다.  
webm의 호환성은 아래와 같습니다.  

![can i use webm?](/assets/images/core-web-vitals/webm.png)

직접 검색해보세요!   
[can i use](https://caniuse.com/?search=webm)  





약 25MB였던 mp4포맷의 비디오를 webm 포맷으로 변환하자 그 크기가 2.7MB가 되었습니다.  
또한, 개발자 도구로 살펴보았을때 콘텐츠 다운로드 시간에도 차이가 생겼습니다.  

- mp4 비디오 요소를 렌더링할때 TBT
![mp4](/assets/images/core-web-vitals/mp4-video.webp)


- webm 비디오 요소를 렌더링할때 TBT
![webm](/assets/images/core-web-vitals/webm-video.webp)


can i use를 통해 호환성을 확인하기는 했지만 크로스 브라우징을 더 안전하게 처리하기 위해 아래와 같이 처리해주었습니다.  
이렇게 하면 `<source>` 하위 요소에 따라서 브라우저가 webm을 지원하지 못할때 mp4로 대체합니다.

```javascript
<VideoItem loop autoPlay muted playsInline width={width} height={height}>
    <source src={getAssetUrl(`videos/${path}_${mediaFeature}.webm`)} type="video/webm" />
    <source src={getAssetUrl(`videos/${path}_${mediaFeature}.mp4`)} type="video/mp4" />
</VideoItem>
```

FID 메트릭은 실제 사용자가 있어야 측정할 수 있고, 사용자마다 상대적인 경험을 제공할 수도 있기 때문에 객관적인 측정이 어려울 수 있습니다.  
하지만, 객관적으로 측정이 가능한 TBT가 개선되면 FID 메트릭도 자연스레 개선될 것이므로 위와 같이 FID 점수 향상을 위해 TBT 단축을 시도해볼 수 있겠습니다.  

## CLS 최적화하기

크롬 개발자 도구의 퍼포먼스 패널에 Experiecce 섹션을 본 적 있으신가요?  
사용자에게 불편감을 줄 수 있는 레이아웃 이동이 포착되면 Experiece라는 섹션이 생깁니다.  
바로 이렇게요!  

![cls](/assets/images/core-web-vitals/cls.webp)

마우스를 좀 내려보면 Experience에 나타난 빨간색 블록을 클릭하거나, Summary 탭의 `Related Node` 섹션을 보면 어떤 요소가 CLS를 유발했는지도 알려줍니다!  
저의 경우 네비게이션에서 보여주는 메뉴 텍스트 옆의 화살표 이미지 때문에 레이아웃 이동이 발생하고 있었습니다.  

### 이미지 리소스의 기본값 

제가 직접 웹 사이트에 접근했을때 육안으로 보더라도 렌더링이 완료되면 해당 화살표 이미지가 자리를 이동하면서 레이아웃이 이동되는 것을 확인할 수 있었습니다.  
화살표 이미지의 초깃값을 빈 문자열로 설정한 것이 원인이었습니다.  

화살표 이미지는 path와 스크롤 여부에 따라서 흰색 혹은 검은색 화살표로 렌더링 되어야 합니다.  
화살표 이미지의 src가 빈 문자열이었다가 렌더링이 완료된 후 path 및 스크롤 여부에 따라서 src가 변경되면서 화면이 버벅이는 것처럼 보였습니다.  
따라서, lazy initialization으로 화살표 이미지의 초깃값으로 하얀색 화살표를 가져올 수 있도록 설정했습니다.  

이는 화살표 이미지 요소가 차지할 영역을 미리 확보하는 효과가 있기 때문에 CLS를 개선할 수 있었습니다.  

```javascript
const [caretSrc, setCaretSrc] = useState(() => {
    return chooseCaretSrc(asPath, false);
  });
```

useState를 사용할때 initialState를 인자로 넘기면 초기 렌더링 시에만 사용하는 state를 설정할 수 있습니다.
화살표 이미지의 src는 렌더링된 이후에 path 및 스크롤 여부에 따라서 계속해서 변경될 것이므로 lazy initialization으로 초기 렌더링 시에만 실행될 수 있는 함수를 제공했습니다.  

이렇게 초깃값을 제대로 설정해준 이후에는 개발자 도구에서 보였던 Experiece 섹션이 사라졌습니다!  
또한, 실제로 사이트를 이용하면서도 덜컹거리는 듯한 불편한 움직임도 느껴지지 않았습니다.  

이미지 리소스의 기본값을 설정해주는 것과 비슷한 맥락으로 가로 세로 비율 상자(Ratio Box)로 이미지나 비디오 요소를 감싸줘서 해당 요소들이 필요한 공간을 미리 확보하여 CLS를 최적화하는 방법도 있습니다.  
이미지나 비디오가 로드되는 동안에도 브라우저가 해당 영역에 올바른 공간을 할당할 수 있기 때문입니다.  

RatioBox의 예시 코드는 아래와 같습니다.  

```javascript
import React from 'react';
import styled from 'styled-components';

const RatioBox = ({
  children, width, height, contain,
}) => (
  <Container style={{ paddingTop: `${height / width * 100}%` }}>
    <Inner>
      {children}
    </Inner>
  </Container>
)

const Container = styled.div`
  position: relative;
`;

const Inner = styled.div`
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #eee;

  img,
  video,
  iframe {
    width: 100%;
    height: 100%;
    object-fit: ${(p) => (p.contain ? 'contain' : 'cover')};
  }
`;
export default RatioBox;
```

### 프리로드를 통해 FOUT를 개선
우리 프로덕트는 구글 웹 폰트를 사용하고 있습니다.  
웹 폰트 다운로드가 완료되기 전까지는 기본 글꼴을 보여주다가 다운로드를 마치면 지정된 웹 폰트로 글꼴이 바뀌면서 글자가 반짝거려 보이는 FOUT 현상이 발생했습니다.  

따라서, 아래와 같이 글꼴을 프리로드 할 수 있도록 변경했습니다.
프리로드 속성은 CRP 초기에 폰트를 다운로드하기 때문에 글자의 반짝거림 없이 사용자에게 보다 안정적인 웹 폰트를 제공할 수 있습니다.  

```javascript
<link
  rel="preload"
  href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR&display=swap"
  as="font"
  type="font/woff2"
  crossorigin
/>
```

## 정리
코어 웹 바이탈을 최적화할 수 있는 방법은 위에 소개된 사례 이외에도 다양합니다.  
아래 도큐먼트 링크를 참고하시어 다양한 방법들을 프로젝트에 맞게 시도해보시면 좋겠습니다.  

개인적으로는 코어 웹 바이탈이 무엇인지 이해하고, 실제로 개선해보며 사용자 경험을 좋게 하려면 어떤 점이 중요한지? 어떻게 개선해야하는지 고민하는 좋은 기회가 되었습니다.   
좋은 사용자 경험을 제공하기 위해 오늘도 열심히 노력중이신 모든 개발자분들을 응원합니다!

## 참고한 아티클
[Optimize Largest Contentful Paint](https://web.dev/i18n/en/optimize-lcp/)  
[Optimize First Input Delay](https://web.dev/optimize-fid/)  
[Optimize Cumulative Layout Shift](https://web.dev/optimize-cls/)  
[Keep request counts low and transfer sizes small](https://developer.chrome.com/docs/lighthouse/performance/resource-summary/)  
[Replace animated GIFs with video for faster page loads](https://web.dev/replace-gifs-with-videos/)  
