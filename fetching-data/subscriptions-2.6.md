# Subscriptions \(2.6\)

GraphQL 사양은 쿼리를 사용하여 데이터를 가져오고 변이를 사용하여 데이터를 수정하는 것 외에도 subscription\(구독\)이라는 세 번째 작업 유형을 지원합니다.

GraphQL 구독은 서버에서 클라이언트로 실시간 메시지를 수신하도록 선택한 데이터를 서버에서 클라이언트로 푸시하는 방법입니다. 구독은 클라이언트에 전달할 필드 세트를 지정한다는 점에서 쿼리와 유사하지만 단일 응답을 즉시 리턴하는 대신 서버에서 특정 이벤트가 발생할 때마다 결과가 전송됩니다.

구독의 일반적인 유스 케이스는 클라이언트 측에 특정 이벤트 \(예 : 새 오브젝트 작성, 업데이트 된 필드 등\)를 알리는 것입니다.

### Overview\(개요\) <a id="overview"></a>

GraphQL 구독은 쿼리 및 변이와 같이 스키마에서 정의해야합니다.

```graphql
type Subscription {
  commentAdded(repoFullName: String!): Comment
}
```

클라이언트에서 구독 쿼리는 다른 종류의 작업과 같습니다.

```graphql
subscription onCommentAdded($repoFullName: String!){
  commentAdded(repoFullName: $repoFullName){
    id
    content
  }
}
```

클라이언트에게 전송 된 응답은 다음과 같습니다.

```text
{
  "data": {
    "commentAdded": {
      "id": "123",
      "content": "Hello!"
    }
  }
}
```

위 예제에서 서버는 GitHunt에 특정 리포지토리에 대한 설명이 추가 될 때마다 새로운 결과를 보내도록 작성되었습니다. 위의 코드는 스키마에서 GraphQL 구독 만 정의합니다. 클라이언트에서 구독 설정 및 서버에 대한 GraphQL 구독 설정을 읽고 앱에 구독을 추가하는 방법을 알아보십시오.

#### When to use subscriptions\(구독을 사용하 경우\) <a id="when-to-use-subscriptions"></a>

대부분의 경우 간헐적 폴링 또는 수동 리 페치는 실제로 클라이언트를 최신 상태로 유지하는 가장 좋은 방법입니다. 그렇다면 언제 구독이 최선의 선택입니까? 구독은 다음과 같은 경우에 특히 유용합니다.

1.초기 상태는 크지 만 증분 변경 세트는 작습니다. 시작 상태는 쿼리로 가져 와서 구독을 통해 업데이트 할 수 있습니다.

2.특정 이벤트의 경우 \(예 : 사용자가 몇 초 만에 새 메시지를받을 것으로 예상되는 채팅 응용 프로그램의 경우\) 지연 시간이 짧은 업데이트에 관심이 있습니다.

향후 버전의 Apollo 또는 GraphQL에는 실시간 쿼리에 대한 지원이 포함될 수 있습니다.이 기능은 대기 시간을 대체하는 지연 시간이 적은 방법 일 수 있지만 현재 GraphQL의 일반적인 실시간 쿼리는 일부 실험적인 설정 외부에서는 아직 가능하지 않습니다.

### Client setup <a id="client-setup"></a>

오늘날 GraphQL 구독에 가장 많이 사용되는 전송 방식은 subscriptions-transport-ws입니다. 이 패키지는 Apollo 커뮤니티에서 관리하지만 모든 클라이언트 또는 서버 GraphQL 구현과 함께 사용할 수 있습니다. 이 기사에서는 클라이언트에서 설정하는 방법을 설명하지만 서버 구현도 필요합니다. Graphcool 또는 Scaphold와 같은 서비스로 GraphQL 백엔드를 사용하는 경우 JavaScript 서버에서 구독을 사용하는 방법에 대해 읽거나 즉시 사용 가능한 구독을 즐길 수 있습니다.

이 전송에 대한 지원을 Apollo Client에 추가하는 방법을 살펴 보겠습니다.

먼저 npm에서 WebSocket Apollo Link \(apollo-link-ws\)를 설치하십시오.

```text
npm install --save apollo-link-ws subscriptions-transport-ws

```

그런 다음 GraphQL 구독 전송 링크를 초기화하십시오.

