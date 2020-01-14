# Server-side rendering

Apollo는 사용자에게 불필요한 지연을 피하면서 애플리케이션을 빠르게로드 할 수있는 두 가지 기술을 제공합니다.

* 초기 쿼리 세트가 서버 왕복없이 즉시 데이터를 반환 할 수있는 Store rehydration.
* 서버 측 렌더링-클라이언트로 보내기 전에 서버에서 초기 HTML보기를 렌더링합니다.

이러한 기술 중 하나 또는 둘 다를 사용하여 더 나은 사용자 경험을 제공 할 수 있습니다.

### Store rehydration\(재수화 저장\) <a id="store-rehydration"></a>

클라이언트에서 UI를 렌더링하기 전에 서버에서 일부 쿼리를 수행 할 수있는 응용 프로그램의 경우 Apollo는 초기 데이터 상태를 설정할 수 있습니다. 데이터가 직렬화되고 초기 HTML 페이로드에 포함될 때 데이터가 "탈수"되기 때문에이를 재수 화라고도합니다.

예를 들어 일반적인 접근 방식은 다음과 같은 스크립트 태그를 포함하는 것입니다.

```text
<script>
  window.__APOLLO_STATE__ = JSON.stringify(client.extract());
</script>
```

그런 다음 서버에서 전달 된 초기 상태를 사용하여 클라이언트를 재수화할 수 있습니다.

```text
const client = new ApolloClient({
  cache: new InMemoryCache().restore(window.__APOLLO_STATE__),
  link,
});
```

Node와 React Apollo의 서버 렌더링 기능을 사용하여 HTML과 Apollo 저장소의 상태를 모두 생성하는 방법을 아래에서 살펴 보겠습니다. 그러나 다른 방법으로 HTML을 렌더링하는 경우 수동으로 상태를 생성해야합니다.

그런 다음 클라이언트가 첫 번째 쿼리 집합을 실행하면 데이터가 이미 저장소에 있기 때문에 데이터가 즉시 반환됩니다!

일부 초기 쿼리에서 fetchPolicy : network-only 또는 fetchPolicy : cache-and-network를 사용하는 경우 초기화 중에 ssrForceFetchDelay 옵션을 전달하여 강제 페치를 건너 뛸 수 있으므로 캐시를 사용하여 해당 쿼리를 실행할 수도 있습니다.

```text
const client = new ApolloClient({
  cache: new InMemoryCache().restore(window.__APOLLO_STATE__),
  link,
  ssrForceFetchDelay: 100,
});
```

### Server-side rendering <a id="server-side-rendering"></a>

React Apollo에 내장 된 렌더링 기능을 사용하여 노드 서버에서 전체 React 기반 Apollo 애플리케이션을 렌더링 할 수 있습니다. 이러한 함수는 구성 요소 트리를 렌더링하는 데 필요한 모든 쿼리를 가져 오는 작업을 처리합니다. 일반적으로 Express와 같은 HTTP 서버 내에서 이러한 기능을 사용합니다.

이를 지원하기 위해 클라이언트 쿼리를 변경할 필요가 없으므로 Apollo 기반 React UI는 기본적으로 SSR을 지원해야합니다.

#### Server initialization\(서버 초기화\) <a id="server-initialization"></a>

서버에서 응용 프로그램을 렌더링하려면 Express와 같은 서버 및 React-Router와 같은 서버 가능 라우터를 사용하여 HTTP 요청을 처리 한 다음 응용 프로그램을 문자열로 렌더링하여 응답을 다시 전달해야합니다. .

다음 섹션에서 구성 요소 트리를 가져 와서 문자열로 바꾸는 방법을 살펴볼 것입니다. 그러나 서버에서 Apollo Client 인스턴스를 구성하여 모든 것이 잘 작동하도록하는 방법에 대해 약간주의를 기울여야합니다.

1.서버에서 Apollo Client 인스턴스를 생성 할 때 API 서버에 올바르게 연결되도록 네트워크 인터페이스를 설정해야합니다. 클라이언트에서 상대 URL을 사용하는 경우 서버에 절대 URL을 사용해야 할 것이기 때문에 클라이언트에서 수행하는 방법과 다르게 보일 수 있습니다.

2.각 쿼리 결과를 한 번만 가져 오기를 원하므로 ssrMode : true 옵션을 Apollo Client 생성자에 전달하여 반복적 인 반입을 피하십시오.

3.여러 요청에 동일한 클라이언트를 재사용하지 않고 각 요청에 대해 새 클라이언트 또는 스토어 인스턴스를 작성해야합니다. 그렇지 않으면 UI에 오래된 데이터가 표시되고 인증에 문제가 발생합니다.

이 모든 것을 합치면 다음과 같은 초기화 코드가 생깁니다.

```text
// This example uses React Router v4, although it should work
// equally well with other routers that support SSR

import { ApolloProvider } from '@apollo/react-common';
import { ApolloClient } from 'apollo-client';
import { createHttpLink } from 'apollo-link-http';
import Express from 'express';
import { StaticRouter } from 'react-router';
import { InMemoryCache } from "apollo-cache-inmemory";

import Layout from './routes/Layout';

// Note you don't have to use any particular http server, but
// we're using Express in this example
const app = new Express();
app.use((req, res) => {

  const client = new ApolloClient({
    ssrMode: true,
    // Remember that this is the interface the SSR server will use to connect to the
    // API server, so we need to ensure it isn't firewalled, etc
    link: createHttpLink({
      uri: 'http://localhost:3010',
      credentials: 'same-origin',
      headers: {
        cookie: req.header('Cookie'),
      },
    }),
    cache: new InMemoryCache(),
  });

  const context = {};

  // The client-side App will instead use <BrowserRouter>
  const App = (
    <ApolloProvider client={client}>
      <StaticRouter location={req.url} context={context}>
        <Layout />
      </StaticRouter>
    </ApolloProvider>
  );

  // rendering code (see below)
});

app.listen(basePort, () => console.log( // eslint-disable-line no-console
  `app Server is now running on http://localhost:${basePort}`
));
```

```text
// ./routes/Layout.js
import { Route, Switch } from 'react-router';
import { Link } from 'react-router-dom';
import React from 'react';

