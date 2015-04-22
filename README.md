<<<<<<< HEAD
Spring Music
============

This is a sample application for using database services on [Cloud Foundry](http://cloudfoundry.com)
with the [Spring Framework](http://www.springframework.org).

This application has been built to store the same domain objects in one of a variety of different persistence technologies - relational, document, and key-value stores. This is not meant to represent a realistic use case for these technologies, since you would typically choose the one most applicable to the type of data you need to store, but it is useful for testing and experimenting with different types of services on Cloud Foundry. 

The application use Spring Java configuration and [bean profiles](http://static.springsource.org/spring/docs/current/spring-framework-reference/html/new-in-3.1.html#new-in-3.1-bean-definition-profiles) to configure the application and the connection objects needed to use the persistence stores. It also uses the [Spring Cloud](https://github.com/spring-projects/spring-cloud) library to inspect the environment when running on Cloud Foundry. See the [Cloud Foundry documentation](http://docs.cloudfoundry.com/docs/using/services/spring-service-bindings.html) for details on configuring a Spring application for Cloud Foundry.

## Running the application locally

One Spring bean profile should be activated to choose the database provider that the application should use. The profile is selected by setting the system property `spring.profiles.active` when starting the app.

The application can be started locally using the following command:

~~~
$ ./gradlew tomcatRun -Dspring.profiles.active=<profile>
~~~

where `<profile>` is one of the following values:

* `in-memory` (no external database required)
* `mysql`
* `postgres`
* `mongodb`
* `redis`

If no profile is provided, `in-memory` will be used. If any other profile is provided, the appropriate database server
must be started separately. The application will use the host name `localhost` and the default port to connect to the database.

If more than one of these profiles is provided, the application will throw an exception and fail to start.

## Running the application on Cloud Foundry

When running on Cloud Foundry, the application will detect the type of database service bound to the application
(if any). If a service of one of the supported types (MySQL, Postgres, Oracle, MongoDB, or Redis) is bound to the app, the
appropriate Spring profile will be configured to use the database service. The connection strings and credentials
needed to use the service will be extracted from the Cloud Foundry environment.

If no bound services are found containing any of these values in the name, then the `in-memory` profile will be used.

If more than one service containing any of these values is bound to the application, the application will throw an
exception and fail to start.

After installing the 'cf' [command-line interface for Cloud Foundry](http://docs.cloudfoundry.com/docs/using/managing-apps/cf/),
targeting a Cloud Foundry instance, and logging in, the application can be built and pushed using these commands:

~~~
$ ./gradlew assemble

$ cf push
...
requested state: started
instances: 1/1
usage: 512M x 1 instances
urls: spring-music--sf.cfapps.io
...
~~~

The application will be pushed using settings in the provided `manifest.yml` file. The output from the command will
show the URL that has been assigned to the application.

### Creating and binding services

Using the provided manifest, the application will be created without an external database (in the `in-memory` profile).
You can create and bind database services to the application using the information below.

#### System-managed services

Depending on the Cloud Foundry service provider, persistence services might be offered and managed by the platform. These
steps can be used to create and bind a service that is managed by the platform:

~~~
# view the services available
$ cf marketplace
# create a service instance
$ cf create-service <service> <service plan> <service name>
# bind the service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### User-provided services

Cloud Foundry also allows service connection information and credentials to be provided by a user. In order for the
application to detect and connect to a user-provided service, a single `uri` field should be given in the credentials
using the form `<dbtype>://<username>:<password>@<hostname>:<port>/<databasename>`.

These steps use examples for username, password, host name, and database name that should be replaced with real values.

~~~
# create a user-provided Oracle database service instance
$ cf create-user-provided-service oracle-db -p '{"uri":"oracle://root:secret@dbserver.example.com:1521/mydatabase"}'
# create a user-provided MySQL database service instance
$ cf create-user-provided-service mysql-db -p '{"uri":"mysql://root:secret@dbserver.example.com:3306/mydatabase"}'
# bind a service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### Changing bound services

To test the application with different services, you can simply stop the app, unbind a service, bind a different
database service, and start the app:

~~~
$ cf unbind-service <app name> <service name>
$ cf bind-service <app name> <service name>
$ cf restart
~~~

#### Database drivers

Database drivers for MySQL, Postgres, MongoDB, and Redis are included in the project. To connect to an Oracle database,
you will need to download the appropriate driver (e.g. from http://www.oracle.com/technetwork/database/enterprise-edition/jdbc-112010-090769.html?ssSourceSiteId=otnjp),
add the driver .jar file to the `src/main/webapp/WEB-INF/lib` directory in the project, and re-build the
application .war file using `./gradlew assemble`.
=======
# Cloud Foundry Java Buildpack
[![Build Status](https://travis-ci.org/cloudfoundry/java-buildpack.svg?branch=master)](https://travis-ci.org/cloudfoundry/java-buildpack)
[![Dependency Status](https://gemnasium.com/cloudfoundry/java-buildpack.svg)](https://gemnasium.com/cloudfoundry/java-buildpack)
[![Code Climate](https://codeclimate.com/repos/5224adaec7f3a3415107004c/badges/bc49f7d7f8dfc47057c8/gpa.svg)](https://codeclimate.com/repos/5224adaec7f3a3415107004c/feed)
[![Code Climate](https://codeclimate.com/repos/5224adaec7f3a3415107004c/badges/bc49f7d7f8dfc47057c8/coverage.svg)](https://codeclimate.com/repos/5224adaec7f3a3415107004c/feed)

The `java-buildpack` is a [Cloud Foundry][] buildpack for running JVM-based applications.  It is designed to run many JVM-based applications ([Grails][], [Groovy][], Java Main, [Play Framework][], [Spring Boot][], and Servlet) with no additional configuration, but supports configuration of the standard components, and extension to add custom components.

## Usage
To use this buildpack specify the URI of the repository when pushing an application to Cloud Foundry:

```bash
cf push <APP-NAME> -p <ARTIFACT> -b https://github.com/cloudfoundry/java-buildpack.git
```

## Examples
The following are _very_ simple examples for deploying the artifact types that we support.

* [Grails](docs/example-grails.md)
* [Groovy](docs/example-groovy.md)
* [Java Main](docs/example-java_main.md)
* [Play Framework](docs/example-play_framework.md)
* [Servlet](docs/example-servlet.md)
* [Spring Boot CLI](docs/example-spring_boot_cli.md)

## Configuration and Extension
The buildpack supports configuration and extension through the use of Git repository forking.  The easiest way to accomplish this is to use [GitHub's forking functionality][] to create a copy of this repository.  Make the required configuration and extension changes in the copy of the repository.  Then specify the URL of the new repository when pushing Cloud Foundry applications.  If the modifications are generally applicable to the Cloud Foundry community, please submit a [pull request][] with the changes.

To learn how to configure various properties of the buildpack, follow the "Configuration" links below. More information on extending the buildpack is available [here](docs/extending.md).

## Additional Documentation
* [Design](docs/design.md)
* [Security](docs/security.md)
* Standard Containers
	* [Dist ZIP](docs/container-dist_zip.md)
	* [Groovy](docs/container-groovy.md) ([Configuration](docs/container-groovy.md#configuration))
	* [Java Main](docs/container-java_main.md) ([Configuration](docs/container-java_main.md#configuration))
	* [Play Framework](docs/container-play_framework.md)
	* [Ratpack](docs/container-ratpack.md)
	* [Spring Boot](docs/container-spring_boot.md)
	* [Spring Boot CLI](docs/container-spring_boot_cli.md) ([Configuration](docs/container-spring_boot_cli.md#configuration))
	* [Tomcat](docs/container-tomcat.md) ([Configuration](docs/container-tomcat.md#configuration))
* Standard Frameworks
	* [AppDynamics Agent](docs/framework-app_dynamics_agent.md) ([Configuration](docs/framework-app_dynamics_agent.md#configuration))
	* [Java Options](docs/framework-java_opts.md) ([Configuration](docs/framework-java_opts.md#configuration))
	* [MariaDB JDBC](docs/framework-maria_db_jdbc.md) ([Configuration](docs/framework-maria_db_jdbc.md#configuration))
	* [New Relic Agent](docs/framework-new_relic_agent.md) ([Configuration](docs/framework-new_relic_agent.md#configuration))
	* [Play Framework Auto Reconfiguration](docs/framework-play_framework_auto_reconfiguration.md) ([Configuration](docs/framework-play_framework_auto_reconfiguration.md#configuration))
	* [Play Framework JPA Plugin](docs/framework-play_framework_jpa_plugin.md) ([Configuration](docs/framework-play_framework_jpa_plugin.md#configuration))
	* [PostgreSQL JDBC](docs/framework-postgresql_jdbc.md) ([Configuration](docs/framework-postgresql_jdbc.md#configuration))
	* [Spring Auto Reconfiguration](docs/framework-spring_auto_reconfiguration.md) ([Configuration](docs/framework-spring_auto_reconfiguration.md#configuration))
	* [Spring Insight](docs/framework-spring_insight.md)
* Standard JREs
	* [OpenJDK](docs/jre-open_jdk_jre.md) ([Configuration](docs/jre-open_jdk_jre.md#configuration))
	* [Oracle](docs/jre-oracle_jre.md) ([Configuration](docs/jre-oracle_jre.md#configuration))
* [Extending](docs/extending.md)
	* [Application](docs/extending-application.md)
	* [Droplet](docs/extending-droplet.md)
	* [BaseComponent](docs/extending-base_component.md)
	* [VersionedDependencyComponent](docs/extending-versioned_dependency_component.md)
	* [ModularComponent](docs/extending-modular_component.md)
	* [Caches](docs/extending-caches.md) ([Configuration](docs/extending-caches.md#configuration))
	* [Logging](docs/extending-logging.md) ([Configuration](docs/extending-logging.md#configuration))
	* [Repositories](docs/extending-repositories.md) ([Configuration](docs/extending-repositories.md#configuration))
	* [Utilities](docs/extending-utiltities.md)
* [Debugging the Buildpack](docs/debugging-the-buildpack.md)
* [Buildpack Modes](docs/buildpack-modes.md)
* Related Projects
	* [Java Buildpack Dependency Builder](https://github.com/cloudfoundry/java-buildpack-dependency-builder)
	* [Java Test Applications](https://github.com/cloudfoundry/java-test-applications)
	* [Java Buildpack System Tests](https://github.com/cloudfoundry/java-buildpack-system-test)

## Building Packages
The buildpack can be packaged up so that it can uploaded to Cloud Foundry using the `cf create-buildpack` and `cf update-buildpack` commands.  In order to create these packages, the rake `package` task is used.

### Online Package
The online package is a version of the buildpack that is as minimal as possible and is configured to connect to the network for all dependencies.  This package is about 50K in size.  To create the online package, run:

```bash
bundle install
bundle exec rake package
...
Creating build/java-buildpack-cfd6b17.zip
```

### Offline Package
The offline package is a version of the buildpack designed to run without access to a network.  It packages the latest version of each dependency (as configured in the [`config/` directory][]) and [disables `remote_downloads`][]. This package is about 180M in size.  To create the offline package, use the `OFFLINE=true` argument:

```bash
bundle install
bundle exec rake package OFFLINE=true
...
Creating build/java-buildpack-offline-cfd6b17.zip
```

### Package Versioning
Keeping track of different versions of the buildpack can be difficult.  To help with this, the rake `package` task puts a version discriminator in the name of the created package file.  The default value for this discriminator is the current Git hash (e.g. `cfd6b17`).  To change the version when creating a package, use the `VERSION=<VERSION>` argument:

```bash
bundle install
bundle exec rake package VERSION=2.1
...
Creating build/java-buildpack-2.1.zip
```

## Running Tests
To run the tests, do the following:

```bash
bundle install
bundle exec rake
```

[Running Cloud Foundry locally][] is useful for privately testing new features.

## Contributing
[Pull requests][] are welcome; see the [contributor guidelines][] for details.

## License
This buildpack is released under version 2.0 of the [Apache License][].

[`config/` directory]: config
[Apache License]: http://www.apache.org/licenses/LICENSE-2.0
[Cloud Foundry]: http://www.cloudfoundry.com
[contributor guidelines]: CONTRIBUTING.md
[disables `remote_downloads`]: docs/extending-caches.md#configuration
[GitHub's forking functionality]: https://help.github.com/articles/fork-a-repo
[Grails]: http://grails.org
[Groovy]: http://groovy.codehaus.org
[Play Framework]: http://www.playframework.com
[pull request]: https://help.github.com/articles/using-pull-requests
[Pull requests]: http://help.github.com/send-pull-requests
[Running Cloud Foundry locally]: http://docs.cloudfoundry.org/deploying/run-local.html
[Spring Boot]: http://projects.spring.io/spring-boot/
>>>>>>> upstream/master
