---
title: "Let's Encrypt: A Free, Automated, and Open Cerificate Authority by Josh Aas"
date: 2019-12-07
---

> SSL관련 자료를 찾다가 우연히 유튜브에서 [Let's Encrypt: A Free, Automated, and Open Cerificate Authority by Josh Aas]( https://www.youtube.com/watch?v=ksqTu7TX83g) 영상을 보게 되었습니다. 영상을 보고 난 후 저는 이 영상이 **웹 보안에 관심 있는 사람**에게 도움이 될 것이라 생각했고, 해당 영상을 다른 분들이 보실 때 이해를 돕고자 이렇게 포스팅을 작성하게 됐습니다. 

영상에 사용된 원문의 해석과 보충 설명이 필요한 부분은 주석을 함께 덧붙여 내용이 잘 전달 될 수 있게끔 노력 했습니다만, 모든 내용을 번역한 것은 아니며 발표 슬라이드에 있는 텍스트 위주로 번역한 것임을 알려드립니다. 슬라이드의 내용을 영어로 먼저 기술하고 그 이후 해석과 주석을 덧붙였습니다. 경우에 따라서는 문장만 있고 주석이 있는 경우도 있으니 양해바랍니다.

### What is HTTPS (HTTPS는 무엇인가요?)

**HTTPS** is HTTP over a connection secured by TLS (used to be called SSL).

기존의 HTTP통신에 **SSL(Secure Socket Layer)**을 추가한 것이 HTTPS 입니다. HTTPS는 HTTP와는 다르게 통신하는 내용이 암호화되어 통신 주체가 아닌 제 3자가 해당 내용을 엿보거나 통제하기 어렵게 되어 있습니다.

### Why is HTTPS Important for Everyone? (왜 HTTPS가 모두에게 중요할까요?)

**Users** aren't in control of their data any more, Web is too complex. Need to treat everything as potentially sensitive.
유저는 자신의 데이터에 대한 통제권이 없습니다. 데이터를 통제한다는 건, 웹에 대한 정확한 이해와 그에 따른 대처를 적절히 할 수 있다는 것인데, 그러기엔 유저가 알아야 할 것이 너무 많고 이는 유저에게 복잡하고 혼란스럽기만 합니다.

단순 웹페이지 접속 또한 쿠키 정보, 헤더 정보, 써드 파티 앱과의 통신 등 우리와 연관된 수 많은 정보들이 빠르게 네트워크를 거쳐갑니다. 사실 이 모든 데이터에 대해 알고 통제한다는 건 불가능에 가깝죠. 이는 기술을 잘 아는 사람에게도 마찬가지입니다.

그렇다면 우리는 어떻게 해야할까요? 먼저 **모든 정보는 민감한 정보이며 반드시 암호화되어 보호받을 필요가 있다**라고 생각하는 것이 중요합니다. 중요하고 보호받아야 할 정보를 선별적으로 관리하는 것이 오히려 더 많은 수고로움을 요구하기 때문이죠. 따라서 수많은 데이터를 분류하기 보단 모든 데이터를 보호하자는 방향으로 접근하는 것이 좀 더 합리적일 겁니다.

**Developers** expect integrity. Unencrypted traffic can be modified, e.g. ad insertion, script injection (e.g. great cannon).

네트워크에 흘러다니는 정보들이 암호화되어 있지 않고 누구나 다 열람해서 확인하고 변조할 수 있다면 그건 개발자한테도 매우 좋지 않습니다. 

만약 여러분이 웹사이트를 개발했다고 해봅시다. 사이트 방문자에게 "A"라는 알파벳만 보여주게 페이지를 만들었다고 해보죠. 사이트를 개발한 여러분은 사이트 방문자들이 "A"라는 알파벳이 볼 거라 기대하지만, 트래픽이 암호화되지 않은 상태에서 중간에 악의적인 누군가가 사이트 컨텐츠에 광고를 끼워넣거나 "A"가 아닌 "ABC"로 내용을 바꿔서 사이트 방문자에게 전달하거나 야한 사진 등을 넣어서 전달할 수도 있습니다. 사이트 방문자 입장에서는 당연히 이런 사실에 대해 알 수도 없고, 일일이 알아보지도 않을 것이기 때문에 피해는 고스란히 사이트 개발자, 운영자에게 돌아갑니다.

### Important for you? (당신에게 HTTPS는 중요한가요?)

* How many of you believe that your cell phone provider actively mines data from your unencrypted Web traffic?
  여러분 중 과연, 스마트폰의 제조회사가 **사용자의 스마트폰 데이터(e.g., 웹 트래픽 중 암호화되지 않은 데이터)**를 제조사의 이익을 위해 사용하지 않는다는 걸 100% 확신할 수 있는 사람이 몇 명이나 될까요?
* How about your home ISP(Internet Service Provider)?
  여러분은 KT, SKT, LG 등 **인터넷 서비스 제공자**를 얼마나 믿나요? 우리가 사용하는 데이터를 훔쳐보거나 빼가거나 하지는 않을까요?
* How about your government?
  정부를 믿나요?

실제로 인터넷 서비스 제공자들과 정부는 우리의 웹 활동 중 일부를 감시한다고 합니다. 대표적인 예로 불법 사이트 접속을 막는 걸 생각해보실 수 있습니다. 웹 서핑하시면서 다들 한 번쯤은 파란 경찰 마크와 함께 해당 불법 사이트 차단 이미지를 봤을 거라 생각합니다. 요즘에는 인터넷으로 접속 자체가 안되게 막아놓았죠.

"불법 사이트를 차단해주다니..! 정말 건전하고 바람직해"라고 생각할 수도 있지만, 다른 말로 **"우리는 매 활동을 검열당하고 있고 감시 당하고 있는 거네."**라고 생각할 수도 있는 겁니다. 통제의 단계가 약할 뿐이지, 언제든 더 강한 강도로 우린 억압받을 수 있고 그럴 환경은 이미 갖추어져 있습니다.

### The Web Isn't Secure (웹, 인터넷은 안전하지 않다.)

As of August 2016, only 45.5% of Firefox page loads are HTTPS. Should be 100%. 

2016년 8월 파이어 폭스 브라우저 기준으로, 파이어 폭스를 거친 웹사이트들 중 45.5%만 HTTPS통신을 구현했습니다. 안전한 인터넷 활동을 위해서 이는 너무나도 적은 수치이고, 100%를 목표로 해야 합니다.

### Why Isn't the Web Secure? (왜 웹은 안전하지 않을까?)

Security is too difficult. We know what needs to be done but we haven't made it easy enough to do. Unitl now? More on that later...

세상에는 이미 매우 유용한 암호화 방법론과 인증 관련 방법론이 많이 존재합니다. 그런데도 왜 웹은 아직까지도 안전하지 않을까요?

그 이유는 바로 보안 자체가 상당히 어렵고 까다로운 분야이기 때문입니다. 적절히 설정하고 사용하는 과정에 대한 진입장벽이 높고 **비용(= 보안을 달성하기 위한 인건비 등)**도 비쌉니다. 오늘 발표에서는 어떻게하면 그 과정을 쉽게할 수 있을지에 대해 나누고자 합니다.

### Secure Connection Requirements

안전한 통신을 제대로 사용하기 위해서는 2가지 요구사항이 존재합니다.

1. Encryption (scrambled bits)
2. Authentication (who am I talking to?)

첫번째는 암호화 입니다. 상대방에게 데이터를 전달할 때 "여기 당신이 원하는 데이터!"라고 직관적으로 주는 것이 아니라 "ㄴㅇ랴ㅐㅈ더갸ㅐㅈ갭ㄱㅂ"와 같이 암호화(난독화)를 하는 겁니다. 해석하기 어려운 난독화된 데이터를 원하는 상대에게 전송하기 때문에 중간에서 가로챈다해도 해석하는 것이 굉장히 어렵고 바로 이러한 부분이 보안으로 연결되는 겁니다.

두번째는 인증 입니다. 난독화된 암호문을 받았는데 적절한 해석방법도 모르고 누구랑 통신하는지도 모르면 해당 암호문을 제대로 해석할 수 없습니다. 이를 위해서는 특별한 인증과정이 별도로 필요하고, 그 인증과정을 통해 "상대방이 내가 통신하려는 사람이 맞구나!"하며 안심하고 통신할 수 있는 것입니다.

### Encryption (easy-ish)

Just a software stack for public key and symmetric crypto that needs to be installed and configured. Most Web servers directly support it. Protect your private keys.

암호화 자체는 이미 널리 알려진 구현체가 많습니다. 대부분의 운영체제에서 자체 지원하고 있는 것들도 많죠. 어려운 것은 **"암호화된 데이터를 해석할 수 있는 아주 중요한 개인키를 어떻게 보호하느냐"**입니다.

### Authentication via CA (too hard)

Certificate Authority issue the x.509 certificates required for SSL/TLS on the Web.

인증과정은 악몽과도 같습니다. 웹에서 인증을 구현하려면 인증서 관리국으로부터 인증서를 받아야 하는데요, 이는 기술을 잘 아는 것과 관계 없이 누구에게나 굉장히 복잡하고 번거롭습니다.

### Traditional Certificate Fun (인증서를 얻기 위한 전통적인 방법론, 매우 길고 짜증나고 고된 방법)

* Figure out you need a cert
  인증서가 필요할 때 우리가 고려해야하는 건 아래와 같습니다.
* Try to figure out where to get a cert from
  **1: 어디서 인증서를 발급 받아야 하는지를 파악합니다.**
* Figure out what kind of cert you need
  **2: 어떤 종류의 인증서를 발급 받아야 하는지 파악합니다.**
  주의: 많은 인증기관들에서 제공하는 다양한 종류의 인증서를 훑어봐야되고 온갖 마케팅 광고에 노출되어 심신이 피폐해질 수 있습니다.
* Figure out how to request a cert (learn what a CSR is and how you make one)
  **3: 인증받고자 하는 인증기관에서는 여러분에게 CSR을 요구할 겁니다. CSR에 대해서도 알아야겠죠?**
  주의: CSR은 이해하는데 굉장히 오래걸리는 개념이고... 이는 할 일이 언제나 많은 우리를 지치게 만드는 포인트입니다. 안 그래도 할 게 많은데...
* Go through painful manual verification process, maybe set up a new email address
  **4: 수동으로 인증 절차를 거쳐야하는데... 이메일 등도 별도로 설정해야 합니다.**
  주의: 매우 재미없고 의미없다고 느껴지는 과정이 될 수 있습니다.
* Pay
  **5: 인증기관에게 돈을 지불합니다.**
  개인으로 돈을 지불하는 건 상관없지만, 만약 여러분이 기업에서 일하는 개발자라면 어떠까요? 추가비용 지출을 위해 별도의 결재를 올려 승인을 받아야하고, 이 과정에 대해 별도의 설명을 해야하는 등... 매우 귀찮은 일이 기다리고 있을 겁니다.
* Figure out how to install your cert
  **6: 서버에 인증서 설치하는 방법론도 공부하셔야합니다.**
* Remember to renew it on time
  **7: 갱신기간이 되면 제때 갱신해주어야 합니다. 이것도 신경써주셔야 하는거죠.**
  관리포인트가 늘어났네요... 맙소사.

### HTTP/2, Sumer 2012

HTTP/2 IETF WG debating requiring encryption. Major objection was that encryption was too hard or even impossible for many.

Josh Aas and Eric Rescorla decide we need to address this problem if the Web is ever going to achieve HTTPS Everywhere.

데이터를 암호화해서 보호하는 게 중요하다는 것에는 모두가 동의하지만, 현실적으로 웹을 이용하는 사람 모두가 HTTPS를 인지하고 사용하는 건 매우 어렵습니다. 논의되고 있는 HTTP/2의 보안 스펙은 TLS보안을 필수로 넣지 않는 쪽으로 논의되었다고 합니다. 실질적으로 모두가 TLS를 적용하는 데에는 너무나도 많은 비용이 들기 때문이죠. 올바른 방법론은 뻔히 있는데 현실적인 제약으로 그러지 못한다는 건 너무나도 슬픈 일입니다. 이런 문제들을 해결하기 위해 많은 고민을 거듭한 결과 아래와 같은 솔루션을 고안하게 되었습니다.

### The Solution

We needed a new CA. A lot of work, but nothing short of that would cut it.

Didn't think we could get an existing CA to fully cooperate. Especially on the time scale we wanted. Also, didn't want to be forever at the mercy of another CA.

바로 새로운 CA(인증기관)을 만드는 겁니다. 현존하는 인증기관들에게 협력을 구하는 것도 어려울 것 같고, 협력을 한다해도 그게 오래 지속될지도 모르는 일입니다. 인증기관의 선심, 도덕성 등에만 의존할 수는 없으니까요.

### Cornerstones for a New CA (새로운 인증기관을 위한 초석)

새로운 인증기관을 설립할 때 필요하다고 생각한 것들은 아래와 같습니다.

**Automated**

Ease of use, reliability, and scalability
자동화로 인한 사용자 편의성이 향상(신뢰성과 확장성 또한 포함)되어야 합니다.

**Free**

Cost excludes, payment adds complexity

완전 무료로 이용가능해야 합니다. 결제하는 과정이 들어가는 건 복잡성만 더하니까요.

**Transparent / Open**

Essential for trust

투명성과 신뢰성이 있어야 합니다. 참고로 현재 운영되는 CA들은 이용자들에게 모든 걸 세세하게 투명하게 공개하지는 않습니다.

**Global**

The secure Web is for everyone

안전한 웹 경험, 인터넷 경험은 소수의 것이 아닌 우리 모두의 것이 되어야 한다고 생각합니다.

### Building a Foundation

Spent ~2 years (2013-2014) clearing schedules, creating an entity, finding initial sponsors, and making a plan for getting trusted (a deal with an existing CA).

위 목표를 달성하기 위해 2년 정도를 할애해서 스폰서들을 구했습니다. CA로서 신뢰를 얻으려면 자기 자신에 대해 개인키를 생성하고 해당 키가 정합한 것인지를 증명하는 과정을 거쳐야하는데, 이를 위해서는 모든 브라우저들과 세상에 존재하는 핵심 서비스들에 전파되고 알려져야합니다. 우리는 그러기 위한 시간이 약 5~10년 정도는 걸릴 것이라 생각했습니다. 5~10년은 너무나도 긴 시간이므로 우리는 어쩔 수 없이 스폰서로 현존하는 CA를 두어 우리가 CA로서 적합하다는 것을 증명하는데 힘을 실었습니다.

### Internet Security Research Group

ISRG is the entity behind Let's Encrypt.

* Founded May 2013
* IRS 501(c)(3) status granted June 2014

Mission is to reduce financial, technological, and education barriers to secure communication over the Internet.

인터넷 연구 그룹(ISRG)으로부터의 지원도 받고 있습니다. 연구 그룹의 미션은 모든 사람이 인터넷에서의 안전한 통신을 누릴 수 있게끔 경제적 장벽, 교육적 장벽, 기술적 장벽을 허무는 것입니다.

### The Linux Foundation

Let's Encrypt became an LF collaborative project in April of 2015.

The Linux Foundation's organizational development support has allowed us to focus on our technical operations.

리눅스 재단의 지원 또한 받을 수 있게 되었고 이제는 공동 프로젝트로 진행하게 되었습니다.

### Building Let's Encrypt

Announced to the public in November 2014. Work still to complete:

* Creating CA policy
* Writing software
* Ordering and installing hardware
* Configuring environment
* Hiring a team
* Making things as secure as possible
* Ensuring compliance, going thorugh audits
* Getting more sponsors

### How Let's Encrypt Works

* ACME Protocol, "DHCP for Certificates"
  ACME라는 자동으로 인증서를 받을 수 있는 프로토콜을 사용합니다.
* Boulder CA software implements ACME, running on Let's Encrypt infrastructure
  서버사이드 프로토콜로 깃허브에서 바로 확인해볼 수 있습니다.
* Clients request certificates from Let's Encrypt via ACME

### API Infrastructure

* ~42 rack units of hardware between two highly secure sites
* HSMs, compute, storage, switches, firewalls
* Lots of physical and logical redundancy
* Linux is our primary operating system
* Heavy use of config mgmt, almost everything automated
* API and OCSP go through Akamai

### Types of Certificates

인증서의 종류는 다음과 같습니다.

* **Domain Validation (DV):** asserts control of a domain (what Let's Encrypt does)
  Sorts control of a domain and ties that to a public key, so basically says "this is the public key for the domain that you're trying to talk to you."
  
  If you encrypt with this public key, the idea is that this domain is the only domain that I'll be able to decrypt.
  도메인을 통제하고 있다는 것을 공개키를 통해 입증하는 절차 입니다. "당신이 소통하고 있는 도메인에 대한 공개키가 바로 여깄어!"라고 말하는 방법으로, 여러분이 공개키로 암호화를 하면 이 도메인은 유일하게 여러분 본인이 복호화할 수 있는 것이죠.
  
* **Organization Validation (OV):** asserts control of a domain as well as basic organizational vetting
  DV랑 비슷하지만 name of an entity를 포함한다는 점이 다릅니다. 
  
* **Extended Validation (EV):** asserts control of a domain as well as extended organizational vetting
  OV랑 비슷하고 entity에 대한 더 많은 정보를 요구합니다.

LE는 오직 DV만 발행하는데, DV만이 유일하게 자동화될 수 있기 때문입니다. 조직차원의 엔티티를 자동으로 증명할 수 있는 방법은 없습니다. 그러나 ACME 프로토콜을 사용하면 도메인에 대한 통제권을 확인하는 과정을 자동화할 수 있습니다. 자동화할 수 있다는 건, 수많은 DV를 발행할 수 있다는 뜻이기도 합니다. 우리가 서비스 스케일을 감당할 수 있는 건 DV 뿐입니다.

### ACME Issuance Overview

ACME 인증 과정을 대략적으로 설명하면 다음과 같습니다.

![ACME](/images/2019-12-07-lets-encrypt-speech/0.png)

1. 먼저 클라이언트가 Lets Encrypt 서버에 "인증서 발행해주세요!" 요청을 합니다.
2. "You need to demonstrate these things"
   "You need to complete these challenges in order to get a cert"
   요청을 받은 서버는 클라이언트에게 도전과제들을 줍니다. 인증서를 얻기 위해서 클라이언트는 서버가 주는 과제들을 잘 수행해야 하죠.
3. 클라이언트는 도전과제들을 마친 뒤에 Lets Encrypt 서버에게 "과제 다 마쳤으니 오셔서 검사하셔도 됩니다."라고 메시지를 보냅니다.
4. LE서버는  "오케이, 클라이언트. 부여한 과제들이 잘 수행 되었는지 확인해볼게요."
5. 확인이 잘 되었다면 인증서를 발행해주고 그게 아니라면 요청을 거절합니다.

### ACME Challenge Types

도전과제들로는 아래와 같은 과제들이 주어집니다.

* **HTTP: Put a file on your web server**
  This is where we give you a special file we tell you to put it at a special place or on this particular path on your web server and then we pick a path that is not likely to be in use for other things and the file is not a file that's likely to be already on your web server anywhere, so if  you demonstrate to us that you can place this special file at a particular predetermined location in your server that's a demonstration of control.
웹 서버의 특정 위치에 파일을 서비스할 것을 지시합니다. 그 위치는 웹서버에서 잘 사용되지 않을 만한 위치로 선정하고 파일 이름은 웹 서버에 기존에 존재하는 파일 이름으로 상상할 수 없는 이름을 지정하게 됩니다. 한 마디로 "우리가 지정한 위치에 우리가 지명한 파일 이름으로 파일을 생성하세요"라는 과제인 것이고, 그걸 해당 도메인이 서비스 되는 웹서버에 위치한다면, 그 도메인을 제어하고 있다고 인정하는 것이죠.
  
* **DVSNI: Provision a virtual host at your domain's IP address**
  we ask you to essentially provision a virtual host that your domain IP address in such a way that it demonstartes proper control. 
  
가장 적게 쓰이는 옵션이지만 특정 상황에서는 꽤 괜찮은 도전 과제입니다.
  
* **DNS: Provision a DNS record for your domain**

  The thrid way is DNS validation and it's a lot like HTTP validation but instead of putting a file on your server you can think of it as taking that file and sticking it in a DNS record because if you can control if you can demontsrate control of DNS for your domain then we're gonna with just assuming that you can contorl the server because you can point DNS wherever you want. DNS is fairly popular and I believe it's growing becase DNS is the only challenge that dosen't require us to actually contact your server for verification so sometime people for example I have an internal web server that's not on the public web but they have a publicly resolvable DNS record they can use the DNS challenge and prove control over the server without having Let's encrypt actually go back to ther server and talk to it to check for a file. DNS is also used a lot for devices which we'll talk about a little later.
  DNS 검증은 HTTP 검증 방식과 유사하지만 특정 파일을 웹서버에 두는 것이 아니라, DNS 레코드에 해당 그 파일 내용을 기록해두는 검증 방식입니다.

### Clients

Basically 3 categories of clients:

1. Simple (drop a cert in the current dir)
2. Full-featured (configure server for you)
3. Built-in (web server just does it)

Built-in is the best client expereience!

### Devices

Many of our cerificates are going to devices for management interfaces.

Synology is a great example. One click and your NAS system is using publicly trusted PKI for management.

Have been talking to other device makers, routers in particular, a lot more of this is coming.

### 90 Day Cert Lifetimes

Shorter lifetimes are important for security.

* Encourage automation
* Limit damage from key compromise or mis-issuance (revocation dosen't work)

Short-lived certs some day? Potentially a nice solution for revocation.

### Phishing and Malware

CAs are not the right place to police phishing and malware.

* Don't have the data
* Can't respond fast enough
* If HTTTPS becomes existential, we don't want to be censors
* Revocation is ineffective

Read our full blog post about this.

구글에서 운영하는 위협데이터베이스와 비교하여 적절치 않은 도메인이라 판단되면 해당 도메인에 대해 인증서는 발행해주지 않습니다.

### Certificate Transparency

CT is an important part of improving the PKI ecosystem 
