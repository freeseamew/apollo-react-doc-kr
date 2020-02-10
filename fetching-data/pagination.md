# Pagination

종종 응용 프로그램에서 일부 데이터를 가져 와서 한 번에 가져 오거나 표시하기에는 너무 많은 데이터가 포함 된 목록을 표시해야하는보기가 있습니다. 페이지 매김은이 문제에 대한 가장 일반적인 솔루션이며, Apollo Client는 기능을 내장하여 매우 쉽게 사용할 수 있습니다.

페이지 매김 된 데이터를 가져 오는 방법에는 기본적으로 번호가 매겨진 페이지와 커서의 두 가지 방법이 있습니다. 페이지 매김 데이터를 표시하는 방법에는 이산 페이지와 무한 스크롤의 두 가지 방법이 있습니다. 차이점에 대한 자세한 설명과 다른 것을 사용하려는 경우 페이지 매김 이해에 대한 블로그 게시물을 읽는 것이 좋습니다.

이 기사에서는 두 가지 접근 방식을 모두 구현하기 위해 Apollo를 사용하는 기술적 세부 사항을 다룹니다.

### Using `fetchMore` <a id="using-fetchmore"></a>

Apollo에서 페이지 매김을 수행하는 가장 쉬운 방법은 fetchMore라는 함수를 사용하는 것입니다.이 함수는 useQuery Hook가 반환 한 결과 객체에 포함되어 있습니다. 이를 통해 기본적으로 새로운 GraphQL 쿼리를 수행하고 결과를 원래 결과에 병합 할 수 있습니다.

새 쿼리에 사용할 쿼리 및 변수와 새 쿼리 결과를 클라이언트의 기존 데이터와 병합하는 방법을 지정할 수 있습니다. 정확히 어떻게하면 어떤 종류의 페이지 매김이 구현되는지 결정됩니다.

### Offset-based <a id="offset-based"></a>

번호가 매겨진 페이지라고도하는 오프셋 기반 페이지 매김은 일반적으로 백엔드에서 가장 구현하기 때문에 많은 웹 사이트에서 볼 수있는 매우 일반적인 패턴입니다. 예를 들어 SQL에서는 OFFSET 및 LIMIT를 사용하여 번호가 매겨진 페이지를 쉽게 생성 할 수 있습니다.

```javascript
const FEED_QUERY = gql`
  query Feed($type: FeedType!, $offset: Int, $limit: Int) {
    currentUser {
      login
    }
    feed(type: $type, offset: $offset, limit: $limit) {
      id
      # ...
    }
  }
`;

const FeedData({ match }) {
  const { data, fetchMore } = useQuery(
    FEED_QUERY,
    {
      variables: {
        type: match.params.type.toUpperCase() || "TOP",
        offset: 0,
        limit: 10
      },
      fetchPolicy: "cache-and-network"
    }
  );

  return (
    <Feed
      entries={data.feed || []}
      onLoadMore={() =>
        fetchMore({
          variables: {
            offset: data.feed.length
          },
          updateQuery: (prev, { fetchMoreResult }) => {
            if (!fetchMoreResult) return prev;
            return Object.assign({}, prev, {
              feed: [...prev.feed, ...fetchMoreResult.feed]
            });
          }
        })
      }
    />
  );
}
```

보다시피, fetchMore는 useQuery Hook 결과 객체를 통해 액세스 할 수 있습니다. 기본적으로 fetchMore는 원래 쿼리를 사용하므로 새 변수 만 전달합니다. 새 데이터가 서버에서 반환되면 updateQuery 함수를 사용하여 기존 데이터와 병합하여 UI 구성 요소를 확장 된 목록으로 다시 렌더링합니다.

위의 접근 방식은 한계 / 오프셋 페이지 매김에 효과적입니다. 번호가 매겨진 페이지 또는 오프셋이있는 페이지 매김의 한 가지 단점은 항목을 동시에 목록에 삽입하거나 목록에서 제거 할 때 항목을 건너 뛰거나 두 번 반환 할 수 있다는 것입니다. 커서 기반 페이지 매김으로 피할 수 있습니다.

fetchMore가 호출 된 후 UI 구성 요소가 업데이트 된로드 소품을 수신하려면 조회 구성 요소의 props에서 notifyOnNetworkStatusChange를 true로 설정해야합니다

### Cursor-based <a id="cursor-based"></a>

커서 기반 페이지 매김에서 "커서"는 데이터 세트에서 다음 항목을 가져와야하는 위치를 추적하는 데 사용됩니다. 때로는 커서가 매우 단순하고 가져온 마지막 개체의 ID를 참조 할 수도 있지만 일부 경우 \(예 : 일부 기준에 따라 정렬 된 목록\) 커서는 마지막 개체의 ID 외에 정렬 기준을 인코딩해야합니다. 가져 왔습니다.

클라이언트에서 커서 기반 페이지 매김을 구현하는 것은 오프셋 기반 페이지 매김과 다르지 않지만 절대 오프셋을 사용하는 대신 가져온 마지막 객체에 대한 참조와 사용 된 정렬 순서에 대한 정보를 유지합니다.

아래 예에서는 fetchMore 쿼리를 사용하여 새 주석을 지속적으로로드합니다.이 주석은 목록 앞에 추가됩니다. fetchMore 쿼리에 사용되는 커서는 초기 서버 응답에서 제공되며 더 많은 데이터가 페치 될 때마다 업데이트됩니다.

