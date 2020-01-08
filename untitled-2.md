---
description: Set up Apollo Client in your React app
---

# 들어가며

> 이 튜토리얼은 React 앱을위한 Apollo Client 설치 및 구성 과정을 안내합니다. GraphQL 또는 Apollo 플랫폼을 시작하는 경우 먼저 풀스 자습서를 완료하는 것이 좋습니다.

Apollo Client를 시작하는 가장 간단한 방법은 권장 설정으로 클라이언트를 구성하는 스타터 키트 인 Apollo Boost를 사용하는 것입니다. Apollo Boost에는 메모리 캐시, 로컬 상태 관리 및 오류 처리와 같은 Apollo 앱 구축에 필수적인 패키지가 포함되어 있습니다. 인증과 같은 기능을 처리 할 수있을만큼 유연합니다.

Apollo Client를 처음부터 구성하려는 고급 사용자 인 경우 Apollo Boost 마이그레이션 안내서를 참조하십시오. 대부분의 사용자에게는 Apollo Boost가 사용자의 요구를 충족시켜야하므로 더 많은 사용자 정의가 필요하지 않으면 전환하지 않는 것이 좋습니다.

### Installation\(설치\) <a id="installation"></a>

먼저 패키지를 설치합니다. 

```text
npm install apollo-boost @apollo/react-hooks graphql
```

* apollo-boost : Apollo Client를 설정하는 데 필요한 모든 것을 포함하는 패키지
* @ apollo / react-hooks : 반응 후크 기반 뷰 레이어 통합
* graphql : 또한 GraphQL 쿼리를 구문 분석합니다

> 이 자습서를 직접 살펴 보려면 create-react-app를 사용하여 로컬로 새 React 프로젝트를 실행하거나 CodeSandbox에서 새 React 샌드 박스를 만드는 것이 좋습니다. 참고로,이 CodeSandbox를 샘플 앱용 GraphQL 서버로 사용하여 Coinbase API에서 환율 데이터를 가져옵니다. 건너 뛰고 우리가 만들려고하는 앱을 보려면 CodeSandbox에서 볼 수 있습니다.

### Client 생성

자, 이제 필요한 모든 종속성이 있으므로 Apollo Client를 작성해 봅시다. 시작해야 할 유일한 것은 GraphQL 서버의 엔드 포인트입니다. uri를 직접 전달하지 않으면 앱이 제공되는 동일한 호스트의 / graphql 엔드 포인트가 기본값으로 사용됩니다.

index.js 파일에서 apollo-boost에서 ApolloClient를 가져 와서 GraphQL 서버의 엔드 포인트를 클라이언트 구성 객체의 uri 속성에 추가합니다.

```text
import ApolloClient from 'apollo-boost';

const client = new ApolloClient({
  uri: 'https://48p1r2roz4.sse.codesandbox.io',
});
```

이제 클라이언트가 데이터 페치를 시작할 준비가되었습니다. Apollo Client를 React에 연결하기 전에 먼저 일반 JavaScript로 쿼리를 보내 봅시다. 동일한 index.js 파일에서 client.query \(\)를 호출하십시오. 쿼리 문자열을 쿼리 문서로 구문 분석하기 위해 gql 함수를 먼저 가져와야합니다.

```text
import { gql } from "apollo-boost";
// or you can use `import gql from 'graphql-tag';` instead

...

client
  .query({
    query: gql`
      {
        rates(currency: "USD") {
          currency
        }
      }
    `
  })
  .then(result => console.log(result));
```

콘솔을 열고 결과 개체를 검사하십시오. loading 및 networkStatus와 같은 다른 속성과 함께 요금이 첨부 된 데이터 속성이 표시되어야합니다. Apollo Client로 데이터를 가져 오는 데 React 또는 다른 프론트 엔드 프레임 워크가 필요하지 않지만 뷰 레이어 통합을 통해 쿼리를 UI에 쉽게 바인딩하고 데이터로 구성 요소를 반응 적으로 업데이트 할 수 있습니다. Apollo 클라이언트를 React에 연결하여 React Apollo를 사용하여 쿼리 구성 요소를 빌드하는 방법을 알아 보겠습니다.

### Connect your client to React\(클라이언트 React와 연결\) <a id="connect-your-client-to-react"></a>

반응 아폴로 클라이언트를 연결하려면에서  @apollo/react-hooks에서 내 보낸ApolloProvider 구성 요소를 사용해야합니다 . ApolloProvider는 React의 Context.Provider와 유사합니다. 그것은 당신의 반작용 응용 프로그램을 래핑하고 구성 요소 트리의 어디에서 액세스 할 수있는 컨텍스트에서 클라이언트를 배치합니다.

 index.js에서의 우리가 ApolloProvider와 응용 프로그램 반응 포장 할 수 있습니다. 우리는 당신이 액세스 GraphQL 데이터에 필요한 모든 장소 위에 앱에 ApolloProvider의 어딘가에 높은 퍼팅하는 것이 좋습니다. 당신이 라우터 반응 사용하는 경우 예를 들어, 루트 경로 구성 요소의 외부 수 있습니다.

