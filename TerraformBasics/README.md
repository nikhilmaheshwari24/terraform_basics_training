# Terraform Basics

## Using Terraform Providers

In this lecture, let's take a look at providers in more detail. We saw in the previous lecture that after we write a Terraform configuration file, the first thing to do is to initialize the directory with the `terraform init` command. When we run Terraform init within a directory containing the configuration files, Terraform downloads and installs plugins for the providers used within the configuration. These can be plugins for cloud providers such as AWS, GCP, Azure, or something as simple as the local provider that we used to create a local file type resource. 

```bash
terraform init
```

Terraform uses a plugin-based architecture to work with hundreds of such infrastructure platforms. Terraform providers are distributed by HashiCorp and are publicly available in the Terraform Registry at the URL registry.terraform.io. 

There are three tiers of providers. The first one is the official provider. These are owned and maintained by **`HashiCorp`**, and include the major cloud providers such as AWS, GCP, and Azure. The local provider that we have used so far is also an official provider. The second type of provider is a **`partner provider`**. A partner provider is owned and maintained by a third-party technology company that has gone through a partner provider process with HashiCorp. Some of the examples are the BIG-IP provider from `F5 Networks`, `Heroku`, `DigitalOcean`, etcetera. Finally, we have the community providers that are published and maintained by **`individual contributors`** of the HashiCorp community. 

A `terraform init` command when run, shows the version of the plugin that has been installed. In this case, we can see that the plugin name **`hashicorp/local`** with the version `2.0.0` has been installed in the directory. The Terraform init is a safe command, and it can be run as many times as needed without impacting the actual infrastructure that is deployed. The plugins are downloaded into a hidden directory called **`.terraform/plugins`** in the working directory containing the configuration files. In our example, the working directory is /root/terraform-local-file. The plugin name that you see here, hashicorp/local, is also known as the `source address`. This is an identifier that is used by Terraform to locate and download the plugin from the registry. 

Let's take a closer look at the format of the name. The first part of the name, which in this case is hashicorp, is the `organizational namespace`. This is followed by the `type`, which is the name of the provider such as local. Other examples of providers are AWS, AzureRM, Google, Random, etcetera. 

The plugin name can also optionally have a `hostname` in front. The hostname is the name of the registry where the plugin is located. If omitted, it defaults to `registry.terraform.io`, which is HashiCorp's public registry. Given the fact that the local provider is stored in the public Terraform Registry within the HashiCorp namespace, the source address for it can be represented as **`registry.terraform.io/hashicorp/local`**, or simply by **`hashicorp/local`** by omitting the hostname. 

```bash
# Hostname/OrganizationalNamespace/Type
registry.terraform.io/hashicorp/local

# OrganizationalNamespace/Type
hashicorp/local
```

By default, Terraform installs the latest version of the provider. Provider plugins, especially the official ones, are constantly updated with newer versions. This is done to bring in new functionality or to add in bug fixes, and these can introduce breaking changes to your code. We can lock down our configuration files to make use of a specific provider version as well. We will see how to do that later in this course.

## Configuration Directory

Now let's take a look at the configuration directory and the file naming conventions used in Terraform. So far, we have been working with a single configuration file called `local.tf` and this is within the directory called terraform/local/file, which is our configuration directory. This directory is not limited to one configuration file. We can create another configuration file like this. The `cat.tf` is another configuration file that makes use of the same local_file resource. When applied, it will create a new file called cat.txt. Terraform will consider any file with the **`.tf`** extension within the configuration directory. 

Another common practice is to have one single configuration file that contains all the resource blocks required to provision the infrastructure. A single configuration file can have as many number of configuration blocks that you need. A common naming convention used for such a configuration file is to call it the **`main.tf`**. There are other configuration files that can be created within the directory such as the **`variables.tf`**, **`outputs.tf`**, and **`providers.tf`**. We will talk more about these files in the later sections of this course. Now let's head over to the hands-on labs and explore working with providers.

## Mutliple Provider

In this lecture, we will see how to use multiple providers and resources in Terraform. Until now we have been making use of a single provider called **`local`** to deploy a local file and the system. Terraform also supports the use of multiple providers within the same configuration. 

