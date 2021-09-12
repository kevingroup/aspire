# ASPIRE
ASPIRE (ASsembly Pipeline with Iterative REfinement) is a pipeline for constructing virus-sized genomes out of NGS reads (short reads).

This is release 1.0.
Copyright &copy; 2021  Lee Sau Dan (<sdlee@cse.cuhk.edu.hk>)


## License
See [LICENSE](LICENSE).


## Prerequisites
- [Perl 5](https://www.perl.org/) (recommended: v5.32.1 or later)
  with the following [modules](https://www.perl.org/cpan.html):
  - App::Cmd::Setup
  - Bio::DB::Sam
  - Bio::Seq
  - Bio::SeqIO
  - Cwd
  - File::Path
  - File::Slurp
  - File::Spec
  - IPC::Run
  - List::Util
  - Math::Round
  - Statistics::Descriptive::Full
- [samtools](http://www.htslib.org/) (v1.13 or later)
- [bcftools](http://www.htslib.org/) (v1.13 or later)
- [cutadapt](https://cutadapt.readthedocs.io/) (v3.4 or later)
- [SPAdes](http://cab.spbu.ru/software/spades/) (v3.13.1 or later)
  - alternative to SPAdes: [SGA](https://github.com/jts/sga) (v0.10.15 or later)
- [MUMmer](http://mummer.sourceforge.net/) (v3.23 or later;
  [version 4.x](https://mummer4.github.io/) also supported)
- [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/) (v2.4.2 or later)
- [BWA](http://bio-bwa.sourceforge.net/) (v0.7.17 or later)
- [GapFiller](https://www.baseclear.com/genomics/bioinformatics/basetools/gapfiller) (v1-10 or later) [PubMed: 22731987](https://pubmed.ncbi.nlm.nih.gov/22731987/)


## Installation
No installation is need.
The file [aspire](aspire) is a Perl script.
Just copy it to somewhere under `$PATH`
and give it **execute** permission (`chmod +x aspire`).
Then, you can run it with the `aspire` command.
Alteratively, you may invoke it via: `perl aspire`.


## Running ASPIRE
`aspire` is a command line tool, which has a dozen subcommands.
Running it with `aspire --help` displays a copyright message
and a list of global options and commands.
The first non-option argument is considered a subcommand,
invoking the various parts of the ASPIRE pipeline.
Each subcommand has its own help page, which can be obtained with

    aspire --help <subcommand> 

Suppose you have start from gzipped FASTQ files,
`reads_1.fastq.gz` and `reads_2.fastq.gz`,
containing paired-end WGS reads for a sample.
You would like to construct the virus genome for this sample,
based on a reference genome for the virus in the file `virus_ref.fasta`.
You begin with the `new` subcommand:

    aspire --job-dir job1 new reads_virus_ref.fasta 1.fastq.gz reads_2.fastq.gz

This creates a new directory `job1` and
initializes it with symbolic links pointing to the given files.

Next, use the `run` command to run the ASPIRE pipeline:

    aspire --job-dir job1 run 2

Here, the argument `2` tells ASPIRE to stop after 2 iterative refinements.
The `run` command does many things:

1. It invokes the `trim` subcommand to trim the reads to remove adapters.
1. Then it invokes the `assemble` command to construct scaffolds out
   of the trimmed reads by denovo assembly.
   By default, SPAdes is used.
   You may use the `--sga` option of the `run` subcommand to
   override this default and use SGA instead.
1. Next, it invokes the `run-pass` command twice
   (or any number of times specified)
   with the appropriate arguments.
   The `run-pass` command, in turn, invokes the commands
   `tile`, `correct` and `fill` commands
   to carry one iterative refinment pass.
1. If the `run` command was given the `--wrap`,
   then an additional round of refinement is done
   by invoking the `wrap` subcommand,
   followed by `correct` and `fill`.

Finally, to retrieve the constructed virus genome,
use the `result` subcommand:

    aspire --job-dir job1 --id vg01 --description "My new genome" > vg01.fasta

Note that the subcommands `run`, `run-pass` are simply convenient
functions for invoking many other subcommands.
If a step in ASPIRE fails,
you may manually fix the problems
and continue from where it broke
by invoking the other subcommands manually.


## Collecting statistics
After constructing the new genome,
you may want to gather various statistics,
such as alignment rates and mapping qualities.
This is done in 2 steps.

1. Use the `align` command to align the reads to the constructed genomes:

        aspire align --pass all

1. Use the `stats` command to gather the results:

        aspire stats alignment-rates > aln-rates.tsv
        aspire stats mapq > mapq.tsv

	   

---
That's all folks.  
2021-09-12  Lee Sau Dan

