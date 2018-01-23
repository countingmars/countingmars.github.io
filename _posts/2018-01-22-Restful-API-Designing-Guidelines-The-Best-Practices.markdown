---
layout: post
title:  "Restful API - The Best Practices"
date:   2018-01-15 18:25:01
author: Mars
categories: tools
---


https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9

### API endpoint
회사와 직원에 대한 API 를 설계해보자.
아래와 같은 API endpoint 들이 있을 수 있다.
- /addNewEmployee
- /updateEmployee
- /deleteEmployee
- /deleteAllEmployees
- /promoteEmployee
- /promoteAllEmployees

이러한 설계는 API 숫자가 증가할 때 유지보수하기 힘들어진다.

#### What is wrong?
가능하면 URL 에는 오직 리소스(명사)만을 포함하게 하고 행위(동사)를 포함하지 않는 것이 좋다.

addNewEmployee 는 addNew 라는 행위와 Employee 라는 리소스를 포함하고 있다.

#### Then what is the correct way?
/companies 라는 endpoint 가 행위가 포함되어 있지 좋은 예제라고 볼 수 있다.

그렇다면 어떻게 행위(action)을 정의할 수 있을까?

HTTP methods (GET, POST, DELETE, PUT) 를 활용하자.
리소스는 가능하면 항상 복수형태인 것이 좋고, 만약 그 중에 하나의 인스턴스에 접근하고 싶다면 id 를 URL 에 전달하자.

- GET /companies 는 모든 회사 목록을 얻어야 한다.
- GET /companies/34 는 하나의 회사를 얻어야 한다.
- DELETE /companies/34 는 회사 34를 삭제해야 한다.

그리고 회사의 직원을 다루는 API endpoint 는 아래의 형태가 좋다.
- GET /companies/3/employees 회사 3의 직원 전체를 얻어야 한다.
- GET /companies/3/employees/45 회사 3의 직원 45를 얻어야 한다.
- POST /companies 새로운 회사 하나를 생성하고 상세 내역을 반환해야 한다.


Conclusion: URL 경로는 리소스의 복수형을 가져야만 하며 and HTTP method 는 리소스에 수행할 행위(action)을 정의해야 한다.

### HTTP methods (verbs)
HTTP methods 가 지켜야 할 규칙이라면 아래와 같다:
- GET method: 리소스에 대한 데이터 요청이어야 하며 사이드 이펙트(데이터 갱신)를 만들지 말아야 한다.
- POST method: 새로운 데이터 생성 요청이어야 하며, 주로 form submit 했을 때 발생한다.
	* POST 는 매 요청마다 새로운 리소스를 생성해야 한다(non-idempotent).
- PUT method: 리소스를 업데이트하거나 만약 존재하지 않는 리소스라면 생성해야 한다.
	* PUT 은 여러번 요청할 때 동일한 결과를 발생시켜야 한다(idempotent).
- DELETE method: 삭제.


### HTTP response status codes
클라이언트는 자신의 요청의 처리결과를 어떻게 알 수 있을까?
HTTP status codes 는 이러한 정보를 표현하기 위한 수단이다.


#### 2xx (Success category)
이 상태 코드는 어떤 형태로든 서버에서의 처리가 성공했음을 나타낸다.

- 200 Ok: GET, PUT or POST.
- 201 Created: 새로운 인스턴스가 생성되었을 때, 특히 POST method 에 대한 응답은 201 상태 코드가 좋다.
- 204 No Content: 요청이 성공적으로 처리되었지만, 반환할 데이터가 없음을 의미한다. 예를 들어 이미 삭제한 인스턴스에 대한 DELETE 요청에 대한 응답으로 일반적으로 쓰인다.

#### 3xx (Redirection Category)
304 Not Modified indicates that the client has the response already in its cache. And hence there is no need to transfer the same data again.

#### 4xx (Client Error Category)
These status codes represent that the client has raised a faulty request.

- 400 Bad Request: 클라이언트의 요청을 이해할 수 없어서, 요청이 처리되지 않았음을 의미한다.
- 401 Unauthorized: 접근 권한이 없음을 의미한다.
- 403 Forbidden: 인증된 사용자이지만, 권한이 없음을 의미한다.
- 404 Not Found: 없다.
- 410 Gone: 리소스가 다른 곳으로 옮겨져서 데이터 이용할 수 없는 상태임을 의미한다.

#### 5xx (Server Error Category)
- 500 Internal Server Error: 유효한 요청을 받았지만, 내부적으로 문제가 발생했음을 의미한다.
- 503 Service Unavailable: 서버 사망했거나 점검중일 때이다.

### Field name casing convention
- Sorting: /companies?sort=rank_asc
- Filtering: /companies?category=banking&location=india
- Searching: /companies?search=Digital Mckinsey
- Pagination: /companies?page=23

* GET methods 의 URI 이 너무 길다면, 서버가 414 URI Too long HTTP 상태코드를 반환할거다, 이렇다면 POST method 를 써도 좋다.

### Versioning
API 가 이미 배포된 상태에서 업그레이드를 시도하면 API 호환성에 문제가 발생할 수 있다.
`http://api.yourservice.com/v1/companies/34/employees` 가 좋은 예제인데,
API 경로에 버전 정보를 넣는 것이다. 만약 API 사용법이 달라진다면 v2 혹은 v1.x.x 형태로 추가하자.
