# A sourmash tutorial

[sourmash](http://sourmash.readthedocs.io/en/latest/) is our lab's
implementation of an ultra-fast lightweight approach to
nucleotide-level search and comparison, called MinHash.

You can read some background about MinHash sketches in this paper:
[Mash: fast genome and metagenome distance estimation using MinHash. Ondov BD, Treangen TJ, Melsted P, Mallonee AB, Bergman NH, Koren S, Phillippy AM. Genome Biol. 2016 Jun 20;17(1):132. doi: 10.1186/s13059-016-0997-x.](http://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-0997-x)

## Installing sourmash

To install sourmash, run:

```
pip install https://github.com/dib-lab/sourmash/archive/2017-ucsc-metagenome.zip
```

(Note, we are installing from [a development branch](https://github.com/dib-lab/sourmash/pull/188); many of the features below are not part of an official sourmash release yet.  They should be included in sourmash 2.0.)

## Fingerprint reads

Use case: how much do two (or more!) unassembled metagenomes resemble each
other?

Compute a scaled MinHash fingerprint from our reads:

```
mkdir ~/sourmash
cd ~/sourmash

sourmash compute --scaled 10000 ~/mapping/SRR*.pe.fq -k 21,31
```

Now, compare the two files at k=21:

```
sourmash compare SRR*.sig -k 21
```

or k=31:
```
sourmash compare SRR*.sig -k 31
```

## Compare reads to assemblies

Use case: how much of the read content is contained in the assembly?

Fingerprint the assembly:

```
sourmash compute --scaled 10000 -k 21,31 ~/mapping/subset_assembly.fa
```

and now evaluate *containment*, that is, what fraction of the read content is
contained in the assembly:

```
sourmash search -k 21 SRR1976948.abundtrim.subset.pe.fq.sig \
    subset_assembly.fa.sig  --containment
```

and you should see:

```
1 matches; showing 3:
         /home/titus/mapping/subset_assembly.fa          0.573   subset_assembly.fa.sig
```


Try the reverse - why is it bigger?
         
```
sourmash search -k 21 subset_assembly.fa.sig \
    SRR1976948.abundtrim.subset.pe.fq.sig --threshold=0.0 --containment
```

what do you get if you do this with the other set of reads?

## Compare many samples

Adjust plotting (this is a bug in sourmash :) --
```
echo 'backend : Agg' > matplotlibrc
```

Do a comparison:

```
sourmash compare SRR*.sig subset*.sig -o comparison
```

and then plot:

```
sourmash plot --pdf comparison
```

which will produce a file `comparison.matrix.pdf` and `comparison.dendro.pdf`
that you can grab view your Jupyter Notebook console.

## What's in my metagenome?

Download and unpack the k=21 RefSeq index described in
[CTB's blog post](http://ivory.idyll.org/blog/2016-sourmash-sbt-more.html):

```
curl -O http://spacegraphcats.ucdavis.edu.s3.amazonaws.com/microbe-sbt-k21-2016-11-27.tar.gz
tar xzf microbe-sbt-k21-2016-11-27.tar.gz
```

This produces a file `microbes.sbt.json` and a whole bunch of hidden
files in the directory `.sbt.microbes`.  This is an index of about 60,000
microbial genomes from RefSeq.

Next, run the 'gather' command to see what's in there --
```
sourmash sbt_gather -k 21 microbes subset_assembly.fa.sig
```

and you should get:

```
Final composition (sorted by percent of original query):

p_query p_genome
  2.4    14.3   NC_010003.1 Petrotoga mobilis SJ95, complete genome
  0.5     1.6   NC_017934.1 Mesotoga prima MesG1.Ag.4.2, complete genome
  3.0%          (percent of query identified)
```

If you go to
[the SRA information](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA278302)
for this project, you'll see that Petrotoga and Mesotoga are both in
there - yay!  Of course, in this case we're looking at largely unknown
critters (3% at most is in genbank!) so we wouldn't expect many
matches.
