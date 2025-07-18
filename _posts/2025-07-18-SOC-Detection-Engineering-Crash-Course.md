---
title: SOC Detection Engineering Crash Course"
date: 2025-07-18 01:09:33 +0300
author: oste
description: SOC Detection Engineering Crash Course.
image: /assets/img/Posts/SOC Detection  Engineering  Crash Course.png
categories: [Detection Engineering]
tags:
  [CTI, Elasticsearch, Fleet, Sysmon, Atomic red, Detection Engineering, detection]
---

## Lab 0.1: CloudLabs Setup


- Navigate to https://app.metactf.com/ and create a MetaCTF account, or sign into your existing account.
- Once logged in, go to the Cloud Labs tab

![image](https://gist.github.com/user-attachments/assets/8f0b2858-d3ad-4dc1-8fb2-33b0096008c7)

- Enter in the class code for this course, then click submit:

![image](https://gist.github.com/user-attachments/assets/d2b44f60-eb0c-4589-8eb3-6c14972ab3a5)

- The page will then refresh and you should see the listing for the workshop, something like: SOC Detection Engineering w/ Hayden Covington. Click on “View & Manage Lab” to access the VM.
- Once there, you can click “Start Lab” or click “Start” on the specific VM you want (this course only has one)


![image](https://gist.github.com/user-attachments/assets/16cbd4d4-d9ff-4732-9cd0-ce4b3fd55919)

- After a few minutes the machine will start. You can connect via your browser* by clicking “Connect via browser (RDP)”.

![image](https://gist.github.com/user-attachments/assets/fda29d17-c0d9-4a7a-8be2-572a3fb29de7)

- Now you should be set!

## LAB 0.2: Elastic Setup

- Elastic cloud setup is very easy and doesn’t require much work. Start by going to: https://cloud.elastic.co/registration and enter a username and password

![image](https://gist.github.com/user-attachments/assets/0a12c148-fec9-46e9-87c5-69b51005fc9e)

- On the next page you may be prompted to enter your reasons and interests for trying Elastic.
- When creating your deployment, you will likely be creating an “Elastic Project”, which doesn’t necessarily allow infrastructure preferences. If you do choose on that page to do a hosted deployment, select the infrastructure of your choice.

![image](https://gist.github.com/user-attachments/assets/83f3c4a2-2316-4fc1-9b3d-b8b1e3cec66e)

- You may be provided with deployment credentials for your root account. If you see these, make sure you save them. Not everyone receives that pop-up, in some cases your account is the root account.
- This should only take a few minutes and then: boom; your infrastructure is set up.


![image](https://gist.github.com/user-attachments/assets/dbe0f1e2-0ec9-4ac8-93e2-1b7df407fae7)

- Now to briefly explore the management side of Elastic. Select the Dropdown menu on the left and select “Manage this deployment”.

![image](https://gist.github.com/user-attachments/assets/5ee46930-4283-446c-88bc-75edc84abed8)

- This takes you to your infrastructure deployment management page.

![image](https://gist.github.com/user-attachments/assets/1b35812f-24dd-4014-85d5-e62911cd3413)

- You can do various things from this page in terms of management, including restarting Elastic should something go horribly wrong. You likely will not need to do anything here, and shouldn’t mess with any of the infrastructure settings.
- This is also where you would scale up or down your deployment should you decide to continue running Elastic after the trial expires.

![image](https://gist.github.com/user-attachments/assets/340aaaaa-bdb8-4b1a-8659-8cd1d1156640)

- You can run queries from this section using an API console. We won’t mess with this, but if you see queries in the Elastic documentation that look to be API-related and don’t conform to the normal syntax; this is where those come in.

![image](https://gist.github.com/user-attachments/assets/3ee4c19e-feb0-499b-9631-f978677380d6)

- You can create filtering rules under `Features → Traffic filters` if you want to limit access to your deployment. I do not recommend you do this right now, because if you do this incorrectly you may lock yourself out.

![image](https://gist.github.com/user-attachments/assets/2e81c6b2-54f6-4dd8-a268-58ac59f60573)

- That’s all we need to see there for now, but let’s secure our account a little since this is a cloud-based application. Click your user in the top right and select “Profile”.
 
![image](https://gist.github.com/user-attachments/assets/c3e7f209-43dc-4de5-ae2c-5454185de1a8)

- Click your way back to the parent deployments page.

![image](https://gist.github.com/user-attachments/assets/ea9da309-6a4d-44b3-99b8-ffedb1f8d80d)

- Now to get back to your frontend Kibana, click the deployment name:

![image](https://gist.github.com/user-attachments/assets/02f8c1e8-a421-4ffb-b85c-9addc5d21de2)

*This is where you would also manage multiple deployments if you had them*

- Now you’re back at your fully functional ElasticSearch cluster.

![image](https://gist.github.com/user-attachments/assets/099c42c1-3b18-4ee7-be8b-0a9981267dc8)


## LAB 1.1: Fleet and Elastic Agent enrollment

- Please begin by booting up your Windows VM that was provided with the class (either the VM, or CloudLabs, depending on what you got). We aren’t going to use it yet, but that way it’s ready when you need it. Also make sure you can copy and paste to that VM. You won’t want to be typing commands manually.
- On your host machine (not the VM), log onto your Elastic instance. If prompted whether to log in with ElasticSearch or Elastic Cloud, always go with Cloud. This is how we created the account.
- You should be on the main page of Elastic now. Let’s configure some logging!
- Click on Assets, and select “Fleet”. Fleet is what we will be using to manage our deployed agents and policies.

![image](https://gist.github.com/user-attachments/assets/6d252a65-5e47-4732-8c7c-f40f59847181)

You should see that we already have a host and an agent policy applied to it. This host is your Fleet server itself.


![image](https://gist.github.com/user-attachments/assets/6e4c489d-a952-41b1-8f86-0d033312778a)

![image](https://gist.github.com/user-attachments/assets/dab1a6c2-0129-4644-b79c-a4b1d70761ad)

- Go over to the Agent policy tab.

![image](https://gist.github.com/user-attachments/assets/650ab53f-fa7d-4368-aec2-0404596413f1)

- Right now we only have one agent policy and one agent using it; and that policy is using two integrations.

![image](https://gist.github.com/user-attachments/assets/9c437934-9634-441e-ae9a-8b58824f1891)

- Click the three dot menu to see some details of the configuration. Within that policy you’ll see a whole slew of information, much of which we won’t ever touch during this class. However take note of some of the information at the top
- This information is showing us where the data from the agents is going under “outputs”, where the fleet host server is under “fleet”, and under “indices:names:” we see the indices where this data will be headed.

![image](https://gist.github.com/user-attachments/assets/7dccc49e-b8a3-4633-96e5-0eddc0228ee1)

- Go back over to the “Agents” tab and select “Add agent”. 

![image](https://gist.github.com/user-attachments/assets/32c00170-b800-492f-9257-66204fe2fc51)

- Name your new agent policy whatever you like, and don’t mess with any of the advanced options. Click “Create policy”.

![image](https://gist.github.com/user-attachments/assets/ac0858c7-32f1-4bfc-a596-e2922bc16fbd)

- Once the policy is created some other data will show up on this screen for you. Let’s walk through it:
    - “Enroll in Fleet?” – Leave this as recommended
    - “Install Elastic Agent on your host” – this is the command you’ll need to run on the host to set up the Elastic Agent. You’ll note that it says this command installs, enrolls, and starts the agent. I’ll explain the command in greater detail below.

![image](https://gist.github.com/user-attachments/assets/bad79afb-4e17-43f6-aa83-6a0cb880b033)

- Move over to the Windows section, this is the command we’ll need if you’re using the Windows VM provided. 

![image](https://gist.github.com/user-attachments/assets/afb9757c-0222-45bc-ba47-94ce43ca9ffb)

- Copy the Windows command and return to your VM. Open up a PowerShell window as Administrator and run the command.

![image](https://gist.github.com/user-attachments/assets/3a2eeac3-9710-47a7-bb47-b12ea0a6ee57)

- The command explained
    - `$ProgressPreference = 'SilentlyContinue'` - Suppresses progress bars from displaying, making the output look cleaner
    - `Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.12.2-windows-x86_64.zip -OutFile elastic-agent-8.12.2-windows-x86_64.zip` - Downloads the Elastic Agent and specifies where to place the file
    - `Expand-Archive .\elastic-agent-8.12.2-windows-x86_64.zip -DestinationPath .` - Unzips the Elastic Agent file and places it in the current directory.
    - `cd elastic-agent-8.12.2-windows-x86_64` - Changes the current directory to be the decompressed Elastic Agent folder
    - `.\elastic-agent.exe install --url=<url> --enrollment-token=<token>` - Runs and installs the Elastic Agent, specifying your cluster as the destination and providing the necessary enrollment token.

- Back in Elastic you should see confirmation that your agent has been enrolled and data is flowing:

![image](https://gist.github.com/user-attachments/assets/5da94635-9872-4a97-8142-124e5f234958)

- Before moving forward, run the below commands in the PowerShell terminal to give us some activity to look into later:
    - whoami 
    - net user


```powershell
PS C:\Users\Administrator\elastic-agent-9.0.3+build202507110136-windows-x86_64> whoami
ec2amaz-urs18df\administrator
PS C:\Users\Administrator\elastic-agent-9.0.3+build202507110136-windows-x86_64> net user

User accounts for \\EC2AMAZ-URS18DF

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
WDAGUtilityAccount
The command completed successfully.
```

- Hit Close on the “Add agent” window in Elastic, but leave the PowerShell window open and the VM on for now. Back in Elastic you should now see your host in the agents listing.

![image](https://gist.github.com/user-attachments/assets/75150528-e1f4-4a3b-b975-db599c47f24b)

- Our host may be enrolled, but spoiler alert: we aren’t done yet. Right now we aren’t getting all of the logs we need. Why? Let’s take a look. Go to “Agent policies” and click on your logging policy. You should see a list of Integrations


![image](https://gist.github.com/user-attachments/assets/45688351-80c8-4be0-bcb1-9e535ddaf57a)

- The logs being gathered by this integration aren’t quite up to par with what we want for our investigations. If you actually click on that integration named “System”, it explains that this integration is for basics logs and metrics.
- The host we're using has Sysmon installed, and we definitely want those logs. They will provide us with a wealth of information.
- Let’s get our Sysmon logs from that host. Go back to your logging policy page and hit Add integration

![image](https://gist.github.com/user-attachments/assets/cce3f52d-557d-4e1d-a2fc-687f8f1ae824)

- Search for “Windows” and select the Windows option highlighted below:

![image](https://gist.github.com/user-attachments/assets/6f3fdc45-b61c-458f-8ff9-b77487715f84)

- If you look at this integration’s page, it notes that it gathers some of the logs that we’re looking for:

![image](https://gist.github.com/user-attachments/assets/2841076b-141b-4e3e-b430-b4e3fb637b57)

- Click “Add Windows” in the top right. 

![image](https://gist.github.com/user-attachments/assets/f1909eb6-b8ca-42ce-b767-adb3dd9239d1)

![image](https://gist.github.com/user-attachments/assets/db444013-d5f3-4ef8-bfe2-993d9d627523)

- Do not change any of the default options, but make sure you’re adding this integration to the Agent policy you created earlier. If so, hit Save and continue.

![image](https://gist.github.com/user-attachments/assets/b62a4c0d-e8aa-4f9c-8cb9-3946b1c21bdd)

- Hit Save and deploy changes when prompted

![image](https://gist.github.com/user-attachments/assets/0f2de9b8-0d9c-4c4f-84be-f158cc2198f0)

- This is going to ship more of the logs we’re looking for. We don’t necessarily want to ship logs we won’t be using, but for now we’re probably fine and likely airing on too few rather than too many log events.

## LAB 1.2: Baby’s First Query

- In this lab you are going to write some KQL queries to look for a few specific log events that you generated earlier:
    - Your initial log-in event to the VM
    - The WHOAMI command you ran after installation of the agent
- Begin by navigating to the Discover tab in your Elastic instance.

![image](https://gist.github.com/user-attachments/assets/6473dca6-14a5-49e5-8135-4bda8bf7a262)

- First, adjust the time range of your search, as well as the dataview.
    - You can guesstimate how long ago you ran the commands in question, or you can search a longer range.
    - Note: on a production server with numerous hosts and more data, searching a longer time range will take longer and cause more load. In our case though, we are only logging from one host that isn’t doing much.
    - In my case I set the time range as “Last 7 days”; and given the state of our cluster this should be absolutely fine.
    - As for the dataview, you may default to metrics-* which is not going to contain the new Windows logs we’re shipping.
- Before you even start searching, you should add a few basic fields to your table. You can always change this later by expanding an event and adding more fields, but to start with, it is recommended to include a few basic fields that you prefer.
    - The basic fields I usually include in all my investigative queries are:
event.action and/or event.code
        - host.name
        - user.name
        - process.pid
        - process.executable
        - process.command_line
    - If a field doesn’t exist in a log event, you can’t add it from the Expanded view.
    - If a field doesn’t exist within an event you are viewing, you can add it from the Sidebar. For that reason I usually will add all my fields from the sidebar before I start searching.
- Now that the search results won’t look like a wall of text, there are a few ways you can find these events in Elastic. In this lab you can find them however you like, but If you need hints I’m going to include the queries I used below.
- Reminder that the three events you need to find are:
    - Your initial log-in event to the VM
        - The KQL query I used is:
            `event.code: 4624 and user.name: Desktop`
    - The WHOAMI command you ran after installation of the agent
        - The KQL query I used is:
            `process.name: whoami.exe`
![image](https://gist.github.com/user-attachments/assets/2110fead-ca18-4349-8b44-95cc9813d52f)

![image](https://gist.github.com/user-attachments/assets/61e2d51e-ef29-4126-b79c-479c0edd5f86)

![image](https://gist.github.com/user-attachments/assets/deef259c-b23c-4329-b9c9-5161fad47d1e)

## LAB 1.3: Baby’s First Detection (KQL)

- For this lab you will be creating a detection to look for the MITRE technique T1087.001, which is Account Discover: Local Account. This technique is used by attackers to get a listing of local system accounts. It can be done a number of ways.
- The specific method we’re going to be looking for today is net user, which lists the local users on a host.
- Every good detection starts with a query to see what results are returned. So start in Discover by building a query to look for that data.
    - There are plenty of ways to look for this. One way you could do is search for something like `process.command_line: *net*user*` but that is a lot of load on the cluster for such a simple search since there are three wildcards.
    - **Why this search would have so much load?**
        - **Leading wildcards**: Elastic would normally have starting point for which to search other results, but a leading wildcard requires Elastic to look at all events
            - For this reason leading wildcards can actually be disabled on a cluster
        - `*user*`: Once Elastic has determined what events contain `net` it must then look for “user” anywhere after that, with any characters after it

    ![image](https://gist.github.com/user-attachments/assets/f41ce251-0922-47d7-8fd5-6385ab1caf0a)

    - This is where I’ll introduce you to the very cool field called `process.args`. This field indexes the arguments in a `process.command_line` field as an array. We can use this to match a command event that meets multiple command line arguments, even with other things before, after, or between!
        - Fun fact, this also means if an attacker were to run `net user`, our detection should still work, since that is still just two arguments. Same for even if they ran `net user > something.txt`
    - Below is the basic query I settled on for my detection. Yours may look different, and that is fine. I encourage you to work with your own query.
    `process.name: net.exe and process.args: user`
        - And it should match if you ran it earlier in Lab 1.1:
        - Note that in this case if I matched on both of the args `“net”` AND `“user”`, it wouldn’t match, since the first argument is actually the net full path.
        - Regardless of where the user runs this event, PowerShell, Cmd, whatever; the actual executable of “net.exe” will be executing this event. So I can specify that executable specifically to hugely optimize my search, and in this case catch any event that is querying the user, regardless of what other arguments they use.

        ![image](https://gist.github.com/user-attachments/assets/076754e4-c14b-41bc-8da2-c00b03d232b9)

- Now that we have a query, let’s assess the volume that this detection may bring our SOC.
    - Run your query for your whole dataset over a long period. A good bet is to usually look at 30-90 days.
    - Every event that returns from your query would figure to be *one alert*, barring any suppression or filtering.
    - For now, you will probably see very few results.
    - On the chance you’re doing this on your own host and you see a lot of events, you either need to refine your query, or apply some filtering.
- Now that we have a query, we can move forward with creating the alert itself.
- Before we get into the actual search syntax, we need to create a detection in Elastic.
    - Head to Security > Rules > Detection rules
    ![image](https://gist.github.com/user-attachments/assets/26d85e7a-08c6-4ba4-8cca-afc9eebd181f)

    - Remember, do not click “Add elastic rules”
    - Select “Create new rule” in the top right
    ![image](https://gist.github.com/user-attachments/assets/aa5eb386-4eca-4899-8864-9092a43ebedd)
    ![image](https://gist.github.com/user-attachments/assets/8ede22ec-754a-4d80-8805-5982bdabf639)

    - We want the Rule type of “Custom query”
    - For “Source” leave that as the Index patterns in question. If we had, for example, a Windows dataview set up we could specify this to limit the alert’s load when it runs- but we haven’t set that up.
    - “Custom query” is where your query itself goes. In my case, process.name: net.exe and process.args: "user"
    - For the Suppression section, suppress by the host.name and user.name for rule execution.
    ![image](https://gist.github.com/user-attachments/assets/f4c854a4-ef2c-4b81-82db-6a3db579cf62)
    This suppression will prevent us from constantly receiving alerts for what is effectively the same activity. If I run net user 6 times on a host in a couple minutes, we don’t need 6 alerts for that- 1 will do just fine, especially when you only need one good alert to find an attacker.
    - Hit “Rule preview” at the top with a time range applied to see what comes back!
    ![image](https://gist.github.com/user-attachments/assets/24153b9b-c0e1-4af5-abdc-926ca30668c0)
    
    Hit continue if the results look good and low.

- Now we’re prompted to enter information about the rule.
    -  Give the rule a name and description.
        - I named mine “Net User Discovery”.
        - The description I gave it is pulled directly from MITRE:
            > Adversaries may attempt to get a listing of local system accounts. This information can help adversaries determine which local accounts exist on a system to aid in follow-on behavior. 
            > Commands such as net user and net localgroup of the Net utility and id and groupson macOS and Linux can list local users and groups. On Linux, local users can also be enumerated through the use of the /etc/passwd file. On macOS the dscl . list /Users command can be used to enumerate local accounts.
- As for the severity and risk score, you can set that as you like. A lot of this will depend on the environment, your specific security concerns, and other context like that.
- In my case I am setting the detection to Low with a risk score of 10. That is because to me, enumerating users on a host is not inherently malicious, there can be benign use cases for doing so.

    ![image](https://gist.github.com/user-attachments/assets/b748e76a-74c8-4760-b996-a1bd870f9eb1)

- Open up the Advanced settings.
    - For the “Reference URLs”, you can list any reference information that led you to create this alert, or would be useful for someone reviewing this detection or its alerts.
        - I included the Atomic Red Team atomic link.
    - “False positive examples” is fairly self explanatory. Add any examples you can think of where this alert may fire on a false positive event. In my case I put “A technician performing maintenance on a host” as an example.
    - For the MITRE section, navigate down the list to add T1087.001. Note that you can add multiple tactics, but for this alert we really only need to add one.

    ![image](https://gist.github.com/user-attachments/assets/d7a5e330-e7b4-4c57-b3a3-b1bb33a5e34d)
    - Down in “Investigation guide”, this is where you put a guide on how to investigate your alert. Spend some time thinking about how to investigate if this is malicious activity or not
        - This will usually be easier if a rule was your idea. If that’s the case, there’s a reason you want to alert on something, so it will come naturally. However that won’t always be the case; you may be assigned to build a detection that wasn’t your idea. That is where your research step comes in.
        - In this case, we’re building this to address that specific MITRE technique. Pull on that - MITRE page for inspiration: https://attack.mitre.org/techniques/T1087/001/
        - Here is what I put in mine

    ![image](https://gist.github.com/user-attachments/assets/aa4c1ca8-3d4b-4181-ba66-df75a03585cd)

    Under “Author” put your name.

- Under “Schedule rule” leave everything at the default settings.

    ![image](https://gist.github.com/user-attachments/assets/834d7d31-ca15-4ad5-8d4f-bff7fca62148)

- Hit “Create and enable” on your rule.

    ![image](https://gist.github.com/user-attachments/assets/c7de5ff5-4cc6-4f5a-a117-cc640fc0065e)

- Your detection should now look something like this

    ![image](https://gist.github.com/user-attachments/assets/efb86c92-8c09-4e4b-b0a6-1b21ae93fe1a)

- Switch over to your VM and run the “net user” command from a PowerShell window.
- Within a few minutes, if everything is done correctly, you should see An Elastic alert


    ![image](https://gist.github.com/user-attachments/assets/557ba5fe-a116-4bcc-9bc4-17953769d2e9)

    ![image](https://gist.github.com/user-attachments/assets/81f37824-81e5-44ef-8a7f-5944ddb74560)
    ![image](https://gist.github.com/user-attachments/assets/42a7909c-d0c6-4c85-a212-1fe74f37488d)
    ![image](https://gist.github.com/user-attachments/assets/400a91dd-f0e3-437a-83a9-5c71151dd88b)


## LAB 2.1: Testing Detections with Atomic Red Team

- Testing your detections with Atomic Red Team (ART) is simple and easy. The detection we just created actually has an atomic associated, so run that atomic now and make sure it triggers your detection.
    - Run PowerShell as administrator and run: `powershell -exec bypass`
    - Next, we’re going to trigger the Atomic associated with the detection we just created. With Atomics, the Atomic identifier always matches up with the MITRE technique id.
    - We’re going to run one part of that Atomic rather than all of the tests associated for the sake of speed. Specifically we are going to run Atomic T1087.001-9 using Invoke-AtomicRedTeam, which is a utility for ART that lets you quickly run specific Atomics.
    ```powershell
    Import-Module Invoke-AtomicRedTeam
    Invoke-AtomicTest T1087.001-9
    ```
- Following the execution of the ART test, you should have triggered your detection which created an alert in Elastic.
- The point of using ART versus executing the commands manually is you can build out an atomic catalog in order to test all of your custom detections with one command, dodging the requirement to run complex or numerous commands yourself.

    ![image](https://gist.github.com/user-attachments/assets/ce8c68cb-8720-46a8-90c7-f73a7b1592fb)
    ![image](https://gist.github.com/user-attachments/assets/db83a400-bc29-4543-a00d-6543365bebc2)


## LAB 2.2: Tuning Detections in Elastic

- A lot of SOC work is tuning your detections to increase their fidelity. You may be tuning due to activity that’s not actually what you want to match on, or maybe activity that is a true positive match but it isn’t malicious. We’re going to filter the first detection we wrote, the Net User Discovery detect.
- Create a new user on your Windows VM named `“Helpdesk”`.
    - Set a password you will remember.
    - With an elevated PowerShell window, run: `net user Helpdesk <password> /add`
    ![image](https://gist.github.com/user-attachments/assets/f99b11ac-3ed0-489d-b92c-e60702d87d1a)

    - Note that you shouldn’t be creating local accounts this way in production, specifically settings the password in this way on the command line. It is fine in this case since this is a lab environment.
- Now, in that same window, run Net User as the account you just created:

    ```powershell
    Runas /user:Helpdesk “net user”
    ```
    ![image](https://gist.github.com/user-attachments/assets/b7f1ba0d-5399-440c-8693-872f502a97f5)

- You’ll be prompted to enter the password of that account you just made, and then the command will execute.
- Back over in Elastic you should see the alerts generated by this activity.

    ![image](https://gist.github.com/user-attachments/assets/0568e001-211b-49c4-8845-96111ea2f46b)
    ![image](https://gist.github.com/user-attachments/assets/0568e001-211b-49c4-8845-96111ea2f46b)


- In this scenario, we’ll say that the “Helpdesk” user we just made is the standard user that our IT helpdesk team uses to help troubleshoot user issues. These accounts are very secure with SuperSecure-NG-v2-XDR™️, and in some cases they may need to execute ‘net user’ to do their job. So let’s filter them from our detection.
    - Head over to your detection page for Net User Discovery (`Elastic → Security → Rules → Detection rules (SIEM) → Net User Discovery`) and scroll down to where you see the Alerts section usually; you should see “Rule exceptions” next to that. Switch over to that tab.
   ![image](https://gist.github.com/user-attachments/assets/9613a334-6ee6-44fa-907f-fcefca45d6e1)
    - This section shows you all the exceptions for your rule, expired or active.
    - In this case the exception we are adding will be permanent. Click “Add rule exception”
    - Name your exception and add the username you created to the conditions section. Including a comment is good practice as well. No need to set an expiration date.
    - You can also check under “Alert actions” the box that says “Close all alert actions that match this exception”, which will empty out your alerts that are sitting in Elastic from here on out that match your new filter criteria.
    - Your exception should look something like this:
    ![image](https://gist.github.com/user-attachments/assets/22b5aacf-945d-44ea-8a45-c826b930851a)

    - Click “Add rule exception”, and you should then see the rule exception in place in Elastic:
    ![image](https://gist.github.com/user-attachments/assets/7f47e4aa-c009-43fb-b441-8d9210e5d3d5)

- Back in your VM run the net user command again as helpdesk.
    ```
    Runas /user:Helpdesk “net user”
    ```
    - If you don’t see an alert show up in Elastic, congratulations, you’ve created your first filter!
    - Running the Detection query over in Discover you can see that the logs are in-fact in your SIEM. And with your detection running every 5 minutes, by the time you wait a few minutes it should have fired on Helpdesk’s command by then if it was going to.
    ![image](https://gist.github.com/user-attachments/assets/61f7ec8b-d88d-48d8-94b9-ec6cb3cd3f00)
- Once you are done with the above, you should consider removing the local account either on your host machine or your VM, depending on where you are doing these labs. Here is how to do so:
- In an elevated PowerShell window run:

```powershell
net user Helpdesk /delete
```