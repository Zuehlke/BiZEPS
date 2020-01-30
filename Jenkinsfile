#!/usr/bin/env groovy

// Uses the common library
library identifier: 'common-pipeline-library@stable', changelog: false,
  retriever: modernSCM([$class: 'GitSCMSource',
    remote: 'git@gitlab.com:ponderbear/pipelinelibrary.git',
    credentialsId: 'jenkins-snowflurry',
    traits: [gitBranchDiscovery(), gitTagDiscovery(),
      [$class: 'CloneOptionTrait', extension: [depth: 1, noTags: false, reference: '', shallow: true]]]
  ])

node {
  def projectSettings = readJSON text: '''{
    "repository": {
      "url": "https://github.com/Zuehlke/BiZEPS.git",
      "credentials": "3bc30eda-c17e-4444-a55b-d81ee0d68981"
    },
    "dockerHub": {
      "user": "bizeps"
    },
    "dockerJobs": [
      {"imageName": "jenkins",        "dockerfilePath": "./buildServer/jenkins/master" },
      {"imageName": "awscli",         "dockerfilePath": "./utils/awsCli"},
      {"imageName": "certgenerator",  "dockerfilePath": "./utils/certGenerator"},
      {"imageName": "gcc",            "dockerfilePath": "./buildTools/gcc"},
      {"imageName": "mcmatools",      "dockerfilePath": "./buildTools/mcmatools"},
    ]
  }'''

  def triggers = jobProperties.getJobBuildTriggers{}

  properties([
    pipelineTriggers(triggers),
    buildDiscarder(logRotator(
      artifactDaysToKeepStr: '5', artifactNumToKeepStr: '5',
      numToKeepStr: '5', daysToKeepStr: '5'))
  ])

  def branchNameParameter = "*/${buildUtils.getCurrentBuildBranch()}"

  repositoryUtils.checkoutBranch {
    stageName = 'Checkout'
    branchName = branchNameParameter
    repoUrl = "${projectSettings.repository.url}"
    repoCredentials = "${projectSettings.repository.credentials}"
  }

  docker.withServer(env.DEFAULT_DOCKER_HOST_CONNECTION, 'default-docker-host-credentials') {
    try {
      stage("Build") {
        def buildTasks = dockerImage.setupBuildTasks {
          dockerRegistryUser = "${projectSettings.dockerHub.user}"
          buildJobs = projectSettings.dockerJobs
        }
        for (task in buildTasks.values()) {
          task.call()
        }
      }

      docker.withRegistry(env.DEFAULT_DOCKER_REGISTRY_CONNECTION, 'default-docker-registry-credentials') {
        stage("Push") {
          parallel dockerImage.setupPushTasks {
            dockerRegistryUser = "${projectSettings.dockerHub.user}"
            buildJobs = projectSettings.dockerJobs
          }
        }
      }
    }
    finally {
      stage("Clean up") {
        def cleanupTasks = dockerImage.setupClenupAllUnusedTask {
          dockerRegistryUser = "${projectSettings.dockerHub.user}"
          buildJobs = projectSettings.dockerJobs
        }
        for (task in cleanupTasks.values()) {
          task.call()
        }
        // clean up workspace
        deleteDir()
      }
    }
  }
}
