📌 WEB Role (ansible-role-web)
WEB Role
=========

이 롤은 CentOS/RHEL 계열 시스템에서 **Apache HTTPD(Web Server)** 를 설치하고,  
NFS 마운트를 기반으로 가상 호스트(VirtualHost)를 설정하여 웹 서비스를 제공합니다.

요구 사항 (Requirements)
------------------------

- 지원 OS: CentOS 8/9, RHEL 8/9 계열
- Ansible 버전: 2.14 이상
- `httpd`, `mod_ssl`, `nfs-utils`, `firewalld` 패키지가 시스템 저장소에서 사용 가능해야 함
- NFS 서버(`nfs_server`)가 사전에 준비되어 있어야 함

그룹 변수 (Group vars)
------------------------
다음과 같은 변수를 설정할 수 있습니다. (예시 값 포함)
```yaml
# NFS 설정
nfs_server: 192.168.10.10
nfs_export: /www
nfs_allowed: 192.168.10.0/24

# LB 설정
lb_ip: 192.168.10.12

# DNS 설정
ns_server:
  domain: lje.com
  name: ns1.lje.com
  ip: 192.168.10.13
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


롤 변수 (Role Variables)
------------------------

다음과 같은 변수를 설정할 수 있습니다. (예시 값 포함)

```yaml
# NFS 서버와 공유 디렉토리
nfs_server: 192.168.10.10
nfs_export: /www
```
VirtualHost 설정
------------------------
```yaml
web_vhosts:
  - { name: "vhost-80", port: 80, server_name: "www.lje.com" }
```

의존 관계 (Dependencies)
------------------------
없음.
단독으로 사용 가능하지만, 다음과 같은 롤과 함께 사용하면 WEB-LB-NFS 아키텍처를 완성할 수 있습니다:

- [NFS 서버 롤 - git](https://github.com/kangbum01/ansible-role-nfs)
- [NFS 서버 롤 - ansible-galaxy](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/nfs/)
- [로드밸런서 롤](https://github.com/kangbum01/ansible-role-lb)
- [로드밸런스 롤 - ansible-galaxy](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/lb/)

예시 플레이북 (Example Playbook)
------------------------
```yaml
- hosts: web
  become: true
  roles:
    - role: kangbum01.web
      vars:
        nfs_server: 192.168.10.10
        nfs_export: /www
        web_vhosts:
          - { name: "vhost-80", port: 80, server_name: "www.lje.com" }
```
라이선스 (License)
------------------------
MIT

작성자 정보 (Author Information)

작성자: kangbum01
GitHub: https://github.com/kangbum01
