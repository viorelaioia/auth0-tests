import groovy.json.JsonOutput

def environments = [
  desktop: [
    browserName: 'Firefox',
    version: '59.0',
    platform: 'Windows 10'
]
]

def writeCapabilities(desiredCapabilities) {
    def defaultCapabilities = [
        build: env.BUILD_TAG,
        public: 'public restricted'
    ]
    def capabilities = defaultCapabilities.clone()
    capabilities.putAll(desiredCapabilities)
    def json = JsonOutput.toJson([capabilities: capabilities])
    writeFile file: 'capabilities.json', text: json
}

pipeline {
  agent {label 'mesos-testing'}
  options {
    timestamps()
    timeout(time: 1, unit: 'HOURS')
  }
  environment {
    VARIABLES = credentials('AUTH_TESTS_VARIABLES')
    PYTEST_ADDOPTS =
      "-n=auto " +
      "--tb=short " +
      "--driver=SauceLabs " +
      "--variables=capabilities.json " +
      "--variables=${VARIABLES}"
    SAUCELABS_API_KEY = credentials('SAUCELABS_API_KEY')
  }
  stages {
    stage('Lint') {
      steps {
        sh "tox -e flake8"
      }
    }
    stage('Test') {
      steps {
        writeCapabilities(capabilities, 'capabilities.json')
        sh "tox -e py27"
      }
      post {
        always {
          archiveArtifacts 'results/*'
          junit 'results/*.xml'
          publishHTML(target: [
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'results',
            reportFiles: "py27.html",
            reportName: 'HTML Report'])
        }
      }
    }
  }
}