# Interacting with cached data \(상호작용\):2.6

ApolloClient 객체는 캐시 된 데이터와 상호 작용하기위한 다음과 같은 방법을 제공합니다.

* `readQuery`
* `readFragment`
* `writeQuery`
* `writeFragment`

이러한 방법은 아래에 자세히 설명되어 있습니다.

> 중요 :이 메서드는 캐시가 아닌 앱의 ApolloClient 개체에서 호출해야합니다. 이렇게하면 ApolloClient 개체가 캐시 변경 사항을 전체 앱에 브로드 캐스트하여 자동 UI 업데이트가 가능합니다. 대신 캐시에서 이러한 메소드를 직접 호출하면 변경 사항이 브로드 캐스트되지 않습니다.

아래의 모든 코드 샘플에서는 ApolloClient 인스턴스를 초기화했으며 graphql-tag에서 gql 태그를 가져 왔다고 가정합니다.

### `readQuery` <a id="readquery"></a>

readQuery 메소드를 사용하면 캐시에서 직접 GraphQL 쿼리를 실행할 수 있습니다.

캐시에 지정된 쿼리를 수행하는 데 필요한 모든 데이터가 포함되어 있으면 readQuery는 GraphQL 서버와 마찬가지로 쿼리 모양으로 데이터 객체를 반환합니다.

캐시에 지정된 쿼리를 수행하는 데 필요한 모든 데이터가 포함되어 있지 않으면 readQuery에서 오류가 발생합니다. 원격 서버에서 데이터를 페치하려고 시도하지 않습니다.

다음과 같이 readQuery에 GraphQL 쿼리 문자열을 전달하십시오.

```text
const { todo } = client.readQuery({
  query: gql`
    query ReadTodo {
      todo(id: 5) {
        id
        text
        completed
      }
    }
  `,
});
```

다음과 같이 GraphQL 변수를 readQuery에 제공 할 수 있습니다.

```text
const { todo } = client.readQuery({
  query: gql`
    query ReadTodo($id: Int!) {
      todo(id: $id) {
        id
        text
        completed
      }
    }
  `,
  variables: {
    id: 5,
  },
});
```

> readQuery의 리턴 값을 수정하지 마십시오. 동일한 객체가 여러 구성 요소로 반환 될 수 있습니다. 캐시에서 데이터를 업데이트하려면 대신 대체 객체를 만들어 writeQuery에 전달하십시오.

### `readFragment` <a id="readfragment"></a>

readFragment 메소드를 사용하면 조회 결과의 일부로 저장된 정규화 된 캐시 오브젝트에서 데이터를 읽을 수 있습니다. readQuery와 달리 readFragment에 대한 호출은 데이터 그래프에서 지원되는 쿼리 중 하나의 구조를 따를 필요가 없습니다.

예를 들면 다음과 같습니다.

```text
const todo = client.readFragment({
  id: ..., // `id` is any id that could be returned by `dataIdFromObject`.
  fragment: gql`
    fragment myTodo on Todo {
      id
      text
      completed
    }
  `,
});
```

첫 번째 인수 인 id는 캐시에서 읽으려는 객체에 지정된 고유 식별자입니다. 이는 저장시 dataIdFromObject 함수가 오브젝트에 지정한 값과 일치해야합니다.

예를 들어 다음과 같이 ApolloClient를 초기화한다고 가정 해 보겠습니다.

```text
const client = new ApolloClient({
  ...,
  cache: new InMemoryCache({
    ...,
    dataIdFromObject: object => object.id,
  }),
});
```

이전에 실행 된 쿼리가 ID가 5 인 Todo 오브젝트를 캐시 한 경우 다음 readFragment 호출을 사용하여 캐시에서 해당 오브젝트를 읽을 수 있습니다.

```text
const todo = client.readFragment({
  id: '5',
  fragment: gql`
    fragment myTodo on Todo {
      id
      text
      completed
    }
  `,
});
```

위의 예에서 id가 5 인 Todo 객체가 캐시에 없으면 readFragment는 null을 반환합니다. Todo 객체가 캐시에 있지만 텍스트 또는 완료된 필드가 누락되면 readFragment에서 오류가 발생합니다.

### `writeQuery` and `writeFragment` <a id="writequery-and-writefragment"></a>

Apollo Client 캐시에서 임의의 데이터를 읽는 것 외에도 writeQuery 및 writeFragment 메소드를 사용하여 임의의 데이터를 캐시에 쓸 수 있습니다.

