# Use Jenkins with GitLab
See resources from
- [Gitlab](gitlab.com)
- [GitLab Jenkins CI](docs.gitlab.com/ee/integration/jenkins.html)
- [GitLab Plugin](https://github.com/jenkinsci/gitlab-plugin)

## Jenkins GitLab API Access
GitLab Jenkins plugin requires access to GitLab to prevent unauthorized build triggers.

### On GitLab
- (Optional: Create a specific user in GitLab for Jenkins)
- Create an API access token
  - `User/Settings/Access Tokens`, e.g. jenkins@test
  - Scope api is suficcient

### On Jenkins
- `Manage Jenkins/Configure System/Gitlab`
  - Create a GitLab connection (e.g. `Default-Gitlab-Connection`)
  - In the credentials, create the GitLab API token with the information from GitLab

## GitLab to Jenkins Access
Jenkins requries access to GitLab to report the build status.
With this setup, we grant per project access.

### On Jenkins
- `Credentials/SSH Username with private key`
  - Create a new Jenkins credential
  - With this key, Jenkins an authenticate to GitLab

### On GitLab
This step has to be repeated for all the projects where Jenkins shall report the build status.
- `Project/Settings/Repository/Deploy Keys`
  - Add the corresponding public key from Jenkins
  - Allow write modifications for this key

## Jenkins Build Trigger
To trigger a build on Jenkins from GitLab
- On GitLab `Project/Settings/Integrations/Add webhook`

### Jenkins Multi-Branch Pipeline
- Jenkins multi-branch pipeline jobs can only be triggered on push notifications
  - See [GitLab plugin](https://github.com/jenkinsci/gitlab-plugin) documentation for details
  - Currently GitLab does not allow to build merge requests with a multi-branch pipeline

### Jenkins Pipeline Project
- Pipeline jobs in Jenkins allow merge request builds
- `Jenkins/Job/Configuration/Build Triggers`
  - Select the allowed build triggers
  - Create a secret token in the `Advanced` menu to allow pipeline built triggers
  - A bug in Pipeline may lead to issues with the build trigger. Add a fake jenkins script in the UI in chenkins configuration and switch back to use jenkinsfile from SCM.

## Simple Jenkins File
```
properties([gitLabConnection('Default-Gitlab-Connection')])

node {
  stage('Checkout') {
    checkout scm
  }

  gitlabCommitStatus {
    stage('Demo'){
      echo 'hello git lab'
    }

  }
}
```

## GitLab Integration Strategy
For gated check-in, automated merge-request builds and create releases by tags,
the following Jenkins pipeline setup could be used.

### Multibranch Pipeline for Development
The multibranch pipeline is used to build the development branches.

- Create a multibranch pipeline which is triggered by GitLab to build all branches in the project
- The multibranch pipeline can use the same jenkinsfile as the other pipeline setups
- The Multibranch pipeline should only build branches (no tags, no merge requests)
- Create a separate hook in GitLab for the Multibranch pipeline
- On Jenkins side, the multibranch pipeline does not need a special configuration

Be aware that without a special setting in the merge pipeline,
the merge-requests will considered as passed as soon as the branch is successfully built in the multibranch pipeline.
To be sure that the merge-request is build correctly, see the chapter about the merge and/or release pipeline.

### Simple Pipeline for Merge and/or Releases
A pipeline for merges and/or releases takes care about merge-request builds
and creates releases when a tag is pushed (if needed).

Jenkins pipeline settings (non multibranch)
- Set the required build triggers for merge-requests
- Consider to add a 'pending build name for pipeline' in the build triggers (e.g. 'MergeAndDeploy')
  - This will notify GitLab about a pending build on Jenkins side
  - A pending merge could be considered as passed by GitLab, because the corresponding branch build passed in the multibranch pipeline job
  - But GitLab should not allow to merge the merge-request, until the merge pipeline built the pre-merged merge-request with the target branch
  - Set the pending build request step to successful at the end of the pipeline in the jenkinsfile
- Create a secret token which is required for simple pipelines (not required for multibranch)
- Set the git connection for the jenkinsfile
  - Hint: A bug in the Jenkins pipeline plug in may require a 'pseudo jenkinsfile script' in the web UI of the pipeline script settings
  - The effect of the bug is that the web hook will only work once from GitLab to Jenkins
- Add the branch specifier for merge-requests and/or tags

Gitlab settings:
- Add webhook with and select the build triggers, add the secret token
  - Build triggers: 'tag push events' and/or 'merge request events'

The following 'branch specifiers' can be used:
- For merge-requests `origin/merge-requests/${gitlabMergeRequestId}`
- For tag builds: `refs/tags/${TAGNAME}`

Make sure to handle the tag requests and the merge-requests properly in the jenkinsfile.
A merge-request should trigger a pre-build-merge to the target branch before building it to get a reliable gated check-in behavior.
