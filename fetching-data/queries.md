---
description: useQuery 후크를 사용하여 데이터를 가져 오는 방법 학습
---

# Queries

> 이 문는 최근 Apollo 클라이언트의 React Hook 지원을 반영하여 업데이트되었습니다. 대신 Apollo Client의 render-prop API로 쿼리를 수행하려면이 기사의 이전 버전을 참조하십시오.

간단하고 예측 가능한 방식으로 데이터를 가져 오는 것이 Apollo Client의 핵심 기능 중 하나입니다. 이 기사에서는 useQuery 후크를 사용하여 React에서 GraphQL 데이터를 가져오고 결과를 UI에 첨부하는 방법을 보여줍니다. 또한 Apollo Client가 오류 및로드 상태를 추적하여 데이터 관리 코드를 단순화하는 방법도 알아 봅니다.

### Prerequisites\(전제 조건\) <a id="prerequisites"></a>

이 문에서는 기본 GraphQL 쿼리 작성에 익숙하다고 가정합니다. 새로 고침이 필요한 경우이 가이드를 읽고 GraphiQL에서 쿼리 실행을 연습하는 것이 좋습니다. Apollo Client 쿼리는 표준 GraphQL이므로 GraphiQL에서 실행되는 모든 쿼리는 query를 사용하도록 제공된 경우에도 실행됩니다.

또한이 기사에서는 이미 Apollo Client를 설정했으며 ApolloProvider 구성 요소에 React 앱을 래핑했다고 가정합니다. 이러한 단계 중 하나에 대한 도움이 필요한 경우 시작 안내서를 읽으십시오

> 아래 예제와 함께 시작하려면 CodeSandbox에서 시작 프로젝트와 샘플 GraphQL 서버를 엽니 다. 여기에서 완성 된 버전의 앱을 볼 수 있습니다.

### 쿼리 실행

useQuery React 후크는 Apollo 애플리케이션에서 쿼리를 실행하기위한 기본 API입니다. React 컴포넌트 내에서 쿼리를 실행하려면 useQuery를 호출하고 GraphQL 쿼리 문자열을 전달하십시오. 구성 요소가 렌더링되면 useQuery는 UI 렌더링에 사용할 수있는로드, 오류 및 데이터 속성이 포함 된 객체를 Apollo Client에서 반환합니다.

예를 봅시다. 먼저 GET\_DOGS라는 GraphQL 쿼리를 만듭니다. 쿼리 문자열을 쿼리 문서로 구문 분석하려면 gql 함수에서 쿼리 문자열을 래핑해야합니다.

```text
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

```text
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

```text
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

```text
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

```text
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

```text
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

```text
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

