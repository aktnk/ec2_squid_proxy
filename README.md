# ec2_squid_proxy
Using Terraform, build a Squid proxy server on AWS EC2.

# How to use

##  Prerequisites

* Running environment : ubuntu 20.04 LTS on wsl2 
* Docker has been installed.
* Executed terraform account configuration by aws configure command.
  ```
  $ aws configure --profile [terraform-account]
  ```
* The environment variable AWS_PROFILE in the .envrc file has been set to [terraform-account] configured above.
  ```
  export AWS_PROFILE=********
  ```

## Usage

1. Preparation
  * Create the squid.conf file according to requirements.
    **Note:**  
    The following line in squid.conf must be changed to the address or network address of the PC that will use this proxy. Otherwise , an error will occur when executing terraform command.
    ```
    acl localnet src <my pc grobal ip address/32>
    ```
  * Place the created file in source/config directory.

2. Initialize
  * Start terraform container
    ```
    $ docker-compose up -d
    ```
  * Enter the terraform container
    ```
    $ docker-compose exec terraform ash
    ```
  * Initializing the terraform environment
    ```
    /workdir # terraform init
    ```
3. Create and edit tf file

You should modify variables.tf according to the VPC configuration you wish to build.

**Note:**  
Access to this server is only allowed from the global IP address assigned to the environment in which terraform is run.

3. run terraform
  * Verify the description of the tf file in the local environment.
    ```
    /workdir # terraform validate
    ```
  * Check for changes.
    ```
    /workdir # terraform plan
    ```
  * Apply the changes and start up the test server.  
  When you are asked if you are sure you want to run it, type `yes` if you are sure.
    ```
    /wordkfir # terraform apply
    ```
  **Note:**  
  When the execution of terraform apply is completed, the IP address and Public DNS name of the server started will be displayed as shown below.
  ```
    Enter a value: yes


  Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

  Outputs:

  ami_id = "ami-04d6e43069fd7e366"
  ec2_global_ips = [
    "*.*.*.*",
  ]
  ec2_public_dns_name = [
    "ec2-*-*-*-*.ap-northeast-1.compute.amazonaws.com",
  ]
  /workdir #
  ```
4. check

  * Access the instance via SSH 
  
  ```
  /workdir # chmod 400 .security/my-net-keypair.id_rsa
  /workdir # ssh -i .security/my-net-keypair.id_rsa ec2-user@ec2-*-*-*-*.ap-northeast-1.compute.amazonaws.com

        __|  __|_  )
        _|  (     /   Amazon Linux 2 AMI
        ___|\___|___|

  https://aws.amazon.com/amazon-linux-2/
  [ec2-user@ip-**-**-**-** ~]$ ls -la
  total 12
  drwx------ 3 ec2-user ec2-user  74 May  6 15:38 .
  drwxr-xr-x 3 root     root      22 May  6 15:38 ..
  -rw-r--r-- 1 ec2-user ec2-user  18 Jul 15  2020 .bash_logout
  -rw-r--r-- 1 ec2-user ec2-user 193 Jul 15  2020 .bash_profile
  -rw-r--r-- 1 ec2-user ec2-user 231 Jul 15  2020 .bashrc
  drwx------ 2 ec2-user ec2-user  29 May  6 15:38 .ssh
  [ec2-user@ip-*-*-*-* ~]$ exit
  logout
  Connection to ec2-*-*-*-*.ap-northeast-1.compute.amazonaws.com closed.
  ```

5. terminate ec2 instance
  * Delete the test server.  
  You will be asked to confirm whether you want to execute it or not.
    ```
    /workdir # terraform destory
    ```

6. terminate terraform container
  * Exit terraform container.
    ```
    /workdir # exit
    ```
  * Stop terraform container.
    ```
    $ docker-compose stop
    ```
  **Note**  
  If you want to restart the terraform container, run `$ docker-compose start`.

# References

In constructing this environment, I referred to the following articles.

* [dockerhub : hashicorp/terraform](https://hub.docker.com/r/hashicorp/terraform)
* [github : terraform稼働開発環境](https://github.com/naritomo08/terraform_docker_public.git)
* [Terraform入門 : AWSのVPCとEC2を構築してみる](https://kacfg.com/terraform-vpc-ec2/)

