# Mapbox GL JS


이 파트는 Mapbox GL JS를 정의하고, 웹 매핑 분야에서 이 라이브러리가 대표하는 기술적 패러다임의 전환을 설명하며 무대를 마련합니다.


Mapbox GL JS는 인터랙티브하고, 사용자 정의가 가능하며, 고성능의 웹 지도를 제작하기 위한 자바스크립트 라이브러리입니다.1 이 라이브러리는 클라이언트 측 렌더러(client-side renderer)로, 지도가 서버에서 미리 그려져 전송되는 것이 아니라 사용자의 웹 브라우저에서 동적으로 그려짐을 의미합니다.3

"GL"이라는 명칭은 Mapbox GL이라는 그래픽 라이브러리에서 유래했습니다. 이 라이브러리는 웹 브라우저에서 별도의 플러그인 없이 OpenGL(웹에서는 WebGL)을 활용하여 2D 및 3D 지도를 동적인 시각 그래픽으로 렌더링합니다.3 이는 과거의 DOM(Document Object Model) 기반 라이브러리들과 근본적으로 다른 접근 방식입니다.

Mapbox GL JS는 정적 및 인터랙티브 지도부터 내비게이션, 복잡한 데이터 시각화에 이르기까지 광범위한 애플리케이션을 위해 설계되었습니다.3 특히 시각적으로 뛰어나고 매력적인 지도 경험을 만드는 데 탁월한 능력을 보여줍니다.6


전통적인 라이브러리들이 DOM 요소를 조작하는 방식과 달리, Mapbox GL JS는 HTML `<canvas>` 요소에 WebGL을 사용하여 직접 렌더링합니다.8 GPU에 대한 이 저수준(low-level) API 접근 방식이 바로 Mapbox GL JS의 높은 성능의 원천입니다.9

이 라이브러리의 핵심 강점은 래스터(raster) 이미지 타일 대신 벡터(vector) 타일을 사용한다는 점에 있습니다.1 이는 브라우저가 미리 스타일이 적용된 이미지를 받는 것이 아니라, 점, 선, 폴리곤과 같은 원시 지리 데이터를 받아 실시간으로 스타일을 적용하고 렌더링한다는 것을 의미합니다.

이러한 접근 방식은 상당한 이점을 가져옵니다. 지도 렌더링 속도가 더 빠르고, 데이터 사용량이 훨씬 적으며(한 자료에 따르면 최대 75% 감소), 보간(interpolation)을 통해 부드럽고 연속적인 확대/축소가 가능합니다. 또한, 상점이나 건물 같은 지리적 특성 데이터에 프론트엔드에서 직접 접근할 수 있게 됩니다.8

이러한 WebGL과 벡터 타일의 채택은 단순히 성능 향상에 그치지 않습니다. 이는 웹 지도를 "미리 렌더링된 이미지 뷰어"에서 "실시간 인터랙티브 장면(scene)"으로 변모시키는 아키텍처적 패러다임의 전환을 의미합니다. 전통적인 래스터 지도(초기 구글 지도나 표준 Leaflet 등)는 각 줌 레벨에 맞춰 서버에 미리 스타일이 적용된 이미지 타일을 요청하고, 클라이언트는 이 이미지들을 이어 붙이는 역할을 주로 수행했습니다. 반면, Mapbox GL JS는 벡터 타일을 사용하여 데이터 자체를 클라이언트로 전송합니다.1 지도 스타일 명세(Mapbox Style Specification)에 정의된 스타일 규칙 또한 클라이언트로 전송됩니다.2 그 결과, 클라이언트의 GPU가 WebGL을 통해 데이터에 스타일을 적용하여 최종 지도를 렌더링하는 책임을 지게 됩니다.9

따라서 지도는 프로그래밍 가능한 캔버스가 됩니다. 이는 "런타임 스타일링(runtime styling)" 10과 "데이터 기반 스타일링(data-driven styling)" 11의 기술적 토대가 됩니다. 개발자는 강의 색상, 도로의 너비, 라벨의 가시성 등 지도의 모든 시각적 요소를 서버에 새로운 이미지 타일을 요청할 필요 없이 즉시 변경할 수 있습니다. 이러한 강력한 기능은 필연적으로 복잡성을 수반합니다. 개발자는 이제 Leaflet에서 타일 레이어를 추가하는 것보다 더 추상적인 `Source`, `Layer`, `Style` 시스템을 이해해야 합니다. 힘과 단순성 사이의 이러한 트레이드오프는 웹 매핑 세계의 중심 주제 중 하나입니다.


Mapbox GL JS의 주요 장점과 단점을 요약하면 다음과 같습니다.

- **주요 장점:** 대용량 데이터셋에 대한 타의 추종을 불허하는 성능 12, 비교할 수 없는 수준의 사용자 정의 및 스타일링 기능 7, 3D 지형 및 건물 지원 1, 그리고 풍부하고 잘 정리된 API 문서 7를 꼽을 수 있습니다.
- **주요 단점:** 버전 2.0 이상부터 적용된 독점 라이선스와 특정 공급업체에 대한 종속(vendor lock-in) 문제 15, 잠재적으로 복잡하고 비용이 많이 드는 가격 모델 17, 그리고 스타일링 시스템의 일부 인지된 한계(예: 다른 지오메트리 유형을 겹쳐 표현하기 위한 추가 작업의 필요성, 스타일링이 간단한 콜백 함수가 아닌 별도의 객체로 분리된 점 등)가 있습니다.7


이 파트는 라이브러리의 아키텍처를 해부하여, Mapbox GL JS 애플리케이션을 구상하고 구조화하기 위한 정신적 모델을 제공합니다.


