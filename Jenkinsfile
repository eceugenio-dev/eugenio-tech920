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
        
		stage ('Workspace Cleanup') {
			when {
				expression {
					return env.CLEAN_WORKSPACE == 'true'
				}
			}
			steps {
				script {
					cleanWs()
				}
			}	
		}
		
		stage ('Library Clean') {
			when {
				expression {
					return env.CLEAN_BUILD == 'true'
				}
			}
			steps {
				CleanLibrary()
			}
		}
		
        stage('Build Setup') {
            steps {
                script {
                    String branchName = env.GIT_BRANCH.replaceAll('origin/', '').replaceAll('/', '-')
                    
                    int buildNum = "$BUILD_NUMBER" as Integer
                    String newBuildNum = buildNum.toString()
                    
                    currentbuild.displayName = "#$BUILD_NUMBER: $(branchName) (Build number $(newBuildNum))"
				}
            }
        }
        
        stage('Build Android') {
			when {
				expression {
					return env.BUILD_ANDROID == 'true' && env.BUILD_DEVELOP == 'true'
				}
			}
			environment {
				/** what would be the following values here if not fpb? */
				KEYSTORE_NAME = 
				KEYSTORE_PASS = 
				KEYALIAS_NAME = 
				KEYALIAS_PASS = 
			}
			steps {
				script {
					env.BUILD_AAB = 'true'
					env.ANDROID_HTTPS_PROXY = 'true'
					PreparePlatform('Android')
					BuildUnityProject('Android')
				}
			}
        }
		
		stage ('Android Clean Up') {
			steps {
				echo 'Cleaning up generated files'
				script {
					Remove('./eugenio-920/Builds/*.zip')
					Remove('*symbols.zip')
					Remove('./eugenio-920/Builds/Jenkins')
				}
			}
		}
		
/**		stage ('iOS') {
			when {
				expression {
					return env.BUILD_IOS == 'true' && env.BUILD_DEVELOP == 'true'
				}
			}
			steps {
				script {
					PreparePlatform('iOS')
					BuildUnityProject()
					ArchiveXcode()
					ExportXcode()
				}
			}
		}
		
		stage ('iOS Clean Up') {
			steps {
				echo 'Cleaning up generated files'
				script {
					Remove('./ipa-share')
					Remove('./eugenio-920/Builds/*.zip')
					Remove('*symbols.zip')
					Remove('./eugenio-920/Builds/Jenkins')
					Remove('*.zip')
				}
			}
		}
	} */
		
	post {
        always {
            echo 'Jenkins reached the post always block'
            build wait: false, job: 'ConsoleLogUploader', parameters: [string(name: 'CONSOLE_LOG_URL', value: env.BUILD_URL+'/consoleText'), string(name: 'GDRIVE_FOLDER', value: GetGoogleDriveFolderId()), string(name: 'BRANCH_NAME', value: env.FOLDER_PATH), string(name: 'LOG_NAME', value: env.JOB_NAME + '-' + env.BUILD_NUMBER + '_' + env.BRANCH + '_' + env.GIT_COMMIT_HASH + '.log')]
        }
		
        success {
            echo "$JOB_NAME Build Successful Link to Jenkins: <$BUILD_URL|Build $BUILD_NUMBER> Link to GDrive: <https://drive.google.com/drive/folders/${googleDriveID.trim()}|Google Drive>"
            }
        }
		
        failure {
            echo "$JOB_NAME Build Failed Link to Jenkins: <$BUILD_URL|Build $BUILD_NUMBER>"
        }
		
        aborted {
            echo "$JOB_NAME Build Aborted Link to Jenkins: <$BUILD_URL|Build $BUILD_NUMBER>"
        }	
    }
}

/** Gets the current git commit hash */
def String GetCommitHash() {
    if (isUnix()) {
        return sh(script: "git rev-parse --short=8 HEAD", returnStdout: true).trim()
    } else {
        return bat(script: '@git rev-parse --short=8 HEAD', returnStdout: true).trim()
    }
}

