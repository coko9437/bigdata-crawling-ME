

1.  **다른 API 사용**: "자전거 교통사고 잦은 곳 정보" API를 사용합니다.
2.  **보안 강화**: SSL/TLS 설정을 명시적으로 적용하여 보안 통신을 강화했습니다.
3.  **한글 폰트 설정**: Matplotlib 시각화 시 한글 깨짐을 방지하기 위해 운영체제별로 폰트를 자동으로 설정하는 기능을 포함했습니다.
4.  **데이터 시각화**: 수집된 데이터를 막대 그래프로 시각화하여 한눈에 파악할 수 있도록 합니다.
5.  **구/군별 데이터 수집**: 서울특별시 내 특정 구/군 또는 전체 구/군의 데이터를 반복적으로 수집할 수 있는 기능을 제공합니다.
6.  **더욱 상세한 오류 처리**: API 응답이 비어있거나 JSON 파싱에 실패하는 경우 등을 명시적으로 처리합니다.
---
### 1. 코드 전체적인 흐름

1.  **초기 설정**: 필요한 라이브러리(`urllib.request`, `json`, `pandas`, `datetime`, `urllib.parse`, `ssl`, `matplotlib.pyplot`, `numpy`, `matplotlib.font_manager`, `platform`) 임포트 및 서울특별시 구/군 코드 딕셔너리, SSL/TLS 설정.
2.  **한글 폰트 설정**: `set_korean_font()` 함수를 정의하고 호출하여 Matplotlib에서 한글이 올바르게 표시되도록 합니다.
3.  **데이터 시각화**: `visualizeData()` 함수를 정의하여 Pandas DataFrame을 입력받아 막대 그래프를 생성하고 파일로 저장합니다.
4.  **`getRequestUrl(url)` 함수**: 주어진 URL로 HTTP 요청을 보내고 응답을 반환하는 함수입니다. SSL/TLS 컨텍스트를 사용하도록 수정되었습니다.
5.  **`fetchAndSaveData(searchYearCd, siDo, guGun, numOfRows, pageNo)` 함수**:
    *   API 기본 URL과 파라미터(`ServiceKey`, `searchYearCd`, `siDo`, `guGun` 등)를 조합하여 최종 요청 URL을 만듭니다.
    *   `getRequestUrl`을 호출하여 API 응답을 받습니다.
    *   응답이 비어있거나 JSON 파싱 실패, 데이터가 없는 경우 등을 상세히 처리합니다.
    *   API 응답에서 필요한 데이터를 추출하여 `jsonResult` (딕셔너리 리스트)와 `result` (리스트의 리스트)에 저장합니다.
    *   데이터를 JSON 파일과 CSV 파일로 각각 저장합니다.
    *   수집된 데이터를 `visualizeData` 함수로 시각화합니다.
6.  **메인 실행 블록 (`if __name__ == "__main__":`)**:
    *   사용자로부터 조회할 연도와 구/군 코드를 입력받습니다.
    *   사용자가 '9999'를 입력하면 서울특별시 전체 구/군의 데이터를 반복적으로 수집하고 시각화합니다.
    *   특정 구/군 코드를 입력하면 해당 구의 데이터만 수집하고 시각화합니다.
---
### 2. 코드 상세 설명

#### 2.1. 초기 설정 및 라이브러리 임포트

```python
import urllib.request
import json
import pandas as pd
import datetime
import urllib.parse
import ssl # SSL/TLS 통신을 위해 추가됨

import matplotlib.pyplot as plt # 데이터 시각화를 위해 추가됨
import numpy as np # matplotlib과 함께 수치 계산에 사용됨
import matplotlib.font_manager as fm # 폰트 설정을 위해 추가됨
import platform # 운영체제 확인을 위해 추가됨

# 공공데이터 API 키 (주석 처리됨)
# API_KEY = "..." # 실제 API 키는 여기에 입력해야 합니다.

SEOUL_DISTRICTS = { # 서울 구/군 이름과 코드 매핑 딕셔너리
    "강남구": 680, "강동구": 740, "강북구": 305, "강서구": 500, "관악구": 620, "광진구": 215,
    "구로구": 530, "금천구": 545, "노원구": 350, "도봉구": 320, "동대문구": 230, "동작구": 590,
    "마포구": 440, "서대문구": 410, "서초구": 650, "성동구": 200, "성북구": 290, "송파구": 710,
    "양천구": 470, "영등포구": 560, "용산구": 170, "은평구": 380, "종로구": 110, "중구": 140, "중랑구": 260
}

# 최신 SSL/TLS 설정 적용
context = ssl.create_default_context()
context.set_ciphers('DEFAULT@SECLEVEL=1') # 특정 암호화 스위트만 허용하여 보안 강화
```

