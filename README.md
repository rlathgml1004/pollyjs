<p align="center">
  <img alt="Polly.JS" width="400px" src="https://netflix.github.io/pollyjs/assets/images/wordmark-logo-alt.png" />
</p>
<h2 align="center">Record, Replay, and Stub HTTP Interactions</h2>

[![Build Status](https://travis-ci.org/Netflix/pollyjs.svg?branch=master)](https://travis-ci.org/Netflix/pollyjs)
[![license](https://img.shields.io/github/license/Netflix/pollyjs.svg)](http://www.apache.org/licenses/LICENSE-2.0)

Polly.JS is a standalone, framework-agnostic JavaScript library that enables recording, replaying, and stubbing of HTTP interactions. By tapping into multiple request APIs across both Node & the browser, Polly.JS is able to mock requests and responses with little to no configuration while giving you the ability to take full control of each request with a simple, powerful, and intuitive API. 
// Polly. JS는 HTTP 상호작용을 기록 및 재생을 가능하게 하는 독립적이고 틀에 얽매이지 않는 JavaScript 라이브러리이다. 노드와 브라우저 모두에서 여러 요청  API를 사용함으로써 사용자가 단순하고 강력하고 즉각적인 API로 각 요청을 완전히 제어할 수 있는 능력을 제공하는 반면, Polly. JS는 두서없이 요청과 응답을 무시할 수 있다.

Interested in contributing or just seeing Polly in action? Head over to [CONTRIBUTING.md](CONTRIBUTING.md) to learn how to spin up the project! 
//Polly의 활동을 보거나 기여하고자 하는 데에 흥미가 있는가? CONTRIBUTING.Md로 향하라! 그리고 어떤 프로젝트를 추진하는지 배워라!

## Why Polly? // 왜 폴리인가?

Keeping fixtures and factories in parity with your APIs can be a time consuming process.
Polly alleviates this by recording and maintaining actual server responses without foregoing flexibility. 
// 비품과 공장을 API와 동등하게 유지하는 것은 시간 낭비이다. Polly는 실제 서버 응답을 기록하고 유지함으로써 이와 같은 문제를 완화한다.

- Record your test suite's HTTP interactions and replay them during future test runs for fast, deterministic, accurate tests.
//빠르고 결정론적이며 정확한 테스트를 위해 테스트 제품군의 HTTP 상호작용을 기록하고 재생해보아라.

- Use Polly's client-side server to modify or intercept requests and responses to simulate different application states (e.g. loading, error, etc.).
//Polly의 사용자 측 서버를 사용해 수정하거나 요청 및 응답을 가로채서 다른 애플리케이션 상태를 실험해보아라.

## Features // 특징

- 🚀 Node & Browser Support // 노드와 브라우저 지원
- ⚡️️ Simple, Powerful, & Intuitive API // 단순, 강력하고 직관적인 API
- 💎 First Class Mocha & QUnit Test Helpers // 일류 Mocha 및 QUnit 테스트 지원
- 🔥 Intercept, Pass-Through, and Attach Events // 이벤트 , 통과 및 연결
- 📼 Record to Disk or Local Storage // 디스크 또는 로컬저장소에 기록
- ⏱ Slow Down or Speed Up Time // 속도저하 또는 속도향상

## Getting Started // 시작하기

Check out the [Quick Start](https://netflix.github.io/pollyjs/#/quick-start) documentation to get started.
// 시작하려면 Quick Start 문서를 확인해라

## Usage // 사용

Lets take a look at what an example test case would look like using Polly.
//Polly를 사용한 테스트 사례를 살펴보자

```js
import { Polly } from '@pollyjs/core';
import XHRAdapter from '@pollyjs/adapter-xhr';
import FetchAdapter from '@pollyjs/adapter-fetch';
import RESTPersister from '@pollyjs/persister-rest';

/*
  Register the adapters and persisters we want to use. This way all future
  polly instances can access them by name.
*/
/*
우리가 사용하고자 하는 어댑터와 'persister'를 지정하라.
이런 방법으로 모든 Polly 인스턴스들이 그들(어댑터, persister)에 해당 이름으로 접근할 수 있다.
*/
Polly.register(XHRAdapter);
Polly.register(FetchAdapter);
Polly.register(RESTPersister);

describe('Netflix Homepage', function() {
  it('should be able to sign in', async function() {
    /*
      Create a new polly instance.

      Connect Polly to both fetch and XHR browser APIs. By default, it will
      record any requests that it hasn't yet seen while replaying ones it
      has already recorded.
    */
    /*
    Polly 인스턴스를 생성하라.
    Polly를 가져오기(fetch) 및 XHR브라우저 API에 연결하라.
    기본적으로, 그것은 이미 녹음된 요청을 재생하는 동안 아직 발견되지 못한 요청을 기록할 것이다. 
    */
    const polly = new Polly('Sign In', {
      adapters: ['xhr', 'fetch'],
      persister: 'rest'
    });
    const { server } = polly;

    /* Intercept all Google Analytic requests and respond with a 200 */
    /* 모든 구글 분석요청을 가로채고 200으로 응답하라 */
    server
      .get('/google-analytics/*path')
      .intercept((req, res) => res.sendStatus(200));

    /* Pass-through all GET requests to /coverage */
    /* 얻어진 요청들 모두를 /coverage로 이동시켜라 */
    server.get('/coverage').passthrough();

    /* start: pseudo test code */
    /* 모의실험 시작 */
    await visit('/login');
    await fillIn('email', 'polly@netflix.com');
    await fillIn('password', '@pollyjs');
    await submit();
    /* end: pseudo test code */
    /* 모의실험 종료 */

    expect(location.pathname).to.equal('/browse');

    /*
      Calling `stop` will persist requests as well as disconnect from any
      connected browser APIs (e.g. fetch or XHR).
    */
    /* 'stop' 을 요청하면 연결된 모든 브라우저 API 에서 연결이 끊긴다. */
    await polly.stop();
  });
});
```

The above test case would generate the following [HAR](http://www.softwareishard.com/blog/har-12-spec/)
file which Polly will use to replay the sign-in response when the test is rerun:
// 위의 테스트 경우는 테스트가 재실행될 때 Polly가 로그인 응답을 재생하는 데 사용할 다음
과 같은 HAR 파일을 생성한다.

```json
{
  "log": {
    "_recordingName": "Sign In",
    "browser": {
      "name": "Chrome",
      "version": "67.0"
    },
    "creator": {
      "name": "Polly.JS",
      "version": "0.5.0",
      "comment": "persister:rest"
    },
    "entries": [
      {
        "_id": "06f06e6d125cbb80896c41786f9a696a",
        "_order": 0,
        "cache": {},
        "request": {
          "bodySize": 51,
          "cookies": [],
          "headers": [
            {
              "name": "content-type",
              "value": "application/json; charset=utf-8"
            }
          ],
          "headersSize": 97,
          "httpVersion": "HTTP/1.1",
          "method": "POST",
          "postData": {
            "mimeType": "application/json; charset=utf-8",
            "text": "{\"email\":\"polly@netflix.com\",\"password\":\"@pollyjs\"}"
          },
          "queryString": [],
          "url": "https://netflix.com/api/v1/login"
        },
        "response": {
          "bodySize": 0,
          "content": {
            "mimeType": "text/plain; charset=utf-8",
            "size": 0
          },
          "cookies": [],
          "headers": [],
          "headersSize": 0,
          "httpVersion": "HTTP/1.1",
          "redirectURL": "",
          "status": 200,
          "statusText": "OK"
        },
        "startedDateTime": "2018-06-29T17:31:55.348Z",
        "time": 11,
        "timings": {
          "blocked": -1,
          "connect": -1,
          "dns": -1,
          "receive": 0,
          "send": 0,
          "ssl": -1,
          "wait": 11
        }
      }
    ],
    "pages": [],
    "version": "1.2"
  }
}
```

## Credits // 권한

_In alphabetical order:_
//알파벳 순서로:

- [Jason Mitchell](https://twitter.com/_jasonmit) - Creator / Maintainer // 제작자/관리자
- [Offir Golan](https://twitter.com/offirgolan) - Creator / Maintainer // 제작자/관리자
- [Sophinie Som](https://twitter.com/s0phinie) - Branding / Logo // 브랜딩/로고

## Prior Art // 선행기술

The "Client Server" API of Polly is heavily influenced by the very popular mock server library [pretender](https://github.com/pretenderjs/pretender). Pretender supports XHR and Fetch stubbing and is a great lightweight alternative to Polly if your project does not require persisting capabilities or Node adapters.
// Polly의 클라이언트 서버 API는 유명한 Mock Server 라이브러리 pretender에 의해 영향을 받았다. pretender는 XHR와 가져오기(fetch)와 같은 기능을 지원하고, Polly에게 좋은 대안이다.

Thank you to all contributors especially the maintainers: [trek](https://github.com/trek), [stefanpenner](https://github.com/stefanpenner), and [xg-wang](https://github.com/xg-wang).
// 모든 기여자, 특히 관리자인 trek, stefanpenner,그리고 xg-wang 에게 감사를 표한다.
## Contributors // 기여자

[//]: contributor-faces

<a href="https://github.com/offirgolan"><img src="https://avatars2.githubusercontent.com/u/575938?v=4" title="offirgolan" width="80" height="80"></a>
<a href="https://github.com/jasonmit"><img src="https://avatars1.githubusercontent.com/u/3108309?v=4" title="jasonmit" width="80" height="80"></a>
<a href="https://github.com/cibernox"><img src="https://avatars2.githubusercontent.com/u/265339?v=4" title="cibernox" width="80" height="80"></a>
<a href="https://github.com/DenrizSusam"><img src="https://avatars1.githubusercontent.com/u/39295979?v=4" title="DenrizSusam" width="80" height="80"></a>
<a href="https://github.com/dustinsoftware"><img src="https://avatars3.githubusercontent.com/u/942358?v=4" title="dustinsoftware" width="80" height="80"></a>
<a href="https://github.com/silverchen"><img src="https://avatars0.githubusercontent.com/u/6683103?v=4" title="silverchen" width="80" height="80"></a>
<a href="https://github.com/tombh"><img src="https://avatars2.githubusercontent.com/u/160835?v=4" title="tombh" width="80" height="80"></a>
<a href="https://github.com/zkwentz"><img src="https://avatars2.githubusercontent.com/u/4832?v=4" title="zkwentz" width="80" height="80"></a>
<a href="https://github.com/agraves"><img src="https://avatars3.githubusercontent.com/u/46964?v=4" title="agraves" width="80" height="80"></a>
<a href="https://github.com/swashcap"><img src="https://avatars1.githubusercontent.com/u/1858316?v=4" title="swashcap" width="80" height="80"></a>
<a href="https://github.com/DanielRuf"><img src="https://avatars1.githubusercontent.com/u/827205?v=4" title="DanielRuf" width="80" height="80"></a>
<a href="https://github.com/dciccale"><img src="https://avatars0.githubusercontent.com/u/539546?v=4" title="dciccale" width="80" height="80"></a>
<a href="https://github.com/ericclemmons"><img src="https://avatars0.githubusercontent.com/u/15182?v=4" title="ericclemmons" width="80" height="80"></a>
<a href="https://github.com/jamesgeorge007"><img src="https://avatars2.githubusercontent.com/u/25279263?v=4" title="jamesgeorge007" width="80" height="80"></a>
<a href="https://github.com/feinoujc"><img src="https://avatars2.githubusercontent.com/u/1733707?v=4" title="feinoujc" width="80" height="80"></a>
<a href="https://github.com/karlhorky"><img src="https://avatars2.githubusercontent.com/u/1935696?v=4" title="karlhorky" width="80" height="80"></a>
<a href="https://github.com/kennethlarsen"><img src="https://avatars3.githubusercontent.com/u/1408595?v=4" title="kennethlarsen" width="80" height="80"></a>
<a href="https://github.com/poteto"><img src="https://avatars0.githubusercontent.com/u/1390709?v=4" title="poteto" width="80" height="80"></a>
<a href="https://github.com/fastfrwrd"><img src="https://avatars3.githubusercontent.com/u/231133?v=4" title="fastfrwrd" width="80" height="80"></a>
<a href="https://github.com/rwd"><img src="https://avatars3.githubusercontent.com/u/218337?v=4" title="rwd" width="80" height="80"></a>
<a href="https://github.com/gribnoysup"><img src="https://avatars2.githubusercontent.com/u/5036933?v=4" title="gribnoysup" width="80" height="80"></a>
<a href="https://github.com/shriyash"><img src="https://avatars0.githubusercontent.com/u/4494915?v=4" title="shriyash" width="80" height="80"></a>
<a href="https://github.com/geigerzaehler"><img src="https://avatars2.githubusercontent.com/u/3919579?v=4" title="geigerzaehler" width="80" height="80"></a>
<a href="https://github.com/vikr01"><img src="https://avatars0.githubusercontent.com/u/28772991?v=4" title="vikr01" width="80" height="80"></a>
<a href="https://github.com/yasinuslu"><img src="https://avatars0.githubusercontent.com/u/1007479?v=4" title="yasinuslu" width="80" height="80"></a>

[//]: contributor-faces

## We're hiring! // 우리는 고용 중이다!

Join the Netflix Studio UI team to help us build projects like this! You can check out our open roles here on our [team page](https://jobs.netflix.com/teams/client-and-ui-engineering).
// Netflix Studio UI 팀에 참여하여 이와 같은 프로젝트를 구축하여라. 다음과 같은 team page를 통해 우리의 역할을 확인할 수 있다.

## License // 저작권

Copyright (c) 2018 Netflix, Inc.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at
// Apache 라이센스, 버전 2.0 (이하 "라이센스")에 따라 라이센스가 부여됩니다; 라이센스를 준수하지 않는 한이 파일을 사용할 수 없습니다.
라이센스 사본은 다음에서 구할 수 있습니다.

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
// 해당 법률에서 요구하거나 서면으로 동의하지 않는 한, 라이센스에 따라 배포된 소프트웨어는 명시적이든 묵시적이든 어떠한 종류의 보증이나 조건 없이 "있는 그대로" 배포됩니다.
라이센스에 따른 특정 언어 관리 권한 및 제한 사항에 대해서는 라이센스를 참조하십시오.
