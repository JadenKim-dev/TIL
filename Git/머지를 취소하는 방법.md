머지를 취소해야 하는 상황에서는 다음과 같은 명령을 사용할 수 있습니다.

먼저, 만약 머지를 시도했는데 conflict가 발생한 상황에서 이를 취소하고 싶다면 `git merge --abort`로 간단히 머지를 취소할 수 있습니다.

만약 이미 머지가 완료되었다면 아래의 방법을 사용합니다.
`git reset` 을 통해 단순히 특정 커밋으로 되돌리는 방법이 있습니다.
`git log` 또는 `git reflog` 로 커밋 내역을 확인하고, `git reset --merge <commit id>` 를 사용합니다.

마지막 커밋의 hash가 명확하지 않을 수 있습니다.
이 때에는 `git reset --merge HEAD~1`을 사용하여 머지 이전 상태로 돌아갑니다.

참고 자료:
https://www.freecodecamp.org/korean/news/giteseo-meoji-doedolrigi-majimag-meojireul-cwisohaneun-bangbeob/
