---
layout: post
title:  "Evaluating-message-brokers-kafka-vs-kinesis-vs-sqs"
date:   2018-03-23 18:25:01
author: Mars
categories: tools
---

https://www.opsclarity.com/evaluating-message-brokers-kafka-vs-kinesis-vs-sqs/

### Simplicity of SQS

OpsClarity 라는 시스템은 real-time monitoring solution 이기에, 
수집된 데이터는 real-time 으로 처리되어야 한다.

incoming data 는 폭증(spike)할 수 있기에, 
소화량(ingest rate)을 조절(smoothen out)할 필요가 있다.

이것은 일반적으로 queuing layer 가 데이터를 잡고(hold) 있음으로써 해결된다.

처음에는(2014년),
사용하기 쉬운 solution 을 원했었고 빠르게 만들어서 확장할 수 있어야 했다.

우리의 목표는 2개였다:
- 시스템 파이프라인 전체가 고객 별로 분리되어 운영되어야 한다: 한 고객의 데이터가 다른 고객의 대쉬보드에 보여서는 안된다.
- 시스템은 항상 이용가능해야 한다: 한 고객의 데이터가 급증하더라도 다른 고객의 파이프라인은 보호되어야 한다.

첫 느낌은, SQS seemed to get us up and running quickly. 
With that, 
we decided to create separate queues for every customer that came onboard, 
which would also help us control 
which queues we wanted to process on a priority basis, 
in case of a data surge. 
This model worked fine 
when we had a single producer and a single consumer 
computing dimensional aggregations from raw metrics. 
That’s straightforward and every monitoring company does that.


