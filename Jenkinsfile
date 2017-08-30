pipeline {

  // agent defines where the pipeline will run.
  agent {
    // This also could have been 'agent any' - that has the same meaning.
    label ""
    // Other possible built-in agent types are 'agent none', for not running the
    // top-level on any agent (which results in you needing to specify agents on
    // each stage and do explicit checkouts of scm in those stages), 'docker',
    // and 'dockerfile'.
  }

   parameters {
    booleanParam(defaultValue: true, description: '', name: 'flag')
    string(defaultValue: '', description: '', name: 'SOME_STRING')
  }
  
  // The tools directive allows you to automatically install tools configured in
  // Jenkins - note that it doesn't work inside Docker containers currently.
  
  environment {
    // Environment variable identifiers need to be both valid bash variable
    // identifiers and valid Groovy variable identifiers. If you use an invalid
    // identifier, you'll get an error at validation time.
    // Right now, you can't do more complicated Groovy expressions or nesting of
    // other env vars in environment variable values, but that will be possible
    // when https://issues.jenkins-ci.org/browse/JENKINS-41748 is merged and
    // released.
    AZURE_PRINCIPLE = 'AZURE_PRINCIPLE_DEV'
  }
  
  stages {
    stage('Checkout') {
        steps{
            echo "Checking out from visualstudio"
            git 'ssh://apcsmartconnect.visualstudio.com:22/_git/APCSmartConnect'
        }
    }
    // At least one stage is required.
    stage("first stage") {
      // Every stage must have a steps block containing at least one step.
      steps {
        // You can use steps that take another block of steps as an argument,
        // like this.
        //
        // But wait! Another validation issue! Two, actually! I didn't use the
        // right type for "time" and had a typo in "unit".
        timeout(time: 40, unit: 'MINUTES') {
          echo "We're not doing anything particularly special here."
          echo "Just making sure that we don't take longer than five minutes"
          
          // This'll output 3.3.3, since that's the Maven version we
          // configured above. Well, once we fix the validation error!
          bat "node -v" 
        }
      }
      
      // Post can be used both on individual stages and for the entire build.
      post {
        success {
          echo "Only when we haven't failed running the first stage"
        }
        
        failure {
          echo "Only when we fail running the first stage."
        }
      }
    }
    
    stage('second stage') {
      // You can override tools, environment and agent on each stage if you want.
      
      steps {
        echo "This time, the node version should be 3.3.9"
        bat "node -v"
      }
    }
    
    stage('third stage') {
      steps {
        // Note that parallel can only be used as the only step for a stage.
        // Also, if you want to have your parallel branches run on different
        // nodes, you'll need control that manually with "node('some-label') {"
        // blocks inside the parallel branches, and per-stage post won't be able
        // to see anything from the parallel workspaces.
        // This'll be improved by https://issues.jenkins-ci.org/browse/JENKINS-41334, 
        // which adds Declarative-specific syntax for parallel stage execution.
        parallel(one: {
                  echo "I'm on the first branch!"
                 },
                 two: {
                   echo "I'm on the second branch!"
                 },
                 three: {
                   echo "I'm on the third branch!"
                   echo "But you probably guessed that already."
                 })
      }
    }
  }
  
  post {
    // Always runs. And it runs before any of the other post conditions.
    always {
      // Let's wipe out the workspace before we finish!
      deleteDir()
    }
    
   
  }
  
  // The options directive is for configuration that applies to the whole job.
  options {
    // For example, we'd like to make sure we only keep 10 builds at a time, so
    // we don't fill up our storage!
    buildDiscarder(logRotator(numToKeepStr:'10'))
    
    // And we'd really like to be sure that this build doesn't hang forever, so
    // let's time it out after an hour.
    timeout(time: 60, unit: 'MINUTES')
  }

}