/** Deletes the Library folder and some other temp folders if CLEAN_BUILD is set to true */
def void CleanLibrary() {
    LogEnvironmentVariables()
	
	echo 'CLEAN_BUILD was true.'
	Remove('eugenio-920/Library')
	Remove('eugenio-920/Logs')
	Remove('eugenio-920/Temp')
	Remove('eugenio-920/obj')

    echo 'Running git checkout -- . to restore addressables files in the Library folder'
    if (isUnix()) {
        sh 'git checkout -- .'
    } else {
        bat 'git checkout -- .'
    }
}

def void PreparePlatform(String platformIdenfitier){
	if(isUnix()){
		PrepareUnix(platformIdenfitier);
	}
	else{
		PrepareWindows(platformIdenfitier);
	}
}

def void PrepareUnix(String platformIdenfitier) {
    LogEnvironmentVariables()

	echo 'Checking what version of Java is used in the shell. Note: This is probably different than the Java that the Unity Editor uses when making builds. I believe by default the Unity Editor is set to use the OpenJDK included with the editor.'
	sh 'java --version'
	
	echo "Preparing ' + platformIdenfitier + ' "
    echo 'Deleting any preexisting symbols file, so we can use wildcards to rename it later'
    Remove('eugenio-920\\Builds\\Jenkins*.symbols.zip')
    Remove('eugenio-920\\Builds\\Jenkins\\Jenkins*.symbols.zip')
    Remove('*.symbols.zip')
	
    sh 'ls'
    sh 'ls eugenio-920'

    echo 'TTT-920. Displaying the current status of the local repository. Keep an eye out for assets that are getting modified at build time. Assets should be committed to the repository in a state where they do not get modified at build time.'
    sh 'git status'
    sh 'git diff'
    echo 'TTT-920 Clearing out modified files and getting back to a clean commit'
    sh 'git checkout -- .'
    sh 'git clean -fd'
	
    echo 'TTT-920 Ran into a problem where a ton of files had modified Unix permissions, making some Git commands fail. Telling Git to ignore file permissions.'
    sh 'git config core.filemode false'
	
    sh 'git status'
    sh 'git diff'

    echo 'default aws cli location is supposed to be in /usr/local/bin which should also be part of the path set on this node'
    sh 'ls /usr/local/bin/'

    echo 'Updating git config --global to make sure git commands will work'
    sh 'git config --global user.email "earleugenio@gmail.com"'
    sh 'git config --global user.name "eceugenio-dev"'

    //setting env.BRANCH here to ensure that we have a value in case of failure scenarios
    env.BRANCH = "$GIT_BRANCH".replaceAll('origin/', "").replaceAll('/', '-')

    env.GIT_COMMIT_HASH = gitCommitHash

    echo 'Versions of Unity found on this build node'
    sh 'ls /Applications/Unity/Hub/Editor'
    
	echo 'Completed preparing the Unity project'

    echo 'TTT-920. Displaying the current status of the local repository. Keep an eye out for assets that are getting modified at build time. Assets should be committed to the repository in a state where they do not get modified at build time.'
    sh 'git status'
    sh 'git diff'
}

/** Runs "del path" and "rmdir path /s /q" on Windows, "rm -rf path" on Bash. Catches any exceptions. Attempts to convert path separators */
def void Remove(String path) {
    if (path == null || path == '') {
        echo 'Cannot RemoveFile a null or empty path'
        return
    }

    // Maybe we should remove this check and just use the catch block for everything?
    // Makes exceptions for wildcards
    if (!path.contains('*') && !path.contains('?') && !fileExists(path)) {
        echo 'fileExists returned false for path ' + path
        return
    }

    echo 'Removing path ' + path
    if (isUnix()) {
        path = path.replace('\\', '/')
        try {
            sh 'rm -rf ' + path
        } catch (isUnixDeleteFileException) {
            echo 'Unable to delete path ' + path + ' most likely because it did not exist'
            echo isUnixDeleteFileException.toString()
        }
    } else {
        path = path.replace('/', '\\')
        try {
            bat 'rmdir ' + path + ' /s /q'
        } catch (removeDirectoryException) {
            echo 'Unable to remove directory ' + path + ' most likely because it did not exist'
            echo removeDirectoryException.toString()
        }
        try {
            bat 'del ' + path
        } catch (deleteException) {
            echo 'Unable to delete path ' + path + ' most likely because it did not exist'
            echo deleteException.toString()
        }
    }
}

