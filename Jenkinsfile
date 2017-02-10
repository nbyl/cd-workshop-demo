node {
    stage('build') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/nbyl/confy.git']]])
        sh './gradlew -Pprod build'
    }

    stage('publish docker image') {
        sh './gradlew publishImage'
    }

    stage('integration test') {
        sh 'helm init'
        sh 'helm version'

        sh 'helm upgrade --install it-db stable/postgresql --namespace=integration-test --set persistence.enabled=false,postgresUser=confy,postgresPassword=confy01,postgresDatabase=confy'
        sh 'helm upgrade --install it-confy helm/confy --namespace=integration-test --set database.driver=org.postgresql.Driver,database.url=jdbc:postgresql://it-db-postgresql/confy,database.username=confy,database.password=confy01'
        sh './gradlew integrationTest -PintegrationTestBaseUrl=http://it-confy-confy.integration-test.svc.cluster.local'
    }
}