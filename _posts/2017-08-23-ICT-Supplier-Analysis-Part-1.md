---
layout: post
title: "Building an App to Analyse Supplier Contracts - Part 1"
date: 2017-08-23
---

*This post is part of a multi-post series that describes the steps to develop an analytics application from scratch.*

The objective of this app is to enable, facilitate and invite analysis of ICT supplier contracts to the Australian Government.

The app should ideally be able to answer questions such as:
- Which supplier has the largest Government footprint? (ie is represented across the highest number of Agencies)
- Which supplier has the largest total / individual contract value?
- Which Agency has the largest total / individual contract?
- How does an Agency's budget/expenses compare with current contracts?
- How much overlap is there between Agencies for similar contracts?
- Are there potential opportunities for consolidation and cost-saving?

To allow these questions to be answered the app will need to present visualisations and tables that allow users to explore and interact with the data.

The app will scrape, store, transform, clean, curate and present data using various technologies and tools across the full tech stack including: 
- Python, Pandas, urllib, BeautifulSoup, Jupyter
- HTML, JSON, Google Cloud Platform (GCP) DataStore
- Flask, GCP AppEngine, Bootstrap, d3

The approach documented below is intended to align with the levels of the Data Value Pyramid as described in Russell Jurney's book: [Agile Data Science 2.0](http://shop.oreilly.com/product/0636920051619.do)

In order for the app to function as expected a number of technologies and tools must be configured to work together.

The steps below describe how to integrate the stack and enable efficient data plumbing between the technologies.

The first step for an analytics app is to ensure we have data. Usable data.

The app we're building will analyse supplier contracts, so we need data on suppliers and contracts. Fortunately the good folks at Australian Government have made available this data through a website: tenders.gov.au.


---
### Data Scrapes and Extracts

&nbsp;

The tenders.gov.au public website lists tenders (business opportunities) for the Australian Government, as well as supplier and contract information that can all be extracted (scraped) using Python libraries such as urllib, Pandas, and json.

&nbsp;

**Dataset 1: Suppliers**


The first dataset we want to extract from tenders.gov.au is a list of ICT Suppliers.

We will be extracting the HTML data and transforming it into the more usable JSON format for our application.

The code in the Jupyter Notebook [scrape_ict_panel_suppliers.ipynb](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jupyter_notebooks/scrape_ict_panel_suppliers.ipynb) extracts HTML data from tenders.gov.au and produces the JSON file as output [ict_panel_suppliers.jsonl](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/datasets/ict_panel_suppliers.jsonl).

The JSON output contains the following elements for each supplier:

- Australian Business Number (ABN),
- Supplier Name, 
- State, and
- Postcode.

&nbsp;

**Dataset 2: Contracts**


The second dataset we want to extract is a list of contracts for each supplier.

To extract the contract information for each supplier we will perform a search against tenders.gov.au using the supplier's ABN. 

The code in the Jupyer Notebook [extract_supplier_contract_data.ipynb](https://github.com/mwportfolio/blob/master/jupyter_notebooks/extract_supplier_contract_data.ipynb) performs this task.

Now that we have extracted our base data we will store it in a NoSQL database from which our web application can read.  


---
### Database Integration

&nbsp;

Google Cloud Platform (GCP) was the platform chosen to host our database and web application for no other reason than the author's free trials had already expired for both AWS and Azure.

GCP's DataStore service provides NoSQL functionality with some limitations. For example the ability to dump nested JSON into DataStore is not available, so some modelling of the data into an appropriate strucutre may need to occur before saving. 

DataStore terms:
- Kind = Collection/Table
- Entity = Record

Since we already have our base data extracts in JSON format we can now save these into DataStore as new Kinds. 

Our JSON supplier list file will be saved into the 'suppliers' Kind, and our list of contracts will be saved into the 'supplier_contracts' Kind.

The code in Jupyter Notebook [store_json_into_nosql.ipynb](https://github.com/mwportfolio/blob/master/jupyter_notebooks/store_json_into_nosql.ipynb) performs this task.

Once data can be saved to DataStore the next step is to verify that it can be read, also shown in store_json_into_nosql.ipynb.

With the data now readable from our database the next step is to present the records to the user through a web interface.


---
### Web Framework & Presentation

&nbsp;

Flask was the web framework chosen to run our app pages. It runs on Python and has Jinja2 templating features which make developing pages a more streamlined process.

GCP's AppEngine allows for Flask apps developed locally to be deployed onto the Google Cloud for serving public users.

The first thing our app needs to do is connect with GCP DataStore, read data, and display it in a simple table.

The flask app is configured per the code in [main.py](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/python/main.py).


&nbsp;

**Routes & Layouts**


The app is configured for a route named "/all_suppliers" which we will configure to list all records from our DataStore Kind "suppliers".

Jinja2 is a templating engine for Python taht allows us to define our HTML template file [layout.html](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jinja2/layout.html) that will include standard layout, header, footer, and styling.

The "/all_suppliers" route uses Flask's render_template function to load the [suppliers.html](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jinja2/suppliers.html) file, which extends our layout.html template and renders a standard HTML table iterating over the records from our DataStore Kind. 

&nbsp;

**Styling**


Adding bootstrap into our layout.html and setting our table class property in suppliers.html adds some nice styling to our rendered table.


---
### Next Post

&nbsp;

That's the end of the first part of the post.

Enhancements to the presentation can be made once the data plumbing and technologies are working in harmony.

Enhancements will include visualisations, integrating additional datasets, and developing interactive analytical features.

The next post will address cleaning, aggregating and curating (mining) the raw data into more digestable formats for exploration through integrated visualisations.

