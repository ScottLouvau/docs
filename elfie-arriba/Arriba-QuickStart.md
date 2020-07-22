Here's how to get Arriba and the website running on your machine with sample CSV data in around 15 minutes.

In this example, we found [CSV data](https://github.com/Microsoft/elfie-arriba/tree/master/Arriba/Databases/Walkthrough) for US Electricity prices by ZIP code and US Population by ZIP code. We'll build an Arriba table with the two data sets combined and explore it with the default website, and then customize the site. 

## Enlist and Build Arriba
See [Enlist and Build Arriba](https://github.com/Microsoft/elfie-arriba/wiki/Arriba-Enlist-and-Build) for the steps.
Run `node build.js` with no argument to build the default website for now.


## Build Arriba Tables from your CSV data. 
Your rows must have a unique ID which is the first column or the first column containing 'ID' in the name.
Arriba will automatically create columns based on the data seen and will infer the column types.
All columns are sorted and indexed by default, for instant search.
Use "build" to create a new table. 
Use "append" to add rows from additional CSVs.
Use "decorate" to add additional columns to existing rows. In our example, we're adding population data to the rows with electricity rate data.

From a Command Prompt:  
```
PUSHD ~\elfie-arriba\Arriba
bin\Release\Arriba.Csv.exe /mode:build /table:Rates /csvPath:"Databases\Walkthrough\PowerRatesByZIP.csv" /maximumCount:60000
bin\Release\Arriba.Csv.exe /mode:decorate /table:Rates /csvPath:"Databases\Walkthrough\PopulationByZIP.csv"
```
[[https://github.com/Microsoft/elfie-arriba/blob/master/images/Arriba_CsvCrawl.PNG|alt=Command Line showing Arriba.Csv commands]]


You can pass /settings to control column types, indexing rules, and permissions of the resulting table. See [SampleSettings.json](https://github.com/Microsoft/elfie-arriba/blob/master/Arriba/Databases/Walkthrough/Settings.json) for an example.
You can also add data to Arriba tables directly via the HTTP service, which isn't shown here.

Your data is written to a 'DiskCache' folder, two folders up from Arriba.Csv.exe [so, in Arriba\DiskCache\Tables\<TableName>. Just delete the table folder to delete the table. You need to restart Arriba.Server.exe if you re-create a table with the same name.

## Run the Arriba service locally
Arriba has an HTTP service which responds to queries. In production it's hosted by IIS using the Arriba.IIS site, but there's a command line version for local debugging, Arriba.Server.exe. You can secure the site with Windows authentication and set table, row, and column permissions. You can upload new data, delete rows, and administer tables via the service as well.

From an ***elevated*** command prompt (so it can open port 42784):  
```
PUSHD ~\elfie-arriba\Arriba\bin\Release
Arriba.Server.exe
```

## Run the Website locally
Arriba has a fast, beautiful website to help users quickly search through and explore data. Search directly for terms in any field or write structured queries. IntelliSense shows you the columns and types in your table. Inline Insights answers questions as you type the query. A configurable listing and customizable details show you matches instantly. The Grid enables quick comparisons and analytics.

From a command prompt:  
```
PUSHD ~\elfie-arriba\Arriba\Arriba.Web
npm start
```

Browse to http://localhost:8080. [Note: Edge won't browse to localhost correctly; Chrome and IE work properly.]

## Customize the Website
The Arriba website is customizable, from the theme colors and the help text to the columns shown by default and the look of the details views when you click on items.

- Copy Arriba.Web/configuration.default to a new Configuration folder [it's already done in: Arriba\Databases\Walkthrough\Configuration]
- In theme.scss, change the theme color (uncomment the second block for orange).
- In Configuration.jsx, change the app name, feedback contacts, default listing columns, and more. [See Walkthrough Configuration.jsx](https://github.com/Microsoft/elfie-arriba/blob/master/Arriba/Databases/Walkthrough/Configuration/Configuration.jsx)
```
        ...
        toolName: "Rates",

        listingDefaults: {
            "Rates": { columns: ["ZIP", "UtilityName", "ResidentialKwhRate"], sortColumn: "ZIP", sortOrder: "asc" }
        },

        startContent: {
            overview:(
                <span>
                    <b><a target="_blank" href="https://catalog.data.gov/dataset/u-s-electric-utility-companies-and-rates-look-up-by-zipcode-feb-2011-57a7c">US Electricity Rates</a></b> and <b><a target="_blank" href="https://blog.splitwise.com/2013/09/18/the-2010-us-census-population-by-zip-code-totally-free/">Population</a></b> by ZIP indexed.
                    Need more <a href="Search.html?help=true">help</a>?
                </span>
            ),
            examples: {
                "WA": <span>See rates by ZIP for Washington.</span>,
                "ResidentialKwhRate < 0.10": <span>Find ZIPs with power &lt; 0.10/KWh.</span>,
            }
        },
        ...
```

- See [Walkthrough Configuration.jsx](https://github.com/Microsoft/elfie-arriba/blob/wdg-release/Arriba/Databases/Walkthrough/Configuration/Configuration.jsx)

- Add your own JSX classes for custom details views.
- Reference them in Configuration.jsx like:  
```
  import RateDetails from "./RateDetails";
  ...
  customDetailsProviders: {
    "Rates": RateDetails
  } 
  ...
```
- See [Walkthrough RateDetails.jsx](https://github.com/Microsoft/elfie-arriba/blob/master/Arriba/Databases/Walkthrough/Configuration/RateDetails.jsx)

## Build the customized site
```
PUSHD ~/elfie-arriba/Arriba/Arriba.Web
node build.js Databases\Walkthrough\Configuration
```
[[https://github.com/Microsoft/elfie-arriba/blob/master/images/Arriba_ThemedHome.PNG|alt=Themed Arriba Site]]

[[https://github.com/Microsoft/elfie-arriba/blob/master/images/Arriba_CustomDetailsView.PNG|alt=Custom Details View]]

[[https://github.com/Microsoft/elfie-arriba/blob/master/images/Arriba_ThemedWithGrid.PNG|alt=Themed Arriba Grid]]


