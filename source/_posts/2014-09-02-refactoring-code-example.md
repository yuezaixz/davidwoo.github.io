---
layout: post
title: '重构-代码示例'
date: 2014-09-02 13:58
comments: true
categories: [重构]
---
#Round.1案例
这里的案例是从重构这本书上直接拿来的，但是我通过不同的理解，做了些不一样的重构，总在先上源码。
```java
class Movie {
    public final static int CHILDRENS = 2;

    public final static int NEW_RELEASE = 1;

    public final static int REGULAR = 0;

    private String _title;

    private int _priceCode;

    public Movie(String title, int priceCode) {
        _title = title;
        _priceCode = priceCode;
    }

    public int getPriceCode() {
        return _priceCode;
    }

    public void setPriceCode(int arg) {
        _priceCode = arg;
    }

    public String getTitle() {
        return _title;
    }
}

class Rental {
    private Movie _movie;

    private int _daysRented;

    public Rental(Movie movie, int daysRented) {
        _movie = movie;
        _daysRented = daysRented;
    }

    public int getDaysRented() {
        return _daysRented;
    }

    public Movie getMovie() {
        return _movie;
    }
}

class Customer {
    private String _name;

    private List<Rental_rentals = new ArrayList<Rental>();

    public Customer(String name) {
        _name = name;
    }

    public void addRental(Rental arg) {
        _rentals.add(arg);
    }

    public String getName() {
        return _name;
    }

    public String statement() {
        double totalAmount = 0;
        int frequentRenterPoints = 0;
        String result = "Rental record for " + getName() + "\n";

        for (int index = 0; index < _rentals.size(); index++) {
            double thisAmount = 0;
            Rental each = (Rental) _rentals.get(index);

            switch (each.getMovie().getPriceCode()) {
            case Movie.REGULAR:
                thisAmount += 2;
                if (each.getDaysRented() 2)
                    thisAmount += (each.getDaysRented() - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                thisAmount += each.getDaysRented() * 3;
                break;
            case Movie.CHILDRENS:
                thisAmount += 1.5;
                if (each.getDaysRented() 3)
                    thisAmount += (each.getDaysRented() - 3) * 1.5;
                break;
            }
            frequentRenterPoints++;
            if (each.getMovie().getPriceCode() == Movie.NEW_RELEASE
                    && each.getDaysRented() 1)
                frequentRenterPoints++;

            result += "\t" + each.getMovie().getTitle() + "\t"
                    + String.valueOf(thisAmount) + "\n";
            totalAmount += thisAmount;
        }
        result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
        result += "You earned " + String.valueOf(frequentRenterPoints)
                + "frequent renter points";
        return result;
    }
}
public class Cp1Demo1{
    public static void main(String[] args) {
        Customer cuntomer = new Customer("David");
        cuntomer.addRental(new Rental(new Movie("Lost Template", Movie.NEW_RELEASE), 5));
        cuntomer.addRental(new Rental(new Movie("The Great Gatsby", Movie.REGULAR), 4));
        cuntomer.addRental(new Rental(new Movie("Wolf and Sheep", Movie.CHILDRENS), 2));
        cuntomer.addRental(new Rental(new Movie("Star Trek", Movie.REGULAR), 2));
        System.out.println(cuntomer.statement());
    }
}
```

