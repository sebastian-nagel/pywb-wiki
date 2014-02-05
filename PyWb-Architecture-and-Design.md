### Design Goals

Here is a list of some of the things that pywb is working towards:


* **Support simplest use case:** replay archive content given a directory of archive files (WARC and ARC) and a directory of corresponding indexes (CDX).

* **Well-tested components:** In-depth testing of content rewriting and other components. Sample archived content used for unit and integration testing, and continuous integration.

* **Run 'out-of-the-box':** Deployable from first pull, both directly and via a [virtual development environment](http://www.vagrantup.com/)

* **Usable Defaults and Easy Configuration:** Simple settings like archive, index, and ui template paths are configured via a .yaml file which comes with usable defaults.

* **Flexible Content Rewriting:** Provide a rule based framework to rewrite and replay difficult web content, such as dynamic content, as best as current technology will allow.

* **Advanced Customization:** Every component of pywb is pluggable and customizable for advanced users to maximize flexibility.



### Component Architecture

Currently, pywb consists of the following components:

* **Config Parsing** -- App configuration is loaded from a .yaml file and further extendable via a custom init module.

* **Routing and Handlers** -- Support for routing archival url (of the form `/pywb/timestamp/url`) requests,  http proxy requests to the appropriate handler defined in the config.

* **Archive Index Loading** -- Archive index CDX files can be loaded locally or via a remote cdx server interface.

* **Archive Content Loading** -- Archive content (WARC and ARC) files are loaded and parsed to retrieve archival content.

* **Content Rewriting** -- HTML, CSS, JS, XML and HTTP Header content is rewritten to ensure proper replay. Rewriting mostly involves rewriting urls in archived content, but may also involve more sophisticated and domain specific rewriting, especially for JS.

* **Views/UI** -- Basic UI for non-archival content is supported via Jinja2 html templates. Very very basic templates are provided for: home page, search page, query page and replay insert.


### PyWb Control Flow

Currently, the pywb requests handling flows through one of the following possible handling chains:

1. **Archival Replay of Text Content:** WSGI <-> Routing <-> Index Loading (CDX) <-> Archive Loading (WARC/ARC) <-> Streaming or Buffered Rewritten HTML/JS/CSS/XML Content

2. **Archival Replay of Binary Content:** WSGI <-> Routing <-> Index Loading (CDX) <-> Streaming Archive Content (WARC/ARC) 

3. **Captures Listing/Query:** WSGI <-> Routing <-> Index Loading (CDX) -> Jinja2/HTML Template

4. **Static Pages (Home, Search) or Errors:** WSGI <-> Routing <-> Jinja2/HTML Response


