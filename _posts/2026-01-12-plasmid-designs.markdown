Suppose we want to express human insulin in E. coli, but we want to tune the genetic sequences because the ones from the
human genome may not work as well in a microbe.  A good way to do this experimentally is to try multiple variants of the
promoter and ribosome binding site and see which combinations work best.  Suppose we also want to track and share which
variants we're using, in an easily accessible way.

In this post, we're going to build on our last post about pooled designs, and show how to use gen to represent a
combinatorial plasmid design.  Specifically, we will put together a library of expression plasmids for E. coli, with the
ultimate goal of finding the combination of promoter and ribosome binding site that result in the highest expression of
an insulin precursor peptide.  This example will be slightly different from the previous one, because we'll be cloning a
design into an existing plasmid, and we'll keep the reference part of the sequence in addition to the cloned library.
Also, the previous post had "dummy" data, and the sequences in this post are more realistic.

As always, we initialize the repo first:

```
> gen init
Gen repository initialized.
```

Then we import the reference for the plasmid we'll be modifying.

```
> gen import fasta puc19.fa
Fasta import called
Parsing Fasta
[00:00:00] . . . . . . . . . . . . . . . . . . . . 🧬       1         Entries Processed.
[00:00:00]                                                            Saving operation
Fasta imported.
```

Then, we update with the library.  We're putting the result into a new "virtual" sample called "design".  The path name
is just the region identifier for the plasmid fasta.  The cloning region starts at locus 106 (inclusive) and ends at
locus 539 (exclusive).  The parts are in parts.fa and the design.csv file lists the buckets of parts to use.  Gen will
create a combinatorial design where all options for each bucket (CSV column in design.csv) are combined with all options
for the other buckets. In this example, we have 3 buckets, with respectively 5, 2, and 1 part options.  This results in
10 possible outcomes (5 * 2 * 1).

```
> gen update library --new-sample design --path-name M77789.2 --start 106 --end 539 --library design.csv --parts parts.fa
Update with library called
Updated with library file: design.csv
```

Then we can view it on the command line.

```
gen view
```

Navigating to the design "sample", we can see the sequence graph we've created:

![Initial screen](../../../../../post-files/plasmid-design/design-view.png)

We've also [pushed the repo to genhub](https://www.genhub.bio/repos/david-genhub-bio/plasmid-design-example), and the
sequence graph [can be viewed
there](https://www.genhub.bio/repos/david-genhub-bio/plasmid-design-example/database?database=.gen%2Fdefault.db&collection=default&sample=design&blockgroup=design&branch=main&operation=)
as well.  Also, the files we used in this example are there, and you can download them if you want to take a look.

Unlike our previous example with importing a library, when updating, the original vector's sequence is preserved and
included in the sequence graph.  This makes it clearer what part of the reference sequence is being replaced.

We have some further directions we want to go with pooled designs, but we'll stop here for now.  If you make plasmid
designs for your work, and want to share what you're doing with a colleague, consider following the example here and
pushing your own design to GenHub!  And let us know if you have feedback!