#Round.2 让代码变得可测试（增加单元测试）
首先，先改造Customer类，让statement中的核心计算方法（算价格和算积分的方法）抽出。
```java Customer.java
public class Customer {
    private String name;

    //......省略部分代码
    
    //去除statement和main部分
    
    //对于statement中的方法，抽出两个关键的核心方法
    public double amountTotalConsume() {
        double totalAmount = 0;
        for (int index = 0; index < getRentals().size(); index++) {
            double thisAmount = 0;
            Rental each = (Rental) getRentals().get(index);

            switch (each.getMovie().getPriceCode()) {
            case Movie.REGULAR:
                thisAmount += 2;
                if (each.getDaysRented() 2)
                    thisAmount += (each.getDaysRented() - 2) * 1.5;
                break;
            case Movie.NEW_RELEASE:
                thisAmount += each.getDaysRented() * 3;
                break;
            case Movie.CHILDRENS:
                thisAmount += 1.5;
                if (each.getDaysRented() 3)
                    thisAmount += (each.getDaysRented() - 3) * 1.5;
                break;
            }
            // 日志
            System.out.println("\t" + each.getMovie().getTitle() + "\t"
                    + String.valueOf(thisAmount));
            totalAmount += thisAmount;
        }
        return totalAmount;
    }
    public int countFrequentRenterPoints() {
        int frequentRenterPoints = 0;
        for (int index = 0; index < getRentals().size(); index++) {
            Rental each = (Rental) getRentals().get(index);

            frequentRenterPoints++;
            if (each.getMovie().getPriceCode() == Movie.NEW_RELEASE
                    && each.getDaysRented() 1)
                frequentRenterPoints++;
        }
        return frequentRenterPoints;
    }
}
```
抽出后，也留有打印信息的事务入口，将入口创建一个新的类CustomerConsumeClinent
```java CustomerConsumeClinent.java
public class CustomerConsumeClinent {
    public String statement(Customer customer) {
        System.out.println("Rental record for " + customer.getName());
        StringBuilder resultSb = new StringBuilder();
        resultSb.append("Amount owed is ")
                .append(String.valueOf(customer.amountTotalConsume()))
                .append("\n").append("You earned ")
                .append(String.valueOf(customer.countFrequentRenterPoints()))
                .append(" frequent renter points");
        return resultSb.toString();
    }
    public static void main(String[] args) {
        Customer customer = new Customer("David");
        customer.addRental(new Rental(new Movie("Lost Template",
                Movie.NEW_RELEASE), 2));
        customer.addRental(new Rental(new Movie("Lost Template2",
                Movie.NEW_RELEASE), 2));
        customer.addRental(new Rental(new Movie("The Great Gatsby",
                Movie.REGULAR), 2));
        customer.addRental(new Rental(new Movie("Wolf and Sheep",
                Movie.CHILDRENS), 2));
        customer.addRental(new Rental(new Movie("Star Trek", Movie.REGULAR), 2));
        System.out.println(new CustomerConsumeClinent().statement(customer));
    }
}
```
然后添加单元测试，这里使用的是Junit4加上cucumber-java（Ruby中cucumber的java版本）进行BDD。
先修改maven的配置文件。
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.david.codejourney</groupId>
	<artifactId>refactoring-book</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.5.1</version>
				<configuration>
					<encoding>UTF-8</encoding>
					<source>1.6</source>
					<target>1.6</target>
				</configuration>
			</plugin>

			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>2.12.2</version>
				<configuration>
					<useFile>false</useFile>
				</configuration>
			</plugin>
		</plugins>
	</build>

	<dependencies>
		<dependency>
			<groupId>info.cukes</groupId>
			<artifactId>cucumber-java</artifactId>
			<version>1.1.2</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>info.cukes</groupId>
			<artifactId>cucumber-junit</artifactId>
			<version>1.1.2</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.rubiconproject.oss</groupId>
			<artifactId>jchronic</artifactId>
			<version>0.2.6</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<properties>
		<file.encoding>UTF-8</file.encoding>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
