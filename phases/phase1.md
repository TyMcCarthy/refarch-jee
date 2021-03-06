# Phase 1 - Modernizing the Existing Application

WebSphere Application Server as a Service on IBM Bluemix allows existing IBM customers running WebSphere applications on-premise and new IBM customers to run these very same WebSphere applications and workloads on the IBM Cloud. However, this WebSphere Application Server as a Service on Bluemix supports only WebSphere Application Server version 8.5.5 onwards. As a result, prescriptive WebSphere guidance for modernizing WebSphere applications has been delivered and it is implemented through the following four steps:

1.  [Plan](#plan)
    * [Migration Strategy Tool](#migration-strategy-tool)
    * [Migration Discovery Tool](#migration-discovery-tool)
2.  [Assess](#assess)
    * [Evaluation](#evaluation)
    * [Inventory](#inventory)
    * [Analysis](#analysis)
3.  [Source Migration](#source-migration)
    * [Development Environment](#development-environment)
    * [Code Migration](#code-migration)
4.  [Config Migration](#config-migration)
    * [Traditional WebSphere Applications](#traditional-websphere-applications)

This readme aims to drive readers through the WebSphere guidance for modernizing WebSphere applications process by using our Customer Order Services reference application for the transition from [Customer Order Services for WAS 7.0](https://github.com/ibm-cloud-architecture/refarch-jee-customerorder/tree/was70-dev) to [Customer Order Serivces for WAS 9.0](https://github.com/ibm-cloud-architecture/refarch-jee-customerorder/tree/was90-dev).

## Plan

Moving your application to newer versions of WebSphere requires proper planning. The following resources helps you to plan successfully.

### Migration Strategy Tool

Before moving to a new version of Websphere, you need to know all the options available for your migration. This tool helps you in evaluating different options available for your WebSphere applications.

![Migration Strategy Tool-1](/phases/phase1_images/StrategyTool1.png?raw=true)
![Migration Strategy Tool-2](/phases/phase1_images/StrategyTool2.png?raw=true)

This tool can be accessed at http://whichwas.mybluemix.net/ 

### Migration Discovery Tool

This tool helps in migrating your applications to WebSphere by estimating the effort required. This tool estimates the size as well as the scope of migration. Planning and budget assumptions can be done. Based upon the answers to the questionnaire, historical data and effort hours, this tool generates a rough estimation for migration.

![Migration Discovery Tool-1](/phases/phase1_images/DiscoveryTool1.png?raw=true)

The tool walks you through a couple of Questions related to the installation, application and testing. Based upon your answers, it generates a summary of the information provided, rough order of magnitude estimations, scope, timeline etc.

![Migration Strategy Tool-2](/phases/phase1_images/DiscoveryTool2.png?raw=true)

This tool can be accessed at http://ibm.biz/MigrationDiscovery.

### WAS V9 TCA Calculator (Subscription and Support Cost Calculator)

This [tool](https://advantage.ibm.com/2015/04/02/was-vs-jboss-license-and-support-cost-calculator-updated/) helps you in evaluating licensing options for various available Java application servers like IBM WebSphere, Oracle WebLogic, Red Hat JBoss and Pivotal tc Server and helps you to make the right choice among them. 

The Total Cost of Acquisition (TCA) calculator is a quick tool that helps you to compare your existing S&S costs against the latest WAS V9 financial benefits.

![Licensing Options](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_images/TCO_licensing.png)

![TCA Calculator](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_images/TCO_cost_est.png)

### WAS V9 TCO Calculator (On-Premise and In Cloud Cost Calculator)

This [tool](https://roi-calculator.mybluemix.net/) helps you to view all the initial projection costs for creating new applications or to move the existing applications to bluemix. This information helps you to decide whether to keep your application on-premise or to move it to Bluemix. 

![roi-calculator](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_images/TCO_cost_calculator.png)

## Assess

Assessing your application helps you to understand all the potential issues during the migration process. During this phase, we will evaluate the programming models in our application and what WebSphere products support them. We will use the [WebSphere Migration Toolkit for Application Binaries](https://developer.ibm.com/wasdev/downloads/#asset/tools-Migration_Toolkit_for_Application_Binaries) and its [manual](https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/wasdev/downloads/wamt/ApplicationBinaryTP/MigrationToolkit_Application_Binaries_en_US.pdf) for this process.

### Evaluation

The Migration Toolkit for Application Binaries can generate an evaluation report that helps to evaluate the Java
technologies that are used by your application. The report gives a high-level review of the programming
models found in the application, the WebSphere products that support these programming models and the right-fit IBM platforms for your JEE application.

To get the report, run `java -jar binaryAppScanner.jar binaryInputPath --evaluate`

_(I) binaryInputPath, which is an absolute or relative path to a JEE archive file or directory that contains JEE archive files_

![Evaluation Report](/phases/phase1_images/EvaluationReport.png?raw=true)

As we can see in the picture above, our Customer Order Services app fit a numerous IBM platforms such as the new Liberty profile either on-premises and on Bluemix as well as the traditional WebSphere server and deployment.

### Inventory

The Migration Toolkit for Application Binaries can generate an inventory report that contains a high-level inventory of the content and structure of each application and information about potential deployment problems. The report includes a dashboard that reports the number and types of archives found. The counts of Java Servlets, JSP files, JPA entities, EJB beans, and web services are reported per application as well as in an overall summary. For each archive, the Java EE module type and version are included. And for utility JAR files, the contained packages are reported up to and including the third sub package.

The report also includes a section for each application about potential deployment problems. The tool detects the following issues:

* Duplicate classes within the application
* Java EE or SE classes that are packaged with the application
* Open Source Software (OSS) classes that are packaged with the application
* WebSphere classes that are packaged with the application
* Unused archives that are packaged with the application
* Dependent classes that are not packaged with the application
* Shared libraries referencing application classes

To get the report, run `java -jar binaryAppScanner.jar binaryInputPath --inventory`

_(I) binaryInputPath, which is an absolute or relative path to a JEE archive file or directory that contains JEE archive files_

[**Initial Report**](http://htmlpreview.github.com/?https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_reports/CustomerOrderServicesApp.ear_InventoryReport_Initial.html)

As we can see in the report, there are several concerning facts such as duplicate or unused libraries we should pay attention to and which we higly recommend fixing before migrating your application to a newer WebSphere Application Server version and/or different IBM platform. Hence, we will go through the process of fixing these potential future problems by making use of this inventory report.

Firstly, we see a summary section which might already raise concerns in us such as the high amount of utility JARs found in the JEE archive of our Customer Order Services application. As we scroll down, we rapidly get to the _Inventory Details by Application_ where we find out that there are several libraries being marked as duplicate or unused which we certainly need to look at:

![Inventory report](/phases/phase1_images/inventory_report/Inventory1.png?raw=true)

After removing the unused libraries and verifying that this does not introduce any other problem/error in the code, we look at the ear structure, its ejb, web and test projects. We realise that there are several libraries being used by multiple projects but yet included in each of them as opposed to packaging them at the ear level as a shared library among all ear project components. After repackaging libraries at the ear level, this is how the inventory report looks like:

![Inventory report 2](/phases/phase1_images/inventory_report/Inventory2.png?raw=true)

We see now that both web projects and the ejb project look fine library wise. However, the report indicates that there are still several duplicate libraries. Looking in detail at what packages those duplicate libraries come with, we realise that, in effect, several libraries packages are included by others (normally broader and heavier Java libraries. For instance, the _jackson-all_ library includes all packages the other three lighter and more specific jackson libraries come with). We then remove remaining duplicate libraries and run the report again:

![Inventory report 3](/phases/phase1_images/inventory_report/Inventory3.png?raw=true)

Now, we do not see any more duplicate libraries. However, we still see warnings due to libraries we are packaging our application with which either include Open Source Software, Java EE or SE classes or WebSphere system classes. Reading at the rules explanation, we are suggested to review what libraries our WebSphere Application Server plus Java Runtime Environment come with since they might already be included:

![Inventory report 4](/phases/phase1_images/inventory_report/Inventory4.png?raw=true)

It is important to use, as much as possible, the libraries that already come with your middleware stack so that we avoid class loading issues. Hence, we look at the WebSphere Application Server and Java Runtime Environment libraries in our environment and we effectively verify that we were including libraries again that were already packaged with our WebSphere Application Server version:

![Inventory report 5](/phases/phase1_images/inventory_report/Inventory5.png?raw=true)

Again, we remove these libraries and run the report. This time, the report comes finally clean except from some warnings regarding missing dependencies on some projects/libraries.

![Inventory report 6](/phases/phase1_images/inventory_report/Inventory6.png?raw=true)
![Inventory report 7](/phases/phase1_images/inventory_report/Inventory7.png?raw=true)

Looking at those, we find out that libraries reporting missing dependencies for the ejb, test and web project are not, in fact, missing so then we should be good there. Classes reported are missing in the _DBUnit.jar_ library are never used so we are good there too.

Finally, we move the _DBunit.jar, junit.jar and junitee.jar_ libraries back into the test project which is where they are needed. No other project uses them so no need for keeping them at the ear level. In fact, due to class loading order and scope, if the _junitee.jar_ librarie is not placed at the test project level, the JUnitEE Servlet might not find the test cases to be run. Hence, this is the final look of our application library/structure wise:

![Inventory report 8](/phases/phase1_images/inventory_report/Inventory8.png?raw=true)

We have repackaged and refactor our application structure to a situation were we do not have duplicate or unused libraries and the libraries we want/need to include are packaged in the proper location. At this point, we are now ready to start the migration to the newer WebSphere Application Server version or IBM Platform.

[**Final Report**](http://htmlpreview.github.com/?https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_reports/CustomerOrderServicesApp.ear_InventoryReport_Final.html)

### Analysis

The Migration Toolkit for Application Binaries can generate an analysis report that contains details about potential migration issues in your application. The tool helps you quickly and easily evaluate WebSphere Application Server traditional applications for their readiness to run on Liberty in both cloud and on-premises environments. The tool also supports migrating between versions of WebSphere traditional, starting with WebSphere Application Server Version 6.1.

The report gives details about which rules were flagged for your application binaries. For each flagged result, the report lists the affected file along with the match criteria, the method name if applicable, and the line number if available. Line numbers are only available for results that occur within a method body.

This analysis report should flag the same issues as the following Eclipse WebSphere Migration Toolkit plugin (Source Migration section). However, the analysis report comes with a show rule option where the user can read the rule explanation.

To get the report, run `java -jar binaryAppScanner.jar binaryInputPath --analyze [OPTIONS]`

_(I) binaryInputPath, which is an absolute or relative path to a JEE archive file or directory that contains JEE archive files_

_(II) in our case, the [OPTIONS] are: --sourceAppServer=was70 --targetAppServer=was90 --sourceJava=ibm6 --targetJava=ibm8
--targetJavaEE=ee7_

[**Report**](http://htmlpreview.github.com/?https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_reports/CustomerOrderServicesApp.ear_AnalysisReport.html)

In this first phase, the goal is to 'lift & shift' our WebSphere applications with the minimum effort/change. As a result, we will only look at the errors reported which in our case are:

![Analyze report](/phases/phase1_images/analyze_report/Analyze1.png?raw=true)

Again, the Migration Toolkit for Application Binaries Analyze report is used for assessment. That is, its main goal is to *quantify the migration effort*. It should not be used for repackaging the application or make code changes. For doing so, we are going to use the Eclipse WebSphere Migration Toolkit plugin in the next section.

## Source Migration

### Development Environment

In order to carry out the migration of your source code from an old WebShere Application Server version to a newer one, we must set up a proper development environment for the task. In this case, we want to move away from IBM proprietary tools as much as possible. Hence, this is the stack of our development environment:

* [Eclipse Mars 2 for JEE developers](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/mars2)
* [IBM WebSphere Application Server V9.x Developer Tools for Eclipse](https://marketplace.eclipse.org/content/ibm-websphere-application-server-v9x-developer-tools#group-details)
* [IBM Installation Manager](http://www-01.ibm.com/support/docview.wss?uid=swg27025142) - (Recommended for the installation of the IBM WebSphere Application Server V9.0)
* IBM WebSphere Application Server V9.0

To install the IBM WebSphere Application Server V9.x Developer Tools for Eclipse:

1. Open eclipse.
2. Drag and drop the Install button into the eclipse main window.
3. Select all the options available.

![Source report 15](/phases/phase1_images/source_report/Source15.png?raw=true)

To install the IBM WebSphere Application Server V9.0:

1. Open the IBM Installation Manager.
2. Click on File --> Preferences...
3. Add the following repository: https://www.ibm.com/software/repositorymanager/V9WASILAN
4. Install the IBM WebSphere Application Server V9.0 with its latest fixpack and the IBM Java SDK Version 8. (There is no need to install Liberty for now).

![Source report 14](/phases/phase1_images/source_report/Source14.png?raw=true)

### Code Migration

The migration toolkit provides a rich set of tools that help you migrate applications from third-party application servers, between versions of WebSphere Application Server, to Liberty, and to cloud platforms such as Liberty for Java on IBM Bluemix, IBM WebSphere on Cloud and Docker.

To install the eclipse-based WebSphere Application Server Migration Toolkit plugin, follow the instructions on the previous [Development Environment](#development-environment) section. Once it has been installed, create a new _Software Analyzer Configuration_ with your migration specifications. In our case, we set the configuration to a WAS 7.0 to WAS 9.0 migration taking JPA and JAX-RS into account.

![Source report 1](/phases/phase1_images/source_report/Source1.png?raw=true)

After running the _Software Analyzer_ you should see a _Software Analyzer Results_ tab at the bottom. In here, we should have the exact same errors and warnings as the _Analyzer report_ in the previous section. The Software Analyzer rules are categorised and so are the errors and warnings produced in its report. As you can see in the image, one warning that will always appear in when you migrate your apps to a newer WebSphere Application Server version is the need to configure the appropriate target runtime for your applications. This is the first and foremost step:

![Source report 2](/phases/phase1_images/source_report/Source2.png?raw=true)

In addition to this, you should also remove old WebSphere Application Server specific libraries. In our case, we should remove the reference to the _IBM WebSphere Application Server traditional V7.0 JAX-RS Library_ which was installed as a Feature pack:

![Source report 3](/phases/phase1_images/source_report/Source3.png?raw=true)

Setting the appropriate target runtime and removing previous WebSphere Application Server version specific libraries might introduce problems in the code:

![Source report 4](/phases/phase1_images/source_report/Source4.png?raw=true)

At this point, we must look at the other errors reported by the WebSphere Application Server Migration Toolkit since they often are because of previous WebSphere Application Server version specific libraries not available (initially) in the new WebSphere Application Server version. In our case, we see that the problems in the code are due to the _org.apache.wink_ library missing which is something the Analyze report and the Migration Toolkit also flag:

![Source report 5](/phases/phase1_images/source_report/Source5.png?raw=true)

As we said previously, the Migration Toolkit for Application Binaries Analyze report is used for assessment. That is, its main goal is to *quantify the migration effort*. It should not be used for repackaging the application or make code changes. Instead, we should use the Eclipse based WebSphere Application Server Migration Toolkit plugin for the source code migration as it should present the user with, at least, the same information as the Migration Toolkit for Application Binaries Analyze report presents the user with.

Among other things, the advantage of using the Eclipse based WebSphere Application Server Migration Toolkit plugin is, in fact, because it is used in an IDE so you can take advantage of using IDE's suggestions, tools and help:

![Source report 6](/phases/phase1_images/source_report/Source6.png?raw=true)

If you want to read the explanation of the rule being applied (and the potential solution it sometimes comes along with) you have to open the help view (window => show view => other => help => help). If you scroll down to the bottom and press on the "detailed help" button it will show you additional ideas on how to resolve that problem:

![Source report 16](/phases/phase1_images/source_report/Source16.png?raw=true)
![Source report 17](/phases/phase1_images/source_report/Source17.png?raw=true)

We have to be careful and thoroughly review each of the errors and warning either of the toolkits report since there might be some that actually do not apply due to the possibility of having WebSphere Application Server fixpacks or feature packs installed. For instance, in our case, both toolkits report an error because we are using the _org.codehaus.jackson_ packages that are exposed as a third-party API in JAX-RS 1.1 but are not part of the new JAX-RS 2.0 version WebSphere Application Server V9.0 comes initially with. However, we find out that our WebSphere Application Server V9.0 comes with such packages possibly because of later fix packs installed as we are really using the 9.0.0.3 version of WebSphere Application Server:

![Source report 8](/phases/phase1_images/source_report/Source8.png?raw=true)

The last error reported is on the persistence.xml file for the JPA version used in our code. In the WebSphere Application Server V7.0, where we are migrating our code from, we were using its Feature Pack for OSGI and JPA which comes with the JPA 2.0 version for which OpenJPA is its provider. In contrast, the WebSphere Application Server V9.0, which we want to migrate our code to, uses the JPA 2.1 version for which eclipselink is its provider. Then, the problem is that some JPA annotations and attributes declared in the persistence.xml file are OpenJPA proprietary:

![Source report 9](/phases/phase1_images/source_report/Source9.png?raw=true)
![Source report 10](/phases/phase1_images/source_report/Source10.png?raw=true)


However, new WebSphere Application Server versions should have backwards compatibility in most of the JavaEE technologies and turns out that JPA is one of those. Remember that our goal is to migrate our existing code/apps running on WebSphere Application Server V7.0 to run on a newer WebSphere Application Server version supported by WASaaS on Bluemix but with the minimum possible changes which is what the 'lift & shift' pattern consist of (modernization of your application, Java technologies wise, might be carried out on a later stage). Because of this, we can take advantage of the backwards compatibility on the JPA version and providers the new WebSphere Application Server V9.0 comes with and configure it to use JPA 2.0 so that we do not need to change anything in our app at all:

![Source report 11](/phases/phase1_images/source_report/Source11.png?raw=true)

We have finally review all the errors reported by either the Migration Toolkit for Application Binaries or the eclipse-based WebSphere Application Server Migration Toolkit plugin. Now, the application and code should be deeply and carfully tested in order to make sure that the migration to a newer WebSphere Application Server version has not changed its behaviour at all.

As an example, WebSphere Application Server V9.0, which we have migrated our code to work on, uses a newer version of the openjpa driver (V2.2.3) which handles booleans at the persistence layer differently. As a result, when we test our migrated code we see the following exceptions being thrown in the WebSphere Application Server log files:

![Source report 12](/phases/phase1_images/source_report/Source12.png?raw=true)

We then make the following changes at the persistence layer for the appropriate POJO classes so that booleans are converted to strings beforehand the persistence layer (or OpenJPA driver) takes over and viceversa:

![Source report 13](/phases/phase1_images/source_report/Source13.png?raw=true)

Hence, **the more unit test, integration test and test in general** your application and code had already implemented and come with for the old WebSphere Application Server version, **the better, easier and trustworthy will this migration verification and validation process be.**

## Config Migration

Migration of configuration should be done before deploying the application on the latest versions of WebSphere. The configurations can be migrated from traditional WebSphere application servers or from third party application servers too. 

Below are the two tools available for Configuration migration.

1. [Migration Wizard](https://www.ibm.com/support/knowledgecenter/en/SSAW57_9.0.0/com.ibm.websphere.migration.nd.doc/ae/tmig_profiles_gui.html)
2. [Eclipse based configuration migration tool]( https://developer.ibm.com/wasdev/downloads/#asset/tools-WebSphere_Configuration_Migration_Tool)

### Traditional WebSphere (Profile Migration)

Full profile migration can be done using WASPreUpgrade and WASPostUpgrade commands as well using the graphical user interface, the Configuration Migration Tool.

#### Cloning the profile using the command line

WASPreUpgarde and WASPostUpgrade commands can be used to perform the migration using the command line.

**WASPreUpgrade** - This allows you to backup all the configuration data from the source profile into a migration backup directory.

**WASPostUpgrade** - This allows you to merge all the source data into the new target profile.

`./WASPreUpgrade.sh /home/vagrant/IBM/WASMigration /home/vagrant/IBM/WebSphere/AppServer/ -oldProfile AppSrv01 -username admin -password password`

![WASPreUpgrade](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_images/WASConfig/WASPreUpgarde.png)

`./manageprofiles.sh -create -profileName AppSrv01 -profilePath /home/vagrant/IBM/WebSphere/AppServer_2/profiles/AppSrv01 -templatePath /home/vagrant/IBM/WebSphere/AppServer_2/profileTemplates/default -defaultPorts -enableAdminSecurity true -adminUserName admin -adminPassword password`

![WASProfileManagement](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_images/WASConfig/WASProgileMgt.png)

`./WASPostUpgrade.sh /home/vagrant/IBM/WebSphere/WSMigration -username admin -password password -oldProfile AppSrv01 -profileName AppSrv01 -setPorts generateNew -resolvePortConflicts incrementCurrent -includeApps false -clone true`

![WASPostUpgrade](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_images/WASConfig/WASPostUpgrade.png)

#### Cloning the profile using the Configuration MigrationTool

Version to version configuration migration support is incorporated within the WebSphere Application Server itself. There is an inbuilt Configuration Migration Tool within the server and this tool walks you through the migration wizard which helps you in the migration process.

IBM WebSphere Application Server V9.0 contains a WebSphere Customization toolbox.

![WebSphere Customization ToolBox1](/phases/phase1_images/TWAS1.png?raw=true)

Create a migration using the New option available. This guide you through several steps and finally returns the migration summary. Verify the summary and proceed with the migration.

![WebSphere Customization ToolBox2](/phases/phase1_images/TWAS2.png?raw=true)

Once the migration is completed, it returns you the migration results. Check them out and complete the migration using Finish option.

![WebSphere Customization ToolBox3](/phases/phase1_images/TWAS3.png?raw=true)

![WebSphere Customization ToolBox4](/phases/phase1_images/TWAS4.png?raw=true)

This procedure can be followed only if the migration is between different versions of WebSphere.

The above tools are best suited for the scenarios where the migration of cells is similar. They are great where the target cell has the same number of node agents, the same number of clusters, the same number of cluster members, etc. 

These tools do not work well if there is some consolidation or other re-architecture of their topologies. When there is refactoring of the topologies then you can make use of Eclipse WebSphere Configuration Migration tool(WCMT).

### Traditional WebSphere (Resource Migration)

[Eclipse-based WebSphere Configuration Migration tool](https://developer.ibm.com/wasdev/downloads/#asset/tools-WebSphere_Configuration_Migration_Tool) can be used to migrate the configuration from different types of servers to WebSphere application server. This tool supports Traditional WebSphere Applications as well as the Third-party server applications.

Once this tool gets installed in your IDE, you can access it using the option Migration Tools.

![Eclipse Plugin1](/phases/phase1_images/EclipsePlugin1.png?raw=true)

Choose your environment and proceed with the migration.

![Eclipse Plugin2](/phases/phase1_images/EclipsePlugin2.png?raw=true)

`./wsadmin.sh –lang jython –c “AdminTask.extractConfigProperties(‘[-propertiesFileName my.props]’)”`

After executing this command, my.props file will be generated. The properties file for this sample application can be accessed here [my.props](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_assets/my.props) 

![Eclipse Plugin3](/phases/phase1_images/EclipsePlugin3.png?raw=true)

![Eclipse Plugin4](/phases/phase1_images/EclipsePlugin4.png?raw=true)

![Eclipse Plugin5](/phases/phase1_images/EclipsePlugin5.png?raw=true)

At this step, save your configuration file and Finish the migration. By the end, you will have jython script saved as the configuration file. 

The configuration file generated for this sample application can be accessed here [migConfig.py](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/phases/phase1_assets/migConfig.py) 

Finally stop the source server.

Now, start your target server. 

Go to the **<WAS_PROFILE_DIR>/bin**, and use the following command.

`<Profile Home>/bin/wsadmin.(bat/sh) –lang jython –f <Location of Jython script>`

![Eclipse Plugin6](/phases/phase1_images/EclipsePlugin6.png?raw=true)

Once the script gets executed successfully, the migration is complete.

![Eclipse Plugin7](/phases/phase1_images/EclipsePlugin7.png?raw=true)

You can verify the migration by opening your admin console and then check if all the resources are correct.


## Other useful links

* [Migrating to WAS 8.5.5](http://www.redbooks.ibm.com/abstracts/sg248048.html) - (Redbook)
* [Migration](https://developer.ibm.com/wasdev/docs/migration/) - (developerWorks Blog)
* [IBM WebSphere Application Server Migration Toolkit](https://www.ibm.com/developerworks/library/mw-1701-was-migration/index.html) - (developerWorks Blog)
* [WebSphere Migration Knowledge Collection](http://www-01.ibm.com/support/docview.wss?uid=swg27008728)
