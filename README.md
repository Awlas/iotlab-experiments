# iotlab-experiments

![](https://img.shields.io/badge/docker-20.10.21-green)
![](https://img.shields.io/badge/docker_compose-1.25.0-green)

This project permit to control openwsn experiments on [IoT-LAB](https://www.iot-lab.info/) using the *iotlab-experiments* docker image where are set all the dependencies and few scripts to lauch the experiments.

## Prerequisites

To use this tool you need to install docker and docker-compose: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/).

All the tests have been made on 20.10.21 docker's version and 1.25.0 docker-compose's version.

## Dependencies

This *iotlab-experiments* docker image set all the dependencies need to this project:

- [https://hub.docker.com/r/awlas/iotlab-experiments](https://hub.docker.com/r/awlas/iotlab-experiments) [https://github.com/iot-lab/ssh-cli-tools](https://github.com/iot-lab/ssh-cli-tools): all the tools to provide a basic set of operations for managing IoT-LAB experiments from the command-line.
- [https://github.com/openwsn-berkeley/openwsn-fw](https://github.com/openwsn-berkeley/openwsn-fw): the firmware (in C)
- [https://github.com/openwsn-berkeley/openvisualizer](https://github.com/openwsn-berkeley/openvisualizer): the software on the PC side (to get the statistics, configure the motes, monitor everything)
-  [https://github.com/Awlas/openwsn-iotlab](https://github.com/Awlas/openwsn-iotlab): scripts to handle an experiment

# Usage

## Run / Stop experiments

To lauch experiments, run:
```
docker-compose up 
```

This command create and run two docker's containers:
- *IOTLab_OpenV_Server_Exp*, to lauch the experiments and run an [OpenVisualizer Server](https://github.com/openwsn-berkeley/openvisualizer) that interact with the network deployed on Iot-LAB or simulate on emulated motes. This container will use [openwsn-iotlab](https://github.com/ftheoleyre/openwsn-iotlab.git) python scripts to do that
- *IOTLab_OpenV_Client_Exp*, to run the web interface client using [OpenVisualizer Client](https://github.com/openwsn-berkeley/openvisualizer)

You can stop the entire application (namely the both containers) at any time by hitting **Ctr+C**. You can also stop each containers by using:
```
docker stop IOTLab_OpenV_Server_Exp
docker stop IOTLab_OpenV_Client_Exp
```

To see if the containers are running, open a new terminal and run:
```
docker ps -a
```
Then you can see their status: *Up* for running and *Exited* for stoped.

## Environment variables

The python pipeline uses the following environments variables on the IOTLab_OpenV_Server_Exp container to set the experiment parameters.

Here are the different variables that exist:

| Environment variable    | Functions     | Possible value  | Default value |
|:----------|:-------------:|:------------:|--------------:|
|`SUB_EXP_DURATION`|duration of an experiment in minutes|\<number\>|60|
|`SAFE_MARGIN_DURATION`|safety margin in minutes|\<number\>|30|
|`NB_EXP`|number of experiments|\<number\>|5|
|`EXP_NAME`|name of the experiment|[\<string\>,RANDOM]|RANDOM|
|`SITE`|site|[grenoble, lille, paris, saclay, strasbourg, toulouse]|grenoble|
|`NB_MOTES`|number of motes|\<number\>|12|
|`MAX_ID`|discard larger node's ids|\<number\>|289|
|`MIN_ID`|discard smaller node's ids|\<number\>|70|
|`MAX_SPACE_ID`|max separation with the closest id|\<number\>|9|
|`SIMULATION`|simulation mode|[True,False]|False|
|`COMPIL`|compile the fireware|[True,False]|True|
|`EXP_RESUME`|restart an already running experiment (if one exists)|[True,False]|True|
|`EXP_RESUME_VERIF`|verification that the motes are those specified (in the running exp)|[True,False]|False|

To change the value of an environment variable, add `<ENVIRONMENT_VARIABLE>=<value>` to the command when you run the application as follow:
```
<ENVIRONMENT_VARIABLE>=<value> ... docker-compose up
```
Note that you can assign value to multiple environment variable on the same command.

If you want to lauch experiments multiple time you can directly change the default value on the *docker-compose.yml*. For each environment variable the following line is set:
```
<ENVIRONMENT_VARIABLE>=${<ENVIRONMENT_VARIABLE>:-<default_value>}
```
You can simply open the file, change the default value, save the file and the changes would be applied when you rerun `docker-compose up`.

## Authentication

To run an experiment, IOTLab_OpenV_Server_Exp container need to be authenticate with an account. After having running the previous command to start the containers, the script will be running and the following warning will be print to you on the execution :

> Waiting for authentication with "iotlab-auth -u \<USER\>" on an other terminal...

And

> Please, check if you are well connected on ssh to your IoT-LAB account...\
> If it's not the case, create ssh key and link it to your account by running "iotlab-auth --add-ssh-key" on an other terminal

Do not be afraid, you have time to perform these actions, the script will not stop.
So to begin open an other terminal and execute the following command:

```
docker exec -it IOTLab_OpenV_Server_Exp bash
```
This command will run an interactive terminal thanks to the **-it** option with **bash** on the running **IOTLab_OpenV_Server_Exp** container. Now you are on the container and you can authenticate yourself by:
- using the following command to set your IoT-LAB credentials:
    ```
    iotlab-auth -u <login>
    ```
- generate a key with the default file name and location by pressing Enter with this command:
    ```
    ssh-keygen -t rsa
    ```
- adding the public key to your account by using:
    ```
    iotlab-auth --add-ssh-key
    ```

These actions to authenticate your self are required when:
- it's the first time you run `docker-compose up`
- you change the default value/value of an environment variable
- the previous running changed the value of an environment variable
