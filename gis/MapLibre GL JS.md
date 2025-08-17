# MapLibre GL JS

## 1.  MapLibre GL JS의 기초

웹 기반 인터랙티브 지도 제작은 현대 웹 애플리케이션의 핵심 요소로 자리 잡았습니다. 사용자의 위치를 표시하거나, 데이터를 지리적으로 시각화하거나, 경로를 안내하는 등 지도의 활용 범위는 무궁무진합니다. 이 보고서는 수많은 지도 라이브러리 중에서도 강력한 성능과 높은 자유도를 자랑하는 오픈소스 라이브러리, MapLibre GL JS의 모든 것을 다룹니다. 라이브러리의 탄생 배경과 철학부터 시작하여, 기본 사용법, 핵심 아키텍처, 고급 시각화 기법, 그리고 방대한 생태계까지 아우르는 포괄적인 가이드를 제공하는 것을 목표로 합니다.

### 1.1  현대 웹 매핑에 대한 소개

MapLibre GL JS는 웹 브라우저에서 벡터 타일을 사용하여 인터랙티브 지도를 렌더링하는 고성능 TypeScript/JavaScript 라이브러리입니다.1 이 라이브러리의 가장 큰 기술적 특징은 래스터(Raster) 이미지가 아닌 벡터(Vector) 데이터를 기반으로 하며, WebGL(Web Graphics Library)을 통해 그래픽 처리 장치(GPU)의 하드웨어 가속을 최대한 활용한다는 점입니다.3

과거의 웹 지도는 주로 래스터 타일 방식에 의존했습니다. 이는 지도를 작은 사각형 이미지(PNG 등)의 격자로 나누어, 사용자가 지도를 확대하거나 이동할 때마다 서버에서 해당 이미지 조각들을 불러오는 방식입니다. 이 방식은 구현이 비교적 간단하지만, 확대/축소 시 해상도 저하가 발생하고, 실시간으로 지도 스타일(색상, 폰트 등)을 변경하기가 매우 어렵다는 단점이 있습니다. 모든 스타일에 대해 별도의 이미지 타일셋을 미리 생성해야 하기 때문입니다.4

반면, MapLibre GL JS가 채택한 벡터 타일 방식은 지리적 데이터를 점, 선, 다각형 등의 기하학적 정보와 속성 데이터로 구성하여 전송합니다. 렌더링, 즉 화면에 지도를 그리는 작업은 서버가 아닌 클라이언트(사용자의 브라우저)에서 직접 수행됩니다. 이 과정에서 GPU 가속을 활용하기 때문에 방대한 양의 데이터를 매우 빠르고 부드럽게 처리할 수 있습니다.3 이러한 아키텍처는 다음과 같은 강력한 장점을 제공합니다.

- **고성능 렌더링:** GPU 가속 덕분에 복잡한 지리 데이터도 부드럽게 탐색할 수 있습니다.3
- **동적 스타일링:** 자바스크립트 코드를 통해 실시간으로 지도의 색상, 라벨, 두께 등 거의 모든 시각적 요소를 변경할 수 있습니다.2
- **유연한 데이터 시각화:** 데이터의 속성값에 따라 스타일을 동적으로 결정하는 '데이터 기반 스타일링'이 가능하여, 인구 밀도, 기온 분포 등 복잡한 정보를 효과적으로 시각화할 수 있습니다.2
- **3D 시각화 지원:** WebGL을 기반으로 하므로, 건물을 입체적으로 표현하거나 3D 지형을 구현하는 것이 가능합니다.2

이러한 특징 때문에 MapLibre GL JS는 Leaflet과 같은 다른 유명 오픈소스 라이브러리와 차별화됩니다. Leaflet은 SVG 또는 Canvas 2D 렌더링을 사용하여 가볍고 사용하기 쉬우며, 구형 브라우저와의 호환성이 높다는 장점이 있습니다. 하지만 복잡한 데이터 시각화나 3D 기능, 최고 수준의 렌더링 성능이 요구되는 애플리케이션에는 MapLibre GL JS가 더 적합한 선택입니다.4

| 기능                   | MapLibre GL JS                                       | Leaflet                                                  |
| ---------------------- | ---------------------------------------------------- | -------------------------------------------------------- |
| **렌더링 기술**        | WebGL (GPU 하드웨어 가속)                            | HTML/CSS/JS (SVG 또는 Canvas 2D)                         |
| **주요 데이터 형식**   | 벡터 타일 (MVT)                                      | 래스터 타일                                              |
| **성능**               | 대용량 데이터셋에 매우 효율적, 부드러운 애니메이션   | 가볍지만, 데이터가 많아지면 성능 저하 발생 가능          |
| **커스터마이징**       | 동적 스타일링, 3D 시각화 등 깊이 있는 맞춤 설정 가능 | 다양한 플러그인을 통한 높은 확장성, 기본 기능은 단순함   |
| **학습 곡선**          | 상대적으로 높음 (벡터 타일 및 스타일 명세 이해 필요) | 매우 낮음, 초보자에게 친화적                             |
| **생태계 규모**        | 성장 중이며 활발함                                   | 매우 크고 성숙함                                         |
| **이상적인 사용 사례** | 복잡한 데이터 시각화, 동적/인터랙티브 지도, 3D 지도  | 단순한 위치 표시, 넓은 브라우저 호환성이 중요한 프로젝트 |

데이터 출처: 4

### 1.2  오픈소스의 장점: 역사와 라이선스

MapLibre GL JS의 기술적 특징만큼이나 중요한 것은 그 탄생 배경과 철학입니다. 이 라이브러리는 원래 Mapbox사에서 개발한 `mapbox-gl-js`라는 매우 성공적인 오픈소스 프로젝트에서 시작되었습니다.6

`mapbox-gl-js`는 2020년 12월, 버전 2.0으로 업데이트되면서 기존의 허용적인 BSD-3-Clause 라이선스를 버리고 독점(proprietary) 라이선스로 전환했습니다.6 이는 더 이상 누구나 자유롭게 소스 코드를 사용하거나 수정할 수 없게 되었음을 의미하며, 많은 개발자와 기업에게 큰 충격을 주었습니다.

이러한 변화에 대응하여, 전 세계의 개발자 커뮤니티는 `mapbox-gl-js`의 마지막 오픈소스 버전(v1.x)을 기반으로 새로운 프로젝트를 시작했습니다. 이것이 바로 MapLibre GL JS의 탄생입니다.4 'MapLibre'라는 이름은 '자유로운(Libre) 지도'를 의미하며, 특정 기업에 종속되지 않는 진정한 오픈소스 지도 기술을 계속 발전시키고자 하는 커뮤니티의 열망을 담고 있습니다.

따라서 MapLibre GL JS를 선택하는 것은 단순히 기술적 결정을 넘어, 특정 기업의 정책 변화에 휘둘리지 않는 지속 가능하고 자유로운 개발 환경을 선택하는 철학적, 비즈니스적 결정이기도 합니다. MapLibre GL JS는 초기 Mapbox GL JS가 가졌던 3-Clause BSD 라이선스를 그대로 계승하고 있습니다.8 이 라이선스는 매우 허용적인 오픈소스 라이선스로, 다음과 같은 자유를 보장합니다 9:

- **상업적 이용:** 상업적인 애플리케이션에 무료로 사용할 수 있습니다.
- **수정 및 배포:** 소스 코드를 자유롭게 수정하고, 수정한 버전을 배포할 수 있습니다.
- **소스 코드 비공개:** 수정한 코드를 공개할 의무가 없습니다.

