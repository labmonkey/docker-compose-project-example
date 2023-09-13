# Docker Compose project example

This is an example project with some explanation of how you can make the best use of compose features when your stack grows. I've created it as I could not find any simple, easy to understand and "raw" example myself that would cover all the information you can find in the web.

## The main features covered

### The `include` attribute for easier maintenance and clear project structure

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

## Features that could not be used

The [fragments](https://docs.docker.com/compose/compose-file/10-fragments/) and [extensions](https://docs.docker.com/compose/compose-file/11-extension/) were not used. Both of them make use of the [Anchors and Aliases](https://docs.docker.com/compose/compose-file/11-extension/) which are built-in YAML features. The reason that these were not used is that anchors and aliases do not work with multiple files which is the main principle of this example (for example you can define aliases in one common file and then reference tham in other files). They can still be defined and used in each of your files individually if it will be useful for you. Personally, I believe this limitation was the main driver for docker team to add [extends](https://docs.docker.com/compose/multiple-compose-files/extends/) attribute which overcomes those limitations and thats why I prefered it this way.
