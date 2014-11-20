# Intellisense Builder

บางครั้งเมื่อจำเป็นต้องสร้างคลาสที่มี constructor ที่รับพารามิเตอร์หลายๆตัว มักจะเกิดปัญหาต่างๆตามมาเช่น

- โค๊ดอ่านยาก แทบไม่มีความหมาย
- ยากต่อการใช้งาน
- หากชนิดของพารามิเตอร์เหมือนกัน การสลับตำแหน่งพารามิเตอร์ ทำให้เกิด logic error

ตัว โดยทั่วไปเราจะใช้ Builder Pattern เข้ามาช่วย (ท่านที่สนใจเรื่อง Builder Pattern ลองดูที่ Reference ข้างล่าง) แต่ว่า Builder Pattern ก็มีข้อเสียอยู่นิดหนึ่งคือ

- ไม่ง่ายต่อการตรวจเช็กว่าเซ็ตค่าครบหรือเปล่า ก่อนจะทำการเรียก .build();
- ใช้ประโยชน์ของอินเทลริเซนส์ได้ไม่เต็มที่

## Intellisense Builder Pattern

คือการดัดแปลงมาจาก Builder Pattern อีกที แต่สามารถเขียนได้ง่ายกว่า เข้าใจได้ง่ายว่า ใช้ประโยชน์จากอินเทลรึเซนส์ได้เต็มที่ และที่สำคัญคือ แพทเทิร์นนี้จะไม่ยอมให้ build() ถ้าค่าต่างๆเซ็ตไม่ครบ โพสนี้เราจะมาลองทำกันตั้งแต่เริ่มจนจบ

## วิธีการ

#####  1. สร้างคลาสตัวอย่างที่รับพารามิเตอร์ 6 ตัว ต่างๆชนิดกัน [download source code]( https://github.com/Charnnarong/Lambda-Java-8/blob/8a570ad8b057a7c2c6f49a41c0f23542aa5bb0b7/src/IntellisenseBuilderPattern.zip?raw=true)

```java
	public class Lenghty
	{
		private int number;
		private Double weight;
		private Double length;
		private String name;
		private char symbol;
		private Map<String, Double> misc;

		public Lenghty(int number, Double weight, Double length, String name,
				char symbol, Map<String, Double> misc)
		{
			this.number = number;
			this.weight = weight;
			this.length = length;
			this.name = name;
			this.symbol = symbol;
			this.misc = misc;
		}

		public void showParam()
		{
			System.out.println(number);
			System.out.println(weight);
			System.out.println(length);
			System.out.println(name);
			System.out.println(symbol);
			for(Entry<String, Double> item : misc.entrySet())
			{
				System.out.println(item.getKey() + " : " + item.getValue());
			}
			
		}

	}

```

##### 2. จากนั้นใน main ก็สร้าง instance ขึ้นมาอันหนึ่ง

```java
	public class Application
	{

		public static void main(String args[])
		{
			int number = 2;
			Double weight = 5.6;
			Double length = 7.1;
			String name = "Charnnarong";
			char symbol = 'K';
			Map<String, Double> misc = new HashMap<>();
			misc.put("M", 0.2);
			misc.put("K", 0.3);

			Lenghty lenghty = new Lenghty(number, weight, length, name, symbol,
					misc);

			lenghty.showParam();

		}

	}
```

การสร้างออปเจกของ `Lenghty` ไม่ใช่เรื่องน่าพิศมัยนัก วิธีที่ง่ายที่สุดที่จะทำให้มันดีขึ้นก็คือสร้าง Bean ขึ้นมาตัวหนึ่ง แล้วก็ใช้ setter จบครบจากนั้นก็ยัดเข้า Constructor ของ Lenghty แต่ในที่นี้เราจะไม่ทำแบบนั้น เราจะทำการลัดตรงเข้าสู่แพทเทิรน์ของเรากันเลย 

##### 3. สร้าง Interface สำหรับ getter ใน Lenghty 

