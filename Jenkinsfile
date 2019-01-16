pipeline {
  agent {
    label 'dotnetcore'
  }
  stages {
    stage('Build') {
      steps {
        sh "dotnet build"
      }
    }
    stage('Build Image') {
      agent { label 'base' }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("peopleapi") {
              openshift.selector("bc", "peopleapi-bc").startBuild("--wait=true")
            }
          }
        }
      }
    }
    stage('Scan Codebase') {
      agent { label 'sonar-dotnet' }
      steps {
        withSonarQubeEnv('SonarQube-Server') {
          sh "dotnet sonarscanner begin /k:peopleapi /d:sonar.host.url=$SONAR_HOST_URL /d:sonar.login=$SONAR_AUTH_TOKEN"
          sh "dotnet build"
          sh "dotnet sonarscanner end /d:sonar.login=$SONAR_AUTH_TOKEN"
        }
      }
    }
    stage('Microscanner security scan'){
      steps {
        aquaMicroscanner imageName: 'peopleapi/peopleapi:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
      }
    }
    stage('Promote to Staging') {
      agent { label 'base' }
      steps {
        timeout(time:60, unit:'MINUTES') {
          input message: "Would you like to promote the latest development image to the staging project?", ok: "Promote"
        }
        script {
          openshift.withCluster() {
            openshift.tag("peopleapi/peopleapi:latest", "peopleapi/peopleapi:stage")
          }
        }
      }
    }
  }
}