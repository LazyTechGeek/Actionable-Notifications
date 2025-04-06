Actionable Notifications

Link to YouTube Video 📺 Watch the video here: https://youtu.be/Nlc3sahKBTc

The following scripts referenced in the video as as follows:

Links:
💻 Alexa Developer Console:
https://developer.amazon.com/alexa/console/ask

🧠 Interaction Model (Skill Package):
https://github.com/keatontaylor/alexa-actions/tree/master/skill-package/interactionModels/custom

📦 AlexaActionsNoBinaryLinux.zip:
https://github.com/keatontaylor/alexa-actions/releases


# Required Code: (input_text & activate_alexa_actionable_notification)

# input_text
Make sure to install it i.e. into /config/configuration.yaml (or split location)

```
input_text:
  alexa_actionable_notification:
    name: Alexa Actionable Notification Holder
    max: 255
    initial: '{"text": "This is a test of the alexa actionable notifications skill. Did it work?", "event": "actionable.skill.test", "suppress_confirmation": "false"}'
```

# activate_alexa_actionable_notification
Make sure to install it i.e. into /config/scripts.yaml (or split location). Replace <Your Skill ID> with your skills ID.

```
activate_alexa_actionable_notification:
  alias: activate_alexa_actionable_notification
  description: Activates an actionable notification on a specific echo device
  fields:
    text:
      description: The text you would like alexa to speak.
      example: 'What would you like the thermostat set to?'
    event_id:
      description: Correlation ID for event responses
      example: 'ask_for_temperature'
    alexa_device:
      description: Alexa device you want to trigger
      example: 'media_player.bedroom_echo'
    suppress_confirmation:
      description: Set true if you want to suppress 'okay' confirmation
      example: 'true'
  sequence:
    - action: input_text.set_value
      target:
        entity_id: input_text.alexa_actionable_notification
      data_template:
        value: '{"text": "{{ text }}", "event": "{{ event_id }}", "suppress_confirmation": "{{ suppress_confirmation }}"}'
    - action: media_player.play_media
      target:
        entity_id: "{{ alexa_device }}"
      data_template:
        media_content_type: skill
        media_content_id: <Your Skill ID>
  mode: single
```

Example Code for automations / Scripts demonstrated in this video

# Mobile Actionable notifications


Name: Mobile-Notification_Cookie Jar Breach - (if away)  
Description: Triggers notification when cookie jar opened and dave is not home  
(save as automation)

```
alias: Mobile-Notification_Cookie Jar Breach - (if away)
description: ""
triggers:
  - entity_id: binary_sensor.cookie_jar_contact
    from: "off"
    to: "on"
    trigger: state
conditions:
  - condition: state
    entity_id: person.dave
    state: not_home
actions:
  - data:
      title: Cookie Jar Breach!
      message: Shall I activate cookie defense mode?
      data:
        actions:
          - action: COOKIE_DEFENSE_YES
            title: Yes, scare them!
          - action: COOKIE_DEFENSE_NO
            title: No, ignore it
    action: notify.mobile_app_samsung_s23
  - delay: "00:00:15"
mode: single

```

Name: Mobile-Notification_Cookie Jar Breach - (if away)  
Description: Triggers notification when cookie jar opened and dave is not home  
(Save as automations)
```
alias: Handle Cookie Jar Mobile Response
description: ""
triggers:
  - event_type: mobile_app_notification_action
    event_data:
      action: COOKIE_DEFENSE_YES
    trigger: event
  - event_type: mobile_app_notification_action
    event_data:
      action: COOKIE_DEFENSE_NO
    trigger: event
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.event.data.action == 'COOKIE_DEFENSE_YES' }}"
        sequence:
          - repeat:
              count: 5
              sequence:
                - target:
                    entity_id: switch.sonoff_kitchen
                  action: switch.turn_off
                  data: {}
                - delay: "00:00:0.4"
                - target:
                    entity_id: switch.sonoff_kitchen
                  action: switch.turn_on
                  data: {}
                - delay: "00:00:0.4"
      - conditions:
          - condition: template
            value_template: "{{ trigger.event.data.action == 'COOKIE_DEFENSE_NO' }}"
        sequence:
          - data:
              title: Cookie breach ignored
              message: You chose not to activate defense mode.
            action: notify.persistent_notification
mode: single
```

