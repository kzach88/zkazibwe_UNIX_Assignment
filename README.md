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
Made a new repository called  Cloned it into my bash working space

https://github.com/ensamba/Unix_Assignment.git

######Step 1

Extracting specific Maize and Teosinte SNP_IDs from the fang et al genotypes

Maize

grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > Maize_file

However, it lacks the headers, so to put back the headers I used the head command head -n1 fang_et_al_genotypes.txt > header

Then to join the maize file with the header so as to restore the headers, I used the catcommand cat header Maize_file1 > Maize_header

Teosinte grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > Teosinte_file head -n1 fang_et_al_genotypes.txt > header

cat header Maize_file1 > Teosinte_header

Now I have the files Maize_header and Teosinte_header.

Stage and commit new files git add Maize_header Teosinte_header git commit -m "initial commit (Maize_header Teosinte_header)" git push origin master

######Step 2: The next step is to transponse, sort and join them together. #####Step 2a: Transposing the files so that rows become columns

Maize SNP files from the Maize_header file that I created awk -f transpose.awk Maize_header > transponsed_Maize_genotypes.txt Just a quick inspection to view file contents sort -k1,1 -k2,2n -k3,3n transponsed_Maize_genotypes.txt | cut -f1-10 | head Sort the file so that it's ready for joining based on c Sorted basing on column 1

sort -k1,1 transponsed_Maize_genotypes.txt > sorted_Maize_genotypes.txt Teosinte SNP files from the Teosinte_header file that I created awk -f transpose.awk Teosinte_header > transponsed_teosinte_genotypes.txt Sorting by column 1 sort -k1,1 transponsed_teosinte_genotypes.txt > sorted_Teosinte_genotypes.txt

Stage and commit new files

$ git add sorted_Teosinte_genotypes.txt sorted_Maize_genotypes.txt $ git commit -m "initial commit (sorted_Teosinte_genotypes.txt sorted_Maize_genotypes.txt)" $ git push origin master

####Note Before joining each of these sorted files to the snp_position.txt file, I first sorted it by the common column 1, then cut out the columns I needeed. sort -k1,1 snp_position.txt | cut -f 1-5 > sorted_snp_position.txt

####Step 2b:Joining the files ######Used the join command

maize group IDs to the snp_sorted file. join -t $'\t' -1 1 -2 1 sorted_snp_position.txt sorted_Maize_genotypes.txt > Maize_SNP_joined.txt Just a quick check to be sure that I have the correct number of columns awk -F "\t" '{print NF; exit}' Maize_SNP_joined.txt 1578

Inspecting the file to ensure that join was complete wc -l sorted_Maize_genotypes.txt sorted_snp_position.txt Maize_snp_joined.txt

986 sorted_Maize_genotypes.txt 984 sorted_snp_position.txt 983 Maize_snp_joined.txt 2953 total Teosinte group IDs to the snp-sorted files. join -t $'\t' -1 1 -2 1 sorted_snp_position.txt sorted_Teosinte_genotypes.txt > Teosinte_SNP_joined.txt Inspecting the file to ensure that join was complete wc -l sorted_Teosinte_genotypes.txt sorted_snp_position.txt Teosinte_snp_joined.txt

986 sorted_Teosinte_genotypes.txt 984 sorted_snp_position.txt 983 Teosinte_snp_joined.txt ####Step 2c: Sorting the joined files by chromosome numerically for processing Maize

sort -k3,3n Maize_SNP_joined.txt > Maize_sorted_chr.txt Teosinte

sort -k3,3n Teosinte_SNP_joined.txt > Teosinte_sorted_chr.txt

Stage and commit new files

$ git add Maize_sorted_chr.txt Teosinte_sorted_chr.txt $ git commit -m "initial commit (Maize_sorted_chr.txt Teosinte_sorted_chr.txt)" $ git push origin master

####Step 2d: Organizing files Made two separate folders to organize Maize files alone and the same to Teosinte files mkdir Maize_data_files Teosinte_data_files

cp Maize_header Maize_SNP_joined.txt Maize_SNPs_multiple Maize_sorted_chr.txt Maize_substituted.txt Maize_substituted_dash.txt sorted_Maize_genotypes.txt Maize_data_files/

cp Teosinte_header Teosinte_SNP_joined.txt Teosinte_sorted_chr.txt Teosinte_substituted_dash.txt Teosinte_substituted.txt sorted_Teosinte_genotypes.txt Teosinte_data_files/

Stage and commit new files git add Maize_data_files/ Teosinte_data_files/

git commit -m "initial commit (Maize_data_files Teosinte_data_files)"

$ git push origin master

######Step 3 ###Using the sed program insert special characters in missing data set as required by the assignment.

By this I substituted tab delimiters for the sorted files into ? for both Maize and Teosinte. sed 's//?/g' Maize_sorted_chr.txt > Maize_substituted.txt

