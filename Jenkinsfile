pipeline {
  agent any

  environment {
    COMMIT_HASH = ''
  }

  stages {
    stage('Build & Push Docker Image') {
      environment {
        DOCKER_CREDS = credentials('dockerhub')
      }
      steps {
        script {
          // 커밋 해시를 변수에 저장
          env.COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        }

        sh '''
        echo "✅ Docker Build 시작"
        echo "🎯 태그: $COMMIT_HASH"
        docker build -t duswjd/nginx:$COMMIT_HASH .
        echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
        docker push duswjd/nginx:$COMMIT_HASH
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
        sed -i "s|duswjd/nginx:.*|duswjd/nginx:$COMMIT_HASH|" deployment.yaml

        echo "✅ Git Commit & Push"
        git config user.email "jenkins@ci.com"
        git config user.name "jenkins-ci"
        git commit -am "♻️ 이미지 태그 업데이트: $COMMIT_HASH"
        git push origin main
        '''
      }
    }
  }
}

