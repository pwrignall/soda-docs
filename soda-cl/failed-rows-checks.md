---
layout: default
title: Failed rows checks
description: Use a SodaCL (Beta) failed rows check to explicitly send sample failed rows to Soda Cloud. 
parent: SodaCL (Beta)
---

# Failed rows checks ![beta](/assets/images/beta.png){:height="50px" width="50px" align="top"}

Use a failed rows check to explicitly send samples of rows that failed a check to Soda Cloud.

```yaml
checks for dim_customer:
# Failed rows defined using common table expression
  - failed rows:
      fail condition: total_children = '2' and number_cars_owned >= 3
# Failed rows defined using SQL query
  - failed rows:
      fail query: |
        SELECT DISTINCT geography_key
        FROM dim_customer as customer
```

[Prerequisites](#prerequisites) <br />
[About failed row samples](#about-failed-row-samples) <br />
[Define failed rows checks](#define-failed-rows-checks) <br />
[Optional check configurations](#optional-check-configurations)<br />
[Go further](#go-further)<br />
<br />

## Prerequisites

To use failed row checks, you must have a **Soda Cloud** account connected to Soda Core. See <a href="https://docs.soda.io/soda-core/configure.html#connect-soda-core-to-soda-cloud" target="_blank">Connect Soda Core to Soda Cloud</a> for details. 


## About failed row samples

When a scan results in a failed check, the CLI output displays information about the check that failed and why. To offer more insight into the data that failed a check, Soda Cloud displays failed rows in a check result’s history.

There are two ways you can configure a SodaCL check to send failed row samples to your Soda Cloud account:

1. Use a [`duplicate_count` metric]({% link soda-cl/numeric-metrics.md %}#send-failed-rows-to-soda-cloud), a [missing metric]({% link soda-cl/missing-metrics.md %}#send-failed-rows-to-soda-cloud), or a [validity metric]({% link soda-cl/validity-metrics.md %}#send-failed-rows-to-soda-cloud) in your check. Checks that use these metrics automatically send failed row samples to your Soda Cloud account.
2. Use failed rows checks in your to explicitly send failed rows to Soda Cloud. Read on!

For security, you can also disable the failed row samples feature entirely; see [Disable failed row samples]({% link soda-cloud/failed-rows.md %}#disable-failed-row-samples) for details.


## Define failed rows checks

In the context of [SodaCL check types]({% link soda-cl/metrics-and-checks.md %}check-types), failed row checks are user-defined. This check is limited in its syntax variation, but you can customize your expression or query as much as you like.

The example below uses <a href="https://www.essentialsql.com/introduction-common-table-expressions-ctes/" target="_blank">common table expression (CTE)</a> to define the `fail condition` that any rows in the `dim_customer` dataset must meet in order to qualify as failed rows, during a scan, get sent to Soda Cloud. 

In this rather silly example, Soda Core sends any rows which contain the value 2 in the `total_children` column and which contain a value greater than or equal to 3 in the `number_cars_owned` column to Soda Cloud as failed row samples. The check also uses the `name` key to customize a name for the check so that it displays in a more readable form in Soda Cloud; see image below.

```yaml
checks for dim_customer:
  - failed rows:
      name: Failed rows with CTE
      fail condition: total_children = '2' and number_cars_owned >= 3
```

![failed-rows-CTE](/assets/images/failed-rows-CTE.png){:height="700px" width="700px"}

<br />

If you prefer, you can use a SQL query to define what qualifies as a failed row for Soda Core to send to Soda Cloud, as in the following simple example.

```yaml
checks for dim_customer:
  - failed rows:
      name: Failed rows query test
      fail query: |
        SELECT DISTINCT geography_key
        FROM dim_customer as customer
```
![failed-rows-SQL](/assets/images/failed-rows-SQL.png){:height="700px" width="700px"}

<br />

## Optional check configurations

| ✓ | Configuration | Documentation |
| :-: | ------------|---------------|
| ✓ | Define a name for a failed rows check; see [example](#example-with-check-name). |  [Customize check names]({% link soda-cl/optional-config.md %}#customize-check-names) |
|   | Define alert configurations to specify warn and fail alert conditions. | - |
|   | Apply a filter to return results for a specific portion of the data in your dataset.| - | 
| ✓ | Use quotes when identifying dataset or column names; see [example](#example-with-quotes) | [Use quotes in a check]({% link soda-cl/optional-config.md %}#use-quotes-in-a-check) |
| ✓  | Use wildcard characters in the value in the check. | Use wildcard values as you would with CTE or SQL. |
|   | Use for each to apply schema checks to multiple datasets in one scan. | - |
| ✓ | Apply a dataset filter to partition data during a scan; see [example](#example-with-dataset-filter). | [Scan a portion of your dataset]({% link soda-cl/optional-config.md %}#scan-a-portion-of-your-dataset) |

#### Example with check name 

```yaml
checks for dim_customer:
  - failed rows:
      name: Failed rows query test
      fail query: |
        SELECT DISTINCT geography_key
        FROM dim_customer as customer
```

#### Example with quotes

```yaml
checks for dim_customer:
  - failed rows:
      name: Failed rows query test
      fail query: |
        SELECT DISTINCT "geography_key"
        FROM dim_customer as customer
```

#### Example with dataset filter

```yaml
coming soon
```

<br />

## Go further

* Learn more about [SodaCL metrics and checks]({% link soda-cl/metrics-and-checks.md %}) in general.
* Use a [schema check]({% link soda-cl/schema.md %}) to discover missing or forbidden columns in a dataset.
* Need help? Join the <a href="http://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
<br />


---
{% include docs-footer.md %}