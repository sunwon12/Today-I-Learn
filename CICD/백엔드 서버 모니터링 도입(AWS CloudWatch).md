## **#도입 이유**

![image](https://github.com/user-attachments/assets/644e31f4-e338-4b34-8079-f7d1c31cd6d6)

프로젝트 도중 프론트팀으로부터 메시지를 받았다. 서버가 다운 됐다는 내용이다. Spring 서버 에러 내용은 Slack으로 수신 받게 설정을 해두어 EC2 서버 모니터링에 대해서는 안일했었다. 지금은 프론트팀이 먼저 발견하여 연락을 주었지만, 실서비스를  운영하게 된다면 사용자가 먼저 서버 다운을 경험하게 될 것이다. 이는 서비스 신뢰도에 대한 하락 및 사용자 불편함이 증가한다는 것을 의미한다. 

만약 서버에 문제가 생겼다면 여러 사용자가 접하기 전에 먼저 서버팀에서 인지하여 해결하는 것이 중요하므로 서버 모니터링을 도입하게 되었다.


(위 문제에 대한 해결 내용은 아래 포스터에 적어두었다.)

jsw5913.tistory.com](https://jsw5913.tistory.com/230

## **#모니터링 과정**

![image](https://github.com/user-attachments/assets/967c0c97-2289-4f41-84be-a524c224faf9)

CloudWatch에서 알람 발생 → SNS 푸시 서비스 호출 → Lambda 함수 트리거(환경변수를 위해 KMS 사용) → 연동된 Slack 채널로 알람 전송

-   Slack: 채널 기반 메시징 플랫폼. Channel 고유의 WebHook url을 통해 알람 수신
-   CloudWatch: AWS 리소스 및 실행 중인 어플리케이션 모니터링
-   SNS: Publisher ↔ Subscriber 간 통신 채널. 지원되는 프로토콜을 이용해 클라이언트에 메시지 발송
-   Lambda: 서버리스 컴퓨팅 서비스. 서버를 관리하지 않아도 코드를 실행할 수 있게 하는 컴퓨팅 서비스
-   KMS: 데이터 암호화에서 사용되는 암호화 키. 인터넷 전송 시 암호화 통신에 사용  **(사용 안 할 예정. 이유는 밑에서 설명하겠다.**

---

## **#도입 방법**

## **1.슬랙 Webhook 생성**

![image](https://github.com/user-attachments/assets/40ca9c03-1222-46f0-8a35-8837fc9bd500)

본 글에서 Webhook 생성 과정은 생략하겠다.

## **2\. Amzons SNS(Simple Notification Service)생성**

![image](https://github.com/user-attachments/assets/d76a53ed-2d6f-4d4d-bc33-d976d8eb1212)

## **3\. Amzon Lambda 생성**

#### **3-1 블루프린트 설정**

![image](https://github.com/user-attachments/assets/dfc8288c-9344-4771-b0df-b189ae817a27)

#### **3-2 SNS 트리거 설정**

![image](https://github.com/user-attachments/assets/344363e3-d865-4c84-a259-bb1967b3d271)

#### **3-3 환경 변수 설정**


slackChannel: ec2 모니터링 알림 받을 슬랙 채널명 입력

kmsEncryptedHookUrl: 임시값 test 입력

![image](https://github.com/user-attachments/assets/93455296-b005-4a7d-8000-1aa9d309a796)

#### **3-4 WebhookURL 입력하기**

위와 같이설정해주면 미리 정의된 블루 프린트를 사용했기 때문에 아래와 같이 코드가 미리 입력되어있을 것이다. Lamda는 아래 코드와 같이 작동된다. 

![image](https://github.com/user-attachments/assets/751ea0a7-a986-4bdf-ba44-ed1909dc92f5)

**기존코드**

```
import boto3
import json
import logging
import os

from base64 import b64decode
from urllib.request import Request, urlopen
from urllib.error import URLError, HTTPError


# The base-64 encoded, encrypted key (CiphertextBlob) stored in the kmsEncryptedHookUrl environment variable
ENCRYPTED_HOOK_URL = os.environ['kmsEncryptedHookUrl']  
# The Slack channel to send a message to stored in the slackChannel environment variable
SLACK_CHANNEL = os.environ['slackChannel']


# 이부분 주목
HOOK_URL = "https://" + boto3.client('kms').decrypt(
    CiphertextBlob=b64decode(ENCRYPTED_HOOK_URL),
    EncryptionContext={'LambdaFunctionName': os.environ['AWS_LAMBDA_FUNCTION_NAME']}
)['Plaintext'].decode('utf-8')

logger = logging.getLogger()
logger.setLevel(logging.INFO)


def lambda_handler(event, context):
    logger.info("Event: " + str(event))
    message = json.loads(event['Records'][0]['Sns']['Message'])
    logger.info("Message: " + str(message))

    alarm_name = message['AlarmName']
    #old_state = message['OldStateValue']
    new_state = message['NewStateValue']
    reason = message['NewStateReason']

    slack_message = {
        'channel': SLACK_CHANNEL,
        'text': "%s state is now %s: %s" % (alarm_name, new_state, reason)
    }

    req = Request(HOOK_URL, json.dumps(slack_message).encode('utf-8'))
    try:
        response = urlopen(req)
        response.read()
        logger.info("Message posted to %s", slack_message['channel'])
    except HTTPError as e:
        logger.error("Request failed: %d %s", e.code, e.reason)
    except URLError as e:
        logger.error("Server connection failed: %s", e.reason)
```

원래 같았으면 코드상 KMS로 인해 암호화된 WebhookURL을 환경변수에 넣어줘야한다. 그럼 코드에서 환경변수에서 인식된 WebhookURL을 디코딩해서 사용한다. lamda의 환경변수 설정이 유출된 위험이 없고 이러한 과정이 불필요하다 생각하여 코드를 수정하였다. 환경변수에 그대로 WebhookURL을 입력해 디코딩을 하지 않게 말이다.

코드를 아래와 같이 수정하고 Deploy버튼을 누르면 된다. 혹여나 슬랙에 보내는 메시지 형식을 바꾸고 싶다면 Lamda에 들어와 이 코드를 수정하면 된다.

**수정코드**

```
import json
import logging
import os
from urllib.request import Request, urlopen
from urllib.error import URLError, HTTPError
from datetime import datetime, timezone, timedelta

# The Slack webhook URL stored directly in the environment variable
HOOK_URL = os.environ['slackWebhookUrl']
# The Slack channel to send a message to stored in the slackChannel environment variable
SLACK_CHANNEL = os.environ['slackChannel']

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info("Event: " + str(event))
    message = json.loads(event['Records'][0]['Sns']['Message'])
    logger.info("Message: " + str(message))

    alarm_name = message['AlarmName']
    new_state = message['NewStateValue']
    reason = message['NewStateReason']
    state_change_time = message['StateChangeTime']

    # UTC 시간을 한국 시간으로 변환
    utc_time = datetime.strptime(state_change_time, "%Y-%m-%dT%H:%M:%S.%fZ")
    korea_time = utc_time.replace(tzinfo=timezone.utc).astimezone(timezone(timedelta(hours=9)))
    korea_time_str = korea_time.strftime("%Y-%m-%d %H:%M:%S")

    slack_message = {
        'channel': SLACK_CHANNEL,
        'text': (
            "*알람 이름*: %s\n"
            "*상태*: %s\n"
            "*원인*: %s\n"
            "*발생 시각*: %s (한국 시간)"
        ) % (alarm_name, new_state, reason, korea_time_str)
    }

    req = Request(HOOK_URL, json.dumps(slack_message).encode('utf-8'))
    try:
        response = urlopen(req)
        response.read()
        logger.info("Message posted to %s", slack_message['channel'])
    except HTTPError as e:
        logger.error("Request failed: %d %s", e.code, e.reason)
    except URLError as e:
        logger.error("Server connection failed: %s", e.reason)

```

![image](https://github.com/user-attachments/assets/f2b06bcd-dccc-4576-8b15-1fb74403345f)

코드를 바꿨으니 환경변수도 바꿔줘야 한다. 

![image](https://github.com/user-attachments/assets/1418d860-d906-4e71-ae06-47e11cbd0590)

#### **3-4 AWS Lamda 테스트**

**아래 테스트 탭에 들어가 이벤트 JSON 아래 코드를 복붙해주고 테스트버튼을 누르면 된다.** 

![image](https://github.com/user-attachments/assets/3b71e0f0-3e13-4e5c-bb6c-5ecf7ba9d0a7)

```
{
  "Records": [
    {
      "EventSource": "aws:sns",
      "EventVersion": "1.0",
      "EventSubscriptionArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "Sns": {
        "Type": "Notification",
        "MessageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "TopicArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms",
        "Subject": "ALARM: \"Example alarm name\" in EU - Ireland",
        "Message": "{\"AlarmName\":\"Example alarm name\",\"AlarmDescription\":\"Example alarm description.\",\"AWSAccountId\":\"000000000000\",\"NewStateValue\":\"ALARM\",\"NewStateReason\":\"Threshold Crossed: 1 datapoint (10.0) was greater than or equal to the threshold (1.0).\",\"StateChangeTime\":\"2017-01-12T16:30:42.236+0000\",\"Region\":\"EU - Ireland\",\"OldStateValue\":\"OK\",\"Trigger\":{\"MetricName\":\"DeliveryErrors\",\"Namespace\":\"ExampleNamespace\",\"Statistic\":\"SUM\",\"Unit\":null,\"Dimensions\":[],\"Period\":300,\"EvaluationPeriods\":1,\"ComparisonOperator\":\"GreaterThanOrEqualToThreshold\",\"Threshold\":1.0}}",
        "Timestamp": "2017-01-12T16:30:42.318Z",
        "SignatureVersion": "1",
        "Signature": "Cg==",
        "SigningCertUrl": "https://sns.eu-west-1.amazonaws.com/SimpleNotificationService-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.pem",
        "UnsubscribeUrl": "https://sns.eu-west-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "MessageAttributes": {}
      }
    }
  ]
}
```

슬랙에 잘 전송된 것을 확인할 수 있다.

![image](https://github.com/user-attachments/assets/97d651c5-18ce-4e6f-ad36-34ada6beb676)

## **4\. AWS CloudWatch 생성**

![image](https://github.com/user-attachments/assets/ce5130fa-a318-4ad2-83c3-8aef88c98c69)
![image](https://github.com/user-attachments/assets/b0c3771b-7f22-4478-8201-1b9d6e1d1c37).
![image](https://github.com/user-attachments/assets/2096a51c-9e4b-4621-a22a-7c94871af015)

Ec2에서 추적할 지표 선택

![image](https://github.com/user-attachments/assets/8a32ef6f-f884-4d7c-ae0c-a93b1d889b72)

트리거할 기준선 설정.

난 1분 평균 cpu사용률리 80이 넘을 때 알림을 보내게 설정했다.

![image](https://github.com/user-attachments/assets/521944b7-5661-4307-9320-c1c1bde3203c)

알림 보낼 AWS SNS 선택

![image](https://github.com/user-attachments/assets/93706047-7443-421f-9eda-480449584fd6)

끝!

---

## **#아쉬웠던 점**

AWS CloudWath, SNS, Lamda, KMS 등 처음 접해보는 서비스였다. 그랬기에  시간비용을 생각하여 인터넷에 자료가 많이 나와있는 방식대로 진행하였다. 나와있는 자료중 대부분이 

CloudWatch에서 알람 발생 → SNS 푸시 서비스 호출 → Lambda 함수 트리거(환경변수를 위해 KMS 사용) → 연동된 Slack 채널로 알람 전송

**위와 같았다. 그러나 난 KMS 사용이 불필요하다 생각하여 Lamda코드를 수정하여 KMS 서비스 사용을 설정하지 않았다. KMS설정 비용을 단축시킨 것이다. 이 부분은 잘했다고 생각이 든다.**

그런데 위 과정을 다 끝내고 나서 위 흐름과 각 서비스에 대한 이해도가 올라가 AWS SNS가 중간에 불필요하다는 것을 느꼈다. 다시 말하자면,

CloudWatch에서 알람 발생 → Lambda 함수 호출 → 연동된 Slack 채널로 알람 전송

위 과정으로 단축시킬 수 있었다는 것이었다. AWS SNS는 메신저 구독/발행 느낌으로 필터링 및  여러 구독자에게 전달 등과 같은 역할을 해줄 수 있지만, 우리 프로젝트에서는 아직까지 이런 세세한 메시지 송신 조절이 필요없었다. 더군다나 필터링 필요하다면 AWS Lamda에서 코드를 수정하면 된다.

이미 오버엔지어링을 하였기에 다음과 같이 간략한 과정으로 바꾸려면 또 오버엔지어링을 하는 것이다. 그래서 간략한 버전으로 수정하지는 않겠다. 다음에 AWS CloudWatch로 모니터링 도입 업무를 맡게 된다며 위와 같은 간략한 흐름으로 설정하겠다. 

만약 간소화 버전으로 AWS CloudWatch를 설정하고 싶다면 아래 링크를 참조해봐도 좋을 것 같다.

(https://www.smileshark.kr/post/lambda%EC%99%80-%EC%B9%9C%ED%95%B4%EC%A7%80%EB%8A%94-%EC%B2%AB%EA%B1%B8%EC%9D%8C-%EC%9E%A5%EC%95%A0-%ED%83%90%EC%A7%80-%EB%A7%8C%EB%93%A4%EA%B8%B0)
