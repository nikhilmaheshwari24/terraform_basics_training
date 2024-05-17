# Working With Terraform

## Terraform Commands

Until now, we have seen quite a few Terraform commands in action, such as the terraform init, plan and apply. Let us now take a look at some more commands available in Terraform. 

The first command we will take a look at is the terraform `validate` command. Once we write our configuration file, it's not necessary to run terraform plan or apply to check if the syntax used is correct. Instead, we can make use of the terraform validate command like this. If everything is correct with the file, we should see a successful validation message like this. If there's an error in the configuration file, the validate command will show you the line in the file that is causing the error with hints to fix it. In this example, we have used an incorrect argument for the local file resource. It should be file permission and not file permissions. 

```bash
terraform validate
```

The next command that we are going to see is the terraform `fmt` or the terraform format command. This command scans the configuration files in the current working directory and formats the code into a canonical format. This is a useful command to improve the readability of the Terraform configuration file. When we run this command, the files that are changed and the configuration directory is displayed on the screen. 

```bash
terraform fmt
```

The terraform `show` command prints out the current state of the infrastructure as seen by Terraform. In this example, we have already created the local file resource. When we run the show command, it displays the current state of the resource including all the attributes created by Terraform for that resource, such as the file name, file and directory permissions, content, and ID of their source. 

```bash
terraform show
```

Additionally, we can make use of the `-JSON` flag to print the contents in a JSON format.

```bash
terraform show -json
```

To see a list of all providers used in the configuration directory, use the terraform `providers` command. 

```bash
terraform providers
```

You can also make use of the mirror subcommand to copy provider plugins needed for the current configuration to another directory like this. This command will mirror the provider configuration in a new path, /root/terraform/new_local_file. 

```bash
terraform providers
```

We saw how to use Terraform output variables in one of the previous lectures. If you want to print all output variables in the configuration directory, use the command terraform output. 

```bash
terraform output
```

You can also print the value of a specific variable by appending the name of the variable to the end of the output command like this. 

```bash
terraform output <variable-name>
```

The terraform refresh command is used to sync Terraform with the real-world infrastructure. For example, if there are any changes made to a resource created by Terraform outside its control, such as a manual update, the terraform refresh command will pick it up and update the state file. This reconciliation is useful to determine what action to take during the next apply. This command will not modify any infrastructure resource but it will modify the state file. 

```bash
terraform refresh
```

As we saw earlier, terraform refresh is also run automatically by commands such as terraform plan and terraform apply. This is done prior to Terraform generating an execution plan. This can however be bypassed by using the -refresh=false option with the commands. 

```bash
terraform plan/apply -refresh=false
```

The terraform `graph` command is used to create a visual representation of the dependencies and a Terraform configuration or an execution plan. In this example, the local file in our main.tf file has a dependency on the random pet resource. This command can be run as soon as you have the configuration file ready even before you have initialized the configuration directory with Terraform in it. 
Upon running the terraform graph command, you should see an output like this. This text generated is hard to comprehend as it is, but it is a graph generated in a format called `DOT`. 

```bash
terraform graph
```

To make more sense of this graph we can pass it through a graph visualization software such as `Graphviz`, and we can install it in Ubuntu using apt like this. Once installed, we can pass the output of the terraform graph to the DOT command which we installed using the Graphviz package, and generate a graphic like this. We can now open this file via a browser and it should show a dependency graph like this. 

```bash
apt udpate
apt install graphviz -y
terraform graph | dot -Tsvg > graph.svg
```

The root is the configuration directory where the configuration for this graph is located. We can see that there are two resources, the local file called pet and the random pet resource called my-pet that makes use of the local and the random provider respectively. Finally, we can see that the local file called pet depends on the random pet resource called my-pet, as we have used a reference expression and the local file resource that points to the ID of the random pet. 

# Mutable vs Immutable Infrastructure

