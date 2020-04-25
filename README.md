---
page_type: sample
languages:
- csharp
products:
- azure-functions
description: "This is a simple sample which shows how to set up and write a function app which writes to a kafka topic"
---

# Azure Functions Kafka extension sample using Confluent Cloud

<!-- 
Guidelines on README format: https://review.docs.microsoft.com/help/onboard/admin/samples/concepts/readme-template?branch=master

Guidance on onboarding samples to docs.microsoft.com/samples: https://review.docs.microsoft.com/help/onboard/admin/samples/process/onboarding?branch=master

Taxonomies for products and languages: https://review.docs.microsoft.com/new-hope/information-architecture/metadata/taxonomies?branch=master
-->

This sample shows how to set up an write a .NET function app which writes to a Kafka Topic. It is using Confluent Cloud for the Kafka cluster. It also shows how to deploy this app on a Premium Function app.

## Prerequisites

* [Azure Functions Core Tools](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash)
* [Az CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) 
* [Confluent Cloud Account](https://www.confluent.io/confluent-cloud/)
Create a Confluent Cloud account. Confluent Cloud is a fully managed pay-as-you-go service. 

## Steps

### Setup the Kafka Cluster
* Log into in your Confluent Cloud account and create a new Kafka cluster.

![CreateConfluentCluster](https://github.com/Azure/azure-functions-kafka-extension-sample-confluent/blob/master/images/kafka-cluster-new.png)

* Create a new Kafka Topic called "users"


* Create a new API Key and Secret - note these values

### Update the code to point to your Kafka cluster

*. In the function trigger ensure that Protocol, AuthenticationMode, Username, Password and SslCaLocation are set.

```c#
public static class ConfluentCloudTrigger
{
    [FunctionName(nameof(ConfluentCloudStringTrigger))]
    public static void ConfluentCloudStringTrigger(
        [KafkaTrigger(
            "BootstrapServer",
            "Users",
            ConsumerGroup = "azfunc",
            Protocol = BrokerProtocol.SaslSsl,
            AuthenticationMode = BrokerAuthenticationMode.Plain,
            Username = "ConfluentCloudUsername",
            Password = "ConfluentCloudPassword",
            SslCaLocation = "confluent_cloud_cacert.pem")]
        KafkaEventData<string> kafkaEvent,
        ILogger logger)
    {
        logger.LogInformation(kafkaEvent.Value.ToString());
    }
}
```

2. In the Function App application settings (or local.settings.json during development), set the authentication credentials for your Confluent Cloud environment<br>
**BootstrapServer**: should contain the value of Bootstrap server found in Confluent Cloud settings page. Will be something like "xyz-xyzxzy.westeurope.azure.confluent.cloud:9092".<br>
**ConfluentCloudUsername**: is you API access key, obtained from the Confluent Cloud web 
site.
**ConfluentCloudPassword**: is you API secret, obtained from the Confluent Cloud web site.

3. Download and set the CA certification location. As described in [Confluent documentation](https://github.com/confluentinc/examples/tree/5.4.0-post/clients/cloud/csharp#produce-records), the .NET library does not have the capability to access root CA certificates.<br>
Missing this step will cause your function to raise the error "sasl_ssl://xyz-xyzxzy.westeurope.azure.confluent.cloud:9092/bootstrap: Failed to verify broker certificate: unable to get local issuer certificate (after 135ms in state CONNECT)"<br>
To overcome this, we need to:
    - Download CA certificate (i.e. from https://curl.haxx.se/ca/cacert.pem).
    - Rename the certificate file to anything other than cacert.pem to avoid any conflict with existing EventHubs Kafka certificate that is part of the extension.
    - Include the file in the project, setting "copy to output directory"
    - Set the SslCaLocation trigger attribute property. In the example we set to `confluent_cloud_cacert.pem`


## Setup

Explain how to prepare the sample once the user clones or downloads the repository. The section should outline every step necessary to install dependencies and set up any settings (for example, API keys and output folders).

## Running the sample

Outline step-by-step instructions to execute the sample and see its output. Include steps for executing the sample from the IDE, starting specific services in the Azure portal or anything related to the overall launch of the code.

## Key concepts

Provide users with more context on the tools and services used in the sample. Explain some of the code that is being used and how services interact with each other.

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
