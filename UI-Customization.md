In addition to high quality archival replay, pywb also provides a bare minimum of UI elements to make navigating between archived content a bit easier. The UI elements are intended to be customized for particular deployments and are easily extendable, either globally or per collection, via custom Jinja2/HTML templates.

Please refer to the default templates in `pywb/ui` templates for example usage.

* `banner_html` -- changes the banner inserted into the archived page or appearing in a frame.

* `home_html` -- changes the home page, eg, `http://localhost:8080/` if running 

* `search_html` -- changes the per collection search page, eg. `http://localhost:8080/pywb/'

* `query.html` -- changes the per collection calendar listing, eg:`http://localhost:8080/pywb/*/example.com/`. This page can display custom information about each capture as well.

Advanced usage:

* `head_insert_html` -- changes the full contents inserted into the `<head>` tag. This templates includes the `banner_html` template by default. Changing this may break aspects of client side rewriting, so change this only for advance cases. For changing the banner, override the `banner_html` template instead.

* `frame_insert_html` -- changes the contents of the top frame insert when in framed mode. This includes `banner_html` by default. Changing this make break certain functionality in frames mode, so use with caution.
