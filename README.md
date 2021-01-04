# Packer

## Amazon Machine Image (AMI)

An AMI is a template that contains a software configuration, for example
- operating system
- application server
- application

From an AMI, launch an _instance_, which is a copy of the AMI running as a virtual server in the cloud

Building an AMI is a useful way to take a snapshot of a machine and its data as a template to quickly and efficiently build multiple EC2 instances

### AMIs and DevOps

For businesses and companies with exponential growth of client demand, tools such as Terraform can help with autoscaling to increase server capacity automatically to meet demand (load balancing)
- Terraform uses the configured AMIs to scale out with multiple EC2 instances, handling all of the necessary networking and deployment

## Packer

Packer is a tool to automate the creation of any type of machine image

Works with a cloud provider (AWS, GCP, Azure) and provisioner (Ansible, shell, chef)

[Installing Packer](https://learn.hashicorp.com/tutorials/packer/getting-started-install)

Packer files are written in JSON
```JSON
{
  "variables": {
    "aws_access_key": "",
    "aws_secret_key": ""
  },
  "builders": [
    {
      "type": "amazon-ebs",
      "access_key": "{{user `aws_access_key`}}",
      "secret_key": "{{user `aws_secret_key`}}",
      "region": "us-east-1",
      "source_ami_filter": {
        "filters": {
          "virtualization-type": "hvm",
          "name": "ubuntu/images/*ubuntu-bionic-18.04-amd64-server-*",
          "root-device-type": "ebs"
        },
        "owners": ["099720109477"],
        "most_recent": true
      },
      "instance_type": "t2.micro",
      "ssh_username": "ubuntu",
      "ami_name": "packer-example {{timestamp}}"
    }
  ]
}
```
The AWS keys can be interpolated using environment variables by exporting the keys to ``~/.bashrc`` and in the packer file
```JSON
{
  "variables":{
    "aws_access_key": "{{env `AWS_ACCESS_KEY`}}",
    "aws_secret_key": "{{env `AWS_SECRET_KEY`}}"
  }
```
}

**CHECK PACKER FILE IS VALID**
```bash
packer validate filename.json
```

**BUILD PACKER FILE**
```bash
packer build filename.json
```

### Provisioning with Packer

Provisioning with Packer allows users to install and configure software into the images as well appending into the packer file
```JSON
{
  "variables": ["..."],
  "builders": ["..."],

  "provisioners": [
    {
      "type": "ansible",
      "playbook_file": "path/to/playbook.yaml"
    },
    {
      "type": "shell",
      "inline": [
        "sudo apt-get update",
        "sudo apt-get install"
      ]
    }
  ]
}
```

- For ``"type": "ansible"``
    - specify the path for the playbook
    - **NOTE: HOSTS IN THE PLAYBOOK SHOULD BE SET TO DEFAULT**
- For ``"type": "shell"``
    - a list of commands can be specified