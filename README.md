```text
┌─ thing/samuel-jeong ───────────────────────────────────────────────────┐
│                                                                        │
│  ███████╗  █████╗  ███╗   ███╗ ██╗   ██╗ ███████╗ ██╗                  │
│  ██╔════╝ ██╔══██╗ ████╗ ████║ ██║   ██║ ██╔════╝ ██║                  │
│  ███████╗ ███████║ ██╔████╔██║ ██║   ██║ █████╗   ██║                  │
│  ╚════██║ ██╔══██║ ██║╚██╔╝██║ ██║   ██║ ██╔══╝   ██║                  │
│  ███████║ ██║  ██║ ██║ ╚═╝ ██║ ╚██████╔╝ ███████╗ ███████╗             │
│  ╚══════╝ ╚═╝  ╚═╝ ╚═╝     ╚═╝  ╚═════╝  ╚══════╝ ╚══════╝             │
│                                                                        │
│  backend engineer  ·  from the device edge to the database             │
│                                                                        │
│  [■] connected    mqtt+tls :8883    qos 1    clean session: false      │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

```console
$ aws iot-data get-thing-shadow --thing-name samuel-jeong /dev/stdout | jq .state
{
  "reported": {
    "role":      "backend engineer, end to end",
    "cloud":     "AWS, the whole catalog",
    "focus":     "AWS IoT Core: fleets, shadows, rules, jobs",
    "iac":       "Terraform, modules over copy-paste",
    "delivery":  "CI/CD, every commit is a release candidate",
    "runtime":   "Java, Spring Boot",
    "practice":  "DevOps: you build it, you run it"
  },
  "desired": {
    "manual_steps": 0,
    "pages_at_3am": 0,
    "drift":        "none"
  },
  "delta": {}
}
```

### what is under management

```console
$ terraform state list
module.iot.aws_iot_thing_group.fleet
module.iot.aws_iot_policy.least_privilege
module.iot.aws_iot_topic_rule.telemetry_ingest
module.iot.aws_iot_provisioning_template.fleet_provisioning
module.stream.aws_kinesis_stream.telemetry
module.stream.aws_lambda_function.decoders
module.api.aws_ecs_service.spring_boot
module.data.aws_rds_cluster.primary
module.data.aws_dynamodb_table.device_state
module.ops.aws_cloudwatch_metric_alarm.everything
module.ci.aws_codepipeline.delivery

$ terraform plan
No changes. Your infrastructure matches the configuration.
```

### the path I like to own

```text
  edge                 AWS IoT Core                 backend
 ┌──────────┐  MQTT   ┌─────────────────┐  rules   ┌───────────────┐
 │ sensors  │────────>│ device gateway  │─────────>│ Kinesis · SQS │
 │ gateways │  x.509  │ shadow · jobs   │          │ Lambda        │
 └──────────┘         │ fleet indexing  │          └───────┬───────┘
                      └─────────────────┘                  │
                                                           v
 ┌──────────────────┐        ┌─────────────────────────────────────┐
 │ RDS · DynamoDB   │<───────│ Spring Boot services  (ECS / EKS)   │
 │ S3 · Timestream  │        │ REST · events · batch               │
 └──────────────────┘        └─────────────────────────────────────┘
          all of it described in Terraform, shipped by a pipeline,
          watched by CloudWatch. nothing clicked in the console.
```

### how code leaves my laptop

```console
$ git push origin main
  [1/6] lint ....................................... ok
  [2/6] unit + integration tests ................... ok
  [3/6] build  gradle -> container image ........... ok
  [4/6] terraform plan ............................. 0 to destroy
  [5/6] deploy  blue/green ......................... ok
  [6/6] smoke test + auto rollback ................. armed
  released. nobody was paged.
```

### runtime

```yaml
spring:
  application:
    name: samuel-jeong
  profiles:
    active: production
  threads:
    virtual:
      enabled: true
server:
  shutdown: graceful
management:
  endpoints:
    web:
      exposure:
        include: health, metrics, prometheus
```

### shipped

[**MacMon**](https://github.com/Samuel-Jeong/MacMon) · P-core / E-core aware system monitor for Apple Silicon. Go, cgo, IOReport.

### subscribe

```console
$ mosquitto_sub -t 'samuel-jeong/#' -v
samuel-jeong/status             online
samuel-jeong/now                wiring devices to backends, one topic rule at a time
samuel-jeong/talk-to-me-about   iot core, terraform modules, spring boot, pipelines
samuel-jeong/reach              open an issue, or find me in the commit log
samuel-jeong/lwt                connection lost. probably bumping the aws provider.
```
