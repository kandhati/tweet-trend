pipeline {
    agent {
        label 'maven'
    }

    environment {
        PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
    }

    stages {
        stage('build') {
            steps {
                echo "----------------------- Build started ------------------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------------------- Build Ended ------------------------"
            }
        }
        stage('test') {
            steps {
                echo "----------------------- Unit test started ------------------------"
                sh 'mvn surefire-report:report'
                echo "----------------------- Unit test Ended ------------------------"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'reddy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('reddy-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }    
}