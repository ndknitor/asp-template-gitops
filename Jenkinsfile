// Generic Webhook Trigger
// SSH Agent
pipeline {
    agent any
    environment {
        REGISTRY = "utility.ndkn.local"
        IMAGE_NAME = "utility.ndkn.local/ndkn/asp-template"

        ARGOCD_SERVER= "192.168.121.104:30892"
        ARGOCD_APP_NAME = "asp-template"
        ARGOCD_NAMESPACE = "asp-template"
        ARGOCD_RESOURCE_NAME_DEVELOPMENT = "asp-template-deployment-development"
        ARGOCD_RESOURCE_NAME_STAGING = "asp-template-deployment-staging"
    }
    stages {
        stage('Dummy')
        {
            steps{
                sh 'echo Hello'
            }
        }
        // stage('Clone repository') {
        //     when {
        //         expression { params.CD == "Development" }
        //     }
        //     steps {
        //         git branch: 'main', credentialsId: 'Ndkn', url: 'https://github.com/ndknitor/AspTemplate'
        //     }
        // }

        // stage('Clone Repositories') {
        //     steps {
        //         script {
        //             // Clone the `project` repository into a specific directory
        //             dir('asp-template-project') { 
        //                 git branch: 'main', 
        //                     credentialsId: 'github_credential', 
        //                     url: 'https://github.com/ndknitor/AspTemplate'
        //             }

        //             // Clone the `ops` repository into a separate directory
        //             dir('asp-template-ops') { 
        //                 git branch: 'main', 
        //                     credentialsId: 'github_credential', 
        //                     url: 'https://github.com/ndknitor/asp-template-gitops'
        //             }
        //         }
        //     }
        // }
        // stage('List Files in Project Repo') {
        //     steps {
        //         script {
        //             dir('project') { // Change directory to 'project'
        //                 sh 'ls -l' // List files in the project directory
        //             }
        //         }
        //     }
        // }
        // stage('List Files in Ops Repo') {
        //     steps {
        //         script {
        //             dir('ops') { // Change directory to 'ops'
        //                 sh 'ls -l' // List files in the ops directory
        //             }
        //         }
        //     }
        // }

        // stage('Build') {
        //     when {
        //         expression { params.CD == "None" || params.CD == "Development" || params.Auto }
        //     }
        //     steps {
        //         sh 'export DOTNET_CLI_TELEMETRY_OPTOUT=0'
        //         sh 'dotnet restore'
        //         sh 'dotnet build --no-restore'
        //     }
        // }
        // stage('Test') {
        //     when {
        //         expression { params.CD == "None" || params.CD == "Development" || params.Auto}
        //     }
        //     parallel {
        //         stage('Unit Tests') {
        //             steps {
        //                 echo 'Running unit tests...'
        //                 // Add your unit test steps here
        //             }
        //         }
        //         stage('Integration Tests') {
        //             steps {
        //                 echo 'Running integration tests...'
        //                 // Add your integration test steps here
        //             }
        //         }
        //     }
        // }
        // stage('Build image') {
        //     when {
        //         expression { params.CD == "Development" || params.Auto}
        //     }
        //     steps {
        //             sh 'docker build -t ${IMAGE_NAME}:development .'
        //     }
        // }
        // stage('Registry authentication') {
        //     steps {
        //         script{
        //             withCredentials([usernamePassword(credentialsId: 'registry_credential', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
        //                 sh 'docker login utility.ndkn.local -u="${DOCKER_USERNAME}" -p="${DOCKER_PASSWORD}"'
        //             }
        //         }
        //     }
        // }
        // stage('Push image to registry') {
        //     when {
        //         expression { params.CD == "Development" || params.Auto}
        //     }
        //     steps {
        //         script {
        //             sh 'docker push ${IMAGE_NAME}:development'
                    
        //         }
        //     }
        // }
        // stage('Deploy development') {
        //     when {
        //         expression { params.CD == "Development" || params.Auto}
        //     }
        //     steps {
        //         script {
        //             withCredentials([string(credentialsId: 'argocd_token', variable: 'ARGOCD_TOKEN')]) {
        //                 sh '''
        //                 curl --insecure -X POST -H "Content-Type: application/json" -d '"restart"' -H "Authorization: Bearer ${ARGOCD_TOKEN}" "https://${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APP_NAME}/resource/actions?appNamespace=argocd&namespace=${ARGOCD_NAMESPACE}&resourceName=${ARGOCD_RESOURCE_NAME_DEVELOPMENT}&version=v1&kind=Deployment&group=apps" 
        //                 '''
        //             }
        //         }
        //     }
        // }
        // stage('Deploy staging') {
        //     when {
        //         expression { params.CD == "Staging" || params.CD == "PassProduction" || params.Auto}
        //     }
        //     steps {
        //             withCredentials([string(credentialsId: 'argocd_token', variable: 'ARGOCD_TOKEN')]) {
        //                 sh 'docker tag ${IMAGE_NAME}:development ${IMAGE_NAME}:staging'
        //                 sh 'docker push ${IMAGE_NAME}:staging'
        //                 sh '''
        //                     curl --insecure -X POST -H "Content-Type: application/json" -d '"restart"' -H "Authorization: Bearer ${ARGOCD_TOKEN}" "https://${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APP_NAME}/resource/actions?appNamespace=argocd&namespace=${ARGOCD_NAMESPACE}&resourceName=${ARGOCD_RESOURCE_NAME_STAGING}&version=v1&kind=Deployment&group=apps" 
        //                 '''
        //             }

        //     }
        // }
        // stage('Scan') {
        //     when {
        //         expression { params.CD == "Staging" || params.CD == "PassProduction"}
        //     }
        //     parallel {
        //         stage('Image scan') {
        //             steps {
        //                 sh 'trivy image ${IMAGE_NAME}:staging'
        //             }
        //         }
        //         // stage('Vulnerability scan') {
        //         //     steps {
        //         //         sshagent(['ssh-remote']) {
        //         //             sh """
        //         //                 ssh -o StrictHostKeyChecking=no ${username}@${devHost} \
        //         //                 '
        //         //                     docker run --rm -t softwaresecurityproject/zap-stable zap-api-scan.py -I -t http://${devHost}:10000/swagger/v1/swagger.json -f openapi \
        //         //                     -z "-config replacer.full_list(0).description=auth1 \
        //         //                     -config replacer.full_list(0).enabled=true \
        //         //                     -config replacer.full_list(0).matchtype=REQ_HEADER \
        //         //                     -config replacer.full_list(0).matchstr=Authorization \
        //         //                     -config replacer.full_list(0).regex=false \
        //         //                     -config replacer.full_list(0).replacement=Bearer\\ \${asp-template-user-jwt}"
        //         //                 '
        //         //             """
        //         //         }
        //         //     }
        //         // }
        //     }
        // }
        // stage('Clone Ops Repository') {
        //     steps {
        //         script {
        //             // Clone the source repository
        //             git credentialsId: "github_credential", url: "https://github.com/ndknitor/asp-template-gitops"
        //         }
        //     }
        // }
    }
    // post {
    //     always {
    //         sh 'docker image prune -f'
    //         sh 'docker logout ${REGISTRY}'
    //     }
    // }
}

        

    //     stage('Deploy production') {
    //         when {
    //             expression { params.CD == "Production" || params.CD == "PassProduction" || params.Auto }
    //         }
    //         steps {
    //             sshagent(['ssh-remote']) {
    //                 sh """
    //                     ssh -o StrictHostKeyChecking=no ${username}@${prodHost} \
    //                     "export KUBECONFIG=/home/${username}/.kube/config.yml 
    //                     kubectl rollout restart deployment/asp-template-deployment"
    //                 """
    //             }
    //         }
    //     }
    //     stage('Scan') {
    //         when {
    //             expression { params.CD == "Staging" || params.CD == "PassProduction"}
    //         }
    //         parallel {
    //             stage('Image scan') {
    //                 steps {
    //                     sshagent(['ssh-remote']) {
    //                         sh """
    //                             ssh -o StrictHostKeyChecking=no ${username}@${devHost} \
    //                             "trivy image ${devHost}:5000/asp-template"
    //                         """
    //                     }
    //                 }
    //             }
    //             stage('Vulnerability scan') {
    //                 steps {
    //                     sshagent(['ssh-remote']) {
    //                         sh """
    //                             ssh -o StrictHostKeyChecking=no ${username}@${devHost} \
    //                             '
    //                                 docker run --rm -t softwaresecurityproject/zap-stable zap-api-scan.py -I -t http://${devHost}:10000/swagger/v1/swagger.json -f openapi \
    //                                 -z "-config replacer.full_list(0).description=auth1 \
    //                                 -config replacer.full_list(0).enabled=true \
    //                                 -config replacer.full_list(0).matchtype=REQ_HEADER \
    //                                 -config replacer.full_list(0).matchstr=Authorization \
    //                                 -config replacer.full_list(0).regex=false \
    //                                 -config replacer.full_list(0).replacement=Bearer\\ \${asp-template-user-jwt}"
    //                             '
    //                         """
    //                     }
    //                 }
    //             }
    //         }
    //     }
    // }