```text
import React from 'react';
import { render } from 'react-dom';

import { ApolloProvider } from '@apollo/react-hooks';

const App = () => (
  <ApolloProvider client={client}>
    <div>
      <h2>My first Apollo app 🚀</h2>
    </div>
  </ApolloProvider>
);

render(<App />, document.getElementById('root'));
```

### Request data\(데이터 요청\) <a id="request-data"></a>

ApolloProvider가 연결되면 useQuery hook로 데이터 요청을 시작할 수 있습니다! useQuery는 @ apollo / react-hooks에서 내 보낸 후크로 Hooks API를 활용하여 UI와 GraphQL 데이터를 공유합니다.

먼저 gql 함수로 래핑 된 GraphQL 쿼리를 useQuery 후크에 전달하십시오. 컴포넌트가 렌더링되고 useQuery 후크가 실행되면로드, 오류 및 데이터 특성이 포함 된 결과 오브젝트가 리턴됩니다. Apollo Client는 오류 및로드 상태를 추적하여로드 및 오류 속성에 반영됩니다. 쿼리 결과가 다시 나타나면 데이터 속성에 첨부됩니다.

useQuery 후크가 작동하는지 확인하기 위해 index.js에 ExchangeRates 구성 요소를 만들어 봅니다.

```text
import React from 'react';
import { useQuery } from '@apollo/react-hooks';
import { gql } from 'apollo-boost';

const EXCHANGE_RATES = gql`
  {
    rates(currency: "USD") {
      currency
      rate
    }
  }
`;

function ExchangeRates() {
  const { loading, error, data } = useQuery(EXCHANGE_RATES);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  return data.rates.map(({ currency, rate }) => (
    <div key={currency}>
      <p>
        {currency}: {rate}
      </p>
    </div>
  ));
}
```

축하합니다. 첫 번째 useQuery 기반 구성 요소를 만들었습니다. 🎉 이전 예제의 App 구성 요소 내에서 ExchangeRates 구성 요소를 렌더링하면 먼저로드 표시기가 표시되고 준비가되면 페이지에 데이터가 표시됩니다. 서버에서 돌아올 때 Apollo Client는이 데이터를 자동으로 캐시하므로 동일한 쿼리를 두 번 실행하면 로딩 표시기가 나타나지 않습니다.

방금 구축 한 앱으로 게임을 즐기고 싶다면 CodeSandbox에서 볼 수 있습니다. 거기서 멈추지 마십시오! useQuery를 활용하는 더 많은 컴포넌트를 구축하고 방금 배운 개념을 실험 해보십시오.

더 자세히 알아 보려면 다음과 같은 다양한 프론트 엔드 라이브러리를 갖춘 예제 앱 버전이 있습니다.

* React Native Web: [https://codesandbox.io/s/xk7zw3n4](https://codesandbox.io/s/xk7zw3n4)
* Angular \(Ionic\): [https://github.com/aaronksaunders/ionicLaunchpadApp](https://github.com/aaronksaunders/ionicLaunchpadApp)

### Apollo Boost <a id="apollo-boost"></a>

예제 앱에서 Apollo Boost를 사용하여 Apollo Client를 신속하게 설정했습니다. GraphQL 서버 엔드 포인트는 시작하는 데 필요한 유일한 구성 옵션이지만 로컬 상태 관리, 인증 및 오류 처리와 같은 기능을 빠르게 구현할 수 있도록 몇 가지 다른 옵션이 포함되어 있습니다.

#### 포함된 내 <a id="whats-included"></a>

Apollo Boost에는 Apollo Client로 개발하는 데 필수적이라고 생각되는 일부 패키지가 포함되어 있습니다. 포함 된 내용은 다음과 같습니다.

* `apollo-client`: 모든 마술이 일어나는 곳
* `apollo-cache-inmemory`: 권장 캐시
* `apollo-link-http`: 원격 데이터 가져 오기를위한 Apollo Link
* `apollo-link-error`: 오류 처리를위한 아폴로 링크

아폴로 부스트의 가장 큰 장점은 직접 설정할 필요가 없다는 것입니다! 이 기능을 사용하려면 몇 가지 옵션 만 지정하면 나머지는 처리해 드리겠습니다.

#### Configuration options\(구성 옵션\) <a id="configuration-options"></a>

다음은 apollo-boost에서 내 보낸 ApolloClient에 전달할 수있는 옵션입니다. 그들 모두는 선택 사항입니다.

> 링크 :[https://www.apollographql.com/docs/react/get-started/](https://www.apollographql.com/docs/react/get-started/)



