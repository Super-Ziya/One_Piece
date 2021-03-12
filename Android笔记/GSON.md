### GSON

#### 1、五个注解

- @Expose：表示某个成员变量暴露于 JSON 序列化/反序列化。只需在 GsonBuilder 中配置`excludeFieldsWithoutExposeAnnotation()` 才会生效。配置后，只有使用`@Expose`注解的成员变量才会参与序列化/反序列化工作

  ```java
  public class User {
      @Expose
      private String firstName;	//参与序列化(JAVA对象转为JSON)/反序列化（JSON转为JAVA对象）
  
      @Expose(serialize = false) 
      private String lastName;	//参与反序列化
  
      @Expose (serialize = false, deserialize = false) 
      private String emailAddress;	//不参与
  
      private String password;	//不参与
  }
  ```

- @SerializedName：表示某个成员变量序列化/反序列化的名字，无需在 GsonBuilder 中配置就能生效，甚至会覆盖掉 `FieldNamingPolicy`，保持驼峰原则

  ```java
  public class MyClass {
      @SerializedName("name")	//a => name
      String a;
      @SerializedName(value="name1", alternate={"name2", "name3"}) 
      String b;
      String c;
      public MyClass(String a, String b, String c) {
       this.a = a;
       this.b = b;
       this.c = c;
      }
  }
  
  MyClass target = new MyClass("v1", "v2", "v3");
  Gson gson = new Gson();
  String json = gson.toJson(target);
  //输入的json如下
  {"name":"v1","name1":"v2","c":"v3"}
  //同理，以下的json数据都能成功反序列化为MyClass。
  {"name":"v1","name1":"v2","c":"v3"}
  {"name":"v1","name2":"v2","c":"v3"}
  {"name":"v1","name3":"v2","c":"v3"}
  ```

- @Since：表示某个成员变量从哪个版本开始生效，只在 GsonBuilder 中配置了 `setVersion()` 才生效

  ```java
  public class User {
      private String firstName;//一直参与
      private String lastName;//一直参与
      @Since(1.0)
      private String emailAddress;//当前版本>=1.0时才会参与序列化/反序列化，否则忽略
      @Since(1.0)
      private String password;//当前版本>=1.0时才会参与序列化/反序列化，否则忽略
      @Since(1.1)
      private Address address;//当前版本>=1.1时才会参与序列化/反序列化，否则忽略
  }
  ```

- @Until：表示某个成员变量从哪个版本开始失效，只在 GsonBuilder 中配置 `setVersion()` 才生效

  ```java
  public class User {
      private String firstName;//一直参与
      private String lastName;//一直参与
      @Until(1.1)
      private String emailAddress;//当前版本<=1.1时参加序列化/反序列化
      @Until(1.1)
      private String password;//当前版本<=1.1时参加序列化/反序列化
  }
  ```

- @JsonAdapter：表示在某一个成员变量或者类上使用 TypeAdapter

  ```java
  public class UserJsonAdapter extends TypeAdapter<User> {
      @Override 
      public void write(JsonWriter out, User user) throws IOException {
          out.beginObject();
          out.name("name");
          out.value(user.firstName + " " + user.lastName);
          out.endObject();
      }
      @Override 
      public User read(JsonReader in) throws IOException {
          in.beginObject();
          in.nextName();
          String[] nameParts = in.nextString().split(" ");
          in.endObject();
          return new User(nameParts[0], nameParts[1]);
      }
  }
  
  private static final class Gadget {
      @JsonAdapter(UserJsonAdapter.class)
      public User user;	//应用到属性，按照 UserJsonAdapter进行序列化/反序列化
  }
  
  @JsonAdapter(UserJsonAdapter.class)	//应用到类，按照 UserJsonAdapter进行序列化/反序列化
  public class User {
      public final String firstName, lastName;
      private User(String firstName, String lastName) {
          this.firstName = firstName;
          this.lastName = lastName;
      }
  }
  ```



#### 2、JsonReader、JsonWriter

> Gson 中，Java 对象与 JSON 字符串之间的转换通过字符流操作。JsonReader 继承于 Reader 用来读取字符，JsonWriter 继承于 Writer 用来写入字符

