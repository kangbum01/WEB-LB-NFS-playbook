# 📘 Web-LB-NFS Ansible Playbook

이 프로젝트는 **온프레미스 환경**에서 **LB (Load Balancer) – WEB – NFS 아키텍처**를  
Ansible Playbook 기반으로 **자동 배포**하기 위해 작성되었습니다.

---

## 🖥️ 아키텍처 구성

- **NFS 서버** : `server1` (192.168.10.10)  
- **WEB & DNS 서버** : `server2` (192.168.10.11), `server4` (192.168.10.14)  
- **Load Balancer** : `server3` (192.168.10.12)  

---

## 📂 프로젝트 구조
```yaml
project/
├── site.yml # 메인 플레이북 (모든 role 실행)
├── inventory # 인벤토리 (NFS, WEB, LB, DNS 서버 정의)
├── group_vars/
│ └── all.yml # 공통 변수 (NFS/LB/WEB/DNS 설정)
└── roles/
├── nfs_server/ # NFS 서버 role
├── web/ # WEB 서버 role
├── lb/ # LB role
└── leeeeejieun.dns # DNS 서버 role
```

> DNS Role 출처: [Ansible Galaxy – leeeeejieun.dns](https://galaxy.ansible.com/ui/standalone/roles/leeeeejieun/dns/documentation/)

---

## 🚀 Roles 설명

### 1. **nfs_server**
- 패키지 설치: `nfs-utils`, `firewalld`  
- 공유 디렉토리 생성: `/www`, `/www/cgi-bin`  
- 파일 배포: `index.html`, `test.cgi`  
- 설정 관리: `/etc/exports` → 핸들러(`exportfs -r`)로 적용  
- 서비스 기동: `rpcbind`, `nfs-server`, `firewalld`  
- 방화벽 등록: `rpc-bind`, `nfs`, `mountd`  

---

### 2. **web**
- 패키지 설치: `httpd`, `firewalld`  
- 서비스 설정:  
  - `/etc/fstab` → **NFS 마운트 자동화**  
  - `/etc/httpd/conf.d/vhost.conf` 템플릿 배포  
  - CGI 실행 가능하도록 설정  
- 서비스 기동: `httpd`, `firewalld`  
- 방화벽 등록: `http`, `https`  

---

### 3. **lb (Load Balancer)**
- 패키지 설치: `haproxy`, `firewalld`  
- 설정 관리: `/etc/haproxy/haproxy.cfg` 템플릿 배포  
- 프론트엔드: `http_front` (포트 80)  
- 백엔드: `http_back` (라운드로빈, health check `/cgi-bin/test.cgi`)  
- 응답 헤더에 `X-Backend` 삽입  
- 서비스 기동: `haproxy`, `firewalld`  
- 방화벽 등록: `http`  

---

### 4. **dns**
- 패키지 설치: `bind`, `bind-utils`  
- Zone 파일 배포:
  - `lje.com.zone` → `www.lje.com` → LB IP (`192.168.10.12`)  
  - `10.168.192.rev` → 역방향 매핑  
- 서비스 기동: `named`  
- 방화벽 등록: `dns`  

---

## ⚙️ group_vars/all.yml 변수 설명

```yaml
# NFS 설정
nfs_server: 192.168.10.10
nfs_export: /www
nfs_allowed: 192.168.10.0/24

# LB 설정
lb_ip: 192.168.10.12

# DNS 설정
ns_servers:
  - { name: "ns1.lje.com", ip: "192.168.10.11" }
  - { name: "ns2.lje.com", ip: "192.168.10.14" }

fwd_zone:
  name: lje.com
  file: lje.com.zone

rev_zone:
  name: 10.168.192.in-addr.arpa
  file: 10.168.192.rev

# LB → WEB
web_backends:
  - { name: "web1", ip: "192.168.10.11", port: 80 }
  - { name: "web2", ip: "192.168.10.14", port: 80 }

web_vhosts:
  - { name: "vhost-80", port: 80, server_name: "www.lje.com" }
```
nfs_export : NFS 공유 디렉토리 경로

nfs_allowed : 접근 허용 CIDR 네트워크

web_backends : LB가 분산할 대상 웹 서버들

web_vhosts : Apache VirtualHost 정의

📦 Ansible Galaxy
각 Role은 Ansible Galaxy 표준 구조를 따릅니다.
배포 시 참고 가능한 Git 링크:

[nfs_server](https://github.com/kangbum01/ansible-role-nfs)

[web_server](https://github.com/kangbum01/ansible-role-web)

[lb_server](https://github.com/kangbum01/ansible-role-lb)

배포 시 참고 가능한 Galaxy 링크:

[nfs_server](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/nfs/)

[web_server](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/web/)

[lb_server](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/lb/)

(📌 실제 등록 후 주소 업데이트 필요)

✅ 실행 방법
1. inventory 수정
```yaml
[lb]
server3-LB ansible_host=192.168.10.12 

[nfs]
server1-NFS ansible_host=192.168.10.10 

[webservers]
server2-WEB ansible_host=192.168.10.11 
server4-WEB ansible_host=192.168.10.14

[all:vars]
ansible_user=ansible
```
2. Playbook 실행
```yaml
ansible-playbook -i inventory site.yml
```
3. 서비스 확인
```yaml
curl http://www.lje.com/cgi-bin/test.cgi
```