writeQuery 및 writeFragment를 사용하여 캐시 된 데이터를 변경하면 GraphQL 서버로 푸시되지 않습니다. 환경을 다시로드하면 이러한 변경 사항이 사라집니다.

이러한 메소드에는 추가 데이터 변수가 필요하다는 점을 제외하고 읽기 메소드와 동일한 서명이 있습니다.

예를 들어, 다음 writeFragment 호출은 ID가 5 인 Todo 오브젝트에 대해 완료된 플래그를 로컬로 업데이트합니다.

```text
client.writeFragment({
  id: '5',
  fragment: gql`
    fragment myTodo on Todo {
      completed
    }
  `,
  data: {
    completed: true,
  },
});
```

Apollo Client 캐시의 모든 가입자는이 변경 사항을보고 응용 프로그램의 UI를 적절히 업데이트합니다.

또 다른 예로, readQuery와 writeQuery를 결합하여 캐시 된 작업 목록에 새 작업 항목을 추가 할 수 있습니다.

```text
const query = gql`
  query MyTodoAppQuery {
    todos {
      id
      text
      completed
    }
  }
`;

// Get the current to-do list
const data = client.readQuery({ query });

const myNewTodo = {
  id: '6',
  text: 'Start using Apollo Client.',
  completed: false,
  __typename: 'Todo',
};

// Write back to the to-do list and include the new item
client.writeQuery({
  query,
  data: {
    todos: [...data.todos, myNewTodo],
  },
});
```

### Recipes <a id="recipes"></a>

다음은 캐시에 직접 액세스해야하는 일반적인 상황입니다. 흥미로운 방식으로 캐시를 조작하고 예제를 소개하려면 풀 요청으로 보내십시오!

#### Bypassing the cache`(캐시 우회)` <a id="bypassing-the-cache"></a>

때로는 특정 작업에 캐시를 사용하지 않는 것이 좋습니다. 캐시없는 fetchPolicy를 사용하여이 작업을 수행 할 수 있습니다. 캐시 없음 정책은 응답으로 캐시에 쓰지 않습니다. 캐시에 보관하고 싶지 않은 비밀번호와 같은 민감한 데이터에 유용 할 수 있습니다.

#### Updating after a mutation \(업데이트 후 변이\) <a id="updating-after-a-mutation"></a>

경우에 따라 dataIdFromObject를 사용하는 것만으로는 응용 프로그램 UI가 올바르게 업데이트되지 않을 수 있습니다. 예를 들어, 전체 목록을 다시 가져 오지 않고 객체 목록에 무언가를 추가하거나 객체 식별자를 할당 할 수없는 객체가있는 경우 Apollo Client는 기존 쿼리를 업데이트 할 수 없습니다. 다른 도구에 대한 정보는 계속 읽으십시오.

refetchQueries는 캐시를 업데이트하는 가장 간단한 방법입니다. refetchQueries를 사용하면 변이의 영향을 받았을 수있는 저장소 부분을 다시 가져 오기 위해 변이가 완료된 후 실행할 하나 이상의 쿼리를 지정할 수 있습니다.

```text
mutate({
  //... insert comment mutation
  refetchQueries: [{
    query: gql`
      query UpdateCache($repoName: String!) {
        entry(repoFullName: $repoName) {
          id
          comments {
            postedBy {
              login
              html_url
            }
            createdAt
            content
          }
        }
      }
    `,
    variables: { repoName: 'apollographql/apollo-client' },
  }],
})
```

문자열 배열로 refetchQueries를 호출하면 Apollo Client는 제공된 문자열과 이름이 같은 이전에 호출 된 쿼리를 찾습니다. 그런 다음 현재 변수를 사용하여 해당 쿼리를 다시 가져옵니다.

refetchQueries를 사용하는 가장 일반적인 방법은 다른 구성 요소에 대해 정의 된 쿼리를 가져와 해당 구성 요소가 업데이트되도록하는 것입니다.

```text
import RepoCommentsQuery from '../queries/RepoCommentsQuery';

mutate({
  //... insert comment mutation
  refetchQueries: [{
    query: RepoCommentsQuery,
    variables: { repoFullName: 'apollographql/apollo-client' },
  }],
})
```

업데이트를 사용하면 캐시를 완전히 제어 할 수 있으므로 원하는 방식으로 돌연변이에 대응하여 데이터 모델을 변경할 수 있습니다. 쿼리 후에 캐시를 업데이트하는 것이 좋습니다. 여기에 자세히 설명되어 있습니다.

