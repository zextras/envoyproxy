pipeline {
    agent {
        node {
            label 'base'
        }
    }
    parameters {
        booleanParam defaultValue: false, description: 'Whether to upload the packages in playground repositories', name: 'PLAYGROUND'
    }
    options {
        timeout(time: 6, unit: 'HOURS')
        skipDefaultCheckout()
    }
    stages {
        stage('Stash') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                }
                stash includes: '**', name: 'project'
            }
        }
        stage('Build') {
            parallel {
                stage('Ubuntu') {
                    agent {
                        node {
                            label 'yap-ubuntu-20-v1'
                        }
                    }
                    steps {
                        container('yap') {
                            unstash 'project'
                            script {
                                if (BRANCH_NAME == 'devel') {
                                    def timestamp = new Date().format('yyyyMMddHHmmss')
                                    sh "sudo yap build ubuntu . -r ${timestamp}"
                                } else {
                                    sh 'sudo yap build ubuntu .'
                                }
                            }
                            stash includes: 'artifacts/', name: 'artifacts-deb'
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'artifacts/*.deb', fingerprint: true
                        }
                    }
                }
                stage('RHEL8') {
                    agent {
                        node {
                            label 'yap-rocky-8-v1'
                        }
                    }
                    steps {
                        container('yap') {
                            unstash 'project'
                            script {
                                sh '''
                                    sudo dnf install -y \
                                        gcc-toolset-13 \
                                        gcc-toolset-13-gcc-c++ \
                                        gcc-toolset-13-binutils \
                                        binutils-devel \
                                        libstdc++-devel \
                                        git\
                                        python312
                                    sudo useradd -m worker
                                    '''                                

                                if (BRANCH_NAME == 'devel') {
                                    def timestamp = new Date().format('yyyyMMddHHmmss')
                                    sh "sudo -u worker yap build rocky-8 . -r ${timestamp} -d"
                                } else {
                                    sh 'sudo -u worker yap build rocky-8 . -d'
                                }
                            }
                            stash includes: 'artifacts/*el8*.rpm', name: 'artifacts-rocky-8'
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'artifacts/*el8*.rpm', fingerprint: true
                        }
                    }
                }
                stage('RHEL9') {
                    agent {
                        node {
                            label 'yap-rocky-9-v1'
                        }
                    }
                    steps {
                        container('yap') {
                            unstash 'project'
                            script {
                                if (BRANCH_NAME == 'devel') {
                                    def timestamp = new Date().format('yyyyMMddHHmmss')
                                    sh "sudo yap build rocky-9 . -r ${timestamp}"
                                } else {
                                    sh 'sudo yap build rocky-9 .'
                                }
                            }
                            stash includes: 'artifacts/*el9*.rpm', name: 'artifacts-rocky-9'
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'artifacts/*el9*.rpm', fingerprint: true
                        }
                    }
                }
            }
        }
        stage('Upload To Playground') {
            when {
                anyOf {
                    branch 'playground/*'
                    expression { params.PLAYGROUND == true }
                }
            }
            steps {
                unstash 'artifacts-deb'
                unstash 'artifacts-rocky-8'
                unstash 'artifacts-rocky-9'

                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec

                    buildInfo = Artifactory.newBuildInfo()
                    uploadSpec = """{
                        "files": [
                            {
                                "pattern": "artifacts/envoyproxy*.deb",
                                "target": "ubuntu-playground/pool/",
                                "props": "deb.distribution=focal;deb.distribution=jammy;deb.distribution=noble;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/(envoyproxy)-(*).el8.x86_64.rpm",
                                "target": "centos8-playground/zextras/{1}/{1}-{2}.el8.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/(envoyproxy)-(*).el9.x86_64.rpm",
                                "target": "rhel9-playground/zextras/{1}/{1}-{2}.el9.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}"
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                }
            }
        }
        stage('Upload To Devel') {
            when {
                branch 'devel'
            }
            steps {
                unstash 'artifacts-deb'
                unstash 'artifacts-rocky-8'
                unstash 'artifacts-rocky-9'

                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec

                    buildInfo = Artifactory.newBuildInfo()
                    uploadSpec = """{
                        "files": [
                            {
                                "pattern": "artifacts/envoyproxy*.deb",
                                "target": "ubuntu-devel/pool/",
                                "props": "deb.distribution=focal;deb.distribution=jammy;deb.distribution=noble;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/(envoyproxy)-(*).el8.x86_64.rpm",
                                "target": "centos8-devel/zextras/{1}/{1}-{2}.el8.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/(envoyproxy)-(*).el9.x86_64.rpm",
                                "target": "rhel9-devel/zextras/{1}/{1}-{2}.el9.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}"
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                }
            }
        }        
        stage('Upload & Promotion Config') {
            when {
                buildingTag()
            }
            steps {
                unstash 'artifacts-deb'
                unstash 'artifacts-rocky-8'
                unstash 'artifacts-rocky-9'

                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec
                    def config

                    // since artifactory doesn't support a build with multiple repository involved
                    // we artificially create 3 different artifactory builds by changing the build
                    // name with "-ubuntu" "-centos8"

                    //ubuntu
                    buildInfo = Artifactory.newBuildInfo()
                    buildInfo.name += '-ubuntu'
                    uploadSpec= """{
                                "files": [
                                    {
                                        "pattern": "artifacts/envoyproxy*.deb",
                                        "target": "ubuntu-rc/pool/",
                                        "props": "deb.distribution=focal;deb.distribution=jammy;deb.distribution=noble;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                                    }
                                ]
                            }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                    config = [
                            'buildName'          : buildInfo.name,
                            'buildNumber'        : buildInfo.number,
                            'sourceRepo'         : 'ubuntu-rc',
                            'targetRepo'         : 'ubuntu-release',
                            'comment'            : 'Do not change anything! just press the button',
                            'status'             : 'Released',
                            'includeDependencies': false,
                            'copy'               : true,
                            'failFast'           : true
                    ]
                    Artifactory.addInteractivePromotion server: server, promotionConfig: config, displayName: 'Ubuntu Promotion to Release'
                    server.publishBuildInfo buildInfo

                    // rhel8
                    buildInfo = Artifactory.newBuildInfo()
                    buildInfo.name += '-centos8'
                    uploadSpec= """{
                                "files": [
                                    {
                                        "pattern": "artifacts/(envoyproxy)-(*).el8.x86_64.rpm",
                                        "target": "centos8-rc/zextras/{1}/{1}-{2}.el8.x86_64.rpm",
                                        "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}"
                                    }
                                ]
                            }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                    config = [
                            'buildName'          : buildInfo.name,
                            'buildNumber'        : buildInfo.number,
                            'sourceRepo'         : 'centos8-rc',
                            'targetRepo'         : 'centos8-release',
                            'comment'            : 'Do not change anything! just press the button',
                            'status'             : 'Released',
                            'includeDependencies': false,
                            'copy'               : true,
                            'failFast'           : true
                    ]
                    Artifactory.addInteractivePromotion server: server, promotionConfig: config, displayName: 'RHEL8 Promotion to Release'
                    server.publishBuildInfo buildInfo

                    // rhel9
                    buildInfo = Artifactory.newBuildInfo()
                    buildInfo.name += '-rhel9'
                    uploadSpec= """{
                                "files": [
                                    {
                                        "pattern": "artifacts/(envoyproxy)-(*).el9.x86_64.rpm",
                                        "target": "rhel9-rc/zextras/{1}/{1}-{2}.el9.x86_64.rpm",
                                        "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}"
                                    }
                                ]
                            }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                    config = [
                            'buildName'          : buildInfo.name,
                            'buildNumber'        : buildInfo.number,
                            'sourceRepo'         : 'rhel9-rc',
                            'targetRepo'         : 'rhel9-release',
                            'comment'            : 'Do not change anything! just press the button',
                            'status'             : 'Released',
                            'includeDependencies': false,
                            'copy'               : true,
                            'failFast'           : true
                    ]
                    Artifactory.addInteractivePromotion server: server, promotionConfig: config, displayName: 'RHEL9 Promotion to Release'
                    server.publishBuildInfo buildInfo
                }
            }
        }
    }
    post {
        always {
            script {
                GIT_COMMIT_EMAIL = sh(
                        script: 'git --no-pager show -s --format=\'%ae\'',
                        returnStdout: true
                ).trim()
            }
            emailext attachLog: true, body: '$DEFAULT_CONTENT', recipientProviders: [requestor()], subject: '$DEFAULT_SUBJECT', to: "${GIT_COMMIT_EMAIL}"
        }
    }
}
