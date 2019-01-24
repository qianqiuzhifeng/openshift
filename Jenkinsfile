pipeline {
  agent any
  stages {
    stage('cucumber') {
      steps {
        cucumber(fileIncludePattern: '**/*.json', buildStatus: 'SUCCESS', mergeFeaturesById: true, skipEmptyJSONFiles: true, stopBuildOnFailedReport: true, jsonReportDirectory: 'feature/cucumber')
        cobertura(coberturaReportFile: 'coverage.xml', autoUpdateHealth: true, autoUpdateStability: true)
      }
    }
    stage('error') {
      steps {
        withSonarQubeEnv('SonarQube Scanner') {
          waitForQualityGate true
        }

      }
    }
  }
}