```java
//从流中读取List<User>
public List<User> readJsonStream(InputStream in) throws IOException {
    JsonReader reader = new JsonReader(new InputStreamReader(in, "UTF-8"));
    try {
        return readUserArray(reader);//读取用户列表
    } finally {
        reader.close();//关闭流
    }
}

//读取List<User>
public List<User> readUserArray(JsonReader reader) throws IOException {
    List<User> users = new ArrayList<User>();
    reader.beginArray();//开始读取数组
    while (reader.hasNext()) {//是否还有下个元素
        users.add(readUser(reader));//读取下个元素
    }
    reader.endArray();//结束读取数组
    return users;
}

//读取User对象  
public User readUser(JsonReader reader) throws IOException {
    String name = null;
    int age = -1;
    List<Double> geo=null;
    reader.beginObject();//开始读取对象
    while (reader.hasNext()) {//是否还有下个元素
        String name = reader.nextName();//读取下一个json属性名
        //判断属性名是哪一个
        if (name.equals("name")) {
            name = reader.nextString();
        } else if (name.equals("age")) {
            age = reader.nextInt();
        } else {
            reader.skipValue();//忽略没有匹配到内容的值
        }
    }
    reader.endObject();//结束读取对象
    return new User(name, age,geo);
}

//将List<User>写入流中
public void writeJsonStream(OutputStream out, List<User> users) throws IOException {
    JsonWriter writer = new JsonWriter(new OutputStreamWriter(out, "UTF-8"));
    writer.setIndent("    ");//设置缩进风格(设置后写入的字符串保持缩进风格，如果没有设置，将会顶格打印）
    writeUsersArray(writer, users);//写入List<User>
    writer.close();
}

//写入List<User>
public void writeUsersArray(JsonWriter writer, List<User> users) throws IOException {
    writer.beginArray();//开始写入数组
    for (User user : users) {
        writeUser(writer, user);
    }
    writer.endArray();//结束写入数组
}
//写入User
public void writeUser(JsonWriter writer, User user) throws IOException {
    writer.beginObject();//开始写入对象
    writer.name("name").value(user.getName());
    writer.name("age").value(user.getAge());
    if (user.getGeo() != null) {
        writer.name("geo");
    } else {
        writer.name("geo").nullValue();
    }
    writer.endObject();//结束写入对象
}
```

#### 3、源码

##### （1）JsonToken：主要用于表示 JSON 字符串中的名字/值的结构

```java
public enum JsonToken {
    BEGIN_ARRAY,	//JSON array开始
    END_ARRAY,	//JSON array结束
    BEGIN_OBJECT,	//JSON object开始
    END_OBJECT,	//JSON object结束
    NAME,	//JSON 属性名,JsonReader#nextName/JsonWriter#name
    STRING,	//JSON 字符串
    NUMBER,	//JSON 数字，代表java中double,long,int
    BOOLEAN,	//JSON 布尔值
    NULL,	//表示 null
    END_DOCUMENT	//表示JSON流的结尾
}
```

##### （2）JsonScope：元素词法作用域，用来标识 JsonReader/JsonWriter 现在读/写到哪了

```java
final class JsonScope {
    static final int EMPTY_ARRAY = 1;	//没有元素的数组（相当于之前刚读了“[”），下一个元素一定不是逗号。
    static final int NONEMPTY_ARRAY = 2;	//非空数组（至少已经有一个元素），下一个元素不是逗号就是“]”
    static final int EMPTY_OBJECT = 3;	//空对象（刚读到“{”,一个name/value对都没有），下一个一定不是逗号。
    static final int DANGLING_NAME = 4;	//名字，下一个元素一定是值。
    static final int NONEMPTY_OBJECT = 5;	//非空对象（至少一个name/value对），下一个元素不是逗号就是“}”
    static final int EMPTY_DOCUMENT = 6;	//空文档，初识状态，啥也没读
    static final int NONEMPTY_DOCUMENT = 7;	//文档中有一个顶级的数组/对象
    static final int CLOSED = 8;	//文档已被关闭
}
```

##### （3）JsonWriter 

- 配置

```java
//设置缩进符号，打印出来的字符串将会拥有缩进风格
public final void setIndent(String indent)
//设置宽松的容错性（顶级值可以不是为object/array，数字可以为无穷）
public final void setLenient(boolean lenient)
//html转义
public final void setHtmlSafe(boolean htmlSafe)
//序列化空，默认true
public final void setSerializeNulls(boolean serializeNulls) 
```

- 用数组标识当前的写入状态

```java
private int[] stack = new int[32];
private int stackSize = 0;
{
    push(EMPTY_DOCUMENT);
}

//将当前状态保存栈顶
private void push(int newTop) {
    if (stackSize == stack.length) {//如果满了就扩大一倍
        int[] newStack = new int[stackSize * 2];
        System.arraycopy(stack, 0, newStack, 0, stackSize);
        stack = newStack;
    }
    stack[stackSize++] = newTop;
}

//查看栈顶的值
private int peek() {
    if (stackSize == 0) {
        throw new IllegalStateException("JsonWriter is closed.");
    }
    return stack[stackSize - 1];
}
```

- name(String name)

