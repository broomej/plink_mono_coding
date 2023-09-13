---
title: How does PLINK encode monomorphic SNPs when converting from plain text to
    binary formats?
author: "Jai Broome"
date: "2023-09-13"
output:
    html_document:
        keep_md: true
        code_folding: hide
---

[PLINK](https://www.cog-genomics.org/plink/) is a popular tool for analysis,
manipulation and summarization of genetic data.
There are two main data formats: plain text and binary. The plain text format uses a
combination of two files: a `.map` file with
[variant information](https://www.cog-genomics.org/plink/1.9/formats#map),
and a `.ped` file with
[pedigree information and genotype calls](https://www.cog-genomics.org/plink/1.9/formats#ped).
While PLINK requires each variant to be biallelic, the `.map` file does not specify
what the valid alleles are; this information is just in the `.ped` file. I had a
question about how PLINK handles the conversion of monomorphic SNPs, but couldn't
find it in the documentation. Specifically, when converting to binary format,
a `.bim` file with variant-level information contains the allele 1 and allele 2
codes, and I want to know how allele 1 and allele 2 get coded when there is only
one allele present in your dataset.

If we make a toy example where the `.map` file looks like this:


```bash
mkdir tmp
echo "1	rs123	0	1000
1	rs456	0	2000
1	rs789	0	3000
" > tmp/demo.map
cat tmp/demo.map
```

```
## 1	rs123	0	1000
## 1	rs456	0	2000
## 1	rs789	0	3000
```
and the `.ped` file looks like this:


```bash
echo "1 1 0 0 0 0 A A G G C C
1 2 0 0 0 0 G A G G C C
1 3 0 0 0 0 A A G G C C
" > tmp/demo.ped
cat tmp/demo.ped
```

```
## 1 1 0 0 0 0 A A G G C C
## 1 2 0 0 0 0 G A G G C C
## 1 3 0 0 0 0 A A G G C C
```
we have data on three samples with genotyping at three sites (the rows in the
`.map` file correspond to pairs of columns starting at column 7 in the `.ped`
file). If you look closely, you'll see that SNPs rs456 and rs789 are monomorphic;
that is, columns 9 and 10 are all `G` and columns 11 and 12 are all `C`.

Using `plink --make-bed`, we convert this to binary format.


```bash
plink --file tmp/demo --out tmp/demo --make-bed
```

```
## PLINK v1.90b7 64-bit (16 Jan 2023)             www.cog-genomics.org/plink/1.9/
## (C) 2005-2023 Shaun Purcell, Christopher Chang   GNU General Public License v3
## Logging to tmp/demo.log.
## Options in effect:
##   --file tmp/demo
##   --make-bed
##   --out tmp/demo
## 
## 24576 MB RAM detected; reserving 12288 MB for main workspace.
## Scanning .ped file... 0%32%65%98%
.ped scan complete (for binary autoconversion).
## Performing single-pass .bed write (3 variants, 3 people).
## 0%1%2%3%4%5%6%7%8%9%10%11%12%13%14%15%16%17%18%19%20%21%22%23%24%25%26%27%28%29%30%31%32%33%34%35%36%37%38%39%40%41%42%43%44%45%46%47%48%49%50%51%52%53%54%55%56%57%58%59%60%61%62%63%64%65%66%67%68%69%70%71%72%73%74%75%76%77%78%79%80%81%82%83%84%85%86%87%88%89%90%91%92%93%94%
--file: tmp/demo-temporary.bed + tmp/demo-temporary.bim +
## tmp/demo-temporary.fam written.
## 3 variants loaded from .bim file.
## 3 people (0 males, 0 females, 3 ambiguous) loaded from .fam.
## Ambiguous sex IDs written to tmp/demo.nosex .
## Using 1 thread (no multithreaded calculations invoked).
## Before main variant filters, 3 founders and 0 nonfounders present.
## Calculating allele frequencies... 0%1%2%3%4%5%6%7%8%9%10%11%12%13%14%15%16%17%18%19%20%21%22%23%24%25%26%27%28%29%30%31%32%33%34%35%36%37%38%39%40%41%42%43%44%45%46%47%48%49%50%51%52%53%54%55%56%57%58%59%60%61%62%63%64%65%66%67%68%69%70%71%72%73%74%75%76%77%78%79%80%81%82%83%84%85%86%87%88%89%90%91%92%93%94%95%96%97%98%99% done.
## Total genotyping rate is exactly 1.
## 3 variants and 3 people pass filters and QC.
## Note: No phenotypes present.
## --make-bed to tmp/demo.bed + tmp/demo.bim + tmp/demo.fam ... 0%1%2%3%4%5%6%7%8%9%10%11%12%13%14%15%16%17%18%19%20%21%22%23%24%25%26%27%28%29%30%31%32%33%34%35%36%37%38%39%40%41%42%43%44%45%46%47%48%49%50%51%52%53%54%55%56%57%58%59%60%61%62%63%64%65%66%67%68%69%70%71%72%73%74%75%76%77%78%79%80%81%82%83%84%85%86%87%88%89%90%91%92%93%94%95%96%97%98%99%done.
```

The `.bim` file contains our variant level information, and we can see that our
monomorphic SNPs have allele 1 encoded as 0 or missing:


```bash
cat tmp/demo.bim
```

```
## 1	rs123	0	1000	G	A
## 1	rs456	0	2000	0	G
## 1	rs789	0	3000	0	C
```


```bash
rm -r tmp
```

Note: see a version of this and the source files on [Github](https://github.com/broomej/plink_mono_coding)
