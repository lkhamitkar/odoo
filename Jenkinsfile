pipeline {
    agent{
        label 'test'
    }
    options {
        timestamps()
        timeout(time: 90, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '50'))
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'RELEASE_VERSION', defaultValue: '')
    }

    environment{
        ARTIFACTORY_CRED = credentials('id')
        DEV_SSH = credentials('dev-id')
    }

    stages{
        stage('Checkout from SCM'){
            steps {
                checkout from SCM
                git branch : 'main',
                url : 'https://github.com/lkhamitkar/odoo.git',
                credentialsId: 'github_token'
                script{ sh 'git --no-pager log -n3 --pretty=oneline'}
            }
        }

        stage('Dependency Validation'){
            steps{
                script{
                    echo 'checking or validating any language dependency'
                }
            }
        }

        stage('Static Analysis and Security Scan'){
            steps{
                retry(2) {
                    sh './scripts/run-linters.sh' //write linters
                    sh './scripts/run-sast.sh || true'
                }
            }
        }


        stage('t29_custom_one'){
                steps{
                    dir('t29_custom_one'){
                        sh './build.sh --ci'
                        sh './run-test.sh'
                        stash includes: '**/target/**', name:'one-artifact'
                    }
                }
            }
        stage('t29_custom_2'){
                steps{
                    dir('t29_custom_2'){
                        sh './build.sh --ci'
                        sh './run-test.sh'
                        stash includes: '**/target/**', name:'two-artifact'
                    }
                }
            }
        stage('t29_custom_3'){
            steps{
                dir('t29_custom_3'){
                    sh './build.sh --ci'
                    sh './run-test.sh'
                    stash includes: '**/target/**', name:'three-artifact'
                }
            }
        }
        
        stage('Integration Tests'){
            steps{
                unstash 'one-artifact'
                unstash 'two-artifact'
                unstash 'three-artifact'
                sh './scripts/run-integration-tests.sh'
            }
        }
        stage('Publish Artifact'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'artifactory-push', usernameVariable: 'ARTI_USER', passwordVariable : 'ARTI_PASS')])
                sh "./scripts/publish-artifact.sh ${params.RELEASE_VERSION}"
            }
        }
    }
}