## Ritvik Choudhary

## Arduino?

- Embedded systems
- Wearable computing
- Low-power systems

## Overview

- Running an HTTP server on the arduino
- Requesting the HTML from the arduino
- Loading angular app
- Communicating to Angular from the Arduino
- Communicating to the Arduino from Angular


## Enter TinyWebServer

Enables simplification of the c HTTP server code

-----------------------

## Connecting to the net

- Ethernet shield
- Wifi shield

## But where's the HTML

## Options

#### Embed HTML in Arduino
#### Read/Send from SD card
#### Load from remote server

-------------------------
## But where's the Angular
----------------------------------

```javascript
(function() {
  var scriptTag = document.getElementsByTagName('script')[0];
  // ...
  createLinkTag('styles/main.css');

  var arr = [
    'scripts/modules/arduino.js',
    'scripts/app.js'
    // ...
  ];

  createScriptTag('bower_components/angular/angular.js');
  createScriptTag('bower_components/angular-route/angular-route.js');

  arr.forEach(function(src) { createScriptTag(src); });
  // Bootstrap
  body.setAttribute('ng-app', 'myApp');
  var app   = document.createElement('div');
  var main  = document.createElement('div');
  main.setAttribute('main-view', '');
  app.appendChild(main);
  body.appendChild(app);
})();
```

## Command and Control

* Arduino -> Angular (√)
* Angular -> Arduino (...)

## Expose the local ip to the browser

-----------

```javascript
angular.module('fsArduino', [])
.provider('Arduino', function() {
  this.setRootUrl = function(u) {
    rootUrl = u || rootUrl;
  };

  this.$get = function($http) {
    return {
      getPins: function() {},
      setPins: function() {}
    };
  }

});
```

```javascript
angular.module('myApp', [
  'fsArduino'
])
.config(function(ArduinoProvider) {
  ArduinoProvider.setRootUrl(window.ip);
});
```

## Summary of hardware hacking

- Turning pins on/HIGH
- Turning pins off/LOW
- Measuring pin voltage

## Getting pin status

```javascript
// ...
getPins: function() {
  return $http({
    method: 'GET',
    url: rootUrl + '/pins'
  }).then(function(data) {
    return data.data;
  });
},
// ...
```

## Serving pin status

```c
// GET /pins
boolean pins_handler(TinyWebServer& web_server) {
  web_server.send_error_code(200);
  web_server.send_content_type("application/javascript");
  web_server.end_headers();
  pinsToString(web_server);
  return true;
}
// ...
bool pinsToString(TinyWebServer& web_server) {
  web_server << F("{\"pins\":[");
  int len = numPins;
  for(int i=0; i<len; i++){
    web_server << F("{\"pin\":");
    web_server << pins[i].getPin();
    web_server << F(",\"value\":");
    web_server << pins[i].getState();
    web_server << F("}");
    if ((i+1) < len) web_server << F(",");
  }
  web_server << F("]}");
  return true;
}
```

## Modifying pin states

Angular works with JSON by default (just javascript), but the Arduino does not... However, parsing a schemaless data structure in a strictly typed language is... <span class="emp">difficult</span>.

## Create our own protocol

Turn JSON from:

```javascript
{ pin: 7, action: 'getTemp' } (24 bytes)
```

to

```c
p7a0 (4 bytes)
```

## Actions

```javascript
  // in mainview directive
  Arduino.setPins([
    { pin: temp, action: 'getTemp' }
  ]);
  // in Arduino provider
  var actions = {
    'getTemp': 0
  };
  var actionifyPins = function(pins) {
    var str = '';
    for (var i = 0; i < pins.length; i++) {
      var p = pins[i];
      str += 'p' + p.pin;
      if (typeof(p.mode) !== 'undefined') {str += 'm' + p.mode;}
      if (typeof(p.value) !== 'undefined') {str += 'v' + p.value;}
      if (typeof(p.action) !== 'undefined') {str += 'a' + actions[p.action];}
    }
    return str;
  };
```
-------------

```javascript
{ pin: 7, action: 'getTemp' } (24 bytes)
```

to

```c
p7a0 (4 bytes)
```

## Using it service

```javascript
setPins: function(pins) {
  var strAction = actionifyPins(pins);
  return $http({
    method: 'POST',
    url: rootUrl + '/pins/digital',
    data: strAction,
    headers: {'X-Action-Len': strAction.length}
  }).then(function(data) {
    return data.data;
  });
}
```

## Parsing in c

-----------------------------

## Thanks

### Ritvik Choudhary