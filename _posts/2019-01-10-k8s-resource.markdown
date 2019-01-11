---
layout: post
title: k8s Resource Policy 
date: 2019-01-06
categories: Kubernetes
tags: [A kubernetes, resource, quota]
author: himang10
description: Resource Quota
---
Resource Quota
============

# Table of Contents
1. [xxx](#xxx)
2. [xxx](#xxx)
3. [xxx](#xxx)


## Pod가 Node에서 실행 시 Scheduling 결정 방법
> scheduler가 스케쥴링할 때만다 개별 리소스가 정확히 얼마나 많이 사용되고 있는지 보지 않고 노드에 배포된 기존 포드가 요청한 리소스의 합께만을 확인해서 할당
> 그러므로 실제 자원이 남아 있더라도 전체 총량에서 request 총량을 뺀 후 나머지는 현재 request보다 작으면 할당 실패

## Pod를 위한 최상의 노드를 선택할 때 스케쥴러가 포드의 요청을 사용하는 방법
자원 기준 노드 선택 우선 순위 방식은 두가지 중 한 가지 방식으로 처리함
* `LeastRequestedPriority` - 요청된 resource가 더 적은 노드 (아직 할당되지 않은 리소스가 더 많은)를 우선 배정
* `MostRequestedPriority` 
   요청된 resource가 더 많은 노드 (할당되지 않은 리소스가 더 적은)를 우선 배정 
   이방식은 이미 자원이 할당되어 있는 워크 노드를 할당 하므로 결국 최소 수의 노드를 사용하도록 보장하는 방식
   포드를 강하게 Pack하여 유지하면 특정 노드가 비어 있게 되고 제거가 쉬워 질 수 있음. 개별 노드에 대한 비용을 지불하므로 비용 절감 효과가 있음

## Conatiner에 사용 가능한 리소스 양 한계 설정
1. 리소스 한계 포드 설정
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: limited-pod
spec:
  containers:
  - image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: main
    resources:
      limits:
        cpu: 1
        memory: 20Mi
````
>> 리소스 request를 지정하지 않았으므로 리소스 Limit와 동일한 값으로 설정된다

2. 한계 오버커밋 (over-committed)
리소스 한계는 노드의 할당 가능한 리소스 양에 의해 제한되지 않는다. 그러므로 모든 포드의 모든 한계의 합이 ***노드의 용량의 100%를 초과***할 수 있다.
>>결국, 이것은 노드가 100% 소모되면 특정 컨테이너를 제거해야 한다.
                            