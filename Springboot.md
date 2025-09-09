# Orta Seviye Java Backend Geliştirici Müfredatı

## İçindekiler

### 1. Java Temel Konular
- [1.1 Java Data Types (Primitive ve Non-Primitive)](#11-java-data-types)
- [1.2 Java Access Modifiers](#12-java-access-modifiers)
- [1.3 Java Collections Framework](#13-java-collections-framework)
- [1.4 Thread Safety ve Collections](#14-thread-safety-ve-collections)

### 2. İleri Seviye Java Konular
- [2.1 Java Concurrency ve Multithreading](#21-java-concurrency-ve-multithreading)
- [2.2 Java Garbage Collection (GC)](#22-java-garbage-collection-gc)
- [2.3 Java Release Features (8-21)](#23-java-release-features-8-21)

### 3. Spring Framework Temel Konular
- [3.1 Spring Bean Scopes](#31-spring-bean-scopes)
- [3.2 Spring Bean Annotations](#32-spring-bean-annotations)
- [3.3 Spring Transactional](#33-spring-transactional)

### 4. İleri Seviye Spring Konular
- [4.1 Spring Async](#41-spring-async)
- [4.2 Spring Boot 3.x Değişiklikleri](#42-spring-boot-3x-değişiklikleri)

### 5. Mesajlaşma Sistemleri
- [5.1 Apache Kafka](#51-apache-kafka)
- [5.2 RabbitMQ](#52-rabbitmq)
- [5.3 Kafka vs RabbitMQ Karşılaştırması](#53-kafka-vs-rabbitmq-karşılaştırması)

### 6. Web Teknolojileri
- [6.1 HTTP Stateful vs Stateless](#61-http-stateful-vs-stateless)
- [6.2 HTTP Request Methods](#62-http-request-methods)
- [6.3 Cron Jobs](#63-cron-jobs)

### 7. Ek Backend Konular
- [7.1 RESTful API Tasarımı](#71-restful-api-tasarımı)
- [7.2 Spring Security](#72-spring-security)
- [7.3 Database İşlemleri ve ORM](#73-database-işlemleri-ve-orm)
- [7.4 Caching Stratejileri](#74-caching-stratejileri)
- [7.5 Performance ve Monitoring](#75-performance-ve-monitoring)

---

## 1. Java Temel Konular

### 1.1 Java Data Types

#### Primitive Data Types (İlkel Veri Tipleri)

Java'da 8 adet primitive data type bulunur. Bunlar JVM tarafından doğrudan desteklenir ve stack memory'de saklanır.

**1. byte (8 bit)**
- **Açıklama**: -128 ile 127 arasında değer alabilen tam sayı tipi
- **Kullanım Alanları**: Dosya okuma/yazma, network protokolleri, memory-efficient storage
- **Alternatifler**: short, int (daha fazla memory kullanır)
- **Varsayılan Değer**: 0

**2. short (16 bit)**
- **Açıklama**: -32,768 ile 32,767 arasında değer alabilen tam sayı tipi
- **Kullanım Alanları**: Küçük sayısal değerler, array indexing
- **Alternatifler**: byte (daha az range), int (daha fazla memory)
- **Varsayılan Değer**: 0

**3. int (32 bit)**
- **Açıklama**: -2,147,483,648 ile 2,147,483,647 arasında değer alabilen tam sayı tipi
- **Kullanım Alanları**: En yaygın kullanılan integer tipi, loop counters, array indexing
- **Alternatifler**: long (daha büyük değerler için), short/byte (memory optimization için)
- **Varsayılan Değer**: 0

**4. long (64 bit)**
- **Açıklama**: -9,223,372,036,854,775,808 ile 9,223,372,036,854,775,807 arasında değer alabilen tam sayı tipi
- **Kullanım Alanları**: Büyük sayısal değerler, timestamp'ler, ID'ler
- **Alternatifler**: BigInteger (sınırsız büyüklük için)
- **Varsayılan Değer**: 0L

**5. float (32 bit)**
- **Açıklama**: IEEE 754 standardına uygun ondalıklı sayı tipi
- **Kullanım Alanları**: Basit ondalıklı hesaplamalar, memory-efficient floating point
- **Alternatifler**: double (daha yüksek precision için)
- **Varsayılan Değer**: 0.0f

**6. double (64 bit)**
- **Açıklama**: IEEE 754 standardına uygun yüksek precision ondalıklı sayı tipi
- **Kullanım Alanları**: Bilimsel hesaplamalar, finansal uygulamalar, en yaygın floating point tipi
- **Alternatifler**: BigDecimal (tam precision için), float (memory optimization için)
- **Varsayılan Değer**: 0.0d

**7. char (16 bit)**
- **Açıklama**: Unicode karakteri temsil eden tip
- **Kullanım Alanları**: Tek karakter işlemleri, string manipulation
- **Alternatifler**: String (çoklu karakter için)
- **Varsayılan Değer**: '\u0000' (null character)

**8. boolean (1 bit)**
- **Açıklama**: true veya false değer alabilen mantıksal tip
- **Kullanım Alanları**: Koşullu ifadeler, flag'ler, state management
- **Alternatifler**: Boolean wrapper class
- **Varsayılan Değer**: false

#### Non-Primitive Data Types (Referans Veri Tipleri)

**1. String**
- **Açıklama**: Karakter dizilerini temsil eden immutable sınıf
- **Kullanım Alanları**: Metin işlemleri, veri gösterimi, API responses
- **Alternatifler**: StringBuilder (mutable string operations için), StringBuffer (thread-safe mutable)
- **Özellikler**: Immutable, String Pool'da cache'lenir

**2. Array**
- **Açıklama**: Aynı tipte elemanları sıralı şekilde saklayan veri yapısı
- **Kullanım Alanları**: Veri koleksiyonları, algoritma implementasyonları
- **Alternatifler**: ArrayList (dynamic sizing için), LinkedList (frequent insertions/deletions için)
- **Özellikler**: Fixed size, indexed access

**3. Class Objects**
- **Açıklama**: Kullanıcı tanımlı sınıfların instance'ları
- **Kullanım Alanları**: OOP modeling, business logic, data encapsulation
- **Alternatifler**: Interface implementations, abstract classes
- **Özellikler**: Reference type, heap memory'de saklanır

**4. Interface**
- **Açıklama**: Sınıfların implement etmesi gereken contract'ları tanımlayan yapı
- **Kullanım Alanları**: Abstraction, polymorphism, loose coupling
- **Alternatifler**: Abstract classes (partial implementation için)
- **Özellikler**: Multiple inheritance destekler, default methods (Java 8+)

**5. Enum**
- **Açıklama**: Sabit değerler kümesini temsil eden özel sınıf türü
- **Kullanım Alanları**: State management, configuration values, type safety
- **Alternatifler**: Constants, final static fields
- **Özellikler**: Type-safe, singleton pattern, methods ve fields içerebilir

### 1.2 Java Access Modifiers

Access modifiers, sınıf, method ve field'ların erişilebilirlik seviyesini kontrol eder.

**1. public**
- **Açıklama**: Her yerden erişilebilir
- **Kullanım Alanları**: API methods, main method, constants
- **Avantajları**: Geniş erişim, kolay kullanım
- **Dezavantajları**: Encapsulation ihlali riski, tight coupling

**2. private**
- **Açıklama**: Sadece aynı sınıf içinden erişilebilir
- **Kullanım Alanları**: Internal implementation, data hiding, encapsulation
- **Avantajları**: Strong encapsulation, implementation hiding
- **Dezavantajları**: Testing zorluğu, inheritance limitations

**3. protected**
- **Açıklama**: Aynı package ve subclass'lardan erişilebilir
- **Kullanım Alanları**: Inheritance hierarchy, package-private API
- **Avantajları**: Controlled inheritance, package cohesion
- **Dezavantajları**: Package coupling, inheritance dependency

**4. default (package-private)**
- **Açıklama**: Aynı package içinden erişilebilir
- **Kullanım Alanları**: Internal package API, implementation details
- **Avantajları**: Package encapsulation, internal cohesion
- **Dezavantajları**: Package coupling, limited reusability

**Access Modifier Seçim Kriterleri:**
- **Encapsulation**: Mümkün olan en kısıtlayıcı modifier kullan
- **API Design**: Public interface'i minimal tut
- **Inheritance**: Protected kullanımını dikkatli planla
- **Testing**: Private method'ları test etmek için reflection gerekebilir

### 1.3 Java Collections Framework

Collections Framework, veri yapılarını ve algoritmalarını standardize eden bir API'dir.

#### List Interface

**ArrayList**
- **Açıklama**: Dynamic array implementation
- **Kullanım Alanları**: Random access, frequent reads, small to medium datasets
- **Avantajları**: O(1) random access, cache-friendly
- **Dezavantajları**: O(n) insertion/deletion, memory overhead
- **Thread Safety**: No (synchronized wrapper gerekir)

**LinkedList**
- **Açıklama**: Doubly linked list implementation
- **Kullanım Alanları**: Frequent insertions/deletions, queue operations
- **Avantajları**: O(1) insertion/deletion, memory efficient
- **Dezavantajları**: O(n) random access, poor cache performance
- **Thread Safety**: No

**Vector**
- **Açıklama**: Synchronized dynamic array
- **Kullanım Alanları**: Thread-safe operations, legacy code
- **Avantajları**: Thread-safe, synchronized
- **Dezavantajları**: Performance overhead, outdated
- **Thread Safety**: Yes (synchronized)

#### Set Interface

**HashSet**
- **Açıklama**: Hash table based implementation
- **Kullanım Alanları**: Unique elements, fast lookups
- **Avantajları**: O(1) average lookup, no duplicates
- **Dezavantajları**: No ordering, hash collision risk
- **Thread Safety**: No

**LinkedHashSet**
- **Açıklama**: Hash table + linked list (insertion order)
- **Kullanım Alanları**: Unique elements with insertion order
- **Avantajları**: O(1) lookup, maintains order
- **Dezavantajları**: Memory overhead, slower than HashSet
- **Thread Safety**: No

**TreeSet**
- **Açıklama**: Red-black tree based implementation
- **Kullanım Alanları**: Sorted unique elements
- **Avantajları**: O(log n) operations, sorted order
- **Dezavantajları**: Slower than HashSet, requires Comparable
- **Thread Safety**: No

#### Map Interface

**HashMap**
- **Açıklama**: Hash table based key-value mapping
- **Kullanım Alanları**: General purpose mapping, caching
- **Avantajları**: O(1) average operations, flexible
- **Dezavantajları**: No ordering, null key/value allowed
- **Thread Safety**: No

**LinkedHashMap**
- **Açıklama**: Hash table + linked list (insertion/access order)
- **Kullanım Alanları**: LRU cache, ordered mapping
- **Avantajları**: O(1) operations, maintains order
- **Dezavantajları**: Memory overhead, slower than HashMap
- **Thread Safety**: No

**TreeMap**
- **Açıklama**: Red-black tree based sorted mapping
- **Kullanım Alanları**: Sorted key-value pairs, range queries
- **Avantajları**: O(log n) operations, sorted keys
- **Dezavantajları**: Slower than HashMap, requires Comparable keys
- **Thread Safety**: No

**Hashtable**
- **Açıklama**: Synchronized hash table
- **Kullanım Alanları**: Thread-safe mapping, legacy code
- **Avantajları**: Thread-safe, synchronized
- **Dezavantajları**: Performance overhead, no null keys/values
- **Thread Safety**: Yes

#### Queue Interface

**LinkedList (Queue)**
- **Açıklama**: FIFO queue implementation
- **Kullanım Alanları**: Task scheduling, BFS algorithms
- **Avantajları**: O(1) enqueue/dequeue, flexible
- **Dezavantajları**: No priority, basic functionality
- **Thread Safety**: No

**PriorityQueue**
- **Açıklama**: Heap-based priority queue
- **Kullanım Alanları**: Task prioritization, Dijkstra's algorithm
- **Avantajları**: O(log n) operations, priority-based
- **Dezavantajları**: No random access, heap overhead
- **Thread Safety**: No

**ArrayDeque**
- **Açıklama**: Resizable array-based deque
- **Kullanım Alanları**: Stack operations, efficient deque
- **Avantajları**: O(1) operations, memory efficient
- **Dezavantajları**: No priority, fixed capacity growth
- **Thread Safety**: No

### 1.4 Thread Safety ve Collections

#### Thread-Safe Collections

**ConcurrentHashMap**
- **Açıklama**: Thread-safe hash map with segment-based locking
- **Kullanım Alanları**: Multi-threaded applications, caching
- **Avantajları**: High concurrency, no global locking
- **Dezavantajları**: Complex implementation, memory overhead
- **Alternatifler**: Collections.synchronizedMap(), Hashtable

**CopyOnWriteArrayList**
- **Açıklama**: Thread-safe list with copy-on-write semantics
- **Kullanım Alanları**: Read-heavy scenarios, event listeners
- **Avantajları**: No locking for reads, thread-safe
- **Dezavantajları**: Memory overhead, slow writes
- **Alternatifler**: Collections.synchronizedList(), Vector

**ConcurrentLinkedQueue**
- **Açıklama**: Lock-free thread-safe queue
- **Kullanım Alanları**: Producer-consumer patterns, task queues
- **Avantajları**: High performance, no blocking
- **Dezavantajları**: No size() method, memory overhead
- **Alternatifler**: BlockingQueue implementations

**BlockingQueue Implementations**
- **ArrayBlockingQueue**: Fixed-size blocking queue
- **LinkedBlockingQueue**: Unbounded blocking queue
- **PriorityBlockingQueue**: Priority-based blocking queue
- **SynchronousQueue**: Direct handoff queue

#### Thread-Safe Olmayan Collections

**HashMap, ArrayList, LinkedList, HashSet, TreeSet**
- **Problem**: Concurrent modification exceptions
- **Çözümler**: 
  - Synchronized wrappers (Collections.synchronizedX())
  - Concurrent collections
  - External synchronization
  - Immutable collections

**Synchronization Stratejileri:**
1. **Synchronized Methods**: Method-level locking
2. **Synchronized Blocks**: Block-level locking
3. **ReentrantLock**: More flexible locking
4. **ReadWriteLock**: Separate read/write locks
5. **Atomic Classes**: Lock-free operations

**Thread Safety Best Practices:**
- Immutable objects kullan
- Thread-safe collections tercih et
- Synchronization granularity'yi optimize et
- Deadlock'ları önle
- Performance vs safety trade-off'larını değerlendir

---

## 2. İleri Seviye Java Konular

### 2.1 Java Concurrency ve Multithreading

#### Thread Kavramı

**Thread Nedir?**
- **Açıklama**: Bir process içinde çalışan en küçük execution unit'i
- **Kullanım Alanları**: Paralel işlemler, responsive UI, background tasks
- **Avantajları**: CPU utilization, responsive applications, parallel processing
- **Dezavantajları**: Complexity, synchronization overhead, debugging zorluğu

**Thread Lifecycle:**
1. **NEW**: Thread oluşturuldu ama henüz başlatılmadı
2. **RUNNABLE**: Thread çalışmaya hazır veya çalışıyor
3. **BLOCKED**: Synchronized resource bekliyor
4. **WAITING**: Başka thread'in notify() beklemesi
5. **TIMED_WAITING**: Belirli süre bekleme
6. **TERMINATED**: Thread tamamlandı

#### Thread Oluşturma Yöntemleri

**1. Thread Class'ını Extend Etme**
- **Açıklama**: Thread sınıfından türetme
- **Kullanım Alanları**: Basit thread işlemleri
- **Avantajları**: Basit implementasyon
- **Dezavantajları**: Single inheritance limitation, tight coupling

**2. Runnable Interface Implement Etme**
- **Açıklama**: Runnable interface'ini implement etme
- **Kullanım Alanları**: Daha esnek thread yönetimi
- **Avantajları**: Multiple inheritance, loose coupling
- **Dezavantajları**: Daha fazla kod

**3. Callable Interface**
- **Açıklama**: Return value ve exception handling ile thread
- **Kullanım Alanları**: Result döndüren asenkron işlemler
- **Avantajları**: Return value, exception handling
- **Dezavantajları**: ExecutorService gerekir

**4. Lambda Expressions (Java 8+)**
- **Açıklama**: Functional interface'ler ile thread oluşturma
- **Kullanım Alanları**: Modern, concise thread creation
- **Avantajları**: Kısa kod, functional programming
- **Dezavantajları**: Debugging zorluğu

#### Synchronization

**Synchronized Keyword**
- **Açıklama**: Method veya block seviyesinde thread synchronization
- **Kullanım Alanları**: Critical section protection, shared resource access
- **Avantajları**: Basit kullanım, built-in support
- **Dezavantajları**: Performance overhead, deadlock riski

**Synchronized Methods:**
- Instance method: Object-level locking
- Static method: Class-level locking
- **Kullanım Alanları**: Method-level thread safety
- **Alternatifler**: Synchronized blocks, ReentrantLock

**Synchronized Blocks:**
- **Açıklama**: Belirli kod bloklarını synchronize etme
- **Kullanım Alanları**: Fine-grained synchronization, performance optimization
- **Avantajları**: Daha az locking, better performance
- **Dezavantajları**: Daha karmaşık kod

**Wait/Notify Pattern:**
- **wait()**: Thread'i bekletme, lock'u serbest bırakma
- **notify()**: Bekleyen bir thread'i uyandırma
- **notifyAll()**: Bekleyen tüm thread'leri uyandırma
- **Kullanım Alanları**: Producer-consumer patterns, coordination
- **Alternatifler**: Condition objects, CountDownLatch, CyclicBarrier

#### Advanced Concurrency

**ReentrantLock**
- **Açıklama**: Synchronized'a alternatif, daha esnek locking
- **Kullanım Alanları**: Complex locking scenarios, tryLock operations
- **Avantajları**: Try-lock, fair locking, interruptible locking
- **Dezavantajları**: Manual lock management, exception handling

**ReadWriteLock**
- **Açıklama**: Read ve write operasyonları için ayrı lock'lar
- **Kullanım Alanları**: Read-heavy scenarios, caching systems
- **Avantajları**: Multiple readers, better performance
- **Dezavantajları**: Writer starvation riski

**Atomic Classes**
- **Açıklama**: Lock-free thread-safe operations
- **Türler**: AtomicInteger, AtomicLong, AtomicReference, AtomicBoolean
- **Kullanım Alanları**: Counter operations, flag management
- **Avantajları**: High performance, no blocking
- **Dezavantajları**: Limited operations, ABA problem

**Volatile Keyword**
- **Açıklama**: Variable'ın main memory'den okunmasını garanti eder
- **Kullanım Alanları**: Simple flags, double-checked locking
- **Avantajları**: Visibility guarantee, no locking
- **Dezavantajları**: Atomicity guarantee yok, limited use cases

#### Executor Framework

**ExecutorService**
- **Açıklama**: Thread pool management ve task execution
- **Kullanım Alanları**: Asynchronous task processing, resource management
- **Avantajları**: Resource reuse, better performance, task management
- **Dezavantajları**: Configuration complexity

**Thread Pool Types:**
- **FixedThreadPool**: Sabit sayıda thread
- **CachedThreadPool**: Dinamik thread sayısı
- **SingleThreadExecutor**: Tek thread
- **ScheduledThreadPool**: Zamanlanmış task'lar

**Future ve CompletableFuture**
- **Future**: Asenkron işlem sonucunu temsil eder
- **CompletableFuture**: Reactive programming, chaining operations
- **Kullanım Alanları**: Asynchronous programming, reactive systems
- **Avantajları**: Non-blocking, composable operations
- **Dezavantajları**: Learning curve, complexity

#### Concurrency Utilities

**CountDownLatch**
- **Açıklama**: Bir veya daha fazla thread'in diğer thread'leri beklemesi
- **Kullanım Alanları**: Initialization synchronization, parallel processing
- **Avantajları**: One-time synchronization, simple usage
- **Dezavantajları**: Reusable değil

**CyclicBarrier**
- **Açıklama**: Thread'lerin belirli bir noktada buluşması
- **Kullanım Alanları**: Parallel algorithms, multi-phase processing
- **Avantajları**: Reusable, multiple phases
- **Dezavantajları**: Complex coordination

**Semaphore**
- **Açıklama**: Resource access control, permit-based synchronization
- **Kullanım Alanları**: Resource limiting, connection pooling
- **Avantajları**: Resource control, flexible permits
- **Dezavantajları**: Deadlock riski

**Phaser**
- **Açıklama**: Flexible barrier synchronization
- **Kullanım Alanları**: Complex multi-phase coordination
- **Avantajları**: Dynamic registration, flexible phases
- **Dezavantajları**: Complex usage

### 2.2 Java Garbage Collection (GC)

#### Garbage Collection Nedir?

**GC Açıklaması:**
- **Amaç**: Kullanılmayan object'leri otomatik olarak temizleme
- **Avantajları**: Memory leak prevention, automatic memory management
- **Dezavantajları**: Performance overhead, unpredictable pauses
- **Alternatifler**: Manual memory management (C/C++), reference counting

**GC Çalışma Prensibi:**
1. **Marking**: Live object'leri işaretleme
2. **Sweeping**: Dead object'leri temizleme
3. **Compacting**: Memory fragmentation'ı azaltma

#### Memory Areas

**Heap Memory:**
- **Young Generation**: Yeni object'ler
  - Eden Space: Yeni oluşturulan object'ler
  - Survivor Spaces (S0, S1): GC'den kurtulan object'ler
- **Old Generation**: Uzun süre yaşayan object'ler
- **Metaspace**: Class metadata (Java 8+)

**Non-Heap Memory:**
- **Method Area**: Class definitions, constants
- **Stack Memory**: Method calls, local variables
- **PC Register**: Current instruction pointer
- **Native Method Stack**: Native method calls

#### GC Algorithms

**Serial GC**
- **Açıklama**: Single-threaded, stop-the-world GC
- **Kullanım Alanları**: Small applications, client-side
- **Avantajları**: Simple, low overhead
- **Dezavantajları**: Long pauses, single-threaded

**Parallel GC (Throughput)**
- **Açıklama**: Multi-threaded, stop-the-world GC
- **Kullanım Alanları**: Batch processing, throughput-focused apps
- **Avantajları**: Better throughput, multi-threaded
- **Dezavantajları**: Still has pauses

**G1 GC (Garbage First)**
- **Açıklama**: Low-latency, region-based GC
- **Kullanım Alanları**: Large heap applications, low-latency requirements
- **Avantajları**: Predictable pauses, concurrent marking
- **Dezavantajları**: Higher overhead, complex tuning

**ZGC (Z Garbage Collector)**
- **Açıklama**: Ultra-low latency GC
- **Kullanım Alanları**: Real-time applications, large heaps
- **Avantajları**: Very low pause times, scalable
- **Dezavantajları**: Experimental, limited platforms

**Shenandoah GC**
- **Açıklama**: Concurrent GC with low pause times
- **Kullanım Alanları**: Responsive applications, large heaps
- **Avantajları**: Concurrent collection, low pauses
- **Dezavantajları**: Higher CPU usage, experimental

#### GC Tuning

**GC Parameters:**
- **-Xms**: Initial heap size
- **-Xmx**: Maximum heap size
- **-XX:NewRatio**: Old/Young generation ratio
- **-XX:SurvivorRatio**: Eden/Survivor ratio
- **-XX:MaxGCPauseMillis**: Target pause time (G1)

**GC Monitoring:**
- **-XX:+PrintGC**: Basic GC information
- **-XX:+PrintGCDetails**: Detailed GC information
- **-XX:+PrintGCTimeStamps**: GC timing information
- **-Xlog:gc**: Unified logging (Java 9+)

**GC Best Practices:**
- Appropriate heap size ayarla
- GC algorithm'ını uygulama tipine göre seç
- GC logs'ları monitor et
- Memory leak'leri önle
- Object pooling kullan (gerekirse)

### 2.3 Java Release Features (8-21)

#### Java 8 (2014) - Lambda ve Stream API

**Lambda Expressions**
- **Açıklama**: Functional programming desteği
- **Kullanım Alanları**: Collection operations, event handling, functional interfaces
- **Avantajları**: Concise code, functional programming
- **Dezavantajları**: Learning curve, debugging complexity

**Stream API**
- **Açıklama**: Functional-style operations on collections
- **Kullanım Alanları**: Data processing, filtering, mapping, reducing
- **Avantajları**: Declarative programming, parallel processing
- **Dezavantajları**: Performance overhead, complex debugging

**Optional Class**
- **Açıklama**: Null-safe value container
- **Kullanım Alanları**: Null pointer exception prevention, API design
- **Avantajları**: Explicit null handling, method chaining
- **Dezavantajları**: Learning curve, overuse riski

**Default Methods**
- **Açıklama**: Interface'lere default implementation
- **Kullanım Alanları**: Backward compatibility, interface evolution
- **Avantajları**: Interface evolution, multiple inheritance
- **Dezavantajları**: Diamond problem, complexity

**Method References**
- **Açıklama**: Lambda'ların kısa yazımı
- **Kullanım Alanları**: Method passing, functional programming
- **Avantajları**: Cleaner code, reusability
- **Dezavantajları**: Readability issues

#### Java 9 (2017) - Module System

**Module System (Project Jigsaw)**
- **Açıklama**: Modular Java platform
- **Kullanım Alanları**: Large applications, dependency management
- **Avantajları**: Better encapsulation, smaller runtime
- **Dezavantajları**: Migration complexity, learning curve

**JShell**
- **Açıklama**: Interactive Java shell
- **Kullanım Alanları**: Learning, prototyping, testing
- **Avantajları**: Quick testing, learning tool
- **Dezavantajları**: Limited IDE features

**Collection Factory Methods**
- **Açıklama**: Immutable collections oluşturma
- **Kullanım Alanları**: Immutable data structures, constants
- **Avantajları**: Concise syntax, immutable by default
- **Dezavantajları**: Limited mutability

#### Java 10 (2018) - Local Variable Type Inference

**var Keyword**
- **Açıklama**: Local variable type inference
- **Kullanım Alanları**: Verbose type declarations, complex generics
- **Avantajları**: Cleaner code, less verbosity
- **Dezavantajları**: Reduced readability, limited scope

#### Java 11 (2018) - LTS

**HTTP Client**
- **Açıklama**: Built-in HTTP client API
- **Kullanım Alanları**: Web services, API calls
- **Avantajları**: No external dependencies, modern API
- **Dezavantajları**: Learning curve

**String Methods**
- **Açıklama**: New string utility methods
- **Kullanım Alanları**: String processing, validation
- **Avantajları**: Built-in functionality, performance
- **Dezavantajları**: Limited scope

**Epsilon GC**
- **Açıklama**: No-op garbage collector
- **Kullanım Alanları**: Testing, performance measurement
- **Avantajları**: Predictable behavior, testing tool
- **Dezavantajları**: Not for production

#### Java 14 (2020) - Records ve Switch Expressions

**Records**
- **Açıklama**: Immutable data classes
- **Kullanım Alanları**: DTOs, value objects, data transfer
- **Avantajları**: Concise syntax, immutable by default
- **Dezavantajları**: Limited flexibility

**Switch Expressions**
- **Açıklama**: Expression-based switch statements
- **Kullanım Alanları**: Pattern matching, value assignment
- **Avantajları**: More concise, expression-based
- **Dezavantajları**: Learning curve

**Text Blocks**
- **Açıklama**: Multi-line string literals
- **Kullanım Alanları**: SQL queries, JSON, HTML templates
- **Avantajları**: Better readability, no escaping
- **Dezavantajları**: Indentation handling

#### Java 17 (2021) - LTS

**Sealed Classes**
- **Açıklama**: Restricted inheritance hierarchies
- **Kullanım Alanları**: Algebraic data types, controlled inheritance
- **Avantajları**: Better type safety, pattern matching
- **Dezavantajları**: Limited flexibility

**Pattern Matching for instanceof**
- **Açıklama**: Type checking and casting in one operation
- **Kullanım Alanları**: Type checking, casting operations
- **Avantajları**: Cleaner code, less boilerplate
- **Dezavantajları**: Limited scope

#### Java 21 (2023) - LTS

**Virtual Threads (Project Loom)**
- **Açıklama**: Lightweight threads managed by JVM
- **Kullanım Alanları**: High-concurrency applications, I/O bound tasks
- **Avantajları**: Millions of threads, better resource utilization
- **Dezavantajları**: Learning curve, compatibility issues

**Pattern Matching for Switch**
- **Açıklama**: Enhanced switch with pattern matching
- **Kullanım Alanları**: Type-based routing, data processing
- **Avantajları**: More expressive, type-safe
- **Dezavantajları**: Learning curve

**String Templates (Preview)**
- **Açıklama**: String interpolation with validation
- **Kullanım Alanları**: String formatting, SQL queries
- **Avantajları**: Type-safe interpolation, validation
- **Dezavantajları**: Preview feature, limited availability

**Sequenced Collections**
- **Açıklama**: Collections with defined encounter order
- **Kullanım Alanları**: Ordered data processing, navigation
- **Avantajları**: Better API, consistent behavior
- **Dezavantajları**: Limited scope

#### Java Release Strategy

**LTS (Long Term Support) Releases:**
- Java 8 (2014)
- Java 11 (2018)
- Java 17 (2021)
- Java 21 (2023)

**Feature Releases:**
- Her 6 ayda bir yeni özellikler
- 6 ay support
- LTS'e migration önerilir

**Migration Best Practices:**
- LTS versiyonları tercih et
- Incremental migration yap
- Testing stratejisi oluştur
- Performance impact'i değerlendir
- Deprecated API'leri güncelle

---

## 3. Spring Framework Temel Konular

### 3.1 Spring Bean Scopes

#### Bean Scope Nedir?

**Bean Scope Açıklaması:**
- **Amaç**: Bean instance'larının yaşam döngüsünü ve kapsamını belirleme
- **Kullanım Alanları**: Object lifecycle management, resource optimization, state management
- **Avantajları**: Flexible object management, performance optimization
- **Dezavantajları**: Configuration complexity, scope confusion

#### Singleton Scope (Varsayılan)

**Singleton Açıklaması:**
- **Açıklama**: Spring container'da tek instance oluşturulur ve paylaşılır
- **Kullanım Alanları**: Stateless services, configuration objects, utility classes
- **Avantajları**: Memory efficient, thread-safe (stateless), performance
- **Dezavantajları**: State sharing riski, testing zorluğu
- **Thread Safety**: Stateless olmalı, synchronized gerekebilir

**Singleton Kullanım Senaryoları:**
- Service layer classes
- Repository classes
- Configuration classes
- Utility classes
- Stateless components

#### Prototype Scope

**Prototype Açıklaması:**
- **Açıklama**: Her istekte yeni instance oluşturulur
- **Kullanım Alanları**: Stateful objects, mutable objects, request-specific data
- **Avantajları**: No state sharing, fresh instance, flexible lifecycle
- **Dezavantajları**: Memory overhead, performance impact
- **Thread Safety**: Her instance ayrı, thread-safe

**Prototype Kullanım Senaryoları:**
- Request-scoped data
- Mutable objects
- Stateful components
- Temporary objects
- User-specific data

#### Request Scope

**Request Açıklaması:**
- **Açıklama**: Her HTTP request için yeni instance
- **Kullanım Alanları**: Web applications, request-specific data, user sessions
- **Avantajları**: Request isolation, automatic cleanup
- **Dezavantajları**: Web context gerekir, memory overhead
- **Thread Safety**: Her request ayrı thread, thread-safe

**Request Kullanım Senaryoları:**
- User session data
- Request-specific services
- Web controllers
- Request-scoped beans
- Temporary data processing

#### Session Scope

**Session Açıklaması:**
- **Açıklama**: Her HTTP session için tek instance
- **Kullanım Alanları**: User-specific data, shopping carts, user preferences
- **Avantajları**: User isolation, persistent across requests
- **Dezavantajları**: Memory overhead, session management
- **Thread Safety**: Session-specific, thread-safe

**Session Kullanım Senaryoları:**
- Shopping cart
- User preferences
- Session-scoped services
- User-specific configuration
- Temporary user data

#### Application Scope

**Application Açıklaması:**
- **Açıklama**: ServletContext seviyesinde tek instance
- **Kullanım Alanları**: Global configuration, shared resources, application-wide data
- **Avantajları**: Global access, shared resources
- **Dezavantajları**: Memory overhead, global state
- **Thread Safety**: Shared state, synchronization gerekir

**Application Kullanım Senaryoları:**
- Global configuration
- Shared resources
- Application-wide caches
- Global services
- System-wide data

#### WebSocket Scope

**WebSocket Açıklaması:**
- **Açıklama**: Her WebSocket session için tek instance
- **Kullanım Alanları**: Real-time applications, WebSocket connections
- **Avantajları**: Connection-specific data, real-time communication
- **Dezavantajları**: WebSocket context gerekir, connection management
- **Thread Safety**: Connection-specific, thread-safe

#### Scope Seçim Kriterleri

**Stateless vs Stateful:**
- Stateless: Singleton
- Stateful: Prototype, Request, Session

**Performance vs Memory:**
- High performance: Singleton
- Memory efficient: Prototype (kısa süreli)

**Thread Safety:**
- Thread-safe: Singleton (stateless)
- Thread-unsafe: Prototype, Request, Session

**Lifecycle Management:**
- Long-lived: Singleton, Application
- Short-lived: Prototype, Request

### 3.2 Spring Bean Annotations

#### Component Annotations

**@Component**
- **Açıklama**: Generic Spring component annotation
- **Kullanım Alanları**: General purpose components, utility classes
- **Avantajları**: Simple, flexible, generic
- **Dezavantajları**: Generic, less specific
- **Alternatifler**: @Service, @Repository, @Controller

**@Service**
- **Açıklama**: Service layer components için özel annotation
- **Kullanım Alanları**: Business logic, service layer, application services
- **Avantajları**: Clear intent, layer separation
- **Dezavantajları**: Specific to service layer
- **Alternatifler**: @Component, @Repository

**@Repository**
- **Açıklama**: Data access layer components için özel annotation
- **Kullanım Alanları**: Data access, database operations, persistence layer
- **Avantajları**: Data access exception translation, clear intent
- **Dezavantajları**: Specific to data layer
- **Alternatifler**: @Component, @Service

**@Controller**
- **Açıklama**: Web layer components için özel annotation
- **Kullanım Alanları**: Web controllers, REST endpoints, MVC controllers
- **Avantajları**: Web-specific features, request mapping
- **Dezavantajları**: Web context gerekir
- **Alternatifler**: @RestController, @Component

**@RestController**
- **Açıklama**: REST API controllers için özel annotation
- **Kullanım Alanları**: REST APIs, JSON responses, web services
- **Avantajları**: @Controller + @ResponseBody, REST-specific
- **Dezavantajları**: Web context gerekir
- **Alternatifler**: @Controller, @Service

#### Configuration Annotations

**@Configuration**
- **Açıklama**: Configuration classes için annotation
- **Kullanım Alanları**: Bean definitions, configuration setup, application config
- **Avantajları**: Centralized configuration, bean management
- **Dezavantajları**: Configuration complexity
- **Alternatifler**: XML configuration, @Component

**@Bean**
- **Açıklama**: Method seviyesinde bean definition
- **Kullanım Alanları**: Custom bean creation, third-party integration
- **Avantajları**: Flexible bean creation, method-based
- **Dezavantajları**: Method-based, complex configuration
- **Alternatifler**: @Component, XML configuration

**@Import**
- **Açıklama**: Configuration classes'ı import etme
- **Kullanım Alanları**: Modular configuration, configuration composition
- **Avantajları**: Modular design, configuration reuse
- **Dezavantajları**: Configuration dependency
- **Alternatifler**: @ComponentScan, XML imports

**@ComponentScan**
- **Açıklama**: Component'leri otomatik tarama
- **Kullanım Alanları**: Automatic component discovery, package scanning
- **Avantajları**: Automatic discovery, less configuration
- **Dezavantajları**: Performance impact, unexpected scanning
- **Alternatifler**: Manual @Bean definitions, XML configuration

#### Dependency Injection Annotations

**@Autowired**
- **Açıklama**: Automatic dependency injection
- **Kullanım Alanları**: Field, method, constructor injection
- **Avantajları**: Automatic injection, flexible placement
- **Dezavantajları**: Reflection-based, runtime errors
- **Alternatifler**: @Resource, @Inject, constructor injection

**@Qualifier**
- **Açıklama**: Bean selection için qualifier
- **Kullanım Alanları**: Multiple bean types, bean disambiguation
- **Avantajları**: Precise bean selection, type safety
- **Dezavantajları**: Configuration complexity
- **Alternatifler**: @Primary, @Resource

**@Primary**
- **Açıklama**: Primary bean selection
- **Kullanım Alanları**: Default bean selection, bean priority
- **Avantajları**: Simple default selection
- **Dezavantajları**: Single primary, configuration dependency
- **Alternatifler**: @Qualifier, @Resource

**@Value**
- **Açıklama**: Property value injection
- **Kullanım Alanları**: Configuration values, environment variables
- **Avantajları**: Externalized configuration, flexible values
- **Dezavantajları**: String-based, type conversion
- **Alternatifler**: @ConfigurationProperties, Environment API

#### Lifecycle Annotations

**@PostConstruct**
- **Açıklama**: Bean initialization sonrası çalışan method
- **Kullanım Alanları**: Initialization logic, resource setup
- **Avantajları**: Automatic initialization, lifecycle management
- **Dezavantajları**: Exception handling, initialization order
- **Alternatifler**: InitializingBean, @Bean initMethod

**@PreDestroy**
- **Açıklama**: Bean destruction öncesi çalışan method
- **Kullanım Alanları**: Cleanup logic, resource release
- **Avantajları**: Automatic cleanup, resource management
- **Dezavantajları**: Exception handling, cleanup order
- **Alternatifler**: DisposableBean, @Bean destroyMethod

#### Profile Annotations

**@Profile**
- **Açıklama**: Environment-specific bean activation
- **Kullanım Alanları**: Environment configuration, conditional beans
- **Avantajları**: Environment separation, conditional loading
- **Dezavantajları**: Profile management, configuration complexity
- **Alternatifler**: @Conditional, @Bean conditions

**@Conditional**
- **Açıklama**: Custom condition-based bean activation
- **Kullanım Alanları**: Complex conditions, custom logic
- **Avantajları**: Flexible conditions, custom logic
- **Dezavantajları**: Complexity, custom implementation
- **Alternatifler**: @Profile, @Bean conditions

#### Annotation Best Practices

**Annotation Seçimi:**
- Specific annotations tercih et (@Service vs @Component)
- Layer separation'ı koru
- Clear intent göster

**Dependency Injection:**
- Constructor injection tercih et
- @Autowired'ı dikkatli kullan
- Circular dependency'leri önle

**Configuration Management:**
- @Configuration classes kullan
- @ComponentScan'ı optimize et
- Profile'ları düzgün yönet

### 3.3 Spring Transactional

#### Transaction Nedir?

**Transaction Açıklaması:**
- **Amaç**: Veritabanı işlemlerinin atomik, tutarlı ve izole edilmiş şekilde yürütülmesi
- **Kullanım Alanları**: Database operations, data consistency, business logic
- **Avantajları**: Data integrity, rollback capability, consistency
- **Dezavantajları**: Performance overhead, deadlock riski, complexity

**ACID Properties:**
- **Atomicity**: Tüm işlemler başarılı olur veya hiçbiri olmaz
- **Consistency**: Veritabanı her zaman tutarlı durumda kalır
- **Isolation**: Eşzamanlı işlemler birbirini etkilemez
- **Durability**: Commit edilen işlemler kalıcıdır

#### @Transactional Annotation

**@Transactional Açıklaması:**
- **Amaç**: Method veya class seviyesinde transaction yönetimi
- **Kullanım Alanları**: Service methods, data access methods, business operations
- **Avantajları**: Declarative transaction, AOP-based, flexible configuration
- **Dezavantajları**: AOP limitations, method visibility, exception handling

**@Transactional Kullanım Yerleri:**
- Service layer methods
- Repository methods
- Business logic methods
- Data modification operations
- Multi-step operations

#### Transaction Propagation

**REQUIRED (Varsayılan)**
- **Açıklama**: Mevcut transaction'a katıl, yoksa yeni oluştur
- **Kullanım Alanları**: Normal business operations, service calls
- **Avantajları**: Simple, predictable behavior
- **Dezavantajları**: Single transaction, rollback affects all

**REQUIRES_NEW**
- **Açıklama**: Her zaman yeni transaction oluştur
- **Kullanım Alanları**: Independent operations, logging, auditing
- **Avantajları**: Independent rollback, isolation
- **Dezavantajları**: Performance overhead, resource usage

**SUPPORTS**
- **Açıklama**: Mevcut transaction varsa katıl, yoksa transaction olmadan çalış
- **Kullanım Alanları**: Optional transactional operations, read operations
- **Avantajları**: Flexible, optional transaction
- **Dezavantajları**: Unpredictable behavior, debugging zorluğu

**MANDATORY**
- **Açıklama**: Mevcut transaction gerekli, yoksa exception fırlat
- **Kullanım Alanları**: Required transaction context, strict operations
- **Avantajları**: Strict transaction requirement, clear error
- **Dezavantajları**: Rigid, error-prone

**NOT_SUPPORTED**
- **Açıklama**: Mevcut transaction'ı suspend et, transaction olmadan çalış
- **Kullanım Alanları**: Non-transactional operations, external calls
- **Avantajları**: Non-transactional execution, external integration
- **Dezavantajları**: Transaction suspension, resource management

**NEVER**
- **Açıklama**: Transaction context'te çalışmaz, varsa exception fırlat
- **Kullanım Alanları**: Strict non-transactional operations
- **Avantajları**: Strict non-transactional, clear error
- **Dezavantajları**: Rigid, limited use cases

**NESTED**
- **Açıklama**: Nested transaction oluştur (savepoint-based)
- **Kullanım Alanları**: Partial rollback scenarios, complex operations
- **Avantajları**: Partial rollback, nested behavior
- **Dezavantajları**: Limited database support, complexity

#### Transaction Isolation Levels

**READ_UNCOMMITTED**
- **Açıklama**: En düşük isolation, dirty read'ler mümkün
- **Kullanım Alanları**: Performance-critical read operations
- **Avantajları**: High performance, no locking
- **Dezavantajları**: Dirty reads, inconsistent data

**READ_COMMITTED**
- **Açıklama**: Committed data'ları okur, dirty read'leri önler
- **Kullanım Alanları**: Most common isolation level
- **Avantajları**: Balance of performance and consistency
- **Dezavantajları**: Non-repeatable reads, phantom reads

**REPEATABLE_READ**
- **Açıklama**: Aynı transaction'da aynı data'ları okur
- **Kullanım Alanları**: Consistent read operations, reporting
- **Avantajları**: Repeatable reads, consistent data
- **Dezavantajları**: Phantom reads, performance impact

**SERIALIZABLE**
- **Açıklama**: En yüksek isolation, tüm anomaly'leri önler
- **Kullanım Alanları**: Critical data operations, financial systems
- **Avantajları**: Highest consistency, no anomalies
- **Dezavantajları**: Low performance, deadlock riski

#### Transaction Rollback Rules

**Default Rollback Behavior:**
- **RuntimeException**: Rollback
- **Error**: Rollback
- **Checked Exception**: No rollback

**@Transactional(rollbackFor = Exception.class)**
- **Açıklama**: Belirli exception'lar için rollback
- **Kullanım Alanları**: Custom rollback rules, business exceptions
- **Avantajları**: Flexible rollback control
- **Dezavantajları**: Configuration complexity

**@Transactional(noRollbackFor = RuntimeException.class)**
- **Açıklama**: Belirli exception'lar için rollback yapma
- **Kullanım Alanları**: Expected exceptions, business logic
- **Avantajları**: Selective rollback control
- **Dezavantajları**: Risk of data inconsistency

#### Transaction Timeout

**@Transactional(timeout = 30)**
- **Açıklama**: Transaction timeout süresi (saniye)
- **Kullanım Alanları**: Long-running operations, resource management
- **Avantajları**: Resource protection, timeout control
- **Dezavantajları**: Configuration complexity, timeout handling

#### Transaction Read-Only

**@Transactional(readOnly = true)**
- **Açıklama**: Read-only transaction, optimization hint
- **Kullanım Alanları**: Query operations, reporting, read-only services
- **Avantajları**: Performance optimization, clear intent
- **Dezavantajları**: Limited to read operations

#### Transaction Best Practices

**Transaction Design:**
- Service layer'da transaction kullan
- Short-lived transactions tercih et
- Appropriate isolation level seç
- Rollback rules'ları dikkatli planla

**Performance Optimization:**
- Read-only transactions kullan
- Timeout değerlerini ayarla
- Batch operations kullan
- Connection pooling optimize et

**Error Handling:**
- Exception handling stratejisi oluştur
- Rollback rules'ları netleştir
- Transaction boundaries'leri dikkatli planla
- Deadlock'ları önle

**Testing:**
- Transaction rollback testleri yaz
- Integration testleri kullan
- Test data isolation sağla
- Transaction behavior'ını verify et

---

## 4. İleri Seviye Spring Konular

### 4.1 Spring Async

#### Asynchronous Processing Nedir?

**Async Açıklaması:**
- **Amaç**: Uzun süren işlemleri non-blocking şekilde yürütme
- **Kullanım Alanları**: Background tasks, long-running operations, external API calls
- **Avantajları**: Better responsiveness, resource utilization, scalability
- **Dezavantajları**: Complexity, error handling, debugging zorluğu

**Synchronous vs Asynchronous:**
- **Synchronous**: Blocking, sequential execution, simple error handling
- **Asynchronous**: Non-blocking, parallel execution, complex error handling

#### @Async Annotation

**@Async Açıklaması:**
- **Amaç**: Method'ları asenkron olarak çalıştırma
- **Kullanım Alanları**: Service methods, background processing, external calls
- **Avantajları**: Simple annotation, automatic thread management
- **Dezavantajları**: AOP limitations, return value handling, exception propagation

**@Async Kullanım Yerleri:**
- Email sending
- File processing
- External API calls
- Data processing
- Background tasks

#### Async Configuration

**@EnableAsync**
- **Açıklama**: Async processing'i aktifleştirme
- **Kullanım Alanları**: Configuration classes, main application class
- **Avantajları**: Simple activation, global configuration
- **Dezavantajları**: Global activation, configuration complexity

**AsyncConfigurer Interface**
- **Açıklama**: Async configuration'ı customize etme
- **Kullanım Alanları**: Custom thread pools, exception handling
- **Avantajları**: Flexible configuration, custom behavior
- **Dezavantajları**: Implementation complexity, configuration overhead

**Thread Pool Configuration**
- **Açıklama**: Async method'lar için thread pool ayarlama
- **Kullanım Alanları**: Performance tuning, resource management
- **Avantajları**: Resource control, performance optimization
- **Dezavantajları**: Configuration complexity, tuning zorluğu

#### Async Method Types

**Void Methods**
- **Açıklama**: Return value olmayan async method'lar
- **Kullanım Alanları**: Fire-and-forget operations, background tasks
- **Avantajları**: Simple implementation, no return handling
- **Dezavantajları**: No result feedback, error handling zorluğu

**Future Return Methods**
- **Açıklama**: Future döndüren async method'lar
- **Kullanım Alanları**: Result beklenen operations, status checking
- **Avantajları**: Result handling, status checking
- **Dezavantajları**: Blocking get() calls, exception handling

**CompletableFuture Return Methods**
- **Açıklama**: CompletableFuture döndüren async method'lar
- **Kullanım Alanları**: Reactive programming, chaining operations
- **Avantajları**: Non-blocking, composable, flexible
- **Dezavantajları**: Learning curve, complexity

#### Async Exception Handling

**Exception Propagation**
- **Açıklama**: Async method'lardaki exception'ların handling'i
- **Kullanım Alanları**: Error handling, logging, monitoring
- **Avantajları**: Centralized error handling, monitoring
- **Dezavantajları**: Complex error propagation, debugging zorluğu

**AsyncUncaughtExceptionHandler**
- **Açıklama**: Uncaught exception'ları handle etme
- **Kullanım Alanları**: Global error handling, logging, monitoring
- **Avantajları**: Centralized error handling, logging
- **Dezavantajları**: Implementation complexity, error context loss

#### Async Best Practices

**Method Design:**
- Public method'lar kullan
- Self-invocation'dan kaçın
- Exception handling planla
- Return type'ları dikkatli seç

**Performance Optimization:**
- Appropriate thread pool size
- Resource management
- Timeout handling
- Monitoring ve logging

**Error Handling:**
- Exception handling stratejisi
- Logging ve monitoring
- Retry mechanisms
- Circuit breaker patterns

### 4.2 Spring Boot 3.x Değişiklikleri

#### Java 17+ Requirement

**Java 17 Minimum Requirement**
- **Açıklama**: Spring Boot 3.x Java 17+ gerektirir
- **Kullanım Alanları**: Modern Java features, performance improvements
- **Avantajları**: Modern Java features, better performance, security
- **Dezavantajları**: Migration complexity, compatibility issues

**Java 17 Features in Spring Boot 3.x:**
- Records support
- Pattern matching
- Text blocks
- Sealed classes
- Virtual threads (Java 21)

#### Jakarta EE Migration

**javax.* to jakarta.* Migration**
- **Açıklama**: Java EE'den Jakarta EE'ye migration
- **Kullanım Alanları**: All javax packages, servlet API, JPA
- **Avantajları**: Modern namespace, future compatibility
- **Dezavantajları**: Breaking changes, migration effort

**Affected Packages:**
- javax.servlet → jakarta.servlet
- javax.persistence → jakarta.persistence
- javax.validation → jakarta.validation
- javax.annotation → jakarta.annotation

**Migration Impact:**
- Import statements
- Configuration classes
- Dependencies
- Third-party libraries

#### Spring Framework 6.x

**Spring Framework 6.x Features**
- **Açıklama**: Spring Framework'in yeni versiyonu
- **Kullanım Alanları**: Core Spring functionality, AOP, IoC
- **Avantajları**: Performance improvements, new features
- **Dezavantajları**: Breaking changes, learning curve

**Key Changes:**
- Jakarta EE support
- Java 17+ requirement
- Performance improvements
- New features and APIs

#### Spring Boot 3.x New Features

**Observability**
- **Açıklama**: Built-in observability features
- **Kullanım Alanları**: Monitoring, tracing, metrics
- **Avantajları**: Better observability, monitoring
- **Dezavantajları**: Learning curve, configuration complexity

**Native Image Support**
- **Açıklama**: GraalVM native image support
- **Kullanım Alanları**: Cloud-native applications, serverless
- **Avantajları**: Fast startup, low memory usage
- **Dezavantajları**: Limited reflection, compatibility issues

**Problem Details API**
- **Açıklama**: RFC 7807 Problem Details standard
- **Kullanım Alanları**: Error responses, API standardization
- **Avantajları**: Standardized error format, better API design
- **Dezavantajları**: Learning curve, migration effort

#### Configuration Changes

**Property Changes**
- **Açıklama**: Configuration property'lerde değişiklikler
- **Kullanım Alanları**: Application configuration, environment setup
- **Avantajları**: Better organization, clearer naming
- **Dezavantajları**: Migration effort, breaking changes

**Profile Changes**
- **Açıklama**: Profile handling'de iyileştirmeler
- **Kullanım Alanları**: Environment-specific configuration
- **Avantajları**: Better profile management, clearer configuration
- **Dezavantajları**: Migration effort, configuration changes

#### Migration Strategies

**Incremental Migration**
- **Açıklama**: Aşamalı migration stratejisi
- **Kullanım Alanları**: Large applications, complex systems
- **Avantajları**: Risk reduction, gradual transition
- **Dezavantajları**: Longer migration time, complexity

**Big Bang Migration**
- **Açıklama**: Tek seferde migration
- **Kullanım Alanları**: Small applications, simple systems
- **Avantajları**: Quick migration, clean transition
- **Dezavantajları**: High risk, potential downtime

**Migration Checklist:**
- Java 17+ upgrade
- Jakarta EE migration
- Dependency updates
- Configuration updates
- Testing strategy
- Performance validation

#### Compatibility Considerations

**Third-party Libraries**
- **Açıklama**: Third-party library compatibility
- **Kullanım Alanları**: External dependencies, integrations
- **Avantajları**: Modern library versions, better features
- **Dezavantajları**: Breaking changes, migration effort

**Database Compatibility**
- **Açıklama**: Database driver ve ORM compatibility
- **Kullanım Alanları**: Data persistence, database operations
- **Avantajları**: Modern database features, better performance
- **Dezavantajları**: Driver updates, configuration changes

**Testing Compatibility**
- **Açıklama**: Testing framework compatibility
- **Kullanım Alanları**: Unit tests, integration tests
- **Avantajları**: Modern testing features, better tooling
- **Dezavantajları**: Test updates, configuration changes

#### Performance Improvements

**Startup Time**
- **Açıklama**: Application startup time improvements
- **Kullanım Alanları**: All applications, cloud deployments
- **Avantajları**: Faster deployment, better user experience
- **Dezavantajları**: Configuration complexity, tuning required

**Memory Usage**
- **Açıklama**: Memory usage optimizations
- **Kullanım Alanları**: Resource-constrained environments
- **Avantajları**: Better resource utilization, cost savings
- **Dezavantajları**: Configuration tuning, monitoring required

**Runtime Performance**
- **Açıklama**: Runtime performance improvements
- **Kullanım Alanları**: High-performance applications
- **Avantajları**: Better throughput, lower latency
- **Dezavantajları**: Configuration complexity, monitoring required

#### Migration Best Practices

**Preparation:**
- Dependency analysis
- Compatibility testing
- Performance benchmarking
- Backup strategy

**Migration Process:**
- Incremental approach
- Comprehensive testing
- Performance validation
- Rollback plan

**Post-Migration:**
- Monitoring setup
- Performance tuning
- Documentation updates
- Team training

---

## 5. Mesajlaşma Sistemleri

### 5.1 Apache Kafka

#### Kafka Nedir?

**Kafka Açıklaması:**
- **Amaç**: Distributed streaming platform, high-throughput message broker
- **Kullanım Alanları**: Real-time data streaming, event-driven architecture, log aggregation
- **Avantajları**: High throughput, scalability, durability, fault tolerance
- **Dezavantajları**: Complexity, operational overhead, learning curve

**Kafka Temel Kavramları:**
- **Producer**: Mesaj gönderen uygulamalar
- **Consumer**: Mesaj alan uygulamalar
- **Broker**: Kafka cluster'ındaki sunucular
- **Topic**: Mesajların kategorize edildiği kanallar
- **Partition**: Topic'lerin bölündüğü parçalar
- **Offset**: Mesajların unique identifier'ı

#### Kafka Architecture

**Distributed Architecture**
- **Açıklama**: Multiple broker'lardan oluşan cluster
- **Kullanım Alanları**: High availability, fault tolerance, scalability
- **Avantajları**: No single point of failure, horizontal scaling
- **Dezavantajları**: Complexity, network overhead

**Partitioning**
- **Açıklama**: Topic'lerin multiple partition'lara bölünmesi
- **Kullanım Alanları**: Parallel processing, load distribution
- **Avantajları**: Parallelism, scalability, ordering within partition
- **Dezavantajları**: Cross-partition ordering complexity

**Replication**
- **Açıklama**: Partition'ların multiple broker'larda kopyalanması
- **Kullanım Alanları**: Data durability, fault tolerance
- **Avantajları**: Data safety, high availability
- **Dezavantajları**: Storage overhead, consistency complexity

#### Kafka Features

**High Throughput**
- **Açıklama**: Yüksek mesaj işleme kapasitesi
- **Kullanım Alanları**: Big data, real-time processing, IoT
- **Avantajları**: Millions of messages per second, low latency
- **Dezavantajları**: Resource intensive, configuration complexity

**Durability**
- **Açıklama**: Mesajların disk'te persistent storage'ı
- **Kullanım Alanları**: Critical data, audit logs, compliance
- **Avantajları**: Data safety, recovery capability
- **Dezavantajları**: Storage overhead, performance impact

**Scalability**
- **Açıklama**: Horizontal ve vertical scaling desteği
- **Kullanım Alanları**: Growing applications, high-volume systems
- **Avantajları**: Linear scaling, resource efficiency
- **Dezavantajları**: Operational complexity, monitoring challenges

**Fault Tolerance**
- **Açıklama**: Broker failure'larına karşı dayanıklılık
- **Kullanım Alanları**: Production systems, critical applications
- **Avantajları**: High availability, automatic failover
- **Dezavantajları**: Configuration complexity, resource overhead

#### Kafka Use Cases

**Event Streaming**
- **Açıklama**: Real-time event processing
- **Kullanım Alanları**: User activity tracking, system events, notifications
- **Avantajları**: Real-time processing, decoupled systems
- **Dezavantajları**: Complexity, monitoring challenges

**Log Aggregation**
- **Açıklama**: Application log'larının centralized collection'ı
- **Kullanım Alanları**: Monitoring, debugging, analytics
- **Avantajları**: Centralized logging, real-time analysis
- **Dezavantajları**: Storage overhead, processing complexity

**Message Queue**
- **Açıklama**: Asynchronous message processing
- **Kullanım Alanları**: Microservices communication, task queues
- **Avantajları**: Decoupling, scalability, reliability
- **Dezavantajları**: Ordering complexity, consumer management

**Data Integration**
- **Açıklama**: Different systems arasında data flow
- **Kullanım Alanları**: ETL processes, data pipelines, real-time sync
- **Avantajları**: Real-time integration, flexible routing
- **Dezavantajları**: Data consistency, error handling

#### Kafka Configuration

**Producer Configuration**
- **Açıklama**: Producer'ların behavior'ını kontrol eden ayarlar
- **Kullanım Alanları**: Performance tuning, reliability, consistency
- **Avantajları**: Flexible configuration, performance optimization
- **Dezavantajları**: Configuration complexity, tuning zorluğu

**Consumer Configuration**
- **Açıklama**: Consumer'ların behavior'ını kontrol eden ayarlar
- **Kullanım Alanları**: Processing speed, reliability, resource usage
- **Avantajları**: Flexible processing, resource control
- **Dezavantajları**: Configuration complexity, monitoring challenges

**Broker Configuration**
- **Açıklama**: Kafka broker'larının ayarları
- **Kullanım Alanları**: Performance, durability, resource management
- **Avantajları**: System-wide optimization, resource control
- **Dezavantajları**: Complex tuning, operational overhead

### 5.2 RabbitMQ

#### RabbitMQ Nedir?

**RabbitMQ Açıklaması:**
- **Amaç**: Message broker, AMQP protocol implementation
- **Kullanım Alanları**: Message queuing, task distribution, microservices communication
- **Avantajları**: Mature, reliable, flexible routing, management tools
- **Dezavantajları**: Lower throughput than Kafka, single point of failure

**RabbitMQ Temel Kavramları:**
- **Producer**: Mesaj gönderen uygulamalar
- **Consumer**: Mesaj alan uygulamalar
- **Exchange**: Mesaj routing logic'i
- **Queue**: Mesajların saklandığı yer
- **Binding**: Exchange ve queue arasındaki connection
- **Routing Key**: Mesaj routing için kullanılan key

#### RabbitMQ Architecture

**Centralized Architecture**
- **Açıklama**: Single broker node (cluster support available)
- **Kullanım Alanları**: Small to medium applications, simple messaging
- **Avantajları**: Simple setup, easy management, reliable
- **Dezavantajları**: Single point of failure, limited scalability

**Exchange Types**
- **Direct**: Routing key'e göre direct routing
- **Topic**: Pattern-based routing
- **Fanout**: Broadcast to all bound queues
- **Headers**: Message header'lara göre routing

**Queue Management**
- **Açıklama**: Queue'ların lifecycle management'ı
- **Kullanım Alanları**: Message storage, consumer management
- **Avantajları**: Flexible queue configuration, durability options
- **Dezavantajları**: Memory usage, configuration complexity

#### RabbitMQ Features

**Reliability**
- **Açıklama**: Message delivery guarantee'leri
- **Kullanım Alanları**: Critical applications, financial systems
- **Avantajları**: At-least-once delivery, message persistence
- **Dezavantajları**: Performance overhead, complexity

**Flexible Routing**
- **Açıklama**: Complex message routing patterns
- **Kullanım Alanları**: Complex workflows, conditional routing
- **Avantajları**: Powerful routing, flexible patterns
- **Dezavantajları**: Configuration complexity, learning curve

**Management Interface**
- **Açıklama**: Web-based management UI
- **Kullanım Alanları**: Monitoring, debugging, configuration
- **Avantajları**: Easy monitoring, visual management
- **Dezavantajları**: Security considerations, resource usage

**Clustering**
- **Açıklama**: Multiple broker nodes'u cluster olarak çalıştırma
- **Kullanım Alanları**: High availability, scalability
- **Avantajları**: Fault tolerance, load distribution
- **Dezavantajları**: Configuration complexity, network overhead

#### RabbitMQ Use Cases

**Task Queues**
- **Açıklama**: Background task processing
- **Kullanım Alanları**: Email sending, image processing, data analysis
- **Avantajları**: Load balancing, fault tolerance, scalability
- **Dezavantajları**: Queue management, monitoring complexity

**Microservices Communication**
- **Açıklama**: Service'ler arası asynchronous communication
- **Kullanım Alanları**: Event-driven architecture, service decoupling
- **Avantajları**: Loose coupling, reliability, flexibility
- **Dezavantajları**: Message ordering, error handling

**Pub/Sub Messaging**
- **Açıklama**: Publish-subscribe pattern implementation
- **Kullanım Alanları**: Notifications, real-time updates, broadcasting
- **Avantajları**: Decoupled systems, flexible subscribers
- **Dezavantajları**: Message delivery guarantees, subscriber management

**RPC (Request-Reply)**
- **Açıklama**: Synchronous request-reply pattern
- **Kullanım Alanları**: Service calls, API communication
- **Avantajları**: Reliable communication, timeout handling
- **Dezavantajları**: Complexity, performance overhead

#### RabbitMQ Configuration

**Connection Management**
- **Açıklama**: Producer/consumer connection'larının yönetimi
- **Kullanım Alanları**: Resource management, performance optimization
- **Avantajları**: Connection pooling, resource efficiency
- **Dezavantajları**: Configuration complexity, monitoring challenges

**Queue Configuration**
- **Açıklama**: Queue'ların behavior'ını kontrol eden ayarlar
- **Kullanım Alanları**: Durability, TTL, message limits
- **Avantajları**: Flexible queue behavior, resource control
- **Dezavantajları**: Configuration complexity, tuning zorluğu

**Exchange Configuration**
- **Açıklama**: Exchange'lerin routing behavior'ını ayarlama
- **Kullanım Alanları**: Message routing, pattern matching
- **Avantajları**: Flexible routing, complex patterns
- **Dezavantajları**: Configuration complexity, debugging zorluğu

### 5.3 Kafka vs RabbitMQ Karşılaştırması

#### Performance Karşılaştırması

**Throughput**
- **Kafka**: Çok yüksek (millions of messages/second)
- **RabbitMQ**: Orta-yüksek (hundreds of thousands/second)
- **Kullanım**: Kafka büyük data, RabbitMQ normal applications

**Latency**
- **Kafka**: Düşük latency (milliseconds)
- **RabbitMQ**: Düşük-orta latency (milliseconds to seconds)
- **Kullanım**: Kafka real-time, RabbitMQ near real-time

**Scalability**
- **Kafka**: Excellent horizontal scaling
- **RabbitMQ**: Good vertical scaling, limited horizontal
- **Kullanım**: Kafka büyük scale, RabbitMQ medium scale

#### Architecture Karşılaştırması

**Message Storage**
- **Kafka**: Disk-based, persistent
- **RabbitMQ**: Memory-based (optional disk)
- **Kullanım**: Kafka durability, RabbitMQ performance

**Message Ordering**
- **Kafka**: Partition level ordering
- **RabbitMQ**: Queue level ordering
- **Kullanım**: Kafka partition-based, RabbitMQ queue-based

**Message Routing**
- **Kafka**: Simple topic-based
- **RabbitMQ**: Complex exchange-based
- **Kullanım**: Kafka simple routing, RabbitMQ complex routing

#### Use Case Karşılaştırması

**Event Streaming**
- **Kafka**: Excellent (designed for this)
- **RabbitMQ**: Good (can be used)
- **Öneri**: Kafka for event streaming

**Message Queuing**
- **Kafka**: Good (can be used)
- **RabbitMQ**: Excellent (designed for this)
- **Öneri**: RabbitMQ for message queuing

**Microservices Communication**
- **Kafka**: Good (event-driven)
- **RabbitMQ**: Excellent (request-reply, pub/sub)
- **Öneri**: RabbitMQ for microservices

**Log Aggregation**
- **Kafka**: Excellent (designed for this)
- **RabbitMQ**: Poor (not suitable)
- **Öneri**: Kafka for log aggregation

#### Operational Karşılaştırması

**Setup Complexity**
- **Kafka**: High (cluster setup, configuration)
- **RabbitMQ**: Low (single node), Medium (cluster)
- **Öneri**: RabbitMQ for simple setups

**Monitoring**
- **Kafka**: Complex (multiple tools needed)
- **RabbitMQ**: Simple (built-in management UI)
- **Öneri**: RabbitMQ for easier monitoring

**Maintenance**
- **Kafka**: High (cluster management, tuning)
- **RabbitMQ**: Medium (single node), High (cluster)
- **Öneri**: RabbitMQ for easier maintenance

#### Seçim Kriterleri

**Kafka Seçimi:**
- High throughput gerekli
- Event streaming use case
- Log aggregation gerekli
- Horizontal scaling gerekli
- Complex operational team

**RabbitMQ Seçimi:**
- Message queuing use case
- Complex routing gerekli
- Simple setup tercih ediliyor
- Request-reply pattern gerekli
- Limited operational resources

**Hybrid Approach:**
- Kafka for event streaming
- RabbitMQ for message queuing
- Different tools for different use cases
- Complexity vs functionality trade-off

#### Migration Considerations

**Kafka'dan RabbitMQ'ya:**
- Throughput azalması
- Routing complexity artışı
- Setup simplification
- Feature loss (partitioning, ordering)

**RabbitMQ'dan Kafka'ya:**
- Throughput artışı
- Setup complexity artışı
- Better scalability
- Learning curve

**Best Practices:**
- Use case'e göre seçim yap
- Performance requirements'ı değerlendir
- Operational capabilities'i göz önünde bulundur
- Pilot project ile test et
- Gradual migration düşün

---

## 6. Web Teknolojileri

### 6.1 HTTP Stateful vs Stateless

#### Stateful HTTP

**Stateful Açıklaması:**
- **Amaç**: Server'ın client state'ini saklaması ve takip etmesi
- **Kullanım Alanları**: Session management, user authentication, shopping carts
- **Avantajları**: Rich user experience, server-side state management
- **Dezavantajları**: Scalability issues, server memory usage, session management

**Stateful Özellikler:**
- **Session Storage**: Server'da client state saklama
- **Session ID**: Client'ı identify etme
- **State Persistence**: Request'ler arası state koruma
- **Server Memory**: State için memory kullanımı

**Stateful Kullanım Senaryoları:**
- User login sessions
- Shopping cart management
- Multi-step forms
- User preferences
- Application state

**Stateful Avantajları:**
- Rich user experience
- Server-side validation
- Centralized state management
- Security control
- Complex workflows

**Stateful Dezavantajları:**
- Scalability limitations
- Server memory usage
- Session management complexity
- Load balancing challenges
- Single point of failure

#### Stateless HTTP

**Stateless Açıklaması:**
- **Amaç**: Her request'in independent olması, server state saklamaması
- **Kullanım Alanları**: REST APIs, microservices, scalable applications
- **Avantajları**: High scalability, simple architecture, load balancing
- **Dezavantajları**: Client-side state management, repeated data transfer

**Stateless Özellikler:**
- **No Server State**: Server client state saklamaz
- **Self-Contained Requests**: Her request gerekli tüm bilgiyi içerir
- **Client State**: State client tarafında saklanır
- **Stateless Protocol**: HTTP'in doğal özelliği

**Stateless Kullanım Senaryoları:**
- REST API services
- Microservices architecture
- Scalable web applications
- Mobile applications
- Third-party integrations

**Stateless Avantajları:**
- High scalability
- Simple architecture
- Easy load balancing
- No session management
- Fault tolerance

**Stateless Dezavantajları:**
- Client-side complexity
- Repeated data transfer
- Limited user experience
- Security challenges
- State synchronization

#### Stateful vs Stateless Karşılaştırması

**Scalability**
- **Stateful**: Limited (server memory, session management)
- **Stateless**: Excellent (horizontal scaling, load balancing)
- **Kullanım**: Stateless for high-scale applications

**Performance**
- **Stateful**: Good (server-side processing, reduced data transfer)
- **Stateless**: Good (no server overhead, but more data transfer)
- **Kullanım**: Depends on use case and data size

**Complexity**
- **Stateful**: High (session management, state synchronization)
- **Stateless**: Low (simple request-response)
- **Kullanım**: Stateless for simple architectures

**User Experience**
- **Stateful**: Rich (server-side state, complex workflows)
- **Stateless**: Limited (client-side state, simple interactions)
- **Kullanım**: Stateful for complex user interfaces

**Security**
- **Stateful**: Good (server-side validation, centralized control)
- **Stateless**: Challenging (client-side validation, token management)
- **Kullanım**: Stateful for high-security applications

#### Hybrid Approaches

**Session-based with Stateless APIs**
- **Açıklama**: Web UI için session, API için stateless
- **Kullanım Alanları**: Web applications with API access
- **Avantajları**: Best of both worlds, flexible architecture
- **Dezavantajları**: Complexity, dual state management

**Token-based Authentication**
- **Açıklama**: JWT tokens ile stateless authentication
- **Kullanım Alanları**: Modern web applications, mobile apps
- **Avantajları**: Stateless, scalable, secure
- **Dezavantajları**: Token management, security considerations

**Client-side State Management**
- **Açıklama**: State client tarafında yönetilir
- **Kullanım Alanları**: Single-page applications, mobile apps
- **Avantajları**: Scalable, responsive, offline capability
- **Dezavantajları**: Client complexity, data synchronization

### 6.2 HTTP Request Methods

#### GET Method

**GET Açıklaması:**
- **Amaç**: Server'dan data retrieve etme
- **Kullanım Alanları**: Data fetching, resource retrieval, search operations
- **Avantajları**: Cacheable, bookmarkable, simple
- **Dezavantajları**: Limited data size, security concerns

**GET Özellikleri:**
- **Idempotent**: Aynı sonucu döndürür
- **Safe**: Server state değiştirmez
- **Cacheable**: Browser ve proxy cache'lenebilir
- **URL Parameters**: Query string ile data gönderilir

**GET Kullanım Senaryoları:**
- User profile retrieval
- Product listing
- Search operations
- Data fetching
- Resource access

**GET Best Practices:**
- Sensitive data gönderme
- Large data sets için pagination kullan
- Appropriate caching headers
- URL length limits'ini göz önünde bulundur

#### POST Method

**POST Açıklaması:**
- **Amaç**: Server'a data gönderme, resource oluşturma
- **Kullanım Alanları**: Form submissions, data creation, file uploads
- **Avantajları**: Large data support, secure, flexible
- **Dezavantajları**: Not cacheable, not idempotent

**POST Özellikleri:**
- **Not Idempotent**: Her çağrı farklı sonuç verebilir
- **Not Safe**: Server state değiştirebilir
- **Not Cacheable**: Cache'lenmez
- **Request Body**: Data request body'de gönderilir

**POST Kullanım Senaryoları:**
- User registration
- Form submissions
- File uploads
- Data creation
- Complex operations

**POST Best Practices:**
- Appropriate content-type headers
- Input validation
- Error handling
- Security considerations

#### PUT Method

**PUT Açıklaması:**
- **Amaç**: Resource'u create veya update etme
- **Kullanım Alanları**: Resource updates, data replacement
- **Avantajları**: Idempotent, clear semantics, complete replacement
- **Dezavantajları**: Complete resource gerekir, partial updates zor

**PUT Özellikleri:**
- **Idempotent**: Aynı sonucu döndürür
- **Not Safe**: Server state değiştirebilir
- **Complete Replacement**: Tüm resource'u replace eder
- **Request Body**: Complete resource data

**PUT Kullanım Senaryoları:**
- User profile updates
- Configuration updates
- Resource replacement
- Data synchronization
- Complete updates

**PUT Best Practices:**
- Complete resource data gönder
- Idempotent operations sağla
- Appropriate status codes
- Conflict handling

#### PATCH Method

**PATCH Açıklaması:**
- **Amaç**: Resource'u partial update etme
- **Kullanım Alanları**: Partial updates, field-specific changes
- **Avantajları**: Efficient, flexible, partial updates
- **Dezavantajları**: Not idempotent, complex implementation

**PATCH Özellikleri:**
- **Not Idempotent**: Partial updates idempotent olmayabilir
- **Not Safe**: Server state değiştirebilir
- **Partial Update**: Sadece değişen fields
- **Request Body**: Partial resource data

**PATCH Kullanım Senaryoları:**
- Field-specific updates
- Partial form submissions
- Incremental updates
- Selective modifications
- Efficient updates

**PATCH Best Practices:**
- Clear update semantics
- Proper error handling
- Validation strategies
- Conflict resolution

#### DELETE Method

**DELETE Açıklaması:**
- **Amaç**: Resource'u silme
- **Kullanım Alanları**: Resource deletion, cleanup operations
- **Avantajları**: Clear semantics, idempotent, simple
- **Dezavantajları**: Irreversible, security concerns

**DELETE Özellikleri:**
- **Idempotent**: Aynı sonucu döndürür
- **Not Safe**: Server state değiştirebilir
- **Resource Deletion**: Resource'u siler
- **No Request Body**: Genellikle body gerekmez

**DELETE Kullanım Senaryoları:**
- User account deletion
- Resource cleanup
- Data removal
- System cleanup
- Maintenance operations

**DELETE Best Practices:**
- Confirmation mechanisms
- Soft delete options
- Audit logging
- Security considerations

#### Other HTTP Methods

**HEAD Method**
- **Açıklama**: GET response'un headers'ını alır, body olmadan
- **Kullanım Alanları**: Resource metadata, cache validation
- **Avantajları**: Efficient, metadata only
- **Dezavantajları**: Limited information

**OPTIONS Method**
- **Açıklama**: Server'ın desteklediği methods'ları öğrenme
- **Kullanım Alanları**: CORS preflight, API discovery
- **Avantajları**: API discovery, CORS support
- **Dezavantajları**: Limited use cases

**TRACE Method**
- **Açıklama**: Request'in path'ini trace etme
- **Kullanım Alanları**: Debugging, network diagnostics
- **Avantajları**: Debugging tool, path verification
- **Dezavantajları**: Security risks, limited use

#### HTTP Method Best Practices

**Method Selection:**
- GET for data retrieval
- POST for data creation
- PUT for complete updates
- PATCH for partial updates
- DELETE for resource removal

**Idempotency:**
- GET, PUT, DELETE idempotent
- POST, PATCH not idempotent
- Design idempotent operations when possible

**Safety:**
- GET, HEAD, OPTIONS safe
- POST, PUT, PATCH, DELETE not safe
- Safe methods don't change server state

**Caching:**
- GET, HEAD cacheable
- POST, PUT, PATCH, DELETE not cacheable
- Use appropriate cache headers

### 6.3 Cron Jobs

#### Cron Nedir?

**Cron Açıklaması:**
- **Amaç**: Scheduled task execution, automated job running
- **Kullanım Alanları**: System maintenance, data processing, backups, reports
- **Avantajları**: Automation, reliability, scheduling flexibility
- **Dezavantajları**: Single point of failure, limited error handling

**Cron Temel Kavramları:**
- **Cron Expression**: Schedule format (minute hour day month weekday)
- **Cron Job**: Scheduled task definition
- **Cron Daemon**: Background process that runs cron jobs
- **Crontab**: Cron job configuration file

#### Cron Expression Format

**Basic Format:**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Special Characters:**
- **\***: Any value
- **,**: Value list separator
- **-**: Range of values
- **/**: Step values
- **?**: No specific value (day fields)

**Cron Expression Examples:**
- `0 0 * * *`: Every day at midnight
- `0 */6 * * *`: Every 6 hours
- `0 9 * * 1-5`: Every weekday at 9 AM
- `0 0 1 * *`: First day of every month
- `0 0 1 1 *`: January 1st every year

#### Cron Job Types

**System Maintenance**
- **Açıklama**: System cleanup, log rotation, maintenance tasks
- **Kullanım Alanları**: Log cleanup, temp file removal, system updates
- **Avantajları**: Automated maintenance, system health
- **Dezavantajları**: System impact, resource usage

**Data Processing**
- **Açıklama**: Batch data processing, ETL operations
- **Kullanım Alanları**: Data migration, report generation, data analysis
- **Avantajları**: Off-peak processing, resource optimization
- **Dezavantajları**: Processing time, error handling

**Backup Operations**
- **Açıklama**: Automated backup creation and management
- **Kullanım Alanları**: Database backups, file backups, system snapshots
- **Avantajları**: Data protection, automated recovery
- **Dezavantajları**: Storage requirements, backup validation

**Monitoring and Alerts**
- **Açıklama**: System monitoring, health checks, alert generation
- **Kullanım Alanları**: System health, performance monitoring, alerting
- **Avantajları**: Proactive monitoring, early warning
- **Dezavantajları**: Alert fatigue, false positives

#### Cron Job Management

**Crontab Commands**
- **crontab -e**: Edit crontab
- **crontab -l**: List crontab entries
- **crontab -r**: Remove crontab
- **crontab -u user**: User-specific crontab

**Cron Job Best Practices**
- **Logging**: Comprehensive logging for all jobs
- **Error Handling**: Proper error handling and notification
- **Resource Management**: Monitor resource usage
- **Testing**: Test cron jobs before deployment

**Cron Job Monitoring**
- **Log Analysis**: Regular log review
- **Status Checking**: Job execution status monitoring
- **Performance Monitoring**: Resource usage tracking
- **Alert Setup**: Failure notification systems

#### Spring Boot Cron Integration

**@Scheduled Annotation**
- **Açıklama**: Spring Boot'da cron job tanımlama
- **Kullanım Alanları**: Application-level scheduled tasks
- **Avantajları**: Spring integration, easy configuration
- **Dezavantajları**: Application dependency, single instance

**@EnableScheduling**
- **Açıklama**: Scheduling functionality'yi aktifleştirme
- **Kullanım Alanları**: Configuration classes, main application
- **Avantajları**: Simple activation, global configuration
- **Dezavantajları**: Global activation, configuration complexity

**Cron Expression in Spring**
- **Açıklama**: Spring'de cron expression kullanımı
- **Kullanım Alanları**: Method-level scheduling, flexible timing
- **Avantajları**: Flexible scheduling, Spring integration
- **Dezavantajları**: Configuration complexity, expression validation

#### Cron Job Alternatives

**Quartz Scheduler**
- **Açıklama**: Java-based job scheduling library
- **Kullanım Alanları**: Complex scheduling, enterprise applications
- **Avantajları**: Advanced features, clustering support
- **Dezavantajları**: Complexity, resource overhead

**Spring Task Scheduler**
- **Açıklama**: Spring Framework'ün built-in scheduler'ı
- **Kullanım Alanları**: Spring applications, simple scheduling
- **Avantajları**: Spring integration, simple configuration
- **Dezavantajları**: Limited features, single instance

**Cloud Schedulers**
- **Açıklama**: Cloud provider'ların scheduler servisleri
- **Kullanım Alanları**: Cloud applications, distributed systems
- **Avantajları**: Managed service, high availability
- **Dezavantajları**: Vendor lock-in, cost considerations

#### Cron Job Best Practices

**Design Principles:**
- Idempotent operations
- Proper error handling
- Resource cleanup
- Logging and monitoring

**Security Considerations:**
- Limited permissions
- Secure execution environment
- Input validation
- Access control

**Performance Optimization:**
- Resource usage monitoring
- Off-peak execution
- Parallel processing
- Resource cleanup

**Monitoring and Alerting:**
- Comprehensive logging
- Failure notifications
- Performance monitoring
- Health checks

---

## 7. Ek Backend Konular

### 7.1 RESTful API Tasarımı

#### REST Nedir?

**REST Açıklaması:**
- **Amaç**: Representational State Transfer, web servisleri için architectural style
- **Kullanım Alanları**: Web APIs, microservices, distributed systems
- **Avantajları**: Simple, scalable, stateless, cacheable
- **Dezavantajları**: Limited functionality, over-fetching, under-fetching

**REST Principles:**
- **Client-Server**: Separation of concerns
- **Stateless**: No client context stored on server
- **Cacheable**: Responses can be cached
- **Uniform Interface**: Consistent API design
- **Layered System**: Hierarchical layers
- **Code on Demand**: Optional executable code

#### RESTful API Design Principles

**Resource-Based URLs**
- **Açıklama**: URLs should represent resources, not actions
- **Kullanım Alanları**: All REST APIs, resource identification
- **Avantajları**: Intuitive, consistent, cacheable
- **Dezavantajları**: Complex resource hierarchies

**HTTP Methods Usage**
- **GET**: Retrieve resources
- **POST**: Create resources
- **PUT**: Update/replace resources
- **PATCH**: Partial updates
- **DELETE**: Remove resources

**Status Codes**
- **2xx Success**: 200 OK, 201 Created, 204 No Content
- **3xx Redirection**: 301 Moved Permanently, 304 Not Modified
- **4xx Client Error**: 400 Bad Request, 401 Unauthorized, 404 Not Found
- **5xx Server Error**: 500 Internal Server Error, 503 Service Unavailable

#### API Versioning

**URL Versioning**
- **Açıklama**: Version in URL path (/api/v1/users)
- **Kullanım Alanları**: Breaking changes, major updates
- **Avantajları**: Clear versioning, easy to understand
- **Dezavantajları**: URL pollution, maintenance overhead

**Header Versioning**
- **Açıklama**: Version in HTTP headers (Accept: application/vnd.api+json;version=1)
- **Kullanım Alanları**: Content negotiation, API evolution
- **Avantajları**: Clean URLs, flexible versioning
- **Dezavantajları**: Less visible, complex implementation

**Query Parameter Versioning**
- **Açıklama**: Version as query parameter (?version=1)
- **Kullanım Alanları**: Optional versioning, backward compatibility
- **Avantajları**: Simple implementation, optional
- **Dezavantajları**: Not RESTful, caching issues

#### API Documentation

**OpenAPI/Swagger**
- **Açıklama**: API specification standard
- **Kullanım Alanları**: API documentation, client generation
- **Avantajları**: Standardized, tool support, interactive docs
- **Dezavantajları**: Learning curve, maintenance overhead

**API Documentation Best Practices**
- Complete endpoint documentation
- Request/response examples
- Error code documentation
- Authentication requirements
- Rate limiting information

### 7.2 Spring Security

#### Spring Security Nedir?

**Spring Security Açıklaması:**
- **Amaç**: Authentication ve authorization framework
- **Kullanım Alanları**: Web security, API security, method security
- **Avantajları**: Comprehensive, flexible, Spring integration
- **Dezavantajları**: Complexity, learning curve, configuration overhead

**Spring Security Features:**
- Authentication (who you are)
- Authorization (what you can do)
- Protection against common attacks
- Integration with Spring ecosystem
- Extensible architecture

#### Authentication

**Authentication Methods**
- **Form-based**: HTML form authentication
- **HTTP Basic**: Username/password in headers
- **JWT**: Token-based authentication
- **OAuth2**: Third-party authentication
- **LDAP**: Directory service authentication

**User Details Service**
- **Açıklama**: User information loading interface
- **Kullanım Alanları**: Custom user loading, database integration
- **Avantajları**: Flexible, database integration
- **Dezavantajları**: Implementation complexity

**Password Encoding**
- **BCrypt**: Recommended password encoder
- **Argon2**: Modern password hashing
- **PBKDF2**: Key derivation function
- **SCrypt**: Memory-hard function

#### Authorization

**Method Security**
- **@PreAuthorize**: Pre-method authorization
- **@PostAuthorize**: Post-method authorization
- **@Secured**: Simple role-based security
- **@RolesAllowed**: JSR-250 annotation

**URL Security**
- **Açıklama**: URL pattern-based security
- **Kullanım Alanları**: Web security, endpoint protection
- **Avantajları**: Simple configuration, pattern matching
- **Dezavantajları**: Limited flexibility

**Expression-based Security**
- **Açıklama**: SpEL expressions for security
- **Kullanım Alanları**: Complex authorization logic
- **Avantajları**: Flexible, powerful expressions
- **Dezavantajları**: Complexity, performance impact

#### OAuth2 Integration

**OAuth2 Grant Types**
- **Authorization Code**: Server-side applications
- **Client Credentials**: Service-to-service
- **Resource Owner Password**: Legacy applications
- **Implicit**: Client-side applications (deprecated)

**JWT Token Support**
- **Açıklama**: JSON Web Token integration
- **Kullanım Alanları**: Stateless authentication, microservices
- **Avantajları**: Stateless, scalable, self-contained
- **Dezavantajları**: Token management, security considerations

### 7.3 Database İşlemleri ve ORM

#### JPA (Java Persistence API)

**JPA Açıklaması:**
- **Amaç**: Java object-relational mapping standard
- **Kullanım Alanları**: Database operations, object persistence
- **Avantajları**: Standardized, vendor independent, Spring integration
- **Dezavantajları**: Performance overhead, learning curve

**JPA Annotations**
- **@Entity**: Marks class as JPA entity
- **@Table**: Specifies table name
- **@Id**: Primary key field
- **@GeneratedValue**: Auto-generated values
- **@Column**: Column mapping
- **@OneToMany**: One-to-many relationship
- **@ManyToOne**: Many-to-one relationship
- **@ManyToMany**: Many-to-many relationship

**Entity Lifecycle**
- **Transient**: Not managed by JPA
- **Persistent**: Managed by JPA
- **Detached**: Previously persistent, not managed
- **Removed**: Marked for deletion

#### Hibernate

**Hibernate Açıklaması:**
- **Amaç**: JPA implementation, ORM framework
- **Kullanım Alanları**: Database operations, object persistence
- **Avantajları**: Feature-rich, mature, Spring integration
- **Dezavantajları**: Complexity, performance tuning

**Hibernate Features**
- **Lazy Loading**: On-demand data loading
- **Caching**: First-level and second-level cache
- **Query Language**: HQL (Hibernate Query Language)
- **Criteria API**: Type-safe queries
- **Batch Processing**: Efficient bulk operations

**Hibernate Configuration**
- **Connection Properties**: Database connection settings
- **Dialect**: Database-specific SQL generation
- **Cache Settings**: Second-level cache configuration
- **Logging**: SQL and query logging

#### Spring Data JPA

**Spring Data JPA Açıklaması:**
- **Amaç**: JPA repository abstraction
- **Kullanım Alanları**: Data access layer, repository pattern
- **Avantajları**: Reduced boilerplate, Spring integration
- **Dezavantajları**: Learning curve, abstraction overhead

**Repository Interfaces**
- **CrudRepository**: Basic CRUD operations
- **PagingAndSortingRepository**: Pagination and sorting
- **JpaRepository**: JPA-specific operations
- **Custom Repository**: Custom query methods

**Query Methods**
- **Açıklama**: Method name-based query generation
- **Kullanım Alanları**: Simple queries, dynamic queries
- **Avantajları**: No SQL, type-safe, Spring integration
- **Dezavantajları**: Limited complexity, naming conventions

**@Query Annotation**
- **Açıklama**: Custom query definition
- **Kullanım Alanları**: Complex queries, native SQL
- **Avantajları**: Flexible, powerful queries
- **Dezavantajları**: Database dependency, maintenance

### 7.4 Caching Stratejileri

#### Caching Nedir?

**Caching Açıklaması:**
- **Amaç**: Frequently accessed data'yı memory'de saklama
- **Kullanım Alanları**: Performance optimization, reduced database load
- **Avantajları**: Fast access, reduced load, better performance
- **Dezavantajları**: Memory usage, data consistency, complexity

**Caching Levels**
- **Browser Cache**: Client-side caching
- **CDN Cache**: Content delivery network caching
- **Application Cache**: Application-level caching
- **Database Cache**: Database query caching

#### Spring Cache Abstraction

**@Cacheable**
- **Açıklama**: Method result caching
- **Kullanım Alanları**: Expensive operations, data retrieval
- **Avantajları**: Automatic caching, Spring integration
- **Dezavantajları**: Configuration complexity

**@CacheEvict**
- **Açıklama**: Cache entry removal
- **Kullanım Alanları**: Data updates, cache invalidation
- **Avantajları**: Cache consistency, selective eviction
- **Dezavantajları**: Cache management complexity

**@CachePut**
- **Açıklama**: Cache update without method execution
- **Kullanım Alanları**: Cache population, data synchronization
- **Avantajları**: Cache control, data consistency
- **Dezavantajları**: Complex cache management

#### Cache Providers

**Caffeine**
- **Açıklama**: High-performance Java cache
- **Kullanım Alanları**: Local caching, high-performance applications
- **Avantajları**: High performance, low overhead
- **Dezavantajları**: Local only, memory limited

**Redis**
- **Açıklama**: In-memory data structure store
- **Kullanım Alanları**: Distributed caching, session storage
- **Avantajları**: Distributed, persistent, feature-rich
- **Dezavantajları**: Network overhead, complexity

**Hazelcast**
- **Açıklama**: In-memory data grid
- **Kullanım Alanları**: Distributed caching, clustering
- **Avantajları**: Distributed, scalable, Java integration
- **Dezavantajları**: Complexity, resource usage

#### Caching Strategies

**Cache-Aside (Lazy Loading)**
- **Açıklama**: Application manages cache
- **Kullanım Alanları**: General purpose caching
- **Avantajları**: Simple, flexible
- **Dezavantajları**: Cache miss handling, consistency

**Write-Through**
- **Açıklama**: Write to cache and database simultaneously
- **Kullanım Alanları**: Critical data, consistency requirements
- **Avantajları**: Data consistency, cache always updated
- **Dezavantajları**: Write performance impact

**Write-Behind (Write-Back)**
- **Açıklama**: Write to cache first, database later
- **Kullanım Alanları**: High write performance, eventual consistency
- **Avantajları**: High write performance, reduced database load
- **Dezavantajları**: Data loss risk, complexity

### 7.5 Performance ve Monitoring

#### Performance Optimization

**JVM Tuning**
- **Heap Size**: -Xms, -Xmx parameters
- **Garbage Collection**: GC algorithm selection
- **Memory Settings**: New generation, old generation
- **Thread Settings**: Thread pool configuration

**Database Optimization**
- **Indexing**: Proper index design
- **Query Optimization**: Efficient SQL queries
- **Connection Pooling**: Database connection management
- **Batch Operations**: Bulk data operations

**Application Optimization**
- **Lazy Loading**: On-demand data loading
- **Caching**: Strategic data caching
- **Async Processing**: Non-blocking operations
- **Resource Management**: Memory and CPU optimization

#### Monitoring ve Observability

**Application Metrics**
- **Response Time**: Request processing time
- **Throughput**: Requests per second
- **Error Rate**: Error percentage
- **Resource Usage**: CPU, memory, disk usage

**Health Checks**
- **Açıklama**: Application health monitoring
- **Kullanım Alanları**: Load balancer integration, monitoring
- **Avantajları**: Proactive monitoring, automated recovery
- **Dezavantajları**: Implementation complexity

**Logging Strategies**
- **Structured Logging**: JSON format, searchable
- **Log Levels**: DEBUG, INFO, WARN, ERROR
- **Centralized Logging**: ELK stack, Splunk
- **Log Aggregation**: Multiple service logs

**Distributed Tracing**
- **Açıklama**: Request flow tracking across services
- **Kullanım Alanları**: Microservices, complex systems
- **Avantajları**: End-to-end visibility, performance analysis
- **Dezavantajları**: Overhead, complexity

#### Monitoring Tools

**APM (Application Performance Monitoring)**
- **New Relic**: Commercial APM solution
- **AppDynamics**: Enterprise APM platform
- **Datadog**: Cloud monitoring platform
- **Prometheus**: Open-source monitoring

**Log Management**
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Splunk**: Enterprise log management
- **Fluentd**: Log collection and forwarding
- **Grafana**: Metrics visualization

**Infrastructure Monitoring**
- **Nagios**: Infrastructure monitoring
- **Zabbix**: Network monitoring
- **Prometheus + Grafana**: Metrics and visualization
- **CloudWatch**: AWS monitoring

#### Performance Testing

**Load Testing**
- **Açıklama**: Normal expected load testing
- **Kullanım Alanları**: Performance validation, capacity planning
- **Avantajları**: Realistic testing, performance baseline
- **Dezavantajları**: Test environment requirements

**Stress Testing**
- **Açıklama**: Beyond normal capacity testing
- **Kullanım Alanları**: Breaking point identification, failure analysis
- **Avantajları**: System limits identification, failure modes
- **Dezavantajları**: System impact, resource requirements

**Performance Testing Tools**
- **JMeter**: Apache JMeter for load testing
- **Gatling**: High-performance load testing
- **Artillery**: Modern load testing tool
- **K6**: Developer-centric load testing

#### Best Practices

**Performance Best Practices:**
- Regular performance testing
- Monitoring and alerting setup
- Capacity planning
- Performance optimization cycles

**Monitoring Best Practices:**
- Comprehensive metrics collection
- Meaningful alerts and thresholds
- Regular monitoring review
- Incident response procedures

**Security Best Practices:**
- Secure configuration
- Regular security updates
- Access control and authentication
- Data encryption and protection

