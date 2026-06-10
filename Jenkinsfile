pipeline {
    agent any

    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn clean verify sonar:sonar \
                      -DskipTests \
                      -Dsonar.projectKey=spring-petclinic \
                      -Dsonar.projectName="Spring PetClinic" \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.token=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Deploy to K3s on App VM') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no nmk@192.168.100.6 '
                  cd ~/spring-petclinic

                  git pull origin main

                  docker build -t petclinic-app:latest .

                  docker save petclinic-app:latest -o petclinic-app.tar

                  sudo k3s ctr images import petclinic-app.tar

                  sudo kubectl apply -f petclinic-k8s.yaml

                  sudo kubectl rollout restart deployment petclinic-app -n petclinic

                  sudo kubectl rollout status deployment petclinic-app -n petclinic
                '
                '''
            }
        }
    }

    post {
        success {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                sh '''
                curl -X POST -H 'Content-type: application/json' \
                --data "{
                  \\"text\\": \\"✅ K3s Deployment Successful!\\nProject: Spring PetClinic\\nNamespace: petclinic\\nService: petclinic-service\\nURL: http://192.168.100.6:30080\\nBuild: #${BUILD_NUMBER}\\nJob: ${JOB_NAME}\\"
                }" \
                $SLACK_WEBHOOK
                '''
            }
        }

        failure {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                sh '''
                curl -X POST -H 'Content-type: application/json' \
                --data "{
                  \\"text\\": \\"❌ K3s Deployment Failed!\\nProject: Spring PetClinic\\nNamespace: petclinic\\nBuild: #${BUILD_NUMBER}\\nJob: ${JOB_NAME}\\"
                }" \
                $SLACK_WEBHOOK
                '''
            }
        }
    }
}
