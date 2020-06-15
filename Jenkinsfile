node{
checkout scm

    def project_path = "spring-boot-samples/spring-boot-sample-atmosphere/"
    def git_repository = "https://github.com/moussena1992/solitaire-systemjs-course.git"
    def credentials = "f57bfdcf-f995-4cd5-b26b-ff0ca8504fbe"

    notify("started")
    try{
        stage('checkout') {
            // checkout SCM
            // git branch: 'jenkins2-course',
            //    credentialsId: "${credentials}",
            //    url: "${git_repository}"
        }
        stage('dependencies, stash, testing') {

            // pull dependencies from npm
            // on windows use: bat 'npm install'
            sh 'npm install'

            // stash code & dependencies to expedite subsequent testing
            // and ensure same code & dependencies are used throughout the pipeline
            // stash is a temporary archive
            stash name: 'everything',
                  excludes: 'test-results/**',
                  includes: '**'

            // test with PhantomJS for "fast" "generic" results
            // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
            sh 'npm run test-single-run -- --browsers PhantomJS'
        }

    } catch (err) {
        notify("Error ${err}")
        currentBuild.result = 'FAILURE'
    }
    try{
        stage('archival'){
            /*
             publishHTML([allowMissing: true,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'target/site/jacoco/',
                reportFiles: 'index.html',
                reportName: 'Coverage Report',
                reportTitles: ''])
                */

            // archive karma test results (karma is configured to export junit xml files)
            step([$class: 'JUnitResultArchiver',
                    testResults: 'test-results/**/test-results.xml'])

            //archiveArtifacts "target/*.?ar"

          }
    } catch (err) {
        notify("Error ${err}")
        currentBuild.result = 'FAILURE'
    }
}

node('mac') {
    sh 'ls'
    sh 'rm -rf *'
    sh 'ls'
    unstash 'everything'
    sh 'ls'
}

//parallel integration testing
stage ('Browser Testing'){
    //stage 'Browser Testing'
    parallel chrome: {
        runTests("Chrome")
    }, firefox: {
        runTests("Firefox")
    }/*, safari: {
        runTests("Safari")
    }*/
}

node{
    notify("Deploy to staging")
}

input('Deploy to staging')

// limit concurrency so we don't perform simultaneous deploys
// and if multiple pipelines are executing,
// newest is only that will be allowed through, rest will be canceled
stage name: 'Deploy to staging', concurrency: 1
node {
    // write build number to index page so we can see this update
    // on windows use: bat "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"

    // deploy to a docker container mapped to port 3000
    // on windows use: bat 'docker-compose up -d --build'
    sh 'docker-compose up -d --build'

    notify 'Solitaire Deployed!'
}

def runTests(browser) {
    node {
        sh 'rm -rf *'

        unstash 'everything'

        sh "npm run test-single-run -- --browsers ${browser}"

        step([$class: 'JUnitResultArchiver',
              testResults: 'test-results/**/test-results.xml'])
    }
}

def notify(status){
    emailext (
      to: "moussena1992@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
