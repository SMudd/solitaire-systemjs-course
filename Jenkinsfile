stage 'CI'
node {

    checkout scm

    //git branch: 'jenkins2-course', url: 'https://github.com/g0t4/solitaire-systemjs-course'
    // pull dependencies from npm
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', excludes: 'test-results/**', includes: '**'
}

stage 'Browser Testing'
parallel    chrome:{runTests("Chrome")},
            firefox:{runTests("Firefox")},
            phantomjs:{runTests("PhantomJS")}    
            
node {
    notify('Deploy to staging?')
}

//FLyweight Executor - Don't tie up HeavyWeight executor
input 'Deploy to staging?'   

stage name: 'Deploy', concurrency: 1
node {
    sh "echo '<h1>${evn.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    sh 'docker-compose up -d --build'


}



def runTests(browser){
    node {
        sh 'ls'
        sh 'rm -rf'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', testResults: 'test-results/**/test-results.xml'])
    }    
}


def notify(status){
    emailext (
      to: "sm160866@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
