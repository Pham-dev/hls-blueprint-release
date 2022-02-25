# HLS Blueprint Release Docs

This repo will be used to store documents that pertain to HLS Blueprints.  This will be continually updated as more documents arise.


# Installation Guide

## HLS Flex for Providers

0. Provisioned Flex Twilio Account (ACCOUNT_SID & AUTH_TOKEN)
1. (Optional) to receive SMS appointment notifications, Quick Deploy [patient appointment management](https://www.twilio.com/code-exchange/appointment-management-healthcare)
2. Install [HLS-EHR](https://github.com/bochoi-twlo/hls-ehr) per https://github.com/bochoi-twlo/hls-ehr/blob/main/README.md
3. Install [Telehealth for Flex]() per TBD
4. Install [Flex Plugins](https://github.com/Pham-dev/hls-emr-flex-plugin) per https://github.com/Pham-dev/hls-emr-flex-plugin/blob/main/README.md

After installation, you MUST launch chrome via command to overcome iframe restrictions of the browser

```shell
open -na Google\ Chrome --args --user-data-dir=/tmp/temporary-chrome-profile-dir --disable-web-security --disable-site-isolation-trials
```
