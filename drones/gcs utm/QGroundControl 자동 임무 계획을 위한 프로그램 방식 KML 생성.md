---
layout: page
title: QGroundControl 자동 임무 계획을 위한 프로그램 방식 KML 생성
permalink: /drones/gcs utm/QGroundControl 자동 임무 계획을 위한 프로그램 방식 KML 생성
---


무인 항공기(UAV) 기술이 농업, 측량, 기반 시설 점검 등 다양한 산업 분야에 통합됨에 따라, 정밀하고 반복 가능하며 자동화된 임무 계획의 필요성이 급증하고 있습니다. 이러한 임무는 종종 지리 정보 시스템(GIS) 데이터베이스, 센서 판독값, 분석 모델과 같은 외부 데이터 소스에서 파생됩니다. 이러한 외부 데이터를 실행 가능한 비행 경로로 변환하는 것은 중요한 과제이며, 이 보고서는 이 프로세스를 자동화하기 위한 강력하고 기술적으로 타당한 방법론을 제시합니다.

KML(Keyhole Markup Language)은 지리 공간 데이터를 시각화하기 위한 OGC(Open Geospatial Consortium) 표준 XML 기반 형식입니다.1 주로 Google Earth와 같은 지구 브라우저에서 데이터를 표시하는 데 사용되지만 4, 드론 임무 계획 워크플로우에서도 독특한 역할을 수행합니다. 그러나 이 역할은 종종 오해를 불러일으킵니다.

이 보고서의 핵심 전제는 QGroundControl(QGC)이 KML을 기본 임무 파일 형식으로 사용하지 않는다는 것입니다. QGC의 기본 형식은 임무, 지오펜스, 랠리 포인트를 포함하는 포괄적인 JSON 기반 `.plan` 파일입니다.6 대신 QGC는 KML을 **기하학적 경계**를 가져오는 편리한 수단으로 활용합니다. 이 경계는 QGC 자체의 정교한 임무 패턴 생성기(예: Survey 또는 Corridor Scan)를 위한 입력값 역할을 합니다.8

이러한 패러다임은 개발자가 직면하는 많은 일반적인 문제를 설명합니다. 사용자는 종종 KML 파일이 비행 경로로 직접 변환될 것이라고 가정합니다. 예를 들어, 비행할 경로를 나타내는 좌표 세트를 가진 사용자는 `<LineString>` 또는 다수의 `<Point>` 요소로 KML을 생성할 수 있습니다.9 그런 다음 QGC의 계획 보기에서 'Survey' 도구를 선택하고 이 KML을 로드하려고 시도합니다.8 그러나 QGC의 Survey 도구 파서는 작동 영역을 정의하기 위해 특별히 `<Polygon>` 요소를 찾도록 설계되었습니다.12

`<Polygon>`을 찾지 못하면 "Unable to find Polygon node in KML"과 같은 구체적인 오류 메시지와 함께 실패합니다.9 이 실패는 KML 파일 자체의 결함이 아니라, KML에 포함된 기하학적 구조와 특정 QGC 도구의 요구 사항 간의 불일치에서 비롯됩니다. 이는 워크플로우가 `KML -> 임무`가 아니라 `KML 기하학 -> QGC 도구 -> 임무`임을 보여줍니다.

따라서, 자체 개발 소프트웨어를 사용하여 QGroundControl용 비행 경로를 성공적으로 프로그래밍 방식으로 생성하는 것은 기능이 풍부한 KML 파일을 만드는 데 달려 있는 것이 아니라, 대상 QGC 임무 패턴 도구에 맞춰진 단일하고 구체적인 기하학 유형(`Polygon` 또는 `LineString`)을 포함하는 최소한의 구조적으로 올바른 KML 파일을 생성하는 데 달려 있습니다. 이 보고서는 이 워크플로우의 각 단계를 체계적으로 분석하여 개발자가 강력하고 안정적이며 효율적인 자동 임무 계획 파이프라인을 구축하는 데 필요한 기술적 지식과 실용적인 지침을 제공합니다.


QGC 임무 계획을 위한 KML 파일을 프로그래밍 방식으로 생성하려면 KML 표준의 특정 하위 집합에 대한 정밀한 이해가 필요합니다. QGC의 가져오기 엔진은 완전한 기능을 갖춘 KML 파서가 아니므로, 성공적인 통합을 위해서는 QGC가 예상하는 특정 태그와 구조에 엄격하게 부합하는 것이 중요합니다. 이 섹션에서는 QGC 임무 계획과 직접적으로 관련된 KML 요소의 기술적 사양을 자세히 설명합니다.


모든 유효한 KML 파일은 특정 상용구 구조로 시작해야 합니다. 이러한 요소는 파일의 무결성을 보장하고 파서가 내용을 올바르게 해석할 수 있도록 하는 데 필수적입니다.

- **XML 선언:** 모든 KML 파일의 첫 번째 줄은 예외 없이 `<?xml version="1.0" encoding="UTF-8"?>` XML 헤더여야 합니다. 이 줄 앞에 공백이나 다른 문자가 있어서는 안 됩니다.4 이 선언은 파일이 XML 표준을 따르고 UTF-8 문자 인코딩을 사용함을 명시합니다.

- **KML 네임스페이스:** 두 번째 줄은 `<kml xmlns="http://www.opengis.net/kml/2.2">` 루트 요소여야 합니다. 이 태그는 두 가지 중요한 목적을 가집니다. 첫째, 문서의 루트 요소 역할을 합니다. 둘째, OGC KML 2.2 표준에 정의된 기본 네임스페이스를 선언하여 파일 내의 모든 중첩 요소(예: `<Placemark>`, `<Point>`)가 해당 표준의 일부로 해석되도록 합니다.3

- **문서/폴더 계층 구조:** KML 콘텐츠는 일반적으로 `<Document>` 또는 `<Folder>` 요소 내에 구성됩니다. 두 요소 모두 지리적 피처를 그룹화하는 컨테이너 역할을 합니다.3 QGC로 가져오는 목적상, 복잡한 계층 구조는 불필요하며 오히려 파싱 오류의 원인이 될 수 있습니다. 따라서 명확성과 견고성을 위해 모든 피처를 단일 `<Document>` 요소 내에 래핑하는 것이 좋습니다.

- **`<Placemark>` 요소:** `<Placemark>`는 지도상의 단일 지리적 피처를 나타내는 기본 컨테이너입니다. 여기에는 이름, 설명 및 하나 이상의 기하학적 요소가 포함될 수 있습니다.4 QGC와의 호환성을 극대화하려면 KML 파일에 단 하나의 `<Placemark>`만 포함하는 것이 이상적입니다. 이는 가져오기 프로세스 중에 모호성을 제거하고 파서가 처리할 단일하고 명확한 기하학적 대상을 제공합니다.


