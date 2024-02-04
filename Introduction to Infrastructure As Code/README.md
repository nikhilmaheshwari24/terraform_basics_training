# Introduction to Infrastructure As Code

## Challenges with IT Infrastructure

Let's start with how application delivery works in a traditional infrastructure model and how it evolved with the emergence of technologies such as cloud computing and Infrastructure as Code. 

Let's go back in time and look at how infrastructure was provisioned in the traditional IT model. Let us consider an organization that wants to roll out a new application. 
- The business comes up with the requirements for the application. 
- The business analyst then gathers the needs from the business, analyzes it, and converts it into a set of high-level technical requirements. 
- This is then passed on to a solution architect. The solution architect then designs the architecture to be followed for the deployment of this application. This would typically include the infrastructure considerations, such as the type, spec, and count of servers that are needed, such as those for front-end web servers, back-end servers, databases, load balancers, etcetera. 
Following the traditional infrastructure model, this would have to be deployed in the organization's on-premise environment, which would mean making use of the assets in the data center. 
- If additional hardware is needed, they would have to be ordered via the procurement team. This team will put in a new hardware request with the vendors. It can then take anywhere between a few days to weeks or even months for the hardware to be purchased and delivered to the data center. 
- Once received at that the data center, the field engineers would be in charge of rack and stack of the equipment. 
- The system administrators perform initial configurations and the network administrators make the systems available on the network. 
- The storage admins assign storage to the servers, and the backup admins configure backups. 
- Finally, once the systems have been set up as per the standards, they can then be handed over to the application teams to deploy their applications. This deployment model, which is still quite commonly used today, has quite a few disadvantages. 
- The turnover time can range between weeks to months, and that's just to get the systems in a ready state to begin the application deployment. This includes the time it takes for the system to be initially procured, and then handed over between teams. 
- Also, scaling up or scaling down the infrastructure on demand cannot be achieved quickly. 
- The overall cost to deploy and maintain this model is generally quite high. While some aspects of the infrastructure provisioning process can be automated, several steps such as the rack and stack, cabling, and other deployment procedures are manual and slow. 
- With so many teams working on so many different tasks, chances of human error are high, and this results in inconsistent environments. 
- Another major disadvantage of using this model is the underutilization of the compute resources. The infrastructure sizing activity is generally carried out well in advance and the servers are sized considering the peak utilization. 
- The inability to scale up or down easily means that most of these resources would not be used during off-peak hours. 

In the past decade or so, organizations have been moving to virtualization and cloud platforms to take advantages of services provided by major cloud providers such as Amazon, AWS, Microsoft Azure, Google Cloud Platform, etcetera. 
By moving to cloud, the time to spin up the infrastructure and the time to market for applications are significantly reduced. This is because with cloud, you do not have to invest in or manage the actual hardware assets that you normally would in case of a traditional infrastructure model. The data center, the hardware assets, and the services are managed by the cloud provider. A virtual machine can be spun up in a cloud environment in a matter of minutes, and the time to market is reduced from several months, as in the case of a traditional infrastructure, to weeks in a cloud environment. Infrastructure costs are reduced when compared with the additional data center management and human resources costs. Cloud infrastructure comes with support for APIs, and that opens up a whole new world of opportunity for automation. 
Finally, the built-in auto-scaling and elastic functionality of cloud infrastructure reduces resource wastage. With virtualization and cloud, you could now provision infrastructure with a few clicks. While this approach is certainly faster and more efficient when compared to the traditional deployment methods, using the management console for resource provisioning is not always the ideal solution. 

It is okay to have this approach when we are dealing with a limited number of resources, but in a large organization with elastic and highly scalable cloud environment with immutable infrastructure, this approach is not feasible. Once provisioned, the systems still have to go through different teams with a lot of process overhead that increases the delivery time. The chances of human error are still at large, resulting in inconsistent environments. 

