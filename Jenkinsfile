def registry = 'https://reddy01.jfrog.io'
def imageName = 'reddy01.jfrog.io/reddy-docker-local/ttrend'
def version   = '2.1.5'

pipeline {
    agent {
        label 'maven'
    }

    environment {
        PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
        scannerHome = tool 'reddy-sonar-scanner'
    }

    stages {
        stage('Build') {
            steps {
                echo "----------------------- Build started ------------------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------------------- Build Ended --------------------------"
            }
        }

        stage('Test') {
            steps {
                echo "----------------------- Unit test started ------------------------"
                sh 'mvn surefire-report:report'
                echo "----------------------- Unit test Ended --------------------------"
            }
        }

        stage('SonarQube analysis') {
            //environment {
                //scannerHome = tool 'reddy-sonar-scanner'
            //}
            steps {
                withSonarQubeEnv('reddy-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("Quality Gate") {
            steps  {
                script {
                    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv

                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                        def server = Artifactory.newServer url:registry+"/artifactory", credentialsId:"Jfrog_cred"
                        def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                        def uploadSpec = """{
                            "files": [
                                {
                                "pattern": "jarstaging/(*)",
                                "target": "libs-release-local/{1}",
                                "flat": "false",
                                "props" : "${properties}",
                                "exclusions": [ "*.sha1", "*.md5"]
                                }
                            ]
                        }"""
                        def buildInfo = server.upload(uploadSpec)
                        buildInfo.env.collect()
                        server.publishBuildInfo(buildInfo)
                        echo '<--------------- Jar Publish Ended --------------->'                  
                }
            }   
        }

        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName+":"+version)
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage ("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->' 
                    docker.withRegistry(registry, 'Jfrog_cred') {
                        app.push()
                    }
                    echo '<--------------- Docker Publish Ended --------------->' 
                }
            }
        }

        // stage ("Deploy") {
        //     steps {
        //         script {
        //             sh './deploy.sh'
        //         }
        //     }
        // }

        stage(" Deploy ") {
            steps {
                script {
                    echo '<--------------- Helm Deploy Started --------------->'
                    sh 'helm install ttrend ttrend-0.1.0.tgz'
                    echo '<--------------- Helm deploy Ends --------------->'
                }
            }
        }

    }    
}
