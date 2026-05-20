pipeline {

    agent {
        label 'agent-1'
    }

    parameters {
        string(name: 'PR_NUMBER', defaultValue: '', description: 'Pull Request Number')
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker Image Tag')
        string(name: 'NAMESPACE', defaultValue: '', description: 'Kubernetes Namespace')
    }

    environment {
        REGION    = "us-east-1"
        PROJECT   = "roboshop"
        COMPONENT = "catalogue"
        ACC_ID    = "430774481266"
    }

    stages {

        stage('Print Variables') {
            steps {

                echo "PR_NUMBER=${params.PR_NUMBER}"
                echo "IMAGE_TAG=${params.IMAGE_TAG}"
                echo "NAMESPACE=${params.NAMESPACE}"
            }
        }

        stage('Create Namespace') {
            steps {

                sh """
                kubectl create namespace ${params.NAMESPACE} \
                --dry-run=client -o yaml | kubectl apply -f -
                """
            }
        }

        stage('Deploy Preview Environment') {

            steps {

                withAWS(credentials: 'aws-cred', region: 'us-east-1') {

                    sh """
                    helm upgrade --install ${COMPONENT} . \
                      --namespace ${params.NAMESPACE} \
                      --set deployment.imageName=${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${params.IMAGE_TAG} \
                      --set ingress.host=${COMPONENT}-${params.PR_NUMBER}.roboshop.dev
                    """
                }
            }
        }
    }
}