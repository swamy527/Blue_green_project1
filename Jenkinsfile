pipeline {
    agent any
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    }
    environment {
        IMAGE_NAME = "dockerswaha/bunkapp"
        TAG = "${params.DOCKER_TAG}"  // The image tag now comes from the parameter
        KUBE_NAMESPACE = 'webapps'
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv( 'sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=nodejsmysql -Dsonar.projectName=nodejsmysql"
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('Docker build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html ${IMAGE_NAME}:${TAG}"
            }
        }
        
         stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push ${IMAGE_NAME}:${TAG}"
                    }
                }
            }
        }
        stage('Deploy SVC-APP') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'roboshop', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D942CF49AA12CC4666768CC28C71338C.gr7.us-east-1.eks.amazonaws.com') {
                       sh """ if ! kubectl get svc app -n ${KUBE_NAMESPACE}; then
                          kubectl apply -f app-service.yml -n ${KUBE_NAMESPACE}
                        fi
                        """
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentFile = ""
                    if (params.DEPLOY_ENV == 'blue') {
                        deploymentFile = 'app-deployment-blue.yml'
                    } else {
                        deploymentFile = 'app-deployment-green.yml'
                    }
                    withKubeConfig(caCertificate: '', clusterName: 'roboshop', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D942CF49AA12CC4666768CC28C71338C.gr7.us-east-1.eks.amazonaws.com') {
						sh "kubectl apply -f pv-pvc.yml -n ${KUBE_NAMESPACE}"
						sh "kubectl apply -f mysql-ds.yml -n ${KUBE_NAMESPACE}"
                        sh "kubectl apply -f ${deploymentFile} -n ${KUBE_NAMESPACE}"
						
                    }
                }
            }
        }
    }
}