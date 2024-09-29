pipeline {
    agent any
    
    triggers {
        githubPush() 
    }

    environment {
        SONAR_SCANNER_PATH = '/opt/sonar-scanner/bin/sonar-scanner'
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        SONAR_HOST = "http://${env.JENKINS_URL.replace('http://','').replace(':8080','').replace('/','')}:9000"
        SONAR_TOKEN = credentials('sonar-token')  // Use Jenkins credentials for security
    }
    
    stages {
        stage('Checkout') {
            steps {
                cleanWs() 
                git url: "https://github.com/AakashSuresh2003/template.git", branch: "main"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    sh """
                    export JAVA_HOME=${JAVA_HOME}
                    export PATH=\$JAVA_HOME/bin:\$PATH
                    ${SONAR_SCANNER_PATH} \
                    -Dsonar.projectKey=MyProject \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=${SONAR_HOST} \
                    -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
        
        stage('Check SonarQube Task Status') {
            steps {
                script {
                    dir('.scannerwork') {
                        def ceTaskId = sh(script: "grep 'ceTaskId' report-task.txt | cut -d'=' -f2", returnStdout: true).trim()
                        def taskStatus = sh(script: "curl -u admin:123456 ${SONAR_HOST}/api/ce/task?id=${ceTaskId}", returnStdout: true).trim()
                        
                        if (taskStatus.contains('"status":"FAILED"')) {
                            error "SonarQube analysis failed!"
                        } else if (taskStatus.contains('"status":"SUCCESS"')) {
                            echo "SonarQube analysis passed!"
                        } else {
                            echo "SonarQube analysis is still in progress."
                        }
                    }
                }
            }
        }
        
        stage('Grant Sudo Privileges to Jenkins') {
            steps {
                script {
                    sh '''
                    echo "jenkins ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/jenkins
                    sudo chmod 440 /etc/sudoers.d/jenkins
                    '''
                    echo "Sudo privileges granted to Jenkins user without password."
                }
            }
        }
        
        stage('Deploy Static Website') {
            steps {
                script {
                    sh '''
                    sudo chown -R www-data:www-data /var/www/html
                    sudo chmod -R 755 /var/www/html
                    sudo cp -r * /var/www/html/
                    echo "Deployment to Apache completed successfully."
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
