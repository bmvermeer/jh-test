#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    stage('clean') {
        sh "chmod +x mvnw"
        sh "./mvnw -ntp clean -P-webpack"
    }
    stage('nohttp') {
        sh "./mvnw -ntp checkstyle:check"
    }

    stage('install tools') {
        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm -DnodeVersion=v12.16.1 -DnpmVersion=6.14.5"
    }

//    stage('npm install') {
//        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
//    }

//    stage('backend tests') {
//        try {
//            sh "./mvnw -ntp verify -P-webpack"
//        } catch(err) {
//            throw err
//        } finally {
//            junit '**/target/test-results/**/TEST-*.xml'
//        }
//    }
//
//    stage('frontend tests') {
//        try {
//            sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.npm.arguments='run test'"
//        } catch(err) {
//            throw err
//        } finally {
//            junit '**/target/test-results/**/TEST-*.xml'
//        }
//    }

    stage('packaging') {
        sh "./mvnw -ntp verify -P-webpack -Pprod -DskipTests"
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    // Not required if you install the Snyk CLI on your Agent
    stage('Download Latest Snyk CLI') {
        latest_version = sh(script: 'curl -Is "https://github.com/snyk/snyk/releases/latest" | grep "Location" | sed s#.*tag/##g', returnStdout: true)
        latest_version = latest_version.trim()
        echo "Latest Snyk CLI Version: ${latest_version}"

        snyk_cli_dl_linux="https://github.com/snyk/snyk/releases/download/${latest_version}/snyk-linux"
        echo "Download URL: ${snyk_cli_dl_linux}"

        sh """
            curl -Lo ./snyk "${snyk_cli_dl_linux}"
            chmod +x snyk
            ls -la
            ./snyk -v
        """
    }

    // Authorize the Snyk CLI
    withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN_VAR')]) {
        stage('Authorize Snyk CLI') {
            sh './snyk auth ${SNYK_TOKEN_VAR}'
        }
    }


    // Run snyk test to check for vulnerabilities and fail the build if any are found
    // Consider using --severity-threshold=<low|medium|high> for more granularity (see snyk help for more info).
    stage('Snyk Test using Snyk CLI') {
        sh './snyk test'
    }


}
