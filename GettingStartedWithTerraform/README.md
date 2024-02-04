# Getting Started With Terraform

## Installing Terraform

In this section, we will learn how to install Terraform. Terraform can be downloaded as a single binary or an executable file from the Terraform download section in www.terraform.io. Installing Terraform is as simple as downloading this file and copying it to the system path. Once installed, we can check the version by running the command Terraform version. The latest version of Terraform as of this recording is 0.13 and we'll be making use of this version throughout the course. Terraform is supported on Windows, Mac and several other Linux-based distributions. Please note that all the examples and labs used in this course will make use of Terraform running on a Linux machine and specifically the version 0.13. That's it. We can now start deploying resources using Terraform. 

As stated earlier, Terraform uses configuration files which are written in HCL to deploy infrastructure resources. These files have a **`.tf`** extension and can be created using any text editor such as `Notepad` or `Notepad++` for Windows or command line text editors such as `Vim` or `eMac` in Linux or it could be any ID of your choice. 

### What is a resource? 

A resource is an object data for managers. It could be a file on a local host or it could be a virtual machine on the cloud such as an EC2 instance. It could be services like S3 buckets, ECS, DynamoDB tables, IAM users or IAM groups, IAM Roles, policies, etcetera. Or it could be resources on major cloud providers such as the Compute an App Engine from GCP, databases on Azure, Azure Active Directory, etcetera. There are literally hundreds of resources that can be provisioned across most of the cloud and on-premise infrastructure using Terraform. 

We will look into some of these examples later in this course, but for the first few sections, we will stick to two very easy to understand resources, the local file type of resource and a special kind of resource called a random pit. It is important to use a simple resource type to really understand the basics of Terraform such as the lifecycle of resources, the HCL format, etcetera. Once we gain a good understanding of the basics, we can easily apply that knowledge to other real-life use cases and we will see that in later sections of this course.

## HashiCorp Configuration Language (HCL) Basics

In this lecture, we will understand the basics of HCL, which is HashiCorp Configuration Language, and then create a resource using Terraform. 

Let us first understand the HCL syntax. The HCL file consists of **`Blocks and Arguments.`** A block is defined within curly braces, and it contains a set of arguments in key value pair format representing the configuration data. 

### But what is a block and what arguments does it contain? 

In its simplest form, a block in Terraform contains information about the infrastructure platform and a set of resources within that platform that we want to create. For example, let us consider a simple task. We want to create a file in the local system where Terraform is installed. To do this, first, let us create a directory called `terraform-local-file` under `/root` directory. This is the directory under which we will create the HCL configuration file. Once we change into this new directory, we can create a configuration file called `local.tf`. And within this file, we can define a resource block, like this. 
And inside the Resource block, we specify the file name to be created as well as its contents using the block arguments. 

```bash
mkdir /root/terraform-local-file
cd /root/terraform-local-file
```

```bash
resource "local_file" "pet" {   # BlockName=Resource; ResourceType=local_file; Provider=local; TypeOfResource=file; ResourceName=pet
    filename = "/root/pets.txt"
    content = "We love pets!"
}
```

Let us break down the local.tf file to understand what each line means. The first element in this file is a block. Now this can be identified by the curly braces inside. The type of block we see here is called the `Resource` block, and this can be identified by the keyword called "Resource" in the beginning of the block. Following the keyword called resource, we have the declaration of the resource type that we want to create. This is a fixed value and depends on the provider where we want to create the resource. In this case, we have the resource type called `local_file`. 

A resource type provides two bits of information. First is the Provider, which is represented by the word before the underscore in the resource type. Here we are making use of the **"Local"** provider. The word following the underscore, which is **"File"** in this case, represents the type of resource. The next and final declaration in this resource block is the resource name. This is the logical name used to identify the resource, and it can be named anything. But in this case, we have called it pet, as the file we are creating contains information about pets. 

