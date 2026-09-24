// SPDX-FileCopyrightText: 2021-2025 Zextras <https://www.zextras.com>
//
// SPDX-License-Identifier: AGPL-3.0-only

library(
    identifier: 'jenkins-lib-common@v4.12.3',
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

        stage('Skip CI') {
            steps {
                script { semanticRelease.guard() }
            }
        }

        stage('Build') {
            steps {
                echo 'Building deb/rpm packages'
                buildStage(
                    buildFlags: '',
                    ubuntuSinglePkg: true,
                    preBuildScriptSudo: false,
                    overrides: [
                        'rocky-8': [
                            buildUser: 'yap',
                            buildFlags: '-d -s',
                            preBuildScript: 'sudo dnf install -y gcc-toolset-11-gcc gcc-toolset-11-gcc-c++ git python39',
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
                    ubuntuSinglePkg: true,
                )
            }
        }

        stage('Semantic Release') {
            steps {
                semanticRelease()
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
