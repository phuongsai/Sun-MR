### Upload file với Firestore Storage
#### 1. Giới thiệu
Với *Cloud Firestore* dùng để lưu trữ cấu trúc database của ứng dụng, tính đặc thù và hạn chế dung lượng lưu trữ của `document` (1MB). Google cung cấp `Cloud Storage` dùng để lưu trữ các nội dung của người dùng như video, ảnh,... bổ  trợ tốt cho `Cloud Firestore`.

#### 2. Tạo đường dẫn (Reference) cho file lưu trữ
- Tương tự như cấu trúc đường dẫn của một `document` trong database của `Firestore`, chúng ta xác định đường dẫn để thao tác (`upload`, `download`, `delete`) với file.
- Đường dẫn bắt đầu từ `root` (**/**)
- Theo cấu trúc của Firebase thì các dữ liệu được tổ chức và phân cấp, tách nhau bởi dấu **`/`** nên không có thư mục thật sự trong `Cloud Storage`
> Ví dụ: Đường dẫn lưu trữ riêng cho một user có id `3eyG7UKzGn6uLxO7nMjN`:
> `const storageRef = firebase.storage().ref();`
> `/medias/3eyG7UKzGn6uLxO7nMjN/albums/kitty.png`

	*Các thao tác lấy đường dẫn (Reference)

- `root`: trỏ đến vị trí root (/)
	> Vị trí file hiện tại:
	> `storageRef.root` => `/`
- `parent`: trỏ đến cha trước 1 cấp của vị trí hiện tại
	> Vị trí file:
	> `storageRef.parent` => `albums`
- `child`: trỏ đến đường dẫn con với tham số truyền vào
	> Vị trí file:
	> `storageRef.child('albums')` => `albums`
	> `storageRef.child('albums/kitty.png')` => `albums/kitty.png`
- Có thể kết hợp các method lại với nhau theo một chuỗi (chain) để lấy đường dẫn
	> `storageRef.parent.child('kitty.png')` => `albums/kitty.png`
	> `root` là đường dẫn cơ sở nên sẽ không có parent
	> `storageRef.root.parent` => `null`
- `fullPath`: lấy đầy đủ đường dẫn
	> `storageRef.fullPath` => `medias/3eyG7UKzGn6uLxO7nMjN/albums/kitty.png`
- `name`: lấy tên file
	> `storageRef.name` => `kitty.png`

	*Chú ý:
	+ Tổng độ dài của đường dẫn (tính từ `root` (**/**)) là từ 1 - 1024 bytes với UTF-8 encoded
	+ Không chứa ký tự `\r` hoặc `\n`
	+ Không chứa các ký tự như `#`, `[`, `]`, `*`, hoặc `?` để tránh gặp lỗi khi thao tác với đường dẫn của `Cloud Storage`

#### 3. Upload file (Web)
`Cloud Storage` hỗ trợ upload các loại tệp của `JavaScript File`, `Blob APIs`, `Uint8Array` với method `put()` và `encoded string` như: `base64`, `base64url`, hoặc `data_url` với method `putString()`.
	

*Một số điểm cần lưu ý khi upload dữ liệu với chuỗi mã hóa (encoded string):
	- Dung lượng file lưu trữ của chuỗi mã hóa `base64` sẽ tăng khoảng 30% so với chuỗi không mã hóa, đồng thời thời gian xử lý cũng nhiều hơn.
	- Tham số thứ 2 của method `putString()` dùng để xác định định dạng của chuỗi mã hóa (`base64`, `base64url`, `data_url`, `raw`) thì tham số thứ 1 chỉ cần nội dung data của chuỗi mã hóa chứ không cần phải nhập thêm các tiền tố xác định.
> Ví dụ:
> Chuỗi `base64`: `data:image/gif;base64,R0lGODlhAQABAAAAACw=`
> Cú pháp upload với Cloud Storage:
> `storageRef.putString('R0lGODlhAQABAAAAACw=', 'base64', {contentType:’image/gif’})`
> Với tham số thứ 3 dùng để xác định metadata cho file

