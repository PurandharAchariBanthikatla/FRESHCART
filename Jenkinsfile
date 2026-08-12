pipeline {
agent any

environment {
    PROJECT_URL = 'https://github.com/PurandharAchariBanthikatla/FRESHCART.git'
    SERVER_IP   = '15.206.94.9'
    APP_NAME    = 'freshcart-backend'
    APP_DIR     = '/opt/freshcart'
    JAR_NAME    = 'freshcart-backend.jar'
}

stages {

    stage('Checkout') {
        steps {
            echo 'Checking out FreshCart source code...'
            checkout scm
        }
    }

    stage('Maven Build') {
        steps {
            echo 'Starting Maven build...'

            sh '''
                mvn clean package -DskipTests

                echo "Maven build completed."

                if [ ! -f "target/${JAR_NAME}" ]; then
                    echo "ERROR: ${JAR_NAME} was not found."
                    echo "Contents of target directory:"
                    ls -lh target/
                    exit 1
                fi

                echo "Generated JAR:"
                ls -lh "target/${JAR_NAME}"
            '''
        }
    }

    stage('Deploy to EC2') {
        steps {
            echo 'Deploying FreshCart JAR to EC2...'

            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo mkdir -p ${APP_DIR}
                    sudo systemctl stop ${APP_NAME} || true
                "

                scp -o StrictHostKeyChecking=no \
                    "target/${JAR_NAME}" \
                    "ubuntu@${SERVER_IP}:/tmp/${JAR_NAME}"

                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo mv /tmp/${JAR_NAME} ${APP_DIR}/${JAR_NAME}
                    sudo chown ubuntu:ubuntu ${APP_DIR}/${JAR_NAME}
                "

                echo "JAR successfully copied to EC2."
            '''
        }
    }

    stage('Restart Application') {
        steps {
            echo 'Restarting FreshCart application...'

            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo systemctl restart ${APP_NAME}
                "
            '''
        }
    }

    stage('Verify Application') {
        steps {
            echo 'Verifying FreshCart application...'

            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo systemctl is-active --quiet ${APP_NAME}
                "

                echo "FreshCart application is running successfully."
            '''
        }
    }
}

post {
    success {
        echo '========================================='
        echo 'FreshCart deployment SUCCESSFUL!'
        echo '========================================='
    }

    failure {
        echo '========================================='
        echo 'FreshCart deployment FAILED!'
        echo 'Check the Jenkins console output.'
        echo '========================================='
    }
}

}
