# Redis vs các kiểu dữ liệu

Redis là một loại non-relational database mà khi nhắc đến mọi người thường nhớ ngay đến các ưu điểm khi sử dụng như dưới đây:

Performance: Tất cả data đều được lưu trong memory, cho phép độ trễ thấp vs tốc độ truy cập dữ liệu. Không giống như các cơ sở dữ liệu truyền thống thì dữ liệu lưu ở memory không bắt buộc phải lưu trên disk nên giảm độ trễ xuống microsecnods. Do đó việc lưu trữ dữ liệu có hiệu suất cức nhanh vs các thao tác đọc ghi vs đáp ứng được hàng triệu thao tác trên mỗi giây. 2.Flexible data structures: Không giống như các kiểu key-value data stores thường giới hạn cấu trúc dữ liệu thì Redis cung cấp 5 loại data phổ biến bạn có thể chọn phù hợp vs ứng dụng của mình. ・Strings ・Lists ・Sets ・Hashes ・Sorted Sets

Simplicity and ease-of-use Redis hỗ trợ nhièu ngôn ngữ như Java, Python, PHP, C, C++, C#, JavaScript, Ruby vs nhiều ngôn ngữ khác. Việc thao tác vs dữ liệu chỉ cần một câu lệnh đơn giản.

Replication and Persistence: Redis sử dụng kiến trúc primary-replica và hỗ trợ sao chép không đồng bọ dữ liệu sang các replica server khác. Điều này cung cấp hiệu suất đọc được cải thiện đáng kể vs khôi phục nhanh hơn khi mà node primary gặp sự cố.

High availability and scalability Như ở trên cũng đã nói redis dùng kiến trúc primary-replica nó giúp bạn có thể build được hệ thống có tính khả dụng cao, cung cấp hiệu suất vs độ tin cậy nhất quán.

Open Source: Redis là open sources được support bởi cộng đồng rộng lớn, được tích hợp lên AWS.

Để sử dụng tốt bất kì loại database nào thì bạn cũng nên tìm hiểu kĩ cấu trúc dữ liệu mà hệ loại databse đó support. Chính vì thế mình muốn nói chi tiết hơn về các loại data struture mà Redis cung cấp.

# 1.Strings

Trong redis thì stirng là kiểu dữ liệu đơn giản nhất. Nhìn hình chú thích trên bản có thể thấy luôn được một giá trị được lưu trữ dưới dạng keys và values. Ví dụ hình trên thì key sẽ là “hello” còn values sẽ lưu trữ dưới dạng string với giá trị là “world” Các cmd mà mọi người hay sử dụng như dưới đây.

GET Lấy gái trị lưu trữ dựa vào key
SET Set giá trị lưu trữ theo key
DEL Xoá giá trị lưu trữ cho key( áp dụng cho tất cả các loại data structure)

# 2.Lists

Khác vs strings thì vs kiểu dữ liệu là lists thì gái trị được lưu trữ sẽ dưới dạng một dãy có thứ tự các giá trị string. Redis cung cấp các function giúp cho bạn dễ dàng thao tác được vs list như dưới đây.

LPUSH/RPUSH Push giá trị vào bên phái hoặc bên phải của list
LRANGE Lấy giá trị trong khoảng được chỉ định từ bên trái của list
LINDEX Lấy giá trị có index tính từ bên trái của chuỗi
LPOP/RPOP Lấy giá trị bên trái hoặc bên phải của list

# 3.Sets

Set thì tương tự như lists nhưng khác là sets được lưu trữ dưới dạng bảng băm dù cho nó không có cấu trúc. Cũng vì thế mà bạn không thì push hoặc pop giá trị trong set mà thay vào đó thì Redis cung cấp các bạn các function dưới đây để bạn thao tác vs dữ liệu

SADD: Thêm phần tử vào set
SMEMBERS Trả về giá trị lưu trong sets
SISMEMBER Kiểm tra xem có tồn tại giá trị trong sets không
SREM Xoá giá trị trong set nếu có tồn tại

# 4.Hashes

Với kiểu hashes thì redis lưu trữ data dưới đạng mapping của keys đến values, giá trị lưu trong values của hash là strings. Nhìn hình thì ta có thể thấy vs hash-key thì giá trị lưu lại dưới dạng hash ( mỗi gía trị con trong hash cũng sẽ có keys vs values). Nhìn hình khá là dễ hiểu đúng không các bạn. Redis thì cung cấp cho chúng ta các function có thể thao tác như dưới đây:

HSET Lưu trữ giá trị trong hash
HGET Lấy giá trị trong hash
HGETALL Lấy tất cả giá trị trong hash
HDEL Remove một key trong hash nếu tồn tại

# 5. Storted set

Storted set thì cũng giống như hashed lưu trữ giá trị con dưới dạng key vs values. Với Storted set thì key sẽ được gọi là member còn value thì sẽ gọi là scores( lưu trữ dưới dạng float-point numbers). Vì nó lưu trữ dưới dạng float-point number thì các bạn có thể tìm các giá trị trong khoảng dữ liệu mà bạn chỉ định. Tương tựng như các loại dữ liệu khác thì redis cũng cung cấp các cmd tương ứng giúp bạn thao tác dễ dàng vs kiểu dữ liệu này.

ZADD Thêm member vs score vào set
ZRANGE Lấy tất cả giá trị theo vị trí được lưu trong stored set
ZRANGEBYSCORE Lấy tất cả giá trị có score nằm trong khoảng được chỉ định
ZREM Xoá item theo member nếu tồn tại trong sets
END

Trên đây thì mình có giới thiệu các loại data structure cơ bản mà redis cung cấp, nếu bạn dùng aws redis thì có thể sẽ support thêm các kiểu dữ liệu khác. Nắm vững được các loại data structure sẽ giúp bạn chọn được kiểu dữ liệu tốt nhất cho ứng dụng của bạn.
