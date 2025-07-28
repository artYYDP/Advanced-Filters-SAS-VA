# 🔍 Filtering terms in different columns

## 🧠 Objective

Allow the user to freely enter a term (e.g., "Maria" or "12345") and have that term searched in **more than one column** of the same table — for example: `Name` **or** `CPF`.

## 🧩 Practical example

Imagine a table with the names of the Brazilian states (UFs) and municipalities, where we want the same text filter to search both the state name and the municipality name.

For this practical example, we'll use an IBGE municipalities table. This table contains the following data:

| Column | Description | Type |
| - | - | - |
| CD_UF | State code | Numeric |
| DS_UF | State description | Character |
| CD_MUN | Municipality code | Numeric |
| DS_MUN | Municipality description | Character |

Click [Download MUNICIPIOS.csv](/files/MUNICIPIOS.csv) to get the CSV and upload the file to your SAS Viya environment. Then, use the code below to load the table into memory in the `PUBLIC` CASLIB:

```sas
/* Define the CSV file path */
%let filepath = [file_location]/MUNICIPIOS.csv;

/* Import the CSV file */
proc import datafile="&filepath." out=PUBLIC.MUNICIPIOS dbms=csv replace;
    delimiter=';';
    getnames=yes;
    guessingrows=32767;
run;

/* Promote the table to the Public CASLIB */
proc casutil;
    promote incaslib="PUBLIC" casdata="MUNICIPIOS" outcaslib="Public";
quit;
```

Now with our table in memory, we can go to SAS Visual Analytics (VA) and create our search.

On the SAS VA home screen, let's create a new report and link the data source that we loaded into memory.

![01](/images/FilterColunms/01.png)

![02](/images/FilterColunms/02.png)

Click on the Objects icon (the second icon on the left, from top to bottom) and search for `text` in the filters. Select and drag the **Text Entry** to the SAS area:

![03](/images/FilterColunms/03.png)

![04](/images/FilterColunms/04.png)

Then, go back to the same screen and select the **List Table**, dragging it to the panel:

![05](/images/FilterColunms/05.png)

Click on **Assign data** in the table and configure as shown below:

![06](/images/FilterColunms/06.png)

![07](/images/FilterColunms/07.png)

![08](/images/FilterColunms/08.png)

Now, let's add a parameter to allow the search through it. To do this, go to **Data > New data item > Parameter**:

![09](/images/FilterColunms/09.png)

Name the parameter and set it as `Character`, leaving the current value empty:

![10](/images/FilterColunms/10.png)

![11](/images/FilterColunms/11.png)

Then, in the **Assign data** field of the text box, assign the created parameter:

![12](/images/FilterColunms/12.png)

![13](/images/FilterColunms/13.png)

Next, select the table, click on the filter icon in the upper right corner of the screen, and choose **New filter**, selecting the **Advanced filter** option:

![15](/images/FilterColunms/15.png)

In this part of the **Filter expression**, this is where the magic happens for everything to work correctly. I will explain each detail separately to ensure that you fully understand what we are doing here.

The filter expression requires a condition, and that's exactly what we're going to apply. The condition we'll use is as follows:

```plaintext
OR [
  FiltroGeral Missing
  OR [
    LowerCase(DS_UF) Contains LowerCase(FiltroGeral)
    LowerCase(DS_MUN) Contains LowerCase(FiltroGeral)
  ]
]
```

- FiltroGeral: is the parameter you created to be used in the search;
- OR: represents the "OR" condition, i.e., the expression will be true if any of the conditions are met;
- Missing: makes the condition return empty when no value is provided, allowing all records to appear in the table;
- LowerCase: converts the text to lowercase, ensuring that the search is case-insensitive. The same effect can be achieved with UpperCase;
- Contains: checks if the field contains the specified text, useful for partial searches.

Apply the condition exactly as shown in the image below:

![16](/images/FilterColunms/16.png)

Then, click OK and test your new search bar:

![17](/images/FilterColunms/17.png)

If you wish, you can expand the search to more columns. To do this, just add more OR conditions and follow the same structure as the previous condition.

## Acknowledgments

- [Geiziane Oliveira](https://www.linkedin.com/in/geiziane-oliveira-0a5882110/) - SAS Mentor, whose guidance was essential for the construction of this parameter.

## References

- [Text Input linked to multiple categories in SAS VA](https://communities.sas.com/t5/SAS-Visual-Analytics/Text-Input-linked-to-multiple-categories-in-SAS-VA/m-p/784471#M15682)