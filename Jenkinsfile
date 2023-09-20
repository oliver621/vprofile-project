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
    }
}