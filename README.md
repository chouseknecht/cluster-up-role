[![Build Status](https://travis-ci.org/chouseknecht/cluster-up-role.svg?branch=master)](https://travis-ci.org/chouseknecht/cluster-up-role)

# cluster-up-role

Install the OpenShift client, and create a local instance using `oc cluster up`. 

Created to support the demo and testing needs of [Ansible Container ](https://github.com/ansible/ansible-container) by automating the tasks in the [Install and Configure OpenShift guide](http://docs.ansible.com/ansible-container/configure_openshift.html). 

Specifically, it performs the following tasks:

- Downloads and installs the oc client
- Installs `socat`, if running on OSX
- Adds a hostname associated with your public IP to /etc/hosts
- Starts the cluster
- Grants cluster admin to the *developer* account
- Creates a route to expose the local registry
- Creates a persistent volume
- Using the `oc` client, logs into the cluster as *developer*, and sets the project to *default* 

### Supported platforms and testing

To date this role has really only been tested on OSX using Docker for Mac. And it actually almost works on Travis, which is an Ubuntu platform. So with that in mind, if you're attempting to use it outside of OSX, you're very likely to discover a bug. If you do, please, open an issue or submit a PR, so that we can keep the role up to date. 

### Hostname

By default the hostname `local.opeshift` is added to your */etc/hosts* file, and associated with your current IP address. Use the *openshift_hostname* parameter, if you prefere a different name.

When the cluster is created, it gets associated with your local network IP address. If you're working on a laptop or other mobile device, you may find yourself having to recreate the cluster whenver you hop to a new network. Creating a hostname that's associated with your actual IP address makes life a little less painful.

### Insecure registry

If you have not added the insecure registry option to Docker, the role will error the first time you execute it. It will provide a message letting you know the subnet that needs to be added. You'll also need to add the *openshift_hostname* value. By default the value is *local.openshift*. After making the change and restarting Docker, run the role again, and this time it will run all the way through.

## Prerequisites 

You'll need to have the following installed:

- Docker Engine or Docker for Mac
- sudo access, for updating */etc/hosts*, and installing the `oc` binary to */usr/local/bin*.

**NOTE**: If you're on a Linux platform, be sure to follow the [create a docker group](https://docs.docker.com/engine/installation/linux/centos/#/create-a-docker-group) instructions, so that you're able to run `docker` commands directly without using *sudo*. 


## Example Playbook

When you run the role, be sure to leave gather_facts set to a truthy value. Without facts, the role cannot determine the host's IP address, nor the OS family. 

Below is a sample playbook that includes all of the default parameters. You'll find this exact example in [files/cluster-up.yml](./files/cluster-up.yml). Copy, and adjust it to fit your environment.

```
    ---
    - hosts: localhost
      remote_user: root
      connection: local
      gather_facts: yes
      roles:
        - role: chouseknecht.cluster-up-role
          openshift_github_user: openshift
          openshift_github_name: origin
          openshift_github_url: https://api.github.com/repos
          openshift_release_tag_name: ""
          openshift_client_dest: /usr/local/bin  
          openshift_force_client_install: yes
          openshift_volume_name: project-data
          openshift_volume_path: "{{ lookup('env','HOME') }}/volumes/project/data"
          openshift_hostname: local.openshift
          openshift_recreate: yes
```

After you install the role, copy *file/cluster-up.yml* to your project directory, and execute it with the `--ask-sudo-pass` option. Here's an example of what that might look like:

```
# Install the role 
$ ansible-galaxy install chouseknecht.cluster-up-role

# Copy the playbook from your roles path to the current working directory 
$ cp ${ANSIBLE_ROLES_PATH}/chouseknecht.cluster-up-role/files/cluster-up.yml .

# Create a localhost inventory file
$ echo "localhost">./inventory

# Run the playbook
$ ansible-playbook -i inventory --ask-sudo-pass cluster-up.yml
```

## Deploying your Ansible Container project

In the following example we'll create a new project, install the Container Enabled role [jenkins-container](https://galaxy.ansible.com/awasilyev/jenkins-container/), and deploy the Jenkins service to our local OpenShift cluster.

**NOTE**: to run this example, you will need to install Ansible Container 0.3.0. See [Installing from Source](http://docs.ansible.com/ansible-container/installation.html#running-from-source), if you need assistance.

```
# Create a new project folder
$ mkdir jenkins

# Set the working directory
$ cd jenkins

# Init the project
$ ansible-container init

# Install the jenkins-container role
$ ansible-container install awasilyev.jenkins-container

# Build the images
$ ansible-container build

# Generate the deployment playbook and role
$ ansible-container shipit openshift --local-images

# Set the working directory to ansible
$ cd ansible

# Run the shipit playbook
$ ansible-playbook shipit-openshift.yml
```

The above created a new project on OpenShift called `jenkins`. To veiw the project, log into the OpenShift console by opening [https://local.openshift:8443/console](https://local.openshift:8443/console). The username is `developer`, and the password is `developer`. Click on `jenkins` to view the project overview.

Click the following image to watch a video of the Jenkins service deployment:

[![Deploy Jenkins](https://github.com/chouseknecht/cluster-up-role/blob/images/images/deploy_jenkins.png)](https://www.youtube.com/watch?v=FQY8hQ-cB1c)

## Role Variables

Use the following variables to modify the role's behavior:

openshift_github_user: openshift
> The owner of the GitHub repo where oc client download targets can be found

openshift_github_name: origin
> The name of the GitHub repo  

openshift_github_url: https://api.github.com/repos
> The GitHub API endpoint to use.

openshift_release_tag_name: ""
> The tag for the desired relase of the `oc` binary. If not provided, the latest release will be installed.

openshift_client_dest: /usr/local/bin  
> The directory where `oc` will be installed. Needs to be something in your PATH.

openshift_force_client_install: yes
> If the `oc` binary already exists, should it be overwritten?

openshift_volume_name: project-data
> Name of the volume.

openshift_volume_path: "{{ lookup('env','HOME') }}/volumes/project/data"
> A local path where space will be allocated for the new volume. 

openshift_hostname: local.openshift
> The hostname you'll use to reference the local registry when you're ready to push images.

openshift_recreate: yes
> If a cluster is already running, should it be killed and recreated? 

openshift_up_options: ''
> Add any options you want to pass to `oc cluster up`. Separate multiple options with a space, just as you would on the command line.

## Dependencies

None

## License

Apache v2

## Author 

[@chouseknecht](https://github.com/chouseknecht)
