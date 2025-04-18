pipeline {
  agent any

  environment {
    DOCKER_CREDS = credentials('dockerhub')
    GITOPS_CREDS = credentials('git-creds')
  }

  stages {
    stage('Build, Push, and GitOps Update') {
      steps {
        sh '''
        echo "✅ 커밋 해시 추출"
        IMAGE_TAG=$(git rev-parse --short HEAD)
        echo "🎯 태그: $IMAGE_TAG"

        echo "✅ Docker 이미지 빌드"
        docker build -t duswjd/nginx:$IMAGE_TAG .

        echo "✅ DockerHub 로그인"
        echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin

        echo "✅ 이미지 푸시"
        docker push duswjd/nginx:$IMAGE_TAG

        echo "✅ GitOps 배포 레포 클론"
        rm -rf gitops-tmp
        git clone https://$GITOPS_CREDS_USR:$GITOPS_CREDS_PSW@github.com/yeonjeong2/gitops.git gitops-tmp

        echo "✅ 이미지 태그 수정"
        cd gitops-tmp/nginx-app
        sed -i "s|duswjd/nginx:.*|duswjd/nginx:$IMAGE_TAG|" deployment.yaml

        echo "✅ Git 커밋 및 푸시"
        git config user.email "jenkins@ci.com"
        git config user.name "jenkins-ci"
        git commit -am "♻️ 이미지 태그 업데이트: $IMAGE_TAG"
        git push origin main
        '''
      }
    }
  }
}

