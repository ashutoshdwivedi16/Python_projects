pipeline {
  agent any
  parameters {
    // App List Parameters -> automatically filled by LT Trigger plugin
   /* string(name: 'ApplicationScope', defaultValue: '', description: 'Comma-separated list of LifeTime applications to deploy.')
    string(name: 'ApplicationScopeWithTests', defaultValue: '', description: 'Comma-separated list of LifeTime applications to deploy (including test applications)')
    string(name: 'TriggeredBy', defaultValue: 'N/A', description: 'Name of LifeTime user that triggered the pipeline remotely.')
  */
  }
  options { 
      //skipStagesAfterUnstable() 
      ansiColor('xterm')
      skipDefaultCheckout()

      }

          

  environment {
    // Artifacts Folder
    ArtifactsFolder = "Artifacts"
    // LifeTime Specific Variables
    // LifeTimeHostname = 'https://pmdemo2-lt.outsystemsenterprise.com/'
    // LifeTimeAPIVersion = 2
    // // Authentication Specific Variables
    // AuthorizationToken = credentials('LifeTimeServiceAccountToken')
    // // Environments Specification Variables
    //   //Environment -> Where your apps are live.
    // DevelopmentEnvironment = 'Development'
    // RegressionEnvironment = 'Beta Testing'
    // ProductionEnvironment = 'Production'
    // // Regression URL Specification
    // ProbeEnvironmentURL = 'https://pmdemo2-tst.outsystemsenterprise.com/'
    // BddEnvironmentURL = 'https://pmdemo2-tst.outsystemsenterprise.com/'
  }
  stages {

      stage('Initializiing ......') {
      steps {
      dir ("${env.ArtifactsFolder}") {
        deleteDir()
      }

      }
    }
      stage('Initializiing ......') {
            steps {
              checkout scm
            }
          }

    stage('Install Python Dependencies') {
      steps {
        echo "Create ${env.ArtifactsFolder} Folder"
        sh "mkdir ${env.ArtifactsFolder}"
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
    stage('Get and Deploy Latest Tags') {
      steps {
        withPythonEnv('python') {
          echo "Pipeline run triggered remotely by '${params.TriggeredBy}' for the following applications (including tests): '${params.ApplicationScopeWithTests}'"
          echo 'Retrieving latest application tags from Development environment...'
          // Retrive the Applications and Environment details from the Source environment
 //         sh "python -m outsystems.pipeline.fetch_lifetime_data --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion}"
          echo 'Deploying latest application tags to Regression...'
          // Deploy the application list, with tests, to the Regression environment
 //         sh "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.DevelopmentEnvironment}\" --destination_env \"${env.RegressionEnvironment}\" --app_list \"${params.ApplicationScopeWithTests}\""
        }
      }
      post {
        // It will always store the cache files generated, for observability purposes
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*.cache", onlyIfSuccessful: true
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
        // If there's a failure, tries to store the Deployment conflicts (if exists), for observability and troubleshooting purposes
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
      }
    }
    stage('Run Regression') {
      steps {
        withPythonEnv('python') {
          echo 'Generating URLs for BDD testing...'
          // Generate the URL endpoints of the BDD tests
  //        sh "python -m outsystems.pipeline.generate_unit_testing_assembly --artifacts \"${env.ArtifactsFolder}\" --app_list \"${params.ApplicationScopeWithTests}\" --cicd_probe_env ${env.ProbeEnvironmentURL} --bdd_framework_env ${env.BddEnvironmentURL}"
          echo "Testing the URLs and generating the JUnit results XML..."
          // Run those tests and generate a JUNIT test report
   //       sh(script: "python -m outsystems.pipeline.evaluate_test_results --artifacts \"${env.ArtifactsFolder}\"", returnStatus: true)
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
    stage('Upload GitHub') {
      steps {
        withPythonEnv('python') {
          echo 'Calling TriggerPipeline API to request upload of OAPs if pipeline is configured to do so.'
          // Call TriggerPipeline API
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
