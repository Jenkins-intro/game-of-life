@Library("github.com/pwolfbees/pipeline") _

pipeline{
   agent docker:'pwolf/cjptower'
   
   notifications {
        success {
            echo "SUCCESS: ${currentBuild.fullDisplayName}"
        }
        failure {
            echo "FAILURE: ${currentBuild.fullDisplayName}"
        }
        unstable {
            mail to:"devops@example.com", subject:"UNSTABLE: ${currentBuild.fullDisplayName}", body: "Build unstable."
        }
        changed {
            echo "CHANGED: ${currentBuild.fullDisplayName}"
        }
    }

   stages {
      stage('Build and Package'){
         sh "mvn clean package -Dtest=WhenYouStoreGamesInADatabase"
      }
      stage ('Publish Artifact to S3'){
        wrap([$class: 'AmazonAwsCliBuildWrapper', credentialsId: 's3-cjptower', defaultRegion: 'us-west-2']) {
            sh 'aws s3 cp gameoflife-web/target/gameoflife.war s3://cjptower/gameoflife.war'
        }
      }
      stage ('Promote Build'){
        script{
            env.TARGET = input message: 'Deploy Application?', ok: 'Go! Go! Go!', parameters: [choice(choices: 'Development\nStaging\nProduction', description: 'Pick Target Environment for Deployment.', name: 'TARGET')]
        }
      }
      stage("Run Tower Playbook"){
         withTower(host:"https://104.198.10.204", credentials:"tower-cli"){
            sh "tower-cli job launch --job-template=41 --monitor --extra-vars='target='${env.TARGET}"
         }
      }   
   }
}
