---
description: useMutation 후크를 사용하여 데이터를 업데이트하는 방법 학습
---

# Mutations

이제 Apollo Client를 사용하여 백엔드에서 데이터를 가져 오는 방법을 배웠으므로 자연스럽게 다음 단계는 해당 데이터를 돌연변이로 업데이트하는 방법을 배우는 것입니다. 이 기사는 useMutation 후크를 사용하여 GraphQL 서버에 업데이트를 보내는 방법을 보여줍니다. 또한 돌연변이를 실행 한 후 Apollo Client 캐시를 업데이트하는 방법과 돌연변이의로드 및 오류 상태를 추적하는 방법에 대해서도 배웁니다.

### Prerequisites\(전제조건\)

이 기사는 기본 GraphQL 돌연변이를 만드는 데 익숙하다고 가정합니다. 새로 고침이 필요한 경우이 가이드를 읽는 것이 좋습니다.

또한이 기사에서는 이미 Apollo Client를 설정했으며 ApolloProvider 구성 요소에 React 앱을 래핑했다고 가정합니다. 이러한 단계 중 하나에 대한 도움이 필요한 경우 시작 안내서를 읽으십시오.

> 아래 예제와 함께 시작하려면 CodeSandbox에서 시작 프로젝트와 샘플 GraphQL 서버를 엽니다. 여기에서 완성 된 버전의 앱을 볼 수 있습니다.

### Executing a mutation\(mutation 실행\)

useMutation React 후크는 Apollo 애플리케이션에서 돌연변이를 실행하기위한 기본 API입니다. 돌연변이를 실행하려면 먼저 React 컴포넌트 내에서 useMutation을 호출하고 돌연변이를 나타내는 GraphQL 문자열을 전달하십시오. 컴포넌트가 렌더링 될 때 useMutation은 다음을 포함하는 튜플을 반환합니다.

* 언제든지 변이를 실행하기 위해 호출 할 수있는 돌연변이 기능
* 변이 실행의 현재 상태를 나타내는 필드가있는 객체

예를 봅시다. 먼저, ADD\_TODO라는 GraphQL 변이를 생성하는데, 이것은 할 일 목록에 항목을 추가하는 것을 나타냅니다. gq 함수에서 GraphQL 문자열을 랩핑하여 쿼리 문서로 구문 분석해야합니다.

```text
//index.js

import gql from 'graphql-tag';
import { useMutation } from '@apollo/react-hooks';

const ADD_TODO = gql`
  mutation AddTodo($type: String!) {
    addTodo(type: $type) {
      id
      type
    }
  }
`;
```

다음으로 할일 목록의 제출 양식을 나타내는 AddTodo라는 컴포넌트를 작성합니다. 그 안에 ADD\_TODO 변이를 useMutation 후크로 전달합니다.

