---
title:  "iOS) HTTP 메시지 구조를 통해 GET, POST 이해하기"
categories:
- iOS

date:   2021-10-04  16:23:00 +0900
author_profile: false
---
## 🍕 HTTP 통신

HTTP 통신은 client 가 데이터가 필요할 때마다 server 에게 요청하고,  server 는 그 데이터를 응답하고 바로 연결이 종료되는 방식이다.

<img width="400" alt="스크린샷 2021-10-04 오후 12 51 32" src="https://user-images.githubusercontent.com/69136340/135809623-d77771ae-0dc7-4012-b225-4fcf2fb15a49.png">

이처럼 HTTP 통신은 **단방향 통신**이라서 client 만 server 에게 요청할 수 있고 server 는 client 에 요청할 수 없다. 응답만 가능하다.

실시간으로 서비스해야할 경우를 제외하고 HTTP 통신을 사용하는데 server 의 부담을 줄여준다. 

## 🍕 HTTP 메시지

client 가 server 에게 요청할 때 일정한 **형식**을 갖춰서 텍스트 기반의 메시지로 변환해서 전달해야한다. 그 형식을 **HTTP 메시지**라고 한다.

### **구조**

요청메시지와 응답 메시지를 알아보자. 공통적으로 `라인 - 헤더 - 바디` 로 구성된다.

<img width="500" alt="스크린샷 2021-10-04 오후 12 58 12" src="https://user-images.githubusercontent.com/69136340/135809653-54c6a99b-dc74-4e96-bc87-c10b3d03abe2.png">

### 라인

메시지의 가장 기본 내용인 **응답/요청 여부, 메시지 전송 방식, 상태** 등이 작성. 한줄.

ex) `POST /user/login HTTP/1.1` 

[전송메서드 정의] [경로] [요청형식에 대한 버전 정보]

### 헤더

메시지 본문에 대한 메타 정보가 들어간다. 필요한 만큼 여러줄로 작성.

길이가 유동적이기 때문에 바디와 구분 짓기 위해서 한 줄의 공백 삽입.

ex) 

`Host: ...`

`Content-Type: application/json`

Host 라는 key 에는 도메인 및 포트번호를 value 로 가짐.

Content-Type 라는 key 에는 메시지 바디의 타입을 나타내는 value 를 가짐.(주로 사용하는 JSON 의 경우는 application/json value 를 가짐)

### 바디

메시지 본문이 들어가는 부분. 길이가 유동적.

바디에 들어가는 메시지 형식은 헤더의 Content-Type 에서 설정한 타입과 일치해야 한다.

## 🍕 GET, POST 의 요청 메세지 특징

### POST

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/135809732-d3fc2b52-042e-4556-bc3b-07ec62610a23.png">

(출처를 남긴 블로그 속 예시를 가져왔다.)

**특징**

- 헤더에 Content-Type 을 작성
- 바디에 Content-Type 에 맞는 데이터를 넣는 것.

### GET

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/135809754-8a03b652-a705-4dc1-a7a5-5386da8c076f.png">

(출처를 남긴 블로그 속 예시를 가져왔다.)

**특징**

- 헤더가 no-cache 로 변경됨.
- 바디가 없음.(본문을 사용하지 않기 때문에 헤더의 Content-Type 이 사용되지 않음.)
- GET 은 메시지 본문을 사용하지 않고 URL 뒤에 파라미터로 연결시킴. 이것이 쿼리 스트링(Query String).
- URL 경로의 허용범위가 1024Byte 이기 때문에 긴 값은 전송할 수 없다.(그래서 GET 과 같이 필요한 정보를 **요청**할 때 사용)

출처 :

[iOS) HTTP / HTTPS / RESTful 이 도대체 뭘까](https://babbab2.tistory.com/70)

---

## 🍕 응답코드

추가로 간단하게 응답코드도 알아보자.

- 1XX : Informational(정보). 정보 교환.
- 2XX : Success(성공). 데이터 전송이 성공적으로 이루어졌거나, 이해되었거나, 수락되었음.
    - 200 : 오류 없이 전송 성공.
- 3XX : Redirection(방향 바꿈). 자료의 위치가 바뀌었음.
- 4XX : Client Error(클라이언트 오류). 클라이언트 측의 오류. 주소를 잘못 입력하였거나 요청이 잘못 되었음.
- 5XX : Server Error(서버 오류). 서버 측의 오류로 올바른 요청을 처리할 수 없음.

더 자세한 정보는 다음 출처를 읽어보자.

출처 : 

[HTTP - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/HTTP)
