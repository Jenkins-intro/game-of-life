pipeline {
  agent { label "docker" }
  
  environment {
    VERSION = readMavenPom().getVersion()
    IMAGE = readMavenPom().getArtifactId()
    ENVIRONMENT = "staging-demo"
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
          sh "docker build -t ${IMAGE} . "
        }
        
      }
    }
    stage('Publish Image - Production') {
      when {
        branch 'master'
      }
      environment {
        ENVIRONMENT = "production-demo"
      }
      steps {
        sh """
           docker tag gameoflife pwolfbees-docker.jfrog.io/pwolfbees/release/${IMAGE}:${VERSION}
           docker push pwolfbees-docker.jfrog.io/pwolfbees/release/${IMAGE}:${VERSION}
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
           docker tag gameoflife pwolfbees-docker.jfrog.io/pwolfbees/staging/${IMAGE}:${VERSION}
           docker push pwolfbees-docker.jfrog.io/pwolfbees/staging/${IMAGE}:${VERSION}
           """
      }
    }
    stage('Deploy Image') {
      when {
        branch 'master'
      }
      steps {
        echo "${ENVIRONMENT}"
      }
    }
  }
  post {
    failure {
      mail(to: 'team@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
    }
  }
}
