# Remote State

## What is Remote State and State Locking?

In this lecture, we will get introduced to a remote state in Terraform. Previously, we saw how Terraform uses the state file to map resources in the configuration file to real world infrastructure. So far, we have been working with the Terraform state file that is stored locally. When we run Terraform apply for the first time, a file called **`terraform.tfstate`** is created inside the configuration directory by default. Some of the benefits of the Terraform state are, mapping Terraform configuration to the real world infrastructure, tracking metadata such as dependencies, which allow Terraform to create and delete resources in the correct order, improving the performance of Terraform operations while working with large configuration files and especially those that make use of a number of different cloud providers. Finally, the Terraform state allows members of a team to collaborate and provision resources as a team. However, in the examples that we have seen so far, Terraform state file is created locally on the client machine and more specifically within the configuration directory that is being used as a default. This is an example of a local state file, and it does not provide a good opportunity to collaborate. This is because the file is only available on the client machine, such as a developer's laptop. 

We have also learned that unlike the configuration files, it's not a good idea to store the state file in a version control system. This is because it would mostly contain sensitive information pertaining to our infrastructure. Let us see why it's a bad idea to store the state file in a version control system. For this, let's see the Terraform workflow when used individually and as a team with the state and configuration saved within a Git Repo. Let us consider that Abdul, who is a developer, writes the Terraform configuration for creating an S3 bucket. He then proceeds to run the Terraform plan. After reviewing the execution plan, proceeds to apply the configuration. At this stage, Terraform creates a local state file called terraform.tfstate. Once the bucket has been created, he checks in all the files in the configuration directory into a Git Repo, including the Terraform state file and if any changes are to be made, the same process is repeated. The files are fetched from the Git Repo, changes are then made to the configuration file. This is followed by a Terraform plan and then apply. Finally, the updated configuration file and the state are pushed to the Git Repo. Now, let us assume that the team is growing and a new developer called Lee also wants to manage the infrastructure using Terraform. The logical way would be for Lee to pull down the Git Repo into his own client machine. He would then follow the same process as Abdul to make changes to the infrastructure, make changes to the configuration as needed, review the execution plan and then apply the configuration. Once applied, he would make sure to push the updated configuration and state file into the remote Git Repo. 

While this can technically work, there are some major flaws when using this approach. As we saw earlier, state file stores all the information about the infrastructure, including IP addresses, initial passwords for some databases, key names, et cetera. Keeping all this information in a Git repository is not recommended. Secondly, when we work as a team, it is important that only one person runs operation against the same configuration and state at any given time. If Abdul and Lee both try to update the same S3 bucket at the same time using the exact same state file but within their individual laptops, it can lead to unintended consequences such as the corruption of the state file. When using local state files as an individual, Terraform protects itself from getting into the situation where concurrent operations are run against the same configuration, and this can be tested by running Terraform operations one after the other, such as running data from apply command simultaneously from two terminals. While the first operation is in progress, Terraform locks the state file. As a consequence, we will not be able to run any operation from the second terminal until the first operation finishes. This is a very important feature of Terraform called **`state locking`**. It ensures that the state file does not get corrupted by multiple operations trying to update it at the same time. Version control systems such as GitHub do not support state locking. As a result, if it is used to store the state file, multiple users can use the same state file simultaneously, which can result in issues like conflicts and data loss within the state file. Finally, if a user forgets to pull the latest version of the state file from the version control system and work with an older obsolete state file, it can result in disastrous ethics such as rollback or even destroying of the resources. 

Because of all these reasons, it's a much better option to store Terraform state in a secure, shared storage by making use of **`remote backends`**. With this option, the state file no longer resides in the configuration directory or version control systems and are moved to a shared storage solution such as **`AWS S3`**, **`HashiCorp Consul`**, **`Google cloud storage`** and **`Terraform cloud`**. When a remote backend is configured, Terraform will automatically load the state file from the shared storage every time it is required by a Terraform operation. It will also upload the updated Terraform state file after every apply. Most importantly, many of these remote backend options such as the AWS S3 allows for state locking. This ensures that the integrity of the state file is maintained. Finally, the remote backends provide different ways to secure the storage such as encryption at rest and in transit to make sure that all the sensitive information stored inside is secured. In the next lecture, we will see how to configure remote backends using AWS S3.

## Remote Backends with S3

In this lecture, we will learn how to make use of an S3 bucket and a DynamoDB table to configure a remote backend for our terraform configuration. For this, we need two things, an S3 bucket which will be used to store the terraform state file and a DynamoDB table which will be used to implement state locking and consistency checks. First, make sure that these two prerequisites have been completed before beginning to configure a remote backend. After these two services have been created, keep a note of the bucket name, the key to be used, the region, and the name of the DynamoDB table. Now let's go over to our configuration directory. We have a main.tf file that creates a local file resource. When we run terraform apply, the resource is created and as expected, terraform creates a local state file called terraform.tfstate. To configure a remote backend in terraform, we must define additional settings by making use of the terraform block. As a recap, we have already used the terraform block once when we wanted a specific version of the provider plugin to be used within our configuration. In this case, we want to configure a remote backend for storing the state file and for that, the terraform block should now look like this. Within the terraform block, we specify another block called backend. Here, we provide the values we recorded as part of the prerequisite step. The backend name specifies the type of the backend that we want to use and for making use of an AWS S3 as the backend, we use the name S3. The backend block with S3 expects three arguments. First is the name of the existing S3 `bucket`. Next is the `key`, which is an S3 object part where the state file should be stored. In this case, we want to store the terraform state file within a folder called finance, and this folder should exist within a bucket called `kodekloud-terraform-state-bucket01`. The final argument that is required is the `region`. This is the region where the S3 bucket has been created and in this example, it is set to `us-west-1`. In order to achieve state locking, we can optionally provide a `dynamodb_table`. **This table should have a primary or a hash key with the name lockID**. We have already created a DynamoDB table called `state-locking` for this purpose. As a standard practice, let's not store the terraform block inside the main.tf file, instead, let's move it to a separate file called `terraform.tf`. 

