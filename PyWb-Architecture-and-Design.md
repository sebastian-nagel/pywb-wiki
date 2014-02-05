pywb is designed to be a simple WSGI web app which replays archival content as best as possible.

It consists of the following components (modules):

* **Config Parsing** -- App configuration is loaded from a .yaml file and further extendable via a custom init module.

* **Routing and Handlers** -- Support for routing archival url (of the form `/pywb/timestamp/url`) requests,  http proxy requests to the appropriate handler defined in the config.

* **Archive Index Loading** -- Archive indices, .cdx, files can be loaded locally or via a remote cdx server interface.

* **Archive File Loading** -- Archive files in .warc and .arc are loaded and parsed to retrieved archival content.

* **Content Rewriting** -- HTML, CSS, JS, XML and HTTP Header content is rewritten to ensure proper replay. Rewriting mostly involves rewriting urls in archived content, but may also involve more sophisticated and domain specific rewriting, especially for JS.

* **Views/UI** -- Basic UI for non-archival content is supported via Jinja2 html templates. Very very basic templates are provided for: home page, search page, query page and replay insert.


### Component Diagram

The following diagram provides a more in-depth look at current request/response control flow through the system. The *CDX Server* component will eventually move into a subpackage as it provides more generic functionality for querying archive indexes.

![Component Control Flow](https://archive.org/~ilya/pywb_arch_diag.png)