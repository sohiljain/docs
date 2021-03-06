---
title: "Customizing your Image"
description: "Adding the right dependencies and packages to your image"
date: 2018-07-17T00:00:00.000Z
slug: "customizing-your-image"
---

### Customizing your Docker Image

Now that you've started running Airflow with the Astro CLI, there will be some Docker images running on your machine with their own mounted volumes.

When going through this doc, keep in mind that the Astro CLI is built on top of Docker Compose.

```
docker ps

CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                                        NAMES
b32be577f24d        airflow-code_66a665/airflow:latest   "tini -- /entrypoint…"   4 hours ago         Up About an hour    5555/tcp, 8793/tcp, 0.0.0.0:8080->8080/tcp   airflowcode66a665_webserver_1
28c8a7db90bd        airflow-code_66a665/airflow:latest   "tini -- /entrypoint…"   4 hours ago         Up About an hour    5555/tcp, 8080/tcp, 8793/tcp                 airflowcode66a665_scheduler_1
c572fe53093e        postgres:10.1-alpine                 "docker-entrypoint.s…"   4 hours ago         Up About an hour    0.0.0.0:5432->5432/tcp                       airflowcode66a665_postgres_1
```

These containers will mount volumes for their respective metadata.

```
docker volumes ls

DRIVER              VOLUME NAME
local               airflow6f4ef6_airflow_logs
local               airflow6f4ef6_postgres_data
local               airflowcode66a665_airflow_logs
```

To enter one of these containers:

```
docker exec -it c572fe53093e /bin/bash


bash-4.4$ ls
Dockerfile             airflow.cfg            airflow_settings.yaml  dags                   include                logs                   packages.txt           plugins                requirements.txt       unittests.cfg
bash-4.4$

```

### Running Commands on Build

Any extra commands you want to run when the image builds can be added in the `Dockerfile` as a `RUN` command - these will run as the last step in the image build.

For example, suppose I wanted to run `ls` when my image builds. My `Dockerfile` could look like:

```
FROM astronomerinc/ap-airflow:0.8.2-1.10.2-onbuild
RUN ls
```

### Adding Dependencies

Any additional python packages or OS level packages can be added in `requirements.txt` or `packages.txt`. For example, suppose I wanted to add `pymongo` into my Airflow instance.

This can be added to `requirements.txt`, run `astro airflow stop` and `astro airflow start` to rebuild my image with this new python package. Every package in the `requirements.txt` file will be installed by `pip` when the image builds (`astro airflow start`).

```
docker exec -it dff1aaef15cf pip freeze | grep pymongo
pymongo==3.7.2
```
This package is now installed.

A list of default packages included in the Astronomer base image can be found [here](https://forum.astronomer.io/t/which-python-packages-come-default-with-astronomer/38).


**Note:** We run Alpine linux as our base image so you may need to add a few os-level packages in `packages.txt` to get your image to build. You can also "throw the kitchen sink" at it if image size is not a concern:

```
libc-dev
musl
libc6-compat
gcc
python3-dev
build-base
gfortran
freetype-dev
libpng-dev
openblas-dev
gfortran
build-base
g++
make
musl-dev
```

In the same way, you can add a folder of `helper_functions` (or any other files for your DAGs to use) to build into your image. To do so, just add the folder into your project directory and rebuild your image.


```
virajparekh@orbiter:~/cli_tutorial$ tree
.
├── airflow_settings.yaml
├── dags
│   └── example-dag.py
├── Dockerfile
├── helper_functions
│   └── helper.py
├── include
├── packages.txt
├── plugins
│   └── example-plugin.py
└── requirements.txt
```

Now going into my scheduler image:
```
docker exec -it c2c7d3bb5bc1 /bin/bash
bash-4.4$ ls
Dockerfile             airflow_settings.yaml  helper_functions       logs                   plugins                unittests.cfg
airflow.cfg            dags                   include                packages.txt           requirements.txt
```


Notice the `helper_functions` folder has been built into the image.

You can also pass direct Airflow CLI commands into your local image following this pattern:

For example, a connection can be added with:

```bash
docker exec -it SCHEDULER_CONTAINER bash -c "airflow connections -a --conn_id test_three  --conn_type ' ' --conn_login etl --conn_password pw --conn_extra {"account":"blah"}"
```

## Environment Variables (--env)

Astronomer's v0.7.5-2 CLI comes with the ability to  bring in Environment Variables from a specified file by running `astro airflow start` with an `--env` flag as seen below:

```
astro airflow start --env .env
```

**Note**: This feature is currently only functional for local development. Whatever `.env` you use locally will _not_ be bundled up when you deploy to Astronomer. To add Environment Variables when you deploy to Astronomer, you'll have to add them via the Astronomer UI (`Deployment` > `Configure` > `Environment Vars`).

**Guidelines:**

1. Add your Environment Variables of choice in an `.env` file

2. Airflow configuration variables found in [`airflow.cfg`](https://github.com/apache/incubator-airflow/blob/master/airflow/config_templates/default_airflow.cfg) can be overwritten with the following format:

```
AIRFLOW__SECTION__PARAMETER=VALUE
```
For example, setting `max_active_runs` to 3 would look like:

```
AIRFLOW__CORE__MAX_ACTIVE_RUNS=3
```

3. Confirm the Environment Variable configuration names match up with the version of Airflow you're using

4. The CLI will look for `.env` by default, but if you have different settings you need to toggle between and want to specify multiple .env files, you can do following:

```
my_project
  ├── .astro
  └──  dags
    └── my_dag
  ├── plugins
    └── my_plugin
  ├── .env
  ├── dev.env
  └── prod.env
```

 5. On `astro airflow start`, just specify which file to use (if not `.env`) with the `--env` or `-e` flag.

 ```
 astro airflow start --env dev.envs
 astro airflow start -e prod.env
 ```

## CLI Debugging

### Error on Building Image

If your image  is failing to build after running `astro airflow start`?

 - You might be getting an error message in your console, or finding that Airflow is not accessible on `localhost:8080/admin`
 - If so, you're likely missing OS-level packages in `packages.txt` that are needed for any python packages specified in `requirements.text`
