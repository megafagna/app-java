pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '5', numToKeepStr: '7')
    }
    stages{
        stage('Build'){
            steps{
                 sh script: 'mvn clean package -DskipTestes'
                 archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            }
        }

    stage('Archive Test Results'){
        sh script: 'junit allowEmptyResults: true, testResults: '**/surefire-reports/*.xml''
    }

    stage('SonarQube analysis') {
//    def scannerHome = tool 'SonarScanner 4.0';
        steps{
        withSonarQubeEnv('sonar') { 
        // If you have configured more than one global server connection, you can specify its name
//      sh "${scannerHome}/bin/sonar-scanner"
        sh "mvn clean verify sonar:sonar \
  -Dsonar.projectKey=pipelineappjava \
  -Dsonar.host.url=http://192.168.0.43:9000 \
  -Dsonar.login=23dc7313c73eeececdf584342cd4a77dd52a8bbd"
    }
        }
        }

        stage('Upload War To Nexus'){
            steps{
                script{

                    def mavenPom = readMavenPom file: 'pom.xml'
                    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "java-app-snapshot" : "java-app-release"
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'my-app', 
                            classifier: '', 
                            file: "target/my-app-${mavenPom.version}.war", 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus3', 
                    groupId: 'com.dme.app', 
                    nexusUrl: '192.168.0.43:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: nexusRepoName,
                    version: "${mavenPom.version}"
                    }
            }
        }
    }
}


