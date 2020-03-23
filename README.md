
# vManage APIs for Application Aware Routing Policies

# Objective 

*   How to use vManage APIs - 
    - Monitor Application Aware Routing statistics (BFD statistics) for Overlay Tunnels
    - Create new SLA class list, Application Aware Routing policy and update active vSmart policy
    - Delete SLA class list and Application Aware Routing policy

# Requirements

To use this code you will need:

* Python 3.7+
* vManage user login details. (User should have privilege level to configure policies)

# Install and Setup

- Clone the code to local machine.

```
git clone https://github.com/suchandanreddy/sdwan-app-route-policy.git
cd sdwan-app-route-policy
```
- Setup Python Virtual Environment (requires Python 3.7+)

```
python3.7 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

- Create **vmanage_login.yaml** using below sample format to provide the login details of vManage

## Example:

```
# vManage Connectivity Info
vmanage_host: 
vmanage_port: 
vmanage_username: 
vmanage_password: 
```

## Sample Outputs

```
(venv) python3 monitor-app-route-stats.py

Enter Router-1 System IP address : 1.1.10.1
Enter Router-2 System IP address : 1.1.100.1

App route statistics between 1.1.10.1 and 1.1.100.1

╒══════════════════════════════════════════════╤══════════════╤═══════════╤═══════════════════╤══════════╕
│ Tunnel name                                  │   vQoE score │   Latency │   Loss percentage │   Jitter │
╞══════════════════════════════════════════════╪══════════════╪═══════════╪═══════════════════╪══════════╡
│ 1.1.10.1:biz-internet-1.1.100.1:biz-internet │           10 │   3.56788 │          0.208333 │  1.20091 │
├──────────────────────────────────────────────┼──────────────┼───────────┼───────────────────┼──────────┤
│ 1.1.10.1:mpls-1.1.100.1:mpls                 │            7 │ 100.02    │          7.39547  │  1.39822 │
╘══════════════════════════════════════════════╧══════════════╧═══════════╧═══════════════════╧══════════╛

App route statistics between 1.1.100.1 and 1.1.10.1

╒══════════════════════════════════════════════╤══════════════╤═══════════╤═══════════════════╤══════════╕
│ Tunnel name                                  │   vQoE score │   Latency │   Loss percentage │   Jitter │
╞══════════════════════════════════════════════╪══════════════╪═══════════╪═══════════════════╪══════════╡
│ 1.1.100.1:mpls-1.1.10.1:mpls                 │            7 │  99.9014  │          7.33819  │  1.37649 │
├──────────────────────────────────────────────┼──────────────┼───────────┼───────────────────┼──────────┤
│ 1.1.100.1:biz-internet-1.1.10.1:biz-internet │           10 │   3.54639 │          0.230564 │  1.16642 │
╘══════════════════════════════════════════════╧══════════════╧═══════════╧═══════════════════╧══════════╛
```



```
(venv) python3 create-app-route-policy.py

Please enter App aware route policy which needs to be replaced : AAR4
Latency for new App aware route policy (ms) (1-1000) : 100
Loss percentage for new App aware route policy (%) (1-100) : 8
Jitter for new App aware route policy (ms) (1-1000) : 2

Created new SLA Class msuchand_sla_class

Retrieved app aware routing policies list

Retrieved app aware route policy definition AAR4

Created app aware route policy msuchand_AAR4

Retrieved activated vsmart policy

Updating vsmart policy with new app aware route policy

Updated vsmart policy with new app aware route policy

 viptela-policy:policy
  sla-class msuchand_sla_class
   latency 100
   loss 8
   jitter 2
  !
 !
 app-route-policy _CorpVPN_msuchand_AAR4
  vpn-list CorpVPN
    sequence 1
     match
      dscp 46
      source-ip 0.0.0.0/0
     !
     action
      sla-class msuchand_sla_class  preferred-color mpls
     !
    !
 !
 lists
  site-list All_Sites
   site-id 10
   site-id 20
   site-id 100
  !
  vpn-list CorpVPN
   vpn 1
  !
 !
!
viptela-policy:apply-policy
 site-list All_Sites
  app-route-policy _CorpVPN_msuchand_AAR4
 !
!
```


```
(venv) python3 delete-app-route-policy.py

Please enter App aware route policy which was replaced : AAR4

Retrieved app aware route policy definition AAR4

Retrieved activated vsmart policy

Updated vsmart policy with old app aware route policy

 viptela-policy:policy
  sla-class DSCP46
   latency 50
   loss 10
   jitter 100
  !
 app-route-policy _CorpVPN_AAR4
  vpn-list CorpVPN
    sequence 1
     match
      dscp 46
      source-ip 0.0.0.0/0
     !
     action
      sla-class DSCP46  preferred-color mpls
     !
    !
 !

 lists
  site-list All_Sites
   site-id 10
   site-id 20
   site-id 100
  !
  vpn-list CorpVPN
   vpn 1
  !
 !
!
viptela-policy:apply-policy
 site-list All_Sites
  app-route-policy _CorpVPN_AAR4
 !
!


Deleted msuchand_ app route policy

Deleted msuchand_sla_class
```