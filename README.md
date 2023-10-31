# Extended OpenAI Conversation
This is custom component of Home Assistant.

Derived from build-in [OpenAI Conversation](https://www.home-assistant.io/integrations/openai_conversation/) with some new features such as call-service.

## Additional Features
- Ability to call service of Home Assistant
- Ability to create automation
- Ability to get data from API or web page

## How it works
Extended OpenAI Conversation uses OpenAI API's feature of [function calling](https://platform.openai.com/docs/guides/gpt/function-calling) to call service of Home Assistant.

Since "gpt-3.5-turbo" model already knows how to call service of Home Assistant in general, you just have to let model know what devices you have by [exposing entities](https://github.com/jekalmin/extended_openai_conversation#preparation)

## Installation
1. Install via HACS or by copying `extended_openai_conversation` folder into `<config directory>/custom_components`
2. Restart Home Assistant
3. Go to Settings > Devices & Services.
4. In the bottom right corner, select the Add Integration button.
5. Follow the instructions on screen to complete the setup (API Key is required).
    - [Generating an API Key](https://www.home-assistant.io/integrations/openai_conversation/#generate-an-api-key)
6. Go to Settings > Voice Assistants.
7. Click to edit Assistant (named "Home Assistant" by default).
8. Select "Extended OpenAI Conversation" from "Conversation agent" tab.
    <details>

    <summary>guide image</summary>
    <img width="500" alt="스크린샷 2023-10-07 오후 6 15 29" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/0849d241-0b82-47f6-9956-fdb82d678aca">

    </details>

## Preparation
After installed, you need to expose entities from "http://{your-home-assistant}/config/voice-assistants/expose".

## Examples
### 1. Turn on single entity
https://github.com/jekalmin/extended_openai_conversation/assets/2917984/938dee95-8907-44fd-9fb8-dc8cd559fea2

### 2. Turn on multiple entities
https://github.com/jekalmin/extended_openai_conversation/assets/2917984/528f5965-94a7-4cbe-908a-e24f7bbb0a93

### 3. Hook with custom notify function
https://github.com/jekalmin/extended_openai_conversation/assets/2917984/4a575ee7-0188-41eb-b2db-6eab61499a99

### 4. Add automation
https://github.com/jekalmin/extended_openai_conversation/assets/2917984/04b93aa6-085e-450a-a554-34c1ed1fbb36

## Configuration
### Options
By clicking a button from Edit Assist, Options can be customized.<br/>
Options include [OpenAI Conversation](https://www.home-assistant.io/integrations/openai_conversation/) options and two new options. 


- `Maximum Function Calls Per Conversation`: limit the number of function calls in a single conversation.
(Sometimes function is called over and over again, possibly running into infinite loop) 
- `Functions`: A list of mappings of function spec to function.
  - `spec`: Function which would be passed to [functions](https://platform.openai.com/docs/api-reference/chat/create#chat/create-functions) of [chat API](https://platform.openai.com/docs/api-reference/chat/create).
  - `function`: function that will be called.


| Edit Assist                                                                                                                                  | Options                                                                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <img width="608" alt="1" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/bb394cd4-5790-4ac9-9311-dbcab0fcca56"> | <img width="591" alt="스크린샷 2023-10-10 오후 10 53 57" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/431e4bc5-87a0-4d7b-8da0-6273f955877f"> |


### Functions

#### Supported function types
- `native`: function that is provided by "extended_openai_conversation".
  - Currently supported native functions and parameters are:
    - `execute_service`
      - `domain`(string): domain to be passed to `hass.services.async_call`
      - `service`(string): service to be passed to `hass.services.async_call`
      - `service_data`(string): service_data to be passed to `hass.services.async_call`
    - `add_automation`
      - `automation_config`(string): An automation configuration in a yaml format
- `script`: A list of services that will be called
- `template`: The value to be returned from function.
- `rest`: Getting data from REST API endpoint.
- `scrape`: Scraping information from website

Below is a default configuration of functions.

```yaml
- spec:
    name: execute_services
    description: Use this function to execute service of devices in Home Assistant.
    parameters:
      type: object
      properties:
        list:
          type: array
          items:
            type: object
            properties:
              domain:
                type: string
                description: The domain of the service
              service:
                type: string
                description: The service to be called
              service_data:
                type: object
                description: The service data object to indicate what to control.
                properties:
                  entity_id:
                    type: string
                    description: The entity_id retrieved from available devices. It must start with domain, followed by dot character.
                required:
                - entity_id
            required:
            - domain
            - service
            - service_data
  function:
    type: native
    name: execute_service
```

## Function Usage
This is an example of configuration of functions.

Copy and paste below yaml configuration into "Functions".<br/>
Then you will be able to let OpenAI call your function. 

### 1. template
#### 1-1. Get current weather

```yaml
- spec:
    name: get_current_weather
    description: Get the current weather in a given location
    parameters:
      type: object
      properties:
        location:
          type: string
          description: The city and state, e.g. San Francisco, CA
        unit:
          type: string
          enum:
          - celcius
          - farenheit
      required:
      - location
  function:
    type: template
    value_template: The temperature in {{ location }} is 25 {{unit}}
```

<img width="300" alt="스크린샷 2023-10-07 오후 7 56 27" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/05e31ea5-daab-4759-b57d-9f5be546bac8">

### 2. script
#### 2-1. Add item to shopping cart
```yaml
- spec:
    name: add_item_to_shopping_cart
    description: Add item to shopping cart
    parameters:
      type: object
      properties:
        item:
          type: string
          description: The item to be added to cart
      required:
      - item
  function:
    type: script
    sequence:
    - service: shopping_list.add_item
      data:
        name: '{{item}}'
```

<img width="300" alt="스크린샷 2023-10-07 오후 7 54 56" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/89060728-4703-4e57-8423-354cdc47f0ee">

#### 2-2. Send messages to another messenger

In order to accomplish "send it to Line" like [example3](https://github.com/jekalmin/extended_openai_conversation#3-hook-with-custom-notify-function), register a notify function like below.

```yaml
- spec:
    name: send_message_to_line
    description: Use this function to send message to Line.
    parameters:
      type: object
      properties:
        message:
          type: string
          description: message you want to send
      required:
      - message
  function:
    type: script
    sequence:
    - service: script.notify_all
      data:
        message: "{{ message }}"
```

#### 2-3. Get events from calendar

In order to pass result of calling service to OpenAI, set response variable to `_function_result`. 

```yaml
- spec:
    name: get_events
    description: Use this function to get list of calendar events.
    parameters:
      type: object
      properties:
        start_date_time:
          type: string
          description: The start date time
        end_date_time:
          type: string
          description: The end date time
      required:
      - start_date_time
      - end_date_time
  function:
    type: script
    sequence:
    - service: calendar.list_events
      data:
        start_date_time: "{{start_date_time}}"
        end_date_time: "{{end_date_time}}"
      target:
        entity_id: calendar.test
      response_variable: _function_result
```

<img width="300" alt="스크린샷 2023-10-31 오후 9 04 56" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/7a6c6925-a53e-4363-a93c-45f63951d41b">

### 3. native

#### 3-1. Add automation

Before adding automation, I highly recommend set notification on `automation_registered_via_extended_openai_conversation` event and create separate "Extended OpenAI Assistant" and "Assistant"

(Automation can be added even if conversation fails because of failure to get response message, not automation)

| Create Assistant                                                                                                                             | Notify on created                                                                                                                                                              |
|----------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <img width="830" alt="1" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/b7030a46-9a4e-4ea8-a4ed-03d2eb3af0a9"> | <img width="1116" alt="스크린샷 2023-10-13 오후 6 01 40" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/7afa3709-1c1d-41d0-8847-70f2102d824f"> |


Copy and paste below configuration into "Functions"

**For English**
```yaml
- spec:
    name: add_automation
    description: Use this function to add an automation in Home Assistant.
    parameters:
      type: object
      properties:
        automation_config:
          type: string
          description: A configuration for automation in a valid yaml format. Next line character should be \n. Use devices from the list.
      required:
      - automation_config
  function:
    type: native
    name: add_automation
```

**For Korean**
```yaml
- spec:
    name: add_automation
    description: Use this function to add an automation in Home Assistant.
    parameters:
      type: object
      properties:
        automation_config:
          type: string
          description: A configuration for automation in a valid yaml format. Next line character should be \\n, not \n. Use devices from the list.
      required:
      - automation_config
  function:
    type: native
    name: add_automation
```

<img width="300" alt="스크린샷 2023-10-31 오후 9 32 27" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/55f5fe7e-b1fd-43c9-bce6-ac92e203598f">

### 4. scrape
#### 4-1. Get current HA version
Scrape version from webpage, "https://www.home-assistant.io"

Unlike [scrape](https://www.home-assistant.io/integrations/scrape/), "value_template" is added at root level in which scraped data from sensors are passed.

```yaml
scrape:
- spec:
    name: get_ha_version
    description: Use this function to get Home Assistant version
    parameters:
      type: object
      properties:
        dummy:
          type: string
          description: Nothing
  function:
    type: scrape
    resource: https://www.home-assistant.io
    value_template: "version: {{version}}, release_date: {{release_date}}"
    sensor:
      - name: version
        select: ".current-version h1"
        value_template: '{{ value.split(":")[1] }}'
      - name: release_date
        select: ".release-date"
        value_template: '{{ value.lower() }}'
```

<img width="300" alt="스크린샷 2023-10-31 오후 9 46 07" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/e640c3f3-8d68-486b-818e-bd81bf71c2f7">

### 5. rest
#### 5-1. Get friend names
- Sample URL: https://jsonplaceholder.typicode.com/users
```yaml
- spec:
    name: get_friend_names
    description: Use this function to get friend_names
    parameters:
      type: object
      properties:
        dummy:
          type: string
          description: Nothing.
  function:
    type: rest
    resource: https://jsonplaceholder.typicode.com/users
    value_template: '{{value_json | map(attribute="name") | list }}'
```

<img width="300" alt="스크린샷 2023-10-31 오후 9 48 36" src="https://github.com/jekalmin/extended_openai_conversation/assets/2917984/f968e328-5163-4c41-a479-76a5406522c1">



## Logging
In order to monitor logs of API requests and responses, add following config to `configuration.yaml` file

```yaml
logger:
  logs:
    custom_components.extended_openai_conversation: info
```