이러한 개방적인 라이선스 정책 덕분에 개발자들은 라이선스 비용이나 사용량 제한에 대한 걱정 없이 혁신적인 지도 애플리케이션을 구축할 수 있습니다. MapLibre 프로젝트는 Mapbox가 오픈소스에 기여한 놀라운 업적에 감사를 표하면서도, 커뮤니티 주도의 건강한 대안을 만들어 나가는 것을 목표로 하고 있습니다.8

## 2.  시작하기: 첫 인터랙티브 지도 만들기

이론적 배경을 이해했다면, 이제 직접 코드를 작성하여 지도 화면을 띄워볼 차례입니다. 이 파트에서는 프로젝트 설정부터 지도 초기화까지의 과정을 단계별로 안내합니다.

### 2.1  프로젝트 설정: CDN vs. npm

MapLibre GL JS 라이브러리를 프로젝트에 포함시키는 방법은 크게 두 가지가 있습니다.

**1. CDN (Content Delivery Network) 사용**

가장 간단하고 빠르게 시작할 수 있는 방법입니다. HTML 파일의 `<head>` 태그 안에 라이브러리의 JavaScript 파일과 CSS 파일 링크를 추가하기만 하면 됩니다. 이 방식은 프로토타이핑이나 간단한 웹 페이지에 적합합니다.

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8' />
    <title>Display a map</title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <link href='https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.css' rel='stylesheet' />
    <script src='https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.js'></script>
    <style>
        body { margin:0; padding:0; }
        #map { position:absolute; top:0; bottom:0; width:100%; }
    </style>
</head>
<body>
    <div id='map'></div>
    <script>
        // 지도 초기화 코드는 여기에 작성됩니다.
    </script>
</body>
</html>
```

코드 출처: 8

**2. npm (Node Package Manager) 사용**

React, Angular, Vue 등 현대적인 프론트엔드 프레임워크를 사용하거나, 웹팩(Webpack)과 같은 모듈 번들러를 사용하는 프로젝트에서는 npm을 통해 라이브러리를 설치하는 것이 표준적인 방법입니다.

터미널에서 다음 명령어를 실행하여 `maplibre-gl` 패키지를 설치합니다.

```Bash
npm install maplibre-gl
```

명령어 출처: 11

설치가 완료되면, JavaScript 파일에서 라이브러리와 CSS 파일을 `import`하여 사용할 수 있습니다.

```JavaScript
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

// 지도 초기화 코드
```

코드 출처: 11

Mapbox GL JS v1에서 마이그레이션하는 경우, 기존의 `mapboxgl`을 `maplibregl`로 변경하고 CDN 주소를 업데이트하는 것만으로도 대부분의 코드가 호환됩니다.12 이는 MapLibre 커뮤니티가 기존 사용자들의 전환 장벽을 낮추기 위해 의도적으로 설계한 부분입니다.

### 2.2  지도 초기화: Map 객체

라이브러리를 프로젝트에 포함시켰다면, 이제 `Map` 객체를 생성하여 지도를 초기화할 수 있습니다. `new maplibregl.Map()` 생성자는 지도에 대한 다양한 옵션을 담은 객체를 인자로 받습니다. 가장 필수적인 네 가지 옵션은 다음과 같습니다.4

- `container`: 지도가 렌더링될 HTML 요소의 `id`입니다. 위 예제에서는 `<div id='map'></div>`에 해당합니다.
- `style`: 지도의 시각적 모양을 정의하는 스타일 JSON 파일의 URL입니다. 처음 시작할 때는 MapLibre에서 제공하는 데모 타일 URL을 사용할 수 있습니다.
- `center`: 지도의 초기 중심점 좌표를 `[경도, 위도]` 형식의 배열로 지정합니다.
- `zoom`: 지도의 초기 확대/축소 레벨을 숫자로 지정합니다.

다음은 이 옵션들을 사용하여 가장 기본적인 지도를 생성하는 전체 코드입니다.

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8' />
    <title>Display a simple map</title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <link href='https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.css' rel='stylesheet' />
    <script src='https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.js'></script>
    <style>
        body { margin: 0; padding: 0; }
        #map { position: absolute; top: 0; bottom: 0; width: 100%; }
    </style>
</head>
<body>
    <div id='map'></div>
    <script>
        const map = new maplibregl.Map({
            container: 'map', // container id
            style: 'https://demotiles.maplibre.org/style.json', // style URL
            center: , // 시작 위치 [경도, 위도]
            zoom: 1 // 시작 줌 레벨
        });
    </script>
</body>
</html>
```

코드 출처: 11

이 코드를 HTML 파일로 저장하고 브라우저에서 열면, 전 세계가 보이는 간단한 지도가 화면에 나타날 것입니다.

### 2.3  필수 구성: CSS와 보안

지도를 성공적으로 띄웠다면, 몇 가지 추가적인 필수 구성 사항을 이해해야 합니다.

MapLibre CSS의 중요성

maplibre-gl.css 파일은 단순히 지도의 미관을 위한 것이 아닙니다. 팝업(Popup), 마커(Marker), 지도 컨트롤(NavigationControl 등)과 같은 UI 요소들이 올바르게 표시되고 작동하기 위해 반드시 필요합니다.11 이 CSS 파일이 없으면 해당 기능들이 깨져 보이거나 전혀 작동하지 않을 수 있습니다. 따라서 CDN을 사용하든 npm을 사용하든, 이 CSS 파일을 반드시 포함해야 합니다.

콘텐츠 보안 정책 (CSP)

웹사이트의 보안을 강화하기 위해 CSP(Content Security Policy)를 사용하는 경우, MapLibre GL JS가 정상적으로 작동하려면 특정 지시문을 허용해야 합니다. MapLibre는 내부적으로 웹 워커(Web Worker)를 사용하여 백그라운드에서 타일 데이터를 처리하는데, 이 워커는 blob: URL을 통해 생성됩니다. 따라서 다음과 같은 CSP 지시문이 필요합니다.11

- `worker-src blob:`
- `child-src blob:`
- `img-src data: blob:`

만약 보안 정책이 매우 엄격하여 `blob:`을 허용할 수 없는 환경이라면, MapLibre는 별도의 CSP 호환 빌드(`maplibre-gl-csp.js`와 `maplibre-gl-csp-worker.js`)를 제공합니다. 이 경우, 워커 파일의 경로를 수동으로 설정해주어야 합니다.11

## 3.  핵심 아키텍처: 스타일, 소스, 레이어

기본적인 지도를 띄우는 데 성공했다면, 이제 MapLibre GL JS의 진정한 힘을 발휘하기 위해 핵심 아키텍처를 이해해야 합니다. MapLibre의 모든 시각적 표현은 **스타일(Style)**, **소스(Source)**, **레이어(Layer)**라는 세 가지 핵심 개념의 상호작용으로 이루어집니다. 이 패러다임을 마스터하는 것은 단순한 지도 표시를 넘어, 데이터를 자유자재로 시각화하는 고급 개발자로 나아가는 가장 중요한 단계입니다.

### 3.1  MapLibre 스타일의 구조

MapLibre 스타일은 지도의 시각적 외형을 정의하는 단일 JSON 객체입니다.14 이 JSON 파일은 지도에 어떤 데이터를, 어떤 순서로, 그리고 어떻게 그려야 하는지에 대한 모든 규칙을 담고 있는 '설계도'와 같습니다. 이 설계도의 구조와 속성은 'MapLibre 스타일 명세(MapLibre Style Specification)'라는 표준에 의해 엄격하게 정의됩니다.15

