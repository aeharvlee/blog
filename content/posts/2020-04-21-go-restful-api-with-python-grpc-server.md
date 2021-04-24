---
title: "Go RESTful API(gRPC Client) + Python gRPC server"
date: 2020-04-21
---

Go와 Python을 활용한 마이크로 서비스 아키텍처 활용 사례를 공유드립니다.

![gopher_looking_at_python](/images/2020-04-21-go-restful-api-with-python-grpc-server/0.png)

## 1. Introduction

얼마 전 런칭한 클라우드브릭의 Threat Intelligence API는 **Go RESTful API** (gRPC Client)와 **Python gRPC Server** (with pandas)로 만들어졌습니다.

본 글에서는 왜 **Go RESTful API**와 **Python gRPC Server**의 조합으로 서비스를 만들었는지를 공유하고자 합니다. :eyes:
비슷한 고민을 하시는 분들께 도움이 되었으면 좋겠습니다. :pray:

## 2. Design

### 2.1. Requirements

Threat Intelligence API를 개발할 때, 아래와 같은 요구사항이 있었습니다.

1. **가볍고 빨라야 한다.**
   서버의 리소스를 적게 사용하면서 사용자들의 요청을 빠르게 처리해야 합니다. **(Cost Efficiency)** :money_with_wings:
2. 실시간으로 위협 정보가 업데이트되는 클라우드브릭의 DB에서 
   **클라이언트가 원하는 데이터를 빠르게 연산하여 즉각 전달**해주어야 합니다.
   **(Fast and flexible data analysis and manipulation)** :open_hands:

1번의 요구사항을 충족하기 위해 저는 Go를 선택했습니다.  사용해보니 실제로 사용하는 리소스도 매우 적고 속도도 매우 빨랐습니다.  선택하길 정말 잘했다는 생각이 듭니다. :slightly_smiling_face:

API를 설계할 때 사실상 제일 중요했던 2**번 요구사항**은 저의 발목을 잡았습니다. :poultry_leg:

Go를 사용하고 싶었지만, Go만으로 2번 요구사항을 만족하는 것이 쉽지 않았기 때문입니다.

클라우드브릭 데이터베이스에 **매일 쌓이는 수많은 로우데이터에서** 클라이언트의 요청에 맞는 데이터만 쏙쏙 추출해줘야하는데, **쿼리만으로는 이 문제를 해결하기가 어려웠습니다.** (데이터 구조도 복잡하고 무엇보다 이 서비스만을 위해 만들어진 디비 스키마가 아니었습니다. 그래서 상당히 많은 조작을 필요로 했습니다.)

복잡한 조건을 가진 쿼리를 많이 요청해야 했고 속도 또한 느렸습니다. 
복잡한 조건의 쿼리를 많이 요청하면**, 디비에 부하가 심해지는 문제가 있었고** 무엇보다 **속도가 느린 게 가장 치명적이었습니다.** :crying_cat_face:

결국 저는 2번 요구사항을 충족하기 위해 “Python의 **pandas 라이브러리**를 사용해서 문제를 해결하자!”라는 결론을 도출하게 되었습니다.

pandas 라이브러리를 사용하면 별도의 복잡한 조건 없이 **SELECT 쿼리만으로 필요한 데이터를 가져와서 메모리에 로드하고, 원하는 결과를 빠르게 도출할 수 있습니다.**

복잡한 조건의 쿼리를 여러 번 디비서버에 요청할 필요가 없으니 **디비 서버에 부하는 줄고,** 더군다나 **원하는 데이터를 추출하는 속도도 빠르다는 점**에서 **pandas**는 정말 사랑스러운 라이브러리가 아닐 수 없습니다. :green_heart:

### 2.2 Design

Go와 Python을 함께 사용하기로 했으니 이제는 설계를 할 차례입니다. :muscle:

![architecture_go_with_python](/images/2020-04-21-go-restful-api-with-python-grpc-server/1.png)

저는 위와 같이 설계를 했고 각 컴포넌트(파란 박스)의 역할은 다음과 같습니다.

#### 2.2.1. Go RESTful API