```text
//index.js

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

#### Calling the mutate function\(변이 함 호출\) <a id="calling-the-mutate-function"></a>

useMutation 후크는 구성 요소가 렌더링 될 때 전달한 돌연변이를 자동으로 실행하지 않습니다. 대신, 첫 번째 위치 \(위 예제에서 addTodo에 할당\)에 mutate 함수가있는 튜플을 반환합니다. 그런 다음 언제든지 mutate 함수를 호출하여 Apollo Client가 돌연변이를 실행하도록 지시합니다. 위의 예에서 사용자가 양식을 제출하면 addTodo를 호출합니다.

#### Providing options\(제공 옵션\) <a id="providing-options"></a>

두 useMutation 자체와의 mutate 기능은 API 참조에 설명 된 옵션을 받아들입니다. 모든 옵션은 옵션 이전 useMutation 제공을 해당 돌연변이 체 기능 재정에 제공합니다. 위의 예에서, 우리는 우리가 돌연변이가 요구하는 어떤 GraphQL 변수를 지정할 수 있습니다 addTodo에 변수 옵션을 제공합니다.

#### Tracking mutation status\(변이 상태 추적\) <a id="tracking-mutation-status"></a>

mutate 함수 외에도 useMutation 후크는 돌연변이 실행의 현재 상태를 나타내는 객체를 반환합니다. 이 객체의 필드 \(API 참조에 완전히 문서화 됨\)에는 mutate 함수가 아직 호출되었는지 여부와 mutation 결과가 현재로드 중인지 여부를 나타내는 부울이 포함됩니다.

### Updating the cache after a mutation\(변이 후 캐시 업데이트\) <a id="updating-the-cache-after-a-mutation"></a>

변이를 실행하면 백엔드 데이터가 수정됩니다. 해당 데이터가 Apollo Client 캐시에도있는 경우 돌연변이 결과를 반영하도록 캐시를 업데이트해야 할 수도 있습니다. 이는 돌연변이가 기존의 단일 엔티티를 업데이트하는지 여부에 따라 다릅니다.

#### Updating a single existing entity\(기존 단일 엔티티 업데이트\) <a id="updating-a-single-existing-entity"></a>

변이가 기존의 단일 엔티티를 업데이트하는 경우 Apollo 클라이언트는 돌연변이가 반환 될 때 캐시에서 해당 엔티티의 값을 자동으로 업데이트 할 수 있습니다. 그렇게하려면 돌연변이는 수정 된 필드의 값과 함께 수정 된 엔티티의 id를 반환해야합니다. 편리하게도 돌연변이는 기본적으로 Apollo Client에서이 작업을 수행합니다.

할 일 목록에서 기존 항목의 값을 수정할 수있는 예를 살펴 보겠습니다.

```text
const UPDATE_TODO = gql`
  mutation UpdateTodo($id: String!, $type: String!) {
    updateTodo(id: $id, type: $type) {
      id
      type
    }
  }
`;

function Todos() {
  const { loading, error, data } = useQuery(GET_TODOS);
  const [updateTodo] = useMutation(UPDATE_TODO);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  return data.todos.map(({ id, type }) => {
    let input;

    return (
      <div key={id}>
        <p>{type}</p>
        <form
          onSubmit={e => {
            e.preventDefault();
            updateTodo({ variables: { id, type: input.value } });

            input.value = '';
          }}
        >
          <input
            ref={node => {
              input = node;
            }}
          />
          <button type="submit">Update Todo</button>
        </form>
      </div>
    );
  });
}
```

이 구성 요소를 사용하여 UPDATE\_TODO 변이를 실행하면 변이는 수정 된 작업 항목의 ID와 항목의 새 유형을 모두 리턴합니다. Apollo Client는 ID별로 엔티티를 캐시하므로 캐시에서 해당 엔티티를 자동으로 업데이트하는 방법을 알고 있습니다. 응용 프로그램의 UI는 캐시의 변경 사항을 반영하여 즉시 업데이트됩니다.

#### Making all other cache updates\(다른 모든 캐시 업데이트\) <a id="making-all-other-cache-updates"></a>

변이가 여러 엔터티를 수정하거나 엔터티를 만들거나 삭제하는 경우 돌연변이 결과를 반영하기 위해 Apollo 클라이언트 캐시가 자동으로 업데이트되지 않습니다. 이 문제를 해결하기 위해 useMutation 호출에 업데이트 기능이 포함될 수 있습니다.

업데이트 기능의 목적은 돌연변이가 백엔드 데이터에 적용한 수정 내용과 일치하도록 캐시 된 데이터를 수정하는 것입니다. 돌연변이 실행의 예에서 ADD\_TODO 돌연변이에 대한 업데이트 기능은 캐시 된 버전의 할 일 목록에 동일한 항목을 추가해야합니다.

다음 샘플은 useMutation 호출에서 업데이트 기능 정의를 보여줍니다.

```text
const GET_TODOS = gql`
  query GetTodos {
    todos
  }
`;

