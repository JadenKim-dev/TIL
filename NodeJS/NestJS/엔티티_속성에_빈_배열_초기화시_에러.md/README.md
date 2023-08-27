# 엔티티 속성에 배열 초기화시 에러

```
InitializedRelationError: Array initializations are not allowed in entity relations. Please remove array initialization (= []) from "Meet#meetToMembers". This is ORM requirement to make relations to work properly. Refer docs for more information.
```

```
constructor() {
  this.meetToMembers = meetToMembers || [];
}

```

```
@AfterLoad()
  async nullChecks() {
    if (!this.meetToMembers) {
      this.meetToMembers = [];
    }
  }
```
