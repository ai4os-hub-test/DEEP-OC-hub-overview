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
                    def REPO = "deep-oc-dogs_breed_det"     // same REPO for DockerHub and GitHub                    
                    def ORG = "vykozlov"                    // same ORG for DockerHub and GitHub
                    def URL_HUB = "https://hub.docker.com/v2"
                    def DOCKER_REPO_URL="${URL_HUB}/repositories/${ORG}/${REPO}/"
                    def README_URL = "https://raw.githubusercontent.com/${ORG}/${REPO}/master/README.md"

                    echo "${README_URL}"

                    // get Docker Hub Token
                    sh "wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 && chmod +x ./jq"
                    TOKEN = sh(
                        script: "curl -s -H \"Content-Type: application/json\" -X POST -d '{\"username\": \"${DOCKER_CREDS_USR}\", \"password\": \"${DOCKER_CREDS_PSW}\"}' ${URL_HUB}/users/login/ | ./jq -r .token | tr -d '\n\t'",
                        returnStdout: true,
                    )               

                    def WORKSPACE = pwd()
                    def README_PATH = "${WORKSPACE}/_README.md"
                    // download GitHub README.md
                    sh("curl -o ${README_PATH} ${README_URL}")
                    RESPONSE_CODE = sh(script:
                        "curl -s --write-out %{response_code} --output /dev/null -H \"Authorization: JWT ${TOKEN}\" -X PATCH --data-urlencode full_description@${README_PATH} ${DOCKER_REPO_URL}",
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
