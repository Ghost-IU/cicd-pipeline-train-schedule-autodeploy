pipeline {
    agent any
    environment {
        //be sure to replace "ghostly95" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "ghostly95/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
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
                CANARY_REPLICAS = 3
                KUBECONFIG_CREDENTIALS = credentials('kubeconfig')
                KUBE_NAMESPACE = 'default' // Change this line to use the default namespace
                APP_NAME = 'train-schedule'
            }
            steps {
                script {
                    // Deploy to Kubernetes using kubectl
                    // sh "envsubst < train-schedule-kube-canary.yml | kubectl apply --kubeconfig=${KUBECONFIG_CREDENTIALS} -f - --namespace=default"
                    sh "kubectl delete --kubeconfig=${KUBECONFIG_CREDENTIALS} deployment/train-schedule-deployment-canary --namespace=default"
                    sh "kubectl apply --kubeconfig=${KUBECONFIG_CREDENTIALS} -f train-schedule-kube-canary.yml --namespace=${KUBE_NAMESPACE}"
                    // Wait for the deployment to stabilize
                    sh "kubectl rollout status --kubeconfig=${KUBECONFIG_CREDENTIALS} deployment/train-schedule-deployment-canary --namespace=${KUBE_NAMESPACE}"
                }
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
                KUBECONFIG_CREDENTIALS = credentials('kubeconfig')
                KUBE_NAMESPACE = 'default'
            }
            steps {
        input 'Deploy to Production?'
        milestone(1)
        script {
            sh "kubectl apply --kubeconfig=${KUBECONFIG_CREDENTIALS} -f train-schedule-kube.yml --namespace=${KUBE_NAMESPACE}"
        }
    }
        }
    }
}

