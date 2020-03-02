# Jenkins Email-Ext Plugin
[![Build Status](https://ci.jenkins.io/job/Plugins/job/email-ext-plugin/job/master/badge/icon)](https://ci.jenkins.io/job/Plugins/job/email-ext-plugin/job/master/)
[![Jenkins Plugin](https://img.shields.io/jenkins/plugin/v/email-ext.svg)](https://plugins.jenkins.io/email-ext)
[![Jenkins Plugin Installs](https://img.shields.io/jenkins/plugin/i/email-ext.svg?color=blue)](https://plugins.jenkins.io/email-ext)


This plugin allows you to configure every aspect of email notifications.
You can customize when an email is sent, who should receive it, and what
the email says.

## Questions and bug reports
If you need help, please post on the mailing
lists <https://jenkins.io/mailing-lists/>, stack overflow or if you have
found a bug log an issue
at [https://issues.jenkins-ci.org](https://issues.jenkins-ci.org/) (no
support tickets please).

## Recipes

Additional recipes for email-ext can be found
[here](/docs/Recipes.md).

## General

This plugin extends Jenkins built in email notification functionality by
giving you more control.  It provides customization of 3 areas.

-   **Triggers** - Select the conditions that should cause an email
    notification to be sent.
-   **Content** - Specify the content of each triggered email's subject
    and body.  
-   **Recipients** - Specify who should receive an email when it is
    triggered.

## Configuration

### Global configuration

Before using email-ext on a project, you must configure some global
settings.  Go to Jenkins System configuration page.  Manage Jenkins -\>
Configure System

The section titled Extended E-mail Notification is where you can
configure global email-ext properties.  The properties here should match
the settings for your smtp mail server.  This section is set up to
mirror Jenkins own email Publisher Reporter (It's different extension
point), however there are a few additions.  The inputs labeled Default
Subject and Default Content, allow you to configure the email content on
a global level.  The input labeled Default Recipients can be used to set
a default list of email addresses for all projects using plugin (it can
be overridden at the project level). This can be used to greatly
simplify the configuration you need to do for all projects.

### Project configuration

For a project to use the email-ext plugin, you need to enable it in the
project configuration page.  Select the checkbox labeled "Editable Email
Notification" in the  "Post-build Actions" section.

#### Basic Configuration

There are three fields that you can edit when the plugin is enabled.

-   **Global Recipient List** - This is a comma (or whitespace)
    separated list of email recipients.  Allows you to specify a single
    recipient list for each email that is sent.
-   **Default Subject** - This allows you to configure a token (more
    about tokens later), that can be used to easily configure all email
    subjects for the project.
-   **Default Content** - Same as Default Subject, but for the email
    body instead of the subject.

### Advanced configuration

To see the advanced configuration for the plugin, first click on
Override Global setting checkbox, then click the "Advanced" button. 
This section allows you to specify recipients for each type of email
trigger as well as a pre-send script that can be used to modify the
email just prior to sending. 

#### Pre-send Script

The pre-send script is a feature which allows you to write a script that
can modify the
[MimeMessage](http://docs.oracle.com/javaee/1.4/api/javax/mail/internet/MimeMessage.html) object
prior to sending. This would allow adding custom headers, modifying the
body, etc. Predefined variables include:

-   msg - the MimeMessage object which can be modified
-   logger - a PrintStream and will write to the job's log. 
-   build - the build this message belongs to (only use with FreeStyle
    jobs)
-   run - the build this message belongs to (may be used with FreeStyle
    or Pipeline jobs)
-   cancel - a boolean, which when set to true will cancel the sending
    of the email

#### Triggers

By default, the only trigger configured is the "Failure" trigger.  To
add more triggers, select one from the dropdown, and it will be added to
the list.  Once you have added a trigger, you have several options.  If
you click "?" (question mark) next to a trigger, it will tell you what
conditions must be met for it to send an email.

-   **Send to Recipient List** - Check this checkbox if you would like
    to have the email sent to the "Global Recipient List" configured
    above.
-   **Send to Developers** - Check this checkbox to send the email to
    anyone who checked in code for the last build.  The plugin will
    generate an email address based on the committer's id and an
    appended "default email suffix" from Jenkins's global configuration
    page.  For instance, if a change was committed by someone with an id
    "first.last", and the default email suffix is "@somewhere.com", then
    an email will be sent to "first.last@somewhere.com"
-   **Send To Requester** - If this is checked, an email will be sent
    user who triggers the build (if triggered by a user manually).
-   **Include Culprits** - If this is checked AND Send To Developers is
    checked, emails will include everyone who committed since the last
    successful build.
-   **More Configuration** - Configure properties at a per-trigger
    level.
    -   **Recipient List** - A comma (and whitespace) separated list of
        email address that should receive this email if it is
        triggered.  This list is appended to the "Global Recipient List"
        above.
    -   **Subject** - Specify the subject line of the selected email.
    -   **Content** - Specify the body of the selected email.
-   **Remove** - Click the delete button next to an email trigger to
    remove it from the configured triggers list.

You can use Trigger Scripts in groovy to define before of after the
build if the email must be send or not.

There are four objects added to the model for the script to use to
interact with the build.

-   **build**: This is the current build, usually a child class of
    AbstractBuild
-   **project**: The project object that the current build was started
    from, usually a child class of AbstractProject
-   **rooturl**: The Jenkins instance root URL, useful for links.
-   **out**: A PrintStream that can be used to log messages to the build
    log.

The last line in the script should resolve to a boolean true or false

Examples:

-   Before build scripts

``` groovy
// this could be used to notify people that a new build is happening
build.previousBuild.result.toString().equals('FAILURE')
```

-   After build scripts

``` groovy
// only send am email if the build failed and 'mickeymouse' had a commit
build.result.toString().equals('FAILURE') && build.hasParticipant(User.get('mickeymouse'))
```

``` groovy
// only send an email if the word {{ERROR}} is found in build logs
build.logFile.text.readLines().any { it =~ /.*ERROR.*/ }
```

#### Email tokens

The email-ext plugin uses ***tokens*** to allow dynamic data to be
inserted into recipient list, email subject line or body.   A
***token*** is a string that starts with a `$` (dollar sign) and is
terminated by whitespace.  When an email is triggered, any tokens in the
subject or content fields will be replaced dynamically by the actual
value that it represents.  Also, the "value" of a token can contain
other tokens, that will themselves be replaced by actual content.  For
instance, the `$DEFAULT_SUBJECT` token is replaced by the text (and other
tokens) that is in the Default Subject field from the **global
configuration page**.  Similarly, the `$PROJECT_DEFAULT_SUBJECT` token
will be replaced by the value of the Default Subject field from the
**project configuration page**. 

The email-ext plugin sets the email content fields with default values
when you enable it for your project.  The Default Subject and Default
Content fields on the project config page default to `$DEFAULT_SUBJECT`
and `$DEFAULT_CONTENT` (respectively), so that it will automatically use
the global configuration.  Similarly, the per-trigger content fields
default to `$PROJECT_DEFAULT_SUBJECT` and `$PROJECT_DEFAULT_CONTENT`, so
that they will automatically use the project's configuration.  Since the
value of a token can contain other tokens, this provides different
points of configuration that can allow you to quickly make changes at
the broadest level (all projects), the narrowest level (individual
email), and in between (individual project).

To see a list of all available email tokens and what they display, you
can click the "?" (question mark) associated with the Content Token
Reference at the top bottom of the email-ext section on the project
configuration screen.

#### Token Macro Tokens

As of version 2.22, email-ext supports tokens provided by the
[token-macro plugin](https://plugins.jenkins.io/token-macro/). You
can see the available token-macro token below the email-ext tokens when
you click the "?" (question mark) associated with the Content Token
Reference at the bottom of the email-ext section on the project
configuration screen.

#### Jelly content

![](docs/images/html.jpg)

![](docs/images/txt.jpg)

New to version 2.9 is the ability to use Jelly scripts. Jelly scripts
are powerful in that you can hook into the Jenkins API itself to get any
information you want or need. There are two Jelly scripts packaged with
the plugin and it is possible to write your own too.

There are two default Jelly scripts available out of the box; one is
designed for HTML emails and the other is design for text emails. See
the screenshots to the right for what these templates look like. You can
specify which script you want by using the *template* argument. The
usage for each script is the following:

-   Text only Jelly script: `${JELLY_SCRIPT,template="text"}`
-   HTML Jelly script: `${JELLY_SCRIPT,template="html"}`

You can also write your own Jelly scripts. The Jelly scripts are
particularly powerful since they provide a hook into the Jenkins API
including
[hudson.model.AbstractBuild](http://javadoc.jenkins-ci.org/hudson/model/AbstractBuild.html)
and
[hudson.model.AbstractProject](http://javadoc.jenkins-ci.org/hudson/model/AbstractProject.html).
For example on how to do this, take a look at the existing
[html](https://github.com/jenkinsci/email-ext-plugin/blob/master/src/main/resources/hudson/plugins/emailext/templates/html.jelly)
and
[text](https://github.com/jenkinsci/email-ext-plugin/blob/master/src/main/resources/hudson/plugins/emailext/templates/text.jelly)
scripts.

Using custom Jelly scripts (those not packaged with email-ext) requires
the cooperation of your Hudson administrator. The steps are relatively
simple:

1.  Create the Jelly script. The name of the script should be
    `<name>.jelly`. It is important the name ends in `.jelly`.
2.  Have your Jenkins administrator place the script inside
    `$JENKINS_HOME/email-templates/`.
3.  Use the Jelly token with the template parameter equal to your script
    filename without the .jelly extension. For example, if the script
    filename is foobar.jelly, the email content would look like this
    `${JELLY_SCRIPT,template="foobar"}`.

Jelly script tips:

-   You get object of other plugin actions by querying build actions
    like:
    `${it.getAction('hudson.plugins.fitnesse.FitnesseResultsAction')}`
-   Then you need to know what all functions are allowed by this action
    object and traverse through result.

#### Script content

New to version 2.15 is the ability to use Groovy scripts. Scripts are
powerful in that you can hook into the Jenkins API itself to get any
information you want or need. There are two scripts with corresponding
templates packaged with the plugin and it is possible to write your own
too.

There are two default scripts and templates available out of the box;
one is designed for HTML emails and the other is design for text emails.
You can specify which script you want by using the *script \_argument,
you can also just leave the default script and specify a different
template file using the \_template* argument. Further, you can also
include an init script that does some initialization using the *init*
argument. The usage for each script is the following:

-   Text only template: `${SCRIPT, template="groovy-text.template"}`
-   HTML template: `${SCRIPT, template="groovy-html.template"}`

You can also write your own scripts and templates. The scripts are
particularly powerful since they provide a hook into the Jenkins API
including [hudson.model.AbstractBuild](http://javadoc.jenkins-ci.org/hudson/model/AbstractBuild.html) and [hudson.model.AbstractProject](http://javadoc.jenkins-ci.org/hudson/model/AbstractProject.html).
For example on how to do this, take a look at the
existing [html](https://github.com/jenkinsci/email-ext-plugin/blob/master/src/main/resources/hudson/plugins/emailext/templates/groovy-html.template) and [text](https://github.com/jenkinsci/email-ext-plugin/blob/master/src/main/resources/hudson/plugins/emailext/templates/groovy-text.template) scripts.

Using custom scripts (those not packaged with email-ext) requires the
cooperation of your Jenkins administrator. The steps are relatively
simple:

1.  Create the script/template. The name of the script end in the
    standard extension for the language (.groovy). The template can be
    named anything
2.  Have your Jenkins administrator place the script inside
    `$JENKINS_HOME\email-templates`.
3.  Use the script token with the template parameter equal to your
    template filename, or in addition the script parameter equal to the
    custom script name. For example, if the template filename is
    foobar.template, the email content would look like this ${SCRIPT,
    template="foobar.template"}.

### Template Examples

These are some useful examples for doing various things with the
email-ext groovy templates.

* [jenkins-matrix-email-html.template](/docs/templates/jenkins-matrix-email-html.template)
* [jenkins-generic-matrix-email-html.template](/docs/templates/jenkins-generic-matrix-email-html.template)

### Pipeline Examples

See [email-ext](https://jenkins.io/doc/pipeline/steps/email-ext/) for
command signatures

Notify Culprits and Requester via default EMail plugin

``` groovy
step([$class: 'Mailer', notifyEveryUnstableBuild: true, 
    recipients: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                    [$class: 'RequesterRecipientProvider']])])
```

Send an email to `abc` plus any addresses returned by the providers

``` groovy
emailext body: 'A Test EMail', 
    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
    subject: 'Test', to: 'abc'
```

  
### Attachments

New to version 2.15 is the ability to add attachments using the Ant
pattern matching syntax used in many places in Jenkins. You can set a
maximum total attachment size in the global configuration page, or it
will be unlimited. 

### Jive Formatter

[jive-formatter.groovy](/docs/templates/jive-formatter.groovy)
contains methods for easy and convenient formatting of emails being sent
from Jenkins to Jive. It should be called from the Pre-send Script area.

Also, it doesn't seem like Jive supports text with multiple formats, so
only call one formatting method per block of text.

Either formatLine or formatText can and should be called on every line
of text that will be sent to the Jive system prior to calling formatting
methods like color or size. Please test on your own instances of Jive
and add functionality as you find it!

The following lines should be added to the Pre-send Script area prior to
attempting to invoke any functions.

**Pre-send Script**

``` groovy
File sourceFile = new File("/your/preferred/path/jive-formatter.groovy");
Class groovyClass = new GroovyClassLoader(getClass().getClassLoader()).parseClass(sourceFile);
GroovyObject jiveFormatter = (GroovyObject) groovyClass.newInstance();
```

### Plugins

-   [Email Ext Recipients Column Plugin](https://plugins.jenkins.io/email-ext-recipients-column/)

-   [Job Direct Mail Plugin](https://plugins.jenkins.io/job-direct-mail/)
   
-   [Pom2Config Plugin](https://plugins.jenkins.io/pom2config/)
   
-   [GitHub Integration Plugin](https://plugins.jenkins.io/github-pullrequest/)

-   [Email-ext Template Plugin](https://plugins.jenkins.io/emailext-template/)

-   [Configuration Slicing Plugin](https://plugins.jenkins.io/configurationslicing/)

-   [View Job Filters](https://plugins.jenkins.io/view-job-filters/)

-   [Run Condition Extras Plugin](https://plugins.jenkins.io/run-condition-extras/)

## Contributing to Email-Ext plugin

Make sure you have installed [Maven 3](http://maven.apache.org/) 
and JDK 8.0 or later. Make also sure you have properly configured your
`~/.m2/settings.xml` as explained in the [Plugin Tutorial](https://jenkins.io/doc/developer/tutorial/).
Those are needed to build properly any Jenkins plugin.

### Check out and build

How to check out the source and build:

``` sh
git clone git@github.com:jenkinsci/email-ext-plugin.git
cd email-ext-plugin
mvn clean install
```

## Version History

Please refer to [the changelog](CHANGELOG.md).
