**Please note: this documentation is for the pywb 0.9.0 beta release.**

**To use, please install via: `pip install pywb==0.9.0b1`**

**Please feel free to [submit an issue](https://github.com/ikreymer/pywb/issues) for any suggestions, improvements or errors found in these docs.**

**Any feedback on new conventions/names introduced is especially welcome.**

**Thanks!**

## Introducing Auto-Configuration and Collection Manager

With release 0.9.0, pywb Wayback Machine features a new 'auto-configuration' or 'convention-over-configuration' system. No config files are required and collections are loaded automatically based on designated directory structure. (Deployments with existing `config.yaml` files will continue to work, as an existing `config.yaml` will take precedence).

pywb requires one or more collections. Each collection contains a set of archived files, indexes to archive files, and additional UI templates and static (non-archive) content (such js, css, etc...).

If no custom `config.yaml` is present, the expected directory structure for collections is as follows:

```
my_archive:
  + collections
    + collA
        archive
        indexes
        static (optional)
        templates (optional)
        metadata.yaml (optional)
        config.yaml (optional)

    + another_collection
        ...
```


## Tutorial

To assist the user in setting up a new collection quickly and easily, pywb comes with a new `wayback-manager` utility. The utility can be used from the command-line to quickly create collections, add archive files (WARC/ARCS), and custom UI templates, static resources, and even user metadata.

The following brief tutorial shows how to use the new management utility.

### Initial setup

First, make sure that the latest pywb is installed -- this can be done by running:

`pip install pywb==0.9.0b1` or, if you've cloned the repo, `python setup.py install`

Once pywb is installed, it is best to start with a clean directory for your archive.

`mkdir ~/myarchive; cd ~/myarchive`

should be a good start.

### Creating a new collection

To create a new collection, you can run

```wayback-manager init collA```

This command will create the `collections` subdirectory, `collA` directory and all the other required directories.

The following directories should have been created in the current directory (eg: `my_archive`):

```
my_archive:
  + collections
    + collA
        archive
        indexes
        static
        templates
```

To verify that the collection has been created you may run:
```wayback-manager list```

This should print out a list:
```
Collections:
  - collA
```

The new collection now exists, so it's time to add some archive files. 

### Adding files to the collection

A collection consists of any number of archive files (WARC or ARC) format. Any number of ARC/WARC files can be added to the collection by running:

```wayback-manager add <collName> <path/to/warc> <path/to/another_warc> ...
```

Multiple files can added at once. For instance, if you have a directory of ARC/WARC files ready, you may add them
at once by running:

```wayback-manager add collA /path/to/warcs/*.warc.gz
```

If you do not have any WARC files, you can use https://webrecorder.io to record any page, then download the WARC file (it will have a `.warc.gz` extension). To add this new file, you can do:

```wayback-manager add ~/Downloads/mynewwarcfile.warc.gz```

All the added files will be copied to `collections/collA/archive` directory and will be automatically indexed.

Now that you've added the WARC, you can run ``wayback``. One the home page, `http://localhost:8080/`, you should see a link to the collection search page, at `http://localhost:8080/collA/`.

If you know a url in the WARC file, you can enter it in the search box to see the capture.

*Automated page listing coming soon*

### Adding Collection Metadata

The pywb Wayback Machine now also supports adding metadata data to collections.

Metadata consists of `name=values` pairs and will be stored in each collection's `metadata.yaml` file.

The `title` metadata will also be used on the home page and collection search page.
To add a title:

```wayback-manager metadata collA --set title="My First Collection"```

To add another metadata, say description, you can run

```wayback-manager metadata collA --set desc="Testing Out Metadata"```

Now, when you run ```wayback``` and navigate to the home page, you should see
`collA - My First Collection` listed.

When you visit ```http://localhost:8080/collA```, you should see all the metadata listed.

### Customizing UI Templates

The UI for the home page and collection search page (and most other parts of pywb) can easily be modified.

pywb looks for templates in the `templates` directory in each collection, and otherwise loads the default from the pywb package.

To make it easier for users to modify any aspect of the html, the manager can copy the template to the local directory.

To copy the home page template, which will be created in `templates/index.html` you can run: 

```wayback-manager template add home_html```

To copy the `search.html` template for collA, which will be created in `collections/collA/templates/search.html`,
you can run:

```wayback-manager template add collA search_html```

To change the home page, you can simply edit `templates/index.html` or replace the file completely.

### Adding static files

To add static files, simply copy them to the `collections/collA/static/` directory and they will be served
by the Wayback application.

For example, if you create `collections/collA/static/mycss.css`, and run `wayback`,
you can access the css file via: `http://localhost:8080/static/collA/mycss.css`


## Advanced Usage

The following are some more advanced usage scenarios.

### Custom Archive Directory Structure

Although the collection manager adds all ARC/WARCS to the root of the `<coll name>/archive` directory, it is possible to have an arbitrary directory structure. For example, a user may add

```
collA/archive/group-1/warc1.gz
collA/archive/group-2/warc2.gz
...
```

### Manual Indexing

If manually adding ARC/WARCs to the archive, it is necessary to update the indexes in the `indexes` directory.

For example, you may run `wayback-manager collA reindex` to automatically reindex all the files in the archive directory.

For larger archives, this may be a bit slower. It is also possible to reindex specific files by running:

```wayback-manager collA index collections/collA/archive/group-1/warc1.gz collections/collA/archive/group-2/warc2.gz```

This command will index the specified files and merged the resulting index (CDX) with the existing index.
This is particularly useful if adding WARC files manually.

### Removing Files

As this is an archive, is not common, and there is no manager command for doing so.
If WARC/ARC files are removed manually from the `archive` directory, you can simply run the `wayback-manager <coll> reindex` to build the index.

HTML templates may be removed with ```wayback-manager template --remove <name>```.

### Custom Indexes

By default, all archive files are indexed into a single `indexes/index.cdx`. However, the entire `indexes` directory is searched for indexes on startup.

The allows for creating very flexible setups, and running the `cdx-indexer` tool manually to create indexes
as desired.

### Custom Config

It is also possible to create a per-collection `config.yaml` to override any of the default setting.

For example, to add a remote archive destination in addition to the local `./archive/` directory, one could specify in the per-collection `config.yaml`

```
archive_paths:
   - ./archive
   - http://archive.path.example.com/path/to/archive/
```

Both locations would then be checked to locate an archive file.
All [existing additional loading options](Additional-Archive-Loading-Options) are supported.



## More Information

For the latest command manager reference, you may run
`wayback-manager --help`



