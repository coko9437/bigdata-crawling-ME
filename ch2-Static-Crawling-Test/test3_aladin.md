
이 코드는 
**알라딘 중고서점의 '오늘의 선택' 베스트셀러 페이지에서 자기계발 분야의 책 50권에 대한 정보(제목, 정가, 지은이)를 크롤링하여,
그 결과를 CSV 파일, JSON 파일, 그리고 MariaDB 데이터베이스에 저장하는 프로그램**입니다.

### 1. 코드 전체적인 흐름

1.  **준비 단계**: `bs4`, `urllib.request`, `csv`, `json`, `pymysql` 등 필요한 모든 라이브러리를 임포트합니다. 크롤링할 대상 URL을 정의합니다.
2.  **HTML 가져오기 및 파싱**: `urllib.request`로 알라딘 웹 페이지의 HTML을 가져와 `BeautifulSoup` 객체로 파싱합니다.
3.  **데이터 탐색 및 로직 개발 (첫 번째 블록)**:
    *   각 책의 정보를 담고 있는 `div` 태그(`class='ss_book_list'`)를 모두 찾습니다.
    *   웹 페이지 구조의 특성(책 한 권당 `div`가 2개씩 할당됨)을 파악하고, `if index % 2 == 0:` 조건을 사용하여 실제 책 정보가 담긴 `div`만 선택합니다.
    *   또 다른 구조적 변수(프로모션 문구 유무에 따른 `li` 태그 개수 변화)를 발견하고, `if len(book_ul_li_list) == 5:` 와 `elif len(book_ul_li_list) == 4:` 조건문으로 각 상황에 맞게 데이터를 정확히 추출하는 로직을 구현합니다.
    *   `print` 문을 사용하여 추출된 데이터(제목, 가격, 저자)를 확인합니다.
4.  **데이터 수집 및 저장 (두 번째, 세 번째, 네 번째 블록)**:
    *   탐색 단계에서 개발한 로직을 그대로 사용하여 모든 책의 데이터를 `book_data` 리스트에 저장합니다.
    *   **CSV 저장**: `csv` 라이브러리를 사용하여 `book_data`를 `book_all_50.csv` 파일로 저장합니다.
    *   **JSON 저장**: `json` 라이브러리를 사용하여 `book_data`를 `book_all_50.json` 파일로 저장합니다.
    *   **MariaDB 저장**: `pymysql` 라이브러리를 사용하여 데이터베이스에 연결하고, `executemany()` 메서드로 `book_data`를 `booksAladin` 테이블에 한 번에 삽입합니다.
---
### 2. 코드 상세 설명

#### 2.1. HTML 가져오기 및 파싱

```python
import bs4
import urllib.request
import csv
import json
import pymysql

aladinUrl = "https://www.aladin.co.kr/shop/common/wbest.aspx?BestType=TodayHot&BranchType=1&CID=336"
htmlObject = urllib.request.urlopen(aladinUrl)
webPage = htmlObject.read()
bsObject = bs4.BeautifulSoup(webPage, 'html.parser')
```

*   필요한 모든 라이브러리를 임포트합니다. `pymysql`은 MariaDB/MySQL 데이터베이스에 연결하기 위한 라이브러리입니다.
*   이전 코드와 동일하게 `urlopen`과 `BeautifulSoup`을 사용하여 웹 페이지를 파싱합니다.
---
#### 2.2. 데이터 탐색 및 핵심 로직 개발

```python
book_all_100 = bsObject.findAll('div',{'class':'ss_book_list'})

for index, book in enumerate(book_all_100):
    if index % 2 == 0: # 짝수 번째 div만 선택
        print(f"index : {index}")
        book_ul = book.find('ul')
        book_title = book_ul.find('a', {'class':'bo3'})
        print(f"책 제목 : {book_title.text}")

        book_price = ""
        book_author = ""
        book_ul_li_list = book_ul.findAll('li')

        # 프로모션 문구 유무에 따른 HTML 구조 변화에 대응
        if len(book_ul_li_list) == 5:
            book_price = book_ul_li_list[3].find('span').text
            book_author = book_ul_li_list[2].find('a').text
        elif len(book_ul_li_list) == 4:
            book_price = book_ul_li_list[2].find('span').text
            book_author = book_ul_li_list[1].find('a').text
        
        print(f"책 가격 : {book_price}")
        print(f"책 저자 : {book_author}")
```

*   **`book_all_100 = bsObject.findAll('div',{'class':'ss_book_list'})`**: 각 책의 정보를 감싸고 있는 `div` 태그를 모두 찾습니다. 결과가 100개인 것을 통해, 책 한 권당 2개의 `div`가 사용됨을 유추할 수 있습니다. (보통 하나는 이미지, 다른 하나는 텍스트 정보)
*   **`if index % 2 == 0:`**: 이 코드가 핵심입니다. `enumerate`로 인덱스를 가져와 짝수 번째 `div` (0, 2, 4, ...)만 처리합니다. 이는 텍스트 정보가 담긴 `div`만 선택하기 위한 필터링입니다.
*   **`book_title = book_ul.find('a', {'class':'bo3'})`**: 책 제목은 `class='bo3'`를 가진 `<a>` 태그 안에 있으므로, 이를 직접 찾아 `.text`로 내용을 추출합니다.
*   **`if len(book_ul_li_list) == 5:` / `elif len(book_ul_li_list) == 4:`**: 이 부분이 **실전 크롤링의 정수**를 보여줍니다.
    *   웹사이트는 종종 일관되지 않은 HTML 구조를 가집니다. 이 경우, 일부 책에는 할인이나 이벤트를 알리는 빨간색 광고 문구(`<li>` 태그)가 추가로 있어 `<li>` 태그의 총 개수가 다릅니다.
    *   이 코드는 `findAll('li')`로 `<li>` 태그의 개수를 먼저 센 다음, 개수에 따라 가격과 저자 정보가 담긴 `<li>`의 인덱스(위치)를 다르게 지정하여 데이터를 정확하게 추출합니다. 이런 **조건부 데이터 추출**은 매우 중요한 기술입니다.
