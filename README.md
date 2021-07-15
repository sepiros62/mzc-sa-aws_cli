# AWS CLI 환경 구성을 자동화 해주는 Ansible Playbook

## Ansible Role

ㅁ `aws_repo` 역할은 EPEL 저장소를 설치합니다. [EPEL REPO](https://fedoraproject.org/wiki/EPEL)

ㅁ `aws_cli` 역할은 AWS 명령줄 인터페이스(CLI)를 설치합니다. [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)

ㅁ `aws_config` 역할은 AWS 명령줄 인터페이스(CLI)을 구성하는데 필요한 파일을 생성합니다. [AWS Configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

## Requirements

RedHat, CentOS, Debian 및 Ubuntu 배포판에서 작동합니다. 
Amazone Linux는 AWS CLI, REPO를 포함하고 있으므로 AWS CLI 구성 파일 생성 작업이 가능합니다.

* RedHat
  * 6
  * 7
  * 8
* CentOS
  * 6
  * 7
  * 8
* Ubuntu
  * 14.*
  * 16.*
  * 18.*
  * 19.*
* Debian
  * jessie (8)
  * stretch (9)
  * buster (10)
  * sid (unstable)

Ansible Playbook 실행을 위해 Ansible, Python, Git이 설치되어 있는지 확인하고 없으면 아래와 같이 설치를 진행합니다.
  * Amazone Linux: `amazon-linux-extras install ansible2 -y; yum install git -y`
  * CentOS / REDHAT: `yum install epel-release -y; yum install ansible git -y`
  * Ubuntu: `apt install software-properties-common; apt-add-repository --yes --update ppa:ansible/ansible; apt install ansible python2.7; ln -s /usr/bin/python /bin/python`
  * Debian: `deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main; apt install ansible`

**ansible install url**: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

## Role Variables

| Variable                           | Default Value  |
| :-------------------------         | :------------- |
| epel_enable_redhat_extras_repos | false             |
| awscliconf_path                    | "~/."          |
| awscliconf_recursive_path_creation | false          |
| awscliconf_files_owner             | "root"         |
| awscliconf_files_group             | "root"         |
| awscliconf_files                   | "" --> Empty   |
자세한 내용은 `aws_config/default/main.yml` 파일을 참조하세요.

## Dependencies

* RedHat / CentOS
  * 6/7 EPEL 저장소 EPEL 저장소를 활성화해야합니다. 이 작업을 위해 `aws_repo` 역할을 사용할 수 있습니다 .

**Note:** RedHat / CentOS 8에서는이 역할을 사용하기 위해 EPEL Repository가 필요하지 않습니다.

## Playbook Pre-Preparation Task

* 소스 코드를 Git Clone으로 다운로드 
```bash
$ git clone https://github.com/sepiros62/mzc-sa-tools.git
$ cd mzc-sa-tool
```

* `site.yml` 파일에서 AWS CLI 구성을 위한 자격 증명(`credentials`), 설정(`config`) 정보를 업데이트
  * 자격 증명(`credentials`): 사용자 정의 profile (이 예제에서는 default, jaehwan 2개의 profile 정의)
  * 설정(`config`): 사용자 정의 profile의 설정 (이 예제에서는 각 profile의 설정으로 region, output, source_profile을 정의)

```yaml
---
- name: 'AWS CLI Environment Configuration Playbook'
  hosts: 'localhost'
  gather_facts: true

  vars:
    - cli_package: 'awscli'
    - ansible_python_interpreter: '/bin/python'

  vars_prompt:
    - name: "select"
      prompt: |
        !!! Choose the number for the task !!!
           [ 0 - ALL ]
           [ 1 - EPEL-REPO Install ]
           [ 2 - AWS CLI Install ]
           [ 3 - AWS CLI Configure ]
      private: false

  roles:
    - role: 'aws_repo'
      tags: [ all, aws_repo ]
      when: select == '0' or select == '1'

    - role: 'aws_cli'
      tags: [ all, aws_cli ]
      when: select == '0' or select == '2'

    - role: 'aws_config'
      tags: [ all, aws_config ]
      vars:
        awscliconf_files:
          credentials:
            - default:
                aws_access_key_id: '{ ACCESS KEY ID}'
                aws_secret_access_key: '{ SECRET ACCESS KEY }'
            - jaehwan:
                aws_access_key_id: '{ ACCESS KEY ID}'
                aws_secret_access_key: '{ SECRET ACCESS KEY }'
          config:
            - default:
                region: ap-northeast-2
                output: table
            - jaehwan:
                region: ap-northeast-2
                output: json
            - profile development:
                region: ap-northeast-2
                source_profile: default
      when: select == '0' or select == '3'
```

## How to run the playbook

### All-in-one 작업 (`0` 입력)
* 한번에 모든 작업이 실행되는 방식으로  `aws_repo`, `aws_cli`, `aws_config` 3가지 Role이 모두 실행 됩니다.
```bash
[root@awx-server mzc-sa-tools]# ansible-playbook site.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
!!! Choose the number for the task !!!
   [ 0 - ALL ]
   [ 1 - EPEL-REPO Install ]
   [ 2 - AWS CLI Install ]
   [ 3 - AWS CLI Configure ]
: 0

...
...

PLAY RECAP ***************************************************************************************************************************************
localhost                  : ok=15   changed=0    unreachable=0    failed=0    skipped=21   rescued=0    ignored=0
```

### EPEL 저장소 설치 작업 (`1` 입력)
* EPEL 저장소 설치를 위해 `aws_repo` Role이 실행 됩니다. 
```bash
[root@awx-server mzc-sa-tools]# ansible-playbook site.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
!!! Choose the number for the task !!!
   [ 0 - ALL ]
   [ 1 - EPEL-REPO Install ]
   [ 2 - AWS CLI Install ]
   [ 3 - AWS CLI Configure ]
: 1

PLAY [AWS CLI Environment Configuration Playbook] ************************************************************************************************
...
...

PLAY RECAP ***************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=28   rescued=0    ignored=0
```

### AWS CLI 설치 작업 (`2` 입력)
* AWS CLI 설치를 위해 `aws_cli` Role이 실행 됩니다. 
```bash
[root@awx-server mzc-sa-tools]# ansible-playbook site.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
!!! Choose the number for the task !!!
   [ 0 - ALL ]
   [ 1 - EPEL-REPO Install ]
   [ 2 - AWS CLI Install ]
   [ 3 - AWS CLI Configure ]
: 2

PLAY [AWS CLI Environment Configuration Playbook] ************************************************************************************************
...
...

PLAY RECAP ***************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=28   rescued=0    ignored=0
```

### AWS CLI 구성을 위한 설정 작업 (`3` 입력)
* AWS CLI 구성을 위한 `aws_config` Role이 실행 됩니다. 
```bash
[root@awx-server mzc-sa-tools]# ansible-playbook site.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
!!! Choose the number for the task !!!
   [ 0 - ALL ]
   [ 1 - EPEL-REPO Install ]
   [ 2 - AWS CLI Install ]
   [ 3 - AWS CLI Configure ]
: 3

PLAY [AWS CLI Environment Configuration Playbook] ************************************************************************************************
...
...

PLAY RECAP ***************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=28   rescued=0    ignored=0
```

## Final Result Check
* AWS CLI 각 profile별 동작 확인
```bash
[root@awx-server mzc-sa-tools]# aws s3 ls --profile jaehwan
2020-06-10 04:11:13 b-abc-shop.cf
2020-08-11 06:39:26 cloud9-051542137113-sam-deployments-us-east-1
2020-08-10 08:26:09 gcc-service-image
2020-07-20 07:11:50 ssm-test-109

[root@awx-server mzc-sa-tools]# aws s3 ls --profile default
2020-07-15 02:30:12 b-abc-shop.cf
2020-08-11 06:39:26 cloud9-051542137113-sam-deployments-us-east-1
2020-08-11 09:43:45 gcc-service-image
2020-07-21 09:14:57 ssm-test-109
```

* AWS CLI의 자동 명령 완성 기능 확인 ( aws 명령어 입력후 `TAB` 을 눌러 자동 완성)
    * 자동 완성 기능이 동작하지 않을 경우 BASH 재접속하여 확인
```bash
[root@awx-server mzc-sa-tools]# aws s
s3                     savingsplans           servicecatalog         signer                 sso                    swf
s3api                  schemas                servicediscovery       sms                    sso-oidc               synthetics
s3control              sdb                    service-quotas         snowball               stepfunctions
sagemaker              secretsmanager         ses                    sns                    storagegateway
sagemaker-a2i-runtime  securityhub            sesv2                  sqs                    sts
sagemaker-runtime      serverlessrepo         shield                 ssm                    support

[root@awx-server mzc-sa-tools]# aws s3
cp       ls       mb       mv       presign  rb       rm       sync     website
```

## Author Information

* [Jaewhan Jeong](https://github.com/sepiros62) 