스타일 JSON 파일은 다음과 같은 최상위 속성들을 포함합니다.

- `version`: 스타일 명세의 버전을 나타냅니다.
- `name`: 스타일의 이름입니다.
- `sources`: 지도에 표시할 데이터의 출처를 정의합니다 (자세한 내용은 3.2절 참조).
- `layers`: `sources`에서 가져온 데이터를 어떻게 시각적으로 표현할지 정의합니다 (자세한 내용은 3.3절 참조).
- `glyphs`: 지도에 텍스트 라벨을 표시하는 데 필요한 글꼴 정보를 담고 있는 URL 템플릿입니다.
- `sprite`: 지도에 아이콘(심볼)을 표시하는 데 필요한 이미지 파일(스프라이트 시트)의 위치를 정의합니다.

이 JSON 파일을 직접 손으로 작성하는 것은 매우 복잡하고 오류가 발생하기 쉽습니다. 그래서 **Maputnik**과 같은 시각적 스타일 편집기를 사용하는 것이 일반적입니다.14 Maputnik을 사용하면 GUI 환경에서 레이어를 추가하고 색상, 두께 등을 조절하면서 실시간으로 변화를 확인하고, 그 결과를 스타일 JSON 파일로 내보낼 수 있습니다.

### 3.2  데이터 소스: 지도의 연료

**소스(Source)**는 지도에 표시될 지리적 데이터를 제공하는 역할을 합니다.16 소스는 데이터가 어디에 있고 어떤 형식인지를 정의합니다. MapLibre는 다양한 유형의 소스를 지원하며, 

`map.addSource(id, source_definition)` 메서드를 사용하여 런타임에 지도에 추가할 수 있습니다.17

주요 소스 유형은 다음과 같습니다.

- **`geojson`**: 가장 흔하게 사용되는 소스 유형 중 하나로, GeoJSON 형식의 데이터를 직접 코드에 포함하거나 URL을 통해 불러옵니다. 클라이언트 측에서 소량의 맞춤형 데이터를 추가할 때 매우 유용합니다.18
- **`vector`**: 벡터 타일셋을 위한 소스입니다. 대용량의 지리 데이터를 효율적으로 처리하기 위해 미리 타일 형태로 가공된 데이터셋의 URL을 지정합니다.
- **`raster`**: 래스터 타일셋(위성 이미지, 항공 사진 등)을 위한 소스입니다.
- **`raster-dem`**: 3D 지형을 표현하기 위한 DEM(Digital Elevation Model) 데이터를 담은 래스터 타일을 위한 소스입니다.
- **`image`**: 단일 이미지를 지도상의 특정 영역에 고정시키는 소스입니다.
- **`video`**: 비디오를 지도 위에 오버레이하는 소스입니다.

소스는 그 자체로는 지도에 아무것도 표시하지 않습니다. 소스는 단지 데이터를 준비하는 단계일 뿐이며, 이 데이터를 실제로 시각화하는 것은 '레이어'의 역할입니다. 이처럼 데이터의 '정의'와 '시각화'를 분리하는 것이 MapLibre 아키텍처의 핵심입니다. 이 분리 덕분에 하나의 소스를 여러 개의 다른 레이어에서 재사용하여 다양하게 표현하는 것이 가능해집니다.

### 3.3  레이어: 데이터의 시각화

**레이어(Layer)**는 소스로부터 받은 데이터를 어떻게 지도에 그릴지 정의하는 스타일 규칙입니다.16 각 레이어는 특정 소스를 참조하며(

`source` 속성), 해당 소스의 데이터를 가져와 지정된 스타일로 렌더링합니다. `map.addLayer(layer_definition, before_id)` 메서드를 사용하여 지도에 추가합니다.17

레이어 정의 객체의 핵심 속성은 다음과 같습니다.

- `id`: 레이어의 고유한 이름입니다.
- `type`: 레이어의 렌더링 유형을 지정합니다. 이 유형에 따라 사용할 수 있는 스타일 속성이 결정됩니다.
- `source`: 이 레이어가 사용할 소스의 `id`를 지정합니다. 이 속성을 통해 레이어와 소스가 연결됩니다.
- `source-layer`: 소스가 벡터 타일(`vector`) 유형일 때 반드시 필요한 속성입니다. 벡터 타일셋은 내부에 여러 개의 데이터 레이어(예: 'road', 'building', 'water')를 포함할 수 있는데, 이 속성을 통해 그중 어떤 데이터 레이어를 사용할지 지정합니다.19 GeoJSON 소스에는 사용하지 않습니다.
- `filter`: 소스의 데이터 중 특정 조건에 맞는 피처(feature)만 선택하여 표시하고 싶을 때 사용하는 필터 표현식입니다.18
- `layout`: 피처를 어떻게 배치하고 그릴지 결정하는 속성입니다 (예: `visibility`, `line-cap`, `symbol-placement`).
- `paint`: 피처에 어떤 색상, 두께, 투명도 등을 적용할지 결정하는 속성입니다 (예: `fill-color`, `line-width`, `icon-opacity`).

이 `layout`과 `paint`의 구분은 중요합니다. `layout` 속성은 줌 레벨에 따라 계산되며, 렌더링 성능에 비교적 큰 영향을 미칩니다. 반면 `paint` 속성은 매 프레임마다 다시 계산될 수 있어, 부드러운 색상 전환이나 투명도 애니메이션 등에 사용되며 성능 부담이 적습니다.

| 레이어 유형          | 설명                                                      | 주요 사용 사례                             |
| -------------------- | --------------------------------------------------------- | ------------------------------------------ |
| **`background`**     | 지도의 배경색이나 패턴을 채웁니다.                        | 지도 전체의 기본 배경 설정                 |
| **`fill`**           | 다각형(Polygon) 피처의 내부를 채웁니다.                   | 행정 구역, 건물 바닥, 호수 등 면적 표현    |
| **`line`**           | 선(LineString) 피처를 그립니다.                           | 도로, 강, 경로, 경계선 등 선형 데이터 표현 |
| **`symbol`**         | 아이콘이나 텍스트 라벨을 표시합니다.                      | 관심 지점(POI), 도시 이름, 도로명 표시     |
| **`circle`**         | 원형(Circle)을 그립니다.                                  | 점(Point) 데이터를 간단한 원으로 시각화    |
| **`heatmap`**        | 점 데이터의 밀도를 색상으로 표현하는 히트맵을 생성합니다. | 인구 밀도, 범죄 발생 지점 등 밀집도 시각화 |
| **`fill-extrusion`** | 다각형을 3D 형태로 돌출시킵니다.                          | 3D 건물, 입체 막대그래프 표현              |
| **`raster`**         | 래스터 타일(위성 이미지 등)을 표시합니다.                 | 위성 지도, 항공 사진 배경                  |
| **`hillshade`**      | DEM 데이터로부터 지형의 음영을 계산하여 입체감을 줍니다.  | 3D 지형도, 등고선과 함께 사용              |

데이터 출처: 19

예를 들어, 미국의 주 경계선이 담긴 `states-source`라는 GeoJSON 소스가 있다고 가정해 봅시다. 이 하나의 소스를 사용하여 두 개의 레이어를 만들 수 있습니다. 첫 번째는 `fill` 타입의 레이어로 각 주의 내부를 파란색으로 칠하고, 두 번째는 `line` 타입의 레이어로 각 주의 경계선을 검은색으로 더 굵게 그리는 것입니다.21 이처럼 소스와 레이어의 분리는 데이터의 재사용성과 표현의 유연성을 극대화하는 강력한 개념입니다.