</project>
```
然后写功能描述文件，也就是单元测试的依据。这里不详细说明cucumber如何使用，有兴趣的自己去搜索相关知识再来看这部分代码，没兴趣的就跳过，知道他是在单元测试就好。
```
Feature: Refactoring Book
  In order to study.

  Scenario Outline: potter tests
    Given Init the customer
    And customer rental <number of 1st moviedays of REGULAR movie
    And customer rental <number of 2nd moviedays of REGULAR movie
    And customer rental <number of 3rd moviedays of NEW_RELEASE movie
    And customer rental <number of 4th moviedays of NEW_RELEASE movie
    And customer rental <number of 5th moviedays of REGULAR movie
    When I calculate the price and point
    Then I should get the lowest price <total amountand frequent points <frequent renter points>

  Examples: test basics
    | number of 1st movie| number of 2nd movie| number of 3rd movie| number of 4th movie| number of 5th movie| total amount | frequent renter points | 
    |         5          |           3        |           4        |         2          |          2         |       32.5   |           7            |
    |         2          |           2        |           2        |         2          |          2         |       17.5   |           7            |

  Examples: test simple discounts
    | number of 1st movie| number of 2nd movie| number of 3rd movie| number of 4th movie| number of 5th movie| total amount | frequent renter points |
    |         1          |           1        |           1        |         1          |          1         |       11.5   |           5            |

```
然后写单元测试类，该类需要和前面的功能文件对应。
```java RefactoringStepdefs.java
public class RefactoringStepdefs {

    private Customer customer;

    private int bookCount;

    @Given("^Init the customer$")
    public void init() throws Throwable {
        customer = new Customer("David");
        bookCount = 0;
    }

    @Given("^customer rental (\\d+) days of (\\w+) movie$")
    public void customerRentalMovieSimpleTest(int days, String movieType)
            throws Throwable {
        bookCount++;
        customer.addRental(new Rental(new Movie("Movie " + bookCount, movieType
                .equals("REGULAR") ? 0 : movieType.equals("NEW_RELEASE") ? 1
                : 2), days));
    }

    @When("^I calculate the price and point$")
    public void calcAndCount() throws Throwable {
    }

    @Then("I should get the lowest price (\\S+) and frequent points (\\d+)$")
    public void I_should_get_the_lowest_price(double totalAmout,
            int frequentRenterPoints) throws Throwable {

        assertEquals("failure - not the same with expected amout",
                totalAmout, customer.amountTotalConsume(),5);
        assertEquals("failure - not the same with expected point",
                frequentRenterPoints, customer.countFrequentRenterPoints());
    }

}
```
#Round.3 大胆的进行重构
现在已经有了单元测试的保证了，可以大胆进行重构了，按前篇博文中说到的进行。
##用简洁易懂的句法，控制一个函数类代码的数量，让一个函数只做一件事情
这个round2已经做了，不在让一个打印输出的方法，又同时做计算价格、统计积分等事情。
##修改变量名
这个较多地方都有问题，比如round1中的成员变量命名不是java的规范，全部重新写过。还比如遍历中用each作为变量名，很难表达变量的意思。
##相同的逻辑出现3次，抽方法
这里功能比较简单，会重复出现的方法已经抽出来了，所以这里用不到该方法。
##一个类中的一个方法出现与另一个类进行更多的交流时，考虑搬移函数到该类中，然后该方法中直接调用该类的新方法
这里对于每个Rental的计算积分和计算价格方法，都更应该移动到Rental中，因为看代码可以看到，他调用了很多次Rental的方法。
重构后的代码
```java Customer.java
public class Customer {
    private String name;
    private List<Rentalrentals = new ArrayList<Rental>();
    public Customer(String name) {
        this.name = name;
    }
    public void addRental(Rental arg) {
        rentals.add(arg);
    }
    public String getName() {
        return this.name;
    }
    public List<RentalgetRentals() {
        return rentals;
    }
    public double amountTotalConsume() {
        double result = 0;
        for (Rental aRental : getRentals()) {
            result += aRental.getCharge();
        }
        return result;
    }
    public int countFrequentRenterPoints() {
        int frequentRenterPoints = 0;
        for (Rental aRental : getRentals()) {
            frequentRenterPoints += aRental.getFrquentRenterPoints();
        }
        return frequentRenterPoints;
    }
}
```

```java Rental.java
public class Rental {
    private Movie movie;
    private int daysRented;
    public Rental(Movie movie, int daysRented) {
        this.movie = movie;
        this.daysRented = daysRented;
    }
    public int getDaysRented() {
        return this.daysRented;
    }
    public Movie getMovie() {
        return this.movie;
    }
    public int getFrquentRenterPoints() {
        int frequentRenterPoints = 0;
        frequentRenterPoints++;
        if (getMovie().getPriceCode() == Movie.NEW_RELEASE
                && getDaysRented() 1) {
            frequentRenterPoints++;
        }
        return frequentRenterPoints;
    }
    public double getCharge() {
        double thisAmount = 0;
        switch (getMovie().getPriceCode()) {
        // 这里的数量和定价策略可以先不改变，当增加新功能需要的时候
        case Movie.REGULAR:
            thisAmount += 2;
            if (getDaysRented() 2)
                thisAmount += (getDaysRented() - 2) * 1.5;
            break;
        case Movie.NEW_RELEASE:
            thisAmount += getDaysRented() * 3;
            break;
        case Movie.CHILDRENS:
            thisAmount += 1.5;
            if (getDaysRented() 3)
                thisAmount += (getDaysRented() - 3) * 1.5;
            break;
        }
        // 日志
        System.out.println("\t" + getMovie().getTitle() + "\t"
                + String.valueOf(thisAmount));
        return thisAmount;
    }
}
```

```java Movie.java
public class Movie {
    public final static int CHILDRENS = 2;

