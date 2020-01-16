# @apollo/react-hooks

### Installation <a id="installation"></a>

```text
npm install @apollo/react-hooks
```

### `useQuery` <a id="usequery"></a>

#### Example <a id="example"></a>

```text
import { useQuery } from '@apollo/react-hooks';
import gql from 'graphql-tag';

const GET_GREETING = gql`
  query getGreeting($language: String!) {
    greeting(language: $language) {
      message
    }
  }
`;

function Hello() {
  const { loading, error, data } = useQuery(GET_GREETING, {
    variables: { language: 'english' },
  });
  if (loading) return <p>Loading ...</p>;
  return <h1>Hello {data.greeting.message}!</h1>;
}
```

> useQuery에 대한보다 자세한 개요는 조회 섹션을 참조하십시오.

#### Function Signature`(서명)` <a id="function-signature"></a>

```text
function useQuery<TData = any, TVariables = OperationVariables>(
  query: DocumentNode,
  options?: QueryHookOptions<TData, TVariables>,
): QueryResult<TData, TVariables> {}
```

#### Params <a id="params"></a>

**query**

| PARAM | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `query` | DocumentNode | graphql-tag에 의해 AST로 구문 분석 된 GraphQL 쿼리 문서 |

**options**

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `query` | DocumentNode | graphQL 쿼리 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. 조회는 첫 번째 매개 변수로 후크에 전달 될 수 있으므로 useQuery 후크의 경우 선택 사항입니다. 쿼리 구성 요소에 필요합니다. |
| `variables` | { \[key: string\]: any } | 쿼리 실행에 필요한 모든 변수를 포함하는 객체 |
| `pollInterval` | number | 구성 요소가 데이터를 폴링 할 간격을 ms 단위로 지정합니다. 기본값은 0입니다 \(폴링 없음\). |
| `notifyOnNetworkStatusChange` | boolean | 네트워크 상태 또는 네트워크 오류에 대한 업데이트가 구성 요소를 다시 렌더링해야하는지 여부 기본값은 false입니다. |
| `fetchPolicy` | FetchPolicy | 구성 요소가 Apollo 캐시와 상호 작용하는 방법 기본값은 "캐시 우선"입니다. |
| `errorPolicy` | ErrorPolicy | 구성 요소가 네트워크 및 GraphQL 오류를 처리하는 방법 기본값은 "없음"으로, GraphQL 오류를 런타임 오류로 처리합니다. |
| `ssr` | boolean | 서버 측 렌더링 중에 쿼리를 건너 뛰려면 false로 전달하십시오. |
| `displayName` | string | React DevTools에 표시 할 컴포넌트의 이름입니다. 기본값은 'Query'입니다. |
| `skip` | boolean | 건너 뛰기가 true이면 쿼리가 완전히 건너 뜁니다. useLazyQuery와 함께 사용할 수 없습니다. |
| `onCompleted` | \(data: TData \| {}\) =&gt; void | 쿼리가 성공적으로 완료되면 콜백이 실행됩니다. |
| `onError` | \(error: ApolloError\) =&gt; void | 오류 발생시 실행 된 콜백 |
| `context` | Record&lt;string, any&gt; | 구성 요소와 네트워크 인터페이스 \(Apollo Link\) 간의 공유 컨텍스트 |
| `partialRefetch` | boolean | 참인 경우 쿼리 결과가 부분적인 것으로 표시되고 캐시 누락으로 인해 반환 된 데이터가 Apollo Client QueryManager에 의해 빈 오브젝트로 재설정되는 경우 쿼리 리 페치를 수행합니다. 이전 버전과의 호환성을 위해 기본값은 false이지만 대부분의 사용 사례에서는 true로 변경해야합니다. |
| `client` | ApolloClient | ApolloClient 인스턴스 기본적으로 useQuery / Query는 컨텍스트를 통해 전달 된 클라이언트를 사용하지만 다른 클라이언트를 전달할 수 있습니다. |
| `returnPartialData` | boolean | 캐시에서 완전히 만족하지 않는 쿼리에 대해 캐시에서 부분 결과를 수신하도록 선택하십시오. 기본적으로 false입니다. |

