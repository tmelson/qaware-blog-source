---
title: "Monitoring cloud costs with unit testing frameworks"
date: 2021-03-05
lastmod: 2021-03-05
author: "Tobias Melson"
type: "post"
image: "cash-register-cc0.jpg"
tags: ["Cloud", "AWS", "JUnit", "Spock"]
summary: "Our new Cloud Cost Fitness framework enables you to monitor your cloud expenses with Spock or JUnit 5 in your nightly build."
draft: true
---

Monitoring the actual expenses of your application in the cloud is important. With a few clicks or the wrong Terraform command, you can easily exceed your credit card limit.
The large cloud providers Amazon AWS, Google Cloud, and Microsoft Azure present the user's billing data in various diagrams and offer sending budget notifications.
However, we are not only interested in the total budget. We have more fine-grained questions, for example, _What is the most expensive virtual machine currently running?_ or _How expensive will that instance be next week given the data of last month?_. Ideally, we would like to define a bunch of questions for various scenarios and answer them regularly as part of a nightly job. This is exactly what the _Cloud Cost Fitness_ framework is about.

## Cloud Cost Fitness

We created a unit testing framework for your cloud costs. It offers an API module covering the core functionality and some interfaces for provider-specific methods. For the current version 1.0.1, we implemented it for __AWS__. Further implementations for other cloud providers are planned in future releases.

If you want to try it with the billing data of your AWS account, clone it from [here](https://github.com/qaware/cloud-cost-fitness/) and perform the following steps:

### Preparational tasks

Before you start writing your tests, you need to ensure that you widely tag your instances with unique names, because this is the basis for fetching data from the AWS API. It is highly recommended to also include your environment (integration or production) in the name to further distinguish the costs. Check out the [AWS Tagging Best Practices](https://d1.awsstatic.com/whitepapers/aws-tagging-best-practices.pdf) for further input on that topic.

For accessing the billing data, user credentials with read access to the AWS Cost Explorer are required. We recommend creating a new special user for this purpose. You can find the required user policy [here](https://github.com/qaware/cloud-cost-fitness/blob/main/cloud-cost-fitness-aws/src/main/bash/aws-billing-policy.json).

### Implement your tests

You are now ready to start implementing your first test! Select the unit testing framework you like most: Spock or JUnit 5. In our repository, you can find examples providing some inspiration about what is possible with our Cloud Cost Fitness framework.

Here is an example of a Spock test verifying that the costs of all instances with tag names _gateway-prod-*_ are below 50 EUR within the last 7 days:

```groovy
@Requires({ sys['aws.access.key'] && sys['aws.secret.key'] })
class CloudCostFitnessSpec extends Specification {
    @Shared
    CostExplorer costExplorer = CloudProvider.AMAZON_AWS.getCostExplorer()

    def "checks the costs for nginx last week"() {
        expect: "the total costs of all instances is less than a threshold"
        costExplorer.during(LAST_7_DAYS).forInstance("gateway-prod-*").getCosts().sum().lessThan(50.0)
    }
}
```

## Future work

In upcoming releases, we want to provide the implementations for Google Cloud and Microsoft Azure. Contributions from the community are also very welcome! Do not hesitate to contact us or send us a pull request.