// A Routes file is a good shared entry-point between client and server
import routes from './routes';

const Layout = () =>
  <div>
    <nav>
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/another">Another page</Link>
        </li>
      </ul>
    </nav>

    {/* New <Switch> behavior introduced in React Router v4
       https://reacttraining.com/react-router/web/api/Switch */}
    <Switch>
      {routes.map(route => <Route key={route.name} {...route} />)}
    </Switch>
  </div>;

export default Layout;
```

```text
// ./routes/index.js
import MainPage from './MainPage';
import AnotherPage from './AnotherPage';

const routes = [
  {
    path: '/',
    name: 'home',
    exact: true,
    component: MainPage,
  },
  {
    path: '/another',
    name: 'another',
    component: AnotherPage,
  },
];

export default routes;
```

다음으로 렌더링 코드가 실제로 무엇을하는지 살펴 보겠습니다.

#### Using `getDataFromTree` <a id="using-getdatafromtree"></a>

getDataFromTree 함수는 React 트리를 가져 와서 렌더링하는 데 필요한 쿼리를 결정한 다음 모두 가져옵니다. 중첩 된 쿼리가 있으면 전체 트리에서 재귀 적 으로이 작업을 수행합니다. Apollo 클라이언트 저장소에서 데이터가 준비되면 해결되는 약속을 반환합니다.

약속이 해결되는 시점에서 Apollo Client 스토어가 완전히 초기화됩니다. 즉, 모든 쿼리가 프리 페치되므로 앱이 즉시 렌더링되고 문자열 화 된 결과를 응답으로 반환 할 수 있습니다.

```text
import { getDataFromTree } from "@apollo/react-ssr";

const client = new ApolloClient(....);

// during request (see above)
getDataFromTree(App).then(() => {
  // We are ready to render for real
  const content = ReactDOM.renderToString(App);
  const initialState = client.extract();

  const html = <Html content={content} state={initialState} />;

  res.status(200);
  res.send(`<!doctype html>\n${ReactDOM.renderToStaticMarkup(html)}`);
  res.end();
});
```

이 경우의 마크 업은 다음과 같습니다.

```text
function Html({ content, state }) {
  return (
    <html>
      <body>
        <div id="root" dangerouslySetInnerHTML={{ __html: content }} />
        <script dangerouslySetInnerHTML={{
          __html: `window.__APOLLO_STATE__=${JSON.stringify(state).replace(/</g, '\\u003c')};`,
        }} />
      </body>
    </html>
  );
}
```

#### Avoiding the network for local queries`(로컬 쿼리를 위한 네트워크 회피)` <a id="avoiding-the-network-for-local-queries"></a>

GraphQL 엔드 포인트가 렌더링중인 서버와 동일한 서버에있는 경우 SSR 조회시 네트워크 사용을 피할 수 있습니다. 특히 로컬 호스트가 프로덕션 환경 \(예 : Heroku\)에서 방화벽으로 설정된 경우 이러한 쿼리에 대한 네트워크 요청은 작동하지 않습니다.

이 문제에 대한 한 가지 해결책은 Apollo Link를 사용하여 네트워크 요청 대신 로컬 graphql 스키마를 사용하여 데이터를 가져 오는 것입니다. 이를 위해 서버에서 Apollo Client를 생성 할 때 추가 네트워크 요청없이 스키마와 컨텍스트를 사용하여 즉시 쿼리를 실행하는 createHttpLink를 사용하는 대신 SchemaLink를 사용할 수 있습니다.

```text
import { ApolloClient } from 'apollo-client'
import { SchemaLink } from 'apollo-link-schema';

// ...

const client = new ApolloClient({
  ssrMode: true,
  // Instead of "createHttpLink" use SchemaLink here
  link: new SchemaLink({ schema }),
  cache: new InMemoryCache(),
});
```

#### Skipping queries for SSR`(ssr에 대한 쿼리 스킵)` <a id="skipping-queries-for-ssr"></a>

SSR 중에 의도적으로 쿼리를 건너 뛰려면 쿼리 옵션에서 ssr : false를 전달할 수 있습니다. 일반적으로 이는 구성 요소가 서버에서로드 상태로 렌더링됨을 의미합니다. 예를 들면 다음과 같습니다.

```text
function withClientOnlyUser() {
  useQuery(GET_USER_WITH_ID, { ssr: false });
  return <span>I won't be run on the server</span>;
}
```

#### Using `renderToStringWithData` <a id="using-rendertostringwithdata"></a>

renderToStringWithData 함수는 위의 내용을 단순화하고 렌더링해야하는 콘텐츠 문자열을 반환합니다. 따라서 단계 수를 약간 줄입니다.

```text
// server application code (integrated usage)
import { renderToStringWithData } from "@apollo/react-ssr";

const client = new ApolloClient(...);

// during request
renderToStringWithData(App).then((content) => {
  const initialState = client.extract();
  const html = <Html content={content} state={initialState} />;

  res.status(200);
  res.send(`<!doctype html>\n${ReactDOM.renderToStaticMarkup(html)}`);
  res.end();
});
```

