pipeline{
  agent any

  environment{
    DOCKERHUB_CREDENTIALS=credentials('DOCKER_ACCOUNT')
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
       // sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'

      }
    }

    stage('Push'){
      steps{
        echo 'Pushing to Docker repository'
        sh 'docker push matsandy/example-kube:latest'
      }
    }

    stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Charger le fichier kubeconfig pour l'accès au cluster Kubernetes
                    withKubeConfig([credentialsId: KUBERNETES_CREDENTIALS]) {
                        // Déployer l'application dans le cluster Kubernetes
                        sh """
                        kubectl set image deployment/techstore-deployment techstore-container=${DOCKER_IMAGE}:${DOCKER_TAG} --record
                        kubectl rollout status deployment/techstore-deployment
                        """
                    }
                }
            }

    /*stage("SSH Into k8s Server") {
        def remote = [:]
        remote.name = 'K8S master'
        remote.host = '192.168.1.186'
        remote.user = 'msandy'
        remote.password = 'msandy'
        remote.allowAnyHosts = true

        stage('Put k8s-spring-boot-deployment.yml onto k8smaster') {
            sshPut remote: remote, from: 'k8s-spring-boot-deployment.yml', into: '.'
        }

        stage('Deploy spring boot') {
            sshCommand remote: remote, command: "kubectl apply -f k8s-spring-boot-deployment.yml"

        }*/
  }

    post{
      always{
        sh 'docker logout'
      }
    }
}

      