sed 's//?/g' Teosinte_sorted_chr.txt > Teosinte_substituted.txt

Then for the inverted file, I substituted the ? into - as follows

sed 's/?/-/g' Maize_sorted_chr.txt > Maize_substituted_dash.txt sed 's/?/-/g' Teosinte_sorted_chr.txt > Teosinte_substituted_dash.txt

######Step 4 ###Use the loop command on the seeded files and out put contents of specific chromosomes 1 to 10 Maize

for i in {1..10}; do awk '$3=='$i'' Maize_substituted.txt > chr"$i"_Maize_substituted.txt; done

for i in {1..10}; do awk '$3=='$i'' Maize_substituted_dash.txt > chr"$i"_Maize_substituted_dash.txt; done Teosinte

for i in {1..10}; do awk '$3=='$i'' Teosinte_substituted_dash.txt > chr"$i"_Teosinte_substituted_dash.txt; done

for i in {1..10}; do awk '$3=='$i'' Teosinte_substituted.txt > chr"$i"_Teosinte_substituted.txt; done

#Questions ######1. 10 files (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol:?

Maize Made a directory to store my output files mkdir chr#_Maize_SNPs_Increase sort -k4,4n chr#_Maize_substituted.txt > chr#_Maize_SNPs_increase.txt where # represents chromosomes 1-10

Alternatively, I tried verifying it by by another set of command which didn't seem to be working fine for i in {1..10}; do sort '$4=='$i'' Maize_substituted.txt > chr"$i"_Maize_SNPs_increase.txt; done

Staging and Commiting new files git add chr#_Maize_SNPs_Increase/ git commit -m "initial commit (chr#_Maize_SNPs_Increase)" git push origin master

Teosinte mkdir chr#_Teosinte_SNPs_Increase

sort -k4,4n chr#_Teosinte_substituted.txt > chr#_Teosinte_SNPs_increase.txt

Staging and commiting new files git add chr#_Teosinte_SNPs_Increase/

git commit -m "initial commit (chr#_Teosinte_SNPs_Increase)"

git push origin master

######2. 10 files (1 for each chromosome) with SNPs ordered based on decreasing position values and with missing data encoded by this symbol: -

Maize mkdir chr#_Maize_SNPs_decrease

sort -k4,4nr chr#_Maize_substituted_dash.txt > chr#_Maize_SNPs_decrease.txt

Teosinte mkdir chr#_Teosinte_SNPs_decrease.txt

sort -k4,4nr chr#_Teosinte_substituted_dash.txt > chr#_Teosinte_SNPs_decrease.txt

Staging and commiting new files git add chr#_Teosinte_SNPs_decrease.txt/ chr#_Maize_SNPs_decrease.txt/

git commit -m "initial commit (chr#_Teosinte_SNPs_decrease.txt/ chr#_Maize_SNPs_decrease.txt/)"

git push origin master

######1 file with all SNPs with unknown positions in the genome (these need not be ordered in any particularway)

Maize grep "unknown" Maize_sorted_chr.txt > Maize_unknown_Snps.txt

Teosinte grep "unknown" Teosinte_sorted_chr.txt > Teosinte_unknown_Snps.txt

Staging and commiting files git add Maize_unknown_Snps.txt Teosinte_unknown_Snps.txt

git commit -m "initial commit(Maize_unknown_Snps.txt Teosinte_unknown_Snps.txt)"

git push origin master

######1 file with all SNPs with multiple positions in the genome (these need not be ordered in any particular way)

Maize grep "multiple" Maize_sorted_chr.txt > Maize_multiple_Snps.txt

Teosinte grep "multiple" Teosinte_sorted_chr.txt > Teosinte_multiple_Snps.txt

Staging and commiting files

git add Maize_multiple_Snps.txt Teosinte_multiple_Snps.txt

git commit -m "initial commit(Maize_multiple_Snps.txt Teosinte_multiple_Snps.txt)"

git push origin master

#A total of 44 files were produced. These were organized into separate folders for both Maize and the Teosinte.

Results:

20 files (1 for each chromosome), for maize and teosinte with SNPs ordered based on increasing position values and with missing data encoded by? chr#_Teosinte_SNPs_increase.txt Chr#_Maize_SNPs_increase.txt

20 files (1 for each chromosome), for maize and teosinte with SNPs ordered based on decreasing position values and with missing data encoded by - chr#_Maize_SNPs_decrease.txt chr#_Teosinte_SNPs_decrease.tx

1 file with all SNPs with multiple positions in the genome for both Teosinte and Maize Maize_multiple_Snps.txt Teosinte_multiple_Snps.txt

1 file with all SNPs with unknown positions in the genome for both Teosinte and Maize Teosinte_unknown_Snps.txt Maize_unknown_Snps.txt
