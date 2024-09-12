//  a JenkinsFile to build iqtree
// paramters
//  1. git branch
// 2. git url


properties([
    parameters([
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build'),
        booleanParam(defaultValue: false, description: 'Use CIBIV cluster?', name: 'USE_CIBIV'),
    ])
])
pipeline {
    agent any
    environment {
        GITHUB_REPO_URL = "https://github.com/iqtree/cmaple.git"
        NCI_ALIAS = "gadi"
        SSH_COMP_NODE = ""
        WORKING_DIR = "/scratch/dx61/tl8625/cmaple/ci-cd"
        GITHUB_REPO_NAME = "cmaple"
        BUILD_SCRIPTS = "${WORKING_DIR}/build-scripts"
        REPO_DIR = "${WORKING_DIR}/${GITHUB_REPO_NAME}"
        BUILD_OUTPUT_DIR = "${WORKING_DIR}/builds"

        // build directories
        /*

            1. build_default --> build cmaple
         */
        BUILD_DEFAULT = "${BUILD_OUTPUT_DIR}/build-default"

    }
    stages {
    	stage('Init variables') {
            steps {
                script {
                    if (params.USE_CIBIV) {
                    	NCI_ALIAS = "eingang"
                    	SSH_COMP_NODE = " ssh -tt cox "
                    	WORKING_DIR = "/project/AliSim/cmaple"
        				BUILD_SCRIPTS = "${WORKING_DIR}/build-scripts"
        				REPO_DIR = "${WORKING_DIR}/${GITHUB_REPO_NAME}"
       					BUILD_OUTPUT_DIR = "${WORKING_DIR}/builds"
       					BUILD_DEFAULT = "${BUILD_OUTPUT_DIR}/build-default"
                    }
                }
            }
        }
    // ssh to NCI_ALIAS and scp build-scripts to working dir in NCI
        stage('Copy build scripts') {
            steps {
                script {
                    sh "pwd"
                    sh """
                        ssh -tt ${NCI_ALIAS} << EOF
                        
                        mkdir -p ${WORKING_DIR}
                        mkdir -p ${BUILD_SCRIPTS}
                        exit
                        EOF
                        """
                    sh "scp -r build-scripts/* ${NCI_ALIAS}:${BUILD_SCRIPTS}"
                }
            }
        }
        stage('Setup environment') {
            steps {
                script {
                    sh """
                        ssh -tt ${NCI_ALIAS} << EOF
                        
                        mkdir -p ${WORKING_DIR}
                        cd  ${WORKING_DIR}
                        git clone --recursive ${GITHUB_REPO_URL}
                        cd ${GITHUB_REPO_NAME}
                        git checkout ${params.BRANCH}
                        mkdir -p ${BUILD_OUTPUT_DIR}
                        cd ${BUILD_OUTPUT_DIR}
                        rm -rf *
                        exit
                        EOF
                        """
                }
            }
        }
        stage("Build: Build Default") {
            steps {
                script {
                    sh """
                        ssh -tt ${NCI_ALIAS} ${SSH_COMP_NODE}<< EOF
                        
                        chmod +x ${BUILD_SCRIPTS}/jenkins-cmake-build-default.sh 
                        sh ${BUILD_SCRIPTS}/jenkins-cmake-build-default.sh ${BUILD_DEFAULT} ${REPO_DIR} 
                       	
                        exit
                        EOF
                        """
                }
            }
        }

        stage ('Verify') {
            steps {
                script {
                    sh "ssh -tt ${NCI_ALIAS} 'cd ${WORKING_DIR} && ls -l'"

                }
            }
        }


    }
    post {
        always {
            echo 'Cleaning up workspace'
            cleanWs()
        }
    }
}

def void cleanWs() {
    // ssh to NCI_ALIAS and remove the working directory
    sh "ssh ${NCI_ALIAS} 'rm -rf ${REPO_DIR} ${BUILD_SCRIPTS}'"
}