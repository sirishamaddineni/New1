pipeline {
    agent any

    stages 
	{
		stage( 'Checkout Source' ) 
		{
		//Checkout source code from GIT
		  steps
		  {
			// workaround to get GIT plugin environment Variables, we need to collect the checkout command output, which is a map that contains them
			// https://issues.jenkins-ci.org/browse/JENKINS-35230
			  script
			  {
				scm_map = checkout scm
				GIT_BRANCH = scm_map['GIT_BRANCH']
				// get just the branch name minus the remote only splitting on first
				// match in case the rest of the branch has more '/' chars.
				GIT_BRANCH_NAME = GIT_BRANCH.split('/',2)[1]				
			  }				
				 bat "echo 'Checkout of GIT branch: ${GIT_BRANCH}'"
			     bat "echo 'GIT_BRANCH_NAME: ${GIT_BRANCH_NAME}'"
			}
		}//End Checkout Source   
		
		stage( 'Build' ) 
		{
		//Build source code
		  steps
		  {
			bat "\"${tool 'MSBuild 14'}\" NunitDemo.sln /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"   
			}
		}//End Build source code 
		stage( 'Test' ) 
		{
		//Build source code
		  steps
		  {
			bat '''"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\OpenCover.4.6.519\\tools\\OpenCover.Console.exe" -target:"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\NUnit.ConsoleRunner.3.7.0\\tools\\nunit3-console.exe" -targetargs:"/work:Reporting --out:TestResult.txt .\\NunitDemo.Test.dll"  -output:"CodeCoverageResult.xml"
				"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\ReportGenerator.3.1.0\\tools\\ReportGenerator.exe" "-reports:CodeCoverageResult.xml" "-targetdir:CodeCoverageReport"
				"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\ReportUnit.1.2.1\\tools\\ReportUnit.exe" "Reporting" "Reporting\\Result"'''
			}
		}//End Build source code 	
		
		stage( 'Package into zip file' ) 
		{
		//Build source code
		  steps
		  {
			bat '"C:\\Program Files\\7-Zip\\7z.exe" a  -r DemoNunit.zip -w NunitDemo.Test\\bin\\Release\\* -mem=AES256'
			}
		}//End Build source code 	
		stage( "IQ Scans") {
		  steps{
			bat "echo 'Uploading to IQ: 'DemoNunit.zip' stage: 'release' file: 'DemoNuint.zip'"
			nexusPolicyEvaluation failBuildOnNetworkError: false,
				iqApplication: 'IQ_app',
				iqScanPatterns: [[scanPattern: 'DemoNunit.zip' ]],
				iqStage: 'release',
				jobCredentialsId: ''
		  }
		}
	        stage("Upload to Repo"){
 	           steps{
 	                  nexusArtifactUploader artifacts: [[artifactId: 'DemoNunit', classifier: '', file: 'DemoNunit.zip', type: 'zip']],
 	                  credentialsId: 'M12345', 
 	                  groupId: 'Repo-group', 
 	                  nexusUrl: 'localhost:9091', 
 	                  nexusVersion: 'nexus3', 
 	                  protocol: 'http', 
 	                  repository: 'Repo', 
 	                  version: '1.0'
 	
 	          		  }
		} // stage	
		stage ( "Tagging" ){                	  
 			steps {
                         bat "git tag 'v19.0'"
                	 bat "git config user.email 'sirishamaddineni25@gmail.com'"
                         bat "git config user.name 'sirishamaddineni'"	
			  
			        					
		}
    		} // stage
	}
}
