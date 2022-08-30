# Table of Contents

* [CDS](#cds) - Core Data Services
  * [General Knowledge](#general-knowledge)
  * [Naming conventions](#naming-conventions)
  * [Referencing other models](#referencing-other-models)
  * [Useful commands](#useful-commands)
  * [Useful APIs](#useful-apis)
  * [OData](#odata)
  * [Deploying to the cloud](#deploying-to-the-cloud)

* [UI](#ui)

## **CDS**

### **General Knowledge**

`package.json` is where metadata for the cds application is stored, e.g. the persistent database where to load the data from
The foreign key for a to-one association is generally `<entity_name>_ID` (e.g. In table books `author_ID` pointing to the author) but for code lists it is `<entity_name>_code` (e.g. In table orders `country_code` pointing to the country)  

### **Naming conventions**

These will make cds watch automatically set up the db and services

* `db/` folder: schema.cds file with the domain modeling (entities, associations, etc)
* `data/` or `csv` folder: where the csv files have the format `<namespace-table>.csv` (e.g. `sap.capire.bookshop-Books.csv` or `sap-capire-bookshop-Books.csv`)
* `srv/` folder:
implementation of services with the same name as the cds file (e.g. cat-service.cds and cat-service.js)
* `app/` folder: where the web apps are stored. If there is an `index.html` file in it `cds watch` will use this as the default page

### **Referencing other models**

Names starting at `./` or `../` are resolved relative to the current model  
If the reference is a directory, it is loaded from a `.../index.<json|cds>` file in that directory (... means to look recursively in that directory)  
Others are absolute references, prefixed with `@`, look under the `node_modules` folder for that path (?)

### **Useful commands**

`cds watch` - starts and keeps track of changes in the application  
`cds deploy --to <db_type:custom_db_name>` - deploys the models to a persistent database (sqlite, hana, etc)  
`cds compile <cds_file_path> --to sql` - to create the Tables/Views needed to represent the models in that file and see the sql commands in the console

### **Useful APIs**

`db_transaction = srv.transaction(req)` - starts or joins a transaction on the db (rolls back if errors occur)
`db_transaction.run(<query>)` - runs the query on the database

### **OData**

OData queries support expansions through a foreign key, for instance `<URL>/catalog/Authors?$expand=books($select=ID,title)` to show the authors with their respective books

### **Deploying to the cloud**

* `cf login` to login to the cloud foundry account and space
* `cds add hana --force` to update `package.json` with the requirement to use hana as the data source
* `cds build/all` to build the application
* `cf create-service <type_of_service>(look up in cf marketplace, probably hana) <service_plan> (hdi-shared) <service_name> (in gen/db/src/manifest.yaml under services)`
* `cf push -f gen/db/` to push the data (tables, views, etc) into the cloud
* `cf push -f gen/srv/ --random-route (creates a new url every time it is deployed)` to push the actual service - the generated url is shown in the console under `routes`

#### For multitarget applications

As an alternative to the built-in method above, this way the application is deployed as a whole - less manual steps, so easier to automate for production

* `cds add mta` generates an `mta.yaml` with various modules (Cloud Foundry Apps) and resources (Cloud Foundry Spaces)
* `mbt build` to build the application archive (builds each module in `mta.yaml`)
* `cf deploy <archive-path>` (mtar file)
* `cf logs <service-name>` to see what is going on with a given service

#### Authentication and Authorization

`cds compile srv/ --to xsuaa > xs-security.json` updates package.json with a requirement for uaa  
`path: xs-security.json` should be added to the mta.yaml under `resources`

## **UI**

UIs can be built through freestyle SAPUI5 applications or using SAP Fiori elements applications.  
The latter is based on SAPUI5, but provides a series of out-of-the-box elements for most common usecases  
A `launchpage.html` file can have all the web apps centralized and shown in a local SAP launchpad
