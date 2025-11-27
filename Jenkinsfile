#!groovy

isMain = (env.BRANCH_NAME == "main") ? true : false
isDevelop = (env.BRANCH_NAME == "develop") ? true : false
isFeature = (env.BRANCH_NAME ==~ "feature/.*") ? true : false
isRelease = (env.BRANCH_NAME ==~ "release/.*") ? true : false
isPR = (env.BRANCH_NAME ==~ "PR/.*") ? true : false
//SonarScanEnabled = isMain || isDevelop || isRelease || isPR 
//PublishNuGet = isMain || isDevelop || isRelease
configuration = "Release"
platform = addQuotes("Any CPU")
buildSwitches = " /p:Configuration=${configuration} /p:Platform=${platform} "

echo "BRANCH_NAME = ${env.BRANCH_NAME}"
echo "Configuration = ${configuration}  Platform = ${platform}"
echo "Is Main: ${isMain}"
echo "Is Develop: ${isDevelop}"
echo "Is Feature: ${isFeature}"
echo "Is Release: ${isRelease}"
//echo "Is SonarScan enabled: ${SonarScanEnabled}"
//echo "Is PublishNuget: ${PublishNuGet}"


pipeline {
    	agent { label 'SMIBOnDemand' }
	
	     // *** Change the version below for each release. ****
		environment {
        	MAJOR					= '9'
        	MINOR 					= '8'
			DOTREL					= '0'
			SPHF					= '00'
			hotfix					= '00'
			// RELEASE_BUILD_NUMBER will be zero (0) for all branches except for release/* 
			//  where it will be the final build number for that release.
			//  Change this variable only in the release/* branch.
			RELEASE_BUILD_NUMBER 	= '0'
			NUGET_PACKAGES	="${env.WORKSPACE}\\NugetCache"
			msBuildTools = addQuotes("${tool 'MSBuild_16_Enterprise'}")
			PythonPath = "C:\\Program Files (x86)\\Microsoft Visual Studio\\Shared\\Python37_64"
			SHEXE = "C:\\Program Files\\Git\\bin\\sh.exe"
			ABS_FULL_VERSION = ""
    }
	
    stages {
			stage('Init')
				{
					steps {
						bat "echo ComputerName = %COMPUTERNAME%, Current Directory = %cd%"
						bat "ipconfig /all"
						bat "whoami"
                        bat "set"
						echo "ABS_BRANCH: ${env.BRANCH_NAME}"
						echo "BUILD_NUMBER: ${env.BUILD_NUMBER}"
						echo "GIT_BRANCH: \"${env.BRANCH_NAME}\""
						echo "GIT_REVISION_SHA1: \"${env.GIT_COMMIT}\""						
					}
				}
	
		stage('Clean') {
			steps {
				bat "echo ComputerName = %COMPUTERNAME%, Current Directory = %cd%"
                		bat "ipconfig /all"
                		bat "whoami"
				bat "git reset --hard"
				bat "attrib *.* -r -h /s"
				bat "rmdir submodules /s /q"
                		bat "git clean -ffxd"
                		bat "git submodule foreach --recursive git clean -ffxd"
			}
        }
		
		stage ('Initial Checks') {
			steps {
				echo "Hello"
				bat "git status"
				bat "git config --list"
				echo "Printing env variables..."
                		bat "set"
				bat "git config --system core.longpaths true"
				
				echo "WORKSPACE=${env.WORKSPACE}"
				echo "Is main branch: ${isMain}"
				echo "Is develop branch: ${isDevelop}"
				echo "Is feature branch: ${isFeature}"
				echo "buildSwitches: ${buildSwitches}"
				
				bat "echo Starting build at %DATE% %TIME%"
			}
		}
		
		stage("Submodules") {
            		steps {
			        bat "git submodule foreach --recursive git clean -ffxd"
                		bat "git submodule init"
                		bat "git submodule update"
            		}
       		}
			
		stage('Create Change List File') {
			steps {
				script {
						echo "Starting Create Change List File stage..."
						def lastReleaseFile = readFile "last_release.txt"
						def last_hash = lastReleaseFile.readLines()[0].trim()
						def last_version = lastReleaseFile.readLines()[1].trim()

						bat "\"%SHEXE%\" %WORKSPACE%\\Submodules\\commonbuildtools\\changeLists\\makeChangeList.sh ${last_hash}"
						bat "ren change_list.txt change_list_since_${last_version}.txt"
						bat "\"%SHEXE%\" %WORKSPACE%\\Submodules\\commonbuildtools\\changeLists\\getJiraCards.sh ${last_hash} > jiras_since_${last_version}.txt"
						archiveArtifacts artifacts: "change_list_since_${last_version}.txt", excludes: ''
						archiveArtifacts artifacts: "jiras_since_${last_version}.txt", excludes: ''

				
						echo "Create Change List File stage done"
				}
			}    
		}
		
		stage ('NuGet restore') {
			steps {
				echo "Nuget clear all"
				bat "nuget locals all -clear"

				echo "Nuget restore"
				bat "nuget restore %WORKSPACE%\\EGS\\ABS\\ABS.sln"
				
			}
		}
		
		stage ("Set Build Version Info") {
            steps {
                script {
					echo "Generating version.inc..."
					bat "%WORKSPACE%\\BatchFiles\\GenVersion.bat ${env.MAJOR} ${env.MINOR} ${env.DOTREL} ${env.SPHF} ${env.hotfix} ${env.BUILD_NUMBER} \"%WORKSPACE%\\tools\\version.inc\" \"%WORKSPACE%\\submodules\\absshare\\EGS\\shared\\inc\"" 
					
                    bat "%WORKSPACE%\\BatchFiles\\GenerateVersionFiles.bat \"%WORKSPACE%\\submodules\\absshare\\EGS\\shared\\inc\" ${env.MAJOR} ${env.MINOR} ${env.DOTREL} ${env.SPHF} ${env.hotfix} ${env.BUILD_NUMBER} ${env.BRANCH_NAME} ${env.GIT_COMMIT}"
					
					def filePath = readFile "versionNumberOnly.txt"
                    def lines = filePath.readLines()
                    ABS_FULL_VERSION = lines[0].trim()
					
					echo "Version create stage done."
                }
            }
        }
		
		stage ('Pre build') {
			steps {
				
				echo "Pre build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\EGS\\EGSTFSPreBuild.sln\" ${buildSwitches} "
			}
		}
		
		stage ('Build ABSSVC') {
			steps {
				
				echo "ABSSVC build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\EGS\\ABS\\ABS.sln\" ${buildSwitches} "
			}
		}
		
		stage ('Build FloorNet') {
			steps {
				
				echo "FloorNet build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\EGS\\ABS\\FloorNet.sln\" ${buildSwitches} "
			}
		}
		
		stage ('Build FloorNet ConfigSvcDB') {
			steps {
				
				echo "FloorNet ConfigSvcDB build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\EGS\\ABS\\FloorNet_ConfigSvcDB.sln\" ${buildSwitches} "
			}
		}
		
		stage ('Build Version Automation') {
			steps {
				
				echo "Version Automation build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\submodules\\sharelib\\tools\\VersionAutomation\\VersionAutomation.sln\" ${buildSwitches} "
			}
		}

		stage ('Build Final Solution') {
			steps {
				
				echo "Final Solution build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\EGS\\ABS\\ABS_FinalSolution.sln\" ${buildSwitches} "
			}
		}

		stage ('Post build') {
			steps{
				echo "Post build..."
				bat "\"${tool 'MSBuild_16_Enterprise'}\" \"%WORKSPACE%\\EGS\\EGSTFSPostBuild.sln\" ${buildSwitches} /p:PdbFolderArchive=ABS"
			}
		}
		
			
		stage('Create Output') {
            steps {
                echo "CreateOutput step..."
				zip archive: true, dir: '', glob: 'EGS/EGSOUT/**', zipFile: "abs_${env.MAJOR}.${env.MINOR}.${env.DOTREL}.${env.SPHF}${env.hotFix}.${env.BUILD_NUMBER}.zip"
				bat "BatchFiles\\CreateOutput.bat \"%WORKSPACE%\\Output\""
                echo "archiveArtifacts zip and txt..."
				dir ('Output')
				{
					archiveArtifacts artifacts: '*.zip, *.txt', excludes: ''
				}

				echo "GenerateBuildDownloadFile step..."
				bat "BatchFiles\\GenerateBuildDownloadFile.bat \"%WORKSPACE%\\tools\\get-release.ps1\" \"${env.BUILD_URL.replace('%','%%')}\" \"${ABS_FULL_VERSION}\" "
				archiveArtifacts artifacts: "get-release_${ABS_FULL_VERSION}.ps1"
            }
        }

		
			
		
			
	}	 

	options { 
		// Keep 5 copies of the build
		buildDiscarder(logRotator(numToKeepStr: '5')) 
		
		// Timeout builds if they do not finish in 60mins
        timeout(time: 360, unit: 'MINUTES')
        
		// Show timestamps on the console
        timestamps()
		
		// Run one build at a time
		disableConcurrentBuilds()
		}
	}	



