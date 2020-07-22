XForm is an easy to use big data exploration tool.

- Explore millions of rows on your laptop with as-you-type feedback in a beautiful web interface.
- Publish the same queries to run as data processing or reporting steps to run in the cloud where your data lives.
- Export results as TSV or CSV easily to use elsewhere.
- Save queries and build them on top of each other to break down complex logic into simple steps. 
- 'Branch' a production database locally to test new queries or data safely and without a full-size second database.
- Extend the query language in C# for things hard to express in queries.

[[https://github.com/Microsoft/elfie-arriba/blob/master/images/XForm_WebUI.PNG|alt=XForm Web Interface]]

## Install XForm
- Unzip the latest [Release](https://github.com/Microsoft/elfie-arriba/releases) to a new folder, or [build XForm](https://github.com/Microsoft/elfie-arriba/wiki/XForm-Build)
- Unzip [SampleDatabase.zip](https://github.com/Microsoft/elfie-arriba/raw/master/XForm/XForm.Test/SampleDatabase.zip) for sample data.
- Add XForm to your System Path to run it easily from anywhere.

## Import Data
An XForm 'database' is just a folder in the file system. CSV or TSV input data, tables in XForm's own binary format, queries, and generated reports are all just files in the folder. XForm can also use a file share or cloud storage as a database.

- Open a Command Prompt
- Go to the folder you'd like to use as your local database.
```
    XForm add WebRequest.Full.20171201.r5.n1000.csv WebRequest
```
XForm copies the CSV to "Source\WebRequest\Full\2017.12.01 00.00.00Z\WebRequest.Full.20171201.r5.n1000.csv". This means XForm sees one version of the 'WebRequest' table, which is current from 2017-12-01 forward.

## Explore Data
XForm provides a great web interface for exploring your data. You can run it on your own computer or host it on a server to share with a team.
- From the command prompt in your database folder, run:
```
    XForm web
```
- Open a browser and go to 'http://localhost:5073'. You may want to bookmark this site.
- Change the Query to:
```
    read WebRequest
    where [ClientBrowser]: "Edge" AND [HttpStatus] != "200"
```
- Click 'CSV' in the top right to download the results as a CSV
- On the top left, type a query name (Edge.FailedRequests) and click 'Save' to save this as a query.

## Export Results
You can explore data in the Web interface or have XForm build tabular outputs from the command prompt.
- From the command prompt in your database folder, run:
```
    XForm build Edge.FailedRequests tsv
```
- This will build a TSV of your query results.
- To export data as of a past moment in time, run:
```
    XForm build Edge.FailedRequests tsv "2017-12-01 18:30:15"
```

## XForm Basics
An XForm 'database' is just a folder in the file system. XForm can also work with data on file shares or in cloud storage.
  - XForm imports and exports data in tabular formats [CSV, TSV, and tabular JSON], or you can implement your own readers and writers. Tabular data is stored in the **Source** folder.
  - XForm converts tabular data into a binary format to provide faster search and processing. Only columns the query requires are downloaded and loaded. Binary format tables are stored in the **Table** folder by table name.
  - XForm queries are written in a language called 'XQL', and are stored in the **Config** and **Query** folders.
  - Queries in the Config folder will have a binary format table with a matching name created with the results.
  - Queries in the Query folder will run dynamically only.
  - Queries begin with 'read <TableName>', which can read a tabular source or the results of any other query or config.
  - XForm provides comprehensive IntelliSense in the Web UX or the interactive command line.