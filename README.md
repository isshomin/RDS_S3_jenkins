# jenkins와 aws RDS, s3를 이용한 웹 서비스 CI/CD

---
<br>

## 목적 ✒

Jenkins, GitHub, 및 AWS S3를 활용하여 CI/CD 프로세스를 자동화하고, 빌드된 .jar 파일을 S3에 자동으로 업로드하여 배포 및 관리 효율성을 극대화합니다. 또한, AWS RDS와 연동하여 데이터베이스 관리 성능을 향상시킵니다.

---

<br>

## 실습환경 🏞

```ruby
virtualbox: version 7.0
ubuntu: version 22.04.4 LTS
docker: version 27.3.1
minikube: version v1.34.0
Jenkins: version 2.462.3
Amazon RDS, Amazon S3
```

---

<br>

## 이미지 생성 🖼

#### 1️⃣) docker compose file을 이용해 jenkins를 설치해줍니다.

```yaml
#compose.yaml
services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    user: root
    container_name: myjenkins
    ports:
      - 8080:8080
      - 50001:50000
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=Asia/Seoul
```

#### 2️⃣) jenkins 포트포워딩을 해줍니다. 

![image](https://github.com/user-attachments/assets/a1a00939-be36-4e0a-94de-60a9988e1fe5)


<br>

#### 1️⃣) ngrok을 활용하여 github repository에 hook 해줍니다. 


[참고](https://github.com/isshomin/jenkins_auto_deploy)

<br>

#### 2️⃣) jenkins에 새 item을 생성하고 파이프라인 스크립트를 작성해줍니다.
 

```ruby
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/isshomin/RDS_jenkins_test.git'
            }
        }
          
        stage('Build') {
            steps {
                sh 'chmod +x gradlew'                    
                sh './gradlew clean build -x test'
            }
        }
        
        stage('Copy jar') { 
            steps {
                script {
                    def jarFile = 'build/libs/step18_empApp-0.0.1-SNAPSHOT.jar'
                    def destDir = '/var/jenkins_home/'                   
                    sh "cp ${jarFile} ${destDir}"
                }
            }
        }
		
		stage('Upload to S3') {
            steps {
                script {
                    def jarFile = 'build/libs/step18_empApp-0.0.1-SNAPSHOT.jar'
                    def bucketName = 'ce04-bucket-01'
                    def s3Path = 'jar' // S3 내 경로

                    // AWS CLI로 JAR 파일을 S3로 업로드
                    sh "aws s3 cp ${jarFile} s3://${bucketName}/${s3Path}/"
                }
            }
        }
    }
}
```

---

<br>

## 요약 📩


<br>

---

## 트러블슈팅 🎯🔫


---