```javascript
const MORE_COMMENTS_QUERY = gql`
  query MoreComments($cursor: String) {
    moreComments(cursor: $cursor) {
      cursor
      comments {
        author
        text
      }
    }
  }
`;

function CommentsWithData() {
  const { data: { comments, cursor }, loading, fetchMore } = useQuery(
    MORE_COMMENTS_QUERY
  );

  return (
    <Comments
      entries={comments || []}
      onLoadMore={() =>
        fetchMore({
          // note this is a different query than the one used in the
          // Query component
          query: MORE_COMMENTS_QUERY,
          variables: { cursor: cursor },
          updateQuery: (previousResult, { fetchMoreResult }) => {
            const previousEntry = previousResult.entry;
            const newComments = fetchMoreResult.moreComments.comments;
            const newCursor = fetchMoreResult.moreComments.cursor;

            return {
              // By returning `cursor` here, we update the `fetchMore` function
              // to the new cursor.
              cursor: newCursor,
              entry: {
                // Put the new comments in the front of the list
                comments: [...newComments, ...previousEntry.comments]
              },
              __typename: previousEntry.__typename
            };
          }
        })
      }
    />
  );
}
```

### Relay-style cursor pagination <a id="relay-style-cursor-pagination"></a>

는 페이지 매김 쿼리의 입력 및 출력에 대해 의견을 나누므로 사람들은 때때로 Relay의 요구에 따라 서버의 페이지 매김 모델을 구축합니다. 릴레이 커서 연결 스펙과 작동하도록 설계된 서버가있는 경우 문제없이 Apollo 클라이언트에서 해당 서버를 호출 할 수도 있습니다.

릴레이 스타일 커서 사용은 기본 커서 기반 페이지 매김과 매우 유사합니다. 가장 큰 차이점은 커서를 얻는 위치에 영향을주는 쿼리 응답 형식입니다.

Relay는 반환 된 커서 연결에 startInfo 및 endCursor 속성으로 각각 반환 된 첫 번째 항목과 마지막 항목의 커서를 포함하는 pageInfo 객체를 제공합니다. 이 개체에는 사용 가능한 결과가 더 있는지 확인하는 데 사용할 수있는 부울 속성 hasNextPage도 포함되어 있습니다.

다음 예제는 한 번에 10 개의 항목 요청을 지정하며 결과는 제공된 커서 다음에 시작되어야합니다. 커서 릴레이에 대해 널이 전달되면이를 무시하고 데이터 세트의 시작부터 시작하여 초기 및 후속 요청 모두에 동일한 조회를 사용할 수있는 결과를 제공합니다`.`

```javascript


    }
  }
`;

function CommentsWithData() {
  const { data: { Comments: comments }, loading, fetchMore } = useQuery(
    COMMENTS_QUERY
  );

  return (
    <Comments
      entries={comments || []}
      onLoadMore={() =>
        fetchMore({
          variables: {
            cursor: comments.pageInfo.endCursor
          },
          updateQuery: (previousResult, { fetchMoreResult }) => {
            const newEdges = fetchMoreResult.comments.edges;
            const pageInfo = fetchMoreResult.comments.pageInfo;

            return newEdges.length
              ? {
                  // Put the new comments at the end of the list and update `pageInfo`
                  // so we have the new `endCursor` and `hasNextPage` values
                  comments: {
                    __typename: previousResult.comments.__typename,
                    edges: [...previousResult.comments.edges, ...newEdges],
                    pageInfo
                  }
                }
              : previousResult;
          }
        })
      }
    />
  );
}
```

### The `@connection` directive\(@connection 지시어\) <a id="the-connection-directive"></a>

페이지 매김 쿼리를 사용하는 경우 쿼리에 전달 된 매개 변수가 기본 저장소 키를 결정하는 데 사용되지만 일반적으로 쿼리를 실행하는 코드 조각 외부에 알려지지 않으므로 누적 쿼리의 결과를 매장에서 찾기가 어려울 수 있습니다. 대상 업데이트에 안정적인 저장소 키가 없기 때문에 이는 필수 저장소 업데이트에 문제가됩니다. 페이지 매김 된 쿼리에 안정적인 저장소 키를 사용하도록 Apollo Client에 지시하려면 선택적 @connection 지시문을 사용하여 쿼리 일부에 대한 저장소 키를 지정할 수 있습니다. 예를 들어 피드 쿼리에 안정적인 저장소 키를 원한다면 @connection 지시문을 사용하도록 쿼리를 조정할 수 있습니다.

```javascript
const FEED_QUERY = gql`
  query Feed($type: FeedType!, $offset: Int, $limit: Int) {
    currentUser {
      login
    }
    feed(type: $type, offset: $offset, limit: $limit) @connection(key: "feed", filter: ["type"]) {
      id
      # ...
    }
  }
`;
```

이로 인해 모든 쿼리에서 누적 된 피드 또는 fetchMore가 피드 키 아래의 스토어에 배치되어 나중에 필수 상점 업데이트에 사용할 수 있습니다. 이 예에서는 @connection 지시문의 선택적 필터 인수를 사용하여 저장소 키에 쿼리의 일부 인수를 포함 할 수 있습니다. 이 경우 상점 키에 유형 쿼리 인수를 포함하여 각 유형의 피드에서 페이지를 누적하는 여러 상점 값이 생성됩니다.

