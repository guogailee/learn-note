## 反射

- 定义：运行时动态获取类的属性和方法，动态调用对象的方法。
- 是各种框架设计的基础。

## 获取class

- Class.forName(String className)—首选
- 类的静态的class属性—需要提前知道类名
- instance.getClass()---需要提前拿到实例对象

## Class的方法

- getMethods()

  ```
  返回类or接口的所有public方法，包含super的
  ```

- getDeclaredMethods()

  ```
  返回类or接口的所有的public,protected,private方法，不包含super的
  ```

- getConstructor() && con.newInstance()/构造一个实例

- getDeclaredFields() 

  ```
  返回类or接口的所有的public,protected,private属性，不包含super的
  ```

## ReflectionUtils--帮我们封装了很多事情，简化代码

- doWithFields(Class<?> clazz, FieldCallback fc, FieldFilter ff)

  ```
  	public static void doWithFields(Class<?> clazz, FieldCallback fc, FieldFilter ff) {
  		// Keep backing up the inheritance hierarchy.
  		Class<?> targetClass = clazz;
  		do {
  			Field[] fields = getDeclaredFields(targetClass);
  			for (Field field : fields) {
  				if (ff != null && !ff.matches(field)) {
  					continue;
  				}
  				try {
  					fc.doWith(field);
  				}
  				catch (IllegalAccessException ex) {
  					throw new IllegalStateException("Not allowed to access field '" + field.getName() + "': " + ex);
  				}
  			}
  			targetClass = targetClass.getSuperclass();
  		}
  		while (targetClass != null && targetClass != Object.class);
  	}
  
  ```

- findField()

  ```
  	public static Field findField(Class<?> clazz, String name, Class<?> type) {
  		Assert.notNull(clazz, "Class must not be null");
  		Assert.isTrue(name != null || type != null, "Either name or type of the field must be specified");
  		Class<?> searchType = clazz;
  		while (!Object.class.equals(searchType) && searchType != null) {
  			Field[] fields = searchType.getDeclaredFields();
  			for (Field field : fields) {
  				if ((name == null || name.equals(field.getName())) && (type == null || type.equals(field.getType()))) {
  					return field;
  				}
  			}
  			searchType = searchType.getSuperclass();
  		}
  		return null;
  	}
  ```

## 应用

- 反射读取配置文件，应用程序更新的时候，不需要更改代码。