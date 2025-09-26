pipeline {
    triggers {
        githubPush()
    }

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

    // environment{
    //     ARTIFACTORY_CRED = credentials('id')
    //     // DEV_SSH = credentials('dev-id')
    // }

    stages{
        stage('Checkout from SCM'){
            steps {
                checkout scm
                script {
                    sh 'git --no-pager log -n3 --pretty=oneline'
                }
            }
        }

        stage('Env Debug') {
            steps {
                    script {
                        echo "Picking Branch from GitHub: ${env.BRANCH_NAME}"
                    }
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
            parallel {
                stage('Python Lint'){
                    steps{
                        retry(2) {
                            echo "running SAST"
                            // sh './scripts/run-linters.sh' //write linters
                            sh ' pip install flake8 pylint '
                            // flake8 custom_addons/*
                            //pylint custom_addons/*
                            
                        }
                    }
                }
                stage('Manifest & XML Validation'){
                    steps{
                        echo "validating manifest file"
                        // for module in custom_addons/*; do
                        //     test -f "$module/__manifest__.py" || echo "$module missing manifest!"
                        // done
                        
                    }
                }
            }
        }

        // build artifact stage

        stage('t29_custom_one'){
                steps{
                    dir('t29_custom_one'){
                        // sh './build.sh --ci' build CI script
                        // sh './run-test.sh' supporting test
                        // stash includes: '**/target/**', name:'one-artifact' artifactory store
                    }
                }
            }
        stage('t29_custom_2'){
                steps{
                    dir('t29_custom_2'){
                        // sh './build.sh --ci'
                        sh './run-test.sh'
                        // stash includes: '**/target/**', name:'two-artifact'
                    }
                }
            }
        stage('t29_custom_3'){
            steps{
                dir('t29_custom_3'){
                    // sh './build.sh --ci'
                    // sh './run-test.sh'
                    // stash includes: '**/target/**', name:'three-artifact'
                }
            }
        }
        
        stage('Deploy to Environment'){
            steps{
                script{
                    if (env.BRANCH_NAME == 'develop'){
                        echo 'deploy develop branch'
                    } else if (env.BRANCH_NAME == 'staging'){
                        echo 'deploy staging branch'
                    } else if (env.BRANCH_NAME == 'main'){
                        echo 'deploy main branch to Production. Approval needed!'
                    }else {
                        echo 'feature branch detected: skipping deploy branch '
                    }
                }
            }
        }
        
        // stage('Integration Tests'){
        //     steps{
        //         unstash 'one-artifact'
        //         unstash 'two-artifact'
        //         unstash 'three-artifact'
        //         sh './scripts/run-integration-tests.sh'
        //     }
        // }
        // stage('Publish Artifact'){
        //     steps{
        //         withCredentials([usernamePassword(credentialsId: 'artifactory-push', usernameVariable: 'ARTI_USER', passwordVariable : 'ARTI_PASS')])
        //         sh "./scripts/publish-artifact.sh ${params.RELEASE_VERSION}"
        //     }
        // }
        post {
            always {
                echo "Pipeline finished for branch : ${env.BRANCH_NAME}"
            }
            success{
                echo " Deployment Successful!"
            }
            failure{
                echo " Deployment failed. Check Logs"
            }
        }
    }
}