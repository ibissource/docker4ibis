# Run Ibis Configurations with Docker

This project has been superseeded by the Frank!Runner, https://github.com/ibissource/frank-runner. This project is not being maintained anymore.

This project will help you run your Ibis configuration(s) with Docker.

It is assumed you already have Docker Desktop installed and running on your computer (for OSX or Windows) or have the docker and docker-compose commands available on your system.

Basically all you have to do is to clone this project from within the same folder that contains your Ibis project folder(s) (or clone to an empty folder to start from scratch):

```
projects> git clone https://github.com/ibissource/docker4ibis
```

Change directory to docker4ibis:

```
projects> cd docker4ibis
```

On Windows run the following command to start Ibis4DockerExample:

```
docker4ibis> ./docker4ibis.bat Ibis4DockerExample
```

When not using Windows run:

```
docker4ibis> ./docker4ibis.sh Ibis4DockerExample
```

You can now browse to the following address to find the Ibis4DockerExample console:

```
http://localhost/ibis4dockerexample/
```

You can stop Docker using the following combination of keys: 

```
ctrl-c
```

In case Ibis4DockerExample doesn't already exist it will be created. By default your folder structure will (need to) look like the following:

```
|--projects
   |--docker4ibis
   |--Ibis4DockerExample
      |--classes
      |--configurations
         |--Config1
         |--Config2
         |--Conf...
      |--tests
   |--Ibis4...
```

You could for example add a file named Configuration.xml with the following content to the classes folder to run this Ibis configuration with Docker:

```
<Configuration name="Ibis4DockerExample">
	<jmsRealms>
		<jmsRealm datasourceName="jdbc/${instance.name.lc}" realmName="jdbc"/>
	</jmsRealms>
	<Adapter name="HelloDockerWorld">
		<Receiver name="HelloDockerWorld">
			<JavaListener name="HelloDockerWorld"/>
		</Receiver>
		<Pipeline firstPipe="HelloDockerWorld">
			<FixedResultPipe name="HelloDockerWorld" returnString="Hello Docker World">
				<Forward name="success" path="EXIT"/>
			</FixedResultPipe>
			<Exit path="EXIT" state="success"/>
		</Pipeline>
	</Adapter>
</Configuration>
```

More information on Ibis configuration files and Ibis property files and how to use them can be found in the Ibis manual which will soon be available.

## Choosing your version of the IAF

You can add a second argument to `docker4ibis.bat` or `docker4ibis.sh` script that defines the version. To search the available versions, please go to https://hub.docker.com/ and search for image `ibissource/iaf`. You should have one search result, please click it. You then have two tabs, "Overview" and "Tags". Choose tab "Tags". Choose one of the tags you see there, for example `7.5-20200124.103215`. You can then do

```
docker4ibis> ./docker4ibis.bat Ibis4DockerExample 7.5-20200124.103215
```

or the equivalent `docker4ibis.sh` command under Linux.

Alternatively, you can build your own image as explained later in this file. When you tag it `ibissource/iaf:my` (docker argument `-t`), you can use it like:

```
docker4ibis> ./docker4ibis.bat Ibis4DockerExample my
```

## Checking on your containers

You can see all your running containers using the docker command:

- docker ps
 
If you also want to see all existing containers you can use:

- docker ps -a

If you want to stop a running container use:

- docker stop (**containername**)

And you can remove a container with:

- docker rm (**containername**)

Sometimes it can be useful to look at the files inside the docker container. The docker container must be running for this to work. To do this use:

- docker exec -it (**containername**) bash

You can now use Linux commands inside the docker container. Type exit to return to your host machine.

## Overwrite default values for Docker4Ibis settings

In case for example you would like to have a different folder structure or like to use a different database you can specify this by adding a file called docker4ibis.properties to your Ibis folder. This file can contain the following properties:

 - database: Set the database you want to use, needs to be one of the following values. Right now, the supported databases are H2, PostgreSQL, MySQL, MariaDB and Microsoft SQL Server:
    - h2 (default)
    - postgresql
    - mysql
    - mariadb
    - mssql