*   **`ssl`**: `urllib.request`를 통한 HTTPS 통신 시 보안 설정을 제어합니다. `create_default_context()`로 기본 컨텍스트를 생성하고, `set_ciphers()`로 지원되는 암호화 스위트(cipher suites)를 제한하여 보안을 강화합니다.
*   **`matplotlib.pyplot as plt`**: Python에서 그래프를 그리는 데 사용되는 표준 라이브러리입니다.
*   **`numpy as np`**: 수치 계산을 위한 라이브러리로, Matplotlib에서 데이터를 다룰 때 유용합니다.
*   **`matplotlib.font_manager as fm`**: Matplotlib의 폰트 관리를 위한 모듈입니다.
*   **`platform`**: 현재 실행 중인 운영체제를 식별하는 데 사용됩니다.
*   **`API_KEY`**: 공공데이터포털에서 발급받은 서비스 키를 입력해야 합니다. 주석 처리된 상태이므로 사용자가 직접 값을 넣어주어야 합니다.
*   **`SEOUL_DISTRICTS`**: 서울특별시의 구 이름과 해당 구의 행정 표준 코드(구/군 코드)를 매핑해놓은 딕셔너리입니다. API 호출 시 `guGun` 파라미터로 사용됩니다.
---
#### 2.2. 한글 폰트 설정

```python
def set_korean_font():
    system_platform = platform.system()
    if system_platform == "Windows":
        font_path = "C:/Windows/Fonts/malgun.ttf"  # Windows: 맑은 고딕
    elif system_platform == "Darwin":  # MacOS
        font_path = "/System/Library/Fonts/Supplemental/AppleGothic.ttf"
    else:  # Linux
        font_path = "/usr/share/fonts/truetype/nanum/NanumGothic.ttf" # Linux: 나눔고딕 (설치되어 있어야 함)

    font_prop = fm.FontProperties(fname=font_path)

    # Matplotlib 기본 폰트 설정
    plt.rcParams["font.family"] = font_prop.get_name()
    plt.rcParams["axes.unicode_minus"] = False  # 음수 기호 깨짐 방지

set_korean_font() # 한글 폰트 설정 적용
```

*   **역할**: Matplotlib으로 그래프를 그릴 때 한글이 올바르게 표시되도록 시스템별 폰트를 설정합니다.
*   **세부**:
    *   `platform.system()`: 현재 운영체제(Windows, Darwin(MacOS), Linux 등)를 식별합니다.
    *   각 운영체제에 맞는 한글 폰트 파일 경로를 지정합니다. (Linux의 경우 나눔고딕 폰트가 설치되어 있어야 합니다.)
    *   `fm.FontProperties(fname=font_path)`: 폰트 파일 경로를 사용하여 폰트 속성 객체를 생성합니다.
    *   `plt.rcParams["font.family"] = font_prop.get_name()`: Matplotlib의 기본 폰트를 설정한 폰트로 변경합니다.
    *   `plt.rcParams["axes.unicode_minus"] = False`: 그래프의 음수 기호(하이픈)가 깨지는 현상을 방지합니다.
---
#### 2.3. 구별 자전거 사고 데이터 시각화

```python
def visualizeData(result_df, guGun_name, searchYearCd):
    plt.figure(figsize=(14, 7))

    categories = ["occrrnc_cnt", "caslt_cnt", "dth_dnv_cnt", "se_dnv_cnt", "sl_dnv_cnt", "wnd_dnv_cnt"]
    x_labels = result_df["sido_sgg_nm"].unique()  # 구 이름

    values = {cat: result_df.groupby("sido_sgg_nm")[cat].sum() for cat in categories}

    x = np.arange(len(x_labels))  # X축 위치 설정
    width = 0.15  # 막대 너비

    # 여러 개의 데이터 세트를 옆으로 배치 (그룹 막대 그래프)
    for i, cat in enumerate(categories):
        plt.bar(x + i * width, values[cat], width, label=cat)

    plt.xlabel("구 이름", fontsize=14)
    plt.ylabel("건수", fontsize=14)
    plt.title(f"{searchYearCd}년 서울특별시 {guGun_name} 자전거 사고 통계", fontsize=16)
    plt.xticks(x + width, x_labels, rotation=45, fontsize=12)
    plt.yticks(fontsize=12)
    plt.legend(fontsize=12)

    plt.savefig(f"bicycle_accidents_chart_{searchYearCd}_서울특별시_{guGun_name}.png", dpi=300) # 그래프 이미지 저장
    plt.show() # 그래프 화면에 표시
    print(f"✅ 그래프 저장 완료: bicycle_accidents_chart_{searchYearCd}_서울특별시_{guGun_name}.png")
```

