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
                 sh script: 'mvn clean package'
                 archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            }
        }

        stage('SonarQube analysis') {
//    def scannerHome = tool 'SonarScanner 4.0';
        steps{
        withSonarQubeEnv('sonar') { 
        // If you have configured more than one global server connection, you can specify its name
//      sh "${scannerHome}/bin/sonar-scanner"
        sh "mvn clean verify sonar:sonar \
  -Dsonar.projectKey=maven \
  -Dsonar.host.url=http://192.168.0.178:9000 \
  -Dsonar.login=sqp_f8ae1c7d1cbc871c7f0caf20a9a617054bf2b356"
    }
        }
        }

        stage('Upload War To Nexus'){
            steps{
                script{

                    def mavenPom = readMavenPom file: 'pom.xml'
                    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "java-app-dev-snapshot" : "java-app-dev-release"
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
                    nexusUrl: '192.168.0.178:8081', 
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: nexusRepoName,
                    version: "${mavenPom.version}"
                    }
            }
        }
    }
}


