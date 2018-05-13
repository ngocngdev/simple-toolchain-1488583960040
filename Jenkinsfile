node {
    def sonarHost;
    def authToken;
    stage('SCM') {
        git 'https://github.com/ccox-IBM/simple-toolchain-1488583960040.git';
    }
    stage('SonarQube analysis') {
        // requires SonarQube Scanner 2.8+
        def scannerHome = tool 'Default SQ Scanner';

        withSonarQubeEnv('Default SQ Server') {
            sh "${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=simple-toolchain-1488583960040 \
                    -Dsonar.sources=. ";
            //save host and auth token used to configure the SQ tool
            sonarHost = SONAR_HOST_URL;
            authToken = SONAR_AUTH_TOKEN + ":";
        }
    }
    stage("Quality Gate") {

        timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate(); // Reuse taskId previously collected by withSonarQubeEnv

            // Query SQ API to get the information we want to send to DLMS
            def QGResults = (sonarHost + "/api/qualitygates/project_status?projectKey=simple-toolchain-1488583960040").toURL()
                        .getText(requestProperties: [Authorization: 'Basic ' + authToken.bytes.encodeBase64().toString()]);
            def IssuesResults = (sonarHost + "/api/issues/search?componentKeys=simple-toolchain-1488583960040&statuses=OPEN").toURL()
                        .getText(requestProperties: [Authorization: 'Basic ' + authToken.bytes.encodeBase64().toString()]);
            def ratings = (sonarHost + "/api/measures/component?metricKeys=reliability_rating,security_rating,sqale_rating&componentKey=simple-toolchain-1488583960040").toURL()
                        .getText(requestProperties: [Authorization: 'Basic ' + authToken.bytes.encodeBase64().toString()]);

            // Convert response strings into JSON objects
            QGResults = new groovy.json.JsonSlurperClassic().parseText(QGResults);
            IssuesResults = new groovy.json.JsonSlurperClassic().parseText(IssuesResults);
            ratings = new groovy.json.JsonSlurperClassic().parseText(ratings);

            def payload = new groovy.json.JsonBuilder();

            // construct JSON object we want to send to DLMS
            payload(
                qualityGate: QGResults.projectStatus,
                issues: IssuesResults.issues,
                ratings: ratings.component.measures
            )

            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
    }
    stage("Upload SonarQube Scan Results") {
        echo "Upload Results Once Support is added";
    }
}
