#!groovy
def countries = ['be', 'ch', 'nl']
def autoDeploy = isMaster() || isReleaseBranch()
def autoUndeploy = !isMaster()
def updateVersion = !isMaster() && !isReleaseBranch()

def buildTag

echo "isMaster: ${isMaster()}"
echo "isReleaseBranch: ${isReleaseBranch()}"

properties([parameters([booleanParam(defaultValue: false, description: '', name: 'merge'),
                        string(defaultValue: 'origin/master', description: '', name: 'mergeTarget')]),
            [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
            pipelineTriggers([])])

stage('Build') {
    node {
        checkout scm

        buildTag = "build/${BRANCH_NAME}/${BUILD_NUMBER}"

        if (merge) {
            bat 'git config --global user.email "you@example.com"'
            bat 'git config --global user.name "Julian F."'

            bat "git checkout $mergeTarget"
            bat "git merge --no-ff origin/${BRANCH_NAME}"
        }

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

    forEachCountry(countries, { country ->
        node {
            lock(quantity: 1, label: 'mimas_it', variable: 'DBUSER') {
                checkout scm
                //bat "git checkout $buildTag"

                echo 'got: ' + ${env.DBUSER}
                echo "bootstrap $country"
                echo "it test $country"
            }
        }
    })
}

stage('UI-Test') {
    if (!autoDeploy) {
        input message: 'Deploy?'
    }

    milestone label: 'UI'

    forEachCountry(countries, { country ->

        withTestEnv(country, { envName, dbUser, envUrl ->
            build job: 'deploy', parameters: [string(name: 'ENV_NAME', value: envName),
                                              string(name: 'DB_USER', value: dbUser),
                                              string(name: 'COUNTRY', value: country)]
            node {
                echo 'checkout'
                echo "geb on $envUrl"
            }

            if (!autoUndeploy) {
                input message: 'Undeploy?'
            }
            // build job undeploy
        })
    })
}

stage('merge') {
    if (merge) {
        input message: "push merge ${BRANCH_NAME} to $mergeTarget ?"
        milestone label: 'merge'

        node {
            checkout scm
            bat "git checkout $buildTag"

            //bat "git push"
        }

        if (!isMaster()) {
            input message: "delete branch ${BRANCH_NAME} ?"
            node {
                checkout scm
                //bat "git push origin --delete ${BRANCH_NAME}"
            }
        }
    }
}

// finally
stage('delete build tag') {
    node {
        checkout scm
        //bat "git push origin :refs/tags/$buildTag"
    }
}

def forEachCountry(countries, task) {
    def tasks = [:]
    for (i = 0; i < countries.size(); i++) {
        def country = countries[i] //TODO
        tasks.put(countries[i], { task.call(country) })
    }

    parallel(tasks)
}

def withTestEnv(country, task) {

    if (isMaster()) {
        def dbUser = "ci_$country"
        def envName = envNameForBranch(BRANCH_NAME)
        def envUrl = "${country}.cosmolb.mgm-edv.de/mimas/ci"
        task.call(envName, dbUser, envUrl)
    } else {
        lock(quantity: 1, label: 'mimas_dev_env', variable: 'DBUSER') {
            def dbUser = ${env.DBUSER}
            def envName = envNameForBranch(BRANCH_NAME)
            def envUrl = "${country}.cosmolb.mgm-edv.de/mimas/$envName"
            task.call(envName, dbUser, envUrl)
        }
    }
}

def lockedResource() {
    org.jenkins.plugins.lockableresources.LockableResourcesManager.class.get().getResourcesFromBuild(currentBuild.getRawBuild())[0].getName()
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