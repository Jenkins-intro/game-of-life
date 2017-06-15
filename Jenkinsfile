pipeline {
    agent { label "docker" }
    
    options {
      buildDiscarder(logRotator(numToKeepStr:'10')) // Keep the 10 most recent builds
    }
    environment {
	  IMAGE = readMavenPom().getArtifactId()
	  REGISTRY = "${readProperties(file: 'Jenkinsfile.properties')['registry']}"
	  DOCKCREDS = 'artifactory'
    }
    
    stages {
        stage('Build') {
            steps {
                echo "${REGISTRY}"
            }
        }
    }
}
