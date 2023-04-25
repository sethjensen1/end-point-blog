---
author: "Alina Arnautova"
title: "Avoiding N+1 queries in Rails applications"
github_issue_number: 1961
tags:
- rails
- performance
- orm
date: 2023-04-25
---

![Looking up a stairwell. A railing ascends at a steep diagonal from the bottom center upward and to the left. The next flight of stairs cuts the railing off, creating a right angle rotated by 45 degrees. On the right side of the image, a light on a wall reflects another diagonal on the right side wall, as well as a yellow glow around the whole stairwell.](/blog/2023/04/avoiding-n-plus-one-queries-in-rails-apps/2022-12-11_162540.webp)

<!-- Photo by Seth Jensen, 2022 -->

The N+1 query problem is a common performance issue if you’re using an Object Relational Mapping (ORM). It slows down page loading and leads to poor user experience and inefficient database usage. The N+1 query problem occurs when instead of grabbing all the data you need in one optimized, efficient query, your app ends up running multiple individual queries just to fetch the associated data.

### Why are N+1 queries so common in Rails?

1. Active Record makes it especially easy to work with databases, but it hides the complexity of underlying SQL queries very well.
2. Lazy loading: Active Record uses lazy loading by default, which means associations are loaded only when they are accessed. While this can be efficient in some cases, it often contributes to the emergence of N+1 problems.

### N+1 queries in detail

Let’s look at the specific examples of where and how an N+1 issue can occur. Let's say we have two models, `Author` and `Book`, with a one-to-many relationship:

```ruby
# app/models/author.rb
class Author < ApplicationRecord
  has_many :books
end

# app/models/book.rb
class Book < ApplicationRecord
  belongs_to :author
end
```

We want to display books along with their authors on our main page:

```ruby
# app/controllers/books_controller.rb
class BooksController < ApplicationController
  def index
    @books = Book.all
  end
end
```

```html
<!-- app/views/books/index.html.erb -->
<% @books.each do |book| %>
  <p>
    Title: <%= book.title %>, Author: <%= book.author.name %>
  </p>
<% end %>
```

For each book, a separate SQL query is executed to retrieve the author’s name. If we have 3 books, there are 4 queries in total: one to fetch all books and three additional queries for fetching associated authors. In the logs it would look something like this:

```ruby
SELECT "books".* FROM "books"
SELECT "authors".* FROM "authors" WHERE "authors"."id" = 1
SELECT "authors".* FROM "authors" WHERE "authors"."id" = 2
SELECT "authors".* FROM "authors" WHERE "authors"."id" = 3
```

This is a classic example of N+1 queries. N being the number of books plus 1 explicit query to fetch all books. 3 + 1 = 4 queries.

With three books, it doesn't seem like a big deal, but imagine if we had a thousand? Making 1001 requests doesn't sound that efficient. 

*Extra tip:* You can also use the `Bullet` gem to help you spot N+1 query issues.

### How to fix it?

Use the `includes` method to eager load associated data:

```ruby
@books = Book.includes(:author).all
```

By preloading the authors, we now consistently have two queries, even if the number of books increases to 1000. This is because the second query relies on the data from the first one to determine which authors to fetch.

```ruby
SELECT "books".* FROM "books"
SELECT "authors".* FROM "authors" WHERE "authors"."id" IN (1, 2, 3)
```

### Does this mean that we should always preload all associations?

Not at all. Preloading should be done thoughtfully and based on your application’s specific requirements. 
Eager-loading associations can improve performance by reducing the number of SQL queries, but it can also fetch more data than necessary, leading to increased memory usage and reduced performance if done carelessly.
