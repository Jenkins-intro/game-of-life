pipeline {
  agent { label "docker" }
  
  options {
	  buildDiscarder(logRotator(numToKeepStr:'10')) // Keep the 10 most recent builds
  }
  
  environment {
	  IMAGE = readMavenPom().getArtifactId()
	  REGISTRY = "pwolfbees-docker.jfrog.io"
	  DOCKCREDS = 'artifactory'
  }
    
  stages {
    stage('Build') {
      agent {
        docker {
			reuseNode true
			registryUrl "${REGISTRY}"
			registryCredentialsId "${DOCKCREDS}"
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
    
    stage('Publish Release Image') {
	    when {
		    branch 'master'
	    }
	    environment {
    		VERSION = readMavenPom().getVersion().replace('-SNAPSHOT', '.' + currentBuild.number)
			REPO = "${REGISTRY}/release"
  	    }
	    steps {
		    sh """
		    docker tag ${IMAGE} ${REPO}/${IMAGE}:${VERSION} 
		    docker push ${REPO}/${IMAGE}:${VERSION}
		    """
		    milestone
		    lock(inversePrecedence: true, quantity: 1, resource: 'ecsDeploy') {
		    	build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "${REPO}/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'gameoflife-prod'), string(name: 'service', value: "gameoflife-service")]
		    	milestone
		    }
	    }
    }
    
    stage('Publish Snapshot Image') {
	    when {
		   branch 'staging'
	    }
	    environment {
    		VERSION = readMavenPom().getVersion()
			REPO = "${REGISTRY}/staging"
  	    }
	    steps {
		    sh """
		    docker tag ${IMAGE} ${REPO}/${IMAGE}:${VERSION}
		    docker push ${REPO}/${IMAGE}:${VERSION}
		    """
		   // build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "${REPO}/${IMAGE}:${VERSION}"), string(name: 'environment', value: 'gameoflife-staging'), string(name: 'service', value: "gameoflife-service")]
	    }
    }
  }
  
  post {
    success {
      mail(to: 'kvissa@cloudbees.com', subject: "Pipeline Complete: ${currentBuild.fullDisplayName}", body: "Was successfully deployed. ${env.BUILD_URL}")
    }
    failure {
      mail(to: 'kvissa@cloudbees.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
    }
  }
}
