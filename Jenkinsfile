node {
    def version

    try {
        stage('build') {
            checkout scm
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'build']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/nbyl/confy.git']]])
            version = sh returnStdout: true, script: 'cd build && ./gradlew -q printVersion'
            sh 'cd build && ./gradlew -Pprod -xtest build'
        }

        stage('publish docker image') {
            sh 'cd build && ./gradlew publishImage'
        }

        stage('integration test') {
            sh 'helm init'
            sh 'helm version'

            sh 'helm upgrade --install it-db helm/postgresql --namespace=integration-test --set persistence.enabled=false,postgresUser=confy,postgresPassword=confy01,postgresDatabase=confy'
            sh "helm upgrade --install it-confy helm/confy --namespace=integration-test --set database.driver=org.postgresql.Driver,database.url=jdbc:postgresql://it-db-postgresql/confy,database.username=confy,database.password=confy01,image.tag=${version}"

            sh 'cd build && ./gradlew integrationTest -PintegrationTestBaseUrl=http://it-confy-confy.integration-test.svc.cluster.local/api/'
        }

        stage('user acceptance test') {
            sh 'helm upgrade --install uat-db helm/postgresql --namespace=uat --set persistence.enabled=true,persistence.storageClass=generic,postgresUser=confy,postgresPassword=confy01,postgresDatabase=confy'
            sh "helm upgrade --install uat-confy helm/confy --namespace=uat --set database.driver=org.postgresql.Driver,database.url=jdbc:postgresql://it-db-postgresql/confy,database.username=confy,database.password=confy01,ingress.enabled=true,ingress.path=/confy-uat,image.tag=${version}"
        }
    } catch (err) {
        echo "Caught: ${err}"
        currentBuild.result = 'FAILURE'
    } finally {
        junit allowEmptyResults: true, testResults: 'build/build/test-results/*.xml'
    }
}