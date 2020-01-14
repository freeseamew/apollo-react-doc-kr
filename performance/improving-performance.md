# Improving performance\(성능향상\)

### Redirecting to cached data\(캐시된 데이터로의 redirecting\) <a id="redirecting-to-cached-data"></a>

경우에 따라 쿼리는 클라이언트 저장소에 이미 존재하는 데이터를 다른 키로 요청합니다. 가장 일반적인 예는 UI에 동일한 데이터를 사용하는 목록보기 및 세부 사항보기가 있는 경우입니다. 목록보기는 다음 쿼리를 실행할 수 있습니다.

```text
query ListView {
  books {
    id
    title
    abstract
  }
}
```

특정 책을 선택하면 상세 조회에이 쿼리를 사용하여 개별 항목이 표시됩니다

```text
query DetailView {
  book(id: $id) {
    id
    title
    abstract
  }
}
```

> 참고 : 목록 쿼리에서 반환 된 데이터에는 특정 쿼리에 필요한 모든 데이터가 포함되어야합니다. 특정 장부 쿼리가 목록 쿼리가 반환하지 않는 필드를 가져 오면 Apollo Client가 캐시에서 데이터를 반환 할 수 없습니다.

데이터는 이미 클라이언트 캐시에 있을 가능성이 높지만 다른 쿼리로 요청 되었으므로 Apollo Client는 이를 알지 못합니다. Apollo 클라이언트에게 데이터를 찾을 위치를 알려주기 위해 다음과 같이 사용자 정의 리졸버를 정의 할 수 있습니다.

```text
import { toIdValue } from 'apollo-utilities';
import { InMemoryCache } from 'apollo-cache-inmemory';

const cache = new InMemoryCache({
  cacheRedirects: {
    Query: {
      book: (_, args) => toIdValue(cache.config.dataIdFromObject({ __typename: 'Book', id: args.id })),
    },
  },
});
```

> 참고 : 동일한 메소드를 사용하는 한 사용자 정의 dataIdFromObject 메소드에서도 작동합니다.

Apollo Client는 사용자 정의 리졸버의 반환 값을 사용하여 캐시에서 항목을 찾습니다. toIdValue는 반환 된 값이 스칼라 값이나 객체가 아니라 id로 해석되어야 함을 나타 내기 위해 사용해야합니다. 이 예에서 "쿼리"키는 루트 쿼리 유형 이름입니다.

**typename 속성에 무엇을 넣어야하는지 파악하려면 GraphiQL에서 쿼리 중 하나를 실행하고** typename 필드를 가져옵니다.

```text
query ListView {
  books {
    __typename
  }
}

# or

query DetailView {
  book(id: $id) {
    __typename
  }
}
```

반환되는 값 \(유형 이름\)은 \_\_typename 속성에 입력해야합니다.

ID 목록을 반환 할 수도 있습니다.

```text
cacheRedirects: {
  Query: {
    books: (_, args) => args.ids.map(id =>
      toIdValue(cache.config.dataIdFromObject({ __typename: 'Book', id: id }))),
  },
},
```

### Prefetching data <a id="prefetching-data"></a>

프리 페치는 Apollo Client를 사용하여 응용 프로그램의 UI를 훨씬 빠르게 느끼게하는 가장 쉬운 방법 중 하나입니다. 프리 페치는 단순히 데이터를 화면에 렌더링하기 전에 캐시에 데이터를로드하는 것을 의미합니다. 기본적으로 사용자가 뷰를 탐색 할 것으로 예상되는 즉시 뷰에 필요한 모든 데이터를로드하려고합니다.

사용자가 링크를 가리킬 때마다 client.query를 호출하면 몇 줄의 코드로 만이 작업을 수행 할 수 있습니다. 이 예제 앱인 Pupstagram의 Feed 구성 요소에서이 작업을 살펴 보겠습니다.

```text
function Feed() {
  const { loading, error, data, client } = useQuery(GET_DOGS);

  let content;
  if (loading) {
    content = <Fetching />;
  } else if (error) {
    content = <Error />;
  } else {
    content = (
      <DogList
        data={data.dogs}
        renderRow={(type, data) => (
          <Link
            to={{
              pathname: `/${data.breed}/${data.id}`,
              state: { id: data.id }
            }}
            onMouseOver={() =>
              client.query({
                query: GET_DOG,
                variables: { breed: data.breed }
              })
            }
            style={{ textDecoration: "none" }}
          >
            <Dog {...data} url={data.displayImage} />
          </Link>
        )}
      />
    );
  }

  return (
    <View style={styles.container}>
      <Header />
      {content}
    </View>
  );
}
```

우리가해야 할 일은 render prop 함수에서 클라이언트에 액세스하고 사용자가 링크를 가리킬 때 client.query를 호출하는 것입니다. 사용자가 링크를 클릭하면 데이터가 이미 Apollo 캐시에서 사용 가능하므로로드 상태가 표시되지 않습니다.

사용자가 UI에 약간의 데이터가 필요할 것으로 예상 할 수있는 다양한 방법이 있습니다. 호버 상태를 사용하는 것 외에도 데이터를 미리로드 할 수있는 다른 위치는 다음과 같습니다.

1.다단계 마법사의 다음 단계는 즉시

2. 클릭 유도 문안 버튼의 경로

3. 해당 영역 내에서 즉시 탐색 할 수 있도록 응용 프로그램의 하위 영역에 대한 모든 데이터

