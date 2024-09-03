pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') { 
            agent {
                docker {
                    image 'qnib/pytest' 
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py' 
            }
            post {
                always {
                    junit 'test-reports/results.xml' 
                }
            }
        }
        stage("Manual Approval") {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                input message: 'Lanjutkan ke tahap Deploy?'
            }
        }
         stage('Deliver') {
                    agent any
                    //This environment block defines two variables which will be used later in the 'Deliver' stage.
                    environment {
                        VOLUME = '$(pwd)/sources:/src'
                        IMAGE = 'cdrx/pyinstaller-linux:python2'
                    }
                    steps {
                        //This dir step creates a new subdirectory named by the build number.
                        //The final program will be created in that directory by pyinstaller.
                        //BUILD_ID is one of the pre-defined Jenkins environment variables.
                        //This unstash step restores the Python source code and compiled byte
                        //code files (with .pyc extension) from the previously saved stash. image]
                        //and runs this image as a separate container.
                        dir(path: env.BUILD_ID) {
                            unstash(name: 'compiled-results')

                            //This sh step executes the pyinstaller command (in the PyInstaller container) on your simple Python application.
                            //This bundles your add2vals.py Python application into a single standalone executable file
                            //and outputs this file to the dist workspace directory (within the Jenkins home directory).
                            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                        }
                    }
                    post {
                        success {
                            //This archiveArtifacts step archives the standalone executable file and exposes this file
                            //through the Jenkins interface.
                            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                        }
                    }
        }
    }
}