## 4.  필수 기능과 상호작용

핵심 아키텍처를 이해했다면, 이제 사용자와 상호작용하는 동적인 지도를 만들기 위한 구체적인 기능들을 구현해볼 차례입니다. 이 파트에서는 가장 빈번하게 사용되는 마커, 팝업, GeoJSON 데이터 시각화, 그리고 이벤트 처리 방법을 상세한 코드 예제와 함께 다룹니다.

### 4.1  마커와 팝업: 관심 지점 표시

지도 위에 특정 위치를 강조하고 추가 정보를 제공하는 것은 가장 기본적인 요구사항입니다. MapLibre는 이를 위해 `Marker`와 `Popup` 클래스를 제공합니다.

**마커(Marker) 추가하기**

`maplibregl.Marker` 객체를 생성하고, `setLngLat()` 메서드로 위치를 지정한 뒤, `addTo()` 메서드로 지도에 추가합니다.

```JavaScript
// 마커를 생성합니다.
const marker = new maplibregl.Marker({
    color: "#FF0000", // 마커 색상 지정 (옵션)
    draggable: true // 드래그 가능 여부 설정 (옵션)
})
.setLngLat([126.9779, 37.5663]) // 마커의 위치를 [경도, 위도]로 설정합니다.
.addTo(map); // 지도에 마커를 추가합니다.
```

코드 출처: 22

**팝업(Popup) 추가 및 마커에 연결하기**

`maplibregl.Popup` 객체를 생성하고, `setHTML()` 또는 `setText()` 메서드로 내용을 채웁니다. 이 팝업을 마커에 연결하려면 `marker.setPopup(popup)` 메서드를 사용합니다. 이렇게 하면 마커를 클릭했을 때 팝업이 나타납니다.

```JavaScript
// 1. 팝업을 생성하고 내용을 설정합니다.
const popup = new maplibregl.Popup({ offset: 25 }) // 마커로부터의 거리(offset) 설정
   .setHTML('<h3>서울시청</h3><p>서울특별시 중구 세종대로 110</p>');

// 2. 마커를 생성하고 위치를 설정합니다.
const marker = new maplibregl.Marker()
   .setLngLat([126.9779, 37.5663])
   .setPopup(popup) // 생성한 팝업을 마커에 연결합니다.
   .addTo(map);
```

코드 출처: 22

**레이어 피처에 팝업 연결하기**

마커를 사용하지 않고, GeoJSON 레이어의 특정 피처(예: 도시 아이콘)를 클릭했을 때 팝업을 띄울 수도 있습니다. 이는 `map.on('click',...)` 이벤트 핸들러를 사용하여 구현합니다. 사용자가 클릭한 지점의 피처 정보를 `map.queryRenderedFeatures()`로 가져와 동적으로 팝업을 생성하는 방식입니다.

```JavaScript
map.on('load', () => {
    //... (GeoJSON 소스 및 레이어 추가 코드)...

    // 'places' 레이어를 클릭했을 때의 이벤트 리스너를 추가합니다.
    map.on('click', 'places', (e) => {
        // 클릭된 피처의 좌표와 속성 정보를 가져옵니다.
        const coordinates = e.features.geometry.coordinates.slice();
        const description = e.features.properties.description;

        // 좌표가 세계 지도를 벗어나 반복 표시될 경우를 처리합니다.
        while (Math.abs(e.lngLat.lng - coordinates) > 180) {
            coordinates += e.lngLat.lng > coordinates? 360 : -360;
        }

        // 팝업을 생성하고 지도에 추가합니다.
        new maplibregl.Popup()
           .setLngLat(coordinates)
           .setHTML(description)
           .addTo(map);
    });

    // 'places' 레이어 위에 마우스가 올라가면 커서를 포인터로 변경합니다.
    map.on('mouseenter', 'places', () => {
        map.getCanvas().style.cursor = 'pointer';
    });

    // 마우스가 레이어를 벗어나면 커서를 원래대로 되돌립니다.
    map.on('mouseleave', 'places', () => {
        map.getCanvas().style.cursor = '';
    });
});
```

코드 출처: 25

### 4.2  GeoJSON으로 지리 데이터 시각화

GeoJSON은 점, 선, 다각형 등 지리 데이터를 표현하는 표준 형식으로, MapLibre에서 맞춤형 데이터를 시각화하는 가장 일반적인 방법입니다. `geojson` 타입의 소스를 추가하고, 이를 참조하는 레이어를 만들어 데이터를 지도에 그립니다.27

**선(Line) 그리기**

`LineString` 타입의 GeoJSON 피처를 사용하여 경로, 도로, 경계선 등을 표현할 수 있습니다. `line` 타입의 레이어를 사용하여 색상(`line-color`), 두께(`line-width`), 끝 모양(`line-cap`) 등을 스타일링합니다.

```JavaScript
map.on('load', () => {
    // 1. GeoJSON 소스를 추가합니다.
    map.addSource('route', {
        'type': 'geojson',
        'data': {
            'type': 'Feature',
            'properties': {},
            'geometry': {
                'type': 'LineString',
                'coordinates': [127.0276, 37.4979], // 강남역
                    [126.9238, 37.5563], // 홍대입구역
                    [127.0473, 37.5124]  // 잠실역
                ]
            }
        }
    });

    // 2. 소스를 참조하는 line 레이어를 추가합니다.
    map.addLayer({
        'id': 'route',
        'type': 'line',
        'source': 'route',
        'layout': {
            'line-join': 'round', // 선이 꺾이는 부분 처리
            'line-cap': 'round'   // 선의 끝 부분 처리
        },
        'paint': {
            'line-color': '#888',
            'line-width': 8
        }
    });
});
```

코드 출처: 28

**다각형(Polygon) 그리기**

`Polygon` 타입의 GeoJSON 피처를 사용하여 행정 구역, 공원, 건물 영역 등을 표현합니다. `fill` 타입의 레이어로 내부를 채우고(`fill-color`, `fill-opacity`), `line` 타입의 레이어를 추가하여 외곽선을 그릴 수 있습니다. 이때 두 레이어 모두 동일한 GeoJSON 소스를 참조합니다.

```JavaScript
map.on('load', () => {
    // 1. 다각형 GeoJSON 소스를 추가합니다.
    map.addSource('seoul-area', {
        'type': 'geojson',
        'data': {
            'type': 'Feature',
            'geometry': {
                'type': 'Polygon',
                // (여기에 서울 지역의 다각형 좌표 배열이 들어갑니다)
                'coordinates': [[126.764, 37.428], [127.183, 37.428],
                    [127.183, 37.701], [126.764, 37.701],
                    [126.764, 37.428]]
            }
        }
    });

    // 2. 내부를 채우는 fill 레이어를 추가합니다.
    map.addLayer({
        'id': 'seoul-fill',
        'type': 'fill',
        'source': 'seoul-area',
        'layout': {},
        'paint': {
            'fill-color': '#0080ff',
            'fill-opacity': 0.5
        }
    });

    // 3. 외곽선을 그리는 line 레이어를 추가합니다.
    map.addLayer({
        'id': 'seoul-outline',
        'type': 'line',
        'source': 'seoul-area',
        'layout': {},
        'paint': {
            'line-color': '#000',
            'line-width': 3
        }
    });
});
```

코드 출처: 21

### 4.3  지도 이벤트 마스터하기

