# Podman Lmpp Stack
Trying to setup a complete rootless Lmpp (Linux MariaDB PHP phpMyAdmin) Stack on Podman for web development purposes

*Note: I am new to Podman and containers in general, I haven't tried to compose or build images yet. This is still a work in progress.*

## *V0.1*

### 1. Install [Podman](https://podman.io/)

### 2. Install [slirp4netns](https://github.com/rootless-containers/slirp4netns) to allow networking for unprivileged network namespaces.

### 3. Open your shell in the root folder of your project and type the following commands

`podman pod create --name lmpp-stack --publish 8000:8000 --publish 80:8080 --publish 3306:3306`
> Creates a pod and opens the ports to listen to PHP, phpMyAdmin and MariaDB.

`podman run --detach --pod lmpp-stack --name mariadb --env MARIADB_ROOT_PASSWORD=root --volume "$PWD"/database:/var/lib/mysql:Z mariadb:latest`
> Creates a container with the latest version of MariaDB inside the pod. Gives a default 'root' password and the data from the database will be stored in the /database folder.

`podman run --detach --pod lmpp-stack --name php-cli --volume "$PWD"/code:/var/www/html:Z php:8.3-cli`
> Creates a container with PHP 8.3 and its builtin webserver inside the pod. By default you need your codebase to be located in the /code folder.

`podman exec php-cli /bin/bash -c 'docker-php-ext-install pdo pdo_mysql'`
> Installs the PDO and PDO-MYSQL extensions to the PHP container.

`podman exec php-cli /bin/bash -c 'php -m | grep -i pdo'`
> Helps you verify if the extensions are correctly installed.

`podman exec php-cli /bin/bash -c 'pecl install xdebug && docker-php-ext-enable xdebug'`
> Installs the Xdebug extension to the PHP container.

`podman exec php-cli /bin/bash -c 'php -m | grep -i xdebug'`
> Helps you verify if the Xdebug extension is correctly installed.

`podman run --detach --pod lmpp-stack --name phpmyadmin --env PMA_HOST=127.0.0.1 --env PMA_PORT=3306 phpmyadmin:latest`
> Creates a container with phpMyAdmin with the latest version inside the pod and connects it to MariaDB.

### 4. Usage

`podman pod start lmpp-stack`
> Starts the pod and all its containers. *Note: the pod already started when we ran the first container above.* 

`podman exec php-cli /bin/bash -c 'php -S localhost:8000'`
> Might be needed to start the builtin webserver.

`podman pod stop lmpp-stack`
> Stops the pod and all its containers.

### 5. Pod Removal

In case something wrong happened and/or you want to remove the pod and its containers, use the `podman pod stop lmpp-stack` command first then:

`podman pod rm lmpp-stack`
> Removes the pod and all its containers.

`podman pod list`
> Shows all the existing pods. Helps you check if the pod has been correctly removed.

`podman ps --pod`
> Shows all the currently running containers in a pod. Help you check if all the containers have been correctly stopped.

`podman rm -f <container_id>`
> Forcefully stops and removes a container. It should not be needed since `podman pod rm lmpp-stack` should have removed the containers as well.

`podman images`
> Shows the downloaded images used to run the containers.

`podman rmi <image_id>`
> Removes the image.

