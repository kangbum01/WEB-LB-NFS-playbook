## **DNS 서버 구축 Role**

Ansible을 사용하여 CentOS 9 환경에서 **BIND(named) DNS 서버**를 설치하고 기본 구성을 자동화하는 Role입니다.  
포워드 존과 리버스 존 설정을 포함하며, 방화벽 규칙 추가 및 DNS 서비스 활성화를 지원합니다.


### **요구 사항**
---
- **Ansible 2.14 이상**  
  이 Role은 Ansible 2.14 버전 이상에서 테스트되었습니다.
- **CentOS 9 Stream 환경**  
  제어 노드 및 관리 노드는 CentOS 9 Stream 기반이어야 합니다.
- **root 권한**  
  작업 실행을 위해 `become: true` 설정이 필요합니다.


### **Role 변수**
---
이 Role에서 사용할 수 있는 주요 변수는 다음과 같습니다.  
(`defaults/main.yml`에 기본값이 정의되어 있으며, 필요 시 플레이북에서 덮어쓸 수 있습니다.)

```yaml
# 포워드 존 설정
fwd_zone:
  name: lje.com
  file: lje.zone

# 리버스 존 설정
rev_zone:
  name: 10.168.192.in-addr.arpa
  file: lje.rev

# 네임서버 레코드 정보
ns_server:
  domain: lje.com
  name: ns1.lje.com
  ip: 192.168.10.11
```

### **종속성**
---
이 Role은 다른 종속 Role이 없습니다.


### **플레이북 예시**
---
```yaml
---
- name: Configure the DNS Server
  hosts: ansible1.example.com
  tasks:
    - name: Run DNS server setup role
      ansible.builtin.include_role:
        name: leeeeejieun.dns
```
