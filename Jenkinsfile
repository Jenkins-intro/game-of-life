pipeline {
    agent any  // Run everything on an agent with the docker daemon
    options {
	  buildDiscarder(logRotator(numToKeepStr:'10')) // Keep the 10 most recent builds
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()  // Using Pipeline Utility Steps plugin read the pom.xml file to get Artifact ID
    }
    stages {    
        stage('Build') {
            agent {
                docker {
                    reuseNode true  // reuse the workspace on the agent defined at top-level
                    image 'maven:3.5.0-jdk-8'  // download use this maven image
                }
            }
            steps {
                sh 'mvn clean install -Dmaven.test.skip=true'  //runs mvn inside the docker container
            }
            post {
                always {
                    junit(allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml')  // always publish test results
                }
            }
        }
        stage('Build Image') {  
            steps {
                dir(path: './gameoflife-web/') {  // shift to the target directory
                    sh "docker build -t ${IMAGE} . "  // build a new image with the game-of-life app based on the local Dockerfile
                }
            }
        }
        stage('Publish Release Image') {
            when {
                branch 'master'  //only run these steps on the master branch
            }
            environment {
                VERSION = readMavenPom().getVersion().replace('-SNAPSHOT', '.' + currentBuild.number)  // read pom.xml to get Version 
                REPO = "myregistry.url/release"  // set the name for the registry
            }
            steps {
                sh """
                docker tag ${IMAGE} ${REPO}/${IMAGE}:${VERSION}
                //docker push ${REPO}/${IMAGE}:${VERSION}  //tag and push the newly created image to my registry
                """
                milestone(10)  // set a milestone to handle concurrent runs of this pipeline.
                lock(inversePrecedence: true, quantity: 1, resource: 'ecsDeploy') {  // only allow one concurrent run of these steps at once.
                    // run a different pipleine called 'ECS Deployment/ecsdeploy'
                    build job: 'ECS Deployment/ecsdeploy', parameters: [string(name: 'image', value: "${REPO}/${IMAGE}:${VERSION}"), 
                                                                        string(name: 'environment', value: 'gameoflife-prod'), 
                                                                        string(name: 'service', value: "gameoflife-service")]
                    milestone(20)  // set a second milestone. This milestone will abort any older builds still waiting to run.
                }
            }
        }
    }
    post {
        success {  // notify users when the Pipeline is successful
            mail(to: 'me@example.com', subject: "Pipeline Complete: ${currentBuild.fullDisplayName}", body: "Was successfully deployed. ${env.BUILD_URL}")
        }
        failure {  // notify users when the Pipeline fails
            mail(to: 'me@example.com', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}")  
        }
    }
}
