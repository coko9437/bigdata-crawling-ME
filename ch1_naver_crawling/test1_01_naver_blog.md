이 코드는 가장 첫 번째로 작성하신 코드이자, 이전에 "두 번째 코드"로 설명해드렸던 코드와 매우 유사합니다.
### 1. 코드 전체적인 흐름

1.  **필요한 라이브러리 임포트**: `urllib.request`, `json`, `csv` 등을 가져옵니다.
2.  **API 인증 정보 및 검색어 설정**: Naver API를 사용하기 위한 `client_id`, `client_secret`과 검색어 `encText`를 정의합니다.
3.  **API 요청 URL 구성**: Naver 블로그 검색 API의 URL과 파라미터(`query`)를 조합하여 최종 요청 URL을 만듭니다. (여기서는 `display`, `sort` 파라미터가 명시적으로 추가되지 않아 기본값으로 동작합니다.)
4.  **API 요청 및 응답**: 구성된 URL로 Naver API에 요청을 보내고 응답을 받습니다.

5.  **응답 처리**:
    *   HTTP 상태 코드가 200(성공)인 경우:
        *   응답 본문을 읽어와 UTF-8로 디코딩하고 출력합니다.
        *   JSON 형식의 응답 데이터를 파싱합니다.
        *   파싱된 데이터에서 블로그 아이템들을 추출합니다.
        *   추출된 블로그 아이템들을 "naver\_blog\_results.csv"라는 CSV 파일로 저장합니다.
    *   오류 발생 시: 오류 코드를 출력합니다.
---
### 2. 코드 상세 설명

#### 2.1. 라이브러리 임포트

```python
import os
import sys
import urllib.request
import json
import csv
```

*   이전 코드들과 동일하게 `urllib.request` (API 요청), `json` (JSON 파싱), `csv` (CSV 저장) 라이브러리를 임포트합니다. `os`, `sys`는 여기서는 직접 사용되지 않습니다.
---
#### 2.2. 인증 정보 및 검색어 설정

```python
client_id = "Kc65Ibtu71R5TLRUF7lL"
client_secret = "CdT4sLf9Ht"
encText = urllib.parse.quote("부산 서면 맛집")
```

*   `client_id`, `client_secret`: Naver 개발자 센터에서 발급받은 애플리케이션의 인증 정보입니다.
*   `encText`: 검색할 쿼리(질의) 문자열입니다. `urllib.parse.quote()`를 사용하여 URL 인코딩을 수행합니다.
---
#### 2.3. API 요청 URL 구성

```python
url = "https://openapi.naver.com/v1/search/blog?query=" + encText # JSON 결과
# url = "https://openapi.naver.com/v1/search/blog.[test1_02_naver_news.md](test1_02_naver_news.md)xml?query=" + encText # XML 결과
```

*   `url`: Naver 블로그 검색 API의 엔드포인트 URL입니다.
    *   `https://openapi.naver.com/v1/search/blog`: 블로그 검색 API의 기본 URL입니다.
    *   `?query=` + `encText`: `query` 파라미터에 인코딩된 검색어를 전달합니다.
    *   `display`나 `sort`와 같은 다른 파라미터가 명시적으로 추가되지 않았으므로, API의 기본값(일반적으로 `display=10`, `sort=sim`)이 사용됩니다. 즉, 이 코드는 최대 10개의 블로그 게시물을 관련도 순으로 가져옵니다.
---
#### 2.4. API 요청 및 응답 받기

```python
request = urllib.request.Request(url)
request.add_header("X-Naver-Client-Id",client_id)
request.add_header("X-Naver-Client-Secret",client_secret)
response = urllib.request.urlopen(request)
rescode = response.getcode()
```

*   `urllib.request.Request(url)`: HTTP 요청 객체를 생성합니다.
*   `request.add_header(...)`: HTTP 요청 헤더에 클라이언트 ID와 시크릿을 추가합니다.
*   `urllib.request.urlopen(request)`: 요청을 보내고 응답 객체를 받습니다.
*   `response.getcode()`: HTTP 상태 코드를 가져옵니다.
---
#### 2.5. 응답 처리 및 JSON 파싱

```python
if(rescode==200):
    response_body = response.read()
    print(response_body.decode('utf-8'))

    # 1. JSON 데이터 파싱
    data = json.loads(response_body.decode('utf-8'))
    items = data['items'] # 블로그 검색 결과 리스트
```

