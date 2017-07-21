# Lay out the data architecture you'd need to implement to build the following scenarios:
***
### 1. A site for pet-sitting (watching someone's pet while they're gone). People can babysit for multiple pets and pets can have multiple petsitters.
##### Migrations:
```ruby
class CreatePeople < ActiveRecord::Migration[5.0]
  def change
    create_table :people do |t|
      t.string :name
      t.timestamps
    end
  end
end

class CreatePets < ActiveRecord::Migration[5.0]
  def change
    create_table :pets do |t|
      t.string :name
      t.references :owner, foreign_key: true
      t.timestamps
    end
  end
end

class CreatePetsittings < ActiveRecord::Migration[5.0]
  def change
    create_table :petsittings do |t|
      t.references :sitter, foreign_key: true
      t.references :sitted_pet, foreign_key: true
      t.timestamps
    end
  end
end
```
##### Models:
```ruby
class Person < ApplicationRecord
    has_many :owned_pets, class_name: "Pet", foreign_key: :owner_id
    has_many :petsittings, foreign_key: :sitter_id
    has_many :sitted_pets, through: :petsittings
end

class Pet < ApplicationRecord
  belongs_to :owner, class_name: "Person"
  has_many :petsittings, foreign_key: :sitted_pet_id
  has_many :sitters, through: :petsittings
end

class Petsitting < ApplicationRecord
  belongs_to :sitter, class_name: "Person"
  belongs_to :sitted_pet, class_name: "Pet"
end
```
***
### 2. You like hosting people for dinner so you want to build a dinner party invitation site. A user can create parties, invite people to a party, and accept an invitation to someone else's party.
##### Migrations:
```ruby
class CreatePeople < ActiveRecord::Migration[5.0]
  def change
    create_table :people do |t|
      t.string :name
      t.timestamps
    end
  end
end

class CreateParties < ActiveRecord::Migration[5.0]
  def change
    create_table :parties do |t|
      t.string :name
      t.references :host, foreign_key: true
      t.timestamps
    end
  end
end

class CreateInvitations < ActiveRecord::Migration[5.0]
  def change
    create_table :invitations do |t|
      t.references :party, foreign_key: true
      t.references :inviter, foreign_key: true
      t.references :invitee, foreign_key: true
      t.boolean :response
      t.timestamps
    end
  end
end
```
##### Models:
```ruby
class Person < ApplicationRecord
  has_many :parties_hosted, foreign_key: :host_id, class_name: "Party"
  has_many :invitations_sent, foreign_key: :inviter_id, class_name: "Invitation"
  has_many :invitations_received, foreign_key: :invitee_id, class_name: "Invitation"
  has_many :invitees, through: :invitations_sent
  has_many :inviters, through: :invitations_received
  has_many :parties_invited_to, through: :invitations_received, source: :party
end

class Party < ApplicationRecord
  belongs_to :host, class_name: "Person"
  has_many :invitations
end

class Invitation < ApplicationRecord
  belongs_to :party
  belongs_to :inviter, class_name: "Person"
  belongs_to :invitee, class_name: "Person"
end
```
***
### 3. You and your friends just love posting things and following each other. How would you set up the models so a user can follow another user?
##### Migrations:
```ruby
class CreatePeople < ActiveRecord::Migration[5.0]
  def change
    create_table :people do |t|
      t.string :name
      t.timestamps
    end
  end
end

class CreateFollows < ActiveRecord::Migration[5.0]
  def change
    create_table :follows do |t|
      t.references :follower, foreign_key: true
      t.references :followed, foreign_key: true
      t.timestamps
    end
  end
end

class CreatePosts < ActiveRecord::Migration[5.0]
  def change
    create_table :posts do |t|
      t.references :author, foreign_key: true
      t.text :content
      t.timestamps
    end
  end
end
```
##### Models:
```ruby
class Person < ApplicationRecord
  has_many :posts, foreign_key: :author_id
  has_many :follows_to, foreign_key: :follower_id, class_name: "Follow"
  has_many :follows_from, foreign_key: :followed_id, class_name: "Follow"
  has_many :followed, through: :follows_to
  has_many :followers, through: :follows_from
end

class Follow < ApplicationRecord
  belongs_to :follower, class_name: "Person"
  belongs_to :followed, class_name: "Person"
end

class Post < ApplicationRecord
  belongs_to :author, class_name: "Person"
end
```

