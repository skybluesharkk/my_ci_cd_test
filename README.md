README.md
# my_ci_cd_test
ci/cd 테스트

# CI 절차
    - github 사이트 이동 -> 현 프로젝트로 이동
    - Settings > Security > Securits and variable > Actions
    - new repository 버튼 클릭
        - git에 노출되면 않되는 정보들
        - 환경변수 (노출되면 않되는 키정보, 디비정보등등)
        - aws 계정관리 iam에서 계정별로 access ID/KEY 발급받아서 등록 -> 외부에서 aws 엑세스 가능
        - action에 의해 .env등 설정 파일로 이관
            - 소스코드상에는 없다
        - 등록(필요한만큼, 수시로)
            - 이름, 값
    
    - gitaction에서 작동한 내용 기술
        - deploy.yml
        - 위치
            - 프로젝트루트/.github/workflows/deploy.yml
            - gitaction을 작동시키는 이벤트(명령, 트리거)는 서브 브런치일 경우 아래처럼 추가 가능
                ```
                pull_request:
                    branches: [dev]
                ```
            
            - jobs > steps > name => 검증할 내용들을 추가할수 있다
                - package.json 추가
                    - "build": "babel routes -d build"
                        - 개발된 소스를 표준
                        - npm install --save-dev babel-cli
                            - npm run build

# CD 절차
    - step 1 
        - 빌드한 소스코드, 리소스, 기타 설정파일 => 압축(zip)
    
    - step 2
        - aws 접속
            - IAM 계정상 엑세스키, 엑세스 ID가 gitaction 시크릿 변수에 등록되어있어야한다
            - 계정생성 => 키/아이디 발급 => 등록
            - 분실, 노출 금지
    
    - step 3
        - 압축(zip)한 리소스 => s3 업로드
    
    - step 4
        - AWS codeDeploy 서비스에 절차에 따라 배포 시작

    - AWS 세팅
        - S3 진입
            - 버킷 만들기 클릭
                - pusan001-deploy-bucket

        - EC2에 역활 부여
            - IAM 진입
                - 엑세스관리 > 역활 > 역활생성
                    - AWS 서비스
                        - 사용사례 > EC2 > 다음
                        - 권한추가 
                            - AmazonS3FullAccess 체크
                            - AWSCodeDeployFullAccess 체크
                            - 다음
                        - 이름 지정, 검토 및 생성
                            - 역활이름 : pusan_deploy
                            - 역활생성 클릭
            - EC2에 위에서 만든 역활을 보안에 적용
                - 인스턴스 선택
                - 작업>보안>IAM 역활수정
                - pusan_deploy 선택
                - IAM 역활 업데이트 클릭

        - CodeDeploy 역활 부여
            - IAM 진입
                - 엑세스관리 > 역활 > 역활생성
                    - AWS 서비스
                        - 사용사례 > CodeDeploy > 다음
                        - 권한추가 
                            - 다음
                        - 이름 지정, 검토 및 생성
                            - 역활이름 : pusan-code-deploy
                            - 역활생성 클릭
            - CodeDeploy 진입
                - 애플리케이션 > 애플리케이션 생성
                    - 이름 : pusan-code-deploy
                    - 플랫폼 : EC2/온프레미스
                - 배포그룹생성
                    - 이름 : dev
                    - 서비스 역활 : pusan-code-deploy
                - 환경구성
                    - Amazon EC2 인스턴스
                        - 키 : Name
                        - 값 : 인스턴스 지정 : aws-cloud9-node-....

                - 로드벨런서 체크 > 품   
                - 배포그룹생성 클릭

        - IAM 사용자 추가
            - 목적 : 엑세스키,ID 발급
            - IAM > 엑세스관리 > 사용자 > 사용자 추가
                - 이름 : user-cicd
                    - 다음
                - 권한옵션
                    - AWSS3FullAccess
                    - AWSCodeDeployFullAccess
                    - 다음
                - 사용자 생성
                    - 해당 유저로 진입
                        - 액세스 키 만들기
                        - 기타>다음
                        - 태그 
                            - access-cicd
                        - 액세스 키 AK...
                        - 비밀 키 DLzC..
                        - 위의 2개 값을 가지고 github에 환경변수 등록


# CD - EC2에서 수행할 작업
    - sudo apt install awscli
    - sudo aws configure
        - ID, 시크릿키, 리전, 형식(json) 입력
    - codeDeploy agent 
        - wget https://aws-codedeploy-ap-northeast-2.s3.amazonaws.com/latest/install
        - chmod +x ./install
        - sudo apt-get install ruby
        - sudo ./install auto
        - sudo service codedeploy-agent status