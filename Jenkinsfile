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
                            def customImage = docker.build("gomint/gomint:${env.BUILD_ID}", "--no-cache --build-arg buildNumber=${GOMINT_BUILD_NUMBER} .")
                            customImage.push()
                            customImage.push("latest")
                        }
                    }
                }
            }
        }

        stage('Update dev server') {
            steps {
                sh """
                    docker stop gomint || true
                    docker pull gomint/gomint:latest
                    docker run -d --rm -p 0.0.0.0:19132:19132/udp --name gomint gomint/gomint:latest
                """
            }
        }
    }
}