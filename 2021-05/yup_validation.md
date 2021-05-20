### Form Validation With Yup

Việc `validate` các giá trị user nhập vào `Form` là một việc quan trọng vì nó có thể kiểm tra được các giá trị nhập vào đáp ứng được điều kiện đặt ra và tăng tính bảo mật cho chương trình.

`Yup` là một "khung" xác thực dạng Object của JavaScript. 
#### Sử dụng library Yup để validate các giá trị trong Form

1. Cài đặt Yup
	Cài đặt `Yup` với package manager tương ứng

	    npm install -S yup
	   
	 Sau đó, import vào và sử dụng
	 
	    import * as Yup from "yup";
	
	> Tham khảo chi tiết: [Yup](https://github.com/jquense/yup)
2. Sử dụng Yup
	Dữ liệu cần validate thường là một object JavaScript dạng `key-value` để dễ dàng truyền các `key-value` cho việc validate nên thường xây dựng schema Yup với kiểu `object()` như sau:
	
	```javascript
	let schema = Yup.object().shape({
	  name: Yup.string().required(),
	  age: Yup.number().required().positive().integer(),
	  email: Yup.string().email(),
	  website: Yup.string().url(),
	  createdOn: Yup.date().default(function () {
	    return new Date();
	  }),
	});
	```
	*Các kiểu dữ liệu `Yup` hỗ trợ validate:
	- `string`
	- `number`
	- `boolean`
	- `date`
	- `array`
	- `object`
	- `mixed` (kiểu dữ liệu cơ sở *base type*  các kiểu dữ liệu trên kế thừa từ kiểu này)
	
	Đặc biệt `Yup.string()` còn hỗ trợ validate email theo chuẩn  [RFC 2822](regexr.com/2rhq7)
	Nhưng nếu không đáp ứng yêu cầu thì có thể sử dụng method `string.matches()` và custom `regex` để validate email theo nhu cầu.
3. Một số method hay sử dụng
	`string`:
	- **`required`**: trả về một chuỗi thông báo khi giá trị là `null`, `undefined` hoặc `''` empty string
	- **`length`**: kiểm tra độ dài và trả về thông báo lỗi. Có thể lấy giá trị length để đưa vào thông báo với `${length}`
	- **`min/max`**: kiểm tra độ dài tối thiểu/tối đa của chuỗi. Có thể lấy giá trị để đưa vào thông báo với `${min}/${max}`
	- **`matches`**: kết hợp với regex để kiểm tra chuỗi

	`number`: 
	- **`min/max`**: kiểm tra giá trị tối thiểu/tối đa của số. Có thể lấy giá trị để đưa vào thông báo với `${min}/${max}`
	- **`integer`**: kiểm tra số là một số nguyên
	- **`positive`**: kiểm tra số là một số dương
	- **`negative`**: kiểm tra số là một số âm

	`date`:
	- **`min/max`**: kiểm tra giá trị hợp lệ với đầu vào là kiểu `Date()` của JS hoặc 1 chuỗi dạng ngày hợp lệ (đúng format)

	`array`:
	- **`of`**: tạo schema để validate các phần tử của mảng chứ không phải chính nó
	- **`min/max`**: kiểm tra độ dài tối thiểu/tối đa của mảng. Có thể lấy giá trị để đưa vào thông báo với `${min}/${max}`
	- **`length`**: kiểm tra độ dài của mảng

5. Một số method nâng cao

- **`addMethod`**: dùng để tạo một hàm tùy biến kiểm tra các giá trị của các giá kiểu dữ liệu mà Yup hỗ trợ.
	Ví dụ: Kiểm tra số nhập vào là số chẵn hay lẻ :smile:
	```javascript
	Yup.addMethod(Yup.number, 'checkOddNumber', function () {
	  return this.test('checkOddNumber', function (number) {
	    const isOdd = (number %  2 === 0);

	    if (!isOdd) {
	      return this.createError({
	        path: this.path,
	        message: 'Not odd number!',
	      });
	    }

	    return true;
	  });

	input_number: Yup.number().checkOddNumber(),
	```
*Chú ý:
Không sử dụng arrow function mà sử dụng function thông thường để có thể sử dụng con trỏ `this`. `this` ở callback function của `addMethod` là kiểu dữ liệu (schemaType) cơ sở *`mixed`* của Yup hỗ trợ nhiều method khác nhau, còn `this` trong callback function của `test` thuộc kiểu `NumberSchema`, tùy thuộc kiểu dữ liệu đầu vào, các method validate sẽ có đặc điểm riêng.

- **`when`**: một method dùng trong trường hợp muốn validate các giá trị phụ thuộc vào giá trị của một input khác.
	Vd: 
	```javascript
	const AVAILABLE_ORDER = 10;
	number_count: Yup.number().required('Number is required'),
	options: Yup.array().when(['number_count'], {
        is: (value) => value <= AVAILABLE_ORDER, // true or false (boolean)
        then: Yup.array().of(Yup.object().shape({
          value: Yup.string().required('Option is required'),
        })),
        
      }),
	```
- **`ref`**: dùng để trỏ đến giá trị của input khác
	Vd: giá trị của `confirmPassword` phải giống với giá trị `password`
	```javascript
	password: Yup.string().required('Password is required'),
	confirmPassword: Yup.string().oneOf([Yup.ref('password'), null], "Passwords don't match").required('Confirm Password is required'),
	```
- **`oneOf`**: method so sánh giá trị đầu vào phải thuộc 1 trong các giá trị được định nghĩa
- **`default`**: dùng để đặt giá trị mặc định cho input cần khi validate khi giá trị này là `undefined`, giá trị `null` hoặc `''` (empty string) không tính
- **`transform`**: dùng để chuyển đổi giá trị của input sang một giá trị mới
	Vd:
	```javascript
	const validBirthday = moment('2000-01-01', 'YYYY-MM-dd');
	function parseDateString(value, originalValue) {
	  const isValidDate = moment(originalValue).isValid();
	  const parsedDate = isValidDate
	    ? originalValue
	    : moment(originalValue, "YYYY-MM-dd");

	  return parsedDate;
	}
	// validate
	const schema = Yup.object({
	  birthday: Yup.string().transform(parseDateString).max(validBirthday),
	});
	```
	*Chú ý:
	- method `transform` sẽ được chạy trước khi validation
	- method `transform` nhận 2 giá trị *currentValue* là giá trị hiện tại trong quá trình chuyển đổi và *originalValue* là giá trị gốc truyền vào
	- không nên thay đổi giá trị gốc mà nên truyền giá trị gốc (raw) này vào method nếu muốn xử lý
- **`lazy`**: tạo một schema validate linh động tùy thuộc vào giá trị của input
	Vd: Tạo một object schema có với `optionObject` là một mảng chứa các giá trị cần validate có kiểu linh động tùy thuộc vào `value`
	```javascript
	const renderable = Yup.lazy((value) => {
	  switch (typeof value) {
	    case 'number':
	      return Yup.number().max(10);
	    case 'string':
	      return Yup.string().required('String is required!');
	    default:
	      return Yup.object().shape({
	        value: Yup.string().unique(),
	      });
	  }
	});

	const validateSchema = Yup.object().shape({
	  optionObject: Yup.array().of(renderable);
	});
	```
- **`typeError`**: trả về thông báo mặc định khi validate false do giá trị input không hợp lệ hoặc ko đáp ứng điều kiện
	Vd: 
	```javascript
	const validateSchema = Yup.object().shape({
	  number_order: Yup.number();
	});
	```
	Khi truyền giá trị không phải `number` (NaN) cho field `number_order` thì sẽ nhận được lỗi
	> this must be a `number` type, but the final value was: `NaN` (cast from the value `"number_order"`).

	Lúc này muốn trả về thông báo lỗi mong muốn thì sử dụng `typeError`
	```javascript
	const validateSchema = Yup.object().shape({
	  number_order: Yup.number().typeError("Invalid number");
	});
	```



