---
description: Apollo 클라이언트의 네트워크 계층을 구성하는 방법
---

# Network layer\(Apollo Link\)

데이터를 읽고 업데이트하는 방법을 배웠으므로 데이터의 출처와 위치를 지정하는 방법을 아는 것이 도움이됩니다! Apollo는 Apollo Link라는 라이브러리를 사용하여 네트워크 계층을 관리하는 강력한 방법을 제공합니다.

### Apollo Link <a id="apollo-link"></a>

Apollo Client에는 플러그 가능한 네트워크 인터페이스 계층이있어 HTTP를 통해 쿼리를 전송하는 방법을 구성하거나 전체 네트워크 부분을 웹 소켓 전송, 조롱 된 서버 데이터 또는 상상할 수있는 다른 항목과 같은 완전히 사용자 정의 된 것으로 바꿀 수 있습니다.

#### Using a link <a id="using-a-link"></a>

Apollo Client와 함께 사용할 링크를 만들려면 npm에서 하나를 설치하고 가져 오거나 직접 만들 수 있습니다. 대부분의 설정에는 apollo-link-http를 사용하는 것이 좋습니다.

HttpLink를 사용하여 사용자 정의 엔드 포인트 URL로 새 클라이언트를 인스턴스화하는 방법은 다음과 같습니다.

```javascript
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';

const link = new HttpLink({ uri: 'https://example.com/graphql' });

const client = new ApolloClient({
  link,
  // other options like cache
});
```

가져올 추가 옵션을 전달해야하는 경우 :

```javascript
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';

const link = new HttpLink({
  uri: 'https://example.com/graphql',
  // Additional fetch options like `credentials` or `headers`
  credentials: 'same-origin',
});

const client = new ApolloClient({
  link,
  // other options like cache
});
```

#### Middleware <a id="middleware"></a>

Apollo Link는 귀하의 요청에 따라 미들웨어를 쉽게 사용할 수 있도록 설계되었습니다. 미들웨어는 링크를 통한 모든 요청 \(예 : 모든 쿼리에 인증 토큰 추가\)을 검사하고 수정하는 데 사용됩니다. 미들웨어를 추가하려면 새 링크를 작성하고 HttpLink와 결합하십시오.

다음 예제는 미들웨어 작성 방법을 보여줍니다. 두 예제 모두 클라이언트가 전송 한 요청의 HTTP 헤더에 인증 토큰을 추가하는 방법을 보여줍니다.

```javascript
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { ApolloLink, concat } from 'apollo-link';

const httpLink = new HttpLink({ uri: '/graphql' });

const authMiddleware = new ApolloLink((operation, forward) => {
  // add the authorization to the headers
  operation.setContext({
    headers: {
      authorization: localStorage.getItem('token') || null,
    }
  });

  return forward(operation);
})

const client = new ApolloClient({
  link: concat(authMiddleware, httpLink),
});
```

위의 예는 HttpLink와 결합 된 단일 미들웨어 사용을 보여줍니다. 토큰 \(예 : JWT\)이 있는지 확인하고 해당 토큰을 요청의 HTTP 헤더에 전달하므로 네트워크 인터페이스를 통해 수행 된 GraphQL과의 상호 작용을 인증 할 수 있습니다.

다음 예는 배열로 전달 된 여러 미들웨어 사용을 보여줍니다.

```javascript
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { ApolloLink, from } from 'apollo-link';

const httpLink = new HttpLink({ uri: '/graphql' });

const authMiddleware = new ApolloLink((operation, forward) => {
  // add the authorization to the headers
  operation.setContext(({ headers = {} }) => ({
    headers: {
      ...headers,
      authorization: localStorage.getItem('token') || null,
    }
  }));

  return forward(operation);
})

const otherMiddleware = new ApolloLink((operation, forward) => {
  // add the recent-activity custom header to the headers
  operation.setContext(({ headers = {} }) => ({
    headers: {
      ...headers,
      'recent-activity': localStorage.getItem('lastOnlineTime') || null,
    }
  }));

  return forward(operation);
})

const client = new ApolloClient({
  link: from([
    authMiddleware,
    otherMiddleware,
    httpLink
  ]),
});
```

위의 코드에서 헤더의 Authorization 값은 authMiddleware에 의한 localStorage의 토큰 값이며, 최근 활동 값은 otherMiddleware에 의해 localStorage에서 lastOnlineTime으로 다시 설정됩니다. 이 예제는 하나 이상의 미들웨어를 사용하여 체인 형태로 처리되는 요청을 여러 개 또는 개별적으로 수정하는 방법을 보여줍니다. 이 예제는 localStorage의 사용을 보여주지 않고 Apollo Link를 사용하여 둘 이상의 미들웨어를 사용하는 것을 보여주기위한 것입니다.