    public final static int NEW_RELEASE = 1;

    public final static int REGULAR = 0;

    private String title;

    private int priceCode;

    public Movie(String title, int priceCode) {
        this.title = title;
        this.priceCode = priceCode;
    }

    public int getPriceCode() {
        return this.priceCode;
    }

    public void setPriceCode(int arg) {
        this.priceCode = arg;
    }

    public String getTitle() {
        return this.title;
    }
}
```
重构完后，运行下单元测试，没问题，那么很好，完美收工。

#Round.4 如果发现增加一个功能很差，那么继续重构吧
一般情况下，做到第3步已经够了，第四部主要是说明，当你遇到新的需求，增加这个需求需要改动过大时候，那么说明代码的设计不够可维护可扩展可复用。
如下重构则为当需要增加一个类型，或者更改价格、积分计算策略时候的调整。
##开始重构，用多态替换逻辑判断
首先，把getCharge与getFrquentRenterPoints方法移动到Movie中，因为我们需要用多态来代替逻辑状态判断。
```java Rental.java
public class Rental {
		//...省略
		public int getFrquentRenterPoints() {
        return getMovie().getFrquentRenterPoints(daysRented);
    }
    public double getCharge() {
        return getMovie().getCharge(daysRented);
    }
}
```
```java Movie.java
public class Movie {
		//...省略部分代码，然后把priceType变量替换成一个枚举类型的变量
    private MovieTypeEnum movieType;
    public Movie(String title, int movieTypeCode) {
        this.title = title;
        movieType = MovieTypeEnum.convertByCode(String.valueOf(movieTypeCode));
    }
    public Movie(String title, String movieTypeName) {
        this.title = title;
        movieType = MovieTypeEnum.convertByName(movieTypeName);
    }
    public int getFrquentRenterPoints(int daysRented) {
        return movieType.getFrquentRenterPoints(daysRented);
    }
    public double getCharge(int daysRented) {
        return movieType.getCharge(daysRented);
    }
    public int getPriceCode() {
        return Integer.valueOf(this.movieType.getCode());
    }
    public void setPriceCode(int movieTypeCode) {
        movieType = MovieTypeEnum.convertByCode(String.valueOf(movieTypeCode));
    }
    public MovieTypeEnum getMovieType() {
        return movieType;
    }
    public void setMovieType(MovieTypeEnum movieType) {
        this.movieType = movieType;
    }
    public String getTitle() {
        return this.title;
    }
}
```
##创建枚举

```java
public enum MovieTypeEnum {
//这部分代码是我重构过程中犯下的低级错误，但是只要有单元测试，就能保证错误一旦出现，很快就能发现
//    CHILDRENS("2","CHILDRENS",new ChildrensPriceHandler(),new DefaultPointHandler()),
//    NEW_RELEASE("1","NEW_RELEASE",new ChildrensPriceHandler(),new DefaultPointHandler()),
//    REGULAR("0","REGULAR",new ChildrensPriceHandler(),new DefaultPointHandler());
    
