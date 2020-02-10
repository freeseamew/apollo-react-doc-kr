---
description: useQuery 후크를 사용하여 데이터를 가져 오는 방법 학습
---

# Queries

> 이 문는 최근 Apollo 클라이언트의 React Hook 지원을 반영하여 업데이트되었습니다. 대신 Apollo Client의 render-prop API로 쿼리를 수행하려면이 기사의 이전 버전을 참조하십시오.

간단하고 예측 가능한 방식으로 데이터를 가져 오는 것이 Apollo Client의 핵심 기능 중 하나입니다. 이 기사에서는 useQuery 후크를 사용하여 React에서 GraphQL 데이터를 가져오고 결과를 UI에 첨부하는 방법을 보여줍니다. 또한 Apollo Client가 오류 및로드 상태를 추적하여 데이터 관리 코드를 단순화하는 방법도 알아 봅니다.

### Prerequisites\(전제 조건\) <a id="prerequisites"></a>

이 문에서는 기본 GraphQL 쿼리 작성에 익숙하다고 가정합니다. 새로 고침이 필요한 경우이 가이드를 읽고 GraphiQL에서 쿼리 실행을 연습하는 것이 좋습니다. Apollo Client 쿼리는 표준 GraphQL이므로 GraphiQL에서 실행되는 모든 쿼리는 query를 사용하도록 제공된 경우에도 실행됩니다.

또한이 문서에서는 이미 Apollo Client를 설정했으며 ApolloProvider 구성 요소에 React 앱을 랩핑했다고 가정합니다. 이러한 단계 중 하나에 대한 도움이 필요한 경우 시작 안내서를 읽으십시오

> 아래 예제와 함께 시작하려면 CodeSandbox에서 시작 프로젝트와 샘플 GraphQL 서버를 엽니 다. 여기에서 완성 된 버전의 앱을 볼 수 있습니다.

### 쿼리 실행

useQuery React hook는 Apollo 애플리케이션에서 쿼리를 실행하기위한 기본 API입니다. React 컴포넌트 내에서 쿼리를 실행하려면 useQuery를 호출하고 GraphQL 쿼리 문자열을 전달하십시오. 구성 요소가 렌더링되면 useQuery는 UI 렌더링에 사용할 수있는로드, 오류 및 데이터 속성이 포함 된 객체를 Apollo Client에서 반환합니다.

예를 봅시다. 먼저 GET\_DOGS라는 GraphQL 쿼리를 만듭니다. 쿼리 문자열을 쿼리 문서로 구문 분석하려면 gql 함수에서 쿼리 문자열을 래핑해야합니다.

```javascript
//index.js

import gql from 'graphql-tag';
import { useQuery } from '@apollo/react-hooks';

const GET_DOGS = gql`
  {
    dogs {
      id
      breed
    }
  }
`;
```

다음으로 Dogs라는 구성 요소를 만듭니다. 그 안에 GET\_DOGS 쿼리를 useQuery 후크에 전달합니다.

```javascript
//index.js

function Dogs({ onDogSelected }) {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return 'Loading...';
  if (error) return `Error! ${error.message}`;

  return (
    <select name="dog" onChange={onDogSelected}>
      {data.dogs.map(dog => (
        <option key={dog.id} value={dog.breed}>
          {dog.breed}
        </option>
      ))}
    </select>
  );
}
```

쿼리가 실행되고로드, 오류 및 데이터 값이 변경되면 Dogs 구성 요소는 쿼리 상태에 따라 다른 UI 요소를 지능적으로 렌더링 할 수 있습니다.

* 로드가 참이면 \(쿼리가 여전히 비행 중임을 나타냄\) 구성 요소는로드 중 ... 통지를 표시합니다.
* 로드가 거짓이고 오류가 없으면 쿼리가 완료된 것입니다. 구성 요소는 서버에서 리턴 한 개 유형 목록으로 채워진 드롭 다운 메뉴를 렌더링합니다.

사용자가 채워진 드롭 다운에서 개 품종을 선택하면 선택은 제공된 onDogSelected 함수를 통해 상위 구성 요소로 전송됩니다.

다음 단계에서는 드롭 다운을 GraphQL 변수를 사용하는보다 정교한 쿼리와 연결합니다.

### Caching query results\(쿼리 결과 캐시\) <a id="caching-query-results"></a>

