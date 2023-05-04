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
            environment {
                GIT_REPO_NAME = "python-jenkins-argocd-k8s"
                GIT_USER_NAME = "vihabobade"
            }
            steps {
                git credentialsId: 'github-uap', 
                // git passwordVariable: 'GIT_PASSWORD', 
                // git usernameVariable: 'GIT_USERNAME',
                url: "https://github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git",
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            environment {
                GIT_REPO_NAME = "python-jenkins-argocd-k8s"
                GIT_USER_NAME = "vihabobade"
            }
            steps {
                script{
                    // withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    withCredentials([usernamePassword(credentialsId: 'github-uap', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        git config user.email "viha.bobade@gmail.com"
                        git config user.name "Viha Bobade"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        cat deploy/deploy.yaml
                        cd deploy
                        sed -i "s/8/${BUILD_NUMBER}/g" deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/vihabobade/python-jenkins-argocd-k8s.git HEAD:main
                        #git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        
                        '''                        
                    }
                }
            }
        }
    }
}
