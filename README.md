# Workshop: Pivotal Cloud Foundry Development

## Goals

To deploy and configure a microservice, leverage the platform for scaling, monitoring & management, and create services that will be bound to the deployed application.

## Prerequisites

1) Install the CF CLI
Download and install the Cloud Foundry Command Line Interface (CF CLI): [Download](https://github.com/cloudfoundry/cli#downloads) and make sure it works: `cf help`

**Note:** If you don't have admin priviledges on your machine, [download](https://github.com/cloudfoundry/cli/releases) the appropriate binary.

2) *Optional* install and configure Java 7 (or later) https://java.com/en/download

## Deploy the application to Cloud Foundry
#### Option A - Clone from a github repository and build the artifact manually
A-1) Clone the sample application 
```
$ git clone https://github.com/cloudfoundry-samples/spring-music
```
A-2) Use Gradle to assemble the app locally:

```bash
./gradlew assemble
```
If you don't have Git installed, you can download a zip file of the app at github.com/cloudfoundry-samples/spring-music/archive/master.zip. Then navigate to the App directory `cd ./spring-music`.

#### Option B - Download the prebuilt artifact
B-1) [Download](https://s3.eu-central-1.amazonaws.com/pnagel/workshop/spring-music.zip) the zip file and extract it. Then navigate to the App directory `cd ./spring-music`.

#### Push die application to Cloud Foundry
1) Login to [Pivotal Web Services](http://run.pivotal.io/) with the credentials given

```bash
$ cf login -a https://api.run.pivotal.io
API endpoint:  api.run.pivotal.io  
Email>     user
Password>  pass
```

2) Push the application

Please give the app a name to identify it later on. E.g. *spring-music-pna* wheras *pna* is derived from **P**atrik **Na**gel. Replace *my_app_name* below accordingly, please.

```bash
$ cf push my_app_name
```

3) Get the url from the response given on the commandline and open the sample app in your browser.
```bash
requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: spring-music-pna-germproof-obedience.cfapps.io

     state     since                    cpu    memory         disk           details
#0   running   2016-07-10 09:49:06 AM   0.0%   426.7M of 1G   155.3M of 1G
```

Congratulations! You have successfully pushed your first application to Pivotal Cloud Foundry!

**Hint:** If you're wondering how the runtime to execute the Java app (Spring MVC) got built for you, have a look at the concept of [buildpacks](http://docs.pivotal.io/pivotalcf/1-7/buildpacks/index.html). More specifically, the [Java buildpack](https://github.com/cloudfoundry/java-buildpack). Cloud Foundry being a polyglot application platform (Java, node.js. ruby, php, go, python, etc.), you could also push a [docker image](http://docs.pivotal.io/pivotalcf/1-7/concepts/docker.html#push-docker).

## Scaling the app

As your application is running and gets more and more consumed by your customers, it’s time to scale.

Scaling your app horizontally adds or removes app instances. Adding more instances allows your application to handle increased traffic and demand.

Increase the number of app instances from one to two:

```bash
$ cf scale spring-music-pna -i 2
```

Check the status of the app and verify there are two instances running:

```bash
$ cf app spring-music-pna
...
state       since                  cpu  memory
#0 running  2016-02-23 10:55:08 AM 0.1% 461M of 512M
#1 running  2016-02-23 01:14:59 PM 0.0% 455.1M of 512M
```

If you access your application again via the web broswer, you will have a round robin load balancing across all your app instances automatically. Sceptical? Add `/env` to the URL and watch for `CF_INSTANCE_PORT` to change while you refresh.

**Hint:** Scaling is a matter of seconds, we don't need to re-stage the app. The original staged artifact called droplet has been stored on the platform's internal blob store already. More on scaling can be found at [Scaling an Application](http://docs.pivotal.io/pivotalcf/1-7/devguide/deploy-apps/cf-scale.html).

### Platform Integrated Application Recovery (optional)

Now that we have two instances running, we might want to check if the automatic recovery i.e. restart works in case we kill one of the running instances. The spring-music application provides an endpoint which will kill the process. Check afterwards if Pivotal Cloud Foundry will properly restart it and route traffic to the healthy instances only.

Open your application in a web browser again (be sure to replace `something` with your random route) at 

```
http://spring-music-something.cfapps.io/errors/kill
```

and you will see (`cf app spring-music-pna`) one application instance being restarted by the Elastic Runtime automatically. As you have two instances by now, the `http://spring-music-something.cfapps.io` (again, be sure to replace `something` with your random route) will still return your application as the traffic is only routed to healthy instances.

## Log Aggregation

PCF provides access to an aggregated view of logs related to you application. This includes HTTP access logs, as well as output from app operations such as scaling, restarting, and restaging.

To view your recent logs use

```bash
$ cf logs spring-music-pna --recent
```

or to get the live stream use

```bash
$ cf logs spring-music-pna
```

Reload the app page to see activity. Press `control-c` to stop streaming.

**Hint:** More on logs can be found at [Streaming Logs](http://docs.pivotal.io/pivotalcf/1-7/devguide/deploy-apps/streaming-logs.html)

## Marketplace and Services

Pivotal Cloud Foundry has a concept of a marketplace where various services can be consumed as needed and bound to an app. The used spring-music application uses a temporary h2 in-memory database by default. However, it also supports MySQL, Postgres, Redis, and MongoDB as a datasource to manage the music albums.

You can see what type of database the spring-music app is currently using by clicking the info icon in the upper right corner.

In the next step we’ll connect to a MySQL database to allow persistence.

### Listing Plans

Display the available services from the built-in marketplace.

```bash
$ cf marketplace
```

As we want to use a MySQL database, let’s display which plans are available.

```bash
$ cf marketplace -s cleardb
```

We’ll create a service using the **free** plan for MySQL. Please use again a unique identifier for the service instance name (*mysql-db-pna*). 

```bash
$ cf create-service cleardb spark mysql-db-pna
```

To make the credentials available to the application we pushed, we need to bind this service to the application. This injects the credentials and connection information into the app via environment variables.

```bash
$ cf bind-service spring-music-pna mysql-db-pna
```

Once bound to the app, environment variables are stored that allow the app to connect to the service after a push, restage or restart command.

Let’s restart the app.

```bash
$ cf restart spring-music-pna
```

To verify the new service is bound to the app, you can either click on the info icon in the upper right corner of the app or check using this command:

```bash
$ cf services
Getting services in org your-org / space development ...
OK
name             service   plan    bound apps
mysql-db-pna     cleardb   spark   spring-music-pna
```

**Hint:** More on services can be found at [Managing Services](http://docs.pivotal.io/pivotalcf/1-7/devguide/services/managing-services.html). Please note that the marketplace is extensible by your own services.

## Additional Steps

If you’re done with this introduction, make sure you fully understand the principles of working with Pivotal Cloud Foundry. However if you want to, you might want to check out [User Provided Services](http://docs.pivotal.io/pivotalcf/1-7/devguide/services/user-provided.html)

This will let you create and bind a service provided by you to an app of your choice. If you have a Papertrail Account, try if you can bind this application to your app and see your logs there!

Hints: 

```bash
cf help cups
```

will provide you with the syntax for binding a log aggragation service to your application.

More on this can be found at [User-Provided Service Instances](http://docs.pivotal.io/pivotalcf/1-7/devguide/services/user-provided.html)
