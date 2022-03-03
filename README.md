# HLS Blueprint Release Docs

This repo will be used to store documents that pertain to HLS Blueprints.  This will be continually updated as more documents arise.


# Installation Guide

## HLS Flex for Providers

### Provisioned Flex Twilio Account

- Create Twilio account for Flex (https://www.twilio.com/docs/flex/tutorials/setup)
  - Assign account name: hls-flex-provider
  - Verify MFA
  - Wait until complete ... 
- Note the ACCOUNT_SID & AUTH_TOKEN of newly created account


### (Optional) to receive SMS appointment notifications, Quick Deploy [patient appointment management](https://www.twilio.com/code-exchange/appointment-management-healthcare)
### Install [HLS-EHR](https://github.com/bochoi-twlo/hls-ehr) per https://github.com/bochoi-twlo/hls-ehr/blob/main/README.md
### Install [Telehealth for Flex](https://github.com/Pham-dev/telehealth-v2) 
### Install [Flex Plugins](https://github.com/Pham-dev/hls-emr-flex-plugin) per https://github.com/Pham-dev/hls-emr-flex-plugin/blob/main/README.md

After installation, you MUST launch chrome via command to overcome iframe restrictions of the browser

```shell
open -na Google\ Chrome --args --user-data-dir=/tmp/temporary-chrome-profile-dir --disable-web-security --disable-site-isolation-trials
```
