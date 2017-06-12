pipeline {
  agent { label "docker" }
  
  environment {
    VERSION = readMavenPom().getVersion()
  }
  
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
        echo "${VERSION}"
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
    stage('Publish Image - Production') {
      when {
        branch 'master'
      }
      steps {
        sh """
           docker tag gameoflife pwolfbees-docker.jfrog.io/pwolfbees/gameoflife:1.0
           docker push pwolfbees-docker.jfrog.io/pwolfbees/gameoflife:1.0
           """
      }
    }
    stage('Publish Image - Staging') {
      when {
        not {
           branch 'master'
        }
      }
      steps {
        sh """
           docker tag gameoflife pwolfbees-docker.jfrog.io/pwolfbees/gameoflife:snapshot-1.0
           docker push pwolfbees-docker.jfrog.io/pwolfbees/gameoflife:snapshot-1.0
           """
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
  post {
    failure {
      mail(to: 'team@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
    }
  }
}
