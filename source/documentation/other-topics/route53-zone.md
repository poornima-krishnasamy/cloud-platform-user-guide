## Creating a Route 53 Hosted Zone

### Overview

This short guide will run through the process of creating a [Route 53 Hosted Zone][aws-hosted-zone] through Terraform for your environment.

### Pre-Requisites

This guide assumes you have an environment already created in the `Live-1` cluster and defined in the [cloud-platform-environments repository][env-repo].

### Terraform files

Copy the Terraform resource code below and save it into the respective 2 files in the `[your namespace]/resources` directory in the [cloud-platform-environments repository][env-repo]:

 * `main.tf`
 * `route53.tf`

#### main.tf
```
terraform {
  backend "s3" {}
}

provider "aws" {
  region = "eu-west-2"
}
```
Note: If you already have that file defined in your environment, do not recreate it.

#### route53.tf

Fill in the _name_ and the _namespace_ fields below, with your domain name & and your existing kubernetes namespace.

```
resource "aws_route53_zone" "example_team_route53_zone" {
  name = "YOUR DOMAIN GOES HERE"

  tags {
    business-unit          = "example-bu"
    application            = "example-app"
    is-production          = "false"
    environment-name       = "dev"
    owner                  = "example-owner"
    infrastructure-support = "example@example.com"
  }
}

resource "kubernetes_secret" "example_route53_zone_sec" {
  metadata {
    name      = "example-route53-zone-output"
    namespace = "YOUR KUBERNETES NAMESPACE GOES HERE"
  }

  data {
    zone_id   = "${aws_route53_zone.example_team_route53_zone.zone_id}"
  }
}

```

### Creating the resource

Make sure to replace the placeholders and example values above with relevant ones, commit your changes to a branch and raise a pull request.    
Once approved, you can merge and the changes will be applied.   
Shortly after, to confirm the zone has been created, you should be able to access the `Zone_ID` as Secret on kubernetes in your namespace.

Please raise a [support ticket](goo.gl/msfGiS) with the Cloud Platform.

Provide them with the domain name, the Cloud Platform team will finalize the process **by creating a a matching NS record in the DSD account.**

[env-repo]: https://github.com/ministryofjustice/cloud-platform-environments
[aws-hosted-zone]: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/AboutHZWorkingWith.html