```text
import CommentAppQuery from '../queries/CommentAppQuery';

const SUBMIT_COMMENT_MUTATION = gql`
  mutation SubmitComment($repoFullName: String!, $commentContent: String!) {
    submitComment(
      repoFullName: $repoFullName
      commentContent: $commentContent
    ) {
      postedBy {
        login
        html_url
      }
      createdAt
      content
    }
  }
`;

const CommentsPageWithMutations = () => (
  <Mutation mutation={SUBMIT_COMMENT_MUTATION}>
    {mutate => {
      <AddComment
        submit={({ repoFullName, commentContent }) =>
          mutate({
            variables: { repoFullName, commentContent },
            update: (store, { data: { submitComment } }) => {
              // Read the data from our cache for this query.
              const data = store.readQuery({ query: CommentAppQuery });
              // Add our comment from the mutation to the end.
              data.comments.push(submitComment);
              // Write our data back to the cache.
              store.writeQuery({ query: CommentAppQuery, data });
            }
          })
        }
      />;
    }}
  </Mutation>
);
```

#### Incremental loading: `fetchMore(추가 로딩)` <a id="incremental-loading-fetchmore"></a>

`fetchMore` can be used to update the result of a query based on the data returned by another query. Most often, it is used to handle infinite-scroll pagination or other situations where you are loading more data when you already have some.

In our GitHunt example, we have a paginated feed that displays a list of GitHub repositories. When we hit the "Load More" button, we don't want Apollo Client to throw away the repository information it has already loaded. Instead, it should just append the newly loaded repositories to the list that Apollo Client already has in the store. With this update, our UI component should re-render and show us all of the available repositories.

Let's see how to do that with the `fetchMore` method on a query:

```text
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

const FeedWithData = ({ match }) => (
  <Query
    query={FEED_QUERY}
    variables={{
      type: match.params.type.toUpperCase() || "TOP",
      offset: 0,
      limit: 10
    }}
    fetchPolicy="cache-and-network"
  >
    {({ data, fetchMore }) => (
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
    )}
  </Query>
);
```

송 될 변수 맵을 사용합니다. 피드에 아직 표시되지 않은 항목을 가져 오도록 오프셋을 feed.length로 설정합니다. 이 변수 맵은 구성 요소와 연관된 조회에 지정된 것과 맵핑됩니다. 이것은 다른 변수, 예를 들어 제한 변수는 구성 요소 쿼리 내에서와 동일한 값을 갖습니다.

더 많은 정보를 가져 오기 위해 가져올 쿼리를 포함하는 GraphQL 문서 일 수있는 argument라는 쿼리를 사용할 수도 있습니다. 이것을 fetchMore 쿼리라고합니다. 기본적으로 fetchMore 쿼리는 컨테이너와 연관된 쿼리 \(이 경우 FEED\_QUERY\)입니다.

fetchMore를 호출하면 Apollo Client는 fetchMore 쿼리를 실행하고 updateQuery 옵션의 논리를 사용하여 해당 결과를 원래 결과에 통합합니다. 명명 된 인수 updateQuery는 구성 요소와 관련된 이전 쿼리 결과 \(이 경우 FEED\_QUERY\)와 fetchMore 쿼리에서 반환 된 정보를 가져 와서이 둘의 조합을 반환하는 함수 여야합니다.

여기서 fetchMore 조회는 구성 요소와 연관된 조회와 동일합니다. updateQuery는 반환 된 새 피드 항목을 가져 와서 이전에 요청한 피드 항목에 추가합니다. 이를 통해 UI가 업데이트되고 피드에 다음 항목 페이지가 포함됩니다!

fetchMore는 종종 페이지 매김에 사용되지만 적용 가능한 다른 많은 경우가 있습니다. 예를 들어, 항목 목록 \(예 : 협업 작업 목록\)이 있고 특정 시간 이후에 업데이트 된 항목을 가져 오는 방법이 있다고 가정하십시오. 그런 다음 업데이트를 얻기 위해 전체 할 일 목록을 다시 가져올 필요가 없습니다. updateQuery 함수가 새 결과를 올바르게 병합하는 한 새로 추가 된 항목을 fetchMore와 통합하면됩니다.

#### The `@connection` directive <a id="the-connection-directive"></a>

