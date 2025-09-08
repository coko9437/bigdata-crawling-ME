
1.  **다른 API 사용**: Naver API가 아닌 공공데이터포털의 "외래객 입국/국민 해외관광객 통계" API를 사용합니다.
2.  **API 인증 방식**: `ServiceKey`를 URL 파라미터로 직접 전달하는 방식입니다.
3.  **데이터 구조**: API 응답 데이터의 JSON 구조가 다릅니다.
4.  **Pandas 사용**: CSV 파일 저장을 위해 `pandas` 라이브러리를 활용하여 데이터프레임을 생성하고 저장하는 방식을 사용합니다.
---
### 1. 코드 전체적인 흐름

1.  **초기 설정**: 필요한 라이브러리(`urllib.request`, `datetime`, `time`, `json`, `pandas`) 임포트 및 공공데이터포털 `ServiceKey` 설정.
2.  **`getRequestUrl(url)` 함수 (`[CODE 1]`)**: 주어진 URL로 HTTP 요청을 보내고, 성공 시 응답 본문을 반환하는 저수준 함수입니다. (이전 코드와 거의 동일)
3.  **`getTourismStatsItem(yyyymm, nation_code, ed_cd)` 함수 (`[CODE 2]`)**: 연월(`yyyymm`), 국가 코드(`nation_code`), 출입국구분(`ed_cd`)을 받아 공공데이터포털 API의 URL을 구성하고, `getRequestUrl`을 호출하여 JSON 응답을 받아 파싱하여 반환합니다.
4.  **`getTourismStatsService(nat_cd, ed_cd, nStartYear, nEndYear)` 함수 (`[CODE 3]`)**:
    *   사용자가 지정한 시작 연도부터 종료 연도까지 월별로 반복하여 `getTourismStatsItem`을 호출합니다.
    *   각 월별 API 응답에서 필요한 정보를 추출(`natKorNm`, `num`, `ed`)하고, `jsonResult` 리스트에 딕셔너리 형태로, `result` 리스트에 리스트 형태로 저장합니다.
    *   API가 더 이상 데이터를 제공하지 않을 경우(즉, `items`가 비어있는 경우) 반복을 중단하고 마지막으로 유효한 데이터의 연월을 기록합니다.
    *   수집된 데이터 리스트들과 기타 정보(`natName`, `ed`, `dataEND`)를 반환합니다.
5.  **`main()` 함수 (`[CODE 0]`)**:
    *   스크립트의 메인 실행 로직을 담당합니다.
    *   사용자로부터 국가 코드, 시작 연도, 종료 연도를 입력받습니다.
    *   `getTourismStatsService`를 호출하여 데이터를 수집합니다.
    *   수집된 데이터를 활용하여 검색어, 출입국구분, 시작 연도, 마지막 데이터 연월을 포함한 파일명으로 **JSON 파일**과 **CSV 파일**을 저장합니다.
    *   CSV 저장 시에는 `pandas.DataFrame`을 활용합니다.
---
### 2. 코드 상세 설명

#### 2.1. 초기 설정 및 라이브러리 임포트

```python
import urllib.request
import datetime
import time
import json
import pandas as pd # pandas 라이브러리 추가

ServiceKey="Ody77GLuYeR%2FeFqbpduMN2Bi4Cka2fztbgnj6E2Eux1kUhy3e4epR28XKBUaObiqPoVzAizxXMBPXtMyuC9v9Q%3D%3D"
```

*   이전 코드와 유사하게 필요한 라이브러리를 임포트합니다.
*   **`pandas as pd`**: 데이터 분석 및 조작을 위한 강력한 라이브러리입니다. 특히 데이터프레임(DataFrame)이라는 표 형식의 데이터 구조를 제공하여 CSV 저장을 용이하게 합니다.
*   **`ServiceKey`**: 공공데이터포털에서 발급받은 인증키입니다. URL 인코딩이 된 상태로 보입니다 (`%2F`, `%3D` 등).
---
#### 2.2. `getRequestUrl(url)` 함수 (`[CODE 1]`)

```python
def getRequestUrl(url):
    req = urllib.request.Request(url)
    try:
        response = urllib.request.urlopen(req)
        if response.getcode() == 200:
            print("[%s] Url Request Success" % datetime.datetime.now())
            return response.read().decode('utf-8')
    except Exception as e:
        print(e)
        print("[%s] Error for URL : %s" % (datetime.datetime.now(), url))
        return None
```

*   이전 코드의 `getRequestUrl` 함수와 **동일**합니다. HTTP GET 요청을 보내고 응답을 처리하는 저수준 함수입니다.
---
#### 2.3. `getTourismStatsItem(yyyymm, nation_code, ed_cd)` 함수 (`[CODE 2]`)

