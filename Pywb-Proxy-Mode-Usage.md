In addition to replaying web archive content by rewriting urls to point to the archive (known as 'archival mode'), pywb also supports 'proxy mode' replay where pywb acts as a proxy server.
Replay in proxy mode poses a few challenges, particularly with https support, as well as collection and date selection. This page lists the latest efforts for supporting proxy mode replay.

# Using Proxy Mode Replay

To use proxy mode, ensure that `enable_http_proxy: true` setting is set in the `config.yaml`

Configure the browser to use *pywb_path*/proxy.pac as the Proxy Auto-Configuration (PAC) script.

For example, if pywb is running on *http://localhost:8080/*, set the browser to `http://localhost:8080/proxy.pac`


# HTTPS Proxy Mode Support

**Currently available only in https-proxy branch: https://github.com/ikreymer/pywb/tree/https-proxy**

To also enable proxy mode with https support, ensure the following is present in the config:

```
enable_http_proxy: True

proxy_options:
    enable_https_proxy: true
    unaltered_replay: true
    
    # optional settings with defaults
    # root_ca_file: ./pywb_ca.pem
    # root_ca_name: pywb https proxy replay CA
    # certs_dir: ./pywb-certs/
```

The `unaltered_replay` option will ensure the replay is performed with no rewriting, which is optimal for proxy mode use. (TODO: Add support for banner insert but no url rewriting).

## Creating a root certificate

To support https replay, pywb will sign each host with its own root certificate. As a one-time setup, the browser must be configured to trust the root certificate. This is a necessary limitation of https proxy replay. The root certificate can be created as a one-time operation using the `proxy-cert-auth` tool:

`proxy-cert-auth ./pywb-root-ca.pem -n "Sample Proxy Replay Certificate"`

This will write the new certificate to./pywb-root-ca.pem with the specified name. This cert can then be set in a browser to trust https proxy requests. Be sure to set the `root_ca_file` properties above to match the new certificate. (If the root certificate doesn't exist, it will automatically be created using the `root_ca_file` and `root_ca_name` settings. However, it is recommended to create the certificate before starting pywb).

Once the certificate has been imported, the browser should accept HTTPS requests to pywb. (Note that from perspective of pywb, the protocol scheme is ignored when performing replay so http and https requests should yield the same results).

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

## Proxy Auth Selection

One way to specify the collection is to overload the Proxy-Authentication feature, which provides a consistent user-supplied username/password that is set with each proxy request.

If there is no default collection, or `use_default_coll: false` is set, the first request to a proxy resource results in a 407 Proxy Authentication required message, requesting the user to enter a username/password.
The username is the collection name, eg: coll_A or coll_B and the password is ignored.
Once set, the user will not be asked again for the collection for the remainder of the session.

TODO: Add explicit way of switching collections when needed.
