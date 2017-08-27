---
layout: post
title: "Building an App to Analyse Supplier Contracts - Part 1"
date: 2017-08-28
---

*This post is part of a multi-post series that describes the steps to develop an analytics application from scratch.*

The objective of this app is to enable, facilitate and invite analysis of ICT supplier contracts to the Australian Government.

The app should ideally be able to answer questions such as:

> - Which supplier has the largest Government footprint? (i.e. is represented across the highest number of Agencies)
> - Which supplier has the largest total / individual contract value?
> - Which Agency has the largest total / individual contract?
> - How does an Agency's budget/expenses compare with current contracts?
> - How much overlap is there between Agencies for similar contracts?
> - Are there potential opportunities for consolidation and cost-saving?

To allow these questions to be answered the app will need to present visualisations and tables that allow users to explore and interact with the data.

The app will scrape, store, transform, clean, curate and present data using various technologies and tools across the full tech stack including: 
- Python, Pandas, urllib, Jupyter
- HTML, JSON, Google Cloud Platform (GCP) DataStore
- Flask, GCP AppEngine, Bootstrap, d3

The approach documented below is intended to align with the levels of the Data Value Pyramid as described in Russell Jurney's book: [Agile Data Science 2.0](http://shop.oreilly.com/product/0636920051619.do)

In order for the app to function as expected a number of technologies and tools must be configured to work together.

The steps below describe how to integrate the stack and enable efficient data plumbing between the technologies.

The first step for an analytics app is to ensure we have data; *usable* data.

The app we're building will analyse supplier contracts, so we need data on suppliers and contracts. Fortunately the good folks at Australian Government have made available this data through a website: tenders.gov.au.


---
### Data Scrapes and Extracts

&nbsp;

The tenders.gov.au public website lists tenders (business opportunities) for the Australian Government, as well as supplier and contract information that can all be extracted (scraped) using Python libraries such as urllib, Pandas, and json.

&nbsp;

**Dataset 1: Suppliers**


The first dataset we want to extract from tenders.gov.au is a list of ICT Suppliers.

We will be extracting the HTML data and transforming it into the more usable JSON format for our application.

The code in the Jupyter Notebook extracts HTML data from tenders.gov.au and produces the JSON file as output [ict_panel_suppliers.jsonl](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/datasets/ict_panel_suppliers.jsonl).

&nbsp;

