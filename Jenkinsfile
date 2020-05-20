//Change build display name
currentBuild.displayName = "decalarative-solitaire-#"+currentBuild.number

pipeline{
    agent any
    environment{
        DOCKER_HOST = 'tcp://192.168.99.100:2376'
        DOCKER_CERT_PATH = 'C:/Users/ekant/.docker/machine/machines/default'
        DOCKER_MACHINE_NAME = 'default'
        DOCKER_TLS_VERIFY = '1'
        DOCKER_TOOLBOX_INSTALL_PATH = 'C:/Program Files/Docker Toolbox'
        DOCKER_TAG = getDockerTag()
        DOCKER_REGISTRY = 'tany2020/pipeline'
    }
    stages{
        stage("install dependencies"){
            steps{
                bat 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
                stash name: 'everything', 
                excludes: 'test-results/**', 
                includes: '**'
            }
        }
        stage("browser testing"){
            steps{
                parallel chrome: {
                runTests("Chrome")
                }, phantomjs: {
                runTests("PhantomJS")
                }
            }
        }
        stage("docker build-push"){
            steps{
                 bat label: 'version check', script: 'C:/"Program Files"/"Docker Toolbox"/docker -v'
                 //bat label: 'package', script: 'cd src'
                 bat label: 'docker build', script: 'C:/"Program Files"/"Docker Toolbox"/docker build . -t %DOCKER_REGISTRY%:%DOCKER_TAG%'
            }
        }
    }
}

def runTests(browser) {
        bat 'del /S /Q *'

        unstash 'everything'

        bat "npm run test-single-run -- --browsers ${browser}"

        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
}

def getDockerTag(){
       stdout = bat(returnStdout:true , script: 'git rev-parse HEAD').trim()
       tag = stdout.readLines().drop(1).join(" ")       
       return tag
    }
    
