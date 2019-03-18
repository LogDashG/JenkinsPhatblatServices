#!/usr/bin/env groovy
/*
 * Jenkinsfile
 * JenkinsfileTemplate
 *
 *
 */

import jenkins.model.*

def MAIL_TO = JenkinsLocationConfiguration.get().getAdminAddress()
echo "MAIL_TO: $MAIL_TO"

properties([
    // Don't trigger this job when changes are found from branch indexing.
    //overrideIndexTriggers(false),
    disableConcurrentBuilds(),
    pipelineTriggers([
        cron('H 1 * * *')
    ]),
    buildDiscarder(logRotator(numToKeepStr: '100')),
])

// Global variables
String output

try {
    timeout(time: 1, unit: 'HOURS') {
        withEnv(['LANG=en_US.UTF-8']) {
            node {
                stage("🛒 Checkout") {
                    checkout scm
                }
                stage("📦 Bundler") {
                    // https://jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#sh-shell-script
                    sh(
                        script: "gem list bundler",
                        label: "💎 List gems"
                    )

                    // Capture stdout
                    output = sh(
                        script: "which bundle",
                        label: "❓ Which",
                        returnStdout: true
                    ).trim()

                    // Suppress build failure
                    Integer status = sh(
                        script: """
                            false
                        """,
                        label: "✔️ Check",
                        returnStatus: true
                    )

                    if (status != 0) {
                        echo "Aborting build. 😞"
                        currentBuild.rawBuild.@result = hudson.model.Result.ABORTED
                    }
                }

                echo "currentBuild.result: ${currentBuild.result}"

                // build.@result = hudson.model.Result.SUCCESS
                // build.@result = hudson.model.Result.NOT_BUILT
                // build.@result = hudson.model.Result.UNSTABLE
                // build.@result = hudson.model.Result.FAILURE
                // build.@result = hudson.model.Result.ABORTED

                if (currentBuild.result && currentBuild.result != 'SUCCESS') {
                    return
                }

                stage("❓ Conditional Stage") {
                }
            }
        }
    }
}
catch (Exception ex) {
    String jobName = env.JOB_NAME
    String buildNumber = env.BUILD_NUMBER

    String subject = "👷🏻‍♂️ [FAILURE] $jobName - Build #$buildNumber"
    String body = """$jobName
                |Build #${buildNumber} - FAILURE
                |
                |${env.BUILD_URL}
                |
                |${ex}
                |
                |${env.BUILD_URL}console
                |""".stripMargin()

    mail to: MAIL_TO,
         subject: subject,
         body: body
    throw ex
}
