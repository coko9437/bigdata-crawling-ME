
### 1. 코드 전체적인 흐름

1.  **초기 설정**: 필요한 라이브러리 임포트 및 Naver API 인증 정보 (`client_id`, `client_secret`) 설정.
2.  **`getRequestUrl(url)` 함수 (`[CODE 1]`)**: 주어진 URL로 Naver API에 HTTP 요청을 보내고, 성공 시 응답 본문을 반환하는 저수준 함수입니다. 오류 처리도 포함되어 있습니다.
3.  **`getNaverSearch(node, srcText, start, display)` 함수 (`[CODE 2]`)**: 검색 대상(`node`), 검색어(`srcText`), 시작 위치(`start`), 결과 개수(`display`)를 받아 Naver 검색 API의 URL을 구성하고, `getRequestUrl`을 호출하여 JSON 응답을 받아 파싱하여 반환합니다.
4.  **`getPostData(post, jsonResult, cnt)` / `getPostData_Blog(post, jsonResult, cnt)` 함수 (`[CODE 3]`)**: Naver API로부터 받은 개별 뉴스 또는 블로그 게시물 데이터를 파싱하여 필요한 정보를 추출하고, 날짜 형식을 변환한 뒤 최종 결과 리스트(`jsonResult`)에 추가하는 역할을 합니다. 여기서는 뉴스 검색을 위해 `getPostData`가 사용됩니다.
5.  **`main()` 함수 (`[CODE 0]`)**:
    *   스크립트의 메인 실행 로직을 담당합니다.
    *   사용자로부터 검색어를 입력받습니다.
    *   `getNaverSearch`를 호출하여 첫 페이지의 데이터를 가져오고 전체 검색 결과 수(`total`)를 확인합니다.
    *   `while` 루프를 사용하여 `total` 건수 또는 Naver API가 허용하는 최대 1000건까지 데이터를 반복적으로 가져옵니다. (한 번에 100개씩)
    *   각 검색 결과(`post`)를 `getPostData` 함수로 처리하여 `jsonResult` 리스트에 누적합니다.
    *   모든 데이터를 수집한 후, `jsonResult` 리스트의 내용을 **CSV 파일**과 **JSON 파일** 두 가지 형식으로 저장합니다.
6.  **`if __name__ == '__main__':`**: 스크립트가 직접 실행될 때 `main()` 함수를 호출하도록 합니다.
---
### 2. 코드 상세 설명

#### 2.1. 초기 설정 및 라이브러리 임포트

```python
import os
import sys
import urllib.request
import datetime # 날짜/시간 처리를 위해 추가됨
import time # 여기서는 직접 사용되지 않지만, API 호출 간 딜레이를 줄 때 유용
import json
import csv

client_id = 'Kc65Ibtu71R5TLRUF7lL'
client_secret = 'CdT4sLf9Ht'
```

*   이전 코드와 동일하게 필요한 라이브러리를 임포트하고, `client_id`, `client_secret`을 설정합니다. `datetime` 모듈이 날짜 형식 변환을 위해 추가되었습니다.

#### 2.2. `getRequestUrl(url)` 함수 (`[CODE 1]`)

```python
def getRequestUrl(url):
    req = urllib.request.Request(url)
    req.add_header("X-Naver-Client-Id", client_id)
    req.add_header("X-Naver-Client-Secret", client_secret)

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

*   **역할**: 특정 URL에 대한 HTTP GET 요청을 보내고, 성공 시 응답 데이터를 UTF-8 문자열로 반환합니다.
*   **세부**:
    *   `urllib.request.Request(url)`: 요청 객체 생성.
    *   `req.add_header(...)`: 인증 헤더 추가.
    *   `try-except` 블록: 네트워크 오류, HTTP 오류 등을 처리하기 위한 예외 처리. `Exception as e`는 모든 종류의 예외를 잡아냅니다.
    *   `datetime.datetime.now()`: 현재 시간을 출력하여 요청 성공/실패 시점을 기록합니다.
    *   `response.read().decode('utf-8')`: 응답 본문을 읽어와 UTF-8로 디코딩하여 반환합니다.
---
#### 2.3. `getNaverSearch(node, srcText, start, display)` 함수 (`[CODE 2]`)

```python
def getNaverSearch(node, srcText, start, display):
    base = "https://openapi.naver.com/v1/search"
    node = "/%s.json" % node # node 예: news, blog
    parameters = "?query=%s&start=%s&display=%s" % (urllib.parse.quote(srcText), start, display)
    url = base + node + parameters

    responseDecode = getRequestUrl(url) # [CODE 1] 호출

    if (responseDecode == None):
        return None
    else:
        return json.loads(responseDecode)
