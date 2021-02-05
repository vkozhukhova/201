#Jenkins 
- a self-contained, open source automation server 
- used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

# Jenkins Pipeline 
- is a suite of plugins that supports implementing and integrating continuous delivery pipelines into Jenkins. 
- Pipeline provides an extensible set of tools for modeling simple-to-complex delivery pipelines "as code" via the Pipeline DSL (domain-specific language).
- A __continuous delivery (CD) pipeline__ is an automated expression of your process for getting software from version control right through to your users and customers. This process involves building the software in a reliable and repeatable manner, as well as progressing the built software through multiple stages of testing and deployment.

- Jenkins is an automation engine which supports a number of automation patterns. 
- Pipeline adds a powerful set of automation tools onto Jenkins, supporting use cases that span from simple continuous integration to comprehensive CD pipelines. 
- By modeling a series of related tasks, users can take advantage of the many features of Pipeline (code, durable, pausable, versatile, extensible)
- Pipeline is also extensible both by users with Pipeline Shared Libraries and by plugin developers

#Pipeline features

1. Code: Pipelines are implemented in code and typically checked into source control, giving teams the ability to edit, review, and iterate upon their delivery pipeline.
1. Durable: Pipelines can survive both planned and unplanned restarts of the Jenkins master.
1. Pausable: Pipelines can optionally stop and wait for human input or approval before continuing the Pipeline run.
1. Versatile: Pipelines support complex real-world CD requirements, including the ability to fork/join, loop, and perform work in parallel.
1. Extensible: The Pipeline plugin supports custom extensions to its DSL and multiple options for integration with other plugins.

# Pipeline concepts

- __Pipeline__ is a user-defined model of a CD pipeline.
- __pipeline block__ is a key part of __Declarative Pipeline syntax__.
- __Node__ is a machine which is part of the Jenkins environment and is capable of executing a __Pipeline__.
- __node block__ is a key part of __Scripted Pipeline syntax__.
- __Stage__ block defines a conceptually distinct subset of tasks performed through the entire Pipeline (e.g. "Build", "Test" and "Deploy" stages), which is used by many plugins to visualize or present Jenkins Pipeline status/progress. 
- __Step__ is a single task. Fundamentally, a step tells Jenkins what to do at a particular point in time. For example, to execute the shell command make use the sh step: `sh 'make'`.

# Declarative pipeline
```
pipeline { 
    agent any  
    stages { 
        stage('Build') {  
            steps {  
                sh 'make'  
            } 
        } 
        stage('Test'){ 
            steps { 
                sh 'make check' 
                junit 'reports/**/*.xml'  
            } 
        } 
        stage('Deploy') { 
            steps { 
                sh 'make publish' 
            } 
        } 
    } 
} 
```

In Declarative Pipeline syntax, the pipeline block defines all the work done throughout your entire Pipeline

Code description:

- Execute this Pipeline or any of its stages, on any available agent.
- Defines the "Build" stage.
- Perform some steps related to the "Build" stage.
- Defines the "Test" stage.
- Perform some steps related to the "Test" stage.
- Defines the "Deploy" stage.
- Perform some steps related to the "Deploy" stage.

# Scripted pipeline
```
node {
    stage('Build') {
        sh 'make'
    }

    stage('Test') {
        sh 'make check'
        junit 'reports/**/*.xml'
    }

    stage('Deploy') {
        sh 'make publish'
    }
}
```

In Scripted Pipeline syntax, __one or more node blocks__ do/es the core work throughout the entire Pipeline. Although this is not a mandatory requirement of Scripted Pipeline syntax, confining your Pipeline’s work inside of a node block does two things:

1. Schedules the steps contained within the block to run by adding an item to the Jenkins queue. As soon as an executor is free on a node, the steps will run.
1. Creates a workspace (a directory specific to that particular Pipeline) where work can be done on files checked out from source control.

Code description:

- Execute this Pipeline or any of its stages, on any available agent.
- Defines the "Build" stage. __stage blocks are optional__ in Scripted Pipeline syntax. However, implementing stage blocks in a Scripted Pipeline provides clearer visualization of each stage's subset of tasks/steps in the Jenkins UI.
- Perform some steps related to the "Build" stage.
- Defines the "Test" stage.
- Perform some steps related to the "Test" stage.
- Defines the "Deploy" stage.
- Perform some steps related to the "Deploy" stage.

> both __stages__ and __steps__ are common elements of __both Declarative and Scripted Pipeline syntax__

# Blue ocean
is a suite of plugins, not installed by default. Provides a new UI for Jenkins

- Blue Ocean UI helps you set up your Pipeline project, and automatically creates and writes your Pipeline (i.e. the Jenkinsfile) for you through the graphical Pipeline editor.
- As part of setting up your Pipeline project in Blue Ocean, Jenkins configures a secure and appropriately authenticated connection to your project’s source control repository. Therefore, __any changes you make to the Jenkinsfile via Blue Ocean’s Pipeline editor are automatically saved and committed to source control__.

# Classic UI

