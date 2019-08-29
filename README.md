Created 28 August 2019
Updated 28 August 2019

Using reconcile-csv to reconcile OpenRefine data from local CSV files
===============================================================================

This repository contains instructions and example files for using the [reconcile-csv](http://okfnlabs.org/reconcile-csv/) software to reconcile OpenRefine records against a local spreadsheet. This method can be used during data cleanup to match unique identifiers to records based on fuzzy matching and/or multiple criteria fields.

Example data files
-------------------------------------------------------------------------------

- **records.csv**: contain sample record data

- **species.csv**: contain sample species taxonomic data

Process
-------------------------------------------------------------------------------

### Set up OpenRefine project and reconcile-csv server

1. Download reconcile-csv (http://okfnlabs.org/reconcile-csv/). It is recommended that **reconcile-csv-0.1.2.jar** be saved in the same directory as species.csv.

2. Start OpenRefine. Double-click on the diamond icon and it should automatically start in the default browser. If needed, navigate to `http://127.0.0.1:3333` in a browser window.

3. Create a new project, then choose `records.csv`. Make sure to specify that columns are separated by comma, give the project a new Project name, then Create Project.

4. Open a Command Line/Terminal window and navigate to the directory with reconcile-csv-0.1.2.jar and species.csv. Then run the following command:
~~~
java -Xmx2g -jar reconcile-csv-0.1.2.jar species.csv scientific_name taxon_id
~~~
The following message should be printed:
~~~
Starting CSV Reconciliation service
Point refine to http://localhost:8000 as reconciliation service
<date> <time>:INFO:oejs.Server:jetty-7.x.y-SNAPSHOT
<date> <time>:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:8000
~~~

### Reconcile species names

5. Go back to OpenRefine. Click on the dropdown arrow next to the species column header, go down to *Reconcile*, then click *Start Reconciling...*

6. At the bottom of the prompt, click **Add Standard Service...**

7. When prompted to enter the service's URL, enter `http://127.0.0.1:8000/reconcile`, then click **Add Service**.

> At this point, the *Reconcile column "species"* prompt should come up quickly. If not, and there is just a prolonged *Working* wheel, it means that there is a problem with the formatting of species.csv. reconcile-csv is very sensitive to CSV formatting, but unfortunately does not produce useful errors stating as much. Fix the formatting of species.csv; make sure every line has the expected number of values, and make sure there are no blank rows at the bottom of the file.

8. On the *Reconcile column "species"* prompt, the defaults should suffice. At the lower right, click the **Start Reconciling** button.

9. Once OpenRefine is done reconciling, each name in the species column will now be a blue link. Clicking on the link opens up a new tab containing the row that was matched in the CSV, and all the associated information to the matched species name.

### Resolve mismatches and ambiguities

10. Several records that did not find perfect matches will need to be manually corrected. OpenRefine will show a list of likely matches, along with a decimal value showing how close each option matches the reconciled value. You can use the auto-generated *species:judgment* facet to isolate unmatched records if you would like.
+ record_id 15: Bos taurus is not in species.csv. Click the single checkbox next to *Create new item*.
+ record_id 24: Faciolela oxyrynca is misspelled. CLick the single checkbox next to *Facciolella oxyrhyncha*.
+ record_id 36: Cofea farabica is misspelled. Click the single checkbox next to *Coffea arabica*.

### Add a new taxon_id column to the dataset

11. Click on the dropdown arrow next to the species column header, go down to *Edit column*, then click *Add column based on this column...*

12. In the *New column name* box, enter `taxon_id`. In the *Expression* box, enter `cell.recon.match.id`. In the *Preview* tab, you should see the column populated with taxon ID numbers. Then click **OK**.

### Add data from other CSV fields

This is an example of adding kingdom data from the reconciliation CSV file to the dataset. This process can be repeated for as many columns as desired; just substitute the field appropriately.

13. Click on the dropdown arrow next to the taxon_id column header, go down to *Edit column*, then click *Add column by fetching URLs*.

14. In *New Column Name*, enter `html`. In the *Expression* box, enter `"http://127.0.0.1:8000/view/"+value`. Then click **OK**. (It may take a while for this step to run.)

15. Click on the dropdown arrow next to the html header, go down to *Edit column*, then click *Add column based on this column...*

16. In *New column name*, enter `kingdom`. In the *Expression* box, enter `value.parseHtml().select("tr:contains(kingdom)")[0].select("td")[1].htmlText()`. Then click **OK**.

17. Steps 15 and 16 can be repeated for as many new columns as desired. For each, replace `kingdom` in `"tr:contains(kingdom)"` with the CSV data field you would like to retrieve.

18. Once all new data have been added, the *html* column can be removed.

### Export new dataset

Once all other cleanup operations have been completed, the new dataset can be exported with the **Export** dropdown menu at the top.

To stop the reconcile-csv server, type `[Ctrl]+c` in the Command Line/Terminal window. Alternatively, closing the Command Line/Terminal window should also stop the server.