```

*   **역할**: Naver 검색 API의 특정 `node`(예: `news`, `blog`)에 대해 검색을 수행하고, JSON 형식의 응답을 Python 딕셔너리로 파싱하여 반환합니다.
*   **세부**:
    *   `base`, `node`, `parameters`를 조합하여 최종 API URL을 구성합니다.
        *   `query=%s`: 검색어 (`srcText`를 URL 인코딩하여 삽입).
        *   `start=%s`: 검색 시작 위치 (1부터 시작).
        *   `display=%s`: 한 번에 가져올 결과 수 (최대 100).
    *   `getRequestUrl(url)`: 실제로 HTTP 요청을 보내고 응답 문자열을 받아옵니다.
    *   `json.loads(responseDecode)`: 받은 JSON 문자열을 Python 딕셔너리로 변환하여 반환합니다.
---
#### 2.4. `getPostData(post, jsonResult, cnt)` 함수 (`[CODE 3]`)

```python
def getPostData(post, jsonResult, cnt): # 뉴스 검색 결과를 위한 함수
    title = post['title'].replace('<b>', '').replace('</b>', '').replace('&quot;', '"')
    description = post['description'].replace('<b>', '').replace('</b>', '').replace('&quot;', '"')
    org_link = post['originallink']
    link = post['link']

    # 날짜 형식 변환 (API 기본 형식 -> 'YYYY-MM-DD HH:MM:SS')
    pDate = datetime.datetime.strptime(post['pubDate'], '%a, %d %b %Y %H:%M:%S +0900')
    pDate = pDate.strftime('%Y-%m-%d %H:%M:%S')

    jsonResult.append({'cnt':cnt, 'title':title, 'description': description,
                       'org_link':org_link, 'link': link, 'pDate':pDate})
    return

def getPostData_Blog(post, jsonResult, cnt): # 블로그 검색 결과를 위한 함수 (주석 처리됨)
    title = post['title'].replace('<b>', '').replace('</b>', '').replace('&quot;', '"')
    description = post['description'].replace('<b>', '').replace('</b>', '').replace('&quot;', '"')
    org_link = post['bloggerlink']
    link = post['link']

    # 날짜 형식 변환 ('YYYYMMDD' -> 'YYYY-MM-DD HH:MM:SS')
    pDate = datetime.datetime.strptime(post['postdate'], '%Y%m%d')
    pDate = pDate.strftime('%Y-%m-%d 00:00:00') # 시간 정보가 없으므로 00:00:00으로 설정

    jsonResult.append({'cnt':cnt, 'title':title, 'description': description,
                       'org_link':org_link, 'link': link, 'pDate':pDate})
    return
