### 面向对象

#### 对象属性
##### 数据属性: 有四个特性，默认都是true,不能直接访问
* [[Configurable]]:	能否通过delete删除
* [[Enumerable]]: 	能否通过for-in枚举
* [[Writable]]: 		能否修改属性的值
* [[Value]]: 			包含这个属性的数据值，默认是undefined

**修改属性默认特性**：Object.defineProperty(object, name, feature),三个参数分别为属性所在的对象，属性的名字，和描述符对象
var person = {}
Object.defineProperty(person, 'name', {
	writable: false,
	configurable: false,
	value: 'Nicholas',
})
设置person的name属性为只读,如果赋新值，在非严格模式下会忽略错误，在严格模式下会抛出错误
configurable是总开关，一旦设置成false，就不能再配置其他的除了writable之外的三个值了,而且第一次设置的时候三个特性的默认值都为false

##### 访问器属性：不包括数据值，包含getter和setter函数,getter和setter不能一起和writable,value设置
* 访问器属性不能直接定义，必须通过Object.defineProperty()来定义
可以多次调用 Object.defineProperty()方法修改同一个属性，但在把 configurable特性设置为 false 之后就会有限制了。
```
Object.defineProperty(person, 'name', {
	get: function() {
		return this.name; //这样做会造成死循环,所以不能返回跟自己属性名一样的值
	},
	set: function(newValue) {
		this.name = newValue;
		this.edition +=1;
	}
})
```
* 定义多个属性：
```
Object.defineProperties(obj, {
	name: {
		value: 'xxxx',
	},
	age: {
		value: 'yyy',
	},
	child: {
		configurable: true,
		get: function() {
			return this.age * 0.5;
		},
		set: function(newValue) {
			this._child = newValue;	
		}
	}
}
```

获取属性的特性：Object.getOwnPropertyDescriptor(obj, name);




#### 创建对象：

##### 工厂模式：抽象了创建具体对象的过程。缺点：无法识别对象的类型
	function createPerson(name, age) {
		var p = new Object();
		p.name = name;
		p.age = age;

		return p;
	}

	var person = createPerson('Nicholas', '22');`

##### 构造函数模式：自定义对象类型的属性和方法。缺点：每个方法都要在每个示例上重新创建一遍
	function Person(name, age) {
		this.name = name;
		this.age = age;
	}

	var person = new Person('Nicholas', '22');

###### new的过程：
1. 创建一个新对象
2. 将构造函数的作用域赋给新对象，这样this就指向了新对象
3. 指向构造函数
4. 返回新对象

###### 模拟new的实现：
	function n(con) {
		var o = new Object();
		o.__proto__ = Person.prototype;	
		con.apply(o, Array.prototype.slice.call(arguments, 1));
		return o;
	}

##### 原型模式：创建的每个====函数=====都有一个prototype属性，指向原型对象
* 原型对象的contructor指向函数: `Person.prototype.constructor === Person;`
* isPrototypeOf(): 判断是否是实例的原型对象`Person.prototype.isPrototypeOf(person);``Object.getPrototypeOf(person);`
* hasOwnProperty(): 检测某个属性是存在于实例中还是原型中，在实例中返回true 	 `person.hasOwnProperty('name');`
* in: 检测是否可以访问到某个属性，不管在实例中还是原型中`'name' in person`
* Object.keys(obj):	返回对象所有可枚举属性的字符串数组
* Object.getOwnPropertyNames():	返回所有属性，包括不可枚举的属性

###### 字面量方式创建原型
	Person.prototype = {
		name: 'Nicholas',
		age: '333',
		// 重写构造函数，设置成不可枚举类型，否则默认的构造函数指向Object，无法通过constructor来确定对象类型了
		Object.defineProperty(Person.prototype, 'construcor', {
			enumerable: false,
			value: Person
		})
	}
##### 动态原型模式
	function Person(name, job) {
		this.name = name;
		this.job = job;
		if (!typeof this.sayName == 'function') {
			Person.prototype.sayName = function() {
				console.log(this.name);
			}
		}
	}
>写在if判断里是因为不需要每次实例化实例的时候都重新初始化一次

##### 为什么要使用动态原型模式，我觉得构造函数和原型模式组合就已经挺好了啊？
答：有的人就希望全部写在一起，即更好的封装性
##### 确定原型和实例的关系
确定原型和实例的关系
可以通过两种方式来确定原型和实例之间的关系。第一种方式是使用 instanceof 操作符，只要用这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回 true。以下几行代码就说明了这一点。
```
alert(instance instanceof Object); //true
alert(instance instanceof SuperType); //true
alert(instance instanceof SubType); //true
```
由于原型链的关系，我们可以说 instance 是 Object、 SuperType 或 SubType 中任何一个类型的实例。因此，测试这三个构造函数的结果都返回了 true。

第二种方式是使用 isPrototypeOf()方法。同样，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，因此 isPrototypeOf()方法也会返回 true，如下所示。
```
alert(Object.prototype.isPrototypeOf(instance)); //true
alert(SuperType.prototype.isPrototypeOf(instance)); //true
alert(SubType.prototype.isPrototypeOf(instance)); //true
```

#### 继承

##### 原型链
```
function SuperType(){
	this.property = true;
}