#### Afterware <a id="afterware"></a>

'Afterware'는 요청이 이루어진 후 즉 응답이 처리 될 때 애프터웨어가 실행된다는 점을 제외하고 미들웨어와 매우 유사합니다. 세션 중에 사용자가 로그 아웃되는 상황에 응답하기에 완벽합니다.

미들웨어와 마찬가지로 Apollo Link는 애프터웨어를 Apollo와 함께 쉽고 강력하게 사용할 수 있도록 설계되었습니다!

다음 예제는 afterware 기능을 구현하는 방법을 보여줍니다.

```javascript
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { onError } from 'apollo-link-error';

import { logout } from './logout';

const httpLink = new HttpLink({ uri: '/graphql' });

const logoutLink = onError(({ networkError }) => {
  if (networkError.statusCode === 401) logout();
})

const client = new ApolloClient({
  link: logoutLink.concat(httpLink),
});
```

위의 예는 응답에서 네트워크 오류를 처리하기 위해 아폴로 링크 오류를 사용하는 것을 보여줍니다. 응답 상태 코드가 401과 같은지 확인하고, 그렇다면 코드를 애플리케이션에서 로그 아웃합니다.

#### Query deduplication\(쿼리 중복 제거\) <a id="query-deduplication"></a>

쿼리 중복 제거는 유선으로 전송되는 쿼리 수를 줄이는 데 도움이됩니다. 기본적으로 설정되어 있지만 각 요청의 컨텍스트에 queryDeduplication : false를 전달하거나 Apollo Client 설정의 defaultOptions 키를 사용하여 해제 할 수 있습니다. 설정하면 쿼리가 네트워크 계층에 도달하기 전에 쿼리 중복 제거가 발생합니다.

많은 구성 요소가 동일한 데이터를 표시하지만 서버에서 해당 데이터를 여러 번 가져오고 싶지 않은 경우 쿼리 중복 제거가 유용 할 수 있습니다. 쿼리를 현재 비행중인 모든 쿼리와 비교하여 작동합니다. 동일한 쿼리가 현재 비행중인 경우 새 쿼리는 동일한 약속에 매핑되고 현재 진행중인 쿼리가 반환 될 때 해결됩니다.

### Other links <a id="other-links"></a>

Apollo Client의 네트워크 스택은 Apollo Link를 사용하여 쉽게 사용자 정의 할 수 있습니다! 오류를 기록하고 부작용을 보내거나 WebSocket 또는 HTTP를 통해 데이터를 보내는 등의 작업을 수행 할 수 있습니다. 아래에 몇 가지 예가 있지만 자세한 내용은 링크 문서를 확인하십시오!

#### GraphQL over WebSocket\(websocket을 통한 graphql\) <a id="graphql-over-websocket"></a>

네트워크 인터페이스의 또 다른 대안은 subscriptions-transport-ws를 사용하는 WebSocket을 통한 GraphQL입니다.

그런 다음 WebSocket을 전체 전송으로 생성하고 WebSocket \(Query, Mutation 및 Subscription\)을 통해 모든 GraphQL 작업을 전달하거나 하이브리드 네트워크 인터페이스를 사용하고 HTTP를 통한 Query and Mutation을 실행하고 WebSocket을 통한 Subscription 만 실행할 수 있습니다.

Apollo Link와 함께 WebSocket을 사용하는 방법에 대한 자세한 내용은 심층 가이드를 확인하십시오.

#### Query Batching <a id="query-batching"></a>

Apollo를 사용하면 특정 간격 내에서 여러 쿼리를 하나의 요청으로 자동 일괄 처리 할 수 있습니다. 즉, 탐색 모음, 사이드 바 및 내용과 같은 여러 구성 요소를 렌더링하고 각 구성 요소가 자체 GraphQL 쿼리를 수행하면 모두 한 번의 왕복으로 전송됩니다. 배치는 배치 된 쿼리를 지원하는 서버 \(예 : graphql-server\)에서만 작동합니다. 일괄 처리를 지원하지 않는 서버에 대한 일괄 처리 요청은 실패합니다. Apollo checkout과 함께 배치를 사용하는 방법에 대한 자세한 내용

#### Mutation batching <a id="mutation-batching"></a>

Apollo를 사용하면 여러 변이를 하나의 요청으로 일괄 처리하여 쿼리와 유사하게 하나의 요청에서 여러 변이를 자동으로 실행할 수 있습니다.

