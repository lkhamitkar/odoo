pipeline {
    triggers {
        githubPush()
    }

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
        stage('Validate Branch Name') {
            when {
                not { anyOf { branch 'develop'; branch 'staging'; branch 'main' } }
            }
            steps {
                githubNotify{
                    context: 'CI/CD Pipeline',
                    description: 'BUILD PENDING',
                    status: 'PENDING'
                }
                script {
                    if (!env.BRANCH_NAME.startsWith("feature/")) {
                        error("Invalid branch name: ${env.BRANCH_NAME}. Must start with 'feature/'.")
                    }
                }
            }
        }
        stage('Build and Dependency Validation'){
            when { branch 'develop' }
            steps{
                script{
                    echo 'checking or validating any language dependency'
                    // write a script to get dependency of modules with fields
                }
            }
        }
        stage('Static Analysis and Security Scan'){
            when { branch 'develop' }
            parallel {
                stage('Python Lint'){
                    steps{
                        retry(2) {
                            echo "running SAST"
                            sh './scripts/run-linters.sh' //write linters
                            sh ' pip install flake8 pylint '
                            // install plugins and run linters.
                            
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
        stage('Build Package'){
            when { branch 'develop' }
            steps{
                sh '''
                mkdir -p build '''
                // tar -czf build/custom_modules-${ARTIFACT_VERSION}.tar.gz $MODULES
                
            }
        }

        stage('Build Artifact'){
            when { branch 'develop' }
            steps{
                script{
                    echo 'building artifact'
                    // def server = Artifactory.server("${ARTIFACTORY_SERVER}")
                    // def uploadSpec = """{
                    //     "files": [{
                    //         "pattern" : "build/*.tar.gz",
                    //         "target" : "${ARTIFACTORY_REPO}/"
                    //     }]
                    // }"""
                    // server.upload spec:uploadSpec
                }
            }
        }
        
        stage('Deploy to Environment'){
            steps{
                script{
                    def targetBranch = env.CHANGE_TARGET
                    echo "Target Branch : ${targetBranch}"
                    if (targetBranch == 'develop'){
                        //deploy script , branch, artifact version here
                        echo 'deploy develop branch'
                    } else if (targetBranch == 'staging'){
                        echo 'deploy staging branch'
                    } else if (targetBranch == 'main'){
                        echo 'deploy main branch to Production. Approval needed!'
                    }else {
                        echo 'feature branch detected: skipping deploy branch '
                    }
                }
            }
        }
        stage('Manual approval'){
            when {expression { return env.BRANCH_NAME == 'main' && !params.FORCE}}
            steps{
                timeout(time: 2, unit: 'HOURS'){
                    input message: "Approve production deploy ${params.RELEASE_VERSION}?", submitter: 'name-release'
                }
            }
        }


    }
    post {
        success {
            githubNotify{
                    context: 'CI/CD Pipeline',
                    description: 'BUILD SUCCESS',
                    status: 'SUCCESS'
                }
            echo "Build succeeded on ${env.BRANCH_NAME}"
        }
        
            
        
        
        failure {
            githubNotify{
                    context: 'CI/CD Pipeline',
                    description: 'BUILD FAILURE',
                    status: 'FAILURE'
                }
            echo " Build failed on ${env.BRANCH_NAME}"
        }
    }

}