A Jenkinsfile created using the classic UI __is stored by Jenkins itself__ (within the Jenkins home directory).

- From the Jenkins home page, click New Item at the top left.
- Specify the name for your new Pipeline project. Scroll down and click Pipeline, then click OK.
- Click the Pipeline tab at the top of the page to scroll down to the Pipeline section.
- In the Pipeline section, ensure that the Definition field indicates the Pipeline script option.
- Enter your Pipeline code into the Script text area.
- Click Save to open the Pipeline project/item view page.

Additional comments:

2. Caution: Jenkins uses this item name to create directories on disk. It is recommended to avoid using spaces in item names, since doing so may uncover bugs in scripts that do not properly handle spaces in directory paths.
3. Note: If instead you are defining your Jenkinsfile in source control, follow the instructions in In SCM below.
5. Note: You can also select from canned Scripted Pipeline examples from the try sample Pipeline option at the top right of the Script text area. Be aware that there are no canned Declarative Pipeline examples available from this field.

# SCM

1. Complex Pipelines are difficult to write and maintain within the classic UI’s Script text area of the Pipeline configuration page.
1. To make this easier, your Pipeline’s Jenkinsfile can be written in a text editor or integrated development environment (IDE) and committed to source control. 1. Jenkins can then check out your Jenkinsfile from source control as part of your Pipeline project’s build process and then proceed to execute your Pipeline.
1. Note: Since Pipeline code is written in Groovy-like syntax, if your IDE is not correctly syntax highlighting your Jenkinsfile, try inserting following line at the top of the Jenkinsfile, which may rectify the issue: `#!/usr/bin/env groovy`

# Shared Libraries
Pipeline has support for creating "Shared Libraries" which can be defined in external source control repositories and loaded into existing Pipelines.


The directory structure of a Shared Library repository is as follows:
```
(root)
+- src                     # Groovy source files
|   +- org
|       +- foo
|           +- Bar.groovy  # for org.foo.Bar class
+- vars
|   +- foo.groovy          # for global 'foo' variable
|   +- foo.txt             # help for 'foo' variable
+- resources               # resource files (external libraries only)
|   +- org
|       +- foo
|           +- bar.json    # static helper data for org.foo.Bar
```
- The __src__ directory should look like standard Java source directory structure. This directory is added to the classpath when executing Pipelines.
- The __vars__ directory hosts script files that are exposed as a variable in Pipelines. The name of the file is the name of the variable in the Pipeline. So if you had a file called vars/log.groovy with a function like `def info(message)…​` in it, you can access this function like `log.info "hello world"` in the Pipeline. You can put as many functions as you like inside this file. 
- The basename of each `.groovy` file should be a Groovy (~ Java) identifier, conventionally `camelCased`. The matching `.txt`, if present, can contain documentation, processed through the system’s configured markup formatter (so may really be HTML, Markdown, etc., though the `.txt` extension is required). This documentation will only be visible on the Global Variable Reference pages that are accessed from the navigation sidebar of Pipeline jobs that import the shared library. In addition, those jobs must run successfully once before the shared library documentation will be generated.
- The Groovy source files in these directories get the same “CPS transformation” as in Scripted Pipeline.
- A __resources__ directory allows the `libraryResource` step to be used from an external library to load associated non-Groovy files. Currently this feature is not supported for internal libraries.

## Global Shared Libraries

Manage Jenkins » Configure System » Global Pipeline Libraries

- will be globally usable, any Pipeline in the system can utilize functionality implemented in these libraries.
- are considered "trusted:" they can run any methods in Java, Groovy, Jenkins internal APIs, Jenkins plugins, or third-party libraries. This allows you to define libraries which encapsulate individually unsafe APIs in a higher-level wrapper safe for use from any Pipeline. Beware that anyone able to push commits to this SCM repository could obtain unlimited access to Jenkins.

##Folder-level Shared Libraries
- Any Folder created can have Shared Libraries associated with it. This mechanism allows scoping of specific libraries to all the Pipelines inside of the folder or subfolder.
- Folder-based libraries are not considered "trusted:" they run in the Groovy sandbox just like typical Pipelines.

## Automatic Shared Libraries

Other plugins may add ways of defining libraries on the fly. For example, the GitHub Branch Source plugin provides a "GitHub Organization Folder" item which allows a script to use an untrusted library such as github.com/someorg/somerepo without any additional configuration. In this case, the specified GitHub repository would be loaded, from the master branch, using an anonymous checkout.

##Using shared libraries

- Shared Libraries marked with `Load implicitly` flag (in Global Pipeline Libraries settings) allows Pipelines to immediately use classes or global variables defined by any such libraries. 
- To access other shared libraries, the Jenkinsfile needs to use the `@Library` annotation, specifying the library’s name:
```
@Library('my-shared-library') _
/* Using a version specifier, such as branch, tag, etc */
@Library('my-shared-library@1.0') _
/* Accessing multiple libraries with one statement */
@Library(['my-shared-library', 'otherlib@abc1234']) _
```