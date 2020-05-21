# 简介
简单介绍Gson对枚举的序列化和反序列化，顺便自定义的序列化器和反序列化器的一个说明。

# 例子
## 创建一个性别枚举
```
public enum Gender{
    @SerializedName("男孩")
    BOY,

    GIRL,

    UNKNOWN
}
```

## 对name()进行序列化和反序列化
```
Gson gson = new Gson();
//序列化
String sexGirl = gson.toJson(Gender.GIRL);
System.out.println(sexGirl); //"GIRL"

//反序列化
Gender gril = gson.fromJson(sexGirl, Gender.class); //GIRL
```

## 通过`@SerializedName`指定枚举的含义
```
Gson gson = new Gson();
//序列化
String sexBoy = gson.toJson(Gender.BOY);
System.out.println(sexBoy); //"男孩"
//反序列化
Gender boy = gson.fromJson(sexBoy, Gender.class); //BOY
```
我们可以通过`@SerializedName`注解来指定枚举的含义

## 序列化为`ordinal()`
要序列化为枚举的ordinal值 这需要自定义序列化器和反序列化器

### 创建序列化器
```
	public  class EnumSerializer implements JsonSerializer<Enum<?>>{
		@Override
		public JsonElement serialize(Enum<?> src, Type typeOfSrc, JsonSerializationContext context) {
			//使用ordinal 来序列化成基本数据值
			return new JsonPrimitive(src.ordinal());
		}
	}
```

### 创建反序列化器
```
public static class EnumDeserialize implements JsonDeserializer<Enum<?>>{

		@Override
		public Enum<?> deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
				throws JsonParseException {
			//判断是否基本类型
			if(json.isJsonPrimitive()) {
				try {
					JsonPrimitive jsonPrimitive = json.getAsJsonPrimitive();
					//获取枚举类对象
					Class classEnum = Class.forName(typeOfT.getTypeName());
					//获取枚举内容
					Enum<?>[] enums = (Enum<?>[]) classEnum.getEnumConstants();
					if(jsonPrimitive.isNumber()) {
						return enums[jsonPrimitive.getAsInt()];
					}else if(jsonPrimitive.isString()){
						for(Enum<?> constant : enums) {
							if(constant.name().equalsIgnoreCase(jsonPrimitive.getAsString())) {
								return constant;
							}
						}
					}
					
				} catch (ClassNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			return null;
		}
		
	}
```

### 调用方式
```
//对ordinal()的，就要自定义序列化和反序列化器了
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeHierarchyAdapter(Enum.class, new EnumSerializer());
gsonBuilder.registerTypeHierarchyAdapter(Enum.class, new EnumDeserialize());

//序列化
Gson gsonB = gsonBuilder.create();
String boyB = gsonB.toJson(Gender.BOY);
System.out.println("boyB:"+boyB); //boyB:0

//反序列化
Gender boyB1 = gsonB.fromJson(boyB, Gender.class);
System.out.println(boyB1); //BOY
```

当然，序列化器和反序列化器可以通过内内部类实现
```
gsonBuilder.registerTypeHierarchyAdapter(Enum.class, new JsonSerializer<Enum<?>>() {
    @Override
    public JsonElement serialize(Enum<?> src, Type typeOfSrc, JsonSerializationContext context) {
    	return new JsonPrimitive(src.ordinal());
    }
});
```

# 总结
这里简单的列举了Gson对枚举的支持，同时也简单的的指明了自定义序列化器和反序列化器的写法。