Name: Cookie Jar Mobile Response - Call Script  
Description: Triggers response when cookie jar opened and dave is not home  
(Save as automation)

```
alias: Cookie Jar Mobile Response - Call Script
description: ""
triggers:
  - event_type: mobile_app_notification_action
    event_data:
      action: COOKIE_DEFENSE_YES
    trigger: event
  - event_type: mobile_app_notification_action
    event_data:
      action: COOKIE_DEFENSE_NO
    trigger: event
conditions: []
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.event.data.action == 'COOKIE_DEFENSE_YES' }}"
        sequence:
          - action: script.flash_lights_cookie_defense
            data: {}
      - conditions:
          - condition: template
            value_template: "{{ trigger.event.data.action == 'COOKIE_DEFENSE_NO' }}"
        sequence:
          - data:
              title: Cookie breach ignored
              message: You chose not to activate defense mode.
            action: notify.persistent_notification
mode: single
```

Name: Flash Lights Cookie Defense  
Description: script to run sequence from response  
(Save as script)

```
alias: Flash Lights Cookie Defense
sequence:
  - repeat:
      count: 7
      sequence:
        - target:
            entity_id: light.sonoff_1000dbd502
          action: light.turn_off
          data: {}
        - delay: "00:00:1.0"
        - target:
            entity_id: light.sonoff_1000dbd502
          action: light.turn_on
          data: {}
        - delay: "00:00:0.3"
mode: single
description: ""
```

# Alexa Actionable notifications

Name: Alexa-Notification_Cookie Jar Breach  
Description: Triggers notification when cookie jar opened and dave is home  
(Save as automation)
```

alias: Alexa-Notification_Cookie Jar Breach
description: ""
triggers:
  - entity_id: binary_sensor.cookie_jar_contact
    from: "off"
    to: "on"
    trigger: state
conditions:
  - condition: state
    entity_id: person.dave
    state: home
actions:
  - data:
      text: Warning. The cookie jar has been accessed. Was that you eating?
      event_id: cookie_jar_breach
      alexa_device: media_player.dave_s_echo_spot
    action: script.activate_alexa_actionable_notification
  - delay: "00:00:15"
mode: single
```

Name: Alexa-Response_Cookie Jar Breach  
Description: Triggers response when cookie jar opened and dave is home  
(Save as automation)

```
alias: Alexa-Response_Cookie Jar Breach
description: Responds based on what Dave tells Alexa
triggers:
  - event_type: alexa_actionable_notification
    event_data:
      event_id: cookie_jar_breach
    trigger: event
conditions: []
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.event.data.event_response == 'ResponseYes' }}"
        sequence:
          - data:
              message: Dave admitted to the cookie theft
              title: Title
            action: notify.alexa_media_dave_s_echo_spot
      - conditions:
          - condition: template
            value_template: "{{ trigger.event.data.event_response == 'ResponseNo' }}"
        sequence:
          - data:
              message: Dave denied it... but we know better
              title: Title
            action: notify.alexa_media_dave_s_echo_spot
mode: single
```

Name: Alexa-Response_Cookie Jar Breach (calls script)  
Description: Triggers response when cookie jar opened and dave is home and calls script to run the sequences  
(Save as automation)

```
alias: Alexa-Response_Cookie Jar Breach (calls script)
description: Responds based on what Dave tells Alexa
triggers:
  - event_type: alexa_actionable_notification
    event_data:
      event_id: cookie_jar_breach
    trigger: event
conditions: []
actions:
  - data:
      response: "{{ trigger.event.data.event_response }}"
    action: script.alexa_response_cookie_jar_breach
mode: single
```

Name: Alexa-Response_Cookie Jar Breach (calls script)  
Description: script to run sequence from response  
(Save as script)
```
alias: Alexa-Response_Cookie Jar Breach
description: Handles Alexa response to cookie jar breach
fields:
  response:
    description: The response from Alexa
    example: ResponseYes
sequence:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ response == 'ResponseYes' }}"
        sequence:
          - data:
              message: Dave admitted to the cookie theft
              title: Title
            action: notify.alexa_media_dave_s_echo_spot
      - conditions:
          - condition: template
            value_template: "{{ response == 'ResponseNo' }}"
        sequence:
          - data:
              message: Dave denied it... but we know better
              title: Title
            action: notify.alexa_media_dave_s_echo_spot
mode: single
```
