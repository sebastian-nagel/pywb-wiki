In addition to replaying web archive content by rewriting urls to point to the archive (known as 'archival mode'), pywb also supports 'proxy mode' replay where pywb acts as a proxy server.
Replay in proxy mode poses a few challenges, particularly with https support, as well as collection and date selection. This page lists the latest efforts for supporting proxy mode replay.

# Using Proxy Mode Replay

To use proxy mode, ensure that `enable_http_proxy: true` setting is set in the `config.yaml`

Configure the browser to use an HTTP (and HTTPS proxy mode), setting the proxy host port
to the where pywb is usually running.

For example, if pywb is running on *http://localhost:8080/*, set the proxy host to *localhost* and the port to *8080*

Now, when you visit HTTP content viewed in the browser will be loaded via pywb. If working correctly, you should see the
pywb banner inserted at the top.

# HTTPS Proxy Mode Support

To also support HTTPS content, a few additional steps are needed.

1. The ``pyopenssl`` package must be installed. This can be done via ``pip install pyopenssl``. If there are any installation errors, some system package may need to be installed too. (On Ubuntu, the following packages were also needed for OpenSSL ``sudo apt-get install openssl libssl-dev libffi-dev``)

2. Update the ``config.yaml`` (or current config) enabled https support:

  ```
  enable_http_proxy: True
  
  proxy_options:
      ...
      enable_https_proxy: true
  ```

3. (Optional) Create a root certificate authority (CA) certificate for pywb. For security reasons, pywb does not include one, but recommends that a unique one be created for your deployment. If no cert is present, pywb will create a default one the first time.

  The certificate can be created by running the `proxy-cert-auth` tool:

  `proxy-cert-auth ./ca/pywb-ca.pem -n "pywb https proxy replay CA"`

  (The above is the default cert created when there is none)

  The path to the cert, can be specified in the proxy_options as well. The current defaults are:
  ```
  # Default settings for CA used by proxy mode
  #    root_ca_file: ./ca/pywb-ca.pem
  #    root_ca_name: pywb https proxy replay CA
  #    certs_dir: ./ca/certs
  ```
  
  If creating a certificate manually, be sure to update the `root_ca_file` option to point to the correct cert.

4. The browser must be set to trust the new certificate. pywb provides a simplified way to do this. By browsing to **http://pywb.proxy/**, pywb will present a special page allowing users to download the certificate. There is a different version of the cert for Windows and for all other systems. Follow the instructions on the page to download the cert. Ensure that the browser is set to trust the cert for web pages.

Once the certificate has been imported, the browser should accept HTTPS requests to pywb!
(Note that from perspective of pywb, the protocol scheme is ignored when performing replay so http and https requests should yield the same results).

## How it Works

HTTPS support is dependent upon being able to access the underlying socket and wrap it in an SSL socket.
This functionality is dependent upon the WSGI container, and fortunately, this is possible to do in uWSGI, gUnicorn and wsgiref (and possibly others as well). Currently, HTTPS support is available only when running in uWSGI, gUnicorn or wsgiref although other containers may work as well or could be supported in the future.

pywb is able to support non-proxy, http and https proxy on the same port by routing the distinct HTTP requests:

Non-Proxy (Normal) HTTP request for: `http://localhost:8080/pywb/example.com/`
```
GET /pywb/example.com/
...
Host: localhost:8080/
```

HTTP Proxy Request for: `http://example.com/`
```
GET http://example.com/
...
```

HTTPS Proxy Request for: `https://example.com/`
```
CONNECT example.com:443
...
GET /
```

The proxy handler in pywb reads the CONNECT request and unwraps the underlying request in a SSL/TLS tunnel.
The SSL tunnel is created by using an on-the-fly generated certificate signed for the host (stored in `certs_dir`), signed with the specified `root_ca_file`

# Collection Selection

In archival mode, a replay collection can be selected simply altering the path, eg:
*/A/http://example.com* or */B/http://example.com* can be used to view contents of *http://example.com/*
in collections A or B, respectively.

In proxy mode, the collection needs to be specified in different ways. Obviously, this only applies if there is more than one collection being used in pywb.

When multiple collections are involved, it is possible to specify a default collection:

```
collections:
    coll_A: ...
    coll_B: ...

proxy_options:
   ...
   use_default_coll: coll_B
```

This will ensure that proxy requests default to `coll_B` unless a collection has been specified in another way.

## Cookie/Session Selection

The 'ideal' way to specify the collection is via some user setting, e.g. a cookie.
However, a cookie can not be shared across multiple domains in proxy mode (there is no proxy-cookie, unfortunately), so the cookie must somehow be passed to each domain.

This can be accomplished through the use of a magic proxy host, like `pywb.proxy` and have the cookie be propagated to individual hosts via series of redirects.

To allow changing the collection, it turns out the collection itself can not be stored in the cookie, but an indirect session id is required. The downside is that this requires some state on the server.

The request may be set as follows: `http://example.com` -> `select.pywb.proxy/http://example.com` -> user makes selection collA-> `collA-set.pywb.proxy/http://example.com/` -> `seshId-sethost.pywb.proxy.example.com/` -> http://example.com/ (with seshId cookie which resolves to collA)

TODO: improve explanation.

## Proxy Auth Selection

Another way to specify the collection is to overload the Proxy-Authentication feature, which provides a consistent user-supplied username/password that is set with each proxy request.

If there is no default collection, or `use_default_coll: false` is set, the first request to a proxy resource results in a 407 Proxy Authentication required message, requesting the user to enter a username/password.
The username is the collection name, eg: coll_A or coll_B and the password is ignored.
Once set, the user will not be asked again for the collection for the remainder of the session,and should work for both http and https requests (if enabled).

The main downside of Proxy-Auth is that switching collection requires going through reauthentication, which requires an ugly browser popup window -- there is no way to change collections via link for instance.