```

*   **역할**: 개별 검색 결과 아이템(`post`)에서 필요한 정보를 추출하고 가공하여 `jsonResult` 리스트에 딕셔너리 형태로 추가합니다.
*   **세부**:
    *   **HTML 태그 제거**: `title`과 `description` 필드에서 `<b>`, `</b>`, `&quot;` 등의 HTML 태그 및 엔티티를 제거하여 순수 텍스트를 얻습니다.
    *   **날짜 형식 변환**:
        *   뉴스 API의 `pubDate`는 `'%a, %d %b %Y %H:%M:%S +0900'` (예: `Thu, 29 Feb 2024 09:00:00 +0900`) 형식입니다. `datetime.datetime.strptime()`을 사용하여 이 문자열을 `datetime` 객체로 변환합니다.
        *   `strftime('%Y-%m-%d %H:%M:%S')`를 사용하여 `datetime` 객체를 'YYYY-MM-DD HH:MM:SS' 형식의 문자열로 다시 포맷팅합니다.
    *   **`jsonResult.append(...)`**: 처리된 데이터를 딕셔너리 형태로 만들어 `jsonResult` 리스트에 추가합니다. `cnt`는 각 항목의 번호를 나타냅니다.
*   **`getPostData_Blog`**: 블로그 검색을 위한 함수로, `originallink`와 `postdate` 필드 처리가 뉴스 API와 다릅니다. 현재 코드에서는 주석 처리되어 사용되지 않습니다.
---
#### 2.5. `main()` 함수 (`[CODE 0]`)

```python
def main():
    node = 'news' # 검색 대상. 'blog'에서 'news'로 변경됨.
    srcText = input('검색어를 입력하세요: ') # 사용자 입력
    cnt = 0
    jsonResult = [] # 모든 검색 결과를 저장할 리스트

    # 첫 페이지 요청으로 total 값 확인 (최대 1000개 제한)
    jsonResponse = getNaverSearch(node, srcText, 1, 100) # [CODE 2]
    total = jsonResponse['total'] if jsonResponse['total'] < 1000 else 1000

    # total 건수 또는 1000건까지 반복해서 데이터를 가져옴
    while ((jsonResponse != None) and (jsonResponse['display'] != 0) and cnt < total):
        for post in jsonResponse['items']:
            cnt += 1
            getPostData(post, jsonResult, cnt) # [CODE 3] 뉴스 데이터 파싱 및 추가
        
        start = jsonResponse['start'] + jsonResponse['display'] # 다음 페이지 시작 위치 계산
        jsonResponse = getNaverSearch(node, srcText, start, 100) # [CODE 2] 다음 페이지 요청

    print('전체 검색 : %d 건' % total)

    # --- CSV 파일 저장 ---
    csv_filename = '%s_naver_%s.csv' % (srcText, node)
    with open(csv_filename, 'w', encoding='utf-8-sig', newline='') as outfile:
        writer = csv.writer(outfile)
        writer.writerow(['번호', '제목', '요약', '원본링크', '네이버뉴스링크', '게시일']) # CSV 헤더
        for item in jsonResult:
            writer.writerow([item['cnt'], item['title'], item['description'], item['org_link'], item['link'], item['pDate']])
    print(f"'{csv_filename}' SAVED")

    # --- JSON 파일 저장 ---
    json_filename = '%s_naver_%s.json' % (srcText, node)
    with open(json_filename, 'w', encoding='utf8') as outfile:
        jsonFile = json.dumps(jsonResult, indent=4, sort_keys=True, ensure_ascii=False) # JSON 예쁘게 포맷팅
        outfile.write(jsonFile)
    print(f"'{json_filename}' SAVED")

    print("가져온 데이터 : %d 건" % (cnt))

if __name__ == '__main__':
    main()