#### Result <a id="result"></a>

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `data` | TData | GraphQL 쿼리 결과가 포함 된 객체입니다. 기본값은 undefined입니다. |
| `loading` | boolean | 요청이 진행 중인지 여부를 나타내는 부울 |
| `error` | ApolloError | graphQLErrors 및 networkError 속성이있는 런타임 오류 |
| `variables` | { \[key: string\]: any } | 쿼리가 호출 된 변수를 포함하는 객체 |
| `networkStatus` | NetworkStatus | 네트워크 요청의 자세한 상태에 해당하는 1-8의 숫자입니다. 리 페치 및 폴링 상태에 대한 정보를 포함합니다. notifyOnNetworkStatusChange prop과 함께 사용됩니다. |
| `refetch` | \(variables?: TVariables\) =&gt; Promise&lt;ApolloQueryResult&gt; | 쿼리를 다시 가져오고 선택적으로 새 변수를 전달할 수있는 기능 |
| `fetchMore` | \({ query?: DocumentNode, variables?: TVariables, updateQuery: Function}\) =&gt; Promise&lt;ApolloQueryResult&gt; | 쿼리에 페이지 매김을 가능하게하는 기능 |
| `startPolling` | \(interval: number\) =&gt; void | 이 함수는 간격을 ms 단위로 설정하고 지정된 간격이 경과 할 때마다 쿼리를 가져옵니다. |
| `stopPolling` | \(\) =&gt; void | 이 함수는 쿼리 폴링을 중지합니다. |
| `subscribeToMore` | \(options: { document: DocumentNode, variables?: TVariables, updateQuery?: Function, onError?: Function}\) =&gt; \(\) =&gt; void | 구독을 설정하는 기능입니다. subscribeToMore는 구독을 취소하는 데 사용할 수있는 함수를 반환합니다. |
| `updateQuery` | \(previousResult: TData, options: { variables: TVariables }\) =&gt; TData | 페치, 돌연변이 또는 구독 컨텍스트 외부의 캐시에서 쿼리 결과를 업데이트 할 수있는 기능 |
| `client` | ApolloClient | ApolloClient 인스턴스 쿼리를 수동으로 발생 시키거나 캐시에 데이터를 쓰는 데 유용합니다 |
| `called` | boolean | useLazyQuery가 사용하는 쿼리 함수가 호출되었는지 여부를 나타내는 부울입니다 \(useQuery / Query에 설정되지 않음\). |

### `useLazyQuery` <a id="uselazyquery"></a>

#### Example <a id="example-1"></a>

```text
import { useLazyQuery } from "@apollo/react-hooks";
import gql from "graphql-tag";

const GET_GREETING = gql`
  query getGreeting($language: String!) {
    greeting(language: $language) {
      message
    }
  }
`;

function Hello() {
  const [loadGreeting, { called, loading, data }] = useLazyQuery(
    GET_GREETING,
    { variables: { language: "english" } }
  );
  if (called && loading) return <p>Loading ...</p>
  if (!called) {
    return <button onClick={() => loadGreeting()}>Load greeting</button>
  }
  return <h1>Hello {data.greeting.message}!</h1>;
}
```

> useLazyQuery에 대한보다 자세한 개요는 조회 섹션을 참조하십시오.

#### Function Signature`(서명)` <a id="function-signature-1"></a>

```text
function useLazyQuery<TData = any, TVariables = OperationVariables>(
  query: DocumentNode,
  options?: LazyQueryHookOptions<TData, TVariables>,
): [
  (options?: QueryLazyOptions<TVariables>) => void,
  QueryResult<TData, TVariables>
] {}
```

#### Params <a id="params-1"></a>

**query**

| PARAM | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `query` | DocumentNode | graphQL 쿼리 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. |

