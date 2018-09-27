---
title: "[django] 장고 트랜잭션 활용"
date: 2016-01-16 23:31:45
tags:
- python
- django
- transaction
---
최근에 python + django로 프로젝트를 진행하고 있습니다. 두달전부터 시작했으니 아직은 python, django 모두 초보라 할 수 있겠죠. 프로젝트를 같이 진행하는 동료들도 저 처럼 거의 저와 비슷한 상황입니다. 그러다보니 프로젝트를 진행하면서 막히는 부분이 생길때마다 검색을 해보거나 파이썬 콘솔을 이용하여 검증, 또는 소스를 뜯어보며 작동원리를 파악하곤 합니다. 이 삽질과도 같은 수많은 과정들이 한순간에 끝나버리는게 아쉬워 블로그에 남겨보기로 결정했습니다. 시간이 남아돌아 글을 쓰는것은 아니지만, 글을 쓰는 행위는 어렴풋이 이해하고 있는 것들을 확실히 나의 지식으로 만드는 최고의 방법이라 생각하기에 시간을 쪼개서 지속해 나갈 생각입니다. 여러가지 부분을 다루겠지만, 체계 따위는 없습니다. 그때 그때 떠오르는 것, 또는 프로젝트 진행중에 특정부분이 막혀 정리가 필요한 시점에는 그 부분을 다루려고 합니다.

첫번째는 트랜잭션(transaction)입니다. 트랜잭션이 뭐냐고 물었을때 대부분의 프로그래머가 잘 알고 있을거라 생각합니다. 간단히 말해 작업 단위라 할 수 있는데, 여러개의 프로세스가 묶여져 마치 하나처럼 동작하는 방식이라 할 수 있겠습니다. 그렇기 때문에 성공 아니면 실패 두가지 결과밖에 존재하지 않겠죠. 좀더 개발적(?)인 언어로는 이렇게 말할수도 있습니다. 데이터베이스를 저장하고 수정하는 여러 작업을 하나의 쿼리로 처리할 수 없기 때문에 여러개의 쿼리로 나눠서 실행해야 하는데, 그 결과의 원자성을 보장해주기 위해 동일한 DB connection 객체를 사용하는 기술이라 할 수 있습니다. 자세한 내용은 직접 검색해 보시기 바랍니다. 


> *django는 1.8.5 버전을 사용하였고, python은 3.4.3 버전을 사용하였습니다.*


### 1. 데코레이터(decorator)를 이용한 python + django 트랜잭션 

django에서 트랜잭션을 이용하는 가장 쉬운 방법으로 데코레이터를 이용하는 방법입니다.
데코레이터를 이용하게 되면, 메서드 안에는 코드를 삽입할 필요가 없습니다. 
"@transaction.atomic" 이라는 데코레이터를 붙여주기만 하면 끝입니다.
django에서 기본적으로 제공해주는 데코레이터이므로, 따로 모듈을 설치해줄 필요도 없습니다.

가장 간단하게 atomic(원자성)한 트랜잭션을 처리하기 위한 손쉬운 방법이죠.

```
from django.db import transaction

@transaction.atomic
def transaction_test1(arg1, arg2):
    # start transaction
    a.save()
    
    b.save()
    # end transaction
```


### 2. with 명령어를 이용한 트랜잭션

메서드 전체가 아닌 메서드의 일부분만 트랜잭션으로 묶어줄 필요가 있을 때 사용합니다.
트랜잭션으로 묶일 부분을 직접 지정해줘야 하는 불편함(?)이 있지만, 데코레이터와 마찬가지로 비교적 간단하게 처리가 가능합니다.

```
from django.db import transaction

def transaction_test2(arg1, arg2):
    
    a.save()    # 항상 save 처리됨, 예외가 발생할 경우 에러 발생
    
    with transaction.atomic():
        # start transaction
        b.save()

        c.save()
        # end transaction
```


### 3. savepoint를 직접 지정해 주는 트랜잭션
 
1번과 2번의 방법의 경우, 메서드 내에서(트랜잭션으로 묶여져있는) exception이 발생하더라도 저절로 롤백이 되기 때문에 예외처리를 따로 해 줄 필요는 없습니다. 
하지만, 3번의 경우에는 savepoint 및 cummit 지점을 직접 지정해 주기 때문에 예외처리 또한 별도로 처리되어야 합니다.

```
from django.db import transaction

def transaction_test3(arg1, arg2):

    a.save()

    sid = transaction.savepoint()
    # start transaction
    try:

        b.save()
        
        c.save()

        transaction.savepoint_commit(sid)
        # end transaction
    except Exception
        # 트랜잭션 내에서 에러 발생시 롤백처리
        transaction.savepoint_rollback(sid)
```


위의 세가지가 django에서 트랜잭션을 처리하는 방법입니다. 간단한 트랜잭션 함수를 호출할 때는 위의 세가지 중에 알맞은 방법을 선택해서 처리하면 되겠습니다. 좀 더 복잡한 상황을 한번 살펴보도록 하죠.
 
```
from django.db import transaction

def method1():
    
    with transaction.atomic():

        # 첫번째 트랜잭션 메서드
        first_transaction()
        
        # 두번째 트랜잭션 메서드
        second_transaction()
        

@transaction.atomic
def first_transaction():
    
    a.update()
    
    b.update()


@transaction.atomic
def second_transaction():
    
    c.update()
    
    d.update()
```


위의 경우에는 트랜잭션 메서드 두개를 묶어서 또다른 트랜잭션을 구성하였습니다.
각각의 메서드는 독자적읜 트랜잭션이기 때문에 각 메서드 내에서 에러가 발생하더라도 데이터의 원자성은 유지될 것입니다. 
문제는 first_transaction() 메서드는 아무 문제 없이 commit 되었는데, second_transaction() 메서드에서 예외가 발생했을 때입니다.

이렇게 되면, 첫번째 트랜잭션만 처리가 되고 두번째는 롤백이 됩니다. 
의도된 로직이면 아무 문제 없겠지만, 두 트랜잭션 메서드가 하나의 트랜잭션으로 묶여 있어야 하는 상황이라면 엄청난 문제가 발생할 수도 있습니다. 
데이터의 원자성을 유지하기 위해서는 first_transaction()과 second_transaction()을 묶어주는 또다른 트랜잭션을 만들어 줘야 합니다. 위에서는 with transaction.atomic(): 으로 묶어 줬지만, 데코레이터를 써도 상관은 없습니다. 다만, 가장 상위의 메서드가 django의 view에 해당하는 경우가 많기 때문에 불필요한 부분까지 같이 묶어주고 싶지 않아서 with 구문을 이용한 것입니다.


### 결론:
> - django 트랜잭션은 쉽다.
> - 트랜잭션을 겹쳐서 사용할 경우, 주의해서 사용해야 한다. (반드시 테스트 해볼것!!) 
