#!/bin/groovy

pipeline {
  node("maven") {
   checkout scm
   stage("inspect openshift objects") {
    steps {
     openshift.withCluster( ) {
      openshift.withProject( 'ocp-ops' ) {
        echo "Hello from project ${openshift.project()} in cluster ${openshift.cluster()}"
      }
     }
    }

   }
  }

}
