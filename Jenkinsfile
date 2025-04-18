pipeline {
  agent any

  environment {
    IMAGE_TAG = ''
  }

  stages {
    stage('Build & Push Docker Image') {
      environment {
        DOCKER_CREDS = credentials('dockerhub') // ⬅️ 자격증명 한 번만 호출
      }
      steps {
        script {
          def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          env.IMAGE_TAG = commitHash
        }

        sh '''
        echo "✅ Docker Build 시작"
        docker build -t duswjd/nginx:$IMAGE_TAG .
        echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
        docker push duswjd/nginx:$IMAGE_TAG
        '''
      }
    }

    stage('Update GitOps Repo') {
      environment {
        GIT_CREDS = credentials('git-creds')
      }
      steps {
        sh '''
        echo "✅ GitOps 배포 레포 클론"
        rm -rf gitops-tmp
        git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/yeonjeong2/gitops-deploy.git gitops-tmp

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

