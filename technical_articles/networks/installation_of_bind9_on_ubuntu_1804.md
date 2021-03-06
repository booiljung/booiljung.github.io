## 목적

- 도메인 이름을 통한 내부 장비 접속
- 개발용 장비와 운영용 장비 네임 통합

## 문제

로컬용으로 `*.aa`을 사용하기로 함.

## Ubuntu 18.04 LTS

### bind9

#### 설치

```
sudo apt install bind9
```

설치하면 자동으로 시작

#### 구성

`/etc/bind/named.conf.options` 변경

```
options {
	directory "/var/cache/bind";

	forwarders {
		168.126.63.1;
		168.126.63.2;
		8.8.8.8;
		8.8.8.4;
	};

	dnssec-validation auto;
	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };
	allow-query { any; };
	recursion yes;
};
```

`/etc/bind/named.conf.local` 변경

```
zone "sm" {
	type master;
	file "/etc/bind/db.aaa";
	allow-update { any; };
	allow-transfer { any; };
	notify no;
};

zone "0.168.192.in-addr.arpa" {
	type master;
	file "/etc/bind/db.aaa.rev";
	allow-update { any; };
	allow-transfer { any; };
};
```

`/etc/bind/db.aaa` 생성 편집

```
$TTL	604800
@	IN	SOA	ns1.sm. user.sm. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
sm.		IN	NS	ns1.sm.
sm.		IN	A	192.168.0.253

; ordered by ALPHBET
gateway		IN	A	192.168.0.1
mavlink		IN	A	192.168.0.248
registry.docker IN	A	192.168.0.249
git		IN	A	192.168.0.250
docker		IN	A	192.168.0.251
file1		IN	A	192.168.0.252
ns1		IN	A	192.168.0.253
ubu.booil	IN	A	192.168.0.102
win.booil	IN	A	192.168.0.103
ubu.sun0	IN	A	192.168.0.202
win.sun0	IN	A	192.168.0.201
```

`/etc/bind/db.aaa.rev` 생성 편집

```
$TTL	604800
@	IN	SOA	ns1.sm. user.sm. (
			     42		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL

; ordered by		ALPHBET
1	IN	PTR	gateway.sm.
248	IN	PTR	mavlink.sm.
249	IN	PTR	registry.docker.sm.
250	IN	PTR	git.sm.
251	IN	PTR	docker.sm.
252	IN	PTR	file1.sm.
253	IN	PTR	ns1.sm.
102	IN	PTR	ubu.booil.sm.
103	IN	PTR	win.booil.sm.
202	IN	PTR	ubu.sun0.sm.
201	IN	PTR	win.sun0.sm.
```

#### 변경사항 업데이트

(필수) 네임 데몬 리로드

```
sudo service bind9 reload
```

(선택) 네임 데몬 재시작

```
sudo service bind9 restart
```

(선택) 네임 데몬 상태 보기

```
sudo service bind9 status
```

(선택) 네트워크 재시작

```
sudo service networking restart
```

(필수) 캐시 삭제
```
rndc flush
```

(필수) 캐시 리로드
```
rndc reload
```

(필수) 캐시 덤프
```
rndc dumpddb -cache
```

(선택) 테스트

```
$ nslookup
> git.aaasoft.co.kr
```

## 라우터 연동

- 라우터 DNS 항목에 네임 서버 IP 적용
- 고정 IP 사용자는 DNS 항목에 네임 서버 IP 적용

## 참조

https://qastack.kr/ubuntu/330148/how-do-i-do-a-complete-bind9-dns-server-configuration-with-a-hostname


