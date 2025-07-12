pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    environment {
        DEBUG = 'true'
        appVersion = '' 
        region = 'us-east-1'
        account_id = '932478401603'
        project = 'expense'
        environment = 'dev'
        component = 'frontend'
    }
    
    stages {
        stage('Read the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App version: ${appVersion}"
                }
            }
        }
        stage('Docker build') {
            steps {
                withAWS(region: "${region}", credentials: 'aws-creds') {
                    script {
                        def imageURI = "${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${environment}/${component}:${appVersion}"
                        sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                        docker build -t ${imageURI} .

                        docker images

                        docker push ${imageURI}
                        """
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                withAWS(region: "${region}", credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                        helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .
                    """
                }
            }
        }
    }

    post {
        always {
            echo "This section runs always"
            deleteDir()
        }
        success {
            echo "This section runs on success"
        }
        failure {
            echo "This section runs on failure"
        }
    }
}