```java
public JsonWriter name(String name) throws IOException {
    if (name == null) {
        throw new NullPointerException("name == null");
    }
    if (deferredName != null) {
        throw new IllegalStateException();
    }
    if (stackSize == 0) {
        throw new IllegalStateException("JsonWriter is closed.");
    }
    deferredName = name;//赋值给deferredName
    return this;
}
```

- value() 有很多重载函数

```java
public JsonWriter value(String value) throws IOException {
    if (value == null) {//如果空的话，写入空值
        return nullValue();
    }
    writeDeferredName();//写入name。
    beforeValue();//写入“:”（会对上次状态进行校验）
    string(value);//写入value（会对特殊字符进行转码）
    return this;
}

private void writeDeferredName() throws IOException {
    if (deferredName != null) {
        beforeName();//校验等
        string(deferredName);//写入名字
        deferredName = null;
    }
}

public JsonWriter beginObject() throws IOException {
    writeDeferredName();//写入name（如果有）
    return open(EMPTY_OBJECT, "{"); //写入“{”
}

public JsonWriter endObject() throws IOException {
    return close(EMPTY_OBJECT, EMPTY_OBJECT, "}");	//调用close写入“}”,EMPTY_OBJECT,EMPTY_OBJECT用来比较在哪种状态下进行关闭
}
```

##### （4）JsonReader

- setlenient 容错性可忽略：
  - )]}'\n 前缀
  - 多个顶级值
  - 顶级值不是 object/array 类型
  - 数字类型为无穷数，或者不是个数字
  - 一行的结尾存在 // 或者 # 注释
  - C 语言风格的注释 /…/
  - name 用了单引号或者没用引号
  - string 用了单引号或者没用引号
  - 数组元素的分隔符用了 ; 而不是 ,
  - name 和 value 不是用：分隔，而是用 = 或 =>
  - name/value 对之间不是逗号分隔，而是 ; 分隔

##### （5）泛型

- \$Gson\$Types

```java
//规范化类型
public static Type canonicalize(Type type) {
    if (type instanceof Class) {//class
        Class<?> c = (Class<?>) type;
        return c.isArray() ? new GenericArrayTypeImpl(canonicalize(c.getComponentType())) : c;
    } else if (type instanceof ParameterizedType) {//泛型
        ParameterizedType p = (ParameterizedType) type;
        return new ParameterizedTypeImpl(p.getOwnerType(),p.getRawType(), p.getActualTypeArguments());
    } else if (type instanceof GenericArrayType) {//数组类型
        GenericArrayType g = (GenericArrayType) type;
        return new GenericArrayTypeImpl(g.getGenericComponentType());
    } else if (type instanceof WildcardType) {//通配符类型
        WildcardType w = (WildcardType) type;
        return new WildcardTypeImpl(w.getUpperBounds(), w.getLowerBounds());
    } else {
        // type is either serializable as-is or unsupported
        return type;
    }
}
//获取原始类型
public static Class<?> getRawType(Type type) {
    if (type instanceof Class<?>) {
        return (Class<?>) type;
    } else if (type instanceof ParameterizedType) {//泛型
        ParameterizedType parameterizedType = (ParameterizedType) type;
        Type rawType = parameterizedType.getRawType();
        checkArgument(rawType instanceof Class);
        return (Class<?>) rawType;
    } else if (type instanceof GenericArrayType) {//数组类型
        Type componentType = ((GenericArrayType)type).getGenericComponentType();
        return Array.newInstance(getRawType(componentType), 0).getClass();
    } else if (type instanceof TypeVariable) {//类型变量的上边界是Object
        return Object.class;
    } else if (type instanceof WildcardType) {//返回上边界
        return getRawType(((WildcardType) type).getUpperBounds()[0]);
    } else {
        String className = type == null ? "null" : type.getClass().getName();
        throw new IllegalArgumentException("Expected a Class, ParameterizedType, or "+ "GenericArrayType, but <" + type + "> is of type " + className);
    }
}
```

- \$Gson\$Preconditions

```java
public final class $Gson$Preconditions {
    private $Gson$Preconditions() {
        throw new UnsupportedOperationException();
    }

    public static <T> T checkNotNull(T obj) {//校验非空
        if (obj == null) {
            throw new NullPointerException();
        }
        return obj;
    }

    public static void checkArgument(boolean condition) {//校验是不是满足条件
        if (!condition) {
            throw new IllegalArgumentException();
        }
    }
}
```

- TypeToken，Gson 用 TypeToken 表示泛型

