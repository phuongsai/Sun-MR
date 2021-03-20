### Thao tác dữ liệu với FireStore

#### Prerequisite Firestore project setup
Xem hướng dẫn đăng kí và tạo mới một Firestore project cho môi trường Web tại đây [Firestore setup](https://firebase.google.com/docs/web/setup)
Các bước thực hiện đăng kí:
- Tạo một Firebase project với [Firebase console](https://console.firebase.google.com/) để lấy **Project ID**.
- Lựa chọn một trong các môi trường Firestore hỗ trợ như: Web, Android, iOS,... làm theo hướng dẫn tạo một **App** cho project và **database** cho App để kết nối với Firestore.
	> Tìm hiểu chi tiết về Project ID và App ID ở đây [Understand Firebase projects](https://firebase.google.com/docs/projects/learn-more#project-id)
- Cài đặt **Firestore CLI**  cùng với các thư viện cần thiết của Firestore vào project của bạn.
	> Xem hướng dẫn thực hiện với Codelab tại đây [firestore-web](https://firebase.google.com/codelabs/firestore-web/)
- Firestore là một module dựa trên Firebase JavaScript SDK để tương tác với các *Collection*, *Document* qua ``` firebase.firestore() ```
#### Init code for example
- Data Model
	```javascript
	restaurants (collection)
	--- restaurant-1 (document)
	------avgRating: 2.5
	------category: "cafe"
	------city: "NA"
	------name: "Best Fire"
	------numRatings: 4
	------photo: "https://url"
	------price: 32
	--- restaurant-2
	....
	```
- Init firestore
	```javascript
	var db = firebase.firestore();
	```
#### Firestore Database
Quản lí cấu trúc Firestore Database với *Rules* và *Indexes*:
- *Rules*: Quản lí phân quyền truy cập, đọc, ghi với từng *Collection*, *Document* tương ứng với *user* đăng nhập hiện tại.
	> Quản lí trực tiếp ở tab *Rules* với **Firestore Console**
	> Quản lí với code qua file *firestore.rules* ở local project sau đó deploy lên Firestore
	> ```javascript
	> service cloud.firestore {
	>   match /databases/{database}/documents {
	>     match /restaurants/{restaurant} {
	>       match /ratings/{rating} {
	>         allow read: if request.auth != null;
	>         allow write: if request.auth.uid == request.resource.data.userId;
	>       }
	>     }
	>   }
	> }
	> ```
- *Indexes*: Quản lí việc đánh *index* các field trong *Document* để query (get) hiệu quả và nhanh chóng hơn.
	> Quản lí trực tiếp ở tab *Indexes* với **Firestore Console**
	> Quản lí với code qua file *firestore.indexes.json* ở local project sau đó deploy lên Firestore
	> ```javascript
	> {
	>  "indexes": [
	>	 {
	> 	   "collectionGroup": "restaurants",
	>	   "queryScope": "COLLECTION",
	>  	   "fields": [
	> 		{ "fieldPath": "city", "order": "ASCENDING" },
	> 		{ "fieldPath": "avgRating", "order": "DESCENDING" }
	>		 ]
	>	 },
	> 	],
	>  "fieldOverrides": []
	> }
	> ```

## I> Đọc dữ liệu từ Firestore
### Đọc cả các Document trong một Collection
#### 1.1 Get tất cả dữ liệu về 1 lần
- Cú pháp:
	```javascript
	const query = db.collection("restaurants"); // trỏ đến collection cụ thể
	const restaurants = query.get()
		  .then((docs)  =>  {
		  if (!docs.empty)
		      docs.forEach((doc)  =>  {
		        if (doc.exists) {
		          // doc.data() is never undefined for query doc snapshots
		          console.log(doc.id,  " => ", doc.data());
		          return doc.data();
		        }
		    });
		  })
		  .catch((error)  =>  {
		      console.log("Error getting documents: ", error);
		  });
	```
Dữ liệu trả về khi get 1 collection là *CollectionReference* chứa một mảng các kết quả (nếu có) nên phải duyệt qua mảng các *DocumentReference* để đọc dữ liệu.
> Sử dụng property *empty* (boolean), *size* (number) để check kết quả collection có đang rỗng hay không.
> Sử dụng property exists của *DocumentReference* để check xem document đó có tồn tại hay không, method data() để lấy dữ liệu.
> Ngoài ra query dữ liệu từ Firestore luôn là một Promise, dựa vào các trạng thái _pending_, _fulfilled_, _rejected_ để xử lý kết quả.

#### 1.2 Đọc và lắng nghe dữ liệu thay đổi real-time với onSnapshot
Method onSnapshot sẽ thực hiện lắng nghe sự thay đổi từ Collection được chỉ định và trả về một ##### [QuerySnapshot](https://firebase.google.com/docs/reference/js/firebase.firestore.QuerySnapshot)
- Cú pháp:
	```javascript
	const query = db.collection("restaurants");
	const restaurants = query.get()
	  .onSnapshot((querySnapShot) => {
	    const { empty, size } = querySnapShot;
	    if (!empty) { // check snapshot is empty or not
	       const docs = querySnapShot.docChanges().map((doc) => ({
	         ...doc.doc.data(),
	         id: doc.doc.id,
	    }));
	    return docs;
	   }
	  });
	```
Mỗi khi có sự thay đổi trong *Collection* khi có document được *added*, *removed*, hoặc *modified* thì sẽ trả về một snapshot mới.
Để xem sự thay đổi của Collection sử dụng method `docChanges()` kết quả trả về một mảng các *document* có sự thay đổi so với snapshot trước.
`onSnapshot()` luôn lắng nghe sự thay đổi trên Collection điều này có ảnh hưởng đến số lượng đọc/ghi của bộ đếm Firestore nên lúc nào không muốn lắng nghe sự thay đổi nữa thì sử dụng method `unsubscribe()` để loại bỏ *listerner*.

### Đọc 1 Document cụ thể trong Collection
- Cú pháp:
	```javascript
	const query = db.collection("restaurants").doc("docABC"); // trỏ đến document trong collection
	const restaurants = query.get()
		  .then((doc)  =>  {
		  if (doc.exists) {
	            console.log(doc.id,  " => ", doc.data());
	            return doc.data();
		   }
		  .catch((error)  =>  {
		    console.log("Error getting documents: ", error);
		  });
	```
Kết quả trả về là một `DocumentReference`, sử dụng property `id` để lấy id hiện tại của document, method `data()` để lấy kết quả.
- Tương tự như *Collection* có thể sử dụng method `onSnapshot()` để lắng nghe sự thay đổi của 1 document.
	```javascript
	const query = db.collection("restaurants").doc("docCDE");
	const restaurants = query.get()
	  .onSnapshot((documentSnapShot) => {
	    const { exists } = documentSnapShot;
	    if (exists) { // check snapshot is empty or not
		console.log(documentSnapShot.id,  " => ", documentSnapShot.data());
		return documentSnapshot.data();
	   }
	  });
	```

### Query dữ liệu có điều kiện
Để query dữ liệu có điều kiện trong một *Collection* chúng ta sử dụng các methods trong `CollectionReference` như: **where**, **limit**,  **orderBy**, ...
> Để đảm bảo performance khi query thì Firestore sẽ yêu cầu đánh **index** cho các filed có trong câu query. Firestore hỗ trợ đánh index mỗi field theo `ASC` và `DESC`
- `where`: query dữ liệu có điều kiện
	> Cú pháp: collectionRef.where("field" , "operator" , "value");
	> Ex:
	> ```javascript
	> 	const restaurantsRef = db.collection("restaurants");
	> 	const query = restaurantsRef.where("category", "==", "cafe");
	> ```
	Firestore hỗ trợ các toán tử so sánh sau:
	-   `<`  less than
	-   `<=`  less than or equal to
	-   `==`  equal to
	-   `>`  greater than
	-   `>=`  greater than or equal to
	-   `!=`  not equal to
		>```javascript
		> const query = restaurantsRef.where("category", "!=", false);
		> ```
		> Toán tử **!=** không so sánh được với giá trị `null` do `null` được xem là `undefined`

	Các toán tử tìm kiếm trong array
	-   `array-contains`
	-   `array-contains-any`
	-   `in`
	-   `not-in`
	> ```javascript
	> const query = restaurantsRef.where("category", in, ["cafe", "ram"]);
	> ```
	> - Mảng chứa các giá trị cần tìm kiếm không vượt quá 10 phần tử.
	> - `array-contains` sử dụng logic AND để tìm kiếm các kết quả còn `array-contains-any` sử dụng logic OR để tìm kiếm. Kết quả của `array-contains-any` không bị trùng lặp.
	> - Không thể kết hợp `array-contains` và `array-contains-any` trong cùng một query.

	**Chú ý**
	> Kết quả truy vấn sẽ so sánh với các field trong điều kiện mà có tồn tại trong *document* và giá trị là các kiểu dữ liệu Firestore hỗ trợ bao gồm cả chuỗi rỗng `""`, `null`, `NaN`
	> Không thể kết hợp 2 toán tử so sánh `not-in` and `!=` trong cùng một câu query.
	> Kết hợp các toán tử so sánh `<`, `<=`, `>`, `>=` và `!=`, `not-in` trong query phải cùng 1 field

- `orderBy`: sắp xếp thứ tự của kết quả trả về
	```javascript
	const query = restaurantsRef.orderBy("category").orderBy("price", "desc");
	```
	> - Mặc định sắp xếp theo ASC.
	> - Các field phải được đánh index mới query thành công.
	> - Có thể kết hợp `orderBy` theo nhiều field khác nhau
	> - Khi kết hợp với toán tử tìm trong khoảng `<`, `<=`, `>`, `>=` thì phải `orderBy` theo field cần tìm.
	> ```javascript
	> restaurantsRef.where("price", ">", 20).orderBy("price", "desc");
	> ```
- `limit` : kết hợp với `orderBy` để giới hạn số lượng kết quả trả về
	```javascript
	restaurantsRef.where("price", ">", 20).orderBy("price", "desc").limit(10);
	```
## II> Ghi dữ liệu với Firestore
Sử dụng các methods của `CollectionReference` để ghi data vào Firestore
	- Có thể tạo 1 document với ID cụ thể do người dùng đặt.
	- Tạo 1 document với ID được tạo tự động.
Ghi dữ liệu vào một document với các phương thức
- `set()`: tạo document với ID tự đặt
	```javascript
	const data = {
	  category: 'tea',
	  price: 12,
	};
	restaurantsRef.doc("docABC").set(data, { merge: true });
	```
	> - Nếu document `docABC` không tồn tại sẽ thực hiện tạo mới document với các field và giá trị tương ứng
	> - Nếu document `docABC` tồn tại thì mặc định sẽ thực hiện **overwrite** dữ liệu (xóa hết các field tồn tại trong document và ghi dữ liệu mới). Nếu có tùy chọn `merge: true` thì giữ nguyên các field đã có và thực hiện **update** dữ liệu mới.
- `add()`
	```javascript
	const data = {
	  category: 'tea',
	  price: 12,
	};
	restaurantsRef.doc("docABC").add(data);
	```
	> Tương đương với `set()`
- `update()`
	Cập nhật giá trị các field mà không ghi đè toàn bộ một document.
	Phải trỏ đến một document cụ thể mới thực hiện update được.
	```javascript
	const data = {
	  category: 'milktea',
	  price: 14,
	};
	restaurantsRef.doc("docABC").update(data);
	```
	> - Khi update các giá trị trong một `map` thì phải chỉ rõ tên field - key nếu không sẽ thực hiện ghi đè dữ liệu mới trong field có kiểu `map` đó.
	> - Sử dụng các method `arrayUnion()` , `arrayRemove()` để update dữ liệu có kiểu `array`

Tạo document ID tự động
	- Sử dụng method `add()` trực tiếp với Collection mà không qua tên của Document nào cả
```javascript
const data = {
  category: "new",
  price: 10,
};
db.collection("restaurants").add(data);
```
hoặc tạo một document rỗng trước rồi thêm dữ liệu vào sau
```javascript
const data = {
  category: "new",
  price: 10,
};
const docRef = db.collection("restaurants").doc();
docRef.set(data);
```

## III> Xóa dữ liệu Firestore
Thực hiện xóa dữ liệu với Firestore qua method `delete()`
#### 1.1 Xóa một Document
Trỏ trực tiếp đến tên Collection và Document cụ thể muốn xóa
```javascript
db.collection("restaurants").doc("docABC").delete()
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.log(error);
  });
```
#### 1.2 Xóa một Field trong Document
Để xóa 1 field trong document sử dụng class `firebase.firestore.FieldValue` và method `FieldValue.delete()`

```javascript
const FieldValue = firebase.firestore.FieldValue;
db.collection("restaurants").doc("docABC").update({
  price: FieldValue.delete()
}).then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.log(error);
  });
```

#### 1.3 Xóa một Collection
Việc thực hiện xóa một *Collection* cần chú ý:
- Một *Collection* bao gồm nhiều *Document* và *Sub-Collection* nên khi xóa hãy sử dụng `batch()` chia nhỏ (chunk) để xóa tránh lỗi *out of memory*.
- Việc xóa Collection chưa được hỗ trợ ở Web, Android, iOS client.
