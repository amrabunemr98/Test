pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "amrabunemr98"
        DOCKER_IMAGE = "test"
        imageTagApp = "build-${BUILD_NUMBER}-app"
        imageNameapp = "${DOCKER_REGISTRY}:${imageTagApp}"
        OPENSHIFT_PROJECT = 'abu-nemr'
        docker_file_app = 'Build-UntitTest/'
        SONAR_PROJECT_KEY = 'test-project'
        SONAR_HOST_URL = 'http://54.193.207.61:9000'
        SONAR_SCANNER_HOME = tool 'SonarQube'
        OPENSHIFT_VERSION = '4.9.0'
    }

    stages {
        stage('Build Docker image for app.py and push it to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhub', usernameVariable: 'DOCKER_REGISTRY_USERNAME', passwordVariable: 'DOCKER_REGISTRY_PASSWORD')]) {
                    // Authenticate with Docker Hub to push Docker image
                    sh "echo \${DOCKER_REGISTRY_PASSWORD} | docker login -u \${DOCKER_REGISTRY_USERNAME} --password-stdin"
                    
                    // Build Docker image for app.py
                    sh "docker build -t ${imageNameapp} ."

                    sh "docker tag ${imageNameapp} docker.io/${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}"
                    
                    // Push the app Docker image to Docker Hub
                    sh "docker push docker.io/${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}"
                    
                    // Remove the locally built app Docker image
                    sh "docker rmi -f ${imageNameapp}"
                }
            }
        }

        stage('Install OpenShift Client Tools') {
            steps {
                script {
                    // Download and install OpenShift Client Tools
                    def ocHome = tool 'openshift', "openshift-client-${OPENSHIFT_VERSION}"
                    env.PATH = "${ocHome}/:${env.PATH}"
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'OpenShiftConfig', variable: 'OPENSHIFT_SECRET')]) {
                    def ocHome = tool name: 'openshift', type: 'oc-tool', version: "${OPENSHIFT_VERSION}"
                    env.PATH = "${ocHome}:${env.PATH}"                   
                        // Replace the placeholder with the actual Docker image in the Kubernetes YAML files
                        sh "sed -i 's|image:.*|image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}|g' ./deployment.yml"
                        
                        // Apply the deployment file
                        sh "oc apply -f deployment.yml -n ${OPENSHIFT_PROJECT} --kubeconfig=\$OPENSHIFT_SECRET"
                    }
                }
            }
        }
    }
}
