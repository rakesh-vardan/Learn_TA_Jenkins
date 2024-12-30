# Jenkins

## Freestyle Job Configuration

### Example-1

```
@echo off
echo Starting Jenkins Build > D:\JenkinsBuildLog.txt
echo ===================== >> D:\JenkinsBuildLog.txt
echo Build started at: %date% %time% >> D:\JenkinsBuildLog.txt
echo Running on: %COMPUTERNAME% >> D:\JenkinsBuildLog.txt
echo ===================== >> D:\JenkinsBuildLog.txt
echo Hello, Jenkins is working! >> D:\JenkinsBuildLog.txt
echo Build finished at: %date% %time% >> D:\JenkinsBuildLog.txt
echo ===================== >> D:\JenkinsBuildLog.txt
```

### Example-2

```
@echo off
setlocal

:: Define file location
set FILE_PATH=D:\JenkinsBuildFile.txt

:: Step 1: Create a new text file or overwrite if it exists
echo This is a Jenkins build file. > %FILE_PATH%

:: Step 2: Add some custom content
echo Jenkins job successfully started! >> %FILE_PATH%
echo Build started at: %date% %time% >> %FILE_PATH%

:: Step 3: Append system environment variables to the file
echo ===================================== >> %FILE_PATH%
echo System PATH: %PATH% >> %FILE_PATH%
echo User Profile: %USERPROFILE% >> %FILE_PATH%

:: Step 4: Display the contents of the file in the console
echo =====================================
echo Contents of the build file:
type %FILE_PATH%

endlocal
```

### Parameters

- For Windows

```
echo "Hello, %USER_NAME%! You have selected the %ENVIRONMENT% environment."
```

- For Linux

```
echo "Hello, ${USER_NAME}! You have selected the ${ENVIRONMENT}% environment."
```

### Pipeline-1

Simple declarative pipeline which prints "Hello World" with echo to the console - can be created from the sample pipeline snippet in Jenkins.

```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

### Pipeline-2

Declarative Pipeline to use GitHub & Maven  - can be created from the sample pipeline snippet in Jenkins.

```
pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "Maven"
    }

    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/jglick/simple-maven-project-with-tests.git'

                // Run Maven on a Unix agent.
                // sh "mvn -Dmaven.test.failure.ignore=true clean package"

                // To run Maven on a Windows agent, use
                bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
    }
}

```

### Pipeline-3

Scripted Pipeline to use GitHub & Maven  - can be created from the sample pipeline snippet in Jenkins.

```
node {
    def mvnHome
    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        git 'https://github.com/jglick/simple-maven-project-with-tests.git'
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'Maven'
    }
    stage('Build') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
            }
        }
    }
    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'target/*.jar'
    }
}

```

### Pipeline-4
```
pipeline {
    
    agent any
    
    parameters {
        string(name: 'NAME', description: 'Please tell me your name?')

        text(name: 'DESC', description: 'Describe about the job details')

        booleanParam(name: 'SKIP_TEST', description: 'Want to skip running Test cases?')

        choice(name: 'BRANCH', choices: ['Master', 'Dev'], description: 'Choose branch')

        password(name: 'SONAR_SERVER_PWD', description: 'Enter SONAR password')
    }
    
    stages {
        stage('Printing Parameters') {
            steps {
                
                echo "Hello ${params.NAME}"

                echo "Job Details: ${params.DESC}"

                echo "Skip Running Test case ?: ${params.SKIP_TEST}"

                echo "Branch Choice: ${params.BRANCH}"

                echo "SONAR Password: ${params.SONAR_SERVER_PWD}"
            }
        }
    }
}
```

### Pipeline-5

```
pipeline {
    agent any  // Run on any available agent

    tools {
        maven "Maven" // Install the Maven version configured as "Maven" and add it to the path.
    }

    environment {
        PROJECT_DIR = "${workspace}" // Adjust for Windows file paths
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository to the workspace
                git url: 'https://github.com/rakesh-vardan/TAF_API_RESTAssured.git', branch: 'main'
            }
        }

        stage('Verify Directory Structure') {
            steps {
                // Debugging: List files in the workspace to check if pom.xml exists
                script {
                    echo "Listing files in the project directory: ${PROJECT_DIR}"
                    bat "dir ${PROJECT_DIR}" // Windows command to list directory contents
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                // Run mvn clean install from the root directory of the project
                dir("${PROJECT_DIR}") {
                    script {
                        echo "Running mvn clean install"
                        bat "mvn clean install -DskipTests"  // Run Maven command on Windows
                    }
                }
            }
        }

        stage('Run API Tests') {
            steps {
                // Run the REST Assured API tests
                dir("${PROJECT_DIR}") {
                    script {
                        echo "Running mvn test"
                        bat "mvn test"  // Run Maven test on Windows
                    }
                }
            }
        }
    }

    post {
        always {
            // Publish the Allure report (ensure correct path)
            allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
            // Clean workspace after build
            cleanWs()
        }

        success {
            echo "Tests and reports were successfully generated!"
        }

        failure {
            echo "Tests failed. Please check the logs for details."
        }
    }
}

```

### Pipeline-6

```
pipeline {
    agent any  // Run on any available agent

    tools {
        maven "Maven" // Install the Maven version configured as "Maven" and add it to the path.
    }

    environment {
        PROJECT_DIR = "${workspace}" // Adjust for Windows file paths
    }

    stages {
        stage('Checkout') {
            steps {
                bat 'figlet "CHECKOUT"'
                // Checkout the Git repository to the workspace
                git url: 'https://github.com/rakesh-vardan/TAF_API_RESTAssured.git', branch: 'main'
            }
        }

        stage('Verify Directory Structure') {
            steps {
                // Debugging: List files in the workspace to check if pom.xml exists
                bat 'figlet "LIST FILES"'
                script {
                    echo "Listing files in the project directory: ${PROJECT_DIR}"
                    bat "dir ${PROJECT_DIR}" // Windows command to list directory contents
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                // Run mvn clean install from the root directory of the project
                bat 'figlet "INSTALL DEPENDENCIES"'
                
                dir("${PROJECT_DIR}") {
                    script {
                        echo "Running mvn clean install"
                        bat "mvn clean install -DskipTests"  // Run Maven command on Windows
                    }
                }
            }
        }

        stage('Run API Tests') {
            steps {
                // Run the REST Assured API tests
                bat 'figlet "RUN API TESTS"'
                
                dir("${PROJECT_DIR}") {
                    script {
                        echo "Running mvn test"
                        bat "mvn test"  // Run Maven test on Windows
                    }
                }
            }
        }
    }

    post {
        always {
            // Publish the Allure report (ensure correct path)
            allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
            // Clean workspace after build
            cleanWs()
        }

        success {
            echo "Tests and reports were successfully generated!"
        }

        failure {
            echo "Tests failed. Please check the logs for details."
        }
    }
}

```