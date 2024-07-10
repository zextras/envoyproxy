pipeline {
    agent {
        node {
            label 'base-agent-v1'
        }
    }
    parameters {
        booleanParam defaultValue: false, description: 'Whether to upload the packages in playground repositories', name: 'PLAYGROUND'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timeout(time: 1, unit: 'HOURS')
        skipDefaultCheckout()
    }

    stages {
        stage('Stash') {
            steps {
                checkout scm
                stash includes: '**', name: 'project'
            }
        }
        stage('Build') {
            parallel {
                stage('Ubuntu') {
                    agent {
                        node {
                            label 'yap-agent-ubuntu-20.04-v2'
                        }
                    }
                    steps {
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
                    post {
                        always {
                            archiveArtifacts artifacts: 'artifacts/*.deb', fingerprint: true
                        }
                    }
                }
                stage('RHEL') {
                    agent {
                        node {
                            label 'yap-agent-rocky-8-v2'
                        }
                    }
                    steps {
                        unstash 'project'
                        script {
                            if (BRANCH_NAME == 'devel') {
                                def timestamp = new Date().format('yyyyMMddHHmmss')
                                sh "sudo yap build rocky . -r ${timestamp}"
                            } else {
                                sh 'sudo yap build rocky .'
                            }
                        }
                        stash includes: 'artifacts/x86_64/', name: 'artifacts-rpm'
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'artifacts/x86_64/*.rpm', fingerprint: true
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
                unstash 'artifacts-rpm'
                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec

                    buildInfo = Artifactory.newBuildInfo()
                    uploadSpec = '''{
                        "files": [
                            {
                                "pattern": "artifacts/envoyproxy*.deb",
                                "target": "ubuntu-playground/pool/",
                                "props": "deb.distribution=focal;deb.distribution=jammy;deb.component=main;deb.architecture=amd64"
                            },
                            {
                                "pattern": "artifacts/x86_64/(envoyproxy)-(*).x86_64.rpm",
                                "target": "centos8-playground/zextras/{1}/{1}-{2}.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras"
                            },
                            {
                                "pattern": "artifacts/x86_64/(envoyproxy)-(*).x86_64.rpm",
                                "target": "rhel9-playground/zextras/{1}/{1}-{2}.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras"
                            }
                        ]
                    }'''
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
                unstash 'artifacts-rpm'
                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec

                    buildInfo = Artifactory.newBuildInfo()
                    uploadSpec = '''{
                        "files": [
                            {
                                "pattern": "artifacts/envoyproxy*.deb",
                                "target": "ubuntu-devel/pool/",
                                "props": "deb.distribution=focal;deb.distribution=jammy;deb.component=main;deb.architecture=amd64"
                            },
                            {
                                "pattern": "artifacts/x86_64/(envoyproxy)-(*).x86_64.rpm",
                                "target": "centos8-devel/zextras/{1}/{1}-{2}.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras"
                            },
                            {
                                "pattern": "artifacts/x86_64/(envoyproxy)-(*).x86_64.rpm",
                                "target": "rhel9-devel/zextras/{1}/{1}-{2}.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras"
                            }
                        ]
                    }'''
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                }
            }
        }        
        stage('Upload & Promotion Config') {
            when {
                buildingTag()
            }
            steps {
                unstash 'artifacts-rpm'
                unstash 'artifacts-deb'
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
                    uploadSpec= '''{
                                "files": [
                                    {
                                        "pattern": "artifacts/envoyproxy*.deb",
                                        "target": "ubuntu-rc/pool/",
                                        "props": "deb.distribution=focal;deb.distribution=jammy;deb.component=main;deb.architecture=amd64"
                                    }
                                ]
                            }'''
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
                    uploadSpec= '''{
                                "files": [
                                    {
                                        "pattern": "artifacts/x86_64/(envoyproxy)-(*).x86_64.rpm",
                                        "target": "centos8-rc/zextras/{1}/{1}-{2}.x86_64.rpm",
                                        "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras"
                                    }
                                ]
                            }'''
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
                    uploadSpec= '''{
                                "files": [
                                    {
                                        "pattern": "artifacts/x86_64/(envoyproxy)-(*).x86_64.rpm",
                                        "target": "rhel9-rc/zextras/{1}/{1}-{2}.x86_64.rpm",
                                        "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras"
                                    }
                                ]
                            }'''
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
