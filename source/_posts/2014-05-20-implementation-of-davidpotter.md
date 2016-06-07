---
layout: post
title: 'DavidPotter的实现、cucumber实现BDD开发'
date: 2014-05-20 12:25
comments: true
categories: [UML, BDD, TDD, 设计, KataPotter, cucumber, 责任链设计模式, 面向接口]
---
#DavidPotter实现
-----------------------------
![](http://i1.tietuku.com/13ad8e00f1cf9723s.png)
[代码Github地址](https://github.com/yuezaixz/DavidPotter)
##包结构
![](http://d2.freep.cn/3tb_140520204718v75v531911.jpg)

1. 按照UML设计图，进行了包结构划分；
2. 每个分包中基本都含有接口和实现类；
3. 每个包职责分明，遵循单一原则；

##实现
不具体说明所有代码，具体分析几个主要的类。
###Basket
```java
public abstract class Basket {

    Map<String, SeriesseriesMap = new HashMap<String, Series>();

    protected IBasketFactory factory;

    protected Basket(final Series... inputSeries) {
        for (final Series serie : inputSeries) {
            this.seriesMap.put(serie.getSeriesName(), serie);
        }
    }

    public int getNoEmptySeriesSize() {
        int num = 0;
        for (Series serie : seriesMap.values()) {
            if (!serie.isEmpty()) {
                num++;
            }
        }
        return num;
    }

    public abstract Price howMany();
    
    public abstract void print();

    public boolean areThereAnyBooksLeft() {
        for (Series serie : seriesMap.values()) {
            if (!serie.isEmpty()) {
                return true;
            }
        }
        return false;
    }

    public int bookNum() {
        int bookNum = 0;
        for (Series serie : seriesMap.values()) {
            bookNum += serie.length();
        }
        return bookNum;
    }

    public boolean push(String name, Book... books) {
        Series series = seriesMap.get(name);
        if (series == null) {
            return false;
        }
        for (Book book : books) {
            series.push(book);
        }
        return true;
    }
    
    public int popForEachSeriesBox() {
        int num = 0;
        for (Series serie : seriesMap.values()) {
            if (!serie.isEmpty()) {
                serie.pop();
                num++;
            }
        }
        return num;
    }

    public Basket pop(int bookNum) {
        Basket emptyBasket = factory.getEmptyBasket();
        int noEmptySeriesSize = getNoEmptySeriesSize();
        if (noEmptySeriesSize >= bookNum) {
            for (Series serie : seriesMap.values()) {
                if (!serie.isEmpty()) {
                    emptyBasket.push(serie.getSeriesName(), serie.pop());
                }
            }
        }
        return emptyBasket;
    }

    public Book popByName(int seriesName) {
        Series series = seriesMap.get(seriesName);
        return series.isEmpty() ? null : series.pop();
    }

    public abstract int[][] convertBasketToTwoDArray();

    @Override
    public String toString() {
        return seriesMap.toString();
    }

    public boolean equals(final Object obj) {
        Basket target = (Basket) obj;

        return this.seriesMap.equals(target.seriesMap);
    }

}
```
该类为购物车的抽象类，内部的数据结构通过系列名称和系列数据结构的Map来实现，是该codejourney的核心数据结构之一。
该类将一些购物车的特征性方法封装，比如获取购物车中的图书类数，是否还有书，剩余书的数量，向购物车内新增图书，	将购物车中所有类别书各取出一本，取出指定本书，根据书的类名取出图书。
另外，还对一些方法进行了抽象，比如当前购物车所有书籍折前价格，打印当前购物车的情况，将当前购物车转换成二维数组的数据结构，这些抽象方法通过子类来分别实现。因为这些方法都是面向具体实现的。

###Series
```java
public class Series implements ISeries {
    private Book[] books = new Book[16];

    private int size = 0;

    private String seriesName;

    public Series(String seriesName) {
        this.seriesName = seriesName;
    }
    
    public Series(String seriesName,Book ... books) {
        this.seriesName = seriesName;
        this.books = books;
    }

    public String getSeriesName() {
        return seriesName;
    }

    public void setSeriesName(String seriesName) {
        this.seriesName = seriesName;
    }

    @Override
    public boolean isEmpty() {
        return size == 0;
    }

    @Override
    public void clear() {
        // 将数组中的数据置为null, 方便GC进行回收
        for (int i = 0; i < size; i++) {
            books[size] = null;
        }
        size = 0;
    }

    @Override
    public int length() {
        return size;
    }

    @Override
    public boolean push(Book data) {
        // 判断是否需要进行数组扩容
        if (size >= books.length) {
            resize();
        }
        books[size++] = data;
        return true;
    }

    /**
     * 数组扩容
     */
    private void resize() {
        Book[] temp = new Book[books.length * 3 / 2 + 1];
        for (int i = 0; i < size; i++) {
            temp[i] = books[i];
            books[i] = null;
        }
        books = temp;
    }

    @Override
    public Book pop() {
        if (size == 0) {
            return null;
        }
        return (Book) books[--size];
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("MyArrayStack: [");
        for (int i = 0; i < size; i++) {
            sb.append(books[i].toString());
            if (i != size - 1) {
                sb.append(", ");
            }
        }
        sb.append("]");
        return sb.toString();
    }
}
```
该类内部数据结构为数组，通过数组实现了一个简易的类似栈，可以向栈内pop和push图书。是该codejourney的核心数据结构之一。
###PotterStrategy
```java
public class PotterStrategy {
    private static final Logger LOGGER = Logger.getLogger(PotterStrategy.class.getName());
    
    public static final List<StringPOTTER_SERIES_NAME_LIST = new ArrayList<String>();
    static{
        POTTER_SERIES_NAME_LIST.add("Harry Potter and the Philosopher's Stone");
        POTTER_SERIES_NAME_LIST.add("Harry Potter and the Chamber of Secrets");
        POTTER_SERIES_NAME_LIST.add("Harry Potter and the Prisoner of Azkaban");
        POTTER_SERIES_NAME_LIST.add("Harry Potter and the Goblet of Fire");
        POTTER_SERIES_NAME_LIST.add("Harry Potter and the Order of Phoenix");
    }
    
    public static final Price PRICE_FOR_EACH_BOOK_WITHOUT_DISCOUNT = new Price(800);
    public static final int MAX_SERIES_NUMBER = 5;
    public static final Price CHEAPER_BY_FIVE_THREE_PATTERN = new Price(40);
    public static final int FIVE_THREE_PATTERN_FIVE = 5;
    public static final int FIVE_THREE_PATTERN_THREE = 3;
    public static final int MAX_NUMBER_OF_COPIES_FOR_EACH_SERIES = 10;
    public static final double DISCOUNT_FOR_TWO_SERIES = 0.95;
    public static final double DISCOUNT_FOR_THREE_SERIES = 0.9;
    public static final double DISCOUNT_FOR_FOUR_SERIES = 0.8;
    public static final double DISCOUNT_FOR_FIVE_SERIES = 0.75;
    private int[] differentSeriesCount = new int[PotterStrategy.MAX_NUMBER_OF_COPIES_FOR_EACH_SERIES];

    public PotterStrategy() {
        // To turn on logging, set level to be Level.INFO.
        LOGGER.setLevel(Level.OFF);
    }

    public boolean hasFiveThreePattern(Basket shoppingBasket) {
        int[][] basketTwoDArray = shoppingBasket.convertBasketToTwoDArray();
        countDifferentSeries(basketTwoDArray);
        for (int i = 0; i < differentSeriesCount.length - 1; i++) {
            LOGGER.info("==Going to check differentSeriesCount[" + i + "]: " + differentSeriesCount[i] + "; differentSeriesCount[" + (i+1) + "]: " + differentSeriesCount[i+1]);
            if (differentSeriesCount[i] == FIVE_THREE_PATTERN_FIVE && differentSeriesCount[i+1] == FIVE_THREE_PATTERN_THREE) {
                LOGGER.info("==Found five three pattern.");
                return true;
            }
        }
        return false;
    }

    public void countDifferentSeries(int[][] basketTwoDArray) {
        int count = 0;
        for (int i = 0; i < differentSeriesCount.length; i++) {
            for (int j = 0; j < basketTwoDArray.length; j++) {
                count += basketTwoDArray[j][i];
            }
            differentSeriesCount[i] = count;
            count = 0;
        }
        printDifferentSeriesCount(differentSeriesCount);
    }

    private void printDifferentSeriesCount(int[] differentSeriesCount) {
        LOGGER.info("==different series count: " + Arrays.toString(differentSeriesCount));
    }

    public static String getSeriesNumOfBookName(int seriesNumOfBook) {
        return POTTER_SERIES_NAME_LIST.get(seriesNumOfBook-1);
    }
}
```
该类的作用主要是提供一些常量（比如折扣、价格、书的类别及名字）及静态方法（判断是否存在5-3的情况和一些日志打印静态方法）。
###handler
最核心的处理逻辑还是封装在handler组件中。IHandler为核心接口。使用的是责任链设计模式，这点和FizzBuzzWhizz类似。
```java
public interface IHandler {
    public void handle(ICalculate calc, PotterStrategy discountStrategy);
}
```
```java
public abstract class PotterHandler implements IHandler {
    private IHandler nextHandler;

    public PotterHandler(IHandler priceHandler) {
        this.nextHandler = priceHandler;
    }

    public void handle(ICalculate calc, PotterStrategy discountStrategy) {
        run(calc, discountStrategy);
        IHandler nextHandler = getNextHandler();
        if (nextHandler != null) {
            nextHandler.handle(calc, discountStrategy);
        }
    }

    public abstract void run(ICalculate calc, PotterStrategy discountStrategy);

    protected IHandler getNextHandler() {
        return nextHandler;
    }
}
```
通过实现该接口的抽象类，该抽象类又用了模板设计模式，将handler方法的责任链处理逻辑封装。子类只需要知道如何实现每个子链条的处理逻辑即可。
例如该项目的难点，5-3折扣同时出现的情况下，改用4-4折扣会更优惠，由于，优惠的价格固定为0.4元，所以当这种情况，就调用FiveThreePatternHandler。

```java
public class FiveThreePatternHandler extends PotterHandler {
    public FiveThreePatternHandler(IHandler wordHandler) {
        super(wordHandler);
    }
    @Override
    public void run(ICalculate calc, PotterStrategy discountStrategy) {
        if (calc.hasFiveThreePattern()) {
            calc.setCalculatedPrice(calc.getCalculatedPrice().minus(PotterStrategy.CHEAPER_BY_FIVE_THREE_PATTERN));
        }
    }
}
```

###price
```java
public class Price {

	private final double price;

	public Price(final double price) {
		this.price = price;
	}
	
	public double getPrice() {
        return price;
    }

    public String toString() {
		return "Price: " + Double.toString(price);
	}

	public boolean equals(final Object obj) {
		return ((Price) obj).price == price;
	}

	public int hashCode() {
		return new Double(price).hashCode();
	}

	public Price lowestPrice(final Price other) {
		return new Price(Math.min(price, other.price));
	}

	public Price add(final Price other) {
		return new Price(price + other.price);
	}
	
	public Price minus(final Price other) {
        return new Price(price - other.price);
    }
	
	public Price mul(final Price other) {
        return new Price(price * other.price);
    }
	
	public Price mul(final int mulNum) {
	    return new Price(price * mulNum);
	}
	
	public Price mul(final double mulNum) {
        return new Price(price * mulNum);
    }

}
```
该类主要用于对价格封装，把加减乘除的方法都封装。
#calc
```java
public interface ICalculate {

    public abstract Price getCalculatedPrice();

    public abstract void setCalculatedPrice(Price calculatedPrice);

    public abstract void calculate();
    
    public void printCalc();

    public abstract void fillSeriesBoxAndCalculatePrice();

    public abstract void initialize();

    public abstract void putIntoBasket(String seriesName, int numberOfBook);

    public abstract void putIntoBasket(int seriesNumberOfBook, int numberOfBook);
    
    public abstract boolean areThereAnyBooksLeft();

    public abstract boolean hasFiveThreePattern();

    public abstract void printBasket();
}

```
该接口封装了很多方法，有些方法可以直接是链式引用，比如hasFiveThreePattern，会通过调用持有的Strategy的相应方法来实现。那接着让我们看看该类的实现类。
```java
public class PriceCaclulate implements ICalculate {

    private Price calculatedPrice = new Price(0);
    
    private IHandler handler ;
    
    private PotterStrategy potterStrategy;
    
    private Basket potterBasket;
    
    public PriceCaclulate() {
        IHandler maxDiffenentSeriesHandler = new MaxDifferentSeriesHandler(null);
        potterStrategy = new PotterStrategy();
        handler = new FiveThreePatternHandler(
                maxDiffenentSeriesHandler);
        potterBasket = BasketFactoryBean.getPotterBasketFactoryInstance().getEmptyBasket();
    }
    public void printCalc() {
        System.out.println("购物车中的哈利波特故事书"+this.calculatedPrice);
    }
    @Override
    public Price getCalculatedPrice() {
        return calculatedPrice;
    }
    @Override
    public void setCalculatedPrice(Price calculatedPrice) {
        this.calculatedPrice = calculatedPrice;
    }
    @Override
    public void calculate() {
        handler.handle(this, this.potterStrategy);
    }
    @Override
    public void fillSeriesBoxAndCalculatePrice() {
        int seriesNum = potterBasket.popForEachSeriesBox();
        switch (seriesNum) {
        case 1:
            calculatedPrice = calculatedPrice
                    .add(PotterStrategy.PRICE_FOR_EACH_BOOK_WITHOUT_DISCOUNT);
            break;
        case 2:
            calculatedPrice = calculatedPrice
                    .add(PotterStrategy.PRICE_FOR_EACH_BOOK_WITHOUT_DISCOUNT
                            .mul(2).mul(PotterStrategy.DISCOUNT_FOR_TWO_SERIES));
            break;
        case 3:
            calculatedPrice = calculatedPrice
                    .add(PotterStrategy.PRICE_FOR_EACH_BOOK_WITHOUT_DISCOUNT
                            .mul(3).mul(
                                    PotterStrategy.DISCOUNT_FOR_THREE_SERIES));
            break;
        case 4:
            calculatedPrice = calculatedPrice
                    .add(PotterStrategy.PRICE_FOR_EACH_BOOK_WITHOUT_DISCOUNT
                            .mul(4)
                            .mul(PotterStrategy.DISCOUNT_FOR_FOUR_SERIES));
            break;
        case 5:
            calculatedPrice = calculatedPrice
                    .add(PotterStrategy.PRICE_FOR_EACH_BOOK_WITHOUT_DISCOUNT
                            .mul(5)
                            .mul(PotterStrategy.DISCOUNT_FOR_FIVE_SERIES));
            break;
        default:
            throw new IllegalStateException(
                    "the number of series should be within "
                            + PotterStrategy.MAX_SERIES_NUMBER);
        }
    }

    @Override
    public void initialize() {
        calculatedPrice = new Price(0);
    }

    @Override
    public void putIntoBasket(String seriesName, int numberOfBook) {
        for (int i = 0; i < numberOfBook; i++) {
            potterBasket.push(seriesName, new Book());
        }
    }
    
    @Override
    public void putIntoBasket(int seriesNumOfBook, int numberOfBook) {
        
        String seriesName = PotterStrategy.getSeriesNumOfBookName(seriesNumOfBook);
        
        for (int i = 0; i < numberOfBook; i++) {
            potterBasket.push(seriesName, new Book());
        }
    }

    @Override
    public boolean areThereAnyBooksLeft() {
        return potterBasket.areThereAnyBooksLeft();
    }

    @Override
    public boolean hasFiveThreePattern() {
        return potterStrategy.hasFiveThreePattern(potterBasket);
    }

    @Override
    public void printBasket() {
        potterBasket.print();
    }
}
```
该类依赖了Strategy、Price、Handler、Basket。该类是对外的API接口，使用该组件不需要对组件的内部实现有什么了解。只要知道接口，已经具体需要的实现类，然后调用相应API即可。

###调用
```java
ICalculate cacluate = new PriceCaclulate();
cacluate.putIntoBasket(PotterStrategy.getSeriesNumOfBookName(1), 1);
cacluate.putIntoBasket(PotterStrategy.getSeriesNumOfBookName(2), 1);
cacluate.putIntoBasket(PotterStrategy.getSeriesNumOfBookName(3), 2);
cacluate.putIntoBasket(PotterStrategy.getSeriesNumOfBookName(4), 1);
cacluate.putIntoBasket(PotterStrategy.getSeriesNumOfBookName(5), 2);

cacluate.printBasket();
cacluate.calculate();
cacluate.printCalc();
```
如上所说，只要调用接口的相应实现子类，然后添加书籍（添加书籍的方法也做好了封装），然后调用相应的API即可。

##单元测试
这次项目的单元测试采用的是一种叫做cucumber的BDD工具，该工具在Ruby业内挺流行的，Java还不是应用广泛，在国内可能找相应的文章都不容易。
什么是BDD？TDD是一种通过测试用例来进行设计、代码驱动的开发方式，但是，这种方式是由程序员作为主导的，一般测试和产品经理并看不懂单元测试，只能说产品经理通过描述，让测试转化为测试用例，再通过程序员编写单元测试来实现。
那么，cucumber就解决了TDD的这一问题，直接通过产品经理设置场景，测试在场景内加入Example数据，再通过程序员来实现。
```
Feature: Kata Potter
 出版社要促销一套哈利波特图书，该套图书共5集，每集单册购买价8元。
 若任意两集各买一本，打95折；若任意三集各买一本，打9折；若任意四集各买一本，打8折；
 若所有这五集都各买一本，打75折。上述优惠之外再购买的单册还是按8元一本计价。
 比如五集各买一本之外再加一本第一集，五本书打75折，这本另加的第一集按8元计价。
 若有人第一集买2本，第二集买2本，第三集买2本，第四集买1本，第五集买1本。
 问打折后最低的优惠价是多少钱？提示，不是51.6元，而是51.2元。

  Scenario Outline: David的Potter测试
    Given 开始我先要初始化我的购物车
    And 我购买 <Harry Potter and the Philosopher's Stone本 1st 系列书
    And 我购买 <Harry Potter and the Chamber of Secrets本 2nd 系列书
    And 我购买 <Harry Potter and the Prisoner of Azkaban本 3rd 系列书
    And 我购买 <Harry Potter and the Goblet of Fire本 4th 系列书
    And 我购买 <Harry Potter and the Order of Phoenix本 5th 系列书
    When 我计算购物车的总价
    Then 我最少应该付 <lowest price>

  Examples: 最基本的测试
    | Harry Potter and the Philosopher's Stone | Harry Potter and the Chamber of Secrets | Harry Potter and the Prisoner of Azkaban | Harry Potter and the Goblet of Fire | Harry Potter and the Order of Phoenix | lowest price |
    |                    0                     |                    0                    |                      0                   |                  0                  |                   0                   |       0      |
    |                    1                     |                    0                    |                      0                   |                  0                  |                   0                   |       800    |
    |                    0                     |                    1                    |                      0                   |                  0                  |                   0                   |       800    |
    |                    0                     |                    0                    |                      1                   |                  0                  |                   0                   |       800    |
    |                    0                     |                    0                    |                      0                   |                  1                  |                   0                   |       800    |
    |                    0                     |                    0                    |                      0                   |                  0                  |                   1                   |       800    |
    |                    2                     |                    0                    |                      0                   |                  0                  |                   0                   |       1600   |
    |                    0                     |                    3                    |                      0                   |                  0                  |                   0                   |       2400   |

  Examples: 简单的打折测试
    | Harry Potter and the Philosopher's Stone | Harry Potter and the Chamber of Secrets | Harry Potter and the Prisoner of Azkaban | Harry Potter and the Goblet of Fire | Harry Potter and the Order of Phoenix | lowest price |
    |                    1                     |                    1                    |                      0                   |                  0                  |                   0                   |       1520   |
    |                    1                     |                    0                    |                      1                   |                  0                  |                   1                   |       2160   |
    |                    1                     |                    1                    |                      1                   |                  0                  |                   1                   |       2560   |
    |                    1                     |                    1                    |                      1                   |                  1                  |                   1                   |       3000   |

  Examples: 复合折扣测试
    | Harry Potter and the Philosopher's Stone | Harry Potter and the Chamber of Secrets | Harry Potter and the Prisoner of Azkaban | Harry Potter and the Goblet of Fire | Harry Potter and the Order of Phoenix | lowest price |
    |                    2                     |                    1                    |                      0                   |                  0                  |                   0                   |       2320   |
    |                    2                     |                    2                    |                      0                   |                  0                  |                   0                   |       3040   |
    |                    2                     |                    1                    |                      2                   |                  1                  |                   0                   |       4080   |
    |                    1                     |                    2                    |                      1                   |                  1                  |                   1                   |       3800   |

  Examples: 复杂折扣测试
    | Harry Potter and the Philosopher's Stone | Harry Potter and the Chamber of Secrets | Harry Potter and the Prisoner of Azkaban | Harry Potter and the Goblet of Fire | Harry Potter and the Order of Phoenix | lowest price |
    |                    2                     |                    2                    |                      2                   |                  1                  |                   1                   |       5120   |
    |                    5                     |                    5                    |                      4                   |                  5                  |                   4                   |       14120   |

```
上为本例中的场景。Given为给定的条件，AND是与的情况，WHEN是进行的操作，Then是结果，Examples是测试用例，<>中的内容会去测试用例中拿。让我们看看对应的代码怎么写。
```java
public class PotterStepdefs {
    private ICalculate calc = new PriceCaclulate();

    @Given("^开始我先要初始化我的购物车$")
    public void I_clear_my_shopping_basket() throws Throwable {
        calc.initialize();
    }

    @Given("^我购买 (\\d+) 本 (\\d+)(?:st|nd|rd|th) 系列书$")
    public void I_buy_copies_of_book(int numberOfBook,int seriesNumberOfBook) throws Throwable {
        System.out.println("wudw"+seriesNumberOfBook);
        calc.putIntoBasket(seriesNumberOfBook, numberOfBook);
    }
    @When("^我计算购物车的总价$")
    public void I_calculate_the_price() throws Throwable {
        calc.calculate();
    }

    @Then("^我最少应该付 (\\d+)$")
    public void I_should_get_the_lowest_price(int expectedPrice) throws Throwable {
        int price = (int) (calc.getCalculatedPrice().getPrice());
        assertEquals("错误 - 计算结果与期望值不一致", expectedPrice, price);
    }

}

```
每个方法上有GIVEN、WHEN、THEN等，内部有相应的正则，会对应前面行为描述文件中的内容，正则()刮号中匹配的{1}、{2}等，都为成为注释方法中的参数。然后按描述文件中的顺序执行。
直接看看单元测试的结果吧。
![](http://d3.freep.cn/3tb_140520204143venn531911.jpg)
还可以生成测试报告。
![](http://d2.freep.cn/3tb_140520204143phb5531911.jpg)
有没觉得很好用？