In this section, we will learn about the difference between mutable and immutable infrastructure. In one of the previous lectures, we saw that when Terraform updates a resource, such as updating the permissions of a local file, it first destroys it and then recreates it with the new permission. 

### Why does Terraform do that? 

To understand this, let us make use of a simple example. Consider an application server running NGINX with the version of 1.17. When a new version of NGINX is released, we upgrade the software running on this web server. First from 1.17 to 1.18. Eventually, when a new version 1.19 is released, we upgraded the same way from 1.18 to 1.19. This can be done using a number of different ways. One simple approach is to download the desired version of NGINX and then use it to manually upgrade the software on the web server during a maintenance window. Of course, we can also make use of tools such as ad-hoc scripts, or configuration management tools such as Ansible to achieve this. For high availability, instead of relying on one web server, we can have a pool of these web servers all running the same software and code. We would have to use the same software upgrade lifecycle for each of the servers using the same approach that we use for the first web server. This type of update is known as **`in-place`** update. This is because the underlying infrastructure remains the same, but the software and the configuration on these servers are changed as part of the update. This here is an example of a `mutable` infrastructure. Updating software on a system can be a complex task, and in almost all cases, there are bound to be a set of dependencies that have to be met before an upgrade can be carried out successfully. 

Let us assume that web server 1 and 2 have every dependency met while we try to upgrade the version from 1.18 to 1.19. As a result, these two servers are upgraded without any issues. Web server 3, on the other hand, does not. The upgrade fails on web server 3 because it has a few dependencies that are not met, and as a result, it remains at version 1.18. A failure in upgrade could be because of a number of reasons such as network issues impacting the connectivity to the software repository, file system full, or different version of operating system running on web server 3 as compared to the other two. However, the important thing here to note is that we now have a pool of three web servers in which one of the servers is running a different version of software as compared to the rest. Over time, with multiple updates and changes to this pool of servers, there is a possibility that each of the servers vary from one another maybe in software, or configuration, or operating system, etcetera. This is known as a `configuration drift`. For example, after a few update windows, our three web servers could look like this. Web server 1 and 2 have NGINX version of 1.19, and web server 3 has a version of 1.18. All three web servers may also be running slightly different versions of operating system on them. This configuration drift can leave the infrastructure in a complex state, making it difficult to plan and carry out subsequent updates. Troubleshooting issues would also be a difficult task as each server would behave slightly differently from the other because of this configuration drift. 

Instead of updating the software versions on the web servers, we can spin up new web servers with the updated software version, and then delete the old web server. So when we want to update NGINX 1.17 to 1.18, a new server is provisioned with 1.18 version of NGINX. If the update goes through, then the old web server is deleted. This is known as `immutable` infrastructure. Immutable means unchanged or something that you cannot change. As a consequence, with immutable infrastructure, we cannot carry out in-place updates of the resources anymore. This doesn't mean that updating web servers this way will not lead to failures. If the upgrade fails for any reason, the old web server will be left intact and the failed server will be removed. As a result, we do not leave much room for configuration drift to occur between our servers, ensuring that it is left in a simple easy-to-understand state. 

Since we are working with Infrastructure as a Code, immutability makes it easier to version the infrastructure and to roll back and roll forward between versions. Terraform as an infrastructure provisioning tool uses this approach. Going back to our example, updating the resource block for our local file resource and changing the permission from 777 to 700 will result in the original file to be deleted and a new file to be created with the updated permission. By default, Terraform destroys the resource first before creating a new one in its place. 

### What if we want the resource to be created first before the old one is deleted, or to ignore deletion completely? How do we do that? 

These can be done by making use of lifecycle rules in our resource block.

## Lifecycle Rules

