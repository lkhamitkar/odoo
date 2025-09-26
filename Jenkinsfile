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
                    sh 'git --no-pager log -n3 --pretty=oneline'
                    echo "Picking Branch from GitHub: ${env.BRANCH_NAME}"
                }
            }
        }

        stage('Dependency Validation'){
            steps{
                script{
                    echo 'checking or validating any language dependency'
                    // write a script to get dependency of modules with fields
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
            steps{
                sh '''
                mkdir -p build '''
                //tar -czf build/custom_modules-${ARTIFACT_VERSION}.tar.gz $MODULES
                
            }
        }

        stage('Build Artifact'){
            steps{
                script{
                    def server = Artifactory.server("${ARTIFACTORY_SERVER}")
                    def uploadSpec = """{
                        "files": [{
                            "pattern" : "build/*.tar.gz",
                            "target" : "${ARTIFACTORY_REPO}/"
                        }]
                    }"""
                    server.upload spec:uploadSpec
                }
            }
        }
        
        stage('Deploy to Environment'){
            steps{
                script{
                    if (env.BRANCH_NAME == 'develop'){
                        //deploy script , branch, artifact version here
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
        stage('Manual approval'){
            when {expression { return !params.FORCE}}
            steps{
                timeout(time: 2, unit: 'HOURS'){
                    input message: "Approve production deploy ${params.RELEASE_VERSION}?", submitter: 'name-release'
                }
            }
        }


    }
}