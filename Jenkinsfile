pipeline {
    agent {
        label '007'
    }
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "auto_maven"
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    stages {
        stage('Clear running apps') {
            steps {
                sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }   
        stage('Build Docker Image'){
            steps {
               sh "mvn package -Pdocker"
            }
        }
        stage('Run Docker') {
            steps {
               sh "docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}"
            }
        }
        stage('Test Selenium') {
            steps {
                sh "mvn test -Pselenium"
            }
        }
        stage('Application Deployment') {
            steps {
                withMaven(globalMavenSettingsConfig: 'null', jdk: 'null', maven: 'auto_maven', mavenSettingsConfig: '636b1cbf-d42c-411e-a6a4-49912261e4b4') {
                    sh 'mvn deploy'
                }
            }     
         }
        post {
            success {  
                sh 'docker stop pandaapp'
                deleteDir()
            }
        }
    }
}
