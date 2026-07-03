# -IKEv2-Configure-una-VPN-site-to-site-punto-a-punto-basado-en-pol-ticas
Lista de Comandos Solos para el Laboratorio
Direccionamiento en las PCs (VPCS)
PC1:

Plaintext
ip 10.11.13.2 255.255.255.128 10.11.13.1
save
PC2:

Plaintext
ip 10.11.13.130 255.255.255.128 10.11.13.129
save
Configuración en R2 (Peer 1)
Plaintext
configure terminal

interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 no shutdown

interface GigabitEthernet1/0
 ip address 10.11.13.1 255.255.255.128
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 192.168.12.1

crypto ikev2 proposal IKEV2_PROP 
 encryption aes-cbc-256
 integrity sha256
 group 14

crypto ikev2 policy IKEV2_POL
 proposal IKEV2_PROP

crypto ikev2 keyring VPN_KEYRING
 peer R3_PEER
  address 192.168.13.3
  pre-shared-key cisco123

crypto ikev2 profile IKEV2_PROFILE
 match identity remote address 192.168.13.3 255.255.255.255
 identity local address 192.168.12.2
 authentication remote pre-share
 authentication local pre-share
 keyring local VPN_KEYRING

ip access-list extended ACL_VPN_TRAFFIC
 permit ip 10.11.13.0 0.0.0.127 10.11.13.128 0.0.0.127

crypto ipsec transform-set TS_ESP esp-aes 256 esp-sha256-hmac 
 mode tunnel

crypto map CMAP_IKEV2 10 ipsec-isakmp 
 set peer 192.168.13.3
 set transform-set TS_ESP 
 set ikev2-profile IKEV2_PROFILE
 match address ACL_VPN_TRAFFIC

interface FastEthernet0/0
 crypto map CMAP_IKEV2
exit
Configuración en R3 (Peer 2)
Plaintext
configure terminal

interface FastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
 no shutdown

interface GigabitEthernet1/0
 ip address 10.11.13.129 255.255.255.128
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 192.168.13.1

crypto ikev2 proposal IKEV2_PROP
 encryption aes-cbc-256
 integrity sha256
 group 14

crypto ikev2 policy IKEV2_POL
 proposal IKEV2_PROP

crypto ikev2 keyring VPN_KEYRING
 peer R2_PEER
  address 192.168.12.2
  pre-shared-key cisco123

crypto ikev2 profile IKEV2_PROFILE
 match identity remote address 192.168.12.2 255.255.255.255
 identity local address 192.168.13.3
 authentication remote pre-share
 authentication local pre-share
 keyring local VPN_KEYRING

ip access-list extended ACL_VPN_TRAFFIC
 permit ip 10.11.13.128 0.0.0.127 10.11.13.0 0.0.0.127

crypto ipsec transform-set TS_ESP esp-aes 256 esp-sha256-hmac 
 mode tunnel

crypto map CMAP_IKEV2 10 ipsec-isakmp 
 set peer 192.168.12.2
 set transform-set TS_ESP 
 set ikev2-profile IKEV2_PROFILE
 match address ACL_VPN_TRAFFIC

interface FastEthernet0/0
 crypto map CMAP_IKEV2
exit
Comandos de Verificación (Ejecutar en R2 o R3 en modo #)
Plaintext
ping 10.11.13.130
show crypto ikev2 sa
show crypto ipsec sa
show crypto map
show crypto session detail
