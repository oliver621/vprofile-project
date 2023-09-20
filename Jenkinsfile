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
        ARTVERSION = "${env.BUILD_ID}"
        NEXUS_CREDENTIAL_ID = "nexus"
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
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

	    stage("quality gate") {
		   steps {
		        timeout(time: 1, unit: 'HOURS') {
			       waitForQualityGate abortPipeline: true
		        }
		    }
	    }	

stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                            groupId: 'qa',
                            version: "${ARTVERSION}",
                            repository: olaproject-release,
                            credentialsId: "${nexus}",
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } 
		    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        
    }
}
