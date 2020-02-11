# Configuring the cache\(캐시 설정\) : 3.0

Apollo Client는 GraphQL 쿼리 결과를 표준화 된 인 메모리 캐시에 저장합니다. 이를 통해 클라이언트는 불필요한 네트워크 요청을 보내지 않고도 동일한 데이터에 대한 향후 쿼리에 응답 할 수 있습니다.

> 이 문서에서는 캐시 설정 및 구성에 대해 설명합니다. 캐시 된 데이터와 상호 작용하는 방법을 알아 보려면 캐시 된 데이터와 상호 작용을 참조하십시오.

### Installation <a id="installation"></a>

Apollo Client 3.0부터 InMemoryCache 클래스는 @ apollo / client 패키지에 의해 제공됩니다. npm install @ apollo / client를 실행 한 후에는 더 이상 별도의 패키지를 설치할 필요가 없습니다.

### Initializing the cache\(캐시 초기화\) <a id="initializing-the-cache"></a>

다음과 같이 InMemoryCache 객체를 만들어 ApolloClient 생성자에 제공하십시오.

```javascript
import { InMemoryCache, HttpLink, ApolloClient } from '@apollo/client';

const client = new ApolloClient({
  link: new HttpLink(),
  cache: new InMemoryCache(options)
});
```

InMemoryCache 생성자는 아래에 설명 된 다양한 명명 된 옵션을 허용합니다.

### Configuring the cache\(캐시 구성\) <a id="configuring-the-cache"></a>

캐시의 기본 동작은 다양한 응용 프로그램에 적합하지만 특정 사용 사례에 더 적합하도록 동작을 구성 할 수 있습니다. 특히 다음을 수행 할 수 있습니다.

* 사용자 정의 기본 키 필드 지정
* 필드 값의 저장 및 검색 사용자 정의
* 필드 인수의 해석을 사용자 정의
* 프래그먼트 매칭을위한 수퍼 타입-서브 타입 관계 정의
* 페이지 매김 패턴 정의
* 클라이언트 측 로컬 상태 관리

캐시 동작을 사용자 지정하려면 옵션 개체를 InMemoryCache 생성자에 제공하십시오. 이 객체는 다음 필드를 지원합니다.



| NAME | TYPE | DESCRIPTION |
| :--- | :--- | :--- |
| `addTypename` | boolean | true 인 경우 캐시는 모든 발신 쿼리에 \_\_typename 필드를 자동으로 추가하므로 수동으로 추가 할 필요가 없습니다. \(기본값 : true\) |
| `resultCaching` | boolean | true이면 기본 데이터가 변경되지 않은 한 캐시는 동일한 쿼리를 실행할 때마다 동일한 \(===\) 응답 객체를 반환합니다. 따라서 쿼리 결과의 변경 사항을보다 쉽게 감지 할 수 있습니다. \(기본값 : true\) |
| `possibleTypes` | `{ [supertype: string]: string[] }` | 스키마 유형 간의 다형성 관계를 정의하려면이 오브젝트를 포함하십시오. 이렇게하면 인터페이스 또는 통합별로 캐시 된 데이터를 조회 할 수 있습니다. 각 항목의 키는 인터페이스 또는 공용체의 **typename이고 값은 해당 공용체에 속하거나 해당 인터페이스를 구현하는 유형의** typename 배열입니다. |
| `typePolicies` | `{ [typename: string]: TypePolicy }` | 유형별로 캐시 동작을 사용자 정의하려면이 오브젝트를 포함하십시오. 각 항목의 키는 유형의 \_\_typename입니다. 자세한 내용은 TypePolicy 유형을 참조하십시오. |
| `dataIdFromObject` **\(deprecated\)** | function | 응답 오브젝트를 가져 와서 상점의 데이터를 정규화 할 때 사용할 고유 ID를 리턴하는 함수입니다. TypePolicy 개체의 keyFields 옵션을 위해 사용되지 않습니다. |

### Data normalization\(데이터 정규화\) <a id="data-normalization"></a>

InMemoryCache는 쿼리 응답 객체를 내부 데이터 저장소에 저장하기 전에 표준화합니다. 정규화에는 다음 단계가 포함됩니다.

* 캐시는 응답에 포함 된 모든 식별 가능한 객체에 대해 고유 한 ID를 생성합니다.
* 캐시는 플랫 룩업 테이블에 ID별로 오브젝트를 저장합니다.
* 객체가 기존 객체와 동일한 ID로 저장 될 때마다 해당 객체의 필드가 병합됩니다. 새 객체는 두 필드 모두에 나타나는 모든 필드의 값을 덮어 씁니다.