In this section, we will learn how to set up lifecycle rules in Terraform. Previously, we saw that when Terraform updates a resource it frees the infrastructure to be immutable, and first deletes a resource before creating a new one with the updated configuration. For example, if we update the file permissions on our local file resource from 777 to 700, and then run terraform apply, you would see that the older file is deleted first and then the new file is created. Now, this may not be a desirable approach in all cases. Sometimes you may want the updated version of the resource to be created first before the older one is deleted, or you may not want the resource to be deleted at all, even if there was a change made in its local configuration. This can be achieved in Terraform by making use of lifecycle rules. 

These rules make use of the same block syntax that we have seen many times so far, and they go directly inside the resource block whose behavior we want to change. The syntax of a resource block with a lifecycle rule looks like this. 

Inside the lifecycle block, we add the rule which we want Terraform to adhere to while updating resources. One such argument, or a rule, is the `create_before_destroy` rule. Here, we have the same resource block that has been updated with the lifecycle rule of create_before_destroy set to true. This rule ensures that when a change in configuration forces the resource to be recreated, a new resource is created first before deleting the old one. 

```bash
lifecycle {
    create_before_destroy = true
}
```

Now, there would be cases where we do not want a resource to be destroyed for any reason. For this, we can make use of the `prevent_destroy` option. When it is set to true, Terraform will reject any changes that will result in the resource getting destroyed and display an error message like this. This is especially useful to prevent your resources from getting accidentally deleted. 

```bash
lifecycle {
    prevent_destroy = true
}
```

For example, a database resource such as MySQL or PostgreSQL may not be something that we want to delete once it's provisioned. One important thing to note here is that the resource can still be destroyed if we make use of the terraform destroy command. This rule will only prevent resource deletion from changes that are made to the configuration and a subsequent apply. 

The last argument type that we're going to see here is the `ignore_changes` rule. This lifecycle rule when applied will prevent a resource from being updated based on a list of attributes that we define within the lifecycle block. To understand this better, let's make use of a sample EC2 instance, which is a virtual machine on the AWS cloud. This EC2 instance is to be used as a web server and can be created with a simple resource block like this. Now, don't worry if this resource block and the arguments look unfamiliar, we will cover it in much more detail in the EC2 section of the course. For now, please note that the resource called web server makes use of three arguments. The AMI and the instance type are used to deploy a specific type of VM with a predefined specification. In this case, the values that we've chosen deploys a Ubuntu server with one CPU and 1GB of RAM. We're also making use of a tag called Name, which has a value of ProjectA-Webserver using the tags argument which is of type map. When we are on terraform apply, the EC2 instance is created as expected with a tag called Name and value of ProjectA-Webserver. Now, if changes are made to any of these arguments, Terraform will attempt to fix it during the next apply as expected. For example, if we modify the tag called Name and change its value from ProjectA-Webserver to, say, ProjectB-Webserver either manually or using any other tool, Terraform will detect this change and it will attempt to change it back to what it was originally, which is ProjectA-Webserver. In some rare cases, we may actually want the change in the name via any other method to be acceptable, and we want to prevent Terraform from reverting back to the old tag. To do this, we can make use of the lifecycle block with ignore changes argument like this. The ignore changes argument accepts a **list** as indicated by the square brackets, and it will accept any valid resource attribute. In this particular case, we have asked Terraform to ignore changes which are made to the tags attribute of the specific EC2 instance. If a change is made to the tags, a subsequent terraform apply should now show that there are no changes to apply. The change made to the tags of a server outside of Terraform is now completely ignored. Since it's a list, we can update more elements like this. You can also replace the list with the **`All`** keyword. This is especially useful if you do not want the resource to be modified for changes in any resource attributes. We will learn more about lifecycle rules later in the course when we work with AWS resources. For now, here's a quick summary of the three-argument types that we have seen in this lecture. Now, let's head over to the hands-on labs and practice working with lifecycle rules in Terraform.

## Datasources