Apollo Client가 서버에서 쿼리 결과를 가져올 때마다 해당 결과를 로컬로 자동 캐시합니다. 따라서 동일한 쿼리의 후속 실행이 매우 빨라집니다.

이 캐싱이 실제로 작동하는지 확인하려면 DogPhoto라는 새 구성 요소를 만들어 보겠습니다. DogPhoto는 Dogs 컴포넌트에서 드롭 다운 메뉴의 현재 값을 반영하는 breed라는 소품을 허용합니다.

```javascript
//index.js

const GET_DOG_PHOTO = gql`
  query Dog($breed: String!) {
    dog(breed: $breed) {
      id
      displayImage
    }
  }
`;

function DogPhoto({ breed }) {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
  );
}
```

이번에는 useQuery 후크에 구성 옵션 \(변수\)을 제공하고 있습니다. 변수 옵션은 GraphQL 쿼리에 전달하려는 모든 변수를 포함하는 객체입니다. 이 경우 현재 선택된 품종을 드롭 다운에서 전달하려고합니다.

드롭 다운에서 불독을 선택하면 사진이 나타납니다. 그런 다음 다른 품종으로 전환 한 다음 불독으로 다시 전환하십시오. 불독 사진은 두 번째로 즉시로드됩니다. 이것은 직장에서 아폴로 캐시입니다!

다음으로, 캐시 된 데이터를 최신 상태로 유지하기위한 몇 가지 기술을 알아 보겠습니다.

### 캐시된 결과 업데이트 

쿼리 결과 캐싱은 편리하고 수행하기 쉽지만 때로는 캐시 된 데이터가 서버에서 최신 상태인지 확인하려고합니다. Apollo Client는이를 위해 두 가지 전략, 즉 폴링 및 리 페칭을 지원합니다.

#### Polling <a id="polling"></a>

폴링은 지정된 간격으로 쿼리가 주기적으로 실행되도록하여 서버와 거의 실시간으로 동기화합니다. 조회 폴링을 사용 가능하게하려면 간격 \(밀리 초\)으로 pollInterval 구성 옵션을 useQuery 후크에 전달하십시오.

```javascript
function DogPhoto({ breed }) {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
    skip: !breed,
    pollInterval: 500,
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
  );
}
```

pollInterval을 500으로 설정하면 0.5 초마다 서버에서 현재 유형의 이미지를 가져옵니다. pollInterval을 0으로 설정하면 쿼리가 폴링되지 않습니다.

> useQuery 후크에 의해 리턴되는 startPolling 및 stopPolling 함수를 사용하여 폴링을 동적으로 시작 및 중지 할 수도 있습니다.

#### Refetching <a id="refetching"></a>

다시 가져 오기를 사용하면 고정 간격을 사용하는 대신 특정 사용자 작업에 대한 응답으로 쿼리 결과를 새로 고칠 수 있습니다.

클릭 할 때마다 쿼리의 다시 불러 오기 기능을 호출하는 DogPhoto 컴포넌트에 버튼을 추가합시다.

선택적으로 리 페치 함수에 새 변수 오브젝트를 제공 할 수 있습니다. 그렇지 않은 경우 \(다음 예에서와 같이\) 쿼리는 이전 실행에서 사용한 것과 동일한 변수를 사용합니다.

```javascript
// index.js

function DogPhoto({ breed }) {
  const { loading, error, data, refetch } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
    skip: !breed,
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={() => refetch()}>Refetch!</button>
    </div>
  );
}
```

버튼을 클릭하면 UI가 새로운 개 사진으로 업데이트됩니다. 리 페치는 새로운 데이터를 보장하는 훌륭한 방법이지만로드 상태에 약간의 복잡성을 초래합니다. 다음 섹션에서는 복잡한로드 및 오류 상태를 처리하기위한 전략을 다룹니다.

### Inspecting loading states\(로딩 상태 검사\) <a id="inspecting-loading-states"></a>

우리는 이미 useQuery 후크가 쿼리의 현재 로딩 상태를 노출시키는 것을 보았습니다. 이는 쿼리가 처음로드 될 때 유용하지만 다시 가져 오거나 폴링 할 때로드 상태는 어떻게됩니까?

이전 섹션의 리 페칭 예제로 돌아가 봅시다. 다시 불러 오기 버튼을 클릭하면 새 데이터가 도착할 때까지 구성 요소가 다시 렌더링되지 않습니다. 사진을 다시 가져오고 있음을 사용자에게 알리려면 어떻게해야합니까?

