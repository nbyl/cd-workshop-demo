node {
    stage('build') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/nbyl/confy.git']]])
        sh './gradlew -Pprod build'
    }
}