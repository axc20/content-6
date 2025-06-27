# Rails - models

```ruby
class EstateDocument < ApplicationRecord
  include DocumentCopyRequestServices
end

class Trust < ApplicationRecord
  include DocumentCopyRequestServices
end

class DocumentCopyRequest < ApplicationRecord
  belongs_to :source, foreign_type: :document_type, polymorphic: true
  belongs_to :target, foreign_type: :document_type, polymorphic: true
end
```

## ActiveRecord Callbacks

```ruby
class UsersController < ApplicationController
  # execute a specific method before a controller action is run
  # will be executed before every action in the controller,
  before_action :require_login
  # or before specifc actions you specify, using `only` or `except`
  before_action :require_login, only: [:index, :show]

  def index
  end

  def show
  end

  private

  def require_logn
  end
end
```

## Alias for a table

```ruby
class User < ApplicationRecord
  has_many :user_messages, class_name: 'Message'
end

class Message < ApplicationRecord
  belongs_to :user
end
```

## Latest record

```ruby
latest_record = Model.order(updated_at: :desc).last

latest_updated_at = Model.maximum(:updated_at)
latest_record = Model.find_by(updated_at: latest_updated_at)
```
