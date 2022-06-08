---
layout: default
title: Quick start for SodaCL
description: Follow the quick start tutorial to get started with SodaCL, a human-readable, domain-specific language for data reliability. 
parent: Soda
---

# Quick start for SodaCL ![beta](/assets/images/beta.png){:height="50px" width="50px" align="top"}

If you are staring at a blank YAML file wondering what SodaCL checks to write to surface data quality issues, this quick start tutorial is for you. 

![blank-page](/assets/images/blank-page.png){:height="600px" width="600px"}

[Soda Core and SodaCL: In brief](#soda-core-and-sodacl-in-brief) <br />
[About this tutorial](#about-this-tutorial) <br />
[Tutorial prerequisites](#tutorial-prerequisites) <br />
[Row count and cross checks](#row-count-and-cross-checks) <br />
[Freshness check](#freshness-check)<br />
[Missing and invalid checks](#missing-and-invalid-checks)<br />
[Reference checks](#reference-checks)<br />
[Go further](#go-further)<br />
<br />


## Soda Core and SodaCL: In brief

**Soda Checks Language (SodaCL)** is a YAML-based, domain-specific language for data reliability. Used in conjunction with Soda Core (Beta), Soda’s open-source, command-line tool, you use SodaCL to write checks for data quality, then use Soda Core to scan the data in your data source and execute those checks.

After installing Soda Core, you connect Soda Core to your data source (Snowflake, BigQuery, etc.) by defining connection details such as host, username, and password, in a **configuration YAML** file. Then, you define your Soda Checks for data quality in a **checks YAML** file. A Soda Check is a test that Soda Core performs when it scans a dataset in your data source. When you use Soda Core to run a scan on data in your data source, you reference both the configuration and checks YAML files in the scan command.

A **Soda scan** executes the checks you defined in the checks YAML file and returns a result for each check: pass, fail, or error. (Optionally, you can configure a check to warn instead of fail by setting an alert configuration.)


## About this tutorial

With over 25 built-in checks and metrics to choose from, it can be hard to know where to begin. This tutorial offers suggestions for some basic checks you can write to begin surfacing missing, invalid, unexpected data in your datasets. 

All the example checks in this tutorial use placeholder values for dataset and column name identifiers, but you can copy+paste the examples into your own checks YAML file and adjust the details to correspond to your own data.

You do not need to follow the tutorial sequentially. 


## Tutorial prerequisites

* You have completed the [Quick start for Soda Core and Soda Cloud]({% link soda/quick-start-soda-core.md %}) <br /> OR <br /> you have followed the instructions to <a href="https://docs.soda.io/soda-core/get-started.html" target="_blank">install</a> and <a href="https://docs.soda.io/soda-core/first-scan.html" target="_blank">configure</a> Soda Core on your own. 
* You have installed a code editor such as Sublime or Visual Studio Code.
* You have created a new YAML file in your code editor and named it `checks.yml`.
* (Optional) You have read the first two sections in [Metrics and checks]({% link soda-cl/metrics-and-checks.md %}) as a primer for SodaCL.


## Row count and cross checks

One of the most basic checks you can write uses the `row_count` metric. When it executes the following check during a scan, Soda simply counts the rows in the dataset you identify in the `checks for` section header to confirm that the dataset is not empty. If it counts one or more rows, the check result is pass.

```yaml
# Check that a dataset contains rows
checks for dataset_name:
  - row_count > 0
```

<br />

If you wish, you can specify a column against which Soda executes the check. In the following example, Soda counts the rows in the `last_name` column, identified as the value in parentheses. If there are fewer than 50 rows, the check result is fail.

```yaml
# Check that a column contains more than 50 rows
checks for dim_customer:
  - row_count(last_name) > 50
```

<br />

The two checks above are examples that use a numeric metric in a standard check pattern. By contrast, the following unique cross check compares row counts between datasets within the same data source without setting a threshold for volume, like `> 50`. 

This type of check is useful when, for example, you want to compare row counts to validate that a transformed dataset contains the same volume of data as the source from which it came.

```yaml
# Compare row counts between datasets
checks for dataset_name:
  - row_count same as other_dataset_name
```

### Read more
* [Numeric metrics]({% link soda-cl/numeric-metrics.md %})
* [Standard check pattern]({% link soda-cl/metrics-and-checks.md %}#check-types)
* [Cross checks]({% link soda-cl/cross-row-checks.md %})


## Freshness check

If your dataset contains a column that stores timestamp information, you can configure a freshness check. This type of check is useful when, for example, you need to validate that the data feeding a weekly report or dashboard is not stale. Timely data is reliable data!

In this example, the check fails if the most-recently added row (in other words, the "youngest" row) in the `timestamp_column_name` column is more than 24 hours old.

```yaml
# Check that data in dataset is less than one day old
checks for dataset_name:
  - freshness(timestap_column_name) < 1d
```

### Read more
* [Freshness checks]({% link soda-cl/freshness.md %})


## Missing and invalid checks

SodaCL's missing metrics make it easy to find null values in a column. You don't even have to specify that `NULL` qualifies as a missing value because SodaCL registers null values as missing by default. The following check passes if there are no null values in `column_name`, identified as the value in parentheses.

```yaml
# Check that there are no null values in a column
checks for dataset_name:
  - missing_count(column_name) = 0
```

<br />

If the type of data a dataset contains is TEXT (string, character varying, etc.), you can use an invalid metric to surface any rows that contain ill-formatted data. This type of check is useful when, for example, you need to validate that all values in an email address column are formatted as `name@domain.extension`.

The following example fails if, during a scan, Soda discovers that more than 5% of the values in the `email_column_name` do not follow the email address format.

```yaml
# Check an email column that all values are in email format
checks for dataset_name:
  - invalid_percent(email_column_name) > 5:
      valid format: email
```

<br />

If you want to surface more than just null values as missing, you can specify a list of values that, in the context of your business rules, qualify as missing. In the example check below, Soda registers `N/A`, `0000`, or `none` as missing values in addition to `NULL`; if it discovers more than 5% of the rows contain one of these values, the check fails. 

Note that the missing value `0000` is wrapped in single quotes; all numeric values you include in such a list must be wrapped in single quotes.

```yaml
# Check that fewer than 5% of values in column contain missing values
checks for dataset_name:
  - missing_percent(column_name) < 5%:
      missing values: [N/A, '0000', none]
```

### Read more
* [Missing metrics]({% link soda-cl/missing-metrics.md %})
* [Validity metrics]({% link soda-cl/validity-metrics.md %})


## Reference checks

If you need to validate that two datasets contain the same values, you can use a reference check. The following unique check compares the values of `column_name` and `another_column`, identified as the values in parentheses, between datasets within the same data source. The check passes if the values in the columns are *exactly* the same. 

```yaml
# Check that values in a column exist in another column in a different dataset
checks for dataset_name:
  - values in (column_name) must exist in different_dataset_name (another_column)
```

<br />

If you wish, you can compare the values of multiple columns in one check. Soda compares the column names respectively, so that in the following example, `column_name1` compares to `other_column1`, and `column_name2` compares to `other_column2`.

```yaml
# Check that values in two columns exist in two other columns in a different dataset
checks for dataset_name:
  - values in (column_name1, column_name2) must exist in different_dataset_name (other_column1, other_column2)
```

### Read more
* [Reference checks]({% link soda-cl/reference.md %})


## Schema checks

To eliminate the frustration of the silently evolving dataset schema, use schema checks with alert configurations to notify you when column changes occur.

If you have set up a Soda Cloud account and [connected it to Soda Core]({% link soda/quick-start-soda-core.md %}#connect-soda-core-to-soda-cloud), you can use a catch-all schema check that results in a warning whenever a Soda scan reveals that a column has been added, removed, moved within the context of an index, or changed data type relative to the results of the previous scan. 

```yaml
# Requires a Soda Cloud account
# Check for any schema changes to dataset
checks for dataset_name:
  - schema:
      warn: 
        when schema changes: any
```

<br />

If you wish to apply a more granular approach to monitoring schema evolution, you can specify columns in a dataset that ought to be present or which should not exist in the dataset. 

The following example warns you when, during a scan, Soda discovers that `column_name` is missing in the dataset; the check fails if either `column_name1` or `column_name2` exist in the dataset. This type of check is useful when, for example, you need to ensure that datasets do not contain columns of sensitive data such as credit card numbers or personally identifiable information (PII).

```yaml
# Check for absent or forbidden columns in dataset
checks for dataset_name:
  - schema:
      warn:
        when required column missing: [column_name]
      fail:
        when forbidden column present: [column_name1, column_name2]
```

Be aware that a check that contains one or more alert configurations only ever yields a *single* check result; one check yields one check result. If your check triggers both a warn and a fail, the check result only displays the more severe, failed check result.

### Read more
* [Alert configuration]({% link soda-cl/optional-config.md %}#add-alert-configurations)
* [Schema checks]({% link soda-cl/schema.md %})
* [Expect one check result]({% link soda-cl/optional-config.md %}#expect-one-check-result)


## Go further

* Learn more about [SodaCL metrics and checks]({% link soda-cl/metrics-and-checks.md %}) in general.
* Read about the [Optional configurations]({% link soda-cl/optional-config.md %}) you can apply to SodaCL checks.
* [Connect Soda Core to Soda Cloud]({% link soda/quick-start-soda-core.md %}#connect-soda-core-to-soda-cloud) to vastly enrich the data quality monitoring experience.
* Need help? Join the <a href="http://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
<br />


---
{% include docs-footer.md %}