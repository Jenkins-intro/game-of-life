pipeline {
  agent { label "docker" }
  stages {
    stage('Build') {
      agent {
        docker {
          reuseNode true
          registryUrl 'https://pwolfbees-docker.jfrog.io'
          registryCredentialsId 'artifactory'
          image 'maven:3.5.0-jdk-8'
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
           docker tag gameoflife pwolfbees-docker.jfrog.io/pwolfbees/gameoflife:snapshot1.0
           docker push pwolfbees-docker.jfrog.io/pwolfbees/gameoflife:snapshot1.0
           """
      }
    }
    stage('Test Image') {
      when {
        branch 'master'
      }
      steps {
        script {
          echo "Testing"
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
