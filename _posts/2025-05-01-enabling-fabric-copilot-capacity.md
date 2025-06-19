---
title: "Copilot on Fabric F2 SKU: Enabling Fabric Copilot Capacity and use cases (Updated June 2025)"
categories:
  - blog
tags:
  - Fabric
  - Copilot
  - Data
---

## What is Fabric Copilot?

**Fabric Copilot** is an AI-powered assistant integrated into Microsoft Fabric, designed to help users analyze data, generate insights, and automate tasks within the Fabric ecosystem. It leverages generative AI to assist with data preparation, report creation, and answering natural language queries, making data analytics more accessible to everyone.

## What is Fabric Copilot Capacity?

**Fabric Copilot Capacity** refers to the dedicated compute resources required to enable and use Copilot features in Microsoft Fabric. These capacities ensure that Copilot’s AI features are available and performant for users in your organization. Previously, only higher F64+ SKUs were eligible, but with the recent announcement,[F2 SKU is also supported](https://blog.fabric.microsoft.com/en-GB/blog/copilot-and-ai-capabilities-now-accessible-to-all-paid-skus-in-microsoft-fabric/).

## Importance of this announcement

As customers of Microsoft, this allows for a far lower barrier to entry for using AI-powered analytics in Microsoft Fabric. The F2 SKU is the entry-level capacity, and now with Copilot features enabled, your organizations can leverage AI capabilities without needing to invest in higher capacities immediately.

For Partners, this is pivotal in your Fabric conversations. Previously, you had to recommend F64 or higher SKUs for Copilot, which limited the audience as the cost started from $8k+, with Fabric Copilot available at F2, this means customer has Copilot at their fingertips from $200+ a month, less with reserved instance. Now, you can target a broader range of customers, including smaller organizations or those just starting with Fabric.

## Fabric Copilot Capacity Calculation

Sample of Fabric Copilot Capacity calculation:

 Copilot usage is measured by the number of tokens processed. Tokens can be thought of as pieces of words. Approximately 1,000 tokens are about 750 words. Prices are calculated per 1,000 tokens, and input and output tokens are consumed at different rates.

 For example, assume each Copilot request has 2,000 input tokens and 500 output tokens. The price for one Copilot request is calculated as follows: (2,000 × 100 + 500 × 400) / 1,000 = 400.00 CU seconds = 6.67 CU minutes.

Since Copilot is a background job, each Copilot request (~6.67 CU minute job) consumes only one CU minute of each hour of a capacity. For a customer on F64 who has 64 * 24 CU Hours (1,536) in a day, and each Copilot job consumes (6.67 CU mins / 60 mins) = 0.11 CU Hours, customers can run over 13,824 requests before they exhaust the capacity. Once exhausted, all operations will shut down.

```plaintext
Total copilot requests = Total CU hours/ (price per copilot request / 60 minutes)
```

CU consumption is billed in 30-second increments on a use-it-or-lose-it basis. If a capacity sits unused for twelve hours, the CU's for those twelve hours are lost, and cannot be used later.

---

## How to Enable Fabric Copilot Capacity in Your Tenant

Follow these steps to enable Fabric Copilot Capacity:

### 1. Assign Fabric Capacity (F2 or higher)

- Go to the [Microsoft Fabric Admin Portal](https://app.fabric.microsoft.com/admin).
- Under **Capacity settings**, ensure you have an F2 or higher capacity assigned to your workspace.

### 2. Enable Copilot Features in the Admin Portal

- In the Fabric Admin Portal, select **Tenant settings**.
- Find the **Copilot and Azure OpenAI Service (preview)** section.
- Set _Users can use Copilot and other features powered by Azure OpenAI_ to **Enabled**.
- ![alt text](/assets/images/2025-05-01/1.png)
- Optionally, restrict Copilot to specific security groups if needed.

### 2. Enable _Data sent to Azure OpenAI can be processed outside your capacity's geographic region, compliance boundary, or national cloud instance_

- ![alt text](/assets/images/2025-05-01/2.png)

### 3. Enable _Data sent to Azure OpenAI can be stored outside your capacity's geographic region, compliance boundary, or national cloud instance_

### 4. Enable _Capacities can be designated as Fabric Copilot capacities_

- ![alt text](/assets/images/2025-05-01/3.png)
  
### 5. (Optional) Enable _Users can access a standalone, cross-item Power BI Copilot experience (preview)_

 When this setting is turned on, users will be able to access a Copilot experience that allows them to find, analyze, and discuss different Fabric items in a **dedicated** tab available via the Power BI navigation pane.

![alt text](/assets/images/2025-05-01/4.png)

### 6. Assign Fabric Copilot Capacity to the Capacity

- Under **Admin Portal>Capacity Settings**, select the tab for **Fabric Capacity**.
- Find **Copilot Capacity** option and enable to the relevant users or groups
- ![alt text](/assets/images/2025-05-01/5.png)


### 7. Test Copilot in Fabric
- Open a workspace assigned to the eligible capacity.
- Select any of the workloads like Data Engineering, Data Science, or Power BI.
- You should see the Copilot icon enabled, indicating that Copilot features are now available in your workspace.
- Try generating a report or asking a data question using Copilot.


### 5. Test Copilot in Fabric

- Open a workspace assigned to the eligible capacity.
- Look for the Copilot icon or features in Data Engineering, Data Science, or Power BI experiences.
- Try generating a report or asking a data question using Copilot.

---
## Troubleshooting Tips

- If you encounter issues of seeing the option for Copilot Capacity in the Fabric Admin Portal, check the capacity region. It must be similar to your Fabric Home region.

- How to check your Fabric Home region:
  - Click on the help(?) button on the top right corner of the Fabric portal.
  - Select **About Fabric**.
  - Your Fabric Home region will be displayed in the dialog box that appears.
  - ![alt text](/assets/images/2025-05-01/6.png)
- The Fabric Copilot capacity has to reside on at least an F2 or P1 SKU.


**Note:** Changes may take a few minutes to propagate. For more details, refer to the official [Microsoft Fabric documentation](https://learn.microsoft.com/fabric/copilot/enable-copilot).