SuperType.prototype.getSuperValue = function(){
	return this.property;
};

function SubType(){
	this.subproperty = false;
}

//继承了 SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
	return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue()); //true
```
> **原型链的问题**<br>
> 1.包含引用类型的值得原型属性会被所有实例共享，所以要在构造函数里定义属性，在原型链继承中，父类的所有属性会被当做子类的原型属性而被子类的所有实例共享。。<br>
> 2.不能在不影响所有对象实例的情况下向超类型的构造函数传递参数。因此实践中很少会单独使用原型链 

##### 借用构造函数
	function SuperType(){
		this.colors = ["red", "blue", "green"];
	}

	function SubType(){
		//继承了 SuperType
		SuperType.call(this);
	}	
通过使用call() 或apply方法，实际上实在新创建的SubType实例的环境下调用了SuperType构造函数。这样一来，将会在新的SubType对象上执行SuperType函数中定义的所有对象初始化代码。**函数只不过是在特定环境中执行代码的对象**
> **问题**<br>
> 1.无法避免构造函数模式存在的问题——方法都在构造函数中定义，因此函数复用就无从谈起了<br>
> 2.而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式,因此实践中也是很少单独使用的**不懂**

##### 组合继承模式：将原型链和借用构造函数的技术组合到一块，是js中最常用的继承模式。
	function SuperType(name){
		this.name = name;
		this.colors = ["red", "blue", "green"];
	}

	SuperType.prototype.sayName = function(){
		alert(this.name);
	}

	function SubType(name, age){
		//继承属性
		SuperType.call(this, name);
		this.age = age;
	}

	//继承方法
	SubType.prototype = new SuperType();

	SubType.prototype.constructor = SubType;

	SubType.prototype.sayAge = function(){
		alert(this.age);
	};

>组合继承的不足：无论在什么情况下，都会调用两次超类型构造函数:一次是在创建子类型原型的时候，另一次是在子类型构造函数内部,
最终造成的结果是产生了两组name 和 color属性，一组在子类实例中，一组在原型中，实例中的属性会覆盖原型中的同名属性

##### 原型式继承：可以在不必预先定义构造函数的情况下实现继承，其本质是执行对给定对象的浅复制。而复制得到的副本还可以得到进一步改造
	function object(o) {
		function F() {};
		F.prototype = o;
		return new F();		
	}
>此方法的意义：以一个对象为基础创建另一个对象来实现继承

Object.create()如果只传入一个函数，行为与object()相同
Object.create()的第二个参数与Object.defineProperties()方法的第二个参数格式相同

	//创建对象的时候直接赋值
	Object.create(person, {name: {
		'value': 'greg',
	}})
适用范围：没有必要兴师动众的创建构造函数，只想让一个对象和另一个对象保持相似的情况下,可以使用原型式继承，需要注意的是引用类型的属性会被共享

##### 寄生式继承：与原型式继承非常相似，也是基于某个对象或某些信息创建一个对象，然后增强对象，最后返回对象。为了解决组合继承模式由于多次调用超类型构造函数而导致的低效率问
题，可以将这个模式与组合继承一起使用
	function createAnother(original){
		var clone = Object.create(original); //通过调用函数创建一个新对象
		clone.sayHi = function(){ //以某种方式来增强这个对象
			alert("hi");
		};
		return clone; //返回这个对象
	}

##### 寄生组合式继承：通过借用构造函数来继承属性，通过原型链的混成形式来继承方法
	
	function inheritPrototype(subType, SuperType) {
		var p = Object.create(SuperType.prototype)   //先复制一份父类原型
		p.prototype.constructor = subType  //重写原型的时候会失去默认的constructor属性，在这里添加一下
		subType.prototype = p;  //实现继承
	}

>特点：它的高效率体现在只调用了一次父类的构造函数，避免了在子类的原型上创建多余的、不必要的属性。
与此同时，原型链还能保持不变，可以正常使用instanctOf()和isPrototypeOf()