인터랙티브 애플리케이션을 만들기 위해서는 사용자의 행동이나 지도의 상태 변화에 반응해야 합니다. MapLibre는 `map.on('event_name', handler_function)` 메서드를 통해 풍부한 이벤트 시스템을 제공합니다.

이벤트 핸들러 함수는 이벤트에 대한 상세 정보를 담은 이벤트 객체(`e`)를 인자로 받습니다. 예를 들어, `click` 이벤트의 경우 `e.lngLat` 속성을 통해 클릭된 지점의 경위도 좌표를 얻을 수 있습니다.

| 이벤트 이름     | 발생 시점                                           | 주요 사용 사례                                               |
| --------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| **`load`**      | 지도의 모든 리소스(스타일, 소스 등)가 로드되었을 때 | 소스 및 레이어 추가, 초기 설정 등 맵이 완전히 준비된 후 실행해야 하는 코드 |
| **`click`**     | 사용자가 지도를 클릭했을 때                         | 팝업 표시, 특정 피처 정보 조회, 클릭 지점으로 줌인           |
| **`mousemove`** | 사용자가 지도 위에서 마우스를 움직일 때             | 마우스 위치의 좌표 표시, 호버(hover) 효과 구현               |
| **`zoomend`**   | 지도 확대/축소 동작이 끝났을 때                     | 현재 줌 레벨에 따라 다른 정보나 레이어 표시                  |
| **`error`**     | 오류가 발생했을 때                                  | 비동기 작업(타일 로딩 등)에서 발생한 오류 처리               |
| **`idle`**      | 지도 렌더링이 필요 없는 유휴 상태가 되었을 때       | 모든 애니메이션과 데이터 로딩이 끝난 후 특정 작업 수행       |

데이터 출처: 31

다음은 지도 클릭 시 해당 위치의 경위도를 콘솔에 출력하는 간단한 예제입니다.

```JavaScript
map.on('click', (e) => {
    console.log(`A click event has occurred at Lng: ${e.lngLat.lng}, Lat: ${e.lngLat.lat}`);
});
```

더블 클릭으로 줌인하는 기능(`doubleClickZoom`)이나 박스를 드래그하여 해당 영역으로 줌인하는 기능(`boxZoom`) 등은 기본적으로 활성화되어 있으며, 지도 생성 시 옵션으로 비활성화할 수 있습니다.32 이러한 이벤트와 핸들러를 조합하면 사용자의 모든 상호작용에 반응하는 풍부한 경험을 제공할 수 있습니다.

## 5.  고급 시각화와 애니메이션

MapLibre GL JS의 진정한 가치는 정적인 지도를 넘어, 데이터를 기반으로 동적으로 변화하고 생동감 넘치는 시각화를 구현할 수 있다는 데 있습니다. 이 파트에서는 표현식(Expressions), 카메라 애니메이션, 3D 기능을 활용하여 한 차원 높은 지도 경험을 만드는 방법을 알아봅니다.

### 5.1  동적 데이터 기반 스타일 만들기

**표현식(Expressions)**은 MapLibre 스타일링의 핵심 기능으로, 피처의 속성 데이터나 지도의 줌 레벨에 따라 `paint` 또는 `layout` 속성값을 동적으로 계산할 수 있게 해줍니다. 정적인 값 대신 표현식을 사용하면 데이터에 생명을 불어넣을 수 있습니다.

- **`get` 표현식**: 피처의 속성값에 접근합니다. `['get', 'property_name']` 형태로 사용합니다.
- **`match` / `case` 표현식**: 조건에 따라 다른 값을 반환합니다. `['match', ['get', 'category'], 'cafe', '#FF0000', 'park', '#00FF00', '#CCCCCC']`와 같이 사용하면 'category' 속성값에 따라 다른 색상을 지정할 수 있습니다.
- **`interpolate` 표현식**: 연속적인 값(줌 레벨, 숫자 속성 등)에 따라 스타일을 부드럽게 변경합니다. 예를 들어, 줌 레벨이 높아질수록 선의 두께를 점진적으로 두껍게 만들 수 있습니다.

다음은 등산로 데이터의 `ELEV_GAIN`(고도 상승량) 속성값에 따라 `interpolate` 표현식을 사용하여 선의 두께를 다르게 지정하는 예제입니다. 고도 상승량이 클수록 등산로가 더 굵게 표시됩니다.18

```JavaScript
map.addLayer({
    id: 'trails-line',
    type: 'line',
    source: 'trails-source',
    paint: {
        'line-color': '#8A2BE2', // 보라색
        'line-width': [
            'interpolate', // 보간 표현식 사용
            ['linear'],    // 선형 보간
            ['get', 'ELEV_GAIN'], // 'ELEV_GAIN' 속성값을 기준으로
            0, 2,        // ELEV_GAIN이 0일 때 두께 2px
            2000, 8      // ELEV_GAIN이 2000일 때 두께 8px
        ]
    }
});
```

코드 출처: 18

이러한 표현식을 활용하면 줌 레벨에 따라 건물의 색상을 점진적으로 바꾸거나 1, 데이터 값에 따라 그라데이션이 적용된 선을 만드는 등 1 매우 정교하고 정보 밀도 높은 시각화가 가능해집니다.

### 5.2  카메라 애니메이션으로 생동감 부여하기

지도 카메라를 프로그래밍 방식으로 제어하여 사용자에게 역동적인 경험을 제공할 수 있습니다. MapLibre는 여러 카메라 제어 메서드를 제공합니다.

- **`jumpTo(options)`**: 애니메이션 없이 즉시 지정된 위치와 줌 레벨로 이동합니다.
- **`easeTo(options)`**: 현재 위치에서 지정된 위치까지 부드럽게 이동합니다. 비교적 짧은 거리 이동에 적합합니다.
- **`flyTo(options)`**: 먼 거리를 이동할 때 마치 비행기를 타고 날아가는 듯한 극적인 효과를 연출합니다. 줌 아웃했다가 목표 지점에서 다시 줌인하는 곡선형 애니메이션을 제공합니다.34

`flyTo` 메서드는 `speed`, `curve`, `essential` 등 다양한 옵션을 통해 애니메이션을 세밀하게 제어할 수 있습니다.36

`essential: true` 옵션은 사용자가 시스템 설정에서 '동작 줄이기(prefers-reduced-motion)'를 활성화했더라도 이 애니메이션은 필수적인 것으로 간주하여 실행하도록 합니다.

다음은 버튼 클릭 시 무작위 위치로 '날아가는' `flyTo` 예제입니다.

```HTML
<button id="fly">Fly to a random location</button>
<script>
    document.getElementById('fly').addEventListener('click', () => {
        map.flyTo({
            center: [
                -74.5 + (Math.random() - 0.5) * 10,
                40 + (Math.random() - 0.5) * 10
            ],
            zoom: 12,
            speed: 0.8,
            essential: true
        });
    });
</script>
```

코드 출처: 34

`flyTo`를 스크롤 이벤트와 결합하면 '스크롤리텔링(scrollytelling)'과 같은 몰입형 콘텐츠를 만들 수 있습니다. 사용자가 페이지를 스크롤하면 스토리에 맞춰 지도가 특정 위치로 날아가며 시각적인 설명을 돕는 방식입니다.38 또한, 

`easing` 함수를 직접 정의하여 애니메이션의 가속/감속 곡선을 커스터마이징하는 것도 가능합니다.39

### 5.3  3차원의 세계 탐험하기

MapLibre GL JS는 WebGL 기반이므로 2D를 넘어 3D 시각화를 지원합니다.2

**다각형 돌출(Fill Extrusion)**

