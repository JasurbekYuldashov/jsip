# React JsSIP Wrapper

[![npm version](https://img.shields.io/npm/v/react-sip.svg)](https://www.npmjs.com/package/react-jssip-wrapper)
[![npm downloads](https://img.shields.io/npm/dy/react-sip.svg)](https://www.npmjs.com/package/react-jssip-wrapper)

React wrapper for [jssip](https://github.com/versatica/JsSIP).

## Installation

```bash
npm install react-jssip-wrapper
```

There is no need to install `jssip` as it is a dependency of `react-jssip-wrapper`.

## Usage

```js
import { SipProvider } from 'react-jssip-wrapper';
import App from './components/App';

ReactDOM.render(
  <SipProvider
    ref={ref} // For get jssip proporties and other functions
    host="sip.example.com"
    port={7443}
    pathname="/ws" // Path in socket URI (e.g. wss://sip.example.com:7443/ws); "" by default
    user="alice"
    password={sipPassword} // usually required (e.g. from ENV or props)
    autoRegister={true} // true by default, see jssip.UA option register
    autoAnswer={false} // automatically answer incoming calls; false by default
    iceRestart={false} // force ICE session to restart on every WebRTC call; false by default
    sessionTimersExpires={120} // value for Session-Expires header; 120 by default
    extraHeaders={{ // optional sip headers to send
      register: ['X-Foo: foo', 'X-Bar: bar'],
      invite: ['X-Foo: foo2', 'X-Bar: bar2']
    }}
    debug={false} // whether to output events to console; false by default
    setAction={setAction} // const [action,setAction] = useState()
    iceServers={[ // optional but recommended
        {
          urls: [
            "stun:stun.l.google.com:19302",
            "stun:stun1.l.google.com:19302",
          ],
        },
      ]}
  >
    <App />
  </SipProvider>
  document.getElementById('root'),
);
```

Child components get access to this context:

```js
{
  sip: sipType,
  call: callType,

  registerSip: PropTypes.func,
  unregisterSip: PropTypes.func,

  answerCall: PropTypes.func,
  startCall: PropTypes.func,
  stopCall: PropTypes.func,
}
```

See [lib/types.ts](./src/lib/types.ts) for technical details of what `sipType` and `callType` are.
An overview is given below:

### sip

`sip.status` represents SIP connection status and equals to one of these values:

- `'sipStatus/DISCONNECTED'` when `host`, `port` or `user` is not defined
- `'sipStatus/CONNECTING'`
- `'sipStatus/CONNECTED'`
- `'sipStatus/REGISTERED'` after calling `registerSip` or after `'sipStatus/CONNECTED'` when `autoRegister` is true
- `'sipStatus/ERROR'` in case of configuration, connection or registration problems

`sip.errorType`:

- `null` when `sip.status` is not `'sipStatus/ERROR'`
- `'sipErrorType/CONFIGURATION'`
- `'sipErrorType/CONNECTION'`
- `'sipErrorType/REGISTRATION'`

`sip.host`, `sip.port`, `sip.user`, `...` – `<SipProvider />`’s props (to make them easy to be displayed in the UI).

### call

`call.id` is a unique session id of the actual established voice call; `undefined` between calls

`call.status` represents the status of the call:

- `'callStatus/IDLE'` between calls (even when disconnected)
- `'callStatus/STARTING'` active incoming or outgoing call request
- `'callStatus/ACTIVE'` during ongoing call
- `'callStatus/STOPPING'` during call cancelation request

`call.direction` indicates the direction of the ongoing call:

- `null` between calls
- `'callDirection/INCOMING'`
- `'callDirection/OUTGOING'`

`call.counterpart` represents the call _destination_ in case of outgoing call and _caller_ for
incoming calls.
The format depends on the configuration of the SIP server (e.g. `"bob" <+998945667725@sip.example.com>`, `+998945667725@sip.example.com` or `Jasurbek@sip.example.com`).

### methods

When `autoRegister` is set to `false`, you can call `sipRegister()` and `sipUnregister()` manually for advanced registration scenarios.

To make calls, simply use these functions:

- `answerCall()`
- `startCall(destination)`
- `stopCall()`

The value for `destination` argument equals to the target SIP user without the host part (e.g. `+998945667725` or `bob`).
The omitted host part is equal to host you’ve defined in `SipProvider` props (e.g. `sip.example.com`).

---

The values for `sip.status`, `sip.errorType`, `call.status` and `call.direction` can be imported as constants to make typos easier to detect:

```js
import {
  SIP_STATUS_DISCONNECTED,
  //SIP_STATUS_...,
  CALL_STATUS_IDLE,
  //CALL_STATUS_...,
  SIP_ERROR_TYPE_CONFIGURATION,
  //SIP_ERROR_TYPE_...,
  CALL_DIRECTION_INCOMING,
  CALL_DIRECTION_OUTGOING,
} from "react-jssip-wrapper";
```

Custom PropTypes types are also provided by the library:

```js
import { callType, extraHeadersType, iceServersType, sipType } from "react-jssip-wrapper";
```
