In this post, we're going to take a different direction from our _E. coli_ example from the last few posts, and show how gen
and GenHub can be used to create, store, and view combinatorial libraries, aka pooled designs.

If you're not familiar with pooled designs, you can think of them as creating variants of different
subsequences of a stretch of DNA and experimentally testing all possible combinations of those variants.  The variants can
all be combined in the same sample, making this a powerful technique for experimentation.  If that doesn't fully explain
it, hopefully the following example will help.

Here are 8 DNA parts in fasta format:

```
>part-1
ATATATAT
>part-2
GCGCGCGC
>part-3
CACACACA
>part-4
TGTGTGTG
>cds-1
AAAAAAAA
>cds-2
TTTTTTTT
>cds-3
GGGGGGGG
>cds-4
CCCCCCCC
```

We have them in a file called [parts.fa][parts-file].

Here is a little table in CSV format that buckets the parts into columns:

```
part-1,cds-1
part-2,cds-2
part-3,cds-3
part-4,cds-4
```

We have those in a file called [combinatorial_design.csv][csv-file].

This example isn't very realistic.  Typically the parts in one bucket will all be much more similar.  However, the
example sequences make it easier to see which parts are in which buckets.

With those files, we can create a directory called pooled-design-example, and run `gen init` to establish it as a gen
repo, and then ask for help with the `gen import library` command:

```
➜  pooled-design-example gen init
Gen repository initialized.
➜  pooled-design-example gen help import library
Import Library files

Usage: gen import library [OPTIONS] <REGION_NAME> [PARTS] [LIBRARY]

Arguments:
  <REGION_NAME>  The name of the region
  [PARTS]        The path to the combinatorial library parts fasta file
  [LIBRARY]      The path to the combinatorial library csv file

Options:
  -n, --name <NAME>      The name of the collection to store the entry under
  -s, --sample <SAMPLE>  A sample name to associate the library with
  -h, --help             Print help
```

With that information, we imported our example library:

```
➜  pooled-design-example gen import library example-design parts.fa combinatorial_design.csv
Library import called
Imported library file combinatorial_design.csv and parts file parts.fa
Library imported.
```

Next, we can create a repository on GenHub and push our repo there:

```
gen remote add origin https://www.genhub.bio/api/repos/david-genhub-bio/combinatorial-design-example
gen remote set-default origin
gen remote login origin
gen push -r origin
```

Our example repo is called
[combinatorial-design-example](https://www.genhub.bio/repos/david-genhub-bio/combinatorial-design-example).  There is a
[viewer for the design we've
created](https://www.genhub.bio/repos/david-genhub-bio/combinatorial-design-example/database?database=.gen%2Fdefault.db&collection=default&sample=&blockgroup=example-design&branch=main&operation=).
The graph view offers a good picture of a combinatorial design:

![GenHub design view](../../../../../post-files/pooled-designs/genhub-design-view.png)

This is a relatively simple design: Two regions, each with four variants.  There are a total of 16 possible combinations
of the two parts.  In more realistic designs, there may be ten variants per bucket, and three or more buckets.  With ten
variants per bucket, and three buckets, there are 1,000 possible combinations.  The number of sequences that can be
tested experimentally goes up exponentially as more buckets are added.

One thing we haven't discussed yet is the gen data model.  It's basically a directed graph structure, with the nodes
being subsequences, and arrows between nodes indicating which subsequences go together.  In the previous _E. coli_
example, the graph structure that allowed us to represent an intentional edit as well as mutations.  In this example,
the graph structure allows us to represent a combinatorial design.  It is a flexible way to represent genetic variation.
We'll discuss the gen data model more in a later post.

So what can we do with this design?  The most obvious thing is, you can send someone a link to the repo so they can take
a look and see what parts are in the design, and maybe provide feedback.  You could also share the repo with a DNA
synthesis provider as part of an order.

There is additional functionality we want to add here.  For one thing, once a sample with a design has been tested and
sequenced, you would want the ability to annotate part combinations with assay results, and to add the entire design as
a subgraph of the overall DNA sequence it's a part of.  Then you could update that sequence with a VCF to add in the
unintended variants that cropped up during the process.  So there is more work to do to record information about designs
in experiments, but we are looking forward to it.  If you think gen would be useful in your experiments with pooled
designs, please give it a try, and let us know if you have feedback!


[parts-file]: ../../../../../post-files/pooled-designs/parts.fa
[csv-file]: ../../../../../post-files/pooled-designs/combinatorial_design.csv
