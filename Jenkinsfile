pipeline {
    agent any
    environment {
        //Simplified pipeline to test deploying docker image to Kubernetes cluster using Jenkins
        DOCKER_IMAGE_NAME = "adilkhanekt/train_schedule_node_js"
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
    }
    stages {
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 0
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
}