```javascript
import { WebSocketLink } from 'apollo-link-ws';

const wsLink = new WebSocketLink({
  uri: `ws://localhost:5000/`,
  options: {
    reconnect: true
  }
});
```

```javascript
import { split } from 'apollo-link';
import { HttpLink } from 'apollo-link-http';
import { WebSocketLink } from 'apollo-link-ws';
import { getMainDefinition } from 'apollo-utilities';

// Create an http link:
const httpLink = new HttpLink({
  uri: 'http://localhost:3000/graphql'
});

// Create a WebSocket link:
const wsLink = new WebSocketLink({
  uri: `ws://localhost:5000/`,
  options: {
    reconnect: true
  }
});

// using the ability to split links, you can send data to each link
// depending on what kind of operation is being sent
const link = split(
  // split based on operation type
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  httpLink,
);
```

이제 쿼리와 돌연변이가 정상적으로 HTTP를 통과하지만 구독은 웹 소켓 전송을 통해 수행됩니다.

### useSubscription Hook <a id="usesubscription-hook"></a>

라이브 데이터를 UI로 가져 오는 가장 쉬운 방법은 Apollo Client의 useSubscription Hook를 사용하는 것입니다. 이를 통해 컴포넌트의 렌더링 기능 내에서 직접 서비스의 데이터 스트림을 렌더링 할 수 있습니다! 주목할 점은 구독은 리스너 일 뿐이며 처음 연결할 때 데이터를 요청하지 않고 새 데이터를 얻기 위해 연결 만 엽니다. 라이브 데이터를 UI에 바인딩하는 것은 다음과 같이 쉽습니다.

```javascript
const COMMENTS_SUBSCRIPTION = gql`
  subscription onCommentAdded($repoFullName: String!) {
    commentAdded(repoFullName: $repoFullName) {
      id
      content
    }
  }
`;

function DontReadTheComments({ repoFullName }) {
  const { data: { commentAdded }, loading } = useSubscription(
    COMMENTS_SUBSCRIPTION,
    { variables: { repoFullName } }
  );
  return <h4>New comment: {!loading && commentAdded.content}</h4>;
}
```

### useSubscription API overview <a id="usesubscription-api-overview"></a>

useSubscription이 허용하는 모든 옵션과 결과 속성에 대한 개요를 찾고 있다면 더 이상 보지 마십시오!

> 참고 : React Apollo의 서브 스크립션 렌더링 소품 구성 요소를 사용하는 경우 아래 나열된 옵션 / 결과 세부 사항이 여전히 유효합니다 \(옵션은 구성 요소 소품이며 결과는 렌더링 소품 기능으로 전달됨\). 유일한 차이점은 구독 소품 \(gql에 의해 AST로 구문 분석 된 GraphQL 구독 문서를 보유 함\)도 필요하다는 것입니다.

#### Options <a id="options"></a>

useSubscription 후크는 다음 옵션을 허용합니다.

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `subscription` | DocumentNode | graphQL 구독 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. 서브 스크립 션을 첫 번째 매개 변수로 후크에 전달할 수 있으므로 useSubscription 후크의 경우 선택 사항입니다. 구독 구성 요소에 필요합니다. |
| `variables` | { \[key: string\]: any } | 구독이 실행해야하는 모든 변수를 포함하는 객체 |
| `shouldResubscribe` | boolean | 구독을 구독 취소하고 다시 구독해야하는지 결정 |
| `onSubscriptionData` | \(options: OnSubscriptionDataOptions&lt;TData&gt;\) =&gt; any | useSubscription Hook / Subscription 구성 요소가 데이터를 수신 할 때마다 트리거되는 콜백 함수의 등록을 허용합니다. 콜백 옵션 객체 매개 변수는 클라이언트의 현재 Apollo 클라이언트 인스턴스와 subscriptionData의 수신 된 구독 데이터로 구성됩니다. |
| `fetchPolicy` | FetchPolicy | 구성 요소가 Apollo 캐시와 상호 작용하는 방법 기본값은 "캐시 우선"입니다. |
| `client` | ApolloClient | ApolloClient 인스턴스 기본적으로 useSubscription / Subscription은 컨텍스트를 통해 전달 된 클라이언트를 사용하지만 다른 클라이언트를 전달할 수 있습니다. |

#### Result <a id="result"></a>

useSubscription Hook가 호출 된 후 다음 속성을 가진 결과 객체를 반환합니다.

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `data` | TData | GraphQL 구독 결과가 포함 된 객체입니다. 빈 개체가 기본값입니다. |
| `loading` | boolean | 초기 데이터가 반환되었는지 여부를 나타내는 부울 |
| `error` | ApolloError | graphQLErrors 및 networkError 속성이있는 런타임 오류 |

### subscribeToMore <a id="subscribetomore"></a>

GraphQL subscripthon 을 사용하면 서버에서 푸시 할 때 클라이언트에게 경고가 표시되므로 응용 프로그램에 가장 적합한 패턴을 선택해야합니다.

* 알림으로 사용하고 사용자에게 알리거나 데이터를 다시 가져 오는 등 원하는 로직을 실행할 수 있습니다.
* 알림과 함께 전송 된 데이터를 사용하여 상점에 직접 병합하십시오 \(기존 쿼리에 자동으로 통지 됨\)

subscribeToMore를 사용하면 후자를 쉽게 수행 할 수 있습니다.

subscribeToMore는 Apollo Client의 React 통합을 사용하여 모든 쿼리 결과에서 사용할 수있는 기능입니다. 구독이 반환 될 때마다 한 번이 아니라 업데이트 함수가 호출된다는 점을 제외하면 fetchMore와 동일하게 작동합니다.

다음은 일반적인 쿼리입니다.

```javascript
const COMMENT_QUERY = gql`
  query Comment($repoName: String!) {
    entry(repoFullName: $repoName) {
      comments {
        id
        content
      }
    }
  }
`;

