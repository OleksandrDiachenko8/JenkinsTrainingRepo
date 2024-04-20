pipeline {
    agent { label 'oracle' }
    environment {
     MYVARIABLE = "some-variable"
    }
    
    options {
        copyArtifactPermission 'read-artifact'
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }
    stages {
        stage('Source') {
            steps {
                sh 'git --version | tee versions.txt' 
                sh 'python3 --version | tee -a versions.txt'
                git branch: 'nodetask',
                    url: 'https://github.com/OleksandrDiachenko8/JenkinsTrainingRepo'
            }
        }           

        stage('Requirements') {
            steps {
                dir("${env.WORKSPACE}/code"){
                    sh 'python3 -m venv venv'
                    sh 'pwd'
                    sh './venv/bin/pip3 install --upgrade --requirement requirements.txt'
                }
            }
        }
        stage('Lint') {
            steps {
                dir("${env.WORKSPACE}/code"){
                    sh 'venv/bin/flake8 --ignore=E501,E231 *.py'
                    sh 'venv/bin/pylint --errors-only --disable=C0301 --disable=C0326 *.py'
                }
            }
        }
        stage('Test') {
            steps {
                dir("${env.WORKSPACE}/code"){
                    sh('''
                        venv/bin/coverage run -m pytest -v test_*.py \
                            --junitxml=pytest_junit.xml
                    ''')
                }
            }
        }
        stage('Build') {
            steps {
                echo "Build the application in this step..."
            }
        }
        stage('Create-env-Artifact') {
            steps {                
                sh "set > envreport.txt"                                
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploy the application in this step..."
            }
        }
    }
    
    post {
        always {
            dir("${env.WORKSPACE}/code"){
                sh 'venv/bin/coverage.xml'
            }

            junit allowEmptyResults: true, testResults: '**/pytest_junit.xml'

            junit allowEmptyResults: true, testResults: '**/pylint_junit.xml'

            publishCoverage adapters: [cobertura('**/coverage.xml')],
                sourceFileResolver: sourceFiles('STORE_LAST_BUILD')
            archiveArtifacts allowEmptyArchive: true,
                artifacts: '**/*.txt, **/*.xml,',
                fingerprint: true,
                followSymlinks: false,
                onlyIfSuccessful: true
        }
    }

}
