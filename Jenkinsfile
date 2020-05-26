pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * 1-5')
    }
    options {
        skipDefaultCheckout(true)
        // Keep the 10 most recent builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }
    environment {
      PATH="/var/lib/jenkins/miniconda3/bin:$PATH"
    }

    stages {

        stage ("Code pull"){
            steps{
                echo 'Code pull'
                checkout scm
            }
        }
        stage('Build environment') {
            steps {
                echo 'Build environment'
                sh '''conda create --yes -n ${BUILD_TAG} python
                      source activate ${BUILD_TAG} 
                      pip install -r requirements.txt
                    '''
            }
        }
        stage('Test environment') {
            steps {
                echo "Testing stage"
                sh  ''' source activate ${BUILD_TAG}
                        python3 -m pytest --verbose --html=reports/report.html --junit-xml reports/results.xml
                    '''
            }
        }
    }
    post {
        always {
                sh 'conda remove --yes -n ${BUILD_TAG} --all'
                // Archive unit tests for the future
                junit 'reports/results.xml', fingerprint: true
                archiveArtifacts artifacts: 'reports/', fingerprint: true
            }
        failure {
            echo "Send e-mail, when failed"
        }
    }
}