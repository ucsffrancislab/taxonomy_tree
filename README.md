# taxonomy_tree

So we wanted to able to access the taxonomy tree for given accession ids.

A python package called taxadb can do this.

https://taxadb.readthedocs.io/

pip install --upgrade --user mysql taxadb


It creates a database from some of the following ...

ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz

ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/nucl_gb.accession2taxid.gz

ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/nucl_wgs.accession2taxid.gz

ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.gz

... depending on given options.


However, it requires python 3.

We are stuck on a system that won't / can't upgrade to python 3 so can't use taxadb.

But I do have access to a different system that does have python 3 and taxadb.

So I can create the taxadb database and then transfer it over and access it without taxadb.




I have found that I only need the "gb" database.
The smaller the better to speed up searching.





mysql 

CREATE DATABASE taxadb_gb;



"taxa", "full", "nucl", "prot", "gb", or "wgs"

full gets prot, nucl_gb and nucl_wgs
nucl gets nucl_gb and nucl_wgs
prot gets prot
gb gets nucl_gb
wgs gets nucl_wgs
No idea what "taxa" gets.

gb seems to be enough for me




taxadb download --outdir taxadb_gb --type gb



This fails for me because `taxadb create` expect the mysql socket to bin /tmp/mysql.sock so I link it.

`ln -s /var/run/mysqld/mysqld.sock /tmp/mysql.sock`

Not sure how to rectify if don't have root privileges.


taxadb create -i taxadb_gb --dbname taxadb_gb --division gb --dbtype mysql --user root --password "" --fast

This can take a while. Only use the --fast option with an empty, newly created database as it doesn't check.
Seriously, hours. About 6 for the gb database.







The resulting database contains 2 tables; accession and taxa.

```
MariaDB [taxadb_full]> describe accession;
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| id        | int(11)      | NO   | PRI | NULL    | auto_increment |
| taxid_id  | int(11)      | NO   | MUL | NULL    |                |
| accession | varchar(255) | NO   | UNI | NULL    |                |
+-----------+--------------+------+-----+---------+----------------+
3 rows in set (0.002 sec)

MariaDB [taxadb_full]> describe taxa;
+---------------+--------------+------+-----+---------+-------+
| Field         | Type         | Null | Key | Default | Extra |
+---------------+--------------+------+-----+---------+-------+
| ncbi_taxid    | int(11)      | NO   | PRI | NULL    |       |
| parent_taxid  | int(11)      | NO   |     | NULL    |       |
| tax_name      | varchar(255) | NO   |     | NULL    |       |
| lineage_level | varchar(255) | NO   |     | NULL    |       |
+---------------+--------------+------+-----+---------+-------+
```