KML은 여러 기하학 유형을 지원하지만, QGC의 자동 임무 패턴 생성기는 특정 유형에 의존합니다. 개발자는 원하는 임무 유형에 따라 올바른 기하학을 선택해야 합니다.

- **`<Polygon>`:** 폴리곤은 QGC에서 가장 일반적으로 사용되는 기하학으로, 닫힌 영역을 정의합니다.

  - **구조:** `<Polygon>` 요소는 `<outerBoundaryIs>` 요소를 포함해야 합니다. 이 요소는 다시 `<LinearRing>` 요소를 포함하고, 최종적으로 `<coordinates>` 태그가 좌표 목록을 담습니다.13

  - **QGC 적용:** 이 기하학은 **Survey** 패턴 도구에서 관심 영역(Area of Interest, AOI)을 정의하는 데 독점적으로 사용됩니다.8 유효한 `<LinearRing>`을 형성하려면 좌표 목록의 첫 번째 좌표 쌍과 마지막 좌표 쌍이 정확히 일치해야 하며, 이는 닫힌 폴리곤을 보장합니다.13

- **`<LineString>`:** 라인스트링은 열린 경로 또는 일련의 연결된 선분을 정의하는 데 사용됩니다.

  - **구조:** `<LineString>`은 `<coordinates>` 태그 내에 두 개 이상의 좌표 쌍 목록을 포함하는 간단한 구조를 가집ed니다.4
  - **QGC 적용:** 이 기하학은 **Corridor Scan** 패턴 도구에서 임무의 중심선을 정의하는 데 독점적으로 사용됩니다.10 QGC는 이 라인스트링을 기반으로 복도 폭과 기타 매개변수를 고려하여 비행 경로를 생성합니다.

- **`<Point>`:** 포인트는 단일 지리적 위치를 정의합니다.

  - **구조:** `<Point>` 요소는 `<coordinates>` 태그 내에 단일 좌표 튜플을 포함합니다.4
  - **QGC 적용:** QGC의 KML *패턴* 가져오기 기능은 `<Point>` 요소를 대부분 무시합니다. KML이 포인트를 지원하지만, Survey 또는 Corridor Scan 임무를 생성하는 데 사용할 수 없습니다. 여러 `<Point>` 플레이스마크로 구성된 KML을 로드하여 웨이포인트 임무를 생성하려는 시도는 실패할 것입니다.11 이러한 경우, QGC의 일반 텍스트 임무 파일 형식이 더 적절한 대안입니다.16


좌표 데이터의 형식은 KML 파일의 정확성과 유효성에 매우 중요합니다. 사소한 오류라도 임무가 완전히 잘못된 위치에 계획되는 결과를 초래할 수 있습니다.

- **좌표 순서:** KML 표준은 좌표를 `경도, 위도, 고도` 순서로 지정하도록 명시하고 있습니다.3 이는 많은 GIS 애플리케이션이나 Google Maps UI에서 흔히 사용되는 `위도, 경도` 순서와 반대이므로 개발자가 자주 저지르는 실수 중 하나입니다. 소프트웨어에서 좌표를 KML로 내보낼 때 이 순서를 반드시 준수해야 합니다.

- **좌표계:** KML은 암시적으로 WGS84(World Geodetic System of 1984) 측지계를 사용합니다.3 이는 개발자의 소스 데이터가 KML 생성 전에 WGS84로 변환되거나 이미 해당 좌표계에 있어야 함을 의미합니다. 다른 좌표계를 사용하면 QGC에서 위치가 크게 어긋날 수 있습니다.

- **고도:** 좌표 튜플의 세 번째 값은 미터 단위의 고도입니다. 이 값이 생략되면 기본값인 0(해수면 근처)으로 간주됩니다.3

- **`<altitudeMode>`:** 이 태그는 고도가 지표면, 해수면 또는 해저면 중 어디를 기준으로 하는지를 정의합니다. 주요 값으로는 `clampToGround`(지면에 고정), `relativeToGround`(지면으로부터의 상대 고도), `absolute`(해수면 기준 절대 고도)가 있습니다. QGC의 패턴 생성기는 UI에서 사용자가 설정한 비행 고도를 우선적으로 사용하므로, KML에서 2D 경계를 정의할 때는 고도를 생략하거나 `clampToGround`를 사용하는 것이 가장 안전하고 간단한 접근 방식입니다. 이는 임무 고도가 나중에 QGC 인터페이스에서 명시적으로 설정될 것임을 보장합니다.

이러한 구조적 요구 사항을 이해하는 것은 QGC와의 안정적인 통합을 위한 첫걸음입니다. QGC 개발자가 "매우 간단한 KML 폴리곤 로딩 코드"를 작성했다고 언급한 것은 중요한 단서입니다.10 이는 QGC의 파서가 Google Earth와 같은 완전한 기능을 갖춘 엔진이 아니라, 특정 태그를 찾는 데 특화된 제한적인 파서임을 시사합니다. 완전한 KML 파일은 스타일, 설명, 네트워크 링크, 시간 범위 등 다양한 요소를 포함할 수 있지만 2, 이러한 추가 요소들은 QGC의 파서에 의해 무시되거나 최악의 경우 파싱 실패를 유발할 수 있습니다. 따라서 개발자를 위한 가장 견고한 전략은 "최소주의적 원칙"을 따르는 것입니다. 즉, 하나의 `<Document>`, 하나의 `<Placemark>`, 그리고 대상 QGC 도구에 필요한 단일 기하학(`Survey`용 `<Polygon>` 또는 `Corridor Scan`용 `<LineString>`)만 포함하는 KML을 생성하는 것입니다. 이 접근 방식은 파싱 오류의 가능성을 최소화하고 QGC가 파일을 사용하는 방식과 완벽하게 일치합니다.


KML 파일의 구조적 요구 사항을 이해했다면, 다음 단계는 사용자 정의 소프트웨어 내에서 이 파일을 프로그래밍 방식으로 생성하는 것입니다. Python은 강력한 지리 공간 라이브러리 생태계 덕분에 이 작업에 이상적인 언어입니다. 이 섹션에서는 Python을 사용하여 QGC 호환 KML 파일을 생성하기 위한 실용적인 코드와 지침을 제공합니다.


Python에는 KML 생성을 위한 여러 라이브러리가 있지만, QGC 임무 계획이라는 특정 사용 사례에 가장 적합한 것을 선택하는 것이 중요합니다.

