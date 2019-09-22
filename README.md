# zkazibwe_UNIX_Assignment
#cd into UNIX Assignment Git pull to update my Unix Assignment folder.
Made new folder called zkazibwe_UNIX_Assignment within Unix_Assignment folder on HPC-class
Cloned it into my bash working space 

ls to inspect the contents of my Unix Assignment folder. Git pull origin master, Already uptodate

##Data Inspection

Structure and dimensions of files File sizes I will use the use the du command, also with the -h

That is du -h fang_et_al_genotypes.txt snp_position.txt

6.1M fang_et_al_genotypes.txt 38K snp_position.txt

Number of lines wc -l fang_et_al_genotypes.txt snp_position.txt

2783 fang_et_al_genotypes.txt 984 snp_position.txt 3767 total The default wc command provides the number of lines, words, and bytes (characters) in these files wc fang_et_al_genotypes.txt snp_position.txt

Fang et al

2783 2744038 11051939 fang_et_al_genotypes.txt Lines 2783 Words 2744038 Characters 11051939 snp-position

984 13198 82763 snp_position.txt Lines 984 Words 13198 Characters 82763 ###Number of columns

fang_et_al_genotypes.txt

awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt

986 Columns snp_position.txt

awk -F "\t" '{print NF; exit}' snp_position.txt 15 Columns #####I used Vi to inspect the files and saw that they have headers so, i tried tail and awk and they all returned the same number of colunms for both files

tail -n +6 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'

grep -v "^#" fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'

###non-ASCII characters

use the file command file snp_position.txt snp_position.txt: ASCII English text

hexdump -c snp_position.txt | wc

Words 5174
Lines 87575
Characters 372464 file fang_et_al_genotypes.txt fang_et_al_genotypes.txt: ASCII text, with very long lines

hexdump -c fang_et_al_genotypes.txt | wc

Words 690748 Lines 11737311 Characters 49710272 Staging and commiting the Data inspection files into the README.md git add README.md

git commit -m "initial commit (README.md)

git push origin master

##Data Processing 
#i BCB546X_UNIX_ASSIGNMENT
1. Extracting Info from Files
First, I took the column headers and put them in new files. Then, I extracted the maize and teosinte data and appended them to the new files.
$ head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
$ head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
$ awk '$3 ~ /ZMMIL|ZMMLR|ZMMMR/ { print $0}' fang_et_al_genotypes.txt >> maize_genotypes.txt
$ awk '$3 ~ /ZMPBA|ZMPIL|ZMPJA/ { print $0}' fang_et_al_genotypes.txt >> teosinte_genotypes.txt
2. Separate Maize and Teosinte genotypes
$ grep -E "(ZMMIL|ZMMLR|ZMMMR|Group)" fang_et_al_genotypes.txt | cut -f 1,4-986 |awk -f transpose.awk  > maize_genotype.txt  
$ sed 's/Sample_ID/SNP_ID/' maize_genotype.txt | sort –k1,1 > maize_sgenotype.txt 
$ grep -E "(ZMPBA|ZMPIL|ZMPJA|Group)" fang_et_al_genotypes.txt | cut -f 1,4-986 |awk -f transpose.awk > teosinte_genotype.txt  
$ sed 's/Sample_ID/SNP_ID/' teosinte_genotype.txt | sort –k1,1 > teosinte_sgenotype.txt

##### grep command is to print out lines containing "ZMMIL", "ZMMLR" "ZMMMR" and "Group" ("ZMPBA", "ZMPIL" "ZMPJA" and "Group" for teosinte), which are maize samples and the header;cut commands is to remove the 2 columns we don't need;awk command is to transpose the table so that it has the same data frame with snp_infor.txt and we can join them later;sed command is change the header in genotype file to the same with SNP file, so that we can join them later;sort command is to sort by the 1st column;

new files saved as maize_sgenotype.txt and teosinte_sgenotype.txt OR maize_genotype.txt and teosinte_genotype.txt

I checked to make sure my extractions worked by looking at the number of lines, words and characters in each file
[zkazibwe@hpc-class My-Unix_AS]$ wc maize_genotypes.txt teosinte_genotypes.txt
    1574  1551964  6250961 maize_genotypes.txt
     976   962336  3884185 teosinte_genotypes.txt
    2550  2514300 10135146 total
 I also checked to see that each file had only the groups that I wanted by cutting the groups from column 3, sorting them and running the command unique and getting a count.
[zkazibwe@hpc-class My-Unix_AS]$ cut -f 3 maize_genotypes.txt | sort | uniq -c
      1 Group
    290 ZMMIL
   1256 ZMMLR
     27 ZMMMR
[zkazibwe@hpc-class My-Unix_AS]$ cut -f 3 teosinte_genotypes.txt | sort | uniq -c
      1 Group
    900 ZMPBA
     41 ZMPIL
     34 ZMPJA
