node {
        stage ('Initialize') {
               def dockerHome = tool 'docker-latest'
               def mavenHome  = tool 'maven-latest'
               env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
               env.DOCKER_HOST = "unix:///var/run/docker.sock"
        }

        stage('Checkout') {
               checkout scm
        }

        stage ('Build and Run Tests') {
	       sh  'mvn install allure:report sonar:sonar  -Dmaven.test.failure.ignore=true' 
           archiveArtifacts "target/**/*"
           junit 'target/surefire-reports/*.xml'
           allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
        }
        

        stage ('Docker Build') {
               sh 'mvn docker:build -Dmaven.test.skip=true'
        }
        
              
        stage ('Stop Old Containers') {
               sh 'docker-compose down'
        }
        stage ('Start New Containers') {
               sh 'docker-compose up -d'
        }
        
        stage('Notify Results'){
            echo currentBuild.result
            emailext body: '$DEFAULT_CONTENT', recipientProviders: [brokenTestsSuspects(), brokenBuildSuspects(), developers()], subject: '$DEFAULT_SUBJECT'
            
        }
   
}
