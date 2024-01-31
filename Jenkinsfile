@Library('deploy-my-project') _

def dockerfileapp = 'Build-UntitTest/'
def Token_Sonar = 'Token_Sonar'
def SonarScannerHome = "tool = 'SonarQube'" // Assuming 'SonarQube' is the tool name
def SonarProjectKey = 'test-project'
def SonarHostUrl = 'http://54.183.182.49:9000'
def Dockerhub = 'Dockerhub'
def DockerRegistry = "amrabunemr98"
def DockerImage = "Project-APP"
def OpenShiftConfig = "OpenShiftConfig"
def OpenShiftProject = 'abu-nemr'

pipeline {
    agent any

    stages {
        stage('Build App and Unit Test') {
            steps {
                script {
                    BuildUnittest(dockerfileapp)
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    SonarQube(Token_Sonar, SonarScannerHome, SonarProjectKey, SonarHostUrl)
                }
            }
        }

        stage('Build Docker image for app.py and push it to docker hub') {
            steps {
                script {
                    COMMIT_HASH = BuildandPushDockerImage(Dockerhub, DockerRegistry, DockerImage)
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    COMMIT_HASH = DeployOpenShift(OpenShiftConfig, DockerRegistry, DockerImage, OpenShiftProject)
                }
            }
        }
    }
}
