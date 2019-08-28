
def s2iimage = 'quay.io/quarkus/ubi-quarkus-native-s2i:19.1.1' 

pipeline {
  agent {
    node {
      label 'maven' 
    }
  }
  options {
    timeout(time: 20, unit: 'MINUTES') 
  }
  environment {
      def templateName = "${s2iimage}~${env.GIT_URL}#${env.GIT_BRANCH}"
      def deploymentName = "${env.JOB_NAME}".replace("/","-").take(52)
  }
  stages {
    stage('preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                    }
                }
                sh "echo ${templateName}"
                sh "echo ${deploymentName}"
            }
        }
    }
//    stage('cleanup') {
//      steps {
//        script {
//            openshift.withCluster() {
//                openshift.withProject() {
//                  openshift.selector("all", [ template : templateName ]).delete() 
//                  if (openshift.selector("secrets", templateName).exists()) { 
//                    openshift.selector("secrets", templateName).delete()
//                  }
//                }
//            }
//        }
//      }
//    }
    stage('create') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def bc = openshift.selector("bc", deploymentName)
                  if(bc) {
                    bc.startBuild()
                  }
                  else {
                    openshift.newApp("${templateName} --name ${deploymentName} --labels=name=${deploymentName},gitUrl=${env.GIT_URL},gitBranch=${env.GIT_BRANCH}") 
                  }
                }
            }
        }
      }
    }
    stage('build') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def builds = openshift.selector("bc", deploymentName).related('builds')
                  timeout(7) { 
                    builds.untilEach(1) {i
                      it.logs()
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }
            }
        }
      }
    }
    stage('deploy') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def rm = openshift.selector("dc", deploymentName).rollout().latest()
                  timeout(7) { 
                    openshift.selector("dc", deploymentName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }
        }
      }
    }
    stage('tag') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.tag("${templateName}:latest", "${templateName}-staging:latest") 
                }
            }
        }
      }
    }
    stage('final delete') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.selector("all", [ name : deploymentName ]).delete() 
                  if (openshift.selector("secrets", deploymentName).exists()) { 
                    openshift.selector("secrets", deploymentName).delete()
                  }
                }
            }
        }
      }
    }

  }
}

