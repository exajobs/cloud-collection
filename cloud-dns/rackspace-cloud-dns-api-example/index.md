---
permalink: rackspace-cloud-dns-api-example
audit_date: '2021-06-02'
title: Rackspace Cloud DNS&mdash;API example
type: article
created_date: '2011-10-19'
created_by: Rackspace Support
last_modified_date: '2021-06-02'
last_modified_by: Ana Corpus
product: Cloud DNS
product_url: cloud-dns
---

**Previous section:** [Rackspace Cloud DNS - Technical details](/support/how-to/rackspace-cloud-dns-technical-details)

This article shows a simple example using the Application Programmer Interface (API).

Every ReST request against the DNS Service requires the inclusion of a
specific authorization token supplied by the **X-Auth-Token** HTTP header.
You can obtain this token by first using the Rackspace Cloud
Authentication Service and supplying a valid Rackspace Cloud account
username and API access key.

**Authentication request**

    curl -D - -H "X-Auth-Key: a86850deb2742ec3cb41518e26aa2d89" -H "X-Auth-User: jdoe" https://auth.api.rackspacecloud.com/v1.0

**Authentication response**

    HTTP/1.1 204 No Content
    (... more ...)
    X-Auth-Token: eaaafd18-0fed-4b3a-81b4-663c99ec1cbb

After authentication, you can send requests to an available API endpoint.
Our DNS service is regionalized, and DNS is therefore
responsible for appropriate replication, caching, and overall
DNS data maintenance across regional boundaries to other DNS servers.
We have endpoints available in the US and the UK.

-   US `https://dns.api.rackspacecloud.com/v1.0/1234`
-   UK `https://ion.dns.api.rackspacecloud.com/v1.0/1234`

### List domain details

Next, consider this simple call to the API. This example shows
a **List domain details** request. The request calls the US endpoint at
**dns.api.rackspacecloud.com** and requests
the domains for account number **1234** and the domain ID **2725511**.

    curl -X GET -H "X-Auth-Token:eaaafd18-0fed-4b3a-81b4-663c99ec1cbb" -H "Accept:application/xml" https://dns.api.rackspacecloud.com/v1.0/1234/domains/2725511

Following is an example XML response from the **List domains** API call,
but the DNS API supports both the JSON and XML data serialization formats.
You specify the response format in requests by using either the
**Accept Header** or adding a **.xml** or **.json** extension to the
request URI. If you specify no response format, the system uses JSON
by default. If you have conflicting formats by using both
an **Accept Header** and a query extension, the query extension takes
precedence.

    <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <domain xmlns="https://docs.rackspacecloud.com/dns/api/v1.0"
        id="2725511" accountId="1234" name="rtb0007.com" ttl="3600" emailAddress="sample@rackspace.com" updated="2011-06-24T01:23:15Z" created="2011-06-24T01:12:51Z" comment="Optional domain comment...">
        <nameservers>
            <nameservername="ns.rackspace.com" />
            <nameservername="ns1.rackspace.com" />
        </nameservers>
        <recordsListtotalEntries="5">
            <record id="A-6822994" type="A" name="rtb0007.com" data="110.11.12.16"  ttl="86400" updated="2011-06-24T01:12:52Z" created="2011-06-24T01:12:52Z" />
            <record id="NS-6251982" type="NS" name="rtb0007.com" data="ns.rackspace.com" ttl="3600" updated="2011-06-24T01:12:51Z"  created="2011-06-24T01:12:51Z" />
            <record id="NS-6251983" type="NS" name="rtb0007.com" data="ns2.rackspace.com" ttl="3600" updated="2011-06-24T01:12:51Z"  created="2011-06-24T01:12:51Z" />
            <record id="MX-3151218" type="MX" name="rtb0007.com"  data="mail2.rtb0007.com" ttl="3600" priority="5" updated="2011-06-24T01:12:53Z" created="2011-06-24T01:12:53Z" />
            <record id="CNAME-9778009" type="CNAME" name="www.rtb0007.com"  data="rtb0007.com" ttl="5400" updated="2011-06-24T01:12:54Z"       created="2011-06-24T01:12:54Z" />
        </recordsList>
    </domain>

**Next steps:** [Rackspace Cloud DNS - Additional resources](/support/how-to/rackspace-cloud-dns-additional-resources)
