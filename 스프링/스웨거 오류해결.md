## **\[버전 주의\]**

**스프링3**을 쓰는 분들은  **Springdoc  2.x.x**를 써야한다. 

Swagger --   springfox(옛날 버전 / 업데이트 멈춤)  
               |  
               |--  springdoc(최신버전)

```
    //swager
	implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'
```

이하 버전을 쓰거나 다른 dependenci를 받게된다면 Whitelabel Page가 뜰 것이다. 

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/26ca669c-4f50-409f-86e7-3f04ab068e75)

만약 버전을 맞게 했는데도 해결이 안되었다면 **접속 URL**를 확인해보세요!

## **\[어노테이션 마이그레이션\]**

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/7dfe02e9-820b-4771-a5a3-bc309c180462)

## **\[한글 깨짐 오류\]**

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/d894bd9d-4680-421a-8b3e-1ebe4e2cfc24)

소스코드르 빌딩할 때 UTF-8로 안되어 있어서 그렇다 

File -> Setting -> File Encodings 

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/1b3b24f8-b460-42f7-8156-6ee9ee2360c3)

다른 궁금점들은 공식문서를 참고하시길 바랍니다. 뭐니뭐니 해도 공식문서가 짱입니다~ 

https://springdoc.org/#how-can-i-hide-an-operation-or-a-controller-from-documentation](https://springdoc.org/#how-can-i-hide-an-operation-or-a-controller-from-documentation)