[zkazibwe@hpc-class My-Unix_AS]$

3. Combine genotype with SNP position
[zkazibwe@hpc-class My-Unix_AS]$ join -1 1 -2 1 -t $'\t' snp_infor.txt teosinte_sgenotype.txt > teosinte_joint.txt
[zkazibwe@hpc-class My-Unix_AS]$ ^C
[zkazibwe@hpc-class My-Unix_AS]$ join -1 1 -2 1 -t $'\t' snp_infor.txt maize_sgenotype.txt > maize_joint.txt

4. Separate SNPs based on Chromosome
[zkazibwe@hpc-class My-Unix_AS]$ for i in {1..10} ; do (awk '$1 ~ /SNP/' maize_joint.txt && awk '$2 == '$i'&& $3 != "multiple"' maize_joint.txt) > maize_chr$i.txt ; done
[zkazibwe@hpc-class My-Unix_AS]$ (awk '$1 ~ /SNP/' maize_joint.txt && awk '$3 == "unknown"' maize_joint.txt )> maize_unknown.txt
[zkazibwe@hpc-class My-Unix_AS]$ (awk '$1 ~ /SNP/' maize_joint.txt && awk '$2 == "multiple" || $3 == "multiple"' maize_joint.txt )> maize_multiple.txt
[zkazibwe@hpc-class My-Unix_AS]$ for i in {1..10} ; do (awk '$1 ~ /SNP/' teosinte_joint.txt && awk '$2 == '$i' && $3 != "multiple"' teosinte_joint.txt) > teosinte_chr$i.txt ; done
[zkazibwe@hpc-class My-Unix_AS]$ (awk '$1 ~ /SNP/' teosinte_joint.txt && awk '$3 == "unknown"' teosinte_joint.txt )> teosinte_unknown.txt
[zkazibwe@hpc-class My-Unix_AS]$ (awk '$1 ~ /SNP/' teosinte_joint.txt && awk '$2 == "multiple" || $3 == "multiple" ' teosinte_joint.txt) > teosinte_multiple.txt
[zkazibwe@hpc-class My-Unix_AS]$

for command is used to do the loop for 10 chromosomes;
awk command is used to first print out header and then print out the records which feature pattern that field2 is the same with value of i;
new file saved as maize_chr$i.txt and teosinte_Chr$i.txt, in which i is the number of chromosome.

5. Sort SNPs based on position
[zkazibwe@hpc-class My-Unix_AS]$ for i in {1..10}; do (head -n 1 maize_chr$i.txt && tail -n +2 maize_chr$i.txt | sort -k3,3n )> incr_maize_chr$i.txt ; done
[zkazibwe@hpc-class My-Unix_AS]$ for i in {1..10}; do (head -n 1 maize_chr$i.txt && tail -n +2 maize_chr$i.txt | sort -k3r,3n ) | sed 's/?/-/g' > decr_maize_chr$i.txt ; done
[zkazibwe@hpc-class My-Unix_AS]$ for i in {1..10}; do (head -n 1 teosinte_chr$i.txt && tail -n +2 teosinte_chr$i.txt | sort -k3,3n )> incr_teosinte_chr$i.txt ; done
[zkazibwe@hpc-class My-Unix_AS]$ for i in {1..10}; do (head -n 1 teosinte_chr$i.txt && tail -n +2 teosinte_chr$i.txt | sort -k3r,3n ) | sed 's/?/-/g' > decr_teosinte_chr$i.txt ; done
[zkazibwe@hpc-class My-Unix_AS]$

for command is used to do the loop for 10 files;
head and tail command are used to keep the header at top and then do the sorting on the other lines;
sort command is the key command here. We do the sorting based on the 3rd column, which shows the position of SNP. -k3 means the result will be listed increasingly and -k3r means the reverse. 3n means they are treated as numeric.
sed command is used to switch the "?", which is encoded to be missing data, to "-";
new files are saves as incr_maize_chr$i.txt / incr_teosinte_chr$i.txt if their position is listed increasingly; decr_maize_chr$i.txt / decr_teosinte_chr$i.txt if their position is listed decreasingly.

6. Preparing all my files to send to git repository
$ mkdir maize teosinte process_file
$ mv decr_maize* incr_maize* maize_m* maize_u* ./maize
$ mv decr_teosinte* incr_teosinte* teosinte_m* teosinte_u* ./teosinte
$ mv * ./process_file   

mkdir command is to make sub-directories under this directory;
mv commands are to move files to different directories;
Here we moved 12 maize-related files to ./MAIZE_FILES directory; 12 teosinte-related files to ./TEOSINTE_FILES directory; the other files generated during the above process and also the original 3 files are moved to ./OTHER_FILES
join command is to combine the two file based on the 1st column of snp_infor.txt and the 1st column of maize_transposed_genotype.txt;
new file saved as maize_joint.txt

