To deploy infrastructure with Terraform:
----------------------------------------

    Scope - Identify the infrastructure for your project.
    Author - Write the configuration for your infrastructure.
    Initialize - Install the plugins Terraform needs to manage the infrastructure.
    Plan - Preview the changes Terraform will make to match your configuration.
    Apply - Make the planned changes.

# What to know

How to create an account on terraform
How to create an organization
How to create a workspace
How to migrate state file from local to terraform cloud


Defining variables
==================
How to define:
``` Terraform
variable "enable_vpn_gateway" {
  description = "Enable a VPN gateway in your VPC."     # required
  type        = bool                                    # required
  default     = false                                   # optional
}
```

Types available:
----------------
Primitive Types
---------------
    bool
    string
    number
    List
    Map
    Set
    Tuple
    Object
    any
    optional() # From Terraform v0.14, experimental support

Collection Types
----------------
list : List of elements where all elements are of the same type
map : A lookup table, matching keys to values, all of the same type.
set : An unordered collection of unique values, all of the same type.

Structural Types
----------------
object : collection of named attributes that each have their own type.

    The schema for objects is { <KEY> = <TYPE>, <KEY> = <TYPE> }

tuple : sequence of elements identified by consecutive whole numbers starting with zero, where each element has its own type

    The schema for tupe is [<TYPE>, <TYPE>, ...]

Terraform does not provide any way to directly represent lists, maps, or sets. However, due to the automatic conversion of complex types (described below), the difference between similar complex types is almost never relevant to a normal user, and most of the Terraform documentation conflates lists with tuples and maps with objects. The distinctions are only useful when restricting input values for a module or resource.

Built in functions
==================
numeric functions:
    abs
    ceil
    floor
    flog
    max
    min
    parseint
    pow
    signum

string functions:
    chomp
    format
    formatlist
    indent
    join
    lower
    regex
    regexall
    replace
    split
    strrev
    substr
    title
    trim
    trimprefix
    trimsuffix
    trimspace
    upper

Commands:
=========
    terraform init:                                             Downloads providers using version in lock file and set lock file 
                                                                and module referenced. If played terraform init again, downloads the new modules added.
    terraform init -upgrade:                                    Upgrades providers to the latest version of their providers and modules
                                                                according to the version set in providers config.
    terraform init -reconfigure                                 Force reconfiguration of terraform backend (disregards current state)
    terraform init -migrate-state                               Force reconfiguration of terraform backend and migrate current state to it
    terraform init -backend=false                               Prevent init step to reconfigure backend
    terraform fmt:                                              Format configuration to consistent style
    terraform validate:                                         Validates that it's a valid terraform file
    teraform show:                                              Inspect current state
    terraform state list:                                       List resources in state file 
    terraform destroy:                                          Destroy infrastructure
    terraform output:                                           Prints all outputs
    terraform output \<name\>                                   Prints output name
    terraform output -raw \<name\>                              Prints output name in machine-readable format (sensitive values are not printed)
    terraform output -json                                      Prints output name in json format (sensitive values are printed)
    terraform plan -replace="aws_instance.example"              Instructs Terraform to plan to replace the resource instance with the   
                                                                given address.
    terraform plan -target="aws_instance.example"               Instructs Terraform to plan to replace only the given infrastructure
    terraform apply -replace="aws_instance.example"             Apply and force Terraform to destroy and recreate the resource
    terraform state mv \[Options\] SOURCE DESTINATION           Move state from one state file to another
    terraform state rm RESOURCE                                 Delete a resource or a module from a state file
    terraform \[global options\] import \[options\] ADDR ID     Import resource as name ADDR using ID
    terraform plan -refresh                                     Shows what a refresh would change
    terraform apply -refresh                                    Apply refresh (This is the new command for terraform refresh)
    terraform apply -var-file \<\>
    terraform refresh                                           Refreshes state file according to resources
    terraform get                                               Downloads modules and or providers (No need to re-run if using a local module)

Workspace
---------
terraform workspace list            : Lists all workspaces
terraform workspace show            : Outputs the currently running workspace
terraform workspace new /<name/>    : Creates a new workspace
terraform workspace select /<name/> : Allows you to select a workspace
terraform workspace delete /<name/> : Deletes a workspace. (Your workspace should be empty and not the currently running one. -f to force)

Resource Adressing
==================
A resource address is a string that identifies zero or more resource instances in your overall configuration.

An address is made up of two parts:
    [module path][resource spec]

