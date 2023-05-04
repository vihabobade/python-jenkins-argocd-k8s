pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'github', 
                url: 'https://github.com/vihabobade/python-jenkins-argocd-k8s',
                branch: 'main'
           }
        }

        // stage('Build Docker'){
        //     // steps{
        //     //     script{
        //     //         sh '''
        //     //         echo 'Buid Docker Image'
        //     //         docker build -t vihabobade/python-jenkins-argocd-k8s:${BUILD_NUMBER} .
        //     //         '''
        //     //     }
        //     // }
        // }

        stage('Build & Push the artifacts'){
        //    steps{
        //         script{
        //             sh '''
        //             echo 'Push to Repo'
        //             docker push vihabobade/python-jenkins-argocd-k8s:${BUILD_NUMBER}
        //             '''
        //         }
        //     }

             environment {
                DOCKER_IMAGE = "vihabobade/python-jenkins-argocd-k8s:${BUILD_NUMBER}"
                // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
                REGISTRY_CREDENTIALS = credentials('docker-hub-cred')
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-hub-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'github', 
                url: 'https://github.com/vihabobade/python-jenkins-argocd-k8s.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                environment {
                    GIT_REPO_NAME = "jenkins-argocd-k8s"
                    GIT_USER_NAME = "vihabobade"
                }
                script{
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i '' "s/5/${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
