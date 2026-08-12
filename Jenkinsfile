pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    pkill -f "freshcart.*\\.jar" || true
                    nohup java -jar target/*.jar > app.log 2>&1 &
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10
                    curl -f http://localhost:8080/actuator/health
                '''
            }
        }
    }

    post {
        success {
            echo 'Maven CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Maven CI/CD pipeline failed!'
        }
    }
}
