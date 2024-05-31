# project name : devops_step0

## git 작업 순서 
### git init
### git add README.md
### git commit -m "first commit"
### git branch -M main
### git remote add origin https://github.com/masungil70/devops_step0.git
### git push -u origin main

# Spring Boot 웹 애플리케이션을 GitHub Actions를 이용하여 Ubuntu 서버에 배포하고 실행하는 방법

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
