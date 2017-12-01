#!groovy

@Library('github.com/red-panda-ci/jenkins-pipeline-library@2.1.1') _

// Initialize global condig (jpl v2.1.1)
cfg = jplConfig('android-testing', 'android' ,'', [hipchat: '', slack: '', email: 'pedroamador.rodriguez+android-testing@gmail.com'])

pipeline {

    agent none

    stages {
        stage ('Initialize') {
            agent { label 'docker' }
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Build') {
            agent { label 'docker' }
            steps  {
                script {
                    def androidBuildEnv = docker.build('jpl-test')
                    androidBuildEnv.inside {
                        sh 'fastlane develop'
                    }


                }
            }
        }
        stage('SonarQube Analysis') {
            agent { label 'docker' }
            when { expression { (env.BRANCH_NAME == 'develop') || env.BRANCH_NAME.startsWith('PR-') } }
            steps {
                jplSonarScanner(cfg)
            }
        }
        stage ('Release confirm') {
            when { branch 'release/v*' }
            steps {
                jplPromoteBuild(cfg)
            }
        }
        stage ('Release finish') {
            agent { label 'docker' }
            when { branch 'release/v*' }
            steps {
                jplCloseRelease(cfg)
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        skipDefaultCheckout()
        timeout(time: 1, unit: 'DAYS')
    }
}
