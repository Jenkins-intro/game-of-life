pipeline {
  agent { label "docker" }
  
  environment {
    VERSION = readMavenPom().getVersion()
    IMAGE = readMavenPom().getArtifactId()
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
    stage('Deploy to Production') {
      when {
        branch 'master'
      }
      steps {
        build job: 'ecsdeploy', parameters: [string(name: 'image', value: "pwolfbees-docker.jfrog.io/pwolfbees/staging/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'production-demo'), string(name: 'service', value: "${IMAGE}-service")]
      }
    }
    stage('Deploy to Staging') {
      when {
        not {
          branch 'master'
        }
      }
      steps {
        build job: 'ecsdeploy', parameters: [string(name: 'image', value: "pwolfbees-docker.jfrog.io/pwolfbees/staging/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'staging-demo'), string(name: 'service', value: "${IMAGE}-service")]
      }
    }
  }
  post {
    failure {
      mail(to: 'team@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
    }
  }
}