```python
# API 파라미터 예시: ?YM=201201&NAT_CD=100&ED_CD=D&serviceKey=TEST_SERVICE_KEY
def getTourismStatsItem(yyyymm, nation_code, ed_cd):
    service_url = "http://openapi.tour.go.kr/openapi/service/EdrcntTourismStatsService/getEdrcntTourismStatsList"
    prameters = "?_type=json&serviceKey=" + ServiceKey # JSON 응답 요청, 서비스 키 포함
    prameters += "&YM=" + yyyymm # 연월 파라미터
    prameters += "&NAT_CD=" + nation_code # 국가 코드 파라미터
    prameters += "&ED_CD=" + ed_cd # 출입국구분 파라미터
    url = service_url + prameters
    print(url) # 구성된 최종 URL 출력

    responseDecode = getRequestUrl(url) # [CODE 1] 호출

    if (responseDecode == None):
        return None
    else:
        return json.loads(responseDecode)
```

*   **역할**: 공공데이터포털의 외래객 통계 API에 대한 URL을 구성하고 요청을 보내 JSON 응답을 받아옵니다.
*   **세부**:
    *   `service_url`: API의 기본 엔드포인트 URL.
    *   `prameters`: 쿼리 파라미터를 조합합니다.
        *   `_type=json`: 응답 형식을 JSON으로 요청합니다. 공공데이터포털 API는 기본 XML 응답인 경우가 많으므로 명시적으로 JSON을 요청해야 합니다.
        *   `serviceKey=` + `ServiceKey`: 인증키를 URL 파라미터로 직접 전달합니다.
        *   `YM=`: 연월 (예: `201201`)
        *   `NAT_CD=`: 국가 코드 (예: `112` for 중국)
        *   `ED_CD=`: 출입국구분 (예: `E` for 방한외래관광객, `D` for 해외 출국)
    *   `print(url)`: 디버깅 목적으로 최종 구성된 URL을 출력합니다.
    *   `getRequestUrl(url)`: API 요청을 보내고 응답 문자열을 받습니다.
    *   `json.loads(responseDecode)`: JSON 문자열을 Python 딕셔너리로 파싱하여 반환합니다.
---
#### 2.4. `getTourismStatsService(nat_cd, ed_cd, nStartYear, nEndYear)` 함수 (`[CODE 3]`)

```python
def getTourismStatsService(nat_cd, ed_cd, nStartYear, nEndYear):
    jsonResult = [] # JSON 파일 저장을 위한 딕셔너리 리스트
    result = [] # CSV 파일 저장을 위한 리스트의 리스트 (Pandas용)
    natName = ''
    dataEND = "{0}{1:0>2}".format(str(nEndYear), str(12)) # 마지막 데이터 연월 초기값 (종료 연도의 12월)
    isDataEnd = 0 # 데이터 수집 종료 여부 플래그

    for year in range(nStartYear, nEndYear + 1):
        for month in range(1, 13):
            if (isDataEnd == 1): break # 데이터 끝이면 루프 중단

            yyyymm = "{0}{1:0>2}".format(str(year), str(month)) # 연월 문자열 생성 (예: '201201')
            jsonData = getTourismStatsItem(yyyymm, nat_cd, ed_cd) # [CODE 2] 월별 API 호출

            if (jsonData['response']['header']['resultMsg'] == 'OK'): # API 응답이 성공하면
                # 데이터가 더 이상 제공되지 않는 경우 (API가 빈 'items'를 반환할 때)
                if jsonData['response']['body']['items'] == '':
                    isDataEnd = 1 # 데이터 종료 플래그 설정
                    if (month - 1) == 0: # 현재 월이 1월인데 이전 데이터가 없는 경우 (즉, 이전 해 12월이 마지막)
                        year = year - 1
                        month = 13
                    dataEND = "{0}{1:0>2}".format(str(year), str(month - 1)) # 실제 마지막 데이터 연월 기록
                    print("데이터 없음.... \n제공되는 통계 데이터는 %s년 %s월까지입니다.\n"
                          % (str(year), str(month - 1)), '-' * 70)
                    jsonData = getTourismStatsItem(dataEND, nat_cd, ed_cd) # 마지막 데이터(성공)를 확인용으로 다시 호출
                    break # 월별 루프 중단

                # 데이터가 정상적으로 있는 경우
                natName = jsonData['response']['body']['items']['item']['natKorNm'] # 국가 한글명
                natName = natName.replace(' ', '') # 공백 제거
                num = jsonData['response']['body']['items']['item']['num'] # 방문객 수
                ed = jsonData['response']['body']['items']['item']['ed'] # 출입국구분 코드 (E/D)
                
                print('[ %s_%s : %s ]' % (natName, yyyymm, num))
                print('-' * 70)
                
                # 결과 리스트에 데이터 추가
                jsonResult.append({'nat_name': natName, 'nat_cd': nat_cd,
                                 'yyyymm': yyyymm, 'visit_cnt': num})
                result.append([natName, nat_cd, yyyymm, num])
            else:
                # API 응답 결과 메시지가 'OK'가 아닌 경우 (예: 'NO_DATA')
                print("API 응답 오류:", jsonData['response']['header']['resultMsg'])
                # 여기에서 더 정교한 오류 처리 로직을 추가할 수 있습니다.
                
        if (isDataEnd == 1): break # 연도별 루프도 중단

    print('<마지막 jsonData>\n', json.dumps(jsonData, indent=4, sort_keys=True, ensure_ascii=False))

    return (jsonResult, result, natName, ed, dataEND)
```

