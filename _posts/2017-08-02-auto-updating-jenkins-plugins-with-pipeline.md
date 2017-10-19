---
layout: post
title: Updating Jenkins Plugins With Pipeline
tags: jenkins automation
comments: True
excerpt_separator: <!--more-->
---

![Question](/images/update_ask.png)

Do you ever notice that Jenkins plugins versions are being updatedso fast? It\`s either good or bad: good, because we`ve got new features and bugfixes so often; bad, because we often need to update them manually. :)

But we can automate plugins update procedure with Jenkins cool pipeline! I write an example `Jenkinsfile` using magic of `shell` and `groovy`:

<!--more-->

```groovy
#!/usr/bin/env groovy

def jenkins_cli = "/var/jenkins_home/war/WEB-INF/jenkins-cli.jar"
def jenkins_url = "http://127.0.0.1:8080/"
def admin_email = "admin@domain.com"
def credentials = "--username \"someuser\" --password \"somepassword\""

node {
    properties(
    [
        [
            $class: 'BuildDiscarderProperty',
            strategy: [$class: 'LogRotator', numToKeepStr: '5']
        ],
        pipelineTriggers([cron('@monthly')]),
    ]
    )

    try {
        stage('Check Updates') {
            updates = sh(returnStdout: true,
                             script: "java -jar ${jenkins_cli} -s ${jenkins_url} list-plugins | \
                                      grep -e ')\$' | \
                                      awk '{ print \$1 }'").trim()
        }
        if (updates) {
            
            stage('Ask') {
                body = "Plugins to update:<b>${updates}</b><br> \
                        Please  <a href='${env.BUILD_URL}/input'>decide</a> what to do.<br> \
                        You better <a href='${env.JENKINS_URL}/pluginManager'>look</a> at plugins</a> before updating.<br> \
                        Will automatically proceed in 24 hours!"
                emailext body: body,
                         mimeType: 'text/html',
                         subject: 'Jenkins wants to update plugins',
                         to: admin_email
                timeout(time: 24, unit: 'HOURS') {
                    input message: "===Updates===\n"+
                                   "${updates}"+
                                   "\n======\n"+
                                   "Proceed to install updates?"
                }
            }
            
            stage('Do Update') {
                updates = updates.replace("\n", " ")
                sh "java -jar ${jenkins_cli} -s ${jenkins_url} install-plugin ${updates} ${credentials}"
            }
            stage('Safe Restart') {
                sh "java -jar ${jenkins_cli} -s ${jenkins_url} safe-restart ${credentials}"
            }
        }
        else {
            echo "No updates available!"
        }
}
    catch (err) {
        throw err
    }
}
```

As a result, this job will be started `@monthly`, check available updates, send an email to you to confirm update. If you don\`t confirm it in 24 hours, job will update plugins automatically.

