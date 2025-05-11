pipeline {
  agent any

  environment {
    // Docker Hub 크리덴셜 ID
    DOCKER_CREDENTIALS = 'by4mon@gmail.com'
    // GitHub PAT 크리덴셜 ID
    GIT_CREDENTIALS    = 'github-pat'
    // Docker 이미지 정보
    REGISTRY           = 'docker.io/manji955'
    IMAGE              = "${REGISTRY}/simple-web-app"
    // Git 리포지터리 URL (HTTPS)
    MANIFEST_REPO      = 'https://github.com/LightYe4r/deploy-manifests.git'
    MANIFEST_DIR       = 'deploy-manifests'
  }

  stages {
    stage('Checkout Code') {
      steps {
        // GitHub PAT을 이용해 HTTPS로 체크아웃
        git url: 'https://github.com/LightYe4r/cicd-pipelilne.git',
            credentialsId: env.GIT_CREDENTIALS,
            branch: 'main'
      }
    }

    stage('Build & Tag Image') {
      steps {
        script {
          // 커밋 해시 앞 7자리로 태그 생성
          GIT_TAG = sh(returnStdout: true, script: 'git rev-parse --short=7 HEAD').trim()
        }
        // index.html 버전 치환
        sh "sed -i 's/{{VERSION}}/${GIT_TAG}/' index.html"
        // 도커 빌드
        sh "docker build -t ${IMAGE}:${GIT_TAG} ."
      }
    }

    stage('Push Image') {
      steps {
        // Docker Hub 로그인 및 푸시
        withCredentials([usernamePassword(
            credentialsId: env.DOCKER_CREDENTIALS,
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PW'
        )]) {
          sh '''
            echo "$DOCKER_PW" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE}:${GIT_TAG}
          '''
        }
      }
    }

    stage('Update Manifests & Push') {
      steps {
        dir(env.MANIFEST_DIR) {
          // manifest repo 체크아웃
          git url: env.MANIFEST_REPO,
              credentialsId: env.GIT_CREDENTIALS,
              branch: 'main'
          // deployment.yaml의 이미지 태그 업데이트
          sh "sed -i 's|image: .*|image: ${IMAGE}:${GIT_TAG}|' deployment.yaml"
          // 커밋 후 푸시
          sh '''
            git config user.email "ci@example.com"
            git config user.name  "Jenkins CI"
            git add deployment.yaml
            git commit -m "ci: bump image to ${GIT_TAG}"
            git push origin main
          '''
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}