기본적으로 페이지 매김 쿼리는 fetchMore를 호출하여 동일한 캐시 키를 업데이트한다는 점을 제외하면 다른 쿼리와 동일합니다. 이러한 쿼리는 초기 쿼리와 해당 매개 변수에 의해 캐시되므로 나중에 캐시에서 페이지 매김 된 쿼리를 검색하거나 업데이트 할 때 문제가 발생합니다. 우리는 더 많은 것을 가져올 필요가없는 한계, 오프셋 또는 커서와 같은 페이지 매김 인수에 신경 쓰지 않으며 캐시 된 데이터에 액세스하기 위해 단순히 인수를 제공하지 않습니다.

이 문제를 해결하기 위해 Apollo Client 1.6에는 @connection 지시문이 도입되어 결과에 대한 사용자 지정 저장소 키를 지정했습니다. 연결을 통해 필드의 캐시 키를 설정하고 실제로 어떤 인수가 쿼리를 변경하는지 필터링 할 수 있습니다.

@connection 지시문을 사용하려면 사용자 지정 저장소 키를 원하는 쿼리 세그먼트에 지시문을 추가하고 키 매개 변수를 제공하여 저장소 키를 지정하십시오. 키 매개 변수 외에도 선택적 필터 매개 변수를 포함 할 수 있습니다.이 매개 변수는 생성 된 사용자 정의 저장소 키에 포함 할 쿼리 인수 이름 배열을 사용합니다.

```text
const query = gql`
  query Feed($type: FeedType!, $offset: Int, $limit: Int) {
    feed(type: $type, offset: $offset, limit: $limit) @connection(key: "feed", filter: ["type"]) {
      ...FeedEntry
    }
  }
`
```

위의 쿼리를 사용하면 여러 fetchMores가 있어도 각 피드 업데이트의 결과는 항상 스토어의 피드 키가 최신 누적 값으로 업데이트됩니다. 이 예에서는 @connection 지시문의 선택적 필터 인수를 사용하여 저장소 키에 형식 쿼리 인수를 포함시켜 각 피드 유형에서 쿼리를 누적하는 여러 저장소 값이 생성됩니다.

이제 안정적인 상점 키가 있으므로 writeQuery를 사용하여 상점 업데이트를 쉽게 수행 할 수 있습니다.이 경우 피드를 지 웁니다.

```text
client.writeQuery({
  query: gql`
    query Feed($type: FeedType!) {
      feed(type: $type) @connection(key: "feed", filter: ["type"]) {
        id
      }
    }
  `,
  variables: {
    type: "top",
  },
  data: {
    feed: [],
  },
});
```

우리는 store 키에서 type 인수 만 사용하기 때문에 오프셋이나 제한을 제공 할 필요가 없습니다.

#### Cache redirects with `cacheRedirects` <a id="cache-redirects-with-cacheredirects"></a>

경우에 따라 쿼리는 클라이언트 저장소에 이미 존재하는 데이터를 다른 키로 요청합니다. 가장 일반적인 예는 UI에 동일한 데이터를 사용하는 목록보기 및 세부 사항보기가있는 경우입니다. 목록보기는 다음 쿼리를 실행할 수 있습니다.

```text
query ListView {
  books {
    id
    title
    abstract
  }
}
```

특정 책을 선택하면 상세 조회에이 쿼리를 사용하여 개별 항목이 표시됩니다.

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

데이터는 이미 클라이언트 캐시에있을 가능성이 높지만 다른 쿼리로 요청되었으므로 Apollo Client는이를 알지 못합니다. Apollo 클라이언트에게 데이터를 찾을 위치를 알려주기 위해 다음과 같이 사용자 정의 리졸버를 정의 할 수 있습니다.

```text
import { InMemoryCache } from 'apollo-cache-inmemory';

const cache = new InMemoryCache({
  cacheRedirects: {
    Query: {
      book: (_, args, { getCacheKey }) =>
        getCacheKey({ __typename: 'Book', id: args.id })
    },
  },
});
```

참고 : 동일한 메소드를 사용하는 한 사용자 정의 dataIdFromObject 메소드에서도 작동합니다.

