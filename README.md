# Ansible

# 1. Introduction

## (1) What is Ansible?

Ansible 은 IT 인프라 및 설정의 자동화 프로그램으로 네트워크 자동화가 가능하다. 네트워크의 자동화란 설정 자동화, 네트워크 상태 시험 자동화 등을 의미한다. Ansible 을 활용하면 여러 장비에 접속하여 네트워크 구성을 할 수 있다. Ansible 은 Agentless 하여 자동화 대상이 되는 네트워크 장비에 별도의 sw 설치가 필요하지 않는 장점을 지니고 있다.

### Managed Node, Control Node

Ansible Document 에서는 Managed Node 와 Control Node 로 자동화 대상과 자동화를 수행시키는 주체를 구분하고 있다. Managed Nodes 란, 자동화 대상이 될 타겟, 즉 제어할 장비들을 지칭한다. Ansible 이 동작하도록 자동화를 하기 위한 작업들이 설정된 곳을 Control Node 라고 한다.

Managed Nodes 에 SSH 접속을 통한 텍스트 기반의 명령어로 장비 제어가 이루어진다. 텍스트 기반의 명령어 전송이란, 사용자가 장비에 직접 CLI 를 통해 설정하듯 CLI text 가 Control Node 로부터 전달되어 장비에 설정이 되는 방식이다.

Control Node 내부에서 동작하는 Ansible 구성은 다음과 같다.

- Inventory : 제어할 장비에 대한 접속 정보 (IP, account/password)
    
    ex. 10.1.11.222, 10.1.11.71  (장비의 ip 주소 list)
    
- Module : 수행하고자 하는 액션의 단위
    
    ex. cli_config : 작성된 CLI text 를 장비로 전송
    
- Playbook : 자동화 수행을 위한 실행 항목 (task) 및 target 별 parameter 정의 (Yaml file)
    
    ![Untitled](Ansible%207bdb4661418b48acabe6802f0f4fcef7/Untitled.png)
    
    Ansible 은 Control Node 내 작성된 제어할 장비의 IP 주소, 장비 ID와 Password 등 접속 정보가 포함된 Inventory(1)와 타겟 장비에서 수행할 동작을 실행시키는 Module(2)이 명시된 Playbook(3)이 동작되어 자동화가 이루어진다.
    

## (2) Ansible Galaxy

Ansible Galaxy은 다양한 모듈을 (주로 벤더사가) 제공하는 커뮤니티이다. Ansible galaxy에서는 파이썬으로 구현된 다양한 모듈, Playbook 등을 제공하고 있어 Ansible 사용자는 상황에 맞는 모듈과 Playbook 을 선택하여 최소한의 수정으로 타겟에 바로 실행을 할 수 있다.

Ansible galaxy 에서 제공하는 모듈을 사용하고자 할 때는 다음과 같은 명령어를 사용한다.

```bash
$ ansible-galaxy collection install <namespace.collection>
```

Ansible Galaxy 는 각 벤더사가 제품에 따라 필요한 모듈들을 구현하여 제공하여 있으며, 주로 벤더들의 이름을 namespace 로 사용한다. 제공하는 모듈과 Playbook 의 집합을 “collection” 이라고 하며, 해당 네트워크 장비의 OS 이름이 사용된다. Ip Infusion 에서는 ipinfusion.ocnos 를 제공하고 있다.

# 2. Basic Concepts

## (1) Inventory

### What is inventory?

Inventory란 호스트, 즉 managed nodes를 관리하는 리스트를 의미한다. 아래와 같이 장비의 IP 주소가 나열된 파일을 예시로 들 수 있다.

```yaml
# this is comment
# Simple Inventory File for Routers
10.1.11.17
10.1.11.22
10.1.11.222
# and so on …
```

Ansible을 실행했을 때 Inventory 에 작성된 host 또는 group 이 자동화 대상으로 지정된다. Group 이란 host들의 집합 개념으로, 여러 host를 하나의 그룹으로 묶어 사용할 수 있다. Inventory를 작성하는 default 위치는 ‘etc/Ansible/hosts’ 이며 해당 파일에 host 정보를 추가할 수 있다.  ‘-i’ 옵션을 사용하는 경우 inventory 참고 위치를 설정 가능하다.

Host 연결 시 필요한 ID 나 password 와 같은 값들은 vars(variables)를 활용하여 전역 변수에 값을 정의하여 관리 가능하다. 공통되는 host 들을 group 으로 묶어 관리하듯, 호스트 간 공통되는 variable 값들을 group_vars 로 정의하여 관리할 수 있다.

### Writing inventory

다음은 각각 10.1.11.222, 10.1.11.71 주소를 가진 라우터를 제어하고자 할 때 inventory 예시이다. 해당 예시에서는 “hosts” 파일에 inventory 를 작성하였고, Ansible 을 실행할 때 ‘-i’ 옵션으로 inventory 파일이 위치한 경로를 명시하였다.