`fill-extrusion` 타입의 레이어를 사용하면 다각형 피처를 입체적으로 돌출시켜 3D 건물을 표현할 수 있습니다. `fill-extrusion-height` 속성에 건물의 높이 값을, `fill-extrusion-base`에 건물 바닥의 고도 값을 지정합니다.1

```JavaScript
map.addLayer({
    'id': '3d-buildings',
    'source': 'composite',
    'source-layer': 'building',
    'filter': ['==', 'extrude', 'true'],
    'type': 'fill-extrusion',
    'minzoom': 15,
    'paint': {
        'fill-extrusion-color': '#aaa',
        // 높이 값은 피처의 'height' 속성을 사용
        'fill-extrusion-height': ['get', 'height'],
        'fill-extrusion-base': ['get', 'min_height'],
        'fill-extrusion-opacity': 0.6
    }
});
```

코드 출처: 1

**3D 지형(Terrain)**

`raster-dem` 타입의 소스(예: MapTiler Terrain-RGB)를 추가하고, `hillshade` 타입의 레이어를 사용하여 지형의 음영을 표현하면 사실적인 3D 지형을 만들 수 있습니다. `map.setTerrain()` 메서드를 호출하여 3D 지형 렌더링을 활성화합니다.

```JavaScript
map.on('load', () => {
    map.addSource('mapbox-dem', {
        'type': 'raster-dem',
        'url': 'https://api.maptiler.com/tiles/terrain-rgb/tiles.json?key=YOUR_API_KEY',
        'tileSize': 256
    });
    map.setTerrain({ 'source': 'mapbox-dem', 'exaggeration': 1.5 });
    map.addLayer({
        'id': 'hillshade',
        'source': 'mapbox-dem',
        'type': 'hillshade',
        'paint': {
            'hillshade-exaggeration': 0.5
        }
    });
});
```

코드 출처: 1

**3D 모델 추가**

더 나아가, Three.js나 Babylon.js와 같은 외부 3D 라이브러리를 `custom` 레이어와 통합하여 지도 위에 복잡한 3D 모델(예: 비행기, 자동차)을 렌더링할 수도 있습니다.1 이는 MapLibre의 확장성을 보여주는 좋은 예입니다.

이러한 고급 기능들은 MapLibre GL JS가 단순한 지도 라이브러리를 넘어, 강력한 지리 정보 시각화 플랫폼으로 기능하게 하는 핵심 요소들입니다.

## 6.  MapLibre 생태계

MapLibre GL JS의 강력함은 코어 라이브러리 자체뿐만 아니라, 그 주변을 둘러싼 방대한 생태계에서도 나옵니다. 다양한 플러그인, 프레임워크 통합, 개발 도구들은 개발자가 더 적은 노력으로 더 풍부한 기능을 구현할 수 있도록 돕습니다.

### 6.1  플러그인으로 기능 확장하기

MapLibre 커뮤니티는 다양한 기능을 플러그인 형태로 개발하여 공유하고 있습니다. 공식 문서의 플러그인 페이지나 "Awesome MapLibre" GitHub 저장소에서 유용한 플러그인 목록을 확인할 수 있습니다.40

- **UI 및 컨트롤 플러그인**:
  - `maplibre-gl-measures`: 지도 위에서 거리나 면적을 측정하는 컨트롤을 추가합니다.42
  - `maplibre-gl-export`: 현재 지도 뷰를 PNG, JPEG, SVG 이미지나 PDF 파일로 내보내는 컨트롤을 추가합니다.41
  - `maplibregl-minimap`: 지도 한쪽에 미니맵(개요도)을 표시하는 컨트롤을 추가합니다.41
- **지오코딩 및 검색 플러그인**:
  - `maplibre-gl-geocoder`: 주소나 장소 이름을 검색하여 해당 위치로 이동하는 지오코딩 컨트롤을 추가합니다. Amazon Location Service 43, MapTiler 44, Stadia Maps 41 등 다양한 서비스와 연동 가능합니다.
- **개발 및 디버깅 플러그인**:
  - `maplibre-gl-inspect`: 지도 위 벡터 타일의 피처 정보와 속성을 쉽게 확인할 수 있는 검사 도구를 추가합니다. 개발 과정에서 데이터 확인에 매우 유용합니다.45
- **그리기 및 편집 플러그인**:
  - `maplibre-geoman`: 사용자가 지도 위에 점, 선, 다각형을 직접 그리거나 편집할 수 있는 기능을 제공합니다.41

### 6.2  프레임워크 및 서비스와의 통합

MapLibre GL JS는 현대적인 웹 개발 환경과 잘 통합됩니다.

**프론트엔드 프레임워크**

React, Angular, Vue.js와 같은 프레임워크에서 MapLibre를 더 쉽게 사용할 수 있도록 도와주는 래퍼(wrapper) 라이브러리들이 있습니다. 이 라이브러리들은 맵 객체의 생명주기를 프레임워크의 컴포넌트 생명주기(예: React의 `useEffect` 훅)에 맞춰 관리해주고, 선언적인 방식으로 맵을 다룰 수 있게 해줍니다.

- **React**: `react-map-gl`, `react-map-components-maplibre` 등의 라이브러리가 있으며, `useEffect` 훅을 사용하여 컴포넌트가 마운트될 때 맵을 초기화하는 패턴이 일반적입니다.27
- **Angular**: `ngx-maplibre-gl`은 Angular 애플리케이션에 MapLibre를 통합하기 위한 강력한 바인딩을 제공합니다.41

**클라우드 및 지도 서비스**

MapLibre는 특정 타일 제공업체에 종속되지 않으므로, 다양한 외부 서비스와 연동하여 사용할 수 있습니다.

- **Amazon Location Service**: AWS에서 제공하는 완전 관리형 위치 정보 서비스로, MapLibre와 원활하게 통합됩니다. 인증 헬퍼와 지오코더 플러그인을 통해 지도, 검색, 라우팅, 추적 등의 기능을 쉽게 애플리케이션에 추가할 수 있습니다.43
- **MapTiler**: 고품질의 벡터 타일과 스타일, 지오코딩 등 다양한 지도 API를 제공합니다. 많은 MapLibre 공식 예제에서 MapTiler의 타일을 사용하고 있으며, API 키를 발급받아 `style` URL에 포함시키는 방식으로 사용합니다.47
- **Stadia Maps**: MapLibre 프로젝트의 창립 멤버 중 하나로, API 키 또는 도메인 기반 인증을 통해 지도 서비스를 제공합니다.49

이러한 서비스와 통합할 때는 보통 `transformRequest` 맵 옵션을 사용하여, 타일이나 스타일을 요청하는 모든 URL에 API 키를 동적으로 추가하는 방식을 사용합니다.

### 6.3  개발자를 위한 도구

MapLibre 생태계에는 개발 워크플로우를 완성하는 데 도움이 되는 핵심적인 도구들이 있습니다.

