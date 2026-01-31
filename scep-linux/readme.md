Prereqs

`dnf install ca-certificates certmonger`

copy the ca chain to trust store

```
cp`chain.pem /etc/pki/ca-trust/source/anchors
update-ca-trust extract
```

copy certs into certmonger
```
mkdir /var/lib/certmonger/certs
cp chain.pem scep.pem /var/lib/certmonger/certs/
chmod 644 /var/lib/certmonger/certs/*
```




Requestion certs from the SCEP server.


- -R = scep cert
- -N = root and sub cert

`getcert add-scep-ca -u https://scep.int.domain.com:8443/scep -R /var/lib/certmonger/certs/scep.pem -c nickname -N /var/lib/certmonger/certs/chain.pem`


- -c name used to identify SCEP from above
- -k where to store the private key
- -f where to store the public key
- -u what you intend to use the key for 
- -N hostname
- -L key from SCEP server

`getcert request -c nickname -k /etc/pki/private/$HOSTNAME.key -f /etc/pki/certs/$HOSTNAME.crt -u digitalSignature -u keyEncipherment -N $HOSTNAME -L key`
