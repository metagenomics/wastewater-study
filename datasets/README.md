
The dataset Accession IDs of this folder can be reproduced by executing the following lines:

```
grep -v Negative samples.tsv | sort -uk 3,3  | cut -f 18 | head -n -1 > used_samples.tsv

sed  -i '1i ACCESSION' used_samples.tsv

cat used_samples.tsv | head -n -1 | split -l 50 -d - dataset_

sed  -i '1i ACCESSION' dataset_*
```
