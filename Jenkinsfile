pipeline {
  agent any
  stages {
    stage('cucumber') {
      steps {
        cucumber(fileIncludePattern: 'feature/cucumber/*.json.', buildStatus: 'SUCCESS', mergeFeaturesById: true, skipEmptyJSONFiles: true, stopBuildOnFailedReport: true)
      }
    }
  }
}