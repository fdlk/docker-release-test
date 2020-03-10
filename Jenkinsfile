pipeline {
    agent {
        kubernetes {
            label 'node-erbium'
        }
    }
    environment {
        DOCKER_CONFIG="/root/.docker"
    }
    stages {
        stage('Build: [ PR ]') {
            when {
                changeRequest()
            }
            steps {
                script {
                    // fetch target branch, to see if directories got changed
                    sh "git fetch --no-tags origin ${CHANGE_TARGET}:refs/remotes/origin/${CHANGE_TARGET}"
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
                                    env.CHANGES = sh(script = "git diff --name-only origin/${CHANGE_TARGET} -- .", returnStdout=true)
                                    if (env.CHANGES) {
                                        container (name: 'kaniko', shell: '/busybox/sh') {
                                            sh "#!/busybox/sh\nmkdir -p ${DOCKER_CONFIG}"
                                            sh "#!/busybox/sh\necho '{\"auths\": {\"https://index.docker.io/v1/\": {\"auth\": \"${DOCKERHUB_AUTH}\"}}}' > ${DOCKER_CONFIG}/config.json"
                                            sh "#!/busybox/sh\n/kaniko/executor --destination molgenis/${dockerFolder}:latest --destination molgenis/${dockerFolder}:${TAG} --context `pwd`"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
        stage('Build: [ master ]') {
            when {
                branch 'master'
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
                                    if (env.TAG != env.OLD_TAG) {
                                        container (name: 'kaniko', shell: '/busybox/sh') {
                                            sh "#!/busybox/sh\nmkdir -p ${DOCKER_CONFIG}"
                                            sh "#!/busybox/sh\necho '{\"auths\": {\"https://index.docker.io/v1/\": {\"auth\": \"${DOCKERHUB_AUTH}\"}}}' > ${DOCKER_CONFIG}/config.json"
                                            sh "#!/busybox/sh\n/kaniko/executor --destination molgenis/${dockerFolder}:latest --destination molgenis/${dockerFolder}:${TAG} --context `pwd`"
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