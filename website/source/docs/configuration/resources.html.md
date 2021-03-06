---
layout: "docs"
page_title: "Configuring Resources"
sidebar_current: "docs-config-resources"
description: |-
  The most important thing you'll configure with Terraform are resources. Resources are a component of your infrastructure. It might be some low level component such as a physical server, virtual machine, or container. Or it can be a higher level component such as an email provider, DNS record, or database provider.
---

# Resource Configuration

The most important thing you'll configure with Terraform are
resources. Resources are a component of your infrastructure.
It might be some low level component such as a physical server,
virtual machine, or container. Or it can be a higher level
component such as an email provider, DNS record, or database
provider.

This page assumes you're familiar with the
[configuration syntax](/docs/configuration/syntax.html)
already.

## Example

A resource configuration looks like the following:

```
resource "aws_instance" "web" {
    ami = "ami-123456"
    instance_type = "m1.small"
}
```

## Description

The `resource` block creates a resource of the given `TYPE` (first
parameter) and `NAME` (second parameter). The combination of the type
and name must be unique.

Within the block (the `{ }`) is configuration for the resource. The
configuration is dependent on the type, and is documented for each
resource type in the
[providers section](/docs/providers/index.html).

There are **meta-parameters** available to all resources:

  * `count` (int) - The number of identical resources to create.
      This doesn't apply to all resources. For details on using variables in
      conjunction with count, see [Using Variables with
     `count`](#using-variables-with-count) below.

  * `depends_on` (list of strings) - Explicit dependencies that this
      resource has. These dependencies will be created before this
      resource. The dependencies are in the format of `TYPE.NAME`,
      for example `aws_instance.web`.

  * `lifecycle` (configuration block) - Customizes the lifecycle
      behavior of the resource. The specific options are documented
      below.

The `lifecycle` block allows the following keys to be set:

  * `create_before_destroy` (bool) - This flag is used to ensure
      the replacement of a resource is created before the original
      instance is destroyed. As an example, this can be used to
      create an new DNS record before removing an old record.

  * `prevent_destroy` (bool) - This flag provides extra protection against the
      destruction of a given resource. When this is set to `true`, any plan
      that includes a destroy of this resource will return an error message.

-------------

Within a resource, you can optionally have a **connection block**.
Connection blocks describe to Terraform how to connect to the
resource for
[provisioning](/docs/provisioners/index.html). This block doesn't
need to be present if you're using only local provisioners, or
if you're not provisioning at all.

Resources provide some data on their own, such as an IP address,
but other data must be specified by the user.

The full list of settings that can be specified are listed on
the [provisioner connection page](/docs/provisioners/connection.html).

-------------

Within a resource, you can specify zero or more **provisioner
blocks**. Provisioner blocks configure
[provisioners](/docs/provisioners/index.html).

Within the provisioner block is provisioner-specific configuration,
much like resource-specific configuration.

Provisioner blocks can also contain a connection block
(documented above). This connection block can be used to
provide more specific connection info for a specific provisioner.
An example use case might be to use a different user to log in
for a single provisioner.

<a id="using-variables-with-count"></a>

## Using Variables With `count`

When declaring multiple instances of a resource using [`count`](#count), it is
common to want each instance to have a different value for a given attribute.

You can use the `${count.index}`
[interpolation](/docs/configuration/interpolation.html) along with a mapping [variable](/docs/configuration/variables.html) to accomplish this.

For example, here's how you could create three [AWS Instances](/docs/providers/aws/r/instance.html) each with their own static IP
address:

```
variable "instance_ips" {
  default = {
    "0" = "10.11.12.100"
    "1" = "10.11.12.101"
    "2" = "10.11.12.102"
  }
}

resource "aws_instance" "app" {
  count = "3"
  private_ip = "${lookup(var.instance_ips, count.index)}"
  # ...
}
```

## Multiple Provider Instances

By default, a resource targets the provider based on its type. For example
an `aws_instance` resource will target the "aws" provider. As of Terraform
0.5.0, a resource can target any provider by name.

The primary use case for this is to target a specific configuration of
a provider that is configured multiple times to support multiple regions, etc.

To target another provider, set the `provider` field:

```
resource "aws_instance" "foo" {
	provider = "aws.west"

	# ...
}
```

The value of the field should be `TYPE` or `TYPE.ALIAS`. The `ALIAS` value
comes from the `alias` field value when configuring the
[provider](/docs/configuration/providers.html).

If no `provider` field is specified, the default (provider with no alias)
provider is used.

## Syntax

The full syntax is:

```
resource TYPE NAME {
	CONFIG ...
	[count = COUNT]
	[depends_on = [RESOURCE NAME, ...]]
	[provider = PROVIDER]

    [LIFECYCLE]

	[CONNECTION]
	[PROVISIONER ...]
}
```

where `CONFIG` is:

```
KEY = VALUE

KEY {
	CONFIG
}
```

where `LIFECYCLE` is:

```
lifecycle {
    [create_before_destroy = true|false]
}
```

where `CONNECTION` is:

```
connection {
	KEY = VALUE
	...
}
```

where `PROVISIONER` is:

```
provisioner NAME {
	CONFIG ...

	[CONNECTION]
}
```
