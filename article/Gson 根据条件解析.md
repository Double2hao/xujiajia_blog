#Gson 根据条件解析
# 概述

Gson在实际使用中，可能出现需要根据其中的某些参数来决定另外的参数的类型的情况，如下数据结构：

```
public class DataOne {<!-- -->
  public int type;//0代表data是DataTwo  1代表data是DataThree
  public Object data;//类型是DataTwo 或者DataThree
}

public class DataThree {<!-- -->
  public int height;
  public int width;
}

public class DataTwo {<!-- -->
  String name;
  int age;
}

```

本文主要展示一种解析这种场景的思路，如有其他更好的解决办法，欢迎读者评论交流。

# 解析的Json

```
{<!-- -->
    "type":0,
    "data":{<!-- -->
        "name":"testname",
        "age":19
    }
}

```

```
{<!-- -->
    "type":1,
    "data":{<!-- -->
        "height":100,
        "width":200
    }
}

```

# 思路与代码

针对DataOne实现JsonDeserializer，在新建Gson的时候注册这个JsonDeserializer，然后使用Gson解析。

### test逻辑

```
  private void gsonTest() {<!-- -->
    String testStringOne = "{\n"
        + "    \"type\":0,\n"
        + "    \"data\":{\n"
        + "        \"name\":\"testname\",\n"
        + "        \"age\":19\n"
        + "    }\n"
        + "}";
    String testStringTwo = "{\n"
        + "    \"type\":1,\n"
        + "    \"data\":{\n"
        + "        \"height\":100,\n"
        + "        \"width\":200\n"
        + "    }\n"
        + "}";

    Gson gson =
        new GsonBuilder()
            .registerTypeAdapter(DataOne.class, new TestDeserializer())
            .create();
    DataOne dataOne1 = gson.fromJson(testStringOne, DataOne.class);
    DataOne dataOne2 = gson.fromJson(testStringTwo, DataOne.class);
    System.out.println("dataOne1:" + dataOne1.toString());
    System.out.println("dataOne2:" + dataOne2.toString());
  }

```

### TestDeserializer.java

```
public class TestDeserializer implements JsonDeserializer&lt;DataOne&gt; {<!-- -->
  public static final Gson mGson = new Gson();

  @Override
  public DataOne deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
      throws JsonParseException {<!-- -->
    try {<!-- -->
      JsonObject jsonObject = json.getAsJsonObject();
      DataOne dataOne = new DataOne();
      dataOne.type = jsonObject.get("type").getAsInt();
      JsonElement dataElement = jsonObject.get("data");
      if (dataOne.type == 0) {<!-- -->
        dataOne.data = mGson.fromJson(dataElement, DataTwo.class);
      } else if (dataOne.type == 1) {<!-- -->
        dataOne.data = mGson.fromJson(dataElement, DataThree.class);
      }
      return dataOne;
    } catch (Exception e) {<!-- -->
      e.printStackTrace();
    }
    return null;
  }
}

```

### DataOne.java

```
public class DataOne {<!-- -->
  public int type;//0代表data是DataTwo  1代表data是DataThree
  public Object data;//类型是DataTwo 或者DataThree

  @Override public String toString() {<!-- -->
    StringBuilder stringBuilder = new StringBuilder("type:" + type);
    if (data != null) {<!-- -->
      try {<!-- -->
        if (type == 0) {<!-- -->
          DataTwo dataTwo = (DataTwo) data;
          stringBuilder.append(" name:").append(dataTwo.name);
          stringBuilder.append(" age:").append(dataTwo.age);
        } else if (type == 1) {<!-- -->
          DataThree dataThree = (DataThree) data;
          stringBuilder.append(" height:").append(dataThree.height);
          stringBuilder.append(" width:").append(dataThree.width);
        }
      } catch (Exception e) {<!-- -->
        e.printStackTrace();
      }
    }
    return stringBuilder.toString();
  }
}

```

### DataTwo.java

```
public class DataTwo {<!-- -->
  String name;
  int age;
}

```

### DataThree.java

```
public class DataThree {<!-- -->
  public int height;
  public int width;
}

```

# 解析结果

>  
 2020-08-20 15:48:26.214 1620-1620/com.example.gsontest I/System.out: dataOne1:type:0 name:testname age:19 2020-08-20 15:48:26.215 1620-1620/com.example.gsontest I/System.out: dataOne2:type:1 height:100 width:200 