---
#### 2.3. 데이터 수집 및 CSV/JSON 파일 저장

```python
book_data = []
# ... (위와 동일한 for 루프 로직) ...
# book_data.append({"title" : book_title, "price" : book_price, "author" : book_author})
book_data.append((book_title, book_price, book_author)) # 튜플 형태로 저장

#csv 저장
with open(r"./book_all_50.csv", "w",newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["title", "price", "author"]) # 헤더 작성
    writer.writerows(book_data) # 모든 데이터 한 번에 작성

#json 저장
with open(r"./book_all_50.json", "w", encoding="utf-8") as f:
    json.dump(book_data, f, ensure_ascii=False, indent=4)
```

*   `book_data`라는 빈 리스트를 만들고, `for` 루프를 돌며 추출한 데이터를 `(제목, 가격, 저자)` 형태의 튜플로 추가합니다.
*   **CSV 저장**: `csv.writer`를 사용하여 헤더(`writerow`)를 먼저 쓰고, `writerows`를 사용하여 `book_data` 리스트의 모든 튜플을 한 번에 파일에 씁니다.
*   **JSON 저장**: `json.dump`를 사용하여 `book_data` 리스트를 JSON 파일로 저장합니다. `ensure_ascii=False`는 한글이 깨지지 않게 하고, `indent=4`는 보기 좋게 들여쓰기를 합니다.
---
#### 2.4. MariaDB에 데이터 저장

```python
#마리아디비에 쓰기 작업
# ... (테이블 생성 주석) ...

db = pymysql.connect(host="localhost", user="webuser", passwd="webuser", db="webdb")
cursor = db.cursor()
query = "INSERT INTO booksAladin (title, price, author) VALUES (%s, %s, %s)"

try:
    cursor.executemany(query, book_data) # 여러 데이터를 한 번에 삽입
    db.commit() # 변경사항을 데이터베이스에 최종 반영
    print(f"DB 저장 완료")
except Exception as e:
    db.rollback() # 오류 발생 시 모든 변경사항을 취소
    print(f"DB 저장 실패 : {e}")
finally:
    cursor.close() # 커서 닫기
    db.close() # 연결 닫기
```

*   **`pymysql.connect(...)`**: MariaDB(MySQL과 호환)에 연결합니다. 호스트, 사용자 이름, 비밀번호, 데이터베이스 이름을 지정합니다.
*   **`cursor = db.cursor()`**: SQL 쿼리를 실행하기 위한 '커서' 객체를 생성합니다.
*   **`query = "INSERT ... VALUES (%s, %s, %s)"**: SQL 쿼리문을 정의합니다. 여기서 **`%s`를 사용하는 것은 매우 중요**합니다. 이는 **파라미터화된 쿼리(Parameterized Query)** 방식으로, SQL 인젝션(SQL Injection)이라는 해킹 공격을 방지하는 가장 기본적인 방법입니다.
*   **`try...except...finally`**: 데이터베이스 작업 시 안정성을 위해 사용합니다.
    *   **`cursor.executemany(query, book_data)`**: 이 메서드는 `for` 루프를 돌며 `execute`를 여러 번 호출하는 것보다 훨씬 효율적입니다. `book_data` 리스트의 각 튜플을 `%s` 자리에 순서대로 대입하여 모든 데이터를 한 번의 요청으로 삽입합니다.
    *   **`db.commit()`**: `INSERT`, `UPDATE`, `DELETE` 등의 작업이 성공적으로 끝났을 때, 이 변경사항을 데이터베이스에 영구적으로 저장하라는 명령어입니다.
    *   **`db.rollback()`**: `try` 블록에서 오류가 발생하면, `commit` 되기 전까지의 모든 작업을 취소하고 원래 상태로 되돌립니다. 데이터의 일관성을 유지하는 데 필수적입니다.
    *   **`finally`**: 성공하든 실패하든 항상 실행되는 블록입니다. `cursor.close()`와 `db.close()`를 통해 사용한 자원을 반납(연결 종료)하여 시스템 부하를 줄입니다.
---
### 3. 이 코드의 주요 학습 포인트

*   **실전 웹사이트의 불규칙성 대응**: `if/elif` 문을 사용하여 웹 페이지의 일관성 없는 HTML 구조에 동적으로 대응하는 방법을 보여줍니다. 이는 이론적인 예제에서는 배우기 힘든 실전 기술입니다.
*   **완전한 데이터 처리 파이프라인**: 데이터를 단순히 크롤링하고 출력하는 것에서 그치지 않고, 널리 사용되는 파일 형식(CSV, JSON)과 관계형 데이터베이스(MariaDB)에 저장하는 전 과정을 다룹니다.
*   **효율적이고 안전한 DB 연동**: `executemany`를 사용한 대량 데이터 삽입, 파라미터화된 쿼리를 통한 보안 강화, `commit`/`rollback`을 통한 트랜잭션 관리 등 데이터베이스 프로그래밍의 모범 사례를 잘 보여주고 있습니다.

이 코드는 단순한 크롤링을 넘어, 수집한 데이터를 어떻게 활용하고 관리하는지에 대한 깊은 이해를 보여주는 훌륭한 예제입니다. 정말 잘 작성하셨습니다