- **`simplekml`:** KML 생성을 단순화하기 위해 설계된 고수준 라이브러리입니다. API는 `newpoint()`, `newlinestring()`, `newpolygon()`과 같이 KML 개념에 직접 매핑되어 직관적입니다.18 XML 구조와 상용구(boilerplate)를 자동으로 처리하므로 개발자는 데이터 자체에 집중할 수 있습니다. 단순성과 직접성 덕분에 이 사용 사례에 **가장 권장되는 라이브러리**입니다.
- **`pykml`:** `lxml`을 기반으로 하는 저수준 라이브러리로, XML 트리를 구축하는 데 더 Pythonic하고 객체 지향적인 방법을 제공합니다.22 더 세분화된 제어가 가능하지만 KML 스키마와 XML 구성에 대한 깊은 이해가 필요합니다. QGC가 요구하는 최소한의 KML을 생성하기에는 과도한 기능입니다.
- **기타 라이브러리:** `fastkml` 26 또는 내장 `xml.dom.minidom` 28과 같은 대안도 있지만, 이 특정 작업에는 `simplekml`보다 덜 이상적입니다.


개발자가 자신의 데이터(좌표 목록)를 올바른 라이브러리 함수에 신속하게 매핑할 수 있도록 다음 표는 필요한 핵심 함수와 매개변수를 요약합니다. 이 표는 광범위한 문서를 읽지 않고도 "폴리곤은 어떻게 만드나?" 또는 "라인은 어떻게 만드나?"와 같은 직접적인 질문에 답하는 빠른 참조 가이드 역할을 합니다.

| 함수                  | QGC에서의 목적                              | 주요 매개변수             | 예제 코드 스니펫                                             |
| --------------------- | ------------------------------------------- | ------------------------- | ------------------------------------------------------------ |
| `simplekml.Kml()`     | KML 문서를 초기화합니다.                    | `name`, `open`            | `kml = simplekml.Kml(name="Survey Mission")`                 |
| `kml.newpolygon()`    | **Survey** 도구용 폴리곤을 생성합니다.      | `name`, `outerboundaryis` | pol = kml.newpolygon(name="AOI")pol.outerboundaryis = [(lon1, lat1),...] |
| `kml.newlinestring()` | **Corridor Scan** 도구용 라인을 생성합니다. | `name`, `coords`          | ls = kml.newlinestring(name="Path")ls.coords = [(lon1, lat1),...] |
| `kml.save()`          | KML 객체를 `.kml` 파일로 저장합니다.        | `filepath`                | `kml.save("mission_boundary.kml")`                           |


다음은 `simplekml`을 사용하여 Survey 임무의 경계로 사용될 `<Polygon>`이 포함된 KML 파일을 생성하는 완전한 Python 스크립트입니다. 이 스크립트는 `lat` 및 `lon` 열이 있는 CSV 파일을 입력으로 받습니다.

```Python
import csv
import simplekml

def create_polygon_kml_for_survey(input_csv_path, output_kml_path):
    """
    CSV 파일에서 좌표를 읽어 QGC Survey 도구용 Polygon KML을 생성합니다.

    Args:
        input_csv_path (str): 'lat', 'lon' 열이 있는 입력 CSV 파일 경로.
        output_kml_path (str): 생성될 KML 파일의 경로.
    """
    coords =
    try:
        with open(input_csv_path, mode='r', encoding='utf-8') as infile:
            reader = csv.DictReader(infile)
            for row in reader:
                # KML은 (경도, 위도) 순서를 요구합니다. [19, 29]
                lon = float(row['lon'])
                lat = float(row['lat'])
                coords.append((lon, lat))

        if not coords:
            print("오류: CSV 파일에서 좌표를 읽을 수 없습니다.")
            return

        # KML 폴리곤은 닫혀 있어야 하므로 첫 번째 좌표를 마지막에 추가합니다. [13]
        if coords!= coords[-1]:
            coords.append(coords)

        # simplekml 객체 생성
        kml = simplekml.Kml(name="Survey Area of Interest", open=1)

        # 폴리곤 생성 및 좌표 할당 [21, 30]
        pol = kml.newpolygon(name="Survey Boundary")
        pol.outerboundaryis = coords
        
        # 폴리곤 스타일 설정 (QGC에서는 무시되지만 Google Earth에서는 유용함)
        pol.style.polystyle.color = simplekml.Color.changealphaint(100, simplekml.Color.green) # 100은 투명도
        pol.style.polystyle.outline = 1
        pol.style.linestyle.color = simplekml.Color.green
        pol.style.linestyle.width = 3

        # KML 파일 저장
        kml.save(output_kml_path)
        print(f"성공: '{output_kml_path}' 파일이 생성되었습니다.")

    except FileNotFoundError:
        print(f"오류: '{input_csv_path}' 파일을 찾을 수 없습니다.")
    except KeyError as e:
        print(f"오류: CSV 파일에 필요한 열({e})이 없습니다. 'lat', 'lon' 열이 있는지 확인하십시오.")
    except Exception as e:
        print(f"알 수 없는 오류가 발생했습니다: {e}")

# --- 스크립트 실행 예제 ---
if __name__ == '__main__':
    # 예제 CSV 파일 생성 (실제 사용 시 이 부분은 제거)
    with open('survey_points.csv', 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['lat', 'lon'])
        writer.writerow([37.824664, -122.364383])
        writer.writerow([37.824322, -122.364152])
        writer.writerow([37.824844, -122.363965])
        writer.writerow([37.825009, -122.363604])

    create_polygon_kml_for_survey('survey_points.csv', 'qgc_survey_boundary.kml')
```


다음은 `simplekml`을 사용하여 Corridor Scan 임무의 중심선으로 사용될 `<LineString>`이 포함된 KML 파일을 생성하는 스크립트입니다. 이 스크립트 역시 `lat` 및 `lon` 열이 있는 CSV 파일을 입력으로 받습니다.

