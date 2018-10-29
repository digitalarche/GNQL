# GNQL
GNQL (GreyNoise Query Language) is a domain-specific query language that uses Lucene deep under the hood. GNQL aims to enable GreyNoise Enterprise and Research users to make complex and one-off queries against the GreyNoise dataset as new business cases arise. GNQL is built with self-defeat and fully featured product lines in mind. If we do our job correctly, each individual GNQL query that brings our users and customers sufficient value will eventually be transitioned into it's own individual offering. 

## Problem
- GreyNoise collects lots and lots of data
- GreyNoise data is composed of many different data structures / features / schemas
- Different users/organizations have different use-cases / requirements for the data
- GreyNoise developers have a limited amount of time/resources to devote for building API endpoints that satisfy every use-case

# Getting started
We've included a small command line tool that takes raw queries from the CLI and spits out raw JSON. This will need to be piped to `jq` or spit into a file for processesing elsewhere. I know it sucks right now but bear with me. 

You can use GNQL to query IPs, tags, CIDR blocks, ASNs, and any other field directly, or create complex queries by adding and subtracting criteria.

## Examples


Return all devices with the "Mirai" tag

```
$ gnql tags:Mirai
```

Return all devices with the "RDP Scanner" tag

```
$ gnql tags:"RDP Scanner"
```

Return all compromised devices located in Belgium

```
$ gnql classification:malicious metadata.country:Belgium
```

Return all compromised devices that include `.gov` in their reverse DNS records

```
$ gnql classification:malicious metadata.rdns:*.gov*
```

Return all compromised devices that belong to Microsoft

```
$ gnql metadata.organization:Microsoft classification:malicious
```

Return all devices scanning the Internet for port 445/TCP running Windows operating systems (Conficker/EternalBlue/WannaCry)

```
$ gnql "(raw_data.scan.port:445 and raw_data.scan.protocol:TCP)" metadata.os:Windows*
```

Return all devices scanning the Internet for port 554

```
$ gnql raw_data.scan.port:554
```

Return all devices crawling the Internet with "GoogleBot" in their useragent from a network that does NOT belong to Google

```
$ gnql -metadata.organization:Google raw_data.web.useragents:GoogleBot
```

Return all devices scanning the Internet for SCADA devices who ARE NOT tagged by GreyNoise as "benign" (Shodan/Project Sonar/Censys/BinaryEdge/Google/Bing/etc)

```
$ gnql tags:"Siemens PLC Scanner" -classification:benign
```

Return all "good guys" scanning the Internet

```
$ gnql classification:benign
```

Return all devices crawling the Internet with a matching client JA3 TLS/SSL fingerprint

```
$ gnql raw_data.ja3.fingerprint:795bc7ce13f60d61e9ac03611dd36d90
```

Return all devices crawling the Internet for the HTTP path "/HNAP1/"

```
$ gnql raw_data.web.paths:"\/HNAP1\/"
```

## Requirements
Python requests
```
$ pip install requests
```

## Installation

```
$ git clone http://github.com/GreyNoise-Intelligence/GNQL

```

## GNQL Field Reference

```
ip - The IP address of the scanning device IP
classification - Whether the device has been categorized as unknown, benign, or malicious
first_seen - The date the device was first observed by GreyNoise
last_seen - The date the device was most recently observed by GreyNoise
actor - The benign actor the device has been associated with, such as Shodan, GoogleBot, BinaryEdge, etc
tags - A list of the tags the device has been assigned over the past 90 days
metadata.country - The full name of the country the device is geographically located in
metadata.country_code - The two-character country code of the country the device is geographically located in
metadata.city - The city the device is geographically located in
metadata.organization - The organization that owns the network that the IP address belongs to
metadata.rdns - The reverse DNS pointer of the IP
metadata.asn - The autonomous system the IP address belongs to
metadata.tor - Whether or not the device is a known Tor exit node
metadata.os - The operating system the device is running
metadata.category - Whether the device is a business, isp, or hosting provider
raw_data.scan.port - The port number(s) the devices has been observed scanning
raw_data.scan.protocol - The protocol of the port the device has been observed scanning
raw_data.web.paths - Any HTTP paths the device has been observed crawling the Internet for
raw_data.web.useragents - Any HTTP user-agents the device has been observed using while crawling the Internet
raw_data.ja3.fingerprint - The JA3 TLS/SSL fingerprint
raw_data.ja3.port - The corresponding TCP port for the given JA3 fingerprint

```
## Record format

```
{
  "ip": "54.90.107.240",
  "seen": true,
  "classification": "unknown",
  "first_seen": "2018-10-19",
  "last_seen": "2018-10-19",
  "actor": "unknown",
  "tags": [
    "Web Crawler",
    "HTTP Alt Scanner"
  ],
  "metadata": {
    "country": "United States",
    "country_code": "US",
    "city": "Ashburn",
    "organization": "Amazon Technologies Inc.",
    "rdns": "ec2-54-90-107-240.compute-1.amazonaws.com",
    "asn": "AS14618",
    "tor": false,
    "os": "unknown",
    "category": "hosting"
  },
  "raw_data": {
    "scan": [
      {
        "port": 8443,
        "protocol": "TCP"
      }
    ],
    "web": {
      "paths": [
        "/"
      ],
      "useragents": [
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
      ]
    },
    "ja3": [
      {
        "fingerprint": "4d7f0c0bee5049725935d4fafbbd507c",
        "port": 8443
      }
    ]
  }
}
```


## Tips
- GNQL hates unescaped forward slashes. Please escape your forward slashes with backslashes when using the CLI
- Certain GNQL fields need to be put into quotes, like IP
- You can query CIDR blocks by doing `"ip:1.0.0.0\/8"`. Please note the escaped slash and quotes
- You can free text search fields by querying `whatever:"*something*"` or search things that start with a string by querying `whatever:"starts with*"`


## API

You can query the GNQL API directly by issuing a URL-encoded HTTP GET request to `https://research.api.greynoise.io/v2/experimental/gnql?query=your_gnql_query_string_here`

The response format is as follows:
```
{
  "count": 0,                   // Total number of IPs matching your GNQL query (not to be confused with total number returned)
  "data": [],                   // response data
  "message": "ok",              // message confirming your request was handled okay
  "query": "your_query_here"    // your GNQL query string
}
```


## Bugs

Please file GNQL bug reports as issues on this GitHub repository. Please use this page specifically for the GNQL API endpoint.

## Feature Requests

Please file GNQL feature requests as issues on this GitHub repository. 

## TODOs

## Notes

- The data updates every hour on the hour
- The API currently returns 10,000 results maximum per request. I still need to implement pagination
- We're in the process of building a much more verbose API that returns datetime
- This service is only loaded with data starting on 10/13/2018. If this feature provides value then we will retroload the full GreyNoise dataset

## Getting access

If you are a GreyNoise Enterprise customer or a GreyNoise Research/Beta member you already have access. If you want to become a GreyNoise Enterprise customer, please email sales@greynoise.io or visit https://greynoise.io/enterprise. If you wish to join our research community, please email hello@greynoise.io. 