```java
public class TypeToken<T> {
    final Class<? super T> rawType;	//T的原始类型
    final Type type; //T 的类型
    final int hashCode;

    protected TypeToken() {
        //获取父类泛型的参数类型
        this.type = getSuperclassTypeParameter(getClass());
        //获取原始类型
        this.rawType = (Class<? super T>) $Gson$Types.getRawType(type);
        this.hashCode = type.hashCode();
    }

    public final Class<? super T> getRawType() {
        return rawType;
    }

    public final Type getType() {
        return type;
    }
    //省略部分源码
}

static Type getSuperclassTypeParameter(Class<?> subclass) {
    Type superclass = subclass.getGenericSuperclass();//获取父类泛型类型
    if (superclass instanceof Class) {
        throw new RuntimeException("Missing type parameter.");
    }
    ParameterizedType parameterized = (ParameterizedType) superclass;
    //获取泛型参数类型，即T的类型
    return $Gson$Types.canonicalize(parameterized.getActualTypeArguments()[0]);
}
```

##### （6）JsonElement：一个抽象类，代表着 JSON 中的元素类型。可以表示 JsonObject，JsonArray，JsonPrimitive，JsonNull

- JsonPrimitive，代表 java 中的基本数据类型

```java
public final class JsonPrimitive extends JsonElement {
    private static final Class<?>[] PRIMITIVE_TYPES = { int.class, long.class, short.class, float.class, double.class, byte.class, boolean.class, char.class, Integer.class, Long.class,Short.class, Float.class, Double.class, Byte.class, Boolean.class, Character.class };	//基本类型列表
    private Object value;	//值
    
    public JsonPrimitive(Boolean bool) {
        setValue(bool);
    }
    //赋值数字类型
    public JsonPrimitive(Number number) {
        setValue(number);
    }
    //赋值String
    public JsonPrimitive(String string) {
        setValue(string);
    }
    //赋值字符
    public JsonPrimitive(Character c) {
        setValue(c);
    }
    //赋值给value
    void setValue(Object primitive) {
        if (primitive instanceof Character) {
            char c = ((Character) primitive).charValue();
            this.value = String.valueOf(c);
        } else {
            $Gson$Preconditions.checkArgument(primitive instanceof Number || isPrimitiveOrString(primitive));
            this.value = primitive;
        }
    }
    //省略了部分源码
}
```

##### （7）TypeAdapter：一个抽象类，可以用来自定义类型转换

```java
public abstract class TypeAdapter<T> {
    //抽象方法，写value（an array, object, string, number, boolean or null）
    public abstract void write(JsonWriter out, T value) throws IOException;
    //抽象方法，读value
    public abstract T read(JsonReader in) throws IOException;
    //包装方法，帮你处理了空值，你可以不用担心空值问题
    public final TypeAdapter<T> nullSafe() {
        return new TypeAdapter<T>() {
            @Override 
            public void write(JsonWriter out, T value) throws IOException {
                if (value == null) {//如果为空，写入null
                    out.nullValue();
                } else {
                    TypeAdapter.this.write(out, value);
                }
            }
            @Override 
            public T read(JsonReader reader) throws IOException {
                if (reader.peek() == JsonToken.NULL) {
                    reader.nextNull();
                    return null;
                }
                return TypeAdapter.this.read(reader);
            }
        };
    }
    //省略了部分源码
}
```

- 继承于 TypeAdapter，实现相应方法就能实现自己的 TypeAdapter

```java
//在 GsonBuilder 中使用，不需要使用相关注解了，但也可以使用，注解优先级最高
Gson gson = new GsonBuilder()
    .registerTypeAdapter(XX.class,new XXTypeAdapter())
    .create();
```

- 空值处理

```java
//第一种：自己处理空值
public class UserJsonAdapter extends TypeAdapter<User> {
    @Override 
    public void write(JsonWriter out, User user) throws IOException {
        if (user == null) {//判断空值
            out.nullValue();
            retrun;
        }
        out.beginObject();
        out.name("name");
        out.value(user.firstName + " " + user.lastName);
        out.endObject();
    }
    @Override 
    public User read(JsonReader in) throws IOException {
        if (reader.peek() == JsonToken.NULL) {//判断空值
            reader.nextNull();
            return null;
        }
        in.beginObject();
        in.nextName();
        String[] nameParts = in.nextString().split(" ");
        in.endObject();
        return new User(nameParts[0], nameParts[1]);
    }
}
//第二种：使用 nullSafe
Gson gson = new GsonBuilder()
    .registerTypeAdapter(User.class,new UserJsonAdapter().nullSafe())
    .create();
```

- TypeAdapterFactory：用来创造一些相似类型的 TypeAdapter

```java
public interface TypeAdapterFactory {
    <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type);
}
```

- 继承 TypeAdapter 需要重写 write 和 read 相关方法，只想处理序列化和反序列化中的一种可以使用 JsonSerializer 和 JsonDeserializer 接口。可以在 @JsonAdapter 注解和 registerTypeAdapter 中注册使用

