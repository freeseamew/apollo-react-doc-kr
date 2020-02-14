---
description: Meteor 애플리케이션에서 Apollo를 사용하는 방법에 대한 세부 사항.
---

# Integrating with Meteor

Meteor 앱에서 Apollo를 사용하는 주요 방법은 두 가지입니다.

* meteor add swydo : ddp-apollo는 DDP 전체에서 Meteor 사용자 계정 및 구독을 지원하는 네트워크 링크를 제공합니다. 
* meteor add apollo는 HTTP를 통한 Meteor 사용자 계정을 아래 문서와 함께 지원합니다.



### Usage\(사용법\) <a id="usage"></a>

```text
meteor add apollo
meteor npm install graphql apollo-server-express apollo-boost
```

#### Client <a id="client"></a>

ApolloClient 인스턴스를 작성하십시오.

```javascript
import { Accounts } from 'meteor/accounts-base'
import ApolloClient from 'apollo-boost'

const client = new ApolloClient({
  uri: '/graphql',
  request: operation =>
    operation.setContext(() => ({
      headers: {
        authorization: Accounts._storedLoginToken()
      }
    }))
})
```

또는 apollo-boost 대신 apollo-client를 사용하는 경우 MeteorAccountsLink \(\)를 사용하십시오.

```javascript
import { ApolloClient } from 'apollo-client'
import { InMemoryCache } from 'apollo-cache-inmemory'
import { ApolloLink } from 'apollo-link'
import { HttpLink } from 'apollo-link-http'
import { MeteorAccountsLink } from 'meteor/apollo'

const client = new ApolloClient({
  link: ApolloLink.from([
    new MeteorAccountsLink(),
    new HttpLink({
      uri: '/graphql'
    })
  ]),
  cache: new InMemoryCache()
})
```

토큰이 저장된 헤더를 변경하려면 다음을 수행하십시오.

```javascript
MeteorAccountsLink({ headerName: 'meteor-login-token' })
```

\(기본설저 `authorization`.\)

#### Server <a id="server"></a>

Apollo Server를 설정하십시오 :

```javascript
import { ApolloServer, gql } from 'apollo-server-express'
import { WebApp } from 'meteor/webapp'
import { getUser } from 'meteor/apollo'

import typeDefs from './schema'
import resolvers from './resolvers'

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async ({ req }) => ({
    user: await getUser(req.headers.authorization)
  })
})

server.applyMiddleware({
  app: WebApp.connectHandlers,
  path: '/graphql'
})

WebApp.connectHandlers.use('/graphql', (req, res) => {
  if (req.method === 'GET') {
    res.end()
  }
})
```

이제 클라이언트가 로그인하면 \(즉, localStorage에 만료되지 않은 Meteor 로그인 토큰이있는 경우\) 해결 프로그램은 사용자 doc과 함께 context.user 속성을 갖게됩니다.

#### IDE <a id="ide"></a>

인증 된 GraphQL 요청을 수행하는 IDE 사용에는 두 가지 옵션이 있습니다.

1. 아폴로 devtools GraphiQL :

* 앱에 로그인 
* GraphiQL 섹션으로 Apollo devtools 열기

2. [GraphQL Playground](https://github.com/prismagraphql/graphql-playground): 

* BRECK CASK 설치와 함께 설치
* 앱에 로그인
* 브라우저 콘솔에서 localStorage.getItem \( 'Meteor.loginToken'\)을 입력하십시오.
* 반환 된 문자열을 복사
* 운동장에서 :

  상단에 http : // localhost : 3000 / graphql을 입력하십시오.

* HTTP HEADERS 아래에 { "authorization": "copied string"}을 입력하십시오.

#### Typings <a id="typings"></a>

Meteor 앱은 TypeScript를 사용한 정적 타이핑에 의존 할 수 있습니다. 그렇다면이 분위기 패키지에 앰비언트 TypeScript 정의를 사용하는 것이 좋습니다.

### Accounts\(인증\) <a id="accounts"></a>

위의 솔루션은 Meteor DDP 메시지를 사용하는 Accounts.createUser 및 Accounts.loginWith \*와 같은 Meteor의 클라이언트 측 계정 기능을 사용한다고 가정합니다.

앱에서 GraphQL 만 사용하려는 경우 nicolaslopezj : apollo-accounts를 사용할 수 있습니다. 이 패키지는 GraphQL의 Meteor Accounts 메소드를 사용하며 데이터베이스에 저장 한 계정과 호환되며 nicolaslopezj : apollo-accounts 및 Meteor의 DDP 계정을 동시에 사용할 수 있습니다.

쿼리에서 현재 사용자에게 의존하는 경우 현재 사용자 상태가 변경 될 때 상점을 지우고 싶을 것입니다. 이렇게하려면 Meteor.logout 콜백에서 client.resetStore \(\)를 사용하십시오.

```javascript
//`client` 변수는`ApolloClient` 인스턴스를 나타냅니다.
// 템플릿으로 가져옵니다.
// 또는 React의`withApollo` 덕분에 소품을 통해 전달되었습니다.

Meteor.logout(function() {
  return client.resetStore(); // make all active queries re-run when the log-out process completed
});
```

### SSR <a id="ssr"></a>

Meteor와 함께 React Server Side Rendering을 사용할 때 명심해야 할 두 가지 추가 구성이 있습니다.

1. isomorphic-fetch를 사용하여 서버 측 페치를 polyfill하십시오 \(Apollo Client의 네트워크 인터페이스에서 사용\).
2. WebApp.connectHandlers.use를 사용하여 Express 서버를 Meteor의 기존 서버에 연결하십시오.
3. res.send \(\) 및 res.end \(\)로 연결을 종료하지 마십시오. req.dynamicBody 및 req.dynamicHead를 대신 사용하고 next \(\)를 호출하십시오. 더 많은 정보

아이디어는 Meteor가 html을 최종적으로 렌더링하도록해야한다는 것입니다. html에 추가 본문 및 / 또는 헤드를 제공 할 수 있으며 Meteor가 추가 할 것입니다. 그렇지 않으면 CSS / JS 및 Meteor가 기본적으로 제공하는 기타 병합 된 html 컨텐츠 \(포함\) 응용 프로그램 기본 .js 파일\)이 없습니다.