In this section, we will take a look at data sources in Terraform. We know by now that Terraform makes use of configuration files along with the state file to provision infrastructure resources, but as we saw earlier, Terraform is just one of the infrastructure as a code tool that can be used for provisioning. Infrastructure can be provisioned using other tools, such as `Puppet` `CloudFormation`, `SaltStack`, `Ansible`, etcetera. Not to mention ad-hoc scripts and manually provisioned infrastructure or even resources that are created by Terraform from another configuration directory. For instance, let us assume that a database instance was provisioned manually in the AWS cloud. Although Terraform does not manage this resource, it can read attributes such as the database name, host address, or the DB user, and use it to provision an application resource that is managed by Terraform. 

Let's take a simpler example. We have a local file resource called "Pet" created with the contents "We love pets." Once this resource is provisioned, the file is created in the /root directory, and the information about this file is also stored in the Terraform state file. Now, let's create a new file using a simple shell script like this. Quite evidently, this file is outside the control and management or Terraform at this point in time. The local file resource that Terraform is in charge of is Petstore.txt under the /root directory and has no relationship with the local file called dogs.txt, which is also created under the /root directory. The dogs.txt has a single line that says, "Dogs are awesome." We would like Terraform to use this file as a data source and use its data as contents of our existing file called Petstore.txt. If you want to make use of the attributes of this new file that is created by the bar script, we can make use of data sources. Data sources allow Terraform to read attributes from resources which are provisioned outside its control. For example, to read the attributes from the local file called dogs.txt, we can define a `data` block within the configuration file, like this. As you may have noticed, the data block is quite similar to the `resource` block. Instead of the keyword called resource, we define a data source block with the keyword called "data." This is followed by the type of resource which we are trying to read. In this example, it's a local file. This can be any valid resource type for any provider supported by Terraform. 

```bash
# main.tf
data local_file "dog" {
    filename = "/root/dog.txt"
}
```

Next comes a logical resource name into which the attributes for a resource will be read. Within the block, we have arguments just like we have in a normal resource block. The data source block consists of specific arguments for a data source, and to know which argument is expected, we can look up the provider documentation in Terraform registry. For the local file data source, we just have one argument that should be used, which is the `filename` to be read. The data read from a data source is then available under the `data object` in Terraform. So, To use this data in the resource called "Pet", we could simply use `data.local_file.dog.content`. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = data.local_file.dog.content
}
data local_file "dog" {
    filename = "/root/dog.txt"
}
```

These details are of course, available in the Terraform documentation, under "Data sources." Within the documentation and under the attributes exported, we can see that the data source for a local file expose two attributes, the content and the Base64 encoded version of the content. 

To distinguish between a resource and data sources, let's do a quick comparison. The resources are created with the resource block, and data sources are created with the data block. Resources in Terraform are used to create, update and destroy infrastructure, whereas a data source is used to read information from a specific resource. Regular resources are also called "managed resources" as it's an extension which is managed by Terraform. Data sources are also called data resources. That's it for this lecture. Now let's head over to the hands-on labs and practice working with data sources in Terraform.

## Meta-Arguments

In this lecture, we will take a look at meta arguments in Terraform. Until now, we have been able to create single resources such as a local file and a random pet resource using Terraform, but what if you want to create multiple instances of the same resource, say, three local files for example? If we were using a shell script or some other programming language, we could create multiple files like this. In this example, we have created a bash script called create files.sh, which uses a four-loop to create empty files inside the root directory. The files will be called pet, followed by the range from one to three. While we cannot use the same script as it is within the resource block, Terraform offers several alternatives to achieve the same goal, and these can be done by making use of specific meta arguments in Terraform. Meta arguments can be used within any resource block to change the behavior of resources. We've already seen two types of meta arguments already in this course. That **`depends_on`** for defining explicit dependency between resources and the **`lifecycle`** rules which define how the resources should be created, updated, and destroyed within Terraform. Now, let us look at some more meta arguments specifically related to loops in Terraform.

## Count

In this lecture, we will take a look at the Count Meta-Argument and its uses in Terraform. One of the easiest ways to create multiple instances of the local file is to make use of the **`count`** meta-argument. To do this, simply add an argument called count with a value greater than one. Here, we have used count is equal to three. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename
    count = 3
}

# variables.tf
variable "filename" {
    default = "/root/pets.txt"
}
```