**options**

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `query` | DocumentNode | graphQL 쿼리 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. 조회는 첫 번째 매개 변수로 후크에 전달 될 수 있으므로 useQuery 후크의 경우 선택 사항입니다. 쿼리 구성 요소에 필요합니다. |
| `variables` | { \[key: string\]: any } | 쿼리 실행에 필요한 모든 변수를 포함하는 객체 |
| `pollInterval` | number | 구성 요소가 데이터를 폴링 할 간격을 ms 단위로 지정합니다. 기본값은 0입니다 \(폴링 없음\) |
| `notifyOnNetworkStatusChange` | boolean | 네트워크 상태 또는 네트워크 오류에 대한 업데이트가 구성 요소를 다시 렌더링해야하는지 여부 기본값은 false입니다 |
| `fetchPolicy` | FetchPolicy | 구성 요소가 Apollo 캐시와 상호 작용하는 방법 기본값은 "캐시 우선"입니다. |
| `errorPolicy` | ErrorPolicy | 구성 요소가 네트워크 및 GraphQL 오류를 처리하는 방법 기본값은 "없음"으로, GraphQL 오류를 런타임 오류로 처리합니다. |
| `ssr` | boolean | 서버 측 렌더링 중에 쿼리를 건너 뛰려면 false로 전달하십시오. |
| `displayName` | string | React DevTools에 표시 할 컴포넌트의 이름입니다. 기본값은 'Query'입니다. |
| `skip` | boolean | 건너 뛰기가 true이면 쿼리가 완전히 건너 뜁니다. useLazyQuery와 함께 사용할 수 없습니다. |
| `onCompleted` | \(data: TData \| {}\) =&gt; void | 쿼리가 성공적으로 완료되면 콜백이 실행됩니다. |
| `onError` | \(error: ApolloError\) =&gt; void | 오류 발생시 실행 된 콜백 |
| `context` | Record&lt;string, any&gt; | 구성 요소와 네트워크 인터페이스 \(Apollo Link\) 간의 공유 컨텍스트 |
| `partialRefetch` | boolean | 참인 경우 쿼리 결과가 부분적인 것으로 표시되고 캐시 누락으로 인해 반환 된 데이터가 Apollo Client QueryManager에 의해 빈 오브젝트로 재설정되는 경우 쿼리 리 페치를 수행합니다. 이전 버전과의 호환성을 위해 기본값은 false이지만 대부분의 사용 사례에서는 true로 변경해야합니다. |
| `client` | ApolloClient | ApolloClient 인스턴스 기본적으로 useQuery / Query는 컨텍스트를 통해 전달 된 클라이언트를 사용하지만 다른 클라이언트를 전달할 수 있습니다. |
| `returnPartialData` | boolean | 캐시에서 완전히 만족하지 않는 쿼리에 대해 캐시에서 부분 결과를 수신하도록 선택하십시오. 기본적으로 false입니다. |

#### Result <a id="result-1"></a>

**Execute function\(실행 기능\)**

| PARAM | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| Execute function | options?: QueryLazyOptions&lt;TVariables&gt;\) =&gt; void | 일시 중단 된 쿼리를 실행하기 위해 트리거 할 수있는 기능입니다. 호출 된 후 useLazyQuery는 useQuery와 동일하게 작동합니다. |

**Result object**

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `data` | TData | GraphQL 쿼리 결과가 포함 된 객체입니다. 기본값은 undefined입니다. |
| `loading` | boolean | 요청이 진행 중인지 여부를 나타내는 부울 |
| `error` | ApolloError | graphQLErrors 및 networkError 속성이있는 런타임 오류 |
| `variables` | { \[key: string\]: any } | 쿼리가 호출 된 변수를 포함하는 객체 |
| `networkStatus` | NetworkStatus | 네트워크 요청의 자세한 상태에 해당하는 1-8의 숫자입니다. 리 페치 및 폴링 상태에 대한 정보를 포함합니다. notifyOnNetworkStatusChange prop과 함께 사용됩니다 |
| `refetch` | \(variables?: TVariables\) =&gt; Promise&lt;ApolloQueryResult&gt; | 쿼리를 다시 가져오고 선택적으로 새 변수를 전달할 수있는 기능 |
| `fetchMore` | \({ query?: DocumentNode, variables?: TVariables, updateQuery: Function}\) =&gt; Promise&lt;ApolloQueryResult&gt; | 쿼리에 페이지 매김을 가능하게하는 기능 |
| `startPolling` | \(interval: number\) =&gt; void | 이 함수는 간격을 ms 단위로 설정하고 지정된 간격이 경과 할 때마다 쿼리를 가져옵니다. |
| `stopPolling` | \(\) =&gt; void | 이 함수는 쿼리 폴링을 중지합니다. |
| `subscribeToMore` | \(options: { document: DocumentNode, variables?: TVariables, updateQuery?: Function, onError?: Function}\) =&gt; \(\) =&gt; void | 구독을 설정하는 기능입니다. subscribeToMore는 구독을 취소하는 데 사용할 수있는 함수를 반환합니다 |
| `updateQuery` | \(previousResult: TData, options: { variables: TVariables }\) =&gt; TData | 페치, 변이 또는 구독 컨텍스트 외부의 캐시에서 쿼리 결과를 업데이트 할 수있는 기능 |
| `client` | ApolloClient | ApolloClient 인스턴스 쿼리를 수동으로 발생 시키거나 캐시에 데이터를 쓰는 데 유용합니다. |
| `called` | boolean | useLazyQuery가 사용하는 쿼리 함수가 호출되었는지 여부를 나타내는 부울 \(useQuery / Query에 설정되지 않음\) |

### `useMutation` <a id="usemutation"></a>

#### Example <a id="example-2"></a>

