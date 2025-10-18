---
layout: post
title:  "A first look at gen"
date:   2025-10-15 12:01:33 -0400
categories: gen genhub
---

Gen (pronounced like "Jen") is a new command line tool for tracking DNA sequence
changes and variants, from plasmids up to chromosome sized sequences.  Gen got
its start at Ginkgo Bioworks, and in this post we'll go through a simplified
version of an example synthetic biology workflow that one might see at Ginkgo.
We envision other uses for gen, but hopefully this example will show enough of
its capabilities to make things interesting.

To keep this example realistic but as simple as possible, we're going to make a
single deletion in an _E. coli_ genome and then incorporate the resulting
sequencing result.  We'll start with a reference genome fasta and then make the
deletion, and then update the edited version with a VCF file containing variants
detected during sequencing.  (In this case we're using an example VCF, but it
does match the _E. coli_ reference fasta.)

In case it's not obvious, gen is intended to be analogous to git, the source
control tool.  We fully recognize that DNA is not the same as computer code, but
it is possible to capture intentional DNA edits and unintentional variants in a
graph-like data structure, and it's a close enough analogy that we think it's
useful.

To begin, we initialize a gen repo:

{% highlight shell %}
➜ gen init
Gen repository initialized.
{% endhighlight %}

Next, we import the [_E coli_ reference fasta file][e-coli-reference] obtained
from NCBI:

{% highlight shell %}
➜ gen import fasta U00096.3.fa
Fasta import called
Parsing Fasta
[00:00:00] . . . . . . . . . . . . . . . . . . . . 🧬       1         Entries Processed.
[00:00:00]                                                            Saving operation
Fasta imported.
{% endhighlight %}

Gen stores sequences (contigs) and their variants in the form of 'graphs', which
can be listed as follows to confirm U00096.3 got imported:

{% highlight shell %}
➜ gen list-graphs
U00096.3
{% endhighlight %}

Now, we make our intentional edit, a partial deletion of the _lacZ_ gene.  This
deletion is a well-known genotype and descendent strains can be used in various laboratory
experiments.

{% highlight shell %}
➜ gen update fasta --new-sample lacZ-partial-deletion --region-name U00096.3 --start 366183 --end 366276 empty.fa
Update with fasta called
Updated with fasta file: empty.fa
{% endhighlight %}

[Source file for empty.fa][empty-fa]

We can also confirm the edit by computing a 'diff' in GFA format between the
default (original) sample and the child, and then inspecting the GFA:

{% highlight shell %}
➜ gen diff --sample2 lacZ-partial-deletion --gfa edited.gfa
➜ tail -5 edited.gfa
L	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.0.366183	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366276.4641652	+0M
L	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.0.366183	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366183.366276	+0M
L	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366183.366276	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366276.4641652	+0M
P	U-00096.3	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.0.366183+,b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366183.366276+,b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366276.4641652+	*
P	Lac-Z-Partial-Deletion.u-00096.3-Start-366183-End-366276-Node-E-3-B-0-C-44298-Fc-1-C-149-Afbf-4-C-8996-Fb-92427-Ae-41-E-4649-B-934-Ca-495991-B-7852-B-855	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.0.366183+,b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.366276.4641652+	*
{% endhighlight %}

We can also generate a fasta with the edit:

{% highlight shell %}
➜ gen export fasta --sample lacZ-partial-deletion deletion.fa 
FASTA export called 
Exported to file deletion.fa
{% endhighlight %}

Next, assuming we've done the edit in the lab and have submitted a sample for
sequencing, we would likely end up with a VCF file that captures the variants
and can be applied to the graph using the 'update' command:

{% highlight shell %}
➜ gen update vcf lacZ-partial-deletion-sequenced.vcf --sample lacZ-partial-deletion
Update with VCF called
Parsing VCF for changes.
[00:00:00] . . . . . . . . . . . . . . . . . . . . 🧬     676         Records Parsed
[00:00:00] █████████████████████████████████████████    1,352/1,352   Changes applied
[00:00:00]                                                            Saving operation
{% endhighlight %}

[Source file for lacZ-partial-deletion-sequenced.vcf][lacZ-partial-deletion-sequenced]

We can export the result after sequencing as a fasta file:

{% highlight shell %}
➜ gen export fasta --sample lacZ-partial-deletion-sequenced sequenced.fa
FASTA export called
Exported to file sequenced.fa
{% endhighlight %}

As we mentioned earlier, internally, gen tracks sequence changes as additions to
a graph (along the lines of a pangenome graph).  We can output the complete
graph generated by our updates in GFA format:

{% highlight shell %}
➜ gen export gfa -s lacZ-partial-deletion-sequenced sequenced-deletion.gfa
GFA export called
Warning: Path U00096.3 is not translatable to current graph.
GFA exported to: sequenced-deletion.gfa

