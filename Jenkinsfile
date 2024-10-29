pipeline{
  agent any

  environment{
    DOCKER_CREDENTIALS=credentials('DOCKER_ACCOUNT')
  }

  stages{
    stage('Compilation'){
      steps{
        echo 'Build application'
        sh './gradlew clean build'
      }
    }

    stage('Docker image'){
      steps{
        echo 'Building docker images'
        sh 'docker build -t matsandy/example-kube:latest .'
        sh 'docker-compose build'
      }
    }

    stage('Log'){
      steps{
        echo 'Loging Dockerhub'
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }

    stage('Push'){
      steps{
        echo 'Pushing to Docker repository'
        sh 'docker push matsandy/example-kube:latest'
      }
    }
  }

    post{
      always{
        sh 'docker logout'
      }
    }
}

      