> Echo Framework를 사용하였으며 [Go Clean Architecture](https://github.com/bxcodec/go-clean-arch)를 참조하여 프로젝트를 구성하였습니다.

**Role**

- 복잡한 연산을 요구하지 않는 간단한 쿼리를 필요로하는 클라이언트의 요청을 직접 처리합니다.
- 복잡한 연산을 요구하는, 즉 간단한 쿼리만으로 해결되지 않고 많은 양의 데이터 처리를 해야하는 작업은 **Python gRPC 서버**에게 요청합니다. 
  **(gRPC Client의 역할)**

#### 2.2.2. Python gRPC Server

**Role**

- **Go RESTful API**로부터 받은 요청을 처리합니다.
  많은 양의 데이터를 **pandas** 라이브러리로 효과적으로 빠르게 처리하고 grpcClient인 Go API에게 추출한 결과 데이터를 반환해줍니다.

이렇게 구성하여 저는 Go와 Python의 이점을 둘 다 누릴 수 있습니다.

## 3. Implementation

### 3.1. Protocol Buffers (= protobuf)

**gRPC 서비스**는 기본적으로 [Protocol Buffers](https://developers.google.com/protocol-buffers)라는 특별한 자료구조를 사용하여 통신을 합니다.

**gRPC**를 사용하길 정말 잘했다고 생각한 부분은, **protobuf**를 사용하기 위해 개발자가 따로 코딩을 할 필요가 없다는 점입니다. 👀

`.proto` 파일을 정의하고 **protoc**(= Protocol Buffers Compiler)로 적절한 옵션을 주어 원하는 언어로 컴파일하면, **probouf**에 정의한 데이터를 주고 받을 수 있는 **인터페이스 코드를 뚝딱 만들어줍니다.** :clap::clap:

제가 필요한 건, Go와 Python으로 **protobuf**를 다룰 수 있는 **인터페이스 코드**이니 아래와 같이 protoc를 사용해서 컴파일을 해줍니다.

```shell
# If you installed protoc using Python...
python -m grpc_tools.protoc --proto_path=./protos \ 
--python_out=./gen/python \ 
--grpc_python_out=./gen/python \ 
./protos/black_ipv4.proto

# Or if you install just plain protoc
protobuf.protoc --go_out=plugins=grpc:./gen/go/ protos/black_ipv4.proto
```

위와 같이 컴파일을 해주면 자신이 정의한 `.proto`를 다룰 수 있는 인터페이스 코드를 Python과 Go 언어로 생성할 수 있습니다. :fried_egg:
개발자가 할 일은 단지 생성된 코드를 import 해서 사용하기만 하면 됩니다. :clap::clap:

### 3.2. Lets Use protobuf

구체적으로 gRPC 클라이언트인 **Go RESTful API**가 어떻게 **Python gRPC 서버**에게 연산을 요청하는지를 `.proto` 파일과 함께 살펴봅시다. :star:

더불어 **Python gRPC서버**가 gRPC 클라이언트인 Go API에게 어떤 데이터를 제공해주는지도 함께 살펴보겠습니다.

{{< gist aeharvlee 4e737aed3b245a17a3b95e815130a0e5 >}}

Threat Intelligence API에는 **특정 블랙 아이피에 대한 평판정보(Reputation)를 조회할 수 있는 기능이 있습니다.**
이 기능은 복잡한 데이터 연산을 요구하는 작업이여서 Python gRPC 서버가 처리하고 있습니다.

27라인의 `message ReputationRequest`을 보면 클라이언트가 gRPC 서버에게 요청하는 **요청 정보**가 정의되어 있습니다.

이 요청 정보는 Threat Intelligence API를 사용하는 사용자가 직접 옵션으로 주는 값입니다. 
Go RESTful API는 이 요청정보를 그대로 Python gRPC 서버에게 전달해주기 때문에 별도로 정의를 해둔 것입니다.

35라인에 정의되어 있는 `message Reputation`은 gRPC 서버가 요청을 처리 한 뒤,
gRPC 클라이언트에게 반환하는 자료구조입니다.

### 3.3. gRPC

Python gRPC가 처리하는 서비스에 대한 정의는 5라인에 `service BlackIPv4Service`로 정의되어 있습니다. (예시를 위해 한 가지 rpc만 정의해두었습니다.)

매개변수로 `ReputationRequest`를 받고 있다는 걸 확인할 수 있습니다.
즉, Go gRPC client가 Python gRPC서버에게 요청할 때는 
해당 자료형에 맞게 요청해야한다는 걸 알 수 있습니다.

{{< gist aeharvlee 8abc65f000b54cd0c0feff71370ea10a >}}

4 ~ 9라인에 그 내용이 기술되어 있습니다. 사용자들이 요청한 정보(`rr *model.ReputationRequest`)를 RPC 호출시 그대로 사용하고 있는 것을 확인하실 수 있습니다. :white_check_mark:

`black_ipv4.proto`의 27라인에 기재되어 있는 `ReputationRequest`의 포맷에 맞춰 gRPC 서버에게 요청하고 있다는 것을 보시면 됩니다. :slightly_smiling_face:

{{< gist aeharvlee d08f58137153f75b301df2e2c3c41185 >}}

**Pyhton gRPC 서버**는 gRPC 클라이언트가 요청한 데이터를 `request`로 받고 `get_reputation_by_ipv4(cnx, request)` 를 통해 처리한 뒤 결과 `reputation`을 **gRPC 클라이언트**에게 리턴해줍니다. :tada:

gRPC 클라이언트인 Go RESTful API 서버는 해당 데이터를 그대로 클라이언트에게 전달만 해주면 끝입니다. :smile:

## 4. Summary

문제해결을 위해 다양한 언어가 필요하다면, 대부분 Microservice Architecture로 구현을 해야합니다.

**gRPC 통신**은 **Protocol Buffers** 자료구조를 활용하여 Microservice들이 서로 통신하기 매우 편리하고 적합한 환경을 제공합니다.

개발 생산성도 좋고 동작도 잘하니 비슷한 고민을 하고 계시다면 적극적으로 도입을 검토하셔도 좋을 것 같습니다.

마이크로 서비스 아키텍처를 고민하신다면, Protocol Buffers를 사용하는 gRPC 통신으로 서비스를 구현해보시는 건 어떨까요?

읽어주셔서 감사합니다!

