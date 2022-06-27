[[Storyblok]]
[[Gitpod]]

The only working option is to use gitpod companion app and run from local host

1.Create new configMap for the Caddyfile
``` yaml
apiVersion: v1
data:
  Caddyfile: "{\n\t# disable automatic SSL certificate generation\n\tauto_https off\n\t#
    disable admin API server\n\t# admin localhost:2019\n\tadmin off\n\n\t# set default
    SNI for old clients\n\tdefault_sni {$GITPOD_DOMAIN}\n\n\t# debug\n\n\t# configure
    plugin order\n\t# https://caddyserver.com/docs/caddyfile/directives#directive-order\n\torder
    gitpod.cors_origin            before header\n\torder gitpod.workspace_download
    \    before redir\n\torder gitpod.headless_log_download\tbefore rewrite\n\torder
    gitpod.sec_websocket_key      before header\n\n\tservers {\n        protocol {\n
    \           allow_h2c\n        }\n    }\n}\n\n(compression) {\n\tencode zstd gzip\n}\n\n#
    configure headers to force HTTPS and enable more strict rules for the browser\n(security_headers)
    {\n\theader {\n\t\t# enable HSTS\n\t\tStrict-Transport-Security  max-age=31536000\n\t\t#
    disable clients from sniffing the media type\n\t\tX-Content-Type-Options     nosniff\n\t\t#
    Define valid parents that may embed a page\n\t\tContent-Security-Policy    \"frame-ancestors
    'self' https://*.{$GITPOD_DOMAIN} https://{$GITPOD_DOMAIN} https://app.storyblok.com\"\n\t\t#
    keep referrer data off of HTTP connections\n\t\tReferrer-Policy            no-referrer-when-downgrade\n\t\t#
    Enable cross-site filter (XSS) and tell browser to block detected attacks\n\t\tX-XSS-Protection
    \          \"1; mode=block\"\n\n\t\tdefer # delay changes\n\t}\n}\n\n# workspace
    security headers\n(workspace_security_headers) {\n\theader {\n\t\t# Disallow sharing
    the same browsing context when opened in a popup\n\t\tCross-Origin-Opener-Policy
    same-origin-allow-popups\n\t}\n\timport security_headers\n}\n\n(enable_log) {\n\tlog
    {\n\t\toutput stdout\n\t\tformat if \"status > 399\" jsonselect \"{severity:level}
    {timestamp:ts} {logName:logger} {httpRequest>requestMethod:request>method} {httpRequest>protocol:request>proto}
    {httpRequest>status:status} {httpRequest>responseSize:size} {httpRequest>userAgent:request>headers>User-Agent>[0]}
    {httpRequest>requestUrl:request>uri} {httpRequest>requestHost:request>host} {cacheStatus:resp_headers>X-Cache-Status>[0]}\"
    {\n\t\t\tlevel_format \"upper\"\n\t\t\ttime_format \"rfc3339_nano\"\n\t\t}\n\t}\n}\n\n(enable_log_debug)
    {\n\tlog {\n\t\toutput stdout\n\t\tformat jsonselect \"{severity:level} {timestamp:ts}
    {logName:logger} {httpRequest>requestMethod:request>method} {httpRequest>protocol:request>proto}
    {httpRequest>status:status} {httpRequest>responseSize:size} {httpRequest>userAgent:request>headers>User-Agent>[0]}
    {httpRequest>requestUrl:request>uri} {httpRequest>requestHost:request>host} {cacheStatus:resp_headers>X-Cache-Status>[0]}\"
    {\n\t\t\tlevel_format \"upper\"\n\t\t\ttime_format \"rfc3339_nano\"\n\t\t}\n\t}\n}\n\n(remove_server_header)
    {\n\theader {\n\t\t-server\n\t\t-x-powered-by\n\t}\n}\n\n(ssl_configuration) {\n\ttls
    /etc/caddy/certificates/tls.crt /etc/caddy/certificates/tls.key {\n\t\tprotocols
    tls1.2\n\t\t#ca_root   <pem_file>\n\t}\n}\n\n(upstream_headers) {\n\theader_up
    X-Real-IP {http.request.remote.host}\n}\n\n(upstream_connection) {\n\tlb_try_duration
    1s\n}\n\n(debug_headers) {\n\theader X-Gitpod-Region {$GITPOD_REGION}.{$GITPOD_INSTALLATION_SHORTNAME}\n}\n\n(workspace_transport)
    {\n\ttransport http {\n\t\ttls_insecure_skip_verify\n\t\tkeepalive 60s\n\t\tkeepalive_idle_conns
    100\n\t}\n}\n\n(google_storage_headers) {\n\theader {\n\t\t-x-guploader-uploadid\n\t\t-etag\n\t\t-x-goog-generation\n\t\t-x-goog-metageneration\n\t\t-x-goog-hash\n\t\t-x-goog-stored-content-length\n\t\t-x-gitpod-region\n\t\t-x-goog-stored-content-encoding\n\t\t-x-goog-storage-class\n\t\t-x-goog-generation\n\t\t-x-goog-metageneration\n\t\t-cache-control\n\t\t-expires\n\n\t\tdefer
    # delay changes\n\t}\n}\n\n# Kubernetes health-check\n:8003 {\n\trespond /live
    200\n\trespond /ready 200\n}\n\n# TODO: refactor once we can listen only in localhost\n:9545
    {\n\tmetrics /metrics {\n\t\tdisable_openmetrics\n\t}\n}\n\n# public-api\napi.{$GITPOD_DOMAIN}
    {\n    log {\n        level DEBUG\n        output stdout\n    }\n\n    reverse_proxy
    h2c://public-api-server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:9001\n}\n\n\n# always
    redirect to HTTPS\nhttp:// {\n\tredir https://{host}{uri} permanent\n}\n\nhttps://{$GITPOD_DOMAIN}
    {\n\timport enable_log\n\timport remove_server_header\n\timport ssl_configuration\n\timport
    security_headers\n\n\t@workspace_download path /workspace-download*\n\thandle
    @workspace_download {\n\t\timport google_storage_headers\n\n\t\theader {\n\t\t\t#
    The browser needs to see the correct archive content type to trigger the download.\n\t\t\tcontent-type
    \"application/tar+gzip\"\n\t\t}\n\n\t\tgitpod.workspace_download {\n\t\t\tservice
    http://server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3000\n\t\t}\n\n\t\t# redirect works
    here because we \"navigate\" to this URL, which makes the browser handle this
    as primary request, and not fuff around with CORS at all\n\t\tredir {http.gitpod.workspace_download_url}
    303\n\t}\n\n\t@headless_log_download path /headless-log-download*\n\thandle @headless_log_download
    {\n\t\theader {\n\t\t\t# Alltough logs are plain text \"text/html\" works for
    reliably for streaming\n\t\t\tcontent-type \"text/html; charset=utf-8\"\n\t\t}\n\n\t\t#
    Perform lookup to server and actual reverse_proxy in one go because caddy's `reverse_proxy`
    is not powerful enough\n\t\tgitpod.headless_log_download {\n\t\t\tservice http://server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3000\n\t\t}\n\t}\n\n\t@backend_wss
    {\n\t\t\tpath /api/gitpod\n\t}\n\thandle @backend_wss {\n\t\t\tgitpod.sec_websocket_key\n\n\t\t\turi
    strip_prefix /api\n\t\t\treverse_proxy server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3000
    {\n\t\t\t\t\timport upstream_headers\n\t\t\t}\n\t}\n\n\t@backend path /api/* /headless-logs/*\n\thandle
    @backend {\n\t\tgitpod.cors_origin {\n\t\t\tbase_domain {$GITPOD_DOMAIN}\n\t\t}\n\n\t\t#
    note: no compression, as that breaks streaming for headless logs\n\n\t\turi strip_prefix
    /api\n\t\treverse_proxy server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3000 {\n\t\t\timport
    upstream_headers\n\t\t\timport upstream_connection\n\n\t\t\t# required for smooth
    streaming of terminal logs\n\t\t\tflush_interval -1\n\t\t}\n\t}\n\n\t@codesync
    path /code-sync*\n\thandle @codesync {\n\t\tgitpod.cors_origin {\n\t\t\tany_domain
    true\n\t\t}\n\n\t\timport compression\n\n\t\treverse_proxy server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3000
    {\n\t\t\timport upstream_headers\n\t\t\timport upstream_connection\n\n\t\t\tflush_interval
    -1\n\t\t}\n\t}\n\n\t@local_app {\n\t\tpath /static/bin/gitpod-local-companion-*\n\t}\n\thandle
    @local_app {\n\t\timport compression\n\n\t\treverse_proxy ide-proxy.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:80
    {\n\t\t\timport upstream_headers\n\t\t\timport upstream_connection\n\t\t}\n\t}\n\n\t@to_server
    path /auth/github/callback /auth /auth/* /apps /apps/*\n\thandle @to_server {\n\t\timport
    compression\n\n\t\treverse_proxy server.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3000
    {\n\t\t\timport upstream_headers\n\t\t\timport upstream_connection\n\t\t}\n\t}\n\n\thandle
    {\n\t\treverse_proxy dashboard.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:3001 {\n\t\t\timport
    upstream_headers\n\t\t\timport upstream_connection\n\t\t}\n\t}\n\n\t@legacy_urls
    path /github.com/* /gitlab.com/* /bitbucket.org/*\n\thandle @legacy_urls {\n\t\tredir
    https://{$GITPOD_DOMAIN}/#{uri} permanent\n\t}\n\n\thandle_errors {\n\t\tredir
    https://{$GITPOD_DOMAIN}/sorry/#Error%20{http.reverse_proxy.status_text} 302\n\t}\n}\n\n#
    workspaces\nhttps://*.*.{$GITPOD_DOMAIN} {\n\timport enable_log\n\timport workspace_security_headers\n\timport
    remove_server_header\n\timport ssl_configuration\n\timport debug_headers\n\n\t@workspace_blobserve
    header_regexp host Host ^blobserve.ws(?P<location>-[a-z0-9]+)?.{$GITPOD_DOMAIN}\n\thandle
    @workspace_blobserve {\n\t\tgitpod.cors_origin {\n\t\t\tbase_domain {$GITPOD_DOMAIN}\n\t\t}\n\n\t\treverse_proxy
    https://ws-proxy.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:9090 {\n\t\t\timport workspace_transport\n\t\t\timport
    upstream_headers\n\n\t\t\theader_up X-WSProxy-Host       {http.request.host}\n\n\t\t\theader_down
    -access-control-allow-origin\n\t\t}\n\t}\n\n\t@workspace_port header_regexp host
    Host ^(?P<workspacePort>[0-9]{2,5})-(?P<workspaceID>[a-z0-9][0-9a-z\\-]+).ws(?P<location>-[a-z0-9]+)?.{$GITPOD_DOMAIN}\n\thandle
    @workspace_port {\n\t\treverse_proxy https://ws-proxy.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:9090
    {\n\t\t\timport workspace_transport\n\t\t\timport upstream_headers\n\n\t\t\theader_up
    X-Gitpod-WorkspaceId {re.host.workspaceID}\n\t\t\theader_up X-Gitpod-Port        {re.host.workspacePort}\n\t\t\theader_up
    X-WSProxy-Host       {http.request.host}\n\t\t}\n\t}\n\n\t@workspace \theader_regexp
    host Host ^(?P<workspaceID>[a-z0-9][0-9a-z\\-]+).ws(?P<location>-[a-z0-9]+)?.{$GITPOD_DOMAIN}\n\thandle
    @workspace {\n\t\treverse_proxy https://ws-proxy.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:9090
    {\n\t\t\timport workspace_transport\n\t\t\timport upstream_headers\n\n\t\t\theader_up
    X-Gitpod-WorkspaceId {re.host.workspaceID}\n\t\t\theader_up X-WSProxy-Host       {http.request.host}\n\t\t}\n\t}\n\n\t#
    foreign content route used by vscode to serve webview and webworker resources
    of the form\n\t# https://{{hash_base_32}}.<cluster>.<gitpod_domain> or https://v--{{hash_base_32}}.<cluster>.<gitpod_domain>\n\t#
    e.g:\n\t# https://0d9rkrj560blqb5s07q431ru9mhg19k1k4bqgd1dbprtgmt7vuhk.ws-us34xl.gitpod.io
    (for webviews)\n\t# https://v--0d9rkrj560blqb5s07q431ru9mhg19k1k4bqgd1dbprtgmt7vuhk.ws-us34xl.gitpod.io
    (for webworker)\n\t# origin should be decoupled from the workspace (port) origin
    but the workspace (port) prefix should be the path root for routing\n\t@foreign_content2
    header_regexp host Host ^(?:v--)?[0-9a-v]+.ws(-[a-z0-9]+)?.{$GITPOD_DOMAIN}\n\thandle
    @foreign_content2 {\n\t\treverse_proxy https://ws-proxy.{$KUBE_NAMESPACE}.{$KUBE_DOMAIN}:9090
    {\n\t\t\timport workspace_transport\n\t\t\timport upstream_headers\n\n\t\t\theader_up
    X-WSProxy-Host       {http.request.host}\n\t\t}\n\t}\n\n\trespond \"Not found\"
    404\n}\n\nimport /etc/caddy/vhosts/vhost.*"
kind: ConfigMap
metadata:
  name: proxy-enhancement
  namespace: gitpod
```
2. Create a new storage volume in the Proxy deployment that points to the new configmap we created. Path is /etc/caddy/Caddyfile
3. Still missing some options to fully enable the features