function CommentsPageWithData({ params }) {
  const result = useQuery(
    COMMENT_QUERY,
    { variables: { repoName: `${params.org}/${params.repoName}` } }
  );
  return <CommentsPage {...result} />;
}
```

이제 구독을 추가하겠습니다.

subscribeToMore를 사용하여 구독 할 updateToNewComments라는 함수를 추가하고 updateQuery를 사용하여 새 데이터로 쿼리 저장소를 업데이트하십시오.

updateQuery 콜백은 초기 쿼리 데이터와 동일한 모양의 객체를 반환해야합니다. 그렇지 않으면 새 데이터가 병합되지 않습니다. 여기에 새로운 의견이 항목의 의견 목록에 표시됩니다.

```javascript
const COMMENT_QUERY = gql`
  query Comment($repoName: String!) {
    entry(repoFullName: $repoName) {
      comments {
        id
        content
      }
    }
  }
`;

const COMMENTS_SUBSCRIPTION = gql`
  subscription onCommentAdded($repoName: String!) {
    commentAdded(repoName: $repoName) {
      id
      content
    }
  }
`;

function CommentsPageWithData({ params }) {
  const { subscribeToMore, ...result } = useQuery(
    COMMENT_QUERY,
    { variables: { repoName: `${params.org}/${params.repoName}` } }
  );

  return (
    <CommentsPage
      {...result}
      subscribeToNewComments={() =>
        subscribeToMore({
          document: COMMENTS_SUBSCRIPTION,
          variables: { repoName: params.repoName },
          updateQuery: (prev, { subscriptionData }) => {
            if (!subscriptionData.data) return prev;
            const newFeedItem = subscriptionData.data.commentAdded;

            return Object.assign({}, prev, {
              entry: {
                comments: [newFeedItem, ...prev.entry.comments]
              }
            });
          }
        })
      }
    />
  );
}
```

subscription 변수로 subscribeToNewComments 함수를 호출하여 실제 구독을 시작하십시오.

```javascript
export class CommentsPage extends Component {
  componentDidMount() {
    this.props.subscribeToNewComments();
  }
}
```

### Authentication over WebSocket\(websocket을 이용한 인증\) <a id="authentication-over-websocket"></a>

많은 경우 가입 결과를 수신하기 전에 클라이언트를 인증해야합니다. 이를 위해 SubscriptionClient 생성자는 connectionParams 필드를 허용합니다.이 필드는 구독을 설정하기 전에 서버가 연결을 확인하는 데 사용할 수있는 사용자 정의 개체를 전달합니다.

```javascript
import { WebSocketLink } from 'apollo-link-ws';

const wsLink = new WebSocketLink({
  uri: `ws://localhost:5000/`,
  options: {
    reconnect: true,
    connectionParams: {
        authToken: user.authToken,
    },
});
```

connectionParams를 인증뿐만 아니라 필요한 다른 용도로 사용할 수 있으며 SubscriptionsServer를 통해 서버 쪽에서 페이로드를 확인할 수 있습니다.

