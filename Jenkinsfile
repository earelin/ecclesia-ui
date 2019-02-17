#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        nodejs '10.15.1'
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Static code analysis') {
            steps {
                sh '''
                    yarn lint:ci
                    yarn cpd:ci
                '''
            }
            post {
                always {
                    recordIssues aggregatingResults: true, sourceCodeEncoding: 'UTF-8', tools: [
                        checkstyle(pattern: 'reports/checkstyle.xml'),
                        cpd(pattern: 'reports/cpd.xml'),
                    ]
                }
            }
        }

        stage('Testing') {
            steps {
                sh '''
                    yarn test
                '''
            }
            post {
                always {
                    junit 'reports/junit.xml'                    
                }
                success {
                    step([
                        $class: "CloverPublisher",
                        cloverReportDir: "coverage",
                        cloverReportFileName: "clover.xml"
                    ])
                }
            }
        }

        stage('Comment pull request') {
            when { changeRequest() }
            environment {
                REPOSITORY_NAME = "${env.GIT_URL.tokenize('/')[3].split('\\.')[0]}"
                REPOSITORY_OWNER = "${env.GIT_URL.tokenize('/')[2]}"
            }
            steps {
                ViolationsToGitHub([
                    gitHubUrl: env.GIT_URL,
                    repositoryName: env.REPOSITORY_NAME,
                    repositoryOwner: env.REPOSITORY_OWNER,
                    pullRequestId: env.CHANGE_ID,

                    createCommentWithAllSingleFileComments: true,
                    createSingleFileComments: false,
                    commentOnlyChangedContent: true,
                    minSeverity: 'INFO',
                    keepOldComments: false,

                    violationConfigs: [
                        [parser: 'CHECKSTYLE', reporter: 'ESLint', pattern: 'reports/checkstyle.xml'],
                        [parser: 'CPD', reporter: 'CPD', pattern: 'reports/cpd.xml']
                   ]])
            }
        }

        stage('Build') {
            steps {
                sh 'yarn build'
            }
        }

//        stage('Publish snapshot') {
//            when { branch "2.x.x" }
//            steps {
//
//            }
//        }
//
//        stage('Publish release') {
//            when { buildingTag() }
//            steps {
//
//            }
//        }
    }
}