```Python
import csv
import simplekml

def create_linestring_kml_for_corridor(input_csv_path, output_kml_path):
    """
    CSV 파일에서 좌표를 읽어 QGC Corridor Scan 도구용 LineString KML을 생성합니다.

    Args:
        input_csv_path (str): 'lat', 'lon' 열이 있는 입력 CSV 파일 경로.
        output_kml_path (str): 생성될 KML 파일의 경로.
    """
    coords =
    try:
        with open(input_csv_path, mode='r', encoding='utf-8') as infile:
            reader = csv.DictReader(infile)
            for row in reader:
                # KML은 (경도, 위도) 순서를 요구합니다. [19, 29]
                lon = float(row['lon'])
                lat = float(row['lat'])
                coords.append((lon, lat))

        if len(coords) < 2:
            print("오류: LineString을 생성하려면 최소 2개의 좌표가 필요합니다.")
            return

        # simplekml 객체 생성
        kml = simplekml.Kml(name="Corridor Scan Path", open=1)

        # LineString 생성 및 좌표 할당 [21, 29, 31]
        ls = kml.newlinestring(name="Corridor Centerline")
        ls.coords = coords
        
        # LineString 스타일 설정 (QGC에서는 무시되지만 Google Earth에서는 유용함)
        ls.style.linestyle.color = simplekml.Color.blue
        ls.style.linestyle.width = 5
        # 고도 모드 설정 (예: 지상에 고정)
        ls.altitudemode = simplekml.AltitudeMode.clamptoground

        # KML 파일 저장
        kml.save(output_kml_path)
        print(f"성공: '{output_kml_path}' 파일이 생성되었습니다.")

    except FileNotFoundError:
        print(f"오류: '{input_csv_path}' 파일을 찾을 수 없습니다.")
    except KeyError as e:
        print(f"오류: CSV 파일에 필요한 열({e})이 없습니다. 'lat', 'lon' 열이 있는지 확인하십시오.")
    except Exception as e:
        print(f"알 수 없는 오류가 발생했습니다: {e}")

# --- 스크립트 실행 예제 ---
if __name__ == '__main__':
    # 예제 CSV 파일 생성 (실제 사용 시 이 부분은 제거)
    with open('corridor_points.csv', 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['lat', 'lon'])
        writer.writerow([37.824664, -122.364383])
        writer.writerow([37.824322, -122.364152])
        writer.writerow([37.824787, -122.364167])
        writer.writerow([37.824423, -122.363917])

    create_linestring_kml_for_corridor('corridor_points.csv', 'qgc_corridor_path.kml')
```

이 두 스크립트는 개발자가 자신의 애플리케이션에 통합할 수 있는 견고한 기반을 제공합니다. 핵심은 입력 데이터(이 경우 CSV)를 QGC의 특정 도구가 요구하는 정확한 KML 구조로 변환하는 것입니다. 이 접근 방식을 통해 개발자는 수동 계획의 지루하고 오류가 발생하기 쉬운 단계를 자동화하여 효율성과 반복성을 크게 향상시킬 수 있습니다.


프로그래밍 방식으로 완벽한 KML 파일을 생성했더라도, QGC가 이 파일을 어떻게 해석하고 사용하는지를 이해하지 못하면 워크플로우는 실패할 수 있습니다. 이 섹션에서는 QGC의 KML 가져오기 엔진의 내부 동작, 기능 및 명백한 제한 사항을 분석하여 "어떻게 생성하는가"에서 "어떻게 사용되는가"로 초점을 전환합니다.


QGC는 KML 파일을 범용 임무 파일로 취급하지 않습니다. 대신, 특정 임무 패턴 도구의 컨텍스트 내에서 기하학적 입력을 처리하는 특수 파서를 사용합니다. 이 매핑을 이해하는 것이 성공의 열쇠입니다.

- **Survey 도구:**

  - 사용자가 Plan 뷰에서 "Pattern" -> "Survey"를 선택하고 KML 파일을 로드하면 8, QGC의 파서는 파일 내에서 유효한 

    `<Polygon>` 요소를 구체적으로 검색합니다.11

  - 성공적으로 찾으면 이 폴리곤의 정점(vertices)이 지도에 임무 경계로 그려집니다.12

  - 이 시점에서 KML 파일의 역할은 끝납니다. QGC는 이제 이 경계를 사용하여 고도, 카메라 사양, 중첩률 등 UI에서 설정된 매개변수를 기반으로 웨이포인트 격자를 자동으로 생성합니다.12 KML 파일 자체는 웨이포인트나 비행 속도와 같은 임무 세부 정보를 포함하지 않습니다.

- **Corridor Scan 도구:**

  - 사용자가 "Pattern" -> "Corridor Scan"을 선택하고 KML을 로드하면 파서는 대신 `<LineString>` 요소를 찾습니다.10
  - 이 라인스트링은 복도의 중심 경로로 사용됩니다.
  - QGC는 이 선을 따라 비행 경로를 생성하며, 복도 폭, 측선 간격 및 기타 매개변수는 UI에서 정의됩니다.6

이러한 도구별 동작은 사용자가 겪는 가장 흔한 실패의 근본 원인입니다. Survey 도구에 `<LineString>` KML을 로드하거나 그 반대의 경우, 파서는 필요한 기하학을 찾지 못하고 오류를 발생시키거나 조용히 실패합니다. 이는 개발자가 KML을 생성할 때 최종 사용자가 QGC에서 어떤 도구를 사용할 것인지를 명확히 이해하고 그에 맞춰 파일을 생성해야 함을 의미합니다.


개발자를 위한 명확하고 간결한 참조를 제공하기 위해 다음 표는 KML 아티팩트와 QGC에서의 필수 사용자 조치를 명시적으로 연결합니다. 이 표는 포럼 게시물과 개발자 의견에서 발견된 "부족 지식"을 공식적인 사양으로 체계화하여 일반적인 오류를 방지하는 것을 목표로 합니다.

| KML 기하학     | QGC 도구          | 필수 사용자 조치                                             | 예상 QGC 동작                                                | 알려진 제한 사항 / 일반적인 오류                             |
| -------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `<Polygon>`    | **Survey**        | Plan 뷰에서 Pattern -> Survey 선택 후, "File" 또는 "Load KML/SHP" 버튼 클릭 8 | 폴리곤 경계가 측량 AOI로 로드됩니다. QGC는 내부에 웨이포인트 격자를 생성합니다. | Corridor Scan으로 로드하면 실패합니다. KML에 유효한 `<Polygon>`이 없으면 "Unable to find Polygon node" 오류 발생.11 |
| `<LineString>` | **Corridor Scan** | Plan 뷰에서 Pattern -> Corridor Scan 선택 후, Polyline Tools 아래의 "Load KML..." 클릭 10 | 라인이 임무의 중심 경로로 로드됩니다.                        | Survey로 로드하면 실패합니다. KML 형식이 잘못된 경우 조용히 실패할 수 있음.32 |
| `<Point>`      | (해당 없음)       | (해당 없음)                                                  | 포인트는 임무 패턴을 생성하는 데 사용되지 않습니다. 지도에 표시될 수는 있지만 경로 생성을 위한 조치는 불가능합니다. | 이 방법으로 간단한 웨이포인트 임무를 만들려는 사용자는 성공할 수 없습니다.11 |
| 다중 기하학    | (해당 없음)       | (해당 없음)                                                  | 동작이 정의되지 않았습니다. QGC의 간단한 파서는 찾은 첫 번째 유효한 기하학만 읽거나 완전히 실패할 가능성이 높습니다. | 가장 좋은 방법은 파일당 하나의 기하학을 사용하는 것입니다.   |
| `.kmz` 파일    | (해당 없음)       | (해당 없음)                                                  | 지원되지 않습니다. QGC는 압축된 아카이브를 처리할 수 없습니다.10 | 사용자는 수동으로 KMZ의 압축을 풀고 루트 `.kml` 파일을 로드해야 합니다. |


