---
title: "Google Assistant"
description: "Setup for Google Assistant integration"
logo: google-assistant.png
ha_category:
  - Voice
featured: true
ha_release: 0.56
---

The `google_assistant` integration allows you to control things via Google Assistant (on your mobile or tablet) or a Google Home device.

## Automatic setup via Home Assistant Cloud

With [Home Assistant Cloud](/cloud/), you can connect your Home Assistant instance in a few simple clicks to Google Assistant. With Home Assistant Cloud you don't have to deal with dynamic DNS, SSL certificates or opening ports on your router. Just log in via the user interface and a secure connection with the cloud will be established. Home Assistant Cloud requires a paid subscription after a 30-day free trial.

For Home Assistant Cloud Users, documentation can be found [here](https://www.nabucasa.com/config/google_assistant/).

## Manual setup

The Google Assistant integration requires a bit more setup than most due to the way Google requires Assistant Apps to be set up.

<div class='note warning'>

To use Google Assistant, your Home Assistant configuration has to be [externally accessible with a hostname and SSL certificate](/docs/configuration/remote/). If you haven't already configured that, you should do so before continuing.

</div>

## Migrate to release 0.80 and above
<div class='note'>

If this is the first time setting up your Google Assistant integration, you can skip this section and continue with the [manual setup instructions](#first-time-setup) below.

</div>

Since release 0.80, the `Authorization Code` type of `OAuth` account linking is supported. To migrate your existing configuration from release 0.79 or below, you need:

1. Change your `Account linking` setting in [Actions on Google console](https://console.actions.google.com/), look for the `Advanced Options` in the bottom left of the sidebar.
    - Change `Linking type` to `OAuth` and `Authorization Code`.
    - In the `Client information` section:
        - Change `Client ID` to `https://oauth-redirect.googleusercontent.com/`, the trailing slash is important.
        - Input any string you like into `Client Secret`, Home Assistant doesn't need this field.
        - Change `Authorization URL` to `https://[YOUR HOME ASSISTANT URL:PORT]/auth/authorize` (replace with your actual URL).
        - Change `Token URL` to `https://[YOUR HOME ASSISTANT URL:PORT]/auth/token` (replace with your actual URL).
    - In the `Configure your client` section:
        - Do **NOT** check `Google to transmit clientID and secret via HTTP basic auth header`.
    - Click 'Save' at the top right corner, then click 'Test' to generate a new draft version of the Test App.
2. Change your `configuration.yaml` file:
    - Remove `client_id`, `access_token`, `agent_user_id` config from `google_assistant:` since they are no longer needed.
3. Restart Home Assistant, open the `Google Home` app on your mobile phone then go to `Account > Settings > Assistant > Home Control`, press the `3 dot icon in the top right > Manage accounts > [test] your app name > Unlink account` Then relink your account by selecting `[test] your app name` again.
4. A browser will open and ask you to login to your Home Assistant instance and will redirect back to the `Google Assistant` app right afterward.

<div class='note'>

If you've added Home Assistant to the home screen, you have to first remove it from home screen, otherwise, this HTML5 app will show up instead of a browser. Using it would prevent Home Assistant to redirect back to the `Google Assistant` app.

If you're still having trouble, make sure that you're not connected to the same network Home Assistant is running on, e.g., use 4G/LTE instead.

</div>

## First time setup

You need to create an API Key with the [Google Cloud API Console](https://console.cloud.google.com/apis/api/homegraph.googleapis.com/overview) which allows you to update devices without unlinking and relinking an account (see [below](#troubleshooting-the-request_sync-service)). If you don't provide one, the `google_assistant.request_sync` service is not exposed. It is recommended to set up this configuration key as it also allows the usage of the following command, "Ok Google, sync my devices". Once you have set up this component, you will need to call this service (or command) each time you add a new device that you wish to control via the Google Assistant integration.

1. Create a new project in the [Actions on Google console](https://console.actions.google.com/).
    1. Add/Import a project and give it a name.
    2. Click on the `Home Control` card, select the `Smart home` recommendation.
    3. Click `Build your Action`, select `Add Action(s)`, and click `Add your first action`. Add your Home Assistant URL: `https://[YOUR HOME ASSISTANT URL:PORT]/api/google_assistant`, replace the `[YOUR HOME ASSISTANT URL:PORT]` with the domain / IP address and the port under which your Home Assistant is reachable.
    4. Click `Done`. Then click on `Overview`, which will lead you back to the app details screen.
2. `Account linking` is required for your app to interact with Home Assistant. Set this up under the `Quick Setup` section.
    1. Leave it at the default `No, I only want to allow account creation on my website` and select Next.
    2. For the `Linking type` select `OAuth` and `Authorization Code`.
    3. Client ID: `https://oauth-redirect.googleusercontent.com/`, the trailing slash is important.
    4. Client Secret: Anything you like, Home Assistant doesn't need this field.
    5. Authorization URL (replace with your actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/auth/authorize`.
    6. Token URL (replace with your actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/auth/token`.
    7. Configure your client: Type `email` and click `Add scope`, then type `name` and click `Add scope` again.
    8. Do **NOT** check `Google to transmit clientID and secret via HTTP basic auth header`.
    9. Testing instructions: Enter anything. It doesn't matter since you won't submit this app.

    <img src='/images/components/google_assistant/accountlinking.png' alt='Screenshot: Account linking'>
3. Under `Build your Action` click `Add Action(s)`.
    1. Under `Fulfillment` fill in this URL (replace with your actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/api/google_assistant`.
4. Back on the overview page. Click `Simulator` under `TEST`. It will create a new draft version Test App. You don't have to actually test, but you need to generate this draft version Test App.
5. Add the `google_assistant` integration configuration to your `configuration.yaml` file and restart Home Assistant following the [configuration guide](#configuration) below.
6. Open the Google Home app and go into `Account > Settings > Assistant > Home Control`.
7. Click the `+` sign, and near the bottom, you should have `[test] your app name` listed under 'Add new.' Selecting that should lead you to a browser to login your Home Assistant instance, then redirect back to a screen where you can set rooms for your devices or nicknames for your devices.

<div class='note'>

If you've added Home Assistant to the home screen, you have to first remove it from home screen, otherwise, this HTML5 app will show up instead of a browser. Using it would prevent Home Assistant to redirect back to the `Google Assistant` app.

</div>

1. If you want to allow other household users to control the devices:
    1. Go to the settings for the project you created in the [Actions on Google console](https://console.actions.google.com/).
    2. Click `Test -> Simulator`, then click `Share` icon in the right top corner. Follow the on-screen instruction:
        1. Add team members: Got to `Settings -> Permission`, click `Add`, type the new user's e-mail address and choose `Project -> Viewer` role.
        2. Copy and share the link with the new user.
        3. When the new user opens the link with their own Google account, it will enable your draft test app under their account.
    3. Have the new user go to their `Google Assistant` app to add `[test] your app name` to their account.
2. If you want to use the `google_assistant.request_sync` service, to update devices without unlinking and relinking, in Home Assistant, then enable Homegraph API for your project:
    1. Go to the [Google API Console](https://console.cloud.google.com/apis/api/homegraph.googleapis.com/overview).
    2. Select your project and click Enable Homegraph API.
    3. Go to Credentials, which you can find on the left navigation bar under the key icon, and select API Key from Create Credentials.
    4. Note down the generated API Key and use this in the configuration.

## Configuration

Now add the following lines to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
google_assistant:
  project_id: YOUR_PROJECT_ID
  api_key: YOUR_API_KEY
  exposed_domains:
    - switch
    - light
  entity_config:
    switch.kitchen:
      name: CUSTOM_NAME_FOR_GOOGLE_ASSISTANT
      aliases:
        - BRIGHT_LIGHTS
        - ENTRY_LIGHTS
    light.living_room:
      expose: false
      room: LIVING_ROOM
    group.all_automations:
      expose: false
```

{% configuration %}
project_id:
  description: Project ID from the Actions on Google console (looks like `words-2ab12`)
  required: true
  type: string
secure_devices_pin:
  description: "Pin code to say when you want to interact with a secure device."
  required: false
  type: string
  default: ""
api_key:
  description: Your Homegraph API key (for the `google_assistant.request_sync` service)
  required: false
  type: string
expose_by_default:
  description: "Expose devices in all supported domains by default. If `exposed_domains` domains is set, only these domains are exposed by default. If `expose_by_default` is set to false, devices have to be manually exposed in `entity_config`."
  required: false
  default: true
  type: boolean
exposed_domains:
  description: List of entity domains to expose to Google Assistant if `expose_by_default` is set to true. This has no effect if `expose_by_default` is set to false.
  required: false
  type: list
entity_config:
  description: Entity specific configuration for Google Assistant
  required: false
  type: map
  keys:
    '`<ENTITY_ID>`':
      description: Entity to configure
      required: false
      type: map
      keys:
        name:
          description: Name of the entity to show in Google Assistant
          required: false
          type: string
        expose:
          description: Force an entity to be exposed/excluded.
          required: false
          type: boolean
          default: true
        aliases:
          description: Aliases that can also be used to refer to this entity
          required: false
          type: list
        room:
          description: Allows for associating this device to a Room in Google Assistant.
          required: false
          type: string
{% endconfiguration %}

### Available domains

Currently, the following domains are available to be used with Google Assistant, listed with their default types:

- camera (streaming, requires compatible camera)
- group (on/off)
- input_boolean (on/off)
- scene (on)
- script (on)
- switch (on/off)
- fan (on/off/speed)
- light (on/off/brightness/rgb color/color temp)
- lock (lock/unlock (to allow assistant to unlock, set the `allow_unlock` key in configuration))
- cover (on/off/set position)
- media_player (on/off/set volume (via set volume)/source (via set input source))
- climate (temperature setting, hvac_mode)
- vacuum (dock/start/stop/pause)
- sensor (temperature setting, only for temperature sensor)
- Alarm Control Panel (Arm/Disarm)

<div class='note warning'>
  The domain groups contains groups containing all items, by example group.all_automations. When telling Google Assistant to shut down everything, this will lead in this example to disabling all automations
</div>

### Secure Devices

Certain devices are considered secure, including anything in the `lock` domain, `alarm_control_panel` domain and `covers` with device types `garage` and `door`.

By default these cannot be opened by Google Assistant unless a `secure_devices_pin` is set up. To allow opening, set the `secure_devices_pin` to something and you will be prompted to speak the pin when opening the device. Closing and locking these devices does not require a pin.

For the Alarm Control Panel if a code is set it must be the same as the `secure_devices_pin`. If `code_arm_required` is set to `false` the system will arm without prompting for the pin.

### Media Player Sources

Media Player sources are sent via the Modes trait in Google Assistant.
There is currently a limitation with this feature that requires a hard-coded set of settings. Because of this, the only sources that will be usable by this feature [are listed here](https://developers.google.com/actions/reference/smarthome/traits/modes).

#### Example Command:

"Hey Google, change input source to TV on Living Room Receiver"

### Room/Area support

Entities that have not got rooms explicitly set and that have been placed in Home Assistant areas will return room hints to Google with the devices in those areas.

### Climate Operation Modes

There is not an exact 1-1 match between Home Assistant and Google Assistant for the available operation modes.
Here are the modes that are currently available:

- off
- heat
- cool
- heatcool (auto)
- fan-only
- dry
- eco

### Troubleshooting the request_sync service

The request_sync service requires that the initial sync from Google includes the agent_user_id. If not, the service will log an error that reads something like "Request contains an invalid argument". If this happens, then [unlink the account](https://support.google.com/googlehome/answer/7506443) from Home Control and relink.

The request_sync service may fail with a 404 if the project_id of the Homegraph API differs from the project_id of the Actions SDK found in the preferences of your project on [Actions on Google console](https://console.actions.google.com). Resolve this by:

  1. Removing your project from the [Actions on Google console](https://console.actions.google.com).
  2. Add a new project to the [Google Cloud API Console](https://console.cloud.google.com). Here you get a new `project_id`.
  3. Enable Homegraph API to the new project.
  4. Generate a new API key.
  5. Again, create a new project in the [Actions on Google console](https://console.actions.google.com/). Described above. But at the step 'Build under the Actions SDK box' choose your newly created project. By this, they share the same `project_id`.

Syncing may also fail after a period of time, likely around 30 days, due to the fact that your Actions on Google app is technically in testing mode and has never been published. Eventually, it seems that the test expires. Control of devices will continue to work but syncing may not. If you say "Ok Google, sync my devices" and get the response "Unable to sync Home Assistant", this can usually be resolved by going back to your test app in the [Actions on Google console](https://console.actions.google.com/) and clicking `Simulator` under `TEST`. Regenerate the draft version Test App and try asking Google to sync your devices again.  If regenerating the draft does not work, go back to the `Action` section and just hit the `enter` key for the URL to recreate the Preview.

### Troubleshooting with NGINX

When using NGINX, ensure that your `proxy_pass` line *does not* have a trailing `/`, as this will result in errors. Your line should look like:

    proxy_pass http://localhost:8123;

### Unlink and relink

If you're having trouble with *Account linking failed* after you unlinked your service, try clearing the browser history and cache.
