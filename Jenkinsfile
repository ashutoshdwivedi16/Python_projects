pipeline {
  agent any
  
  
  stages {
    stage('Install Python Dependencies') {
      steps {
        echo "Create ${env.ArtifactsFolder} Folder"
        
        if(fileExists('./ArtifactsFolder')){
        sh "mkdir ArtifactsFolder"
        }
        
        // Only the virtual environment needs to be installed at the system level
        echo "Install Python Virtual environments"
        sh 'pip install -q -I virtualenv --user'
        // Install the rest of the dependencies at the environment level and not the system level
        withPythonEnv('python') {
          echo "Install Python requirements"
          sh 'pip install -U outsystems-pipeline'
        }
      }
    }

    stage('Run Regression') {
      steps {
        withPythonEnv('python') {
          echo 'Generating URLs for BDD testing...'
          // Generate the URL endpoints of the BDD tests
          sh "python -m outsystems.pipeline.generate_unit_testing_assembly --artifacts \"${env.ArtifactsFolder}\" --app_list \"${params.ApplicationScopeWithTests}\" --cicd_probe_env ${env.ProbeEnvironmentURL} --bdd_framework_env ${env.BddEnvironmentURL}"
          echo "Testing the URLs and generating the JUnit results XML..."
          // Run those tests and generate a JUNIT test report
          sh(script: "python -m outsystems.pipeline.evaluate_test_results --artifacts \"${env.ArtifactsFolder}\"", returnStatus: true)
        }
      }
      post {
        always {
          withPythonEnv('python') {
            echo "Publishing JUnit test results..."
            junit(testResults: "${env.ArtifactsFolder}\\junit-result.xml", allowEmptyResults: true)
          }
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
      }
    }
  
    stage('Deploy Production') {
      steps {
        // Wrap the confirm option in a timeout to avoid hanging Jenkins forever
        timeout(time:1, unit:'DAYS') {
          input 'Accept changes and deploy to Production?'
        withPythonEnv('python') {
           echo 'Deploying latest application tags to Production...'
           // Deploy the application list, without tests, to the Production environment
          sh "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.PreProductionEnvironment}\" --destination_env \"${env.ProductionEnvironment}\" --app_list \"${params.ApplicationScope}\""
        }
        }      }
      post {
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
      }
    }
  }
  post {
    always { 
      echo 'Deleting artifacts folder content...'
      dir ("${env.ArtifactsFolder}") {
        deleteDir()
      }
    }
  }
}
