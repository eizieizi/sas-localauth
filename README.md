<div id="top"></div>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://www.thalesgroup.com/">
    <img src="./images/thales.png" alt="Logo" width="250" height="150">
  </a>
  <h3 align="center">SAS Local User Authentication</h3>

  <p align="center">
    · <a href="https://github.com/eizieizi/sas-localauth/issues">Report Bug</a>
    · <a href="https://github.com/eizieizi/sas-localauth/issues">Request Feature</a>
  </p>
</div>

<br/>

## About the Safenet Authentication Service Deployment
<br>
Thales provides a customized docker image which contains a freeradius instance which connects to the customer SAS tenant to validate MFA tokens. Unfortunatley the FreeRadius Integration is not very well done because

- SMS/Push OTP is not triggered automatically after validating credentials
- The deployment for multiple customers does not scale very well <br><br>

This docker-compose deployment spins up a additional FreeRadius server which will takeover the on-premise username and password validation and then afterwards triggers a push/sms with the vanilla Thales FreeRadius container.

The additional FreeRadius container allows to either specify a push/sms only configuration or a hybrid configuration where a push/sms is triggered after successful LDAP authentication or in case of using the MobilePass app the OTP can be supplied within the password.

To enable this, specify the  "CUSTOMER_DELIMITER" enviroment variable.<br>

```bash
CUSTOMER_DELIMITER="-" #Empty variable is push/sms only
``` 
<br>

## Getting Started
<br/>
To run the container, copy this repository to the desired radius server and edit the docker file/ enviroment variables.
<br/>
<br/>
Configurable valuse for the original thales container, named **"FreeRADIUSv3"** in docker-compose file.

**Do not edit any mounted config files manually.** <br><br>
```bash
container_name: FreeRADIUSv3_customer1
TV_URL: "https://cloud.eu.safenetid.com/TokenValidator/TokenValidator.asmx" #In case you are not in EU Cloud..
BACKEND_SHARED_SECRET: "*****" #Should be different for every docker compose instance and different for every customer - Needs to be the same in the FreeRADIUSv3 container
```

Configurable valuse for the addtional FreeRadius container, named **"freeradius-ldap"** in docker-compose file. <br><br>

```bash
container_name: freeradius-ldap_customer1
ports:
    - '192.168.41.102:1645:1645/udp' #Use secondary IPs on Host if running multiple containers with same port

CUSTOMER_REALM_NAME: "customername" #Necessary for every customer to allow SAS to route the request into the right tenant / VS - User never sees the realm / must also be specified in SAS Console!
CUSTOMER_ALLOWED_IPRANGE: "0.0.0.0/0" #Eventually, restrict Radius Access per customer IP
CUSTOMER_SHARED_SECRET: "********"
CUSTOMER_DELIMITER: "-" #Specify if u want to allow customer to add OTP directly in the Password (Example: PW-OTP, Pa$$word-565841) to prevent challenge (when using MobilePass) the delimter must not appear in the PW of Users -empty variable does not enable the feature
BACKEND_SHARED_SECRET: "********" #Should be different for every docker compose instance and different for every customer - Needs to be the same in the FreeRADIUSv3 containe
LDAP_SERVER_1: "ldaps://*****.eizi.at" #Use "dc01.eizi.at" for LDAP and ldaps://dc01.eiz.at" for LDAPs
LDAP_SERVER_2: "ldaps://*****.eizi.at"
LDAP_PORT: 636 #Use Port 636 for LDAPs
LDAP_BIND_DN: "CN=*****,OU=*****,OU=*****,OU=*****,DC=eizi,DC=at"
LDAP_BIND_PW: "**********"
LDAP_BASE_DN: "DC=eizi,DC=at"
LDAP_USER_SCHEME: "sAMAccountName" # "sAMAccountName" or "userPrincipalName"

```
<br>

```bash
ToDo for new customers:

#ToDo´s for new customers.

#1) Copy this folder and rename it to SASfreeRadius-customerNAME

#2) Change the enviroment variables as desired
  #2a) In Case of using LDAPs add the right ca-chain to the file ./cacerts/cacerts.pem - in base64 - no binary format (.der)
  #2b) Take note of CUSTOMER_REALM_NAME - you have to add it to the SAS Console configuration for proper radius request routing
  #2c) All enviroment variables which are used in multiple containes/services must have the same values ($BACKEND_SHARED_SECRET)

#3) Assign per customer additional host IPs to the server over the interface config and bind them afterwards to the two containers (Example: '192.168.41.101:1645:1645/udp')
#4) Copy the whole folder to the second server for and change the  interface config and bind them afterwards to the two containers (Example: '192.168.51.101:1645:1645/udp')
#3) Run docker-compose up in the base customer folder on both servers to activate the radius service.

```

After editing / inserting the right values, start the deployment with <br>
```bash
docker-compose up
```