*   **역할**: `pandas.DataFrame`으로 정리된 자전거 사고 데이터를 입력받아 그룹 막대 그래프를 생성하고 이미지 파일로 저장합니다.
*   **세부**:
    *   `plt.figure(figsize=(14, 7))`: 그래프의 크기를 설정합니다.
    *   `categories`: 그래프로 표현할 사고 통계 항목들 (발생건수, 사상자수, 사망자수 등).
    *   `x_labels`: X축에 표시될 구 이름들을 `result_df["sido_sgg_nm"].unique()`를 통해 가져옵니다.
    *   `values`: 각 카테고리별로 구별 합계를 계산합니다 (`groupby("sido_sgg_nm")[cat].sum()`).
    *   `np.arange(len(x_labels))`: X축 막대들의 기본 위치를 생성합니다.
    *   `width`: 각 막대의 너비를 설정합니다.
    *   `for i, cat in enumerate(categories): plt.bar(x + i * width, values[cat], width, label=cat)`: 여러 카테고리의 막대를 X축의 각 레이블(구 이름) 옆에 나란히 배치하는 그룹 막대 그래프를 생성합니다. `x + i * width`를 통해 각 카테고리의 막대 위치를 조금씩 이동시킵니다.
    *   `plt.xlabel`, `plt.ylabel`, `plt.title`, `plt.xticks`, `plt.yticks`, `plt.legend`: 그래프의 레이블, 제목, 눈금, 범례 등을 설정하여 가독성을 높입니다.
    *   `plt.savefig(...)`: 생성된 그래프를 PNG 이미지 파일로 저장합니다. `dpi=300`은 해상도를 높입니다.
    *   `plt.show()`: 그래프를 화면에 표시합니다.
---
#### 2.4. `getRequestUrl(url)` 함수

```python
def getRequestUrl(url):
    req = urllib.request.Request(url)
    try:
        response = urllib.request.urlopen(req, context=context) # context=context 추가
        if response.getcode() == 200:
            print("[%s] URL 요청 성공" % datetime.datetime.now())
            return response.read().decode("utf-8")
    except Exception as e:
        print("[%s] 오류 발생: %s" % (datetime.datetime.now(), str(e)))
        return None
```

*   이전 `getRequestUrl` 함수와 거의 동일하지만, `urllib.request.urlopen()` 호출 시 `context=context` 매개변수를 추가하여 위에서 설정한 SSL/TLS 컨텍스트를 사용하도록 했습니다. 이는 보안 통신을 강화하는 목적입니다.
---
#### 2.5. `fetchAndSaveData(searchYearCd, siDo, guGun, numOfRows, pageNo)` 함수