```java
//序列化
public interface JsonSerializer<T> {
    public JsonElement serialize(T src,Type typeOfSrc,JsonSerializationContext context);
}
//反序列化
public interface JsonDeserializer<T> {
    public T deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException;
}
```

##### （8）序列化策略

- LongSerializationPolicy：枚举类，指定长整型的序列化类型，默认 DEFAULT，STRING 两种类型，可继承它实现其他类型
- InstanceCreator：实例创造器，当反序列化需实例化对象，但对象没有默认构造方法，那就自定义自己的实例创造器。InstanceCreator 一般配合 ConstructorConstructor 一起使用

```java
public interface InstanceCreator<T> {
    public T createInstance(Type type);
}

class UserInstanceCreator implements InstanceCreator<User> {
    public User createInstance(Type type) {
        return new User(null, -1);
    }
}
```

- FieldNamingStrategy：提供一个自定义的字段命名机制

```java
public interface FieldNamingStrategy {
    public String translateName(Field f);
}
```

- FieldNamingPolicy：一个枚举类，实现了 FieldNamingStrategy，提供了一些默认的字段命名机制
  - IDENTITY：原始机制
  - UPPER_CAMEL_CASE：首字母大写的驼峰映射
  - UPPER_CAMEL_CASE_WITH_SPACES：用空格分隔的大写驼峰
  - LOWER_CASE_WITH_UNDERSCORES：下划线相连的小写映射
  - LOWER_CASE_WITH_DASHES：虚线相连的小写映射

- FieldAttributes：存取字段的属性
  - getName：获取字段名
  - getDeclaringClass：获取声明的类
  - getDeclaredType：获取字段的声明类型
  - getAnnotation：获取注解
- ExclusionStrategy：一个用于定义排除策略的的接口

```java
public interface ExclusionStrategy {
    //是否应该忽略该属性
    public boolean shouldSkipField(FieldAttributes f);
    //是否应该忽略该类
    public boolean shouldSkipClass(Class<?> clazz);
}
```

##### （9）其他

- Excluder：排除器，主要用于根据策略和注解来判断哪些字段应该被忽略

```java
private static final double IGNORE_VERSIONS = -1.0d;
public static final Excluder DEFAULT = new Excluder();
//默认忽略版本号
private double version = IGNORE_VERSIONS;
//默认以下修饰符的字段会被忽略
private int modifiers = Modifier.TRANSIENT | Modifier.STATIC;
private boolean serializeInnerClasses = true;//序列化内部类
private boolean requireExpose;//需要Expose注解？
//存放序列化排除策略
private List<ExclusionStrategy> serializationStrategies = Collections.emptyList();
//存放反序列化排除策略
private List<ExclusionStrategy> deserializationStrategies = Collections.emptyList();
```

- Primitives：一个工具类，用于在原始类型和包装类型间转化，wrap 包装，unwrap 解开

- ObjectConstructor：一个人通用的构造器接口

```java
public interface ObjectConstructor<T> {
  public T construct();
}
```

- ConstructorConstructor：保存实例构造器集合的类

```java
public final class ConstructorConstructor {
    //实例创造器集合
    private final Map<Type, InstanceCreator<?>> instanceCreators;
    //通过类型返回一个构造器
    public <T> ObjectConstructor<T> get(TypeToken<T> typeToken) {
        final Type type = typeToken.getType();//获取类型
        final Class<? super T> rawType = typeToken.getRawType();//获取真实类型
        //首先从集合中获取看有没有相同的Type，有就创造实例并返回
        final InstanceCreator<T> typeCreator = (InstanceCreator<T>) instanceCreators.get(type);
        if (typeCreator != null) {
            return new ObjectConstructor<T>() {
                @Override
                public T construct() {
                    return typeCreator.createInstance(type);
                }
            };
        }
        //然后根据原始类型取，去集合中取
        final InstanceCreator<T> rawTypeCreator = (InstanceCreator<T>) instanceCreators.get(rawType);
        if (rawTypeCreator != null) {
            return new ObjectConstructor<T>() {
                @Override
                public T construct() {
                    return rawTypeCreator.createInstance(type);
                }
            };
        }
        //获取默认构造方法
        ObjectConstructor<T> defaultConstructor = newDefaultConstructor(rawType);
        if (defaultConstructor != null) {
            return defaultConstructor;
        }
        //获取集合类型的构造器（Collection，EnumSet，Set，Queue，Map）
        ObjectConstructor<T> defaultImplementation = newDefaultImplementationConstructor(type, rawType);
        if (defaultImplementation != null) {
            return defaultImplementation;
        }
        // 使用不安全的分配器
        return newUnsafeAllocator(type, rawType);
    }
    //省略了部分源码
}
```

- JsonStreamParser：解析器，解析为 JsonElement
  - hasNext：是否有下一个元素
  - next：取下一个元素，返回 JsonElement