다행히 useQuery 후크의 결과 객체는 networkStatus 속성을 통해 쿼리 상태에 대한 세분화 된 정보를 제공합니다. 이 정보를 이용하려면 리 페치가 진행되는 동안 쿼리 구성 요소가 다시 렌더링되도록 notifyOnNetworkStatusChange 옵션을 true로 설정해야합니다.

```javascript
// index.js

function DogPhoto({ breed }) {
  const { loading, error, data, refetch, networkStatus } = useQuery(
    GET_DOG_PHOTO,
    {
      variables: { breed },
      skip: !breed,
      notifyOnNetworkStatusChange: true,
    },
  );

  if (networkStatus === 4) return 'Refetching!';
  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={() => refetch()}>Refetch!</button>
    </div>
  );
}
```

networkStatus 속성은 서로 다른 로딩 상태를 나타내는 1에서 8까지의 숫자 값을 가진 열거 형입니다. 4는 리 페치에 해당하지만 폴링 및 페이지 매김 값도 있습니다. 가능한 모든 로딩 상태의 전체 목록을 보려면 소스를 확인하십시오.

### Inspecting error states\(오류 상태 검사\) <a id="inspecting-error-states"></a>

usePolicy 후크에 errorPolicy 구성 옵션을 제공하여 쿼리 오류 처리를 사용자 정의 할 수 있습니다. 기본값은 none이며, Apollo 클라이언트는 모든 GraphQL 오류를 런타임 오류로 처리하도록 지시합니다. 이 경우 Apollo Client는 서버에서 반환 한 모든 쿼리 응답 데이터를 삭제하고 useQuery 결과 개체의 error 속성을 true로 설정합니다.

errorPolicy를 all로 설정하면 useQuery는 쿼리 응답 데이터를 버리지 않으므로 부분 결과를 렌더링 할 수 있습니다.

### Executing queries manually\(수동으로 쿼리 검사\) <a id="executing-queries-manually"></a>

React가 useQuery 후크를 호출하는 구성 요소를 마운트하고 렌더링하면 Apollo Client가 지정된 쿼리를 자동으로 실행합니다. 그러나 사용자가 버튼을 클릭하는 것과 같은 다른 이벤트에 대한 응답으로 쿼리를 실행하려면 어떻게해야합니까?

useLazyQuery 후크는 구성 요소 렌더링 이외의 이벤트에 대한 응답으로 쿼리를 실행하는 데 적합합니다. 이 후크는 한 가지 주요 예외를 제외하고는 useQuery와 동일하게 작동합니다. useLazyQuery가 호출되면 연관된 쿼리가 즉시 실행되지 않습니다. 대신 쿼리를 실행할 준비가 될 때마다 호출 할 수있는 함수를 결과 튜플에 반환합니다.

```javascript
// index.js

import React, { useState } from 'react';
import { useLazyQuery } from '@apollo/react-hooks';

function DelayedQuery() {
  const [dog, setDog] = useState(null);
  const [getDog, { loading, data }] = useLazyQuery(GET_DOG_PHOTO);

  if (loading) return <p>Loading ...</p>;

  if (data && data.dog) {
    setDog(data.dog);
  }

  return (
    <div>
      {dog && <img src={dog.displayImage} />}
      <button onClick={() => getDog({ variables: { breed: 'bulldog' } })}>
        Click me!
      </button>
    </div>
  );
}
```

