
The dataset Accession IDs of this folder can be reproduced by executing the following lines:

```
grep -v Negative 41467_2022_34312_MOESM3_ESM.tsv | sort -uk 3,3  | cut -f 18 | head -n -1 | split -l 50 -d - dataset_
sed  -i '1i ACCESSION' dataset_*
```
