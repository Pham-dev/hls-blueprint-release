# HLS Blueprint Release Docs

This repo will be used to store documents that pertain to HLS Blueprints. This will be continually updated as more documents arise.

- [Flex for Provider](#flex-for-provider-f4p-blueprint)
- [Developer Notes](#developer-notes)

HLS team provides the latest version of installers for blueprint via docker hub.

---

### Prerequisites

#### Clear Cached Images/File in Chrome

```diff
!!! Always do this before every (re-)install to ensure removal of old images/files.
```

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

- [HLS Blueprint Release Docs](#hls-blueprint-release-docs)
    - [Prerequisites](#prerequisites)
      - [Clear Cached Images/File in Chrome](#clear-cached-imagesfile-in-chrome)
      - [Docker Desktop](#docker-desktop)
      - [ngrok](#ngrok)
  - [Flex for Provider (F4P) Blueprint](#flex-for-provider-f4p-blueprint)
    - [F4P Provision a New Flex Account](#f4p-provision-a-new-flex-account)
    - [F4P Install OpenEMR](#f4p-install-openemr)
    - [F4P Install Telehealth](#f4p-install-telehealth)
      - [Common Errors](#common-errors)
    - [F4P Install Flex Plugin](#f4p-install-flex-plugin)
      - [Common Errors for Flex Plugin Install](#common-errors-for-flex-plugin-install)
      - [[Optional] Enable Inbound and Outbound Calling + 3 Way External Warm Transfers](#optional-enable-inbound-and-outbound-calling--3-way-external-warm-transfers)
    - [F4P Install OwlHealth Website](#f4p-install-owlhealth-website)
    - [F4P Launch Flex Blueprint](#f4p-launch-flex-blueprint)
    - [F4P Unistall Blueprint](#f4p-unistall-blueprint)
    - [F4P [Optional] Get SMS Translation on your Flex Instance via LionBridge Translations](#f4p-optional-get-sms-translation-on-your-flex-instance-via-lionbridge-translations)
  - [Developer Notes](#developer-notes)
    - [Blueprint Git Repos](#blueprint-git-repos)

### F4P Provision a New Flex Account

- Create ❗**NEW** ❗ Twilio account for Flex per [Set up your Twilio flex instance](https://www.twilio.com/docs/flex/tutorials/setup). ❗❗❗DO NOT INSTALL TO EXISTING FLEX ACCOUNT WITH OTHER PLUG-INS❗❗❗
  - Assign account name: **hls-flex-provider** (or some variation of)
  - Verify MFA
  - Wait until complete ... 
- Launch the Flex console and choose 'Login in with Twilio'. You ❗MUST❗ do this step to properly create a flex worker using your twilio user.
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

- Remove previous docker image
```shell
docker image rm twiliohls/hls-ehr-installer
```

- Start installer and wait 1 minute to start up (watch the terminal output)
```shell
docker run --name hls-ehr-installer --rm --publish 3000:3000  \
--volume /var/run/docker.sock:/var/run/docker.sock \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-ehr-installer
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

- Remove previous docker image
```shell
docker image rm twiliohls/hls-telehealth-installer
```

- Start installer and wait 1 minute to start up (watch the terminal output)
```shell
docker run --name hls-telehealth-installer --rm --publish 3000:3000 \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-telehealth-installer
```

- Open installer url below in chrome
```shell
http://localhost:3000/installer/index.html
```

- Deploy using installer UI entering required information and watch the terminal output

- Once installation is complete, close the installer via either
    - stop button `hls-telehealth-installer` in Docker desktop; or
    - control-C in your terminal
    
#### Common Errors
- When pressing the deploy button for the Telehealth installer, it will occassionally fail to deploy some `assets`.
  - To fix this, just click redeploy until the deployment process successfully finishes.  You may need to do this a handful of times.


### F4P Install Flex Plugin

```diff
!!! You can ONLY deploy on a new Flex UI 2.0 Account as you cannot upgrade from 1.3 to 2.0
```

- Note your ngrok domain name (e.g., `bochoi.ngrok.io`)

- Remove previous docker image
```shell
docker image rm twiliohls/hls-flex4p-installer
```

- Start installer
```shell
docker run --name hls-flex4p-installer --rm --publish 3000:3000 \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-flex4p-installer
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

- Remove previous docker image
```shell
docker image rm twiliohls/hls-website-installer
```

- Start installer and wait 1 minute to start up (watch the terminal output)
```shell
docker run --name hls-website-installer --rm --publish 3000:3000  \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty twiliohls/hls-website-installer
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

- Set the language we want text to be translated to
  - Launch Flex and click on the controls button on the left Panel
  - Click on Skills and under "Add New Skill" add the following Languages depending on prefered language to translate to:
    - English: add "lanuage*en-us"
    - Spanish: add "language*es-xl
    - Brazilian Portuguese: add "language*pt-br"
    - ![Add Skill](https://i.imgur.com/5SLkhiim.png)
- Now assign the language skill to Agents:
  - In the Agent Tab, 3rd down on the left panel of the Flex Instance:
    - Click on an agent
    - On the right there will be the Agent's skills listed
    - Click the "Add skill" Dropdown and pick the prefered language to translate to
    - Click the Blue Plus button to the right of it and make sure it is checked on
    - NOTE: Only have 1 language chosen at a time
    - Hit save and you're good to go.
    - ![Select Skill](https://i.imgur.com/kGiPY5Dm.png)
- Enable the Lionbridge plugin
  - Go to the Flex admin dashboard and select Plugins
  - Select the "Lionbridge Language Cloud" plugin
  - ![Select plugin](https://i.imgur.com/RerB3bi.png)
  - Select the "enabled" radiobutton
  - Select version either version 1.0.0 or 1.0.1 from the dropdown.
  - Select "Save"
  - ![Enable plugin](https://i.imgur.com/tlLfBh0m.png)
  - You will be prompted to type in a release name and description. Enter "InitialRelease" as the name and "Initial release" as the description.
- Now incoming SMS will be translated by Lionbridge.

---

## Developer Notes

### Blueprint Git Repos

|github repo                                   |notes
|----------------------------------------------|---------
|https://github.com/twilio/hls-frontline-pharma|Frontline for Pharma
|https://github.com/bochoi-twlo/hls-website    |Website with flex webchat client
|https://github.com/twilio/hls-telehealth      |Either deployed as part of flex or stand-alone
|https://github.com/twilio/hls-flex-provider   |Flex plugin for Flex4Provider
|https://github.com/bochoi-twlo/hls-ehr        |OpenEMR/Mirth stack for Flex4Provider & PAM
|https://github.com/twilio/hls-outreach-sms/tree/docker-installer|Outreach SMS
|https://github.com/twilio/hls-patient-appointment-management    |Patient Appointment Management /w OpenEMR integration

