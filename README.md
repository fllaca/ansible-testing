# Workshop - Testing Ansible Roles

As the Ansible docs about [Testing Strategies](http://docs.ansible.com/ansible/latest/test_strategies.html) says:

> Ansible resources are models of desired-state. As such, it should not be necessary to test that services are started, packages are installed, or other such things. Ansible is the system that will ensure these things are declaratively true. Instead, assert these things in your playbooks.

But, sometimes, we may want to perform some validations to verify that an Ansible role is doing the things we expect. In this case, we are talking about **Functional or Acceptance tests**, that have to be ran on a previously Ansible provisioned environment. This is the approach that most testing frameworks for infrastructure follow:

```
+-------------------------------+       +-----------------------------+    +------------------------------+
|                               |       |                             |    |                              |
|  Generate Test Environment    +------->   Run Role (test Playbook)  +---->   Execute checks on the Env  |
|                               |       |                             |    |                              |
+-------------------------------+       +-----------------------------+    +------------------------------+
```

Rather than "check" that some software is present, we try to consume that software and verify that we get the correct outputs.

The following sections describe some frameworks and tools reviewed for this kind of testing purposes.


## Tools and Frameworks

### Docker Provision

[Docker Provison](https://github.com/chrismeyersfsu/provision_docker) is an Ansible role that generates Docker containers and adds them to an "in-memory" Ansible inventory. Then we can write a "test" playbook that performs checks on that container-based inventory. It uses Docker Compose behind the scenes.

### Molecule

[Molecule](https://molecule.readthedocs.io/en/latest/usage.html) generates scenarios in different platforms to test roles using [TestInfra](http://testinfra.readthedocs.io/en/latest/) for specs written in Python. It also checks the syntax and idempotence of the role.

### KitchenCI

[KitchenCI](http://kitchen.ci/docs/getting-started) is a similar project to Molecule. It also has [drivers](https://docs.chef.io/kitchen.html) supporting a lot of infrastructure providers. By default it uses the Serverspec framework.

### Vagrant

An approach is to create a Vagrantfile in the test subfolder, and configure it to be provisioned with an ansible playbook `test.yml`, that would run the role under test and additional checks. The test is run with a `vagrant up --provision` command, which will start the VM and run the playbook on it.

### Comparison Table

| Framework/Tool | Pros | Cons |
|----------------|----------------|----------------|
| Docker Provision Role | - Easy to setup <br>  - Only Ansible Code | - Only runs containers <br> - Limited testing capabilities <br> - Docker only tool |
| Molecule | - Well integrated with Ansible <br> - Multiple platforms supported * <br> - write tests in Python(testinfra) <br> - Syntax and idempotence checks| - Limited to Ansible and Testinfra <br> - Hard to configure (many files) <br> - Limited community support |
| Kitchen | - Multiple platforms supported * <br> - Flexibility <br> - Good community support <br> -Easy to configure | - Code tests with Serverspec (Ruby) by default <br> - Not "Ansible oriented" |
|Custom Vagrantfile| - Easy to setup <br>  - Only Ansible Code | - Limited to Vagrant VMs <br> - Hard to integrate in the CI pipeline (Vagrant needed in the server) |
|Ansible Check Mode| Easy to Setup <br> - Only Ansible Code | - Does not generate a test environment (we have to combine with one of the above tools) |


(*): EC2, Openstack, Docker, Vagrant

## Three kind of tests

### "Unit" Testing

We can't really talk about actual unit tests for Ansible role, but we propose an approach that can be analagous to the way we unit test functions of a program. We can think of an Ansible role as an interface with inputs, the variables, and a result, that is the state of the machine. As we mock dependencies in the code we can "mock" servers by using containers or virtual machines, and then perform validations on those instances. In the last section of this document, we will do some exercises of this kind of tests.

### Smoke Tests

Smoke tests are verification checks that you perform after a deployment to see that everything was OK. In our scenario, we can perform some kind of "functional" or "acceptance" test for our ansible roles by using a Test playbook. Ansible playbooks can be ran in [check mode](http://docs.ansible.com/ansible/latest/playbooks_checkmode.html), which only performs facts gathering and validations

### Server Validation - GOSS Tests

There are a lot of tools and libraries for evaluating server status at low level, like checking running processes, listening ports, services running, etc. [GOSS](https://github.com/aelsabbahy/goss) is a very easy-to-use alternative that allows you to write this kind of specifications in **YAML** format.

## Proposed Exercises

### Requirements

* a Git client
* Docker and DockerCompose
* Your favourite code editor.
* Not mandatory, but you would need to install `sshpass` in your PC for some exercises.

### Exercises
We will use a Sample Ansible Role: ansible-role-httpd. It is very simple: just installs and runs the Apache **httpd** service. 

You'll see in this repository two folders:

* **ansible-role-web-server:** containes the full complete role after exercises completion. We recommend to only wath this folder to validate and compare your answers.
* **ansible-role-web-server:** Is the same folder above, but with some empty files and sections to be completed in the next exercises.

#### 0 - Preparation

1. Run our "fake" server: `docker run -d --name instance --privileged chrismeyers/centos7`
1. Get the container IP: `docker inspect --format='{{.NetworkSettings.IPAddress}}' instance`
1. Write a simple inventory file to use later. The ssh password of the container is `docker.io`.  

#### 1 - Write two playbooks: converge and verify
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