    pipeline {
      environment {
        /*
        * Uses a Jenkins credential called "FOOCredentials" and creates {params.Environment} variables:
        * "$FOO" will contain string "USR:PSW"
        * "$FOO_USR" will contain string for Username
        * "$FOO_PSW" will contain string for Password
        */
        FOO = credentials("dockerhub")
      }
      agent any
      parameters {
              choice(
                  name: 'Component',
                  choices:"vue-app")
              string(
                  name: 'Branchname',
                  defaultValue:"master")
      }
      stages {
          stage('SCM') {
              steps {
                echo 'Check out cdp on master'
                      cleanWs()
                      checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '$Branchname']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/sambitsonu1992/vue-application.git']]]
                  script {
                      GIT_COMMIT_HASH = sh (script: "git rev-parse --short HEAD | tr '\n' ' '", returnStdout: true)
                      GIT_VERSION = sh (script: "git describe --tags", returnStdout: true)
                      GIT_LAST_UPGRADE_DATE = sh (script: "git rev-parse HEAD | git show -s --format=%ci", returnStdout: true)
                      echo "**************************************************"
                      echo "${GIT_COMMIT_HASH}"
                      echo "**************************************************"
                      sh "touch app-version.info"
                      sh "echo ${GIT_COMMIT_HASH} >> app-version.info"
                  }
                }
          }
        stage ('BUILD, PUBLISH & DEPLOY vue-test') {
          when {
              expression { params.Component == 'vue-app' }
          }
          steps {
            echo 'Building docker image for Vue Application vue-application'
            sh 'id && docker images ; docker ps -a'
            sh "docker build -t anandpatnaik/vue:${params.Environment}-$GIT_COMMIT_HASH  --build-arg GIT_VERSION=\"$GIT_VERSION\" --build-arg GIT_LAST_UPGRADE_DATE=\"$GIT_LAST_UPGRADE_DATE\"  ."

            echo 'Publishing image to docker hub'
            sh "docker login --username=$FOO_USR --password=$FOO_PSW"
            sh "docker push anandpatnaik/vue:${params.Environment}-$GIT_COMMIT_HASH"
			echo 'Image Pushed to Docker Hub'
			
            }
          }
        }		
     }
} 