다음은 apollo @ 2. \* \(구식\)를 사용한 전체 작업 예입니다.

```javascript
meteor add apollo webapp 
meteor npm install --save react react-dom apollo-client redux react-apollo react-router react-helmet express isomorphic-fetch
```



```javascript
import { Meteor } from 'meteor/meteor';
import { WebApp } from 'meteor/webapp';
import { meteorClientConfig, createMeteorNetworkInterface } from 'meteor/apollo';
import React from 'react';
import ReactDOM from 'react-dom/server';
import ApolloClient from 'apollo-client';
import { createStore, combineReducers, applyMiddleware, compose } from 'redux';
import { ApolloProvider, renderToStringWithData } from 'react-apollo';
import { match, RouterContext } from 'react-router';
import Express from 'express';
// #1 import isomorphic-fetch so the network interface can be created
import 'isomorphic-fetch';
import Helmet from 'react-helmet';

import routes from '../both/routes';
import rootReducer from '../../ui/reducers';
import Body from '../both/routes/body';

// 1# do not use new
const app = Express(); // eslint-disable-line new-cap

app.use((req, res, next) => {
  match({ routes, location: req.originalUrl }, (error, redirectLocation, renderProps) => {
    if (redirectLocation) {
      res.redirect(redirectLocation.pathname + redirectLocation.search);
    } else if (error) {
      console.error('ROUTER ERROR:', error); // eslint-disable-line no-console
      res.status(500);
    } else if (renderProps) {
      // createMeteorNetworkInterface를 사용하여 미리 구성된 네트워크 인터페이스를 얻습니다.
      // polyfilled`fetch '덕분에 # 1 네트워크 인터페이스를 서버 측에서 사용할 수 있습니다
      const networkInterface = createMeteorNetworkInterface({
        opts: {
          credentials: 'same-origin',
          headers: req.headers,
        },
        // 덕분에 쿠키에 저장된 현재 사용자 로그인 토큰 가능
        // meteorhacks : fast-render와 같은 타사 패키지
        loginToken: req.cookies['meteor-login-token'],
      });

      // meteorClientConfig를 사용하여 사전 구성된 Apollo Client 옵션 객체를 얻습니다.
      const client = new ApolloClient(meteorClientConfig({ networkInterface }));

      const store = createStore(
        combineReducers({
          ...rootReducer,
          apollo: client.reducer(),
        }),
        {}, // initial state
        compose(
          applyMiddleware(client.middleware()),
        ),
      );

      const component = (
        <ApolloProvider store={store} client={client}>
          <RouterContext {...renderProps} />
        </ApolloProvider>
      );

      renderToStringWithData(component).then((content) => {
        const initialState = client.store.getState()[client.reduxRootKey].data;
        // 추가하려는 본문 내용
        const body = <Body content={content} state={initialState} />;
        // # 3`req.dynamicBody`는 몸을 잡고 유성은 돌봐 줄 것이다
        // 실제로 최종 결과에 추가
        req.dynamicBody = ReactDOM.renderToStaticMarkup(body);
        const head = Helmet.rewind();
        // # 3`req.dynamicHead`이 경우`react-helmet`을 사용하여 seo 태그를 추가합니다
        req.dynamicHead = `  ${head.title.toString()}
  ${head.meta.toString()}
  ${head.link.toString()}
`;
        // # 3 중요한 것은 우리가 이것을 돌려주고 싶지 않다는 것입니다.
        next();
      });
    } else {
      console.log('not found'); // eslint-disable-line no-console
    }
  });
});
// #2 connect your express server with meteor's
WebApp.connectHandlers.use(Meteor.bindEnvironment(app));
```

### Importing `.graphql` files <a id="importing-graphql-files"></a>

GraphQL을 사용하는 쉬운 방법은 import 구문을 사용하여 .graphql 파일을 직접 가져 오는 것입니다.

```text
meteor add swydo:graphql
```

/imports/api/schema.js 파일 대신 이전과 동일한 내용으로 /imports/api/schema.graphql 파일을 작성하십시오.

```text
type Query {
  say: String
}
```

바로 얻을 수있는 이점 중 하나는 GitHub와 IDE가 강조하는 것입니다!

이제 스키마를 가져올 수 있습니다 :

```javascript
import typeDefs from '/imports/api/schema.graphql';
```

위의 예에서와 같이 typeDefs를 사용하십시오. 이전처럼 makeExecutableSchema에 직접 전달할 수 있습니다.

가져 오기 구문은 기본 스키마 외에 다른 .graphql 파일에서도 작동합니다. 따라서 query, mutation 및 subscription 파일을 graphql-tag로 수동으로 구문 분석하지 않고도 가져올 수 있습니다.

더 많은 혜택을 보려면 GrahpQL 빌드 플러그인 README를 참조하십시오.

### Blaze <a id="blaze"></a>

Apollo를 Blaze와 통합하려는 경우 swydo : blaze-apollo 패키지를 사용할 수 있습니다

```javascript
import { setup } from 'meteor/swydo:blaze-apollo';

