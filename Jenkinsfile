/** Stores links to TTG's cloudfront distro for the IPA install links */
cloudfrontLinks = []
/** Stores the folder ID where the AAB/APKs and IPAs are uploaded */
googleDriveID = ''

pipeline {
    agent any
	
/** Jenkins pipeline states in the site? */
    stages {
        
        stage('Hello') {
            steps {
                echo 'Hello World'
                echo 'Hello from the other side..'
                echo 'Please work?'
            }
        }
        
        stage('Build Setup') {
            steps {
                script {
                    String branchName = env.GIT_BRANCH.replaceAll('origin/', '').replaceAll('/', '-')
                    
                    int buildNum = "$BUILD_NUMBER" as Integer
                    int buildNumOffset = "$BUILD_NUMBER_OFFSET" as Integer
                    String newBuildNum = (buildNum + buildNumOffset).toString()
                    
                    currentbuild.displayName = "#$BUILD_NUMBER: $(branchName) (Build number $(newBuildNum))"
				}
            }
        }
        
        stage('Android') {
			when {
				expression {
					return env.BUILD_ANDROID == 'true' && env.BUILD_DEVELOP == 'true'
				}
			}
			environment {
				KEYSTORE_NAME = 'ProductionAndroid.keystore'
				KEYSTORE_PASS = credentials('funko-keystore-pass')
				KEYALIAS_NAME = 'funkopopblitz_store'
				KEYALIAS_PASS = credentials('funko-keyalias-pass')
			}
			steps {
				script {
					env.BUILD_AAB = 'true'
					env.ANDROID_HTTPS_PROXY = 'true'
					PreparePlatform('Android','ENABLE_CHEATS')
					BuildUnityProject('Android')
					GoogleDriveUploadTheAndroidAabAndApks('Develop')
				}
			}
        }
		
		stage ('iOS') {
			when {
				expression {
					return env.BUILD_IOS == 'true' && env.BUILD_DEVELOP == 'true'
				}
			}
			steps {
				script {
					PreparePlatform('iOS', 'ENABLE_CHEATS', 'ESG')
					BuildUnityProject()
					ArchiveXcode()
					ExportXcode()
					GoogleDriveUploadTheIosIpas('Develop')
				}
			}
		}
    }

/** Functions? */
					PreparePlatform('Android','ENABLE_CHEATS')
					BuildUnityProject('Android')
					GoogleDriveUploadTheAndroidAabAndApks('Develop')
	
					ArchiveXcode()
					ExportXcode()
    }
}
