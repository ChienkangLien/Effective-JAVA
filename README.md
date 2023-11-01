# Effective JAVA
## 1.考慮使用靜態工廠方法替代構造方法
優點
1. 具有描述性名稱：給定一個有意義的名稱，根據方法名稱知道其用途。
2. 不強制每次調用都創建新對象：根據需求控制物件的創建和緩存，返回現有物件或是創建新物件。
3. 能夠返回子類型的對象：可以返回其子類型的物件。
4. 返回的類可以根據輸入參數的不同而不同：搭配方法中的操作返回指定的類型。
5. 返回的對象的類不需要存在：允許返回的類可以是在運行時動態加載的或基於配置文件的，不需事先聲明import。
```java=
// 接口：圖形
interface Shape {
    void draw();
}

// 具體圖形類1：圓形
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("繪制圓形");
    }
}

// 具體圖形類2：矩形
class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("繪制矩形");
    }
}

// 具體圖形類3：正方形
class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("繪制正方形");
    }
}

// 圖形工廠類，使用靜態工廠方法
class ShapeFactory {
    // 靜態工廠方法，根據輸入參數返回不同類型的圖形對象
    public static Shape createShape(String shapeType) {
        if (shapeType == null) {
            return null;
        }
        if (shapeType.equalsIgnoreCase("CIRCLE")) {
            return new Circle();
        } else if (shapeType.equalsIgnoreCase("RECTANGLE")) {
            return new Rectangle();
        } else if (shapeType.equalsIgnoreCase("SQUARE")) {
            return new Square();
        }
        return null;
    }
}

public class FactoryPatternExample {
    public static void main(String[] args) {
        // 使用靜態工廠方法創建不同類型的圖形對象
        Shape circle = ShapeFactory.createShape("CIRCLE");
        circle.draw();

        Shape rectangle = ShapeFactory.createShape("RECTANGLE");
        rectangle.draw();

        Shape square = ShapeFactory.createShape("SQUARE");
        square.draw();
    }
}
```
缺點
1. 沒有公共或受保護構造方法的類不能被子類化：靜態工廠方法通常用於創建不可變對象。如果類的構造方法是私有的或受保護的，子類無法繼承。
2. 程序員很難找到它們：命名自由，相對難取得。
```java=
public class BaseClass {
    // 私有構造方法，防止直接創建對象
    private BaseClass() {
        // 初始化操作
    }
    
    public void someMethod() {
        // 執行一些操作
    }
}

// 試圖繼承BaseClass，但無法訪問其構造方法
public class SubClass extends BaseClass {
    // 子類的構造方法
    public SubClass() {
        // 編譯錯誤：無法訪問BaseClass構造方法
    }
}
```
## 2.當構造方法參數過多時使用 builder 模式
通過構建器物件逐步設置物件的屬性，然後最終創建所需的物件
```java=
// 產品類
public class Computer {
    private String processor;
    private int ram;
    private int storage;
    private String graphicsCard;
    
    // 私有構造方法，只能通過內部的 Builder 創建 Computer 對象
    private Computer(ComputerBuilder builder) {
        this.processor = builder.processor;
        this.ram = builder.ram;
        this.storage = builder.storage;
        this.graphicsCard = builder.graphicsCard;
    }
    
    // Getter 方法
    // ...
    
    // 靜態內部 Builder 類
    public static class ComputerBuilder {
        private String processor;
        private int ram;
        private int storage;
        private String graphicsCard;
        
        public ComputerBuilder(String processor, int ram) {
            this.processor = processor;
            this.ram = ram;
        }
        
        public ComputerBuilder setStorage(int storage) {
            this.storage = storage;
            return this;
        }
        
        public ComputerBuilder setGraphicsCard(String graphicsCard) {
            this.graphicsCard = graphicsCard;
            return this;
        }
        
        public Computer build() {
            return new Computer(this);
        }
    }
}
```

## 3.使用私有構造方法或枚舉類實現 Singleton 屬性
* 外部無法直接創建實例，而是通過公共的靜態方法來獲取唯一的實例；需要額外處理以保證線程安全
1. 餓漢式（Eager Initialization）：類加載時就被創建。這保證了線程安全，但可能在應用程序啟動時造成一些性能開銷
```java=
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {
        // 私有構造方法
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```
2. 懶漢式（Lazy Initialization）雙重檢查鎖定雙重檢查鎖定（Double-Checked Locking）：單例實例在第一次調用 getInstance 方法時才被創建
```java=
public class Singleton {
    private static volatile Singleton instance; // 使用 volatile 保證可見性

    private Singleton() {
        // 私有構造方法
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
3. 懶漢式（Lazy Initialization）靜態內部類：利用了類加載的延遲初始化特性，確保線程安全，同時保持了懶加載。這是一種推薦的方式
```java=
public class Singleton {
    private Singleton() {
        // 私有構造方法
    }

    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }

    private static class SingletonHolder {
        private static final Singleton instance = new Singleton();
    }
}
```
* 枚舉類是單例的，因此它天生就適合實現 Singleton，單例實例是在枚舉類加載時就創建的，也是一種餓漢式
```java=
public enum Singleton {
    INSTANCE;

    // 在枚舉類中可以添加其他方法
    public void doSomething() {
        System.out.println("Doing something in Singleton");
    }
}

public class Main {
    public static void main(String[] args) {
        // 獲取 Singleton 實例
        Singleton instance = Singleton.INSTANCE;

        // 調用 Singleton 實例的方法
        instance.doSomething();
    }
}
```
## 4.使用私有構造方法執行非實例化
通常用於包含靜態方法或靜態成員的工具類或實用類、不需要實例化的類。通過將構造方法私有化並拋出異常來實現，以阻止類的實例化。
```java=
public final class UtilityClass {
    // 將構造方法私有化，防止實例化
    private UtilityClass() {
        throw new AssertionError("This class cannot be instantiated");
    }

    // 其他靜態方法或靜態成員
    public static void doSomething() {
        // 執行某些操作
    }
}
```
## 5.使用依賴注入取代硬連接資源（hardwiring resources）
硬連接資源指的是在類內部直接創建或引用外部資源的方式，而依賴注入則是將這些資源從外部傳遞給類，以減少類之間的耦合性。
```java=
public class Car {
    private Engine engine;

    //public Car() {
    //    this.engine = new Engine();
    //}
    
    public Car(Engine engine) {
        this.engine = engine;
    }

    // 其他方法使用引擎
}
```
## 6.避免創建不必要的對象
1. 使用對象池（Object Pooling）：對象池是一種重用對象的技術，它可以減少對象的創建和銷毀，提高性能。對象池可以用於管理對象的生命周期，當需要對象時，從對象池中獲取，使用完成後再放回對象池。
2. 避免不必要的字符串拼接：在 Java 中，字符串是不可變的，因此每次字符串拼接時都會創建一個新的字符串對象。如果需要頻繁進行字符串拼接操作，應該使用 StringBuilder 或 StringBuffer，它們是可變的，避免不必要的字符串對象創建。
3. 使用靜態工廠方法：靜態工廠方法可以重用對象，避免多次創建相同的對象。它可以在內部維護一個對象池，以根據需要返回現有對象或創建新對象。
4. 盡量使用基本數據類型：基本數據類型（如 int、double）比包裝類對象（如 Integer、Double）更節省內存和具有更好的性能。只在需要時才使用包裝類。
5. 使用緩存：對於可以緩存的數據，例如計算結果或中間結果，可以將其緩存以避免重覆計算或對象創建。
6. 使用不可變對象：不可變對象在多線程環境下更容易共享而不會引起競態條件。不可變對象的創建開銷較小，因為它們的值不會發生變化。
7. 謹慎使用裝箱（Boxing）和拆箱（Unboxing）：將基本數據類型包裝為對象（裝箱）和從對象中提取基本數據類型（拆箱）可能導致不必要的對象創建。避免頻繁的裝箱和拆箱操作。
8. 避免過多的對象引用：對象引用可能導致不必要的對象保持在內存中，應該及時清除不再需要的引用，以便垃圾收集器回收內存。

* 對象池基本範例
```java=
public class ObjectPool<T> {
    private List<T> pool;

    public ObjectPool(int size, Supplier<T> objectFactory) {
        pool = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            pool.add(objectFactory.get());
        }
    }

    public T borrowObject() {
        if (pool.isEmpty()) {
            throw new IllegalStateException("Object pool is empty.");
        }
        return pool.remove(0);
    }

    public void returnObject(T obj) {
        pool.add(obj);
    }
}
```
如果一個對象在被某個線程借用後，尚未被歸還，而其他線程也嘗試借用該對象，通常會有兩種常見的處理方式：
1. 拋出異常：參照以上範例
2. 阻塞等待：可以使用 BlockingQueue 實現，它是 Java 中的線程安全隊列
```java=
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ObjectPoolWithBlockingQueue<T> {
    private BlockingQueue<T> pool;

    public ObjectPoolWithBlockingQueue(int maxPoolSize) {
        pool = new ArrayBlockingQueue<>(maxPoolSize);
    }

    public T borrowObject() throws InterruptedException {
        return pool.take(); // 阻塞等待直到對象可用
    }

    public void returnObject(T obj) throws InterruptedException {
        pool.put(obj);
    }
}
```
## 7.消除過期的對象引用
1. 手動管理對象引用：確保當不再需要對象時，將對象的引用設置為 null，以便垃圾回收器能夠識別並回收這些對象。因為對象的引用可能會一直存在，直到應用程序終止。
2. 避免使用全局對象引用：盡量避免使用全局的對象引用，因為這些引用往往會一直存在，導致對象不能被回收。使用弱引用或軟引用來管理全局引用，以便垃圾回收器可以更自由地回收對象。
3. 使用緩存時小心謹慎：如果您使用緩存來存儲對象，確保緩存中的對象可以根據某些條件（例如時間戳、大小等）自動過期。
4. 避免內部類持有外部類的引用：內部類持有對外部類的引用時，可能導致外部類對象無法被垃圾回收。要避免這種情況，可以使用靜態內部類或弱引用。
5. 即時關閉資源：確保在不再需要時即時關閉文件、數據庫連接、網絡連接等資源，以釋放相關的對象引用。
6. 注意循環引用：確保對象之間的相互引用沒有形成循環引用，否則垃圾回收器無法正常工作。
7. 使用專門的工具和分析器：一些工具和分析器可以幫助您識別和解決內存泄漏問題，例如使用 Java VisualVM 或 Memory Analyzer Tool (MAT)。
* 全局引用（強引用）：
最常見的引用類型，是通過直接賦值給對象變量的方式創建的。只有在沒有任何引用指向對象時才會被回收。垃圾回收器不會主動回收。生命周期由程序員控制，通常在不再需要對象時手動將引用設置為 null。
* 弱引用：
通過 java.lang.ref.WeakReference 類創建。在下一次垃圾回收運行時，如果對象只被弱引用引用，就會被回收。主要用於實現緩存或管理不太重要的對象，當內存緊張時，垃圾回收器可以回收這些對象。
* 軟引用：
通過 java.lang.ref.SoftReference 類創建。在內存不足時，垃圾回收器可以回收對象，但它通常會盡量保留對象，直到內存真正不足。通常用於緩存數據，以允許在內存不足時釋放部分緩存。
```java=
import java.lang.ref.SoftReference;
import java.lang.ref.WeakReference;

public class ReferenceExample {
    public static void main(String[] args) {
        String data = "This is some data.";
        SoftReference<String> softRef = new SoftReference<>(data);
        WeakReference<String> weakRef = new WeakReference<>(data);

        data = null; // 刪除對強引用的引用

        // 嘗試獲取弱引用引用的對象
        String retrievedData = weakRef.get();
        if (retrievedData != null) {
            System.out.println("Data: " + retrievedData);
        } else {
            System.out.println("Data has been garbage collected.");
        }
         
        // 嘗試獲取軟引用引用的對象
        retrievedData = softRef.get();
        if (retrievedData != null) {
            System.out.println("Data: " + retrievedData);
        } else {
            System.out.println("Data has been reclaimed due to low memory.");
        }
    }
}
```
## 8.避免使用 Finalizer 和 Cleaner 機制
1. 不可靠性：執行時間是不確定的，無法保證對象在何時被垃圾回收。這導致了對象的生命周期難以控制。
2. 性能開銷：對性能產生負面影響。它們需要額外的系統資源來執行清理操作，因此可能會導致應用程序的性能下降。
3. 資源泄漏：開發人員很容易出現資源泄漏的情況。這是因為對象的清理操作可能依賴於外部資源，如文件句柄或網絡連接，而這些資源的釋放時間無法控制。
4. 無法捕獲異常：機制中的異常無法捕獲和處理。如果清理操作中發生異常，它們只會被記錄，而不會傳遞給應用程序，因此開發人員無法采取適當的措施來處理異常情況。

相反，推薦使用顯式資源管理和關閉資源的方式，如在 try-with-resources 語句塊中關閉文件或網絡連接。這種方式更可靠，更容易控制資源的生命周期，同時也能提高性能並降低資源泄漏的風險。
```java=
class MyResource implements AutoCloseable {
    public MyResource() {
        // 執行資源初始化操作
        System.out.println("Resource initialized.");
    }

    public void performOperation() {
        // 執行某些操作
        System.out.println("Operation performed.");
    }

    @Override
    public void close() {
        // 關閉資源
        System.out.println("Resource closed.");
    }
}

public class AutoCloseableExample {
    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            // 使用資源執行操作
            resource.performOperation();
        }
        // 資源已自動關閉
    }
}
```
## 9.使用 try-with-resources 語句替代 try-finally 語句
1. 自動關閉資源：try-with-resources 語句會自動關閉在括號內聲明的資源（實現 AutoCloseable 接口的對象），而無需顯式調用 close 方法。
2. 異常處理：還能更好地處理異常。如果在資源的使用過程中發生異常，try-with-resources 會自動關閉資源，並且在異常處理之後，可以使用 getSuppressed 方法獲取與關閉資源相關的異常。這可以減少資源泄漏的風險。
3. 簡潔性：使用 try-with-resources 語法使代碼更簡潔，因為無需編寫顯式的 finally 塊來關閉資源。
```java=
public class SuppressedExceptionExample {
    public static void main(String[] args) {
        try {
            processResource();
        } catch (Exception e) {
            // 處理資源使用時的異常
            e.printStackTrace();
            // 獲取由 try-with-resources 語句抑制的所有異常
            Throwable[] suppressed = e.getSuppressed();
            for (Throwable t : suppressed) {
                System.err.println("Suppressed: " + t.getMessage());
            }
        }
    }

