library(
    identifier: 'jenkins-packages-build-library@1.0.2',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'git@github.com:zextras/jenkins-packages-build-library.git',
        credentialsId: 'jenkins-integration-with-github-account'
    ])
)

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

    parameters {
        booleanParam defaultValue: false,
            description: 'Whether to upload the packages in playground repositories',
            name: 'PLAYGROUND'
    }

    tools {
        jfrog 'jfrog-cli'
    }

    stages {
        stage('Stash') {
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
                            'buildUser': 'worker',
                            'buildFlags': ' -d ',
                            'preBuildScript': '''
                                dnf install -y gcc-toolset-11-gcc gcc-toolset-11-gcc-c++ git python39
                                useradd -m worker
                            ''',
                        ]
                    ]
                )
            }
        }

        stage('Upload artifacts')
        {
            steps {
                uploadStage(
                    packages: yapHelper.getPackageNamesFromFiles(),
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
