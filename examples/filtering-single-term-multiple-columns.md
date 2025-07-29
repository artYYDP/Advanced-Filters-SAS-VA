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

![01](https://www.dropbox.com/scl/fi/ymvrzlje0gqglusyuw4qv/01.png?rlkey=dy0ht1jmfs0y6oc90c9uizx23&st=dvvns0xy&raw=1)

![02](https://www.dropbox.com/scl/fi/twsjb0s9a9elw68rlte1w/02.png?rlkey=zurwhhsch4ckiollfvcdfbynp&st=4hcma1dx&raw=1)

Click on the Objects icon (the second icon on the left, from top to bottom) and search for `text` in the filters. Select and drag the **Text Entry** to the SAS area:

![03](https://www.dropbox.com/scl/fi/gx0eqpxnlvwjdfcstqx8f/03.png?rlkey=wix9rl6yi6orb62nvj3abhs76&st=e5lfxpod&raw=1)

![04](https://www.dropbox.com/scl/fi/ukxicsrrz5f2v2o8yj2us/04.png?rlkey=nkd5swa41x3in7wodmenihsmd&st=t8f949ru&raw=1)

Then, go back to the same screen and select the **List Table**, dragging it to the panel:

![05](https://www.dropbox.com/scl/fi/mwtkmjnlqynpyo52doxys/05.png?rlkey=ckpkw1sm8rpfkn4y48l0r1eqk&st=oiwoypwv&raw=1)

Click on **Assign data** in the table and configure as shown below:

![06](https://www.dropbox.com/scl/fi/24drtzbwz4yan8g27nxle/06.png?rlkey=1236kxwgc2i95xxfr3v87nsdx&st=mnwlevaa&raw=1)

![07](https://www.dropbox.com/scl/fi/ksz3obt3cn98k2726gk00/07.png?rlkey=xdyi0ipehtzbnpyko4sphips2&st=xxwggtzv&raw=1)

![08](https://www.dropbox.com/scl/fi/2eww2905ism5997c42ad3/08.png?rlkey=i6pjes26yko05kche6gf5sm3f&st=xuf7padu&raw=1)

Now, let's add a parameter to allow the search through it. To do this, go to **Data > New data item > Parameter**:

![09](https://www.dropbox.com/scl/fi/65abdhe4ziwjvmlflr0xk/09.png?rlkey=fx8q4s2x0lno2k1f40gp2yh75&st=8w9qq0wv&raw=1)

Name the parameter and set it as `Character`, leaving the current value empty:

![10](https://www.dropbox.com/scl/fi/ximazapqhn0lxo8iik7yn/10.png?rlkey=d9lsks8nv957lccw1kuvq282f&st=k5403bbc&raw=1)

![11](https://www.dropbox.com/scl/fi/v7y0yd7sqm6zuc1ipjq4y/11.png?rlkey=pzjmcg4g0yn5ig1sgfg7qbjdw&st=v9f3typi&raw=1)

Then, in the **Assign data** field of the text box, assign the created parameter:

![12](https://www.dropbox.com/scl/fi/jwgkjc025dim0h1h7v4wy/12.png?rlkey=thsu8bm4rhxtdrhdxtqy8ch3e&st=lo70nuga&raw=1)

![13](https://www.dropbox.com/scl/fi/4y26iko9eak5kehrsszjt/13.png?rlkey=gdvesuoregepyfolmpt8mh0uv&st=ecr3p6d1&raw=1)

Next, select the table, click on the filter icon in the upper right corner of the screen, and choose **New filter**, selecting the **Advanced filter** option:

![15](https://www.dropbox.com/scl/fi/9hnb7o9pkoxpz59oa9fki/15.png?rlkey=5y1ouijm1cvjmng080ychwos0&st=onnub4zv&raw=1)

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

![16](https://www.dropbox.com/scl/fi/htpvpriszu7p6wgq46io4/16.png?rlkey=o7pd3cvezjxc7a4jhbi85wre6&st=g6qxmu9a&raw=1)

Then, click OK and test your new search bar:

![17](https://www.dropbox.com/scl/fi/qszpqsywasbp2erymhlzn/17.png?rlkey=0xgz8wpb17jeeosafvwtx28en&st=kl10t991&raw=1)

If you wish, you can expand the search to more columns. To do this, just add more OR conditions and follow the same structure as the previous condition.

## Acknowledgments

- [Geiziane Oliveira](https://www.linkedin.com/in/geiziane-oliveira-0a5882110/) - SAS Mentor, whose guidance was essential for the construction of this parameter.

## References

- [Text Input linked to multiple categories in SAS VA](https://communities.sas.com/t5/SAS-Visual-Analytics/Text-Input-linked-to-multiple-categories-in-SAS-VA/m-p/784471#M15682)