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

Rất nên sử dụng Hash để xác định rõ ràng các giá trị enum – nếu không Rails sẽ sử dụng chỉ mục của giá trị enum khi lưu trữ nó vào cơ sở dữ liệu.

Nếu bạn thay đổi thứ tự hoặc loại bỏ các tùy chọn, bạn sẽ phá vỡ ánh xạ tham chiếu.

Ngoài hộp, bạn có thể sắp xếp theo giá trị enum giống như bất kỳ cột nào khác và nó sẽ sử dụng giá trị.

```ruby
JobSubmission.all.order(:status)
# "SELECT \"job_submissions\".* FROM \"job_submissions\" ORDER BY \"job_submissions\".\"status\" ASC
```

Nhưng nếu bạn muốn thay đổi thứ tự kết quả khác đi? Bạn có thể thay đổi giá trị enum như kiểu thế này:

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

Thay vì đó, bạn có thể dùng `in_order_of` để thay đổi thứ tự order theo giá trị của enum.\
Điều tuyệt vời là gì? Việc sắp xếp thứ tự thực hiện trong SQL (sử dụng command CASE).

## Usage

Để sort bảng JobSubmission theo status enum với thứ tự khác, bạn có thể dùng:

```ruby
JobSubmission.all.in_order_of(:status, %w[accepted hold submitted draft rejected canceled])
```

Giá trị đầu là tên của cột cần sort, và giá trị thứ 2 là thứ tự mong muốn.

SQL query được sinh ra sẽ như thế này:

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

Bạn có thể thêm điều kiện sort.

```ruby
JobSubmission.all
  .in_order_of(:status, %w[accepted hold submitted draft rejected canceled])
  .order(:name)
```
