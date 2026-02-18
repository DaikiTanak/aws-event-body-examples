

## SQS->Lambda
SQSキューからLambdaがポーリングして呼び出されるパターン

```json
{
  "Records": [
    {
      "messageId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "receiptHandle": "AQEB...long-receipt-handle...==",
      "body": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
      "attributes": {
        "ApproximateReceiveCount": "1",
        "SentTimestamp": "1739880000000",
        "SenderId": "123456789012",
        "ApproximateFirstReceiveTimestamp": "1739880000100"
      },
      "messageAttributes": {},
      "md5OfBody": "e99a18c428cb38d5f260853678922e03",
      "eventSource": "aws:sqs",
      "eventSourceARN": "arn:aws:sqs:ap-northeast-1:123456789012:my-queue",
      "awsRegion": "ap-northeast-1"
    }
  ]
}
```

## SNS -> SQS -> Lambda
SNSトピックにサブスクライブしたSQSキューを経由してLambdaが呼び出されるパターン。
SQSレコードの `body` にSNS通知がJSON文字列として入る。

```json
{
  "Records": [
    {
      "messageId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "receiptHandle": "AQEB...long-receipt-handle...==",
      "body": "{\"Type\":\"Notification\",\"MessageId\":\"c3d4e5f6-a7b8-9012-cdef-a12345678901\",\"TopicArn\":\"arn:aws:sns:ap-northeast-1:123456789012:my-topic\",\"Subject\":\"Test Subject\",\"Message\":\"{\\\"key1\\\": \\\"value1\\\", \\\"key2\\\": \\\"value2\\\"}\",\"Timestamp\":\"2026-02-18T12:00:00.000Z\",\"SignatureVersion\":\"1\",\"Signature\":\"EXAMPLE_SIGNATURE==\",\"SigningCertUrl\":\"https://sns.ap-northeast-1.amazonaws.com/SimpleNotificationService-xxxxx.pem\",\"UnsubscribeUrl\":\"https://sns.ap-northeast-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:ap-northeast-1:123456789012:my-topic:abcdef01-2345-6789-abcd-ef0123456789\",\"MessageAttributes\":{\"AttributeKey\":{\"Type\":\"String\",\"Value\":\"attribute-value\"}}}",
      "attributes": {
        "ApproximateReceiveCount": "1",
        "SentTimestamp": "1739880000000",
        "SenderId": "AIDAIT2UOQQY3AUEKVGXU",
        "ApproximateFirstReceiveTimestamp": "1739880000100"
      },
      "messageAttributes": {},
      "md5OfBody": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
      "eventSource": "aws:sqs",
      "eventSourceARN": "arn:aws:sqs:ap-northeast-1:123456789012:my-queue",
      "awsRegion": "ap-northeast-1"
    }
  ]
}
```

`body` をパースすると以下のSNS通知が得られる。

```json
{
  "Type": "Notification",
  "MessageId": "c3d4e5f6-a7b8-9012-cdef-a12345678901",
  "TopicArn": "arn:aws:sns:ap-northeast-1:123456789012:my-topic",
  "Subject": "Test Subject",
  "Message": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
  "Timestamp": "2026-02-18T12:00:00.000Z",
  "SignatureVersion": "1",
  "Signature": "EXAMPLE_SIGNATURE==",
  "SigningCertUrl": "https://sns.ap-northeast-1.amazonaws.com/SimpleNotificationService-xxxxx.pem",
  "UnsubscribeUrl": "https://sns.ap-northeast-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:ap-northeast-1:123456789012:my-topic:abcdef01-2345-6789-abcd-ef0123456789",
  "MessageAttributes": {
    "AttributeKey": {
      "Type": "String",
      "Value": "attribute-value"
    }
  }
}
```

さらに `Message` をパースすると、元のメッセージ本文が得られる。

```json
{"key1": "value1", "key2": "value2"}
```

## S3 object created event -> SNS -> SQS -> Lambda
S3にオブジェクトが作成された際、S3イベント通知 -> SNS -> SQS -> Lambda の順で伝播するパターン。
Lambda側の `body` にはSQSメッセージ本文が文字列として入り、その中にSNS通知がJSON文字列として含まれ、さらにその `Message` フィールドにS3イベントがJSON文字列としてネストされる。

