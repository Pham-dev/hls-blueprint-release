# HLS Blueprint Release Docs

This repo will be used to store documents that pertain to HLS Blueprints. This will be continually updated as more documents arise.

- [Flex for Provider](#flex-for-provider-f4p-blueprint)
- 

HLS team provides the latest version of installers for blueprint via docker hub.

---

### Prerequisites

#### Clear Cached Images/File in Chrome

:exclamation: *Always do this before every (re-)install to ensure removal of old images/files.*

- Goto Settings of Chrome
- Click 'Privacy and security' on left pan
- Click 'Clear browsing data'
- Select 'Cached images and files' and click 'Clear data' button


#### Docker Desktop

- Goto https://www.docker.com/products/docker-desktop/
- Download and install
- *Increase the Docker Desktop memory from default 2GB to 6GB*
- Before you begin, free up resources used by docker by running the following in a terminal: 
```shell
docker system prune --force
```

#### ngrok

When you accept a task in Flex, the name of the customer in the chat is queried in OpenEMR
in order to obtain patient data and displayed in the information pane.
Thus, OpenEMR must be installed locally on your laptop. `ngrok` is also required,
as it allows the plugin to communicate over the internet into the OpenEMR instance running on your local machine.

- Goto https://ngrok.com/download
- Download and install ngrok
- If you have twilio email, register at http://ngrok.com using your twilio email
- Get yourself invited from hls team member to assign a static ngrok url for yourself
  (e.g., `bochoi.ngrok.io`)
- Accept the invite and login
- Note your own authtoken at https://dashboard.ngrok.com/get-started/your-authtoken
- Add your authtoken to your ngrok via executing the following in the terminal
```shell
ngrok authtoken your-ngrok-auth-token
```
- Go to Cloud Edge → Domains
  - Click '+ New Domain' button
  - Add a domain name of your choosing. (e.g., `ssepac.ngrok.io` using your twilio login)
  - Note this ngrok domain name for later use

If your `ngrok` version is not 3.3 or higher, run the following to update
```shell
brew reinstall --cask ngrok
```

and then upgrade your ngrok configuration too by
```shell
ngrok config upgrade --relocate
```

Note location of configuration file (`ngrok.yml`) will be `~/Library/Application Support/ngrok/ngrok.yml`
as you'll need to edit this later.

---

## Flex for Provider (F4P) Blueprint

- [Provision a New Flex Account](#f4p-provision-a-new-flex-account)
- [Install OpenEMR](#f4p-install-openemr)
- [Install Telehealth](#f4p-install-telehealth)
- [Install Flex Plugin](#f4p-install-flex-plugin)
- [Install OwlHealth Website](#f4p-install-owlhealth-website)
- [Launch Flex Blueprint](#f4p-launch-flex-blueprint)
- [Uninstall Flex Blueprint](#f4p-unistall-blueprint)
- [Translation Services via Lionsbridge](#f4p-optional-get-sms-translation-on-your-flex-instance-via-lionbridge-translations)

### F4P Provision a New Flex Account

- Create **NEW** Twilio account for Flex per [Set up your Twilio flex instance](https://www.twilio.com/docs/flex/tutorials/setup)
  - Assign account name: **hls-flex-provider** (or some variation of)
  - Verify MFA
  - Wait until complete ... 
- Upgrade this trial account to regular account via Monkey (you can do this a bit later before you run the demo)

- Note the ACCOUNT_SID & AUTH_TOKEN of newly created account from the Twilio console

- In your terminal, execute the following, substituting your Twilio account sid & auth token

```shell
export TWILIO_ACCOUNT_SID=your-flex-twilio-account-sid
```
```shell
export TWILIO_AUTH_TOKEN=your-flex-twilio-auth-token
```

Keep the terminal open as you will use it throughout the installation.
Just copy-n-paste the commands below as is and monitor the terminal output for any error messages.


### F4P Install OpenEMR

- Start installer and wait 1 minute to start up (watch the terminal output)
```shell
docker run --name hls-ehr-installer --rm --publish 3000:3000  \
--volume /var/run/docker.sock:/var/run/docker.sock \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-ehr-installer:latest
```

- Open installer url below in chrome
```shell
http://localhost:3000/installer/index.html
```

- If you wish to reinstall, you must first remove existing installation by either
  - Clicking on the 'Remove...' button at the bottom or
  - Manually delete the `hls-ehr` docker stack via the docker desktop.
    If you are unable to delete the stack, you can delete it's container one-by-one
  
- Deploy using installer UI entering required information and watch the terminal output

- Once installation is complete, close the installer via either
  - stop button `hls-ehr-installer` in Docker desktop; or
  - control-C in your terminal


### F4P Install Telehealth

- Start installer and wait 1 minute to start up (watch the terminal output)
```shell
docker run --name hls-telehealth-installer --rm --publish 3000:3000 \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-telehealth-installer:latest
```

- Open installer url below in chrome
```shell
http://localhost:3000/installer.html
```

- Deploy using installer UI entering required information and watch the terminal output

- Once installation is complete, close the installer via either
    - stop button `hls-telehealth-installer` in Docker desktop; or
    - control-C in your terminal
    
#### Common Errors
- When pressing the deploy button for the Telehealth installer, it will occassionally fail to deploy some `assets`.
  - To fix this, just click redeploy until the deployment process successfully finishes.  You may need to do this a handful of times.


### F4P Install Flex Plugin

- Note your ngrok domain name (e.g., `bochoi.ngrok.io`)

- Start installer
```shell
docker run --name hls-flex4p-installer --rm --publish 3000:3000 \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-flex4p-installer:latest
```

- Open installer url below in chrome
```shell
http://localhost:3000/installer/index.html
```

- Deploy using installer UI entering required information and watch the terminal output

- Once installation is complete, close the installer via either
     - stop button `hls-flex4p-installer` in Docker desktop; or
     - control-C in your terminal


#### Common Errors for Flex Plugin Install

- Re-installing will produce the latest version of the plugin


#### [Optional] Enable Inbound and Outbound Calling + 3 Way External Warm Transfers

To enable any sort of calling in Flex you will need to enable the dialpad for Flex so you'll need to:
- Login to your Flex Twilio account
- navigate to Flex -> Manage -> Voice on the left-hand panel
- Under "Flex Dialpad":
   - Flip on Enable Dialpad
   - Caller Id should be your Flex Phone Number (usually 1 phone number if it's a new account)
   - Change Task Queue to "Schedulers"
   - Task Router Workflow: "Intake by Schedulers"
   - Country: "United States of America"

Next, to enable 3 Way External Warm Transfers (3 way calling), you can send a message to a Twilio Team member and they will be able to turn on that feature flag for your Flex Account.  Once enabled, You are allowed to call in a 3rd person into the call (for our use case it will be a Spanish Interpreter).


### F4P Install OwlHealth Website

- Start installer and wait 1 minute to start up (watch the terminal output)
```shell
docker run --name hls-website-installer --rm --publish 3000:3000  \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-website-installer:latest
```

- Open installer url below in chrome
```shell
http://localhost:3000/installer/index.html
```

- Deploy using installer UI entering required information and watch the terminal output

- Once installation is complete, close the installer via either
    - stop button `hls-website-installer` in Docker desktop; or
    - control-C in your terminal

- Note the URL for the `hls-website` service in the Twilio console.
- website URL is `http://your-hls-website-hostname/index.html`


### F4P Launch Flex Blueprint

- Open a new terminal window and run ngrok from your local machine substituting `your-ngrok-domain-name` with your own
  (e.g., `bochoi.ngrok.io`)
```shell
ngrok http --region=us --hostname=your-ngrok-domain-name 80
```

- To launch Flex, you **MUST** launch chrome via command to overcome iframe restrictions of the browser

```shell
open -na Google\ Chrome --args --user-data-dir=/tmp/temporary-chrome-profile-dir --disable-web-security --disable-site-isolation-trials
```

- Open `http://your-hls-website-hostname/index.html` in chrome substituting `your-hls-website-hostname` from above.


---

### F4P Unistall Blueprint

- OpenEMR
  - run the installer and click 'Remove...' button in the installer page; or
  - from your docker desktop manually delete the stack or containers individually
- Telehealth
  - remove the Serverless service `telehealth` via Twilio console
- Flex Plugin
  - **Cannot be removed**
- OwlHealth Website
  - remove the Serverless service `hls-website` via Twilio console



### F4P [Optional] Get SMS Translation on your Flex Instance via LionBridge Translations

This is an optional step and is not required to get things working.

- First, downgrade Flex to Version 1.30.2:
  - In your Flex Instance, click on the controls Tab
  - Then Click on "Versions"
  - Then choose Version: 1.30.2 and install that
- Create the Lionbridge Plugin by running this command in your terminal
```shell
curl -X POST https://flex-api.twilio.com/v1/PluginService/Plugins \
--data-urlencode "FriendlyName=Lionbridge Language Cloud" \
--data-urlencode "Description=Real-time language translation provided by Lionbridge" \
--data-urlencode "UniqueName=lionbridge" \
-u ${TWILIO_ACCOUNT_SID}:${TWILIO_AUTH_TOKEN}
```
- Next use the response from the above command and make note of the PluginSid (ex: FPXXXXXXXXXXX) and run this command in the terminal but replace the FPXXXXXXX.... string with your PluginSid
```shell
curl -X POST https://flex-api.twilio.com/v1/PluginService/Plugins/FPXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Versions \
--data-urlencode "Private=True" \
--data-urlencode "Version=1.0.0" \
--data-urlencode "PluginUrl=https://developers.lionbridge.com/geofluent/plugin/plugin-geofluent.js" \
-u -u ${TWILIO_ACCOUNT_SID}:${TWILIO_AUTH_TOKEN}
```
- Release the Plugin to your Flex account
```shell
twilio flex:plugins:release --plugin lionbridge@1.0.0 --name "Lionbridge" --description "Adding Lionbridge integration"
```
- Next Configure your Lionbridge Plugin to call the ```/get-credentials``` function that was deployed in the plugin-backend which should already been deployed alongside your Flex install.  ```PLUGIN_BACKEND_URL``` can be found in the Twilio console under Functions and Assets -> Services.  Then click on your ```plugin-backend``` and take note of the url at the bottom left of the screen.  It should look like this: ```plugin-backend-XXXX-dev.twil.io```.  You will need this URL for the below command. Fill out the command and run it in the terminal
```shell
curl -s 'https://flex-api.twilio.com/v1/Configuration' -u ${TWILIO_ACCOUNT_SID}:${TWILIO_AUTH_TOKEN} | jq -r -c '{ attributes: .attributes } * { "account_sid": "${TWILIO_ACCOUNT_SID}", "attributes": { "lionbridge": {"credentials": "https://${PLUGIN_BACKEND_URL}/get-credentials"} }}' | curl -s -X POST 'https://flex-api.twilio.com/v1/Configuration' -u ${TWILIO_ACCOUNT_SID}:${TWILIO_AUTH_TOKEN} -H 'Content-Type: application/json' -d @-
```
- We'll now set the language we want text to be translated to
  - Launch Flex and click on the controls button on the left Panel
  - Click on Skills and under "Add New Skill" add the following Languages depending on prefered language to translate to:
    - English: add "lanuage*en-us"
    - Spanish: add "language*es-xl
    - Brazilian Portuguese: add "language*pt-br"
- Now assign the language skill to Agents:
  - In the Agent Tab, 3rd down on the left panel of the Flex Instance:
    - Click on an agent
    - On the right there will be the Agent's skills listed
    - Click the "Add skill" Dropdown and pick the prefered language to translate to
    - Click the Blue Plus button to the right of it and make sure it is checked on
    - NOTE: Only have 1 language chosen at a time
    - Hit save and you're good to go.
- Now incoming SMS will be translated by Lionbridge.

---
