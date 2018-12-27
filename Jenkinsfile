pipeline {
    agent any

    parameters {
        string(name: 'GOMINT_BUILD_NUMBER', defaultValue: 'lastSuccessfulBuild', description: 'Which Gomint build should i bundle?')
    }

    stages {
        def app

        stage('Clone repository') {
            steps {
                /* Let's make sure we have the repository cloned to our workspace */
                checkout scm
            }
        }

        stage('Build image') {
            steps {
                /* This builds the actual image; synonymous to
                 * docker build on the command line */
                dir("docker") {
                    app = docker.build("gomint/gomint", "--build-arg buildNumber=${GOMINT_BUILD_NUMBER}")
                }
            }
        }

        stage('Push image') {
            steps {
                /* Finally, we'll push the image with two tags:
                 * First, the incremental build number from Jenkins
                 * Second, the 'latest' tag.
                 * Pushing multiple tags is cheap, as all the layers are reused. */
                docker.withRegistry('https://registry.hub.docker.com', 'docker-gomint') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                }
            }
        }
    }
}