pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "oracle"
    }

    environment {
        SNAP_REPO = 'olaproject-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'olaproject-release'
        CENTRAL_REPO = 'olaproject-maven'
        NEXUSIP = '172.31.1.248'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'olaproject-group'
        NEXUS_LOGIN = 'nexus'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarq'
    }

    stages {
        stage('Build Bro'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "archiving bro"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('test'){
          steps {
            sh 'mvn -s settings.xml test'
          }
        }

        stage('checkstyle analy'){
          steps {
            sh 'mvn -s settings.xml checkstyle:checkstyle'
           }
        }

        stage('CODE ANALYSIS with SONARQUBE'){
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=olaproject \
                   -Dsonar.projectName=olaproject \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
    }
}