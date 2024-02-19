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