def void PrepareWindows(String platformIdenfitier) {
    LogEnvironmentVariables()

	echo 'Checking what version of Java is used in the shell. Note: This is probably different than the Java that the Unity Editor uses when making builds. I believe by default the Unity Editor is set to use the OpenJDK included with the editor.'
	sh 'java --version'
	
	echo "Preparing ' + platformIdenfitier + ' "
    echo 'Deleting any preexisting symbols file, so we can use wildcards to rename it later'
    Remove('eugenio-920\\Builds\\Jenkins*.symbols.zip')
    Remove('eugenio-920\\Builds\\Jenkins\\Jenkins*.symbols.zip')
    Remove('*.symbols.zip')
	
    bat 'dir'
    bat 'dir eugenio-920'

    echo 'TTT-920. Displaying the current status of the local repository. Keep an eye out for assets that are getting modified at build time. Assets should be committed to the repository in a state where they do not get modified at build time.'
    bat 'git status'
    bat 'git diff'
    echo 'TTT-920 Clearing out modified files and getting back to a clean commit'
    bat 'git checkout -- .'
    bat 'git clean -fd'
    bat 'git status'
    bat 'git diff'

    echo 'Updating git config --global to make sure git commands will work'
    bat 'git config --global user.email "earleugenio@gmail.com"'
    bat 'git config --global user.name "eceugenio-dev"'

    //setting env.BRANCH here to ensure that we have a value in case of failure scenarios
    env.BRANCH = "$GIT_BRANCH".replaceAll('origin/', "").replaceAll('/', '-')

    env.GIT_COMMIT_HASH = gitCommitHash

    echo 'Versions of Unity found on this build node'
    bat 'dir "C:\\Program Files\\Unity\\Hub\\Editor"'
    
	echo 'Completed preparing the Unity project'

/**    echo 'TTT-920. Displaying the current status of the local repository. Keep an eye out for assets that are getting modified at build time. Assets should be committed to the repository in a state where they do not get modified at build time.'
    bat 'git status'
    bat 'git diff' */
}

def void BuildUnityProject(String platformIdenfitier) {
	LogEnvironmentVariables()

	echo 'Removing old builds'
	Remove('./*.apk')
	Remove('./*.ipa')
	Remove('./builds')
	Remove('./FunkoPopProject/Builds')

	echo 'Building the Unity project'
	if (isUnix()) {
		sh '/Applications/Unity/Hub/Editor/2021.3.11f1/Unity.app/Contents/MacOS/Unity -batchmode -nographics -quit -projectPath ./eugenio-920 -buildTarget ' + platformIdenfitier + ' -username earl@tictocgames.com -password iamdaShyUnity003! -executeMethod TicToc.EditorTools.Editor.PlatformBuilder.Build' + platformIdenfitier 
		sh 'ls eugenio-920/Builds/Jenkins'
		sh 'git status'
	}
	else {
		bat '"C:\\Program Files\\Unity\\Hub\\Editor\\2021.3.11f1\\Editor\\Unity.exe" -batchmode -nographics -quit -projectPath ./eugenio-920 -buildTarget ' + platformIdenfitier + ' -username earl@tictocgames.com -password iamdaShyUnity003! -executeMethod TicToc.EditorTools.Editor.PlatformBuilder.Build' + platformIdenfitier 
		bat 'dir eugenio-920\\Builds\\Jenkins'
		bat 'git status'
	}
}

def void LogEnvironmentVariables() {
    echo 'Logging current environment variables'
    if (isUnix()) {
        sh 'printenv'
    }
    else {
        bat 'set'
    }
}
