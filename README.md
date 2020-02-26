# oci_waf_nsg
# Lockdown OCI WAF Origin using CloudShell (Network Security Groups) #
_An easy way to lockdown your OCI WAF Origin_ 


When using a Web Application Firewall, it is important to apply a set of security rules around your Web Application to block all requests, which are not passing through the WAF service.

OCI WAF stand in front of your web application to detect and block unwanted/malicious access. . In most cases, your Web Application itself sits behind a Load Balancer. Depending on your architecture and preferences, you may want to assign security rules to your Load Balancer or Web Application subnet by using a Security List, or alternatively you may prefer to assign the security rules to your Load Balancer or Web Application Network Interfaces by using a Network Security Group.

This document will guide you through the steps needed to create and assign a ***Network Security Group*** containing a list of OCI WAF public IP addresses used to send the traffic to your load balancer / Web Application. 




> ***Important Note:*** 
> If you are using a Load Balancer in front of your Web Application, Security rules must be applied to your Load Balancer subnet (if using Security Lists) or Load Balancer Network Interfaces (if using network Security Groups).


***Prerequisites:***

- An OCI user account with enough permissions to create and assign Network Security Groups in the desired Compartment / VCN. 
- At least one Virtual Cloud Network, one Subnet and one Network Interface (LBaaS , Compute, etc.) already created. 
 
 
 
 
### 1- Create a New (empty) Network Security Group.    

 1.1-	Sign-in to the OCI web console with your OCI user account. 

1.2-	Open the OCI Menu (top left), expand the ***‘Networking’*** section and click on ***‘Virtual Cloud Networks’***.  

1.3-	Click on your VCN then select the ***‘Network Security Groups’*** Resources type. 

![PMScreens](/img/01.JPG)

1.4-	Click on the ***‘Create Network Security Group’*** button. 

![PMScreens](/img/02.JPG)

1.5-	Provide a meaningful name for this new Network Security Group. (Ie. ‘OCIWAF-NSG’)

1.6-	Select proper compartment, then click on ***‘Next’*** button. 

1.7-  Click ***‘Create’*** button to create an empty NSG. 

![PMScreens](/img/03.JPG)

1.7-	In the 'Network Security Group Information' section copy the NSG OCID (click the 'Copy' link in front of 'OCID')  

![PMScreens](/img/04.JPG)
 
### 2-    Import Security Rules using Cloud Shell commands.

2.1-	Start your OCI Cloud Shell session. In the OCI Console top right section, click on the Cloud Shell icon:  

![PMScreens](/img/05.JPG)

2.2-	Wait few seconds for your Cloud Shell instance to be started and ready to use.

![PMScreens](/img/06.JPG)

2.3-	Copy and Paste (CTRL+SHIFT+’V’) the command below in your Cloud Shell session.

```
wafnsg=ocid1.networksecuritygroup.oc1.eu-frankfurt-1.aaaaaaaxxxxx
```
(Replace ‘ocid1.networksecuritygroup.oc1.eu-frankfurt-1.aaaaaaaxxxxx’ by your NSG OCID copied in the previous step.)

![PMScreens](/img/07.JPG)


2.4-	Import Security rules. 

2.4.1-	***[OPTION 1]*** Allow incoming **HTTP (TCP80) and HTTPS (TCP443)** traffic, copy and Paste (_CTRL+SHIFT+V_) the commands below in your Cloud Shell session.

```
wget -N https://raw.githubusercontent.com/BaptisS/oci_waf_nsg/master/wafnsg-80-jan20_part1.json
wget -N https://raw.githubusercontent.com/BaptisS/oci_waf_nsg/master/wafnsg-80-jan20_part2.json
wget -N https://raw.githubusercontent.com/BaptisS/oci_waf_nsg/master/wafnsg-443-jan20_part1.json
wget -N https://raw.githubusercontent.com/BaptisS/oci_waf_nsg/master/wafnsg-443-jan20_part2.json

oci network nsg rules add --nsg-id $wafnsg --ingress-security-rules file://wafnsg-80-jan20_part1.json
oci network nsg rules add --nsg-id $wafnsg --ingress-security-rules file://wafnsg-80-jan20_part2.json
oci network nsg rules add --nsg-id $wafnsg --ingress-security-rules file://wafnsg-443-jan20_part1.json
oci network nsg rules add --nsg-id $wafnsg --ingress-security-rules file://wafnsg-443-jan20_part2.json

```

2.4.1- 	***[OPTION 2]*** Allow incoming **HTTPS (TCP443) only**, copy and Paste (_CTRL+SHIFT+V_) the commands below in your Cloud Shell session.

```
wget -N https://raw.githubusercontent.com/BaptisS/oci_waf_nsg/master/wafnsg-443-jan20_part1.json
wget -N https://raw.githubusercontent.com/BaptisS/oci_waf_nsg/master/wafnsg-443-jan20_part2.json

oci network nsg rules add --nsg-id $wafnsg --security-rules file://wafnsg-443-jan20_part1.json
oci network nsg rules add --nsg-id $wafnsg --security-rules file://wafnsg-443-jan20_part2.json
```


### 3-    Review the Network Security Group content. 

![PMScreens](/img/08.JPG)

3.1-	Ensure the new NSG has been populated successfully with 50 security rules.  

### 4-   [Option 1] Assign NSG to your Load Balancer Virtual Network Interfaces.
4.1-	Go to your Load Balancer dashboard, (OCI Menu -> Networking -> Load Balancers) and then select the Network Security Groups ***‘Edit’*** link . 

![PMScreens](/img/09.JPG)

4.2-	Select the new NSG (ie. OCIWAF-NSG). 

![PMScreens](/img/10.JPG)

4.3-	Click on ***‘Save Changes’*** button.  

### 4-   [Option 1] Assign NSG to your Load Balancer Virtual Network Interfaces.
4.1-	Go to your Load Balancer dashboard, (OCI Menu -> Networking -> Load Balancers) and then select the Network Security Groups ***‘Edit’*** link . 

![PMScreens](/img/09.JPG)

4.2-	Select the new NSG (ie. OCIWAF-NSG). 

![PMScreens](/img/10.JPG)

4.3-	Click on ***‘Save Changes’*** button.  




4.4-	Select the newly created Security List (Ie. OCIWAF-SL)  

![PMScreens](/img/10.JPG)

4.5-	Click ***‘Add Security List’*** button to assign the Security to the subnet.  

![PMScreens](/img/11.JPG)

### 5-   Remove any permissive rules 
5.1-	Remove any pre-existing permisive rules to lockdown your WAF Origin and allow only incoming traffic from the OCI WAF Public IPs.





