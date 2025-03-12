pipeline {
    agent any

    environment {
        WEB_REPO = 'git@github.com:ihsanguci/sample-web.git'
        TEST_REPO = 'git@github.com:ihsanguci/automation-base.git'
        WEB_DIR = '/var/www/html'  // Sesuaikan dengan direktori web di Nginx
        TAG = 'smoke'  // Default tag untuk testing
    }

    stages {
        stage('Checkout Repositories') {
            steps {
                script {
                    echo "Cloning Web Repository..."
                    sh "rm -rf web && git clone ${WEB_REPO} web"

                    echo "Cloning Automation Test Repository..."
                    sh "rm -rf automation && git clone ${TEST_REPO} automation"
                }
            }
        }

        stage('Run Automation Testing') {
            steps {
                dir('automation') {
                    script {
                        echo "Running Automation Tests..."
                        sh """
                            chmod +x ./gradlew
                            ./gradlew clean test

                            chmod +x ./runner.sh  # Pastikan file bisa dieksekusi
                            ./runner.sh "@smoke"  # Jalankan script dengan parameter TAG
                        """
                    }
                }
            }
        }

        stage('Generate Allure Report') {
            steps {
                dir('automation') {
                    script {
                        echo "Generating Allure Report..."
                        sh "allure generate ${ALLURE_RESULTS} -o ${ALLURE_REPORT} --clean"
                    }
                }
            }
        }

        stage('Publish Allure Report') {
            steps {
                allure([
                    results: [[path: 'automation/allure-results']],
                    reportBuildPolicy: 'ALWAYS'
                ])
            }
        }

        stage('Deploy to Nginx') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    echo "Deploying to Nginx..."
                    sh "sudo rm -rf ${WEB_DIR}/*"
                    sh "sudo cp -r web/* ${WEB_DIR}/"
                    sh "sudo systemctl restart nginx"
                }
            }
        }
    }

    post {
        failure {
            echo "Tests failed. Deployment stopped!"
        }
         always {
            archiveArtifacts artifacts: 'automation/allure-report/**', fingerprint: true
        }
    }
}