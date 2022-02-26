pipeline {
    agent any
    stages {
        stage('Static Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'SonarQube') {
                        sh 'mvn sonar:sonar'
                    }
                    timeout(time:1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if(qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage('Checking for Vulnerabilities') {
            steps {
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'Dependency Checker'
            }
        }
        stage('Building Artifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Building Image') {
            steps {
                sh 'docker image build -t webapp .'
            }
        }
    }
    post {
        always {
            dependencyCheckPublisher (
            pattern: '',
            failedTotalLow: 1,
            failedTotalMedium: 1,
            failedTotalHigh: 1,
            failedTotalCritical: 1
          )
        }
        steps {
            if(currentBuild.result == 'UNSTABLE') {
                unstable('UNSTABLE: Dependency check')
            } else if(currentBuild.result == 'FAILURE') {
                error('FAILED: Dependency check')
            }
        }
    }
}