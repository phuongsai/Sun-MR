

## Tìm hiểu Firestore (phần 1)

### Cấu trúc Firestore



Firestore là một dạng cơ sở dữ liệu NoSQL có cấu trúc chính là các __Collection__, __Document__ và __Field__ để lưu dữ liệu.



![firestore-structure](https://firebase.google.com/docs/firestore/images/structure-data.png)



Dữ liệu từ một đối tượng sẽ được lưu trong _Document_ với mỗi thuộc tính là một _Field_ theo dạng _key-value_ tương tự JSON.

- Mỗi _Document_ có một ID duy nhất không trùng lặp (trong cùng một _Collection_), ID có thể đặt tên cụ thể hoặc được tạo tự động (auto-ID) với Firestore.

- Giới hạn dung lượng của 1 _Document_ là **1MB** và có tối đa không quá 20000 _Field_.

- Trong mỗi _Document_ có thể có nhiều sub-collections chứa các _documents_ con riêng. Tương tự đối với các _document_ trong _sub-collections_ nhưng số lượng _sub-collections_ lồng nhau không quá 100.

> Ví dụ: tên của _Document_ trong Firestore

> Tên cụ thể: `book-1`

> Tên tạo bởi firestore: `jH0Gbq19XxcixlXybi9C`



![firestore-collection-documents](https://4.bp.blogspot.com/-djTTPRRjwzw/XFHyXdg_0mI/AAAAAAAADU4/TEDA6qFSKuAvm27lghESOXApm28l1k_RwCLcBGAs/s1600/image8.png)




Tập hợp các __Documents__ chung được nhóm thành 1 __Collections__ nhờ đó có thể tạo một cấu trúc phân cấp để dễ dàng quản lí và truy vấn dữ liệu hơn.

- Tất cả các _Collections_ đều được lưu ở _Root Collection_ (**/**)

-  _Document_ có giới hạn 1MB nên các _Field_ có tính chất lưu trữ như kiểu master data (object, map, array) có thể chuyển sang một _collection_ ở root để lưu trữ.

>**BIG** collection (bao gồm nhiều _Document_)

>**SMALL** document (_< 1MB_)



![collection-documents-structure](https://4.bp.blogspot.com/-3EdZLM79eko/XFHyh49gjKI/AAAAAAAADU8/wI0w0h3fHhotrBeeS0C5nbdTXHsmn_fLACLcBGAs/s1600/image3.png)



### Dữ liệu lưu trữ

![collection-documents-demo](https://images.viblo.asia/dedd0b70-75e0-4f1d-8315-de7d08febb9f.png)


```

restaurants

  60N7PqEmotRw3T305y8d

  avgRating: 4

  category: "Coffee"

  name: "Espresso"

  price: 20

65SFE7YGq5WDx1HAsqsn

  avgRating: 3.5

  category: "Tea"

  name: "Green Tea"

  price: 15

```



### Document Path

Đường dẫn dùng để truy vấn dữ liệu có dạng:



>/__Collection__/_Document-ID_/__Sub-Collection__/_Document-ID_

>> /__restaurants__/65SFE7YGq5WDx1HAsqsn/__ratings__/diwXwQEzFbrgPkwb8B81




### Thuật ngữ tương ứng

| Firestore (NoSQL) | SQL |

| ------------- |:-------------:|

| Collection | Table |

| Document | Row |

| Field | Column |




## Các kiểu dữ liệu trong Firestore



* Các kiểu dữ liệu lưu trữ được hỗ trợ trong Firestore

  -- string

  -- number

  -- map

  -- array

  -- null

  -- boolean

  -- timestamp

  -- geopoint

  -- reference



> Link tham khảo:

>  [Firestore Data type](https://firebase.google.com/docs/firestore/manage-data/data-types)

### Thứ tự ưu tiên của các kiểu dữ liệu

Khi query dữ liệu, các kiểu dữ liệu có thứ tự ưu tiên trong kết quả trả về như sau:

1. Null values

2. Boolean values

3. Integer and floating-point values, sorted in numerical order

4. Date values

5. Text string values

6. Byte values

7. Cloud Firestore references

8. Geographical point values

9. Array values

10. Map values



### Đặc điểm của kiểu dữ liệu map

- Có thể xem như là một object trong một _document_ vì cũng lưu dữ liệu theo key-value.

- Có thể đánh index và query các sub-filed trong map. Nếu loại bỏ index của map thì các subfiled cũng bị mất index và không thể query được nữa.

- Luôn áp dụng việc lưu theo thứ tự các filed trong map, ưu tiên so sánh key trước rồi đến value, sau đó là length của key-value đó.



#### map vs sub-collection

Ý nghĩa đánh index khác nhau là chủ yếu.

-  __map__ đánh index dựa vào key-value. Dữ liệu lúc fetch về sẽ fetch tất cả các key-value có trong map.

-  __sub-collection__ có tên collection và đánh index dựa vào _documentID_. Có thể query lấy các dữ liệu cần thiết tránh việc fetch dữ liệu thừa.

Firestore quy định _documentID_ phải duy nhất nên nếu trùng ID sẽ báo lỗi còn _map_ sẽ thực hiện ghi đè dữ liệu mới (xóa dữ liệu cũ). Ngoài ra việc sử dụng _sub-collection_ hay _map_ tùy thuộc vào cấu trúc dữ liệu muốn lưu trữ, _sub-collection_ sẽ thuận tiện cho việc mở rộng hơn là _map_ bị giới hạn theo dung lượng lưu trữ của _document_. Lưu trữ các dữ liệu liên quan vào _map_ sẽ thuận tiện cho việc truy vấn hơn _collection_.



#### map vs array

Khác biệt chính của __map__ và __array__ là:

-  __map__ đánh index dựa vào key

-  __array__ đánh index theo số tăng dần (auto increment)

Vậy nên lúc thực hiện CRUD dựa vào index là key cụ thể sẽ chính xác hơn là index theo số. Firestore không hỗ trợ truy vấn dữ liệu dựa vào index của mảng, thêm/xóa dữ liệu ở vị trí cố định.



>Ví dụ:

>var number = ['hi', 'two', 'one'];

>**~~collection('restaurants').where(number[2] === 'one')~~**



Vậy nên _array_ thường áp dụng để lưu một mảng các #hashtag để phân loại sẽ tối ưu hơn.



### top-level vs hierarchically (phân cấp)

| Top-level | hierarchically |

| ------------- |:-------------:|

![top-level-hierarchically-collection](https://3.bp.blogspot.com/-muaAg5LVkgw/XQqshwT6OyI/AAAAAAAADqM/Axcfy61jDg4R120Vlb-dk7ciFAQJAwh6gCLcBGAs/s1600/12.png)



**Top-level**

```

name: 'ABC'

rating: 3

street_name: "Ly Thuong Kiet"

street_number: "16"

state: "Hai Chau"

city: "Da Nang"

```



**hierarchically**

```

name: 'ABC'

rating: 3

address: {

  street_name: "Ly Thuong Kiet"

  street_number: "16"

  state: "Hai Chau"

  city: "Da Nang"

}

```



**Sự khác nhau giữa cách cấu trúc dữ liệu theo top-level và hierarchically**

- Khác nhau về câu query để lấy dữ liệu

-- Top-level: dùng _where, orderBy_ để chỉ rõ điều kiện truy vấn, có thể lấy dữ liệu ở các _documentID_ khác nhau, tốc độ truy vấn sẽ chậm hơn một chút.

-- hierarchically: sử dụng đúng _documentID_ để lấy dữ liệu.

- Phân cấp (hierarchically) theo sub-collection bảo mật hơn về việc phân quyền cho từng sub-collection.