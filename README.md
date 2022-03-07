# HLS Blueprint Release Docs

This repo will be used to store documents that pertain to HLS Blueprints.  This will be continually updated as more documents arise.

HLS-Flex-Provider blueprint requires installing the following:
- new Flex Twilio Account (DO NOT REUSE existing account you have)
- HLS-EHR that includes openEMR
- Telehealth for Flex (NOTE that this is different from regular Telehealth deployed)
- HLS Flex plug-in
- Owl Health website that has Webchat client


## IG for HLS Flex for Providers Blueprint

Please increase the Docker Desktop memory from default 2GB to at least 4GB.

---
### Provision a New Flex Twilio Account

- Create **NEW** Twilio account for Flex per [Set up your Twilio flex instance](https://www.twilio.com/docs/flex/tutorials/setup)
  - Assign account name: **hls-flex-provider**
  - Verify MFA
  - Wait until complete ... 
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

---
### Install HLS-EHR

If you have earlier docker for openemr running please remove it.

- Build installer
 ```shell
docker build --tag hls-ehr-installer --no-cache https://github.com/bochoi-twlo/hls-ehr.git#main
```

- Run installer
```shell
docker run --name hls-ehr-installer --rm --publish 3000:3000  \
--volume /var/run/docker.sock:/var/run/docker.sock \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty hls-ehr-installer
```

- Open installer `http://localhost:3000/installer/index.html`

- Deploy using installer UI entering required information.

- Once installation is complete, close the installer via either
  - stop button `hls-ehr-installer` in Docker desktop; or
  - control-C in your terminal

---
### Install Telehealth for Flex

- Build installer
```shell
docker build --tag hls-telehealth-flex-installer -no-cache  https://github.com/Pham-dev/telehealth-v2.git#main
```

- Run installer
```shell
docker run --name hls-telehealth-flex-installer --rm --publish 3000:3000 \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty hls-telehealth-flex-installer
```

- Open installer `http://localhost:3000/installer.html`

- Deploy using installer UI entering required information.

- Once installation is complete, close the installer via either
    - stop button `hls-telehealth-flex-installer` in Docker desktop; or
    - control-C in your terminal

- Note the deployed service URL (e.g., `telehealth-v2-6531-dev.twil.io`) and execute below replacing `your-react-app-backend-url`
```shell
export REACT_APP_BACKEND_URL=your-react-app-backend-url
```

---
### Install Flex Plugin

- Launch your Flex instance and click "Edit" under development setup. Here you'll want to change the React Version to the latest; which is 16.13.1

- Build installer
```shell
docker build --build-arg TWILIO_ACCOUNT_SID=${TWILIO_ACCOUNT_SID} \
--build-arg TWILIO_AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--build-arg REACT_APP_BACKEND_URL=${REACT_APP_BACKEND_URL} \
--no-cache --tag hls-flex-plugin-installer https://github.com/Pham-dev/hls-emr-flex-plugin.git#main
```

- Run installer
```shell
docker run --name hls-flex-plugin-installer --rm --publish 3000:3000 --publish 3001:3001 \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty hls-flex-plugin-installer 
```

- Open installer `http://localhost:3000/`

- Deploy using installer UI entering required information.

- Once installation is complete, close the installer via either
    - stop button `hls-flex-plugin-installer` in Docker desktop; or
    - control-C in your terminal

---
### Install HLS Website (aka OwlHealth)

- Build installer
 ```shell
docker build --tag hls-website-installer --no-cache https://github.com/bochoi-twlo/hls-website.git#main
```

- Run installer
```shell
docker run --name hls-website-installer --rm --publish 3000:3000  \
--env ACCOUNT_SID=${TWILIO_ACCOUNT_SID} --env AUTH_TOKEN=${TWILIO_AUTH_TOKEN} \
--interactive --tty hls-website-installer
```

- Open installer `http://localhost:3000/installer/index.html`

- Deploy using installer UI entering required information.

- Once installation is complete, close the installer via either
    - stop button `hls-website-installer` in Docker desktop; or
    - control-C in your terminal

---
### Launching Flex Blueprint

After installation, you **MUST** launch chrome via command to overcome iframe restrictions of the browser

```shell
open -na Google\ Chrome --args --user-data-dir=/tmp/temporary-chrome-profile-dir --disable-web-security --disable-site-isolation-trials
```