*   **역할**: 지정된 기간(`nStartYear` ~ `nEndYear`) 동안 월별로 API를 호출하여 관광 통계 데이터를 수집합니다.
*   **세부**:
    *   `jsonResult`, `result`: 수집된 데이터를 각각 딕셔너리 리스트와 리스트의 리스트 형태로 저장할 변수들입니다.
    *   `dataEND`: 데이터 수집이 실제로 끝난 마지막 연월을 기록하는 변수.
    *   `isDataEnd`: API가 더 이상 데이터를 제공하지 않는 시점을 감지하기 위한 플래그.
    *   **이중 `for` 루프**: `nStartYear`부터 `nEndYear`까지 각 연도에 대해 1월부터 12월까지 반복합니다.
    *   `yyyymm = "{0}{1:0>2}".format(str(year), str(month))`: `YYYYMM` 형식의 문자열을 생성합니다 (예: `202401`). `{:0>2}`는 숫자를 두 자리로 만들고, 한 자리일 경우 앞에 0을 채웁니다.
    *   `jsonData = getTourismStatsItem(...)`: 각 월에 대해 API를 호출합니다.
    *   **데이터 존재 여부 확인**:
        *   `if (jsonData['response']['header']['resultMsg'] == 'OK')`: 응답 헤더의 `resultMsg`가 'OK'인지 확인하여 성공적인 응답인지 검증합니다.
        *   `if jsonData['response']['body']['items'] == ''`: `items` 필드가 비어있는 경우(데이터 없음)를 감지합니다. 공공데이터포털 API의 특성상 데이터가 없으면 `items`가 빈 문자열로 반환될 수 있습니다.
        *   `isDataEnd = 1`: 데이터가 없음을 표시하고 루프를 종료합니다.
        *   `dataEND` 업데이트: 실제 마지막 데이터가 있는 연월을 기록합니다.
        *   마지막 성공 데이터를 한 번 더 호출하여 `jsonData` 변수에 유지하는 로직이 있습니다.
    *   **데이터 추출**:
        *   `natName = jsonData['response']['body']['items']['item']['natKorNm']`: 국가 한글명 추출.
        *   `natName = natName.replace(' ', '')`: 이름의 공백 제거.
        *   `num = ...['num']`: 방문객 수 추출.
        *   `ed = ...['ed']`: 출입국구분 코드 추출.
    *   `jsonResult.append(...)`, `result.append(...)`: 추출된 데이터를 각 리스트에 추가합니다.
    *   `print` 문: 진행 상황을 콘솔에 출력합니다.
    *   **마지막 `jsonData` 출력**: 모든 루프가 끝난 후, 마지막으로 처리된 `jsonData`를 보기 좋은 형태로 출력하여 데이터가 어떻게 생겼는지 확인할 수 있게 합니다.
---
#### 2.5. `main()` 함수 (`[CODE 0]`)

