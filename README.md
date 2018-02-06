[![Build Status](https://travis-ci.org/flysen/shibboleth.svg?branch=master)](https://travis-ci.org/flysen/shibboleth)

Shibboleth
=========

A template role to handle SSO against swamid for higher education in Sweden, shibboleth authentication with Apache service.

Requirements
------------

Beside Shibboleth, this role will install Apache (httpd) with mod_ssl extension.

Role Variables
--------------

```
applicationdefaults_entityid: "{{ inventory_hostname }}"
sso_entityid: "https://weblogin.uu.se/idp/shibboleth"
errors_supportcontact: "root@{{ inventory_hostname }}"
metadataprovider_uri: "https://mds.swamid.se/md/swamid-idp.xml"
metadataprovider_backingfilepath: "swamid-testing-idp.xml"
```

I recommend to use a CNAME to ```applicationdefaults_entityid```.

Dependencies
------------

Apache with mod_ssl which this role make sure exist. 
Also make sure that firewalld are open, it is not taken care of in this role. I suggest to add a rich-rule for that in the playbook when testing:
```
## open firewall for httpds
  - name: Open firewalld for service https for its-uu-net in public zone
    firewalld:
      rich_rule: 'rule family="ipv4" source address=130.238.xxx.0/24 service name="https" accept'
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
      applicationdefaults_entityid: 'https://SERVER.uu.se'
      sso_entityid: 'https://weblogin.uu.se/idp/shibboleth' 
      errors_supportcontact: 'my@email.se'
      metadataprovider_uri: 'https://mds.swamid.se/md/swamid-idp.xml'
      metadataprovider_backingfilepath: 'swamid-testing-idp.xml'
```

Post Configuration
------------------

When shibboleth installed via anslible-playbool some configuration steps need to be taken:
1. Change entityID ```applicationdefaults_entityid``` in /etc/shibboleth/shibboleth2.xml to match your service
2. generate metadata template that you need to send to operation@swamid.se
3. Add some extra tags in the metadata to make Swamid happy
4. Validate your metadata file
5. Send it to Swamid

## entityID

The entityID will probably be the CNAME of the server ```applicationdefaults_entityid```

## Metadata

The metadata can easily be generated via ``` /etc/shibboleth/metagen.sh -c sp-cert.pem -h MY_CNAME.ub.uu.se ```. The output will then be displayed to stout in the terminal. Copy that to a new textfile for example named metadata.xml, we will use that later.

## Extra metadata

To meet Swamids requiments we need to add some extensions to previous created file metadata.xml. 
* First we need to tell what attributes (federation) we will use. Se full example in this role example folder (metadata.xml).
* We also need to add some information regarding our purpose and organisation Se full example in this role example folder (metadata.xml).

## validate metadata.xml

https://www.samltool.com/validate_xml.php and use schema Metadata

## Send to Swamid

Attache the validated metadata.xml to swamid@operation.se

To Do
-----

* Validate generated metadata against xsd with xmllint
* Add the extra block of xml with xmlstarlet and revalidate the xml against xsd

License
-------

BSD

Author Information
------------------
Fredrik Lysén