Mapbox 지도는 네 가지 핵심 개념의 상호작용으로 구성됩니다. 이들의 관계를 이해하는 것은 Mapbox GL JS를 효과적으로 사용하기 위한 첫걸음입니다.

- **`Map` 객체:** 모든 프로젝트의 기초가 되는 요소입니다. 페이지 위의 지도를 나타내는 메인 객체로, 지도를 담을 `container` 요소, 초기 `center`(중심점)와 `zoom`(확대/축소 레벨), 그리고 `accessToken`(액세스 토큰)과 같은 필수 파라미터로 초기화됩니다.3

- **`Style` 객체:** 지도의 시각적 외형에 대한 마스터 청사진 역할을 하는 JSON 문서입니다. 배경색, 사용할 데이터 소스, 렌더링할 레이어 등 모든 것을 정의합니다.2 이 문서는(

  [https://docs.mapbox.com/style-spec/)을](https://docs.mapbox.com/style-spec/)을) 따릅니다.

- **`Source` 객체:** 데이터 그 자체입니다. 소스는 지도에 표시될 지리 정보를 제공합니다. Mapbox GL JS는 벡터 타일, 래스터 타일, GeoJSON, 이미지, 비디오 등 다양한 유형의 소스를 지원합니다.2

- **`Layer` 객체:** 소스의 시각적 표현입니다. 레이어는 지정된 소스로부터 데이터를 가져와 지도에 어떻게 스타일을 적용할지 정의합니다 (예: 폴리곤을 위한 `fill` 레이어, 도로를 위한 `line` 레이어, 아이콘과 라벨을 위한 `symbol` 레이어). 레이어는 `layout`과 `paint` 속성을 통해 외형을 제어합니다.3

데이터(`Source`)와 표현(`Layer`)의 엄격한 분리는 이 라이브러리의 동적 기능을 가능하게 하는 핵심 아키텍처 원칙입니다. 이는 최신 웹 개발 패턴(예: HTML 구조와 CSS 스타일링의 분리)을 반영하는 의도적인 설계 선택입니다. 예를 들어, 개발자는 한 국가의 모든 도시를 담은 GeoJSON 파일을 `Source`로 한 번 정의할 수 있습니다.21 이것이 원시 데이터입니다. 그런 다음, 이 단일 

`Source`를 참조하는 여러 개의 `Layer`를 생성할 수 있습니다. 한 `Layer`는 각 도시를 점으로 표시하기 위해 `circle` 유형일 수 있고 22, 다른 

`Layer`는 도시 이름을 표시하기 위해 `symbol` 유형일 수 있습니다.23

이는 데이터는 한 번만 로드되지만, 여러 독립적인 방식으로 시각화될 수 있음을 의미합니다. 따라서 개발자는 라벨을 위한 `symbol` 레이어에 영향을 주지 않고 `circle` 레이어의 `paint` 속성(예: 인구에 따라 원 크기를 크게 만들기)을 프로그래밍 방식으로 변경할 수 있습니다. 이러한 분리(decoupling)는 복잡하고 데이터 기반의 인터랙티브 지도를 효율적이고 관리 가능하게 만듭니다. 이는 데이터 로딩과 스타일링 로직이 뒤섞여 발생할 수 있는 "스파게티 코드"를 방지합니다.


Mapbox GL JS의 진정한 힘은 정적인 지도를 넘어 동적으로 변화하는 지도를 만드는 능력에 있습니다. 이는 '표현식(Expressions)'이라는 강력한 도구를 통해 실현됩니다.

- **Mapbox GL JS 표현식:** 스타일 속성 내에서 동적으로 스타일링 결정을 내리기 위해 사용되는 강력한 리스프(Lisp)와 유사한 구문입니다.24 이를 통해 개발자는 스타일 JSON에 직접 로직을 작성할 수 있습니다.
- **속성 표현식 (Property Expressions):** 지리적 피처(feature) 자체의 속성을 참조하는 표현식입니다. 예를 들어, 각 주(state) 폴리곤의 `population` 속성 값에 따라 색상을 다르게 스타일링할 수 있습니다.11
- **줌 표현식 (Zoom Expressions):** 지도의 현재 줌 레벨을 참조하여 사용자가 지도를 확대하거나 축소함에 따라 스타일이 변경되도록 합니다. 예를 들어, 높은 줌 레벨에서 가시성을 높이기 위해 원을 더 크게 만들 수 있습니다.24
- **조건부 로직 (Conditional Logic):** `case`나 `match`와 같은 표현식을 사용하여 특정 조건에 따라 다른 스타일을 적용함으로써, 단계 구분도(choropleth map)와 같은 정교한 데이터 시각화를 생성할 수 있습니다.11


이 섹션은 신규 사용자를 위한 실용적이고 단계적인 가이드를 제공합니다.


Mapbox GL JS를 사용하기 위한 첫 번째 필수 단계는 액세스 토큰을 확보하는 것입니다.

- **목적:** 액세스 토큰은 필수적입니다. 이는 지도 요청을 개발자의 Mapbox 계정과 연결하여 과금 및 분석에 사용됩니다.3
- **유형:** 클라이언트 측에서 사용되는 공개 토큰(`pk.*`로 시작)과 서버 측 API 호출에 사용되는 비밀 토큰(`sk.*`로 시작)을 구분해야 합니다. 비밀 토큰은 절대로 프론트엔드 코드에 노출되어서는 안 됩니다.26
- **생성 및 보안:** 사용자가 계정 대시보드에서 토큰을 생성하는 방법을 안내합니다.27 공개 토큰에 대한 중요한 보안 조치로 URL 제한 기능을 소개하여, 토큰이 지정된 도메인에서만 작동하도록 보장합니다.26 또한, 보안 모범 사례로 토큰 순환(token rotation)을 설명합니다.27
- **토큰 형식:** 토큰은 헤더, 페이로드(스코프 포함), 서명으로 구성된 JSON 웹 토큰(JWT)이라는 점을 간략히 설명합니다.28


액세스 토큰이 준비되었다면, 프로젝트에 Mapbox GL JS 라이브러리를 추가해야 합니다.

- **방법 1: CDN (콘텐츠 전송 네트워크):** 가장 간단하게 시작하는 방법입니다. HTML 파일에 포함해야 할 `<script>` 및 `<link>` 태그를 보여줍니다.29
- **방법 2: 모듈 번들러 (npm):** 현대 웹 개발의 표준 방식입니다. `npm install mapbox-gl` 명령어를 실행하고, 자바스크립트 파일에서 라이브러리와 관련 CSS를 `import`하는 방법을 보여줍니다.29
- **트랜스파일링 고려사항:** 현대적인 설정에서 중요한 지점입니다. Mapbox GL JS v2 이상은 최신 자바스크립트 구문을 사용하므로 Babel과 같은 트랜스파일러에서 올바르게 처리해야 할 수 있음을 설명합니다. 권장되는 세 가지 전략, 즉 `browserslist` 정렬, `mapbox-gl`을 트랜스파일링에서 명시적으로 제외, 또는 웹 워커를 별도로 로드하는 방법을 제공합니다.29


기본 지도를 표시하기 위한 완전하고 상세한 주석이 달린 코드 예제를 제공합니다. 이는 모든 학습의 기초가 되는 "Hello, World!"입니다.31

- **HTML:** 지도를 담을 `<div>` 컨테이너를 포함한 기본 HTML 구조를 보여줍니다.
- **CSS:** 지도 컨테이너에 크기를 부여하는 데 필요한 CSS를 포함합니다.
- **JavaScript:**
  1. `mapboxgl.accessToken`을 설정합니다.
  2. `new mapboxgl.Map({})`을 인스턴스화합니다.
  3. `container`, `style`(예: `mapbox://styles/mapbox/streets-v12`와 같은 기본 Mapbox 스타일 URL 사용), `center`, `zoom`과 같은 각 필수 파라미터를 설명합니다.3

**기본 지도 표시 예제 (HTML/JS):**

```HTML
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Display a map on a webpage</title>
<meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no">
<link href="https://api.mapbox.com/mapbox-gl-js/v3.12.0/mapbox-gl.css" rel="stylesheet">
<script src="https://api.mapbox.com/mapbox-gl-js/v3.12.0/mapbox-gl.js"></script>
<style>
  body { margin: 0; padding: 0; }
  #map { position: absolute; top: 0; bottom: 0; width: 100%; }
</style>
</head>
<body>
<div id="map"></div>
<script>
  // 중요: 'YOUR_MAPBOX_ACCESS_TOKEN'을 실제 Mapbox 액세스 토큰으로 교체해야 합니다.
  mapboxgl.accessToken = 'YOUR_MAPBOX_ACCESS_TOKEN';
  const map = new mapboxgl.Map({
    container: 'map', // 지도를 표시할 HTML 요소의 ID
    style: 'mapbox://styles/mapbox/streets-v12', // Mapbox에서 제공하는 스타일 URL
    center: [-74.5, 40], // 시작 중심 좌표 [경도, 위도]
    zoom: 9 // 시작 줌 레벨
  });
</script>
</body>
</html>
```

- **React 통합:** React의 보편성을 고려하여, Mapbox의 공식 튜토리얼에서 보여주는 것처럼 `useRef`와 `useEffect` 훅을 사용하여 지도를 React 컴포넌트에 통합하는 병렬 예제를 제공합니다.30

**React 컴포넌트 예제:**

```JavaScript
import React, { useRef, useEffect, useState } from 'react';
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = 'YOUR_MAPBOX_ACCESS_TOKEN';

function App() {
  const mapContainer = useRef(null);
  const map = useRef(null);
  const [lng, setLng] = useState(-74.5);
  const [lat, setLat] = useState(40);
  const [zoom, setZoom] = useState(9);

  useEffect(() => {
    if (map.current) return; // map이 이미 초기화되었다면 중단
    map.current = new mapboxgl.Map({
      container: mapContainer.current,
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [lng, lat],
      zoom: zoom
    });
  });

  return (
    <div>
      <div ref={mapContainer} className="map-container" style={{ height: '100vh' }} />
    </div>
  );
}

export default App;
```


이 파트는 가장 일반적인 지도 제작 작업을 위한 실용적인 튜토리얼 시리즈입니다. 각 섹션은 특정 기능을 마스터하기 위한 코드 예제와 상세한 설명을 제공합니다.


지도 위에 특정 위치를 표시하는 가장 기본적인 방법은 마커와 팝업을 사용하는 것입니다.

- **기본 마커:** `new mapboxgl.Marker().setLngLat(...).addTo(map)`을 사용하여 간단한 기본 마커를 추가하는 방법을 보여줍니다.32
- **사용자 정의 HTML 마커:** 가장 유연한 방법입니다. 사용자 정의 `div` 요소를 만들고, CSS로 스타일을 지정한 후(예: 배경 이미지 사용), 이를 `Marker` 생성자에 전달하는 방법을 시연합니다.32
- **팝업:**
  - 간단한 팝업 생성: `new mapboxgl.Popup().setLngLat(...).setHTML(...).addTo(map)`.33
  - 마커에 팝업 연결하기: `Marker` 인스턴스에 `.setPopup()` 메서드를 사용하여 클릭 시 팝업이 나타나도록 합니다.32

**사용자 정의 마커 및 팝업 코드 예제:**

```JavaScript
// GeoJSON 데이터 예시
const geojson = {
  'type': 'FeatureCollection',
  'features': [
    {
      'type': 'Feature',
      'geometry': {
        'type': 'Point',
        'coordinates': [-77.0323, 38.9131]
      },
      'properties': {
        'title': 'Mapbox DC',
        'description': 'Washington, D.C.'
      }
    }
  ]
};

// 각 피처에 대해 마커와 팝업 추가
for (const feature of geojson.features) {
  // 마커를 위한 커스텀 HTML 요소 생성
  const el = document.createElement('div');
  el.className = 'marker'; // CSS에서 스타일을 지정하기 위한 클래스
  el.style.backgroundImage = `url(https://docs.mapbox.com/help/demos/custom-markers-gl-js/mapbox-icon.png)`;
  el.style.width = `50px`;
  el.style.height = `50px`;
  el.style.backgroundSize = '100%';

  // 팝업 생성
  const popup = new mapboxgl.Popup({ offset: 25 }) // 팝업이 마커에서 약간 떨어지도록 오프셋 설정
   .setHTML(
      `<h3>${feature.properties.title}</h3><p>${feature.properties.description}</p>`
    );

  // 마커를 생성하고 팝업을 연결한 후 지도에 추가
  new mapboxgl.Marker(el)
   .setLngLat(feature.geometry.coordinates)
   .setPopup(popup)
   .addTo(map);
}
```

- **API 데이터로부터 동적 마커 생성 (React):** API(예: USGS 지진 데이터)에서 GeoJSON 데이터를 가져와 React 상태(state)에 저장하고, 각 피처에 대해 동적으로 마커를 생성하는 고급 예제를 제공합니다.35


GeoJSON은 웹에서 지리 데이터를 표현하는 표준 형식이며, Mapbox GL JS는 이를 매우 효과적으로 처리합니다.

- **GeoJSON 소스 추가:** `map.addSource()`를 사용하여 `type: 'geojson'`으로 소스를 추가하고, 데이터를 인라인 객체로 제공하거나 외부 파일 URL로 제공하는 방법을 시연합니다.21
- **폴리곤 스타일링:** GeoJSON 소스의 폴리곤을 시각화하기 위해 `fill` 레이어를 추가하는 방법을 보여줍니다.37
- **외곽선 추가:** 폴리곤 주위에 외곽선을 그리기 위해 *동일한* 소스를 참조하는 두 번째 `line` 레이어를 추가하는 방법을 시연합니다.37
- **여러 지오메트리 처리:** 서로 다른 지오메트리 유형(예: 점과 폴리곤)을 포함하는 단일 GeoJSON 소스를 사용하고, `addLayer` 호출 시 `filter` 옵션(`['==', '$type', 'Polygon']`)을 사용하여 각각 다르게 스타일을 지정하는 방법을 설명합니다.22
- **포인트에 이미지 사용 (심볼 레이어):** 사용자 정의 아이콘이 있는 `symbol` 레이어를 사용하여 포인트를 시각화하는 더 고급 방법을 보여줍니다. 이 방법은 `map.loadImage()`, `map.addImage()`, 그리고 `icon-image` 레이아웃 속성 설정을 포함합니다.23

**GeoJSON 폴리곤 및 외곽선 추가 예제:**

```JavaScript
map.on('load', () => {
  // GeoJSON 데이터를 포함하는 소스 추가
  map.addSource('maine', {
    'type': 'geojson',
    'data': {
      'type': 'Feature',
      'geometry': {
        'type': 'Polygon',
        'coordinates': [
          //... 메인 주의 좌표 배열...
        ]
      }
    }
  });

  // 폴리곤을 시각화하기 위한 채우기(fill) 레이어 추가
  map.addLayer({
    'id': 'maine-fill',
    'type': 'fill',
    'source': 'maine',
    'layout': {},
    'paint': {
      'fill-color': '#0080ff',
      'fill-opacity': 0.5
    }
  });

  // 폴리곤 주위에 외곽선을 그리기 위한 선(line) 레이어 추가
  map.addLayer({
    'id': 'maine-outline',
    'type': 'line',
    'source': 'maine',
    'layout': {},
    'paint': {
      'line-color': '#000',
      'line-width': 3
    }
  });
});
```


Mapbox GL JS의 진가는 사용자와 상호작용하는 동적인 지도를 만들 때 드러납니다.

- **이벤트 처리:** `map.on('event',...)` 패턴을 설명합니다.

  - **클릭 이벤트:** 특정 레이어에서 `click` 이벤트를 수신하고(`map.on('click', 'my-layer', (e) => {... })`), 이벤트 객체(`e.features.properties`)에서 피처 속성에 접근하여 팝업에 표시하는 방법을 보여줍니다.38
  - **마우스 이벤트를 이용한 호버 효과:** `mousemove`, `mouseenter`, `mouseleave`를 사용하여 호버 상호작용을 만드는 방법을 시연합니다.38

- **호버 효과를 위한 피처 상태(Feature State):** 더 성능이 좋은 `feature-state` 메커니즘을 소개합니다. `mousemove`과 `mouseleave`에서 `setFeatureState`를 사용하여 피처에 `hover` 상태(`true` 또는 `false`)를 설정하는 방법을 설명합니다.40 그런 다음, 레이어의 

  `paint` 속성에서 `case` 표현식을 사용하여 `hover` 상태에 따라 스타일(예: `fill-opacity`)을 변경하는 방법을 보여줍니다.40

**피처 상태를 이용한 호버 효과 예제:**

```JavaScript
// 'state-fills' 레이어의 페인트 속성
'paint': {
  'fill-opacity': [
    'case',
    ['boolean', ['feature-state', 'hover'], false], // 'hover' 상태가 true이면
    1, // 불투명도 1
    0.5 // 그렇지 않으면 불투명도 0.5
  ]
}

//...

let hoveredStateId = null;

// 마우스가 'state-fills' 레이어 위로 움직일 때
map.on('mousemove', 'state-fills', (e) => {
  if (e.features.length > 0) {
    if (hoveredStateId!== null) {
      // 이전에 호버된 피처의 상태를 리셋
      map.setFeatureState(
        { source: 'states', id: hoveredStateId },
        { hover: false }
      );
    }
    hoveredStateId = e.features.id;
    // 현재 마우스 아래에 있는 피처의 'hover' 상태를 true로 설정
    map.setFeatureState(
      { source: 'states', id: hoveredStateId },
      { hover: true }
    );
  }
});

// 마우스가 'state-fills' 레이어를 떠날 때
map.on('mouseleave', 'state-fills', () => {
  if (hoveredStateId!== null) {
    // 호버된 피처의 상태를 리셋
    map.setFeatureState(
      { source: 'states', id: hoveredStateId },
      { hover: false }
    );
  }
  hoveredStateId = null;
});
```

- **런타임 스타일링:**
  - **레이어 속성 변경:** 버튼 클릭과 같은 이벤트에 응답하여 `map.setPaintProperty()` 및 `map.setLayoutProperty()`를 사용하여 레이어가 추가된 후에도 레이어의 속성을 변경하는 방법을 보여줍니다.42
  - **전체 스타일 전환:** `map.setStyle()`을 사용하여 "밝은" 테마에서 "어두운" 테마로 기본 지도 스타일을 동적으로 완전히 변경하는 방법을 시연합니다.42


이 섹션은 Mapbox GL JS를 가장 중요한 동반 도구인 Mapbox Studio의 맥락에서 설명합니다.


Mapbox GL JS가 코드로 지도를 제어하는 엔진이라면, Mapbox Studio는 그 지도의 시각적 디자인을 책임지는 아틀리에입니다.

- **Mapbox Studio란?** Studio는 사용자 정의 지도 스타일을 만들기 위한 웹 기반의 시각적 편집기입니다.44 Studio에서 생성된 스타일은 Mapbox GL JS에서 소비됩니다.
- **작업 흐름:**
  1. **데이터 업로드:** 사용자가 자신의 데이터(예: GeoJSON, CSV)를 업로드하면 Mapbox가 이를 타일셋(tileset)으로 변환하는 방법을 안내합니다.24
  2. **스타일 생성:** 기본(Basic)이나 거리(Streets)와 같은 템플릿으로 시작하여 레이어를 수정합니다.46
  3. **레이어 스타일링:** UI를 사용하여 색상, 글꼴, 불투명도를 변경하고, 데이터 기반 및 줌 레벨에 따른 스타일링을 시각적으로 설정합니다.46
  4. **슬롯 사용 (Mapbox Standard):** Mapbox Standard 스타일의 `슬롯(slots)` 개념을 설명합니다. 이는 사용자 정의 레이어를 삽입하기 위한 미리 정의된 위치입니다(예: 라벨 아래, 육지 위).45
  5. **게시:** 스타일을 고유한 스타일 URL을 통해 사용할 수 있도록 만드는 게시 과정을 설명합니다.46
  6. **통합:** 게시된 스타일 URL을 가져와 코드의 `mapboxgl.Map` 생성자에서 사용하는 방법을 보여줍니다.46


이것은 기술에 대한 진지한 약속을 하는 모든 개발자에게 중요한 섹션입니다. Mapbox GL JS 선택의 비즈니스 및 철학적 측면을 다룹니다.


Mapbox의 가격 정책은 유연하지만, 사용 방식에 따라 비용이 크게 달라질 수 있으므로 정확한 이해가 필수적입니다.

- **종량제 (Pay-As-You-Go):** 핵심 모델은 넉넉한 무료 제공량 이후 사용량 기반으로 과금되는 방식입니다.5
- **"맵 로드(Map Load)" 지표:** 웹 지도에 대한 주요 과금 지표가 어떻게 작동하는지 상세히 설명합니다. `mapboxgl.Map` 객체가 초기화될 때마다 "맵 로드" 1회가 계산됩니다.18 무료 제공량은 일반적으로 월 50,000 맵 로드입니다.15
- **가격 등급:** 무료 제공량을 초과한 후 1,000 맵 로드당 비용을 명확하게 설명하는 표를 포함합니다.17
- **아키텍처적 영향:** 이 모델이 전통적인 다중 페이지 웹사이트보다 단일 페이지 애플리케이션(SPA)에 유리하며, CI/CD 환경에서 예상치 못한 비용을 초래할 수 있다는 점을 다시 강조합니다.18


2020년 12월, Mapbox는 웹 매핑 커뮤니티에 큰 파장을 일으킨 결정을 내렸습니다. 이는 Mapbox GL JS의 미래와 사용 방식을 근본적으로 바꾸었습니다.

- **변경 이전 (v1):** Mapbox GL JS v1.x는 허용적인 3-Clause BSD 라이선스 하에 완전한 오픈 소스였습니다.15 개발자들은 Mapbox 계정 없이도 자신의 데이터와 인프라를 사용하여 자유롭게 라이브러리를 사용할 수 있었습니다.
- **변경 시점 (2020년 12월):** Mapbox가 v2.0을 출시하면서 라이선스를 BSD에서 Mapbox 서비스 약관에 귀속되는 독점 라이선스로 변경한 결정적인 순간을 상세히 설명합니다.16
- **새로운 현실 (v2 이상):**
  - v2 이상 버전의 사용은 유효한 Mapbox 액세스 토큰, 즉 활성 Mapbox 계정을 요구합니다.15
  - 100% Mapbox 외부 데이터 소스를 사용하더라도 사용량은 "맵 로드" 단위로 과금됩니다.15 이는 사실상 라이브러리 자체를 유료 제품으로 전환시킨 것입니다.
  - v1 브랜치는 더 이상 적극적으로 개발되지 않으며, 제한된 기간 동안 중요한 보안 수정만 제공됩니다.15
- **커뮤니티 반응:** 영구적인 오픈 소스 프로젝트라고 믿고 제품을 구축했던 개발자 커뮤니티의 충격과 반발을 설명합니다. 많은 이들이 이를 "미끼 상품(bait and switch)" 전략으로 느꼈습니다.16

이 라이선스 변경은 Mapbox가 초기에 오픈 소스 정책을 통해 달성한 시장 지배력을 수익화하기 위한 계산된 비즈니스 전략이었습니다. 이는 "오픈 코어(open core)" 모델이 완전한 독점 플랫폼으로 진화하는 전형적인 사례를 보여줍니다. Mapbox는 우수한 렌더링 라이브러리를 만드는 데 막대한 투자를 하고 이를 오픈 소스로 공개했습니다.50 이는 광범위한 채택을 장려하고 대규모 커뮤니티를 구축하여 기술적 리더로 자리매김하게 했습니다. 일단 시장 지배력을 확보하자, 회사는 지속 가능성을 확보하고 투자를 수익화해야 한다는 압박에 직면했습니다.16 단순히 데이터/API에 대해 요금을 부과하는 것(이미 하고 있던 방식)을 넘어, 클라이언트 측 라이브러리 자체의 사용에 대해서도 요금을 부과하는 전략적 결정을 내렸습니다. 라이선스 변경은 이를 강제하기 위한 법적 장치였습니다. 초기화에 토큰을 요구함으로써, 라이브러리 사용을 계정에 연결하고 과금할 수 있게 된 것입니다.18 이 사건의 3차적 영향은 신뢰의 결손을 초래하고 실행 가능한 오픈 소스 대안의 등장을 강제했다는 점입니다. 이는 개발자들에게 단일 공급업체 오픈 소스 프로젝트에 의존하는 것의 위험에 대한 경고성 사례로 작용합니다.


Mapbox의 라이선스 변경은 커뮤니티에 큰 도전이었지만, 동시에 새로운 기회를 창출했습니다. 그 결과물이 바로 MapLibre GL입니다.

- **포크(Fork)의 탄생:** MapLibre GL을 Mapbox GL JS의 마지막 오픈 소스 버전(v1.x)에서 파생된, 커뮤니티 주도의 직접적인 포크(fork)로 소개합니다.15
- **가치 제안:** MapLibre GL의 목표는 Mapbox GL JS v1 API 및 Mapbox 스타일 명세와의 호환성을 유지하면서, 독점 라이선스나 필수 Mapbox 토큰 없이 사용할 수 있는 자유롭고 오픈 소스인 대안이 되는 것입니다.15
- **누가 사용해야 하는가?** MapLibre GL은 오픈 소스를 우선시하거나, 특정 공급업체에 대한 종속을 피하고 싶거나, Mapbox에 사용료를 지불하지 않고 전체 매핑 스택을 자체 호스팅해야 하는 프로젝트에 가장 적합한 선택으로 자리매김합니다.13


이 섹션은 개발자가 정보에 입각한 선택을 할 수 있도록 명확하고 기능 기반의 비교를 제공합니다.


개발자의 매핑 라이브러리 선택은 위치 기반 애플리케이션에서 가장 중요한 아키텍처 결정 중 하나입니다. 이 표는 수많은 출처 13의 방대한 정보를 단일 참조 자료로 압축하여 그 가치가 매우 높습니다. 개발자는 성능, 사용자 정의, 사용 편의성, 비용 간의 트레이드오프를 신속하게 평가하고, 라이브러리의 강점을 프로젝트의 특정 요구 사항과 일치시킬 수 있습니다.

| 기능                    | Mapbox GL JS                     | Google Maps Platform    | Leaflet              | MapLibre GL       |
| ----------------------- | -------------------------------- | ----------------------- | -------------------- | ----------------- |
| **렌더링 기술**         | WebGL (벡터)                     | WebGL/래스터            | DOM (래스터)         | WebGL (벡터)      |
| **핵심 강점**           | 타의 추종을 불허하는 사용자 정의 | 데이터 및 서비스 생태계 | 단순성 및 확장성     | 오픈 소스 파워    |
| **사용자 정의**         | 매우 높음                        | 보통                    | 높음 (플러그인 통해) | 매우 높음         |
| **성능(대용량 데이터)** | 뛰어남                           | 좋음                    | 보통                 | 뛰어남            |
| **라이선스**            | 독점                             | 독점                    | BSD (오픈 소스)      | BSD (오픈 소스)   |
| **비용 모델**           | 종량제 (맵 로드)                 | 종량제 (API 호출)       | 무료 (라이브러리)    | 무료 (라이브러리) |


이 비교의 핵심은 **힘과 단순성**의 대결입니다.

- **Mapbox GL JS:** 고성능, WebGL 기반, 벡터 우선, 가파른 학습 곡선, 복잡한 시각화와 깊이 있는 스타일링을 위해 제작되었습니다.51
- **Leaflet:** 경량, DOM 기반, 래스터 우선(플러그인을 통해 벡터 지원), 배우기 쉬움, 방대한 플러그인 생태계, 거대한 데이터셋의 성능이 최우선 순위가 아닌 간단한 지도에 이상적입니다.13


이 비교는 **디자인 제어와 데이터 생태계**에 초점을 맞춥니다.

- **Mapbox GL JS:** Mapbox Studio를 통한 우수한 지도 디자인 제어, 유연한 데이터 통합, 사용자 정의 데이터 시각화에 뛰어난 벡터 중심 아키텍처를 제공합니다.14
- **Google Maps:** 비교할 수 없는 POI(관심 지점) 데이터, 강력한 지오코딩 및 길찾기 API, 강력한 브랜드 인지도를 자랑하지만, 역사적으로 깊이 있는 시각적 사용자 정의 측면에서는 유연성이 떨어집니다.14 Google의 데이터는 Google 지도가 아닌 다른 지도에서는 사용할 수 없습니다.54


이 마지막 파트는 전체 보고서를 종합하고 명확한 의사 결정 프레임워크를 제공합니다.


어떤 매핑 도구를 선택할지는 프로젝트의 목표와 제약 조건에 따라 달라집니다. 다음은 최종 결정을 돕기 위한 가이드라인입니다.

- **Mapbox GL JS를 선택해야 할 때:** 프로젝트의 핵심 가치가 독특하고, 브랜드화되었으며, 고성능의 지도 경험일 때. 깊이 있는 스타일링 제어와 3D 기능이 필요하며, 독점 라이선스와 종량제 가격 모델에 불편함이 없을 때 선택합니다.
- **MapLibre GL을 선택해야 할 때:** GL 렌더링 엔진의 강력함과 성능이 필요하지만, 오픈 소스 솔루션이 필수적이거나, 특정 공급업체에 대한 종속을 피하고 싶거나, 전체 스택을 자체 호스팅할 계획일 때 선택합니다.
- **Leaflet을 선택해야 할 때:** 프로젝트에 간단하고, 안정적이며, 가벼운 인터랙티브 지도가 필요할 때. 사용 편의성, 큰 플러그인 생태계, 그리고 무료 오픈 소스 라이브러리가 최우선 순위일 때 선택합니다.
- **Google Maps를 선택해야 할 때:** 애플리케이션이 Google의 생태계, 특히 세계적 수준의 검색, POI 데이터, 스트리트 뷰에 크게 의존할 때. 깊이 있는 지도 제작 사용자 정의보다 브랜드 인지도와 표준 사용 사례에 대한 통합 용이성이 더 중요할 때 선택합니다.


클라이언트 측, GPU 가속 렌더링은 웹 매핑의 대세로 자리 잡았습니다. 이 기술은 앞으로도 계속 발전하며 더욱 풍부하고 몰입감 있는 경험을 제공할 것입니다.

Mapbox의 라이선스 변경에 대한 반응으로 등장한 MapLibre GL의 커뮤니티 주도 개발은 주요 플랫폼의 독점적 성격에 대한 중요한 견제 장치 역할을 합니다. 이는 건강한 생태계를 위해 오픈 소스 대안의 존재가 얼마나 중요한지를 보여줍니다.

결론적으로, 현대 개발자는 강력하지만 복잡한 선택지들을 마주하고 있습니다. 기술적, 재정적, 그리고 철학적 트레이드오프를 이해하는 것이 그 어느 때보다 중요해졌으며, 이 보고서가 그 길을 안내하는 신뢰할 수 있는 나침반이 되기를 바랍니다.


1. Mapbox API(Mapbox GL JS) - 개발 성장 일ブl - 티스토리, accessed July 6, 2025, https://bom-na.tistory.com/145
2. mapbox/mapbox-gl-js: Interactive, thoroughly customizable maps in the browser, powered by vector tiles and WebGL - GitHub, accessed July 6, 2025, https://github.com/mapbox/mapbox-gl-js
3. Mapbox GL JS | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/guides/
4. 5 JavaScript mapping APIs compared - LogRocket Blog, accessed July 6, 2025, https://blog.logrocket.com/javascript-mapping-apis-compared/
5. Mapbox Maps, accessed July 6, 2025, https://www.mapbox.com/maps
6. Create 3D and Dynamic Web Maps with Mapbox GL JS, accessed July 6, 2025, https://www.mapbox.com/mapbox-gljs
7. [지도 데이터 시각화] Part 5. Mapboxgl 살펴보기 - 하나씩 점을 찍어 나가며, accessed July 6, 2025, https://dailyheumsi.tistory.com/145
8. An efficient webGL map with Mapbox-gl-js and react-mapbox-gl | by Alex R | Medium, accessed July 6, 2025, https://medium.com/@alex_picprod/an-efficient-webgl-map-with-mapbox-gl-js-and-react-mapbox-gl-4ac7f3d41570
9. Understanding WebGL and Mapbox GL JS rendering - StudyRaid, accessed July 6, 2025, https://app.studyraid.com/en/read/15000/518436/understanding-webgl-and-mapbox-gl-js-rendering
10. runtime styling | Help - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/help/glossary/runtime-styling/
11. data-driven styling | Help - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/help/glossary/data-driven-styling/
12. Deck.gl 의 기초, accessed July 6, 2025, https://www.vw-lab.com/107
13. leaflet vs mapbox-gl vs react-map-gl vs maplibre-gl vs deck.gl vs google-maps-react, accessed July 6, 2025, https://npm-compare.com/leaflet,mapbox-gl,react-map-gl,maplibre-gl,deck.gl,google-maps-react
14. Mapbox vs Google Maps - What are the differences? - SoftKraft, accessed July 6, 2025, https://www.softkraft.co/mapbox-vs-google-maps/
15. Mapbox GL new license and 6 free alternatives | Geoapify, accessed July 6, 2025, https://www.geoapify.com/mapbox-gl-new-license-and-6-free-alternatives/
16. Mapbox GL JS Is No Longer Open Source - WP Tavern, accessed July 6, 2025, https://wptavern.com/mapbox-gl-js-is-no-longer-open-source
17. Pricing | Mapbox, accessed July 6, 2025, https://www.mapbox.com/pricing
18. v2.0 changes merged with updated TOS and license / Issue #10162 ..., accessed July 6, 2025, https://github.com/mapbox/mapbox-gl-js/issues/10162
19. Mapbox GL 말고 개발자 경험 더 좋은 다른 대안 있어? : r/gis - Reddit, accessed July 6, 2025, https://www.reddit.com/r/gis/comments/1hpkc8z/are_there_alternatives_to_mapbox_gl_that_provide/?tl=ko
20. Map Styles | Mapbox GL JS | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/guides/styles/
21. Add a new layer below labels | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/geojson-layer-in-stack/
22. Add multiple geometries from one GeoJSON source | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/multiple-geometries/
23. Add markers to a web map with a symbol layer | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/geojson-markers/
24. Get started with Mapbox GL JS expressions | Help, accessed July 6, 2025, https://docs.mapbox.com/help/tutorials/mapbox-gl-js-expressions/
25. access token | Help - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/help/glossary/access-token/
26. Access Tokens | Help - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/help/dive-deeper/access-tokens/
27. Token management | Accounts and pricing - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/accounts/guides/tokens/
28. Tokens | API Docs - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/api/accounts/tokens/
29. Getting Started | Mapbox GL JS | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/guides/install/
30. Use Mapbox GL JS in a React app | Help, accessed July 6, 2025, https://docs.mapbox.com/help/tutorials/use-mapbox-gl-js-with-react/
31. Display a map on a webpage | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/simple-map/
32. Add custom markers in Mapbox GL JS | Help | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/help/tutorials/custom-markers-gl-js/
33. Display a popup | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/popup/
34. Attach a popup to a marker instance | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/set-popup/
35. Add Dynamic Markers and Popups to a Map in a React app | Help - Mapbox Documentation, accessed July 6, 2025, https://docs.mapbox.com/help/tutorials/dynamic-markers-react/
36. Load data from an external GeoJSON file | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/external-geojson/
37. Add a polygon to a map using a GeoJSON source | Mapbox GL JS ..., accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/geojson-polygon/
38. Show polygon information on click | Mapbox GL JS | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/polygon-popup-on-click/
39. Display a popup on click | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/popup-on-click/
40. Create a hover effect | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/hover-styles/
41. Create interactive hover effects with Mapbox GL JS | Help, accessed July 6, 2025, https://docs.mapbox.com/help/tutorials/create-interactive-hover-effects-with-mapbox-gl-js/
42. Set a style | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/guides/styles/set-a-style/
43. Change a map's style | Mapbox GL JS | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/setstyle/
44. Mapbox Studio manual | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/studio-manual/guides/
45. Getting started with Mapbox Standard in Studio | Help | Mapbox, accessed July 6, 2025, https://docs.mapbox.com/help/tutorials/aa-standard-in-studio/
46. Build your own customized map with Mapbox GL JS and Mapbox Studio - Medium, accessed July 6, 2025, https://medium.com/@edanlewis/build-your-own-customized-map-with-mapbox-gl-js-and-mapbox-studio-98ca392ba65c
47. How does mapbox pricing work? - Reddit, accessed July 6, 2025, https://www.reddit.com/r/mapbox/comments/hx1m2y/how_does_mapbox_pricing_work/
48. Mapbox, once touted as the opensource competitor to Google Maps, is no longer free, accessed July 6, 2025, https://www.caliper.com/maptitude/blog/mapbox-no-longer-free/default.htm
49. Our Thoughts as MapboxGL JS v2.0 Goes Proprietary - Carto, accessed July 6, 2025, https://carto.com/blog/our-thoughts-as-mapboxgl-js-2-goes-proprietary
50. Mapbox-gl-js is no longer under the 3-Clause BSD license | Hacker News, accessed July 6, 2025, https://news.ycombinator.com/item?id=25347310
51. leaflet vs mapbox-gl vs react-native-maps vs react-google-maps | Mapping Libraries for Web and Mobile Applications Comparison - NPM Compare, accessed July 6, 2025, https://npm-compare.com/leaflet,mapbox-gl,react-google-maps,react-native-maps
52. 지도 API 별 장단점 - velog, accessed July 6, 2025, [https://velog.io/@songyeonji_/%EC%A7%80%EB%8F%84-API-%EB%B3%84-%EC%9E%A5%EB%8B%A8%EC%A0%90](https://velog.io/@songyeonji_/지도-API-별-장단점)
53. What are Leaflet and Mapbox, and what are their differences? - Stack Overflow, accessed July 6, 2025, https://stackoverflow.com/questions/12262163/what-are-leaflet-and-mapbox-and-what-are-their-differences
54. Maps JavaScript API에 대한 정책 - Google for Developers, accessed July 6, 2025, https://developers.google.com/maps/documentation/javascript/policies?hl=ko