QGC의 KML 가져오기 기능은 강력하지만 모든 KML 사양을 지원하지는 않습니다. 개발자는 다음과 같은 제한 사항을 인지해야 합니다.

- **KMZ 지원 부재:** QGC 로더는 크로스 플랫폼 압축 해제 기능이 없으므로 `.kmz` 파일은 사용자가 가져오기 전에 수동으로 압축을 풀어야 합니다.10 소프트웨어는 항상 압축되지 않은 

  `.kml` 파일을 생성해야 합니다.

- **스타일 무시:** KML 내의 모든 `<Style>` 또는 `<StyleMap>` 요소는 폐기됩니다. QGC에서 임무의 시각적 표현은 QGC 자체의 UI 렌더링에 의해 결정됩니다. 코드 예제에 스타일을 포함한 것은 Google Earth와 같은 다른 도구에서의 시각화를 돕기 위한 것입니다.

- **설명 및 메타데이터 무시:** `<name>` 및 `<description>` 태그의 정보는 일반적으로 임무 패턴 자체에서 사용되지 않습니다. 이름은 가져오기 프로세스 중에 잠시 표시될 수 있지만 임무 계획에는 영향을 미치지 않습니다.

- **단일 기하학 집중:** 앞서 설명했듯이 파서는 특정 유형의 단일 기하학을 찾도록 설계되었습니다. 여러 `<Placemark>` 또는 혼합된 기하학이 포함된 복잡한 KML은 공식적으로 지원되지 않으며 예측할 수 없는 결과를 초래할 수 있습니다.

결론적으로, QGC의 KML 가져오기 엔진은 특정 목적을 위해 제작된 고도로 전문화된 도구입니다. 그 동작과 한계를 존중하는 것이 자동화된 임무 계획 워크플로우를 성공적으로 구축하는 데 필수적입니다. 개발자는 QGC가 기대하는 정확한 형식으로 데이터를 제공하는 "최소주의적"이고 목표 지향적인 KML 파일을 생성하는 데 집중해야 합니다.


이전 섹션의 모든 개념을 종합하여, 이 섹션에서는 개발자와 운영자를 위한 실용적이고 단계적인 운영 절차를 제시합니다. 이 워크플로우는 사용자 정의 소프트웨어에서 생성된 데이터가 기체에서 실행될 준비가 된 완전한 비행 계획으로 변환되는 전체 과정을 다룹니다.


워크플로우의 첫 단계는 자동화된 소프트웨어 내에서 이루어집니다. 소프트웨어는 원하는 임무의 성격에 따라 생성할 올바른 기하학적 기본 요소를 결정해야 합니다.

- **영역 커버리지(예: 농경지 매핑, 재해 지역 조사):** 이 경우 목표는 정의된 경계 내의 전체 영역을 체계적으로 비행하는 것입니다. 따라서 소프트웨어는 **`<Polygon>`** 기하학을 생성하도록 설계되어야 합니다. 입력 데이터는 폴리곤의 정점을 정의하는 일련의 경위도 좌표가 됩니다.
- **선형 피처(예: 파이프라인, 도로, 강 조사):** 이 시나리오의 목표는 선형 경로를 따라 비행하는 것입니다. 소프트웨어는 **`<LineString>`** 기하학을 생성해야 합니다. 입력 데이터는 경로를 구성하는 순서가 지정된 좌표 목록이 됩니다.

이 초기 결정은 전체 워크플로우의 성공에 매우 중요하며, 소프트웨어 로직에 명시적으로 코딩되어야 합니다.


임무 유형과 해당 기하학이 결정되면, 사용자 정의 소프트웨어는 III장에서 자세히 설명된 Python 스크립트와 유사한 로직을 실행합니다.

- 소프트웨어는 임무별 좌표를 입력으로 사용하여 KML 생성 함수를 호출합니다.
- **출력:** 선택한 기하학(`Polygon` 또는 `LineString`)을 포함하는 단일 `<Placemark>`가 있는 최소한의 깨끗한 `.kml` 파일이 생성됩니다. 이 파일은 운영자가 접근할 수 있는 위치(로컬 디렉토리, 네트워크 공유 또는 클라우드 스토리지)에 저장됩니다.


이제 워크플로우는 자동화된 소프트웨어에서 운영자의 영역으로 전환됩니다. 운영자는 생성된 KML 파일을 사용하여 QGC에서 실제 비행 계획을 구성합니다.

- 운영자는 QGC를 열고 화면 상단의 **Plan** 뷰로 이동합니다.33
- 그런 다음 올바른 임무 패턴 도구를 선택합니다. 폴리곤 KML의 경우 **Survey**를, 라인스트링 KML의 경우 **Corridor Scan**을 선택합니다.8
- 도구의 가져오기 기능("File" 또는 "Load KML/SHP")을 사용하여 2단계에서 생성된 `.kml` 파일을 로드합니다.
- 기하학적 경계가 지도에 나타납니다. 이제 운영자는 QGC의 강력한 UI를 사용하여 다음과 같은 다른 모든 임무 매개변수를 구성해야 합니다:
  - 비행 고도 및 속도
  - 카메라 모델 및 설정
  - 카메라 트리거 간격 (거리 또는 시간 기반)
  - 이미지 중첩률 (전방 및 측면)
  - 기타 패턴별 설정 (예: 선회 시 촬영 여부).12


임무를 기체에 보내기 전에 최종적이고 중요한 검증 단계가 필요합니다.

- 운영자는 Plan 뷰에서 자동 생성된 전체 비행 경로를 신중하게 검토합니다. 여기에는 각 웨이포인트의 위치, 카메라 트리거 지점, 그리고 임무 통계(총 비행 거리, 예상 시간 등)가 포함됩니다.33
- **Terrain Altitude Overlay**를 확인하여 다양한 지형 고도에 대해 안전한 비행고도가 유지되는지 확인하는 것이 특히 중요합니다.33
- 모든 것이 만족스러우면 운영자는 기체를 연결하고 **Upload** 버튼을 클릭합니다. QGC는 시각적 계획을 MAVLink 임무로 변환하여 기체의 비행 컨트롤러로 전송합니다.33 이제 임무는 기체에 저장되어 자율 실행 준비가 완료됩니다.