And within this block and inside the curly braces, we define the arguments for resource which are written in key value pair format. **`These arguments are specific to the type of resource we are creating`**, which in this case is the local_file. The first argument is the filename. To this, we assign the absolute path to the file we want to create. In this example, it is set to /root/pets.txt. Now we can also add some content to this file by making use of the content argument. To this, let us add the value "We love pets!". The words filename and content are specific to the local_file resource we want to create, and they cannot be changed. In other words, the resource type of `local_file` expects that we provide the argument of `filename` and `content`. **`Each resource type has specific arguments that they expect.`** We will see more of that as we progress through the course. And that's it, we now have a complete HCL configuration file that we can use to create a file by the name of "pets.txt". This file will be created in the /root directory, and it'll contain a single line of data. The resource block that we see here is just one example of the configuration blocks used in HCL, but it is also a mandatory block needed to deploy a resource using Terraform. 

Here is an example of a resource file created for provisioning an AWS ec2 instance. The resource type is aws_instance. We name the resource webserver and the arguments that we have used here is the `ami` and the `instance_type`. 

```bash
resource "aws_instance" "webserver" {
    ami = ""
    instance_type = ""
}
```

Here is another example of a resource file used to create an AWS S3 bucket. The resource type in this case is aws_s3_bucket. The resource name that we have chosen is data and the arguments that we have provided is the **`bucket_name`** and the **`acl`**. 

```bash
resource "aws_s3_bucket" "data" {
    bucket_name = ""
    acl = ""
}
```

A simple Terraform workflow consists of four steps. First, write the configuration file. Next, run the Terraform Init command. And after that, review the execution plan using the terraform plan command. Finally, once we are ready, apply the changes using the Terraform Apply command. 

```bash
terraform init
terraform plan
terraform apply
```

With the configuration file ready, we can now create the file resource using the terraform commands as follows, first, run the **`terraform init`** command. This command will check the configuration file and initialize the working directory containing the .TF file. One of the first things that this command does is to understand that we are making use of the Local provider based on the resource type declared in the resource block. It will then download the **`plugin`** to be able to work on the resources declared in the .TF file. From the output of this command, we can see that terraform init has installed a plugin called local. 

Next, we are ready to create the resource, but before we do that, if we want to see the execution plan that will be carried out by Terraform, we can use the command **`terraform plan`**. This command will show the actions that will be carried out by Terraform to create the resource. Terraform knows that it has to create resources, and this is displayed in the output similar to a diff command in GIT. The output has a + symbol next to the `local_file` type resource called `pet`. This includes all the arguments that we specified in the .TF file for creating the resource. But you'll also notice that some default or optional arguments which we did not specifically declare in the configuration file is also displayed on the screen. The + symbol implies that the resource will be created. Now remember, this step will not create the infrastructure resource yet. This information is provided for the user to review and ensure that all the actions to be performed in this execution plan is desired. 

After the review, we can create the resource. And to do this, we will make use of the **`terraform apply`** command. This command will display the execution plan once again, and it will then ask the user to confirm by typing `Yes` to proceed. Once we confirm, it will proceed with the creation of the resource, which in this case is a file. We can validate that the file was indeed created by running the cat command to view the file. 

We can also run the **`terraform show command`** within the configuration directory to see the details of the resource that we just created. This command inspects the state file and displays the resource details. We will learn more about this command and the state in a later lecture. 

So, we have now created our first resource using Terraform. Before we end this section, let us go back and look at the configuration blocks in local.tf file. In this example, we used the resource type of local_file and learnt that the keyword before the underscore here is the provider name called "local". 

### But how do we know that? How do we know what resource types other than local_file are available under the provider called local? And finally, how do we know what arguments are expected by the local_file resource? 

Earlier, we mentioned that Terraform supports over 100 providers, including the local provider we have used in this example. Other common examples are AWS to deploy resources in Amazon AWS cloud, Azure, GCP, Ali Cloud, etcetera. Each of these providers have a unique list of resources that can be created on that specific platform. And each resource can have a number of required or optional arguments that are needed to create that resource And we can create as many resources of each type as needed. It is impossible to remember all of these options, and of course, we don't have to do that. **`Terraform documentation`** is extremely comprehensive, and it is the single source of truth that we need to follow. If we look up the `local` provider within the documentation, we can see that it only has one type of resource called the `local_file`. Under the `arguments` section, we can see that there are several arguments that the resource block accepts, out of which only one is mandatory, the `filename`. The rest of the arguments are optional. That's it for this lecture. Now let's head over to the hands on labs and practice working with HCL and create our first resource using Terraform.