![Jupyter logo](http://jupyter.org/favicon.ico) [scrape_ict_panel_suppliers.ipynb](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jupyter_notebooks/scrape_ict_panel_suppliers.ipynb)

~~~ python
ict_panel_url = 'https://url_goes_here'

# scrape and save to local file
f = urllib.request.urlretrieve(ict_panel_url, "ict_panel_suppliers.html")

# read HTML as DataFrame and set index
t = pd.read_html("ict_panel_suppliers.html", attrs={'class': 'genT'}, header=0)
suppliers_df = t[0]
suppliers_df.index = suppliers_df.ABN

# convert DataFrame to JSON and save
f = open("ict_panel_suppliers.jsonl", "w")
f.write(suppliers_df.to_json(orient='records').replace('},{', '}\n{').replace('[','').replace(']',''))
f.close()
~~~

The JSON output contains the following elements for each supplier:

- Australian Business Number (ABN),
- Supplier Name, 
- State, and
- Postcode.

&nbsp;

**Dataset 2: Contracts**


The second dataset we want to extract is a list of contracts for each supplier.

To extract the contract information for each supplier we will perform a search against tenders.gov.au using the supplier's ABN. 

An important function from the Jupyter Notebook is listed below which is called for each supplier, passing in their ABN number and performing a scrape of their contracts from tenders.gov.au.

&nbsp;

![Jupyter logo](http://jupyter.org/favicon.ico) [extract_supplier_contract_data.ipynb](https://github.com/mwportfolio/blob/master/jupyter_notebooks/extract_supplier_contract_data.ipynb)

~~~ python
contracts = []
def getContracts(ABN):
    current_url = 'https://urlgoeshere' + ABN + 'restofurl'
    current_ABN_contracts_df = (
      pd.read_csv(current_url,
      skiprows=16,
      delimiter='\t',
      parse_dates=['Contract Start Date', 'Contract End Date', 'Publish Date']
      ))            
      
    if current_ABN_contracts_df.iloc[0].str.contains('no results', case=False, na=False)['CN ID'] == False:
        contracts.append(current_ABN_contracts_df)
        return True
     else:
        return None
~~~

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

For each line in the suppliers JSON file, create a DataStore Entity with the appropriate properties, then save/put the Entity into the Kind.

&nbsp;

![Jupyter logo](http://jupyter.org/favicon.ico) [store_json_into_nosql.ipynb](https://github.com/mwportfolio/blob/master/jupyter_notebooks/store_json_into_nosql.ipynb)

~~~ python
from google.cloud import datastore    
import jsonlines, json

client = datastore.Client()
collection = 'suppliers'
f = jsonlines.open('ict_panel_suppliers.jsonl', 'r')
for r in f.iter():
    key = client.key(collection, r['ABN'])
    supplier = datastore.Entity(key=key)
    supplier['Name'] = r['Supplier Name']
    supplier['ABN'] = r['ABN']
    supplier['State'] = r['State']
    supplier['Postcode'] = r['Postcode']
    client.put(supplier)
    print('Saved {}: {}'.format(supplier.key.name, supplier['Name']))
    
f.close()
~~~

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


One of the URL routes/stubs (/contracts/supplier/ABN) has been configured to list the contracts for a single supplier. The supplier is identified in the URL by the ABN number parameter after the /contracts/supplier/ string. 

Requests to this route perform a query against the DataStore Kind 'supplier_contracts' and render the results through the [contracts.html](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jinja2/contracts.html) template.

The "/contracts/supplier/ABN" route uses Flask's render_template function to load the [contracts.html](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jinja2/contracts.html) file, which extends our [layout.html](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jinja2/layout.html) template and renders a standard HTML table iterating over the records passed from main.py. 

Jinja2 is a templating engine for Python that allows us to define our HTML template file [layout.html](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/jinja2/layout.html) that will include standard layout, header, footer, and styling.

&nbsp;

![Python logo](https://www.python.org/static/favicon.ico) [main.py](https://github.com/mwportfolio/ICT-Supplier-Analysis/blob/master/python/main.py)

~~~ python
@app.route("/contracts/supplier/<ABN>")
def supplier_contracts(ABN):

    client = datastore.Client()
    collection = 'supplier_contracts'
    query = client.query(kind=collection)
    query.add_filter('ABN', '=', ABN)
    query.order = ['Title']

    results = list(query.fetch())
    contract_count = len(results)


    contracts = []
    for r in results:
        s = datastore.Entity(r)
        contract = {}
        contract['ContractID'] = s.key.key.name
        contract['Title'] = s.key.get('Title')
        contract['Agency'] = s.key.get('Agency')
        contract['Category'] = s.key.get('Category')
        contract['Value'] = s.key.get('Value')
        contracts.append(contract)

    query_suppliername = client.query(kind='suppliers')
    query_suppliername.add_filter('ABN', '=', ABN)
    results_suppliername = list(query_suppliername.fetch(limit=1))
    supplier_name = datastore.Entity(results_suppliername[0]).key.get('Name')


    return render_template('contracts.html',
        contracts=contracts,
        ABN=ABN,
        supplier_name=supplier_name,
        contract_count=contract_count)
~~~

&nbsp;

**Styling**


Adding bootstrap into our layout.html and setting our table class property in suppliers.html adds some nice styling to our rendered table. 

The D3 horizontal bar chart shows the top 5 total contract value across Agencies per supplier.

&nbsp;

![Screenshot contracts for supplier](https://raw.githubusercontent.com/mwportfolio/ICT-Supplier-Analysis/master/screenshots/screenshot_app_contracts_for_supplier.PNG)


---
### Next Post

&nbsp;

That's the end of the first part of the post.

Enhancements to the presentation can be made once the data plumbing and technologies are working in harmony.

Enhancements will include visualisations, integrating additional datasets, and developing interactive analytical features.

The next post will address cleaning, aggregating and curating (mining) the raw data into more digestable formats for exploration through integrated visualisations.

