#!groovy
def countries = ['be', 'ch', 'nl']
def autoDeploy = isMaster() || isReleaseBranch()
def autoUndeploy = !isMaster()
def skipUndeploy = isMaster()
def updateVersion = !isMaster() && !isReleaseBranch()

def buildTag

echo "isMaster: ${isMaster()}"
echo "isReleaseBranch: ${isReleaseBranch()}"

stage('Build') {
    node {
        checkout scm

        buildTag = "build/${BRANCH_NAME}/${BUILD_NUMBER}"

        bat "git tag $buildTag"
        //bat "git push origin $buildTag"

        // if master or release
        if (updateVersion) {
            echo 'update pom version'
        }

        echo 'mvn deploy'
        bat 'set'
    }
}

stage('IT-Test') {
    milestone label: 'IT'

    // don't move to a methode see JENKINS-38268
    def tasks = [:]
    for (i = 0; i < countries.size(); i++) {
        def country = countries[i]

        tasks.put(country, {
            node {
                lock(quantity: 1, label: 'mimas_it', variable: 'DBUSER') {
                    checkout scm
                    //bat "git checkout $buildTag"

                    echo 'got: ' + env.DBUSER
                    echo "bootstrap $country"
                    echo "it test $country"
                }
            }
        })
    }

    parallel(tasks)
}

stage('UI-Test') {
    if (!autoDeploy) {
        input message: 'Deploy?'
    }

    milestone label: 'UI'

    // don't move to a methode see JENKINS-38268
    def tasks = [:]
    for (i = 0; i < countries.size(); i++) {
        def country = countries[i]

        tasks.put(country, {
            withTestEnv(country, { envName, dbUser, envUrl ->
                build job: 'deploy', parameters: [string(name: 'ENV_NAME', value: envName),
                                                  string(name: 'DB_USER', value: dbUser),
                                                  string(name: 'COUNTRY', value: country)]
                node {
                    echo 'checkout'
                    echo "geb on $envUrl"
                }

                if (!skipUndeploy) {
                    if (!autoUndeploy) {
                        input message: 'Undeploy?'
                    }
                    echo "undeploy country: $country envName:$envName"
                    // build job undeploy
                }
            })
        })
    }

    parallel(tasks)
}

// finally
stage('delete build tag') {
    node {
        checkout scm
        //bat "git push origin :refs/tags/$buildTag"
    }
}

def withTestEnv(country, task) {

    if (isMaster()) {
        def dbUser = "ci_$country"
        def envName = envNameForBranch(BRANCH_NAME)
        def envUrl = "${country}.cosmolb.mgm-edv.de/mimas/ci"
        task.call(envName, dbUser, envUrl)
    } else {
        lock(quantity: 1, label: 'mimas_dev_env', variable: 'DBUSER') {
            def dbUser = env.DBUSER
            def envName = envNameForBranch(BRANCH_NAME)
            def envUrl = "${country}.cosmolb.mgm-edv.de/mimas/$envName"
            task.call(envName, dbUser, envUrl)
        }
    }
}

def envNameForBranch(branch) {

    if (branch == 'master') {
        return 'ci'
    }

    def group = (branch =~ /(.+)\/(.+)/)

    def type = group[0][1]
    def qualifier = group[0][2]

    if (type == 'release') {
        return "rel-$qualifier"
    }

    return qualifier
}

def isMaster() {
    BRANCH_NAME == 'master'
}

def isReleaseBranch() {
    def group = (BRANCH_NAME ==~ /release\/.+/)
    return group
}