정규화는 응용 프로그램 상태가 변경 될 때 그래프를 읽고 업데이트하기에 최적화 된 형식으로 클라이언트에 데이터 그래프의 부분 복사본을 구성합니다.

#### Generating unique identifiers\(고유 식별자 생성\) <a id="generating-unique-identifiers"></a>

Apollo Client 3 이상에서 InMemoryCache는 식별자 생성에 실패하거나 비활성화 된 경우 개체에 대한 대체 "가짜"식별자를 생성하지 않습니다.

**Default identifier generation\(기본 색별자 생성\)**

기본적으로 InMemoryCache는 개체의 **typename을 해당 id 또는 \_id 필드 \(둘 중 정의 된 필드\)와 결합하여** typename 필드를 포함하는 모든 개체에 대한 고유 식별자를 생성합니다. 이 두 값은 콜론 \(:\)으로 구분됩니다.

예를 들어 \_\_typename이 Task이고 id가 14 인 객체에는 기본 식별자 Task : 14가 할당됩니다.

**Customizing identifier generation by type\(유형별로 식별 생성자 저의\)**

유형 중 하나가 id 또는 \_id 이외의 필드로 기본 키를 정의하는 경우 유형에 대해 TypePolicy를 정의하여 InMemoryCache가 고유 식별자를 생성하는 방법을 사용자 정의 할 수 있습니다. InMemoryCache 생성자에 제공하는 옵션 객체에 모든 캐시의 typePolicies를 지정합니다.

다음과 같이 관련 TypePolicy 객체에 keyFields 필드를 포함하십시오.

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    Product: {
      // In most inventory management systems, a single UPC code uniquely
      // identifies any product.
      keyFields: ["upc"],
    },
    Person: {
      // In some user account systems, names or emails alone do not have to
      // be unique, but the combination of a person's name and email is
      // uniquely identifying.
      keyFields: ["name", "email"],
    },
    Book: {
      // If one of the keyFields is an object with fields of its own, you can
      // include those nested keyFields by using a nested array of strings:
      keyFields: ["title", "author", ["name"]],
    },
  },
});
```

이 예는 세 가지 typePolicies를 보여줍니다. 하나는 Product type, 하나는 Person type, 다른 하나는 Book type입니다. 각 TypePolicy의 keyFields 배열은 형식의 기본 필드를 나타내는 형식의 필드를 정의합니다.

Book 유형은 기본 키의 일부로 서브 필드를 사용합니다. \[ "name"\] 항목은 배열 \(author\)에서 이전 필드의 이름 필드가 기본 키의 일부임을 나타냅니다. 책의 저자 필드는 이것이 유효하기 위해 이름 필드를 포함하는 객체 여야합니다.

위의 예에서 Book 객체의 고유 식별자 문자열은 다음과 같은 형식입니다.

```text
Book:{"title":"Fahrenheit 451","author":{"name":"Ray Bradbury"}}
```

고유성을 보장하기 위해 객체의 기본 키 필드는 항상 동일한 순서로 나열됩니다.

이러한 keyFields 문자열은 항상 스키마에 정의 된 실제 필드 이름을 참조하므로 ID 계산은 필드 별명에 민감하지 않습니다. 이 필드는 keyField를 구현하기 위해 함수를 사용하려는 경우 중요합니다.

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      keyFields(responseObject, { typename, selectionSet, fragmentMap }) {
        let id: string | null = null;
        selectionSet.selections.some(selection => {
          if (selection.kind === 'Field') {
            // If you fail to take aliasing into account, your custom
            // normalization is likely to break whenever a query contains
            // an alias for key field.
            const actualFieldName = selection.name.value;
            const responseFieldName = (selection.alias || selection.name).value;
            if (actualFieldName === 'socialSecurityNumber') {
              id = `${typename}:${responseObject[responseFieldName]}`;
              return true;
            }
          } else {
            // Handle fragments using the fragmentMap...
          }
          return false;
        });
        return id;
      },
    },
  },
});
```

이 경우가 모호한 경우, 자신의 keyFields 함수를 구현하지 말고 대신 keyFields에 대한 문자열 배열을 전달하여 이러한 미묘한 버그에 대해 걱정할 필요가 없습니다. 일반적으로 typePolicies API를 사용하면 처음 캐시를 작성할 때 한 곳에서 정규화 동작을 구성 할 수 있으며 필드 별명 또는 지시문을 사용하여 쿼리를 다르게 작성할 필요가 없습니다.

전 세계적으로 식별자 생성 사용자 정의 특정 \_\_typename과 관련이없는 단일 폴백 키 필드 함수를 정의해야하는 경우 Apollo Client 2.x의 이전 dataIdFromObject 함수가 계속 지원됩니다.

