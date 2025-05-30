# Purple Teaming Okta Detections with LimaCharlie

This guide will provide you with a step-by-step of all the commands we will use throughout this workshop. Please reference it as we move forward. If you have questions, feel free to ask your group moderator.

### Free tools we will use:

Please create your free accounts for both Okta and LimaCharlie, we will walk through how to install Dorothy in lab 2:

- [LimaCharlie Community Edition](https://free.limacharlie.io/)
- [Okta Developer Account](https://developer.okta.com/signup/)
- [Dorothy Okta Adversary Emulation Tool](https://github.com/elastic/dorothy)
- [LimaCharlie Detection & Response Documentation](https://docs.limacharlie.io/docs/detection-and-response-examples)



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
You will be prompted to enter some information. You will be asked for an API token, I suggest you create a new one in our Okta Developer Account and name it something like "EvilHackerKey" as this is the API token we will use to simulate the compromise of an access token. You will want to cretea a new configuration profile, provide the URL of your Okta Developer Account instance, insert an API token, you will want to save it in the local config to use later, and answer "n" to indexing the logs in Elasticsearch...unless you want to log them. 

```
[*] Do you want to load an existing configuration profile? Answer no to create a new one [Y/n]: n
[*] Creating a new configuration profile
[*] Enter description for target Okta environment: LCPurpleTeam
[*] Enter URL for target Okta environment. E.g. https://my-company.okta.com: https://[YOUR INSTANCE].okta.com
[*] Enter your Okta API token to execute actions. The input for this value is hidden: 
[*] Do you want to store the API token in the local config file? [Y/n]: y
[*] Do you want to index Dorothy's logs in Elasticsearch? [y/N]: n
```

![Dorothy Setup](/img/dorothy1.png)

Congratulations!!! You now have your Okta Purple Teaming lab setup! Now let's run some attacks! 


## Lab 3: Dorothy Attack Simulation Create New Admin User

1. Run the whoami commmand to view permissions of the API token you created to ensure that you have Super Administrator permissions
```
whoami
```
2. Run the "list-modules" command to review the modules available in Dorothy, notice that the modules are grouped in tactics, each of these tactics is a menu you can navigate into:
```
list-modules
```
![Dorothy Setup](/img/dorothy2.png)

3. Next we will create a new user, navigate to the "persistence" menu and then enter "create-user"
4. Type the "info" command to view the fields that are required for this module. 
5. To set the values we will use the "set" command followed by the paramters we want to set prefixed with a double dash "--"

Example: 
```
set --first-name Evil --last-name Hacker --email evilhacker@gmail.com --login eveilhacker@gmail.com
```
6. Then enter the "execute" command, you will be prompted to create a password for this user (the password will need to comply with Okta's password policy otherwise you will get an error).

![Dorothy Setup](/img/dorothy3.png)

7. The simply created a new user, in order to escalate the user's privileges we will need to use "create-admin-user" in the persistance menu. However, to do so we will need to get the unique id of the user we just created. Go to the discovery menu and enter the 'get-users" command. Copy and paste the ID somwhere to use in the next command.

![Dorothy Setup](/img/dorothy_get_users.png)

8. Go the the "persistence -> create-admin-user" and enter the info command to see the required paraemters for this user: 

![Dorothy Setup](/img/create_admin_user1.png)

9. Use the "set" command to pass the parameter " --id" followed by the ID of the user we just created. When you "execute" this command you will be prompted for what permissions you wnat to set for the user. Select "9. Super_Admin" to give this user admin privileges.

![Dorothy Setup](/img/create_admin_user2.png)

## Lab 3: From Logs to Detection in LimaCharlie 

- [LimaCharlie Detection & Response Documentation](https://github.com/elastic/dorothy)

Note: There is some latency between Okta events happening and when the logs are sent, especially with the free Developer Accounts. Keep in mind it may take a few minutes before events show up in the logs. 

1. Go to your LimaCharlie account and go to "Sensors List" and click on the Okta sensor we created and then go to the "Timeline" page for the sensor. 

![Dorothy Setup](/img/okta_sensor_timeline.png)

Spend a few minutes exploring the log events that we created using Dorothy. 

2. In the sensor "Timeline" you should see the token you created for Dorothy (if not create another one in your Okta Developer instance). Click on the event to see the log data and then click the icon to create a detection rule.

![Dorothy Setup](/img/okta_token_1.png)

You should now see a test event along with detection logic pre-built for you. 

![Dorothy Setup](/img/okta_token_2.png)

Override the pre-built rule with this code, this will work as a pass through detection, which will trigger the alert in LimaCharlie if the event simply occurs. 

Detection
``` yaml
event: system.api_token.create
op: exists
path: routing/hostname
```

Response
``` yaml
- action: report
  name: Okta API Key Created for {{ .routing.hostname }} {{ .event.actor.alternateID }}
```

You can test the event to ensure the syntax is correct and should also get a positive result. 

![Dorothy Setup](/img/okta_token_3.png)

Congratulations you just wrote your first Okta detection in LimaCharlie! Go ahead and click "Create" to save your detection. Feel free to create a new API token in you Okta Developer instance to test it. 

## Lab 4: Okta Admin Login Detection

Looking at the timeline we can see a lot of interesting events, one that occurs is the ```user.session.access_admin_app``` event if the user who logs in is an adminstrator, they get redirected to the admin page and this particular log event is generated. Let's create a rule that detects if someone other than ourselves logs in as administrator. 

<details>
<summary>Click to view an answer (multiple ways to do this) </summary>

Detection

``` yaml

event: user.session.access_admin_app
op: and
rules:
  - not: true
    op: is
    path: event/actor/alternateId
    value: ken@cybersecurity.io
  - op: exists
    path: routing/hostname
```

Response

``` yaml

- action: report
  name: Unauthorized Okta Admin Access Page on {{ .routing.hostname }} by {{ .event.actor.alternateID }}
```
</details>


## Lab 5: Detect New User Creation

If you have not run Dorothy to create a new user and escalate their privileges do so now. Looking at your Okta sensor Timeline, find the event where a user was created. Create a rule that detects when a new user is created and send the email of the new user, who created the user and what Okta host in the response. 

<details>
<summary>Hint 1 </summary>

Look for the ```  user.lifecycle.create ``` event

</details> 
<details>
<summary>Hint 2 </summary>

Look for the user created in the "target" parameter

</details> 

<details>
<summary>Answer </summary>

Detection
``` yaml

event: user.lifecycle.create
op: exists
path: routing/hostname

``` 

Response
``` yaml

- action: report
  name: New User Created {{ .event.target.alternateID }} {{ .routing.hostname }} {{ .event.actor.alternateID }}


``` 
</details> 



## Lab 6: More Detections

Now create detections for other events such as privilege escalation, disabling of MFA etc. Experiment! 



<details>
<summary>MFA Disabled</summary>

Detection
``` yaml

event: user.mfa.factor.deactivate
op: exists
path: routing/hostname

``` 

Response
``` yaml

- action: report
  name: Okta MFA Deactivated for {{ .event.target.alternateID }} on {{ .routing.hostname }} by {{ .event.actor.alternateID }}

``` 
</details> 

