---
title: 학교 도메인을 포트까지는 연결할 수 없는 것에 대한 해결책
author: Jongmin Mun
---



# Problem: The university network team says they cannot register up to the port numbers
mail content:

안녕하세요 연세대학교 인프라서비스팀 입니다
유선상으로 안내해드렸습니다 (포트번호 까지는 등록불가)
감사합니다

인프라서비스팀  드림
(02-2123-3411)


# Analysis of Yonsei DNS server

- 연세대학교(registrant)가 요금을 지불하고 고용한 registrar는 Inames Co., Ltd.이다.
- 연세대학교가 보유한 도메인은 yonsei.[ac.kr](http://ac.kr)이다.
- 연세대학교가 운영하는 name server는 165.132.10.21([ns.yonsei.ac.kr](http://ns.yonsei.ac.kr/))이다.
- 165.132.10.21([ns.yonsei.ac.kr](http://ns.yonsei.ac.kr/))는 NS record type으로 .ac.kr이나 .kr을 담당하는 name server에 등록되어 있을 것이다. 레코드 내용은 다음과 같을 것이다.
    
    ```latex
    [yonsei.ac.kr](http://yonsei.ac.kr) NS ns.yonsei.ac.kr
    ```
    
- bk21-bigdata.yonsei.ac.kr등은 yonsei.ac.kr의 서브도메인으로, 165.132.10.21([ns.yonsei.ac.kr](http://ns.yonsei.ac.kr/))에 레코드로 기록되어 있을 것이다. 레코드 내용은 다음과 같을 것이다(실제 해당 사이트의 IP임. IP로 접속은 차단되어 있음)
    
    ```latex
    bk21-bigdata A 112.175.184.12
    ```
    
- bk21-bigdata.yonsei.ac.kr의 IP주소는 112.175.184.12로, 교내 IP 주소인 165.132.xx.xx와 다르다. 교내에서 호스팅하는 것이 아니라 [닷홈](https://www.dothome.co.kr/) 이라는 업체에서 호스팅하고 있다.[](https://www.dothome.co.kr/)

# 목표: 두 가지 방향

## 방향 1(현재 목표)

1. 통계데이터사이언스학과 자체 name server 구축
    1. NAVER GLOBAL DNS 서비스 사용
    2. 계약이 불발될 경우 다른 name server 서비스를 사용
    3. 혹은 직접 name server 구동

 2. bk21-bigdata라는 subdomain을 165.132.10.21([ns.yonsei.ac.kr](http://ns.yonsei.ac.kr/))에서 이관(delegate)해 온다.

즉,  165.132.10.21([ns.yonsei.ac.kr](http://ns.yonsei.ac.kr/))에서 다음 레코드를 지우고

```latex
bk21-bigdata A 112.175.184.12
```

아래 레코드를 기록하게 한다.

```latex
bk21-bigdata NS ns.naverglobal....
```

1. 앞으로 배포할 다양한 서비스에 자유롭게 서브도메인을 할당하여 사용한다. 즉, 아래와 같은 레코드를 우리 name server에 기록한다.

```latex
jupyter A 192.xxx...
rstudio A 192.xxx..
wiki A 192.xxx...
```

## 방향 2

- 통계데이터사이언스학과 자체 name server를 구축하지 않고, 그냥 도메인이 필요할 때마다 학교 네트워크팀에 요청한다.


## Next thing to do
1. NS 레코드로 서브도메인을 이관해 올 수 있는지 학교 네트워크팀에 확인한다. 학교 네트워크팀에서 안 된다고 하면, Inames에도 문의한다. 여기서도 안 된다고 하면, 방향 2로 선회한다.


# configurations of Naver Cloud global DNS

## Example of use suggested on the official site

Global DNS에서 하위 도메인을 운영하는 경우[^fn5]

1. 하위 도메인을 신규로 생성하고, 생성한 하위 도메인의 리소스 레코드 정보를 입력합니다. 기존에 DNS를 관리하던 하위 도메인을 이관하는 경우에는 기존 리소스 레코드를 이관(복제)합니다.
2. 상위 도메인의 리소스 레코드 설정에서 하위 도메인의 네임서버를 Global DNS의 네임서버로 등록합니다.

**detailed instructions:**

example.com을 운영 중인 도메인(상위 도메인), sub.example.com을 하위 도메인이라고 가정할 때 등록하는 방법은 다음과 같습니다.[^fn8]

1. 네이버 클라우드 플랫폼 DNS에 하위 도메인을 신규 생성합니다.


![setting](https://cdn.document360.io/6998976f-9d95-4df8-b847-d375892b92c2/Images/Documentation/networking-1-2-103-2-1_ko.png)

2. 하위 도메인에 필요한 리소스 레코드를 등록합니다. *기존에 하위 도메인이 있고 외부 DNS에서 서비스하고 있었다면* 리소스 레코드를 동일하게 등록합니다.

![setting](https://cdn.document360.io/6998976f-9d95-4df8-b847-d375892b92c2/Images/Documentation/networking-1-2-103-2-2_ko.png)

3. 네이버 클라우드 플랫폼 DNS에서 신규 생성한 하위 도메인의 네임서버 정보를 확인할 수 있습니다.

![setting](https://cdn.document360.io/6998976f-9d95-4df8-b847-d375892b92c2/Images/Documentation/networking-1-2-103-2-3_ko.png)

4. 상위 도메인(example.com)에서 하위 도메인(sub.example.com)에 대한 NS 레코드를 등록합니다. 이때 NS 레코드는 하위 도메인의 네임서버 정보로 등록합니다.

![setting](https://cdn.document360.io/6998976f-9d95-4df8-b847-d375892b92c2/Images/Documentation/networking-1-2-103-2-4_ko.png)


## Informations on the university domain

- Authorized Agency           : Inames Co., Ltd.[^fn7]
- Primary Name Server
   - Host Name                : ns.yonsei.ac.kr
   - IP Address               : 165.132.10.21

- Secondary Name Server
   - Host Name                : ns2.yonsei.ac.kr
   - IP Address               : 165.132.5.21
   - Host Name                : ns3.yonsei.ac.kr
   - IP Address               : 165.132.237.21
   - Host Name                : yumciris.yonsei.ac.kr
   - IP Address               : 128.134.207.17
[^fn6]

# Elements of DNS(for reference)
## 1. Hierarchical structure of domain names

[https://youtu.be/2EIgPYRzVwY](https://youtu.be/2EIgPYRzVwY)


예시: blog.example.com. 

**[ . ]** : Root domain. 사실 도메인 뒤에는 .이 항상 생략되어 있음

**[.com]**: Top-level domain. ex) .net, .kr 등

**[.example]**: Second-level domain

**[blog]**: subdomain

### 1.1. Hierarchical structure of name servers

각 계층을 담당하는 독자적인 name server가 존재 

- 상위  name server는 한 단계 하위 name server를 알고 있음:
    - Root domain을 담당하는  name server는 top-level을 담당하는  name server들의 목록과 ip주소를 알고 있다.
    - Top-level을 담당하는 name server는 second-level을 담당하는  name server들의 목록과 ip주소를 알고 있다.
    - **Second-level을 담당하는 DNS 서버는 sub-domain을 담당하는 DNS 서버 목록과 ip주소를 알고 있다.**
- 단, 두 단계 아래의 name server에 대해서는 알지 못함: root dmain 담당  name server는 second-level 담당  name server에 대해 알지 못함.

### 1.2. Hierarchical procedure of IP query

- 내 컴퓨터가 blog.example.com. 에 해당하는 ip 주소를 알고 싶어 한다.
- 그 IP 주소는 subdomain을 전담하고 있는 name server만이 알고 있다.  내 컴퓨터가 수만 개 name server 중 그  name server를 쉽게 찾아낼 수 있도록 하는 과정은 아래와 같다.

1. 모든 컴퓨터는 root name server의 IP주소는 알고 있다. Root name server는 .com에 해당하는 top-level name 서버의 IP를 브라우저에게 알려준다.
2. .com 담당 top-level name server는 example.com. 담당 second-level name server의 IP를 브라우저에게 알려준다.
3. example.com. 담당하는 second-level name serer는 blog.example.com.을 담당하는 subdomain name server의 IP를 알려준다.
4. blog.example.com.을 담당하는 subdomain name server가 브라우저에게 blog.example.com.의 IP를 드디어 알려 준다.

## 2. DNS register

[https://youtu.be/hfj0IGgKAgU](https://youtu.be/hfj0IGgKAgU)

DNS의 많은 부분이 행정적인 것이고, 그 행정을 기술이 자동화하고 있는 것.

### 2.1. Three players in the DNS register game

1. Registrant(등록자, 우리와 같은 개인 및 회사 서버를 도메인 등록하려는 사람)
2. Registry(등록소)
    
    .com, .net 등 top-level domain을 관리
    
    [예시] .com.을 관리하는 서버의 주소가 a.gtld-servers.net이라 하자.
    
3. Registrar(등록 대행자)
    
    registrant는 registry에 직접 요청할 수 없음
    
    registrar에게 돈을 내고 요청하면, registrar가 registry에 등록해 준다.
    

### 2.2. Procedure of register

1. ICANN이 운영하는 root name server 13개가 전세계에 흩어져 있음(a.root-servers.net ~ m.root-servers.net)
    1. root name server는 전세계의 top-level domain name server들의 의 주소를 기억하는게 역할
    2. root name server에는 레코드가 이렇게 적혀 있음:

```latex
com NS a.gtld-servers.net
```

해석: com이라는 top-level domain의 네임서버는 a.gtld-servers.netf라는 주소를 가진다.

1. 등록소에 내 서버의 ip주소를 직접 등록할 수는 없다!
    1. 내가 운영하는 name server를 하나 마련해야 한다.
    2. [예시] 그 name server의 주소를 a.iana-servers.net이라 하자.
    3. 많은 경우 registrar가 name server 또한 제공한다.
    4. **무료나 저렴한 네임서버 서비스도 많다. (ex. Naver Global DNS)**

2. DNS register의 의미: top-level domain name server가 내 name server를 알게 한다.
    1. registrar에게, .com 담당 top-level domain name server(registry)에 다음과 같이 기록하라고 요청한다.

```latex
[example.com](http://example.com) NS a.iana-servers.net
```

해석: example.com이라는 도메인의 name server는 a.iana-servers.net이다.

이제 .com 담당 top-level domain name server는 내 name server를 알고 있다.

3. 이제 나의 name server에서, 내 IP와 주소를 매칭한다.

```latex
[example.com](http://example.com) A 93.183.216.34
```

해석: example.com의 주소(A stands for address)는 93.183.216.34이다.


**주의:**

root name server가 top level domain name server를 기억하는 방식은 NS

top level domain name server가 내 name server를 기억하는 방식도 NS

하지만 내  name server에서는 NS로 하지 않고 A로 함. 더 이상 다른 name server를 부를 필요가 없으므로.

정보 한건 한건을 record라고 하고, NS, A 등을 record type이라고 함.



[^fn1]: https://www.tuwlab.com/ece/5863
[^fn2]: https://www.ncloud.com/product/networking/globalDns
[^fn3]: https://brunch.co.kr/@topasvga/1938
[^fn4]: https://www.gov-ncloud.com/product/compute/server
[^fn5]: https://www.ncloud.com/product/networking/globalDns
[^fn6]: https://www.whois.com/whois/yonsei.ac.kr
[^fn7]: http://www.inames.co.kr
[^fn8]: https://guide.ncloud-docs.com/docs/ko/networking-networking-1-2
