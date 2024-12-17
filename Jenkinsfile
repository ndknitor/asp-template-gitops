// Generic Webhook Trigger
// SSH Agent
def PRODUCTION_ALLOWED_USERS = ["kn"]


pipeline {
    agent any
    environment {
        OPS_REPOSITORY = "github.com/ndknitor/asp-template-gitops"
        PROJECT_REPOSITORY = "https://github.com/ndknitor/AspTemplate"

        GIT_CREDENTIALS_ID = "github_credential"
        REGISTRY_CREDENTIAL_ID = "registry_credential"
        ARGOCD_CREDENTIAL_ID = "argocd_token"

        REGISTRY = "utility.ndkn.local"
        IMAGE_NAME = "utility.ndkn.local/ndkn/asp-template"
        ARGOCD_SERVER= "192.168.121.104:30892"
        ARGOCD_APP_NAME = "asp-template"
        ARGOCD_NAMESPACE = "asp-template"
        ARGOCD_RESOURCE_NAME_DEVELOPMENT = "asp-template-deployment-development"
        ARGOCD_RESOURCE_NAME_STAGING = "asp-template-deployment-staging"
    }
    stages {
        stage('Clone project repository') {
            when {
                expression { params.CD == "None" || params.CD == "Development" || params.Auto }
            }
            steps {
                script {
                    dir('projects') {
                       git branch: 'main', credentialsId: env.GIT_CREDENTIALS_ID, url: env.PROJECT_REPOSITORY
                    }
                }
            }
        }
        stage('Build') {
            when {
                expression { params.CD == "None" || params.CD == "Development" || params.Auto }
            }
            steps {
                script {
                    dir('projects') {
                        sh 'export DOTNET_CLI_TELEMETRY_OPTOUT=0'
                        sh 'dotnet restore'
                        sh 'dotnet build --no-restore'
                    }
                }
            }
        }
        stage('Test') {
            when {
                expression { params.CD == "None" || params.CD == "Development" || params.Auto}
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests...'
                        // Add your unit test steps here
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests...'
                        // Add your integration test steps here
                    }
                }
            }
        }
        stage('Build image') {
            when {
                expression { params.CD == "Development" || params.Auto}
            }
            steps {
                script {
                    dir('projects') {
                        sh 'docker build -t ${IMAGE_NAME}:development .'
                    }
                }
            }
        }
        stage('Registry authentication') {
            when {
                expression { params.CD != "None" || params.Auto}
            }
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDENTIAL_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login ${REGISTRY} -u="${DOCKER_USERNAME}" -p="${DOCKER_PASSWORD}"'
                    }
                }
            }
        }
        stage('Push image to registry') {
            when {
                expression { params.CD == "Development" || params.Auto}
            }
            steps {
                script {
                    sh 'docker push ${IMAGE_NAME}:development'
                    
                }
            }
        }
        stage('Deploy development') {
            when {
                expression { params.CD == "Development" || params.Auto}
            }
            steps {
                script {
                    withCredentials([string(credentialsId: env.ARGOCD_CREDENTIAL_ID, variable: 'ARGOCD_TOKEN')]) {
                        sh '''
                        curl --insecure -X POST -H "Content-Type: application/json" -d '"restart"' -H "Authorization: Bearer ${ARGOCD_TOKEN}" "https://${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APP_NAME}/resource/actions?appNamespace=argocd&namespace=${ARGOCD_NAMESPACE}&resourceName=${ARGOCD_RESOURCE_NAME_DEVELOPMENT}&version=v1&kind=Deployment&group=apps" 
                        '''
                    }
                }
            }
        }
        stage('Deploy staging') {
            when {
                expression { params.CD == "Staging" || params.CD == "PassProduction" || params.Auto}
            }
            steps {
                withCredentials([string(credentialsId: env.ARGOCD_CREDENTIAL_ID, variable: 'ARGOCD_TOKEN')]) {
                    sh 'docker tag ${IMAGE_NAME}:development ${IMAGE_NAME}:staging'
                    sh 'docker push ${IMAGE_NAME}:staging'
                    sh '''
                        curl --insecure -X POST -H "Content-Type: application/json" -d '"restart"' -H "Authorization: Bearer ${ARGOCD_TOKEN}" "https://${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APP_NAME}/resource/actions?appNamespace=argocd&namespace=${ARGOCD_NAMESPACE}&resourceName=${ARGOCD_RESOURCE_NAME_STAGING}&version=v1&kind=Deployment&group=apps" 
                    '''
                }
            }
        }
        stage('Scan') {
            when {
                expression { params.CD == "Staging" || params.CD == "PassProduction"}
            }
            parallel {
                stage('Image scan') {
                    steps {
                        sh 'trivy image ${IMAGE_NAME}:staging'
                    }
                }
                // stage('Vulnerability scan') {
                //     steps {
                //         sshagent(['ssh-remote']) {
                //             sh """
                //                 ssh -o StrictHostKeyChecking=no ${username}@${devHost} \
                //                 '
                //                     docker run --rm -t softwaresecurityproject/zap-stable zap-api-scan.py -I -t http://${devHost}:10000/swagger/v1/swagger.json -f openapi \
                //                     -z "-config replacer.full_list(0).description=auth1 \
                //                     -config replacer.full_list(0).enabled=true \
                //                     -config replacer.full_list(0).matchtype=REQ_HEADER \
                //                     -config replacer.full_list(0).matchstr=Authorization \
                //                     -config replacer.full_list(0).regex=false \
                //                     -config replacer.full_list(0).replacement=Bearer\\ \${asp-template-user-jwt}"
                //                 '
                //             """
                //         }
                //     }
                // }
            }
        }
        stage ('Push change to Ops Repository') {
            when {
                allOf {
                    expression {
                        def triggeredBy = currentBuild.getBuildCauses()[0]?.userId
                        return triggeredBy in PRODUCTION_ALLOWED_USERS &&
                               (params.CD == "Production" || params.CD == "PassProduction" || params.Auto)
                    }
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github_credential', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    script {
                        NEW_VERSION = sh(
                            script: 'echo $(($(<k8s/VERSION) + 1))',
                            returnStdout: true
                        ).trim()
                        sh "echo ${NEW_VERSION}"
                        sh 'sed -e "s/{{VERSION}}/${NEW_VERSION}/g" k8s/template/production.yaml > k8s/value/production.yaml'
                        sh 'git add k8s/'
                        sh 'git commit -m "Triggered production Build: ${NEW_VERSION}"'
                        sh 'git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${OPS_REPOSITORY} HEAD:main'
                        sh 'docker tag ${IMAGE_NAME}:staging ${IMAGE_NAME}:${NEW_VERSION}'
                        sh 'docker push ${IMAGE_NAME}:${NEW_VERSION}'
                        sh 'docker image rm ${IMAGE_NAME}:${NEW_VERSION}'
                    }
                }
            }
        }
        // stage('Deploy production') {
        //     when {
        //         allOf {
        //             expression {
        //                 def triggeredBy = currentBuild.getBuildCauses()[0]?.userId
        //                 return triggeredBy in PRODUCTION_ALLOWED_USERS &&
        //                        (params.CD == "Production" || params.CD == "PassProduction" || params.Auto)
        //             }
        //         }
        //     }
        //     steps {
        //         withCredentials([string(credentialsId: 'argocd_token', variable: 'ARGOCD_TOKEN')]) {
        //             script {
        //                 sh '''
        //                     curl --insecure -X POST -H "Content-Type: application/json" -d '"restart"' -H "Authorization: Bearer ${ARGOCD_TOKEN}" "https://${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APP_NAME}/resource/actions?appNamespace=argocd&namespace=${ARGOCD_NAMESPACE}&resourceName=${ARGOCD_RESOURCE_NAME_STAGING}&version=v1&kind=Deployment&group=apps" 
        //                 '''
        //             }
        //         }
        //     }
        // }
    }
    post {
        always {
            sh 'docker image prune -f'
            sh 'docker logout ${REGISTRY}'
        }
    }
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