Module Path
-----------
A module path addresses a module within the tree of modules. It takes the form:
    module.module_name[module index]

* module - Module keyword indicating a child module (non-root). Multiple module keywords in a path indicate nesting.
* module_name - User-defined name of the module.
* [module index] - (Optional) Index to select an instance from a module call that has multiple instances, surrounded by square bracket characters ([ and ]).

Example:
    module.foo[0].module.bar["a"]

Resource Spec
-------------
A resource spec addresses a specific resource instance in the selected module. It has the following syntax:
    resource_type.resource_name[instance index]

* resource_type - Type of the resource being addressed.
* resource_name - User-defined name of the resource.
* [instance index] - (Optional) Index to select an instance from a resource that has multiple instances, surrounded by square bracket characters ([ and ]).


TFVARS
------
.tfvars file are files that will help input variables into a configuration

By default, terraform.tfvars and *.auto.tfvars will be loaded automatically

Variable Definition Precedence

Terraform loads variables in the following order, with later sources taking precedence over earlier ones:
    Environment variables
    The terraform.tfvars file, if present.
    The terraform.tfvars.json file, if present.
    Any *.auto.tfvars or *.auto.tfvars.json files, processed in lexical order of their filenames.
    Any -var and -var-file options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)

String interpolation
--------------------
To interpolate strings you should just add ${} and it will return the evaluation

example

    "lb-sg-${var.resource_tags["project"]}-${var.resource_tags["environment"]}"

    To interpolate strings you should

Terraform output
----------------
You can mark terraform outputs as sensitive to avoid accidentally printing them out to console

Example:
``` terraform
output "db_password" {
  description = "Database administrator password"
  value       = aws_db_instance.database.password
  sensitive   = true
}
```

NOTE: Output values are stored as plain text in your state file — including those flagged as sensitive. 

Depends on
----------
Implicit dependencies between resources and modules are found by inspecting parameters

Explicit dependencies can be set using depends_on parameter

Example to set explicit dependency
    depends_on = [aws_s3_bucket.example, aws_instance.example_c]

Template file
-------------
Use file as input and replace values using the ${} syntax by the values from map

How to call:
    templatefile(path, map)

input:
    path: path of template file. You can use the ${} syntax to insert values from map
    map: contains variables that could be replaced in the templatefile

Lookup function
---------------
Lookup from a map the value of a key

How to call:
    lookup(map, key)

input:
    map: a map containing a key string and variables associated to each key
    key

Modules
=======

Why Modules?
------------
* To manage complexity
* Make changes easier to do
* Avoid code duplication
* Share configuration files between projects and teams

In other words:
* Organize configuration 
* Encapsulate configuration   
* Re-use configuration
* Provide consistency and ensure best practices

What is a Module?
-----------------
A terraform module is a set of terraform configuration files in a single directory.  
Even a simple configuration consisting of a single directory with one or more .tf files is a considered a module.

When you run Terraform commands directly from a directory, it is considered the root module.  
So in this sense every Terraform configuration is part of a module.

Calling Modules
---------------
Terraform commands will only directly use the configuration files in one directory, which is usually the current working directory. However, your configuration can use module blocks to call modules in other directories. When Terraform encounters a module block, it loads and processes that module's configuration files.

A module that is called by another configuration is sometimes referred to as a "child module" of that configuration.

Modules can either be loaded from the local filesystem, or a remote source.

Terraform supports a variety of remote sources, including the Terraform Registry, most version control systems, HTTP URLs, and Terraform Cloud or Terraform Enterprise private module registries.

Creating Modules
----------------
Notice that there is no provider block in this configuration. When Terraform processes a module block, it will inherit the provider from the enclosing configuration. Because of this, we recommend that you do not include provider blocks in modules.

A module intended to be called by one or more other modules must not contain any provider blocks. A module containing its own provider configurations is not compatible with the for_each, count, and depends_on arguments that were introduced in Terraform v0.13.

In a module you should only define required_providers to enable users to change terraform versions easily

Terraform get vs Terraform init
-------------------------------
Whenever you add a new module to a configuration, Terraform must install the module before it can be used. Both the terraform get and terraform init commands will install and update modules. The terraform init command will also initialize backends and install plugins.