➜ tail -5 sequenced-deletion.gfa
L	fe02b676320df897c35436af651c51fe3af36534f5a0f36fec5b26abc4d43d74.0.1	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.4004741.4016215	+0M
L	fe2b4e76caf4a06d766fa58686f207666c590ebf15da3ace5c7a6bac360f74db.0.1	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.2078280.2080649	+0M
L	fec1410349e95aea38290eb467de227c86c1e519d9f6598184e982c25ca5bed2.0.1	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.949547.963730	+	0M
L	ff0277b0d1785f821bd2b8712c86bf32ad5af2f63ce12a562193c5ab97c51e63.0.1	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.275303.275315	+	0M
L	ff1974eed255cde4bbb2cd29255ebfd3bf82cc321b145c9a35008627cd092119.0.1	+	b9957b8891756d872e8b9409d2eebeb12aea06dff5976b78723fa2d3f554711e.2168265.2168375	+0M
{% endhighlight %}

We can also propagate annotations to descendant samples and check the
coordinates.  For example, suppose we have a reference GFF file where _lacZ_ is
annotated in the coordinates of the reference genome sequence. We can propagate
this GFF file to our edited sequence and then 'grep' (search) for _lacZ_ in the
result as a way to check the deletion happened (the coordinate range should be
smaller in the sequence of the descendant):

{% highlight shell %}
➜ gen propagate-annotations -t lacZ-partial-deletion -g e-coli-lacZ.gff -o e-coli-lacZ-propagated.gff
GFF propagated to e-coli-lacZ-propagated.gff
{% endhighlight %}

[Source file for e-coli-lacZ.gff][e-coli-lacz-gff]

The resulting GFF file has _lacZ_ annotations with start/end coordinates updated
to the sequence in the sequenced sample:

{% highlight shell %}
➜ grep lacZ e-coli-lacZ.gff
U00096.3	Genbank	gene	363231	366305	.	-	.	ID=gene-b0344;Dbxref=ASAP:ABE-0001183,ECOCYC:EG10527;Name=lacZ;gbkey=Gene;gene=lacZ;gene_biotype=protein_coding;gene_synonym=ECK0341;locus_tag=b0344
U00096.3	Genbank	CDS	363231	366305	.	-	0	ID=cds-AAC73447.1;Parent=gene-b0344;Dbxref=UniProtKB/Swiss-Prot:P00722,NCBI_GP:AAC73447.1,ASAP:ABE-0001183,ECOCYC:EG10527;Name=AAC73447.1;gbkey=CDS;gene=lacZ;locus_tag=b0344;product=beta-galactosidase;protein_id=AAC73447.1;transl_table=11
➜ grep lacZ e-coli-lacZ-propagated.gff
U00096.3	Genbank	gene	363231	366212	.	-	.	ID=gene-b0344;Dbxref=ASAP:ABE-0001183,ECOCYC:EG10527;Name=lacZ;gbkey=Gene;gene=lacZ;gene_biotype=protein_coding;gene_synonym=ECK0341;locus_tag=b0344
U00096.3	Genbank	CDS	363231	366212	.	-	0	ID=cds-AAC73447.1;Parent=gene-b0344;Dbxref=UniProtKB/Swiss-Prot:P00722,NCBI_GP:AAC73447.1,ASAP:ABE-0001183,ECOCYC:EG10527;Name=AAC73447.1;gbkey=CDS;gene=lacZ;locus_tag=b0344;product=beta-galactosidase;protein_id=AAC73447.1;transl_table=11
{% endhighlight %}

This was a very simplified version of how one might use gen with a microbial
workflow, but hopefully it gives a taste of what it can be used for.  Gen can
handle many intentional edits, and any number of corresponding VCF updates.
Each would end up in a separate sample belonging to a lineage tracked in the
current repo.

There are a couple new general capabilities that gen adds for bioinformatic
workflows.  The first is that it is iterative.  It allows you to capture
information across multiple rounds of genetic changes.  The second is that it is
collaborative.  The person designing intended edits is often not the person who
is incorporating the sequencing data.  With gen, those people have a shared data
repository to collaborate on.  We're working on a web frontend that will people
view all these changes similar to how one can view a code diff on GitHub.

Gen is written in Rust, so is fast and reliable, and uses SQLite as a storage
backend, so everything we just did can be reproduced locally.

If gen looks interesting to you, please try it out.  [There are builds available
for Mac OS X and Linux][gen-nightly-builds].  Be aware that it is still in
active development, so it is quite possible you will see bugs.  But we are
committed to making this software better, so please let us know if you have
feedback!  We welcome bug reports and feature requests.  We want to make it
easier for people to manage genomic data, and we hope gen is useful.

[e-coli-reference]: https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000005845.2/
[empty-fa]: ../../../../../post-files/introducing-gen/empty.fa
[lacZ-partial-deletion-sequenced]: ../../../../../post-files/introducing-gen/lacZ-partial-deletion-sequenced.vcf
[e-coli-lacz-gff]: ../../../../../post-files/introducing-gen/e-coli-lacZ.gff
[gen-nightly-builds]: https://github.com/genhub-bio/gen/releases/tag/nightly
