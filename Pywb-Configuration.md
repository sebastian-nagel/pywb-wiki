This page explains how to configure pywb wayback to run with your WARC and ARC files.

PyWb requires the creation of sorted index files, called 'cdx' files, in order to search through the archives. It should be possible to run pywb with existing cdx filee, although creating new SURT ordered cdx is recommended for more flexibility.

### Running with Existing CDX/WARCs

If you have existing .warc/.arc and .cdx files, you can adjust the `index_paths` and `archive_paths` sections in the [config.yaml](../blob/master/config.yaml) to point to the location of those files.

#### SURT

By default, pywb expects the cdx files to be Sort-friendly URL Reordering Transform (SURT) ordering.
This is an ordering that transforms: `example.com` -> `com,example)/` to faciliate better search.
It is recommended for future indexing, but is not required.

Non-SURT ordered cdx indexs will work as well, but be sure to specify:

`surt_ordered: False` in the [config.yaml](../blob/master/config.yaml)

### Creating CDX from WARCs (or ARCs)

pywb (from version 0.2.x) includes its a command-line tool for generating cdx files,
to use run `cdx-indexer` after installing pywb.

To create a single sorted index `mypath/cdx/mywarc.cdx` from a single warc `mypath/warcs/mywarc.gz` run:
`cdx-indexer -s mypath/cdx/mywarc.cdx mypath/warcs/mywarc.gz`

To index a full directory of warcs, and merge the cdx into a single index, run:
`cdx-indexer -s mypath/cdx/all.cdx mypath/warcs/`


* The default output uses SURT key ordered cdx files (as explained), but the `-u` flag is used to output non-surt ordered cdx. If using `-u`, set `surt_ordered: False`

* The default output is not sorted, but the `-s` flag is used to have the `cdx-indexer` output a sorted cdx. For use with pywb, sorted cdxs are necessary.

* For very large warcs, or lots of warcs in a directory, it may be desirable to use the system sort command instead of `-s` to do the sorting. `cdx-indexer` can be used to generate the unsorted cdx to stdout and then
piped to system sort command:

* ```
  export LC_ALL=C
  cdx-indexer mypath/lots_of_warcs/ | sort > mypath/cdx/all.cdx
  ```

### Old Way -- Creating CDX from WARCs

**This is an older way to create cdxs, left here for reference. pywb now includes its own cdx indexer which removes much of this complexity and is explained above**

If you have warc files without cdxs, the following steps can be taken to create the indexs.

cdx indexs are sorted plain text files indexing the contents of archival records in one or more WARC/ARC files.

(The cdx_writer tool creates SURT ordered keys by default)

pywb does not currently generate indexs automatically, but this may be added in the future.

For production purposes, it is recommended that the cdx indexs be generated ahead of time.


** Note: these recommendations are subject to change as the external libraries are being cleaned up **

The directions are for running in a shell:

1. Clone https://bitbucket.org/rajbot/warc-tools
                                                                                                                                                        
2. Clone https://github.com/internetarchive/CDX-Writer to get **cdx_writer.py**

3. Copy **cdx_writer.py** from `CDX_Writer` into **warctools/hanzo** in `warctools`

4. Ensure sort order set to byte-order `export LC_ALL=C` to ensure proper sorting.

5. From the directory of the warc(s), run `<FULL PATH>/warctools/hanzo/cdx_writer mypath/warcs/mywarc.gz | sort > mypath/cdx/mywarc.cdx`

   This will create a sorted `mywarc.cdx` for `mywarc.gz`. Then point `pywb` to the `mypath/warcs` and `mypath/cdx` directories in the yaml config.



6. pywb sort merges all specified cdx files on the fly. However, if dealing with larger number of small cdxs, there will be performance benefit

    from sort-merging them into a larger cdx file before running pywb. This is recommended for production.

    An example sort merge post process can be done as follows:

   ```
   export LC_ALL=C
   sort -m mypath/cdx/*.cdx > mypath/merged_cdx/merge_1.cdx
   ```

   (The merged cdx will start with several ` CDX` headers due to the merge. These headers indicate the cdx format and should be all the same!
    They are always first and pywb ignores them)

   In the yaml config, set `index_paths` to point to `mypath/merged_cdx/merged_1.cdx`
                                                                                                                   