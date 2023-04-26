# Tip: Sorting ActiveRecord results by enum values (in SQL)

> Tip: Order kết quả active record theo giá trị enum (trong SQL)

`enums` trong rails là một cách model hoá dữ liệu như là `status` trong một model trong ActiveRecord. \
Họ cung cấp một tập các phương thức cho con người có thể đọc trong khi lưu kết quả là kiểu số nguyên trong database.

```ruby
class JobSubmission < ApplicationRecord
  enum status: {
    draft: 0,
    submitted: 1,
    hold: 2,
    rejected: 3,
    accepted: 4,
    canceled: 5
  }
end
```

It is highly recommended to use a Hash to explicitly define the enum values – otherwise Rails will use the index of the enum value when storing it to the database.
If you were were to change the order or remove options, you would break the reference mapping.

Out of the box, you can sort by the enum value like any other column and it will use the value.

```ruby
JobSubmission.all.order(:status)
# "SELECT \"job_submissions\".* FROM \"job_submissions\" ORDER BY \"job_submissions\".\"status\" ASC
```

But what if you wanted sort the results differently? You might be tempted to do something like this:

```ruby
class JobSubmission < ApplicationRecord
  STATUS_SORT = {
    accepted: 0,
    hold: 1,
    submitted: 2,
    draft: 3,
    rejected: 4,
    canceled: 5
  }

  def status_sort_value
    STATUS_SORT[status]
  end
end

JobSubmission.all.sort(&:status_sort_value)
```

Instead you can use the in_order_of to specify the sort order of the enum values. And the best part? The sorting will happen in SQL (using a CASE command under-the-hood).\
Thay vào đó, bạn có thể sử dụng `in_order_of` để order theo enums. Và điều gì sẽ xảy ra? Sorting sẽ thực hiện trong SQL ( sử dụng CASE command ).

## Usage

To sort the JobSubmission records by the status enum value in a different order, you can use:

```ruby
JobSubmission.all.in_order_of(:status, %w[accepted hold submitted draft rejected canceled])
```
The first argument is the column to sort by and the second argument is an ordered array of values.

The SQL generated for this query will be:

```sql
SELECT "job_submissions".*
FROM "job_submissions"
  WHERE "job_submissions"."status" IN (4, 2, 1, 0, 3, 5)
ORDER BY
  CASE "job_submissions"."status"
    WHEN 4 THEN 1
    WHEN 2 THEN 2
    WHEN 1 THEN 3
    WHEN 0 THEN 4
    WHEN 3 THEN 5
    ELSE 6
  END ASC
```
You can also add additional sorting to the relation. For instance, you would probably want to sort records by the status and then maybe alphabetically by the name column to break ties.

```ruby
JobSubmission.all
  .in_order_of(:status, %w[accepted hold submitted draft rejected canceled])
  .order(:name)
```