When we try to run Terraform plan now, we can see that Terraform tries to create three resources instead of one. In the output of the Terraform apply command, we can see that three resources are created. The resources are identified by pet[0], where zero is within square brackets, pet[1] and pet[2]. The resource is now considered to be a list of resources with elements at index 0, 1 and 2. However, there is one problem with this approach. Since we have only specified the count, Terraform will try to create the same resource three times. Since the filename is not unique, Terraform will recreate the same file three times rather than creating three separate files which defeats the purpose of this task. A better way to do this and make sure that all the three resources have unique filenames is to make use of a list variable for filename. To do this, we have used default values with three elements, each corresponding to the name of the file that we want to create. Next, we want Terraform to make use of each element of this list as the value of the filename argument. In this example, Terraform should make three iterations as the count has a value of three. The first iteration should pick up the element at index 0 which is the file called pets.txt. This is followed by element at index 1 which is dogs.txt and finally cats.txt at index 2. To use this within the configuration file, we can make use of count.index in the expression for filename like this. Now when we are on Terraform apply, we can see that there are three files created inside this /root directory. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename[count.index]
    count = 3
}

# variables.tf
variable "filename" {
    default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt"]
}
```

#### What if we were to add a few more elements to the list in the future? 

Say we wanted to add a few more files by the name of /root/cows.txt and ducks.txt. If we were to apply this configuration now, we would see that it would still create only three files, because we have set the count to a static value of 3. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename[count.index]
    count = 3
}

# variables.tf
variable "filename" {
    default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt", "/root/cows.txt", "/root/ducks.txt"]
}
```

We want the count to automatically pick up the number of items that are defined within the filename variable. To do this, we can set set the value of count to use a built-in function that would return the length of the list. This built-in function called **`length`** will set the value of count to 5. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename[count.index]
    count = length(var.filename)
}

# variables.tf
variable "filename" {
    default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt", "/root/cows.txt", "/root/ducks.txt"]
}
```

Terraform offers several built-in functions that allow us to manipulate values within expressions. One simple function that we can use here is the length function. The Length function is used to calculate the size of a list and we can use this function in the count meta-argument to dynamically determine the size of the filename variable. We are now ready to run Terraform apply. Before we do that, let’s change the default value for the filename variable back to three elements. That is it. If we run Terraform plan and apply now, we should see that three resources with distinct files names have been created. There is however a significant drawback when using the count meta-argument to loop through variables this way. To illustrate this, lets see the same example but this time, let us remove the element /root/pets.txt from the list. If we run Terraform plan now, we see that instead of deleting just one resource with the filename /root/pets.txt, Terraform is replacing two resources and deleting one resource. Why does it do that? We only want to remove the first element in this list and it looks like all elements are going to be replaced by this operation. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename[count.index]
    count = 3
}

# variables.tf
variable "filename" {
    default = ["/root/dogs.txt", "/root/cats.txt"]
}
```

