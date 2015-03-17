## Auto-Configuration Directory Structure

With release 0.9.0, pywb Wayback Machine features a new 'zero-configuration' or 'convention-over-configuration' system. No config files are required and collections are loaded automatically based on designated directory structure. (Configuration with existing `config.yaml` files will continue to work).

pywb requires one more collections. Each collection contains a set of archived files, indexes to archive files, and additional UI templates and static (non-archive) content (such js, css, etc...).

The expected directory structure for collections is as follows:

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

## Wayback Manager

To assist the user in setting up a new collection quickly and easily, pywb comes with a new `wayback-manager` utility. The utility can be used from the command-line to quickly create collections, add archive files (WARC/ARCS), and custom UI templates, static resources, and even user metadata.

The following brief tutorial shows how to use the new management utility.

### Initial setup

First, make sure that the latest pywb is installed -- this can be done by running:

`pip install pywb` or, if you've cloned the repo, `python setup.py install`

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

The new collection exists but is still empty. 

### Adding files to the Collection Archive

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

### Adding Metadata

The pywb Wayback Machine now also supports adding metadata data to collections.

Metadata consists of `name=values` pairs and will be stored in each collection's `metadata.yaml` file.

The `title` metadata will also be used on the home page and collection search page.
To add a title:

```wayback-manager metadata collA --add title="My First Collection"```

To add another metadata, say description, you can run

```wayback-manager metadata collA --set desc="Testing Out Metadata"```

Now, when you run ```wayback``` and navigate to the home page, you should see
`collA - My First Collection` listed.

When you visit ```http://localhost:8080/collA```, you should see all the metadata listed.

### Customizing UI -- Add UI Template

The UI for the home page and collection search page (and most other parts of pywb) can easily be modified.

pywb looks for templates in the `templates` directory in each collection, and otherwise loads the default from the pywb package.

To make it easier for users to modify any aspect of the html, the manager can copy the template to the local directory.

To copy the home page template, which will be created in `templates/index.html` you can run: 

```wayback-manager template add home_html```

To copy the `search.html` template for collA, which will be created in `collections/collA/templates/search.html`,
you can run:

```wayback-manager template add collA search_html```

To change the home page, you can simply edit `templates/index.html` or replace the file completely.




