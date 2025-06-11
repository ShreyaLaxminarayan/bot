pipeline {
    agent any
    triggers {
        GenericTrigger(
            causeString: 'Triggered by PR comment',
            token: 'jenkins-bot-secret-8392ghfu3',
            printContributedVariables: true,
            printPostContent: true,
            regexpFilterText: '$comment_body',
            regexpFilterExpression: '.*@jenkins-bot.*'
        )
    }

    environment {
        GITHUB_TOKEN = credentials('GITHUB_TOKEN') 
    }

    stages {
        stage('Parse Comment') {
            steps {
                script {
                    def prNumber = env.pull_request_number
                    def comment = env.comment_body

                    echo "PR #${prNumber}: ${comment}"

                    // Add logic to check if a build is already running
                    def builds = Jenkins.instance.getItemByFullName(env.JOB_NAME).getBuilds()
                    def running = builds.find { b -> 
                        b.isBuilding() && b.getEnvironment(null)?.get('PR_NUMBER') == prNumber 
                    }

                    if (running) {
                        echo "Build already running for PR #${prNumber}"

                        // Optional: post comment back to GitHub
                        postComment(prNumber, "⚠️ Build already running.", GITHUB_TOKEN)

                        currentBuild.result = 'NOT_BUILT'
                        error("Stopping since build already running")
                    }
                }
            }
        }

        stage('Trigger Build') {
            steps {
                echo "No running build. Proceeding with build logic..."
                // your actual build logic here (clone repo, run tests, etc)
            }
        }
    }
}

def postComment(prNumber, message, token) {
    def repo = "ShreyaLaxminrayan/runtime" 
    def apiUrl = "https://api.github.com/repos/${repo}/issues/${prNumber}/comments"

    sh """
        curl -s -H "Authorization: token ${token}" \
             -H "Content-Type: application/json" \
             -d '{ "body": "${message}" }' ${apiUrl}
    """
}