다른 아이디어가 있으면이 기사에 PR을 보내고 코드 스 니펫을 더 추가하십시오. 프리 페칭의 특별한 형태는 서버에서 수화를 저장하는 것이므로 첫 페이지로드에 실제로 필요한 것보다 더 많은 데이터를 수화하여 다른 상호 작용을 더 빠르게 수행 할 수도 있습니다.

### Query splitting\(쿼리 분할\) <a id="query-splitting"></a>

프리 페치는 응용 프로그램 UI를 더 빨리 느끼게하는 쉬운 방법입니다. 마우스 이벤트를 사용하여 필요한 데이터를 예측할 수 있습니다. 이것은 강력하고 브라우저에서 완벽하게 작동하지만 모바일 장치에는 적용 할 수 없습니다.

UI 환경을 개선하기위한 한 가지 솔루션은 쿼리에 더 많은 데이터를 미리로드하기 위해 조각을 사용하는 것이지만, 사용자에게 보여주지 않는 대량의 데이터를로드하는 것은 비용이 많이 듭니다.

또 다른 해결책은 큰 쿼리를 두 개의 작은 쿼리로 나누는 것입니다.

* 첫 번째는 이미 상점에있는 데이터를로드 할 수 있습니다. 즉, 즉시 표시 할 수 있습니다.
* 두 번째 쿼리는 아직 상점에없는 데이터를로드 할 수 있으며 서버에서 먼저 가져와야합니다.

이 솔루션은 너무 많은 데이터를 가져 오지 않고 서버가 응답하기 전에 뷰 데이터의 일부를 표시 할 수있는 이점을 제공합니다.

다음과 같은 스키마가 있다고 가정 해 보겠습니다.

```text
type Series {
  id: Int!
  title: String!
  description: String!
  episodes: [Episode]!
  cover: String!
}

type Episode {
  id: Int!
  title: String!
  cover: String!
}

type Query {
  series: [Series!]!
  oneSeries(id: Int): Series
}
```

그리고 당신은 두 가지보기가 있습니다 :

* 시리즈 개요 : 설명과 표지가있는 모든 시리즈 목록
* 시리즈 DetailView : 설명, 표지 및 에피소드 목록이있는 시리즈의 상세보기

시리즈 개요에 대한 쿼리는 다음과 같습니다.

```text
query SeriesOverviewData {
  series {
    id
    title
    description
    cover
  }
}
```

Series DetailView에 대한 쿼리는 다음과 같습니다.

```text
query SeriesDetailData($seriesId: Int!) {
  oneSeries(id: $seriesId) {
    id
    title
    description
    cover
  }
}
```

```text
query SeriesEpisodes($seriesId: Int!) {
  oneSeries(id: $seriesId) {
    id
    episodes {
      id
      title
      cover
    }
  }
}
```

oneSeries 필드에 대한 사용자 정의 분석기를 추가하고 캐시를 정규화하는 dataIdFromObject 함수를 사용하여 서버 왕복없이 상점에서 데이터를 즉시 분석 할 수 있습니다.

```text
import { ApolloClient } from 'apollo-client';
import { toIdValue } from 'apollo-utilities';
import { InMemoryCache } from 'apollo-cache-inmemory';

const cache = new InMemoryCache({
  cacheResolvers: {
    Query: {
      oneSeries: (_, { id }) => toIdValue(cache.config.dataIdFromObject({ __typename: 'Series', id })),
    },
  },
  dataIdFromObject,
})

const client = new ApolloClient({
  link, // your link,
  cache,
})
```

두 쿼리를 구현하는 두 번째보기의 구성 요소는 다음과 같습니다.

```text
const QUERY_SERIES_DETAIL_VIEW = gql`
  query SeriesDetailData($seriesId: Int!) {
    oneSeries(id: $seriesId) {
      id
      title
      description
      cover
    }
  }
`;

const QUERY_SERIES_EPISODES = gql`
  query SeriesEpisodes($seriesId: Int!) {
    oneSeries(id: $seriesId) {
      id
      episodes {
        id
        title
        cover
      }
    }
  }
`;

function SeriesDetailView({ seriesId }) {
  const {
    loading: seriesLoading,
    data: { oneSeries }
  } = useQuery(
    QUERY_SERIES_DETAIL_VIEW,
    { variables: { seriesId } }
  );

  const {
    loading: episodesLoading,
    data: { oneSeries: { episodes } = {} }
  } = useQuery(
    QUERY_SERIES_EPISODES,
    { variables: { seriesId } }
  );

  return (
    <div>
      <h1>{seriesLoading ? `Loading...` : oneSeries.title}</h1>
      <img src={seriesLoading ? `/dummy.jpg` : oneSeries.cover} />
      <h2>Episodes</h2>
      <ul>
        {episodesLoading ? (
          <li>Loading...</li>
        ) : (
          episodes.map(episode => (
            <li key={episode.id}>
              <img src={episode.cover} />
              <a href={`/episode/${episode.id}`}>{episode.title}</a>
            </li>
          ))
        )}
      </ul>
    </div>
  );
}
```

불행히도 사용자가 첫 번째보기를 방문하지 않고 두 번째보기를 방문하는 경우 두 번째 네트워크 요청이 발생합니다 \(첫 번째 쿼리의 데이터가 아직 저장소에 없기 때문에\). BatchedHttpLink를 사용하면 두 개의 쿼리를 하나의 네트워크 요청으로 서버에 보낼 수 있습니다.

