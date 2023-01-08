---
title: "KongPlugin CORS: fixing "Access-Control-Allow-Origin header missing" error"
date: 2021-12-07T17:55:37+01:00
draft: false
tags: ["kong", "kubernetes", "cors", "kong-plugin"]
---
In our current environment we have **Kong** as our Ingress controller in front of our applications.
 
We are also using the **Kong CORS Plugin** to enable browsers to make cross-origin requests to our application's backend. [The CORS plugin](https://docs.konghq.com/hub/kong-inc/cors/) lets you configure the API gateway behavior to support Cross-Origin Resource Sharing (CORS). If you want to dig deeper into what CORS is, please check the [CORS glossary link](https://developer.mozilla.org/en-US/docs/Glossary/CORS).

We also make use of `helm` charts to handle our deployment.

**The issue**
We had it set up to allow all origins and wanted to add our domains only. Adding the list of domains to the Kong CORS Plugin origins parameter resulted in "Access-Control-Allow-Origin header missing" error and CORS blocking the requests.

**The initial setup of our CORS plugin**

In order to get the plugin to work the first thing we needed to do is to:

#### Enable the plugin on a service
```yml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: service-cors-plugin
config: 
  origins:
  - '*'
  credentials: true
  max_age: 3600
  preflight_continue: false
plugin: cors
```

Apart from enabling it, we have a few other configurations set:

**credentials** - _required_ 
Documentation:
>_Flag to determine whether the `Access-Control-Allow-Credentials` header should be sent with **true** as the value._

What this means:
By default, CORS does not include cookies on cross-origin requests. If you need the requests to transfer cookies (or other user credentials) then it needs to be enabled. This entails that the server will allow cookies to be included on cross-origin requests.

For more details on what the `Access-Control-Allow-Credentials` header does, please check the [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials).

**origins** - _optional_
Documentation:
>_List of allowed domains for the `Access-Control-Allow-Origin` header._

What this means:
You can whitelist every domain that's allowed to call the API or allow everyone using the wildcard.
The latter was our initial setup and we'll come back to this.

**preflight_continue** - _required_
Documentation:
>_A boolean value that instructs the plugin to proxy the OPTIONS preflight request to the Upstream service._

What this means:
This flag decides if the preflight request should be handled by the upstream service (i.e. your API) or if the request is handled by the Ingress itself.

The purpose of a CORS preflight request is to check whether the CORS protocol is understood by the server for specific methods and headers.
The preflight request is an OPTIONS request, using three HTTP request headers: `Access-Control-Request-Method`, `Access-Control-Request-Headers`, and the `Origin` header.
If the server allows it, then it will respond to the preflight request with another request which contains the `Access-Control-Allow-Methods` response header, which lists the desired method (`PUT`, `DELETE`, etc).

We set it to **false** meaning we are letting Kong handle the preflight requests instead of passing them to the upstream service.

For more details on what the preflight request is, please check the [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request).

**max_age** - _optional_
Documentation:
>_The preflight response can be optionally cached for the requests created in the same URL using `Access-Control-Max-Age` header._

We set it to: 3600 seconds.

For the full list of configurations available, refer to the  [CORS plugin documentation](https://docs.konghq.com/hub/kong-inc/cors/#parameters).

After the plugin is configured, then:
#### Apply the Kong CORS plugin to the Ingress
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kong-service
  labels:
    <labels-omitted>
  annotations:
    <either add it here or in the values file depending on what you use>
spec:
  rules:
  - http:
      paths:
      - path: /path-to-your-service
        backend:
          serviceName: service-name
          servicePort: <service-port>
```
and
#### Reference the annotation to the CORS plugin in environment values file

```yml
ingress:
  annotations:
    kubernetes.io/ingress.class: "kong"
    konghq.com/plugins: "service-cors-plugin"
```
As I mentioned we use `helm` to handle our deploys so we have follow the basic structure (aside from Ingress and the specific project structure).
- a `deployment.yaml`: A basic manifest for creating a Kubernetes deployment
- a `service.yaml`: A basic manifest for creating a service endpoint for your deployment
- a `_helpers.tpl`: A place to put template helpers that you can re-use throughout the chart
- a `values.yaml` file that contains the default values for a chart. These values may be overridden by users during `helm install` or `helm upgrade`.

For more details on what `helm` is, charts and how to use them, please take a look over the [Helm Docs](https://helm.sh/docs/chart_template_guide/getting_started/). They are pretty well documented and can give you a good starting point if you are unfamiliar with this topic.

_But ..._

Allowing all origins is not a best practice unless you are implementing a public API so we wanted to limit the whitelisted domains.

Check [here](https://portswigger.net/web-security/cors#server-generated-acao-header-from-client-specified-origin-header) to see an example of how the headers can be exploited if you use a wildcard.

**Attempts**
* Failed attempt 1

We tried adding the domain using the ASP.Net Core flavor: `https://*.domain.com`, but it didn't work as the wildcards cannot be used within any other value. For example, the following header is not valid:
```
Access-Control-Allow-Origin: https://*.domain.com
```

In the plugin documentation it says that for the `origins` param:  
>_The accepted values can either be flat strings or PCRE regexes._

* Failed attempt 2

We tried adding a PCRE compliant regex enclosed in double quotes, but `helm` deploy failed as it rejected the format.

Adding all the domains with subdomains to a list was not feasible because it's something we register dynamically.

**The solution**

So what worked for us was: **adding the PCRE regex to whitelist all our subdomains, but enclosed in single quotes.** This allowed `helm` deploy to succeed.

To validate your regex you can use the awesome [regex101](https://regex101.com/). 
> Just make sure you select the PCRE engine flavor.

```yml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: service-cors-plugin
config: 
  origins:
  - 'https:\/\/[\w-_]+\.domain1\.com'
  - 'https://domain2.dev'
  - 'https:\/\/[\w-_]+\.domain2\.dev'
  credentials: true
  max_age: 3600
  preflight_continue: false
plugin: cors

```
Alternatively, if you use go templating and use a `_helpers.tpl` file then you can also define it as below:
```yml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: {{ include "service.name" . }}-cors-plugin
config: 
  origins:
  {{- range .Values.CorsAllowedOrigins }}
    - {{ . | squote }}
  {{- end }}
  credentials: true
  max_age: 3600
  preflight_continue: false
plugin: cors
```
If you are unfamiliar with what the `values.yaml` file actually is, it's just a place to put template helpers that you can re-use throughout the chart. 

For more details on how to read this, please check the [Helm Docs](https://helm.sh/docs/chart_template_guide/values_files/) on the Values files.

And in the `values.yaml` file just add your domains or your regex as:
```yml
CorsAllowedOrigins: 
  - 'https:\/\/[\w-_]+\.domain1\.com'
  - 'https://domain2.dev'
  - 'https:\/\/[\w-_]+\.domain2\.dev'
```

And that was that! :)

I wrote this blog post because all the examples from the internet were using the `'*'` format for origins and we wanted to allow the specific domains using a regex.

Hope it helps someone else!
