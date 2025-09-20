LB Role
=========

이 롤은 CentOS/RHEL 계열 시스템에서 **HAProxy**를 설치하고 Layer 4 로드밸런서를 구성합니다.  
또한 `firewalld`에 HTTP 서비스를 등록하고, 포트 충돌 방지를 위해 `httpd` 서비스를 중지/비활성화합니다.

요구 사항 (Requirements)
------------------------

- 지원 OS: CentOS 8/9, RHEL 8/9 계열
- Ansible 버전: 2.14 이상
- `firewalld` 패키지가 시스템 저장소에서 사용 가능해야 함

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
# HAProxy가 분산할 웹 서버 목록
web_backends:
  - { name: "web1", ip: "192.168.10.11", port: 80 }
  - { name: "web2", ip: "192.168.10.14", port: 80 }
```

의존 관계 (Dependencies)
------------------------
없음.
단독으로 동작하지만, 다음과 같은 롤과 함께 사용하면 WEB-LB-NFS 아키텍처를 완성할 수 있습니다:

- [NFS 서버 롤 - git](https://github.com/kangbum01/ansible-role-nfs)
- [NFS 서버 롤 - ansible-galaxy](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/nfs/)
- [WEB 롤 - git](https://github.com/kangbum01/ansible-role-web)
- [WEB 롤 - ansible-galaxy](https://galaxy.ansible.com/ui/standalone/roles/kangbum01/web/)

예시 플레이북 (Example Playbook)
------------------------
```yaml
- hosts: lb
  become: true
  roles:
    - role: kangbum01.lb
      vars:
        web_backends:
          - { name: "web1", ip: "192.168.10.11", port: 80 }
          - { name: "web2", ip: "192.168.10.14", port: 80 }
```

라이선스 (License)
------------------------
MIT

작성자 정보 (Author Information)

작성자: kangbum01
GitHub: https://github.com/kangbum01
