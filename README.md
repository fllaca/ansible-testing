# Workshop - Testing Ansible Roles

## "Unit" Testing

TODO

## Smoke Tests

Smoke tests are verification checks that you perform after a deployment to see that everything was OK. In our scenario, we can perform some kind of "functional" or "acceptance" test for our ansible roles by using a Test playbook.

## Server Validation GOSS Tests

There are a lot of tools and libraries for evaluating server status at low level, like checking running processes, listening ports, services running, etc. GOSS is a very easy-to-use alternative that allows you to write this kind of specifications in **YAML** format.

## Other tools

### KitchenCI

### Molecule

### Vagrant

## Proposed Exercises

### Requirements

* a Git client
* Docker and DockerCompose
* Your favourite code editor.
* Not mandatory, but you would need to install `sshpass` in your PC for some exercises.

### Exercises
We will use a Sample Ansible Role: ansible-role-httpd. It is very simple: just installs and runs the Apache **httpd** service.

#### 0 - Preparation

1. Run our "fake" server: `docker run -d --name instance --privileged chrismeyers/centos7`
1. Get the container IP: `docker inspect --format='{{.NetworkSettings.IPAddress}}' instance`
1. Write a simple inventory file to use later. The ssh password of the container is `docker.io`.  

#### 1 - Write two playbooks
First,we will write a playbook, `verify.yaml`, that will execute following tests on the fake server:
1. Wait for port 80
1. Send a request to 80 port asking for `index.html`.

Of course, this playbook will fail, because our fake server is "empty".

Now write a playbook to run the role in our fake server. We will call this playbook `converge.yaml`. Run it on the server, and then rerun the verify playbook. Ey... this looks like "Ansible TDD", isn't it? ;).

NOTE: if you don't want to install Ansible, you can use Ansible tester image like this:
```
docker run --rm -it \
    -v $(pwd):/ansible-role-web-server \
    --link instance \
    -w /ansible-role-web-server
    fllaca/ansible-tester \
    ansible-playbook -i tests/inventory tests/converge.yaml
```

#### 2 - Write the Docker-Compose file
Now we are going to write a `docker-compose.yml` file that will start two containers: one with an empty fresh Centos 7 instance, and another one with a special Ansible Docker Image that will run the tests: `fllaca/ansible-tester:latest`.

NOTES: 
* For the Centos 7 container you can use this Docker Image: `chrismeyers/centos7`
* The Centos 7 container must start in privileged mode.
* The Ansible tester container must have the code of the role, mounted in a volume.

#### 3 - Run converge and verify inside Ansible tester container
Use the `docker-compose run` command to execute the playbooks in the Ansible tester container.

#### 4 - Write the Makefile with the tests
Create a `Makefile` with different targets to run all what we did before:
* setup: starts docker-compose containers
* clean: kills the docker-compose containers
* converge: run the converge playbook
* verify: run the verify playbook
* test: run setup-converge-verify in a sequence.