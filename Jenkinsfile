#!/bin/groovy

pipeline {
    agent any

    tools {
        maven 'M3'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('inspect') {
            steps {
                sh 'Inspect OpenShift objects'
                openshift.withCluster( ) {
                  openshift.withProject( 'ocp-ops' ) {
                    echo "Hello from project ${openshift.project()} in cluster ${openshift.cluster()}"
                  }
                }
            }
        }
    }
}

