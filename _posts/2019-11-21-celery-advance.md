---
title: "Python Celery"
date: 2019-11-21 00:00:00 +0900
categories: python async queue celery prefetch
---

## (Python)Celery?

파이썬을 이용해서 긴 작업을 비동기로 처리하려고 하는 경우, 가볍게 thread나 subprocess 이용해서 처리하는 경우도 있지만, 대부분의 경우 Celery를 선택하게 됩니다.

> Celery는 파이썬으로 작성된 비동기 작업 큐 입니다. 작업(**Task**)를 중개자(**Broker**)로 전달하면 하나 이상의 작업자(**Worker**)가 작업을 받아서 처리하는 구조입니다.
>
> 파이썬에서 비동기 작업을 처리하는 가장 일반적인 해결책입니다.

## Celery의 동작 방식

Celery는 기본적으로 작업(**Taks**)을 **Prefetch** 방식으로 가져와서 처리합니다.

작업자(Worker)가 미리 처리하려는 작업 특정 사이즈만큼 미리 가져와서 대기 시켜놓고, 작업이 진행 중인 작업이 완료되면 중개자(Broker)로 요청을 보내지 않고 미리 대기시켜두었던 작업(Task)를 꺼내와서 처리하는 방식입니다.

worker_prefetch_multiplier 옵션으로 통해서 몇 개의 작업을 미리 가져와서 대기시켜 놓을지 결정합니다.(기본값 4)

([worker_prefetch_multiplier 옵션](http://docs.celeryproject.org/en/latest/userguide/configuration.html?highlight=celeryd_prefetch_multiplier#worker-prefetch-multiplier))

## Prefetch 이점

Prefetch 방식으로 처리할 작업을 한번에 가져와서 미리 Worker의 메모리로 옮겨놓기 떄문에 실행 시 Broker와 통신하는 횟수를 줄일 수 있는 이점이 있습니다.

따라서 많은 갯수의 단기 작업들로 이루어진 worker인 경우 한번에 prefetch를 통해서 가져오는 messge 수를 크게 잡을 수록 네트워크 지연시간과 실행시간에서 이득을 볼 수 있습니다.

적절한 값은 실행할 worker의 갯수와 thread를 사용하는 경우 thread의 수 등의 상황에 따라 다를 수 있기 때문에 비슷한 환경의 Task를 만들어서 실험을 해봐야 알 수 있습니다.

## Prefetch 단점

반면에 Prefetch를 통해서 오히려 처리 성능의 손해를 보는 상황도 발생할 수 있습니다.

Worker에서 처리하는 Task들의 실행 시간이 불규칙하고, 긴 작업들이 섞여있는 경우는 Prefetch 방식으로 미리 Worker의 메모리에 할당된 후 실행되기까지 긴 시간이 필요할 수 있고, 경우에 따라서는 나중에 Broker로 전달되어서 다른 Worker에 할당된 Task보다 더 늦게 실행되는 경우도 발생할 수 있습니다.

이 경우 가장 좋은 핵결책은 긴 시간의 Task와 짧은 시간의 Task를 처리하는 Worker를 분리하는 것입니다.

분리하는 방법이 힘들다면 prefetch를 사용하지 않는 것이 더 효율적일 수도 있습니다.

## Prefetch 옵션 최적화

참고: [Celery Optimize](http://docs.celeryproject.org/en/latest/userguide/optimizing.html#optimizing-prefetch-limit)

### worker_prefetch_multiplier

worker_prefetch_multiplier 옵션의 기본값은 4로 설정되어있습니다. 일반적인 경우 이 값을 조정할 필요가 없습니다.

- worker_prefetch_multiplier = 1

worker_prefetch_multiplier 값을 1으로 설정하게되면 하나의 Worker에 하나의 작업만 대기시켜 놓습니다. (**If you have many tasks with a long duration you want the multiplier value to be one: meaning it’ll only reserve one task per worker process at a time.**)

이 설정이 document 상에서는 prefetch를 disable하는 방법이라고 설명하고 있습니다.

- worker_prefetch_multiplier = 0

Celery 문서를 제대로 읽지 않은 경우 가장 착각하기 쉬운 설정입니다.

worker_prefetch_multiplier 값을 0으로 바꾸게되면 worker의 상황을 고려하지 않고 계속 Task를 가져와서 대기시켜 놓습니다.
(**If it is zero, the worker will keep consuming messages, not respecting that there may be other available worker nodes that may be able to process them sooner, or that the messages may not even fit in memory.**)

일반적으로 0으로 설정하면 prefetch하지 않고 실행할 수 있는 Worker에 Task가 바로 할당될 것으로 기대하기 쉬운데 그렇게 동작하지 않습니다.

오히려 처리 가능한지 여부를 확인하지 않고 마구잡이로 Task가 Worker로 할당되기 때문에 더 비효율적으로 동작하는 경우가 대부분입니다.

### -O fair

Prefetch disabling

Configuration 목록에 자세하게 표시되어있지 않아서 알기 힘들 옵션 중에 하나입니다.

O fair 옵션을 전달하게되면 worker에 prefetch하지 않고 실행 시점에 실행가능한 worker로 바로 할당을 합니다. 따라서 작업들의 실행 시간이 제각각인 경우에도 최적화된 실행을 할 수 있습니다.

### task_acks_late

기본설정으로 worker 가 broker로 부터 메시지를 수신한 후 실행되기 전에 broker로 ack를 날려서 실행될 것이다라는 것을 알려주고 broker에서 해당 메시지를 제거하는 식으로 동작합니다.

이 옵션을 true로 설정하면, ack 신호를 worker가 작업을 끝낸 후 전달하게 됩니다. 이 경우 작업이 실행 중 또는 대기 중에 어떤 이유로 작업이 실패하고 인스턴스가 죽어버린다던가하는 상황이 발생했을 때 이 후에 해당 작업을 재시작할 수 있는데, 이런 경우에 작업이 실행 중에 중지되었을 수 있기 때문에 작업의 멱등성이 보장되는 것이 안전합니다.
