# devise 이메일 전송 없이 confirmable 처리하기

## 1. 기본 confirmable
[정식 문서](https://github.com/plataformatec/devise/wiki/How-To:-Add-:confirmable-to-Users)


## 2. 이메일 없이 confirmable 사용하기

#### models/user.rb
## ```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable

  # 메일 전송 X & 인증 처리 완료
  before_save :skip_confirmation!, if: :student?
  # 메일 전송 X & 인증 처리 전
  before_save :skip_confirmation_notification!, if: :teacher?

  devise :database_authenticatable, :registerable, :confirmable,
         :recoverable, :rememberable, :trackable, :validatable

end
```
중요한 메소드는 두 가지이다.
1. [`skip_confirmation!`](https://www.rubydoc.info/github/plataformatec/devise/Devise%2FModels%2FConfirmable:skip_confirmation!) : 이메일 전송하지 않고 인증 처리를 완료한다.
2. [`skip_confirmation_notification!`](https://www.rubydoc.info/github/plataformatec/devise/Devise%2FModels%2FConfirmable%3Askip_confirmation_notification%21): 이메일 전송하지 않고 인증 처리는 하지 않는다.

위 두 가지 메소드를 사용해서 개발하면 된다.
