/* The "node" statement specifies that we want to run
   this code on any available Jenkins worker. Without the
   node call, nothing runs. */
node {
    /* Set up a stage, which is a step in the build,
       called "Setup Build Environment". */
    stage("Setup Build Environment") {
        // Checkout the code from source control.
        checkout scm
        // Clone all submodules.
        sh "echo hello"
        sh "git submodule update --init --recursive"
    }

    // Set the desired Python version
    def pythonVersion = "2.7"

    /* Set the docker image name that for this build container.
       It's generally a good idea to use the same docker image per
       branch, so that containers are cached and thus the builds are faster. */
    def ci_image_name = "simple_jenkins_test_${pythonVersion}_${env.BUILD_NUMBER}"

    /* use the docker global variable to build an image named
       ci_image_name. We build with a dockerfile named Dockerfile.simple,
       pass PYTHON_VERSION as a build-arg, and then set the context to "."
       Note that we assign the result to the variable ci_environment. */
    def ci_environment = docker.build(
        ci_image_name,
        "-f Dockerfile " +
        "--build-arg PYTHON_VERSION=${pythonVersion} " + ".")

    /* With the variable ci_environment, we can use ci_environment.inside
       to run more stages within the built Docker container. */
    ci_environment.inside {
        /* Set up another stage, and run bash commands to log
           some information about the build environment. */
        stage("Print Build Environment Info") {
            sh '''#!/bin/bash		
                conda info
                python --version
                pwd
                ls
            '''
        }
        // Run your tests here (there's a placeholder statement for now).
        stage("Run Tests") {
            sh '''#!/bin/bash		
                echo "Running Tests"
            '''
        }
        /* Run your linter / post-test steps here (there's a placeholder
           statement for now). */
        stage("Check Lint") {
            sh '''#!/bin/bash		
                echo "Checking Lint"
            '''
        }
    }
    // At the very end, have a stage for cleanup.
    stage("Cleanup") {
        /* These shell commands kill docker containers that are exited,
           and then remove dangling images to save disk space. */
        sh '''#!/bin/bash
            docker rm -v $(docker ps -a -q -f status=exited) || true
            docker rmi $(docker images -f "dangling=true" -q) || true
        '''
        // Delete the workspace when the build is done
        cleanWs()
        deleteDir()
    }
}