```bash
ansible-playbook –i inventory/ ocnos_command.yml
```

다음은 10.1.11.222, 10.1.11.71 IP 주소를 가진 장비를 타겟으로 “show hostname” 과 “show ip route” CLI 를 전송하고자 하는 예시이다.

![Untitled](Ansible%207bdb4661418b48acabe6802f0f4fcef7/Untitled%201.png)

## (2) Playbook

Playbook은 하나의 스크립트 개념으로, Playbook 을 통해 여러 장비를 동시에, 여러 번 제어 가능하다. Playbook은 제어하고자 하는 대상 target과 target 에 실행하고자 하는 1개 이상의 task 로 구성된다.

![Untitled](Ansible%207bdb4661418b48acabe6802f0f4fcef7/Untitled%202.png)

Playbook 에 명시된 task 내의 module 들이 순차적으로 실행된다. Module 이란, 타겟에서 실행되는 action의 단위를 의미하며, 각 module 은 python으로 구현되어 있어 해당 모듈의 python code 가 호출되어 실행되는 방식으로 동작한다.

# 3. Installing OcNos

## (1) Prerequisites

### Control Nodes

Control Nodes, 즉 Ansible을 실행시키는 장비에서의 조건이다.

- Python 3 (**versions 3.5 and higher**) installed
- 본 문서에서의 예시 실행 환경
    - Ubuntu 20.04
    - linux 5.8.0-55-generic
    - python 3.8.10
    - ansible 2.10.6
    - ansible-galaxy 2.10.6

### Managed Nodes

Ansible을 통해 자동화를 수행하고자하는 대상 장비에서의 조건이다. Ansible은 SSH 접속을 통해 managed node에 연결이 되므로, 장비에 따라 ssh 접속 허용을 해야하는 경우, 허용해 놓는다.

## (2) Installing Ansible on Ubuntu

### Install python

### Install paramiko using pip

### Install Ansible

```bash
sudo apt install ansible
```

## (3) Installing OcNOS from Ansible galaxy

```bash
ansible-galaxy collection install ipinfusion.ocnos
```

### 설치한 collection 확인하기

```bash
ansible-galaxy collection list
```

# 4. Automation on Network Configuration

## (1) LDP Configuration

본 예시는 Ansible 의 기본 모듈(cli_config)을 활용하여 2개의 라우터 장비에 LDP CLI를 text 로 전송하여 설정하는 예시이다.

### Overall

LDP configuration 을 진행하는 과정은 다음과 같다.

1. Ansible Galaxy에서 필요한 collection 을 설치한다. 
2. 이후 타겟 장비의 IP 주소, 접속하기 위해 필요한 ID/password 등을 설정한다. 또한 인터페이스 이름, LDP targeted-peer 등 LDP 설정을 할 때 필요한 정보를 각 host 의 variable 파일에 작성하여 Managed nodes 관련 설정을 마친다. 
3. CLI 는 text based 로 장비에 전송이 되기 때문에 전송할 CLI 를 jinja template 파일에 작성한다. 
4. Playbook에는 Managed nodes 정보가 담긴 inventory, 수행하고자 하는 모듈(cli_config)과 모듈을 실행시킬 때 사용할 CLI text 가 담긴 template 을 명시한다. 
5. 마지막으로 Ansible-playbook 을 실행시켜 제어를 진행한다.

### 관련 collection 설치

```bash
ansible-galaxy collection install ipinfusion.ocnos
ansible-galaxy collection install ansible.netcommon
```

### Inventory 작성: 적용할 타겟 명시

Inventory 파일 관리 구조는 다음과 같다. ‘hosts-net’ 파일에는 host들의 IP 주소가 작성되어 있다. ‘hosts-vars’ 디렉터리에는 각 host 에 대한 variable 정보, ‘group_vars’ 디렉터리에는 host를 묶어놓은 그룹(“E7100”)에 대한 group variable 정보가 담겨있다.

![파일 구조](Ansible%207bdb4661418b48acabe6802f0f4fcef7/Untitled%203.png)

파일 구조

### Jinja template 작성: 전송할 CLI config 작성

Jinja template (.j2)에 장비에 전송할 CLI config를 작성한다.

```bash
$ cat templates/ocnos_ldp.j2
{%if ldp is defined%}
router ldp
  {% for peer in ldp.peers -%}
  targeted-peer ipv4 {{ peer.address }}
    exit
  {% endfor %}
exit
{% for interface in ldp_interfaces -%}
interface {{ interface.name }}
  enable-ldp {{ interface.protocol }}
label-switching
exit
{% endfor %}
{%endif%}
```

### Playbook 작성: 적용할 타겟과 수행하고자 하는 모듈 명시

```bash
$ cat ldp-playbook.yml 
---
- hosts: E7100
  gather_facts: false 
  tasks:
  - name: configure LDP config on OcNOS 
    cli_config:
      config: "{{ lookup('template', 'templates/ocnos_ldp.j2') }}"
```