void GenVersionInc() {
		def buildNumber = (env.BRANCH_NAME == "main") ? env.BUILD_NUMBER : (env.BRANCH_NAME.startsWith("release/") ? env.RELEASE_BUILD_NUMBER : 0)
		bat "IF exist ${env.WORKSPACE}\\EGS\\shared\\nul ( echo ${env.WORKSPACE}\\EGS\\shared exists ) ELSE ( mkdir ${env.WORKSPACE}\\EGS\\shared)"
		bat "IF exist ${env.WORKSPACE}\\EGS\\shared\\inc\\nul ( echo ${env.WORKSPACE}\\EGS\\inc exists ) ELSE ( mkdir ${env.WORKSPACE}\\EGS\\shared\\inc)"
		echo "MAJOR ${env.MAJOR}" 
		echo "MINOR ${env.MINOR}" 
		echo "DOTREL ${env.DOTREL}" 
		echo "SPHF ${env.SPHF}" 
		echo "hotfix ${env.hotfix}" 
		echo "build number ${buildNumber}"
		echo "workspace env ${env.WORKSPACE}"

		def cmd = "${env.WORKSPACE}\\EGS\\batfiles\\GenVersion.bat ${env.MAJOR} ${env.MINOR} ${env.DOTREL} ${env.SPHF} ${env.hotfix} ${buildNumber} ${env.WORKSPACE}\\EGS\\shared\\inc\\version.inc ${env.WORKSPACE}\\Submodules"
		echo "${cmd}"
		bat "${cmd}"
}


String addQuotes(String text) {
	return String.format("\"%s\"", text);
}
