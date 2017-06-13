pipeline {
  agent { label "docker" }
  
  options {
		buildDiscarder(logRotator(numToKeepStr:'10')) // Keep the 10 most recent builds
	}
  
  environment {
    VERSION = readMavenPom().getVersion()
    IMAGE = readMavenPom().getArtifactId()
    REPO = "pwolfbees-docker.jfrog.io/pwolfbees/${BRANCH_NAME = 'master' ? 'release' : 'staging'}"
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
    
    stage('Publish Image') {
      steps {
        sh """
           docker tag ${IMAGE} ${REPO}/${IMAGE}:${VERSION}
           docker push ${REPO}/${IMAGE}:${VERSION}
           """
      }
    }
    stage('Deploy to Production') {
      when {
        branch 'master'
      }
      steps {
	      build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "${REPO}/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'production-demo'), string(name: 'service', value: "gameoflife-service")]
      }
    }
    
    stage('Deploy to Staging') {
      when {
        not {
          branch 'master'
        }
      }
      steps {
	      build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "${REPO}/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'staging-demo'), string(name: 'service', value: "sample-webapp")]
      }
    }
  }
  
  post {
    success {
      mail(to: 'team@example.com', subject: "Pipeline Complete: ${currentBuild.fullDisplayName}", body: "${IMAGE}:${VERSION} Was successfully deployed. ${env.BUILD_URL}")
    }
    failure {
      mail(to: 'team@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
    }
  }
}
