**Updating Documentation for EditableGrid 2.0**

# EditableGrid v2.0 

**Learn EditableGrid by playing with the examples on here: https://davelab6.github.io/editablegrid/examples/index.html**

# EditableGrid v1.0.11 

**Documentation below is NOT up to date as it does not contain the latest features added in EditableGrid 2.0.**

**As of September 14th, 2017, the documentation for below has been SIMPLIFIED.** 

##### Table of Contents

[Introduction](#introduction)
[A Simple Example](#a-simple-example)
[Introduction](#introduction)
[Improving the First Example](#improving-the-first-example)
[PHP Helper Class](#php-helper-class)
[Charts](#charts)
[Extensions](#extensions)

## Introduction

EditableGrid is a Javascript library that allows you to make the tables in your web sites and applications editable.
EditableGrid focuses on simplicity: only a few lines of code are required to get your first table up and running.

You can use the EditableGrid library in 2 ways:

* **Hook up to an existing HTML table**

   - Useful for attaching EditableGrid to an existing application

* **Build your grid from an XML description of the table structure and contents**

   - Useful for writing new web app or improving upon existing web app

## API Documentation

In directory **doc/jsdoc** you will find a reference documentation of all classes and functions available in this library.

## A Simple Example

This library is simple to use. *The best way to learn is through reading the examples*.
Let's have a look at **examples/demo.html** in your favorite browser.
* You can edit various types of cells *(string, numeric, enum, email, boolean, date)*.
* You can sort columns by clicking on the column header and remove rows.

To get started, we use a simple demo.
In file **demo.html** at line 12, you just have to replace **"demo.js" with "demo_simple.js"** and refresh the file in your browser. What you'll see is the same editable grid, but with some features removed.

Here's the source code:

* demo.html/css: main html page and stylesheet for the table
* demo.xml: xml file containing the table structure and contents
* demo_simple.js: javascript code to create our editable grid

Let's analyze each file:

1. HTML Page: includes important javascript, css files, and divs.

2. The XML file contains two parts:

   **a. The table structure, enclosed in the <metadata> tag:**

      Table structure declaration consists of declaring a list of columns with information in each column:
      - **name**: name can for example be the name of the corresponding field in a database
      - **label**: label will be used in the table header (if no label given -> the name is used) 
      - **type**: type can be one of the following: string, integer, double, boolean, date, email, website, html (if not given -> "string" is assumed)
      - **editable**: table editable or not (if not specified -> then not editable)
      
      For **double and integer columns**, the type can also provide additional information, such as the unit, the precision, and/or the symbol to use for NaN, e.g.:
      - **double(m)**: unit is 'm'
      - **double(1)**: precision is 1 (and unit is not specified)
      - **double(m, 1)**: unit is 'm' and precision is 1
      - **double(m, 1, n/a)**: unit is 'm', precision is 1 and symbol for NaN is 'n/a'
      - **double(€, 2, -)**: unit is '€', precision is 2 and symbol for NaN is '-'
 
      For string and email columns, you can specify the length of the text field, e.g.: string(40) or email(24).
      The default length is 12 for string and 32 for email and website. 

      **Validations**
      
      A specific style will be applied to numeric columns (bold and aligned to the right).
      Also, when a numeric cell is edited, the user won't be able to apply changes until it's a valid numeric value.
      For emails and dates also, we check that the entered value is a valid email address, resp. date.
   
      You can also specify a list of predefined values for each column.
      In this case, when a cell is edited, the user will be presented with a drop-down list containing these predefined values.  

   b. **The table contents enclosed in the <data> tag:**

      - This is a list of rows, where each row contains a value for each column.
      - It is recommended to name your columns: this way you can declare them in any order, and skip some columns. 
      - Naming the columns is not mandatory: the order used in metadata must then be respected.
      - You can also assign a unique id with each row (for example, the id of the row in a database).

   Just have a look at the XML file demo.xml; it's self-explanatory.

3. In the Javascript file **demo_simple.js**, all we do is to create an EditableGrid object and specify 2 things:

   a. **A function "tableLoaded" that will be called when the grid has been fully loaded.**
      - Here we simply display a message and define a renderer for the action column in order to display a delete button (see below for more details about cell renderers).
      - Then we render the loaded grid in our div "tablecontent" using out css style "testgrid".

   b. **A function "modelChanged" that will be called each time a cell has been modified by the uses.**
      - Here we just display a message showing the new value.
      - In real life, it's here that you can for example update your database (with an AJAX request). 

   *When the EditableGrid object is created with the necessary callbacks activated, we use function "loadXML" to load the XMl file demo.xml.*

That ends the basics of learning EditableGrid.

## Improving the First Example

You remember we asked you to replace **demo.js** with **demo_simple.js** in demo.html.
Well now that you are ready to see the real power of EditableGrid, you can use "demo.js" again.

In this part, we'll see how to further customize our editable grid, using 3 mechanisms:

* **cell and header renderers**: allows customizing the way values are displayed
* **cell validators**: allows you to detect when user value is valid/not valid
* **enumeration providers**: allows you to dynamically build a column's list of possible values
 
The file **"demo.js"** will provide 8 following new features to our 1st example:

1. **show more details** when a cell gets modified
2. **add a validator** to strictly verify age bounds [16, 100] (e.g.)
3. display the **unit** in the HEIGHT column
4. use a **flag image** to display the selected country
5. propose a **list of countries** that depends on the selected continent
6. **clear the selected country** when another continent is selected
7. **display an information icon** in the table header for some columns
8. **attach to an existing HTML table** (see file demo_attach.html)

Let's see how each feature's 8 implementations in the **demo.js** example:

1. The modelChanged callback function receives the parameters **rowIndex, columnIndex, oldValue, newValue, row**.
   The last parameter **"row"** can be used to **get the row id** that was specified in the XML file.  
   In the example, we display a message:
   
       "Value for '" + this.getColumnName(columnIndex) + "' in row " + row.id + " has changed from '" + oldValue + "' to '" + newValue + "'"

2. To associate a custom validation with a column, you can use the addCellValidator function which takes a CellValidator object, like this:
 
       addCellValidator("age", new CellValidator({
       isValid: function(value) { return value == "" || (parseInt(value) >= 16 && parseInt(value) < 100); }
       }));
 
3. To customize the way cells in a particular column are displayed you must associate a CellRenderer, using the function setCellRenderer:

       setCellRenderer("height", new CellRenderer({
       render: function(cell, value) { new NumberCellRenderer().render(cell, value ? value + " m" : ""); }
       })); 

4. Here again, to display a flag for the country we associate a custom CellRenderer with the column 'country':

       setCellRenderer("country", new CellRenderer({
      render: function(cell, value) { cell.innerHTML = value ? "<img src='images/flags/" + value.toLowerCase() + ".png' alt='" + value + "'/>" : ""; }
       })); 

5. To achieve this, we must associate a custom EnumProvider with our column, using the function setEnumProvider:

```
   setEnumProvider("country", new EnumProvider({ 
       getOptionValues: function (grid, column, rowIndex) {
           var continent = grid.getValueAt(rowIndex, 4);
           if (continent == "eu") return { "be" : "Belgique", "fr" : "France", "uk" : "Great-Britain", "nl": "Nederland"};
           else if (continent == "am") return { "br" : "Brazil", "ca": "Canada", "us" : "USA" };
           else if (continent == "af") return { "ng" : "Nigeria", "za": "South Africa", "zw" : "Zimbabwe" };
           return null;
       }
   }));
```


   The function getOptionValues is called each time the cell must be edited.
   Here we do only client-side Javascript processing, but you could use Ajax here to communicate with your server.
   If you do,then don't forget to use Ajax in synchronous mode. 

6. In the modeChanged function we can check the modified column: if it is the continent then we clear the country in the same row:

```
   if (this.getColumnName(columnIndex) == "continent") 
		this.setValueAt(rowIndex, this.getColumnIndex("country"), "");
```

7. We will use the method **setHeaderRenderer** that allows us to **customize the way column headers are rendered**.
   Since we want to use the same renderer for several columns, we are going to declare a class InfoHeaderRenderer which extends CellRenderer:

```
   function InfoHeaderRenderer(message) { this.message = message; };
   InfoHeaderRenderer.prototype = new CellRenderer();
   InfoHeaderRenderer.prototype.render = function(cell, value) { /* create the icon, see demo.js for the code */ }
```
   We use this class with setHeaderRender as follows:
   
```	    
   setHeaderRenderer("age", new InfoHeaderRenderer("The age must be an integer between 16 and 99"));
   setHeaderRenderer("height", new InfoHeaderRenderer("The height is given in meters"));
```
8. To attach to an existing HTML table you **won't use the loadXML function as before**.
   Instead **you will use the function attachToHTMLTable**, as follows:

```   
   	editableGrid.attachToHTMLTable($('htmlgrid'), 
		[ new Column({ name: "name", datatype: "string(24)" }),
		  new Column({ name: "firstname", datatype: "string" }),
		  new Column({ name: "age", datatype: "integer" }),
		  new Column({ name: "height", datatype: "double" }),
		  new Column({ name: "continent", datatype: "string", optionValues: {"eu": "Europa", "am": "America", "af": "Africa" }}),
		  new Column({ name: "country", datatype: "string" }),
		  new Column({ name: "email", datatype: "email(26)" }),
		  new Column({ name: "freelance", datatype: "boolean" }) ]);
```   
   - The HTML table you're attaching to will define the contents of the table.
   - Since we don't have the XML metadata to define the table columns, you must declare these columns using Javascript as shown above.
   - In this case you don't have to register the callback method "tableLoaded" since nothing has to be loaded.
 
## PHP helper class

If you work with PHP on the server side, you can use the EditableGrid.php class to ease the creation and rendering of the XML data.
 
## Charts
 
EditableGrid allows you to create **bar and pie charts** from the grid data.

This utilizes OpenFlashCharts 2.0, which you should include in your client application.
The following javascript files must be included (provided by the official OpenFlashChart 2.0 download links):

* openflashchart/js-ofc-library/open_flash_chart.min.js
* openflashchart/js-ofc-library/ofc.js
* openflashchart/js/swfobject.js
* openflashchart/js/json/json2.js
 
If any of these files is missing, trying to use the charting methods will display an error indicating the missing library.
The SWF file must be named "open-flash-chart.swf" and must be placed in the "openflashchart" directory: it will then be detected automatically.
 
**Creating Charts with EditableGrid**

The 1st thing to do is to **have some empty DIV** in your page that will be used as a **container** for the chart. 
You can control the chart dimensions by setting the "width" and "height" attributes, inline or through a CSS stylesheet.
Otherwise some default dimensions will be used (500*200px).
 
Based upon the chart you desire, you have to call one of the following methods:
 
1. **renderBarChart(divId, title, labelColumnIndexOrName, legend)**
 
   - This method will create a bar chart: labelColumnIndexOrName is the name or index of the column that will be used for the chart categories.
   - For each category, you will have one bar per numerical column having the "bar" attribute set to true (which is the default).
   - The 'bar' attribute can be set in the grid XML metadata when declaring a column.
   - The 'legend' parameter is optional: by default the label of the column given by "labelColumnIndexOrName" will be used.
   You don't have to care about the chart's scale: it will be computed automatically to best fit the data.

   **Example**: Imagine you have the following data: # of kM ran by person every year.
 
```
	name    2009   2010
	--------------------
	John    40000  30000
 	Jack    60000  20000
 	Paul    20000  50000
``` 	
   Calling renderBarChart(divId, "Kilometers", "name") will produce the following chart:
 
```
	# 2009  * 2010
             
     60,000           #
     50,000           #          *
     40,000  #        #          *
     30,000  # *      #          *
     20,000  # *      # *      # *
     10,000  # *      # *      # *
     
          0  John     Jack     Paul
          
                   Person	   
```
 
2. **renderPieChart(divId, title, valueColumnIndexOrName, labelColumnIndexOrName, startAngle)**

   This method will create a **pie chart**: 
   
   * **valueColumnIndexOrName**: name or index of the column that will be used as the **value** for each pie part
   * **labelColumnIndexOrName**: name or index of the column that will be used as the **label** for each pie part
 
   - In other words, you can display the distribution of the values of "valueColumnIndexOrName" as a pie, using "labelColumnIndexOrName" to label each value.
   - The percentage of each value w.r.t. the column's total will also be displayed in the label.
   - The startAngle parameter is optional: it gives the angle of the first pie part (default is 0).
   - If no title is given (i.e. title is null) the label of the column given by "valueColumnIndexOrName" will be used.
 
Both methods renderPieChart and renderBarChart can be called any number of times: if the chart already exists it will be updated (ie. not rebuilt).
For example, you can call one of these methods each time the table is sorted (tableSorted) or edited (modelChanged), in order to update the chart to match the new data.
Updating a chart is very fast, which gives a very nice effect: when sorting or editing the grid, the chart "follows" beautifully.
 
If your grid has a row displaying e.g. the total, you can ignore it in the charts by setting EditableGrid.ignoreLastRow to true.
In this case, this last row will also be ignored when sorting the grid data.

## Extensions

The directory "extensions" is there for interesting cell editors, renderers and validators.
It contains a subdirectory "jquery" with the jQuery library v1.4.2.
Extensions may indeed be based on jQuery, unlike the core library which stays jQuery-independent.

Current extensions:

* two variants of "AutocompleteCellEditor", each based on a different autocomplete jQuery plugin