이 워크플로우를 분석하면 중요한 운영상의 현실이 드러납니다. 이 프로세스는 완전히 "무인" 자동화가 아닙니다. KML 파일은 종종 수동 계획에서 가장 지루하고 오류가 발생하기 쉬운 부분인 **경계 정의**를 자동화합니다. 그러나 운영자는 여전히 고도 및 카메라 설정과 같은 임무별 매개변수를 구성하고 최종 검증 및 업로드를 수행하는 데 필수적인 역할을 합니다.

이것은 명확한 **관심사 분리(separation of concerns)**를 정의합니다. 개발자의 소프트웨어는 **"어디서(where)"** 비행할 것인지를 처리하고, 인간 운영자는 **"어떻게(how)"** 비행할 것인지를 처리하며 최종적인 **"실행/중단(go/no-go)"** 결정을 내립니다. KML은 단지 임무를 위한 템플릿 또는 스캐폴드를 제공할 뿐이며, 운영자가 이를 완성하고 검증하며 승인합니다. 이 점을 이해하는 것은 안전하고 효과적인 자동화 시스템을 설계하는 데 있어 매우 중요합니다.


성공적인 통합을 위해서는 KML 생성 및 가져오기의 기본 사항을 넘어서는 깊이 있는 이해가 필요합니다. 이 섹션에서는 개발자가 "내부적으로" 어떤 일이 일어나는지 이해하고 가장 일반적이고 혼란스러운 문제를 진단하는 데 도움이 되는 고급 기술 개념을 탐구합니다.


전체 워크플로우의 최종적이고 가장 중요한 변환 단계는 QGC가 시각적 계획을 기체의 자동 조종 장치가 직접 실행할 수 있는 명령어로 변환하는 과정입니다. 이 데이터 변환 체인을 이해하는 것은 근본적인 원인을 파악하는 데 중요합니다.

1. **KML 파일 (기하학적 추상화):** 이것은 시작점입니다. `<Polygon>` 또는 `<LineString>`과 같은 기하학적 모양을 정의하는 추상적인 표현입니다.
2. **QGC `.plan` 파일 (GCS별 표현):** KML을 가져오고 UI에서 매개변수를 설정하면 QGC는 이 모든 정보를 자체적인 JSON 기반 `.plan` 파일 형식으로 저장합니다.6 이 파일은 웨이포인트, 카메라 설정, 임무 속도 등 완전한 임무 정의를 포함합니다.
3. **MAVLink 임무 (실행 가능한 명령어):** 운영자가 "Upload"를 클릭하면 QGC는 `.plan` 파일을 MAVLink 프로토콜을 사용하여 자동 조종 장치로 전송되는 일련의 구체적인 명령어로 변환합니다.35 각 명령어는 `MAV_CMD` 열거형의 값에 해당합니다.
   - 단순 웨이포인트는 `MAV_CMD_NAV_WAYPOINT`가 됩니다.37
   - Survey 임무의 경우, QGC는 거리 기반 카메라 트리거를 활성화하기 위해 `MAV_CMD_DO_SET_CAM_TRIGG_DIST`와 같은 명령어를 자동으로 삽입하고, 각 웨이포인트에서 카메라를 제어하기 위해 `MAV_CMD_IMAGE_START_CAPTURE` / `MAV_CMD_IMAGE_STOP_CAPTURE` 또는 `MAV_CMD_DO_DIGICAM_CONTROL`과 같은 명령어를 추가합니다.38

개발자의 관점에서 볼 때, KML 생성의 궁극적인 목표는 QGC의 MAVLink 생성 엔진을 올바르게 트리거하는 것입니다. 개발자는 MAVLink를 직접 작성하지는 않지만, 자신이 생성한 KML이 궁극적으로 어떤 MAVLink 명령어로 변환될 것인지를 이해해야 합니다.


- **오류: "Unable to find Polygon node in KML"**
  - **원인:** 이 보고서에서 반복적으로 강조된 바와 같이, 이는 KML 기하학과 QGC 도구 간의 불일치입니다. 사용자가 `<Polygon>`이 없는 KML(예: `<LineString>`만 있는 KML)을 Survey 도구로 로드하려고 할 때 발생합니다.11
  - **해결책:** Survey 도구를 사용하는 경우 KML에 유효하고 닫힌 `<Polygon>`이 포함되어 있는지 확인하십시오. `<LineString>`을 사용하는 경우 Corridor Scan 도구로 전환하십시오.
- **오류: 가져온 기하학이 잘못된 위치에 표시됨**
  - **원인 1: 지도 제공자 불일치:** QGC에서 다른 지도 타일 제공자(예: Bing 대 Google)가 약간의 지리 참조 오프셋을 가질 수 있다는 것은 알려진 문제입니다. 가져온 KML이 한 지도 배경에서는 정확하게 보이지만 다른 배경에서는 이동한 것처럼 보일 수 있습니다.42 이 경우 기본 좌표 데이터는 일반적으로 정확합니다.
  - **원인 2: 잘못된 좌표계:** KML의 소스 데이터가 WGS84가 아니어서 투영 오류가 발생했습니다. KML 표준 자체는 WGS84를 사용하므로 3, 개발자는 KML을 생성하기 전에 소스 데이터가 올바르게 변환되었는지 확인할 책임이 있습니다.43
  - **해결책:** 먼저 Google Earth와 같은 참조 애플리케이션에서 KML을 확인하여 좌표가 올바른지 검증하십시오. QGC 내에서 지도 제공자를 전환하여 정렬이 바뀌는지 확인하십시오.42 항상 사용자 정의 소프트웨어의 좌표 파이프라인이 KML 출력에 WGS84를 사용하도록 보장하십시오.
- **오류: KML 가져오기가 조용히 실패하거나 QGC가 충돌함**
  - **원인:** KML이 잘못된 형식의 XML이거나 QGC의 간단한 파서가 처리할 수 없는 복잡한 구조(예: 유효하지 않은 문자, 닫히지 않은 태그, 자체 교차 폴리곤)를 포함할 가능성이 높습니다.44
  - **해결책:** 외부 XML 유효성 검사기를 사용하여 생성된 KML 파일의 유효성을 검사하십시오. "최소주의적 원칙"을 준수하고 생성된 파일에서 필수적이지 않은 모든 요소를 제거하여 파서의 부담을 줄이십시오.


