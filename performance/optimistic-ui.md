---
description: 낙관적 UI
---

# Optimistic UI

낙관적 UI는 서버에서 응답을 받기 전에도 변이 결과를 시뮬레이션하고 UI를 업데이트하는 데 사용할 수있는 패턴입니다. 서버에서 응답을 받으면 낙관적 결과가 삭제되고 실제 결과로 대체됩니다.

낙관적 UI를 사용하면 UI 응답 속도가 훨씬 빨라지고 데이터가 도착했을 때 데이터가 실제 응답과 일치하게됩니다

### Basic optimistic UI <a id="basic-optimistic-ui"></a>

"코멘트 편집"돌연변이가 있고 사용자가 서버 응답을 기다리지 않고 돌연변이를 제출하면 UI가 즉시 업데이트되기를 원한다고 가정 해 봅시다. 이것이 mutate 함수에 대한 optimisticResponse 매개 변수가 제공하는 것입니다.

Apollo를 사용하여 GraphQL 데이터를 UI 구성 요소로 가져 오는 주요 방법은 쿼리를 사용하는 것입니다. 따라서 낙관적 응답이 UI를 업데이트하도록하려면 올바른 쿼리 결과를 업데이트하는 낙관적 응답을 반환해야합니다. dataIdFromObject 옵션으로이를 수행하는 방법에 대해 자세히 학습하십시오.

코드에서 다음과 같이 보입니다.

```text
const UPDATE_COMMENT = gql`
  mutation UpdateComment($commentId: ID!, $commentContent: String!) {
    updateComment(commentId: $commentId, content: $commentContent) {
      id
      __typename
      content
    }
  }
`;

function CommentPageWithData() {
  const [mutate] = useMutation(UPDATE_COMMENT);
  return (
    <Comment
      updateComment={({ commentId, commentContent }) =>
        mutate({
          variables: { commentId, commentContent },
          optimisticResponse: {
            __typename: "Mutation",
            updateComment: {
              id: commentId,
              __typename: "Comment",
              content: commentContent
            }
          }
        })
      }
    />
  );
}
```

dataIdFromObject가 전역 적으로 고유 한 객체 ID를 결정하는 데 사용하는 id이기 때문에 id와 \_\_typename을 선택합니다. Apollo가 어떤 객체를 참조하는지 알 수 있도록 해당 필드에 올바른 값을 제공해야합니다.

### Adding to a list\(목록에 추가\) <a id="adding-to-a-list"></a>

위의 예에서, 우리는 낙관적 돌연변이 결과로 상점의 기존 객체를 완벽하게 편집하는 방법을 보여주었습니다. 그러나 많은 돌연변이는 상점의 기존 객체를 업데이트 할뿐만 아니라 새로운 객체를 삽입합니다.

이 경우 새 데이터를 기존 쿼리와 UI에 통합하는 방법을 지정해야합니다. 캐시 된 데이터와 상호 작용하는 기사에서이를 수행하는 방법에 대해 자세히 읽을 수 있습니다. 특히 업데이트 기능을 사용하여 기존 쿼리의 결과 세트에 결과를 삽입 할 수 있습니다. 업데이트는 낙관적 결과와 서버에서 반환 된 실제 결과에 대해 정확히 동일한 방식으로 작동합니다.

다음은 주석을 기존 주석 목록에 삽입하는 GitHunt의 구체적인 예입니다.

```text
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

function CommentsPageWithMutations({ currentUser }) {
  const [mutate] = useMutation(SUBMIT_COMMENT_MUTATION);
  return (
    <CommentsPage
      submit={(repoFullName, commentContent) =>
        mutate({
          variables: { repoFullName, commentContent },
          optimisticResponse: {
            __typename: "Mutation",
            submitComment: {
              __typename: "Comment",
              postedBy: currentUser,
              createdAt: new Date(),
              content: commentContent
            }
          },
          update: (proxy, { data: { submitComment } }) => {
            // Read the data from our cache for this query.
            const data = proxy.readQuery({ query: CommentAppQuery });
            // Write our data back to the cache with the new comment in it
            proxy.writeQuery({ query: CommentAppQuery, data: {
              ...data,
              comments: [...data.comments, submitComment]
            }});
          }
        })
      }
    />
  );
}
```

