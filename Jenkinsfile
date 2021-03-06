properties([pipelineTriggers([githubPush()])])
void setBuildStatus(String message, String state) {
  step([
      $class: "GitHubCommitStatusSetter",
      reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/SophiaHadash/ArgueView"],
      contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
      errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
      statusResultSource: [ $class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]] ]
  ]);
}
def buildBadge = addEmbeddableBadgeConfiguration(id: "build", subject: "ArgueView Build");

pipeline {
  agent {
    dockerfile {
    	dir '.'
    	args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  options {
	skipStagesAfterUnstable()
	buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
  }
  stages {
    stage('test build context') {
      steps {
		sh 'python3 --version && pip3 --version && pipenv --version && docker --version'
      }
    }
    stage('build') {
      steps {
        setBuildStatus("Building...", "PENDING");
        script{ buildBadge.setStatus('running'); }
        sh 'bash build.sh'
      }
    }
	stage('publish') {
	  when {
        branch 'master'
      }
	  steps {
	    script {
	      try {
	        sh 'printf "[pypi]\nusername = __token__\npassword = ${TWINE_PASSWORD}" > ~/.pypirc'
		    sh 'twine upload ./dist/*'
	      } catch(err) {
	        echo err.getMessage()
	      }
	    }
	  }
	}
  }
  post {
    success {
	  setBuildStatus("Build succeeded", "SUCCESS");
	  script{ buildBadge.setStatus('passing'); }
    }
    failure {
      setBuildStatus("Build failed", "FAILURE");
  	  script{ buildBadge.setStatus('failing'); }
    }
  }
}
