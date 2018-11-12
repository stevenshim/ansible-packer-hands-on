
### 사전 준비 1 - CloudFormation Template을 이용해 VPC 네트워크 준비하기 (5~6 mins)
> 먼저 Hands-on을 진행할 VPC를 미리 만드신 분은 CloudFormation을 이용해 VPC 생성 과정을 진행하지 않으셔도 됩니다.

> NAT Gateway가 Ansible Hands-on 에는 꼭 필요하지 않으나, Script에는 NAT Gateway 생성을 포함하므로 **비용** 이 발생합니다. 

VPC 구성은 Public Subnet 2개 / Private Subnet 2개 / NAT Gateway 1개__(비용 발생)__ 로 구성됩니다. <br>

1. AWS Console > CloudFormation으로 이동합니다.
2. Create Stack > Upload a template to Amazon S3 에서 [cloudformation_vpc/01.vpc.yml](cloudformation_vpc/01.vpc.yml)을 Upload 후 다음 단계로 진행 합니다.
3. Specify Details에서 Stack Name과 ClusterStack 이름을 기입합니다.
<br> 이 값은 vpc 이름 및 subnet 이름 등에 사용됩니다.
4. VPC CIDR 및 PublicSubnet / PrivateSubnet 등 미리 입력된 정보는 바로 사용이 가능합니다.<br>
원하실 경우 적절히 바꿔서 쓰시면 됩니다.<br>
Next로 다음 단계로 이동 합니다.
5. 다음 단계는 입력 없이 Next로 넘어갑니다.
6. Review 화면에서는 기입력된 값을 검토 하시고 Create 로 VPC를 생성합니다. 약 3~4분 정도 소요 됩니다.

### 사전 준비 2 - EC2 생성하기
Hands-on 과정을 위해 EC2는 __2대__ 를 생성 합니다.<br>
1대는 ansible playbook을 작성하고 관리하는 ansible-server. <br>
다른 1대는 ansible playbook을 이용해 provisioning 할 목적의 target-server의 역할을 합니다.

1. AWS Console 접속
2. 서울 리전(ap-northeast-2)으로 이동
3. AWS EC2 Console 로 이동
4. EC2 생성하기 - Amazon Linux AMI 2018.03.0 으로 생성
5. EC2 Type - t2.micro 로 선택
6. 기 생성된 VPC 중 Public Subnet 에 위치하도록 선택
7. Auto-assign Public IP 옵션 True
8. Security Group 에 22번 포트 접속 가능하도록 설정
9. Keypair 생성 후 다운로드 - 편의를 위해 2대 모두 같은 keypair로 선택
10. EC2 생성 완료

> EC2 인스턴스 2대 중 1대는 이름을 Ansible 으로 정합니다.<br>
나머지 인스턴스 한 대는 Target 으로 정합니다.
