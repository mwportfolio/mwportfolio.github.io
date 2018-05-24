---
layout: post
title: "Encouraging Data Literacy - Data Cleaning"
date: 2018-05-23
---

*This poat is designed to improve data literacy by encouraging good data cleaning habits.*

We will walk through a data cleansing process using data scraped from a public website.

### Why is data cleaning important?

Clean datasets also allow our data processing/visualisation tools to function properly with improved performance and reduced errors.

So if you want to perform analysis on a dataset you will find it much easier to work with a clean dataset than with one found in the "wild".
 
The problem is that most of the datasets found on public websites or even in internal organisations' databases consist of dirty data.

Some examples of dirty data might include:

- date/time values saved as text format
- numeric values saved as text format
- missing values
- empty columns

So before we can analyse or visualise our data properly we first need to resolve these issues.

Let's start by having a look at a public dataset found on [GrantsConnect](http://grants.gov.au).

This website has a page which provides a handy list of downloadable Excel files per time period - thanks GrantsConnect!

We will be using Pandas library to access/manipulate the dataset, so let's pick one of the downloads from the list, copy it's URL, and open it in Pandas.


![Python logo](https://www.python.org/static/favicon.ico)
~~~python
import pandas as pd

url = 'http://grants.gov.au/xxxxx77w3485w8tsgDownloadURLgoeshere'

test = pd.read_excel(url)

test.head()
~~~

The above commands attempt to read the Excel file located at the address in "url" variable, which we copied from the GrantsConnect website, then display the first 5 rows of the file.

&nbsp;

![Raw Dataset](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawData.png)

&nbsp;

We can see that something does not look exactly right - for example, the column headers are labelled "Unnamed" instead of having meaningful names.

Looking at values of the rows it seems like the column headers are not at the top of the file. In fact, they are on the 3rd row.

The first 2 rows of this file don't seem to contain anything valuable so we can tell Pandas to skip these rows and re-load the file.

![Python logo](https://www.python.org/static/favicon.ico)
~~~python
test2 = pd.read_excel(url, skiprows=2)

test2.head()
~~~

&nbsp;

![Raw dataset with headers](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataWithHeader.png)

&nbsp;

Now we can see that our column headers seem to be in the right place. Great!

The dataset is looking cleaner already, but let's keep exploring because there may be other areas of the file which also need cleaning.

We can see some date/time columns such as "Approved Date" and "Start Date", and it's important that these columns are identified as date/times by Pandas so that it can handle them correctly.

Let's have a look. The `test2.dtypes` command lists the columns and datatypes from our DataFrame.

&nbsp;

![Raw dataset without datatypes](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataWithoutDatatypes.png)

&nbsp;

Interestingly, the "Approval Date" column is set to an "object" datatype instead of a date/time datatype.

In fact all of the date/time columns are all set to the "object" datatype, which for the sake of simplicity let's assume "object" means "text".

We've found the second issue with our dirty data file which needs to be cleaned: date/time columns, so let's re-load the data file and specify which columns should be parsed as dates.


![Python logo](https://www.python.org/static/favicon.ico)
~~~python
test3 = pd.read_excel(url, skiprows=2, parse_dates=["Approval Date", "Variation Date", "Published Date/Time", "Start Date", "End Date"])

test3.dtypes
~~~

Great, now our date/time columns have changed from being "object" datatype to "datetime" datatype!

&nbsp;

![Raw dataset with datatypes](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataWithDatatypes.png)

&nbsp;

Having these columns correctly datatyped will ensure we can perform calculations on the date/time columns correctly.

Is there anything else left in our dataset to clean?

You may have noticed the "Unnamed: 39" column when we previously ran the test3.dtypes command.

&nbsp;

![Raw dataset empty column](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataEmptyColumn.png)

&nbsp;

It's odd that all of the columns have meaningful names except this one. Perhaps the column is not going to be valuable for our analysis.

Before we ignore it, let's have a quick look at the values in the column by running the command `test3.loc[:, "Unnamed: 39"]`

&nbsp;

![Raw dataset empty column values](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataEmptyColumnValues.png)

&nbsp;

Okay, the column seems like it's empty, or at least not contain any valuable data, so let's delete (drop) it from our dataset.

![Python logo](https://www.python.org/static/favicon.ico)
~~~python
test4 = test3.drop("Unnamed: 39", axis=1)

test4.dtypes
~~~

The above command drops/deletes the column from our DataFrame, so now we have relatively clean dataset to perform our analysis from our original dirty data file scraped from the web!

&nbsp;

![Raw dataset empty column dropped](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataEmptyColumnDropped.png)

&nbsp;

The DataFrame is ready to perform analysis to answer questions such as:

- Which Government Departments/Agencies are providing the most funding?

`test4.loc[:, ["Agency", "Value (AUD)"]].groupby("Agency").sum().sort_values(by="Value (AUD)", ascending=False)`

&nbsp;

![Raw dataset top agencies](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/GrantConnectPublicRawDataTopAgencies.png)

&nbsp;

By starting your analysis with a clean dataset you can minimise your errors and troubleshooting time, and it will also improve your data literacy over time! 
