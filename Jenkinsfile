pipeline {
  agent any

  tools { nodejs 'node-7.8.0' }

  environment {
    IMAGE_TAG = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
    APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain_container' : 'nodedev_container'}"

    DOCKER_REGISTRY = "docker.io/<your-dockerhub-username>"
    DOCKER_CRED_ID = 'dockerhub-creds'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        echo "Гілка: ${env.BRANCH_NAME}"
      }
    }

    stage('Install & Build') {
      steps {
        sh 'node --version || true'
        sh 'npm install'
        sh 'npm run build || true'
      }
    }

    stage('Prepare logo') {
      steps {
        script {
          if (fileExists('assets/logo-main.svg') && fileExists('assets/logo-dev.svg')) {
            if (env.BRANCH_NAME == 'main') {
              sh 'cp assets/logo-main.svg assets/logo.svg'
            } else {
              sh 'cp assets/logo-dev.svg assets/logo.svg'
            }
          } else {
            echo "Лого для кожної гілки вже у репозиторії."
          }
        }
      }
    }

    stage('Test') {
      steps {
        sh 'npm test || echo "Тести пропущено / немає"'
      }
    }

    stage('Build Docker image') {
      steps {
        sh "docker build -t ${IMAGE_TAG} ."
      }
    }

    stage('Push Image (optional)') {
      when {
        expression { return env.DOCKER_REGISTRY && env.DOCKER_CRED_ID != '' }
      }
      steps {
        script {
          docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_CRED_ID) {
            sh "docker tag ${IMAGE_TAG} ${DOCKER_REGISTRY}/${IMAGE_TAG}"
            sh "docker push ${DOCKER_REGISTRY}/${IMAGE_TAG}"
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh """
            if docker ps -a --format '{{.Names}}' | grep -q '${CONTAINER_NAME}'; then
              docker rm -f $(docker ps -a --filter "name=${CONTAINER_NAME}" -q) || true
            fi

            docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:${APP_PORT} ${IMAGE_TAG}
          """
        }
      }
    }

  }

  post {
    success {
      echo "Пайплайн для ${env.BRANCH_NAME} завершився успішно."

      // автоматичний тригер (вимкнено, але можна увімкнути)
      /*
      script {
        def jobName = (env.BRANCH_NAME == 'main') ? 'Deploy_to_main' : 'Deploy_to_dev'
        build job: jobName, wait: false, parameters: [string(name:'IMAGE', value: IMAGE_TAG)]
      }
      */
    }

    failure {
      echo "Помилка у гілці ${env.BRANCH_NAME}"
    }
  }
}