KML이 적합하지 않은 경우(예: 복잡한 패턴이 없는 단순한 웨이포인트 목록의 경우), 개발자와 사용자는 다른 옵션을 고려할 수 있습니다.

- **Mission Plain-Text 파일 (`.waypoints`):** QGC는 임무를 위한 간단한 탭으로 구분된 텍스트 형식을 지원합니다.46 이는 기본적인 웨이포인트 간 임무에 대해 KML보다 더 간단하고 직접적인 대안이 될 수 있습니다. 사용자들은 KML이 실패한 사용 사례에서 이 형식을 성공적으로 사용했습니다.11
- **JSON `.plan` 파일:** 궁극적인 제어를 위해 개발자는 QGC `.plan` 파일을 직접 프로그래밍 방식으로 생성할 수 있습니다. 이는 임무 항목, 지오펜스, 카메라 설정 등을 포함하여 QGC의 내부 JSON 구조를 복제해야 하므로 훨씬 더 복잡합니다.6 하지만 완전한 자동화를 제공하며, KML 가져오기의 한계를 뛰어넘는 고급 기술입니다.

문제 해결은 체계적인 접근이 필요합니다. 오류가 발생하면 먼저 KML 파일 자체의 유효성을 검사하고, 그 다음 QGC 내에서의 사용 컨텍스트(올바른 도구 선택)를 확인하고, 마지막으로 QGC 자체의 제한 사항(예: 지도 제공자)을 고려해야 합니다.


이 기술 보고서는 자체 개발 소프트웨어를 사용하여 KML 파일을 생성하고 이를 QGroundControl에 입력하여 비행 경로를 만드는 전체 워크플로우를 심층적으로 분석했습니다. 분석을 통해 성공적인 통합은 KML 표준에 대한 일반적인 지식을 넘어 QGC의 특정 동작과 한계에 대한 미묘한 이해에 달려 있음이 분명해졌습니다.


분석의 핵심 결과는 다음과 같이 요약할 수 있습니다. KML은 QGC에서 직접적인 임무 파일이 아니라, Survey 및 Corridor Scan과 같은 내장 패턴 생성기를 위한 **기하학적 입력**으로 기능합니다. 따라서 성공은 복잡하고 기능이 풍부한 KML을 만드는 것이 아니라, 선택한 QGC 도구와 정확히 일치하는 특정 기하학(`Survey`용 `<Polygon>`, `Corridor Scan`용 `<LineString>`)을 포함하는 최소한의 KML을 생성하는 데 달려 있습니다. 전체 워크플로우는 `사용자 정의 데이터 -> KML -> QGC 패턴 ->.plan -> MAVLink`라는 다단계 데이터 변환 프로세스로 이해해야 합니다. 이 과정에서 KML은 중요한 초기 단계를 자동화하지만, 운영자는 비행 매개변수 구성 및 최종 안전 검증이라는 필수적인 역할을 수행합니다.


이러한 발견을 바탕으로, KML과 QGroundControl을 사용하여 견고하고 안정적인 프로그래밍 방식의 임무 계획 파이프라인을 구축하려는 개발자를 위해 다음과 같은 전략적 권장 사항을 제시합니다.

1. **"최소주의적 원칙"을 수용하십시오:** 가능한 가장 간단한 KML 파일을 생성하십시오. 필수적인 XML/KML 상용구와 단일 `<Placemark>`에 포함된 단일 유효 기하학만 포함하십시오. 스타일, 설명, 다중 피처 또는 기타 고급 KML 기능은 파싱 오류의 잠재적 원인이므로 피해야 합니다.
2. **형식뿐만 아니라 도구를 목표로 하십시오:** 다양한 임무 유형에 대해 서로 다른 KML 파일을 생성하도록 소프트웨어를 설계하십시오. 소프트웨어의 로직은 다음과 같아야 합니다: `if (영역 측량) then generate_polygon_kml(); else if (선형 측량) then generate_linestring_kml();`. 이는 생성된 파일이 QGC의 올바른 도구와 일치하도록 보장합니다.
3. **모든 단계에서 검증하십시오:** 사용자 정의 소프트웨어 내에 검증 검사를 구현하십시오. 생성된 XML 구조의 유효성을 검사하고, 좌표가 올바른 순서와 범위 내에 있는지 확인하십시오. 운영자에게는 임무를 구성하기 전에 QGC에서 가져온 기하학을 시각적으로 확인하고, 업로드 전에 최종 비행 경로를 다시 한 번 검토하도록 안내해야 합니다.
4. **좌표계를 명시적으로 관리하십시오:** 사용자 정의 소프트웨어 내의 모든 내부 좌표 처리가 투영을 인식하고 KML 파일로의 최종 출력이 항상 WGS84(`경도, 위도`) 형식인지 확인하십시오. 이는 가장 흔한 위치 오류의 원인 중 하나를 제거합니다.
5. **운영자를 교육하십시오:** 사용자 정의 소프트웨어의 문서는 전체 워크플로우를 명확하게 설명해야 합니다. KML이 경계만 정의하며, 운영자가 QGC에서 비행 고도, 속도, 안전 설정과 같은 중요한 비행 매개변수를 설정할 책임이 있음을 강조해야 합니다. 이는 안전하고 책임감 있는 운영을 보장합니다.
6. **대안을 고려하십시오:** Survey 또는 Corridor Scan 패턴에 맞지 않는 간단한 웨이포인트 임무의 경우, KML 대신 일반 텍스트 임무 파일 형식을 사용하는 것을 고려하십시오. 이는 더 간단하고 직접적인 해결책이 될 수 있으며, KML 파싱과 관련된 복잡성을 피할 수 있습니다.

이러한 권장 사항을 따르면 개발자는 QGroundControl의 강력한 임무 계획 기능을 활용하는 동시에 수동 데이터 입력과 관련된 오류와 비효율성을 최소화하는 자동화된 시스템을 구축할 수 있습니다. 핵심은 KML을 만능 해결책으로 보지 않고, 더 큰 자동화 생태계 내에서 고도로 전문화된 도구로 취급하는 것입니다.