Best Practices
--------------
1. Name your provider terraform-\<PROVIDER\>-\<NAME\> You must follow this convention in order to publish to the Terraform Cloud or Terraform Enterprise module registries.
2. Start writing your configuration with modules in mind. Even for modestly complex Terraform configurations managed by a single person, you'll find the benefits of using modules outweigh the time it takes to use them properly.
3. Use local modules to organize and encapsulate your code. Even if you aren't using or publishing remote modules, organizing your configuration in terms of modules from the beginning will significantly reduce the burden of maintaining and updating your configuration as your infrastructure grows in complexity.
4. Use the public Terraform Registry to find useful modules. This way you can more quickly and confidently implement your configuration by relying on the work of others to implement common infrastructure scenarios.
5. Publish and share modules with your team. Most infrastructure is managed by a team of people, and modules are important way that teams can work together to create and maintain infrastructure. As mentioned earlier, you can publish modules either publicly or privately. We will see how to do this in a future tutorial in this series.

Modules meta arguments
----------------------
* source: defines where it will call the module from
* version: When using modules installed from a module registry, we recommend explicitly constraining the acceptable version numbers to avoid unexpected or unwanted changes. The version argument accepts a version constraint string.
* count: creates multiple instances of a module from a single module block. Creates **count** many instances of the resource or module. 
* for_each: Create multiple instances of a module from a single module block. 
* providers: Passes provider configurations to a child module. If not specified, the child module inherits all of the default (un-aliased) provider configurations from the calling module.
* depends_on: Creates explicit dependencies between the entire module and the listed targets.

Moving modules
--------------
To move resources to modules define refactoring blocks to say from where to where you want to move the resource
and then play a terraform plan -replace to force terraform to propose replacing that object.

Because replacing is a very disruptive action, Terraform only allows selecting individual resource instances. There is no syntax to force replacing all resource instances belonging to a particular module.

When to use for_each instead of count
-------------------------------------
If your instances are almost identical, count is appropriate. If some of their arguments need distinct values that can't be directly derived from an integer, it's safer to use for_each.

For each extra info
-------------------
The for_each meta-argument accepts map or set expressions. However, unlike most arguments, the for_each value must be known before Terraform performs any remote resource actions. This means for_each can't refer to any resource attributes that aren't known until after a configuration is applied (such as a unique ID generated by the remote API when an object is created).

Modules information
-------------------
Variables declared in modules that aren't given a default value are required, and so must be set whenever you use the module.

When creating a module, consider which resource arguments to expose to module end users as input variables.

WHat to know
------------
* Create a Terraform module
* Use local Terraform modules in your configuration
* Configure modules with variables
* Use module outputs

Best practices when building modules
------------------------------------
* Encapsulation: Group infrastructure that is always deployed together. Including more infrastructure in a module makes it easier for an end user to deploy that infrastructure but makes the module's purpose and requirements harder to understand.
* Privileges: Restrict modules to privilege boundaries.
* If infrastructure in the module is the responsibility of more than one group, using that module could accidentally violate segregation of duties. Only group resources within privilege boundaries to increase infrastructure segregation and secure your infrastructure.
* Volatility: Separate long-lived infrastructure from short-lived.
* For example, database infrastructure is relatively static while teams could deploy application servers multiple times a day. Managing database infrastructure in the same module as application servers exposes infrastructure that stores state to unnecessary churn and risk.

Moved block
-----------
How to use:
    moved {
    from = module.vpc
    to   = module.learn_vpc
    }

Note: We strongly recommend you retain all moved blocks in your configuration as a record of your changes. Removing a moved block plans to delete that existing resource instead of moving it.

Locals vs Vars
--------------
locals: Used for hard coding values
vars: Used for letting user choose values
    required: if var has no default value then it is required
    optional: if var has a default value then it is optional


Things to review:
-----------------
https://www.terraform.io/language/expressions/dynamic-blocks

Version Constraint
==================
Example
-------
    version = ">= 1.2.0, < 2.0.0"

Version Operators
-----------------
* = (or no operator): Allows only one exact version number. Cannot be combined with other conditions.
* !=: Excludes an exact version number.
* >, >=, <, <=: Comparisons against a specified version, allowing versions for which the comparison is true. "Greater-than" requests newer versions, and "less-than" requests older versions.
* ~>: Allows only the rightmost version component to increment. For example, to allow new patch releases within a specific minor release, use the full version number: ~> 1.0.4 will allow installation of 1.0.5 and 1.0.10 but not 1.1.0. This is usually called the pessimistic constraint operator.

Best Practices
==============
Module Versions
---------------
* When depending on third-party modules, require specific versions to ensure that updates only happen when convenient to you.
* For modules maintained within your organization, specifying version ranges may be appropriate if semantic versioning is used consistently or if there is a well-defined release process that avoids unwanted updates.

