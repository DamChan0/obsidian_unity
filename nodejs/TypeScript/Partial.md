``` ts
Partial<T>

export const createUser = async (userData: Partial<User>) => {
  return await User.create(userData);
};
```

|문법|뜻|설명|
|---|---|---|
|Partial<T>|T타입 안의 모든 속성을 선택(optional)으로 만듬|객체 생성 시 일부 값만 전달해도 허용|

###  User type
``` TS
class User {
  id!: number;
  username!: string;
  password!: string;
  email!: string;
}
```

Partial 테크가 없다면 모든 필드를 채워서 데이터를 다뤄야한다.

```ts
const newUser: User = {
  id: 1,
  username: 'damchan',
  password: '1234',
  email: 'abc@abc.com'
}
```

#### Partial \<User \>
``` ts
const newUser: Partial<User> = {
  username: 'damchan',
  password: '1234'
}
```
위처럼 필드의 일부만 추가해도 가능하다