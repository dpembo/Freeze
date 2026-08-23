pipeline {
    agent {
        label "jdk25"
    }

    options {
        githubProjectProperty(projectUrlStr: "https://github.com/SirBlobman/Freeze")
    }

    environment {
        DISCORD_URL = credentials('PUBLIC_DISCORD_WEBHOOK')
        MAVEN_DEPLOY = credentials('MAVEN_DEPLOY')
    }

    triggers {
        githubPush()
    }

    stages {
        stage("Gradle: Build") {
            steps {
                withGradle {
                    script {
                        if (env.BRANCH_NAME == "main") {
                            sh("./gradlew --refresh-dependencies --no-daemon clean build publish")
                        } else {
                            sh("./gradlew --refresh-dependencies --no-daemon clean build")
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'build/libs/Freeze-*.jar', fingerprint: true
        }

        always {
            script {
                def description = """
                    **Branch:** ${env.GIT_BRANCH}
                    **Build:** ${env.BUILD_NUMBER}
                    **Status:** ${currentBuild.currentResult}
                """

                discordSend(
                    webhookURL: DISCORD_URL,
                    title: "Freeze",
                    link: env.BUILD_URL,
                    result: currentBuild.currentResult,
                    description: description.stripIndent(),
                    enableArtifactsList: false,
                    showChangeset: true
                )
            }
        }
    }
}