To see why this is happening, let us first understand how count works. As we saw before, when we use count, the resources become a list of resources. To see this using Terraform, let us add an output variable to the main.tf file to print all details of this resource. Using the Terraform output command, we can see that the resource is now in the format of a list. Originally, the resource called pet is a list with three resources, each identified by its index. The first resource in the list creates the file by the name pets.txt and it is identified by pet with the index [0], zero enclosed in square brackets since it's a list. The second resource element is pet[1] which creates a file called dogs.txt and the third resource is pet[2], which creates a file called cats.txt. The first element in a list is always zero. As a result, when we deleted the element called /root/pets.txt, which was at index 0 to begin with, the element with value /root/dogs.txt shifts up and takes its place at index 0. Likewise, /root/cats.txt becomes the element at index 1 and the list now has only two elements in it. When we run Terraform plan now, Terraform can see that that the resources at index pet[0] and pet[1] have to be destroyed and replaced. This is owing to the change in their filenames. There is no resource pet[2], so it will delete this resource entirely. Although after the apply operation we will have the resources created as per our intended end state, this is not an ideal approach. We may not want the resources to be destroyed and recreated just because we removed an unrelated element from the list. We will see how to do fix this in the next lecture. Now, let’s head over to the hands-on labs and practice working with the count meta-arguments in Terraform.

## For Each

In this lecture, we will take a look at the for_each meta argument and its uses in Terraform. In the previous lecture, we saw that when we use count, the resources are created as a list, and this can have undesirable results when updating them. One way to overcome this is to make use of the **`for_each`** argument insert of count, like this. Next, to set the value of file name to each element in the list, we can make use of the expression **`each.value`** like this. However, there's a catch. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = each.value
    for_each = var.filename
}

# variables.tf
variable "filename" {
    default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt"]
}
```

If we run Terraform plan now, we will see an error. The for_each argument only works with a **`map`** or a **`set`**. In the variables.tf file, we are currently making use of a list containing string elements. There are a couple of ways to fix this. Either change the variable called filename to the type set. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = each.value
    for_each = var.filename
}

# variables.tf
variable "filename" {
    type = set(string)
    default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt"]
}
```

In the variables lecture, we learned that a set is similar to a list, but it cannot contain duplicate elements. Once we change the variable type and then run Terraform plan, we should see that there are three files to be created. Another way to fix this error while retaining the variable type as a list is to make use of another built-in function. This time, we'll make use of the **`toset`** function, which will convert the variables from a list to a set. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = each.value
    for_each = toset(var.filename)
}

# variables.tf
variable "filename" {
    type = list(string)
    default = ["/root/pets.txt", "/root/dogs.txt", "/root/cats.txt"]
}
```

Once this is done, Terraform plan command should now work as expected. Now let us replicate the same scenario as the one we did earlier with the count meta argument and delete the first element with the value/slash/root pets.txt from the list. When we run Terraform plan now, we can see that only one resource is said to be destroyed, the file with the name pets.txt. The other resources will be untouched. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = each.value
    for_each = toset(var.filename)
}

# variables.tf
variable "filename" {
    type = list(string)
    default = ["/root/dogs.txt", "/root/cats.txt"]
}
```

To see how this is working, let us create an output variable called pets to print the resource details like we did with the example using count. From the Terraform output command, we can now see that the resources are stored as a map and not a list. When we use for_each instead of count, the resources are created as a map and not a list. This means that the resources are no longer identified by the index, thereby bypassing the issues that we saw when we use count. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = each.value
    for_each = toset(var.filename)
}

