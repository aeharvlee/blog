---
title: "Reverse DNS Lookup Concurrently with Go"
date: 2020-08-20
---

Go Routine을 활용하여 대량의 아이피에 대해 Reverse DNS Lookup을 효과적으로 처리한 사례를 공유드립니다.

![gopher](/images/2020-08-20-even-faster-reverse-dns-lookup-using-goroutines/0.png)

Image Source: https://en.wikipedia.org/wiki/Gopher

## Verify Good Bot

자세한 요구사항을 말씀드리긴 어렵지만, 보유하고 있는 **대량의 아이피 주소에 대해 각 아이피가 Good Bot**(구글 봇, 애플 봇, 마이크로소프트 봇 등)**인지를 확인하는 작업이 필요했습니다.** 그것도 빠르게..! :horse_racing: 처리해야하는 상황입니다. 

Good Bot인지를 검증하는 작업은 생각보다 심플한데요, 해당 아이피 주소에 대해 **reverse DNS lookup**을 해주면 됩니다. 일반적으로 잘 알려져 있는 봇들은 <u>DNS에 자신의 정보를 등록해두기 때문</u>입니다. 구글에서 이에 대해 작성한 [Verifying Googlebot](https://support.google.com/webmasters/answer/80553?hl=en) 가이드가 있어요! :wink:

오케이! 그렇다면 해야하는 작업은 심플하네요. **각 아이피에 대해 Reverse DNS Lookup**을 하면 됩니다. :white_check_mark: Go언어로 해당 작업내용을 어떻게 작성했는지 지금부터 같이 보시겠습니다.



## Linear Reverse DNS Lookup  :tired_face:

가장 먼저 해본 작업은 선형 룩업입니다. For문으로 아이피 리스트에 대해 Reverse DNS Lookup을 처리하는 것이죠. 이렇게 하면 과연 우리가 원하는 시간 내에 원하는 작업을 할 수 있을까요?

이렇게 선형으로 처리하면 처리는 되지만, 원하는 시간 내에 빠르게 처리할 수 없었습니다. 테스트를 해보니, 고작 500개의 룩업을 처리하는데 약 33초가 걸립니다. 미리 테스트 결과를 보여드리면 아래와 같습니다. 첫번째로 테스트 된 함수가 **TestLinearReverseDNSLookup**이며, 그 다음으로는 고루틴을 활용하여 처리한 **TestReverseDNSLookup**가 테스트 되었습니다. 

```
=== RUN   TestLinearReverseDNSLookup
    TestLinearReverseDNSLookup: bot_specifier_test.go:123: Start Look up 500 addresses.
    TestLinearReverseDNSLookup: bot_specifier_test.go:134: Looked up 61 addresses.
--- PASS: TestLinearReverseDNSLookup (32.95s)
=== RUN   TestReverseDNSLookup
    TestReverseDNSLookup: bot_specifier_test.go:147: Start Look up 1000 addresses.
    TestReverseDNSLookup: bot_specifier_test.go:149: Looked up 561 addresses.
--- PASS: TestReverseDNSLookup (6.52s)
```

사용한 방법은, 아래와 같이 Google DNS 서버를 통해 룩업하는 Resolver를 할당하여 선형 룩업을 진행하였습니다. (단순 반복문으로 처리한 거니까 자세한 코드는 생략합니다. 😇)

```go
Resolver := &net.Resolver{
	PreferGo: true,
	Dial: func(ctx context.Context, network, address string) (net.Conn, error) {
		d := net.Dialer{
			Timeout: time.Second * time.Duration(3),
		}
		return d.DialContext(ctx, "udp", "8.8.8.8:53")
	},
}
```



## Reverse DNS Lookup with Goroutine 😸

오늘 포스팅의 주인공이라고 할 수 있는 고루틴이 등장하였습니다. 아래와 같이 Reverse DNS Lookup을 고루틴으로 처리하였고 성능은 선형 룩업을 했을 때보다 최소 10배 이상은 빠른 것 같습니다. (하드웨어 사양에 따라 성능이 달라지겠지만요.)

{% gist 4da4f1859c82e4632e5528d011a557bb %}

위 코드에서 중점적으로 봐주셔야 할 부분은 `bs.Resolver.LookupAddr(ctx, ipv4)` 를 감싸고 있는 go func 입니다. 각 고루틴들이 룩업한 결과를 리스트에 추가하는 형태로 되어 있으며, 각각의 고루틴들이 임계영역을 서로 침범하지 않게끔 Mutex를 사용해서 처리해주는 것을 보실 수 있습니다.

위 코드를 바탕으로 10만개 아이피에 대해 룩업을 실행한 결과는 아래와 같습니다. 처리한 시간이 1분이 채 안되는 걸 보실 수 있습니다.

```
=== RUN   TestSpecifyGoodBots
    TestSpecifyGoodBots: bot_specifier_test.go:161: Start Look up 100000 addresses.
    TestSpecifyGoodBots: bot_specifier_test.go:163: Looked up 17690 addresses.
    TestSpecifyGoodBots: bot_specifier_test.go:165: There are 0 Good Bots in bs.DnsBook.
--- PASS: TestSpecifyGoodBots (54.92s)
PASS
```



## More?

10만개 아이피에 대해 고루틴으로 Reverse DNS Lookup을 처리할 때의 고루틴 개수 변화 추이, 해당 프로그램에서 사용하는 CPU, Memory 변화량 등에 대해 모니터링을 해보고 싶습니다.