```javascript
import { defaultDataIdFromObject } from '@apollo/client';

const cache = new InMemoryCache({
  dataIdFromObject(responseObject) {
    switch (object.__typename) {
      case 'Product': return `Product:${object.upc}`;
      case 'Person': return `Person:${object.name}:${object.email}`;
      default: return defaultDataIdFromObject(object);
    }
  }
});
```

어쨌든이 함수가 특정 object .\_\_ typename 문자열을 기반으로 다른 키를 선택해야하므로 typePolicies를 통해 제품 및 개인 유형에 대해 keyFields 배열을 사용했을 수도 있습니다. 또한이 코드는 앨리어싱 실수에 민감하며 정의되지 않은 객체 속성으로부터 보호 할 수 없으며 실수로 다른 시간에 다른 키 필드를 사용하면 캐시에 불일치가 발생할 수 있습니다.

dataIdFromObject API는 Apollo Client 2.x에서 3.0으로 쉽게 전환 할 수 있도록 만들어졌으며 이후 버전의 @ apollo / client에서 제거 될 수 있습니다.

#### Disabling normalization**\(정규화 비활성화\)** <a id="disabling-normalization"></a>

InMemoryCache가 특정 유형의 객체를 정규화하지 않도록 지시 할 수 있습니다. 이는 타임 스탬프로 식별되고 업데이트를받지 않는 메트릭 및 기타 임시 데이터에 적합합니다.

유형에 대한 정규화를 비활성화하려면 유형별로 식별자 생성 사용자 지정에 표시된대로 유형에 대한 TypePolicy를 정의하되 정책의 keyFields 필드를 false로 설정하십시오.

정규화되지 않은 객체는 대신 캐시의 상위 객체 내에 포함됩니다. 이러한 개체에 직접 액세스 할 수 없으며 대신 부모를 통해 액세스해야합니다.

### The `TypePolicy` type <a id="the-typepolicy-type"></a>

캐시가 스키마의 특정 유형과 상호 작용하는 방식을 사용자 정의하기 위해 새 InMemoryCache 객체를 만들 때 \_\_typename 문자열을 TypePolicy 객체에 매핑하는 객체를 제공 할 수 있습니다.

TypePolicy 개체는 다음 필드를 포함 할 수 있습니다.

```javascript
type TypePolicy = {
  // Allows defining the primary key fields for this type, either using an
  // array of field names or a function that returns an arbitrary string.
  keyFields?: KeySpecifier | KeyFieldsFunction | false;

  // If your schema uses a custom __typename for any of the root Query,
  // Mutation, and/or Subscription types (rare), set the corresponding
  // field below to true to indicate that this type serves as that type.
  queryType?: true,
  mutationType?: true,
  subscriptionType?: true,

  fields?: {
    [fieldName: string]:
      | FieldPolicy<StoreValue>
      | FieldReadFunction<StoreValue>;
  }
};

// Recursive type aliases are coming in TypeScript 3.7, so this isn't the
// actual type we use, but it's what it should be:
type KeySpecifier = (string | KeySpecifier)[];

type KeyFieldsFunction = (
  object: Readonly<StoreObject>,
  context: {
    typename: string;
    selectionSet?: SelectionSetNode;
    fragmentMap?: FragmentMap;
  },
) => string | null | void;
```

#### Overriding root operation types \(uncommon\)**\(**루트 작업 유형 재정의 \(드문 경우\)\) <a id="overriding-root-operation-types-uncommon"></a>

keyField 외에도 TypePolicy는 queryType, mutationType 또는 subscriptionType을 true로 설정하여 루트 쿼리, 돌연변이 또는 가입 유형을 나타내는 것을 나타낼 수 있습니다.

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    UnconventionalRootQuery: {
      // The RootQueryFragment can only match if the cache knows the __typename
      // of the root query object.
      queryType: true,
    },
  },
});

const result = cache.readQuery({
  query: gql`
    query {
      ...RootQueryFragment
    }
    fragment RootQueryFragment on UnconventionalRootQuery {
      field1
      field2 {
        subfield
      }
    }
  `,
});

