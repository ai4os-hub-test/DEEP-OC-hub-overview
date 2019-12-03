#!/usr/bin/groovy

@Library(['github.com/indigo-dc/jenkins-pipeline-library@1.2.3']) _

pipeline {
    agent {
        label 'docker-build'
    }

    stages {
        stage('Docker image delete tag') {
            when {
                anyOf {
                   branch 'master'
               }
            }
            environment {
                DOCKER_CREDS = credentials('indigobot')
            }
            steps{
                checkout scm
                script {
                    def REPO = "deep-oc-dogs_breed_det"    // same REPO for DockerHub and GitHub                    
                    def ORG = "vykozlov"                    // same ORG for DockerHub and GitHub
                    def URL_HUB = "https://hub.docker.com/v2"
                    def README_URL = "https://raw.githubusercontent.com/${ORG}/${REPO}/master/README.md"
                    def DOCKER_REPO_URL="${URL_HUB}/repositories/${ORG}/${REPO}/"

                    echo "${README_URL}"

                    // get Docker Hub Token
                    sh "wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 &&  chmod +x ./jq"
                    TOKEN = sh(
                        script: "curl -s -H \"Content-Type: application/json\" -X POST -d '{\"username\": \"${DOCKER_CREDS_USR}\", \"password\": \"${DOCKER_CREDS_PSW}\"}' ${URL_HUB}/users/login/ | ./jq -r .token",
                        returnStdout: true,
                    )               

                    def README_PATH = "_README.md"
                    RESPONSE_CODE = sh(
                        script: "export README_PATH="${PWD}/${README_PATH}" && \
                        curl -o ${README_PATH} ${README_URL} && \
                        cat ${README_PATH} && \
                        curl -s --write-out %{response_code} --output /dev/null -H \"Authorization: JWT ${TOKEN}\" -X PATCH --data-urlencode full_description@${README_PATH} ${DOCKER_REPO_URL}",
                        returnStdout: true,
                    )

                    echo "[INFO] Received response code: ${RESPONSE_CODE}";
                }
            }
            post {
                always {
                    DockerClean()
                }
            }
        }
    }
}