To illustrate this, let's make use of another provider called **`random`**. This provider allows us to create random resources such as a `random ID` or `random integer` or `random password`, etcetera. Let us see how to use this provider and create a resource called **`random_pet`**. This resource type will generate a random pet name when applied. By making use of the documentation, we can add a resource block to the existing `main.tf` file like this. Here, we are making use of the resource type called random_pet. In an earlier lecture, we saw that the resource type can be broken down to two parts. The keyword before the underscore is the provider which in this case is random. The keyword following it is the resource type which is the pet. Let's call this resource "my pet". Within this resource block, we will use three arguments. One is the `prefix` that is to be added to the pet name. The second argument is the `separator` between the prefix and the pet name that is generated. The final argument is the `length` which is the length of the pet name to be generated in words.

```bash
# main.tf
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
}

resource "random_pet" "my-pet" {
    prefix = "Mrs"
    separator = "."
    length = "1"
}
```

A main.tf file now has resource definition for two different providers, one resource of the local_file type that we have already created earlier and another resource of type random_pet. Before we generate an execution plan and create these resources, we have to run the Terraform init command again. Now, this is a mandatory step, as the plugin for the random provider should be initializing the configuration directory before we can make use of it. 

```bash
terraform init
```

In the command output of the Terraform init, we can see that the local provider was previously installed and it will be reused. The plug-in for the random provider, on the other hand, will be installed as it was not used before. We can now run the Terraform plan to review the execution plan. As expected, the local file resource called "pet" will not be updated as it is unchanged from the previous apply. A new resource by the name of "my pet" will be created based on the new resource block that we just added. Now let's apply the configuration using Terraform apply. As expected, the local file resource is left as it is but now a new resource has been created which is called "my pet". Random provider is a logical provider and it displays the results of the pet name on the screen like this. Here, an attribute called ID which contains the name of the pet is written by the apply command. Before we move on, please know that in our illustration, the dog icon stands for a pet and we'll be making use of it throughout this course. The random_pet can generate any pet name and it does not have to be specifically a dog. Now, let's head over to the hands-on labs and explore working with multiple providers in Terraform.

```bash
terraform plan
terraform apply
```

## Using input Variables

In this lecture, we will see how to make use of variables in Terraform. We have used several arguments in our Terraform blocks so far. For the local file, we have a filename and content. For the random pet resource, we have used the prefix, separator, and length as the arguments. Since these values are directly defined within the main configuration files, they are considered to be hardcoded values. Hardcoding values is not a good idea. For one, this limits the reusability of the code, which defeats the purpose of using IAC. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
}

resource "random_pet" "my-pet" {
    prefix = "Mrs"
    separator = "."
    length = "1"
}
```

We want to make sure that the same code can be used again and again to deploy resources based on a set of input variables that can be provided during the execution. That is where input variables come in to the picture. Just as in any general-purpose programming language, such as Bash Scripting or PowerShell, we can make use of input variables in Terraform. To assign variables, let us create a new configuration file called **`variables.tf`** and define the values like this.

```bash
# variables.tf
variable "filename" {
    default = "/root/pets.txt"
}
variable "content" {
    default = "We love pets!"
}
variable "prefix" {
    default = "Mrs"
}
variable "separator" {
    default = "."
}
variable "length" {
    default = "1"
}
```

The variables.tf file, just like the main.tf file, consists of blocks and arguments. To create a variable, use the keyword called **`variable`**. This is followed by the variable name. This can be named anything but as a standard, use an appropriate name, such as the argument name for which we are using the variable. Within this block, we can provide a **`default`** value for the variable. This is an optional parameter, but it is a quick and simple way to assign values to the variables. We will see the other methods to do so in a later lecture. Great. 

### Now we have our variable configuration file, but how do we use it within the main.tf file? 

To do this, we can replace the argument values with the variable names prefixed with a **`.var`** like this. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = var.content
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```

When using variables, you do not have to enclose the values inside double quotes as you would when providing actual values. Using the same execution flow that we have seen many times by now, we can create the resources using Terraform plan followed by the Terraform apply. The resources have now been created as expected. 

Now, if you want to make an update to the resources by making changes to the existing arguments, we can do that by just updating the variables.tf file. The main.tf may not be modified. For example, let us update the local file resource to create the file at the same location but with an updated content that reads, "My favorite pet is Mrs. hiskers", and for the random pet resource, let's change the length of the pet name to two. We can do that like this. As expected, when we run Terraform apply, it will recreate the resources. The content of the file has been changed and the pet name now has two words following the prefix. Before we conclude this lecture, here is an example of what our configuration files would look like when creating an EC2 instance in AWS with Terraform while making use of input variables. 

