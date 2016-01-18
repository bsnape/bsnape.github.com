---
layout: post
title: "Terraform Design Patterns: the Terrafile"
---

## Introduction

Taken straight from the documentation, [Terraform](https://www.terraform.io/)

> provides a common configuration to launch infrastructure — from physical and virtual servers to email and DNS providers. Once launched, Terraform safely and efficiently changes infrastructure as the configuration is evolved.

Here is an example which creates a [VPC](https://aws.amazon.com/vpc/) using a [Terraform module](https://www.terraform.io/intro/getting-started/modules.html)
(similar to a class in a programming language).

{% highlight go %}
module "vpc" {
  source = "github.com/terraform-community-modules/tf_aws_vpc/?ref=v1.0.0"

  name            = "my-vpc"
  cidr            = "10.0.0.0/16"
  private_subnets = "10.0.1.0/24,10.0.2.0/24,10.0.3.0/24"
  public_subnets  = "10.0.101.0/24,10.0.102.0/24,10.0.103.0/24"
  azs             = "us-west-2a,us-west-2b,us-west-2c"
}
{% endhighlight %}

Straightforward so far.

However, what immediately struck me (coming from a development background) was the way that modules were referenced - i.e. specifying the module `source` within
the body of the module implementation.

Combining the module source with its use feels like a mix of concerns to me.

Additionally, each time you reference a module you must specify its source - even if you used that module elsewhere in your project.

I believe that abstracting the `source` location to another file (separate from the implementation) would make much more sense.


## Module Versioning

Before we cover how we might achieve that, let's quickly cover [Terraform module versioning](https://www.terraform.io/docs/modules/sources.html#ref).

It is also possible (and good practice) to tack on a version at the end of the `source` parameter.

{% highlight go %}
module "vpc" {
  source = "github.com/terraform-community-modules/tf_aws_vpc/?ref=v1.0.0"
  ...
}
{% endhighlight %}

Specifying the version (e.g. a git tag) is great as it means multiple teams can contribute to the same Terraform modules without breaking functionality
for others.

However, upgrading even a single module in a project can be quite a laborious and manual process. Consider a setup with dozens or even
hundreds of ASGs (autoscale groups), spread across numerous `.tf` files and various environments (QA, SIT, Stage, Prod etc.) with each
using a Terraform module to implement said ASGs.
 
Any non-trivial project will require many other modules e.g. Security Groups, VPCs, subnets, Route53, EBS etc. - suddenly you have
a lot of things to change!


## Terrafile

The combination of _a mix of concerns with the module source and implementation_ with _a potentially laborious and error prone module upgrade process_ resulted
in the creation of a `Terrafile` to address these issues.

A `Terrafile` is simple YAML config that gives you a single, convenient location that lists all your external module dependencies.

The idea is modelled on similar patterns in other languages - e.g. Ruby with its `Gemfile` (technically provided by the `bundler` gem).
 
Here's what a `Terrafile` might look like.

{% highlight yaml %}
tf-aws-vpc:
 source:  "git@github.com:ITV/tf-aws-vpc"
 version: "v0.1.1"
tf-aws-iam:
 source:  "git@github.com:ITV/tf-aws-iam"
 version: "v0.1.0"
tf-aws-sg:
 source:  "git@github.com:ITV/tf-aws-sg"
 version: "v1.3.1"
tf-aws-asg:
 source:  "git@github.com:ITV/tf-aws-asg"
 version: "v2.0.0"
{% endhighlight %}


Below is a simplistic example in Ruby/Rake of how you might implement the `Terrafile` pattern. No gems are required (except Rake of course).
Simply place the code in a `Rakefile` and execute using `rake get_modules`.
 
{% highlight ruby %}
require 'yaml'
require 'fileutils'

# You may want to change this.
def modules_path
  'vendor/modules'
end

# You may want to change this.
def terrafile_path
  'Terrafile'
end

def read_terrafile
  if File.exist? terrafile_path
    YAML.load_file terrafile_path
  else
    fail('[*] Terrafile does not exist')
  end
end

def create_modules_directory
  unless Dir.exist? modules_path
    puts "[*] Creating Terraform modules directory at '#{modules_path}'"
    FileUtils.makedirs modules_path
  end
end

def delete_cached_terraform_modules
  puts "[*] Deleting cached Terraform modules at '#{modules_path}'"
  FileUtils.rm_rf modules_path
end

desc 'Fetch the Terraform modules listed in the Terrafile'
task :get_modules do
  terrafile = read_terrafile

  create_modules_directory
  delete_cached_terraform_modules

  terrafile.each do |module_name, repository_details|
    source  = repository_details['source']
    version = repository_details['version']
    puts "[*] Checking out #{version} of #{source} ...".colorize(:green)

    Dir.mkdir(modules_path) unless Dir.exist?(modules_path)
    Dir.chdir(modules_path) do
      `git clone -b #{version} #{source} #{module_name} &> /dev/null`
    end
  end
end
{% endhighlight %}


## Implementation

Implementation is quick and easy. We've covered most of it already but let's recap.

1. Create your `Terrafile`.
2. Implement a way to read your `Terrafile` and fetch the required modules (working Ruby example above).
3. Modify all `source` variables in your Terraform project to point to your new cached modules directory (provided by the `modules_path` method above)
rather than GitHub e.g.

{% highlight go %}
module "vpc" {
  source = "../vendor/modules/tf_aws_vpc"
  ...
}
{% endhighlight %}

You can read more about Terraform module sources in the [official documentation](https://www.terraform.io/docs/modules/sources.html).
  
  
## Other Considerations and Limitations

If you need to support multiple different versions of the same module (an incremental upgrade for instance), the Ruby/Rake implementation
above takes the `Terrafile` key name into account. For example, the following will be deployed to `vendor/modules/vpc-0.0.1` and
`vendor/modules/vpc-2.0.0` respectively.

{% highlight go %}
vpc-0.0.1:
 source:  "git@github.com:ITV/tf-aws-vpc"
 version: "v0.0.1"
vpc-2.0.0:
 source:  "git@github.com:ITV/tf-aws-vpc"
 version: "v2.0.0"
{% endhighlight %}

Additionally, the deletion and subsequent fetching of the Terrafile modules is very simplistic. Each time `rake get_modules` is executed, all cached
modules are removed and re-fetched.


## Conclusion

It feels repetitive and prone to error to keep specifying modules and their version information, especially for larger teams who share
modules. Terraform is rapidly evolving - to keep up you must frequently update your modules.

Probably my favourite aspect of using a `Terrafile` is being to see at a glance exactly which modules and which versions are being used
by a project, just like a `Gemfile` or a `Puppetfile`. Outdated or buggy dependencies are often the root cause of runtime issues and
this shortens the debugging and investigation considerably when things go wrong.

I'm a huge fan of Terraform and other Hashicorp products (especially Consul and Vagrant). I hope this design pattern helps others to use
Terraform even more productively.