```

*   **`node = 'news'`**: 검색 대상을 뉴스로 명시적으로 설정했습니다.
*   **`srcText = input('검색어를 입력하세요: ')`**: 사용자로부터 검색어를 입력받습니다.
*   **`jsonResult = []`**: 모든 수집된 데이터를 저장할 빈 리스트를 초기화합니다.
*   **`total = jsonResponse['total'] if jsonResponse['total'] < 1000 else 1000`**:
    *   `getNaverSearch`를 통해 첫 페이지의 응답을 받아 `total` 키를 통해 전체 검색 결과 수를 확인합니다.
    *   Naver API는 한 번의 쿼리에 최대 1000건까지만 조회할 수 있으므로, `total`이 1000보다 크더라도 최대 1000으로 제한합니다.
    
*   **`while` 루프**:
    *   `jsonResponse != None`: API 요청이 성공했는지 확인.
    *   `jsonResponse['display'] != 0`: 현재 페이지에 결과가 있는지 확인.
    *   `cnt < total`: 전체 목표 건수까지 도달했는지 확인.
    *   이 조건들을 만족하는 동안 루프를 계속 실행하여 다음 페이지의 데이터를 가져옵니다.
    
*   **`for post in jsonResponse['items']`**: 현재 페이지의 각 뉴스 아이템을 순회하며 `getPostData` 함수를 호출하여 `jsonResult`에 추가합니다.
*   **`start = jsonResponse['start'] + jsonResponse['display']`**: 다음 API 요청 시 시작 위치를 현재 페이지의 시작 위치 + 한 번에 가져온 개수로 업데이트합니다.

*   **CSV 파일 저장**:
    *   `csv_filename = '%s_naver_%s.csv' % (srcText, node)`: 검색어와 검색 대상(`node`)을 포함한 파일명을 생성합니다 (예: `부산 서면 맛집_naver_news.csv`).
    *   `with open(..., encoding='utf-8-sig', newline='') as outfile:`: 한글 깨짐 방지 및 빈 줄 제거를 위해 설정.
    *   `writer.writerow([...])`: 헤더와 각 `item`의 데이터를 행 단위로 CSV 파일에 작성합니다.
    
*   **JSON 파일 저장**:
    *   `json_filename = '%s_naver_%s.json' % (srcText, node)`: JSON 파일명을 생성합니다.
    *   `json.dumps(jsonResult, indent=4, sort_keys=True, ensure_ascii=False)`:
        *   `jsonResult` 리스트를 JSON 형식 문자열로 변환합니다.
        *   `indent=4`: JSON을 들여쓰기하여 사람이 읽기 쉬운 형태로 만듭니다.
        *   `sort_keys=True`: 딕셔너리의 키를 알파벳 순으로 정렬합니다.
        *   `ensure_ascii=False`: 한글이 유니코드 이스케이프 시퀀스(`\uXXXX`)로 변환되지 않고 실제 한글 문자 그대로 저장되도록 합니다.
    *   `outfile.write(jsonFile)`: JSON 문자열을 파일에 씁니다.
---
### 3. 이전 코드와의 차이점 및 개선 사항

1.  **함수 분리 및 모듈화**: 각 기능(API 요청, 검색 파라미터 조합, 데이터 파싱)이 별도의 함수로 분리되어 코드가 훨씬 깔끔하고 재사용성이 높아졌습니다.
2.  **페이지네이션(Pagination)**: `while` 루프를 사용하여 `start` 파라미터를 조절함으로써 여러 페이지에 걸쳐 최대 1000개의 검색 결과를 가져올 수 있습니다. 이전 코드는 한 번의 요청으로 최대 100개까지만 가져올 수 있었습니다.
3.  **향상된 오류 처리**: `getRequestUrl` 함수에 `try-except` 블록이 명시적으로 추가되어 API 요청 중 발생할 수 있는 다양한 네트워크 오류나 HTTP 오류를 처리합니다.
4.  **날짜 형식 변환**: `datetime` 모듈을 사용하여 API에서 제공하는 날짜 형식을 'YYYY-MM-DD HH:MM:SS'와 같은 표준 형식으로 변환합니다.
5.  **두 가지 저장 형식**: 결과를 CSV 뿐만 아니라 JSON 파일로도 저장합니다. JSON은 데이터 교환에 널리 사용되는 형식이며, Python 딕셔너리와 직접적으로 매핑되기 때문에 프로그램에서 다루기 용이합니다.
6.  **사용자 입력**: `input()` 함수를 사용하여 검색어를 직접 입력받아 더욱 유연하게 사용할 수 있습니다.
7.  **`__name__ == '__main__'`**: 파이썬 스크립트의 표준 관례를 따릅니다. 이 스크립트가 직접 실행될 때만 `main()` 함수가 호출되도록 합니다.

이 코드는 Naver API를 활용한 데이터 수집의 좋은 예시이며, 실제 프로젝트에서 데이터를 크롤링하고 처리하는 데 필요한 많은 기술을 담고 있습니다. 아주 좋은 학습 자료가 될 것입니다!