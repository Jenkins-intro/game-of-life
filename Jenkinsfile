pipeline {
    agent any

    environment {
        ARTF = credentials('artifactory')
        
    }
    
    post {
        failure {
            mail to: 'team@example.com',
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
        }
    }

    stages {
        stage('Build') {
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
            post {
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        stage('Build Image') { 
            steps {
                dir('./gameoflife-web/') {
                    sh "docker build -t gameoflife ."
                }
            }
        }
        stage("Publish Image") {
            when {
                branch 'master'
            }
            steps {
                sh """
                        docker login -u ${ARTF_USR} -p ${ARTF_PSW} pwolfbees-docker-local.jfrog.io
                        docker tag gameoflife pwolfbees-docker-local.jfrog.io/pwolfbees:gameoflife
                        docker push pwolfbees-docker-local.jfrog.io/pwolfbees:gameoflife
                    """
            }
        }
        stage('Test Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.image('pwolfbees-docker-local.jfrog.io/pwolfbees:gameoflife').withRun("-d -p 8088:8080") {
                        input 'Approve'
                    }
                }
            }
        }
        stage('Deploy Image') {
            when {
                branch 'master'
            }
            steps {
                echo "Deploying"
            }
        }
    }
}
