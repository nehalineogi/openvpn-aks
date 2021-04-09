## Client Configuration

Download and install OpenVPN client. This azure article explains how to setup OpenVPN Client for various platforms. [Use this as a reference only](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-openvpn-clients). Detailed steps may vary.

## Server Configuration

Launch the OpenVPN management portal: https://<IP_Address_openvpn_server>/admin

1. Setup Client VPN Pool
   ![alt text for image](images/openvpn-config.png)
2. Create a user named user-s
   ![alt text for image](images/openvpn-create-static-user.png)
3. Download the user-s profile and distribute it to the appropriate users.