function AddTodo() {
  let input;
  const [addTodo] = useMutation(
    ADD_TODO,
    {
      update(cache, { data: { addTodo } }) {
        const { todos } = cache.readQuery({ query: GET_TODOS });
        cache.writeQuery({
          query: GET_TODOS,
          data: { todos: todos.concat([addTodo]) },
        });
      }
    }
  );

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault();
          addTodo({ variables: { type: input.value } });
          input.value = "";
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

표시된 것처럼 업데이트 기능에는 Apollo Client 캐시를 나타내는 캐시 개체가 전달됩니다. 이 객체는 GraphQL 서버와 상호 작용하는 것처럼 캐시에서 GraphQL 작업을 실행할 수있는 readQuery 및 writeQuery 함수를 제공합니다.

> 캐시 된 데이터와 상호 작용에서 지원되는 캐시 기능에 대해 자세히 알아보십시오.

업데이트 함수는 또한 돌연변이의 결과를 포함하는 데이터 속성을 가진 객체로 전달됩니다. cache.writeQuery로 캐시를 업데이트하려면이 값을 사용하십시오.

변이가 [optimistic response](https://www.apollographql.com/docs/react/performance/optimistic-ui/),을 지정하는 경우 업데이트 기능이 두 번 호출됩니다. 낙관적 결과로 한 번, 그리고 돌연변이가 발생할 때 돌연변이의 실제 결과로 다시 한 번 호출됩니다.

위의 예에서 업데이트 함수는 먼저 cache.readQuery를 사용하여 캐시에서 기존 작업 목록을 읽습니다. 그런 다음 돌연변이에서 새로운 할 일 항목을 목록에 추가하고 cache.writeQuery를 사용하여 캐시에 다시 씁니다.

업데이트 기능 내에서 캐시 된 데이터에 대한 변경 사항은 해당 데이터의 변경 사항을 수신하는 쿼리로 자동 브로드 캐스트됩니다. 결과적으로 애플리케이션의 UI가 새로 캐시 된 값을 반영하도록 업데이트됩니다.

### Tracking loading and error states\(로딩 및 오류상태 추적\) <a id="tracking-loading-and-error-states"></a>

useMutation 후크는 돌연변이의 로딩 및 오류 상태를 추적하기위한 메커니즘을 제공합니다.

기존의 단일 엔티티 업데이트에서 Todos 컴포넌트를 다시 방문하십시오.

```text
function Todos() {
  const { loading: queryLoading, error: queryError, data } = useQuery(
    GET_TODOS,
  );

  const [
    updateTodo,
    { loading: mutationLoading, error: mutationError },
  ] = useMutation(UPDATE_TODO);

  if (queryLoading) return <p>Loading...</p>;
  if (queryError) return <p>Error :(</p>;

  return data.todos.map(({ id, type }) => {
    let input;

    return (
      <div key={id}>
        <p>{type}</p>
        <form
          onSubmit={e => {
            e.preventDefault();
            updateTodo({ variables: { id, type: input.value } });

            input.value = '';
          }}
        >
          <input
            ref={node => {
              input = node;
            }}
          />
          <button type="submit">Update Todo</button>
        </form>
        {mutationLoading && <p>Loading...</p>}
        {mutationError && <p>Error :( Please try again</p>}
      </div>
    );
  });
}
```

위에서 볼 수 있듯이, useMutation에 의해 반환 된 결과 객체에서 loading 및 error 속성을 구조 해제하여 UI에서 돌연변이 상태를 추적 할 수 있습니다. 콜백을 사용하려는 경우 useMutation 후크는 onCompleted 및 onError 옵션도 지원합니다.

API 참조에서 useMutation에 의해 리턴 된 모든 필드에 대해 학습하십시오.

### `useMutation` API <a id="usemutation-api"></a>

useMutation 후크에 지원되는 옵션 및 결과 필드가 아래에 나열되어 있습니다.

useMutation에 대한 대부분의 호출은 이러한 옵션의 대부분을 생략 할 수 있지만 옵션이 존재한다는 것을 아는 것이 유용합니다. 사용법 예제와 함께 useMutation 후크 API에 대해 자세히 학습하려면 API 참조를 참조하십시오.

#### Options <a id="options"></a>

useMutation 후크는 다음 옵션을 허용합니다.

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `mutation` | DocumentNode | A GraphQL mutation document parsed into an AST by `graphql-tag`. **Optional** for the `useMutation` Hook since the mutation can be passed in as the first parameter to the Hook. **Required** for the `Mutation` component. |
| `variables` | { \[key: string\]: any } | An object containing all of the variables your mutation needs to execute |
| `update` | \(cache: DataProxy, mutationResult: FetchResult\) | A function used to update the cache after a mutation occurs |
| `ignoreResults` | boolean | If true, the returned `data` property will not update with the mutation result. |
| `optimisticResponse` | Object | Provide a [mutation response](https://www.apollographql.com/docs/react/performance/optimistic-ui/) before the result comes back from the server |
| `refetchQueries` | Array&lt;string\|{ query: DocumentNode, variables?: TVariables}&gt; \| \(\(mutationResult: FetchResult\) =&gt; Array&lt;string\|{ query: DocumentNode, variables?: TVariables}&gt;\) | An array or function that allows you to specify which queries you want to refetch after a mutation has occurred. Array values can either be queries \(with optional variables\) or just the string names of queries to be refeteched. |
| `awaitRefetchQueries` | boolean | Queries refetched as part of `refetchQueries` are handled asynchronously, and are not waited on before the mutation is completed \(resolved\). Setting this to `true` will make sure refetched queries are completed before the mutation is considered done. `false` by default. |
| `onCompleted` | \(data: TData\) =&gt; void | A callback executed once your mutation successfully completes |
| `onError` | \(error: ApolloError\) =&gt; void | A callback executed in the event of an error. |
| `context` | Record&lt;string, any&gt; | Shared context between your component and your network interface \(Apollo Link\). Useful for setting headers from props or sending information to the `request` function of Apollo Boost. |
| `client` | ApolloClient | An `ApolloClient` instance. By default `useMutation` / `Mutation` uses the client passed down via context, but a different client can be passed in. |

#### Result <a id="result"></a>

useMutation 결과는 첫 번째 위치에 mutate 함수가 있고 튜플은 두 번째 위치에 mute 결과를 나타냅니다.

mutate 함수를 호출하여 UI에서 변이를 트리거합니다.

변이 기능 :

| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `mutate` | \(options?: MutationOptions\) =&gt; Promise&lt;FetchResult&gt; | A function to trigger a mutation from your UI. You can optionally pass `variables`, `optimisticResponse`, `refetchQueries`, and `update` in as options, which will override options/props passed to `useMutation` / `Mutation`. The function returns a promise that fulfills with your mutation result. |

**Mutation result:**  


| PROPERTY | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `data` | TData | The data returned from your mutation. It can be undefined if `ignoreResults` is true. |
| `loading` | boolean | A boolean indicating whether your mutation is in flight |
| `error` | ApolloError | Any errors returned from the mutation |
| `called` | boolean | A boolean indicating if the mutate function has been called |
| `client` | ApolloClient | Your `ApolloClient` instance. Useful for invoking cache methods outside the context of the update function, such as `client.writeData` and `client.readQuery`. |

### 다음단계

useQuery 및 useMutation 후크는 함께 GraphQL 작업을 수행하기위한 Apollo Client의 핵심 API를 나타냅니다. 두 가지 모두에 익숙해 졌으므로 다음을 포함하여 Apollo Client의 전체 기능 세트를 활용할 수 있습니다.

* Optimistic UI\(낙관적 UI\) : 돌연변이 결과가 서버에서 되돌아 오기 전에 낙관적 응답을 반환하여인지 성능을 향상시키는 방법을 배웁니다.
* 로컬 상태 : Apollo Client를 사용하여 클라이언트 측 변이를 실행하여 응용 프로그램의 로컬 상태 전체를 관리하십시오.
* Apollo에서의 캐싱 : Apollo Client 캐시와 그 정규화 방법에 대해 자세히 알아보십시오. 캐시를 이해하면 돌연변이에 대한 업데이트 기능을 작성할 때 도움이됩니다!



