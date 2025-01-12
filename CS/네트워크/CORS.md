## **CORS는 왜 필요한가요?**

---
<img width="743" alt="image" src="https://github.com/user-attachments/assets/bfa06346-5715-4c9b-8cef-b7ce5fd342be" />

CORS(Cross-Origin Resource Sharing)는 보안상의 이유로 웹 브라우저가 **다른 출처(도메인, 프로토콜, 포트)의 리소스에 접근하는 요청을 제한**하기 위해 만들어졌습니다.

- **기본 보안 정책**: 브라우저는 SOP(Same-Origin Policy)을 따릅니다. 이는 스크립트가 자신이 로드된 출처에서만 리소스를 요청할 수 있도록 제한합니다.
- **SOP** **문제점**: 동일 출처 정책은 다른 도메인의 API나 리소스를 사용하는 현대 웹 애플리케이션의 요구(ex, 프론트와 백이 나뉨)를 충족하지 못합니다.
- **CORS의 역할**: CORS는 **서버가 명시적으로 허용한 출처에 대해 브라우저가 교차 출처 요청을 허용하도록** 돕는 메커니즘입니다.

## **CORS 작동원리는? 🤔**

---

### 2. **CORS 작동 원리**

CORS는 브라우저와 서버 간의 요청/응답에 **특정 HTTP 헤더**를 추가하여 작동합니다. 단순요청이 아니라면, 브라우저는 요청을 보내기 전 `OPTIONS` 메서드를 사용하여 사전 요청(`Preflight`)을 보냅니다. 서버가 `Access-Control-Allow-Origin` 헤더를 응답으로 보내면 요청이 허용됩니다. 이후 클라이언트는 실제 요청을 보냅니다.
매번 사전 요청을 보낼 수는 없으니 브라우저는 `Preflight` 의 결과를 캐싱합니다.

- **1단계: Preflight 요청 (사전 요청)**
    - "브라우저는 , 요청을 보내기 전에 서버의 허용 여부를 확인합니다.
    - ""
- **2단계: 실제 요청 (Actual Request)**
    - "사전 요청이 허용되었을 경우, 브라우저는 실제 데이터 요청을 전송합니다."

### (1) **단순 요청(Simple Request)**

- GET, POST, HEAD 메서드로 이루어진 요청이며, Content-Type이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 인 것입니다.
- 브라우저는 요청 헤더에 `Origin`을 추가하여 서버에 요청의 출처를 명시하고, 서버는 응답 헤더에 `Access-Control-Allow-Origin`을 추가하여 허용 여부를 명시합니다.

### (2) **사전 요청(Preflight Request)**

- 단순요청이 아닌 것들
    - **GET, POST, HEAD 외의 메서드**를 사용하거나, 커스텀 헤더, 비표준 Content-Type 등을 사용하는 경우 **OPTIONS 메서드**로 사전 확인 요청을 보냅니다.
- 브라우저는 서버에 **`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`** 등을 요청하고, 서버가 이를 허용하면 실제 요청을 보냅니다.
- 서버는 응답에 `Access-Control-Max-Age`를 추가하여 Preflight 결과를 **`캐싱`**하도록 설정할 수 있습니다.

### (3) **자격 증명 포함 요청**

- 쿠키나 인증 토큰 등의 자격 증명을 포함하려면 클라이언트는 `credentials` 옵션을 설정해야 하며, 서버는 응답에 `Access-Control-Allow-Credentials: true`를 추가해야 요청이 허용됩니다.

## 참고

---

- • [토스페이먼츠 - CORS 대응하기](https://docs.tosspayments.com/blog/payment-window-cors-error#%EA%B2%B0%EC%A0%9C%EC%B0%BD%EC%97%90%EC%84%9C-cors-%EB%8C%80%EC%9D%91%ED%95%98%EA%B8%B0)
- • [MDN - 교차 출처 리소스 공유 (CORS)](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)
- • [테코블 - CORS란?](https://tecoble.techcourse.co.kr/post/2020-07-18-cors/)
