### Spread syntax (...) và một số tip thú vị
I. Giới thiệu
	Spread syntax (...) có bản chất là lấy tất cả các phần tử trong một biến có tính lặp lại như mảng, object hoặc chuỗi. 
II. Các ứng dụng hữu ích
1. Thay thế các đối số cần truyền vào input của function chỉ với một biến
Ví dụ, khi muốn tính toán phần tử lớn nhất trong 1 mảng ta có thể sử dụng cách sau:

```javascript
	const array = [10, 2, 7];
	const findMax = (arr) => {
		return Math.max.apply(null, arr);
	};
	const result = findMax(array);
```
Cách này chạy ổn nhưng có cách khác ngắn gọn hơn với (...):
```javascript
	const array = [10, 2, 7];
	const result = Math.max(...array);
```

2. Sử dụng để Clone ra một biến khác
Trong JS chúng ta có thể clone 1 biến mới bằng cách:
```javascript
	const array1 = [10, 2, 7];
	const array2 = array1;
```
Giả sử trường hợp chúng ta thay đổi giá trị biến array2
```javascript
	const array2.push(0);
```
Lúc này biến array1 cũng sẽ thay đổi theo do kiểu dữ liệu của array1 là array là một dạng của object mà trong JS object sẽ copy theo kiểu tham chiếu (Reference) chứ không phải theo biến.
Để giải quyết vấn đề này chúng ta sử dụng như sau:
```javascript
	const array1 = [10, 2, 7];
	const array2 = [...array1];
```
Copy tất cả các element có trong array1 và lưu với kiểu dữ liệu array ([])

3. Kết hợp nhiều mảng lại thành một
Khi muốn kết hợp nhiều mảng lại thành một và ở dạng flatten chúng ta có thể sử dụng các cách khác nhau:
Ví dụ convert 1 mảng như sau:
```javascript
	const arr = [[1, 2],[3, 4],[5, 6, 7, 8, 9]];
	// expected result: [1, 2, 3, 4, 5, 6, 7, 8, 9];
	// Method 1:
	const flattenedArray = [].concat.apply([], arr);
	// Method 2:
	const flattened = arr.reduce((acc, curVal) => {
		return acc.concat(curVal);
	}, []);
	// Method 3:
	const flattened = arr.flat();
	// Method 4:
	const flattened = [].concat(...arr);
```
4. Ứng dụng với Destructuring
 
```javascript
	const programingLanguage = {
		php: 'php',
		js: 'javascript',
		node_js: 'nodeJS',
		python: 'python',
		java: 'java',
	};
	
	const { php, node_js, ...other } = programingLanguage;
	// split to 3 variable
	// php, node_js, {js: "javascript", python: "python", java: "java"}
```
Lúc này biến programingLanguage vẫn có giá trị như cũ, khi chúng ta sử dụng (...) trong Destructuring biến programingLanguage nghĩa là sẽ clone các biến còn lại thành một object mới (đặt trong {}) thay vì chỉ cụ thể từng key cần được tách ra và gộp lại.

5. Thêm mới các giá trị vào một object

```javascript
	const programingLanguage = {
		php: 'php',
		js: 'javascript',
		node_js: 'nodeJS',
		python: 'python',
		java: 'java',
	};
	
	const { php, node_js, ...other } = programingLanguage;
	// split to 3 variable
	// php, node_js, {js: "javascript", python: "python", java: "java"}

	const currentYear = '2021';
	const newPL = {
		ruby: 'ruby',
		swift: 'Swift',
		go: 'go lang',
	};
	const topPL = { 
		...other, 
		...newPL,
		...(currentYear  ===  '2021'  &&  { flutter:  'flutter'  }),
	};
	// result: topPL = {
		js: "javascript", 
		python: "python",
		java: "java",
		ruby: "ruby",
		swift: "Swift",
		go: "go lang",
		flutter: "flutter",
	};
```
6.  Tách các ký tự trong chuỗi (string)

```javascript
	const greeting = "Hello";
	const greetingChars = [...greeting];
	console.log(greetingChars) ;
	// ["H", "e", "l", "l", "o"]
```
(...) 
