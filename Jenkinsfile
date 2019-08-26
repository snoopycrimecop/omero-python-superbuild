pipeline {
    agent {
        label 'testintegration'
    }

    environment {
        // Default credentials for testing on devspace
        MAVEN_SNAPSHOTS_REPO_URL = 'http://nexus:8081/nexus/repository/maven-internal/'
        MAVEN_USER = 'admin'
        MAVEN_PASSWORD = 'admin123'

        // Disable Gradle daemon
        GRADLE_OPTS = '-Dorg.gradle.daemon=false'
    }

    stages {
        stage('Versions') {
            steps {

                // Currently disabled. Eventually, this should copy the blitz python zip
                // copyArtifacts(projectName: 'OMERO-build-build', flatten: true, filter: 'version.properties')

                // build is in .gitignore so we can use it as a temp dir
                sh """
                    mkdir ${env.WORKSPACE}/build
                    cd ${env.WORKSPACE}/build && curl -sfL https://github.com/joshmoore/build-infra/archive/python.tar.gz | tar -zxf -
                    export PATH=$PATH:${env.WORKSPACE}/build/build-infra-python/
                    cd ..
                    # Workaround for "unflattened" file, possibly due to matrix
                    find . -name version.properties -exec cp {} . \\;

                    echo versions.omero-blitz=5.5.3 >> version.properties
                    echo versions.omero-common-test=5.5.2 >> version.properties
                    echo versions.omero-gateway=5.5.3 >> version.properties
                    echo FIXME test -e version.properties
                    foreach-get-version-as-property >> version.properties
                """
                archiveArtifacts artifacts: 'version.properties'
            }
        }
        stage('Build') {
            steps {
                sh """
                    cd omero-py
                    python setup.py sdist
                """
                archiveArtifacts artifacts: 'omero-py/dist/omero-py-*'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo see https://help.sonatype.com/repomanager3/formats/pypi-repositories'
                sh 'echo was gradle --init-script init-ci.gradle publish'
            }
        }
    }

    post {
        always {
            // Cleanup workspace
            deleteDir()
        }
    }

}