```java

	public class Lenghty
	{
		private int number;
		private Double weight;
		private Double length;
		private String name;
		private char symbol;
		private Map<String, Double> misc;
		
		
		interface ParamInjector{
			 int getNumber();
			 Double getWeight();
			 Double getLength();
			 String getName();
			 char getSymbol();
			 Map<String, Double> getMisc();
		}

		public Lenghty(int number, Double weight, Double length, String name,
				char symbol, Map<String, Double> misc)
		{
			this.number = number;
			this.weight = weight;
			this.length = length;
			this.name = name;
			this.symbol = symbol;
			this.misc = misc;
		}

		public void showParam()
		{
			System.out.println(number);
			System.out.println(weight);
			System.out.println(length);
			System.out.println(name);
			System.out.println(symbol);
			for(Entry<String, Double> item : misc.entrySet())
			{
				System.out.println(item.getKey() + " : " + item.getValue());
			}
			
		}

	}

```

##### 3. เพิ่ม constructor

ทำการ Overload Constructor ขึ้นมาอีกตัวหนึ่ง ซึ่งตัวนี้จะรักแค่พรารามิเตอร์ชนิด ParamInjector อย่างเดียว ตัว `injector` นี้จะถูกสร้างมาจาก `Builder` อีกที

```java
	public Lenghty(ParamInjector injector)
	{
		this.number = injector.getNumber();
		this.weight = injector.getWeight();
		this.length = injector.getLength();
		this.name = injector.getName();
		this.symbol = injector.getSymbol();
		this.misc = injector.getMisc();		
	}
```

##### 4. สร้าง interfaces ที่จำเป็น
นี้คือขึ้นตอนแรกสูตรเฉพาะของ Intellisense Builder Pattern. ง่ายมากๆ เพียงแค่สร้าง Interface ที่มีชื่อเหมือน Field ทุกตัวอยู่ในคลาส `Lenghty` ไว้ในคลาสตัวมันเองนั่นแหละ 

```java
	public interface Begin{}
	public interface SetNumber{}
	public interface SetWeight{}
	public interface SetLength{}
	public interface SetName{}
	public interface SetSymbol{}
	public interface SetMisc{}
	public interface Build{}
```

ที่สำคัญคือ `interface Begin{}` กับ `interface Build{}` ที่เพิ่มเข้ามา จะเป็นอะไร จะหมายความว่าอย่างไร พอท่านทำเสร็จก็อาจไม่จำเป็นต้องอธิบายให้เสียเวลาก็ได้ แล้วเราค่อยไปดูผลลัพท์กันก่อนอธิบาย


##### 5. สร้าง Setter สำหรับ Field ทั้งหมด
ดูโค๊ดทั้งคลาสก่อน ส่วนคำอธิบายอยู่ข้างล่างครับ

```java
public class Lenghty
{
	private int number;
	private Double weight;
	private Double length;
	private String name;
	private char symbol;
	private Map<String, Double> misc;

	public interface Begin
	{
		SetNumber begin();
	}

	public interface SetNumber
	{
		SetWeight setNumber(int number);
	}

	public interface SetWeight
	{
		SetLength setWeight(Double weight);
	}

	public interface SetLength
	{
		SetName setLength(Double length);
	}

	public interface SetName
	{
		SetSymbol setName(String name);
	}

	public interface SetSymbol
	{
		SetMisc setSymbol(char symbol);
	}

	public interface SetMisc
	{
		Build setMisc(Map<String, Double> misc);
	}

	public interface Build
	{
		ParamInjector build();
	}

	public interface ParamInjector
	{
		int getNumber();

		Double getWeight();

		Double getLength();

		String getName();

		char getSymbol();

		Map<String, Double> getMisc();
	}

	public Lenghty(int number, Double weight, Double length, String name,
			char symbol, Map<String, Double> misc)
	{
		this.number = number;
		this.weight = weight;
		this.length = length;
		this.name = name;
		this.symbol = symbol;
		this.misc = misc;
	}

	public Lenghty(ParamInjector injector)
	{
		this.number = injector.getNumber();
		this.weight = injector.getWeight();
		this.length = injector.getLength();
		this.name = injector.getName();
		this.symbol = injector.getSymbol();
		this.misc = injector.getMisc();
	}

	public void showParam()
	{
		System.out.println(number);
		System.out.println(weight);
		System.out.println(length);
		System.out.println(name);
		System.out.println(symbol);
		for (Entry<String, Double> item : misc.entrySet())
		{
			System.out.println(item.getKey() + " : " + item.getValue());
		}

	}

}

```

