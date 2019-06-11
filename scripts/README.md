# Microchip Secure Element Tools

These tools are used to set up an example chain of trust ecosystem for use with
Microchip ATECC508A and ATECC608A parts.   
Included are utilities to create the ecosystem keys and certificates.

## Dependencies

Python scripts will require python 3 to be installed.   
Once python is installed
install the requirements (from the path of this file):

```
> pip install -r requirements.txt
```

## Set up a Certificate Ecosystem

The first step is to set up a certificate chain that mirrors how a secure iot
ecosystem would be configured.  
For this we'll create a dummy root certificate
authority (normally this would be handled by a third party, or an internal
PKI system) and an intermediate (Signing) certificate authority.

### Create the Root CA
This command interactively enters the organization name and common name, and creates the related certificate and private key.
```
> create_root.py 
organization name:<your own name>
common name:<your own common name>

```

### Create the Signing CA
This command interactively enters the organization name and common name, and creates the related certificate and private key.  
make sure not to set the same name as accepted by "create_root.py".
```
> create_signer.py
organization name:<your own name>
common name:<your own common name>
```

### Create the Device Certificate

1) Output from [ECC608-Configure](https://github.com/kmwebnet/ECC608-Configure) program would be the public key required for an
certificate:

```
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEojEQk85EaT1RU3Ip5SddaSqB5/Wm
+Vnxtu96G3i+gQRb8tb5xylXTXHQawL68SPW4/oCXXS4x7KGV0MNPneB6g==
-----END PUBLIC KEY-----
```

2) Save this key as public_key.pem for use in the device certificate creation

3) Run the create_device script:
This command interactively enters the organization name and common name, and creates the related certificate and private key.  
you may use device unique ID output from [ECC608-Configure](https://github.com/kmwebnet/ECC608-Configure) program as device common name.
```
> create_device.py --deveicekey public_key.pem
organization name:testcorp
common name:0123xxxxxxxxxxxxee
```

This performs the certificate creation and signing process but does not load the
certificates into the device (provisioning).

### Create certificate definition file
```
> cert2certdef.py --signer-cert signer-ca.crt >> cert_chain.c
> cert2certdef.py --device-cert device.crt >> cert_chain.c
```
This creates the file that is used by provision.c as certificate definitions to save the generated
certificates into the secure element. 

### Export the provision.h file
```
> export_header.py
```
This creates the file that is used by provision.c to save the generated
certificates into the secure element. 

### Run the provisioning

A build and run [ECC608-Provision](https://github.com/kmwebnet/ECC608-Provision) will write the certificates into
the device.
