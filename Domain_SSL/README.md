---
title: 학교 도메인을 포트까지는 연결할 수 없는 것에 대한 해결책
author: Jongmin Mun
---



## Problem: The university network team says they cannot register up to the port numbers
mail content:

안녕하세요 연세대학교 인프라서비스팀 입니다
유선상으로 안내해드렸습니다 (포트번호 까지는 등록불가)
감사합니다

인프라서비스팀  드림
(02-2123-3411)

2021년 11월 22일 (월) 오후 3:45, ‍문종민(학부학생/상경대학 응용통계학과) <jm.moon@yonsei.ac.kr>님이 작성:


선생님, 빠른 답변 감사드립니다.
아까 유선상으로 문의하신 내용 다시 메일로 보내드립니다.

서버 접속 시 주소창에 ip주소가 나오는 상태이며, 
서버의 공인 ip는 101.79.8.237 입니다.

포트가 2개인데, 101.79.8.237:8000는 Jupyter hub이고, 101.79.8.237:8787는  RStudio Server입니다. 현재 브라우저로 접속이 되는 상황입니다.

도움에 감사드립니다.



## Possible solutions
학교 측의 답변은 포트까지는 연결할 수가 없다는 것입니다. 제 부족한 지식으로 조사해 본 대안은 다음과 같습니다.

### 1. 포트 기반 virtualHost
학교에서 서브도메인으로 도메인 2개를 발급받고, 학교 DNS 서버에서 다음과 같이 설정한 다음,
![setting](https://www.tuwlab.com/files/attach/images/2382/863/005/4fbd7aa9468673247683ca8f9a4de18a.png)


저희 서버 설정에서 주피터 로그인 페이지와 RStudio 로그인 페이지에 각각 서브도메인을 대응시키는 것입니다. 이 링크는 Apache로 설정하는 방법에 대한 내용입니다.[^fn1]

- 문제점: 저희가 쓰는 네이버 클라우드 플랫폼에서 Apache와 같은 설정을 할 수 있는지가 불확실합니다.

### 2. 네이버 클라우드 플랫폼 글로벌 DNS 서비스 사용

네이버 클라우드 플랫폼의 글로벌 DNS 서비스를 구독한 다음(쿼리 100만개당 220원 요금 발생), CNAME을 이용해서 학교 DNS에서 서브도메인을 네이버 글로벌 DNS에 위임하는 방법입니다.
- 요금정보 [^fn2]
- 이 링크의 3번 항목에 방법이 나와 있음 [^fn3]
- 문제점: 요금발생

### 3. 네이버 클라우드 플랫폼에서 공인 IP 하나 더 할당받기
- 문제점: 요금 월 4032원 발생 [^fn4]

### 4. 그냥 기본 80포트로 도메인 연결하고, 그 페이지에서 주피터와 Rstudio로 가는 링크 제공
- 문제점: SSL 적용 못할 것 같음


Provisional conclusion : 2nd method

## configurations of Naver Cloud global DNS

### Example of use suggested on the official site

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


### Informations on the university domain

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

### plan
1. http://bk21-bigdata.yonsei.ac.kr subdomain에 대한 정보를 수집하고, subdomain 이관 문제를 학교 네트워크 팀에서 설정할 수 있는지, 아니면 Inames에서 설정해야 하는지 확인하기


[^fn1]: https://www.tuwlab.com/ece/5863
[^fn2]: https://www.ncloud.com/product/networking/globalDns
[^fn3]: https://brunch.co.kr/@topasvga/1938
[^fn4]: https://www.gov-ncloud.com/product/compute/server
[^fn5]: https://www.ncloud.com/product/networking/globalDns
[^fn6]: https://www.whois.com/whois/yonsei.ac.kr
[^fn7]: http://www.inames.co.kr
[^fn8]: https://guide.ncloud-docs.com/docs/ko/networking-networking-1-2