```json
{
  "Records": [
    {
      "messageId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "receiptHandle": "AQEB...long-receipt-handle...==",
      "body": "{\"Type\":\"Notification\",\"MessageId\":\"b2c3d4e5-f6a7-8901-bcde-f12345678901\",\"TopicArn\":\"arn:aws:sns:ap-northeast-1:123456789012:s3-event-topic\",\"Subject\":\"Amazon S3 Notification\",\"Message\":\"{\\\"Records\\\":[{\\\"eventVersion\\\":\\\"2.1\\\",\\\"eventSource\\\":\\\"aws:s3\\\",\\\"awsRegion\\\":\\\"ap-northeast-1\\\",\\\"eventTime\\\":\\\"2026-02-18T12:00:00.000Z\\\",\\\"eventName\\\":\\\"ObjectCreated:Put\\\",\\\"userIdentity\\\":{\\\"principalId\\\":\\\"EXAMPLE\\\"},\\\"requestParameters\\\":{\\\"sourceIPAddress\\\":\\\"127.0.0.1\\\"},\\\"responseElements\\\":{\\\"x-amz-request-id\\\":\\\"EXAMPLE123456789\\\",\\\"x-amz-id-2\\\":\\\"EXAMPLE123/abcdefg\\\"},\\\"s3\\\":{\\\"s3SchemaVersion\\\":\\\"1.0\\\",\\\"configurationId\\\":\\\"testConfigRule\\\",\\\"bucket\\\":{\\\"name\\\":\\\"my-bucket\\\",\\\"ownerIdentity\\\":{\\\"principalId\\\":\\\"EXAMPLE\\\"},\\\"arn\\\":\\\"arn:aws:s3:::my-bucket\\\"},\\\"object\\\":{\\\"key\\\":\\\"example/key.json\\\",\\\"size\\\":1024,\\\"eTag\\\":\\\"d41d8cd98f00b204e9800998ecf8427e\\\",\\\"sequencer\\\":\\\"0A1B2C3D4E5F678901\\\"}}}]}\",\"Timestamp\":\"2026-02-18T12:00:00.000Z\",\"SignatureVersion\":\"1\",\"Signature\":\"EXAMPLE_SIGNATURE==\",\"SigningCertUrl\":\"https://sns.ap-northeast-1.amazonaws.com/SimpleNotificationService-xxxxx.pem\",\"UnsubscribeUrl\":\"https://sns.ap-northeast-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:ap-northeast-1:123456789012:s3-event-topic:abcdef01-2345-6789-abcd-ef0123456789\"}",
      "attributes": {
        "ApproximateReceiveCount": "1",
        "SentTimestamp": "1739880000000",
        "SenderId": "AIDAIT2UOQQY3AUEKVGXU",
        "ApproximateFirstReceiveTimestamp": "1739880000100"
      },
      "messageAttributes": {},
      "md5OfBody": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
      "eventSource": "aws:sqs",
      "eventSourceARN": "arn:aws:sqs:ap-northeast-1:123456789012:s3-event-queue",
      "awsRegion": "ap-northeast-1"
    }
  ]
}
```

`body` をパースすると以下のSNS通知が得られる。

```json
{
  "Type": "Notification",
  "MessageId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "TopicArn": "arn:aws:sns:ap-northeast-1:123456789012:s3-event-topic",
  "Subject": "Amazon S3 Notification",
  "Message": "{\"Records\":[{\"eventVersion\":\"2.1\",\"eventSource\":\"aws:s3\",\"awsRegion\":\"ap-northeast-1\",\"eventTime\":\"2026-02-18T12:00:00.000Z\",\"eventName\":\"ObjectCreated:Put\",\"userIdentity\":{\"principalId\":\"EXAMPLE\"},\"requestParameters\":{\"sourceIPAddress\":\"127.0.0.1\"},\"responseElements\":{\"x-amz-request-id\":\"EXAMPLE123456789\",\"x-amz-id-2\":\"EXAMPLE123/abcdefg\"},\"s3\":{\"s3SchemaVersion\":\"1.0\",\"configurationId\":\"testConfigRule\",\"bucket\":{\"name\":\"my-bucket\",\"ownerIdentity\":{\"principalId\":\"EXAMPLE\"},\"arn\":\"arn:aws:s3:::my-bucket\"},\"object\":{\"key\":\"example/key.json\",\"size\":1024,\"eTag\":\"d41d8cd98f00b204e9800998ecf8427e\",\"sequencer\":\"0A1B2C3D4E5F678901\"}}}]}",
  "Timestamp": "2026-02-18T12:00:00.000Z",
  "SignatureVersion": "1",
  "Signature": "EXAMPLE_SIGNATURE==",
  "SigningCertUrl": "https://sns.ap-northeast-1.amazonaws.com/SimpleNotificationService-xxxxx.pem",
  "UnsubscribeUrl": "https://sns.ap-northeast-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:ap-northeast-1:123456789012:s3-event-topic:abcdef01-2345-6789-abcd-ef0123456789"
}
```

