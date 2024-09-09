## **#모니터링 과정**

[##_Image|kage@cPqYwz/btsJvp9ncKD/lLyc48TWPozwI0qSBRCHK1/img.png|CDM|1.3|{"originWidth":1111,"originHeight":200,"style":"alignCenter"}_##]

CloudWatch에서 알람 발생 → SNS 푸시 서비스 호출 → Lambda 함수 트리거(환경변수를 위해 KMS 사용) → 연동된 Slack 채널로 알람 전송

-   Slack: 채널 기반 메시징 플랫폼. Channel 고유의 WebHook url을 통해 알람 수신
-   CloudWatch: AWS 리소스 및 실행 중인 어플리케이션 모니터링
-   SNS: Publisher ↔ Subscriber 간 통신 채널. 지원되는 프로토콜을 이용해 클라이언트에 메시지 발송
-   Lambda: 서버리스 컴퓨팅 서비스. 서버를 관리하지 않아도 코드를 실행할 수 있게 하는 컴퓨팅 서비스
-   KMS: 데이터 암호화에서 사용되는 암호화 키. 인터넷 전송 시 암호화 통신에 사용  **(사용 안 할 예정. 이유는 밑에서 설명하겠다.**

---

## **#도입 방법**

## **1.슬랙 Webhook 생성**

[##_Image|kage@J5hIm/btsJubkjKQO/iPvpD0dxI4SnRSb8248qKK/img.png|CDM|1.3|{"originWidth":709,"originHeight":665,"style":"alignCenter"}_##]

본 글에서 Webhook 생성 과정은 생략하겠다.

## **2\. Amzons SNS(Simple Notification Service)생성**

[##_Image|kage@cUxK0R/btsJuFkW0D5/MI1Bnz23bgoHbv8XLno4j1/img.png|CDM|1.3|{"originWidth":952,"originHeight":677,"style":"alignCenter"}_##]

## **3\. Amzon Lambda 생성**

#### **3-1 블루프린트 설정**

[##_Image|kage@xvAZc/btsJvXEYptF/D6k5zpyGiMih8cNfbwRjU1/img.png|CDM|1.3|{"originWidth":826,"originHeight":815,"style":"alignCenter"}_##]

#### **3-2 SNS 트리거 설정**

[##_Image|kage@AC2PM/btsJvSXQyPP/vfPXS3MTNo9zLboJWtO5p1/img.png|CDM|1.3|{"originWidth":847,"originHeight":333,"style":"alignCenter"}_##]

#### **3-2 환경 변수 설정**

slackChannel: ec2 모니터링 알림 받을 슬랙 채널명 입력

kmsEncryptedHookUrl: 임시값 test 입력

[##_Image|kage@bJ2WiY/btsJvVAokgV/1E1kr6WTnSDOjRqteKR1yK/img.png|CDM|1.3|{"originWidth":855,"originHeight":414,"style":"alignCenter"}_##]

#### **3-3 WebhookURL 입력하기**

위와 같이설정해주면 미리 정의된 블루 프린트를 사용했기 때문에 아래와 같이 코드가 미리 입력되어있을 것이다. Lamda는 아래 코드와 같이 작동된다. 

[##_Image|kage@WRQPt/btsJwdOekXz/fjjnQV2kEnOgd1JjIp4QnK/img.png|CDM|1.3|{"originWidth":1338,"originHeight":885,"style":"alignCenter"}_##]

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

[##_Image|kage@sST7p/btsJvl64d2D/vfhB0IUos3gn81r7WD8p4k/img.png|CDM|1.3|{"originWidth":1309,"originHeight":569,"style":"alignCenter"}_##]

코드를 바꿨으니 환경변수도 바꿔줘야 한다. 

[##_Image|kage@cFbupI/btsJwBBlygW/NcnKKkgwqClklfvXZkqViK/img.png|CDM|1.3|{"originWidth":1390,"originHeight":873,"style":"alignCenter"}_##]

#### **3-4 AWS Lamda 테스트**

**아래 테스트 탭에 들어가 이벤트 JSON 아래 코드를 복붙해주고 테스트버튼을 누르면 된다.** 

[##_Image|kage@zr4QN/btsJu9sjs3f/2JOk5JdYjA7i9Q54n2gBA1/img.png|CDM|1.3|{"originWidth":1321,"originHeight":709,"style":"alignCenter"}_##]

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

[##_Image|kage@cgH4F7/btsJugMOZ7j/E6MUUN1yGAAAqZ9cO041b0/img.png|CDM|1.3|{"originWidth":707,"originHeight":662,"style":"alignCenter"}_##]

slack으로 전송하려면 Lambda를 사용

## **4\. AWS CloudWatch 생성**

[##_Image|kage@pjIVG/btsJu8UpaTH/MvioKwEpVbC9oM8I5NLpA0/img.png|CDM|1.3|{"originWidth":1147,"originHeight":438,"style":"alignCenter"}_##][##_Image|kage@dpnnvo/btsJvnRmBFA/ukVRbpw2nCxM7rHoKV7o81/img.png|CDM|1.3|{"originWidth":1185,"originHeight":667,"style":"alignCenter"}_##][##_Image|kage@MsURV/btsJvTvGIOO/NKbFw2XKBRdXs8NodIMkaK/img.png|CDM|1.3|{"originWidth":1732,"originHeight":724,"style":"alignCenter"}_##]

Ec2에서 추적할 지표 선택

[##_Image|kage@bytocM/btsJuB3Qgj0/P0k8HhzKfEjzLhfsR9GS10/img.png|CDM|1.3|{"originWidth":929,"originHeight":841,"style":"alignCenter"}_##]

트리거할 기준선 설정.

난 1분 평균 cpu사용률리 80이 넘을 때 알림을 보내게 설정했다.

[##_Image|kage@bY5CH8/btsJvQZ4crA/vt1U6yjgMOjJLvJbFQ6uS1/img.png|CDM|1.3|{"originWidth":924,"originHeight":863,"style":"alignCenter","filename":"blob"}_##]

알림 보낼 AWS SNS 선택

![](https://t1.daumcdn.net/keditor/emoticon/friends1/large/001.gif)

끝!

---

## **#아쉬웠던 점**

AWS CloudWath, SNS, Lamda, KMS 등 처음 접해보는 서비스였다. 그랬기에  시간비용을 생각하여 인터넷에 자료가 많이 나와있는데로 진행하였다. 나와있는 자료중 대부분이 

CloudWatch에서 알람 발생 → SNS 푸시 서비스 호출 → Lambda 함수 트리거(환경변수를 위해 KMS 사용) → 연동된 Slack 채널로 알람 전송

**위와 같았다. 그러나 난 KMS 사용이 불필요하다 생각하여 Lamda코드를 수정하여 KMS 서비스 사용을 설정하지 않았다. KMS설정 비용을 단축시킨 것이다. 이 부분은 잘했다고 생각이 든다.**

그런데 위 과정을 다 끝내고 나서 위 흐름과 각 서비스에 대한 이해도가 올라가 AWS SNS가 중간에 불필요하다는 것을 느꼈다. 다시 말하자면,

CloudWatch에서 알람 발생 → Lambda 함수 호출 → 연동된 Slack 채널로 알람 전송

위 과정으로 단축시킬 수 있었다는 것이었다. AWS SNS는 메신저 구독/발행 느낌으로 필터링 및  여러 구독자에게 전달 등과 같은 역할을 해줄 수 있지만, 우리 프로젝트에서는 아직까지 이런 세세한 메시지 송신 조절이 필요없었다. 더군다나 필터링 필요하다면 AWS Lamda에서 코드를 수정하면 된다.

이미 오버엔지어링을 하였기에 다음과 같이 간략한 과정으로 바꾸려면 또 오버엔지어링을 하는 것이다. 그래서 간략한 버전으로 수정하지는 않겠다. 다음에 AWS CloudWatch로 모니터링 도입 업무를 맡게 된다며 위와 같은 간략한 흐름으로 설정하겠다. 

만약 간소화 버전으로 AWS CloudWatch를 설정하고 싶다면 아래 링크를 참조해봐도 좋을 것 같다.

[https://www.smileshark.kr/post/lambda%EC%99%80-%EC%B9%9C%ED%95%B4%EC%A7%80%EB%8A%94-%EC%B2%AB%EA%B1%B8%EC%9D%8C-%EC%9E%A5%EC%95%A0-%ED%83%90%EC%A7%80-%EB%A7%8C%EB%93%A4%EA%B8%B0](https://www.smileshark.kr/post/lambda%EC%99%80-%EC%B9%9C%ED%95%B4%EC%A7%80%EB%8A%94-%EC%B2%AB%EA%B1%B8%EC%9D%8C-%EC%9E%A5%EC%95%A0-%ED%83%90%EC%A7%80-%EB%A7%8C%EB%93%A4%EA%B8%B0)

 [Lambda와 친해지는 첫걸음 - 장애 탐지 만들기

들어가며: 처음 접하는 AWS Lambda많은 사람들이 AWS를 처음 접하는 서비스로 Lambda를 선택하지 않습니다.AWS Lambda는 서버를 준비(프로비저닝)하거나 관리할 필요 없이 사용할 수 있는 서비스 입니다.

www.smileshark.kr](https://www.smileshark.kr/post/lambda%EC%99%80-%EC%B9%9C%ED%95%B4%EC%A7%80%EB%8A%94-%EC%B2%AB%EA%B1%B8%EC%9D%8C-%EC%9E%A5%EC%95%A0-%ED%83%90%EC%A7%80-%EB%A7%8C%EB%93%A4%EA%B8%B0)
