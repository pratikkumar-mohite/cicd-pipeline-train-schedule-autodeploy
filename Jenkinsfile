pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "mohite770/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'sudo ./gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'echo $CANARY_REPLICAS'
                    sh "sed -i 's/\$CANARY_REPLICAS/$CANARY_REPLICAS/g' train-schedule-kube-canary.yml"
                    sh "sed -i 's/\$DOCKER_IMAGE_NAME:\$BUILD_NUMBER/$DOCKER_IMAGE_NAME:$BUILD_NUMBER/g' train-schedule-kube-canary.yml"
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                    //sh 'kubectl config get-contexts'
                 }
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                //input 'Deploy to Production?'
                //milestone(1)
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                    
                 }
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f train-schedule-kube.yml'
                    
                 }
            
            }
        }
    }
}


