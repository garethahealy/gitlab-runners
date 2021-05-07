#!/usr/bin/groovy

pipeline {
    agent {
        label 'maven'
    }

    options {
        skipDefaultCheckout()
    }

    stages {
        stage('build ubi7-gitlab-runner') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def models = openshift.process("https://raw.githubusercontent.com/redhat-cop/containers-quickstarts/master/ubi7-gitlab-runner/.openshift/bc-ubi7-gitlab-runner.yml", "-p", "CONTEXT_DIR=ubi7-gitlab-runner")
                            def created = openshift.apply(models)

                            def bc = created.narrow('bc')
                            def build = bc.startBuild()
                            build.logs('-f')
                        }
                    }
                }
            }
        }

        stage('deploy ubi7-gitlab-runner') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def imageStreamSelector = openshift.selector("is/ubi7-gitlab-runner")
                            def imageUri = imageStreamSelector.object().status.dockerImageRepository

                            def models = openshift.process("https://raw.githubusercontent.com/garethahealy/gitlab-runners/master/template.yaml", "-p", "PROJECT_NAME=${openshift.project()}", "-p", imageUri)
                            def created = openshift.apply(models)

                            def resource = openshift.selector("deployment/ukiconsulting-gitlab-runner")
                            def rolloutManager = resource.rollout()
                            rolloutManager.status("--watch=true")
                        }
                    }
                }
            }
        }
    }
}