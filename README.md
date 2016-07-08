# Workshop: Pivotal Cloud Foundry Development

## Goals

To deploy and configure a microservice, leverage the platform for monitoring & management of the microservice, and to create services that will be bound to the application.

## Prerequisites

1) Install the CF CLI
Download and install the Cloud Foundry Command Line Interface (cf CLI): [Download](https://github.com/cloudfoundry/cli#downloads) and make sure it works: `cf help`

2) Make sure you have Java 7 (or later) installed and configured [Java](https://java.com/en/download/)

## Deploy the application

1) Clone the sample application 
```
$ git clone https://github.com/cloudfoundry-samples/spring-music
```

If you don't have Git installed, you can download a zip file of the app at github.com/cloudfoundry-samples/spring-music/archive/master.zip. Then navigate to the App directory `cd ./spring-music`.

2) Login to Pivotal Web Services with the credentials given

```bash
$ cf login -a https://api.run.pivotal.io
API endpoint:  api.run.pivotal.io  
Email>     user
Password>  pass
```

3) Use Gradle to assemble the app locally:

```bash
./gradlew assemble
```

4) Push the application

```bash
$ cf push
```

5) Get the url from the response given on the commandline and open the sample app in your browser.

Congratulations! You have successfully pushed your first application to Pivotal Cloud Foundry!


## Log Aggregation

PCF provides access to an aggregated view of logs related to you application. This includes HTTP access logs, as well as output from app operations such as scaling, restarting, and restaging.

To view your recent logs use

```bash
$ cf logs spring-music --recent
```

or to stream them constantly (live) use

```bash
$ cf logs spring-music
```

Reload the app page to see activity. Press Control C to stop streaming.

More on logs can be found at [Streaming Logs](http://docs.pivotal.io/pivotalcf/1-7/devguide/deploy-apps/streaming-logs.html)

## Databases

If  a database isn't available, the sample app uses a temporary in-memory database. This app supports MySQL, Postgres, Redis, and MongoDB.

You can see what type of database this app is using by clicking the info icon in the upper right corner of your app.

In this step we’ll connect to a MySQL Database to allow persistence.

### Listing Plans

Pivotal Cloud Foundry has a built-in marketplace where you may obtain services as needed. To display available services use

```bash
$ cf marketplace
```

As we want to use a MySQL Database, let’s display which plans are available.

```bash
$ cf marketplace -s p-mysql
```

We’ll create a service using the free plan for MySQL.

```bash
$ cf create-service p-mysql 512mb my-spring-db
```

To make the credentials available to the application we pushed, we need to bind this service to the application.

```bash
$ cf bind-service spring-music my-spring-db
```

Once this this bound to the app, environment variables are stored that allow the app to connect to the service after a push, restage or restart command.

Let’s restart the app.

```bash
$ cf restart spring-music
```

To verify the new service is bound to the app, you can either click on the top right info icon in the upper right corner of the sample app or check using this command:

```bash
$ cf services
Getting services in org your-org / space development ...
OK
name             service   plan    bound apps
spring-music-db  p-mysql   512mb   spring-music
```

More on services can be found at [Managing Services](http://docs.pivotal.io/pivotalcf/1-7/devguide/services/managing-services.html)


## Scaling the app

As your application is running and gets more and more consumed by your customers, it’s time to scale your application to serve your app as needed.

Scaling your app horizontally adds or removes app instances. Adding more instances allows your application to handle increased traffic and demand.

Increase the number of app instances from one to two:

```bash
$ cf scale spring-music -i 2
```

Check the status of the app and verify there are two instances running:

```bash
$ cf app spring-music
...
state       since                  cpu  memory
#0 running  2016-02-23 10:55:08 AM 0.1% 461M of 512M
#1 running  2016-02-23 01:14:59 PM 0.0% 455.1M of 512M
```

Scaling your app vertically changes the disk space limit or memory limit for each app instance.

Increase the memory limit for each app instance:

```bash
$ cf scale spring-music -m 1G
```

Increase the disk limit for each app instance:

```bash
$ cf scale spring-music -k 512M
```

More on this can be found at [Scaling an Application](http://docs.pivotal.io/pivotalcf/1-7/devguide/deploy-apps/cf-scale.html)

### Killing one instance (optional)

Now that we have two instances running, we might want to check if the automatic restart works if we kill one instance. The spring-music application provides a route which will let you kill the instance, and check if Pivotal Cloud Foundry will properly restart it.

Visit your application (be sure to replace `something` with your random route) at 

```
http://spring-music-something.cfapps.io/errors/kill
```

and you will see the application being restarted by the Elastic Runtime. As you have two instances by now, the `http://spring-music-something.cfapps.io` (again, be sure to replace `something` with your random route) should still return your application, only the killed instance will be restarted.


## Additional Steps

If you’re done with this introduction, make sure you fully understand the principles of working with Pivotal Cloud Foundry. However if you want to, you might want to check out [User Provided Services](http://docs.pivotal.io/pivotalcf/1-7/devguide/services/user-provided.html)

This will let you create and bind a service provided by you to an app of your choice. If you have a Papertrail Account, try if you can bind this application to your app and see your logs there!

Hints: 

```bash
cf help cups
```

will provide you with the syntax for binding a log aggragation service to your application.

More on this can be found at [User-Provided Service Instances](http://docs.pivotal.io/pivotalcf/1-7/devguide/services/user-provided.html)
