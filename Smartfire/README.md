# Smartfire status to Home assistant with Node-RED

[Smartfire](https://smartfirebbq.com/) is a hardware that is controlling your smoker or BBQ temperature when preparing brisket, ribs, pulled pork or whatever you want. Smartfire is controlled via bluetooth or wifi using a mobile application (and cloud service in wifi model). In the newest firmware there is an option to send data also to IFTTT webhook. When I first time heard about this, I could also hear my home assistant to tell mee: *"Markus, your brisket is ready after 5 degrees"*.  
Integrating IFTTT webhook to Home Assistant webhook through e.g. Nabu Casa is quite simple. Create a *If this, then that* application in IFTTT with a customized body. Mine looks like this:  
![IFTTT Webhook](http://lintuala.fi/pics/hass-ifttt-smartfire-webhook.png)  
Smartfire sends it’s status in the event called **smartfire-status**. So If you receive a webhook to <https://maker.ifttt.com/trigger/smartfire-status/your-ifttt-webhook-key>, you get the temperature payload from Smartfire to IFTTT.
When data is in Home Assistant, I’ve created a dashboard to my wall iPad and I can use the values in notifications normally.
![Hass Dashboard](http://lintuala.fi/pics/hass-smartfiredashboard-v1.png)

## How to setup the integration

1. Sign in to IFTTT and get your IFTTT Webhook key (<https://ifttt.com/maker_webhooks>) and select *Documentation*. It says Your key is: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx. Copy that key.
2. Go to your Smart fire app (I prefer a browser <https://app.smartfirebbq.com>, but native mobile app is also fine) and paste the key only under *Advanced Settings* of your *Pit*.
![Smartfire webhook key](http://lintuala.fi/pics/hass-smartfire-webhook-key.jpg)
3. Create your IFTTT If this then that -application under <https://ifttt.com/create>. Select webhook for the trigger part (with a parameter *smartfire-status*) and webhook for the action part of the app. You can copy my values from my example screenshot above. I pass the eventname (for filtering purposes in Home Assistant) , timestamp, value1, value2 and value3 (these are sent by the hardware itself) to the webhook body.
4. I use node-red for my automations in home assistant, so I created an automation to there: If IFTTT Webhook is received and it’s from smartfire, do some magic, convert temperatures ot celsius and pass them to the Home Assistant entities.
![Node-RED](http://lintuala.fi/pics/hass-smartfire-nodered.png)  
[Find the JSON to Import to Node-RED here](./NodeRed.json).
5. The magic what is done in the Node-RED is to replace `=>` -marks with a colon `:`. This is done, because Smartfire is sending temperatures as a JSON Object and IFTTT understands only JSON Strings. IFTTT converts colons to => -marking, when it’s converting the payload from the Object to the String. In the Node-RED after converting payload to objects, we convert Fahrenheits to Celsius and round those to have not more than one decimal. If Celsius conversion is not needed, remove it from the function and live only the rounding.
6. Now you have all entities in the home assistant for dashboarding ready.

## Notifications

To get more value of the integration you can use values also for notifications. There are several ways to implement it, but I did it with Node-RED. In home assistant, insert input values for selecting master probe and the name of the food.
![Node-RED](http://lintuala.fi/pics/hass-smartfire-selectors.png)
[Find the YAML to configuration.yaml here](./smartfire-selector.yaml).
When these are available, create the automation to Node-RED.

My flow looks like this:
![Node-RED](http://lintuala.fi/pics/hass-smartfire-nodered-notifications.png)
[Find the JSON to Import ot Node-RED here](./NodeRedNotifications.json).
How does it then work:

- When the sensor state changes, it get's values and if the particular probe has been marked for a master probe, it goes forward
- In function it calculates the percentage of difference and also parses integer of difference
- When the master probe reaches >94% of readiness, it will notify to my sonos (change to whatever you want)
  - Narrowband alerts once
- When the master probe reaches >99% of readiness, it will notify to my sonos (change to whatever you want)
  - Narrowband alerts once