Terraform Core and Provider Versions
------------------------------------
* Reusable modules should constrain only their minimum allowed versions of Terraform and providers, such as >= 0.12.0. This helps avoid known incompatibilities, while allowing the user of the module flexibility to upgrade to newer versions of Terraform without altering the module.
* Root modules should use a ~> constraint to set both a lower and upper bound on versions for each provider they depend on.

Troubleshooting workflow
========================
Commands
--------
    terraform fmt: to format configuration
    terraform validate: To validate configuration 

Enable terraform core logging (Helps troubleshoot core related bugs)
    export TF_LOG_CORE=TRACE        : To enable core log tracing
    export TF_LOG_PATH=logs.txt     : To chose output file
    terraform refresh               : To generate an example of core and provider logs

What can you log
----------------
TF_LOG_CORE         : set logging for only core
TF_LOG_PROVIDER     : set logging only for providers
TF_LOG_PATH         : To set output file and persist logs
TF_LOG              : set logging for everything (must be set for any logging to be enabled)

Log levels
----------
TRACE, DEBUG, INFO, WARN or ERROR to change the verbosity of the logs.

For Loop Example
----------------
    [for instance in objects: instance.value]

Variable declaration
====================
Terraform CLI defines the following optional arguments for variable declarations:

    default - A default value which then makes the variable optional.
    type - This argument specifies what value types are accepted for the variable.
    description - This specifies the input variable's documentation.
    validation - A block to define validation rules, usually in addition to type constraints.
    sensitive - Limits Terraform UI output when the variable is used in configuration.
    nullable - Specify if the variable can be null within the module.

Example
-------

    variable my-named-variable {
        default = value
        type    = string
        description = is a named variable to use
        validation {
            condition     = length(var.my-named-variable) >= 4
            error_message = "The my-named-variable value must be of at least 4 characters."
        }
        sensitive = true
        nullable = false
    }


Things to learn later:
----------------------
    How to migrate multiple state files to hashicorp cloud
    What does init do in details

Resources Meta-arguments
========================
depends_on, count, for_each, provider, lifecycle

depends_on: Forces resource to be created after another

count: Creates count instances of a resource

for_each: meta-argument accepts a map or a set of strings, and creates an instance for each item in that map or set. Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.

Example:
    resource "azurerm_resource_group" "rg" {
        for_each = {
            a_group = "eastus"
            another_group = "westus2"
        }
        name     = each.key
        location = each.value
    }

lifecycle: The general lifecycle for resources is described in the Resource Behavior page. Some details of that behavior can be customized using the special nested lifecycle block within a resource block bodyo

The arguments available within a lifecycle block are create_before_destroy, prevent_destroy, ignore_changes, and replace_triggered_by.

    resource "aws_appautoscaling_target" "ecs_target" {
        # ...
        lifecycle {
            replace_triggered_by = [
            # Replace `aws_appautoscaling_target` each time this instance of
            # the `aws_ecs_service` is replaced.
            aws_ecs_service.svc.id
            ]
        }
    }

precondition and postcondition blocks with a lifecycle block to specify assumptions and guarantees about how resources and data sources operate. The following examples creates a precondition that checks whether the AMI is properly configured.

    resource "aws_instance" "example" {
        instance_type = "t2.micro"
        ami           = "ami-abc123"

        lifecycle {
            # The AMI ID must refer to an AMI that contains an operating system
            # for the `x86_64` architecture.
            precondition {
            condition     = data.aws_ami.example.architecture == "x86_64"
            error_message = "The selected AMI must be for the x86_64 architecture."
            }
        }
    }

IMPORTANT NOTE: Literal Values Only

The lifecycle settings all affect how Terraform constructs and traverses the dependency graph. As a result, only literal values can be used because the processing happens too early for arbitrary expression evaluation.

Data sources:
-------------

To reference data sources

    data.datasource_name...

Example:
    output "aws_region" {
        description = "AWS region"
        value       = data.aws_region.current.name
    }

Pre-conditions and post-condition can be applied to data
in order to validate input and output

data sources support all meta-arguments as resources except for the **lifecycle** configuration block

Ternary operator (Dynamic expressions)
--------------------------------------
    (var.name != "" ? var.name : random_id.id.hex)

Sentinel
--------

Note: Sentinel policies are a paid feature, available as part of the Team & Governance upgrade package.
