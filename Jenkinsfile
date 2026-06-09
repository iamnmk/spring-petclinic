pipeline {
    agent any

    stages {
        stage('SonarQube Analysis') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=spring-petclinic \
                          -Dsonar.projectName="Spring PetClinic"
                        '''
                    }
                }
            }
        }

        stage('Deploy to App VM') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no nmk@192.168.100.6 '
                  cd ~/spring-petclinic
                  git pull origin main
                  docker build -t petclinic-app .
                  docker rm -f petclinic-app || true
                  docker run -d \
                    --name petclinic-app \
                    --network spring-petclinic_default \
                    -p 8080:8080 \
                    -e SPRING_PROFILES_ACTIVE=postgres \
                    -e SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/petclinic \
                    -e SPRING_DATASOURCE_USERNAME=petclinic \
                    -e SPRING_DATASOURCE_PASSWORD=petclinic \
                    petclinic-app
                '
                '''
            }
        }
    }
}
