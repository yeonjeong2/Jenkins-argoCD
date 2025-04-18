pipeline {
  agent any

  environment {
    DOCKER_USERNAME = credentials('dockerhub')
    DOCKER_PASSWORD = credentials('dockerhub')
    GITOPS_USERNAME = credentials('git-creds')
    GITOPS_PASSWORD = credentials('git-creds')
  }

  stages {
    stage('Build & Push Docker Image') {
      steps {
        script {
          def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          env.IMAGE_TAG = commitHash
        }

        sh '''
        echo "✅ Docker Build 시작"
        docker build -t duswjd/nginx:$IMAGE_TAG .
        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
        docker push duswjd/nginx:$IMAGE_TAG
        '''
      }
    }

    stage('Update GitOps Repo') {
      steps {
        sh '''
        echo "✅ GitOps 배포 레포 클론"
        rm -rf gitops-tmp
        git clone https://$GITOPS_USERNAME:$GITOPS_PASSWORD@github.com/yeonjeong2/gitops-deploy.git gitops-tmp

        echo "✅ 이미지 태그 수정"
        cd gitops-tmp/my-nginx-app
        sed -i "s|duswjd/nginx:.*|duswjd/nginx:$IMAGE_TAG|" deployment.yaml

        echo "✅ Git Commit & Push"
        git config user.email "jenkins@ci.com"
        git config user.name "jenkins-ci"
        git commit -am "♻️ 이미지 태그 업데이트: $IMAGE_TAG"
        git push origin main
        '''
      }
    }
  }
}

