# 揭秘依賴注入(Dependency injection)

Ref. [http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html][1]

當我第一次聽到依賴注入(Dependency injection)的時候，心裡第一個反應是，"什...麼...是...依賴注入?"。然後就遺忘這件事情。之後當我花時間認真去搞懂時，我笑了。就這樣子?


## 簡而言之

依賴注入就是給予物件一個實例變數(instance variables)。真的!!就這樣。

## Part I: Dependency Non-Injection

我們稱之為 "依賴(Dependency)"。大多數人也稱為 "變數(variables)"，甚至有些人稱為 "實例變數(instance variables)"。

```text
public class Example {
    private DatabaseThingie myDatabase;

    public Example() {
        myDatabase = new DatabaseThingie();
    }

    public void DoStuff() {
        ...
        myDatabase.GetData();
        ...
    }
}
```

這裡我們有一個變數，恩...， 依賴(dependency)...，名為 "myDatabase"。在建構子中初始化它。

## Part II: Dependency Injection

我們可以傳遞一個變數到建構子。這將會注入(inject)依賴(dependency)到class內。當我們使用這個變數(依賴)\~\~variable(dependency)\~\~，現在使用的是外面傳入的物件，而不是我們自行建立的了。

```text
public class Example {
    private DatabaseThingie m yDatabase;

    public Example() {
        myDatabase = new DatabaseThingie();
    }

    // 外面傳入
    public Example(DatabaseThingie useThisDatabaseInstead) {
        myDatabase = useThisDatabaseInstead;
    }

    public void DoStuff() {
        ...
        myDatabase.GetData();
        ...
    }
}
```

整個概念就是這樣。其他就是依據情況去做些變化使用。你可以藉由一個 setter 傳入依賴(dependency)。也可透過一個介面，並定義 setter 方法，最後實作呼叫這些 setter 方法。你可以將你的依賴設計為一個介面或是多行等等。

### Part III: 為什麼我們要這麼做?

除此之外，下面是一個 Example 類別的測試。

```text
public class ExampleTest {
    TestDoStuff() {
        MockDatabase mockDatabase = new MockDatabase();

        // MockDatabase is a subclass of DatabaseThingie, so we can
        // "inject" it here:
        Example example = new Example(mockDatabase);

        example.DoStuff();
        mockDtabase.AssertGetDataWasCalled();
    }
}

public class Example {
    private DatabaseThingie myDatabase;

    public Example() {
        myDatabase = new DatabaseThingie();
    }

    public Example(DatabaseThingie useThisDatabaseInstead) {
        myDatabase = useThisDatabaseInstead;
    }

    public void DoStuff() {
        ...
        myDatabase.GetData();
        ...
    }
}
```

依賴注入(dependency injection)就是在傳遞實例變數(instance variable)。

## 延伸閱讀

有時概念很簡單，但因為使用上的遇到的情況或其他因素，讓整件事情複雜化了。但一開始學習新東西時，這不是我想要的！這裡就先不討論這些問題了。如果你真的想要知道更多，我推薦下面兩篇文章：

* [Inversion of Control Containers and the Dependency Injection Pattern.][2] - Martin Fowler 是一位我很喜歡的作者。通常他的文章清楚簡潔。他將依賴注入使用到~~複雜化~~出神入化。文章中還提及了許多其他依賴注入的使用方式。

* [A Beginner's Guide to Dependency Injection.][3] - 這篇較偏重 "依賴注入容器(containers)"。使用一個簡單的範例，去實踐如何使用多個現成的容器(containers)。我不知道這樣的意義在哪，但我本身是個異教徒，所以請忽略我在這方面的看法。

[1]:	http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html
[2]:	http://www.martinfowler.com/articles/injection.html
[3]:	http://www.theserverside.com/articles/article.tss?l=IOCBeginners