
**WORK IN PROGRESS**


## Copying Easy RSA content
```
mkdir -p /config/auth/openvpn/easy-rsa; cp -R /usr/share/doc/openvpn/examples/easy-rsa/2.0/* /config/auth/openvpn/easy-rsa/
pushd /config/auth/openvpn/easy-rsa
```

## Changing the default variables
```
sed -i 's/^\(export\ KEY_SIZE=\).*$/\12048/' vars
sed -i 's/^\(export\ KEY_COUNTRY="\).*$/\1KR"/' vars
sed -i 's/^\(export\ KEY_PROVINCE="\).*$/\1Seoul"/' vars
sed -i 's/^\(export\ KEY_CITY="\).*$/\1Seoul"/' vars
sed -i 's/^\(export\ KEY_ORG="\).*$/\1Example Inc."/' vars
sed -i 's/^\(export\ KEY_EMAIL="\).*$/\1example@example.local"/' vars
echo 'export KEY_OU="Organization"' >> vars
echo 'export KEY_CN="ROOT CA"' >> vars
#echo 'export KEY_NAME="VyOS_FW"' >> vars
```

### Get ready for creating certificates
```
mkdir -p keys
touch keys/index.txt
echo 01 > keys/serial
source ./vars
./clean-all
```

### Building a CA certificate and key
> I used pkitool directly instead of using build-ca script which will
> show you the prompt to verify the variables
```
./pkitool --initca
```

### Building an Intermediate certificate authority certificate and key
```
./pkitool --server "$(hostname -f)"
```
### Building a DIFFIE-HELLMAN PARAMETERS
```
./build-dh
Generating a client certificate and key for any client
./pkitool client
```
Now copy these file to your PC or other systems

* keys/client.crt
* keys/client.key
* keys/ca.crt
