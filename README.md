# DZ02SLA
Настроить политику маршрутизации в офисе Чокурдах;
Распределить трафик между 2 линками.
```
Prov1#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                188.1.23.1      YES NVRAM  up                    up      
Ethernet0/1                unassigned      YES NVRAM  administratively down down    
Ethernet0/2                unassigned      YES NVRAM  administratively down down    
Ethernet0/3                unassigned      YES NVRAM  administratively down down 
```
```
Prov2#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                185.1.23.1      YES manual up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
```
```
Router#sh run
Building configuration...

Current configuration : 1550 bytes
!
! Last configuration change at 11:40:16 UTC Thu Feb 19 2026
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Router
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!         
!
!
!
!
!
!


!
!
!
!
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!         
!
!
redundancy
!
!
track 100 ip sla 100 reachability
 delay down 5 up 5
!
track 101 ip sla 101 reachability
 delay down 5 up 5
!
! 
!
!
!
!
!
!
!
!
!
!
!         
!
interface Ethernet0/0
 ip address 188.1.23.2 255.255.255.252
!
interface Ethernet0/1
 ip address 185.1.23.2 255.255.255.252
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 188.1.23.1 track 100
ip route 0.0.0.0 0.0.0.0 185.1.23.1 254
!         
ip access-list extended NAT
 permit ip 192.168.10.0 0.0.0.255 any
!
ip sla 100
 icmp-echo 188.1.23.1 source-interface Ethernet0/0
ip sla schedule 100 life forever start-time now
ip sla 101
 icmp-echo 185.1.23.1 source-interface Ethernet0/1
ip sla schedule 101 life forever start-time now
!
route-map Prov2 permit 10
 match ip address NAT
 match interface Ethernet0/1
!
route-map Prov1 permit 10
 match ip address NAT
 match interface Ethernet0/0
!
!
!
control-plane
!
!         
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end
```
```
Router#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 188.1.23.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 188.1.23.1
      185.1.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        185.1.23.0/30 is directly connected, Ethernet0/1
L        185.1.23.2/32 is directly connected, Ethernet0/1
      188.1.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        188.1.23.0/30 is directly connected, Ethernet0/0
L        188.1.23.2/32 is directly connected, Ethernet0/0
```
```
Prov1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Prov1(config)#int eth
Prov1(config)#int ethernet 0/0
Prov1(config-if)#sh
Prov1#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                188.1.23.1      YES NVRAM  administratively down down    
Ethernet0/1                unassigned      YES NVRAM  administratively down down    
Ethernet0/2                unassigned      YES NVRAM  administratively down down    
Ethernet0/3                unassigned      YES NVRAM  administratively down down
```
```
Router#sh track 
Track 100
  IP SLA 100 reachability
  Reachability is Down
    2 changes, last change 00:00:48
  Delay up 5 secs, down 5 secs
  Latest operation return code: Timeout
  Tracked by:
    Static IP Routing 0
Track 101
  IP SLA 101 reachability
  Reachability is Up
    1 change, last change 00:10:00
  Delay up 5 secs, down 5 secs
  Latest operation return code: OK
  Latest RTT (millisecs) 1
Router#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 185.1.23.1 to network 0.0.0.0

S*    0.0.0.0/0 [254/0] via 185.1.23.1
      185.1.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        185.1.23.0/30 is directly connected, Ethernet0/1
L        185.1.23.2/32 is directly connected, Ethernet0/1
      188.1.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        188.1.23.0/30 is directly connected, Ethernet0/0
L        188.1.23.2/32 is directly connected, Ethernet0/0
```