- hostport: Specify the port used to connect to the IAF container, default value is port 80.
- otap_stage: Specify the otap stage, default value is LOC.
- ibis_classes: Give a path to the directory containing your main configuration, relative to the Ibis directory containing this properties file. Default is "classes".
- ibis_config: Give a path to the directory containing your other configurations, relative to the Ibis directory containing this properties file. Default is "configurations".
- ibis_tests: Give a path to the directory containing your Larva test scenarios, relative to the Ibis directory containing this properties file. Default is "tests".

In this file you can set a path to your **classes**, **configurations**, and **tests** folders. The **classes** folder should contain your main configuration.

If you have multiple configurations for your Ibis, the other configurations should be placed in the **configurations** folder. When doing so make sure to set the classLoaderType of these configurations to DirectoryClassLoader in a DeploymentSpecifics file in the **classes** folder. For example, if I have an Ibis called Ibis4Example with a main configuration Ibis4Example and one extra configuration called MyConfig, I will need to add the following to a DeploymentSpecifics.properties file in the **classes** folder:
```
configurations.names=Ibis4Example,MyConfig
configurations.MyConfig.classLoaderType=DirectoryClassLoader
```

In the **tests** folder you can place your Larva test scenarios.  If you do not have any extra configurations or any Larva test scenarios you can give an empty string to the properties ibis_config and ibis_tests respectively.
It is also possible not to give a docker4ibis.properties file, in this case default values will be used when starting the IAF docker image.

The docker4ibis directory from github also contains a file called docker4ibis.properties. In this file you can give the path to the directory containing your Ibis configurations. Just change the value of the projects_directory property to the path you want. This projects directory should contain one directory for every Ibis you have. The name of these directories should be the same as their corresponding Ibis. The value of the instance.name property will be the same as the name of the directory containing the Ibis, so make sure the directory has the correct name.

## Building the IAF image

The IAF Docker image can be found on DockerHub at https://hub.docker.com/r/ibissource/iaf. This means that it is not necassary to build the IAF image yourself, it will automatically be pulled from DockerHub when trying to start an IAF container. However if you still want to build the IAF image yourself you need to do the following:

The files needed to build the IAF image can be found in the IAF_Image directory. Open up your favorite command-line-interface and go to the IAF_Image directory on your computer. First make sure you are logged in with your Docker account. Use the command:

- docker login

Now, use the following command to build the image:

- docker build --build-arg IAF_VERSION=**version** -t ibissource/iaf:**yourtag** .

Wait for the building process to finish and you should be able to use the IAF image.

## Troubleshooting

- Only one container can run using the same hostport at a time, so if a specific port number is not available make sure there are no other containers still running that use the same port. Check the Checking on your containers section of this README to see how to stop any running containers. If a different application is using the specified hostport you need to either stop this other application or change the hostport in the docker4ibis.properties file of your Ibis as specified in the Overwrite default values for Docker4Ibis settings section of this README.

- When opening the Ibis console make sure that the name of the Ibis in the URL is in all lowercase characters. The correct URL will be:
```
http://localhost:{HOSTPORT}/{Ibis Name in all lowercase}/
```
Replace {HOSTPORT} with the port number specified in the docker4ibis.properties file or the default port 80. If I want to run my Ibis called Ibis4DockerExample using port 8080 as the hostport, I can browse to the following address to find the Ibis4DockerExample console:

```
http://localhost:8080/ibis4dockerexample/
```

- You will need to be able to share data between your local drive and the docker container. Sharing drives might not be set by default causing an error or making it so your configurations and tests cannot be found in the Ibis console. In order to share your local drive with docker you will need to go to the settings of Docker Desktop. In the settings menu go to the Shared Drives tab. Check the drive(s) you want to share with docker and click apply. Docker should now be able to use docker volumes and share files between your local drive and a docker container.
