pipeline {
	
	agent { label JenkinsAgent }

	options {
		checkoutToSubdirectory("${AppName}") // Added AppName for checkout dir of implizit checkout
		disableConcurrentBuilds() 			 // throttle builds
	}
	environment {
		GIT_TRACE       = 'false'               // Trace git for testing  true/false,1,2.
		GIT_TRACE_SETUP = 'false'               // really cool trace tools.
	}
	
	stages {
		
		stage ('Pipeline Setup') {
			steps {
				script {

					// Required Input Parameters
					properties([
						parameters([
							choice(
								choices : ["build", "release", "preview"],
								description: 'Please select the pipeline type',
								defaultValue: 'build',
								name: 'PipelineType'
							),
							choice(
								choices : ["major", "minor", "patch"],
								description: 'Please provide the release type for automated computing of tags',
								defaultValue: 'patch',
								name: 'ReleaseType'
							),
							choice(
								choices : ["no", "yes"],
								description: 'Enable pipeline logging',
								defaultValue: 'no',
								name: 'PipelineVerboseLogging'
							)
						])
					])

					println("${PipelineName}[INFO]: Start Date/Time              = " + StartDateTime)
			
					// Assess user provided parameters
					if (params.PipelineVerboseLogging.equals("yes")){
						zAppBuildVerbose = "-v"
						pipeverbose = true
					} else {
						zAppBuildVerbose = ""
						pipeverbose = false
					}

					PipelineType = params.PipelineType
					ReleaseType = params.ReleaseType

					Buildrc = 0

					if (pipeverbose) {
						println("${PipelineName}[DEBUG]:   Format yyyyMMdd.hhmmss.mmm = " + StartDateTime.format("yyyyMMdd.hhmmss.mmm"))
						println("${PipelineName}[DEBUG]:   Format yyyyMMdd.HHmmss.mmm = " + StartDateTime.format("yyyyMMdd.HHmmss.mmm"))
						sh "env"
					}
					
					// Pick up the AppBranch from Jenkins.
					AppBranch = "${env.BRANCH_NAME}"
					
					// Fetch the Pipeline Name and Build Number.  Set up and format the DBB Working
					// directory for the build results.
					PipelineName   = env.JOB_NAME.split("/")
					PipelineName   = PipelineName[0]

					BuildNumber    = "${env.BUILD_NUMBER}"
					BuildNumber    = BuildNumber.padLeft(8,"0") // Make Build Directories easier to sort.
					
					//Build directory
					BuildDir       = "${WORKSPACE}"
					baselineReferenceFile = "${BuildDir}/${AppName}/application-conf/baselineReference.config"


					// Branch Control Setup
					Branch         = env.BRANCH_NAME.toUpperCase()
					BranchOkay     = false
					PipelineTypeUpper = PipelineType.toUpperCase()
					
					// Configure the stages for the main branch
					if  ( (Branch.startsWith('MAIN') == true) ) {

						dbbDoBuild      = true  // Perform DBB Build.
						sonarQubeDoScan = true  // Perform Sonar scan.
						ucdDoPackaging  = true  // Perform the Packaging.
						ucdDoDeployment = true  // Perform the Deployment.

						ucdVersionName = "${ucdVersionName}-Main-${PipelineTypeUpper}" // Package Version Name Prefix.

						BranchOkay = true
					}

					// Configure the stages for feature and hotfix branches
					if  (Branch.startsWith('FEATURE') == true || Branch.startsWith('HOTFIX') == true) {

						dbbDoBuild      = true  // Perform DBB Build.
						sonarQubeDoScan = true  // Perform Sonar scan.
						ucdDoPackaging  = true  // Perform the Packaging. 
						ucdDoDeployment = false // Do Not Perform the Deployment. Up to the developer to install it.

						ucdVersionName = "${ucdVersionName}-${AppBranch}".replaceAll("/",".") // Prelimenary Package Version Name Prefix.

						BranchOkay = true
					}

					// Configure the stages for RELEASE and EPIC branches
					if  (Branch.startsWith('RELEASE') == true || Branch.startsWith('EPIC') == true) {
						
						dbbDoBuild      = true  // Perform DBB Build.
						sonarQubeDoScan = true  // Perform Sonar scan.
						ucdDoPackaging  = true  // Perform the Packaging. 
						ucdDoDeployment = false // Do Not Perform the Deployment. Up to the developer to install it.

						ucdVersionName = "${ucdVersionName}-${AppBranch}-${BuildNumber}".replaceAll("/",".")

						BranchOkay = true
					}

					// Manage preview pipelines
					if  ( (PipelineTypeUpper.startsWith('PREVIEW') == true) ) {
						dbbDoBuild          = true
						// Skip any subsequent steps
						sonarQubeDoScan     = false
						ucdDoPackaging      = false
						ucdDoDeployment     = false
					}
					
					// If Branch Proceesing has not been defined, gracefully terminate the Pipeline.
					if (BranchOkay == false) {
						autoCancelled       = true
						dbbDoBuild          = false
						sonarQubeDoScan     = false
						ucdDoPackaging      = false
						ucdDoDeployment     = false
					}

					println("${PipelineName}[INFO]: AppBranch = ${AppBranch} .")
					
				}
			}
		} // End: stage ('Pipeline Setup')