```bash
# main.tf
resorurce "aws_instance" "webserver" {
    ami = var.ami
    instance_type = var.instance_type
}

# variables.tf
variable "ami" {
    default = ""
}
variable "instance_type" {
    default = "t2.micro"
}
```

Don't worry if the resource block and the arguments are unfamiliar. We have a separate lecture where we will be making use of AWS resources later in the course.

## Understanding the Variable Block

In this lecture, we will take a close look at the variables block in Terraform. First, let us look at the different arguments that a variable block uses. The variable block in Terraform access three parameters. The first one, which we have already used is the **`default`** parameter. This is where we specify the default value for a variable. The others are **`type`** and **`description`**. Description is optional, but it is a good practice to use this argument to describe what the variable is used for. The type argument is optional as well but when used, it enforces the type of variable being used. The basic variable types that can be used are **`string`**, **`number`** and **`boolean`**. 

String variables, as we have seen in our examples so far, accept a single value, which can be alphanumeric, that is, consisting of alphabets and numbers. The number variable type accepts a single value of a number, which can be positive or negative, and the boolean variable type accepts a value of true or false. The type parameter, as mentioned previously, is optional. If it is not specified in the variable block, it is set to the type **`any`** by default. Besides these three simple variable types, Terraform also supports additional types such as **`list`**, **`map`**, **`set`**, **`object`**, and **`tuple`**. 

Let us now see how to use all of these in Terraform. Let us start with list. A list is a numbered collection of values and it can be defined like this. In this example, we have a variable called prefix that uses a list of values; Mr, Mrs, and Sir. 

```bash
# variable.tf
variable "prefix" {
    default = ["Mr", "Mrs", "Sir"]  # prefix[0] = Mr; prefix[1] = Mrs; prefix[2] = Sir; 
    type = list
}

# main.tf
resource "random_pet" "my-pet" {
    prefix = var.prefix[0]  # Uses prefix Mr.
}
```

### Why do we call it a numbered collection? 

That is because each value, which is also known as an element, can be referenced by their number or index within that list. The index of a list always begins at 0. In this case, the first element of the list at index 0 is the word Mr, the element at index 1 is Mrs, and the final element at index 2 is Sir. These variables can be accessed within a configuration file like this, with the index specified within square brackets. Hence, the expression var.prefix with index 0 uses the value Mr, var.prefix with the index 1 uses the value Mrs, and with index 2, it uses the value of Sir. 

Next, let us look at the type called **`map`**. A map is data represented in the format of key-value pairs. In the variables.tf file, let us create a new variable called file-content with the type set to map. In the default values, we can specify as many key-value pairs enclosed within curly braces. Here, the statement1 and statement2 are keys, and the string data following them are values. Now, to access a specific value from the map within the Terraform configuration file, we can make use of key matching. In this case, we want the content of the local_file resource to be the value of the key called statement2, and for that, we use an expression var.file-content, which is the name of the map type variable, followed by the matching key within square brackets. 

```bash
# variable.tf
variable "file-content" {
    default = {
        "statement1" = "We love pets!"
        "statement2" = "We love animals!"
    } 
    type = list
}

# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = var.file-content["statement2"]
}
```

We can also combine type constraints. For example, if you want a list of string type elements, we can declare it like this. To use a list of numbers, change it like this. 

```bash
# variable.tf
variable "prefix" {
    default = ["Mr", "Mrs", "Sir"]  # prefix[0] = Mr; prefix[1] = Mrs; prefix[2] = Sir; 
    type = list(string)
}
# variable.tf
variable "prefix" {
    default = [1, 2, 3]  # prefix[0] = 1; prefix[1] = 2; prefix[2] = 3; 
    type = list(number)
}
```

If the variable values used do not match the type of constraint, the Terraform commands will fail. In this example, we have used the type list where the elements should be of type number, but the default values are all of type string. Now, when we run Terraform commands such as a plan or an apply, you will see an error like this that states that the default value is not compatible with the variable type constraint, and that a number is required and not a string. The same is applicable with maps as well. We can use type constraints to make sure that the values of a map are of a specific type. In the first variable block, we are using a map of type string, and in the second one, we are making use of a map that uses numbers. 