```text
import { useMutation } from '@apollo/react-hooks';
import gql from 'graphql-tag';

const ADD_TODO = gql`
  mutation AddTodo($type: String!) {
    addTodo(type: $type) {
      id
      type
    }
  }
`;

function AddTodo() {
  let input;
  const [addTodo, { data }] = useMutation(ADD_TODO);

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault();
          addTodo({ variables: { type: input.value } });
          input.value = '';
        }}
      >
        <input
          ref={node => {
            input = node;
          }}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
}
```

> useMutation에 대한 자세한 내용은 Mutations 섹션을 참조하십시오.

#### Function Signature**\(서명\)** <a id="function-signature-2"></a>

```text
function useMutation<TData = any, TVariables = OperationVariables>(
  mutation: DocumentNode,
  options?: MutationHookOptions<TData, TVariables>,
): MutationTuple<TData, TVariables> {}
```

#### Params <a id="params-2"></a>

**mutation**

| PARAM | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `mutation` | DocumentNode | GraphQL 돌연변이 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. |

**options**

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `mutation` | DocumentNode | GraphQL 돌연변이 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. 돌연변이는 후크에 대한 첫 번째 매개 변수로 전달 될 수 있으므로 useMutation Hook의 경우 선택 사항입니다. 돌연변이 구성 요소에 필요합니다. |
| `variables` | { \[key: string\]: any } | 돌연변이가 실행하는 데 필요한 모든 변수를 포함하는 객체 |
| `update` | \(cache: DataProxy, mutationResult: FetchResult\) | 돌연변이가 발생한 후 캐시를 업데이트하는 데 사용되는 함수 |
| `ignoreResults` | boolean | true이면 반환 된 데이터 속성이 돌연변이 결과로 업데이트되지 않습니다. |
| `optimisticResponse` | Object | 서버에서 결과가 나오기 전에 돌연변이 응답 제공 |
| `refetchQueries` | Array&lt;string\|{ query: DocumentNode, variables?: TVariables}&gt; \| \(\(mutationResult: FetchResult\) =&gt; Array&lt;string\|{ query: DocumentNode, variables?: TVariables}&gt;\) | 돌연변이가 발생한 후 다시 가져올 쿼리를 지정할 수있는 배열 또는 함수입니다. 배열 값은 쿼리 \(선택적 변수 포함\)이거나 refeteched 할 쿼리의 문자열 이름 일 수 있습니다 |
| `awaitRefetchQueries` | boolean | refetchQueries의 일부로 리 페치 된 쿼리는 비동기 적으로 처리되며, 돌연변이가 완료 \(해결\)되기 전에 대기하지 않습니다. 이를 true로 설정하면 돌연변이가 완료된 것으로 간주되기 전에 리 페치 된 쿼리가 완료됩니다. 기본적으로 false입니다. |
| `onCompleted` | \(data: TData\) =&gt; void | 돌연변이가 성공적으로 완료되면 콜백이 실행됩니다 |
| `onError` | \(error: ApolloError\) =&gt; void | 오류 발생시 실행 된 콜백 |
| `context` | Record&lt;string, any&gt; | Shared context between your component and your network interface \(Apollo Link\). |
| `client` | ApolloClient | ApolloClient 인스턴스 기본적으로 useMutation / Mutation은 컨텍스트를 통해 전달 된 클라이언트를 사용하지만 다른 클라이언트를 전달할 수 있습니다. |

#### Result <a id="result-2"></a>

**Mutate function:**

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `mutate` | \(options?: MutationOptions\) =&gt; Promise&lt;FetchResult&gt; | UI에서 돌연변이를 유발하는 기능입니다. 선택적으로 변수, optimisticResponse, refetchQueries를 전달하고 옵션으로 업데이트하여 useMutation / Mutation에 전달 된 옵션 / 소품을 대체합니다. 이 함수는 돌연변이 결과를 충족시키는 약속을 반환합니다. |

**Mutation result:**

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `data` | TData | 돌연변이로부터 데이터가 반환되었습니다. ignoreResults가 true이면 정의되지 않을 수 있습니다. |
| `loading` | boolean | 변이가 비행 중인지 여부를 나타내는 부울 |
| `error` | ApolloError | 변이에서 반환 된 모든 오류 |
| `called` | boolean | mutate 함수가 호출되었는지 여부를 나타내는 부울 |
| `client` | ApolloClient | ApolloClient 인스턴스 client.writeData 및 client.readQuery와 같이 업데이트 함수 컨텍스트 외부에서 캐시 메소드를 호출하는 데 유용합니다. |

### `useSubscription` <a id="usesubscription"></a>

#### Example <a id="example-3"></a>