const client = new ApolloClient(meteorClientConfig());

setup({ client });
```

템플릿에 반응 형 GraphQL 쿼리가 제공됩니다.

```javascript
Template.hello.helpers({
  hello() {
    return Template.instance().gqlQuery({
      query: HELLO_QUERY
    }).get();
  }
});
```

### Subscriptions <a id="subscriptions"></a>

이 섹션은 오래된`apollo @ 2.` API를 사용합니다. \*

필요한 경우 Meteor 앱과 함께 GraphQL 구독을 사용할 수도 있습니다. 다음 코드는 기본 GraphQL 외에도 구독의 모든 기능을 사용할 수있는 완전한 구성의 예를 제공합니다.

#### Client <a id="client-1"></a>

```javascript
import { ApolloClient } from 'apollo-client';
import { SubscriptionClient, addGraphQLSubscriptions } from 'subscriptions-transport-ws';
import { getMeteorLoginToken, createMeteorNetworkInterface } from 'meteor/apollo';

// "basic" Meteor network interface
const networkInterface = createMeteorNetworkInterface();

// create a websocket uri based on your app absolute url (ROOT_URL), ex: ws://localhost:3000
const websocketUri = Meteor.absoluteUrl('subscriptions').replace(/^http/, 'ws');

// create a websocket client
const wsClient = new SubscriptionClient(websocketUri, {
  reconnect: true,
  // pass some extra information to the subscription, like the current user:
  connectionParams: {
    // getMeteorLoginToken = get the Meteor current user login token from local storage
    meteorLoginToken: getMeteorLoginToken(),
  },
});

// enhance the interface with graphql subscriptions
const networkInterfaceWithSubscriptions = addGraphQLSubscriptions(networkInterface, wsClient);

// enjoy graphql subscriptions with Apollo Client
const client = new ApolloClient({ networkInterface: networkInterfaceWithSubscriptions });
```

#### Server <a id="server-1"></a>

리졸버와 GraphQL 구독에 동일한 컨텍스트가 사용됩니다. 이는 웹 소켓 전송의 인증이 기본적으로 구성됨을 의미합니다.

graphql-subscriptions의 PubSub는 프로덕션에 적합하지 않습니다. 프로덕션 앱에서 Subscription을 사용하려는 경우 SubscriptionManager를 Redis 구독 또는 MQTT 구독과 연결해야합니다.

```javascript
import { SubscriptionManager } from 'graphql-subscriptions';
import { SubscriptionServer } from 'subscriptions-transport-ws';
import { createApolloServer, addCurrentUserToContext } from 'meteor/apollo';

// your executable schema
const schema = ...

// any additional context you use for your resolvers, if any
const context = {};

// the pubsub mechanism of your choice, for instance:
// - PubSub from graphql-subscriptions (not recommended for production)
// - RedisPubSub from graphql-redis-subscriptions
// - MQTTPubSub from graphql-mqtt-subscriptions
const pubsub = new PubSub();

// subscriptions path which fits witht the one you connect to on the client
const subscriptionsPath = '/subscriptions';

// start a graphql server with Express handling a possible Meteor current user
createApolloServer({
  schema,
  context
});

// create the subscription manager thanks to the schema & the pubsub mechanism
const subscriptionManager = new SubscriptionManager({
  schema,
  pubsub,
});

// start up a subscription server
new SubscriptionServer(
  {
    subscriptionManager,
    // on connect subscription lifecycle event
    onConnect: async (connectionParams, webSocket) => {
      // if a meteor login token is passed to the connection params from the client,
      // add the current user to the subscription context
      const subscriptionContext = connectionParams.meteorLoginToken
        ? await addCurrentUserToContext(context, connectionParams.meteorLoginToken)
        : context;

      return subscriptionContext;
    },
  },
  {
    // bind the subscription server to Meteor WebApp
    server: WebApp.httpServer,
    path: subscriptionsPath,
  }
);
```

