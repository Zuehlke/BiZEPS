#!/usr/bin/env groovy

// Uses the common library form 'https://github.com/icebear8/pipelineLibrary'
library identifier: 'common-pipeline-library@stable',
  retriever: modernSCM(github(
    id: '18306726-fec7-4d80-8226-b78a05add4d0',
    credentialsId: '3bc30eda-c17e-4444-a55b-d81ee0d68981',
    repoOwner: 'icebear8',
    repository: 'pipelineLibrary',
    traits: [
      [$class: 'org.jenkinsci.plugins.github_branch_source.BranchDiscoveryTrait', strategyId: 1],
      [$class: 'org.jenkinsci.plugins.github_branch_source.OriginPullRequestDiscoveryTrait', strategyId: 1],
      [$class: 'org.jenkinsci.plugins.github_branch_source.ForkPullRequestDiscoveryTrait', strategyId: 1, trust: [$class: 'TrustContributors']]]))

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
