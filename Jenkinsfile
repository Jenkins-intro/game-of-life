pipeline {
  agent { label "docker" }
  stages {
    stage('Build') {
      agent {
        docker {
          reuseNode true
          registryUrl 'https://pwolfbees-docker-local.jfrog.io'
          registryCredentialsId 'artifactory'
          image 'pwolfbees:build-tools'
        }
        
      }
      steps {
        
          sh 'mvn clean install -DskipTests -DfailIfNoTests=false'
      }
      post {
        always {
          junit(allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml')
        }
        
      }
    }
    stage('Build Image') {
      steps {
        dir(path: './gameoflife-web/') {
          sh 'docker build -t gameoflife .'
        }
        
      }
    }
    stage('Publish Image') {
      when {
        branch 'master'
      }
      steps {
        sh """
           docker ps -a
           docker images
           docker push pwolfbees-docker-local.jfrog.io/gameoflife/pwolfbees:gameoflife
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
            input 'Is this running okay?'
          }
        }
        
      }
    }
    stage('Deploy Image') {
      when {
        branch 'master'
      }
      steps {
        echo 'Deploying Game of Life'
      }
    }
    stage('Clean Up') {
      steps {
        deleteDir()
      }
    }
  }
  environment {
    ARTF = credentials('artifactory')
  }
  post {
    failure {
      mail(to: 'team@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")
      
    }
    
  }
}
