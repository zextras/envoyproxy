library(
    identifier: 'jenkins-lib-common@1.1.2',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        credentialsId: 'jenkins-integration-with-github-account',
        remote: 'git@github.com:zextras/jenkins-lib-common.git',
    ])
)

properties(defaultPipelineProperties())

pipeline {
    agent {
        node {
            label 'base'
        }
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        parallelsAlwaysFailFast()
        skipDefaultCheckout()
        timeout(time: 6, unit: 'HOURS')
    }

    stages {
        stage('Setup') {
            steps {
                checkout scm
                script {
                    gitMetadata()
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building deb/rpm packages'
                buildStage(
                    buildFlags: '',
                    ubuntuSinglePkg: true,
                    overrides: [
                        'rocky-8': [
                            buildUser: 'worker',
                            buildUserCreate: true,
                            buildFlags: '-d',
                            preBuildScript: 'dnf install -y gcc-toolset-11-gcc gcc-toolset-11-gcc-c++ git python39',
                        ]
                    ]
                )
            }
        }

        stage('Upload artifacts') {
            when {
                expression { return uploadStage.shouldUpload() }
            }
            tools {
                jfrog 'jfrog-cli'
            }
            steps {
                uploadStage(
                    packages: yapHelper.resolvePackageNames(),
                    ubuntuSinglePkg: true,
                )
            }
        }
    }

    post {
        always {
            emailext([
                attachLog: true,
                body: '$DEFAULT_CONTENT',
                recipientProviders: [requestor()],
                subject: '$DEFAULT_SUBJECT',
                to: "${GIT_COMMIT_EMAIL}"
            ])
        }
    }
}
