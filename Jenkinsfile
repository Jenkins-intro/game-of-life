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
    
    stage('Publish Image - Production') {
      when {
        branch 'master'
      }
      environment {
	REPO = "pwolfbees-docker.jfrog.io/pwolfbees/release"
      }
      steps {
        sh """
           docker tag ${IMAGE} ${REPO}/${IMAGE}:${VERSION}
           docker push ${REPO}/${IMAGE}:${VERSION}
           """
      }
    }
    
    stage('Publish Image - Staging') {
      when {
        not {
           branch 'master'
        }
      }
      environment {
	REPO = "pwolfbees-docker.jfrog.io/pwolfbees/release"
      }
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
        build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "pwolfbees-docker.jfrog.io/pwolfbees/staging/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'production-demo'), string(name: 'service', value: "${IMAGE}-service")]
      }
    }
    
    stage('Deploy to Staging') {
      when {
        not {
          branch 'master'
        }
      }
      steps {
        build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "pwolfbees-docker.jfrog.io/pwolfbees/staging/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'staging-demo'), string(name: 'service', value: "${IMAGE}-service")]
      }
    }
  }
  
  post {
    failure {
      mail(to: 'team@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
    }
  }
}
