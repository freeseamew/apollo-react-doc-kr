---
description: 데이터를 관리하기 위해 Apollo Client를 선택해야 하는 이유
---

# 왜 Apollo Client 인가?

데이터 관리가 그렇게 어려울 필요는 없다! React 애플리케이션에서 원격 및 로컬 데이터 관리를 간소화하는 방법을 알고 싶다면 올바른 위치에 도달하면 된다. 우리의 예제 앱인 Pupstagram에서 영감을 받은 실용적인 예를 통해, 당신은 어떻게 Apollo의 지능형 캐싱과 데이터 페어링에 대한 선언적 접근법이 어떻게 당신이 코드를 덜 쓰면서 더 빨리 반복하도록 도울 수 있는지 알게 될 것이다.

## Declarative data fetching\(데이터 가져오기\)

데이터 가져오기\(data fetching\)에 대한 아폴로의 선언적 접근방식으로 데이터를 검색하고, 로딩 및 오류 상태를 추적하고, UI를 업데이트하는 모든 논리가 useQuery Hook에 의해 캡슐화됩니다. 이 캡슐화를 통해 쿼리 결과를 현재의 구성요소에 통합하는 것은 쉬운 일이 될 수 있습니다. 리액트 아폴로와 실제로 어떤 모습인지 알아보도록 하겠습니.

```javascript
function Feed() {
  const { loading, error, data } = useQuery(GET_DOGS);
  if (error) return <Error />;
  if (loading || !data) return <Fetching />;

  return <DogList dogs={data.dogs} />;
}
```

여기서는 useQuery Hook를 사용하여 GraphQL 서버에서 일부 개를 가져 와서 목록에 표시합니다. `useQuery`는 React의 Hooks API를 활용하여 쿼리를 컴포넌트에 바인딩하고 쿼리 결과에 따라 렌더링합니다. 데이터가 다시 반환되면 `<DogList>` 구성 요소가 필요한 데이터에 따라 반응 적으로 업데이트됩니다.

Apollo Client는로드 및 오류 상태 추적을 포함하여 요청주기를 처음부터 끝까지 처리합니다. 첫 번째 요청을하기 전에 작성할 미들웨어 나 상용구가 없으며 응답을 변환하고 캐싱하는 것에 대해 걱정할 필요가 없습니다. 구성 요소에 필요한 데이터를 설명하고 Apollo 클라이언트가 무거운 작업을 수행하도록하기 만하면됩니다.

Apollo Client로 전환하면 데이터 관리와 관련된 많은 불필요한 코드를 삭제할 수 있습니다. 정확한 금액은 응용 프로그램에 따라 다르지만 일부 팀은 최대 수천 줄을보고했습니다. Apollo로 더 적은 코드를 작성한다고해서 기능을 타협해야하는 것은 아닙니다. optimistic UI, refetching 및 pagination 같은 고급 기능은 useQuery 옵션을 통해 쉽게 액세스 할 수 있습니다.

## Zero-config caching

Apollo Client를 다른 데이터 관리 솔루션과 차별화하는 주요 기능 중 하나는 정규화 된 캐시입니다. Apollo Client를 설정하기 만하면 추가 구성없이 지능적으로 캐시를 얻을 수 있습니다. Pupstagram 예제 앱의 홈 페이지에서 개 중 하나를 클릭하여 세부 사항 페이지를보십시오. 그런 다음 홈페이지로 돌아갑니다. Apollo 캐시 덕분에 홈페이지의 이미지가 즉시로드됩니다.

```javascript
import ApolloClient from 'apollo-boost';

// the Apollo cache is set up automatically
const client = new ApolloClient();
```

그래프 캐싱은 쉬운 일이 아니지만 2 년 동안 그래프를 해결하는 데 집중했습니다. 동일한 데이터로 이어지는 여러 경로를 가질 수 있으므로 정규화는 여러 구성 요소에서 데이터의 일관성을 유지하는 데 필수적입니다. 실용적인 예를 살펴 보겠습니다.

```javascript
const GET_ALL_DOGS = gql`
  query {
    dogs {
      id
      breed
      displayImage
    }
  }
`;

const UPDATE_DISPLAY_IMAGE = gql`
  mutation UpdateDisplayImage($id: String!, $displayImage: String!) {
    updateDisplayImage(id: $id, displayImage: $displayImage) {
      id
      displayImage
    }
  }
`;
```

`GET_ALL_DOGS` 쿼리는 개의 목록과 해당 `displayImage`를 가져옵니다. 돌연변이 `UPDATE_DISPLAY_IMAGE`는 개의 `displayImage`를 업데이트합니다. 특정 개에서 `displayImage`를 업데이트하는 경우 새 데이터를 반영하기 위해 모든 개 목록에서 해당 항목이 필요합니다. Apollo Client는 `__typename` 및 `id` 속성을 사용하여 GraphQL 결과의 각 객체를 Apollo 캐시의 자체 항목으로 분리합니다. 이는 id를 가진 변이에서 값을 반환하면 동일한 id를 가진 객체를 가져 오는 모든 쿼리를 자동으로 업데이트합니다. 또한 동일한 데이터를 반환하는 두 개의 쿼리가 항상 동기화되도록합니다.

dog에 대한 쿼리는 다음과 같습니다.

```graphql
const GET_DOG = gql`
  query {
    dog(id: "abc") {
      id
      breed
      displayImage
    }
  }