จะเห็นว่าการหยอดโค๊ดเข้าไปใน Interface ก็แสนจะง่าย โดยมีหลักอยู่สองข้อคือ 
1) สร้างเมททอดที่ชื่อเหมือนอินเตอร์เฟสนั้นแหละ เพียงแค่เปลี่ยนเป็นเริ่มต้นด้วยตัวเล็ก 
2.) `return` ชนิดของอินเตอร์เฟสตัวถัดไป ที่เราต้องการจะให้ Intellisense แสดงเป็นลำดับ สุดท้ายจบลงที่ `Build` อินเตอร์เฟส 

จุดนี้แหละที่จะบังคับให้ Intellisense แสดงแค่ชื่อฟังก์ชั่นตัวถัดไป แค่นี้แหละครับ สิ่งที่ `Lenghty` ต้องเขียนเพิ่มเข้ามา ไม่ยากเลย ต่อไปเราจะไปสร้างตัว `Builder` ตัวนี้ก็แสนจะง่าย ไปดู


##### 6. สร้างคลาสใหม่ขึ้นมาตัวหนึ่ง ทำการอิมพรีเมนต์อินเตอร์เฟสทั้งหมดที่อยู่ใน `Lenghty`
ก็จะได้โครงแบบนี้. หมายเหตุ ในที่นี้ server คือชื่อแพกเกจ

```java
	import server.Lenghty.Begin;
	import server.Lenghty.Build;
	import server.Lenghty.ParamInjector;
	import server.Lenghty.SetLength;
	import server.Lenghty.SetMisc;
	import server.Lenghty.SetName;
	import server.Lenghty.SetNumber;
	import server.Lenghty.SetSymbol;
	import server.Lenghty.SetWeight;

	public class LengthyBuilder implements ParamInjector, Begin, SetName,
			SetWeight, SetLength, SetSymbol, SetNumber, SetMisc, Build
	{

	}

```

จากนั้น Eclipse ก็ขึ้นเส้นหยัก สีแดงบอกว่าให้ implements อินเตอร์เฟสพวกนั้น ก็คลิกเลยครับ 

##### 6. สร้าง Builder 

ในที่นี้เราจำเป็นต้องสร้าง field สำหรับเก็บค่าไว้ก่อน ก็ไปก๊อปมาจาก `Lenghty` มาแปะไว้ และหยอดโค๊ดง่ายๆ ตามนี้

```java

	public class LengthyBuilder implements ParamInjector, Begin, SetName,
			SetWeight, SetLength, SetSymbol, SetNumber, SetMisc, Build
	{
		private int number;
		private Double weight;
		private Double length;
		private String name;
		private char symbol;
		private Map<String, Double> misc;
		
		
		@Override
		public ParamInjector build()
		{		
			return this;
		}
		@Override
		public Build setMisc(Map<String, Double> misc)
		{
			this.misc = misc;
			return this;
		}
		@Override
		public SetWeight setNumber(int number)
		{
			this.number = number;
			return this;
		}
		@Override
		public SetMisc setSymbol(char symbol)
		{
			this.symbol = symbol;
			return this;
		}
		@Override
		public SetName setLength(Double length)
		{
			this.length = length;
			return this;
		}
		@Override
		public SetLength setWeight(Double weight)
		{
			this.weight = weight;
			return this;
		}
		@Override
		public SetSymbol setName(String name)
		{
			this.name = name;
			return this;
		}
		@Override
		public SetNumber begin()
		{
			return this;
		}
		@Override
		public int getNumber()
		{
			return number;
		}
		@Override
		public Double getWeight()
		{
			return weight;
		}
		@Override
		public Double getLength()
		{
			return length;
		}
		@Override
		public String getName()
		{
			return name;
		}
		@Override
		public char getSymbol()
		{
			return symbol;
		}
		@Override
		public Map<String, Double> getMisc()
		{
			return misc;
		}

		
	}

```

