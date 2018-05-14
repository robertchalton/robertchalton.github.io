---
layout: post
published: true
title: Loading Excel w/ SQLcl
date: '2018-05-14'
---


## Processing the Spreadsheet

In SQL Developer, we use the [Apache POI project](https://poi.apache.org/) for all loading of excel. I tried to avoid that and instead use a Javascript based option named [SheetJS](https://github.com/sheetjs/js-xlsx) I've been using this library for parsing the spreadsheet the local youth hockey uses for games and locker room assignments.  The javascript in the browser would grab the xls file filter down today's details then print a webpage which would be displayed on a screen when entering the rink.  My thought was to try and use this same library in SQLcl leveraging Nashorn.

I'm using the minified version of SheetJS which will be seen later in the load command of nashorn with `load('xlsx.full.min.js')`


## The Data

Being NHL Playoffs and alll...  I needed some data for this blog and sports in general are a data producing machine. I found a site called [hockeyabstract](http://www.hockeyabstract.com/testimonials/nhl2017-18) that puts more stats than can be imagined into a spreadsheet for all NHLers. As a spreadshett goes this is A->GJ and a few sheets of various data. The guy behind the site is 
[https://twitter.com/robvollmanNHL](https://twitter.com/robvollmanNHL)





## The config

The real data I needed to load was 25 of the same layout spreadsheet. That means it was time to make something easily repeatable.  This is the start of something I'll keep adding to for now it served the purpose. The basics are quite simple map excel column A to oracle column B maybe the need for a function or static value.

Here's my config JSON object so far.

- table  - Name of the table to insert into
- sheetName - Name of the sheet in the workbook
- debug - duh
- offset - number of rows to skip over
- colmap - array of the mapping of excel columns to database

For colmap, there's a couple options. The simpilist is just a straight a:b mapping. 
> `"A":"first_name` 

Then I needed to trim whitespace one a field so added the transform 
> `"W":{ "colName":"goals"  ,"transform":"trim(:W)" }` 

Lastly, there was a static value which had to go with each row so this was added.  
> `"static_value_testing" : { "colName" : "static_test" , "static" : "Test Junk"}`

Here's the resulting config object for this spreadsheet


```
var config = { "table"  : "nls_stats",
               "offset" : 3,
               "debug" : false,
               "sheetName": "TOT",
               "colmap" : {"O":"id",
                           "S":"first_name",
                           "R":"last_name",
                           "U":"team",
                           "T":"position",
                           "V":"games_played",
                           "W": { "colName":"goals"  ,"transform":"trim(:W)" },
                           "X": { "colName":"assists","transform":"trim(:X)" },
                           "AJ" : "toi",
                           "static_value_testing" : { "colName" : "static_test" , "static" : "Test Junk"}
                       }
           };

```

## Gluing config to SheetJS

I made a fairly simple ~150 lines of javascript that use the above config and process the spreadsheet. This object has 3 'public' methods and 3 more internal intended ones.

`XLSLoader.getDDL` - This function will use the config object and return a create table command where all the columns are defaulted to varchar2(4000). Then I'd adjust the datatypes as needed and create the table.

`XLSLoader.getSQL` - This function creates the statement that is used to perform the insert of the data.

`XLSLoader.load` - Does the work.

```
-- Internal fucntions --
XLSLoader.execBatch - executes the insert and reports on success/failures
XLSLoader.debug     - duh
XLSLoader.b2a       - convert byte array to js array

```

The code... 



```
var global = (function(){ return this; }).call(null);
load('xlsx.full.min.js');
var DBUtil = Java.type('oracle.dbtools.db.DBUtil');
var Statement = Java.type('java.sql.Statement');

var XLSLoader = {
				/* Helper to gen a simple all VARCHAR2(4000) version of the create table */
				getDDL: function(config){
					if ( ! this.config ) { this.config=config}
					this.getSQL(config);
					return this.createDDL;
				},
				/* debug messages */
				debug: function(s){
					if ( typeof this.config["debug"] != undefined && this.config.debug ) {
						ctx.write(s+"\n");
						out.flush();
					}
				},
				/* helper to convert byte array to plain JS array */
				b2a : function (b) {
					var out = new Array(b.length);
					for(var i = 0; i < out.length; i++) out[i] = (b[i] < 0 ? b[i] + 256 : b[i]);
					return out;
				},
				/* bulk of the work */
				load: function (file,config) {
					this.config = config;
					/* Creat the insert to be used */
					this.getSQL(config);

				  ctx.write("Processing : " + file + "\n");
					ctx.write("Using Insert : \n" + config.formattedSQL + "\n");
					if ( ! config.offset ){ config.offset=0;}
					/* read data */
				  var path = java.nio.file.Paths.get(file);
				  var bytes = java.nio.file.Files.readAllBytes(path);
				  var u8a = this.b2a(bytes);


				  var wb = XLSX.read(u8a, {type:"array"});
					this.debug("\t Sheet Loaded.\n");

					/* If a sheetName is passed in try that otherwise sheet 0 will be used  */
					var ws;
					if ( config.sheetName ) {
						ws = wb.Sheets[ config.sheetName];
						if ( ws == null ){
							ctx.write("No Sheet Named: " + config.sheetName + "\n");
						}
					} else {
						ws = wb.Sheets[wb.SheetNames[0]];
					}

					/* Get the Sheet requested or defaulted as json */
					var js = XLSX.utils.sheet_to_json(ws, {header:'A'});
					this.debug("\t Sheet Converted to JSON.\n");

					/* Time the db portion only*/
					var start = new Date().getTime();

				  /* build up a binds object of all columns */
					var stmt;
					var currBinds=[];
					for ( var i = config.offset;i<js.length;i++) {
							this.debug(JSON.stringify(js[i]) + "\n");
				      if ( stmt == null ){
								/* First time get a statement and add current binds to the batch */
								stmt = util.prepareExecute(config.sql,js[i]);
								currBinds[i] = js[i];
								stmt.addBatch();
							} else {
								/* Nth time just add to the batch */
								DBUtil.bind(stmt,js[i]);
								currBinds[i] = js[i];
								stmt.addBatch();
							}
							/* perform insert every 50th batched set*/
							if ( i % 50 == 0 ){
									this.execBatch(stmt,currBinds);
									currBinds.length=0;
							}
				  };
					// flush last batch
					this.execBatch(stmt,currBinds);
					var elapsed =  (new Date().getTime()) - start ;
					ctx.write("\n Rows Loaded:" + (i-config.offset) + " "+ elapsed + "ms"+"\n");

				},
				execBatch: function ( stmt,currBinds){
						ctx.write("Execing Batch...");
						var results = stmt.executeBatch();
						var rows =0;
						for(var i=0;i<results.length;i++){
							/* Check the return array to see which rows worked vs failed. */
							if ( results[i] == Statement.EXECUTE_FAILED) {
									ctx.write("ROW FAILED : " + r + "\n\t:" + JSON.stringify(currBinds[r])+"\n");
							} else if ( results[i] == Statement.SUCCESS_NO_INFO) {
									ctx.write("ROW UNKOWN STATE : " + r + "\n\t:" + JSON.stringify(currBinds[r])+"\n");
							} else {
								rows = rows + results[i];
							}
						}
						ctx.write(rows + "\n");
				},


				getSQL : function (config){
				  /* Build out the SQL insert statement */
				  if ( config.sql == null ) {
				          var sql = "insert into " + config.table +"(";
									var create = "create table " +  config.table +"(\n";
				          var cols = "";
				          var values = "";
				          var i=0;
				            for (var key in config.colmap) {
				                if (config.colmap.hasOwnProperty(key)) {
				                    if ( i>0){
				                      cols = cols + ",";
				                      values = values + ",";
															create = create + " ,";
				                    }
														if ( config.colmap[key] instanceof Object ){
															create = create + "\t" +config.colmap[key].colName;
															if ( config.colmap[key].transform != null ){
																/*  transforms */
																cols = cols  + config.colmap[key].colName;
				                    		values = values  + config.colmap[key].transform;
															} else if ( config.colmap[key].static != null ){
																/*  static */
																cols = cols  + config.colmap[key].colName;
																values = values  + "'"+config.colmap[key].static + "'";
															}else{
																/*  plain */
																cols = cols  + config.colmap[key].colName;
																values = values  + ":"+config.colmap[key].colName;
															}
														} else {
															/*  simple values */
															values = values + ":" +key;
				                    	cols = cols  +config.colmap[key];
															create = create +"\t" + config.colmap[key];
														}
				                    i++;
													create = create + " \tvarchar2(4000)\n";
				                }
				            }
									create = create + ")\n";
									this.createDDL = create;
				          sql = sql + cols + ")values(" + values + ")";
				          /* Format the SQL to read easier */
				          var Format = Java.type("oracle.dbtools.app.Format");
				          config.formattedSQL = new Format().format(sql);
                  this.debug("Using Insert Statement >> \n" + config.formattedSQL + "\n");
				          config.sql = sql;
				    }
				}
}

```


## Loading 

Now putting it all together into the single sql file below. There is quite a bit of work happening as SheetJS has to load the entire XLS file into memory and only then can it start processing the contents.

```
drop table nls_stats
/
create table nls_stats(
  id number,
  first_name varchar2(50),
  last_name varchar2(50),
  team varchar2(50),
  position varchar2(50),
  games_played number,
  goals number,
  toi number,
  assists number,
  static_test varchar2(20)
)
/
script
load('xls_function_batch.js')

var config = { "table"  : "nls_stats",
               "offset" : 3,
               "debug" : false,
               "sheetName": "TOT",
               "colmap" : {"O":"id",
                           "S":"first_name",
                           "R":"last_name",
                           "U":"team",
                           "T":"position",
                           "V":"games_played",
                           "W": { "colName":"goals"  ,"transform":"trim(:W)" },
                           "X": { "colName":"assists","transform":"trim(:X)" },
                           "AJ" : "toi",
                           "static_value_testing" : { "colName" : "static_test" , "static" : "Test Junk"}
                       }   
           };  

XLSLoader.load("NHL 2017-18.xls",config);

/
```

![nhl_load.png](/img/nhl_load.png)


## Next steps

I'm wondering if there should be a dedicated github repo of `sqlcl-extras` Then this library along with lots of the other examples and added commands could all be put there with a install.sql to load them all????