```bash
# variable.tf
variable "pet_count" {
    default = {
        "dogs" = 3
        "cats" = 1
        "goldfish" = 2
    } 
    type = map(number)
}
variable "cats" {
    default = {
        "color" = "brown"
        "name" = "bella"
    }
    type = map(string)
}

# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = var.file-content["statement2"]
}
```

Let us now look at **`sets`**. Set is similar to a list. The difference between a set and a list is that a set cannot have duplicate elements. In these examples, we have a variable type of set of strings or a set of numbers. The examples on the left are good, but the ones on the right aren't. They have duplicate values in them that will throw an error. The default values are declared just like you would do for a list, but remember that there shouldn't be any duplicate values here. 

```bash
variable "prefix" {                                     |variable "prefix" {
    default = ["Mr", "Mrs", "Sir"]      ✅              |   default = ["Mr", "Mrs", "Sir", "Sir"]   ❌   
    type = set(string)                                  |   type = set(string)
}                                                       |}
variable "fruit" {                                      |variable "fruit" { 
    default = ["apple", "banana"]       ✅              |   default = ["apple", "banana", "banana"] ❌
    type = set(string)                                  |   type = set(string)
}                                                       |}
variable "age" {                                        |variable "age" { 
    default = [10, 12, 15]              ✅              |   default = [10, 12, 15, 10]              ❌
    type = set(number)                                  |   type = set(number)
}                                                       |}
```

The next type of variable that we are going to look at are **`objects`**. With objects, we can create complex data structures by combining all the variable types that we have seen so far. For example, let us consider a new variable called "bella", which is the name of a cat. This variable is used to define the different features of this cat, such as its name which is a string; the color, which is a string as well; age, which is a number; the food that it eats, which is a list of strings; and a Boolean value indicating if it's a favorite pet or not. 

Let us now assign some values to this variable. Let us use name is equal to bella, color is equal to brown, age is equal to 7, food is "fish", "chicken", and "turkey", and favorite_pet, which is set to true. We can use the same default values within our variable block like this. 

```bash
variable "bella" {
    type = object({
        name = string
        color = string
        age = number
        food = list(string)
        favourite_pet = bool
    })
    default = {
        name = "bella"
        color = "brown"
        age = 7
        food = ["fish", "chicken", "turkey"]
        favourite_pet = true
    }
}
```

The last variable type that we are going to look at is **`tuples`**. Tuple is similar to a list and consists of a sequence of elements. The difference between a tuple and a list is that list uses elements of the same variable type such as string or number but in case of tuple, we can make use of elements of different variable types. The type of variables to be used in a tuple is defined within the square brackets. In this example, we have three types of elements defined. The first is a string, second is a number, and finally a Boolean. The variables to be passed to this should exactly be three in number and of that specific type for it to work. Here, we have passed the value of cat to the string element, the number 7 to the number element, and true to the Boolean. Adding additional elements or incorrect type will result in an error as seen here. If you add an additional value of dog to the variable, it will fail as the tuple only expects three elements of type string, number, and Boolean.

```bash
variable "kitty" {
    type = tuple([string, number, bool])        ✅
    default = ["cat", 7, true]
}

variable "kitty" {
    type = tuple([string, number, bool])        ❌
    default = ["cat", 7, true, "dog"]
}
```

That's it for this lecture. Let's head over to the hands-on labs and explore working with variable types in Terraform.

## Using Variables in Terraform

In this lecture, we will take a look at the different ways in which we can make use of input variables in Terraform. So far, we have created input variables in Terraform, and assigned default values to it based on the variable type. This is just one of the ways to pass in values to the variable. Earlier, we learned that the default parameter in a variable block is optional. This means that we can very well have our variable block look like this. 

```bash
# variables.tf
variable "filename" {

}
variable "content" {

}
variable "prefix" {
    
}
variable "separator" {

}
variable "length" {

}
```

### What would happen if we run Terraform commands now? 

When we run Terraform apply, we will be prompted to enter values for each variable used in an interactive mode. If we do not want to supply values in an interactive mode, we can also make use of command line flags like this. 

```bash
terraform apply -var "filename=/root/pets.txt" -var "content=We love Pets!" -var "prefix=Mrs" -var "separator=." -var "length=2"
```

With the Terraform command, we can make use of the **`-var`** option with the variable name equals to the value format. We can pass in as many variables as we want with this method by making use of the -var flag multiple times. We can also make use of environment variables with the **`TF_VAR_`** followed by the name of a declared variable like this. In this example, the TF_VAR_ filename sets the value of the variable called "filename" to the value "/root/pets.txt". Similarly, the variable called length now has the value of 2. 

