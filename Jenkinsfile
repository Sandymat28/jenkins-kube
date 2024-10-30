pipeline{
  agent any

  environment{
    DOCKERHUB_CREDENTIALS=credentials('DOCKER_ACCOUNT')
    DOCKER_IMAGE = 'matsandy/example-kube:latest'
    DOCKER_TAG = 'latest'
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
        echo 'Deploying to Kubernetes'
        sh 'kubectl apply -f k8s-spring-boot-deployment.yml'
        
        /*script {
                    // Charger le fichier kubeconfig pour l'accès au cluster Kubernetes
            withKubeConfig([credentialsId: k8s-master-ssh	]) {
                        // Déployer l'application dans le cluster Kubernetes
              sh """
              kubectl set image deployment/k8s-spring-boot-deployment example-kube-container=${DOCKER_IMAGE}:${DOCKER_TAG} --record
              kubectl rollout status deployment/k8s-spring-boot-deployment
              """
                    }
                }*/
            }
    }

    stage('Verify the deploying'){
      steps{
        sh 'kubectl get po'
        sh 'kubectl get deployement'
        sh 'kubectl get all'
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

    /*withDockerRegistry(credentialsId: 'DOCKER_ACCOUNT') {
    // some block
}*/
    }

    post{
      always{
        sh 'docker logout'
      }
    }
}

      
