[TOC]

Introduction
===========

Run a captive portal on your raspberry (or any linux box) to allow your guests to register before accessing your Wifi at home. 
Users will be requested for an OTP code that you can generate on your phone. Get rid of the typical captive portal static 
username and password without the need for a radius server.

OTPspot (since version 2.0) is fully compatible with nodogsplash and can run as a FAS service. In this configuration, 
nodogsplash will take care of the networking whereas OTPspot will just authenticate the users.

How it works
---------------------

Once a user connects to the guest wireless network, nodogsplash will make te device detecting additional credentails are required and present the user with the OTPspot web-based and responsive captive portal where an access code is requested for
the authentication. 

This code is generated by the Google Authenticator/Authy app running on the host's phone and aligned with a service running on the raspberry used to validate the authentication. This requires the google-authenticator service running on the raspberry and
a custom pam service which is provided by OTPspot and installed automatically.

Once a valid OTP is provided the portal will notify nodogsplash the user has successfully authenticated and Internet access
will be then granted. 

Installation
===========

nodogsplash
---------------------

To install and configure nodogsplash on a Openwrt-based router:
- Refresh the list of packages with: opkg update
- Install nodogsplash with: opkg install nodogsplash2
- If you want your guests to connect to a dedicated guest wireless network:
	- Add a new, open wifi network in Network->Wireless
	- Add an interface with a static IP address in Network->Interfaces
	- Map the interface to the newly created wifi network 
	- Enable the DHCP service for this interface
- Edit nodogsplash configuration file in /etc/config/nodogsplash
	- To run on the newly created wifi interface only add: option gatewayinterface 'wlan0-1'
	- To allow connections from preauthenticated users to the captive portal running on the raspberry add: list preauthenticated_users 'allow tcp port 8000 to <raspberry_ip>'
- Edit nodogsplash captive portal by editing /etc/nodogsplash/htdocs/splash.html:
	- To redirect users to the OTPspot captive portal add within the <head> section the following: <meta http-equiv="refresh" content="0;URL='http://<raspberry_ip>:8000/index.html?authaction=$authaction&amp;tok=$t
ok&amp;redir=$redir&amp;mac=$clientmac&amp;ip=$clientip&amp;gatewayname=$gatewayname'" />

OTPspot
---------------------

- Create on the raspberry a directory of your choice and transfer all the files of the package
- As root, run the install.py script. This will install all the required dependencies, the init and the pam services.
- Download, compile and install Google Authenticator from https://github.com/google/google-authenticator
- Create a user (e.g. adduser otpspot) and set it a valid password
- Become that user (e.g. su otpspot)
- Run google-authenticator
- On your phone, install Google Authenticator or Authy from the app store
- Scan the bar code on the screen to allow Google Authenticator on your phone to generate valid OTP codes. The same can be installed on multiple phones
- Edit the configuration file config.py and restart the service
	

Configuration
===========
- Edit the config.py file:
	- Customize if needed the port the captive portal runs on
	- Customize the username and password for the user on the system associated with the Google Authenticator service
	- Customize if needed the HTML template based on your language

Changelog
===========
- v2.0:
	- Added support for nodogsplash
	- Removed support for hostapd and tp-link router
	- Added support to customize the portal's language
	- Added support for logging on file
- v1.0:
	- Support for hostapd running on the raspberry
	- Support for creating MAC expections on a tp-link router