Quá trình upload file có thể đính kèm mô tả metadata cho file. `Cloud Storage` hỗ trợ custom các thông số cho file như: `cacheControl` (có lưu cache hay không), `contentDisposition` hỗ trợ đặt tên cho file khi save/download,...
	Reference: [File Metadata Properties](https://firebase.google.com/docs/storage/web/file-metadata#file_metadata_properties)

> Ví dụ:
> ```javascript
> const metadata = {
>   contentType:  'image/jpeg',
>   contentDisposition: 'custom-file-name',
> };
> storageRef.child('albums/kitty.png').put(file, metadata);
> ```

** Theo dõi quá trình upload file
Quá trình upload file lên `Cloud Storage` là một tiến trình của `UploadTask` với các phương thức điều khiển như: `pause()`, `resume()`, `cancel()` và `on()`.
Phương thức `on()` sử dụng `TaskEvent` để theo dõi trạng thái (`state`) và một callback function để xử lý cho trạng thái tương ứng với các `TaskState` như: `CANCELED`, `ERROR`, `PAUSED`, `RUNNING`, `SUCCESS`.
	Reference: [TaskState](https://firebase.google.com/docs/reference/js/firebase.storage#taskstate_1)
	
> Ví dụ về quá trình lắng nghe các thay đổi của state khi upload để theo dõi và xử lý khi gặp lỗi. 
> [Full Example](https://firebase.google.com/docs/storage/web/upload-files#full_example)

```javascript
// Create the file metadata
var metadata = {
  contentType: 'image/jpeg'
};

// Upload file and metadata to the object 'images/mountains.jpg'
var uploadTask = storageRef.child('images/' + file.name).put(file, metadata);

// Listen for state changes, errors, and completion of the upload.
uploadTask.on(firebase.storage.TaskEvent.STATE_CHANGED, // or 'state_changed'
  (snapshot) => {
    // Get task progress, including the number of bytes uploaded and the total number of bytes to be uploaded
    var progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
    console.log('Upload is ' + progress + '% done');
    switch (snapshot.state) {
      case firebase.storage.TaskState.PAUSED: // or 'paused'
        console.log('Upload is paused');
        break;
      case firebase.storage.TaskState.RUNNING: // or 'running'
        console.log('Upload is running');
        break;
    }
  }, 
  (error) => {
    // A full list of error codes is available at
    // https://firebase.google.com/docs/storage/web/handle-errors
    switch (error.code) {
      case 'storage/unauthorized':
        // User doesn't have permission to access the object
        break;
      case 'storage/canceled':
        // User canceled the upload
        break;

      // ...

      case 'storage/unknown':
        // Unknown error occurred, inspect error.serverResponse
        break;
    }
  }, 
  () => {
    // Upload completed successfully, now we can get the download URL
    uploadTask.snapshot.ref.getDownloadURL().then((downloadURL) => {
      console.log('File available at', downloadURL);
    });
  }
);
```


#### 4. Download file (Web)
- Sử dụng method `getDownloadURL()` để lấy đường dẫn URL của file hoặc có thể sử dụng phương thức `gs://` hay `https://` để tạo đường dẫn mới trỏ đến file.
> Lưu ý:
> method `https://` chứa các ký tự *[escaped](https://docs.microfocus.com/OMi/10.62/Content/OMi/ExtGuide/ExtApps/URL_encoding.htm)* trong đường dẫn
> `https://firebasestorage.googleapis.com/bucket/images%20stars.jpg`

- Để có thể download file trực tiếp về máy cần cấu hình CORS cho bucket lưu trữ của bạn. Đơn giản có thể lưu rule sau dưới tên `cors.json`
```javascript
[
  {
    "origin": ["*"],
    "method": ["GET"],
    "maxAgeSeconds": 3600
  }
]
```
Sau đó chạy câu lệnh với `gsutil` CLI để áp dụng rule trên cho bucket của bạn.
```gsutil cors set cors.json gs://<your-cloud-storage-bucket>```

> Tham khảo thêm thông tin về cài đặt nâng cao CORS cho `Cloud Storage` [Cross-origin resource sharing (CORS)](https://cloud.google.com/storage/docs/cross-origin)

#### 5. Delete File

Để xóa 1 file chúng ta cần trỏ đến đường dẫn lưu file sau đó sử dụng method `delete()` để xóa. Phương thức `delete()` trả về một `Promise`.

> Ví dụ:
```javascript
// Create a reference to the file to delete
var desertRef = storageRef.child('albums/kitty.png');

// Delete the file
desertRef.delete().then(() => {
  console.log('File deleted successfully!');
}).catch((error) => {
  console.log(error);
});
```

#### 6. Lấy danh sách các file

Với đặc điểm cấu trúc của *Firebase* thì việc lấy danh sách các file trong *"thư mục"* sẽ dựa vào tiền tố `prefix` của đường dẫn để lấy kết quả phù hợp tương ứng. Và tính chất query dữ liệu của *Firebase* là `shallow` lấy các kết quả cụ thể trong đường dẫn hiện tại chứ không lấy các kết quả có chứa dữ liệu phân cấp con.
> Ví dụ cấu trúc của 1 *Storage*:
> ```javascript
> /users/u1/avatar.jpg
> /users/u1/albums/photo1.jpg
> /users/u1/albums/photo2.jpg
> /users/u1/albums/private/pic1.png
> ```
> Kết quả khi lấy danh sách các file của `/users/u1/albums` là  `photo1.jpg` và `photo2.jpg`

Để lấy danh sách các file cần cung cấp tiền tố `prefix` đường dẫn và sử dụng method `list()` hoặc `listAll()` để lấy kết quả.
- Phương thức `listAll()` sẽ trả về hết tất cả các kết quả trong 1 lần truy vấn vậy nên cẩn thận sử dụng method này vì có thể gây ra vấn đề tiêu tốn bộ nhớ của client.
- Có thể phân trang `pagination` với method `list()`

#### 7. Xử lý lỗi khi thao tác với Cloud Storage
*Cloud Storage* cung cấp các `error message` được định nghĩa sẵn như: `storage/unknown`, `storage/unauthorized`,... để mô tả lỗi trong quá trình xử lý upload/download/delete file. Sử dụng `try-catch` để bắt các lỗi trả về từ `Promise`.
> Tham khảo các mã lỗi của Storage [Handle Error Messages](https://firebase.google.com/docs/storage/web/handle-errors#handle_error_messages)



