pipeline {
    agent{
        label 'dev'
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
        ARTIFACTORY_SERVER = 'jfrog-artifactory'
        ARTIFACTORY_REPO = 'odoo-custom-modules-local'
        ARTIFACT_VERSION = "${env.BUILD_NUMBER}"
        MODULES = "t29_custom_one t29_custom_2 t29_custom_3" // import the modules
    }

    stages{
        stage('Checkout from SCM'){
            steps {
                checkout scm
                
                script {
                    // configured via webhook as checkout from scm doesnt give any control.
                    sh 'git --no-pager log -n3 --pretty=oneline'
                    // webhook doesnt work with localhost hence used ngrok- create public API for localhost
                    echo "Picking Branch from GitHub: ${env.BRANCH_NAME}"
                }
            }
        }

        //developers are forced to create feature/* branches 
        stage('Validate Branch Rules') {
            when {
                // this can be written in script. As currently it will skip the stage which sometimes is unclear.
                // currently skipping stage is a passing condition
                not {
                    anyOf {
                        allOf {
                            expression { env.CHANGE_TARGET == 'develop' && env.CHANGE_BRANCH?.startsWith('feature/') }
                        }
                        allOf {
                            expression { env.CHANGE_TARGET == 'staging' && env.CHANGE_BRANCH == 'develop' }
                        }
                        allOf {
                            expression { env.CHANGE_TARGET == 'main' && env.CHANGE_BRANCH == 'staging' }
                        }
                        allOf {
                            expression { env.CHANGE_TARGET == 'main' && env.CHANGE_BRANCH?.startsWith('hotfix/') }
                        }
                    }
                }
            }
            steps {
                script {
                    error("Invalid PR flow detected: ${env.CHANGE_BRANCH} → ${env.CHANGE_TARGET}")
                }
            }
        }
    
        stage('Static Analysis and Security Scan'){
            when {
                anyOf {
                    expression { env.CHANGE_TARGET == 'develop' }
                    expression { env.CHANGE_BRANCH?.startsWith('hotfix/') }
                }
            }
            parallel {
                stage('Python Linting'){
                    steps{
                        retry(2) {
                            echo "running SAST"
                            // run script for linters such as Pylint or code check Black
                            
                        }
                    }
                }
                stage('Manifest & XML Validation'){
                    steps{
                        echo "validating manifest file"
                        // validate the XML or perform pre-build steps
                    }
                }
            }
        }


        stage('Run only for PRs targeting develop') {
            when {
                expression { env.CHANGE_TARGET == 'develop' }
            }
            steps {
                echo "This stage runs only when the PR target branch is 'develop'"
                //Write test Scripts
            }
        }

        stage('Run only for PRs targeting staging') {
            when {
                expression { env.CHANGE_TARGET == 'staging' }
            }
            steps {
                echo "This stage runs only when the PR target branch is 'develop'"
                //Write test Scripts
            }
        }

        stage('Run only for PRs targeting main') {
            when {
                expression { env.CHANGE_TARGET == 'main' }
            }
            steps {
                echo "This stage runs only when the PR target branch is 'develop'"
                //Write test Scripts
            }
        }

        stage('Manual approval'){
            when {expression { return env.CHANGE_TARGET == 'main' && !params.FORCE}}
            steps{
                timeout(time: 2, unit: 'HOURS'){
                    input message: "Approve production deploy ${params.RELEASE_VERSION}?", submitter: 'name-release'
                }
            }
        }
    }
    post {
        success {
            githubNotify context: "CI/CD Pipeline", description: "BUILD SUCCESS", status: "SUCCESS",  credentialsId: 'github_token'
            echo "Build succeeded on ${env.BRANCH_NAME}"
        }
        
        failure {
            githubNotify context: "CI/CD Pipeline", description: "BUILD FAILURE", status: "FAILURE",  credentialsId: 'github_token'
            echo " Build failed on ${env.BRANCH_NAME}"
        }
    }

}