pipeline {

    agent any

    stages {
        stage("building and quality checking"){
            parallel{
                stage('sonarqube quality check') {
                    steps {
                        sh 'echo well, TBD'
                        // withSonarQubeEnv("SonarCloud")
                        // {
                        // sh "./gradlew sonarqube -Dsonar.branch.name=\"master\""
                        // sleep(10)
                        // }
                    }
                }
                stage('building') {
                    // agent {
                    //     docker {
                    //     image 'docker'
                    //     }
                    // }
                    steps {
                        sh 'docker-compose build'
                    }
                post {
                    success {
                        slackSend(color: 'good', message: " Project '${JOB_NAME}' [${GIT_BRANCH}] has been updated and pulled from Github.")
                        }
                    failure {
                        slackSend(color: 'danger', message: "Project '${JOB_NAME}' [${GIT_BRANCH}] failed in building.")
                        }
                    }
                }    
            }
        }

        stage('sonar quality gate') {
            steps {
                sh 'echo well'
                // timeout(time: 1, unit: 'HOURS') {
                //     waitForQualityGate abortPipeline: true
                // }
            }
            post {
                success {
                    slackSend(color: 'good', message: "project '${JOB_NAME}' [${GIT_BRANCH}] has passed the SonarQube quality gate.")
                }
                failure {
                    slackSend(color: 'danger', message: "project '${JOB_NAME}' [${GIT_BRANCH}] failed the Sonar quality gate.")
                }
            }
        }

        stage('docker push to dockerhub') {
            steps {
                withDockerRegistry([credentialsId: 'DockerCred', url: '']) {
                    sh "docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 dougliu/azure-vote-front:${currentBuild.number}"
                    sh "docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 dougliu/azure-vote-front:latest"
                    sh "docker push dougliu/azure-vote-front:${currentBuild.number}"
                    sh "docker push dougliu/azure-vote-front:latest"
                }
            }
            post {
                success {
                    slackSend(color: 'good', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] new image is built and pushed to Dockerhub.")
                    }
                failure {
                    slackSend(color: 'danger', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] failed push the new image to dockerhub.")
                    }
                }
            }

        stage('deploy to remote K8S cluster') {
            steps {
                withKubeConfig([credentialsId: 'k8sCred', serverUrl: '${KUBEURL}']) {
                    sh 'kubectl apply -f azure-vote-all-in-one-redis.yaml'
                    sh 'kubectl rollout restart deployment.apps/azure-vote-front'
                 }
             }
            post {
                success {
                    slackSend(color: 'good', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] deployment to K8S is successful")
                }
                failure {
                    slackSend(color: 'danger', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] failed to deployer to k8s")
                }
            }
        }
    }
}