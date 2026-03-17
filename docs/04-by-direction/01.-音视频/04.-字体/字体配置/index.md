# 字体配置<no value>

# 1. 字体格式


# 2. 字体名称的格式

```text
sarasa-fixed-slab-cl-regular.ttf
```

1. 字体家族名称：sarasa。每个文件名都由它开头.
2. 字体风格 (Style)：fixed。字体风格决定了西文字符的字形，分为 Gothic、UI、Mono、Term、Fixed 五种。详见下图。
3. 字体衬线：slab。只有一部分文件有这个部分，代表其加入了衬线，数字和字母的笔画末端有短线修饰。
4. 汉字字形 (Orthography)：cl。根据不同国家和地区的标准，字形分为 CL、HC、J、K、SC、TC 六种，例如中国大陆、中国香港、日本的“冷”字写法都不一样，详见下图。
5. 字重：regular。这一项代表的是字体的粗细，从细到粗为 extralight、light、regular、semibold、bold。加上 italic 后缀的为意大利体 (斜体)。
6. 后缀名：ttf。最通用的字体格式，不解释。


## 2.1. Style

|风格|等宽|弯引号|破折号|连字|
|--|--|--|--|--|
|Gothic|否|全宽|全宽|否|
|UI|否|半宽|全宽|否|
|Mono|是|半宽|全宽|是|
|Term|是|半宽|半宽|是|
|Fixed|是|半宽|半宽|否|


## 2.2. Orthography


|名称|场景|
|--|--|
|CL|Classical，旧字体汉字，用于需要使用旧字形的场景|
|HC|HongKong Chinese，香港繁体和澳门繁体的字形|
|J|Japanese，日本语，日本新字体的字形|
|K| Korean，朝鲜语(韩语)，朝鲜半岛和朝鲜族的字形|
|SC|Simplified Chinese，简体中文，中国内地、大马、新加坡等地字形|
|TC|traditional Chinese，繁体中文，中国台湾地区的字形|


## 2.3. weight

Ultra Light < Extralight < Light < Semi Light（Demi Light)< Regual < Medium < Semi Bold(Demi Bold)< Bold <  Extra Bold  < Ultra Bold < Heavy(Black)

## 2.4. slant 倾斜

- Roman :倾斜度0
- Italic :倾斜度100，专门的斜体写法，类似于手写样式
- Oblique :倾斜度110，把常规写法倾斜以下变化得到 


## 2.5. 通用的字体类型名称

一般前端CSS设置不会直接写具体的字体，只要写通用字体类型名称，由系统自己选择对应的字体

### 2.5.1. sans-serif (sans)
ArialVerdana               
"Sans"是指无 - 这些字体在末端没有额外的装饰

### 2.5.2. serif  
Times New Roman Georgia    
Serif字体中字符在行的末端拥有额外的装饰
### 2.5.3. monospace (mono)

Courier NewLucida Console  
所有的等宽字符具有相同的宽度


# 3. 开源字体项目
在维护的开源中文字体就一套，同时被 [Noto](https://en.wikipedia.org/wiki/Noto_fonts) 和[思源](https://en.wikipedia.org/wiki/Source_Han_Sans)两个项目收录


Noto 系列字族名只支持英文，命名规则是 Noto + Sans 或 Serif + 文字名称。其中汉字部分叫 Noto Sans/Serif CJK SC/TC/HK/JP/KR，最后一个词是地区变种。