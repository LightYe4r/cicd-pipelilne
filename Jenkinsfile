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
    MANIFEST_REPO      = 'https://github.com/LightYe4r/cicd-pipeline-deploy.git'
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
            env.GIT_TAG = sh(returnStdout: true, script: 'git rev-parse --short=7 HEAD').trim()
            echo "Using tag: ${env.GIT_TAG}"
            }
            sh "sed -i 's/{{VERSION}}/${env.GIT_TAG}/' index.html"
            sh "docker build -t ${IMAGE}:${env.GIT_TAG} ."
        }
    }


    stage('Push Image') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: env.DOCKER_CREDENTIALS,
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PW'
            )]) {
            sh """
                echo "${DOCKER_PW}" | docker login -u "${DOCKER_USER}" --password-stdin
                docker push ${IMAGE}:${env.GIT_TAG}
            """
            }
        }
    }


    stage('Update Manifests & Push') {
        steps {
            dir(env.MANIFEST_DIR) {
            // 매니페스트 리포 클론
            git url: env.MANIFEST_REPO,
                credentialsId: env.GIT_CREDENTIALS,
                branch: 'main'

            // image 태그 교체
            sh "sed -i 's|image: .*|image: ${IMAGE}:${GIT_TAG}|' deployment.yaml"

            // 커밋
            sh '''
                git config user.email "ci@example.com"
                git config user.name  "Jenkins CI"
                git add deployment.yaml
                git commit -m "ci: bump image to ${GIT_TAG}"
            '''

            // 여기서 PAT 을 사용해 remote URL 을 재설정한 뒤 push
            withCredentials([usernamePassword(
                credentialsId: env.GIT_CREDENTIALS,
                usernameVariable: 'GIT_USER',
                passwordVariable: 'GIT_TOKEN'
            )]) {
                sh '''
                # origin URL 을 PAT 가 포함된 형태로 바꿔줍니다.
                git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/LightYe4r/cicd-pipeline-deploy.git
                git push origin main
                '''
            }
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