Different organizations started solving these challenges within themselves by developing their own scripts and tools. While some use simple shell scripts, others used programming languages such as Python, Ruby, Perl, or PowerShell. Everyone was solving the same problems, trying to automate infrastructure provisioning to deploy environments faster, and in a consistent fashion by leveraging the API functionalities of the various cloud environments. These evolved into a set of tools that came to be known as Infrastructure as Code. In the next lecture, we will see what Infrastructure as Code is in more detail.

## Types of IAC Tools

In this lecture, we will get introduced to Infrastructure as Code, which is commonly known as IAC. We will also take a look at commonly used IAC tools. Earlier, we discussed about provisioning by making use of the management console of various cloud providers. 

The better way to provision cloud infrastructure is to codify the entire provisioning process. This way we can write and execute code, to define, provision, configure, update, and eventually destroy infrastructure resources. This is called as Infrastructure as Code or IAC. 

With Infrastructure as Code, you can manage nearly any infrastructure component as code, such as databases, networks, storage, or even application configuration. The code you see here is a shell script. However, it is not easy to manage it. It requires programming or development skills to build and maintain. There's a lot of logic that you will need to code, and it is not easily reusable. That's where tools like `Terraform` and `Ansible` help with code that is easy to learn, is human readable and maintain. 

A large shell script can now be converted into a simple Terraform configuration file like this. With Infrastructure as Code, we can define infrastructure resources using simple, human readable high-level language. Here is another example where we will be making use of Ansible to provision three AWS EC2 instances, making use of a specific AMI. Although Ansible and Terraform are IAC tools, they have some key differences in what they are trying to achieve, and as a result, they have some very different use cases. Here, we'll see these differences next. 

There are several different tools part of the Infrastructure as Code family; `Ansible`, `Terraform`, `Puppet`, `CloudFormation`, `Packer`, `SaltStack`, `Vagrant`, `Docker`, etcetera. Although you can possibly make use of any of these tools to design similar solutions, they have all been created to address a very specific goal. 

With that in mind, IAC can be broadly classified into three types; **`configuration management`**: Ansible, Puppet, SaltStack fall into this category. Tools used for **`server templating`**: Docker, Packer, and Vagrant fall into this category. Finally, we have **`infrastructure provisioning`** tools, such as Terraform and CloudFormation. 

Let's look at these in a bit more detail. The first type of IAC tool that we are going to take a look at is configuration management tools. These include tools like Ansible, Chef, Puppet, and SaltStack, and these are commonly used to install and manage software on existing infrastructure resources such as servers, databases, networking devices, etcetera. Unlike the ad-hoc shelf scripts that we saw earlier, configuration management tools maintain a consistent and a standard structure of code, and this makes it easier to manage and update it as needed. They are also designed to run on multiple remote resources at once. An Ansible playbook or a role can be checked into a version control repository. This allows us to reuse and distributed it as needed. However, perhaps the most important feature of a configuration management tool is that they are *`idempotent`*. This means that you can run the code multiple times and every time you run it, it will only make changes that are necessary to bring the environment into a defined state. It would leave anything already in place as it is, without us having to write any additional code. 

Next, let's look at server templating tools. These are tools like Docker, Vagrant, and Packer from HashiCorp that can be used to create a custom image of a virtual machine or a container. These images already contain all the required software and dependencies installed on them, and for the most part, this eliminates the need of installing software after a VM or a container is deployed. The most common examples for server templated images are VM images such as those that are offered on osboxes.org, custom AMIs in Amazon AWS, and Docker images on DockerHub and other container registries. Server templating tools also promote *`immutable`* infrastructure unlike configuration management tools. This means that, once the VM or a container is deployed, it is designed to remain unchanged. If there are changes to be made to the image, instead of updating the running instance, like in the case of configuration management tools such as Ansible, we update the image and then re-deploy a new instance using the updated image. We have a section on immutable infrastructure later in this course where we will look at it in much more detail. 