```
mysql taxadb_gb -e "select count(1) from accession where accession IN ( 'NC_006273', 'CP048030','CP031704','NC_001638','NC_023091','NC_039755','NC_039754','NC_033969','NC_027859','NC_028331','NC_042478','NC_009681','NC_002387','NC_000926','NC_041490' )"
+----------+
| count(1) |
+----------+
|       15 |
+----------+

mysql taxadb_gb -e "select a.accession, t1.ncbi_taxid, t1.tax_name, t1.lineage_level, t2.ncbi_taxid, t2.tax_name, t2.lineage_level, t3.ncbi_taxid, t3.tax_name, t3.lineage_level, t4.ncbi_taxid, t4.tax_name, t4.lineage_level, t5.ncbi_taxid, t5.tax_name, t5.lineage_level , t6.ncbi_taxid, t6.tax_name, t6.lineage_level , t7.ncbi_taxid, t7.tax_name, t7.lineage_level , t8.ncbi_taxid, t8.tax_name, t8.lineage_level, t9.ncbi_taxid, t9.tax_name, t9.lineage_level, t10.ncbi_taxid, t10.tax_name, t10.lineage_level from accession a join taxa t1 on t1.ncbi_taxid = a.taxid_id join taxa t2 on t2.ncbi_taxid = t1.parent_taxid join taxa t3 on t3.ncbi_taxid = t2.parent_taxid join taxa t4 on t4.ncbi_taxid = t3.parent_taxid join taxa t5 on t5.ncbi_taxid = t4.parent_taxid join taxa t6 on t6.ncbi_taxid = t5.parent_taxid join taxa t7 on t7.ncbi_taxid = t6.parent_taxid join taxa t8 on t8.ncbi_taxid = t7.parent_taxid join taxa t9 on t9.ncbi_taxid = t8.parent_taxid join taxa t10 on t10.ncbi_taxid = t9.parent_taxid where a.accession IN ( 'NC_006273', 'CP048030','CP031704','NC_001638','NC_023091','NC_039755','NC_039754','NC_033969','NC_027859','NC_028331','NC_042478','NC_009681','NC_002387','NC_000926','NC_041490' )"
+-----------+------------+---------------------------------+---------------+------------+----------------------------------+---------------+------------+---------------------+---------------+------------+-------------------+---------------+------------+---------------+---------------+------------+---------------------+---------------+------------+--------------------+---------------+------------+---------------+---------------+------------+--------------------+---------------+------------+--------------------+---------------+
| accession | ncbi_taxid | tax_name                        | lineage_level | ncbi_taxid | tax_name                         | lineage_level | ncbi_taxid | tax_name            | lineage_level | ncbi_taxid | tax_name          | lineage_level | ncbi_taxid | tax_name      | lineage_level | ncbi_taxid | tax_name            | lineage_level | ncbi_taxid | tax_name           | lineage_level | ncbi_taxid | tax_name      | lineage_level | ncbi_taxid | tax_name           | lineage_level | ncbi_taxid | tax_name           | lineage_level |
+-----------+------------+---------------------------------+---------------+------------+----------------------------------+---------------+------------+---------------------+---------------+------------+-------------------+---------------+------------+---------------+---------------+------------+---------------------+---------------+------------+--------------------+---------------+------------+---------------+---------------+------------+--------------------+---------------+------------+--------------------+---------------+
| CP031704  |    2303331 | Solimonas sp. K1W22B-7          | species       |    2637139 | unclassified Solimonas           | no rank       |     413435 | Solimonas           | genus         |     568386 | Sinobacteraceae   | family        |    1775403 | Nevskiales    | order         |       1236 | Gammaproteobacteria | class         |       1224 | Proteobacteria     | phylum        |          2 | Bacteria      | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root               | no rank       |
| CP048030  |    2698684 | Sinimarinibacterium sp. NLF-5-8 | species       |    2621506 | unclassified Sinimarinibacterium | no rank       |    1861863 | Sinimarinibacterium | genus         |     568386 | Sinobacteraceae   | family        |    1775403 | Nevskiales    | order         |       1236 | Gammaproteobacteria | class         |       1224 | Proteobacteria     | phylum        |          2 | Bacteria      | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root               | no rank       |
| NC_000926 |      55529 | Guillardia theta                | species       |      55528 | Guillardia                       | genus         |     589343 | Geminigeraceae      | family        |     589342 | Pyrenomonadales   | order         |       3027 | Cryptophyceae | class         |       2759 | Eukaryota           | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root          | no rank       |          1 | root               | no rank       |          1 | root               | no rank       |
| NC_001638 |       3055 | Chlamydomonas reinhardtii       | species       |       3052 | Chlamydomonas                    | genus         |       3051 | Chlamydomonadaceae  | family        |       3042 | Chlamydomonadales | order         |       3166 | Chlorophyceae | class         |    2692248 | core chlorophytes   | no rank       |       3041 | Chlorophyta        | phylum        |      33090 | Viridiplantae | kingdom       |       2759 | Eukaryota          | superkingdom  |     131567 | cellular organisms | no rank       |
| NC_002387 |       4787 | Phytophthora infestans          | species       |       4783 | Phytophthora                     | genus         |       4777 | Peronosporaceae     | family        |       4776 | Peronosporales    | order         |       4762 | Oomycetes     | class         |      33634 | Stramenopiles       | no rank       |    2698737 | Sar                | no rank       |       2759 | Eukaryota     | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root               | no rank       |
| NC_006273 |      10359 | Human betaherpesvirus 5         | species       |      10358 | Cytomegalovirus                  | genus         |      10357 | Betaherpesvirinae   | subfamily     |      10292 | Herpesviridae     | family        |     548681 | Herpesvirales | order         |      10239 | Viruses             | superkingdom  |          1 | root               | no rank       |          1 | root          | no rank       |          1 | root               | no rank       |          1 | root               | no rank       |
| NC_009681 |      34116 | Pleurastrum terricola           | species       |      34114 | Pleurastrum                      | genus         |    2682564 | Pleurastraceae      | family        |       3042 | Chlamydomonadales | order         |       3166 | Chlorophyceae | class         |    2692248 | core chlorophytes   | no rank       |       3041 | Chlorophyta        | phylum        |      33090 | Viridiplantae | kingdom       |       2759 | Eukaryota          | superkingdom  |     131567 | cellular organisms | no rank       |
| NC_023091 |     353565 | Polytomella magna               | species       |       3049 | Polytomella                      | genus         |       3051 | Chlamydomonadaceae  | family        |       3042 | Chlamydomonadales | order         |       3166 | Chlorophyceae | class         |    2692248 | core chlorophytes   | no rank       |       3041 | Chlorophyta        | phylum        |      33090 | Viridiplantae | kingdom       |       2759 | Eukaryota          | superkingdom  |     131567 | cellular organisms | no rank       |
| NC_027859 |     143453 | Pseudoperonospora cubensis      | species       |     143452 | Pseudoperonospora                | genus         |       4777 | Peronosporaceae     | family        |       4776 | Peronosporales    | order         |       4762 | Oomycetes     | class         |      33634 | Stramenopiles       | no rank       |    2698737 | Sar                | no rank       |       2759 | Eukaryota     | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root               | no rank       |
| NC_028331 |     230439 | Peronospora tabacina            | species       |      70742 | Peronospora                      | genus         |       4777 | Peronosporaceae     | family        |       4776 | Peronosporales    | order         |       4762 | Oomycetes     | class         |      33634 | Stramenopiles       | no rank       |    2698737 | Sar                | no rank       |       2759 | Eukaryota     | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root               | no rank       |
| NC_033969 |      51707 | Yamagishiella unicocca          | species       |      79754 | Yamagishiella                    | genus         |       3065 | Volvocaceae         | family        |       3042 | Chlamydomonadales | order         |       3166 | Chlorophyceae | class         |    2692248 | core chlorophytes   | no rank       |       3041 | Chlorophyta        | phylum        |      33090 | Viridiplantae | kingdom       |       2759 | Eukaryota          | superkingdom  |     131567 | cellular organisms | no rank       |
| NC_039754 |      51707 | Yamagishiella unicocca          | species       |      79754 | Yamagishiella                    | genus         |       3065 | Volvocaceae         | family        |       3042 | Chlamydomonadales | order         |       3166 | Chlorophyceae | class         |    2692248 | core chlorophytes   | no rank       |       3041 | Chlorophyta        | phylum        |      33090 | Viridiplantae | kingdom       |       2759 | Eukaryota          | superkingdom  |     131567 | cellular organisms | no rank       |
| NC_039755 |      51714 | Volvox africanus                | species       |       3066 | Volvox                           | genus         |       3065 | Volvocaceae         | family        |       3042 | Chlamydomonadales | order         |       3166 | Chlorophyceae | class         |    2692248 | core chlorophytes   | no rank       |       3041 | Chlorophyta        | phylum        |      33090 | Viridiplantae | kingdom       |       2759 | Eukaryota          | superkingdom  |     131567 | cellular organisms | no rank       |
| NC_041490 |      55529 | Guillardia theta                | species       |      55528 | Guillardia                       | genus         |     589343 | Geminigeraceae      | family        |     589342 | Pyrenomonadales   | order         |       3027 | Cryptophyceae | class         |       2759 | Eukaryota           | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root          | no rank       |          1 | root               | no rank       |          1 | root               | no rank       |
| NC_042478 |     162120 | Pseudoperonospora humuli        | species       |     143452 | Pseudoperonospora                | genus         |       4777 | Peronosporaceae     | family        |       4776 | Peronosporales    | order         |       4762 | Oomycetes     | class         |      33634 | Stramenopiles       | no rank       |    2698737 | Sar                | no rank       |       2759 | Eukaryota     | superkingdom  |     131567 | cellular organisms | no rank       |          1 | root               | no rank       |
+-----------+------------+---------------------------------+---------------+------------+----------------------------------+---------------+------------+---------------------+---------------+------------+-------------------+---------------+------------+---------------+---------------+------------+---------------------+---------------+------------+--------------------+---------------+------------+---------------+---------------+------------+--------------------+---------------+------------+--------------------+---------------+
```