```text
const COMMENTS_SUBSCRIPTION = gql`
  subscription onCommentAdded($repoFullName: String!) {
    commentAdded(repoFullName: $repoFullName) {
      id
      content
    }
  }
`;

function DontReadTheComments({ repoFullName }) {
  const {
    data: { commentAdded },
    loading,
  } = useSubscription(COMMENTS_SUBSCRIPTION, { variables: { repoFullName } });
  return <h4>New comment: {!loading && commentAdded.content}</h4>;
}
```

> useSubscription에 대한 자세한 개요는 구독 섹션을 참조하십시오.

#### Function Signature <a id="function-signature-3"></a>

```text
function useSubscription<TData = any, TVariables = OperationVariables>(
  subscription: DocumentNode,
  options?: SubscriptionHookOptions<TData, TVariables>,
): {
  variables: TVariables;
  loading: boolean;
  data?: TData;
  error?: ApolloError;
} {}
```

#### Params <a id="params-3"></a>

**subscription**

| PARAM | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `subscription` | DocumentNode | graphQL 구독 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. |

**options**

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `subscription` | DocumentNode | graphQL 구독 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. 서브 스크립 션을 첫 번째 매개 변수로 후크에 전달할 수 있으므로 useSubscription 후크의 경우 선택 사항입니다. 구독 구성 요소에 필요합니다 |
| `variables` | { \[key: string\]: any } | 구독이 실행해야하는 모든 변수를 포함하는 객체 |
| `shouldResubscribe` | boolean | 구독을 구독 취소하고 다시 구독해야하는지 결정 |
| `onSubscriptionData` | \(options: OnSubscriptionDataOptions&lt;TData&gt;\) =&gt; any | useSubscription Hook / Subscription 구성 요소가 데이터를 수신 할 때마다 트리거되는 콜백 함수의 등록을 허용합니다. 콜백 옵션 객체 매개 변수는 클라이언트의 현재 Apollo 클라이언트 인스턴스와 subscriptionData의 수신 된 구독 데이터로 구성됩니다 |
| `fetchPolicy` | FetchPolicy | useSubscription Hook / Subscription 구성 요소가 데이터를 수신 할 때마다 트리거되는 콜백 함수의 등록을 허용합니다. 콜백 옵션 객체 매개 변수는 클라이언트의 현재 Apollo 클라이언트 인스턴스와 subscriptionData의 수신 된 구독 데이터로 구성됩니다. |
| `client` | ApolloClient | ApolloClient 인스턴스 기본적으로 useSubscription / Subscription은 컨텍스트를 통해 전달 된 클라이언트를 사용하지만 다른 클라이언트를 전달할 수 있습니다 |

#### Result <a id="result-3"></a>

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `data` | TData | GraphQL 구독 결과가 포함 된 객체입니다. 빈 개체가 기본값입니다. |
| `loading` | boolean | 초기 데이터가 반환되었는지 여부를 나타내는 부울 |
| `error` | ApolloError | graphQLErrors 및 networkError 속성이있는 런타임 오류 |

### `useApolloClient` <a id="useapolloclient"></a>

#### Example <a id="example-4"></a>

```text
function SomeComponent() {
  const client = useApolloClient();
  // `client` is now set to the `ApolloClient` instance being used by the
  // application (that was configured using something like `ApolloProvider`)
}
```

#### Function Signature <a id="function-signature-4"></a>

```text
function useApolloClient(): ApolloClient<object> {}
```

#### Result <a id="result-4"></a>

| PARAM | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| Apollo Client instance | ApolloClient&lt;object&gt; | 응용 프로그램에서 사용중인 ApolloClient 인스턴스 |

#### `ApolloProvider` <a id="apolloprovider"></a>

ApolloProvider 구성 요소는 React의 컨텍스트 API를 활용하여 구성된 Apollo 클라이언트 인스턴스를 React 구성 요소 트리 전체에서 사용할 수 있도록합니다. 이 컴포넌트는 @ apollo / react-common 패키지 또는 패키지가있는 @ apollo / react-hooks, @ apollo / react-components 및 @ apollo / react-hoc 패키지에서 직접 가져올 수 있습니다.

```text
import { ApolloProvider } from '@apollo/react-hooks';
```

#### Props <a id="props"></a>

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `client` | ApolloClient&lt;TCache&gt; | An `ApolloClient` instance. |

#### Example <a id="example"></a>

```text
ReactDOM.render(
  <ApolloProvider client={client}>
    <MyRootComponent />
  </ApolloProvider>,
  document.getElementById('root'),
);
```

