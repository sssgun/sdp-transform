# SDP Transform [![Build Status](https://secure.travis-ci.org/clux/sdp-transform.png)](http://travis-ci.org/clux/sdp-transform) [![Dependency Status](https://david-dm.org/clux/sdp-transform.png)](https://david-dm.org/clux/sdp-transform)

A simple parser and writer of SDP. Defines internal grammar based on [RFC4566 - SDP](http://tools.ietf.org/html/rfc4566), and [RFC5245 - ICE](http://tools.ietf.org/html/rfc5245).

For simplicity it will force values that are integers to integers and leave everything else as strings when parsing. The module should be simple to extend or build upon, and is constructed rigorously.


## Usage - Parser
Require it and pass it an unprocessed SDP string.

```js
var transform = require('sdp-transform');

var sdpStr = "v=0\n\
o=- 20518 0 IN IP4 203.0.113.1\n\
s= \n\
t=0 0\n\
c=IN IP4 203.0.113.1\n\
a=ice-ufrag:F7gI\n\
a=ice-pwd:x9cml/YzichV2+XlhiMu8g\n\
a=fingerprint:sha-1 42:89:c5:c6:55:9d:6e:c8:e8:83:55:2a:39:f9:b6:eb:e9:a3:a9:e7\n\
m=audio 54400 RTP/SAVPF 0 96\n\
a=rtpmap:0 PCMU/8000\n\
a=rtpmap:96 opus/48000\n\
a=ptime:20\n\
a=sendrecv\n\
a=candidate:0 1 UDP 2113667327 203.0.113.1 54400 typ host\n\
a=candidate:1 2 UDP 2113667326 203.0.113.1 54401 typ host\n\
m=video 55400 RTP/SAVPF 97 98\n\
a=rtpmap:97 H264/90000\n\
a=fmtp:97 profile-level-id=4d0028;packetization-mode=1\n\
a=rtpmap:98 VP8/90000\n\
a=sendrecv\n\
a=candidate:0 1 UDP 2113667327 203.0.113.1 55400 typ host\n\
a=candidate:1 2 UDP 2113667326 203.0.113.1 55401 typ host\n\
";

var res = transform.parse(sdpStr);

res;
{ version: 0,
  origin:
   { username: '-',
     sessionId: 20518,
     sessionVersion: 0,
     netType: 'IN',
     ipVer: 4,
     address: '203.0.113.1' },
  name: '',
  timing: { start: 0, stop: 0 },
  connection: { version: 4, ip: '203.0.113.1' },
  iceUfrag: 'F7gI',
  icePwd: 'x9cml/YzichV2+XlhiMu8g',
  fingerprint:
   { type: 'sha-1',
     hash: '42:89:c5:c6:55:9d:6e:c8:e8:83:55:2a:39:f9:b6:eb:e9:a3:a9:e7' },
  media:
   [ { rtp: [Object],
       fmtp: [],
       type: 'audio',
       port: 54400,
       protocol: 'RTP/SAVPF',
       payloads: '0 96',
       ptime: 20,
       sendrecv: 'sendrecv',
       candidates: [Object] },
     { rtp: [Object],
       fmtp: [Object],
       type: 'video',
       port: 55400,
       protocol: 'RTP/SAVPF',
       payloads: '97 98',
       sendrecv: 'sendrecv',
       candidates: [Object] } ] }


// each media line is parsed into the following format
res.media[1];
{ rtp:
   [ { payload: 97,
       codec: 'H264',
       rate: 90000 },
     { payload: 98,
       codec: 'VP8',
       rate: 90000 } ],
  fmtp:
   [ { payload: 97,
       config: 'profile-level-id=4d0028;packetization-mode=1' } ],
  type: 'video',
  port: 55400,
  protocol: 'RTP/SAVPF',
  payloads: '97 98',
  sendrecv: 'sendrecv',
  candidates:
   [ { foundation: 0,
       component: 1,
       transport: 'UDP',
       priority: 2113667327,
       ip: '203.0.113.1',
       port: 55400,
       type: 'host' },
     { foundation: 1,
       component: 2,
       transport: 'UDP',
       priority: 2113667326,
       ip: '203.0.113.1',
       port: 55401,
       type: 'host' } ] }
```

In this example, only slightly dodgy string coercion case here is for `candidates[i].foundation`, which can be a string, but in this case can be equally parsed as an integer.

### Parser Postprocessing
No excess parsing is done to the raw strings apart from maybe coercing to ints, because the writer is built to be the inverse of the parser. That said, a few helpers have been built in:

```js
// to parse the fmtp.config from the previous example
transform.parseFmtpConfig(res.media[1].fmtp[0].config);
{ 'profile-level-id': '4d0028',
  'packetization-mode': 1 }

// what payloads where actually advertised in the main m-line ?
transform.parsePayloads(res.media[1].payloads);
[97, 98]
```


## Usage - Writer
The writer is the inverse of the parser, and will need a struct equivalent to the one returned by it.

```js
transform.write(res).split('\n'); // res parsed above
[ 'v=0',
  'o=- 20518 0 IN IP4 203.0.113.1',
  's= ',
  'c=IN IP4 203.0.113.1',
  't=0 0',
  'a=ice-ufrag:F7gI',
  'a=ice-pwd:x9cml/YzichV2+XlhiMu8g',
  'a=fingerprint:sha-1 42:89:c5:c6:55:9d:6e:c8:e8:83:55:2a:39:f9:b6:eb:e9:a3:a9:e7',
  'm=audio 54400 RTP/SAVPF 0 96',
  'a=rtpmap:0 PCMU/8000',
  'a=rtpmap:96 opus/48000',
  'a=ptime:20',
  'a=sendrecv',
  'a=candidate:0 1 UDP 2113667327 203.0.113.1 54400 typ host',
  'a=candidate:1 2 UDP 2113667326 203.0.113.1 54401 typ host',
  'm=video 55400 RTP/SAVPF 97 98',
  'a=rtpmap:97 H264/90000',
  'a=rtpmap:98 VP8/90000',
  'a=fmtp:97 profile-level-id=4d0028;packetization-mode=1',
  'a=sendrecv',
  'a=candidate:0 1 UDP 2113667327 203.0.113.1 55400 typ host',
  'a=candidate:1 2 UDP 2113667326 203.0.113.1 55401 typ host' ]
```

The only thing different from the original input is we follow the order specified by the SDP RFC, and we will always do so.

## Installation
Install locally from npm:

```bash
$ npm install sdp-transform
```

## License
MIT-Licensed. See LICENSE file for details.