`;
```

다음은 캐시 재 지정입니다. `apollo-boost` 클라이언트의 `cacheRedirects` 속성에 맵을 지정하여 쉽게 설정할 수 있습니다. 캐시 리디렉션은 쿼리가 캐시에서 데이터를 조회하는 데 사용할 수있는 키를 반환합니다.

```javascript
import ApolloClient from 'apollo-boost';

const client = new ApolloClient({
  cacheRedirects: {
    Query: {
      dog: (_, { id }, { getCacheKey }) => getCacheKey({ id, __typename: 'Dog' })
    }
  }
})
```

## Combine local & remote data

수천 명의 개발자가 Apollo Client가 원격 데이터를 관리하는 데 뛰어나며 이는 데이터 요구의 약 80 %에 해당한다고 말합니다. 그러나 파이의 다른 20 %를 구성하는 로컬 데이터 \(글로벌 플래그 및 장치 API 결과 등\)는 어떻습니까? Apollo Client에는 기본적으로 local state management\(로컬 상태 관리\) 기능이 포함되어있어 Apollo 캐시를 애플리케이션의 데이터에 대한 단일 진실 소스로 사용할 수 있습니다.

Apollo Client로 모든 데이터를 관리하면 GraphQL을 모든 데이터에 대한 통합 인터페이스로 활용할 수 있습니다. 이를 통해 GraphiQL을 통해 Apollo DevTools에서 로컬 및 원격 스키마를 모두 검사 할 수 있습니다.

```javascript
const GET_DOG = gql`
  query GetDogByBreed($breed: String!) {
    dog(breed: $breed) {
      images {
        url
        id
        isLiked @client
      }
    }
  }
`;
```

Apollo Client의 로컬 상태 기능을 활용하여 클라이언트 측 전용 필드를 원격 데이터에 완벽하게 추가하고 구성 요소에서 쿼리 할 수 있습니다. 이 예에서는 서버 데이터와 함께 클라이언트 전용 필드 isLiked를 쿼리합니다. 컴포넌트는 로컬 및 원격 데이터로 구성되어 있으므로 이제 쿼리도 가능합니다.

## 풍부한 생태

Apollo Client는 시작하기 쉽지만 고급 기능을 구축해야 할 때 확장 할 수 있습니다. 앱별 미들웨어 또는 캐시 지속성과 같이 apollo-boost가 적용되지 않는 사용자 지정 기능이 필요한 경우 Apollo 캐시를 연결하고 네트워크 스택을 Apollo Link와 함께 연결하여 고유 한 클라이언트를 만들 수 있습니다.

이러한 유연성 덕분에 Apollo 위에 확장을 구축하여 드림 클라이언트를 간단하게 만들 수 있습니다. 우리는 항상 기여자들이 아폴로를 기반으로 구축 한 것에 깊은 감명을 받았습니다. 패키지 중 일부를 확인하십시오.

* Apollo Link 커뮤니티 링크 : 커뮤니티가 생성 한 플러그 가능한 링크
* apollo-cache-persist : 아폴로 캐시에 대한 간단한 지속성 \(@jamesreggio\)
* apollo-storybook-decorator : Apollo Client로 React Storybook 스토리를 감 쌉니다 \(@ abhiaiyer91\)
* AWS의 AppSync : Amazon의 실시간 GraphQL 클라이언트는 Apollo Client를 사용합니다

데이터를 관리하기 위해 Apollo를 선택하면 놀라운 커뮤니티의 지원을받을 수 있습니다. Apollo Spectrum 커뮤니티에는 아이디어를 공유 할 수있는 수천 명의 개발자가 있습니다. 또한 자주 업데이트되는 모범 사례 및 Apollo 블로그 공지 사항에 대한 기사를 읽을 수 있습니다.

## Case studies

기업에서 신생 기업에 이르기까지 다양한 회사는 Apollo Client를 신뢰하여 가장 중요한 웹 및 기본 응용 프로그램을 구동합니다. GraphQL 및 Apollo 로의 전환이 엔지니어의 워크 플로우를 단순화하고 제품을 개선하는 방법에 대해 자세히 알아 보려면 다음 사례 연구를 확인하십시오.

* The New York Times : New York Times가 Relay에서 Apollo로 전환하고 SSR 및 지속 쿼리와 같은 앱의 기능을 구현 한 방법
* Express : Apollo를 사용한 사용하기 쉬운 페이지 매김으로 Express eCommerce 팀의 주요 제품 페이지 개선
* 메이저 리그 축구 : 상태 관리를 위해 MLS의 Redux에서 Apollo로 전환하여 거의 모든 Redux 코드를 삭제할 수있었습니다.
* Expo : Apollo로 React Native 앱을 개발함으로써 Expo 엔지니어들은 데이터 가져 오기 로직을 작성하는 대신 제품 개선에 집중할 수있었습니다.
* KLM : KLM 팀이 GraphQL 및 Apollo를 사용하여 Angular 앱을 어떻게 확장했는지 알아보십시오

회사에서 프로덕션 환경에서 Apollo Client를 사용하는 경우 블로그에 사례 연구를 소개하고자합니다. Apollo 사용 방법에 대해 자세히 알아볼 수 있도록 Spectrum을 통해 연락하십시오. 또는 블로그 게시물이나 컨퍼런스 토크가 이미 있으시면 여기로 보내 주시기 바랍니다.