    CHILDRENS("2","CHILDRENS",new ChildrensPriceHandler(),new DefaultPointHandler()),
    NEW_RELEASE("1","NEW_RELEASE",new NewReleasePriceHandler(),new NewReleasePointHandler()),
    REGULAR("0","REGULAR",new RegularPriceHandler(),new DefaultPointHandler());
    public final String code;
    public final String name;
    public final IPriceHandler priceHandler;
    public final IPointHandler pointHandler;    
    MovieTypeEnum(String code,String name,IPriceHandler priceHandler , IPointHandler pointHandler){
        this.code = code;
        this.name = name;
        this.priceHandler = priceHandler;
        this.pointHandler = pointHandler;
    }
    public String getCode() {
        return code;
    }
    public String getName() {
        return name;
    }
    public IPriceHandler getPriceHandler() {
        return priceHandler;
    }
    public IPointHandler getPointHandler() {
        return pointHandler;
    }
    public int getFrquentRenterPoints(int daysRented) {
        return pointHandler.getFrquentRenterPoints(daysRented);
    }
    
    public double getCharge(int daysRented) {
        return priceHandler.getCharge(daysRented);
    }
    public static MovieTypeEnum convertByCode(String code){
        if(MovieTypeEnum.CHILDRENS.getCode().equals(code)){
            return MovieTypeEnum.CHILDRENS;
        } else if(MovieTypeEnum.NEW_RELEASE.getCode().equals(code)){
            return MovieTypeEnum.NEW_RELEASE;
        } else if(MovieTypeEnum.REGULAR.getCode().equals(code)){
            return MovieTypeEnum.REGULAR;
        } else {
            return null;
        }
    }
    public static MovieTypeEnum convertByName(String name){
        if(MovieTypeEnum.CHILDRENS.getName().equals(name)){
            return MovieTypeEnum.CHILDRENS;
        } else if(MovieTypeEnum.NEW_RELEASE.getName().equals(name)){
            return MovieTypeEnum.NEW_RELEASE;
        } else if(MovieTypeEnum.REGULAR.getName().equals(name)){
            return MovieTypeEnum.REGULAR;
        } else {
            return null;
        }
    }
    
}
```

##处理器
这里分别实现了价格处理器和积分处理器，接口如下：
```java IPointHandler.java
public interface IPointHandler {
    public int getFrquentRenterPoints(int daysRented);
}
```

```java IPriceHandler.java
public interface IPriceHandler {
    public double getCharge(int daysRented);
}

```

具体处理器的实现这里就举一个例子，顺便看看同个处理逻辑两段代码的实现有多大差别。下面两段代码功能一样，代码前者多了那么多代码和临时变量。可以看出，写的优美的代码，不但易懂，而且优美。

```java 旧NewReleasePointHandler.java
public class NewReleasePointHandler extends DefaultPointHandler {
    public int getFrquentRenterPoints(int daysRented) {
				int frequentRenterPoints = 1;
        if (daysRented 1) {
            frequentRenterPoints++;
        }
        return frequentRenterPoints;
    }
}
```

```java 新NewReleasePointHandler.java
public class NewReleasePointHandler extends DefaultPointHandler {
    public int getFrquentRenterPoints(int daysRented) {
        return (daysRented 1) ? 2 : 1;
    }
}
```
运行单元而是，没问题，很好，又可以收工了（当然中间有出现错误，还有很多的调试）。
#Round.5 to be continue
重构不是一种状态，他是不断根据环境、需求、设计的变化去变化的。俗话说，只有变化是不变的，因此，重构一定会陪伴着整个软件过程，有了重构，也不用再怕这些变化了。