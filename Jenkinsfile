pipeline {
    agent {
        kubernetes {
            label 'node-erbium'
        }
    }
    stages {
        // stage('Prepare') {
        //     steps {
        //         container('vault') {
        //             script {
        //                 env.GITHUB_TOKEN = sh(script: "vault read -field=value secret/ops/token/github", returnStdout: true)
        //                 env.DOCKERHUB_AUTH = sh(script: "vault read -field=value secret/gcc/token/dockerhub", returnStdout: true)
        //             }
        //         }
        //     }
        // }
        stage('Build: [ master ]') {
            when {
                branch 'master'
            }
            environment {
                DOCKER_CONFIG="/root/.docker"
            }
            steps {
                script {
                    def projectFolders = sh(returnStdout: true, script: 'ls -d */').trim().split('\n')
                    for (String projectFolder : projectFolders) {
                        // remove trailing slash
                        def project = projectFolder.replaceAll("/\\z", "");
                        dir(project) {
                            def subFolders = sh(returnStdout: true, script: 'ls -d */').trim().split('\n')
                            for (String subFolder : subFolders) {
                                // remove trailing slash
                                def dockerFolder = subFolder.replaceAll("/\\z", "");
                                dir(dockerFolder) {
                                    container('node') {
                                        env.OLD_TAG = sh(script: "node -p \"require('./package').version\"", returnStdout: true)
                                        sh "npm install --ci"
                                        sh "npm run release --ci"
                                        env.TAG = sh(script: "node -p \"require('./package').version\"", returnStdout: true)
                                    }
                                    print(env.TAG)
                                    print(env.OLD_TAG)
                                    if (env.TAG != env.OLD_TAG) {
                                        container (name: 'kaniko', shell: '/busybox/sh') {
                                            sh "#!/busybox/sh\nmkdir -p ${DOCKER_CONFIG}"
                                            sh "#!/busybox/sh\necho '{\"auths\": {\"https://index.docker.io/v1/\": {\"auth\": \"${DOCKERHUB_AUTH}\"}}}' > ${DOCKER_CONFIG}/config.json"
                                            sh "#!/busybox/sh\n/kaniko/executor --destination fdlk/${dockerFolder}:latest --destination fdlk/${dockerFolder}:${TAG} --context `pwd`"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}