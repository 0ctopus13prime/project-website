---
layout: post
title: "Unlocking Agentic AI experiences with OpenSearch"
authors:
  - arjunkumargiri
  - jiapingzeng
date: 2025-06-09
categories:
  - technical-posts
meta_keywords: MCP, Agentic, Agent, Model context protocol, root cause analysis, MCP client, OpenTelemetry, LLM integration, observability agent
meta_description: Learn how to integrate Amazon Q Developer CLI with OpenSearch's agentic AI tools via Model Context Protocol (MCP) to extract analytics and insights using natural language queries. This guide demonstrates setup, configuration and usage examples for developers and data engineers.
---

## Overview

As search evolves from text-based inputs to conversational, interactive experiences, the power of Agentic AI is unlocking new ways to connect with systems, applications, and large language models (LLMs). Recently, we released a [Model Context Protocol](https://opensearch.org/blog/introducing-mcp-in-opensearch/) (MCP) server for OpenSearch as an open source solution. In this blog post, we’ll illustrate how you can integrate the [Amazon Q Developer](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line.html)command line interface (CLI) with OpenSearch’s agentic AI tools via MCP. With a step-by-step approach, we’ll demonstrate you how you can extract impactful analytics and insights into your applications using natural language.

Whether you're a developer, data engineer, or solutions architect, this guide will walk you through the transformative potential of AI-powered capabilities in OpenSearch, helping you deliver more intelligent and efficient search experiences.

## Prerequisites

* [Amazon Q Developer CLI](https://github.com/aws/amazon-q-developer-cli)
* Python 3.10+
* OpenSearch cluster, either self-managed or Amazon OpenSearch Service.

## Setup

### 1. Install Required Tools

First, install the `uv` package manager, which is a required tool for running the OpenSearch MCP server:

```
pip install uv
```

### 2. Configure Q Developer CLI

Create or edit the MCP configuration file for Amazon Q Developer CLI at `~/.aws/amazonq/mcp.json` . To connect Q dev CLI with Amazon OpenSearch Service, provide the cluster endpoint and AWS credentials with permissions to access the cluster.

```json
{
  "mcpServers": {
    "opensearch-mcp-server": {
      "command": "uvx",
      "args": [
        "opensearch-mcp-server-py"
      ],
      "env": {
        "OPENSEARCH_URL": "<your_opensearch_domain_url>",

        // IAM Authentication
        "AWS_REGION": "<your_aws_region>",
        "AWS_ACCESS_KEY_ID": "<your_aws_access_key>",
        "AWS_SECRET_ACCESS_KEY": "<your_aws_secret_access_key>",
        "AWS_SESSION_TOKEN": "<your_aws_session_token>"
      }
    }
  }
}
```

Alternatively, you can set up the OpenSearch URL and IAM authentication credentials directly as environment variables in your terminal session instead of using the mcp.json configuration file.

### 3. Using Q Developer CLI

Start a chat session with Q Developer CLI via `q chat`. We should see that it has successfully loaded the `opensearch_mcp_server`:

```bash
> q  
✓ opensearch_mcp_server loaded in 0.36 s
✓ 1 of 1 mcp servers initialized.
```

Using the `/tools` command within Q, you can verify that four OpenSearch tools are loaded: `GetShardsTool`, `IndexMappingTool`, `ListIndexTool`, and `SearchIndexTool`. Additional OpenSearch tools will be added in future releases to expand the capabilities and functionalities of the system.

Note that by default, new tools added are not trusted, meaning that Q will ask for confirmation any time it tries to use the tool. Optionally, we can trust the tools via `/tools trust <full_tool_name>` or `/tools trustall` to skip the confirmations.

```bash
> /tools

𝗧𝗼𝗼𝗹 𝗣𝗲𝗿𝗺𝗶𝘀𝘀𝗶𝗼𝗻
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
𝗕𝘂𝗶𝗹𝘁-𝗶𝗻:
- execute_bash * trust read-only commands
- fs_read  * 𝘁𝗿𝘂𝘀𝘁𝗲𝗱
- fs_write * not trusted
- report_issue * 𝘁𝗿𝘂𝘀𝘁𝗲𝗱
- use_aws  * trust read-only commands

𝗼𝗽𝗲𝗻𝘀𝗲𝗮𝗿𝗰𝗵_𝗺𝗰𝗽_𝘀𝗲𝗿𝘃𝗲𝗿 (𝗠𝗖𝗣):
- opensearch_mcp_server___GetShardsTool  * not trusted
- opensearch_mcp_server___IndexMappingTool * not trusted
- opensearch_mcp_server___ListIndexTool  * not trusted
- opensearch_mcp_server___SearchIndexTool  * not trusted
```

## Using OpenSearch with Q CLI

Now that we have configured Q CLI with the OpenSearch MCP server, we can start querying OpenSearch data:

```bash
> list indices
I'll help you list the indices in OpenSearch. Let me retrieve that information 
for you.

🛠️ Using tool: ListIndexTool from mcp server opensearch_mcp_server
 ⋮ 
 ● Running ListIndexTool with the param:
 ⋮ {
 ⋮ "arguments": {},
 ⋮ "name": "ListIndexTool"
 ⋮ }
Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:

> y
 ⋮ 
 ● Completed in 1.8s
Here are the indices in your OpenSearch cluster:
1. .plugins-ml-model-group
2. tmdb
3. .plugins-ml-memory-meta
4. .ql-datasources
5. .plugins-ml-task
...
```

With the ability to list and search indices, we can now use Q Developer CLI for more advanced use cases, such as data summary:

```bash
> give me a quick summary of my eCommerce data
I'll provide a summary of your eCommerce data. Let me query the OpenSearch eCommerce sample data 
index to get that information.

🛠️ Using tool: SearchIndexTool from mcp server opensearch_mcp_server
...

# eCommerce Data Summary
Here's a quick summary of your eCommerce sample data:

## Order Statistics
• Total orders: 4,675
• Average order price: $75.06
• Minimum order price: $6.99
• Maximum order price: $2,250.00

## Top Categories
1. Men's Clothing (2,024 orders)
2. Women's Clothing (1,903 orders)
3. Women's Shoes (1,136 orders)
4. Men's Shoes (944 orders)
5. Women's Accessories (830 orders)

## Top Manufacturers
1. Low Tide Media (1,553 orders)
2. Elitelligence (1,370 orders)
3. Oceanavigations (1,218 orders)
4. Tigress Enterprises (1,055 orders)
5. Pyramidustries (947 orders)

## Customer Demographics
• Female customers: 2,433 orders (52%)
• Male customers: 2,242 orders (48%)

This data appears to be a sample dataset with a focus on clothing and accessories, with a fairly 
even gender distribution among customers.
```

OpenSearch observability customers can make use of Q dev CLI to perform detailed investigations of their system performance, as shown in this example:

```bash
> check my otel traces, show me latency by service name

I'll help you analyze the OpenTelemetry (OTel) traces in your system to show latency by service 
name. Let me query the relevant index.

🛠️ Using tool: SearchIndexTool from mcp server opensearch_mcp_server
 ⋮ 
 ● Running SearchIndexTool with the param:
...

Based on your OpenTelemetry traces, here's a breakdown of latency by service name:

## Service Latency Summary (in nanoseconds)
| Service | Requests | Min Latency | Avg Latency | Max Latency | Avg (ms) |
|---------|----------|------------|-------------|-------------|----------|
| frontend | 4,486 | 5,376 | 4,212,749 | 297,229,056 | 4.21 ms |
| cartservice | 1,151 | 573,500 | 908,025 | 5,473,600 | 0.91 ms |
| productcatalogservice | 976 | 5,025 | 1,179,214 | 9,671,364 | 1.18 ms |
| loadgenerator | 792 | 4,075,763 | 12,225,651 | 298,967,030 | 12.23 ms |
| checkoutservice | 662 | 258,431 | 6,623,067 | 288,126,051 | 6.62 ms |
| paymentservice | 403 | 8,448 | 108,115 | 4,312,064 | 0.11 ms |
| featureflagservice | 385 | 468,824 | 786,792 | 4,978,760 | 0.79 ms |
| recommendationservice | 271 | 880,730 | 8,177,191 | 13,767,667 | 8.18 ms |
| emailservice | 196 | 63,342 | 3,739,964 | 189,894,340 | 3.74 ms |
| adservice | 170 | 36,500 | 3,126,550 | 141,348,925 | 3.13 ms |

## Key Observations:
1. Highest Traffic Services:
 • frontend (4,486 requests)
 • cartservice (1,151 requests)
 • productcatalogservice (976 requests)

2. Highest Average Latency:
 • loadgenerator (12.23 ms)
 • recommendationservice (8.18 ms)
 • checkoutservice (6.62 ms)

3. Highest Maximum Latency (Potential Outliers):
 • loadgenerator (298.97 ms)
 • frontend (297.23 ms)
 • checkoutservice (288.13 ms)

4. Most Efficient Services:
 • paymentservice (0.11 ms avg)
 • featureflagservice (0.79 ms avg)
 • cartservice (0.91 ms avg)

The data shows that while the frontend handles the most requests, the loadgenerator, 
recommendationservice, and checkoutservice have the highest average latencies. The paymentservice 
appears to be the most efficient with the lowest average latency.
```

After getting the initial analysis of OpenTelemetry traces and latency data, we can ask follow-up questions for deeper investigation, as demonstrated in the following example where we investigate why the loadgenerator has high average latencies.

```bash
> why does loadgenerator have high average latencies?

To understand why the loadgenerator has high average latencies, I'll need to examine more details 
about its traces. Let me query for more specific information about the loadgenerator service.

🛠️ Using tool: SearchIndexTool from mcp server opensearch_mcp_server
 ⋮ 
 ● Running SearchIndexTool with the param:
...
...

After analyzing the OpenTelemetry trace data in detail, I can explain why the loadgenerator serviceshows significantly higher latencies compared to other services in your microservices 
architecture.

**## Root Cause of High Loadgenerator Latencies**

The high latencies in the loadgenerator service are **by design** and represent the complete end-to-end 
request time, including all downstream service processing. Here's what's happening:

1. Client-Side Measurement: The loadgenerator is using Python's requests library with OpenTelemetry 
instrumentation to make HTTP calls. The spans from loadgenerator are **CLIENT spans** that measure the 
entire request lifecycle from start to finish.

2. End-to-End Timing: Looking at trace 753ef52011e443ed190dbf6f31c30cad, we can see:
 • The loadgenerator span starts at 2023-11-23T00:16:14.063978483Z
 • It ends at 2023-11-23T00:16:14.362945513Z
 • Total duration: 298.97ms

3. Nested Service Calls: The frontend service receives this request and creates its own span that 
lasts for 297.23ms, nearly the entire duration of the loadgenerator's span.

4. Cascading Service Calls: Following the trace through the system:
 • Frontend makes a gRPC call to CheckoutService (291.63ms)
 • CheckoutService processes the order (288.13ms)
 • CheckoutService makes an HTTP POST to EmailService (193.04ms)
 • And so on through multiple services

## Confirmation from Trace Analysis

When examining the complete trace 6f0611913ca16432c5952b2c76bd3f1f, we can see:
• The frontend initiates a synthetic request (linked to the loadgenerator's original request)
• This cascades through multiple services (checkout, email, etc.)
• Each service adds its own processing time
• The loadgenerator's span encompasses all of this processing time

## Why This Is Normal

1. Load Testing Tool: The loadgenerator is designed to simulate user traffic and measure complete 
response times. Its high latencies are actually a feature, not a bug.

2. Instrumentation Difference: While other services report internal processing times, the 
loadgenerator reports the complete client experience time.

3. Network Overhead: The loadgenerator spans include network latency between all services, which 
adds up significantly.

## Recommendations

1. This is expected behavior: The loadgenerator's high latencies are normal and reflect its role in 
the system.

2. Use for Performance Testing: These measurements are valuable for end-to-end performance testing.

3. Separate Metrics: When evaluating service performance, consider:
 • Internal service latencies (from each service's SERVER spans)
 • End-to-end latencies (from loadgenerator's CLIENT spans)

4. Optimization Focus: If you want to improve overall system performance, focus on the services with
the highest internal processing times (checkout, recommendation, and email services) rather than 
the loadgenerator itself.

The loadgenerator is correctly measuring the complete user experience time, which naturally 
includes all downstream service processing and network overhead.
```

## Conclusion

OpenSearch’s Agentic AI capabilities demonstrates real-world value through powerful capabilities for analytics, investigation, summarization, and reporting. The integration demonstrated here not only enhances traditional search capabilities but also provides deeper insights into system performance and data analysis, offering a valuable tool for developers and data engineers. If you’d like to learn more, you can find more details in our recent [blog post](https://opensearch.org/blog/introducing-mcp-in-opensearch/) on MCP, and we welcome conversation and contributions on the [OpenSearch MCP repo](https://github.com/opensearch-project/opensearch-mcp-server-py/tree/main).
