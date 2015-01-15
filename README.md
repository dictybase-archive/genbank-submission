# genbank-submission

___Generate dictyBase files to update the GenBank (NIH)___


### Jan 2015

Following directions specified at [Genbank_Submission](http://bcwiki.bioinformatics.northwestern.edu/bcwiki/index.php/Genbank_Submission) generated the following problems (on testdb):

* Running the main Perl script that generates three different files per chromosome:

`perl write_genbank_files.pl`

which throws the following error...

    cannot open file Permission denied at write_genbank_files.pl line 33

However, with sudo:

```
------------- EXCEPTION: Bio::Root::Exception -------------
MSG: Unable to retrieve genbank release value
STACK: Error::throw
STACK: Bio::Root::Root::throw /usr/local/share/perl/5.8.7/Bio/Root/Root.pm:357
STACK: dicty::Root::throw /usr/local/dicty/lib/dicty/Root.pm:83
STACK: dicty::Feature::GENE::_get_release /usr/local/dicty/lib/dicty/Feature/GENE.pm:331
STACK: dicty::Feature::GENE::release /usr/local/dicty/lib/dicty/Feature/GENE.pm:313
STACK: dicty::Writer::Genbank_table::write_feature /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm:491
STACK: dicty::Writer::Genbank_table::write_contig_tbl /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm:436
STACK: dicty::Writer::Genbank_table::write_chromosome_tbl /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm:355
STACK: write_genbank_files.pl:33
-----------------------------------------------------------
Attempt to free unreferenced scalar: SV 0xbfb8ffc, Perl interpreter: 0x814b008 during global destruction.
```

After debugging the error, it turns out that `_get_release` is searching for a `databank_entry` in the database that throws an error.

Test: comment line 31 of the main script:

    # $genbank_writer->is_update( 1 );

but <i></i>t throws this error:

```
$ sudo perl write_genbank_files.pl

ERROR with DDB0216454

------------- EXCEPTION: Bio::Root::Exception -------------
MSG: Bio::SeqFeature::Gene::Transcript=HASH(0x105525f8) is not contained within parent feature, and expansion is not valid
STACK: Error::throw
STACK: Bio::Root::Root::throw /usr/local/share/perl/5.8.7/Bio/Root/Root.pm:357
STACK: Bio::SeqFeature::Generic::add_SeqFeature /usr/local/share/perl/5.8.7/Bio/SeqFeature/Generic.pm:764
STACK: dicty::Feature::CONTIG::transcript_features /usr/local/dicty/lib/dicty/Feature/CONTIG.pm:168
STACK: dicty::Feature::CONTIG::_get_genes /usr/local/dicty/lib/dicty/Feature/CONTIG.pm:105
STACK: dicty::Feature::CONTIG::genes /usr/local/dicty/lib/dicty/Feature/CONTIG.pm:205
STACK: dicty::Feature::CONTIG::transcript_features /usr/local/dicty/lib/dicty/Feature/CONTIG.pm:159
STACK: dicty::Feature::CONTIG::float /usr/local/dicty/lib/dicty/Feature/CONTIG.pm:229
STACK: dicty::Writer::Genbank_table::write_chromosome_tbl /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm:355
STACK: write_genbank_files.pl:33
-----------------------------------------------------------
 at /usr/local/dicty/lib/dicty/Feature/CONTIG.pm line 169
        dicty::Feature::CONTIG::transcript_features('dicty::Feature::CONTIG=HASH(0xb6f97f8)', 'ARRAY(0xfffc774)') called at /usr/local/dicty/lib/dicty/Feature/CONTIG.pm line 105
        dicty::Feature::CONTIG::_get_genes('dicty::Feature::CONTIG=HASH(0xb6f97f8)') called at /usr/local/dicty/lib/dicty/Feature/CONTIG.pm line 205
        dicty::Feature::CONTIG::genes('dicty::Feature::CONTIG=HASH(0xb6f97f8)') called at /usr/local/dicty/lib/dicty/Feature/CONTIG.pm line 159
        dicty::Feature::CONTIG::transcript_features('dicty::Feature::CONTIG=HASH(0xb6f97f8)') called at /usr/local/dicty/lib/dicty/Feature/CONTIG.pm line 229
        dicty::Feature::CONTIG::float('dicty::Feature::CONTIG=HASH(0xb6f97f8)') called at /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm line 355
        dicty::Writer::Genbank_table::write_chromosome_tbl('dicty::Writer::Genbank_table=HASH(0x8f14e4c)', 'dicty::Feature::CHROMOSOME=HASH(0x8f78e18)') called at write_genbank_files.pl line 33
unexpected gap type:  on chromosome 5. at /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm line 1559.

```
This was as consequence of running the script only for chromosome 5, but testing chr 6, 4 or 3, it gives this error:

    unexpected gap type:  on chromosome 6. at /usr/local/dicty/lib/dicty/Writer/Genbank_table.pm line 1559.


