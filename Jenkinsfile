pipeline {
    agent any
    tools {
        jdk 'jdk'
        maven 'maven'
    }
    environment {
        DOCKER_IMAGE = 'heyanoop/spring-boot-hello-world'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS = credentials('dockerhub-token')
    }


    stages {
        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/heyanoop/argocd-sample.git'
            }
        }
        stage('Maven Build') {
            steps {
                script {
                def result = sh(script: "mvn clean package", returnStatus: true)
                if (result != 0) {
                error "Maven build ccd."
            }
        }
    }
}
        
       
        stage('Docker Login & Build') {
            steps {
                script {
                    sh "echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin"
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }
        
        stage('Docker Tag & Push') {
            steps {
                script {
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        
        stage('Docker Logout') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker logout"
                }
            }
        }
          stage('Modify Helm Chart') {
            steps {
                script {
                    sh "sed -i 's/tag: .*/tag: '${DOCKER_TAG}'/' helm/argo-helm/values.yaml"
                }
            }
        }
        stage('Package Helm Chart') {
            steps {
                script {
                    def chartVersion = "${env.BUILD_NUMBER}"
                    sh "helm package helm/argo-helm --version ${chartVersion} -d ."
                }
            }
        }
        stage('Push Helm Chart to GitHub') {
            steps {
                script {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-ecr', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')])  {
                        def chartVersion = "${env.BUILD_NUMBER}"
                        sh """
                            aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
                            aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
                            aws configure set region ap-south-1
                            aws ecr get-login-password --region ap-south-1 | helm registry login 699475958586.dkr.ecr.ap-south-1.amazonaws.com --username AWS --password-stdin
                            mkdir helm-repo
                            cp argo-helm-${chartVersion}.tgz helm-repo/
                            cd helm-repo
                            helm push argo-helm-${chartVersion}.tgz oci://699475958586.dkr.ecr.ap-south-1.amazonaws.com
                        """
                    }
                }
            }
        }
    }
        post {
    	always {
            sh 'rm -rf helm-repo'
        }
        success {
            sh 'echo "Pipeline completed successfully!"'
        }
        failure {
            sh 'echo "Pipeline failed. Check the logs for more details."'
        }
    }

 }
    