Apollo Client는 사용자 정의 리졸버가 반환 한 ID를 사용하여 캐시에서 항목을 찾습니다. getCacheKey는 \_\_typename 및 id를 기반으로 객체의 키를 생성하기 위해 세 번째 인수 내에서 확인 자로 전달됩니다.

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
    books: (_, args, { getCacheKey }) =>
      args.ids.map(id =>
        getCacheKey({ __typename: 'Book', id: id }))
  }
}
```

#### Resetting the store <a id="resetting-the-store"></a>

때때로 사용자가 로그 아웃 할 때와 같이 상점을 완전히 재설정하려고 할 수 있습니다. 이를 위해 client.resetStore를 사용하여 Apollo 캐시를 지우십시오. client.resetStore도 활성 쿼리를 다시 가져 오기 때문에 비동기식입니다.

```text
export default withApollo(graphql(PROFILE_QUERY, {
  props: ({ data: { loading, currentUser }, ownProps: { client }}) => ({
    loading,
    currentUser,
    resetOnLogout: async () => client.resetStore(),
  }),
})(Profile));
```

저장소가 재설정 된 후 실행될 콜백 함수를 등록하려면 client.onResetStore를 호출하고 콜백을 전달하십시오. 여러 개의 콜백을 등록하려면 client.onResetStore를 다시 호출하십시오. 모든 콜백이 배열로 푸시되어 동시에 실행됩니다.

이 예에서는 client.onResetStore를 사용하여 기본값을 apollo-link-state의 캐시에 씁니다. 로컬 상태 관리를 위해 아폴로 링크 상태를 사용하고 응용 프로그램의 어느 곳에서나 client.resetStore를 호출하는 경우에 필요합니다.

```text
import { ApolloClient } from 'apollo-client';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { withClientState } from 'apollo-link-state';

import { resolvers, defaults } from './resolvers';

const cache = new InMemoryCache();
const stateLink = withClientState({ cache, resolvers, defaults });

const client = new ApolloClient({
  cache,
  link: stateLink,
});

client.onResetStore(stateLink.writeDefaults);
```

React 컴포넌트에서 client.onResetStore를 호출 할 수도 있습니다. 상점을 재설정 한 후 UI를 강제로 다시 렌더링하려는 경우에 유용합니다.

resetStore에서 콜백을 구독 취소하려면 구독 취소 기능에 client.onResetStore의 리턴 값을 사용하십시오.

```text
import { withApollo } from "react-apollo";

export class Foo extends Component {
  constructor(props) {
    super(props);
    this.unsubscribe = props.client.onResetStore(
      () => this.setState({ reset: false })
    );
    this.state = { reset: false };
  }
  componentDidUnmount() {
    this.unsubscribe();
  }
  render() {
    return this.state.reset ? <div /> : <span />
  }
}

export default withApollo(Foo);
```

저장소를 지우고 싶지만 활성 쿼리를 다시 가져 오지 않으려면 client.resetStore \(\) 대신 client.clearStore \(\)를 사용하십시오.

#### Server side rendering <a id="server-side-rendering"></a>

먼저 서버에서 InMemoryCache를 초기화하고 ApolloClient 인스턴스를 만들어야합니다. 서버의 초기 직렬화 된 HTML 페이로드에는 캐시에서 데이터를 추출하는 스크립트 태그가 포함되어야합니다. 스크립트 삽입 공격을 방지하려면 .replace \(\)가 필요합니다.

```text
`<script>
  window.__APOLLO_STATE__=${JSON.stringify(cache.extract()).replace(/</g, '\\u003c')}
</script>`
```

클라이언트에서 서버에서 전달 된 초기 데이터를 사용하여 캐시를 재수화할 수 있습니다.

```text
cache: new Cache().restore(window.__APOLLO_STATE__)
```

서버 측 렌더링에 대한 자세한 내용은 여기에서 자세한 내용을 확인하십시오.

#### Cache persistence\(캐시 지속성\) <a id="cache-persistence"></a>

AsyncStorage 또는 localStorage와 같은 스토리지 공급자로부터 Apollo Cache를 유지하고 재수 화하려면 apollo-cache-persist를 사용할 수 있습니다. apollo-cache-persist는 InMemoryCache & Hermes를 포함한 모든 Apollo 캐시 및 다양한 스토리지 공급자와 함께 작동합니다.

시작하려면 간단히 Apollo Cache와 스토리지 공급자를 persistCache에 전달하십시오. 기본적으로 Apollo Cache의 내용은 비동기 적으로 즉시 복원되며 구성 가능한 짧은 디 바운스 간격으로 캐시에 쓸 때마다 지속됩니다.

> 참고 : persistCache 메소드는 비동기이며 Promise를 리턴합니다.

```text
import { AsyncStorage } from 'react-native';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { persistCache } from 'apollo-cache-persist';

const cache = new InMemoryCache();

persistCache({
  cache,
  storage: AsyncStorage,
}).then(() => {
  // Continue setting up Apollo as usual.
})
```

앱이 백그라운드에있을 때 캐시 유지 및 추가 구성 옵션과 같은 고급 사용법을 보려면 apollo-cache-persist의 README를 확인하십시오.