The last type of IAC tool which is specifically of interest for this course is provisioning tools. These tools are used to provision infrastructure components using a simple declarative code. These infrastructure components can range from servers such virtual machines, databases, VPCs, subnets, security groups, storage, and just about any services based on the provider we choose. While CloudFormation is specifically used to deploy services in AWS, Terraform is vendor agnostic and supports provider plugins for almost all major cloud providers. In the upcoming lecture, we will see how Terraform helps in provisioning infrastructure.

## Why Terraform?

Let's now talk about Terraform and go over some of its features at a high level. As we discussed, Terraform is a popular IAC tool which is specifically useful as an infrastructure provisioning tool. Terraform is a free and open source tool which is developed by `HashiCorp`. It installs as a single binary which can be set up very quickly, allowing us to build, manage, and destroy infrastructure in a matter of minutes. 

One of the biggest advantages of Terraform is its ability to deploy infrastructure across multiple platforms including private and public cloud, such as on-premise vSphere cluster or cloud solutions such as AWS, GCP, or Azure to name a few. These are just a few of the many resources that Terraform can manage. 

### How does Terraform manage infrastructure on so many different kinds of platforms? 

This is achieved through **`providers`**. A provider helps Terraform manage third-party platforms through their API. Providers enable Terraform manage cloud platforms like AWS, GCP or Azure, as we have just seen, as well as network infrastructure like BigIP CloudFlare, DNS, Palo Alto Networks, and Infoblox. As well as monitoring and data management tools like DataDog, Grafana, Auth0, Wavefront, and Sumo Logic. Databases like InfluxDB, MongoDB, MySQL, PostgreSQL, and version control systems like GitHub, Bitbucket, or GitLab. Terraform supports hundreds of such providers, and as a result, can work with almost every infrastructure platform. 

Terraform uses **`HCL`**, which stands for **`HashiCorp Configuration Language`**, which is a simple, declarative language to define the infrastructure resources to be provisioned as blocks of code. All infrastructure resources can be defined within configuration files that has a `.tf` file extension. The configuration syntax is easy to read and write and pick up for a beginner. The sample code is used to provision a new EC2 instance on the AWS cloud. This code is declarative and can be maintained in a version control system, allowing it to be distributed to other teams. 

### We said that the code is declarative, but what does declarative mean? 

The code we defined is the state that we want our infrastructure to be in. That's the desired state. This on the right is the current state where there's nothing. Terraform will take care of what is required to go from the current state to the desired state without us having to worry about how to get there. 

#### How does Terraform do that? 

Terraform works in three phases; Init, Plan, and Apply. During the init phase, Terraform initializes the project and identifies the providers to be used for the target environment. During the plan phase, Terraform drafts a plan to get to the target state. Then in the apply phase, Terraform makes the necessary changes required on the target environment to bring it to the desired state. If for some reason the environment was to shift from the desired state, then a subsequent Terraform apply will bring it back to the desired state, by only fixing the missing component. 

Every object that Terraform manages is called a resource. A resource can be a compute instance, a database server in the cloud, or in a physical server on-premise that Terraform manages. Terraform manages the lifecycle of the resources from its provisioning to configuration to decommissioning. 

Terraform records the state of the infrastructure as it is seen in the real world and based on this, it can determine what actions to take when updating resources for a particular platform. Terraform can ensure that the entire infrastructure is always in the defined state at all times. The **`state`** is a blueprint of the infrastructure deployed by Terraform. Terraform can read attributes of existing infrastructure components by configuring data sources. This can later be used for configuring other resources within Terraform. 

Terraform can also import other resources outside of Terraform that were either created manually or by the means of other IAC tools, and bring it under its control so that it can manage those resources going forward. 

Terraform Cloud and Terraform Enterprise provide additional features that allow simplified collaboration between teams managing infrastructure, improved security, and a centralized UI to manage Terraform deployments. All these features make Terraform an excellent, enterprise-grade infrastructure provisioning tool. That was a quick introduction to Terraform at a high level. Let's dive in and explore all of these in much more detail in the upcoming lectures.