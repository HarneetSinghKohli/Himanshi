#!groovy

/*
 ------------------------------------------------------------------- 
              Jenkinsfile (Declarative Pipeline) 
 ------------------------------------------------------------------- 

Info
- Pipeline syntax: https://jenkins.io/doc/book/pipeline/syntax/
- Pipeline Steps Reference: https://jenkins.io/doc/pipeline/steps/
- Syntax Reference examples: https://github.com/jenkinsci/pipeline-model-definition-plugin/wiki/Syntax-Reference

*/
pipeline {
  agent {
    label 'lbrusdkrengine05'
  }
  options {
    buildDiscarder(logRotator(numToKeepStr:'10'))
    // skipDefaultCheckout() 
    timeout(time: 5, unit: 'MINUTES')
    gitLabConnection('DSI')
    gitlabBuilds(builds: ['Deploy Developments', 'Deploy Production'])
  }
  triggers {
    gitlab(triggerOnPush: true, 
           triggerOnMergeRequest: true,
           triggerOpenMergeRequestOnPush: 'both',
           triggerOnNoteRequest: false,
           noteRegex: "REBUILD!",
           skipWorkInProgressMergeRequest: true,
           setBuildDescription: true,
           addNoteOnMergeRequest: true,
           acceptMergeRequestOnSuccess: false,
           branchFilterType: 'All',
           targetBranchRegex: "",
           includeBranchesSpec: "",
           excludeBranchesSpec: "")
  }
  stages {

    stage('Deploy Developments') {
      when {
        branch 'developments'
      }
      steps {
        updateGitlabCommitStatus(name: 'Deploy Developments', state: 'running')
        sh "cd /dkrsbcbs/CENV/dashboard-dev/cenv-dashboard && git fetch origin developments && git reset --hard origin/developments && git clean -f && chmod -R 754 ."
      }
      post {
        always {
          updateGitlabCommitStatus(name: 'Deploy Production', state: 'success')
        }
        success {
          updateGitlabCommitStatus(name: 'Deploy Developments', state: 'success')
          emailext (
            subject: "[CDK-Monitoring] SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: """<p>Hi,</p><p>Your modifications for the Monitoring Scripts deployed for testing.<br>
                  You are able to test your modifications by modifying the associated jenkins job.</p>
                  <p><a href='http://dlnxsdlc01:8183'>Jenkins</a></p><p>Thank you for your contribution,<br>The SDLC Team</p>""",
            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
          )
        }
        failure {
          updateGitlabCommitStatus(name: 'Deploy Developments', state: 'failed')
        }
      }
    }

    stage('Deploy Production') {
      when {
        branch 'master'
      }
      steps {
        updateGitlabCommitStatus(name: 'Deploy Production', state: 'running')
        sh "cd /dkrsbcbs/CENV/dashboard-prod/cenv-dashboard && git fetch origin master && git reset --hard origin/master && git clean -f && chmod -R 754 ."
      }
      post {
        always {
          updateGitlabCommitStatus(name: 'Deploy Developments', state: 'success')
        }
        success {
          updateGitlabCommitStatus(name: 'Deploy Production', state: 'success')
          emailext (
            subject: "[CDK-Monitoring] SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: """<p>Hi,</p><p>Your modifications for the Monitoring Scripts are deployed in production.<br>
                  You are able to revert your test modification for the associated jenkins job.</p>
                  <p><a href='http://dlnxsdlc01:8183'>Jenkins</a></p><p>Thank you for your contribution,<br>The SDLC Team</p>""",
            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
          )
        }
        failure {
          updateGitlabCommitStatus(name: 'Deploy Production', state: 'failed')
        }
      }
    }

  }
  post {
    // always {
    //   deleteDir()
    // }
    failure {
      emailext (
        subject: "[CDK-Monitoring] FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
          <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
      )
    }
  }
}