```python
def fetchAndSaveData(searchYearCd, siDo, guGun, numOfRows=100, pageNo=1):
    base_url = "https://apis.data.go.kr/B552061/frequentzoneBicycle/getRestFrequentzoneBicycle"

    params = {
        "ServiceKey": API_KEY, # API 키 사용
        "searchYearCd": searchYearCd, # 조회 연도
        "siDo": siDo, # 시도 코드 (11: 서울)
        "guGun": guGun, # 구군 코드
        "type": "json", # JSON 응답 타입 요청
        "numOfRows": str(numOfRows), # 한 페이지 결과 수
        "pageNo": str(pageNo), # 페이지 번호
    }

    query_string = urllib.parse.urlencode(params, doseq=True) # 파라미터를 URL 쿼리 문자열로 인코딩
    url = f"{base_url}?{query_string}" # 최종 API URL 구성

    responseDecode = getRequestUrl(url)

    # 상세한 오류 처리
    if responseDecode is None or responseDecode.strip() == "":
        print(f"🚨 [오류] API 응답이 비어 있습니다. 파일을 생성하지 않습니다. (구/군 코드: {guGun})")
        return

    try:
        jsonData = json.loads(responseDecode)
    except json.JSONDecodeError as e:
        print(f"🚨 [오류] JSON 변환 실패. 파일을 생성하지 않습니다. (구/군 코드: {guGun}): {e}")
        print(f"응답 내용: {responseDecode[:200]} ...")
        return

    # API 응답 구조 확인 및 데이터 추출
    # 이 API는 items 아래에 item이 직접 리스트로 있을 수도, 단일 딕셔너리로 있을 수도 있습니다.
    # 안전하게 리스트로 처리하도록 합니다.
    dataList = []
    if "response" in jsonData and "body" in jsonData["response"] and "items" in jsonData["response"]["body"]:
        items_data = jsonData["response"]["body"]["items"]["item"]
        if isinstance(items_data, list):
            dataList = items_data
        elif isinstance(items_data, dict):
            dataList = [items_data] # 단일 항목도 리스트로 처리
    
    if not dataList:
        print(f"🚨 [오류] 응답 데이터가 없습니다. 파일을 생성하지 않습니다. (구/군 코드: {guGun})")
        return

    jsonResult = []
    result = []

    for item in dataList: # 각 데이터 항목 순회
        jsonResult.append(item)
        result.append([ # CSV 저장을 위한 리스트 추가
            item.get("afos_fid", ""), item.get("afos_id", ""), item.get("bjd_cd", ""),
            item.get("spot_cd", ""), item.get("sido_sgg_nm", ""), item.get("spot_nm", ""),
            item.get("occrrnc_cnt", ""), item.get("caslt_cnt", ""), item.get("dth_dnv_cnt", ""),
            item.get("se_dnv_cnt", ""), item.get("sl_dnv_cnt", ""), item.get("wnd_dnv_cnt", ""),
            item.get("lo_crd", ""), item.get("la_crd", ""),
        ])

    # 파일 이름 설정 (구/군 코드 -> 이름 변환)
    guGun_name = "전체" if guGun == "9999" else [k for k, v in SEOUL_DISTRICTS.items() if v == int(guGun)][0]
    json_filename = f"bicycle_accidents_{searchYearCd}_서울특별시_{guGun_name}.json"
    csv_filename = f"bicycle_accidents_{searchYearCd}_서울특별시_{guGun_name}.csv"

    # JSON 파일 저장
    with open(json_filename, "w", encoding="utf-8") as outfile:
        json.dump(jsonResult, outfile, indent=4, ensure_ascii=False)
    print(f"✅ JSON 데이터 저장 완료: {json_filename}")

    # CSV 파일 저장
    columns = ["afos_fid", "afos_id", "bjd_cd", "spot_cd", "sido_sgg_nm", "spot_nm",
               "occrrnc_cnt", "caslt_cnt", "dth_dnv_cnt", "se_dnv_cnt", "sl_dnv_cnt", "wnd_dnv_cnt",
               "lo_crd", "la_crd"]
    result_df = pd.DataFrame(result, columns=columns) # Pandas DataFrame 생성
    result_df.to_csv(csv_filename, index=False, encoding="utf-8-sig") # CSV 저장
    print(f"✅ CSV 데이터 저장 완료: {csv_filename}")

    visualizeData(result_df, guGun_name, searchYearCd) # 시각화 함수 호출
```

*   **역할**: 공공데이터포털 "자전거 교통사고 잦은 곳 정보" API를 호출하여 데이터를 가져오고, 파일로 저장하며, 시각화까지 담당하는 핵심 함수입니다.
*   **세부**:
    *   `base_url`: API의 기본 엔드포인트 URL.
    *   `params`: API 요청에 필요한 파라미터 딕셔너리. `ServiceKey`가 중요합니다. `siDo`는 서울(`11`)로 고정, `guGun`은 동적으로 받습니다.
    *   `urllib.parse.urlencode(params, doseq=True)`: 파라미터 딕셔너리를 URL 쿼리 문자열로 변환합니다. `doseq=True`는 리스트 형태의 값도 올바르게 인코딩할 수 있도록 합니다.
    *   **상세한 오류 처리**: `responseDecode`가 `None`이거나 비어있는 경우, `json.JSONDecodeError` 발생 시, API 응답에 `items` 또는 `item` 데이터가 없는 경우를 각각 세심하게 처리하여 프로그램의 안정성을 높였습니다.
    *   **데이터 추출**: API 응답의 JSON 구조가 `response -> body -> items -> item` 형태이므로, 이를 따라 데이터를 추출합니다. 이때 `item`이 단일 딕셔너리일 수도 있고 딕셔너리 리스트일 수도 있는 경우를 `isinstance`로 확인하여 항상 `dataList`를 리스트로 만듭니다.
    *   `item.get("키", "")`: 딕셔너리의 `get()` 메서드를 사용하여 키가 없을 때 오류 대신 기본값(`""`)을 반환하도록 하여 코드를 더 견고하게 만듭니다.
    *   **파일명 생성**: `guGun_name`을 `SEOUL_DISTRICTS` 딕셔너리를 역참조하여 구/군 코드를 실제 이름으로 변환하여 파일명에 사용합니다.
    *   **JSON/CSV 저장**: 이전 코드와 동일하게 `json.dump()`와 `pd.DataFrame.to_csv()`를 사용하여 데이터를 저장합니다.
    *   `visualizeData()`: 저장된 `result_df`를 사용하여 시각화 함수를 호출합니다.