*   `if(rescode==200)`: HTTP 상태 코드가 200이면 성공적으로 응답을 받은 것입니다.
*   `response_body = response.read()`: 응답 본문을 읽어옵니다.
*   `print(response_body.decode('utf-8'))`: 읽어온 바이너리 응답 본문을 UTF-8로 디코딩하여 콘솔에 출력합니다.
*   `data = json.loads(response_body.decode('utf-8'))`: JSON 문자열을 Python 딕셔너리로 변환합니다.
    *   Naver 블로그 API의 JSON 응답 구조는 대략 다음과 같습니다:
        ```json
        {
            "lastBuildDate": "...",
            "total": ...,
            "start": ...,
            "display": ...,
            "items": [
                {
                    "title": "...",
                    "link": "...",
                    "description": "...",
                    "bloggername": "...",
                    "bloggerlink": "...",
                    "postdate": "YYYYMMDD"
                },
                ...
            ]
        }
        ```
*   `items = data['items']`: 파싱된 딕셔너리 `data`에서 `'items'` 키에 해당하는 값을 가져옵니다. 이 값은 블로그 게시물 각각을 담고 있는 딕셔너리들의 리스트입니다.
---
#### 2.6. CSV 파일로 저장

```python
    # 2. CSV 파일로 저장
    filename = "naver_blog_results.csv"
    with open(filename, 'w', newline='', encoding='utf-8-sig') as f:
        writer = csv.writer(f)

        # CSV 헤더 작성
        writer.writerow(['제목', '링크', '요약', '블로거 이름', '게시일'])

        # 각 검색 결과를 한 행씩 작성
        for item in items:
            writer.writerow([
                item['title'],
                item['link'],
                item['description'],
                item['bloggername'],
                item['postdate']
            ])

    print(f"'{filename}' 파일로 저장되었습니다.")
```

*   `filename = "naver_blog_results.csv"`: 저장할 CSV 파일의 이름을 정의합니다.
*   `with open(filename, 'w', newline='', encoding='utf-8-sig') as f:`: CSV 파일을 쓰기 모드(`'w'`)로 엽니다. `newline=''`과 `encoding='utf-8-sig'`는 이전 코드와 동일하게 CSV 파일 저장 시 발생할 수 있는 문제(빈 줄 추가, 한글 깨짐)를 방지합니다.
*   `writer = csv.writer(f)`: CSV `writer` 객체를 생성합니다.
*   `writer.writerow(...)`: CSV 파일의 첫 번째 줄에 헤더(열 이름)를 작성합니다.
    *   `title`: 블로그 게시물의 제목 (HTML 태그 포함될 수 있음)
    *   `link`: 해당 블로그 게시물의 고유 주소
    *   `description`: 블로그 게시물의 요약 설명 (HTML 태그 포함될 수 있음)
    *   `bloggername`: 블로그 작성자의 이름
    *   `postdate`: 블로그 게시물이 작성된 날짜 (`YYYYMMDD` 형식)
*   `for item in items:`: `items` 리스트의 각 블로그 게시물(`item`)에 대해 반복합니다.
*   `writer.writerow([...])`: 현재 `item`의 각 필드 값들을 리스트로 묶어 CSV 파일의 한 행으로 작성합니다. **참고**: 이 코드에서는 `title`과 `description`에서 HTML 태그를 제거하는 로직이 없습니다. 따라서 CSV 파일에는 `<b>부산</b>`과 같이 태그가 포함된 채로 저장될 수 있습니다. (이후 수정된 코드에서는 이 부분을 개선했습니다.)
---
#### 2.7. 오류 처리

```python
else:
    print("Error Code:" + rescode)
```

*   `if(rescode==200)` 조건에 해당하지 않을 경우, 오류 코드를 출력합니다. (예: `401: Unauthorized` 등)

### 3. 이전 코드들과의 관계 및 학습 관점

*   **기본 구조**: 이 코드는 API를 사용하여 데이터를 가져오고 파일로 저장하는 가장 기본적인 형태를 보여줍니다.
*   **발전 과정**:
    *   **이 코드 (블로그 검색)**: 단일 페이지에서 기본 파라미터로 데이터를 가져와 CSV로 저장. HTML 태그 제거 로직 없음.
    *   **두 번째로 설명한 코드 (뉴스 검색)**: 이 코드와 거의 동일하지만, `display=100`, `sort=sim` 파라미터를 추가하고, `title`, `description`에서 HTML 태그 제거 로직을 추가하여 데이터 정제 기능을 개선했습니다.
    *   **세 번째로 설명한 코드 (함수화된 Naver 뉴스 검색)**: 함수 분리, 여러 페이지(`start` 파라미터)를 통한 데이터 수집, 날짜 형식 변환, CSV/JSON 동시 저장, 사용자 입력, 에러 처리 강화 등 코드의 구조와 기능이 크게 발전했습니다.
    *   **네 번째로 설명한 코드 (공공데이터포털)**: 다른 API(`ServiceKey` 인증), 다른 응답 JSON 구조, Pandas 활용 등 더욱 복잡한 데이터 수집 시나리오를 처리하는 방법을 보여줍니다.