| 도구/플러그인 이름       | 카테고리        | 설명                                                         |
| ------------------------ | --------------- | ------------------------------------------------------------ |
| **Maputnik**             | 스타일 편집기   | GUI를 통해 시각적으로 맵 스타일을 생성하고 편집할 수 있는 도구입니다. 복잡한 JSON을 직접 작성할 필요가 없습니다.14 |
| **Martin**               | 타일 서버       | Rust로 작성된 고성능 벡터 타일 서버입니다. PostGIS 데이터베이스나 MBTiles, PMTiles 파일로부터 동적으로 타일을 생성하여 서비스합니다.14 |
| **maplibre-gl-geocoder** | 지오코딩        | 주소 및 장소 검색 기능을 지도에 쉽게 추가할 수 있는 컨트롤 플러그인입니다.41 |
| **maplibre-gl-inspect**  | 개발/디버깅     | 지도 위 피처의 데이터를 실시간으로 확인하여 디버깅을 용이하게 하는 필수 개발 도구입니다.45 |
| **maplibre-geoman**      | 그리기/편집     | 사용자가 지도 위에 도형을 그리거나 편집할 수 있는 기능을 제공합니다.41 |
| **ngx-maplibre-gl**      | 프레임워크 통합 | Angular 애플리케이션에서 MapLibre를 사용하기 위한 래퍼 라이브러리입니다.41 |
| **react-map-gl**         | 프레임워크 통합 | React 애플리케이션에서 MapLibre(및 Mapbox)를 선언적으로 사용하도록 돕는 래퍼 라이브러리입니다.41 |

데이터 출처: 14

- **MapLibre Font Maker**: 커스텀 폰트 파일을 MapLibre가 요구하는 SDF(Signed Distance Field) 글리프 형식으로 손쉽게 변환해주는 웹 기반 도구입니다. 별도 설치 없이 드래그 앤 드롭 방식으로 사용할 수 있습니다.50

이러한 도구와 플러그인들을 적극적으로 활용하면, 개발자는 지도 기능의 '바퀴를 재발명'하는 대신 애플리케이션의 핵심 비즈니스 로직에 더 집중할 수 있습니다.

## 7.  모범 사례 및 추가 자료

이 마지막 파트에서는 프로덕션 수준의 애플리케이션을 구축하기 위한 성능 최적화 팁과, 지속적인 학습을 위한 유용한 자료들을 소개합니다.

### 7.1  성능 최적화 및 고려사항

빠르고 반응성 좋은 지도는 사용자 경험의 핵심입니다. 다음은 MapLibre GL JS 애플리케이션의 성능을 최적화하기 위한 몇 가지 실용적인 조언입니다.

- **WebGL 지원 확인**: 지도를 초기화하기 전에 사용자의 브라우저가 WebGL을 지원하는지 확인하는 것이 좋습니다. `maplibregl.supported()` 메서드는 boolean 값을 반환하므로, 이를 통해 WebGL을 지원하지 않는 사용자에게는 대체 콘텐츠를 보여주는 등의 예외 처리를 할 수 있습니다.51

- **줌 레벨 제한**: 레이어를 추가할 때 `minzoom`과 `maxzoom` 속성을 설정하여, 해당 레이어가 특정 줌 레벨 범위에서만 렌더링되도록 제한하십시오.20 예를 들어, 상세한 건물 레이어는 지도를 많이 확대했을 때(

  `minzoom: 15`)만 표시되도록 하여 불필요한 렌더링 부하를 줄일 수 있습니다.

