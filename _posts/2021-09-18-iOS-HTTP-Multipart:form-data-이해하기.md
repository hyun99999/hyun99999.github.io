---
title:  "iOS) HTTP Multipart/form-data 이해하기"
categories:
- iOS

date:   2021-09-18  14:31:00 +0900
author_profile: false
---
아래의 글은 

[HTTP multipart/form-data 이해하기](https://lena-chamna.netlify.app/post/http_multipart_form-data/)

를 토대로 여러 내용을 종합한 글입니다.

### 클라이언트에서 서버로 파일 업로드과정 이해하기

파일 업로드를 구현할 때, 웹브라우저는 HTTP 메시지는 Content-Type 속성이 multipart/form-data 로 지정된다. 그리고 이 형식에 따라서 메시지를 인코딩하여 전송한다. 서버는 이를 처리하기 위해서 각 파트별로 분리하여 개별 파일의 정보를 얻는 것이다.

그렇기 때문에 앱에서 구현하기 위해서 multipart/form-data 의 형식을 따르는 것이다.

이미지 파일을 png 나 jpg 파일 자체를 전송하는 것이 아니다. 이미지 파일도 문자로 이루어져 있기 때문에 문자로 생성해서 HTTP request body 에 담아 서버로 전송하는 것이다.

<img width="250" alt="스크린샷 2021-09-18 오후 2 12 51" src="https://user-images.githubusercontent.com/69136340/133873941-ad5e486a-590f-4ec0-b744-d859efdf770b.png">

HTTP(request 와 response) 는 간단하게 4개의 파트로 나눌 수 있다. Message Body 로 들어가는 데이터 타입을 HTTP Header 에 명시할 수 있다. 이때 명시할 수 있는 필드가 바로 Content-Type 이다.

추가적으로 Content-type 필드에 MIME(Multipurpose Internet Mail Extensions) 타입을 기술해줄 수 있는데, 여러 타입 중 하나가 바로 multipart 입니다.

- [ Multipart 메세지 ]
    - **서로 붙어있는 여러 개의 메세지를 포함하여 하나의 복합 메세지**로 보내집니다.
    - MIME multipart 메세지는 **“Content-type:” 헤더**에 boundary 파라미터를 포함합니다.
    - **boundary는 메세지 파트를 구분하는 역할을 하며, 메세지의 시작과 끝 부분도 나타납니다.**
    - 첫번째 Boundary 전에 나오는 내용은 MIME을 지원하지 않는 클라이언트를 위해 제공됩니다.
    - **boundary 를 선택하는 것은 클라이언트의 몫**입니다. **보통 무작위의 문자를 선택**해 메세지의 본문과 충돌을 피합니다. Ex) UUID
    - 멀티파트 폼 제출:
    - HTTP form을 채워서 제출하면, 가변 길이 텍스트 필드와 업로드 될 객체는 각각 멀티파트 본문을 구성하는 하나의 파트가 되어 보내집니다. 멀티 파트 분몬은 여러 다른 종류와 길이의 값으로 채워진 form을 허용합니다.
    - `multipart/form-data`: 사용자가 양식을 작성한 결과 값의 집합을 번들로 만드는데 사용합니다.

### Multipart/form-data 형식

파일을 업로드하는 형식을 흉내내야하는데 그 형식이 multipart/form-data 것이다. 즉, request body 부분을 형식의 규칙에 맞게 구성해야한다. 간략히 알아보자.

1. 바운더리를 구분하기 위한 문자열을 임의로 정의한다.
2. 각 폼필드 요소의 값은 `-바운더리` 모양의 라인 하나로 구분된다.
3. 이후 해당 필드 요소 데이터에 대한 헤더를 정의한다. (Content-Disposition 헤더는 주로 여기서 사용된다.)
4. HTTP헤더와 마찬가지로 각 파트의 헤더와 본체는 빈줄 1개로 구분된다.
5. 해당 데이터값을 쓴다.
6. 다시 줄을 바꾸고 두 번째 바운더리를 시작한다. (2~5의 과정)
7. 모든 요소의 기입이 끝났으면 줄을 바꾸고 `-바운더리--`의 모양으로 데이터를 기록하고 끝낸다.

- HTTP 요청에서 줄 바꿈은 플랫폼에 상관없이 `\r\n` 으로 고정되어있다.
- header 와 body 를 구분하는 것은 개행문자 2개이다.
- 전송되는 request body 는 이진데이터이기 때문에 문자열을 그대로 쓸 수 없다. 그래서 헤더 정보는 문자열로 작성한 후 UTF8 로 인코딩해주어야 한다.

결론적으로 martipart/form-data 는 다음과 같은 형태를 가진다.

<img width="600" alt="스크린샷 2021-09-18 오후 1 22 56" src="https://user-images.githubusercontent.com/69136340/133873946-462a3ffb-3e3d-4797-bd79-4c6c9957edfa.png">

### 출처

[HTTP multipart/form-data 이해하기](https://lena-chamna.netlify.app/post/http_multipart_form-data/)

[multipart/form-data 타입의 HTTP 메시지 구성 방법 · Wireframe](https://soooprmx.com/multipart-form-data-타입의-http-메시지-구성-방법/)

[multipart/form-data 이용해서 사진/이미지 배열 업로드하기](https://lena-chamna.netlify.app/post/uploading_array_of_images_using_multipart_form-data_in_swift/)

[URLSession을 통한 간단한 업로드 - Swift · Wireframe](https://soooprmx.com/urlsession을-통한-간단한-업로드-swift/)
