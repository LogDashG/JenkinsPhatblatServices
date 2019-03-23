#!/usr/bin/env groovy
/*
 * Jenkinsfile
 * JenkinsPhatblatServices
 *
 * Updates pbjenkins formula in phatblat/homebrew-services.
 */

import jenkins.model.*

def MAIL_TO = JenkinsLocationConfiguration.get().getAdminAddress()
echo "MAIL_TO: $MAIL_TO"

properties([
    buildDiscarder(logRotator(numToKeepStr: '100')),
    disableConcurrentBuilds(),
    // Don't trigger this job when changes are found from branch indexing.
    //overrideIndexTriggers(false),
    parameters([
        string(
            name: 'newVersion',
            defaultValue: '2.168',
            description: 'New version of Jenkins.'
        ),
        string(
            name: 'fileHash',
            defaultValue: '83756847b09e754829aaf343d2f11a8c84b097e922d1770dc53b8c8267184e62',
            description: 'SHA-256 hash of war file.'
        ),
    ]),
])

// Parmaeters
String newVersion = params.newVersion
String fileHash = params.fileHash

// Global variables
String fileName = "Formula/pbjenkins.rb"
String fileContents

try {
    timeout(time: 1, unit: 'HOURS') {
        withEnv(['LANG=en_US.UTF-8']) {
            node {
                stage("✔️ Parameters") {
                    echo "newVersion: $newVersion"
                    echo "fileHash: $fileHash"

                    if (newVersion == null || fileHash == null) {
                        String message = "Required parameters are missing"
                        echo message
                        currentBuild.rawBuild.@result = hudson.model.Result.FAILURE
                        throw new Exception(message)
                    }
                }
                stage("🛒 Checkout") {
                    git url: "git@github.com:phatblat/homebrew-services.git"
                }
                stage("⚖️ Compare Version") {
                    // https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/#readfile-read-file-from-workspace
                    String oldContents = readFile fileName
                    echo oldContents

                    def lines = oldContents.split('\n')

                    lines.eachWithIndex { line, index ->
                        echo "line $index: $line"

                        // Version in url
                        if (line.startsWith("  url")) {
                            // Find version
                            // url "http://mirrors.jenkins.io/war/2.167/jenkins.war"
                            if (line.contains(newVersion)) {
                                String message = "Version $newVersion is already in formula."
                                echo message
                                currentBuild.rawBuild.@result = hudson.model.Result.ABORTED
                                throw new Exception(message)
                            }

                            fileContents += "  url \"http://mirrors.jenkins.io/war/$newVersion/jenkins.war\"\n"
                        }
                        else if (line.startsWith("  sha256")) {
                            // sha256 "5218a0db16e5815eec7f286006b677d935bc4be7a3ea9fef8aba087041c8a37e"
                            fileContents += "  sha256 \"$filehash\"\n"
                        }
                        else {
                            fileContents += line + '\n'
                        }
                    }
                }

                echo "currentBuild.result: ${currentBuild.result}"
                if (currentBuild.result && currentBuild.result != 'SUCCESS') {
                    return
                }

                stage("🍼 Update Formula") {
                    // https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/#writefile-write-file-to-workspace
                    writeFile file: fileName, text: fileContents
                    archiveArtifacts fileName
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
