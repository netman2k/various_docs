# Import CentOS errata into pulp

* [pulp_centos_errata_import](https://github.com/rdrgmnzs/pulp_centos_errata_import)

### Installing requires
```
yum install -y perl-Text-Unidecode perl-XML-Simple git bunzip2
```

### Installing pulp_centos_errata_import script
```
cd /usr/local/src
git clone https://github.com/brdude/pulp_centos_errata_import.git
chmod +x /usr/local/src/pulp_centos_errata_import/errata_import.pl
 
ln -s /usr/local/src/pulp_centos_errata_import/errata_import.pl /sbin/errata_import.pl
 
cat <<'EOF' > /usr/local/bin/import_centos_errata.sh
#!/bin/bash
 
wget -q -N -P /tmp http://cefs.steve-meier.de/errata.latest.xml.bz2
wget -q -N -P /tmp https://www.redhat.com/security/data/oval/com.redhat.rhsa-all.xml.bz2
 
bunzip2 -f /tmp/errata.latest.xml.bz2
bunzip2 -f /tmp/com.redhat.rhsa-all.xml.bz2
 
/sbin/errata_import.pl --errata=/tmp/errata.latest.xml --rhsa-oval=/tmp/com.redhat.rhsa-all.xml
EOF
 
chmod u+x /usr/local/bin/import_centos_errata.sh
```


### Import
```
/usr/local/bin/import_centos_errata.sh
```

