# Configuring the cache\(캐시 설정\): 2.6

Apollo Client는 정규화 된 인 메모리 캐시를 사용하여 실시간 데이터에 의존하지 않는 쿼리의 실행 속도를 획기적으로 향상시킵니다. 이 기사에서는 캐시 설정 및 구성에 대해 설명합니다.

### Installation <a id="installation"></a>

InMemoryCache 클래스는 Apollo Client 코어와 다른 패키지에 있습니다. 프로젝트에 apollo-cache-inmemory 패키지가 설치되어 있는지 확인하십시오 :

```text
npm install apollo-cache-inmemory --save
```

### Initializing the cache\(캐시 초기화\) <a id="initializing-the-cache"></a>

```javascript
import { InMemoryCache } from 'apollo-cache-inmemory';
import { HttpLink } from 'apollo-link-http';
import { ApolloClient } from 'apollo-client';

const client = new ApolloClient({
  link: new HttpLink(),
  cache: new InMemoryCache()
});
```

InMemoryCache 생성자는 캐시 구성에 설명 된 다양한 옵션을 허용합니다.

### Configuring the cache\(캐시 설정\) <a id="configuring-the-cache"></a>

InMemoryCache 생성자에 구성 개체를 제공하여 동작을 사용자 지정할 수 있습니다. 이 객체는 다음 필드를 지원합니다.

| NAME | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `addTypename` | boolean | \_\_typename을 문서에 추가할지 여부를 나타냅니다 \(기본값 : true\). |
| `dataIdFromObject` | function | 데이터 오브젝트를 가져오고 상점에서 데이터를 정규화 할 때 사용할 고유 ID를 리턴하는 함수입니다. 맞춤 식별자에서 dataIdFromObject를 맞춤 설정하는 방법에 대해 자세히 알아보세요. |
| `fragmentMatcher` | object | 기본적으로 InMemoryCache는 휴리스틱 조각 매처를 사용합니다. 공용체와 인터페이스에서 조각을 사용하는 경우 IntrospectionFragmentMatcher를 사용해야합니다. 자세한 내용은 공용체 및 인터페이스에 조각 일치 설정 가이드를 참조하십시오. |
| `cacheRedirects` | object | 요청이 발생하기 전에 캐시의 다른 항목으로 쿼리를 리디렉션하는 함수 맵. 항목 목록이 있고 개별 항목을 쿼리하는 세부 정보 페이지에서 목록 쿼리의 데이터를 사용하려는 경우에 유용합니다. 여기에 더 자세한 내용이 있습니다. |

### Data normalization\(데이터 정규화\) <a id="data-normalization"></a>

InMemoryCache는 다음을 통해 쿼리 결과를 캐시에 저장하기 전에 쿼리 결과를 정규화합니다.

1.결과를 개별 객체로 나누기 

2.각 객체에 고유 식별자 할당 

3.평평한 데이터 구조로 객체 저장

#### Assigning unique identifiers\(고유 식별자 할당\) <a id="assigning-unique-identifiers"></a>

**Default identifiers\(기본 식별자\)**

기본적으로 InMemoryCache는 객체의 \_\_typename 필드와 해당 id 또는 \_id 필드를 결합하여 객체의 고유 식별자를 생성하려고 시도합니다.

객체가 \_\_typename 또는 id 또는 \_id 중 하나를 지정하지 않으면 InMemoryCache는 연결된 쿼리 내에서 객체의 경로를 사용하는 것으로 돌아갑니다 \(예 : allPeople 루트 쿼리에 대해 반환 된 첫 번째 레코드의 경우 ROOT\_QUERY.allPeople.0\). 캐시 된 개체를 개별 쿼리로 범위를 지정하므로 가능할 때마다이 대체 전략을 사용하지 마십시오. 이는 여러 쿼리가 모두 동일한 개체를 반환하는 경우 각 쿼리는 해당 개체의 개별 인스턴스를 비효율적으로 캐시합니다.

> 경고 : 캐시하는 각 객체 유형은 항상 id 필드를 포함하거나 id 필드를 포함하지 않아야합니다. 특정 유형에 대해이 필드가 있거나 없을 때 불일치가 발생하면 InMemoryCache에서 오류가 발생합니다.

**Custom identifiers\(맞춤 식별자\)**

캐시 된 개체의 고유 식별자를 생성하기위한 사용자 지정 전략을 정의 할 수 있습니다. 그렇게하려면 dataIdFromObject 구성 옵션을 InMemoryCache 생성자에 제공하십시오. 이 옵션은 객체를 가져 와서 해당 객체의 고유 식별자를 반환하는 함수입니다.

예를 들어, 객체 유형이 모두 고유 식별자로 사용하려는 키 필드를 정의하는 경우 다음과 같이 dataIdFromObject를 정의 할 수 있습니다.

```javascript
const cache = new InMemoryCache({
  dataIdFromObject: object => object.key || null
});
```

InMemoryCache는 dataIdFromObject가 반환하는 정확한 문자열을 사용합니다. 고유 식별자에 객체의 \_\_typename 필드를 포함 시키려면 함수의 로직의 일부로 포함해야합니다.

다음과 같이 객체의 \_\_typename 속성을 키 오프하여 서로 다른 논리를 사용하여 각 객체 유형에 대한 고유 식별자를 생성 할 수 있습니다.

```javascript
import { InMemoryCache, defaultDataIdFromObject } from 'apollo-cache-inmemory';

const cache = new InMemoryCache({
  dataIdFromObject: object => {
    switch (object.__typename) {
      case 'foo': return object.key; // use the `key` field as the identifier
      case 'bar': return `bar:${object.blah}`; // append `bar` to the `blah` field as the identifier
      default: return defaultDataIdFromObject(object); // fall back to default handling
    }
  }
});
```

### Automatic cache updates\(자동 캐시 업데이트\) <a id="automatic-cache-updates"></a>

캐시 정규화를 사용하여 저장소를 올바르게 업데이트하는 경우를 살펴 보겠습니다. 다음 쿼리를 수행한다고 가정 해 봅시다.

```graphql
{
  post(id: '5') {
    id
    score
  }
}
```

그런 다음 다음과 같은 변이를 수행합니다.

```graphql
mutation {
  upvotePost(id: '5') {
    id
    score
  }
}
```

두 결과의 id 필드가 일치하면 UI의 모든 위치에있는 점수 필드가 자동으로 업데이트됩니다! 이 속성을 최대한 활용하는 좋은 방법 중 하나는 돌연변이 결과에 이전에 가져온 쿼리를 업데이트하는 데 필요한 모든 데이터를 갖도록하는 것입니다. 이를위한 간단한 트릭은 조각을 사용하여 쿼리와 영향을받는 돌연변이간에 필드를 공유하는 것입니다.

