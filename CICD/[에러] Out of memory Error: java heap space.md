## **에러원인:  Out of memory Error / java heap space**

![image](https://github.com/user-attachments/assets/038b006e-5a45-4e96-974b-be7327e92b02)

```
//컨테이너 메모리 사용량 보기
docker stats <container-id>
```

![image](https://github.com/user-attachments/assets/22d844d9-cc83-48e0-a505-c8648848c3f3)

위 명령어로 컨테이너 메모리 사용량을 모니터링했습니다. MEM USAGE / LIMIT 열 부분을 확인해 보면 1/2이나 메모리 공간이 남은 것을 알 수 있습니다. 그러면 log파일이 커져서 문제가 된건 아니구나를 다시 한 번 알 수 있습니다. 

## **사전 지식**

JVM은 기본적으로 Heap 크기를 최소 1/64로 설정합니다. 그래서 만약 heap space를 늘려주고 싶다면 dockerfile이나 도커 명령어를 사용하여 

## **해결법**

여기서 -Xmx는 최대 힙 메모리 크기를, -Xms는 초기 힙 메모리 크기를 설정합니다. 예를 들어, -Xmx512m는 최대 512MB의 힙 메모리를 JVM에 할당하는 것입니다.

```
// dockerfile의 ENTRYPOINT 부분을 아래와 같이 수정
ENTRYPOINT ["java", "-Xmx512m", "-Xms512m", "-jar", "-Dspring.profiles.active=production", "project.jar"]
```

```
//먼저, 수정된 server 컨테이너의 이미지를 다시 빌드합니다:

Copydocker-compose build server

//그 다음, server 컨테이너만 중지하고 제거한 후 다시 시작합니다:

Copydocker-compose up -d --no-deps --build server
```