```bash
export TF_VAR_filename="/root/pets.txt"
export TF_VAR_content="We love pets!"
export TF_VAR_prefix="Mrs."
export TF_VAR_separator="."
export TF_VAR_length="2"
```

Finally, when we are dealing with a lot of variables, we can load values by making use of variable definition files like this. These variable definition files can be named anything but should always end in either **`.tfvars`** or **`.tfvars.json`**. Here, we have declared variables and their values in a file called terraform.tfvars.

```bash
# terraform.tfvars
filename = "/roots/pets.txt"
content = "We love pets!"
prefix = "Mrs"
separator = "."
length = "2"
```

If you look at the syntax used to create this file, you'll observe that this is using the same syntax of an HCL file, but it only consists of variable assignments. The variable definition file is called terraform.tfvars or terraform.tfvars.json or by any other name ending with **`.auto.tfvars`** or **`.auto.tfvars.json`** will be automatically loaded by Terraform. If you use any other filename such as variable.tfvars, for example, you will have to parse it along with a command line flag called **`-var-file`** like this. 

```bash
terraform apply -var-file variables.tfvars
```

Finally, it is important to note that we can use any of the options that we have seen in this lecture to assign values to the variables. If we use multiple ways to assign values for the same variable, Terraform follows the variable definition precedence to understand which value it should accept. To illustrate this, let us make use of a simple example. In this case, we have a main configuration file with a single resource. A local file which will create file at a path declared in a variable called filename. In the variables.tf file, we have not specified a default value for this variable, and we have assigned different values to this variable in multiple ways. We have exported the environment variable called **`TF_VAR_filename`** with the value of /root/cats.txt. The **`terraform.tfvar`** file has a value of /root/pets.txt for the same variable. We have also made use of a variable definition file with the name **`variable.auto.tfvars`** with the value of /root/mypet.txt, and finally, we are also making use of the **`-var`** option while running the Terraform apply command with a value of /root/best-pet.txt. 

```bash
export TF_VAR_filename="/root/cats.txt"

# terraform.tfvars
filename = "/root/pets.txt"

# variable.auto.tfvars
filename = "/root/mypet.txt"

terraform apply -var "filename=/root/best-pet.txt"
```

### In this case, which one of these values would be accepted? 

Terraform follows a variable definition precedence order to determine this. First, it loads the environment variables. Next, the value in the terraform.tfvars file. This is followed by any file that ends with the .auto.tfvars or .auto.tfvars.json in an alphabetical order. Finally, Terraform considers the command line flag of -var or -var-file, which takes the highest priority and will overwrite any of the previous values. 

```text
-var or -var-file (CLI Flags) > *.auto.tfvars (alphabetical order) > terraform.tfvars > Environment Variables
```

In this case, the variable filename will be assigned the value of /root/best-pet.txt. That's it for this lecture. Let's head over to the hands-on lab and practice working with the concepts that we learned in this lecture.

## Resource Attributes

In this lecture, we will learn how to link two resources together by making use of resource attributes. In the last few lectures, we saw how to use variables to improve the reusability of our code. Right now, we have two resources in our configuration file. Each of this resource has a set of arguments that are used to create that resource. For the file resource, we have used the filename and content as the arguments. For the pet resource, we have used the prefix, separator and length. When this configuration is applied, Terraform creates a file and a random_pet resource. The name of the random_pet is displayed on the screen as an ID, which is Mr. Bull in this case. As it stands, there is no dependency between these two resources, but this rarely ever happens in a real world infrastructure provisioning process. There are bound to be multiple resources that are dependent on each other. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = "My favorite pet is Mr.Cat"
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```

### What if we want to make use of the output of one resource and use it as an input for another one? What if we want the content of the file to use the name generated by the random_pet resource? 

Currently, the content of the file is set to "My favorite pet is Mr. Cat". What if we want it to be set to the name that is generated by the random_pet resource? To understand this, let us jump back to the documentation for the random_pet resource in registry.terraform.io. You would have noticed that there are plenty of examples which are provided in the documentation, including examples for the arguments, but there is also a section called attribute reference, which provides the list of attributes returned back from the resource after you run a Terraform apply. 

In this case, the random_pet resource returns just one attribute called ID, which is of type string. The ID is the pet name generated after we run Terraform apply as we have seen before. Our goal is to make use of the attribute called ID and to make use of it as the content of the local_file resource. For this, we can make use of an expression like this. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = "My favorite pet is ${random_pet.my-pet.id}"
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```

