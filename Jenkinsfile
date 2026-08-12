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
            checkout scm
        }
    }

    stage('Maven Build') {
        steps {
            sh '''
                echo "Starting Maven build..."

                mvn clean package -DskipTests

                echo "Build completed."

                if [ ! -f target/${JAR_NAME} ]; then
                    echo "ERROR: JAR file not found!"
                    ls -lh target/
                    exit 1
                fi

                echo "JAR file created successfully:"
                ls -lh target/${JAR_NAME}
            '''
        }
    }

    stage('Deploy to EC2') {
        steps {
            sh '''
                echo "Deploying FreshCart to EC2..."

                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo mkdir -p ${APP_DIR}
                    sudo systemctl stop ${APP_NAME} || true
                "

                scp -o StrictHostKeyChecking=no \
                    target/${JAR_NAME} \
                    ubuntu@${SERVER_IP}:/tmp/${JAR_NAME}

                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo mv /tmp/${JAR_NAME} ${APP_DIR}/${JAR_NAME}
                    sudo chown ubuntu:ubuntu ${APP_DIR}/${JAR_NAME}
                "

                echo "Deployment completed."
            '''
        }
    }

    stage('Restart Application') {
        steps {
            sh '''
                echo "Restarting FreshCart application..."

                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo systemctl restart ${APP_NAME}
                "
            '''
        }
    }

    stage('Verify Application') {
        steps {
            sh '''
                echo "Checking FreshCart application status..."

                ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "
                    sudo systemctl is-active ${APP_NAME}
                "

                echo "FreshCart application is running."
            '''
        }
    }
}

post {
    success {
        echo "FreshCart deployment completed successfully!"
    }

    failure {
        echo "FreshCart deployment failed. Check the Jenkins console output."
    }
}
```

}
