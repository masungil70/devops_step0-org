# project name : devops_step0

1. github에 저장소(devops_step0)를 생성한다
2. 로컬PC에 스프링 부트로 프로젝트(devops_step0)를 생성한다
3. 로컬PC에서 생성한 프로젝트 폴더로 이동한다
4. 만약 위 2. 3.번을 생략하고 나의 저장소 있는 소스를 clone하여 자신의 저장소에 작업을 하려면 다음과 같이한다
   1. git clone https://github.com/masungil70/devops_step0.git
   2. cd devops_step0
   3. .git폴더 삭제한다 
      1. win의 경우 rm /S /Q .git
      2. mac 또는 linux 경우  rm -rf .git
5. 원격저장소의 github persnal access tokens은 반드시 workflow가 선택된 권한이 있어야 한다(없으면 추가할 것)
6. 로컬 저장소 생성하고 추가한 후 원격저장소를 등록하고 push 한다
   1. git init
   2. git add .
   3. git commit -m "first commit"
   4. git branch -M main
   5. git remote add origin https://github.com/{{계정명}}/devops_step0.git
   6. git push -u origin main

# 빌드와 배포를 수동으로 작업하는 위해서는 ssh를 사용하여 작업을 진행합니다 
  1. ssh로 서버에 연결 할 때 비밀번호를 입력을 하는데 비밀 번호를 입력하지 않고 개인인증키를 사용하여 접근하는 방식을 사용하면 조금 편하게 연결할 수 있음
  2. ssh 접속을 위한 개인인증키와 공개키를 생성한다 -> [ssh-keygen 사용법 참조](https://handari.tistory.com/8)
  3. 생성된 공개키를 접속을 하려고 하는 서버에 등록(~/.ssh/autorized_keys 파일에 공개키 추가함)한다
  4. 로컬PC에 있는 파일을 서버에 secure copy  명령(scp)으로 복사할 있음 (이것이 실제 배포하는 방법임)
  5. ssh로 접속하여 복사한 파일을 사용하여 실행 하면 배포 작업은 완료이 완료됨
  6. 실제 작업 순서 (빌드 서버)
     1. mkdir ~/source-folder (소스 폴더를 생성한다)
     2. cd ~/source-folder (소스 폴더로 이동한다)
     3. git init (git 저장소를 생성한다)
     4. git remote add origin {github 저장소 주소} (원격저장소를 추가하는 명령어입니다)
     5. git fetch origin (원격 저장소의 최신 변경 사항을 가져오는 명령어입니다)
     6. git checkout main (원격저장소의 origin/main 브랜치로 체크 아웃하는 명령어입니다)
     7. ./gradlew build (스프링 프로젝트를 빌드합니다. 최종 결과물은 build/libs/*.jar 파일 생성)
     8. scp -i ~/.ssh/id_ras build/libs/*.jar {계정}@{배포서버아이피}:/home/{계정}/app (id_ras : ssh 개인키파일명, app 폴더명은 배포서버의 파일이 복사되는 곳의 폴더명, 사전에 폴더가 있어야 됨)
     9. ssh -i ~/.ssh/id_ras build/libs/*.jar {계정}@{배포서버아이피} (배포서버에 연결함)
     10. java -jar ~/app/{복사된파일}.jar (배포한 프로그램 실행)

# 위 수동작업을  GitHub Actions를 이용하여 Ubuntu 서버에 배포하고 실행하는 방법

1. GitHub Actions 워크플로우 설정
2. Ubuntu 서버 설정
3. 배포 스크립트 작성

*1. GitHub Actions 워크플로우 설정
먼저, 프로젝트의 루트 디렉토리에 .github/workflows 디렉토리를 만들고, 여기에 배포 워크플로우 파일을 생성합니다. 예를 들어 deploy.yml 파일을 생성합니다.

<hr/>

파일명 : deploy.yml 

```
name: Deploy to Ubuntu Server

on:
  push:
    branches:
      - main  # main 브랜치에 푸시될 때 트리거

jobs:
  build:
    runs-on: ubuntu-latest

    steps: 
    # 코드 체크아웃
    - name: Checkout repository
      uses: actions/checkout@v2

    # JDK 11 설정
    - name: Set up JDK 17
      uses: actions/setup-java@v1
      with:
        java-version: '17'

    - name: Build with Gradle
      run: ./gradlew build

    - name: Deploy to Server
      uses: appleboy/scp-action@v0.1.1
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SERVER_SSH_KEY }}
        port: ${{ secrets.SERVER_PORT }}
        source: "build/libs/*.jar"
        target: "~/app/"

    - name: Run SSH commands
      uses: appleboy/ssh-action@v0.1.3
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SERVER_SSH_KEY }}
        port: ${{ secrets.SERVER_PORT }}
        script: |
          pkill -f 'java -jar' || true
          nohup java -jar ~/app/devops_step1-0.0.1-SNAPSHOT.jar > log.txt 2>&1 &
```

위의 deploy.yml 파일은 GitHub Actions를 사용하여 다음 작업을 수행합니다:



Gradle을 사용하여 빌드
빌드된 .jar 파일을 서버로 전송
서버에서 Spring Boot 애플리케이션 실행
<hr/>
2. Ubuntu 서버 설정
서버에 필요한 환경을 설정합니다.

Java 설치: 서버에 Java를 설치해야 합니다.

sh
sudo apt update
sudo apt install openjdk-17-jdk
SSH 설정: GitHub Actions가 서버에 접속할 수 있도록 SSH 키를 설정합니다. 로컬 머신에서 SSH 키를 생성하고, 이를 GitHub Secrets에 추가합니다.

sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
~/.ssh/id_rsa.pub 파일의 내용을 서버의 ~/.ssh/authorized_keys에 추가합니다.

GitHub Secrets 설정: GitHub 리포지토리 설정에서 아래와 같은 Secrets을 추가합니다.
SERVER_HOST: 서버의 IP 주소
SERVER_USER: 서버의 사용자 이름
SERVER_SSH_KEY: 로컬 머신에서 생성한 SSH 키 (~/.ssh/id_rsa 파일의 내용)
SERVER_PORT: SSH 포트 (기본값은 22)
<hr/>
3. 배포 스크립트 작성
위의 GitHub Actions 워크플로우에 포함된 배포 스크립트는 서버에 .jar 파일을 업로드하고, 이를 실행하는 작업을 포함합니다. 추가적으로, 필요에 따라 더 복잡한 스크립트를 작성하여 서버 설정 및 애플리케이션 관리를 자동화할 수 있습니다.

이제, 위의 설정을 통해 Spring Boot 웹 애플리케이션을 GitHub Actions를 사용하여 Ubuntu 서버에 배포하고 실행할 수 있습니다. 문제가 발생하면 로그를 확인하고 설정을 조정해야 합니다.
