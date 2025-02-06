# Purple Teaming Okta Detections with LimaCharlie

This guide will provide you with a step-by-step of all the commands we will use throughout this workshop. Please reference it as we move forward. If you have questions, feel free to ask your group moderator.

### Free tools we will use:

Please create your free accounts for both Okta and LimaCharlie, we will walk through how to install Dorothy in lab 2:

- [LimaCharlie Community Edition](https://free.limacharlie.io/)
- [Okta Developer Account](https://developer.okta.com/signup/)
- [Dorothy Okta Adversary Emulation Tool](https://github.com/elastic/dorothy)

## Lab 1: Building the Okta Dev Lab

1. [Sign up for a free Okta Developer](https://developer.okta.com/signup/) account if you have not done so.
2. In your Okta Developer acccount go to Security > API and click on the "Tokens" tab
![Okta Token Page](/img/okta1.png)
3. Click "Create Token" and give it a unique name
![Okta Token Page](/img/okta2.png)
4. Copy the new token and past it somehwere safe as backup
![Copy Token](/img/okta3a.png)
5. Log into your LimaCharlie account, if you have not created an org please creat one.
6. Go to "Sensors" in the left hand menu and clik on the "Add Sensor" button on the right hand side of the page
7. You will see a list of possible sensors, scroll down until you find "Okta" and click "Select"
![Copy Token](/img/lcokta0.png)
8. For Installation Key click "Create New" and call it "OktaPurpleWorkshop"
9. Next we will fill in information about our Okta Developer account, give it a unique name, copy the API key we created in Okta in the "APIkey" field and the URL of your Okta instance. 
![Copy Token](/img/lcokta1.png)
10. After a few minutes the data from Okta should start flowing and you should see your new Okta sensor in your list of active sensors.
![Copy Token](/img/lcokta2.png)

## Lab 2: Installing and Configuring Dorothy

1. Dorothy is an adversary emulation tool written in Python by security researcher David French. Dorothy is availble in PyPy so if PIP is installed on your system you can install it by simply entering: 

```
 pip install dorothy
```

You can also install Dorothy from source, the code is availble [on Github here](https://github.com/elastic/dorothy)

2. Once you have Dorothy installed in your Python environment you can fire it up by simply typing "dorothy" on the command line. 
You will be prompted to enter some information. 
```
[*] Do you want to load an existing configuration profile? Answer no to create a new one [Y/n]: n
[*] Creating a new configuration profile
[*] Enter description for target Okta environment: LCPurpleTeam
[*] Enter URL for target Okta environment. E.g. https://my-company.okta.com: https://dev-46185139.okta.com
[*] Enter your Okta API token to execute actions. The input for this value is hidden: 
[*] Do you want to store the API token in the local config file? [Y/n]: y
[*] Do you want to index Dorothy's logs in Elasticsearch? [y/N]: n
[*] Consider executing "whoami" to get user information and roles associated with current API token
[*] Execute "list-modules" to show all of Dorothy's modules
```


![Dorothy Setup](/img/dorothy1.png)
