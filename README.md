# Docker Compose project example

This is an example project with some explanation of how you can make the best use of compose features when your stack grows. I've created it as I could not find any simple, easy to understand and "raw" example myself that would cover all the information you can find in the web.

## Running the project

A simple `docker compose up` is enough as this solution is using purely Docker provided features. This repository contains just an example project but it is runnable if needed (unless you have port binding conflicts).

Tested on Ubuntu Linux.

## The main features covered

### The `include` attribute for easier maintenance and clear project structure

`Note: Please note this requires Docker Compose version 2.20.0 or later. So you might need to update especially if you are on Synology device as they tend to be behind.`

The more services you have the bigger your `docker-compose.yml` file gets. To the point that it can be hard to maintain. With [include](https://docs.docker.com/compose/compose-file/14-include/) attribute you can split your compose file into smaller chunks and group them as you like. Each of those "groups" can have its own `.env` files and extra directories. Take control of that mess!

- Your `docker-compose.yml` file from now on will be meant to act as a central point of your whole stack and define as few settings as possible, mainly list all the files that your whole stack contains by using [include](https://docs.docker.com/compose/compose-file/14-include/) attribute.
- Each of those "groups" can have its own `env_file` list as well. This way your environment configuration related to a specific service can be in its own file, next to its compose file.
- The main `.env` file will contain only the common environment settings that you usually define for multiple service such az `TZ`, `PUID`, `PGID`. Additionally i've added `INCLUDE_PATH` so that it's easier to define paths to your compose files with `include` attribute like this:
  
  ```yaml
  include:
    - path:
        - ${INCLUDE_PATH}/admin/pihole.yml
  ```

### The `extends` attribute for inheritance or templating

With [extends](https://docs.docker.com/compose/multiple-compose-files/extends/) you can easily define common settings for all your services in central place, globally. No more code repetition.

- The `.env` file contains the `TEMPLATES_PATH` variable. It points to abolute path of `docker-compose.templates.yml` file. It just makes it easier to define the confguration using [extends](https://docs.docker.com/compose/multiple-compose-files/extends/) attribute. Each of your services simply needs to define it as follows:
  
  ```yaml
  extends:
      file: ${TEMPLATES_PATH}
      service: default
  ```

  Of course you can have more services defined to inherit from.
- In the `docker-compose.templates.yml` file itself I've put the common settings for my containers in the base service called `default`. It contains settings such as logs, security or even the restart mode. No need to repeat those for each service anymore!

## Alternative approaches

There are other ways you can manage your docker stack that I did not use myself.

### Having multiple docker-compose.yml files

This is possibly the most basic and most common approach. You can split your stack into multiple `docker-compose.yml` files (each in its own directory!) where each of them would contain a set of containers that are related to each other (such as appp + db) and then use the `-f` flag to point to the file you wish to manage. For example `docker compose -f project/admin/pihole.yml up`. Simple and works but it has some limitations:

- You have to manage each of the "Stacks" separately and they do not know about each other. This makes having common configuration or templates or using `depends_on` more complicated.
- If you edit your stack a lot and use `--remove-orphans` to clean the leftovers then deploying one stack like that will remove all other stacks. Because at that point Docker thinks that all you want to be up and running is that single stack.

This solution is pretty much same as the approach I explain in this repo but the files are just not "tied" together by using `include`.

### Combining docker-compose.yml files by special ENV variables

This is what I was using initally because of its simplicity. It is possible to create an `.env` file in your project root directory and there to define `COMPOSE_FILE` and/or `COMPOSE_ENV_FILES` where you can define multiple files. [Set or change pre-defined environment variables in Docker Compose](https://docs.docker.com/compose/environment-variables/envvars/).

Example `.env` file:

```bash
COMPOSE_FILE=project/admin/pihole.yml:project/media/jellyfin.yml:project/web/server.yml
COMPOSE_ENV_FILES=project/admin/.env,project/media/.env,project/web/.env
```

### YAML Anchors and Aliases

The [fragments](https://docs.docker.com/compose/compose-file/10-fragments/) and [extensions](https://docs.docker.com/compose/compose-file/11-extension/) were not used. Both of them make use of the [Anchors and Aliases](https://docs.docker.com/compose/compose-file/11-extension/) which are built-in YAML features. The reason that these were not used is that anchors and aliases do not work with multiple files which is the main principle of this example (for example you can define aliases in one common file and then reference tham in other files). They can still be defined and used in each of your files individually if it will be useful for you. Personally, I believe this limitation was the main driver for docker team to add [extends](https://docs.docker.com/compose/multiple-compose-files/extends/) attribute which overcomes those limitations and thats why I prefered it this way.

Still, there is a "workaround" that would allow you to use YAML anchors with multiple files. The way to do it would be to have an additional script that you would use instead of running `docker compose update` etc. directly and that would combine all your compose files into one before running docker command. An example implementation would look like this `cat docker-compose.yml project/admin/pihole.yml project/admin/watchtower.yml etc... | docker compose -f - up` (The `-f -` makes compose read `docker-compose.yml` content from `stdin` instead of actual file). Although I did not test it myself. You could also find a better way to collect recursively file contents other that `cat`.

### Portrainer

Some people like to use [Portainer](https://www.portainer.io). It is a big project on its own so I will not go much into details here. Generally you can deploy it to your server and manage your stack with a nice GUI and many, many options. Their solution to the problem of having too big docker stack is actually [Stacks](https://www.portainer.io/blog/stacks-docker-compose-the-portainer-way) which is their own alternative to docker-compose files which allows advanced templating etc. I did not like it because it seemed to me to be an overkill for my simple home server, also I wanted to follow pure docker solution and "Stacks" is a Portainer product.

### Dockge

I know [Dockge](https://github.com/louislam/dockge) can be also used but I did not try it myself. Check releases page because people complain about lack of development.

### Compose Farm

If you need to run docker compose on multiple hosts (run `compose up` via ssh in multiple destinations) then youy can try [Compose Farm](https://github.com/basnijholt/compose-farm/)

### Komodo

Another one I found that might be worth checking is [Komodo](https://komo.do/).
