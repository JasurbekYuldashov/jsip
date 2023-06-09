# React JsSIP Wrapper
React wrapper for [jssip](https://github.com/versatica/JsSIP).
For discussion [TelegramGroup](https://t.me/react_jssip_wrapper)

## Installation

```bash
npm install react-jssip-wrapper
```

There is no need to install `jssip` as it is a dependency of `react-jssip-wrapper`.

## Usage

```js
import React, { useCallback, useRef } from "react";
import { SipProvider } from "react-jssip-wrapper";
import { IStore, setSip } from "store";
import { useDispatch, useSelector } from "react-redux";

const Sip = () => {
  const dispatch = useDispatch();
  const ref = useRef<any>();
  const connectionConfig = useSelector(
    (state: IStore) => state.sip.connectionConfig
  );

  const onRefChange = useCallback((node: any) => {
    if (node === null) {
      // DOM node referenced by ref has been unmounted
    } else {
      dispatch(setSip({ ref: node }));
      ref.current = node;
    }
  }, []);

  if (!connectionConfig) {
    return null;
  }

  // const call = () =>
  //   ref.current?.startCall(`sip:${phone}@${connectionConfig.server}`);
  //
  // const transfer = () => {
  //   ref.current?.state?.rtcSession?.refer(
  //     `sip:${transferPhone}@${connectionConfig.server}`
  //   );
  // };

  return (
      <SipProvider
        host={connectionConfig.server as string}
        port={7443}
        pathname="" // Path in socket URI (e.g. wss://sip.example.com:7443/ws); "" by default
        user={connectionConfig.user as string}
        password={connectionConfig.password as string} // usually required (e.g. from ENV or props)
        autoRegister={false} // true by default, see jssip.UA option register
        // autoAnswer={true} // automatically answer incoming calls; false by default
        iceRestart={true} // force ICE session to restart on every WebRTC call; false by default
        sessionTimersExpires={30000}
        debug={false} // wh
        ref={onRefChange}
        iceServers={[
          {
            urls: [
              "stun:stun.l.google.com:19302",
              "stun:stun1.l.google.com:19302",
            ],
          },
        ]}
        setAction={(data: any) => {
          dispatch(setSip({ ...data, ref: ref.current }));
        }}
        audioId="newAudioId" // default 'sip-provider-audio' for output audio
      />
  );
};

export default Sip;
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