1. What is KML?-ArcGIS Pro | Documentation, accessed July 10, 2025, https://pro.arcgis.com/en/pro-app/3.3/help/data/kml/what-is-kml-.htm
2. Keyhole Markup Language (KML) - Natural Resources Canada, accessed July 10, 2025, https://natural-resources.canada.ca/science-data/data-analysis/geospatial-data-tools-services/keyhole-markup-language-kml
3. Keyhole Markup Language - Wikipedia, accessed July 10, 2025, https://en.wikipedia.org/wiki/Keyhole_Markup_Language
4. KML Tutorial | Keyhole Markup Language - Google for Developers, accessed July 10, 2025, https://developers.google.com/kml/documentation/kml_tut
5. Mastering KML for Geospatial Insights - Number Analytics, accessed July 10, 2025, https://www.numberanalytics.com/blog/ultimate-guide-to-kml-in-geospatial-analysis
6. Plan File Format | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/file_formats/plan.html
7. QGroundControl v3.2 Release Notes (Detailed) | QGC Guide (4.3), accessed July 10, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/releases/stable_v3.2_long.html
8. Uploading KMLs into QGC - INSPIRED DOCUMENTATION, accessed July 10, 2025, https://docs.inspiredflight.com/inspired-documentation/products/additional-software/qgc/uploading-kmls-into-qgc
9. How to upload a .kml into QGroundControl - PX4 Discussion Forum, accessed July 10, 2025, https://discuss.px4.io/t/how-to-upload-a-kml-into-qgroundcontrol/6146
10. KML/SHP/POLY import support for polygons / Issue #5752 / mavlink/qgroundcontrol - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/5752
11. KML files for mission plans in QGC - QGroundControl - Blue Robotics Community Forums, accessed July 10, 2025, https://discuss.bluerobotics.com/t/kml-files-for-mission-plans-in-qgc/15978
12. Survey (Plan Pattern) | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/plan_view/pattern_survey.html
13. A Look at KML, an Open Standard to Represent and Visualize Spatial Information, accessed July 10, 2025, https://www.geographyrealm.com/look-kml-open-standard-represent-visualize-spatial-information/
14. QGC v3 Release Notes | QGC Guide (master) - QGroundControl Guide, accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/releases/release_note_stable_v3.html
15. KML format, accessed July 10, 2025, https://www.dcs.gla.ac.uk/~jhw/socialgravity/kml_format.html
16. KML files for mission plans in QGC - #2 by zhrandell - Blue Robotics Community Forums, accessed July 10, 2025, https://discuss.bluerobotics.com/t/kml-files-for-mission-plans-in-qgc/15978/2
17. Introduction to KML - Digital humanities - obdurodon.org, accessed July 10, 2025, http://dh.obdurodon.org/kml/kml-tutorial.xhtml
18. Overview - SIMPLEKML 1.3.6 documentation, accessed July 10, 2025, https://simplekml.readthedocs.io/
19. Create a KML file with Python • fredgibbs.net, accessed July 10, 2025, http://fredgibbs.net/tutorials/create-kml-file-python.html
20. simplekml - PyPI, accessed July 10, 2025, https://pypi.org/project/simplekml/
21. Getting Started - SIMPLEKML 1.3.6 documentation - Read the Docs, accessed July 10, 2025, https://simplekml.readthedocs.io/en/latest/gettingstarted.html
22. pyKML Tutorial - Pythonhosted.org, accessed July 10, 2025, https://pythonhosted.org/pykml/tutorial.html
23. pyKML - Google Code, accessed July 10, 2025, https://code.google.com/archive/p/pykml
24. pyKML v0.1.0 documentation - Pythonhosted.org, accessed July 10, 2025, https://pythonhosted.org/pykml/
25. pykml/docs/source/tutorial.rst at master - GitHub, accessed July 10, 2025, https://github.com/shabble/pykml/blob/master/docs/source/tutorial.rst
26. Alternative KML libraries - FastKML's documentation!, accessed July 10, 2025, https://fastkml.readthedocs.io/en/latest/alternatives.html
27. Getting points from KML LineString with Python - GIS Stack Exchange, accessed July 10, 2025, https://gis.stackexchange.com/questions/89543/getting-points-from-kml-linestring-with-python
28. Converting CSV files to KML | Keyhole Markup Language - Google for Developers, accessed July 10, 2025, https://developers.google.com/kml/articles/csvtokml
29. Creating a KML linestring from a CSV with Python and simplekml - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/43229424/creating-a-kml-linestring-from-a-csv-with-python-and-simplekml
30. How to create several polygons with simple kml python? - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/71641039/how-to-create-several-polygons-with-simple-kml-python
31. Linestring Tutorial - SIMPLEKML 1.3.6 documentation - Read the Docs, accessed July 10, 2025, https://simplekml.readthedocs.io/en/latest/tut_linestring.html
32. Corridor Scan Polyline import from KML failing without error #12832 - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/12832
33. Plan View | QGC Guide (4.3), accessed July 10, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/plan_view/plan_view.html
34. Mission Planning in QGroundControl - YouTube, accessed July 10, 2025, https://www.youtube.com/watch?v=zBmC0EeP54Q
35. Mission Protocol / ham_mavdevguide - Hamish Willee, accessed July 10, 2025, https://hamishwillee.gitbooks.io/ham_mavdevguide/en/services/mission.html
36. Mission Protocol - MAVLink Guide, accessed July 10, 2025, https://mavlink.io/en/services/mission.html
37. Is there a MAVLink mission commands list available? - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/29527513/is-there-a-mavlink-mission-commands-list-available
38. MAVLink Camera Triggering from GCS - Google Groups, accessed July 10, 2025, https://groups.google.com/g/drones-discuss/c/huY_BSlIP5w/m/g_NNoU9PCgAJ
39. Camera Protocol v1: Camera Triggering | MAVLink Guide, accessed July 10, 2025, https://mavlink.io/en/services/camera_v1.html
40. Camera trigger not working in QGC missions. / Issue #9292 / mavlink/qgroundcontrol, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/9292
41. KML file format for QGC - QGroundControl - PX4 Discussion Forum, accessed July 10, 2025, https://discuss.px4.io/t/kml-file-format-for-qgc/36680
42. Load kml: Displacement error / Issue #8704 / mavlink/qgroundcontrol - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/8704
43. Imported KML file is not in the correct location in AutoCAD Map 3D or Civil 3D - Autodesk, accessed July 10, 2025, https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/Imported-KML-file-is-not-in-the-correct-location-in-AutoCAD-Map-3D-or-Civil-3D.html
44. Problems in uploading a kml file to create a new place - General - iNaturalist Forum, accessed July 10, 2025, https://forum.inaturalist.org/t/problems-in-uploading-a-kml-file-to-create-a-new-place/3664
45. How to fix a KML import file within QGIS - Geographic Information Systems Stack Exchange, accessed July 10, 2025, https://gis.stackexchange.com/questions/311152/how-to-fix-a-kml-import-file-within-qgis
46. File Formats - MAVLink Guide, accessed July 10, 2025, https://mavlink.io/en/file_formats/

