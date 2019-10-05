# Rails 6.1 Case Sensitive

작성자 : gmlwo530

Rails 6으로 올리고 이런 경고가 떴다

```plain text
DEPRECATION WARNING: Uniqueness validator will no longer enforce case sensitive comparison in Rails 6.1. To continue case sensitive comparison on the :name attribute in User model, pass `case_sensitive: true` option explicitly to the uniqueness validator.
```

`case_sensitive`는 Rails 6에 새로 생긴 점인데, 오늘은 이 부분에 대해서 알아보겠다.

보통 서비스에서 `uniqueness`를 제일 많이 쓰는 부분이 User 모델이다.
User 모델의 name 필드에 `uniqueness` 속성이 걸려 있다고 해보자.

```ruby
# app/models/user.rb
class User < ApplicationRecord
  validates :name, uniqueness: true
end
```

그리고 콘솔에서 다음과 같이 이름이 *Maru*인 유저를 생성하자

```ruby
User.create(name: 'Maru')
```

그리고 다음과 같이 콘솔에서 [valid?](https://apidock.com/rails/ActiveResource/Validations/valid%3F) 이용하여 테스트를 해보자

```ruby
> user = User.new(name: 'maru')
=> #<User id: nil, name: "taro", created_at: nil, updated_at: nil>

> user.valid?
# DEPRECATION WARNING: Uniqueness validator will no longer enforce case sensitive comparison in Rails 6.1. To continue case sensitive comparison on the :name attribute in User model, pass `case_sensitive: true` option explicitly to the uniqueness validator. (called from irb_binding at (irb):2)
User Exists? (0.9ms)  SELECT 1 AS one FROM `users` WHERE `users`.`name` = BINARY 'maru' LIMIT 1
=> true
```

여기서 SQL 구문의 WHERE 절에서 유효성 체크를 할 때 binary string으로 바꿔주는 `BINARY`가 붙어 있다는 것을 주목해야한다.

```sql
WHERE `users`.`name` = BINARY 'maru'
```

- _`binary`에 관해서는 아래 두 예제를 보면 바로 이해 할 수 있을 것이다._
  - [https://www.w3schools.com/sql/trymysql.asp?filename=trysql_func_mysql_binary2](https://www.w3schools.com/sql/trymysql.asp?filename=trysql_func_mysql_binary2)
  - [https://www.w3schools.com/sql/trymysql.asp?filename=trysql_func_mysql_binary3](https://www.w3schools.com/sql/trymysql.asp?filename=trysql_func_mysql_binary3)

<br><br><br>
객체가 담겨 있는 `user`를 저장해보자

```ruby

> user.save
=> true

> User.where(name: 'maru').count
=> 2
```

그럼 오늘의 주제인 `case_sensitive`를 사용해보자

#### 1. case_sensitive가 true 일 때

```ruby
class User < ApplicationRecord
  validates :name, uniqueness: { case_sensitive: true }
end
```

```ruby
> user = User.new(name: 'taRo')
=> #<User id: nil, name: "taRo", created_at: nil, updated_at: nil>

> user.valid?
  User Exists? (0.8ms)  SELECT 1 AS one FROM `users` WHERE `users`.`name` = BINARY 'taRo' LIMIT 1
=> true
```

#### 2. case_sensitive가 false 일 때

```ruby
class User < ApplicationRecord
  validates :name, uniqueness: { case_sensitive: false }
end
```

```ruby
> user = User.new(name: 'taRo')
=> #<User id: nil, name: "taRo", created_at: nil, updated_at: nil>
> user.valid?
  User Exists? (0.8ms)  SELECT 1 AS one FROM `users` WHERE `users`.`name` = 'taRo' LIMIT 1
=> false
```

- SQL 절에 `BINARY`가 빠져있다.

<br><br><br>

---

**Reference**

- [https://viblo.asia/p/try-a-new-feature-of-rails6-case-sensitive-XL6lA8eNZek](https://www.w3schools.com/sql/func_mysql_binary.asp)

- [https://www.w3schools.com/sql/func_mysql_binary.asp](https://www.w3schools.com/sql/func_mysql_binary.asp)

```

```
