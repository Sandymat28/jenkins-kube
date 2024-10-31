/*def remote = [:]
remote.name = 'sandy'
remote.host = '192.168.1.133'
remote.allowAnyHosts = true*/
    //remote.user = 'root'
    //remote.password = 'password'

pipeline{
  agent any

  environment{
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_ACCOUNT')
    DOCKER_IMAGE = 'matsandy/example-kube:latest'
    DOCKER_TAG = 'latest'
    //SANDY_CREDENTIALS = credentials('remote_credentials')
      
    HOSTNAME_DEPLOY = '192.168.1.133'
    DEPLOY_USER = 'sandy'
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
      
    stage ('Deploy in staging') {
        steps {
                sshagent(credentials: ['remote_credentials']) {
                    script {
                        // Commandes pour déployer l'image Docker sur la machine distante
                        sh """
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${HOSTNAME_DEPLOY} << EOF
                          docker compose down || true
                          docker pull ${DOCKERHUB_CREDENTIALS_USR}/${DOCKER_IMAGE}:${DOCKER_TAG}

                          # Redémarrez les conteneurs avec la nouvelle image
                          docker compose up -d
                        EOF
                        """
                    }
                }
            }
        }
    /*  when {
        expression { GIT_BRANCH == 'origin/main' }
         }
        steps {
            sshagent(credentials: ['remote_credentials']) { 
                sh '''
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa ${HOSTNAME_DEPLOY_STAGING} >> ~/.ssh/known_hosts
                    command1="cd deploy && echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    command2="echo 'IMAGE_VERSION=${DOCKERHUB_CREDENTIALS_USR}/${DOCKER_IMAGE}:${DOCKER_TAG}'"
                    command3="docker compose down && docker pull ${DOCKERHUB_CREDENTIALS_USR}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    command4="docker compose up -d"
                    ssh -t sandy@${HOSTNAME_DEPLOY_STAGING} \
                      -o SendEnv=IMAGE_NAME \
                      -o SendEnv=IMAGE_TAG \
                      -o SendEnv=DOCKERHUB_CREDENTIALS_USR \
                      -o SendEnv=DOCKERHUB_CREDENTIALS_PSW \
                      -C "$command1 && $command2 && $command3 && $command4"
                    '''
                }
            }
        }
      
    /*stage('Remote') {
      steps {
        echo 'Connexion to remote'
        sshagent(['remote_credentials']) {
        sh 'ssh -o StrictHostKeyChecking=no -t sandy@192.168.1.133 "sudo ls -lrt /root"'
}

        script {
          remote.user = env.SANDY_CREDENTIALS_USR
          remote.password = env.SANDY_CREDENTIALS_PSW
        }
        echo 'executing command in remote'
        sshCommand remote: remote, command: "ls -lrt"
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
      }
    }
}