    static void processResource() throws Exception {
        try (AutoCloseableResource resource = new AutoCloseableResource()) {
            // 模擬資源使用過程
            throw new Exception("Resource usage exception");
        }
    }
}

class AutoCloseableResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new Exception("Exception during resource close");
    }
}
```
## 10.重寫 equals 方法時遵守通用約定
1. 自反性（Reflexive）：equals 方法必須是自反的，即對於任何非空引用值 x，x.equals(x) 必須返回 true。換句話說，對象必定等於自身。
2. 對稱性（Symmetric）：equals 方法必須是對稱的，即對於任何非空引用值 x 和 y，如果 x.equals(y) 返回 true，那麽 y.equals(x) 也必須返回 true。換句話說，對象間的相等性檢查不受順序影響。
3. 傳遞性（Transitive）：equals 方法必須是傳遞的，即對於任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，且 y.equals(z) 也返回 true，那麽 x.equals(z) 也必須返回 true。
4. 一致性（Consistent）：equals 方法在多次調用中的行為必須保持一致。如果對象的比較條件沒有改變，那麽無論調用多少次，equals 方法的結果都應該一致。
5. 非空性（Non-null）：equals 方法必須能夠處理 null 值，即對於任何非空引用值 x，x.equals(null) 必須返回 false。

為了滿足這些約定，通常需要按照以下步驟重寫 equals 方法：

* 檢查是否是相同的引用。如果是相同的引用（即對象的地址相同），則返回 true。
* 檢查是否是 null。如果傳入的對象為 null，則返回 false。
* 檢查對象是否屬於相同的類。如果不是相同的類，通常返回 false。
* 比較對象的關鍵屬性，以確定它們是否相等。這可能需要比較字段、屬性或其他標識對象相等性的內容。
* 返回比較的結果。

請注意，重寫 equals 方法時，還應該同時重寫 hashCode 方法，以確保哈希碼的一致性，這是因為在使用哈希表等數據結構時，hashCode 方法的一致性非常重要。
## 11.重寫 equals 方法時同時也要重寫 hashcode 方法
1. equals 方法定義了對象相等性的規則：equals 方法用於確定兩個對象是否在邏輯上相等。你可以根據對象的內部狀態來定義相等性的規則。如果兩個對象在你的應用程序中被視為相等，那麽它們的 equals 方法應該返回 true。
2. hashCode 方法生成散列碼：hashCode 方法用於生成一個整數散列碼（哈希碼），通常用於將對象存儲在散列表（如 HashMap）中。散列表根據散列碼將對象分組並進行快速查找。因此，hashCode 方法的實現應該與 equals 方法保持一致，即對於相等的對象，它們應該生成相同的散列碼。

這就是為什麽在重寫 equals 方法時必須同時重寫 hashCode 方法的原因。如果你不重寫 hashCode 方法，那麽相等的對象可能會具有不同的散列碼，這會導致在使用散列表等數據結構時出現問題，因為對象將無法正確查找和存儲。
```java=
import java.util.Objects;

public class MyClass {
    private int id;
    private String name;

