---
title: "Postman에서 AWS SQS로 Message 보내기"
excerpt: "Postman에서 AWS SQS로 Message 보내기"

categories:
  - AWS
  - Tool
tags:
  - [ AWS, SQS, Postman ]

permalink: /categories/cloud/aws/postman-send-message-to-aws-sqs

toc: true
toc_sticky: true

date: 2023-09-03
last_modified_at: 2023-09-03
---

AWS SQS를 사용한 테스트시 AWS SQS 콘솔에 접속해서 메시지 전송 및 수신기능을 사용해서 해당 Queue에 Message를 보낼 수 있지만 Postman을 사용하면 더 편리하게 보낼 수 있다.

## 1. API EndPoint

요청할 API EndPoint는 `protocol://service-code.region-code.amazonaws.com`이 규약이며, HTTP Protocol을 사용해서 ap-northeast-2(Seoul)
리전에 SQS로 Message를 보낼 경우 API EndPoint는 `https://sqs.ap-northeast-2.amazonaws.com` 이 된다.

> AWS
> service-code는 [서비스 엔드포인트 및 할당량 - AWS 일반 참조](https://docs.aws.amazon.com/ko_kr/general/latest/gr/aws-service-information.html)
> 문서를 참고하였으며, SQS API EndPoint 관련 상세
> 정보는  [Amazon 심플 큐 서비스 엔드포인트 및 할당량 - AWS 일반 참조](https://docs.aws.amazon.com/ko_kr/general/latest/gr/sqs-service.html) 문서를
> 참고하였다.

## 2. Authorization

API 호출시 사용할 인증 정보는 다음과 같다.

1. Type은`AWS Signature`로 지정한다.
2. AccessKey 및 SecretKey에는 `AWS IAM에서 발급받은 access key 및 secret access key`를 입력한다.
    - 해당 Access Key를 가진 사용자가 AWS SQS 관련 권한이 있어야 SQS에 Message 전송시 문제가 없다.
3. AWS Region은 `사용중인 리전`을 입력한다.
4. Service Name에는 `sqs`를 입력한다.

![](/assets/images/posts_img/2023-09-03-Postman에서-AWS-SQS로-Message-보내기/image.png)

## 3. HTTP Header

- x-amz-target : API 버전과 요청하는 Action을 지정하는 Header로 여기서는 SQS로 Message를 보내는 Action이기 때문에 `AmazonSQS.SendMessage`로 지정하였다.
- Content-Type : `application/x-amz-json-1.0`

![](/assets/images/posts_img/2023-09-03-Postman에서-AWS-SQS로-Message-보내기/image2.png)

> x-amz-target 관련
> 내용은  [Making API requests - AWS Transfer Family](https://docs.aws.amazon.com/transfer/latest/userguide/making-api-requests.html)
> 문서를 참고하였으며, SQS
> Action은 [Actions - Amazon Simple Queue Service](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_Operations.html)
> 문서를 참고하였다.

## 4. Request Body

* QueueUrl : Message를 보낼 Queue의 전체 URL 이다.
* MessageBody : 보낼 Message 이다.

다음 요청은 hello라는 SQS에 `"{\iame\": \"hun\"\"}"` 메시지를 보내는 Request Body 이다.

```json
{
  "QueueUrl": "https://sqs.ap-northeast-2.amazonaws.com/474524867563/hello",
  "MessageBody": "{\"name\": \"hun\"\"}"
}
```

> 위의 속성외에도 다양한 속성이 있으니 자세한
> 내용은 [SendMessage - Amazon Simple Queue Service](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html)
> 문서를 참고하면 된다.

요청에 문제가 없다면 다음처럼 message의 고유 ID 값인 MessageId를 Response로 받는다.

![](/assets/images/posts_img/2023-09-03-Postman에서-AWS-SQS로-Message-보내기/image3.png)

만약 Standard Queue가 아닌 FIFO Queue일 경우 MessageGroupId가 필수인데 Postman에 있는 Globally Unique Identifier인 `guid` 를 사용하면 요청시 마다
다른 난수가 부여되므로 편리하게 사용할 수있다.

![](/assets/images/posts_img/2023-09-03-Postman에서-AWS-SQS로-Message-보내기/image5.png)

이제 AWS SQS에 가서 메시지 폴링을 하면 해당 Message가 SQS에 전송된 것을 알 수 있으며 Response시 받은 MessageId와 메시지에 있는 id가 동일한 것을 알 수 있다.

![](/assets/images/posts_img/2023-09-03-Postman에서-AWS-SQS로-Message-보내기/image4.png)