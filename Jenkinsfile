pipeline {
    agent any
    environment {
        //Simplified pipeline to test deploying docker image to Kubernetes cluster using Jenkins
        DOCKER_IMAGE_NAME = "adilkhanekt/train_schedule_node_js"
        CANARY_REPLICAS = 0
    }
    stages {
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kube_creds',
                    configs: 'canary_train_schedule_instance.yaml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('SmokeTest') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                script {
                    sleep (time: 5)
                    def response = httpRequest {
                        url: "http://${KUBE_MASTER_IP}:30100"
                        timeout: 30
                    }
                    if (response.status != 200) {
                        error ("Smoke test failed.")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps 
            {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kube_creds',
                    configs: 'canary_train_schedule_instance.yaml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kube_creds',
                    configs: 'stable_train_schedule_instance.yaml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
    post {
        cleanup {
            kubernetesDeploy(
                kubeconfigId: 'kube_creds',
                configs: 'canary_train_schedule_instance.yaml',
                enableConfigSubstitution: true
            )    
        }
    }
}