さらに `Message` をパースすると以下のS3イベントが得られる。

```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "ap-northeast-1",
      "eventTime": "2026-02-18T12:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "EXAMPLE"
      },
      "requestParameters": {
        "sourceIPAddress": "127.0.0.1"
      },
      "responseElements": {
        "x-amz-request-id": "EXAMPLE123456789",
        "x-amz-id-2": "EXAMPLE123/abcdefg"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "testConfigRule",
        "bucket": {
          "name": "my-bucket",
          "ownerIdentity": {
            "principalId": "EXAMPLE"
          },
          "arn": "arn:aws:s3:::my-bucket"
        },
        "object": {
          "key": "example/key.json",
          "size": 1024,
          "eTag": "d41d8cd98f00b204e9800998ecf8427e",
          "sequencer": "0A1B2C3D4E5F678901"
        }
      }
    }
  ]
}
```

## Python->SQS->Lambda
PythonコードからSQSにメッセージを発行し、Lambdaでポーリングするパターン

Pythonコード側で `sqs.send_message()` を使ってメッセージを送信した場合、Lambda側で受け取るイベントの `body` にはPythonから送信した `MessageBody` の内容がそのまま文字列として入る。

```python
import boto3
import json

sqs = boto3.client("sqs")
sqs.send_message(
    QueueUrl="https://sqs.ap-northeast-1.amazonaws.com/123456789012/my-queue",
    MessageBody=json.dumps({"key1": "value1", "key2": "value2"}),
    MessageAttributes={
        "CustomAttribute": {
            "DataType": "String",
            "StringValue": "custom-value"
        }
    },
)
```

上記のPythonコードで送信した場合、Lambda側では以下のイベントを受け取る。

```json
{
  "Records": [
    {
      "messageId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "receiptHandle": "AQEB...long-receipt-handle...==",
      "body": "{\"key1\": \"value1\", \"key2\": \"value2\"}",
      "attributes": {
        "ApproximateReceiveCount": "1",
        "SentTimestamp": "1739880000000",
        "SenderId": "AROA3XFRBF23:my-function",
        "ApproximateFirstReceiveTimestamp": "1739880000100"
      },
      "messageAttributes": {
        "CustomAttribute": {
          "stringValue": "custom-value",
          "binaryListValues": [],
          "stringListValues": [],
          "dataType": "String"
        }
      },
      "md5OfBody": "e99a18c428cb38d5f260853678922e03",
      "md5OfMessageAttributes": "d25a6aea97eb8f585bfa92d314504a92",
      "eventSource": "aws:sqs",
      "eventSourceARN": "arn:aws:sqs:ap-northeast-1:123456789012:my-queue",
      "awsRegion": "ap-northeast-1"
    }
  ]
}
```


## Kafka -> Lambda
Amazon MSK または セルフマネージドKafkaからLambdaがトリガーされるパターン。
Lambdaイベントソースマッピングにより、KafkaトピックのレコードがバッチとしてLambdaに渡される。キーと値はBase64エンコードされている。

```json
{
  "eventSource": "aws:kafka",
  "eventSourceArn": "arn:aws:kafka:ap-northeast-1:123456789012:cluster/my-cluster/abcdef01-2345-6789-abcd-ef0123456789-1",
  "bootstrapServers": "b-1.my-cluster.abcdef.c1.kafka.ap-northeast-1.amazonaws.com:9092,b-2.my-cluster.abcdef.c1.kafka.ap-northeast-1.amazonaws.com:9092",
  "records": {
    "my-topic-0": [
      {
        "topic": "my-topic",
        "partition": 0,
        "offset": 1234,
        "timestamp": 1739880000000,
        "timestampType": "CREATE_TIME",
        "key": "bXkta2V5",
        "value": "eyJrZXkxIjogInZhbHVlMSIsICJrZXkyIjogInZhbHVlMiJ9",
        "headers": [
          {
            "headerKey": [104, 101, 97, 100, 101, 114, 86, 97, 108, 117, 101]
          }
        ]
      }
    ]
  }
}
```

`key` と `value` はBase64エンコードされている。デコード例:
- `key`: `bXkta2V5` -> `my-key`
- `value`: `eyJrZXkxIjogInZhbHVlMSIsICJrZXkyIjogInZhbHVlMiJ9` -> `{"key1": "value1", "key2": "value2"}`
