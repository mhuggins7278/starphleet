# Starphleet

**Repositories + Buildpacks + Containers = Autodeploy Services**

___

Starphleet is a toolkit for turning [virtual](http://aws.amazon.com/ec2/) or physical machine infrastructure into a continuous deployment stack, running multiple Git-backed services on one more nodes via [Linux containers](https://linuxcontainers.org/).

Starphleet borrows heavily from the concepts of the [Twelve-Factor App](http://12factor.net), and uses an approach that avoids many of the problems inherent in existing autodeployment solutions:

* Conventional virtualization, with multiple operating systems running on shared
  physical hardware, wastes resources, specifically RAM and CPU.  This costs real money.
* Autodeploy PaaS has the same vendor lock-in risks of old proprietary software.
* Continuous deployment is almost always a custom scripting exercise.
* Multiple machine / clustered deployment is extra work.
* Making many small services is more work than making megalith services.
* Seeing what is going on across multiple machines is hard.
* Deployment systems all seem to be at the *system* not *service* level.
* Every available autodeploy system requires that you set up servers to
  deploy your servers, which themselves aren't autodeployed.


# Overview

* **Orders**: The atomic unit of Starphleet.  An individual Ruby, Python, NodeJS, or plain HTML **service** run in a [Linux container](https://linuxcontainers.org/).

* **Ship**: A virtual machine instance with one or more running orders.  Launched manually with `$ vagrant up` or the [Starphleet CLI](https://github.com/wballard/starphleet-cli).

* **Phleet**: A collection of one or more ships.  Phleets are intended to correspond to a single load-and-geo-balanced resource, such as `services.example.com`.

* **Git Repositories**: Starphleet requires the use of the following types of repositories.

  * **Starphleet Core**: Hosted by us and accessed at `github.com/wballard/starphleet.git`, contains the source code for Starphleet allowing you to bootstrap your headquarters repository.  The Starphleet Core repository should exist in one location.
  * **Headquarters**: Hosted by you at `{headquarters_git_url}` (with an example  [available](https://github.com/wballard/starphleet.headquarters.git) from us), contains the configuration for the ships (virtual machine instances) and associated [Linux containers](https://linuxcontainers.org/).  You will need one headquarters repository per phleet.
  * **Services**: Hosted by you at `{service_git_url}` and referenced in your headquarter's **orders** files, contains the source code for the individual services which run in [Linux containers](https://linuxcontainers.org/) on ships.  Service repositories can be referenced by multiple orders files across phleets.

* **Environment Variables**: Starphleet is configured entirely by environmental variables, saving you the chore of repeatedly typing the same text.

# Get Started

1.  Clone the Starphleet repository to your workstation, then change your current directory to the cloned folder.

  ```bash
  $ git clone https://github.com/wballard/starphleet.git
  $ cd starphleet
  ```

1.  Set the environment variable for the Git URL to your Starphleet headquarters, which contain the configuration for your phleet.  We suggest you start by forking our [base headquarters](https://github.com/wballard/starphleet.headquarters.git).  Your headquarters Git URL **must be network reachable** from your hosting cloud, making [public git hosting services](https://github.com/) a natural fit for Starphleet.


  ```bash
  $ export STARPHLEET_HEADQUARTERS={headquarters_git_url}
  ```

1.  Set the environment variable for the locations of your public and private key files which are associated with the Git repository for your Starphleet headquarters.  If you have not yet generated these files, you can do so using [ssh-keygen](https://help.github.com/articles/generating-ssh-keys).  

  ```bash
  $ export STARPHLEET_PRIVATE_KEY=~/.ssh/{private_keyfile}
  $ export STARPHLEET_PUBLIC_KEY=~/.ssh/{public_keyfile}
  ```

After completing the above configuration steps, you can choose to deploy Starphleet (a) on your local workstation using **[Vagrant](http://www.vagrantup.com)**, or (b) into the cloud with **[Amazon Web Services (AWS)](http://aws.amazon.com)**.

## Locally (Vagrant)

[Vagrant](http://www.vagrantup.com) is a handy way to get a working autodeployment system inside a virtual machine right on your local workstation. Prebuilt base images are provided in the `Vagrantfile` for VMWare, VirtualBox and Parallels. The [Vagrant](http://www.vagrantup.com) option is great for figuring if your services will start/run/autodeploy without worrying about cloud configuration.

1.  From the cloned [Starphleet](https://github.com/wballard/starphleet) directory, us Vagrant's `up` command, which will launch a new ship (virtual machine instance), perform a `$ git pull` on your `STARPHLEET_HEADQUARTERS`, deploy a new [Linux container](https://linuxcontainers.org/), and configure the service specified in the Starphleet headquarters (including automatically running `$ npm install` and `$ npm start`).

  ```bash
  $ vagrant up
  ```

1.  Get the IP address, `{ship_ip}`, of your new virtual machine instance.

  ```bash
  $ vagrant ssh -c "ifconfig eth0 | grep 'inet addr'"
  ```
1.  Navigate in your web browser to `http://{ship_ip}/echo/hello_world`.  There will be an availability delay while Starphleet runs bootstrap code, however subsequent service deployments and updates will completely quickly.

## In the Cloud (AWS)
Starphleet includes [Amazon Web Services (AWS)](http://aws.amazon.com) support out of the box.  To initialize your phleet, you need to have an AWS account.

1.  Set additional environment variables required for AWS use.

  ```bash
  $ export AWS_ACCESS_KEY_ID={your_aws_key_id}
  $ export AWS_SECRET_ACCESS_KEY={your_aws_access_key}
  ```

1.  Install the Starphleet command line interface (CLI) tool.

  ```bash
  $ npm install -g starphleet-cli
  ```

1.  Use the Starphleet CLI's `init` and `add` commands to launch a new ship (virtual machine instance), perform a `$ git pull` on your `STARPHLEET_HEADQUARTERS`, deploy a new [Linux container](https://linuxcontainers.org/), and configure the service specified in the Starphleet headquarters (including automatically running `$ npm install` and `$ npm start`).

  ```bash
  $ starphleet init ec2
  $ starphleet add ship ec2 us-west-1
  ```

1.  Get the IP address, `{ship_ip}`, of your new ship.

  ```bash
  $ starphleet info ec2
  ```

1.  Navigate in your web browser to `http://{ship_ip}/echo/hello_world`.  There will be an availability delay while Starphleet runs bootstrap code, however subsequent service deployments and updates will completely quickly.




## All Running?
Once you are up and running, look in your `{headquarters_git_url}` repository at `echo/orders`. The contents of the orders directory are all that is required to get a web service automatically deploying on a ship (virtualized machine instance).

* `export PORT=3000` to know on what port your service runs.  This port is mapped, by Starphleet, back to `http://{ship_ip}/echo` (on port 80).
* `autodeploy https://github.com/wballard/echo.git` to know what to deploy.  Starphleet will automatically deploy your Ruby, Python, NodeJS, and static NGNIX projects with buildpacks.

Order-ing up your own service is just as easy as adding a new directory and creating the `orders` file. Add. Commit. Push. Magic, your service will be available.  Any time that a Git repository referenced in an orders file is updated, for example `github.com/wballard/echo.git`, it will be autodeployed to every ship watching your headquarters.

# MicroService Clouds
The primary idea of Starphleet is to let you quickly create a cloud of related microservice. To make this easier, the key abstraction is that your services are at **paths** not **ports** by default. This lets you order up a series of services and have them all on one DNS name, saving you a lot of heartache with CORS, load balancers, and DNS configuration.

Say you have a simple service with a todo list, a mail queue, and a website. You can set up a headquarters like:

```
./orders
./todo
  orders
./mail
  orders
```

Then when running you get your site at `http://example.co/`, and your services at `http://example.co/todo` and `http://example.co/mail`.

# Reference


## Headquarters
A headquarters is a Git repository that instructs the phleet (one or more virtual machine instances) how to operate. Using Git in this manner

* Provides a versioned database of your configuration
* Allows editing and working with your own tools
* Provides multiple hosting options
* Avoids the need for a separate Starphleet server

By default, all services are federated together behind one host name. This is particularly useful for single page applications making use of a set of small, sharp back end services, without all the fuss of CORS or other cross domain technique.  Note that the Git URL value assigned to `STARPHLEET_HEADQUARTERS` **must be network reachable** from each ship in the phleet.

### File Structure

The `STARPHLEET_HEADQUARTERS` repository is the primary location for phleet, ship, and service configuration, and can contain the following directories and files:

```
authorized_keys/
  user1@example.com.pub
  user2@example.com.pub
beta_groups/
  friends
ldap_servers/
  boss
{service_name}/
  .htpasswd
  .ldap
  orders
  remote
overlay/
shipscripts/
  dynamic_dns_example.sh
  maintenance_example.sh
ssl/
  crt
  key
jobs
.starphleet
```

#### authorized\_keys/
A directory containing public key files, one key per file, which allows ssh access to the ships as follows:

```bash
$ ssh admiral@{ship_ip}
```

Every user shares the same username, `admiral`, which is a member of the sudoers group.  Once pushed to your headquarters, updates to the `authorized_keys/` directory will be reflected on your ships within seconds.  This open ssh access to each ship lets you do what you want, when you want.  If you manage to wreck a ship, you can always add a new one using the [Starphleet CLI](https://github.com/wballard/starphleet-cli).

#### beta\_groups/
Working together with authentication, you can create **beta groups**. These groups provide a named list of users.

```
wballard
fred
```

#### ldap\_servers/
Authenticate users to your services via LDAP by specifying LDAP servers. Each named file in this directory requires three environment variables to connect a service account for end user LDAP Authentication. Here is how we hook up to our active directory:

```bash
export LDAP_URL='ldap://guardian-gc.glgresearch.com:3268/dc=glgroup,dc=com?sAMAccountName?sub?(objectCategory=person)(objectClass=User)'
export LDAP_USER='glgroup\\sampleServiceAccount'
export LDAP_PASSWORD='****'
```

You can have multiple LDAP servers. Individual user authentication will be cached, so that services with the same `.ldap` value will not multiply prompt for login.

#### {service_name}/
A directory which defines the relative path from which your service is served (`echo/` in the case of the [base headquarters](https://github.com/wballard/starphleet.headquarters.git)) and which contains the service configuration files as its contents.  Starphleet will treat as a service any root directory in your headquarters which contains an orders file and which does not use a reserved name (which are the other folder names in this section).  It is also possible to launch a service on `/`, by including an `orders` file at the root of your Starphleet headquarters repository.

* **.htpasswd**: Similar to good old fashioned Apache setups, you can put an `.htpasswd` file in each order directory, right next to `orders`. This will automatically protect that service with HTTP basic, useful to limit access to an API.

  **.ldap**: Place a single string in here that matches a file name under `{headquarters}/ldap_servers`. This will require users to be authenticated against LDAP.

* **orders**:  An `orders` file is a shell script which controls the autodeployment of a service inside a [Linux container](https://linuxcontainers.org/) on a ship.

  We chose to use shell scripts for the orders files to allow your creativity to run wild without needing to learn another autodeployment tool.

  ```bash
  export PORT={service_port}
  autodeploy {service_git_url}
  beta {beta_group} {service_url}
  stop-before-autodeploy
  ```

  You can specify your `{service_git_url}` like `{service_git_url}#{branch}`, where branch can be a branch, a tag, or a commit sha -- anything you can check out. This hashtag approach lets you specify a deployment branch, as well as pin services to specific versions when needed.  The service specified in the orders file with the `{service_git_url}` must support the following:
  1. Serve HTTP traffic to the port number specified in the `PORT` environment variable.  Websockets can also be utilized, it is just http after all.
  1. Be hosted in a Git repository, either publicly available or accessible via key-authenticated git+ssh.
  1. Able to be installed and run with a buildpack.

  The [Linux containers](https://linuxcontainers.org/) which run the services are thrown away on each new service deployment and on each ship reboot.  While local filesystem access is available with a container, it is not persistent and should not be relied upon for persistent data storage.  Note that your `0service_git_url}` **must be network reachable** from each ship in the phleet.

  The `beta` command references a `beta_group` by name, and will transparently send any user identified in the group to the alternate `service_url`. Users are identified vit `.htpasswd` or `.ldap`. The `service_url` is just another ordered service on the ship, most likely a different branch to test a new feature.

  The `stop-before-autodeploy` command is useful for services that are not rolling deployment friendly. For example, we have an index service we call `pi` that uses [SOLR](http://lucene.apache.org/solr/) that has a nasty habit of not retrying to get at locked files. So, this makes sure to stop first, releasing the file lock.

* **remote**: A file specifying data to be autodeployed to to `/var/data/{service_name}` inside the [Linux container](https://linuxcontainers.org/) for `service_name`.  Starphleet symlinks `/var/data/` for all [Linux containers](https://linuxcontainers.org/) to `/var/data/` on the parent ship.  As a result, data specified by the `remote` file is visible to all [Linux container](https://linuxcontainers.org/) instances on a ship.  Only one item is needed inside a `remote` file:

  ```bash
  autodeploy {data_git_url}
  ```

Each ship in the phleet runs every ordered service. This makes things nice and symmetrical, and simplifies scaling. Just add more ships if you need more capacity. If you need full tilt performance, you can easily make a phleet with just one ordered service at `/`. Need a different mixture of services? Launch another phleet!

While each [Linux container](https://linuxcontainers.org/) (and by extension, service) has its own independent directory structure, Starphleet symlinks `/var/data/` in each [Linux container](https://linuxcontainers.org/)  to `/var/data/` on the ship, allowing

  1. Data that lives between autodeploys of your service
  1. Collaboration between services

As `/var/data/` is persistent across autodeploys, care must be taken to ensure the ship's storage does not become full.  Also, note that `/var/data/` is a shared **local** fileystem across [Linux containers](https://linuxcontainers.org/) on the same ship.  It does not provide a shared filesystem between ships in a phleet.

#### overlay/
This is a directory full of any files you see fit. Whenever the headquarters is updated, these files are recursively copied, as root from the the headquarters to `/` on your ship, such that a file at `{headquarters/overlay/hi_mom` ends up at `/hi_mom` on your running ship.

#### shipscripts/
A directory containing scripts, which will run on individual ships when the headquarters is updated.

Scripts in this folder can be used to implement many different kinds of functionality, including dynamic DNS registration.  If you want to simulate dynamic DNS with Amazon Route53, look at the [starphleet-cli](https://github.com/wballard/starphleet-cli) command `starphleet name ship ec2`.  Note that scripts in this directory **must be marked as [executable](http://www.dslreports.com/faq/linux/7.1_chmod_-_Make_a_file_executable)** in order to run on the ships.  Scripts also run as root on the ships - take care to avoid a shipwreck.

#### ssl/
Provide two files `crt` and `key`, which must be set to not need a password, in order to enable
the ship to service SSL.

#### jobs
A file which allows scheduling of phleet-level cron tasks to call specified service endpoints.  
The jobs file uses cron syntax, the only difference being a URL in place of a local command.


```shell
* * * * * http://localhost/workflow?do=stuff
* * * 1 * http://localhost/workflow?do=modaystuff
```

All [Linux containers](https://linuxcontainers.org/) support `cron`, but the jobs file allows you to dodge a few common problems with cron:

* The logging gets captured to syslog for you automatically and then piped to `logger`.
* The environment is fixed inside your container for your service.
* You don't need to think about which account runs the jobs.
* One-off instances of the jobs can be run manually with curl.

#### .starphleet
A file which contains environment variables that apply to all services in your phleet.  This file is a good place to store usernames, passwords, and connection strings, if your headquarters is in a private, hidden repository reachable by `git+ssh`.  The environment variables set in this file take precedence over all others set elsewhere, and apply to every service deployment within a phleet.


## Environment Variables
Starphleet is configured entirely by environmental variables and encourages the use of custom environment variables.  Some environment variables may apply to certain levels (an individual service or a phleet) or may need to remain private for security reasons (login credentials), and as a result Starphleet will apply environment variables in the following order:

1. **`{service_git_url}/.env`**

  The environment variable file sourced from the root directory of your service (at the root of the `service_git_url` specified in your service `orders` file).  As services will typically be hosted publically, the environment variables added to a `.env` file should concern development mode or public settings, such as `BUILDPACK_URL`.  These variables have the lowest precedence and apply to all of this particular service's deployments, regardless of phleet.

2. **`{headquarters_git_url}/{service_name}/orders`**

  The shell script used to autodeploy your service. Environment variables commonly set here include `PORT`, and apply to all service instances spawned from your headquarters.  These variables have precedence over those set in your service's `.env` file and apply to all of the specified particular service's deployments within a phleet.

3. **`{headquarters_git_url}/.starphleet`**

  The environment variable file which applies to all services in your phleet.  This file is a good place to store usernames, passwords, and connection strings, if your headquarters is in a private, hidden repository reachable by `git+ssh`.  These variables have the highest precedence and apply to every service deployment within a phleet.

### Environment Variable Reference

#### For Your Workstation

| Name | Value | Description
| --- | --- | ---
| AWS_ACCESS_KEY_ID | string | Used for [AWS](http://aws.amazon.com) access.  Set this on your workstation prior to using Starphleet CLI.
| AWS_SECRET_ACCESS_KEY | string | Used for [AWS](http://aws.amazon.com) access.  Set this on your workstation prior to using Starphleet CLI.
| EC2_INSTANCE_SIZE | string | Override the size of EC2 instance with this variable.  Set this on your workstation prior to using Starphleet CLI.
| STARPHLEET_HEADQUARTERS | string | The Git repository URL to your phleet's headquarters.  Set this on your workstation prior to using Starphleet CLI.
| STARPHLEET_PRIVATE_KEY | string | The path to the private keyfile associated with your Git repository, such as `~/.ssh/{private_keyfile}`.  Set this on your workstation prior to using Starphleet CLI.
| STARPHLEET_PUBLIC_KEY | string | The path to the public keyfile associated with your Git repository, such as `~/.ssh/{public_keyfile}`.  Set this on your workstation prior to using Starphleet CLI.
| STARPHLEET_VAGRANT_MEMSIZE | number | The memory size, in megabytes, of the [Vagrant](http://www.vagrantup.com) instance.  Set this on your workstation prior to using Starphleet Vagrant for testing.

#### For Your Headquarters
These variables can be set in your headquarters `.starphleet` or `orders`.

| Name | Value | Description
| --- | --- | ---
| BUILDPACK_URL | &lt;git_url&gt; | Specifies a custom buildpack to be used for autodeployment.  Set this in your Starphleet headquarters or in your service Git repository.
| PORT | number | This is an **all important environment variable**, and it is expected your service will honor it, publishing traffic here. This `PORT` is used to know where to connect the ship's proxy to your individual service.  Set this in your orders file.
| PUBLISH_PORT | number | Allows your service to be accessible on the ship at `http://{SHIP_DNS}:PUBLISH_PORT` in addition to `http://{SHIP_DNS}/{service_name}`.  Set this in your orders file.
| STARPHLEET_PULSE | number | The number of seconds between autodeploy checks, defaulting to a value of 10.
| USER_IDENTITY_HEADER | string | When using LDAP or basic authentication, Starphleet will write the user identity into this header so that your services can see it.


## Buildpacks
Buildpacks autodetect and provision services in containers for you.  We would like to give a huge thanks to Heroku for having open buildpacks, and to the open source community for making and extending them. The trick that makes the Starphleet orders file so simple is the use of buildpacks and platform package managers to install dynamic, service specific code, such as `rubygems` or `npm` and associated dependencies, that may vary with each push of your service.  Note that **Starphleet will only deploy one buildpack per Linux container** - for services which are written in multiple languages, custom buildpacks may be required.

Starphleet currently includes support for Ruby, Python, NodeJS, and NGINX static buildpacks. You can specify your own in your `orders` with `export BUILDPACK_URL=`.

### [Procfile](https://devcenter.heroku.com/articles/procfile)
Starphleet only supports `web` process types.

### [NodeJS](https://github.com/heroku/heroku-buildpack-nodejs.git)
You will need a proper, working `package.json`. The buildpack will call `npm install` and `npm start`, with the `PORT` environment variable set.

Been at this a while? You can check in your `node_modules` directory, and the buildpack will call `npm reinstall`. This can speed up deployment greatly.

### [Ruby](https://github.com/heroku/heroku-buildpack-ruby.git)
This will run `bundle install` and make use of your `Procfile`.

### [NGINX](https://github.com/wballard/nginx-buildpack.git)
This looks for a `nginx.conf.erb` template. The configuration will be rendered with all available environment variables.

## Maintenance

### Service Start
There is no need in Starphleet to explicitly call `nodemon`, `forever`, or any other keep alive system with your services, Starphleet will fulfill your dependencies and start your service automatically.  In NodeJS projects, this means Starphleet will load the proper buildpack (NodeJS), resolve dependencies by issuing an `$ npm install` command, and then (absent a procfile, see below) start your service by issuing an `$ npm start` command.  In order to use the automatic start functionality, ensure that:

1.  You include functional [procfiles](https://devcenter.heroku.com/articles/procfile) in your service repository, or
2.  Your project can be built and run with package manager specific features, such as `npm start` and `npm install`.

### Service Updates
Just commit and push to the repository referenced in your orders file, `{service_git_url}`, which will result in a service autodeployment to every associated ship (even across phleets if a service is used in more than one phleet).  As new versions of services are updated, fresh containers are built and run in parallel to prior versions with a drainstop. As a result, in-process requests to existing services should not interrupted, with one caveat: database and storage systems maintained outside of Starphleet.  Many software components are developed in a database-heavy manner with no real notion of backward compatibility for data storage.  In order to unlock the full benefit of autodeployment and rolling upgrades in Starphleet, you must think about how different versions of your code will interact with your database and storage systems.

#### Healthcheck
Each service repository can supply a `healthcheck` file, located at `/healthcheck`, which contains the following content:
  ```bash
  /{snippet}
  ```

Upon deployment of a service update, Starphleet will issue a GET request to `http://{container_ip}:PORT/{snippet}`, and will expect an HTTP 200 response within 60 seconds.  The PORT in the preceding URL will have the value of the PORT environment variable specified in your service's `orders` file.

### Service Rollbacks
If bad update goes out to a service, it can be easily reverted by using `$ git revert` to pull out the problem commits, then re-pushing to the `{service_git_url}` referenced in `{headquarters_git_url}/{service_name}/orders`.  This approach also preserves your commit and deploy history.

### Service Crashes
Starphleet monitors running services and will restart them on failure.

### Starphleet Updates
To check for the latest version of Starphleet on a ship and install an update, if needed, run the following:

```bash
$ ssh admiral@{ship} sudo starphleet_update
```

You will need to ensure your public key has been added to the `authorized_users/` directory in your Starphleet headquarters.

### Starphleet Status
You can always see what is going on with your ship using:

```bash
$ ssh admiral@{ship} starphleet_status
$ ssh admiral@{ship} starphleet_logs
```

### Self Healing Phleet
Each ship uses a pull strategy to keep up to date. This strategy has been chosen over push deployments for the following reasons:

* Ships go [up and down](http://en.wikipedia.org/wiki/Heavy-lift_ship#Submerging_types), and a pull-based strategy lets ships catch up easily if they are offline when a new version of a service is released.
* After a new ship is added to the phleet (`$ starphleet add ship`), the pull mechanism will catch it up automatically.
* The developer does not have to wait for a Heroku-style push completes, watching the build go by, and can instead move on to developing the next feature.


## Amazon Web Services

### EC2 Instance Sizes
Don't cheap out and go small. The default instance size in Starphleet is `m2.xlarge`, which is roughly the power of a decent laptop.  You can change this with by setting the `EC2_INSTANCE_SIZE` environment variable.

### Phleets
Don't feel limited to just one phleet. Part of making your own PaaS is to give you the freedom to mix and match services across phleets as you see fit.