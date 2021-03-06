************
Text Outputs
************

Two text outputs are available from Tombo:

1. Genome Broser Files - Genome browser compatible per-genomic-base statistics
2. Fasta - Genomic sequence output surrounding identified modified base sites

``tombo text_output browser_files``
-----------------------------------

The ``tombo text_output browser_files`` command takes in a set of reads (``--fast5-basedirs``) and/or a statistics file generated from a ``tombo detect_modifications`` command (``--statistics-filename``). A control set of reads can also be provided (``--control-fast5-basedirs``). Output files will be produced for each requested statistic (both plus and minus strands) in `variableStep wiggle format <https://genome.ucsc.edu/goldenpath/help/wiggle.html>`_ (or `bedgraph format <https://genome.ucsc.edu/goldenpath/help/bedgraph.html>`_ for ``--file-type coverage``).

Several statistics are available for output:

* ``coverage`` - The coverage level for mapped and validly re-squiggled reads
* ``valid_coverage`` - The coverage level for reads that are mapped, validly re-squiggled and outside the interval specified by ``--single-read-threshold`` specified in a ``--statistics-filename``.
* ``dampened_fraction`` - The estimated fraction of significantly modified reads
* ``fraction`` - The raw fraction of significantly modified reads
* ``signal`` - The mean signal level across all reads mapped to this location
* ``signal_sd`` - The mean signal standard deviation across all reads mapped to this location (not available unless ``--include-event-stdev`` was provided in ``tombo resquiggle`` command)
* ``dwell`` - The mean number of raw observations observed assigned to this location
* ``difference`` - The difference in normalized signal level between a sample and control set of reads

.. note::

   ``signal``, ``signal_sd``, ``dwell`` and ``difference`` require each reads' event level data to be extracted from the raw read files and thus may be quite slow. ``coverage``, ``valid_coverage``, ``fraction`` , and ``dampened_fraction`` can be extracted simply from the tombo statistics files, which is much faster.

   The ``signal``, ``signal_sd``, ``dwell`` and ``difference`` outputs all require the ``--fast5-basedirs`` option, the ``valid_coverage``, ``fraction`` , and ``dampened_fraction`` outputs require the ``--statistics-filename`` option, and ``coverage`` output requires one or the other.

Files will be output to individual wiggle files (two per statistic for plus and minus genomic strand) in the following format ``[wiggle-basename].[wiggle-type].[sample|control]?.[plus|minus].wig``

``tombo text_output signif_sequence_context``
---------------------------------------------

The ``tombo text_output signif_sequence_context`` command writes the genome sequence surrounding unique genomic positions with the largest estimated fraction of modified bases. This can be useful for several tasks related to modified base detection including motif discovery.

To run ``tombo text_output signif_sequence_context``, a ``--statistics-filename`` is required to extract the most significant locations and either a ``--fast5-basedirs`` or ``--genome-fasta`` is required to extract the genomic sequence. Several options are availble for selecting the sequence to be output:

* ``--num-regions`` - Defines the number of unique locations to be output
* ``--num-bases`` - Defines the number of bases to be output surrounding the significant locations

The output of this command could be used to determine sequence contexts consistently modified within a sample. Example `meme <http://meme-suite.org/doc/meme.html>`_ command line modified base motif detection command.

.. code-block:: bash

   tombo detect_modifications de_novo --fast5-basedirs <fast5s-base-directory> \
                --statistics-file-basename sample.de_novo
   tombo text_output signif_sequence_context --statistics-filename sample.de_novo.tombo.stats \
                --genome-fasta reference.fasta --num-regions 1000 --num-bases 50
   ./meme -oc tombo.de_novo_motif_detection.meme -dna -mod zoops tombo_results.significant_regions.fasta