- Streams 内部使用 TypeAdapters.JSON_ELEMENT 写入/读取下一个 JsonElement

#### 4、GSON 源码

##### （1）构造方法

```java
Gson(final Excluder excluder, final FieldNamingStrategy fieldNamingStrategy, final Map<Type, InstanceCreator<?>> instanceCreators, boolean serializeNulls, boolean complexMapKeySerialization, boolean generateNonExecutableGson, boolean htmlSafe, boolean prettyPrinting, boolean lenient, boolean serializeSpecialFloatingPointValues, LongSerializationPolicy longSerializationPolicy, List<TypeAdapterFactory> typeAdapterFactories) {
    this.constructorConstructor = new ConstructorConstructor(instanceCreators);//实例构造器
    this.excluder = excluder;//排除器
    this.fieldNamingStrategy = fieldNamingStrategy;//字段命名策略
    this.serializeNulls = serializeNulls;//序列化空
    this.generateNonExecutableJson = generateNonExecutableGson;//生成不可执行前缀（用来防止攻击）
    this.htmlSafe = htmlSafe;//html转义
    this.prettyPrinting = prettyPrinting;//缩进打印
    this.lenient = lenient;//宽松的容错性
    List<TypeAdapterFactory> factories = new ArrayList<TypeAdapterFactory>();//TypeAdapter的工厂列表
    //以下都是往工厂列表加入 TypeAdapterFactory
    // 构建不能被重载的TypeAdapter
    factories.add(TypeAdapters.JSON_ELEMENT_FACTORY);
    factories.add(ObjectTypeAdapter.FACTORY);
    // 排除器必须在所有的用于自定义的typeAdapter之前
    factories.add(excluder);
    //用户自定义的typeAdapter工厂
    factories.addAll(typeAdapterFactories);
    //以下为默认的TypeAdapter
    factories.add(TypeAdapters.STRING_FACTORY);
    factories.add(TypeAdapters.INTEGER_FACTORY);
    factories.add(TypeAdapters.BOOLEAN_FACTORY);
    factories.add(TypeAdapters.BYTE_FACTORY);
    factories.add(TypeAdapters.SHORT_FACTORY);
    TypeAdapter<Number> longAdapter = longAdapter(longSerializationPolicy);
    factories.add(TypeAdapters.newFactory(long.class, Long.class, longAdapter));
    factories.add(TypeAdapters.newFactory(double.class, Double.class, doubleAdapter(serializeSpecialFloatingPointValues)));
    factories.add(TypeAdapters.newFactory(float.class, Float.class, floatAdapter(serializeSpecialFloatingPointValues)));
    factories.add(TypeAdapters.NUMBER_FACTORY);
    factories.add(TypeAdapters.ATOMIC_INTEGER_FACTORY);
    factories.add(TypeAdapters.ATOMIC_BOOLEAN_FACTORY);
    factories.add(TypeAdapters.newFactory(AtomicLong.class, atomicLongAdapter(longAdapter)));
    factories.add(TypeAdapters.newFactory(AtomicLongArray.class, atomicLongArrayAdapter(longAdapter)));
    factories.add(TypeAdapters.ATOMIC_INTEGER_ARRAY_FACTORY);
    factories.add(TypeAdapters.CHARACTER_FACTORY);
    factories.add(TypeAdapters.STRING_BUILDER_FACTORY);
    factories.add(TypeAdapters.STRING_BUFFER_FACTORY);
    factories.add(TypeAdapters.newFactory(BigDecimal.class, TypeAdapters.BIG_DECIMAL));
    factories.add(TypeAdapters.newFactory(BigInteger.class, TypeAdapters.BIG_INTEGER));
    factories.add(TypeAdapters.URL_FACTORY);
    factories.add(TypeAdapters.URI_FACTORY);
    factories.add(TypeAdapters.UUID_FACTORY);
    factories.add(TypeAdapters.CURRENCY_FACTORY);
    factories.add(TypeAdapters.LOCALE_FACTORY);
    factories.add(TypeAdapters.INET_ADDRESS_FACTORY);
    factories.add(TypeAdapters.BIT_SET_FACTORY);
    factories.add(DateTypeAdapter.FACTORY);
    factories.add(TypeAdapters.CALENDAR_FACTORY);
    factories.add(TimeTypeAdapter.FACTORY);
    factories.add(SqlDateTypeAdapter.FACTORY);
    factories.add(TypeAdapters.TIMESTAMP_FACTORY);
    factories.add(ArrayTypeAdapter.FACTORY);
    factories.add(TypeAdapters.CLASS_FACTORY);
    //复合的TypeAdapter
    factories.add(new CollectionTypeAdapterFactory(constructorConstructor));
    factories.add(new MapTypeAdapterFactory(constructorConstructor, complexMapKeySerialization));
    this.jsonAdapterFactory = new JsonAdapterAnnotationTypeAdapterFactory(constructorConstructor);
    factories.add(jsonAdapterFactory);
    factories.add(TypeAdapters.ENUM_FACTORY);
    factories.add(new ReflectiveTypeAdapterFactory(constructorConstructor, fieldNamingStrategy, excluder, jsonAdapterFactory));
    this.factories = Collections.unmodifiableList(factories);
}

public final class ObjectTypeAdapter extends TypeAdapter<Object> {
    //工厂类
    public static final TypeAdapterFactory FACTORY = new TypeAdapterFactory() {
        //该工厂用来创建Object类型的TypeAdapter
        @Override 
        public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {
            if (type.getRawType() == Object.class) {
                return (TypeAdapter<T>) new ObjectTypeAdapter(gson);
            }
            return null;
        }
    };
}
```

