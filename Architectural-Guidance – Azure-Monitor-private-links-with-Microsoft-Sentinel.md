
Author: Mahmoud Elsayed


///

###### This article was published on Microsoft's official techcommunity on 08.12.2022 at following link: 
https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/architectural-guidance-azure-monitor-private-links-with/ba-p/3692527

\\\


Firstly, I would like to thank  **Benjamin Kovacevic** for his help with this article.

In this blog post, I will try to simplify one of the confusions and a popular question seen by many organizations around the ability to use private links together with Microsoft Sentinel.

### Starting with basics:

### Microsoft Sentinel
Microsoft Sentinel is a cloud-native SIEM solution that is built on top of the log analytics workspace and hence, Microsoft Sentinel requires data to be ingested into that same log analytics workspace for its operation.


### Log analytics workspace
Log analytics workspace is a native Azure monitoring resource that is part of Azure monitor as Azure monitor contains also resources like application insights and DCR resources and so on.


### Microsoft Sentinel log source types
Microsoft Sentinel log sources are either:

+ **Diagnostic-based data sources**: This type covers data ingested through the diagnostic settings from Azure PaaS and/or Saas services. Examples like: Activity logs, Azure AD audit logs, Azure Data factories, Key vaults, and so on. Once configured, data starts to flow from the Azure resource to the log analytics workspace.
+ **Service-to-service data integration**: This type covers direct connections from other Microsoft services like Defender for Endpoint, Defender for Cloud, Defender for Office365, and so on. Once the connection is turned on, data starts to flow automatically through the Azure backend to the log analytics workspace.
+ **Agent-based-ingestion log sources:** This covers all ingestion that is based on either AMA or LAA (MMA) agents. Data sources could be VMs that are running in Azure, on-premises, or in other cloud platforms.
+ **REST API based ingestion:** This covers data ingestion and queries through pipelines line LogicApp connectors, Function Apps, and some 3rd party connectors in Microsoft Sentinel.
 

### Azure Monitor Private link
Private link in Azure Monitor is a network restriction and security mechanism that could be used to force traffic to flow only through private connections from a VNET to an azure monitor resource. In this context, we will focus on log analytics workspace as our Azure monitor resource.

**Type1:** Ingesting data through diagnostic settings

As mentioned in this document under exception section, data ingested through diagnostic settings pipeline by default go over a secure private channel and is not impacted by private links.


 
 ![This is an image](https://github.com/mahelsay/msft/blob/main/images/1.png)



The same goes for type2 service-to-service data integrations as they also flow through Azure backbone.

**Type3:** Agent-based-ingestion log sources

The best way to look at the concept under the context of this type3 is to examine the following diagram taken from this document


 ![This is an image](https://github.com/mahelsay/msft/blob/main/images/2.png)


**Note** that On-premises here could also be replaced by VNETs on azure as well because the same concept applies.

 

So the idea is simply that traffic from on-premises (or any VNET on Azure) will communicate to the private endpoint IP address that is associated with the private link scope object.

**Fact 1:** This basically means that it primarily depends on how DNS is configured.

**Fact 2:** On the workspace level, an On/Off setting exists to control whether to accept data ingestion not originating from private link scope or not.

**Fact 3:** On the workspace level, an On/Off setting exists to control whether to accept log queries not originating from private link scope or not.

**Fact 4:** The private link scope could be covering all log analytic workspaces or some of them. At this point, we need to pay attention to the private link mode (private only or open).

+   **Private only mode:** allows the traffic VNet to only reach resources in the link scope. traffic to log analytics workspace out of the link scope is blocked.

+   **Open mode:** allows the VNet to reach log analytic workspaces that are covered by the private link scope AND log analytics workspaces that are not covered by the private link scope. (if they accept traffic from public networks). The Open mode is useful for a mixed mode of work (accessing some resources publicly and others over a Private Link), or during a gradual onboarding process.


So to simplify it, the following matrix should give an idea of how the result looks like for interactions between these four items. For other workspaces that are not covered by the same link scope, the following matrix applies


| **link scope mode** | **Workspace Settings** | outcome for Public traffic  |
|---------------------|------------------------|-----------------------------|
|     private only    | On                     | Blocked                     |
|     private only    | Off                    | Blocked                     | 
|     Open            | On                     | Allowed                     |
|     Open            | Off                    | Blocked                     | 



As expected, any log ingestion traffic for log analytics workspaces that are not covered by the same private link scope will be denied and only allowed if the link scope mode is set to **Open**


### **Our Recommendations from the field**

+ Considering Azure monitor private link should be associated with either a concrete requirement or certain compliance obligations.
+ Use **link mode: open** when newly onboarding Microsoft Sentinel and switch to **link mode: private only** mode only after careful assessment of implication on all log analytics workspaces that are available and assessment of network and DNS design.


Thank you

*Mahmoud Elsayed, mahelsay@gmail.com*