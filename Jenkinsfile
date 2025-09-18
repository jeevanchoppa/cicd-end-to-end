pipeline {
    
    // agent any
    agent {
        docker { image 'node:22.19.0-alpine3.22' }
    } 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_HUB_CREDENTIALS = credentials('a662f70b-c5e8-4a3a-9abe-e36285be61c1')
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: '7d7fdf97-5d40-43b4-8f89-4b3c8ae5cfcf', 
                url: 'https://github.com/jeevanchoppa/cicd-end-to-end',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t jeevanchoppa/demos:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        // stage('Push the artifacts'){
        //    steps{
        //         script{
        //             sh '''
        //             echo 'Push to Repo'
        //             echo \"${DOCKER_HUB_CREDENTIALS_PSW}\" | docker login -u \"${DOCKER_HUB_CREDENTIALS_USR}\" --password-stdin
        //             docker push jeevanchoppa/cicd-e2e:${BUILD_NUMBER}
        //             docker logout
        //             echo "Logged out"
        //             '''
        //         }
        //     }
        // }

        stage('Push the artifacts') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub_credentials') {
                        // def image = docker.image("jeevanchoppa/cicd-e2e:${env.BUILD_NUMBER}")
                        // echo "Pushing image ${image}"
                        // image.push()
                        // echo "Pushing image with 'latest' tag"
                        // image.tag("jeevanchoppa/cicd-e2e:latest")
                        docker.image("jeevanchoppa/demos:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: '7d7fdf97-5d40-43b4-8f89-4b3c8ae5cfcf', 
                url: 'https://github.com/jeevanchoppa/cicd-demo-manifests-repo.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i '' "s/32/${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/iam-veeramalla/cicd-demo-manifests-repo.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}