##### （2）接口实现

```java
//读取
@Override 
public Object read(JsonReader in) throws IOException {
    JsonToken token = in.peek();//查看元素类型
    switch (token) {
        case BEGIN_ARRAY://数组
            List<Object> list = new ArrayList<Object>();
            in.beginArray();//开始标识
            while (in.hasNext()) {//还有下一个元素
                list.add(read(in));
            }
            in.endArray();//结束标识
            return list;
        case BEGIN_OBJECT://对象
            Map<String, Object> map = new LinkedTreeMap<String, Object>();
            in.beginObject();
            while (in.hasNext()) {
                map.put(in.nextName(), read(in));
            }
            in.endObject();
            return map;
        case STRING:
            return in.nextString();
        case NUMBER:
            return in.nextDouble();
        case BOOLEAN:
            return in.nextBoolean();
        case NULL:
            in.nextNull();
            return null;
        default:
            throw new IllegalStateException();
    }
}
//写为json
@Override 
public void write(JsonWriter out, Object value) throws IOException {
    if (value == null) {
        out.nullValue();
        return;
    }
    //gson.getAdapter获取对应类型的TypeAdapter
    TypeAdapter<Object> typeAdapter = (TypeAdapter<Object>) gson.getAdapter(value.getClass());
    if (typeAdapter instanceof ObjectTypeAdapter) {
        out.beginObject();
        out.endObject();
        return;
    }
    typeAdapter.write(out, value);
}
```

##### （3）getAdapter

> 将 Java 对象序列化为 Json 字符串时，需要调用 `gson.getAdapter(XX)` 获取相应类型的转换器，按照 TypeAdapter 规定好的规则进行序列化/反序列化
>
> 流程：从 typeTokenCache 中获取有没有相应类型的 Adapter，第一次肯定没有，然后利用中间变量 FutureTypeAdapter 从工厂遍历，放入 typeTokenCache 中。FutureTypeAdapter 是一个代理 TypeAdapter，内部还是原 TypeAdapter 进行处理

```java
public <T> TypeAdapter<T> getAdapter(TypeToken<T> type) {
    //从 typeTokenCache 的 Map 中取出
    //NULL_KEY_SURROGATE 表示 Object 类型
    TypeAdapter<?> cached = typeTokenCache.get(type == null ? NULL_KEY_SURROGATE : type);
    if (cached != null) {
        return (TypeAdapter<T>) cached;
    }
    //threadCalls 是一个中间线程变量
    Map<TypeToken<?>, FutureTypeAdapter<?>> threadCalls = calls.get();
    boolean requiresThreadLocalCleanup = false;
    if (threadCalls == null) {
        threadCalls = new HashMap<TypeToken<?>, FutureTypeAdapter<?>>();
        calls.set(threadCalls);
        requiresThreadLocalCleanup = true;
    }
    //取回 FutureTypeAdapter，FutureTypeAdapter 是一个代理 TypeAdapter
    FutureTypeAdapter<T> ongoingCall = (FutureTypeAdapter<T>) threadCalls.get(type);
    if (ongoingCall != null) {
        return ongoingCall;
    }
    try {
        //new FutureTypeAdapter
        FutureTypeAdapter<T> call = new FutureTypeAdapter<T>();
        threadCalls.put(type, call);//放入线程变量中
        //从工厂中遍历，加入typeTokenCache
        for (TypeAdapterFactory factory : factories) {
            TypeAdapter<T> candidate = factory.create(this, type);//create创造实例
            if (candidate != null) {
                call.setDelegate(candidate);
                typeTokenCache.put(type, candidate);
                return candidate;
            }
        }
        throw new IllegalArgumentException("GSON cannot handle " + type);
    } finally {
        threadCalls.remove(type);
        if (requiresThreadLocalCleanup) {
            calls.remove();
        }
    }
}
```