    // 構造函數和其他方法

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        MyClass myClass = (MyClass) o;
        return id == myClass.id && Objects.equals(name, myClass.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```
## 12.始終重寫 toString 方法
1. 提供有用的信息：重寫的 toString 方法應該返回包含有關對象狀態的有用信息。這可以包括對象的字段值、標識信息或其他描述性信息，取決於對象的類型和用途。
2. 避免返回敏感信息：不要在 toString 方法中返回敏感或私有信息，以免泄漏敏感數據。如果需要記錄敏感信息，應該使用專門的日志工具，並根據需求進行合適的安全性處理。
3. 格式清晰：返回的字符串應該具有清晰的格式，使其易於閱讀和解釋。考慮使用標準的格式或約定，以便其他開發人員能夠理解並使用這些信息。
4. 考慮性能：雖然 toString 方法通常用於調試目的，但仍應考慮性能。不要生成過於覆雜或冗長的字符串，以避免對性能造成不必要的負擔。
5. 重寫基類的 toString 方法：如果你的類是其他類的子類，通常應該重寫父類的 toString 方法以包含子類特有的信息，並可以調用 super.toString() 來包含父類信息。
## 13.謹慎地重寫 clone 方法
1. 考慮實現 Cloneable 接口：如果你打算重寫 clone 方法以支持對象的克隆，首先要確保你的類實現了 Cloneable 接口。
2. 使用 super.clone()：在重寫 clone 方法時，通常應該調用 super.clone() 方法以獲得對象的淺拷貝。它會創建一個新對象，但不會覆制對象內部的引用類型字段。這意味著原始對象和克隆對象的某些部分可能會共享相同的引用對象。
3. 深度覆制和不可變性：如果一個類希望支持深拷貝（deep copy），即在克隆過程中也覆制對象內部的引用類型字段，那麽該類需要在重寫 clone 方法時進行深度覆制操作。這需要程序員自己實現，而不是由 Cloneable 或 Object 類提供。如果對象是不可變的，深度覆制通常是不必要的。
4. 不要調用構造函數：不要在 clone 方法中調用構造函數來創建新對象，因為這將導致對象狀態被重新初始化。應該使用 super.clone() 或其他適當的方式來創建新對象。
5. 異常處理：在 clone 方法中處理異常是非常重要的。如果你的對象包含可拋出異常的操作（如 IO 操作），那麽在 clone 方法中捕獲並處理這些異常是必要的。
6. 覆制相關對象：如果你的對象引用其他對象，你需要確保這些相關對象也能夠正確克隆。這可能需要遞歸地調用 clone 方法。
7. 返回類型：clone 方法應該返回相同類型的對象，通常使用類型轉換。例如，如果你的類是 MyClass，則 clone 方法應該返回 MyClass 類型。
8. 文檔說明：應該在 clone 方法的文檔中明確說明克隆的行為，包括是否進行深度覆制、是否克隆相關對象等。
9. 考慮替代方法：有時，使用其他方式（例如拷貝構造函數或工廠方法）來創建對象的副本可能更加清晰和安全。因此，應該在真正需要 clone 方法的情況下使用它。

總之，重寫 clone 方法需要小心謹慎，因為它可能會引發覆雜的問題。應該明確克隆的行為，確保正確處理異常，並確保克隆對象的狀態與原始對象保持一致。在某些情況下，考慮使用其他覆制方式可能更好。
## 14.考慮實現 Comparable 接口
1. 實現Comparable接口：首先，你的類應該顯式聲明實現Comparable接口，該接口是一個泛型接口，你需要指定你的類類型作為泛型參數。
2. 實現compareTo方法：在你的類中實現compareTo方法，在compareTo方法中，你需要定義自然排序的規則。通常，你會比較對象的某些屬性或字段，以確定它們的順序。
3. 使用compareTo進行排序：一旦你的類實現了Comparable接口，你可以使用Java提供的排序方法，如Collections.sort()（用於List集合）或Arrays.sort()（用於數組）來排序對象。

另可搭配Comparator類
Comparator 是一個獨立的類，用於為對象的排序提供外部的比較邏輯。
你可以創建一個獨立的 Comparator 實現，該實現包含排序邏輯，而不需要修改被排序的類的源代碼。
通過將 Comparator 對象傳遞給排序方法，如 Collections.sort()，你可以控制對象的排序方式，而不必修改對象的類或實現 Comparable 接口。
```java=
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

class Book implements Comparable<Book> {
    private String title;
    private int publicationYear;

    public Book(String title, int publicationYear) {
        this.title = title;
        this.publicationYear = publicationYear;
    }

    public int getPublicationYear() {
        return publicationYear;
    }

    @Override
    public int compareTo(Book other) {
        return Integer.compare(this.publicationYear, other.publicationYear);
    }

    @Override
    public String toString() {
        return "Book{title='" + title + "', publicationYear=" + publicationYear + '}';
    }
}

public class BookSortingExample {
    public static void main(String[] args) {
        List<Book> books = new ArrayList<>();
        books.add(new Book("Book A", 2005));
        books.add(new Book("Book B", 1998));
        books.add(new Book("Book C", 2010));

        // 使用默認自然排序（按出版年份）
        Collections.sort(books);

        System.out.println("Sorted by publication year:");
        for (Book book : books) {
            System.out.println(book);
        }

        // 使用 Comparator 來按書名排序
        Comparator<Book> titleComparator = Comparator.comparing(Book::getTitle);
        books.sort(titleComparator);

        System.out.println("Sorted by title:");
        for (Book book : books) {
            System.out.println(book);
        }
    }
}
```
## 15.使類和成員的可訪問性最小化
1. 使用訪問修飾符：根據需要，選擇最合適的修飾符來限制訪問。
2. 使用private修飾字段：將類的字段標記為private，以防止直接訪問和修改。通過提供公共方法（getter和setter）來控制字段的訪問。
3. 使用不可變類：如果可能，設計不可變類，其中所有字段都是final和private的，且沒有公共setter方法。確保對象的狀態不可更改。
4. 使用接口和抽象類：通過定義接口和抽象類，將類的實現細節與接口分離，以隱藏實現細節並提供一致的公共接口。
5. 將方法標記為final或private：對於不希望被子類覆蓋的方法，將其標記為final。對於只在本類內部使用的方法，將其標記為private。
6. 使用包作用域：將類、字段或方法聲明為默認（包作用域）的，以便它們只能在同一包內部訪問。這有助於實現隱藏內部細節的目標。
7. 使用內部類：將某些類限制在另一個類的內部，以確保只有包含類的代碼可以訪問它們。內部類可以有效地隱藏實現細節。
8. 謹慎使用反射：反射機制允許繞過可訪問性限制，因此要小心使用它，以確保代碼的安全性和封裝性。
## 16.在公共類中使用訪問方法而不是公共屬性
1. 封裝性：使用訪問方法可以隱藏對象內部的實現細節，使類的內部狀態對外部代碼不可見。
2. 控制：通過訪問方法，你可以在讀取或修改屬性時添加額外的邏輯。例如，你可以在setter方法中添加驗證邏輯，確保屬性值的有效性。這有助於保持對象的一致性。
3. 可變性控制：使用只有getter方法而沒有setter方法的屬性可以創建不可變類，從而提高了線程安全性和可靠性。
4. 兼容性：如果在後續版本中需要添加更多的邏輯或控制，你可以在不破壞現有的客戶端代碼的情況下修改訪問方法。如果使用公共屬性，任何直接訪問該屬性的客戶端代碼都可能受到影響。
## 17.最小化可變性
1. 設計不可變類：不可變類是那些創建後狀態不可更改的類。這些類的字段都應該聲明為final，並且不提供修改字段值的方法（setter）。不可變類是線程安全的，因為它們的狀態不會發生變化。
2. 使類和字段私有：將類的字段聲明為私有，以確保它們不會被外部直接訪問。通過提供訪問方法（getter和setter）來控制字段的訪問。甚至不提供setter。
3. 防禦性編程：若有setter方法則在其中添加驗證邏輯，以確保輸入的數據是有效的，防止無效數據進入對象。
4. 避免共享可變狀態：如果多個對象需要共享數據，確保共享的數據是不可變的，或者通過覆制來共享。共享可變狀態會引發並發問題，因此應盡量避免。
```java=
public class MutableList {
    private List<String> data = new ArrayList<>();

    public List<String> getData() {
        return new ArrayList<>(data); // 返回字段的副本
    }

    public void addData(String value) {
        data.add(value);
    }
}
```
5. 使用不可變集合：Java提供了不可變集合類（如Collections.unmodifiableXXX()），它們允許你創建不可變的集合，從而避免集合數據的不安全變化。
```java=
public class ImmutableCollectionExample {
    private final List<String> names;

    public ImmutableCollectionExample(List<String> names) {
        this.names = Collections.unmodifiableList(new ArrayList<>(names));
    }

    public List<String> getNames() {
        return names;
    }
}
```
6. 謹慎使用clone方法：clone方法可以用來創建對象的拷貝，但要小心處理，確保深拷貝以避免共享可變狀態。
## 18.組合優於繼承
以下是繼承可能遇到的錯誤
```java=
public class InstrumentedHashSet<E> extends HashSet<E> {
// The number of attempted element insertions
    private int addCount = 0;

    @Override
    public boolean add(E e) {
    	addCount++;
    	return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
    	addCount += c.size();
    	return super.addAll(c);
    }

    public int getAddCount() {
    	return addCount;
    }
	
    public static void main(String[] args) {
    	InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
    	s.addAll(List.of("Snap", "Crackle", "Pop"));
    	System.out.println(s.getAddCount()); // 6
    }
}
```
我們期望 getAddCount 方法返回的結果是 3，但實際上返回了 6。在 HashSet 內部, addAll方法是基於它的 add 方法來實現的，即使 HashSet 文檔中沒有指名其實現細節。
InstrumentedHashSet 中的 addAll 方法首先給 addCount 屬性設置為 3，然後使用 super.addAll 方法調用了 HashSet 的 addAll 實現。然後反過來又調用在 InstrumentedHashSet 類中重寫的 add 方法，每個元素調用一次。這三次調用又分別給 addCount 加 1，所以，一共增加了 6：通過 addAll 方法每個增加的元素都被計算了兩次。

避免問題應優先使用組合，通過將現有類的對象嵌入到新類中
1. 松散耦合： 使用組合可以創建松散耦合的類，因為新類僅依賴於其成員對象，而不繼承其父類的實現。
2. 靈活性： 組合允許你動態選擇要包含的成員對象，而不受繼承層次結構的限制。
3. 避免深層次的繼承： 深層次的繼承層次結構往往難以理解和維護。
4. 避免"死胎"問題： 繼承時，可能會繼承一些不需要的行為，導致浪費和混亂。使用組合允許你明確定義需要的行為，避免不必要的代碼。
5. 避免覆蓋方法： 繼承時，可能會覆蓋父類的方法，但這可能導致不必要的覆雜性。
```java=
// 组合示例
class Engine {
    void start() {
        System.out.println("Engine started.");
    }
}

class Car {
    private final Engine engine = new Engine();

    void start() {
        engine.start();
        System.out.println("Car started.");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.start();
    }
}
```
## 19.如使用繼承設計，應當文檔說明，否則不該使用
應該清晰地文檔化你的類的行為，包括父類和子類之間的關系、如何覆蓋父類方法以及如何擴展父類功能。這可以通過注釋、文檔或其他文檔化方式來完成，以便其他開發者能夠理解你的設計意圖。

除非你知道有一個真正的子類需要，否則你可能最好是通過聲明你的類為 final 禁止繼承，或者確保沒有可訪問的構造方法。
* @implSpec：在代碼文檔中的方法說明中使用，用於提醒其他開發者或實現方法的開發者關於方法應該如何被實現，以及方法的一些特定規範或要求。
```java=
/**
 * Returns the sum of two numbers.
 *
 * @param a The first number.
 * @param b The second number.
 * @return The sum of a and b.
 *
 * @implSpec This method should handle both positive and negative numbers.
 *           The result should not overflow.
 */
public int add(int a, int b) {
    // Implementation of the add method
    // ...
}
```

## 20.接口優於抽象類
1. 多重繼承：在許多編程語言中，類只能單繼承，這意味著如果一個類已經繼承了一個抽象類，它無法再繼承其他類，但它可以實現多個接口。
2. 松散耦合：接口是一種更松散耦合的方式。當一個類實現一個接口時，它只需要實現接口中定義的方法，而不需要繼承所有抽象類的實現。
3. 適用於不相關的類：接口可以用於不相關的類，它們可能沒有共同的繼承層次結構，但可以實現相同的接口以提供相似的功能。
4. 適用於輕量級類：抽象類通常包含一些具體的方法實現，而接口只包含方法的簽名。
## 21.為後代設計接口
1. 抽象性：接口應該是足夠抽象，以允許不同的後代實現，但不應該過於具體。過於具體的接口可能會限制後代實現的靈活性。
2. 穩定性：接口的設計應該是穩定的，不容易頻繁更改。頻繁修改接口可能會破壞依賴它的實現類，導致代碼的不穩定性。
3. 擴展性：接口應該容易擴展，以適應未來的需求。這包括添加新的方法或屬性而不會破壞現有的實現。
4. 向後兼容性：對接口的更改應該是向後兼容的，這意味著現有的實現類不會因為接口的變化而中斷。這可以通過默認方法（Java 8及更高版本支持的功能）來實現。
```java=
// 定義一個接口
interface MyInterface {
    // 抽象方法
    void abstractMethod();

    // 默認方法
    default void defaultMethod() {
        System.out.println("This is a default method.");
    }
}

// 實現接口的類
class MyClass implements MyInterface {
    @Override
    public void abstractMethod() {
        System.out.println("Abstract method implementation.");
    }
}

public class Main {
    public static void main(String[] args) {
        MyClass myClass = new MyClass();

        // 調用抽象方法
        myClass.abstractMethod();  // 輸出: Abstract method implementation

        // 調用默認方法
        myClass.defaultMethod();   // 輸出: This is a default method.
    }
}
```
## 22.接口僅用來定義類型
接口應該用於定義某種類型或契約，而不應該用於實現。這意味著接口的主要目的是為類提供一個共同的類型，並定義這種類型應該具有的方法簽名，而不是為了提供方法的具體實現。
* 不應使用常量接口：建議使用類或枚舉來定義常量，而不是將它們放入接口中。
* 默認方法的合理使用：通常涉及在已經存在的接口中添加新方法，以便現有的實現類不必立即提供這些新方法的實現。當然實現類可選擇是否覆寫默認方法。
## 23.優先使用類層次而不是標簽類
標簽類（Tagged Class）是一種特殊的類設計，它包含一個或多個標簽字段，這些字段用來標識對象的類型、狀態或屬性。標簽類通常使用整數、字符串或枚舉類型的字段來區分不同的對象。
1. 標簽字段：標簽類包含一個或多個標簽字段，這些字段用於標識對象的類型或狀態。標簽字段的值通常是整數、字符串或枚舉。
2. 條件分支：在使用標簽類的代碼中通常需要條件分支語句來根據標簽字段的值執行不同的邏輯。
3. 類型混合：標簽類中的對象通常具有不同的類型，但它們在同一個類中共存，這可能導致類型混亂和不一致性。
4. 難以擴展：如果需要添加新的類型或狀態，標簽類的設計可能需要修改，而不是簡單地添加新的類或枚舉值。
5. 缺乏類型安全性：標簽字段通常沒有類型安全性，因為編譯器無法強制執行類型檢查。
```java=
//Tagged class - vastly inferior to a class hierarchy!
class Figure {
	enum Shape {
		RECTANGLE, CIRCLE
	};

//Tag field - the shape of this figure
	final Shape shape;
//These fields are used only if shape is RECTANGLE
	double length;
	double width;
//This field is used only if shape is CIRCLE
	double radius;

//Constructor for circle
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

// Constructor for rectangle
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	double area() {
		switch (shape) {
		case RECTANGLE:
			return length * width;
		case CIRCLE:
			return Math.PI * (radius * radius);
		default:
			throw new AssertionError(shape);
		}
	}
}
```
## 24.優先考慮靜態成員類
1. 清晰性：靜態成員類與外部類之間的關系更清晰，因為它們不依賴於外部類的實例。
2. 降低耦合：靜態成員類與外部類之間的耦合較低，使得代碼更容易理解和維護。
3. 性能：內部非靜態類會在每個實例中保存對外部類的引用，而靜態成員類不需要。這可以提高內存利用率和性能。

四種嵌套類
* 靜態成員類（Static Member Class）：
嵌套在外部類內部的類，它使用 static 修飾。
不依賴於外部類的實例，可以像普通類一樣實例化。
示例：輔助類、工具類、叠代器等。
使用方式：OuterClass.StaticMemberClass innerObject = new OuterClass.StaticMemberClass();
* 非靜態成員類（Inner Class）：
嵌套在外部類內部的類，不使用 static 修飾。
可以訪問外部類的實例字段和方法，通常用於與外部類密切關聯的操作。
示例：事件處理、面板內部組件、叠代器等。
使用方式：需要先創建外部類的實例，然後通過該實例創建內部類的實例：OuterClass outer = new OuterClass(); OuterClass.InnerClass innerObject = outer.new InnerClass();
* 局部類（Local Class）：
定義在方法內部的類，通常用於局部作用域。
可以訪問方法內的局部變量，但這些變量必須是 final 或 effectively final（Java 8+）。
示例：方法內部的幫助類、適配器、過濾器等。
使用方式：定義在方法內，通常用於特定方法中的邏輯。
* 匿名類（Anonymous Class）：
沒有名字的類，通常用於創建臨時對象或實現接口。
通常作為方法參數傳遞給其他方法，用於實現回調、事件處理等。
示例：事件處理、接口實現、簡單的線程、回調函數等。
使用方式：通常在方法內部創建匿名類的實例，可以直接實現接口或繼承類並重寫方法。
```java=
public class OuterClass {
    private static int outerField = 10;

    public static class StaticMemberClass {
        public void staticMethod() {
            // 可以在靜態成員類中訪問外部類的靜態成員
            System.out.println("Static Member Class - outerField: " + outerField);
        }
    }

    public class InnerClass {
        public void innerMethod() {
            // 可以在非靜態成員類中訪問外部類的實例成員
            System.out.println("Inner Class - outerField: " + outerField);
        }
    }

    public void someMethod() {
    	int localVariable = 42;

        class LocalClass {
            public void localMethod() {
                // 可以在局部類中訪問方法內的局部變量
                System.out.println("Local Class - localVariable: " + localVariable);
            }
        }
        LocalClass local = new LocalClass();
        local.localMethod();

        Runnable anonymousClass = new Runnable() {
            @Override
            public void run() {
                // 可以在匿名類中實現接口方法
                System.out.println("Anonymous Class - outerField: " + outerField);
            }
        };
        anonymousClass.run();
    }

    public static void main(String[] args) {
    	OuterClass.StaticMemberClass staticMember = new OuterClass.StaticMemberClass();
    	staticMember.staticMethod();

        OuterClass outer = new OuterClass();
        InnerClass inner = outer.new InnerClass();
        inner.innerMethod();

        outer.someMethod();
    }
}
```
## 25.將源文件限制為單個頂級類
頂級類（Top-level Class）是指沒有嵌套在其他類中的類。這些類是獨立的，它們不是其他類的內部類或成員類，而是在頂級命名空間中定義的類。
* 內部類（Inner Classes）不受此規則限制：內部類是位於其他類內部的類，因此它們可以存在於同一個源文件中，並不受單個頂級類規則的限制。一個源文件可以包含多個內部類，每個內部類都有自己的定義。
## 26.不要使用原始類型
原始類型在類型安全性方面存在問題。它們不提供編譯時類型檢查，因此可以在運行時引發未檢查的類型轉換異常。使用泛型可以在編譯時捕獲類型錯誤，提高代碼的安全性、並使代碼更具可讀性。
| 術語 | 中文含義 | 舉例 |
| -------- | -------- | -------- |
|Parameterized type| 參數化類型 |List`<`String`>` |
|Actual type parameter |實際類型參數| String |
|Generic type |泛型類型 |List`<`E`>` |
|Formal type parameter |形式類型參數| E |
|Unbounded wildcard type| 無限制通配符類型| List`<`?`>` |
|Raw type |原始類型| List |
|Bounded type parameter |限制類型參數| `<`E extends Number`>` |
|Recursive type bound |遞歸類型限制 |`<`T extends Comparable`<`T`>>` |
|Bounded wildcard type |限制通配符類型 |List`<`? extends Number`>` |
|Generic method |泛型方法 |static `<`E`>` List`<`E`>` asList(E[] a) |
|Type token |類型令牌 |String.class |
## 27.消除非檢查警告
在某些情況下，你可能確切地知道某個操作是類型安全的，但編譯器無法推斷出來。在這種情況下，可以使用 @SuppressWarnings 注解來告訴編譯器忽略特定類型的警告。
## 28.列表優於數組
列表是泛型類型，可以提供類型安全性。你可以定義一個列表，其中只能包含特定類型的元素。數組沒有這種類型檢查，因此容易導致運行時的數組存取錯誤。
```java=
public class Chooser<T> {
    private final T[] choiceArray;
    public Chooser(Collection<T> choices) {
    	choiceArray = (T[]) choices.toArray(); // 這沒有了錯誤，而是得到一個警告：Type safety: Unchecked cast from Object[] to T[] 
    }

    private final List<T> choiceList;
    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices); // 編譯時沒有錯誤或警告
    }
}
```
## 29.優先考慮泛型
使用泛型可以讓編譯器在編譯時檢查類型，避免在運行時出現類型錯誤。使用時，應盡量避免使用原始類型和無限定通配符類型。原始類型會失去泛型的優勢，而無限定通配符類型可能導致類型不確定性。
## 30.優先使用泛型方法
泛型方法允許你在方法級別指定類型參數，這意味著你可以在編譯時進行類型檢查，從而減少在運行時出現的類型錯誤。
```java=
public static <E> Set<E> union2(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
## 31.使用限定通配符來增加 API 的靈活性
* 上界通配符通常在需要讀取集合中的元素、傳遞集合到方法中或編寫通用方法時使用。
```java=
public static double sumOfList(List<? extends Number> list) {
    double sum = 0.0;
    for (Number number : list) {
        sum += number.doubleValue();
    }
    return sum;
}
```
* 下界通配符通常在需要向集合中添加元素、處理集合中的元素或編寫通用的泛型類或接口時使用。
```java=
public static void addIntegers(List<? super Integer> list) {
    list.add(1);
}
```
## 32.合理地結合泛型和可變參數
可變參數和泛型不能很好地交互，因為可變參數機制是在數組上面構建的脆弱的抽象，並且數組具有與泛型不同的類型規則。 雖然泛型可變參數不是類型安全的，但它們是合法的。 如果選擇使用泛型(或參數化)可變參數編寫方法，請首先確保該方法是類型安全的，然後使用 @SafeVarargs 注解對其進行標注，以免造成使用不愉快。
```java=
static void dangerous(List<String>... stringLists) { // 警告Type safety: Potential heap pollution via varargs parameter stringLists
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // Heap pollution
    String s = stringLists[0].get(0); // ClassCastException
}
```
"堆污染"（Heap Pollution）是在Java中用於泛型的術語，指的是在編譯時允許的類型不安全操作，這可能導致在運行時出現類型不匹配的異常。
## 33.優先考慮類型安全的異構容器
異構容器是一種可以存儲多種類型元素的容器，通常用於需要跨越不同數據類型的情況。例如，異構容器可以用於實現插件系統，存儲不同類型的插件或處理來自不同源的數據。
可以通過將類型參數放在鍵上而不是容器上來解決此限制。 可以使用 Class 對象作為此類型安全異構容器的鍵。 以這種方式使用的Class 對象稱為類型令牌。 也可以使用自定義鍵類型。例如，可以有一個表示數據庫行(容器)的DatabaseRow 類型和一個泛型類型 Column`<`T`>` 作為其鍵。
```java=
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), instance);
	}

	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type));
	}

	public static void main(String[] args) {
		Favorites f = new Favorites();
		f.putFavorite(String.class, "Java");
		f.putFavorite(Integer.class, 0xcafebabe);
		f.putFavorite(Class.class, Favorites.class);
		String favoriteString = f.getFavorite(String.class);
		int favoriteInteger = f.getFavorite(Integer.class);
		Class<?> favoriteClass = f.getFavorite(Class.class);
		System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
	}
}
```
## 34.使用枚舉類型替代整型常量
```java=
Day today = Day.MONDAY;
if (today == Day.SATURDAY || today == Day.SUNDAY) {
    System.out.println("It's the weekend!");
} else {
    System.out.println("It's a weekday.");
}
// ------------------------------------------
int today = 2; // 假設2代表星期一
if (today == 0 || today == 6) {
    System.out.println("It's the weekend!");
} else {
    System.out.println("It's a weekday.");
}
```
1. 枚舉類型的順序： 枚舉常量的順序非常重要，因為它們的順序決定了它們的 ordinal 值。在添加、刪除或重新排列枚舉常量時要小心，以免破壞現有代碼。
2. values() 方法： 枚舉類型具有一個 values() 方法，該方法返回包含所有枚舉常量的數組。這個方法通常用於叠代枚舉常量。但要注意，它返回一個新的數組實例，不要修改該數組，否則可能會影響其他代碼。
3. 不可變性： 枚舉常量是不可變的，不能在運行時修改它們的值。這是枚舉的一個關鍵特性，確保常量的穩定性。
4. valueOf() 方法： 枚舉類型具有一個 valueOf(String name) 方法，用於將字符串轉換為相應的枚舉常量。要小心處理可能引發 IllegalArgumentException 的情況，例如在給定字符串與任何枚舉常量的名稱都不匹配時。
5. 自定義字段和方法： 枚舉類型可以具有自定義字段和方法，但要小心確保它們不會導致意外行為或破壞枚舉常量的不可變性。
6. null 值： 枚舉常量不會為 null，因此在處理枚舉類型時要小心檢查 null 值。

帶參數的枚舉
```java=
public enum Fruit {
    APPLE("Apple", "Red"),
    BANANA("Banana", "Yellow"),
    ORANGE("Orange", "Orange"),
    STRAWBERRY("Strawberry", "Red"),
    GRAPE("Grape", "Purple");

    private final String name;
    private final String color;

    Fruit(String name, String color) {
        this.name = name;
        this.color = color;
    }

    public String getName() {
        return name;
    }

    public String getColor() {
        return color;
    }

    @Override
    public String toString() {
        return name + " (" + color + ")";
    }
}

public class EnumExample {
    public static void main(String[] args) {
        System.out.println("Fruit Details:");
        for (Fruit fruit : Fruit.values()) {
            System.out.println(fruit.getName() + " is " + fruit.getColor());
        }

        Fruit favoriteFruit = Fruit.APPLE;
        System.out.println("My favorite fruit is " + favoriteFruit);

        // You can also use switch statements with the enum
        switch (favoriteFruit) {
            case APPLE:
                System.out.println("I like apples!");
                break;
            case BANANA:
                System.out.println("I like bananas!");
                break;
            default:
                System.out.println("I like other fruits too.");
        }
    }
}
```
## 35.使用實例屬性替代序數
所有枚舉都有一個 ordinal 方法，它返回每個枚舉常量類型的數值位置。但是，永遠不要從枚舉的序號中得出與它相關的值；請將其保存在實例屬性中。甚至避免使用ordinal 方法。
```java=
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9),
    DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) {
    	this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
    	return numberOfMusicians;
    }
}
```
## 36.使用 EnumSet 替代位屬性
位屬性（Bit Flags）是一種用二進制位來表示和存儲多個開關狀態或選項的方式。通常，每個位都代表一種狀態或選項，可以將它們設置為 1 或 0 來表示開啟或關閉。這種技術通常用於節省內存和提高性能，因為每個位可以表示多個狀態，而不是使用更大的數據類型來存儲每個狀態。
以下是一個使用位屬性的示例，表示一個文件的權限：
```java=
public class FileAccess {
    private static final int READ = 1;   // 0001
    private static final int WRITE = 2;  // 0010
    private static final int EXECUTE = 4;  // 0100

    private int permissions = 0;

    public void addPermission(int permission) {
        permissions |= permission;
    }

    public void removePermission(int permission) {
        permissions &= ~permission;
    }

    public boolean hasPermission(int permission) {
        return (permissions & permission) == permission;
    }

    public static void main(String[] args) {
        FileAccess file = new FileAccess();

        file.addPermission(READ); // 添加讀權限
        file.addPermission(WRITE); // 添加寫權限

        System.out.println("File has read permission: " + file.hasPermission(READ));
        System.out.println("File has write permission: " + file.hasPermission(WRITE));
        System.out.println("File has execute permission: " + file.hasPermission(EXECUTE));
    }
}
```
EnumSet 是 Java 中的一個專門為枚舉類型設計的集合類，只能包含枚舉類型的值。此例改用EnumSet可以更清晰和類型安全地管理權限，避免了使用位屬性時的位運算操作和可能的錯誤：
```java=
import java.util.EnumSet;
enum Permission {
    READ, WRITE, EXECUTE, DELETE
}

public class File {
    private String name;
    private EnumSet<Permission> permissions;

    public File(String name) {
        this.name = name;
        this.permissions = EnumSet.noneOf(Permission.class); // 初始化為空的權限集合
    }

    public void addPermission(Permission permission) {
        permissions.add(permission);
    }

    public void removePermission(Permission permission) {
        permissions.remove(permission);
    }

    public boolean hasPermission(Permission permission) {
        return permissions.contains(permission);
    }

    public String getName() {
        return name;
    }

    public static void main(String[] args) {
        File file = new File("myFile");

	file.addPermission(Permission.READ); // 添加讀權限
	file.addPermission(Permission.WRITE); // 添加寫權限

	System.out.println("File has read permission: " + file.hasPermission(Permission.READ));
	System.out.println("File has write permission: " + file.hasPermission(Permission.WRITE));
	System.out.println("File has execute permission: " + file.hasPermission(Permission.EXECUTE));
    }
}
```
## 37.使用 EnumMap 替代序數索引
```java=
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

public class WeekdayTaskPlanner {
    private String[] tasks = new String[Day.values().length];

    public void setTask(Day day, String task) {
        if (day.ordinal() >= 0 && day.ordinal() < tasks.length) {
            tasks[day.ordinal()] = task;
        }
    }

    public String getTask(Day day) {
        if (day.ordinal() >= 0 && day.ordinal() < tasks.length) {
            return tasks[day.ordinal()];
        }
        return null;
    }

    public static void main(String[] args) {
        WeekdayTaskPlanner planner = new WeekdayTaskPlanner();
        planner.setTask(Day.MONDAY, "Do the laundry");
        planner.setTask(Day.WEDNESDAY, "Go to the gym");

        System.out.println("Monday's task: " + planner.getTask(Day.MONDAY));
        System.out.println("Wednesday's task: " + planner.getTask(Day.WEDNESDAY));
    }
}
```
易出錯: 如果我們更改 Day 枚舉的順序或添加/刪除枚舉常量，就需要相應地更新數組的大小和索引值，容易出錯。
不具備類型安全性: 如果我們不小心使用一個不合法的序數作為索引，會導致數組越界或產生不正確的結果。

透過EnumMap ，EnumMap 是 Java 中的一個集合類，專門用於處理枚舉類型的鍵值對。它的內部實現是一個數組，索引與枚舉常量的順序相對應。
```java=
import java.util.EnumMap;

public class WeekdayTaskPlanner {
    private EnumMap<Day, String> tasks = new EnumMap<>(Day.class);

    public void setTask(Day day, String task) {
        tasks.put(day, task);
    }

    public String getTask(Day day) {
        return tasks.get(day);
    }
}
```
## 38.使用接口模擬可擴展的枚舉
雖然不能編寫可擴展(繼承)的枚舉類型，但是可以編寫一個接口來配合實現接口的基本的枚舉類型，來對它進行模擬。
```java=
interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
	public double apply(double x, double y) {
		return x + y;
	}
    },
    MINUS("-") {
	public double apply(double x, double y) {
		return x - y;
	}
    },
    TIMES("*") {
	public double apply(double x, double y) {
		return x * y;
	}
    },
    DIVIDE("/") {
	public double apply(double x, double y) {
		return x / y;
	}
    };

    private final String symbol;

    BasicOperation(String symbol) {
	this.symbol = symbol;
    }

    @Override
    public String toString() {
	return symbol;
    }
}
```
## 39.注解優於命名模式
當可以使用注解清楚地表示用意，沒有理由使用命名模式。
例如：測試框架、Spring MVC
## 40.始終使用 Override 注解
1. 提高可讀性：通過在方法前面添加 @Override 注解，你可以明確表示你的意圖，即你是有意要重寫父類中的方法。
2. 防止錯誤：如果你使用 @Override 注解，當你的方法名稱與父類中的方法名稱拼寫錯誤或參數列表不匹配時，編譯器將產生錯誤，這有助於在編譯時捕獲錯誤，而不是在運行時。
## 41.使用標記接口定義類型
標記接口（Marker Interface）是一種特殊類型的接口，它不包含任何方法定義。它僅用於標識一個類屬於某個特定的類別或具有某種特定的行為。
1. 類別標識：標記接口可以用於標識類屬於某個特定的類別。例如，Java的Serializable接口用於標記可序列化的類，以便在序列化和反序列化過程中進行處理。
2. 行為標識：標記接口可以用於標識類具有某種行為或特性。例如，Cloneable接口表示類可以被克隆。

當然也可自定義接口：
```java=
// 定義一個標記接口
interface MyMarkerInterface {
}

// 實現一個類，標記它屬於MyMarkerInterface類別
class MyClass implements MyMarkerInterface {
    // 類的實現
}

public class Main {
    public static void main(String[] args) {
        MyClass obj = new MyClass();
        if (obj instanceof MyMarkerInterface) {
            System.out.println("obj屬於MyMarkerInterface類別");
        }
    }
}
```
## 42.lambda 表達式優於匿名類
Java 8 引入的語法，簡潔易讀並帶有更好的性能，自身的類型推斷功能，可以根據上下文自動推斷參數類型，使代碼更具靈活性。
Lambda 表達式通常用於函數式接口，它可以用來替代匿名內部類：
```java=
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, Anonymous Inner Class!");
    }
};
// ------------------------------------
Runnable runnable = () -> {
    System.out.println("Hello, Lambda!");
};
// 更簡潔的寫法：
Runnable runnable = () -> System.out.println("Hello, Lambda!");
```
預定義的函數式接口
```java=
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(new Consumer<String>() {
    @Override
    public void accept(String name) {
        System.out.println("Hello, " + name);
    }
});
// ------------------------------------
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println("Hello, " + name));
```
## 43.方法引用優於 lambda 表達式
另外Java 8 引入的方法引用（Method Reference）更是一種簡化Lambda表達式的語法：
1. 靜態方法引用： 這是對靜態方法的引用，語法是ClassName::staticMethodName。例如：
```java=
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(new Consumer<String>() {
    @Override
    public void accept(String name) {
        System.out.println(name);
    }
});

names.forEach(name -> System.out.println(name));
// ------------------------------------
names.forEach(System.out::println);
```
這里System.out::println引用了System.out類的println方法。

2. 實例方法引用： 這是對現有對象的實例方法的引用，語法是object::instanceMethodName。例如：
```java=
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
PrintHelper printHelper = new PrintHelper();
names.forEach(new Consumer<String>() {
    @Override
    public void accept(String name) {
        printHelper.print(name);
    }
});
// ------------------------------------
names.forEach(name -> printHelper.print(name));
// ------------------------------------
names.forEach(printHelper::print);
```
printHelper::print引用了PrintHelper對象的print方法。

3. 類構造方法引用： 這是對構造方法的引用，語法是ClassName::new。例如：
```java=
Supplier<List<String>> listSupplier = new Supplier<List<String>>() {
    @Override
    public List<String> get() {
        return new ArrayList<>();
    }
};
// ------------------------------------
Supplier<List<String>> listSupplier = () -> new ArrayList<>();
// ------------------------------------
Supplier<List<String>> listSupplier = ArrayList::new;
List<String> myList = listSupplier.get();
```
ArrayList::new引用了ArrayList的構造方法。

4. 數組構造方法引用： 這是對數組構造方法的引用，語法是Type[]::new。例如：
```java=
Function<Integer, int[]> intArrayMaker = new Function<Integer, int[]>() {
    @Override
    public int[] apply(Integer size) {
        return new int[size];
    }
};
// ------------------------------------
Function<Integer, int[]> intArrayMaker = size -> new int[size];
// ------------------------------------
Function<Integer, int[]> intArrayMaker = int[]::new;
int[] myArray = intArrayMaker.apply(5);
```
int[]::new引用了一個int數組的構造方法。
|方法引用類型 | 舉例 | 等同的 Lambda|
| -------- | -------- | -------- |
|Static | Integer::parseInt | str -> Integer.parseInt(str)|
|Bound | Instant.now()::isAfter | Instant then = Instant.now(); t -> then.isAfter(t)|
|Unbound | String::toLowerCase | str -> str.toLowerCase()|
|Class Constructor | TreeMap<K,V>::new | () -> new TreeMap<K,V>|
|Array Constructor | int[]::new | len -> new int[len]|
## 44.優先使用標準的函數式接口
|接口 | 方法 | 示例|
| -------- | -------- | -------- |
|UnaryOperator`<`T`>`| T apply(T t) |String::toLowerCase|
|BinaryOperator`<`T`>`| T apply(T t1, T t2) |BigInteger::add|
|Predicate`<`T`>`| boolean test(T t) |Collection::isEmpty|
|Function`<`T,R`>`| R apply(T t) |Arrays::asList|
|Supplier`<`T`>`| T get()| Instant::now|
|Consumer`<`T`>`| void accept(T t) |System.out::println|
## 45.明智審慎地使用 Stream
優
1. 集合操作：流非常適合用於對集合或數組中的元素執行過濾、映射、篩選、排序等操作。
2. 並行處理：流提供了並行處理元素的能力，適合於多核處理器的優化。通過 parallelStream() 方法，你可以在流中並行處理數據，提高處理性能。
3. 函數式編程：流支持函數式編程的風格，可以用於實現函數式編程的原則，如映射、過濾、歸約等操作。
4. 簡化代碼：使用流可以減少樣板代碼，使代碼更簡潔和易讀。
5. 延遲執行：流通常采用延遲執行的方式，只有在需要結果時才會實際執行操作，這有助於提高效率。
6. 與 Lambda 表達式協同工作：流與 Lambda 表達式協同工作，可以更容易地傳遞行為和操作。
7. 數據篩選和轉換：流非常適合進行數據篩選、轉換和聚合操作，尤其是在數據處理方面。

劣
1. 簡單叠代：如果只需要對一個集合做簡單的叠代或循環操作，使用傳統的for循環可能更加簡單和有效。
2. 性能要求高：在某些性能要求高的場景，特別是對大型數據集的處理，流的性能可能不如手動叠代或傳統循環。
3. 數據操作不支持流：有時，操作的數據結構可能不支持流，或者使用流不容易實現。在這種情況下，選擇傳統的叠代方法可能更合適。
4. 維護困難：雖然流可以使代碼更簡潔，但過度使用流可能導致代碼難以維護，尤其是在多個連續操作嵌套時。
5. 需要早期退出或幹涉：某些情況下，你可能需要在處理數據時實現早期退出或對流中的元素進行幹涉，這時使用流可能不夠靈活。
6. 多線程需求：在需要進行多線程並行處理時，需要謹慎處理流的線程安全性，而有些情況下傳統的並發操作可能更為合適。
7. 可讀性降低：過多的連續流操作可能會降低代碼的可讀性，使代碼難以理解。在某些情況下，傳統的叠代方式可能更清晰。
## 46.優先考慮流中無副作用的函數
使用流處理數據時，應該盡量避免更改或修改流中的數據，而是使用無狀態的操作，這些操作不會修改原始數據，而是生成新的數據流。

常見 "流中無副作用的函數" 的示例：
* map()：映射操作創建一個新流，不修改原始流的元素。
* filter()：過濾操作創建一個新流，不修改原始流。
* sorted()：排序操作生成一個新流，原流的順序不受影響。
* distinct()：去重操作創建一個新流，原始流保持不變。
* collect()：收集操作將流中的元素聚合成一個集合，但不會改變原始流。

相反，一些可能引入副作用的操作包括：
* forEach()：遍歷元素執行操作，可能修改外部狀態。
* peek()：執行某些操作後，繼續傳遞元素，但不應用修改。
* reduce()：將元素聚合成單個值，但可能需要修改外部狀態。
* sum(), max(), min(), 等：雖然返回單個值，但通常會涉及狀態的累積。

Collectors是 Java 標準庫中提供的一個工具類，用於操作和處理流中的元素，並將它們收集到不同類型的數據結構中。Collectors 類包含了各種靜態工廠方法，用於創建常見的收集器，以便對流進行操作和處理。
下面是一些使用 Collectors 與流的常見操作示例：

1. 將流元素收集到 List 或 Set 中:
```java=
List<String> nameList = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

Set<Integer> ageSet = people.stream()
    .map(Person::getAge)
    .collect(Collectors.toSet());
```
2. 將流元素收集到 Map 中:
```java=
Map<String, Integer> nameToAgeMap = people.stream()
    .collect(Collectors.toMap(Person::getName, Person::getAge));

Map<Integer, String> ageToNameMap = people.stream()
    .collect(Collectors.toMap(Person::getAge, Person::getName));
```
3. 對流元素進行分組:
```java=
Map<Gender, List<Person>> peopleByGender = people.stream()
    .collect(Collectors.groupingBy(Person::getGender));
```
這個示例將人按照性別進行分組，創建了一個 Map，其中鍵是性別，值是屬於該性別的人的列表。

4. 對流元素進行聚合操作:
```java=
int totalAge = people.stream()
    .collect(Collectors.summingInt(Person::getAge));

double averageAge = people.stream()
    .collect(Collectors.averagingDouble(Person::getAge));
```
5. 連接流元素為字符串:
```java=
String names = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));
```
## 47.優先使用 Collection 而不是 Stream 
Stream 在處理元素時是惰性的，它只有在遇到終端操作時才會執行計算，而且一旦執行終端操作，該 Stream 就會被消耗掉，不可覆用。這可能導致一些問題：

1. 不便於重覆遍歷: 一旦你使用了 Stream 進行終端操作，你不能再次遍歷相同的元素集合，因為該 Stream 已經被消耗掉。
2. 可能引發異常: 如果返回的 Stream 在方法外部被處理，且方法內部對元素進行了修改，可能引發並發修改異常或非法狀態異常。
3. 性能損失: Stream 的處理是延遲的，這意味著在方法內部生成的 Stream 可能需要額外的內存和計算資源，這可能導致性能問題。

因此，為了提高代碼的可讀性和避免潛在的問題，通常更好的做法是將方法返回類型聲明為 Collection（如 List, Set, Map 等）而不是 Stream。這允許方法返回一個已經計算好的集合，而不是需要在方法外部再次執行計算的 Stream。但在某些情況下，如果返回的數據集很大，且僅需要部分元素，使用 Stream 可能是一個更好的選擇，因為它可以避免不必要的計算和內存開銷。
## 48.謹慎使用流並行
通常，並行性帶來的性能收益在 ArrayList 、 HashMap 、 HashSet 和 ConcurrentHashMap 實例、數組、 int 類型範圍和 long 類型的範圍的流上最好。這些數據結構的共同之處在於，它們都可以精確而廉價地分割成任意大小的子程序，這使得在並行線程之間劃分工作變得很容易。

而在某些情形，並行化一個流不僅會導致糟糕的性能，包括活性失敗(liveness failures)；它會導致不正確的結果和不可預知的行為 (安全故障)。

總之，甚至不要嘗試並行化流管道，除非你有充分的理由相信它將保持計算的正確性並提高其速度。不恰當地並行化流的代價可能是程序失敗或性能災難。如果您認為並行性是合理的，那麽請確保您的代碼在並行運行時保持正確。
## 49.檢查參數有效性
1. 檢查參數的非空性： 如果參數不能為null，應在方法的開頭檢查參數是否為null。如果參數為null，則可能拋出NullPointerException，或者返回適當的錯誤或異常。在 Java 7 中添加的 Objects.requireNonNull 方法靈活方便，因此沒有理由再手動執行空值檢查。 還可以指定自定義異常詳細消息。
2. 檢查數值參數的範圍： 對於接受數值參數的方法，檢查參數的取值範圍是否在有效範圍內。例如，如果方法要求一個正整數參數，可以檢查參數是否大於零。
3. 檢查集合參數： 對於接受集合或數組作為參數的方法，應確保集合不為null，並檢查集合的大小或數組的長度是否滿足要求。
4. 驗證字符串參數： 如果方法接受字符串參數，應檢查字符串是否符合特定的格式或長度要求。這可以通過正則表達式或其他字符串處理方法來完成。
5. 參數協同性檢查： 有時，多個參數之間需要相互協同工作，以滿足方法的預期行為。在這種情況下，需要檢查參數之間的協同性。
6. 參數類型檢查： 檢查參數的類型是否符合方法的預期類型。如果方法期望整數參數，應檢查參數是否為整數。
7. 文檔說明： 在方法的文檔注釋中，應明確說明參數的預期條件和有效性要求，以便其他開發人員了解如何正確使用方法。如：@param、@return、@throws。
8. 自定義異常： 如果參數無效，可以拋出自定義異常，以提供更多的上下文信息和錯誤消息，幫助開發人員更容易診斷問題。
9. 斷言： 使用斷言（assertions）來驗證參數的有效性。在開發和測試時，可以啟用斷言，以確保參數有效，而在生產環境中禁用它們。
## 50.必要時進行防禦性拷貝
防禦性拷貝（Defensive Copy）是一種編程技巧，用於確保對象的封裝和數據安全性。它的基本思想是在操作可能被外部修改的對象之前，創建對象的一份拷貝，以防止外部代碼對原始對象的未經授權修改。防禦性拷貝通常用於處理可變對象，以防止不受信任的代碼修改對象的內部狀態。
1. 時機： 防禦性拷貝通常在接受參數、返回對象或將對象傳遞給不受信任的代碼之前進行。這確保了方法的調用者不能修改原始對象。
2. 性能影響： 防禦性拷貝可能對性能產生一定影響，因為它涉及覆制對象的數據。因此，在需要性能的關鍵代碼路徑上，需要權衡性能和安全性。
3. 深拷貝和淺拷貝： 防禦性拷貝可以是淺拷貝（僅拷貝對象本身）或深拷貝（同時拷貝對象內部的嵌套對象）。選擇何種類型的拷貝取決於對象的覆雜性和需求。
```java=
import java.util.Date;

public class DateContainer {
    private Date date;

    public DateContainer(Date date) {
        // 進行防禦性拷貝，創建一個新的Date對象，以防止外部代碼修改原始對象
        this.date = new Date(date.getTime());
    }

    public Date getDate() {
        // 返回一個防禦性拷貝，而不是原始Date對象
        return new Date(date.getTime());
    }
}

public class Main {
    public static void main(String[] args) {
        Date originalDate = new Date();
        DateContainer container = new DateContainer(originalDate);

        // 外部代碼嘗試修改日期對象
        Date modifiedDate = container.getDate();
        modifiedDate.setTime(0); // 這里不會影響原始對象，因為已經進行了防禦性拷貝

        System.out.println("Original Date: " + originalDate);
        System.out.println("Modified Date: " + modifiedDate);
    }
}
```
## 51.仔細設計方法簽名
1. 方法及參數名稱：應該具有描述性，清晰地表達方法的功能及參數的用途。方法名通常是動詞或動賓短語，以清晰傳達操作的目的。
2. 參數類型：參數的類型應該選擇最合適的數據類型，並且優先選擇接口而不是類。如果方法接受多個參數，考慮拆分方法為多個方法、創建輔助類來保存參數組、使用 Builder 模式。
4. 返回類型：方法的返回類型應該清晰地反映方法的結果。如果方法沒有返回值，使用void。如果方法返回一個值，確保返回類型的名稱和值具有明確的含義。
5. 異常：方法簽名中應明確指定方法可能拋出的異常。這有助於客戶端代碼正確處理異常情況。避免在方法簽名中使用過於寬泛的異常，應該盡量使用特定的異常類型。
6. 重載：如果在同一個類中存在多個具有相同名稱的方法，確保方法簽名在參數數量或參數類型上有明顯的區別，以便編譯器能夠正確識別調用的方法。
7. 文檔注釋：為方法編寫清晰的文檔注釋，描述方法的功能、參數和返回值，以便其他開發人員能夠理解和正確使用方法。
8. 與布爾型參數相比，優先使用兩個元素枚舉類型
```java=
public void process(boolean isEnabled) {
    if (isEnabled) {
        // 執行啟用功能的邏輯
    } else {
        // 執行禁用功能的邏輯
    }
}
// ------------------- 以下更佳
public enum Status {
    ENABLED, DISABLED
}
public void process(Status status) {
    if (status == Status.ENABLED) {
        // 執行啟用功能的邏輯
    } else {
        // 執行禁用功能的邏輯
    }
}
```
## 52.明智審慎地使用重載
1. 避免過多的重載：不要濫用方法重載，避免創建過多的相似的方法。
2. 使用參數列表明確：在進行方法重載時，要確保參數列表不僅在數量上有區別，而且在類型上有明顯區別。一個安全和保守的策略是永遠不要導出兩個具有相同參數數量的重載。
3. 考慮使用命名約定：有時候，可以通過使用不同的方法名稱來代替重載，這樣可以更清晰地表達方法的功能。例如，add(int) 和 add(String) 比 add(int) 和 add(String, int) 更容易理解。
4. 謹慎使用自動裝箱：重載方法時，要小心使用自動裝箱（autoboxing）和自動拆箱（unboxing），因為它們可能引發不明顯的問題。
5. 明智選擇默認參數值：如果必須使用重載來支持不同的參數組合，要確保在參數的默認值上做出明智的選擇，以避免歧義。
## 53.明智審慎地使用可變參數
運行時會將傳入的參數打包成一個數組。對於大量數據或頻繁調用的情況，這可能會影響性能。
如果你從經驗上確定負擔不起這個成本，但是還需要可變參數的靈活性，那麽有一種模式可以讓你魚與熊掌兼得。假設你已確定95% 的調用是三個或更少的參數的方法，那麽聲明該方法的五個重載。每個重載方法包含 0 到 3 個普通參數，當參數數量超過 3 個時，使用一個可變參數方法:
```java=
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
如果您的方法參數包含一組枚舉值，並且您希望提高性能和代碼的清晰度，可以使用 EnumSet 
```java=
import java.util.EnumSet;

enum Color {
    RED, GREEN, BLUE
}

public class EnumSetExample {
    public static void printColors(EnumSet<Color> colors) {
        for (Color color : colors) {
            System.out.println(color);
        }
    }

    public static void main(String[] args) {
        EnumSet<Color> selectedColors = EnumSet.of(Color.RED, Color.GREEN);
        printColors(selectedColors);
    }
}
```
## 54.返回空的數組或集合,不要返回 null
沒有理由對沒有奶酪 (Cheese) 可供購買的情況做額外的代碼處理。
```java=
private final List<Cheese> cheesesInStock = ...;
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
// ---------------------------
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
## 55.明智審慎地返回 Optional
不包含任何內容的 Optional 被稱為空(empty)。非空的包含值稱的 Optional 被稱為存在(present)。Optional 的本質上是一個不可變的集合，最多可以容納一個元素。

1. 避免過度使用 Optional： 只在真正需要返回缺失值時才使用 Optional，對於性能關鍵的方法，最好返回 null 或拋出異常。
2. 並不是所有的返回類型都能從 Optional 的處理中獲益。容器類型，包括集合、映射、Stream、數組和Optional，不應該封裝在 Optional 中。與其返回一個空的 Optional`<`List`<`T`>>` ，不還如返回一個空的 List`<`T`>`
3. 適當處理映射操作： 在使用 Optional 的 map 和 flatMap 操作時，確保不會引入過多的嵌套層級。在這種情況下，可能需要考慮將邏輯拆分成多個步驟。
4. 謹慎使用 orElse 方法： Optional 提供了 orElse 方法，用於在值不存在時提供默認值，但它會導致潛在的性能開銷。如果計算默認值需要昂貴的操作，可以使用 orElseGet 方法來傳遞一個 Supplier，以便只有在需要時才執行。
5. 不要在構造函數中接受 Optional 參數： 在類的構造函數中，不應該接受 Optional 作為參數。相反，應該接受普通的值作為參數，然後在構造函數內部使用 Optional 來表示可能的 null 值。

Optional.stream() 方法是 Java 9 引入的新功能，用於將一個 Optional 轉換為一個 Stream 對象。
```java=
Optional<String> optionalValue = Optional.of("Hello, Optional!");

// 將 Optional 轉換為流，可能包含一個元素或沒有元素
Stream<String> stream = optionalValue.stream();

// 使用流操作處理 Optional 中的值（在這種情況下，將字符串轉換為大寫）
stream.map(String::toUpperCase).forEach(System.out::println);

// 如果 Optional 為空，上述操作什麽都不會執行
Optional<String> emptyOptional = Optional.empty();
Stream<String> emptyStream = emptyOptional.stream();
emptyStream.forEach(System.out::println); // 什麽都不會輸出
```
## 56. 為所有已公開的 API 元素編寫文檔注釋
```java=
/**
* <p>This method is <i>not</i> guaranteed to run in constant
* time. 
*
* @param index index of element to return
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
* ({@code index < 0 || index >= this.size()})
*/
E get(int index){...};

/**
* This method complies with the {@index IEEE 754} standard.
* @implSpec
* This implementation returns {@code this.size() == 0}.
*
* @return true if {@literal |r| < 1}.
* @see {@link AnotherClass}
* @author Your Name
* @version 1.0
*/
public boolean isEmpty() { ... }
```
Javadoc 實用工具將文檔注釋轉換為 HTML，文檔注釋中的任意 HTML 元素最終都會生成 HTML 文檔。 有時候，程序員甚至會在他們的文檔注釋中嵌入 HTML 表格，盡管這種情況很少見。
* {@code}和{@literal}：使代碼片段以代碼字體形式呈現，並且它抑制了代碼片段中 HTML 標記和嵌套 Javadoc 標記的處理。
* {@index}：對文檔注釋中出現的術語進行索引。
* {@implSpec}：提供關於類的實現規範的描述。
* {@inheritDoc}：從超類繼承部分文檔注釋。
* @see：引用其他相關類、方法或文檔。
* {@link}：引用
## 57. 最小化局部變量的作用域
1. 將變量聲明推遲到首次使用： 不要在方法的開頭就聲明所有的局部變量，而是在需要使用它們的地方進行聲明。這可以減少變量的生存周期，使代碼更易於理解。
2. 在盡可能小的範圍內聲明變量： 變量的作用域應該盡可能小，以便在不再需要時及時釋放資源。
3. 使用大括號限定變量的作用域： 在循環、條件語句等塊結構中，使用大括號明確指定變量的作用域，以避免變量泄漏。
## 58. for-each 循環優於傳統 for 循環
for-each 循環在清晰度，靈活性和錯誤預防方面提供了超越傳統 for 循環的令人注目的優勢，而且沒有性能損失。
但是,有三種常見的情況是你不能分別使用 for-each 循環的:
1. 有損過濾(Destructive filtering)：如果需要遍歷集合，並刪除指定選元素，則需要使用顯式叠代器，以便可以調用其 remove 方法。 通常可以使用在 Java 8 中添加的 Collection 類中的 removeIf 方法，來避免顯式遍歷。
```java=
public static void main(String[] args) {
    List<Integer> numbers = new ArrayList<>();
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);

    // 創建 Predicate 來定義條件，這里刪除偶數
    numbers.removeIf(n -> n % 2 == 0);

    System.out.println(numbers); // 輸出 [1, 3]
}
```
2. 轉換：如果需要遍歷一個列表或數組並替換其元素的部分或全部值，那麽需要列表叠代器或數組索引來替換元素的值。
3. 並行叠代：如果需要並行地遍歷多個集合，那麽需要顯式地控制叠代器或索引變量，以便所有叠代器或索引變量都可以同步進行。
## 59. 了解並使用庫
如果你需要做一些看起來相當常見的事情，那麽庫中可能已經有一個工具可以做你想做的事情。如果有，使用它;如果你不知道，檢查一下。
## 60. 若需要精確答案就應避免使用 float 和 double 類型
如果希望系統來處理十進制小數點，並且不介意不使用基本類型帶來的不便和成本，請使用 BigDecimal。使用 BigDecimal 的另一個好處是，當執行需要舍入的操作時，可以從八種舍入模式中進行選擇。如果性能是最重要的，那麽你不介意自己處理十進制小數點，而且數值不是太大，可以使用 int 或 long。如果數值不超過 9 位小數，可以使用 int；如果不超過 18 位，可以使用 long。如果數量可能超過 18 位，則使用BigDecimal。
## 61. 基本數據類型優於包裝類
只要有選擇，就應該優先使用基本類型，而不是包裝類型。基本類型更簡單、更快。
除非需要表示可空值、需要與集合框架（如List、Map）或其他使用對象而不是基本數據類型的API協作時、需要使用包裝類提供的附加功能，如 Integer 類的方法時。
## 62. 當使用其他類型更合適時應避免使用字符串
使用適當的數據類型和對象可以提高類型安全性、可讀性和維護性。
多線程環境中使用字符串全局標識符時可能出現的競態條件和線程安全性問題，可透過java.lang.ThreadLocal 解決
```java=
public class GlobalResource {
    private static Map<String, String> data = new HashMap<>();

    public static void updateData(String key, String value) {
        data.put(key, value);
    }

    public static String getData(String key) {
        return data.get(key);
    }

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                String threadName = Thread.currentThread().getName();
                String newValue = "Value from " + threadName;
                updateData("key", newValue);
                String retrievedValue = getData("key");
                System.out.println(threadName + " retrieved: " + retrievedValue);
            }).start();
        }
    }
}
```
多個線程同時訪問 updateData 和 getData 方法，它們都使用相同的字符串標識符 "key"。由於沒有同步機制，這可能導致並發問題，例如競爭條件或不一致的數據。
```java=
public class GlobalResource {
    private static ThreadLocal<Map<String, String>> threadLocalData = ThreadLocal.withInitial(HashMap::new);

    public static void updateData(String key, String value) {
        Map<String, String> threadData = threadLocalData.get();
        threadData.put(key, value);
    }

    public static String getData(String key) {
        Map<String, String> threadData = threadLocalData.get();
        return threadData.get(key);
    }

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                String threadName = Thread.currentThread().getName();
                String newValue = "Value from " + threadName;
                updateData("key", newValue);
                String retrievedValue = getData("key");
                System.out.println(threadName + " retrieved: " + retrievedValue);
            }).start();
        }
    }
}
```
## 63. 當心字符串連接引起的性能問題
除非性能無關緊要，否則使用 StringBuilder 的append 方法。或者，使用字符數組，再或者一次只處理一個字符串，而不是組合它們。
## 64. 通過接口引用對象
使用接口引用對象，程序將更加靈活和流行。如果沒有合適的接口，就使用類層次結構中提供所需功能的最底層的類。
## 65. 接口優於反射
如果編寫的程序必須在編譯時處理未知的類，則應該盡可能只使用反射實例化對象，並使用在編譯時已知的接口或超類訪問對象。

核心反射機制 java.lang.reflect 提供對任意類的編程訪問。給定一個 Class 對象，可以獲得Constructor、Method 和 Field 實例，分別代表了該 Class 實例所表示的類的構造器、方法和字段。這些對象提供對類的成員名、字段類型、方法簽名等的編程訪問。
```java=
// 獲取類的構造函數
Constructor<MyClass> constructor = MyClass.class.getConstructor(); // 無參數構造函數

// 創建對象實例
MyClass instance = constructor.newInstance();

// 獲取類的方法
Method method = MyClass.class.getMethod("someMethod", int.class, String.class);

// 調用方法
Object result = method.invoke(instance, 42, "Hello");

// 獲取類的字段
Field field = MyClass.class.getDeclaredField("someField");

// 使字段可訪問（如果是私有字段）
field.setAccessible(true);

// 獲取字段的值
Object value = field.get(instance);

// 設置字段的值
field.set(instance, "New Value");
```
反射的使用會失去了編譯時類型檢查的所有好處，包括異常檢查。執行反射訪問所需的代碼既笨拙又冗長，且性能會降低。
## 66. 明智審慎地本地方法
本地方法（Native Methods）是 Java 中的一項高級功能，允許您使用其他編程語言（通常是 C 或 C++）編寫方法，然後從 Java 代碼中調用這些方法。
從歷史上看，本地方法主要有三種用途。它們提供對特定於平台的設施(如注冊中心)的訪問；合法但是很少有必要。它們提供對現有本地代碼庫的訪問，包括提供對遺留數據訪問。最後，本地方法可以通過本地語言編寫應用程序中注重性能的部分，以提高性能；現今JVM性能已足夠故不建議。
## 67. 明智審慎地進行優化
不要努力寫快的程序，要努力寫好程序；速度自然會提高。但是在設計系統時一定要考慮性能，特別是在設計API、線路層協議和持久數據格式時。當你完成了系統的構建之後，請度量它的性能。如果足夠快，就完成了。如果沒有，利用分析器找到問題的根源，並對系統的相關部分進行優化。第一步是檢查算法的選擇：再多的底層優化也不能彌補算法選擇的不足。
## 68. 遵守被廣泛認可的命名約定
|Identifier Type | Example |
| -------- | -------- |
|Package or module| org.junit.jupiter.api , com.google.common.collect|
|Class or Interface| Stream, FutureTask, LinkedHashMap,HttpClient|
|Method or Field| remove, groupingBy, getCrc|
|Constant Field |MIN_VALUE, NEGATIVE_INFINITY|
|Local Variable| i, denom, houseNum|
|Type Parameter| T, E, K, V, X, R, U, V, T1, T2|
* 可實例化的類，包括枚舉類型，通常使用一個或多個名詞短語來命名，例如 Thread、PriorityQueue 或 ChessPiece。
* 不可實例化的實用程序類通常使用覆數名詞來命名，例如 collector 或 Collections。
* 接口的名稱類似於類，例如集合或比較器，或者以 able 或ible 結尾的形容詞，例如 Runnable、Iterable 或 Accessible。
* 因為注解類型有很多的用途，所以沒有哪部分占主導地位。名詞、動詞、介詞和形容詞都很常見，例如，BindingAnnotation、Inject、ImplementedBy 或 Singleton。
* 方法通常用動詞或動詞短語(包括對象)命名，例如，append 或 drawImage。返回布爾值的方法的名稱通常以單詞 is 或 has(通常很少用)開頭，後面跟一個名詞、一個名詞短語，或者任何用作形容詞的單詞或短語，例如 isDigit、isProbablePrime、isEmpty、isEnabled 或 hasSiblings。
* 轉換對象類型(返回不同類型的獨立對象)的實例方法通常稱為 toType，例如toString 或 toArray。返回與接收對象類型不同的視圖的方法通常稱為 asType，例如 asList。返回與調用它們的對象具有相同值的基本類型的方法通常稱為類型值，例如 intValue。靜態工廠的常見名稱包括 from、of、valueOf、instance、getInstance、newInstance、getType 和 newType。
## 69. 只針對異常的情況下才使用異常
1. 只捕獲你知道如何處理的異常：捕獲異常時，應該確保你了解如何處理該異常，而不是簡單地將其吞下（即忽略它）。
2. 不要將異常用作正常的控制流：異常不應該用於實現正常的程序控制流。
3. 異常處理應該具體而不是泛化：異常處理代碼應該盡可能具體，而不是捕獲通用的 Exception 類型，因為這樣可以更精確地了解出現了什麽問題。
4. 避免在循環中使用異常：在循環中拋出和捕獲異常可能會導致性能問題。在循環內部，最好使用條件語句來處理可能的情況，而不是依賴異常。
## 70. 對可恢覆的情況使用受檢異常，對編程錯誤使用運行時異常
Java 程序設計語言提供了三種 throwable：受檢異常(checked exceptions)、運行時異常(runtime exceptions)和錯誤(errors)。
* 如果期望調用者能夠合理的恢覆程序運行，對於這種情況就應該使用受檢異常：通過拋出受檢異常,強迫調用者在一個 catch 子句中處理該異常，或者把它傳播出去
* 用運行時異常來表明編程錯誤：大多數運行時異常都表示前提違例(precondition violations)。所謂前提違例是指 API 的客戶沒有遵守 API 規範建立的約定。例如，數組訪問的預定指明了數組的下標值必須在 0 和數組長度-1 之間。ArrayIndexOutOfBoundsException 表明違反了這個前提。
## 71. 避免不必要的使用受檢異常
如果一個方法可以預見並處理某些異常情況，並且希望迫使調用者處理這些情況，那麽可以考慮返回Optional值而不是拋出受檢異常。另一方面，如果異常是不可恢覆的，且沒有明顯的合適返回值，那麽可以考慮將其定義為未受檢異常。
```java=
import java.util.Optional;

public Optional<User> findUserById(int userId) {
    // 嘗試從數據庫中查找用戶
    User user = database.findUserById(userId);
    
    // 如果找到用戶，返回一個包含用戶的 Optional 對象
    return Optional.ofNullable(user);
}

// 調用方法
Optional<User> userOptional = findUserById(123);
if (userOptional.isPresent()) {
    User user = userOptional.get();
    // 處理用戶對象
} else {
    // 處理用戶不存在的情況
}
//--------------------
public User findUserById(int userId) throws UserNotFoundException {
    // 嘗試從數據庫中查找用戶
    User user = database.findUserById(userId);
    
    // 如果找到用戶，返回用戶對象
    if (user != null) {
        return user;
    } else {
        throw new UserNotFoundException("User not found");
    }
}

// 調用方法
try {
    User user = findUserById(123);
    // 處理用戶對象
} catch (UserNotFoundException e) {
    // 處理用戶不存在的情況
}
```
## 72. 優先使用標準的異常
適當的挑選庫中已存在的異常。例如：
|異常 | 使用場合 |
| -------- | -------- |
|IllegalArgumentException |非 null 的參數值不正確|
|IllegalStateException |不適合方法調用的對象狀態|
|NullPointerException |在禁止使用 null 的情況下參數值為 null|
|IndexOutOfBoundsExecption |下標參數值越界
|ConcurrentModificationException |在禁止並發修改的情況下,檢測到對象的並發修改|
|UnsupportedOperationException| 對象不支持用戶請求的方法|
## 73. 拋出與抽象對應的異常
更高層的實現應該捕獲低層的異常，同時拋出可以按照高層抽象進行解釋的異常。這種做法稱為異常轉譯 (exception translation)
如果不能阻止或者處理來自更低層的異常，一般的做法是使用異常轉譯，只有在低層方法的規範碰巧可以保證“它所拋出的所有異常對於更高層也是合適的”情況下，才可以將異常從低層傳播到高層。
```java=
public void readFile(String fileName) throws CustomFileException {
    try {
        // 讀取文件的代碼
    } catch (IOException e) {
        // 捕獲低級別異常並將其轉譯為高級別異常
        // CustomFileException is a IOException
        throw new CustomFileException("Error reading file: " + fileName, e);
    }
}
```
## 74. 每個方法拋出的異常都需要創建文檔
使用 Javadoc 的 @throws 標簽記錄下一個方法可能拋出的每個未受檢異常，但是不要使用 throws 關鍵字將未受檢的異常包含在方法的聲明中。
## 75. 在細節消息中包含失敗一捕獲信息
為了捕獲失敗，異常的細節信息應該包含“對該異常有貢獻”的所有參數和字段的值。
## 76. 保持失敗原子性
一般而言，失敗的方法調用應該使對象保持在被調用之前的狀態。 具有這種屬性的方法被稱為具有失敗原子性 (failure atomic) 。
幾種方式保持原子性：
* 調整計算處理過程的順序，使得任何可能會失敗的計算部分都在對象狀態被修改之前發生。
* 在對象的一份臨時拷貝上執行操作，當操作完成之後再用臨時拷貝中的結果代替對象的內容。
* 編寫一段恢覆代碼 (recoverycode )，由它來攔截操作過程中發生的失敗，以及便對象回滾到操作開始之前的狀態上。

作為方法規範的一部分，它產生的任何異常都應該讓對象保持在調用該方法之前的狀態。如果違反這條規則， API 文檔就應該清楚地指明對象將會處於什麽樣的狀態。遺憾的是，大量現有的 API 文檔都未能做到這一點。
## 77. 不要忽略異常
如果不得已選擇忽略異常，catch 塊中應該包含一條注釋，說明為什麽可以這麽做，並且變量應該命名為ignored。
## 78. 同步訪問共享的可變數據
當多個線程共享可變數據的時候，每個讀或者寫數據的線程都必須執行同步。
幾個同步措施：
* 使用synchronized關鍵字：
```java=
public class SynchronizedExample {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }
}
```
在上面的示例中，increment方法使用synchronized關鍵字，確保只有一個線程可以同時執行該方法。
* 使用ReentrantLock鎖：
```java=
import java.util.concurrent.locks.ReentrantLock;

public class LockExample {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```
這里使用ReentrantLock鎖來實現同步。在increment方法中，首先調用lock.lock()獲取鎖，然後執行增加操作，最後在finally塊中釋放鎖。
* 使用ConcurrentHashMap：
```java=
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    private Map<String, Integer> map = new ConcurrentHashMap<>();

    public void putValue(String key, int value) {
        map.put(key, value);
    }
}
```
ConcurrentHashMap是一個線程安全的映射類，可以在多線程環境中安全地進行讀寫操作。
* 使用AtomicInteger：
```java=
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntegerExample {
    private AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }
}
```
AtomicInteger是一個原子整數類，它可以在不需要顯式鎖的情況下執行原子操作。

volatile 是 Java 中的一個關鍵字，用於修飾變量。它主要用於保證變量對於多個線程的可見性。使用 volatile 修飾的變量在多線程環境下，當一個線程修改了這個變量的值，其他線程能夠立即看到修改後的值。volatile 僅保證變量的可見性，不提供原子性。
## 79. 避免過度同步
為了避免死鎖和數據破壞，千萬不要從同步區字段內部調用外來方法。更通俗地講，要盡量將同步區字段內部的工作量限制到最少。如果你必須要執行某個很耗時的動作，則應該設法把這個動作移到同步區域的外面。
## 80. executor 、task 和 stream 優先於線程
Executor（執行器）：
* Executor 是一個框架，用於管理線程的生命周期，以及在應用程序中執行異步任務。它的主要接口是 Executor、ExecutorService 和 ScheduledExecutorService。
* Executor 封裝了線程的創建、管理、調度和銷毀等操作，使你無需手動管理線程的生命周期。一般情況下，使用 Executor 來執行任務，而不是直接創建和啟動線程。根據實際需求，你可以選擇不同類型的 Executor，如 newFixedThreadPool()、newCachedThreadPool() 等，或者根據需要自定義 Executor。
* ExecutorService 可以提交帶返回值的任務（通過 submit() 方法），並能夠處理任務執行過程中的異常。你可以通過 Future 對象獲取任務執行的返回值，或處理可能拋出的異常。Executor 接口通常只能執行 Runnable 任務，不具備返回值和異常處理的功能。
* ScheduledExecutorService 具備任務調度的功能，可以按照一定的時間間隔或在未來的某個時間點執行任務，並可以使用 scheduleAtFixedRate() 和 scheduleWithFixedDelay() 等方法來執行重覆性任務。
* ExecutorService 和 ScheduledExecutorService 提供了方法來關閉線程池，以釋放資源。你可以調用 shutdown() 或 shutdownNow() 方法來關閉線程池，確保不再接受新的任務，並等待已提交的任務執行完成。

Task（任務）：
* 任務代表要在多線程環境中執行的工作單元。通常，你可以創建實現 Runnable 或 Callable 接口的任務，並將它們提交給 Thread 或是 ExecutorService 來執行。如果你需要任務返回結果，你可以使用 Callable 接口，它允許任務返回一個值。
* Thread： Thread 是 Java 的基本線程構建塊。通過創建 Thread 對象，你可以直接控制線程的生命周期和行為。需要手動處理線程的生命周期、同步和資源管理等方面的細節。但這也可能會導致代碼更覆雜和容易出錯。
* ExecutorService： ExecutorService 是 Java 並發框架提供的一種更高級的線程管理方式。它封裝了線程的創建、調度和執行，提供了更高級的線程管理功能。你可以通過提交任務給 ExecutorService 來運行它們，而不必關心線程的創建和資源管理。通常情況下，推薦使用 ExecutorService 來管理線程，因為它更安全、更高效。
```java=
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class RunnableExample {
    public static void main(String[] args) {
        // Thread + Runnable
        Runnable task1 = new MyRunnable();
        Thread thread = new Thread(task1);
        thread.start();
        
        // ExecutorService + Runnable
        ExecutorService executor = Executors.newFixedThreadPool(2);

        Runnable task2 = new MyRunnable();
        Runnable task3 = new MyRunnable();

        executor.execute(task2);
        executor.execute(task3);

        executor.shutdown(); // Shutdown the executor when done
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println("Count: " + i);
            try {
                Thread.sleep(1000); // Sleep for 1 second
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
```java=
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class CallableExample {
    public static void main(String[] args) {
        // Thread + Callable
        Callable<Integer> callable1 = new MyCallable();

        FutureTask<Integer> futureTask = new FutureTask<>(callable1);
        Thread thread = new Thread(futureTask);

        thread.start();

        try {
            int result = futureTask.get();
            System.out.println("Thread Result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        
        // ExecutorService + Callable
        Callable<Integer> callable2 = new MyCallable();

        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<Integer> future = executor.submit(callable2);

        try {
            int result = future.get();
            System.out.println("ExecutorService Result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        executor.shutdown();
    }
}

class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 1; i <= 5; i++) {
            sum += i;
            try {
                Thread.sleep(1000); // Sleep for 1 second
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return sum;
    }
}
```
* Stream（流）：流的操作是無狀態的，因此在並行處理時更容易避免共享狀態和數據競爭，減少了並發錯誤的風險。且Stream 內部可以自動進行並行處理，這意味著你可以輕松地並行處理數據集合，而不必擔心線程管理的細節。
```java=
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.stream.Collectors;

public class ThreadHandlingWithStream {
    public static void main(String[] args) {
        // 創建一組任務
        List<CompletableFuture<Integer>> tasks = List.of(
                CompletableFuture.supplyAsync(() -> compute(1)),
                CompletableFuture.supplyAsync(() -> compute(2)),
                CompletableFuture.supplyAsync(() -> compute(3))
        );

        // 並行處理這些任務
        List<Integer> results = tasks.stream()
                .map(CompletableFuture::join) // 等待任務完成
                .collect(Collectors.toList());

        // 對結果進行聚合
        int sum = results.stream().mapToInt(Integer::intValue).sum();

        System.out.println("Sum of results: " + sum);
    }

    public static int compute(int value) {
        // 模擬一個耗時的任務
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return value * 2;
    }
}
```
以上範例，CompletableFuture 是 Java 8 引入的用於異步編程的類，它提供了一種簡單的方式來管理異步任務的執行和結果獲取。supplyAsync 是 CompletableFuture 類的一個靜態方法，用於創建一個異步執行的CompletableFuture對象，並在該對象上執行一個任務。
具體來說：
* CompletableFuture：這是一個 Java 類，用於表示一個可能異步執行的任務的結果。你可以將它看作一個容器，用於存放任務的執行結果。
* supplyAsync：這是 CompletableFuture 類的一個靜態工廠方法。它接受一個 Supplier 函數，該函數描述了異步執行的任務，並返回一個 CompletableFuture 對象。supplyAsync 將任務提交給線程池異步執行。

例如，CompletableFuture.supplyAsync(() -> compute(1)) 創建了一個 CompletableFuture 對象，其中 compute(1) 是一個模擬的異步任務，這個任務將在另一個線程中執行。CompletableFuture 對象將在任務完成後保存任務的結果，或在結果可用時等待獲取。

在示例中的代碼中，supplyAsync 用於創建異步任務，然後使用 join 方法等待任務的完成，將所有任務的結果收集到一個列表中，最後聚合這些結果。

這種異步編程模型使你能夠更輕松地編寫多線程或並發代碼，而不必手動管理線程的創建和同步。
## 81. 相比 wait 和 notify 優先使用並發工具
Java 中提供了一些並發工具，用於管理多線程並發執行的任務和協同工作。這些並發工具相比傳統的 wait 和 notify 更強大和易於使用，以下幾個常見例子：
* CountDownLatch：用於等待一組線程執行完畢。你可以設置一個初始計數值，然後多個線程可以減小這個計數值，當計數值為零時，等待的線程將被喚醒。計數值只能減小，不能增加。一旦計數值為零，就不能再重置或重新使用 CountDownLatch。
```java=
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) {
        int threadCount = 3; // 模擬三個線程

        CountDownLatch latch = new CountDownLatch(threadCount);

        // 計數值3，超過3條的線程不會被等待、小於3條則會一直等待
        for (int i = 0; i < threadCount; i++) {
            new Thread(new Worker(latch)).start();
        }

        try {
            latch.await(); // 等待傳入的數量的線程執行完畢
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("All threads have finished.");
    }
}

class Worker implements Runnable {
    private final CountDownLatch latch;

    public Worker(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            // 模擬工作
            Thread.sleep(1000);
            System.out.println("Thread finished its work.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            latch.countDown(); // 減小計數值
        }
    }
}
```
* CyclicBarrier：用於多個線程相互等待，然後在屏障處同時開始執行。通常用於分解覆雜任務為多個子任務並行執行，然後在子任務完成後合並結果。計數值是可以重置的，可以多次使用。當線程到達屏障點時，會等待其他線程，然後一起繼續執行。
```java=
import java.util.concurrent.CyclicBarrier;

public class ReusableCyclicBarrierExample {
    public static void main(String[] args) {
        final int numThreads = 3;
        CyclicBarrier barrier = new CyclicBarrier(numThreads);

        for (int i = 0; i < numThreads; i++) {
            final int threadNum = i;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + threadNum + " is ready");
                    barrier.await();
                    System.out.println("Thread " + threadNum + " continues");

                    System.out.println("Thread " + threadNum + " is ready for the next phase");
                    barrier.await();
                    System.out.println("Thread " + threadNum + " continues again");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```
在大多數情況下，不需要顯式調用 reset() 來重置 CyclicBarrier。它會在達到計數後自動重用，而且 reset() 方法主要用於在一些特殊情況下，手動將 CyclicBarrier 重置為初始狀態。
* Semaphore：用於控制同時訪問某一資源的線程數量。它允許設置資源的數量，然後多個線程可以嘗試獲取資源的許可。當許可用完時，其他線程必須等待。具有一定的彈性，可以通過釋放許可來增加資源的可用性，也可以通過獲取許可來限制資源的使用。
```java=
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
    public static void main(String[] args) {
        // 創建一個Semaphore並設置允許的並發線程數
        Semaphore semaphore = new Semaphore(3); // 允許同時有3個線程訪問共享資源

        // 創建多個線程來模擬並發訪問
        for (int i = 1; i <= 5; i++) {
            Thread thread = new Thread(new Worker(semaphore, i));
            thread.start();
        }
    }
}

class Worker implements Runnable {
    private final Semaphore semaphore;
    private final int id;

    public Worker(Semaphore semaphore, int id) {
        this.semaphore = semaphore;
        this.id = id;
    }

    @Override
    public void run() {
        try {
            // 嘗試獲取一個許可
            System.out.println("Worker " + id + " is trying to acquire a permit.");
            semaphore.acquire();
            System.out.println("Worker " + id + " has acquired a permit.");

            // 模擬工作
            Thread.sleep(10000);

            // 釋放許可
            System.out.println("Worker " + id + " is releasing the permit.");
            semaphore.release();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```
## 82. 文檔應包含線程安全屬性
文檔中線程安全屬性的說明應該包括以下信息：
1. 線程安全級別： 描述代碼的線程安全級別，如線程安全、有條件線程安全、非線程安全等。這有助於其他開發者理解在何種情況下可以安全地使用你的代碼。
2. 同步策略： 如果你的代碼采用了某種同步策略，例如使用鎖或其他並發工具來保證線程安全，應該說明這些策略的具體實現和原理。
3. 線程安全限制： 說明你的代碼中是否存在特定的線程安全限制。例如，是否有方法或操作需要外部同步，或者是否有條件約束需要滿足。
4. 並發性能： 提供關於代碼的並發性能的信息，例如在多線程環境下是否能獲得良好的性能。
5. 使用建議： 提供關於如何正確使用你的代碼以確保線程安全的建議，例如是否需要外部同步。
6. 非線程安全操作： 如果你的代碼包含非線程安全的操作或方法，應該明確標明這一點，並提供建議或警告。
```java=
/**
 * 一個線程安全的隊列，使用內部鎖來保證線程安全。
 *
 * <p>該隊列是線程安全的，多個線程可以同時進行入隊和出隊操作。
 * 請注意，雖然隊列本身是線程安全的，但並不保證對隊列中元素的操作是線程安全的。
 *
 * <p>此類采用了內部鎖來同步訪問隊列，因此具有較好的並發性能。
 *
 * <p>使用建議：
 * <ul>
 *   <li>入隊操作：調用 {@link #enqueue(Object)} 方法
 *   <li>出隊操作：調用 {@link #dequeue()} 方法
 *   <li>注意：對隊列元素的操作應該在調用入隊和出隊方法前自行同步。
 * </ul>
 */
public class ThreadSafeQueue<T> {
    // 實現細節省略
}
```
## 83. 明智審慎的使用延遲初始化
與大多數優化一樣，延遲初始化的最佳建議是「除非需要，否則不要這樣做」。
延遲初始化是一把雙刃劍。它降低了初始化類或創建實例的成本，代價是增加了訪問延遲初始化字段的成本。根據這些字段中最終需要初始化的部分、初始化它們的開銷以及初始化後訪問每個字段的頻率，延遲初始化實際上會損害性能(就像許多「優化」一樣)。
延遲初始化也有它的用途。如果一個字段只在類的一小部分實例上訪問，並且初始化該字段的代價很高，那麽延遲初始化可能是值得的。唯一確定的方法是以使用和不使用延遲初始化的效果對比來度量類的性能。
如果多個線程可能同時訪問需要延遲初始化的對象，確保延遲初始化的機制是線程安全的。常見的線程安全延遲初始化方法包括使用雙重檢查鎖定（Double-Checked Locking）或使用java.util.concurrent工具類中的延遲初始化工具。
* 雙重鎖定：
```java=
public class LazyInitializationExample {
    private volatile ExpensiveObject instance;

    public ExpensiveObject getInstance() {
        if (instance == null) {  // 第一次檢查
            synchronized (this) {
                if (instance == null) {  // 第二次檢查
                    instance = new ExpensiveObject();
                }
            }
        }
        return instance;
    }
}

class ExpensiveObject {
    // 這里可以是一個初始化成本很高的對象
}
```
* java.util.concurrent包中的LazyInitialization：
```java=
import java.util.concurrent.atomic.AtomicReference;

public class LazyInitializationExample {
    private AtomicReference<ExpensiveObject> instance = new AtomicReference<>();

    public ExpensiveObject getInstance() {
        ExpensiveObject result = instance.get();
        if (result == null) {
            result = new ExpensiveObject();
            if (instance.compareAndSet(null, result)) {
                // 只有一個線程能夠成功設置對象
                // 其他線程會返回第一個成功設置的對象
                return result;
            } else {
                // 如果設置失敗，意味著其他線程已經設置了對象
                // 返回其他線程設置的對象
                return instance.get();
            }
        }
        return result;
    }
}

class ExpensiveObject {
    // 這里可以是一個初始化成本很高的對象
}
```
## 84. 不要依賴線程調度器
線程調度器負責控制線程的執行順序和時間分配，但它通常是非確定性的，無法預測線程何時會執行或以何種順序執行。
以下是一些關於不依賴線程調度器的建議：
1. 使用適當的同步機制： 在多線程程序中，使用合適的同步機制（如鎖、信號量、條件變量等）來確保線程的正確執行順序，而不是依賴線程調度器的干預。
2. 避免 Busy-Waiting： 不要編寫忙等待（busy-waiting）的代碼，這會占用 CPU 資源並增加系統負擔。應該使用適當的等待和通知機制來實現線程之間的協作。
3. 使用線程池： 使用線程池可以更好地管理線程的生命周期，減少線程的創建和銷毀開銷，以及優化線程的執行。
4. 避免競爭條件： 設計代碼時應該避免競爭條件，確保多線程訪問共享資源的安全性。
5. 編寫可預測的代碼： 編寫多線程代碼時，應該力求代碼的行為對於不同的線程調度器也是可預測的。
## 85. 優先選擇 Java 序列化的替代方案
永遠要反序列化不可信的數據。
序列化是危險的，應該避免。如果你從頭開始設計一個系統，可以使用跨平台的結構化數據：
1. JSON 序列化： 使用 JSON 序列化庫（如Jackson、Gson、org.json等）可以將對象序列化為 JSON 格式。
2. Protocol Buffers（ProtoBuf）： Protocol Buffers 是一種由 Google 開發的二進制數據序列化格式，通過使用 Protocol Buffers 插件，您可以在 Java 中使用 ProtoBuf。
3. XML 序列化： XML 序列化是另一種替代方案，但通常比 JSON 或 Protocol Buffers 更冗長。Java 提供了內置的 XML 序列化支持，但也有第三方庫可用。
4. 自定義序列化： 在某些情況下，可能需要手動編寫自定義序列化和反序列化方法以滿足特定需求。這需要更多的工作，但可以獲得更精確的控制。
5. 外部化存儲： 如果對象的序列化和反序列化性能不是主要關注點，您可以考慮將對象持久化到數據庫或文件中，而不是進行二進制序列化。
## 86. 非常謹慎地實現 Serializable
* 版本控制： 一旦一個類實現了 Serializable 接口，它的序列化格式就變成了公共API的一部分。因此，如果未來對類的結構進行了更改，例如添加或刪除字段，就會破壞向後兼容性。為了解決這個問題，可以通過聲明序列化版本ID（serialVersionUID）來進行版本控制。這樣可以確保在反序列化時不會引發 InvalidClassException。
* 不要序列化敏感信息： 避免將敏感信息序列化到對象中，因為序列化後的數據可以在不受控制的環境中傳播。如果需要序列化對象，確保對敏感數據進行適當的處理，如加密。
* 自定義序列化： 對於某些對象，可能需要自定義序列化和反序列化過程，以便更好地控制數據的格式。這可以通過實現 writeObject 和 readObject 方法來實現。
* 序列化ID： 如果類中包含內部類、匿名類或 lambda 表達式，需要格外小心，因為這些類的序列化ID可能會導致問題。在這種情況下，最好提供自定義的 serialVersionUID。
* 避免對不可序列化的對象進行序列化： 一些對象，如線程、文件句柄等，是不可序列化的。試圖序列化它們可能會導致異常。
## 87. 考慮使用自定義的序列化形式
如果你已經決定一個類應該是可序列化的，那麽請仔細考慮一下序列化的形式應該是什麽。只有在合理描述對象的邏輯狀態時，才使用默認的序列化形式；否則，設計一個適合描述對象的自定義序列化形式。設計類的序列化形式應該和設計導出方法花的時間應該一樣多，都應該嚴謹對待。
以坐標數據的點對象為例：
```java=
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class Point implements Serializable {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // 自定義序列化方法
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject(); // 默認序列化
        out.writeInt(getSomeDerivedValue()); // 寫入其他屬性
    }

    // 自定義反序列化方法
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject(); // 默認反序列化
        int derivedValue = in.readInt(); // 讀取其他屬性
        setSomeDerivedValue(derivedValue); // 設置屬性
    }

    // 其他方法...

    // 獲取其他屬性的方法
    private int getSomeDerivedValue() {
        // 在實際應用中，根據x和y計算其他屬性的邏輯
        return x * y;
    }

    // 設置其他屬性的方法
    private void setSomeDerivedValue(int value) {
        // 設置其他屬性的邏輯
        // ...
    }
}
```
## 88. 保護性的編寫 readObject 方法
為了防止在惡意序列化或不可信的環境中執行潛在的安全漏洞或其他問題，這一技巧通常用於自定義序列化的類。
1. 使用 defaultReadObject 和 readFields：如果你需要調用默認的反序列化行為，確保使用 defaultReadObject 方法或 readFields 方法來讀取默認的字段值。
2. 驗證輸入數據：在讀取字段之前，驗證輸入數據的有效性。這包括檢查字段的合法性、範圍、長度等，確保輸入數據是可信的。
3. 執行安全性檢查：執行與安全性相關的檢查。例如，確保只有授權用戶可以執行反序列化操作，或確保數據沒有被篡改。
4. 初始化未序列化的字段：當你有字段在序列化時被排除（transient），並且需要在反序列化後進行初始化時，確保在 readObject 方法中進行這些初始化。
5. 特殊處理敏感字段：對於敏感字段，可能需要特殊處理。這包括密碼、密鑰、或其他需要額外保護的數據。
```java=
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    // 默認的反序列化行為，必須在最開始調用
    in.defaultReadObject();

    // 驗證字段的有效性
    if (age < 0) {
        throw new InvalidObjectException("Invalid age: " + age);
    }

    // 執行安全性檢查
    if (!isSafeObject()) {
        throw new SecurityException("Object is not safe");
    }

    // 初始化未序列化的字段
    initializeDerivedFields();

    // 特殊處理敏感字段
    decryptSensitiveData();
}
```
## 89. 對於實例控制，枚舉類型優於 readResolve
* readResolve 方法：在反序列化期間由 Java 對象輸入流調用的方法。該方法允許你返回一個替代對象，以替代通過反序列化創建的對象。這可用於控制對象的單例性，即確保反序列化後只有一個對象實例存在。使用 readResolve 需要在每個需要單例控制的類中編寫該方法。
* 枚舉類型：枚舉類型是天然的單例，因此無需特殊處理來確保只有一個實例。

不得已要使用 readResolve 方法的話，你需要在要序列化的類中定義它，並確保方法的簽名和訪問修飾符正確：
1. 在需要進行序列化和反序列化的類中添加 private Object readResolve() throws ObjectStreamException 方法。readResolve 方法應該是私有的，以確保只有類內部能夠調用它。
2. 在 readResolve 方法中編寫代碼，以返回你希望用於替代反序列化對象的對象。這可以是單例實例、特定狀態的對象或任何你需要的對象。
3. 當使用 ObjectInputStream 反序列化對象時，readResolve 方法將自動被調用。確保 readResolve 方法的邏輯返回你希望的替代對象。
4. 為了確保反序列化後的對象的完整性和一致性，若使用readResolve 方法來做實例受控，需確保該類的所有實例化字段都被基本類型，或者是transient 的。
```java=
import java.io.ObjectStreamException;
import java.io.Serializable;

public class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
        // 私有構造函數，確保不能通過構造函數創建新實例
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }

    private Object readResolve() throws ObjectStreamException {
        // 返回單例實例，確保在反序列化時只有一個實例
        return INSTANCE;
    }
}
```
* readObject 方法: 用於自定義反序列化過程，允許程序員手動控制如何讀取和恢覆對象的狀態。
* readResolve 方法: 用於返回代理對象或單例實例、特定狀態的對象，以便在反序列化時替代原始對象。通常不需要在 readResolve 方法中恢覆對象的狀態，因為它只返回一個代理對象。
## 90. 考慮用序列化代理代替序列化實例
序列化代理是一種設計模式，用於在序列化和反序列化對象時提供更大的靈活性和安全性。使用序列化代理時，你可以為某個類定義另一個類，該類的對象充當原始對象的代理，負責序列化和反序列化過程：
* 隱藏對象的實現細節： 有時，對象的實現細節可能會發生變化，但你不希望破壞現有的序列化兼容性。使用序列化代理，你可以隱藏對象的實現，並在序列化代理類中進行處理。
* 處理敏感數據： 如果對象包含敏感數據，你可以在序列化代理中將其加密，以確保數據在序列化和反序列化過程中的安全性。
* 當使用序列化代理類時，它會完全替代原始類的序列化和反序列化操作。即使外部類的屬性被聲明為 transient，序列化代理類的序列化和反序列化操作仍然會覆蓋它們。
```java=
import java.io.*;

class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }

    // 若要使用代理才需要宣告此方法
    private Object writeReplace() throws ObjectStreamException {
        return new SerializationProxy(this);
        // 不使用代理若欲減少年紀，但如此原對象的屬性會改變
        // this.age -= 5;
        // return this;
    }

    // 若使用代理，此方法不會被執行
//    private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException {
//       stream.defaultReadObject();
//        age += 5;  // 在反序列化時增加年齡
//    }

    // 序列化代理類
    private static class SerializationProxy implements Serializable {
        private String name;
        private int adjustedAge;

        SerializationProxy(Person person) {
            this.name = person.name;
            this.adjustedAge = person.age - 5;  // 在序列化時減少年齡
        }

        private Object readResolve() throws ObjectStreamException {
            return new Person(name, adjustedAge + 5);  // 在反序列化時還原並增加年齡
        }
    }
}

public class SerializationProxyExample {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        // 創建一個 Person 實例
        Person originalPerson = new Person("Alice", 30);

        // 序列化 Person 對象
        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("person.ser"));
        outputStream.writeObject(originalPerson);
        outputStream.close();

        // 反序列化 Person 對象
        ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream("person.ser"));
        Person deserializedPerson = (Person) inputStream.readObject();
        inputStream.close();

        System.out.println("Original Person: " + originalPerson);
        System.out.println("Deserialized Person: " + deserializedPerson);
    }
}
```
