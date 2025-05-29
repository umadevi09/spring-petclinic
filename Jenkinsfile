pipeline{
    agent any
        stages{

            stage("build"){
                steps{
                    sh 'rm -rf .scannerwork'
                    sh "./mvnw install"
                }
            }
            
            stage("Run Code Analysis"){
            environment {
                SCANNER_HOME = tool 'sonar-scan'
            }
            steps {

                withSonarQubeEnv('SonarServer') {
                   sh '''$SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectKey=myPETC \
                       -Dsonar.projectName=mypetclinc \
                       -Dsonar.sources=. \
                       -Dsonar.java.binaries=target/classes \
                       -Dsonar.exclusions=src/test/java/****/*.java \
                       -Dsonar.analysis.mode=publish \
                       -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}
                    
                    '''
                }
            }
        }
         stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }

        }        
        stage('Upload Artifact to Nexus') {
            steps {
                script {
                    // Extract version from pom.xml
                    def version = sh(
                        script: "./mvnw help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()

                    // Choose repository based on version
                    def repository = version.contains('SNAPSHOT') ? 'maven-snapshots' : 'maven-releases'

                    // Derive artifactId and JAR file
                    def jarFile = sh(
                        script: "ls target/*.jar | grep spring-petclinic | head -n 1",
                        returnStdout: true
                    ).trim()

                    def artifactId = 'spring-petclinic' // This must match the filename and pom.xml <artifactId>

                    // Upload to Nexus
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'nexus:8081',
                        groupId: 'com.example',
                        version: version,
                        repository: repository,
                        credentialsId: 'nexus-creds',
                        artifacts: [
                            [
                                artifactId: artifactId,
                                classifier: '',
                                file: jarFile,
                                type: 'jar'
                            ]
                        ]
                    )
                }
            }
        }
        }
    }
