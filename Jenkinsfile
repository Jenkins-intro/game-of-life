pipeline {
    agent any

    environment {
        ARTF = credentials('artifactory')
    }

    stages {
        stage('Build WAR') {
            agent { 
                docker {
                    reuseNode true
                    image "pwolfbees-docker-local.jfrog.io/pwolfbees:build-tools"
                }
            }
            steps {
                withMaven(mavenSettingsConfig: '41cf650a-117d-4000-9709-47a84331026b') {
                    sh "mvn deploy -Dtest=WhenYouStoreGamesInADatabase -DfailIfNoTests=false"
                }
            }
        }
        stage('Build Image') { 
            steps {
                dir('./gameoflife-web/') {
                    sh "docker build -t gameoflife ."
                }
            }
            post {
                success {
                    sh """
                        docker login -u ${ARTF_USR} -p ${ARTF_PSW} pwolfbees-docker-local.jfrog.io
                        docker tag gameoflife pwolfbees-docker-local.jfrog.io/pwolfbees:gameoflife
                        docker push pwolfbees-docker-local.jfrog.io/pwolfbees:gameoflife
                    """
                    
                }
            }
        }
        stage('Test Image') {
            steps {
                script {
                    docker.image('pwolfbees-docker-local.jfrog.io/pwolfbees:gameoflife').withRun("-d -p 8088:8080") {
                        input 'Approve'
                    }
                }
            }
        }
   }
}
