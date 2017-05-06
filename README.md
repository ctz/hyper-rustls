# hyper-rustls
This is an integration between the [rustls TLS stack](https://github.com/ctz/rustls)
and the [hyper HTTP library](https://github.com/hyperium/hyper).

[![Build Status](https://travis-ci.org/ctz/hyper-rustls.svg?branch=master)](https://travis-ci.org/ctz/hyper-rustls)
[![Crate](https://img.shields.io/crates/v/hyper-rustls.svg)](https://crates.io/crates/hyper-rustls)

[API Documentation](https://docs.rs/hyper-rustls/)

Implementations are provided of
[`hyper::net::SslClient`](http://hyper.rs/hyper/v0.9.10/hyper/net/trait.SslClient.html),
[`hyper::net::SslServer`](http://hyper.rs/hyper/v0.9.10/hyper/net/trait.SslServer.html)
and [`hyper::net::NetworkStream`](http://hyper.rs/hyper/v0.9.10/hyper/net/trait.NetworkStream.html).

By default clients verify certificates using the `webpki-roots` crate, which includes
the Mozilla root CAs.

# Examples
These are provided as an example of the minimal changes needed to
use rustls in your existing hyper-using program.

Note that these are derived works of original hyper source, and are
distributed under hyper's license.

## Client

```diff
--- ../hyper/examples/client.rs	2016-10-03 23:29:00.850098245 +0100
+++ examples/client.rs	2016-10-08 07:36:05.076449122 +0100
@@ -1,6 +1,8 @@
 #![deny(warnings)]
 extern crate hyper;
 
+extern crate hyper_rustls;
+
 extern crate env_logger;
 
 use std::env;
@@ -8,6 +10,7 @@
 
 use hyper::Client;
 use hyper::header::Connection;
+use hyper::net::HttpsConnector;
 
 fn main() {
     env_logger::init().unwrap();
@@ -32,7 +35,7 @@
             }
             Client::with_http_proxy(proxy, port)
         },
-        _ => Client::new()
+        _ => Client::with_connector(HttpsConnector::new(hyper_rustls::TlsClient::new()))
     };
 
     let mut res = client.get(&*url)
```

## Server

```diff
--- ../hyper/examples/server.rs	2016-10-03 23:29:00.850098245 +0100
+++ examples/server.rs	2016-10-08 07:31:38.720667338 +0100
@@ -1,5 +1,6 @@
 #![deny(warnings)]
 extern crate hyper;
+extern crate hyper_rustls;
 extern crate env_logger;
 
 use std::io::copy;
@@ -41,7 +42,10 @@
 
 fn main() {
     env_logger::init().unwrap();
-    let server = Server::http("127.0.0.1:1337").unwrap();
+    let certs = hyper_rustls::util::load_certs("examples/sample.pem");
+    let key = hyper_rustls::util::load_private_key("examples/sample.rsa");
+    let tls = hyper_rustls::TlsServer::new(certs, key);
+    let server = Server::https("127.0.0.1:1337", tls).unwrap();
     let _guard = server.handle(echo);
-    println!("Listening on http://127.0.0.1:1337");
+    println!("Listening on https://127.0.0.1:1337");
 }
```

# License
hyper-rustls is distributed under the following three licenses:

- Apache License version 2.0.
- MIT license.
- ISC license.

These are included as LICENSE-APACHE, LICENSE-MIT and LICENSE-ISC
respectively.  You may use this software under the terms of any
of these licenses, at your option.