---
#### 2.6. 메인 실행 블록

```python
if __name__ == "__main__":
    searchYearCd = input("조회할 연도를 입력하세요 (예: 2023): ").strip()
    siDo = "11"  # 서울특별시 고정

    print("\n서울특별시 내 구/군 선택 (9999 입력 시 전체 선택)")
    for district, code in SEOUL_DISTRICTS.items():
        print(f"{district}: {code}", end=" | ")

    guGun = input("\n구/군 코드를 입력하세요 (9999 입력 시 전체 선택): ").strip()

    if guGun == "9999":
        # 전체 구 데이터 요청
        for gu in SEOUL_DISTRICTS.values(): # SEOUL_DISTRICTS 딕셔너리의 모든 구 코드를 순회
            fetchAndSaveData(searchYearCd, siDo, str(gu)) # 각 구에 대해 데이터 수집/저장/시각화
    else:
        if guGun not in map(str, SEOUL_DISTRICTS.values()): # 입력된 구 코드가 유효한지 검증
            print("잘못된 구/군 코드입니다. 프로그램을 종료합니다.")
        else:
            fetchAndSaveData(searchYearCd, siDo, guGun) # 특정 구에 대해 데이터 수집/저장/시각화
```

*   **사용자 입력**: 조회 연도와 구/군 코드를 입력받습니다.
*   `siDo = "11"`: 시도 코드를 서울특별시로 고정합니다.
*   **전체 구/군 데이터 수집**: 사용자가 `9999`를 입력하면, `SEOUL_DISTRICTS.values()`를 반복하여 서울의 모든 구 코드를 `fetchAndSaveData` 함수에 전달하여 각각 데이터를 수집하고 시각화합니다.
*   **특정 구/군 데이터 수집**: 특정 구/군 코드를 입력하면, 해당 코드가 유효한지 검증한 후 `fetchAndSaveData`를 호출합니다.
---
### 3. 주요 특징 및 발전 사항

*   **종합적인 데이터 분석 파이프라인**: 데이터 수집, 저장, 그리고 시각화까지 한 번에 처리하는 완전한 파이프라인을 구축했습니다.
*   **보안 및 안정성 강화**: SSL/TLS 설정과 상세한 오류 처리를 통해 프로그램의 신뢰성을 높였습니다.
*   **사용자 친화적인 인터페이스**: 사용자로부터 검색 조건을 입력받고, 구/군 선택 가이드를 제공하여 사용 편의성을 높였습니다.
*   **데이터 시각화**: Matplotlib을 활용하여 수집된 데이터를 그래프 형태로 제공함으로써, 데이터를 더욱 쉽게 이해하고 분석할 수 있도록 합니다. 특히 한글 폰트 설정은 시각화의 실용성을 크게 높였습니다.
*   **Pandas 활용**: `result` 리스트를 `pd.DataFrame`으로 변환하여 CSV 저장 및 시각화 데이터 준비 과정이 효율적이고 체계적입니다.
*   **구/군별 반복 수집**: 서울시 전체 구/군에 대한 데이터를 자동으로 수집하고 처리할 수 있어 대규모 데이터 수집에 용이합니다.

이 코드는 API를 활용한 데이터 수집 및 분석 프로젝트의 좋은 예시이며, 실제 현업에서 데이터를 다루는 과정과 유사한 흐름을 보여줍니다. 아주 체계적이고 잘 작성된 코드입니다!