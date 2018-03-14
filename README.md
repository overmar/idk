<div>
<img src="https://www.novisci.com/img/novisci_banner.png" height="65" width="200">
</div>


# NoviSci - Saxapahaw #

### Explanation of the NoviSci Common Data Format (CDF) ###

In an effort to create a data structure which can easily be utilized in a noSQL environment, ie all of the information we could possibly need is in one single table, this structure was created. In its initial iterations the ES has been mapped to both the Flatiron and Marketscan data, which is remarked in the function ```ns.cdm```.

Formatted Variables

-  **patient_id:**
    REQUIRED
    + description: synthetic patient identifier
    + The patient identifier is stripped in the formatting process leaving a number starting at 1. 
    <br>To create this table, one creates a table with 2 columns, the first is the unique identifier in the database, and the second is a number from 1 to n. This 2 column table is then used in ```ns.cdm``` as the id.table argument. It is important to keep this table as a permanent dataset, Because it is the only linkage between the CDM id's and the id variable in the data.
    + type: numeric
    + example: 1,2,1500087
<!---
Figure out how to turn this into a better looking table
--->

|linkedid |patient_id |
|---      |---        |
|25681483 |1          |
|18945632478  |2      |

- **event_id:**
    REQUIRED
    + description: synthetic event identifier
    <br>The variable used in this table is essentially the row number of the data in its table. A permanent variable should be created in the source data to be referenced as the event_id to facilitate quickly finding relevant information in the source data when needed.
    + type: numeric
    + example: 1, 2, n...

- **start_date:**
    DEFAULT (NA)
    + description: start date of an event or missing for information without a time dimension
    <br>For events which only have one date, ie procedures, start_date should be the date of the procedure and end_date should be left blank. However for items which are timeless, ie gender, race then both the start_date and end_date are left blank. Although this can be reported as Year only or Month and Year only, our current convention is that these variables should be converted to day 15 of the month for YYYY-MM and June 1 for Year only. In future versions this may be done internally by a function rather than having to be set before entering into the CDF.<br>This variable also has to be agreed upon before utilizing the database, as some tables will present multiple values which could be used, ie the Flatiron lab table which contains values for both test and result dates, or Medication orders which both have an order date and possibly an administered date. It shouldn't be a problem as long as we are consistent with which variable we choose.
    + type: date or numeric, needs to be consistent in for the entire format
    + format: "YYYY-MM-DD HH:MM:SS TZ", "YYYY-MM-DD", "YYYY-MM", "YYYY", or "NA" for missing values

- **end_date:**
    DEFAULT (NA)
    + description: end date of an event
    <br>This variable can be left as NA if no the type of variable makes no sense to end. For medications this date should be calculated as start date + days supply. This should be calculated before it goes into the CDF. <span style="color:red;text-decoration: underline">Important</span> This variable should be the raw start and end dates of enrollment and medication, without any algorithms to define continuous use.
    + type: date
    + format: same as start_date

- **source_db:**
    REQUIRED
    + description: the name of the source database
    <br>This particularly important if we combine multiple data sources into one database of the CDF. This way it is easy to determine what the data is and where it came from.
    + type: character
    + format: string of the name
    + example: Flatiron, Marketscan
- **source_table:**
    REQUIRED
    + description: name of the table within the source database 
    <br>Important for reference back to the original data. Also necessary to differentiate between diagnoses in the inpatient and outpatient environments.
    + type: character
    + format: string of name
    + example: MCTVOutpatientServices, Diagnosis, etc...
- **source_column:**
    REQUIRED
    + description: name of the column within the table
    <br>This is necessary to easily be able to go back to the specific variable, and combined with the id and the event_id a specific record can be retrieved. Also it is important to know which place the variable occurred in, ie dx1, dx2, dx3. <br>TODO: This may need to be changed from source_column to something more ambiguous if we find that other databases rather than having different variables as wide columns, as different rows. Maybe something like relative position, or what is it?
    + type: character
    + format: string
    + example: dx1, dx2, proc1, proc15, diagnosiscode
- **type:** 
    DEFAULT (NA)
    + description: field for collecting additional information about a code and/or value
    <br>TODO: Possibly used as an omnibus category, Diagnosis, Procedure, or Inpatient, Outpatient, Medication
    Currently this is a variable which really isn't used, but we were likely to need another column.
    + type: character or possibly a number which corresponds to a category to make indexing and querying faster
    + format: TBD
    + example: Unknown
- **code:** 
    REQUIRED (Except maybe for enrollment)
    + description: Value of the source_column. 
    <br>Generally this is thought of as a code Because the most common thing that we are exporting are diagnosis and procedure codes, however it may make sense for it to be called something else as it can be stored as different things. This column would also include things like NDC numbers, diagnosis and procedure codes, demographic variables. This value really is only useful 
    + type: character
    + format: string
    + example: 733.00, M80, 99211
- **value:** 
    DEFAULT (NA)
    + description: Additional value from source_table. 
    <br>When there are more than one value in the source_table that we decide that we need, we can store it in the value variable. An example is labs where the LOINC would take up the code variable, but we need to know what the result of the lab is, in case we need to later characterize the result, rather than simply the knowledge that the event occurred. Another example of this would be dosage of medication when the the NDC is not be captured, but rather than generic name of the drug.<br>TODO: Labs and medications offer a question of if another variable is need to capture information such as unit of measure, as some tests can be reported in two different ways. However if we capture both the dosage and unit of measure it will make it more difficult to query.
    + type: character
    + format: string

-----------
#### Example Mapping ####


|patient_id |event_id   |start_date |end_date |source_db  |source_table |source_column  |type |code |value  |
|---        |---        |---        |---      |---        |---          |---            |---  |---  |---    |
|1          |1          |-50        |10       |Marketscan |OutpatientServices |dx1  |NA |73301 |NA |
|15         |1856       |75         |NA       |Flatiron   |Lab          |LOINC  |Lab  |`1742-6` |25 |
|4500       |8575       |NA         |NA       |Flatiron   |Demographics |race |Demographics |2 |NA |
|4500       |8575       |1954-06-01 |NA       |Flatiron   |Demographics |birthyear |Demographics |1954 |NA |
|5          |25         |2010-01-15 |2010-02-28 |Marketscan |OutpatientDrugClaims |ndcnum |Drugs |00603516632 |NA |
|5          |25         |2010-01-15 |2010-07-15 |Flatiron |MedicationAdministation |commondrugname |Drugs |Denosumab |60 |
|5          |25         |2010-01-15 |2010-07-15 |Marketscan |OutpatientServices |proc1 |Drugs |J0897 |NA |
|5          |200        |2012-06-01 |2012-06-06 |Marketscan |InpatientServices  |pproc  |Inpatient  |99238 |NA |
|5          |15         |2010-01-01 |2010-01-30 |Marketscan |DetailEnrollment |NA  |Enrollment |NA |NA|
