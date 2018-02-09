[![Build Status](https://travis-ci.org/flysen/shibboleth.svg?branch=master)](https://travis-ci.org/flysen/shibboleth)

Shibboleth
=========

A template role to handle SAML SSO within the SWAMID federation for higher education in Sweden, using shibboleth authentication with Apache service.

Requirements
------------

Besides Shibboleth, this role will install Apache (httpd) with mod_ssl extension.

Role Variables and their default values
--------------
Metadata in file "/etc/shibboleth/metadata.xml"
```
applicationdefaults_entityid: "{{ inventory_hostname }}"
sso_entityid: "https://IdP.URL/idp/shibboleth"
errors_supportcontact: "root@{{ inventory_hostname }}"
metadataprovider_uri: "https://mds.swamid.se/md/swamid-idp.xml"
metadataprovider_backingfilepath: "swamid-testing-idp.xml"
```
Extensions in file "/etc/shibboleth/extensions.xml"
```
# Entity Categories for Service Providers. (*MUST*)
attributevalue: ['https://provider/category/one', 'https://provider/category/two']
# Friendly name of the Service Provider, shall not be a domain name. (*SHOULD*)
displayname: "My Higer Education and University"
# A shorter description (140 characters or less) of the Service Provider. (*MAY*)
description: "Applicationserver with limited access"
# A URL to a web-page that complements the description with further information about the service that the Service Provider offers. (*MAY*)
informationurl: "https://github.com"
# A URL to the a image file of the service logotype. (*MAY*)
logo: "https://www.example.se/images/logo.png"
```

I recommend to use a CNAME to ```applicationdefaults_entityid```.

Dependencies
------------

Apache with mod_ssl which this role make sure exist. 
Also make sure that firewalld are open, it is not taken care of in this role. I suggest to add a rich-rule for that in the playbook when testing:
```
## open firewall for httpds
  - name: Open firewalld for service https in the public zone
    firewalld:
      rich_rule: 'rule family="ipv4" source address=xxx.xxx.xxx.0/24 service name="https" accept'
      zone: public
      permanent: true
      state: enabled
      immediate: yes
```
And when moving into production you maybe open for rest of the world using service https:
```
- name: Add https to zone public and make it permanent
  firewalld:
    zone: public
    service: https
    permanent: true
    state: enabled
```

Example Playbook
----------------

```
---
- hosts: localhost
  remote_user: root
  tasks:
  - include_role:
      name: shibboleth
    vars:
      applicationdefaults_entityid: 'https://SP.URL'
      sso_entityid: 'https://IdP.URL/idp/shibboleth' 
      errors_supportcontact: 'my@email.se'
      metadataprovider_uri: 'https://mds.swamid.se/md/swamid-idp.xml'
      metadataprovider_backingfilepath: 'swamid-testing-idp.xml'
      attributevalue: ['https://provider/category/one', 'https://provider/category/two']
      displayname: "My Higer Education and University"
      description: "Applicationserver with limited access"
      informationurl: "https://github.com"
      logo: "https://www.example.se/images/logo.png"
```

Post Configuration
------------------

When shibboleth installed via anslible-playbook your configuration are done and no further steps need to be taken.
1. Validate your metadata file /etc/shibboleth/metadata.xml
2. Grab the file /etc/shibboleth/metadata.xml and send to operation@swamid.se


## Metadata

The metadata can easily be regenerated via ``` /etc/shibboleth/metagen.sh -c sp-cert.pem -h MY_CNAME.URL ```. The output will then be displayed to stout in the terminal. Copy that to a new textfile for example named metadata.xml. Note that /etc/shibboleth/extensions.xml vill not be included then.

## Extra metadata

To meet Swamids requiments we need to add some extensions. Those are created by ansible and found in /etc/shibboleth/extensions.xml. 

## validate /etc/shibboleth/metadata.xml

https://www.samltool.com/validate_xml.php and use schema Metadata

To Do
-----

Add Logotype URL (<mdui:Logo>) *MAY* https://wiki.sunet.se/pages/viewpage.action?pageId=17138098 (not mandatory)

License
-------

BSD

Author Information
------------------
Lords of the Shib
