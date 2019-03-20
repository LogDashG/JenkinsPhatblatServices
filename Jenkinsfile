#!/usr/bin/env groovy
/*
 * Jenkinsfile
 * JenkinsPhatblatServices
 *
 * Updates jenkins formula in phatblat/homebrew-services
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
            defaultValue: null,
            description: 'New version of Jenkins.'
        ),
        string(
            name: 'fileHash',
            defaultValue: null,
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
            node
                stage("✔️ Parameters") {
                    echo "newVersion: $newVersion"
                    echo "fileHash: $fileHash"

                    if (newVersion == null || fileHash == null) {
                        echo "Required parameters are missing"
                        currentBuild.rawBuild.@result = hudson.model.Result.FAILURE
                        return
                    }
                }
                stage("🛒 Checkout") {
                    git url: "git@github.com:phatblat/homebrew-services.git"
                }
                stage("⚖️ Compare Version") {
                    // https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/#readfile-read-file-from-workspace
                    String oldContents = readFile fileName
                    oldContents.eachLine { line, count ->
                        println "line $count: $line"

                        // Version in url
                        if (line.startsWith("  url")) {
                            // Find version
                            // url "http://mirrors.jenkins.io/war/2.167/jenkins.war"
                            if (line.contains(newVersion)) {
                                echo "Version $newVersion already in build, aborting."
                                currentBuild.rawBuild.@result = hudson.model.Result.ABORTED
                                return
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

                    if (status != 0) {
                        echo "Aborting build. 😞"
                        currentBuild.rawBuild.@result = hudson.model.Result.ABORTED
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