##### （4）toJson

```java
//传入一个Object
public String toJson(Object src) {
    if (src == null) {
        return toJson(JsonNull.INSTANCE);
    }
    return toJson(src, src.getClass());//重载调用
}
//重载方法
public String toJson(Object src, Type typeOfSrc) {
    StringWriter writer = new StringWriter();
    toJson(src, typeOfSrc, writer);
    return writer.toString();//返回String
}
//将StringWriter转为JsonWriter
public void toJson(Object src, Type typeOfSrc, Appendable writer) throws JsonIOException {
    try {
        JsonWriter jsonWriter = newJsonWriter(Streams.writerForAppendable(writer));//转为JsonWriter
        toJson(src, typeOfSrc, jsonWriter);
    } catch (IOException e) {
        throw new JsonIOException(e);
    }
}

public void toJson(Object src, Type typeOfSrc, JsonWriter writer) throws JsonIOException {
    //调用getAdapter获取相应类型的TypeAdapter
    TypeAdapter<?> adapter = getAdapter(TypeToken.get(typeOfSrc));
    //设置配置
    boolean oldLenient = writer.isLenient();
    writer.setLenient(true);
    boolean oldHtmlSafe = writer.isHtmlSafe();
    writer.setHtmlSafe(htmlSafe);
    boolean oldSerializeNulls = writer.getSerializeNulls();
    writer.setSerializeNulls(serializeNulls);
    try {
        //调用adapter的write方法
        ((TypeAdapter<Object>) adapter).write(writer, src);
    } catch (IOException e) {
        throw new JsonIOException(e);
    } finally {
        writer.setLenient(oldLenient);
        writer.setHtmlSafe(oldHtmlSafe);
        writer.setSerializeNulls(oldSerializeNulls);
    }
}
```

##### （5）fromJson

```java
//传入一个json字符串和一个转换类型。
public <T> T fromJson(String json, Class<T> classOfT) throws JsonSyntaxException {
    Object object = fromJson(json, (Type) classOfT);	//调用重载
    return Primitives.wrap(classOfT).cast(object);	//如果是基本数据类型还会被包装返回（即 int => Integer）
}
//重载方法 
public <T> T fromJson(String json, Type typeOfT) throws JsonSyntaxException {
    if (json == null) {
        return null;
    }
    StringReader reader = new StringReader(json);//使用StringReader包装
    T target = (T) fromJson(reader, typeOfT);
    return target;
}
//重载方法，将StringReader转为JsonReader
public <T> T fromJson(Reader json, Type typeOfT) throws JsonIOException, JsonSyntaxException {
    JsonReader jsonReader = newJsonReader(json);
    T object = (T) fromJson(jsonReader, typeOfT);
    assertFullConsumption(object, jsonReader);//判断是不是读完了
    return object;
}
//最终方法
public <T> T fromJson(JsonReader reader, Type typeOfT) throws JsonIOException, JsonSyntaxException {
    boolean isEmpty = true;
    boolean oldLenient = reader.isLenient();
    reader.setLenient(true);
    try {
        reader.peek();
        isEmpty = false;
        //转为TypeToken
        TypeToken<T> typeToken = (TypeToken<T>) TypeToken.get(typeOfT);
        //获取TypeAdapter
        TypeAdapter<T> typeAdapter = getAdapter(typeToken);
        //调用TypeAdapter的read
        T object = typeAdapter.read(reader);
        return object;
    } catch (EOFException e) {
        //省略部分源码
    } finally {
        reader.setLenient(oldLenient);
    }
}
```

##### （6）registerTypeAdapter

> 可以注册新的类型转换器，实例创造器等，越先注册的优先级越高

```java
public GsonBuilder registerTypeAdapter(Type type, Object typeAdapter) {
    //校验参数
    $Gson$Preconditions.checkArgument(typeAdapter instanceof JsonSerializer<?> || typeAdapter instanceof JsonDeserializer<?> || typeAdapter instanceof InstanceCreator<?> || typeAdapter instanceof TypeAdapter<?>);
    //实例构造器   
    if (typeAdapter instanceof InstanceCreator<?>) {
        instanceCreators.put(type, (InstanceCreator) typeAdapter);
    }
    //序列化/反序列化
    if (typeAdapter instanceof JsonSerializer<?> || typeAdapter instanceof JsonDeserializer<?>) {
        TypeToken<?> typeToken = TypeToken.get(type);
        factories.add(TreeTypeAdapter.newFactoryWithMatchRawType(typeToken, typeAdapter));
    }
    //TypeAdapter
    if (typeAdapter instanceof TypeAdapter<?>) {
        factories.add(TypeAdapters.newFactory(TypeToken.get(type), (TypeAdapter)typeAdapter));
    }
    return this;
}
```

