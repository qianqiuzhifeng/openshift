pipeline {
  agent any
  stages {
    stage('cucumber') {
      steps {
        cucumber(fileIncludePattern: '**/*.json', buildStatus: 'SUCCESS', mergeFeaturesById: true, skipEmptyJSONFiles: true, stopBuildOnFailedReport: true, jsonReportDirectory: 'feature/cucumber')
      }
    }
  }
}