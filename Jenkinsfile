pipeline {
    agent {
        kubernetes {
            label 'node-erbium'
        }
    }
    environment {
    }
    stages {
        // stage("Retrieve secrets") {
        //     steps {
        //         container('vault') {
        //             script {
        //                 env.GITHUB_TOKEN = sh(script: 'vault read -field=value secret/ops/token/github', returnStdout: true)
        //             }
        //         }
        //     }
        // }
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
                                        sh "npm install --ci"
                                        sh "npm run release --ci"
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