const equivalentResult = cache.readQuery({
  query: gql`
    query {
      field1
      field2 {
        subfield
      }
    }
  `,
});
```

캐시는 일반적으로 서버에 전송하는 모든 쿼리 선택 세트에 **typename 필드를 추가하여** typename 정보를 얻습니다. 기술적으로 모든 작업의 가장 바깥 쪽 선택 세트에 대해 동일한 트릭을 사용할 수 있지만 루트 쿼리 또는 돌연변이의 \_\_typename은 거의 항상 "Query"또는 "Mutation"이므로 캐시는 TypePolicy에서 달리 지시하지 않는 한 이러한 공통 기본값을 가정합니다. .

적절한 식별 및 정규화에 꼭 필요한 Books 또는 Persons와 같은 엔터티 개체의 **typename과 비교할 때 루트 쿼리 또는 돌연변이 유형의** typename은 그다지 유용하지 않거나 중요하지 않습니다. 이러한 유형은 클라이언트 당 하나의 인스턴스 만있는 싱글 톤이기 때문입니다. .

#### The `fields` property\(필드 속성\) <a id="the-fields-property"></a>

TypePolicy 내의 최종 속성은 fields 속성으로, 문자열 필드 이름에서 FieldPolicy 객체로의 맵입니다. 다음 섹션에서는 필드 정책에 대해 자세히 설명합니다.

### Field policies\(현장 정책\) <a id="field-policies"></a>

Typetype-having 엔티티 객체의 식별 및 정규화를 구성하는 것 외에도 TypePolicy는 해당 유형이 지원하는 모든 필드에 대한 정책을 제공 할 수 있습니다.

FieldPolicy 유형 및 관련 유형은 다음과 같습니다.

```javascript
export type FieldPolicy<TValue> = {
  keyArgs?: KeySpecifier | KeyArgsFunction;
  read?: FieldReadFunction<TValue>;
  merge?: FieldMergeFunction<TValue>;
};

type KeyArgsFunction = (
  field: FieldNode,
  context: {
    typename: string;
    variables: Record<string, any>;
  },
) => string | null | void;

type FieldReadFunction<TExisting, TResult = TExisting> = (
  existing: Readonly<TExisting> | undefined,
  options: FieldFunctionOptions,
) => TResult;

type FieldMergeFunction<TExisting> = (
  existing: Readonly<TExisting> | undefined,
  incoming: Readonly<StoreValue>,
  options: FieldFunctionOptions,
) => TExisting;

interface FieldFunctionOptions {
  args: Record<string, any>;
  parentObject: Readonly<StoreObject>;
  field: FieldNode;
  variables?: Record<string, any>;
}
```

아래 섹션에서는 이러한 유형을 설명과 예제로 분류합니다.

#### Key arguments <a id="key-arguments"></a>

TypePolicy 객체의 keyFields 속성과 유사하게 FieldPolicy 객체의 keyArgs 속성은 필드에 전달 된 인수가 값을 둘러싸고있는 엔터티 개체와 함께 필드 값을 결정한다는 의미에서 "중요한"캐시를 알려줍니다.

기본적으로 캐시는 모든 필드 인수가 중요 할 것으로 가정하므로 해당 필드에 대해 수신 한 인수 값의 고유 한 각 조합에 대해 별도의 필드 값을 저장합니다. 이는 인수에 차이가있는 경우 필드 값이 서로 충돌하지 않도록하는 안전한 정책입니다. 그러나이 정책으로 인해 필드 값의 불필요한 사본이 발생할 수 있으며 필드의 인수가 약간 다른 경우에도 필드에서 동일한 논리 값을 공유 할 수있는 기회가 누락 될 수 있습니다.

예를 들어, 주어진 키에 따라 비밀 값을 반환하지만 요청을 인증하기 위해 액세스 토큰이 필요한 필드가 있다고 가정합니다.

```javascript
query GetSecret {
  secret(key: $secretKey, token: $secretAccessToken) {
    message
  }
}
```

유효한 액세스 토큰이 있으면이 필드의 값은 키에만 의존합니다. 다시 말해, 다른 \(유효한\) 토큰을 사용했다고해서 다른 비밀 메시지를받지 못할 것입니다.

이와 같은 경우 액세스 토큰 팩터를 필드 값의 스토리지에 저장하는 것은 낭비적이고 일관성이 없으므로, 필드 정책의 keyArgs 옵션을 사용하여 키만 "중요"하다는 것을 캐시에 알려야합니다. 비밀 필드 :

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        secret: {
          keyArgs: ["key"],
        },
      },
    },
  },
});
```

즉, 토큰이 항상 동일하다고 가정하거나 캐시에서 필드 값 복제에 대해 걱정하지 않아도되므로 keyArgs : \[ "key"\]를 지정하지 않으면 큰 문제가 발생하지 않습니다. 도움이 될 때 keyArgs를 사용하십시오.

반면에, 액세스 토큰을 사용하여 서버에서 비밀을 요청했지만 토큰을 몰라도 페이지의 다양한 구성 요소가 키만 사용하여 비밀에 액세스 할 수 있기를 원할 것입니다. 키만 사용하여 캐시에 값을 저장하면이 검색이 가능합니다.