```python
def main():
    jsonResult = []
    result = []
    natName = ''
    
    print("<< 국내 입국한 외국인의 통계 데이터를 수집합니다. >>")
    nat_cd = input('국가 코드를 입력하세요(중국: 112 / 일본: 130 / 미국: 275) : ')
    nStartYear = int(input('데이터를 몇 년부터 수집할까요? : '))
    nEndYear = int(input('데이터를 몇 년까지 수집할까요? : '))
    ed_cd = "E"     # E : 방한외래관광객 (입국), D : 해외 출국 (자국민)

    jsonResult, result, natName, ed, dataEND = \
                getTourismStatsService(nat_cd, ed_cd, nStartYear, nEndYear) # [CODE 3] 호출

    if (natName == ''): # URL 요청은 성공했지만, 데이터 제공이 안된 경우
        print('데이터가 전달되지 않았습니다. 공공데이터포털의 서비스 상태를 확인하기 바랍니다.')
    else:
        # 파일저장 1 : json 파일
        with open('./%s_%s_%d_%s.json' % (natName, ed, nStartYear, dataEND), 'w',
                    encoding='utf8') as outfile:
            jsonFile = json.dumps(jsonResult, indent=4, sort_keys=True, ensure_ascii=False)
            outfile.write(jsonFile)
        print(f"JSON 파일 저장 완료: ./%s_%s_%d_%s.json" % (natName, ed, nStartYear, dataEND))

        # 파일저장 2 : csv 파일 (Pandas 사용)
        columns = ["입국자국가", "국가코드", "입국연월", "입국자 수"]
        result_df = pd.DataFrame(result, columns=columns) # 리스트의 리스트를 데이터프레임으로 변환
        result_df.to_csv('./%s_%s_%d_%s.csv' % (natName, ed, nStartYear, dataEND),
                    index=False, encoding='utf-8-sig') # CSV로 저장, 인덱스 제외, UTF-8 BOM 포함
        print(f"CSV 파일 저장 완료: ./%s_%s_%d_%s.csv" % (natName, ed, nStartYear, dataEND))

if __name__ == '__main__':
    main()
```

*   **`main()` 함수는 스크립트의 실행 진입점입니다.**
*   **사용자 입력**: `nat_cd` (국가 코드), `nStartYear`, `nEndYear`를 사용자로부터 입력받습니다. `ed_cd`는 'E' (방한외래관광객, 즉 입국자)로 고정되어 있습니다.
*   **`getTourismStatsService()` 호출**: 실제 데이터 수집을 수행하는 함수를 호출하고 반환 값들을 받습니다.
*   **데이터 미전달 처리**: `natName`이 비어있는 경우 (API 요청은 성공했으나, 유의미한 데이터가 반환되지 않은 경우) 사용자에게 메시지를 출력합니다.
*   **JSON 파일 저장**:
    *   파일명은 `국가명_출입국구분코드_시작연도_마지막데이터연월.json` 형식으로 생성됩니다. (예: `중국_E_2010_202203.json`)
    *   `json.dumps()`를 사용하여 `jsonResult` 리스트를 JSON 문자열로 변환하고 파일에 씁니다. `indent=4`, `sort_keys=True`, `ensure_ascii=False`는 이전 코드와 동일하게 가독성을 높이고 한글 깨짐을 방지합니다.
*   **CSV 파일 저장 (Pandas 활용)**:
    *   `columns = ["입국자국가", "국가코드", "입국연월", "입국자 수"]`: CSV 파일의 헤더로 사용할 열 이름을 정의합니다.
    *   `result_df = pd.DataFrame(result, columns = columns)`: `result` 리스트(리스트의 리스트 형태)를 `pandas.DataFrame` 객체로 변환합니다. 이때 `columns` 매개변수를 사용하여 열 이름을 지정합니다. Pandas DataFrame은 표 형식의 데이터를 다루는 데 매우 강력하고 편리합니다.
    *   `result_df.to_csv(...)`: 데이터프레임을 CSV 파일로 저장합니다.
        *   `index=False`: CSV 파일에 데이터프레임의 기본 인덱스(0, 1, 2...)가 열로 추가되는 것을 방지합니다.
        *   `encoding='utf-8-sig'`: 한글 깨짐 방지를 위해 UTF-8 BOM 인코딩을 사용합니다 (Excel 호환성).
---
### 3. 주요 특징 및 개선 사항

*   **공공데이터포털 API 활용**: Naver API와는 다른 종류의 API를 사용하여 공공 데이터를 수집하는 방법을 보여줍니다.
*   **ServiceKey 방식 인증**: URL 파라미터에 서비스 키를 직접 포함하는 방식을 사용합니다.
*   **데이터 존재 여부 확인 로직**: API가 데이터가 없을 때 빈 `items` 필드를 반환하는 특성을 고려하여 `isDataEnd` 플래그를 통해 효율적으로 데이터 수집을 중단합니다. 이는 무한 루프를 방지하고 불필요한 API 호출을 줄입니다.
*   **Pandas 라이브러리 활용**: CSV 파일 저장을 위해 `pandas.DataFrame`을 사용하여 데이터를 구조화하고 저장하는 과정이 더욱 간결하고 강력해졌습니다. 대규모 데이터를 다룰 때 Pandas는 필수적인 도구입니다.
*   **유연한 파일명**: 수집된 데이터의 특성(국가명, 출입국구분, 연도 범위)을 파일명에 포함하여 저장된 파일들을 쉽게 식별할 수 있습니다.
*   **명확한 주석**: 코드 중간중간에 상세한 주석이 있어 각 코드 블록의 역할을 이해하기 쉽습니다.