> 현재 까지 모든 소스 링크 :[https://codesandbox.io/s/n3jykqpxwm](https://codesandbox.io/s/n3jykqpxwm)

### useQuery API <a id="usequery-api"></a>

useQuery 후크에 지원되는 옵션 및 결과 필드가 아래에 나열되어 있습니다.

useQuery에 대한 대부분의 호출은 이러한 옵션의 대부분을 생략 할 수 있지만 해당 옵션이 존재한다는 것을 아는 것이 유용합니다. 사용법 예제를 통해 useQuery 후크 API에 대해 자세히 학습하려면 API 참조를 참조하십시오.

#### Options <a id="options"></a>

> 링크 :[https://www.apollographql.com/docs/react/data/queries/\#polling](https://www.apollographql.com/docs/react/data/queries/#polling)

| OPTION | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `query` | DocumentNode | graphQL 쿼리 문서는 graphql-tag에 의해 AST로 구문 분석됩니다. 조회는 첫 번째 매개 변수로 후크에 전달 될 수 있으므로 useQuery 후크의 경우 선택 사항입니다. 쿼리 구성 요소에 필요합니다. |
| `variables` | { \[key: string\]: any } | 쿼리 실행에 필요한 모든 변수를 포함하는 객체 |
| `pollInterval` | number | 구성 요소가 데이터를 폴링 할 간격을 ms 단위로 지정합니다. 기본값은 0입니다 \(폴링 없음\). |
| `notifyOnNetworkStatusChange` | boolean | 네트워크 상태 또는 네트워크 오류에 대한 업데이트가 구성 요소를 다시 렌더링해야하는지 여부 기본값은 false입니다. |
| `fetchPolicy` | FetchPolicy | 구성 요소가 Apollo 캐시와 상호 작용하는 방법 기본값은 "cache-first"입니다. . |
| `errorPolicy` | ErrorPolicy | 구성 요소가 네트워크 및 GraphQL 오류를 처리하는 방법 기본값은 "없음"으로, GraphQL 오류를 런타임 오류로 처리합니다. |
| `ssr` | boolean | 서버 측 렌더링 중에 쿼리를 건너 뛰려면 false로 전달하십시오. |
| `displayName` | string | React DevTools에 표시 할 컴포넌트 이름입니다. 기본값은 'Query'입니다. |
| `skip` | boolean | 건너 뛰기가 true이면 쿼리가 완전히 건너 뜁니다. useLazyQuery와 함께 사용할 수 없습니다. |
| `onCompleted` | \(data: TData \| {}\) =&gt; void | 쿼리가 성공적으로 완료되면 콜백이 실행됩니다. |
| `onError` | \(error: ApolloError\) =&gt; void | 오류 발생시 실행 된 콜백 |
| `context` | Record&lt;string, any&gt; | 구성 요소와 네트워크 인터페이스 \(Apollo Link\) 간의 공유 컨텍스트 소품에서 헤더를 설정하거나 Apollo Boost의 요청 기능으로 정보를 보내는 데 유용합니다.. |
| `partialRefetch` | boolean | 참인 경우 쿼리 결과가 부분적인 것으로 표시되고 캐시 누락으로 인해 반환 된 데이터가 Apollo Client QueryManager에 의해 빈 오브젝트로 재설정되는 경우 쿼리 리 페치를 수행합니다. 이전 버전과의 호환성을 위해 기본값은 false이지만 대부분의 사용 사례에서는 true로 변경해야합니다. |
| `client` | ApolloClient | ApolloClient 인스턴스 기본적으로 useQuery / Query는 컨텍스트를 통해 전달 된 클라이언트를 사용하지만 다른 클라이언트를 전달할 수 있습니다. |
| `returnPartialData` | boolean | 캐시에서 완전히 만족하지 않는 쿼리에 대해 캐시에서 부분 결과를 수신하도록 선택하십시오. 기본적으로 false입니다. |

#### Result <a id="result"></a>

호출 된 후 useQuery 후크는 다음 특성을 가진 결과 오브젝트를 리턴합니다. 이 개체에는 쿼리 결과와 함께 다시 가져 오기, 동적 폴링 및 페이지 매김에 유용한 기능이 포함되어 있습니다.

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
| `client` | ApolloClient | ApolloClient 인스턴스 쿼리를 수동으로 발생 시키거나 캐시에 데이터를 쓰는 데 유용합니다. |
| `called` | boolean | useLazyQuery가 사용하는 쿼리 함수가 호출되었는지 여부를 나타내는 부울입니다 \(useQuery / Query에 설정되지 않음\). |

### 다음장에서는

useQuery 후크를 사용하여 데이터를 가져 오는 방법을 이해 했으므로 useMutation 후크를 사용하여 데이터를 업데이트하는 방법을 학습하십시오!

그 후, 다른 유용한 Apollo Client 기능에 대해 알아보십시오.

* 로컬 상태 관리 : 로컬 데이터를 쿼리하는 방법을 배웁니다.
* Pagination : Apollo Client의 fetchMore 기능 덕분에 목록 작성이 쉬워졌습니다. 페이지 매김 자습서에서 자세히 알아보십시오.



