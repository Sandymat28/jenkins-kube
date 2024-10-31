def remote = [:]
remote.name = 'sandy'
remote.host = '192.168.1.133'
remote.allowAnyHosts = true
    //remote.user = 'root'
    //remote.password = 'password'

pipeline{
  agent any

  environment{
    DOCKERHUB_CREDENTIALS=credentials('DOCKER_ACCOUNT')
    DOCKER_IMAGE = 'matsandy/example-kube:latest'
    DOCKER_TAG = 'latest'
    SANDY_CREDENTIALS = credentials('remote_credentials')
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

    stage('Deploy') {
      steps {
        script {
          withKubeConfig([credentialsId: "mykubeconfig"]) {
            echo "Deploying applications to cluster"
            sh 'kubectl apply -f k8s-spring-boot-deployment.yml'
            }
          }
        }
      }

    stage('Remote') {
      steps {
        echo 'Connexion to remote'
        sshagent(['remote_credentials']) {
        sh 'ssh -o StrictHostKeyChecking=no -t sandy@192.168.1.133 "sudo ls -lrt /root"'
}

        /*script {
          remote.user = env.SANDY_CREDENTIALS_USR
          remote.password = env.SANDY_CREDENTIALS_PSW
        }
        echo 'executing command in remote'
        sshCommand remote: remote, command: "ls -lrt"*/
      }
    }
      

    /*stage('Build') {
      steps {
        echo 'Building in remote device'
          sshagent(['remote_credentials']) {
            sh "ssh -o StrictHostKeyChecking=no -l sandy 192.168.1.133 'cd ~/public_html && ./bash_script.sh'"
            }
         }
      }*/
   }
    
post{
      always{
        sh 'docker logout'
        sleep 5
      }
    }
}