output "pets" {
    value = local_file.pet
}
# variables.tf
variable "filename" {
    type = list(string)
    default = ["/root/dogs.txt", "/root/cats.txt"]
}
```

They are now identified by the keys which are file names/root/..txt and /root/cat.txt, which are used by the for_each argument in the configuration file. If we compare this to the earlier output that we got when we use count, we can see the difference in how the resources are created. The count option created it as a list and the for_each created it as a map. There are a few other meta arguments in Terraform such as provisioners, providers, backends, et cetera. We will see that later in this course. Now let's head over to the hands-on labs and practice working with the for_each meta argument in Terraform.

## Version Constraint

In this lecture, we will see how to make use of specific provider versions in Terraform. We saw earlier that providers use a plugin-based architecture and that most of the popular ones are available in the public Terraform registry. Without additional configuration, the Terraform init command downloads the latest version of the provider plugins that are needed by the configuration files. However, this is not something that we may desire every day. The functionality of a provider plugin may vary drastically from one version to another. Our Terraform configuration may not work as expected when using a version different than the one it was written in. Fortunately, we can make sure that a specific version of the provider is used by Terraform when we are on the Terraform init command. The instructions to you use a specific version of a provider is available in the provider documentation in the registry. For example, if we look up the local provider within the registry, the default version is 2.0.0, which is also the latest version as of this recording. To you use a different version, click on the version tab and it should open a drop down with all the older versions of the provider. Let us select version 1.4.0. To use this specific version of the local provider, click on the use provider tab on the right. This should open up the code block that we can copy and paste within our configuration. 

```bash
terraform {
    required_providers {
        local = {
            source = "hashicorp/local"
            version = "1.4.0"
        }
    }
}
```

Here, we are making use of a new block called **`terraform`**, which is used to configure settings related to Terraform itself. To make use of a specific version of the provider, we need to make use of another block called required providers inside the Terraform block. Inside the required providers block, we can have multiple arguments for every provider that we want to use. In this example, we have one argument with the key called local for the local provider. The value for this argument is an object with the source address of the provider and the exact version that we want to install, which in this case is 1.4.0. With the Terraform block configured to use the version 1.4.0 of the local provider, when we run a Terraform init, we should see a message like this. Before we wrap up this lecture, let us look at the syntax used to define version constraints in Terraform. In the configuration for the local provider, we have specified version equal to 1.4.0. This allows Terraform to find and download this exact version of the local provider. However, there are other ways to use the version constraint. If we use the not equal to symbol instead, Terraform form will ensure that this specific version is not downloaded. In this case, we have specifically asked Terraform to not use the version 2.0.0. 

```bash
terraform {
    required_providers {
        local = {
            source = "hashicorp/local"
            version = "!=2.0.0"
        }
    }
}
```

It downloads the previous version available, which is 1.4.0. If we want Terraform to make use of a version lesser than a given version, we can do that by making use of comparison operators like this, and to make use of a version greater than a specific version, we can make use of the greater than operator like this. 

```bash
terraform {
    required_providers {
        local = {
            source = "hashicorp/local"
            version = "< 2.0.0"
        }
    }
}
```

We can also combine the comparison operators like this to make use of a specific version within a range. In this example, we want to make use of any version greater than 1.2.0, but lesser than 2.0.0, but also not the version 1.4.0 specifically. As a result, Terraform downloads the version 1.3.0, which is acceptable in this case. 

```bash
terraform {
    required_providers {
        local = {
            source = "hashicorp/local"
            version = "> 1.2.0, < 2.0.0, !=1.4.0"
        }
    }
}
```

Finally, we can also make use of pessimistic constraint operators. This is defined by making use of the tilde greater than symbol like this. This operator allows Terraform to download the specific version or any available incremental version based on the value we provided. For example, here, we have given the value of 1.2, following the tilde greater than symbol. This means that Terraform can either download the version 1.2 or incremental version such as 1.3, 1.4, 1.5, all the way up until 1.9. 

```bash
terraform {
    required_providers {
        local = {
            source = "hashicorp/local"
            version = "~> 1.2"
        }
    }
}
```

However, if we look at the provider documentation, we do not have a version 1.5 or anything above. The maximum version that we can make use of in this case is 1.4.0 and this is the version that is downloaded when we run Terraform init. Let us make use of another value. This time, 1.2.0, with the same pessimistic constraint operator. This time, Terraform can download the version 1.2.0 or the version 1.2.1 or the version 1.2.2 all the way up until 1.2.9. 

```bash
terraform {
    required_providers {
        local = {
            source = "hashicorp/local"
            version = "~> 1.2.0"
        }
    }
}
```

Again, we only have a maximum version of 1.2.2 in the registry, and that's the version that will be downloaded when we run Terraform init. That's it for this lecture, head over to the hands-on labs and practice working with version constraints in Terraform.