สำหรับ getter นั้นก็แสนจะง่าย เพียงแค่รีเทิร์นค่าตรงๆ ส่วน setter ก็ไม่มีอะไรเลย แค่ เซ็ตให้ตรงกับ field แล้วก็ `return this;` ทุกอัน
แค่นี้ก็เสร็จแล้วครับ ต่อไปเราลองไปดูวิธีใช้ใน main

```java
	public static void main(String args[])
		{
			int number = 2;
			Double weight = 5.6;
			Double length = 7.1;
			String name = "Charnnarong";
			char symbol = 'K';
			Map<String, Double> misc = new HashMap<>();
			misc.put("M", 0.2);
			misc.put("K", 0.3);

			Lenghty lenghty = new Lenghty(number, weight, length, name, symbol,
					misc);

			lenghty.showParam();
			
			System.out.println("-----Created by Intellisense Builder Pattern---");
			Lenghty.ParamInjector injector = new LengthyBuilder().begin()
					.setNumber(8).setWeight(weight).setLength(length)
					.setName(name).setSymbol('H').setMisc(misc).build();
			
			Lenghty lenghty2 = new Lenghty(injector);
			lenghty2.showParam();

		}
```

จะเห็นว่า ก่อนเราจะสร้าง `Lenghty` ใหม่เราจะสร้าง parameter builder ก่อน และ builder เวอร์ชั่นนี้ก็สามารถบังคับให้ Intellisense แสดงเฉพาะ setter ตัวถัดไป ตามที่เราได้วางแผนไว้ใน return type ในอินเตอรเฟส และมันจะทำการ Build ไม่ได้จนกว่าจะสร้างไปถึงตัวสุดท้าย 

แพทเทิร์นนี้นอกจากจะตัดปัญหาเรื่องส่งค่าผิด แล้ว ยังทำให้อ่านง่าย เข้าใจง่าย ใช้ง่าย ปลอดภัย และใช้ Intellisense ได้อย่างเต็มประสิทธิภาพครับ

1 เริ่มต้น 

![alt text](https://github.com/Charnnarong/Lambda-Java-8/blob/Lambda/images/1ibp.png "K")

2 Intellisense แสดงแค่ method ที่ควรตั้งค่า

![alt text](https://github.com/Charnnarong/Lambda-Java-8/blob/Lambda/images/2ibp.png "O")

3 เมททอดอื่นก็เหมือนกัน

![alt text](https://github.com/Charnnarong/Lambda-Java-8/blob/Lambda/images/3ibp.png "N")

4 เมื่อถึง setter สุดท้ายแล้วจะทำการเซ๊ตอีก intellisense ก็ไม่แสดง

![alt text](https://github.com/Charnnarong/Lambda-Java-8/blob/Lambda/images/4ibp.png "E")

5. สุดท้ายเรียก build เมื่อทุกค่าถูกเซ็ตเรียบร้อย

![alt text](https://github.com/Charnnarong/Lambda-Java-8/blob/Lambda/images/5ibp.png ":)")


```
* หมายเหตุ Intellisense Builder Pattern นี้ผมต่อยอดเองจาก Builder Pattern คิดเองเออเอง อาจไม่สมบูรณ์ อาจมีข้อผิดพลาดที่ยังหาไม่พบ หรือพลาดตาไป ถือว่าเป็นการแชร์ความรู้ แต่ไม่ขอรับผิดชอบใดๆทั้งสิ้น :)  
```

## Reference

[http://en.wikipedia.org/wiki/Builder_pattern](http://en.wikipedia.org/wiki/Builder_pattern)
[http://www.javaworld.com/article/2074938/core-java/too-many-parameters-in-java-methods-part-3-builder-pattern.html](http://www.javaworld.com/article/2074938/core-java/too-many-parameters-in-java-methods-part-3-builder-pattern.html)
[http://stackoverflow.com/questions/2332158/can-we-define-an-interface-within-an-interface](http://stackoverflow.com/questions/2332158/can-we-define-an-interface-within-an-interface)
