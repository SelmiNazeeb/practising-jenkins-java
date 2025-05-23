pipeline {
    agent any
    environment {
        JAVA_IMAGE = 'selmi1999/java-spring-image:latest'
    }
    stages {
        stage ('mvn install'){
            steps{
                script {
                 mvn clean install
                 echo 'success mvn'
                }
            }
        }
        stage ('build') {
            steps {
                script {
            sh '''
            docker build -t ${JAVA_IMAGE} .
            echo ' success build'
            if (docker ps -a | grep "java-image-container")
            then
                docker stop java-image-container
                docker rm java-image-container
            fi
            echo 'delete success'
            docker run -d --name java-image-container -p 8081:8080 ${JAVA_IMAGE}
            '''
                }
            }
        }
        stage ('docker login and push') {
            steps{
                script {
                     withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME' , passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                    docker push ${JAVA_IMAGE}
                    echo 'success push'
                    '''
                     }
                }
            }
        }
        stage ('deploy') {
            steps{
                    sh '''
                    if (kubectl get deploy | grep 'java-image-deploy')
                    then
                        kubectl delete deploy java-image-deploy
                    fi
                    echo 'deploy success'
                    kubectl apply -f deploy.yaml
                    echo 'deploy success'
                    '''
            }
        }
    }
}