```bash
# main.tf
ressource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
}

terraform {
    backend "s3" { 
        bucket = "kodekloud-terraform-state-bucket01"
        key = "finance/terraform.tfstate"
        region = "us-west-1"
        dynamodb_table = "state-locking"
    }
}
```

We now have the infrastructure configuration and the backend configuration segregated into main.tf and terraform.tf file respectively. The terraform state is now configured to be stored in a remote S3 bucket. However, if we run the terraform apply command now, we will see an error that says backend reinitialization required. Running the terraform init command will initialize the new backend to be used. 

```bash
terraform init
```

Since we already have a local state file in the configuration directory the init process gives us an option to copy the terraform state file into the remote S3 backend. Giving a value of yes will copy the state file to the S3 bucket. We can now delete the local state file from the configuration directory. Running a terraform plan or apply now will lock the state file and pull it down from the S3 bucket into the memory. Any subsequent changes to the state will be uploaded to the backend instantaneously and once the operation is complete, the lock will be released. The state file will not be stored in the local configuration directory anymore. That's it for this lecture. Let's head over to the hands-on labs and practice working with remote backends in terraform.

## Terraform State Commands

In this lecture, we will learn how to list and manipulate the Terraform state using the Terraform state command. In one of the previous lectures, we saw that the Terraform state file is stored in JSON format. It should not be edited manually by opening it with a text editor. Terraform provides a set of commands that allows us to list, pull and manipulate state. These should be used instead of manually editing these files. The Terraform state command uses a syntax like this, the command to be used is Terraform state. This is followed by a subcommand such as **`list`**, **`move`**, **`pull`**, **`remove`**, **`show`**, etcetera. We will take a look at each of these next. The first command that we are going to take a look at is the list command.This will list all the resources recorded within the Terraform state file. As you can see from the output, it will only print the resource address and no other details about the resource. We can also pass in an additional argument to this command for a matching resource address like this. If available, the list command will return the matching resource address from the state. 

```bash
terraform state list

terraform state list aws_s3_bucket.finance-2020922
```

To get detailed information about a resource from the state file, we can make use of the Terraform state show command. This command will show the attributes of a single resource in the state file that matches the given address. For example, to print all the attributes for an S3 bucket with the resource name finance-2020922, use the Terraform state show command like this. 

```bash
terraform state show 

terraform state show aws_s3_bucket.finance-2020922
```

Next, let's take a look at the Terraform state move command. This command is used to move items in a Terraform state file. The syntax of this command has a source and a destination following the move or MV sub command. The items can be moved within the same state file, which would mean moving a resource from its current resource address to another, which essentially means renaming a resource, or it can move items from one state file to another state file maintained by a different configuration directory completely. For example, let's consider that we have a configuration file that we have used to create a dynamo-db table called `state-locking`. Sure enough, the state file also has the managed source under the same name. If you want to rename the resource name from state-locking to, say, state-locking-db, one way to do it without having to recreate the table would be to use the Terraform state move command like this. Specify the source name as state-locking and the target name as state-locking-db. Once this is done, the name of the resource and the state file changes to the new name. Next, we must manually rename the resource name in the configuration file. 

```bash
terraform state mv aws_dynamodb_table.state-locking aws_dynamodb_table.state-locking-db
```

Now we run a Terraform apply, there should be no changes to be applied for this resource. We saw earlier that when using remote state, the state file is no longer stored in the configuration directory under the name terraform.tfstate. Then how do we view the contents of this remote state? For this, we can make use of the Terraform state pull command. With this command, we can download and display the remote state on the screen like this. The output of this command can then be passed to Json query tools like JQ to filter the required data. For example, to filter the hash key used by the dynamodb table with the name state-locking-db, we can make use of a query like this. This returns the value of lock ID as expected. 

```bash
terraform state pull

terraform state pull | jq '.resources[] | select(.name == "state-locking-db") | .instance[].attributes.hash_key'
```

The next state command is the Terraform state rm or remove command. It is used to delete items from the Terraform state file. This command is used when you no longer wish to manage one or more resources via the current Terraform configuration and state. The syntax of this command is Terraform state rm followed by the resource address. Once it is removed from the state file, make sure to remove the associated resource block from the configuration file as well. Please note that the resource removed from a state file are not actually destroyed from the real world infrastructure. It is only removed from the Terraform management. 

```bash
terraform state rm aws_s3_bucket.finance-2020922
```

That's it for this lecture. Now, let's head over to the hands-on labs and practice working with Terraform state commands.