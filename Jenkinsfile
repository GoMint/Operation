pipeline {
    agent any

    parameters {
        string(name: 'GOMINT_BUILD_NUMBER', defaultValue: 'lastSuccessfulBuild', description: 'Which Gomint build should i bundle?')
    }

    stages {
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
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-gomint') {
                            def customImage = docker.build("gomint/gomint:${env.BUILD_ID}", "--build-arg buildNumber=${GOMINT_BUILD_NUMBER} .")
                            customImage.push()
                            customImage.push("latest")
                        }
                    }
                }
            }
        }
    }
}