- **GeoJSON 데이터 단순화**: 복잡한 다각형이나 선으로 구성된 대용량 GeoJSON 파일을 사용하는 경우, 서버 측에서 미리 데이터를 단순화(예:([https://turfjs.org/)의](https://turfjs.org/)의) `simplify` 사용)하여 전송되는 데이터의 양을 줄이는 것이 좋습니다.

- **타일 로딩 정책 이해**: 지도 생성 시 `cancelPendingTileRequestsWhileZooming` 옵션(기본값 `true`)은 사용자가 줌인할 때 아직 로드되지 않은 이전 줌 레벨의 타일 요청을 취소합니다. 이를 `false`로 설정하면 더 부드러운 시각적 전환을 제공할 수 있지만, 단기간에 더 많은 타일을 요청하게 되어 느린 기기에서는 부담이 될 수 있습니다.33

- **레이어와 표현식의 복잡도**: 너무 많은 레이어를 사용하거나, 모든 피처에 대해 복잡한 `interpolate` 표현식을 적용하면 렌더링 성능에 영향을 줄 수 있습니다. 꼭 필요한 경우가 아니라면 스타일을 최대한 단순하게 유지하는 것이 좋습니다.

### 7.2  문서 및 커뮤니티 활용하기

정적 가이드는 모든 문제를 해결해 줄 수 없습니다. 훌륭한 개발자는 공식 문서와 커뮤니티를 통해 스스로 문제를 해결하고 새로운 것을 학습하는 방법을 압니다. MapLibre는 이를 위한 훌륭한 자원들을 제공합니다.

- **공식 문서 (Official Documentation)**: 모든 정보의 시작점입니다. 튜토리얼, 가이드, API 레퍼런스 등 모든 공식 자료로 연결됩니다.3
- **예제 (Examples)**: 특정 기능을 어떻게 구현하는지 보여주는 수백 개의 실용적인 코드 예제가 있습니다. 아이디어를 얻거나 코드를 복사하여 시작하기에 가장 좋은 곳입니다.1
- **API 레퍼런스 (API Reference)**: `Map`, `Marker`, `Popup` 등 모든 클래스와 메서드, 옵션에 대한 상세한 설명을 제공합니다. 특정 기능의 세부 사항이 궁금할 때 참조해야 합니다.52
- **스타일 명세 (Style Specification)**: 레이어 유형, `paint` 및 `layout` 속성, 표현식 등 스타일링에 대한 모든 규칙을 정의한 최종 권위 문서입니다. 스타일링의 '왜'가 궁금할 때 깊이 있게 파고들 수 있습니다.15
- **커뮤니티 참여**:
  - **GitHub**: 버그를 보고하거나, 새로운 기능을 제안하거나, 직접 코드 기여를 통해 프로젝트에 참여할 수 있습니다.3
  - **Slack**: OpenStreetMap US 슬랙의 `#maplibre` 채널에서 전 세계의 다른 개발자들과 질문을 주고받고 토론할 수 있습니다.8

효과적인 학습 전략은 다음과 같습니다. 먼저 **예제**를 통해 원하는 기능과 유사한 구현을 찾아보고, **API 레퍼런스**를 참조하여 코드를 자신의 요구에 맞게 수정합니다. 그리고 스타일링의 근본적인 원리가 궁금해지면 **스타일 명세**를 읽어보는 순서로 접근하는 것이 좋습니다. 이 가이드가 그 여정의 훌륭한 출발점이 되기를 바랍니다.

## 8. 결론

MapLibre GL JS는 단순한 지도 라이브러리를 넘어, 고성능 데이터 시각화를 위한 강력하고 유연한 오픈소스 플랫폼입니다. WebGL 기반의 GPU 가속 렌더링과 벡터 타일 처리 능력은 기존의 래스터 방식으로는 불가능했던 부드러운 상호작용과 동적인 시각화를 가능하게 합니다.

이 보고서를 통해 살펴본 바와 같이, MapLibre GL JS의 핵심은 **스타일, 소스, 레이어**로 구성된 명확한 아키텍처에 있습니다. 데이터의 정의(소스)와 시각적 표현(레이어)을 분리함으로써 개발자는 높은 수준의 유연성과 재사용성을 확보할 수 있습니다. GeoJSON, 마커, 팝업과 같은 기본 기능을 손쉽게 구현할 수 있을 뿐만 아니라, 표현식(Expressions)을 활용한 데이터 기반 동적 스타일링, `flyTo`와 같은 극적인 카메라 애니메이션, `fill-extrusion`을 이용한 3D 시각화 등 고급 기능들을 통해 정보 전달력이 뛰어난 몰입형 애플리케이션을 구축할 수 있습니다.

또한, MapLibre GL JS를 선택하는 것은 기술적 우수성을 넘어 **오픈소스와 커뮤니티**라는 가치를 선택하는 것이기도 합니다. 특정 기업의 정책에 종속되지 않는 3-Clause BSD 라이선스는 개발자에게 완전한 자유를 보장하며, 활발한 커뮤니티는 플러그인, 개발 도구, 프레임워크 통합 등 풍부한 생태계를 통해 라이브러리의 가능성을 지속적으로 확장하고 있습니다.

물론 Leaflet에 비해 초기 학습 곡선이 다소 높을 수 있지만, 복잡한 지리 정보를 다루거나, 고성능의 동적 시각화가 필수적이거나, 3D 기능이 요구되는 프로젝트라면 MapLibre GL JS는 그 복잡성을 상쇄하고도 남을 강력한 결과물을 제공할 것입니다. 이 보고서가 MapLibre GL JS의 세계로 첫발을 내딛는 개발자들에게 든든한 나침반이 되기를 바랍니다.

#### **참고 자료**

1. Overview - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/
2. Mapbox API(Mapbox GL JS) - 개발 성장 일ブl - 티스토리, accessed July 6, 2025, https://bom-na.tistory.com/145
3. MapLibre GL JS - Interactive vector tile maps in the browser - GitHub, accessed July 6, 2025, https://github.com/maplibre/maplibre-gl-js
4. Chapter 14 MapLibre GL JS | Introduction to Web Mapping, accessed July 6, 2025, https://geobgu.xyz/web-mapping-2022/maplibre.html
5. MapLibre GL JS vs. Leaflet: Choosing the right tool for your interactive map, accessed July 6, 2025, https://blog.jawg.io/maplibre-gl-vs-leaflet-choosing-the-right-tool-for-your-interactive-map/
6. MapLibre GL - MapboxGL JS의 오픈소스 포크 - GeekNews, accessed July 6, 2025, https://news.hada.io/topic?id=4026
7. Mapbox GL JS Is No Longer Open Source - WP Tavern, accessed July 6, 2025, https://wptavern.com/mapbox-gl-js-is-no-longer-open-source
8. maplibre-gl - NPM, accessed July 6, 2025, https://www.npmjs.com/package/maplibre-gl
9. ngx-maplibre-gl/LICENCE at main - MIT license - GitHub, accessed July 6, 2025, https://github.com/maplibre/ngx-maplibre-gl/blob/main/LICENCE
10. MapLibre GL JS - Best of JS, accessed July 6, 2025, https://bestofjs.org/projects/maplibre-gl-js
11. MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/
12. MapBox migration guide - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/guides/mapbox-migration-guide/
13. Display a map - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/simple-map/
14. MapLibre, accessed July 6, 2025, https://maplibre.org/
15. Style Specifications - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/style-spec/
16. Work with sources and layers | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/guides/styles/work-with-layers/
17. Style - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/classes/Style/
18. Style a feature layer | MapLibre GL JS - Esri Developer - ArcGIS Online, accessed July 6, 2025, https://developers.arcgis.com/maplibre-gl-js/styles-and-data-visualization/style-a-feature-layer/
19. Layers - MapLibre Style Spec, accessed July 6, 2025, https://maplibre.org/maplibre-style-spec/layers/
20. Layers | Mapbox Style Spec | Mapbox Docs, accessed July 6, 2025, https://docs.mapbox.com/style-spec/reference/layers/
21. Add a polygon to a map using a GeoJSON source | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/geojson-polygon/
22. Marker - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/classes/Marker/
23. Attach a popup to a marker instance - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/attach-a-popup-to-a-marker-instance/
24. MapLibre GL JS | Display a popup - Jawg Maps, accessed July 6, 2025, https://www.jawg.io/docs/integration/maplibre-gl-js/add-popup/
25. Display a popup on click | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/popup-on-click/
26. Display a popup on hover | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/popup-on-hover/
27. Interactive Maps with MapLibre GL JS & Ola Maps: A Developer's Guide 🗺️ - Medium, accessed July 6, 2025, [https://medium.com/@topi9864/interactive-maps-with-maplibre-gl-js-ola-maps-a-developers-guide-%EF%B8%8F-622e2b41815c](https://medium.com/@topi9864/interactive-maps-with-maplibre-gl-js-ola-maps-a-developers-guide-️-622e2b41815c)
28. Add a GeoJSON line - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/add-a-geojson-line/
29. Add a line to a map using a GeoJSON source | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/geojson-line/
30. Add a GeoJSON polygon - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/add-a-geojson-polygon/
31. MapEventType - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/type-aliases/MapEventType/
32. DoubleClickZoomHandler - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/classes/DoubleClickZoomHandler/
33. MapOptions - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/type-aliases/MapOptions/
34. Fly to a location - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/fly-to-a-location/
35. Fly to a location | Mapbox GL JS, accessed July 6, 2025, https://docs.mapbox.com/mapbox-gl-js/example/flyto/
36. FlyToOptions - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/type-aliases/FlyToOptions/
37. Fly to a location - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/flyto/
38. Fly to a location based on scroll position - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/scroll-fly-to/
39. Customize camera animations - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/examples/camera-animation/
40. A collection of awesome things that use or support MapLibre! - GitHub, accessed July 6, 2025, https://github.com/maplibre/awesome-maplibre
41. Plugins - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/plugins/
42. maplibre-gl-measures CDN by jsDelivr - A CDN for npm and GitHub, accessed July 6, 2025, https://www.jsdelivr.com/package/npm/maplibre-gl-measures
43. Amazon Location Service, MapLibre GL JS Geocoder용 Places 통합 플러그인 릴리스 - AWS, accessed July 6, 2025, https://aws.amazon.com/ko/about-aws/whats-new/2024/04/amazon-location-service-plugin-maplibre-gl-js-geocoder/
44. MapLibre GL JS | Geocoding | JavaScript maps SDK - MapTiler documentation, accessed July 6, 2025, https://docs.maptiler.com/sdk-js/modules/geocoding/api/usage/maplibre-gl-js/
45. Maplibre GL Inspect - NPM, accessed July 6, 2025, https://www.npmjs.com/package/@maplibre/maplibre-gl-inspect
46. How to display MapLibre GL JS map using React JS - MapTiler documentation, accessed July 6, 2025, https://docs.maptiler.com/react/maplibre-gl-js/how-to-use-maplibre-gl-js/
47. Get Started with Angular and MapLibre GL JS - MapTiler documentation, accessed July 6, 2025, https://docs.maptiler.com/angular/maplibre-gl-js/get-started/
48. Using MapLibre GL JS with Amazon Location Service, accessed July 6, 2025, https://docs.aws.amazon.com/location/previous/developerguide/tutorial-maplibre-gl-js.html
49. Quickstart: MapLibre GL JS - Stadia Maps Documentation, accessed July 6, 2025, https://docs.stadiamaps.com/tutorials/vector-maps-with-maplibre-gl-js/
50. MapLibre Font Maker: 맵 폰트 변환 도구 소개 - 요긴소프트, accessed July 6, 2025, https://www.yoginsoft.com/67
51. Check if MapLibre GL JS is supported | Guides | Maps apis - MapTiler documentation, accessed July 6, 2025, https://docs.maptiler.com/guides/maps-apis/maps-platform/check-if-maplibre-gl-js-is-supported/
52. Map - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/classes/Map/
53. Intro - MapLibre GL JS, accessed July 6, 2025, https://maplibre.org/maplibre-gl-js/docs/API/

