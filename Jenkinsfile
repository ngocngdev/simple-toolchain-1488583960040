node {
    def sonarHost;
    def authToken;
    stage('SCM') {
        git 'https://github.com/ccox-IBM/simple-toolchain-1488583960040.git'
    }
    stage('SonarQube analysis') {
        // requires SonarQube Scanner 2.8+
        def scannerHome = tool 'Default SQ Scanner';

        withSonarQubeEnv('Default SQ Server') {
            sh "${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=simple-toolchain-1488583960040 \
                    -Dsonar.sources=. \
                    -Dsonar.organization=ccox-ibm-github"
            echo SONAR_AUTH_TOKEN
            sonarHost = SONAR_HOST_URL;
            echo SONAR_HOST_URL
            authToken = SONAR_AUTH_TOKEN + ":";
        }

        //def slurper = new groovy.json.JsonSlurper()
        //results = slurper.parseText(results)
        //println results.projectStatus.conditions[0].metricKey;


    }
    stage("Quality Gate") {
        echo "IN THE GATE"
        echo authToken;
        timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            def QGResults = (sonarHost + "/api/qualitygates/project_status?projectKey=simple-toolchain-1488583960040").toURL()
                        .getText(requestProperties: [Authorization: 'Basic ' + authToken.bytes.encodeBase64().toString()])
            echo QGResults;
            def IssuesResults = (sonarHost + "/api/issues/search?componentKeys=simple-toolchain-1488583960040&statuses=OPEN").toURL()
                        .getText(requestProperties: [Authorization: 'Basic ' + authToken.bytes.encodeBase64().toString()])
            echo IssuesResults;
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }

        }
    }
}
