# AWS

AWS����Ƃ��āAVPC + EC2�𖳗��g�ō\�z���āA�N���C�A���g�[������SSH�ڑ�������@�B

![VPC+EC2.png](VPC+EC2.png)


## VPC

�yAWS�@�z�l�b�g���[�N���\�z���Ă݂�  
https://zenn.dev/oreo2990/articles/bf3112bb6ccb48  

### IP�A�h���X

- VPC             :10.0.0.0/16 65,536��IP
- PUBLIC  SUBNET 1:10.0.1.0/24    256��IP
- PUBLIC  SUBNET 2:10.0.2.0/24    256��IP
- PRIVATE SUBNET 3:10.0.3.0/24    256��IP
- PRIVATE SUBNET 4:10.0.4.0/24    256��IP

### AZ

- PUBLIC  SUBNET 1: ap-northeast-1a
- PUBLIC  SUBNET 2: ap-northeast-1b
- PRIVATE SUBNET 3: ap-northeast-1a
- PRIVATE SUBNET 4: ap-northeast-1b

### ���[�g�e�[�u���Aigw

- `VPC`��igw���A�^�b�`����Ă��邱��
- `VPC`�̃��[�g�e�[�u����igw���ݒ肳��Ă��邱��
- `PUBLIC SUBNET`�̃��[�g�e�[�u����igw���ݒ肳��Ă��邱��
- `PRIVATE SUBNET`�̃��[�g�e�[�u���ɂ́Aigw���ݒ肳��Ă��Ȃ�����  
  ��igw=�C���^�[�l�b�g�Q�[�g�E�F�C

### ACL

���ׂċ���

### �Z�L�����e�B�O���[�v

- �z�u��T�u�l�b�g��T�[�o��ނ̒P�ʂɍ쐬
- �Ⴆ�΁A��Ɨp�ɃC���o�E���h��SSH��������

### �\��

- �Ⴆ�΁APUBLIC  SUBNET�ɂ́AAPI GW�AALB�AWEB�T�[�o�A���ݑ�
- �Ⴆ�΁APRIVATE SUBNET�ɂ́ADB�AEC2�����T�[�o�iAP�T�[�o�A�o�b�`�T�[�o�j�AECS�i�T�[�r�X�A�^�X�N�j


## EC2

�yAWS�zEC2�C���X�^���X�̍쐬���@����I�T�[�o�[���쐬���Đڑ����Ă݂�  
https://engineer-ninaritai.com/aws-ec2-make/  

- VPC��I��
- SUBNET��I��
- �Z�L�����e�B�O���[�v��I��
- OS
- �C���X�^���X�^�C�v ��1�N�Ԗ����g
- �X�g���[�W�@ ��1�N�Ԗ����g
- �L�[�y�A

�ڑ����@  
```
ssh -i key.pem ec2-user@IP�A�h���X
```

���炩�̃A�v���N�����@
```
sudo yum install git
sudo yum install java-11-amazon-corretto.x86_64
git clone https://github.com/namickey/spring-boot-mybatis-docker.git
cd spring-boot-mybatis-docker
cd web-css
./mvnw spring-boot:run

�u���E�U����A�N�Z�X
`http://�p�u���b�NIP:8080/`
```


### PUBLIC SUBNET�֔z�u

- OS�́A`Amazon Linux 2`
- �C���X�^���X�^�C�v�́At2.micro
- VPC�́A1�Œ�
- �T�u�l�b�g�́A`PUBLIC SUBNET`��1 or 2
- Public IP��t�^����
- Elastic IP�͕t�^���Ȃ��Ă悢
- �X�g���[�W��8GB
- �Z�L�����e�B�O���[�v�́APUBLIC������SSH�����B�C���^�[�l�b�g����A�N�Z�X�����邽�ߑ��M��IP�͂ł��邾�����肷��B
- �A�N�Z�X���@�́A�Ⴆ�Ύ���}�V������SSH����B

### PRIVATE SUBNET�֔z�u

- OS�́A`Amazon Linux 2`
- �C���X�^���X�^�C�v�́At2.micro
- VPC�́A1�Œ�
- �T�u�l�b�g�́A`PRIVATE SUBNET`��1 or 2
- Public IP�͖���
- Elastic IP�͖���
- �X�g���[�W��8GB
- �Z�L�����e�B�O���[�v�́APRIVATE������SSH�����B���M����PUBLIC�̃Z�L�����e�B�O���[�v���w��B
- �A�N�Z�X���@�́A�Ⴆ�΁A�܂�����}�V������PUBLIC SUBNET��EC2��SSH���āA���ɂ���EC2������PRIVATE SUBNET��EC2��SSH����B  
  ��PUBLIC��EC2��`���ݑ�`�ƌĂ�


�ڑ����@  
```
scp -i key.pem key.pem ec2-user@publicIP�A�h���X�iPUBLIC��EC2�j:~/
ssh -i key.pem ec2-user@IP�A�h���X�iPUBLIC��EC2�j

PUBLIC��EC2 > chmod 400 key.pem
PUBLIC��EC2 > ssh -i key.pem privateIP�A�h���X�iPRIVATE�^EC2�j
```


### �R�X�g�A�����g


- �l�b�g���[�N
  - �����FVPC
  - �����FSUBNET
  - �����F���[�g�e�[�u��
  - �����F�C���^�[�l�b�g�Q�[�g�E�F�C
  - �����FACL
  - �����F�Z�L�����e�B�O���[�v
  - �����F�ʐM�i�O��AWS�j
  - �����g�F�ʐM�iAWS�ˊO�j
  - �����F�ʐM�i����AZ���j
  - `�L��`�F�ʐM�i�قȂ�AZ�ԁj
- EC2
  - �����FPublic IP
  - `�L��`�FElastic IP
  - �����g�FEC2�C���X�^���X
  - �����g�FELB�i�X�g���[�W�j

- WEB API
  - �����g�FAPI Gateway
- CloudWatch
  - �����g�FCloudWatch Logs
- �X�g���[�W
  - �����g�FS3
- �f�[�^�x�[�X
  - �����g�FRDS
  - `�L��`�FAurora
- �R���e�i
  - `�L��`�FECS, Fargate
  - �����g�FECR
- �l�b�g���[�N
  - �����g�FALB
  - `�L��`�FNAT�Q�[�g�E�F�C
  - `�L��`�FECR VPC�G���h�|�C���g
  - `�L��`�FCloudWatch VPC�G���h�|�C���g
  - �����FS3 VPC�G���h�|�C���g
  - �����FACM
  - `�L��`�FROUTE53
- ���̑�
  - `�L��`�FSecrets Manager


### PRIVATE SUBNET���g���Ȃ�A`NAT�Q�[�g�E�F�C`or`VPC�G���h�|�C���g`���K�v

���̃g���t�B�b�N�ANAT�Q�[�g�E�F�C��ʂ��K�v����܂����H�K�؂Ȍo�H�ŕs�v�ȃf�[�^���������͍팸���܂��傤  
https://dev.classmethod.jp/articles/reduce-unnecessary-costs-for-nat-gateway/  

ECS+fargate+�v���C�x�[�g�T�u�l�b�g�ŃR���e�i�𗧂ĂăA�N�Z�X����  
https://blog.not75743.com/post/ecs_private/  

- NAT�Q�[�g�E�F�C�����FPRIVATE SUBNET�ɔz�u���ꂽEC2����C���^�[�l�b�g�A�N�Z�X�ł��Ȃ��B`yum update`���ł��Ȃ��BECR����pull�ł��Ȃ��BS3��Cloud Watch���_���B
- NAT�Q�[�g�E�F�C�L��FPRIVATE SUBNET�ɔz�u���ꂽEC2����C���^�[�l�b�g�A�N�Z�X�ł���B�L���ō����B

- VPC�G���h�|�C���g�����FPRIVATE SUBNET�ɔz�u���ꂽEC2����C���^�[�l�b�g�A�N�Z�X�ł��Ȃ��B`yum update`���ł��Ȃ��BECR����pull�ł��Ȃ��BS3��Cloud Watch���_���B
- VPC�G���h�|�C���g�L��FPRIVATE SUBNET�ɔz�u���ꂽEC2����C���^�[�l�b�g�A�N�Z�X�ł��Ȃ��B`yum update`���ł��Ȃ��BECR�AS3�ACloud Watch�͎g����悤�ɂȂ�B


### ALB



python3 -m http.server