Is this tree correct? Seems so.

Is this the best way to do this? Unlikely.




```
mysqldump taxadb_gb | gzip > taxadb_gb.sql.gz
```

I've noticed that when loading the results into other mysql systems, I get a failure.

'Specified key was too long; max key length is 767 bytes'

This is fixed by shrinking the size of the accession column.
It is currently set to 255, but it only needs to be 18.

```
select max(length(accession)) from accession;
+------------------------+
| max(length(accession)) |
+------------------------+
|                     18 |
+------------------------+
```


Change the column length before dumping ...

```
MariaDB [taxadb_gb]> ALTER TABLE accession MODIFY accession VARCHAR(20);
Query OK, 250187841 rows affected (1 hour 46 min 8.727 sec)
Records: 250187841  Duplicates: 0  Warnings: 0
```

```
mysqldump taxadb_gb | gzip > taxadb_gb.sql.gz
```

or during dumping ...

```
mysqldump taxadb_gb | sed 's/`accession` varchar(255)/`accession` varchar(20)/' | gzip > taxadb_gb.sql.gz
```


Should change this in the taxadb code





So, not only does our cluster not support python3, it does not support mysql!

Creating sqlite database.

```
taxadb create -i taxadb_gb --dbname taxadb_gb.sqlite --division gb --dbtype sqlite --fast
```

```
module load sqlite

sqlite3 /francislab/data1/refs/taxadb_gb.sqlite  "select a.accession, t1.ncbi_taxid, t1.tax_name, t1.lineage_level, t2.ncbi_taxid, t2.tax_name, t2.lineage_level, t3.ncbi_taxid, t3.tax_name, t3.lineage_level, t4.ncbi_taxid, t4.tax_name, t4.lineage_level, t5.ncbi_taxid, t5.tax_name, t5.lineage_level, t6.ncbi_taxid, t6.tax_name, t6.lineage_level from accession a join taxa t1 on t1.ncbi_taxid = a.taxid_id join taxa t2 on t2.ncbi_taxid = t1.parent_taxid join taxa t3 on t3.ncbi_taxid = t2.parent_taxid join taxa t4 on t4.ncbi_taxid = t3.parent_taxid join taxa t5 on t5.ncbi_taxid = t4.parent_taxid join taxa t6 on t6.ncbi_taxid = t5.parent_taxid where a.accession = 'NC_006273'"
```




