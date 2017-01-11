[![Build Status](https://travis-ci.org/chouseknecht/cluster-up-role.svg?branch=master)](https://travis-ci.org/chouseknecht/cluster-up-role)

# cluster-up-role

Install the OpenShift client and create a local instance using `oc cluster up`. 

Created to support the demo and testing needs of [Ansible Container ](https://github.com/ansible/ansible-container) by automating the tasks in the [Install and Congigure OpenShift guide](http://docs.ansible.com/ansible-container/configure_openshift.html). 

Specifically, it performs the following tasks:

- Downloads and installs the oc client
- Installs `socat`, if running on OSX
- Adds a hostname associated with your public IP to /etc/hosts
- Starts the cluster
- Grants cluster admin to the *developer* account
- Creates a route to expose the local registry
- Creates a persistent volume
- Connects the `oc` client to the cluster as the developer, and sets the project to `default` 

### Supported platforms and testing

To date this role has really only been tested on OSX using Docker for Mac. And it actually almost works on Travis, which is an Ubuntu platform. So with that in mind, if you're attempting to use it not on a OSX, you're very likely to discover a bug. If you do, please, open an issue, and let us know, so that we can keep the role up to date and make adjustements. 

## Prerequisites 

- Docker or Docker for Mac
- sudo access, for updating */etc/hosts*, and installing the `oc` binary to */usr/local/bin*.

### Hostname

By default the hostname `local.opeshift` is added to your */etc/hosts* file, and associated with your current IP address. Use the *openshift_hostname* parameter, if you prefere a different name.

When the cluster is created, it gets associated with your local network IP address. If you're working on a laptop or other mobile device, you may find yourself having to recreate the cluster whenver you hop to a new network. Creating a hostname that's associated with your actual IP address makes life a little less painful.

### Insecure registry

If you have not added the insecure registry option to Docker, the role will error the first time you execute it. It will provide a message letting you know the subnet that needs to be added. You'll also need to add the *openshift_hostname* value. By default the value is *local.openshift*. After making the change and restarting Docker, run the role again, and this time it will run all the way through.

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

To view the project, use a browser to log into the OpenShift console at `https://local.openshift:8443/console`. The username is `developer`, and the password is `developer`. If you followed the example above, you will see a `jenkins` project. Click on `jenkins` to open the project overview.

Click the following image to watch a video of the Jenkins service deployment:

## Role Variables

Use the following variables to modify the role's behavior:

openshift_github_user: openshift
> The owner of the GitHub repo where oc client download targets can be found

openshift_github_name: origin
> The name of the GitHub repo  

openshift_github_url: https://api.github.com/repos
> The GitHub API endpoint to use.

openshift_release_tag_name: "v1.3.1"
> The tag for the desired relase of the `oc` binary. If none provided, the latest release will be installed.

openshift_client_dest: /usr/local/bin  
> The directory where `oc` will be installed. Needs to be something in your PATH.

openshift_force_client_install: no
> If the `oc` binary already exists, should it be overwritten?

openshift_volume_name: project-data
> Name of the volume.

openshift_volume_path: "{{ lookup('env','HOME') }}/volumes/project/data"
> A local path where space will be allocated for the new volume. 

openshift_hostname: local.openshift
> The hostname you'll use to reference the local registry when you're ready to push images.

openshift_recreate: no
> If a cluster is already running, should it be killed and recreated? 

openshift_up_options: ''
> Add any options you want to pass to `oc cluster up`. Separate multiple options with a space, just as you would on the command line.

## Dependencies

None

## Example Playbook

When you run the role, be sure to leave gather_facts set to a truthy value. Without facts, the role cannot determine your default IP address or OS family. 

Below is a sample playbook that includes all of the default parameters. You'll find this exact example in the *files* folder, called *cluster-up.yml*. Copy, and adjust it to fit your environment.

When you run the playbook, be sure to pass the ``--ask-sudo-pass`` option.

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
          openshift_release_tag_name: "v1.3.1"
          openshift_client_dest: /usr/local/bin  
          openshift_force_client_install: no
          openshift_volume_name: project-data
          openshift_volume_path: "{{ lookup('env','HOME') }}/volumes/project/data"
          openshift_hostname: local.openshift
          openshift_recreate: no
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
$ ansible-container -i inventory --ask-sudo-pass cluster-up.yml
```

## License

Apache v2

## Author 

[@chouseknecht](https://github.com/chouseknecht)