This expression is used to reference the attribute from the resource called my-pet. The syntax for using this reference expression is resource type followed by the resource name and the attribute to be used, all of which are separated by a period or a dot. The resource type in this example is random_pet. This is followed by my-pet, which is the resource name and the attribute used is ID. 

You would have also noticed that we are making use of the dollar symbol followed by the expression which is enclosed within curly braces. This is known as an **`interpolation`** sequence. Since the content argument already uses a string type data, this sequence is used to evaluate the expression given between the curly braces, convert the result to a string, and then insert it to the final string, like this. 

Now, let us apply the changes made and allow Terraform to recreate the local file. In the output, you can see that the content is being replaced to our desired value, and it contains the name of the pet that was generated by the random_pet resource.

## Resource Dependencies

In this lecture, we will take a look at the different types of resource dependencies in Terraform. in the previous lecture, we saw how to link one resource to another using reference attributes. By making use of reference expression and interpolation, we were able to make use of the output of the random_pet resource as an input for the local file resource. now when Terraform creates these resources, it knows about the dependency since the local file resource depends on the output of the random_pet resource. As a result, it uses the following order to provision them. first Terraform crazy random_pet resource, and then it creates a local file resource. When resources are deleted, and Terraform deletes it in the reverse order, the local file first and then the random bid. This type of dependency is called the **`implicit dependency`**. Here, we're not explicitly specifying which resource is dependent on which other resource. Terraform figures it out by itself. 

However, there's another way to specify dependency within the configuration file. For example, let us make use of the older configuration file without making use of the reference expression for the file content. If you still want to make sure that the local_file resource is created after the random_pet, we can do this by using the depends on argument like this. here we have added a **`depends_on`** argument inside the resource block for the local file, and we are provided a list of dependencies that include the random_pet resource called the my pet. This will ensure that the local file is only created after the random_pet resource is created. This type of dependency is called an **`explicit dependency`**. 

```bash
# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = "My favorite pet is Mr.Cat"

    depends_on = [
        random_pet.my-pet
    ]
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```

Explicitly specifying a dependency is only necessary when a resource relies on some other resource indirectly, and it does not make use of a reference expression as seen in this case.

## Output Variables

Let us now look at output variables in Terraform. So far, we have used input variables and reference expressions in our Terraform configuration files. Along with input variables, Terraform also supports output variables. These variables can be used to store the value of an expression in Terraform. For example, let us go back to the configuration file that we used in the previous lecture. We already know that the random pet resource will generate the random pet name using the attribute called ID when we apply the configuration. To save this ID in an output variable called pet name, we can create an output block like this. The syntax used to create this output block is the keyword called output followed by the name that we want to call this variable. Inside this block, the mandatory argument for value is the reference expression. We can also add a description, which is an optional argument to describe what this output variable will be used for. 

```bash
# variables.tf
variable "filename" {
    default = "/root/pets.txt"
}
variable "content" {
    default = "We love pets!"
}
variable "prefix" {
    default = "Mrs"
}
variable "separator" {
    default = "."
}
variable "length" {
    default = "1"
}

# main.tf
resource "local_file" "pet" {
    filename = var.filename
    content = "My favorite pet is ${random_pet.my-pet.id}"
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}

output pet-name {
    value = random_pet.my-pet.id
    description = "Record the value of pet ID generated by the random_pet resource"
}
```

Once this block has been created, when we run Terraform apply, we can see that the output variable is printed on the screen. Once the resource has been created, we can also make use of the Terraform output command to print the value of the output variables. The command Terraform output by itself will print all the output variables defined and all the files in the current configuration directory. We can also use this command specifically to print the value of an existing output variable like this. 

```bash
terraform output

terraform output pet-name
```

Now, you may wonder where we can make use of these output variables. We already saw that dependent resources can make use of reference expressions to get the output from one resource block as an input to another block. As such, output variables are not really required here. The best use of Terraform output variables is when you want to quickly display details about a provision resource on the screen or to feed the output variables to other IaC tools such as an ad-hoc script or answerable playbook for configuration management and testing.