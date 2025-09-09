# Java Spring Boot Mülakat Soruları

## İçindekiler
1. [Java Collections](#java-collections)
2. [Thread Safety](#thread-safety)
3. [Data Types](#data-types)
4. [Java 21 & Spring Boot 3.x Değişiklikleri](#java-21--spring-boot-3x-değişiklikleri)
5. [Spring Bean Scopes](#spring-bean-scopes)
6. [Spring Bean Annotations](#spring-bean-annotations)
7. [Spring Transactional](#spring-transactional)
8. [Java Release Features (8-21)](#java-release-features-8-21)
9. [Java Access Modifiers](#java-access-modifiers)
10. [Java Garbage Collection](#java-garbage-collection)
11. [Cron Jobs](#cron-jobs)
12. [HTTP Stateful vs Stateless](#http-stateful-vs-stateless)
13. [HTTP Request Methods](#http-request-methods)
14. [Apache Kafka](#apache-kafka)
15. [Spring Async](#spring-async)
16. [Kafka vs RabbitMQ](#kafka-vs-rabbitmq)
17. [Java Concurrency and Multithreading](#java-concurrency-and-multithreading)
18. [Spring Framework Temelleri](#spring-framework-temelleri)
19. [Spring Security](#spring-security)
20. [Spring Data JPA](#spring-data-jpa)
21. [Spring Boot Auto Configuration](#spring-boot-auto-configuration)
22. [RESTful Web Services](#restful-web-services)
23. [Microservices](#microservices)
24. [Design Patterns](#design-patterns)
25. [Database & ORM](#database--orm)
26. [Testing](#testing)
27. [Performance & Optimization](#performance--optimization)
28. [DevOps & Deployment](#devops--deployment)

---

## Java Collections

### Temel Seviye

#### 1. Java Collections Framework'ün temel hiyerarşisini açıklayın.

**Cevap:**

**JavaScript'ten Java'ya Geçiş - Collections Framework**

**JavaScript'te Ne Kullanıyordun?**
```javascript
// JavaScript'te veri yapıları
let array = [1, 2, 3, 4, 5];           // Dizi
let set = new Set([1, 2, 3, 4, 5]);    // Benzersiz değerler
let map = new Map([['key1', 'value1']]); // Anahtar-değer çiftleri
let object = {name: 'Ahmet', age: 25};  // Obje
```

**Java'da Ne Kullanacaksın?**
```java
// Java'da Collections Framework
List<Integer> list = new ArrayList<>();        // JavaScript array gibi
Set<Integer> set = new HashSet<>();            // JavaScript Set gibi
Map<String, String> map = new HashMap<>();     // JavaScript Map gibi
// Java'da obje yerine class kullanılır
```

**Collections Framework Nedir?**
Java Collections Framework, JavaScript'teki array, Set, Map gibi veri yapılarının Java karşılığıdır. Ancak Java'da daha organize ve tip güvenli (type-safe) bir yapı vardır.

**JavaScript vs Java Karşılaştırması:**

| JavaScript | Java | Açıklama |
|------------|------|----------|
| `let arr = [1,2,3]` | `List<Integer> list = new ArrayList<>()` | Dizi/Liste |
| `new Set([1,2,3])` | `Set<Integer> set = new HashSet<>()` | Benzersiz değerler |
| `new Map()` | `Map<String, String> map = new HashMap<>()` | Anahtar-değer |
| `{name: 'Ahmet'}` | `Person person = new Person("Ahmet")` | Obje/Class |

**Neden Gereklidir?**
- **Tip Güvenliği**: JavaScript'te `let arr = [1, "hello", true]` yapabilirsin, Java'da `List<Integer>` sadece sayı alır
- **Performans**: Java'da daha optimize edilmiş veri yapıları
- **Organizasyon**: JavaScript'teki dağınık yapı yerine organize sınıflar

**Temel Hiyerarşi - JavaScript'ten Java'ya:**

```
Collection Interface (JavaScript'teki Array, Set'in atası)
├── List Interface (JavaScript Array gibi)
│   ├── ArrayList (JavaScript Array'e en yakın)
│   ├── LinkedList (JavaScript Array ama farklı iç yapı)
│   └── Vector (Eski ArrayList - thread-safe)
├── Set Interface (JavaScript Set gibi)
│   ├── HashSet (JavaScript Set'e en yakın)
│   ├── LinkedHashSet (JavaScript Set + sıralama)
│   └── TreeSet (JavaScript Set + otomatik sıralama)
└── Queue Interface (JavaScript'te yok - yeni kavram)
    ├── PriorityQueue (Öncelikli kuyruk)
    └── Deque Interface (Çift yönlü kuyruk)
        └── ArrayDeque (Dizi tabanlı deque)

Map Interface (JavaScript Map gibi - Collection'dan bağımsız)
├── HashMap (JavaScript Map'e en yakın)
├── LinkedHashMap (JavaScript Map + sıralama)
├── TreeMap (JavaScript Map + otomatik sıralama)
└── Hashtable (Eski HashMap - thread-safe)
```

**JavaScript'ten Java'ya Geçiş Rehberi:**

**1. List Interface - JavaScript Array'in Java Karşılığı:**

**JavaScript:**
```javascript
let meyveler = ["elma", "armut", "elma"];  // Duplicate var
console.log(meyveler[0]);                  // "elma" - index ile erişim
meyveler.push("muz");                      // Sonuna ekle
meyveler.length;                           // 4 - uzunluk
```

**Java:**
```java
List<String> meyveler = new ArrayList<>();
meyveler.add("elma");                      // Duplicate var
meyveler.add("armut");
meyveler.add("elma");
System.out.println(meyveler.get(0));       // "elma" - index ile erişim
meyveler.add("muz");                       // Sonuna ekle
meyveler.size();                           // 4 - uzunluk
```

**2. Set Interface - JavaScript Set'in Java Karşılığı:**

**JavaScript:**
```javascript
let benzersizMeyveler = new Set(["elma", "armut", "elma"]);
console.log(benzersizMeyveler);            // Set(2) {"elma", "armut"} - duplicate yok
benzersizMeyveler.add("muz");
benzersizMeyveler.has("elma");             // true - var mı kontrolü
```

**Java:**
```java
Set<String> benzersizMeyveler = new HashSet<>();
benzersizMeyveler.add("elma");
benzersizMeyveler.add("armut");
benzersizMeyveler.add("elma");             // Duplicate eklenmez
System.out.println(benzersizMeyveler);     // [elma, armut] - duplicate yok
benzersizMeyveler.add("muz");
benzersizMeyveler.contains("elma");        // true - var mı kontrolü
```

**3. Map Interface - JavaScript Map'in Java Karşılığı:**

**JavaScript:**
```javascript
let kisi = new Map();
kisi.set("isim", "Ahmet");
kisi.set("yas", 25);
kisi.set("sehir", "Istanbul");
console.log(kisi.get("isim"));             // "Ahmet"
console.log(kisi.has("yas"));              // true
```

**Java:**
```java
Map<String, Object> kisi = new HashMap<>();
kisi.put("isim", "Ahmet");
kisi.put("yas", 25);
kisi.put("sehir", "Istanbul");
System.out.println(kisi.get("isim"));      // "Ahmet"
System.out.println(kisi.containsKey("yas")); // true
```

**4. Queue Interface - JavaScript'te Yok, Java'da Yeni:**

**JavaScript'te böyle bir yapı yok, ama Java'da var:**
```java
Queue<String> kuyruk = new LinkedList<>();
kuyruk.offer("birinci");                   // Kuyruğa ekle
kuyruk.offer("ikinci");
String ilk = kuyruk.poll();                // "birinci" - ilk giren ilk çıkar
String ikinci = kuyruk.poll();             // "ikinci"
```

**JavaScript'te Benzer Yapı (Manuel):**
```javascript
let kuyruk = [];
kuyruk.push("birinci");                    // Sonuna ekle
kuyruk.push("ikinci");
let ilk = kuyruk.shift();                  // "birinci" - baştan çıkar
let ikinci = kuyruk.shift();               // "ikinci"
```

**Temel Farklar:**

| Özellik | JavaScript | Java |
|---------|------------|------|
| **Tip Güvenliği** | `let arr = [1, "hello", true]` | `List<Integer> list` - sadece sayı |
| **Null/Undefined** | `undefined` | `null` |
| **Length/Size** | `array.length` | `list.size()` |
| **Add/Push** | `array.push(item)` | `list.add(item)` |
| **Get/Index** | `array[0]` | `list.get(0)` |
| **Has/Contains** | `set.has(item)` | `set.contains(item)` |
| **Map Get** | `map.get(key)` | `map.get(key)` |
| **Map Set** | `map.set(key, value)` | `map.put(key, value)` |

**Gerçek Hayat Örnekleri:**

**JavaScript'te:**
```javascript
// Telefon rehberi
let rehber = ["Ahmet", "Mehmet", "Ahmet"];

// Benzersiz öğrenci listesi
let ogrenciler = new Set(["Ali", "Veli", "Ali"]);

// Kullanıcı profili
let kullanici = new Map();
kullanici.set("isim", "Ahmet");
kullanici.set("email", "ahmet@email.com");
```

**Java'da:**
```java
// Telefon rehberi
List<String> rehber = new ArrayList<>();
rehber.add("Ahmet");
rehber.add("Mehmet");
rehber.add("Ahmet");

// Benzersiz öğrenci listesi
Set<String> ogrenciler = new HashSet<>();
ogrenciler.add("Ali");
ogrenciler.add("Veli");
ogrenciler.add("Ali"); // Eklenmez

// Kullanıcı profili
Map<String, String> kullanici = new HashMap<>();
kullanici.put("isim", "Ahmet");
kullanici.put("email", "ahmet@email.com");
```

**Özet:**
- **List**: JavaScript Array'in Java karşılığı
- **Set**: JavaScript Set'in Java karşılığı  
- **Map**: JavaScript Map'in Java karşılığı
- **Queue**: JavaScript'te yok, Java'da yeni kavram
- **Tip Güvenliği**: Java'da daha katı tip kontrolü
- **Metod İsimleri**: JavaScript'te `push()`, Java'da `add()`

#### 2. List, Set ve Map arasındaki farklar nelerdir?

**Cevap:**

**JavaScript'ten Java'ya Geçiş - List, Set, Map Farkları**

**JavaScript'te Ne Kullanıyordun?**
```javascript
// JavaScript'te hepsi farklı ama benzer görünüyor
let array = [1, 2, 3, 2];                    // Tekrarlı, sıralı
let set = new Set([1, 2, 3, 2]);             // Benzersiz, sıralama yok
let map = new Map([['a', 1], ['b', 2]]);     // Anahtar-değer
let object = {a: 1, b: 2};                   // Obje (Map'e benzer)
```

**Java'da Ne Kullanacaksın?**
```java
// Java'da daha organize ve tip güvenli
List<Integer> list = new ArrayList<>();      // JavaScript Array gibi
Set<Integer> set = new HashSet<>();          // JavaScript Set gibi
Map<String, Integer> map = new HashMap<>();  // JavaScript Map gibi
// Java'da obje yerine class kullanılır
```

**Temel Farklar Tablosu:**

| Özellik | List (Array) | Set | Map |
|---------|--------------|-----|-----|
| **Tekrarlı Veri** | ✅ İzin verir | ❌ İzin vermez | 🔑 Anahtar unique, değer tekrarlanabilir |
| **Sıralama** | ✅ Ekleme sırasını korur | ❌ Garantisi yok (TreeSet hariç) | ❌ Garantisi yok (LinkedHashMap hariç) |
| **Index Erişimi** | ✅ Var (0, 1, 2...) | ❌ Yok | ❌ Anahtar ile erişim |
| **Null Değer** | ✅ İzin verir | ⚠️ Bir tane (TreeSet hariç) | ✅ Anahtar ve değer null olabilir |
| **Kullanım Amacı** | Sıralı veri listesi | Benzersiz veri kümesi | Anahtar-değer eşleştirmesi |

**Detaylı Açıklamalar - JavaScript'ten Java'ya:**

**1. List (Liste) - JavaScript Array'in Java Karşılığı:**

**JavaScript'te:**
```javascript
// JavaScript Array - telefon rehberi gibi
let telefonRehberi = ["Ahmet", "Mehmet", "Ahmet", "Ayşe"];
console.log(telefonRehberi[0]);        // "Ahmet" - index ile erişim
console.log(telefonRehberi.length);    // 4 - uzunluk
telefonRehberi.push("Ali");            // Sonuna ekle
telefonRehberi[1] = "Mehmet Yeni";     // Index ile değiştir
```

**Java'da:**
```java
// Java List - telefon rehberi gibi
List<String> telefonRehberi = new ArrayList<>();
telefonRehberi.add("Ahmet");           // Index: 0
telefonRehberi.add("Mehmet");          // Index: 1
telefonRehberi.add("Ahmet");           // Index: 2 - Aynı isim tekrar eklenebilir
telefonRehberi.add("Ayşe");            // Index: 3

System.out.println(telefonRehberi);    // [Ahmet, Mehmet, Ahmet, Ayşe]
System.out.println(telefonRehberi.get(0)); // "Ahmet" - index ile erişim
System.out.println(telefonRehberi.size()); // 4 - uzunluk
telefonRehberi.add("Ali");             // Sonuna ekle
telefonRehberi.set(1, "Mehmet Yeni");  // Index ile değiştir
```

**List'in Özellikleri (JavaScript Array'e Benzer):**
- ✅ Aynı değeri birden fazla kez ekleyebilir (JavaScript Array gibi)
- ✅ Ekleme sırasını korur (JavaScript Array gibi)
- ✅ Index ile erişim (0, 1, 2, 3...) - JavaScript Array gibi
- ✅ Null değer eklenebilir (JavaScript'te undefined gibi)
- ✅ Iterator ile tüm elemanları gezebilir (JavaScript for...of gibi)

**2. Set (Küme) - JavaScript Set'in Java Karşılığı:**

**JavaScript'te:**
```javascript
// JavaScript Set - öğrenci listesi gibi
let ogrenciListesi = new Set(["Ahmet", "Mehmet", "Ahmet", "Ayşe"]);
console.log(ogrenciListesi);           // Set(3) {"Ahmet", "Mehmet", "Ayşe"}
console.log(ogrenciListesi.has("Ahmet")); // true - var mı kontrolü
ogrenciListesi.add("Ali");             // Ekle
ogrenciListesi.delete("Mehmet");       // Sil
// ogrenciListesi[0]; // HATA! - index ile erişim yok
```

**Java'da:**
```java
// Java Set - öğrenci listesi gibi
Set<String> ogrenciListesi = new HashSet<>();
ogrenciListesi.add("Ahmet");           // Eklendi
ogrenciListesi.add("Mehmet");          // Eklendi
ogrenciListesi.add("Ahmet");           // EKLENMEDİ - Zaten var!
ogrenciListesi.add("Ayşe");            // Eklendi

System.out.println(ogrenciListesi);    // [Ahmet, Mehmet, Ayşe] - Sadece 3 eleman
System.out.println(ogrenciListesi.contains("Ahmet")); // true - var mı kontrolü
ogrenciListesi.add("Ali");             // Ekle
ogrenciListesi.remove("Mehmet");       // Sil
// ogrenciListesi.get(1); // HATA! - index ile erişim yok
```

**Set'in Özellikleri (JavaScript Set'e Benzer):**
- ❌ Aynı değeri sadece bir kez ekler (JavaScript Set gibi)
- ❌ Index ile erişim yok (JavaScript Set gibi)
- ❌ Sıralama garantisi yok (TreeSet hariç)
- ✅ Hızlı arama (contains() metodu - JavaScript has() gibi)
- ✅ Null değer eklenebilir (bir tane)

**3. Map (Harita) - JavaScript Map'in Java Karşılığı:**

**JavaScript'te:**
```javascript
// JavaScript Map - sözlük gibi
let sozluk = new Map();
sozluk.set("kitap", "book");           // Anahtar: "kitap", Değer: "book"
sozluk.set("kalem", "pen");            // Anahtar: "kalem", Değer: "pen"
sozluk.set("masa", "table");           // Anahtar: "masa", Değer: "table"
sozluk.set("kitap", "book");           // Aynı anahtar - değer güncellenir

console.log(sozluk);                   // Map(3) {"kitap" => "book", "kalem" => "pen", "masa" => "table"}
console.log(sozluk.get("kitap"));      // "book" - anahtar ile değer al
console.log(sozluk.has("kalem"));      // true - anahtar var mı?
console.log(sozluk.size);              // 3 - toplam çift sayısı
```

**Java'da:**
```java
// Java Map - sözlük gibi
Map<String, String> sozluk = new HashMap<>();
sozluk.put("kitap", "book");           // Anahtar: "kitap", Değer: "book"
sozluk.put("kalem", "pen");            // Anahtar: "kalem", Değer: "pen"
sozluk.put("masa", "table");           // Anahtar: "masa", Değer: "table"
sozluk.put("kitap", "book");           // Aynı anahtar - değer güncellenir

System.out.println(sozluk);            // {kitap=book, kalem=pen, masa=table}
System.out.println(sozluk.get("kitap")); // "book" - anahtar ile değer al
System.out.println(sozluk.containsKey("kalem")); // true - anahtar var mı?
System.out.println(sozluk.size());     // 3 - toplam çift sayısı
```

**Map'in Özellikleri (JavaScript Map'e Benzer):**
- 🔑 Anahtar benzersiz olmalı, değer tekrarlanabilir (JavaScript Map gibi)
- 🔑 Anahtar ile değere erişim (JavaScript Map gibi)
- ❌ Index ile erişim yok (JavaScript Map gibi)
- ❌ Sıralama garantisi yok (TreeMap hariç)
- ✅ Hızlı anahtar-değer arama (JavaScript Map gibi)

**Gerçek Hayat Örnekleri - JavaScript'ten Java'ya:**

**1. List Kullanım Alanları (JavaScript Array'e Benzer):**

**JavaScript'te:**
```javascript
// Telefon rehberi - aynı isim birden fazla numara
let rehber = ["Ahmet: 555-1234", "Mehmet: 555-5678", "Ahmet: 555-9999"];

// Alışveriş listesi - aynı ürün birden fazla kez
let alisveris = ["elma", "süt", "elma", "ekmek"];

// Müzik çalma listesi - aynı şarkı tekrar edebilir
let playlist = ["Şarkı1", "Şarkı2", "Şarkı1", "Şarkı3"];
```

**Java'da:**
```java
// Telefon rehberi - aynı isim birden fazla numara
List<String> rehber = new ArrayList<>();
rehber.add("Ahmet: 555-1234");
rehber.add("Mehmet: 555-5678");
rehber.add("Ahmet: 555-9999");

// Alışveriş listesi - aynı ürün birden fazla kez
List<String> alisveris = new ArrayList<>();
alisveris.add("elma");
alisveris.add("süt");
alisveris.add("elma");
alisveris.add("ekmek");

// Müzik çalma listesi - aynı şarkı tekrar edebilir
List<String> playlist = new ArrayList<>();
playlist.add("Şarkı1");
playlist.add("Şarkı2");
playlist.add("Şarkı1");
playlist.add("Şarkı3");
```

**2. Set Kullanım Alanları (JavaScript Set'e Benzer):**

**JavaScript'te:**
```javascript
// Öğrenci listesi - her öğrenci sadece bir kez
let ogrenciler = new Set(["Ahmet", "Mehmet", "Ahmet", "Ayşe"]);

// Etiket sistemi - benzersiz etiketler
let etiketler = new Set(["javascript", "java", "python", "javascript"]);

// Arama sonuçları - tekrarlı sonuçları filtrele
let aramaSonuclari = new Set(["sonuç1", "sonuç2", "sonuç1", "sonuç3"]);
```

**Java'da:**
```java
// Öğrenci listesi - her öğrenci sadece bir kez
Set<String> ogrenciler = new HashSet<>();
ogrenciler.add("Ahmet");
ogrenciler.add("Mehmet");
ogrenciler.add("Ahmet"); // Eklenmez
ogrenciler.add("Ayşe");

// Etiket sistemi - benzersiz etiketler
Set<String> etiketler = new HashSet<>();
etiketler.add("javascript");
etiketler.add("java");
etiketler.add("python");
etiketler.add("javascript"); // Eklenmez

// Arama sonuçları - tekrarlı sonuçları filtrele
Set<String> aramaSonuclari = new HashSet<>();
aramaSonuclari.add("sonuç1");
aramaSonuclari.add("sonuç2");
aramaSonuclari.add("sonuç1"); // Eklenmez
aramaSonuclari.add("sonuç3");
```

**3. Map Kullanım Alanları (JavaScript Map'e Benzer):**

**JavaScript'te:**
```javascript
// Sözlük - kelime-anlam
let sozluk = new Map();
sozluk.set("kitap", "book");
sozluk.set("kalem", "pen");

// Kullanıcı profilleri - ID-bilgi
let kullanicilar = new Map();
kullanicilar.set("user1", {name: "Ahmet", email: "ahmet@email.com"});
kullanicilar.set("user2", {name: "Mehmet", email: "mehmet@email.com"});

// Konfigürasyon ayarları - anahtar-değer
let ayarlar = new Map();
ayarlar.set("database.url", "localhost:3306");
ayarlar.set("server.port", "8080");
```

**Java'da:**
```java
// Sözlük - kelime-anlam
Map<String, String> sozluk = new HashMap<>();
sozluk.put("kitap", "book");
sozluk.put("kalem", "pen");

// Kullanıcı profilleri - ID-bilgi
Map<String, User> kullanicilar = new HashMap<>();
kullanicilar.put("user1", new User("Ahmet", "ahmet@email.com"));
kullanicilar.put("user2", new User("Mehmet", "mehmet@email.com"));

// Konfigürasyon ayarları - anahtar-değer
Map<String, String> ayarlar = new HashMap<>();
ayarlar.put("database.url", "localhost:3306");
ayarlar.put("server.port", "8080");
```

**Hangi Durumda Hangisini Kullanmalı?**

**List Kullan Eğer (JavaScript Array gibi):**
- Verilerin sırası önemliyse
- Aynı değeri birden fazla kez saklaman gerekiyorsa
- Index ile erişim yapman gerekiyorsa
- JavaScript'teki array gibi davranmasını istiyorsan

**Set Kullan Eğer (JavaScript Set gibi):**
- Sadece benzersiz değerler saklaman gerekiyorsa
- Hızlı arama yapman gerekiyorsa
- Duplicate verileri otomatik filtrelemek istiyorsan
- JavaScript'teki Set gibi davranmasını istiyorsan

**Map Kullan Eğer (JavaScript Map gibi):**
- Anahtar-değer ilişkisi kurman gerekiyorsa
- Anahtar ile hızlı değer araması yapman gerekiyorsa
- İlişkili verileri saklaman gerekiyorsa
- JavaScript'teki Map gibi davranmasını istiyorsan

**JavaScript'ten Java'ya Geçiş Özeti:**

| JavaScript | Java | Kullanım Amacı |
|------------|------|----------------|
| `let arr = [1,2,3]` | `List<Integer> list = new ArrayList<>()` | Sıralı, tekrarlı veri |
| `new Set([1,2,3])` | `Set<Integer> set = new HashSet<>()` | Benzersiz veri |
| `new Map()` | `Map<String, String> map = new HashMap<>()` | Anahtar-değer çiftleri |
| `array.push(item)` | `list.add(item)` | Eleman ekleme |
| `array.length` | `list.size()` | Uzunluk |
| `array[0]` | `list.get(0)` | Index ile erişim |
| `set.has(item)` | `set.contains(item)` | Var mı kontrolü |
| `map.get(key)` | `map.get(key)` | Değer alma |
| `map.set(key, value)` | `map.put(key, value)` | Değer ekleme |

// Map - Key-value pairs
Map<String, Integer> map = new HashMap<>();
map.put("Java", 1);
map.put("Python", 2);
map.put("Java", 3); // Overwrites previous value
System.out.println(map); // {Java=3, Python=2}
```

#### 3. ArrayList ve LinkedList arasındaki farklar nelerdir?

**Cevap:**

**JavaScript'ten Java'ya Geçiş - ArrayList vs LinkedList**

**JavaScript'te Ne Kullanıyordun?**
```javascript
// JavaScript'te sadece Array var, farklı türleri yok
let array = [1, 2, 3, 4, 5];
array.push(6);                    // Sonuna ekle - hızlı
array.unshift(0);                 // Başına ekle - yavaş (tüm elemanlar kayar)
array[2];                         // Index ile erişim - hızlı
array.splice(2, 0, 2.5);          // Ortaya ekle - yavaş (elemanlar kayar)
```

**Java'da İki Seçeneğin Var:**
```java
// 1. ArrayList - JavaScript Array'e daha yakın
List<Integer> arrayList = new ArrayList<>();
arrayList.add(6);                 // Sonuna ekle - hızlı
arrayList.add(0, 0);              // Başına ekle - yavaş
arrayList.get(2);                 // Index ile erişim - hızlı
arrayList.add(2, 2);              // Ortaya ekle - yavaş

// 2. LinkedList - JavaScript Array'den farklı
List<Integer> linkedList = new LinkedList<>();
linkedList.add(6);                // Sonuna ekle - hızlı
linkedList.add(0, 0);             // Başına ekle - hızlı!
linkedList.get(2);                // Index ile erişim - yavaş
linkedList.add(2, 2);             // Ortaya ekle - hızlı!
```

**Temel Farklar Tablosu:**

| Özellik | ArrayList | LinkedList | JavaScript Array |
|---------|-----------|------------|------------------|
| **İç Yapı** | Dinamik dizi (array) | Çift yönlü bağlı liste | Dinamik dizi |
| **Bellek Yerleşimi** | Ardışık bellek | Dağınık bellek | Ardışık bellek |
| **Rastgele Erişim** | O(1) - Çok hızlı | O(n) - Yavaş | O(1) - Çok hızlı |
| **Ekleme/Silme** | O(n) - Yavaş | O(1) - Hızlı (konum biliniyorsa) | O(n) - Yavaş |
| **Bellek Kullanımı** | Düşük | Yüksek (işaretçiler) | Düşük |
| **Önbellek Performansı** | İyi | Kötü | İyi |

**Detaylı Açıklamalar - JavaScript'ten Java'ya:**

**1. ArrayList - JavaScript Array'e En Yakın:**

**JavaScript'te Nasıl Çalışır?**
```javascript
// JavaScript Array - dinamik dizi
let array = [1, 2, 3, 4, 5];
// Bellekte: [1][2][3][4][5] (ardışık)
array[2];                    // Direkt erişim - hızlı
array.push(6);               // Sonuna ekle - hızlı
array.unshift(0);            // Başına ekle - yavaş (tüm elemanlar kayar)
array.splice(2, 0, 2.5);     // Ortaya ekle - yavaş (elemanlar kayar)
```

**Java ArrayList'te Nasıl Çalışır?**
```java
// Java ArrayList - JavaScript Array'e benzer
List<Integer> arrayList = new ArrayList<>();
arrayList.add(1); arrayList.add(2); arrayList.add(3);
// Bellekte: [1][2][3] (ardışık)
arrayList.get(2);            // Direkt erişim - hızlı (JavaScript array[2] gibi)
arrayList.add(6);            // Sonuna ekle - hızlı (JavaScript push() gibi)
arrayList.add(0, 0);         // Başına ekle - yavaş (JavaScript unshift() gibi)
arrayList.add(2, 2);         // Ortaya ekle - yavaş (JavaScript splice() gibi)
```

**ArrayList'in İç Yapısı:**
```java
// ArrayList'in iç yapısı (basitleştirilmiş)
class ArrayList {
    private Object[] array;  // Gerçek veriler burada (JavaScript Array gibi)
    private int size;        // Kaç eleman var (JavaScript length gibi)
    private int capacity;    // Dizinin boyutu (JavaScript'te otomatik)
}
```

**ArrayList'in Avantajları (JavaScript Array'e Benzer):**
- ✅ **Hızlı Erişim**: Index ile direkt erişim (JavaScript array[0] gibi)
- ✅ **Bellek Verimli**: Sadece veri saklar, ekstra işaretçi yok
- ✅ **Önbellek Dostu**: Ardışık bellek kullanır (JavaScript Array gibi)
- ✅ **Basit Yapı**: JavaScript Array gibi anlaşılması kolay

**ArrayList'in Dezavantajları (JavaScript Array'e Benzer):**
- ❌ **Yavaş Ekleme/Silme**: Ortaya ekleme/silme tüm elemanları kaydırır
- ❌ **Bellek Taşması**: Dizi dolduğunda yeni dizi oluşturur
- ❌ **Sabit Başlangıç Boyutu**: Başlangıçta bellek rezerve eder

**2. LinkedList - JavaScript'te Yok, Java'da Yeni:**

**JavaScript'te Böyle Bir Yapı Yok:**
```javascript
// JavaScript'te sadece Array var, LinkedList yok
let array = [1, 2, 3, 4, 5];
// Tüm işlemler Array üzerinden yapılır
```

**Java LinkedList'te Nasıl Çalışır?**
```java
// Java LinkedList - JavaScript'te olmayan yapı
List<Integer> linkedList = new LinkedList<>();
linkedList.add(1); linkedList.add(2); linkedList.add(3);
// Bellekte: [1] -> [2] -> [3] -> null (dağınık)
linkedList.get(2);            // Yavaş (tüm liste gezilir)
linkedList.add(6);            // Sonuna ekle - hızlı
linkedList.add(0, 0);         // Başına ekle - hızlı! (JavaScript'te yavaş)
linkedList.add(2, 2);         // Ortaya ekle - hızlı! (JavaScript'te yavaş)
```

**LinkedList'in İç Yapısı:**
```java
// LinkedList'in iç yapısı (basitleştirilmiş)
class Node {
    Object data;        // Veri
    Node next;          // Sonraki eleman
    Node previous;      // Önceki eleman (doubly linked)
}

class LinkedList {
    private Node head;  // İlk eleman
    private Node tail;  // Son eleman
    private int size;   // Eleman sayısı
}
```

**LinkedList'in Avantajları (JavaScript'te Yok):**
- ✅ **Hızlı Ekleme/Silme**: Başa/sona ekleme O(1) (JavaScript'te yavaş)
- ✅ **Dinamik Boyut**: Sadece gerektiği kadar bellek
- ✅ **Esnek Yapı**: Herhangi bir yere kolayca ekleme
- ✅ **Bellek Taşması Yok**: Her eleman ayrı ayrı oluşturulur

**LinkedList'in Dezavantajları:**
- ❌ **Yavaş Erişim**: Index ile erişim için tüm liste gezilir
- ❌ **Yüksek Bellek Kullanımı**: Her eleman için işaretçiler
- ❌ **Önbellek Dostu Değil**: Dağınık bellek kullanımı

**Görsel Karşılaştırma - JavaScript'ten Java'ya:**

**JavaScript Array & Java ArrayList - Dizi Yapısı:**
```
JavaScript:  [0]    [1]    [2]    [3]    [4]
             "A"    "B"    "C"    "D"    "E"
Bellek:      [A][B][C][D][E]  (ardışık)

Java ArrayList:  [0]    [1]    [2]    [3]    [4]
                  "A"    "B"    "C"    "D"    "E"
Bellek:           [A][B][C][D][E]  (ardışık)
```

**Java LinkedList - Bağlı Yapı (JavaScript'te Yok):**
```
[A] -> [B] -> [C] -> [D] -> [E] -> null
 ↑      ↑      ↑      ↑      ↑
data   data   data   data   data
next   next   next   next   next
```

**Performans Karşılaştırması - JavaScript vs Java:**

**1. Erişim (Access) - Index ile veri alma:**

**JavaScript Array:**
```javascript
// JavaScript Array - Çok hızlı
let array = ["A", "B", "C"];
let value = array[1]; // O(1) - Direkt erişim
```

**Java ArrayList:**
```java
// Java ArrayList - JavaScript Array gibi hızlı
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("A"); arrayList.add("B"); arrayList.add("C");
String value = arrayList.get(1); // O(1) - Direkt erişim (JavaScript array[1] gibi)
```

**Java LinkedList:**
```java
// Java LinkedList - Yavaş (JavaScript'te böyle bir yapı yok)
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A"); linkedList.add("B"); linkedList.add("C");
String value = linkedList.get(1); // O(n) - Tüm liste gezilir
```

**2. Ekleme (Insertion) - Yeni veri ekleme:**

**JavaScript Array:**
```javascript
// JavaScript Array - Ortaya ekleme yavaş
let array = ["A", "B", "C"];
array.splice(1, 0, "X"); // O(n) - B ve C kaydırılır
```

**Java ArrayList:**
```java
// Java ArrayList - JavaScript Array gibi yavaş
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("A"); arrayList.add("B"); arrayList.add("C");
arrayList.add(1, "X"); // O(n) - B ve C kaydırılır (JavaScript splice() gibi)
```

**Java LinkedList:**
```java
// Java LinkedList - Hızlı (JavaScript'te böyle bir yapı yok)
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A"); linkedList.add("B"); linkedList.add("C");
linkedList.add(1, "X"); // O(1) - Sadece işaretçiler değişir
```

**3. Silme (Deletion) - Veri silme:**

**JavaScript Array:**
```javascript
// JavaScript Array - Ortadan silme yavaş
let array = ["A", "B", "C"];
array.splice(1, 1); // O(n) - C sola kaydırılır
```

**Java ArrayList:**
```java
// Java ArrayList - JavaScript Array gibi yavaş
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("A"); arrayList.add("B"); arrayList.add("C");
arrayList.remove(1); // O(n) - C sola kaydırılır (JavaScript splice() gibi)
```

**Java LinkedList:**
```java
// Java LinkedList - Hızlı (JavaScript'te böyle bir yapı yok)
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A"); linkedList.add("B"); linkedList.add("C");
linkedList.remove(1); // O(1) - Sadece işaretçiler değişir
```

**Gerçek Hayat Örnekleri - JavaScript'ten Java'ya:**

**1. ArrayList Kullan Eğer (JavaScript Array gibi):**

**JavaScript'te:**
```javascript
// Veri analizi - büyük veri setlerinde hızlı erişim
let veriler = [100, 200, 300, 400, 500];
let ortalama = veriler[2]; // Hızlı erişim

// Oyun skorları - skor tablosunda hızlı erişim
let skorlar = [1000, 2000, 3000, 4000, 5000];
let enYuksekSkor = skorlar[4]; // Hızlı erişim

// Telefon rehberi - isme göre hızlı arama
let rehber = ["Ahmet", "Mehmet", "Ayşe", "Fatma"];
let kisi = rehber[1]; // Hızlı erişim
```

**Java'da:**
```java
// Veri analizi - büyük veri setlerinde hızlı erişim
List<Integer> veriler = new ArrayList<>();
veriler.add(100); veriler.add(200); veriler.add(300);
int ortalama = veriler.get(2); // Hızlı erişim (JavaScript array[2] gibi)

// Oyun skorları - skor tablosunda hızlı erişim
List<Integer> skorlar = new ArrayList<>();
skorlar.add(1000); skorlar.add(2000); skorlar.add(3000);
int enYuksekSkor = skorlar.get(2); // Hızlı erişim

// Telefon rehberi - isme göre hızlı arama
List<String> rehber = new ArrayList<>();
rehber.add("Ahmet"); rehber.add("Mehmet"); rehber.add("Ayşe");
String kisi = rehber.get(1); // Hızlı erişim
```

**2. LinkedList Kullan Eğer (JavaScript'te Yok, Java'da Yeni):**

**JavaScript'te Böyle Bir Yapı Yok:**
```javascript
// JavaScript'te sadece Array var, LinkedList yok
// Tüm işlemler Array üzerinden yapılır
let playlist = ["Şarkı1", "Şarkı2", "Şarkı3"];
playlist.unshift("YeniŞarkı"); // Yavaş (tüm elemanlar kayar)
```

**Java'da:**
```java
// Müzik çalma listesi - şarkı ekleme/çıkarma sık
List<String> playlist = new LinkedList<>();
playlist.add("Şarkı1"); playlist.add("Şarkı2"); playlist.add("Şarkı3");
playlist.add(0, "YeniŞarkı"); // Hızlı! (JavaScript'te yavaş)

// Undo/Redo sistemi - komut geçmişi yönetimi
List<String> komutlar = new LinkedList<>();
komutlar.add("Komut1"); komutlar.add("Komut2");
komutlar.add(0, "YeniKomut"); // Hızlı!

// Chat mesajları - yeni mesaj ekleme sık
List<String> mesajlar = new LinkedList<>();
mesajlar.add("Mesaj1"); mesajlar.add("Mesaj2");
mesajlar.add(0, "YeniMesaj"); // Hızlı!
```

**Hangi Durumda Hangisini Seçmeli?**

**ArrayList Seç Eğer (JavaScript Array gibi):**
- Verilere sık erişim yapıyorsan (okuma > yazma)
- Bellek kullanımı önemliyse
- Veri seti büyükse
- Index ile erişim yapıyorsan
- JavaScript Array gibi davranmasını istiyorsan

**LinkedList Seç Eğer (JavaScript'te Yok):**
- Sık ekleme/silme yapıyorsan (yazma > okuma)
- Veri seti küçükse
- Bellek kullanımı önemli değilse
- Sıralı işlem yapıyorsan
- JavaScript'te olmayan hızlı ekleme/silme istiyorsan

**JavaScript'ten Java'ya Geçiş Özeti:**

| Özellik | JavaScript Array | Java ArrayList | Java LinkedList |
|---------|------------------|----------------|-----------------|
| **İç Yapı** | Dinamik dizi | Dinamik dizi | Bağlı liste |
| **Erişim** | O(1) - Hızlı | O(1) - Hızlı | O(n) - Yavaş |
| **Ekleme/Silme** | O(n) - Yavaş | O(n) - Yavaş | O(1) - Hızlı |
| **Bellek** | Düşük | Düşük | Yüksek |
| **JavaScript'e Benzerlik** | ✅ Aynı | ✅ Çok benzer | ❌ Farklı |

**Özet:**
- **JavaScript Array**: "Hızlı erişim, yavaş değişim" - Dizi gibi
- **Java ArrayList**: "Hızlı erişim, yavaş değişim" - JavaScript Array'e çok benzer
- **Java LinkedList**: "Yavaş erişim, hızlı değişim" - JavaScript'te yok, Java'da yeni

**Kod Örneği:**
```java
// ArrayList - Random access hızlı
ArrayList<Integer> arrayList = new ArrayList<>();
for (int i = 0; i < 1000000; i++) {
    arrayList.add(i);
}
long start = System.currentTimeMillis();
int value = arrayList.get(500000); // O(1) - Very fast
long end = System.currentTimeMillis();
System.out.println("ArrayList access time: " + (end - start) + "ms");

// LinkedList - Random access yavaş
LinkedList<Integer> linkedList = new LinkedList<>();
for (int i = 0; i < 1000000; i++) {
    linkedList.add(i);
}
start = System.currentTimeMillis();
value = linkedList.get(500000); // O(n) - Slow
end = System.currentTimeMillis();
System.out.println("LinkedList access time: " + (end - start) + "ms");
```

#### 4. HashMap'in çalışma prensibi nasıldır?

**Cevap:**

**JavaScript'ten Java'ya Geçiş - HashMap**

**JavaScript'te Ne Kullanıyordun?**
```javascript
// JavaScript'te Map ve Object var
let map = new Map();
map.set("isim", "Ahmet");
map.set("yas", 25);
console.log(map.get("isim")); // "Ahmet"

// Veya Object kullanabilirsin
let obj = {
    isim: "Ahmet",
    yas: 25
};
console.log(obj.isim); // "Ahmet"
console.log(obj["yas"]); // 25
```

**Java'da Ne Kullanacaksın?**
```java
// Java'da HashMap - JavaScript Map'e çok benzer
Map<String, Object> map = new HashMap<>();
map.put("isim", "Ahmet");
map.put("yas", 25);
System.out.println(map.get("isim")); // "Ahmet"

// Java'da Object yerine class kullanılır
Person person = new Person("Ahmet", 25);
System.out.println(person.getIsim()); // "Ahmet"
```

**HashMap Nedir?**
HashMap, Java'da anahtar-değer çiftlerini saklamak için kullanılan en popüler veri yapısıdır. JavaScript'teki Map'e çok benzer, ancak daha optimize edilmiş ve tip güvenli.

**Temel Kavramlar:**

**1. Hash (Özet) Fonksiyonu:**
- Her anahtar için benzersiz bir sayı üretir
- Bu sayı "hash code" olarak adlandırılır
- Aynı anahtar her zaman aynı hash code'u verir
- JavaScript'te de benzer şekilde çalışır

**2. Bucket (Kova) Sistemi:**
- HashMap içinde birçok "kova" vardır
- Her kova bir dizi elemanıdır
- Anahtarlar hash code'a göre kovalara dağıtılır
- JavaScript Map'te de benzer optimizasyon var

**3. Collision (Çakışma):**
- İki farklı anahtar aynı hash code'a sahip olabilir
- Bu durumda aynı kovaya düşerler
- HashMap bu durumu çözer
- JavaScript Map'te de benzer durumlar olabilir

**Çalışma Prensibi - JavaScript'ten Java'ya:**

**JavaScript'te Nasıl Çalışır?**
```javascript
// JavaScript Map - basit kullanım
let sozluk = new Map();
sozluk.set("kitap", "book");
let deger = sozluk.get("kitap"); // "book"
```

**Java'da Nasıl Çalışır?**
```java
// Java HashMap - JavaScript Map'e benzer ama daha optimize
Map<String, String> sozluk = new HashMap<>();
sozluk.put("kitap", "book");
String deger = sozluk.get("kitap"); // "book"
```

**Adım 1: Anahtar Ekleme (put) - JavaScript vs Java**

**JavaScript'te:**
```javascript
let sozluk = new Map();
sozluk.set("kitap", "book");
```

**Ne Olur?**
1. "kitap" anahtarının hash code'u hesaplanır (JavaScript engine tarafından)
2. Bu hash code kullanılarak hangi kovaya gideceği belirlenir
3. Anahtar-değer çifti o kovaya eklenir

**Java'da:**
```java
Map<String, String> sozluk = new HashMap<>();
sozluk.put("kitap", "book");
```

**Ne Olur?**
1. "kitap" anahtarının hash code'u hesaplanır (String.hashCode() metodu)
2. Bu hash code kullanılarak hangi kovaya gideceği belirlenir
3. Anahtar-değer çifti o kovaya eklenir

**Adım 2: Anahtar Arama (get) - JavaScript vs Java**

**JavaScript'te:**
```javascript
let deger = sozluk.get("kitap");
```

**Ne Olur?**
1. "kitap" anahtarının hash code'u hesaplanır
2. Bu hash code ile hangi kovada olduğu bulunur
3. O kovada "kitap" anahtarı aranır
4. Bulunursa değeri döndürülür

**Java'da:**
```java
String deger = sozluk.get("kitap");
```

**Ne Olur?**
1. "kitap" anahtarının hash code'u hesaplanır
2. Bu hash code ile hangi kovada olduğu bulunur
3. O kovada "kitap" anahtarı aranır
4. Bulunursa değeri döndürülür

**Görsel Açıklama - JavaScript'ten Java'ya:**

**JavaScript Map vs Java HashMap İç Yapısı:**

**JavaScript Map (Basitleştirilmiş):**
```
Bucket 0: [null]
Bucket 1: [null]
Bucket 2: [("kitap", "book")]  <- "kitap" buraya düştü
Bucket 3: [null]
Bucket 4: [("kalem", "pen")]   <- "kalem" buraya düştü
Bucket 5: [null]
...
```

**Java HashMap (Daha Optimize):**
```
Bucket 0: [null]
Bucket 1: [null]
Bucket 2: [("kitap", "book")]  <- "kitap" buraya düştü
Bucket 3: [null]
Bucket 4: [("kalem", "pen")]   <- "kalem" buraya düştü
Bucket 5: [null]
...
```

**Hash Code Hesaplama - JavaScript vs Java:**

**JavaScript'te:**
```javascript
let anahtar = "kitap";
// JavaScript engine otomatik olarak hash code hesaplar
// Sen bunu göremezsin, engine içinde yapılır
```

**Java'da:**
```java
String anahtar = "kitap";
int hashCode = anahtar.hashCode(); // Örnek: 12345
int bucketIndex = hashCode % bucketSayisi; // Örnek: 12345 % 16 = 9
// "kitap" 9. kovaya gider
```

**Collision (Çakışma) Yönetimi - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript Map'te de collision olabilir
let sozluk = new Map();
sozluk.set("kitap", "book");    // Kova 2'ye gider
sozluk.set("masa", "table");    // Kova 2'ye gider (çakışma!)
// JavaScript engine otomatik olarak çözer
```

**Java'da:**
```java
// Java HashMap'te collision yönetimi daha gelişmiş
Map<String, String> sozluk = new HashMap<>();
sozluk.put("kitap", "book");    // Kova 2'ye gider
sozluk.put("masa", "table");    // Kova 2'ye gider (çakışma!)
```

**Java 8 Öncesi - Linked List:**
```
Bucket 2: [("kitap", "book")] -> [("masa", "table")] -> null
```

**Java 8+ - Linked List + Red-Black Tree:**
- 8'den az eleman: Linked list kullanır
- 8+ eleman: Red-Black tree kullanır (daha hızlı arama)
- JavaScript Map'te böyle bir optimizasyon yok

**Load Factor (Yük Faktörü) - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript Map'te load factor otomatik yönetilir
let map = new Map();
// JavaScript engine otomatik olarak büyütür
// Sen bunu kontrol edemezsin
```

**Java'da:**
```java
// Java HashMap'te load factor kontrol edilebilir
Map<String, String> map1 = new HashMap<>(); // Varsayılan: 0.75
Map<String, String> map2 = new HashMap<>(16, 0.5f); // Özel: 0.5
```

**Load Factor Nedir?**
- HashMap'in ne kadar dolduğunu gösterir
- Varsayılan değer: 0.75 (75%)
- %75 dolduğunda HashMap büyütülür (rehashing)
- JavaScript Map'te böyle bir kontrol yok

**Rehashing (Yeniden Hashleme) - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript Map'te rehashing otomatik yapılır
let map = new Map();
// JavaScript engine otomatik olarak büyütür
// Sen bunu göremezsin, engine içinde yapılır
```

**Java'da:**
```java
// Java HashMap'te rehashing manuel kontrol edilebilir
Map<String, String> map = new HashMap<>();
// HashMap büyütüldüğünde
Eski boyut: 16 kova
Yeni boyut: 32 kova
// Tüm anahtarlar yeniden hesaplanır ve yeni kovalara dağıtılır
```

**Detaylı Kod Örneği - JavaScript'ten Java'ya:**

**1. HashMap Oluşturma:**

**JavaScript'te:**
```javascript
// JavaScript Map - basit oluşturma
let yaslar = new Map();

// JavaScript Object - alternatif
let yaslar2 = {};
```

**Java'da:**
```java
// Boş HashMap oluşturma
HashMap<String, Integer> yaslar = new HashMap<>();

// Başlangıç kapasitesi belirleme
HashMap<String, Integer> yaslar2 = new HashMap<>(16);

// Load factor belirleme
HashMap<String, Integer> yaslar3 = new HashMap<>(16, 0.5f);
```

**2. Veri Ekleme:**

**JavaScript'te:**
```javascript
// JavaScript Map
let yaslar = new Map();
yaslar.set("Ahmet", 25);
yaslar.set("Mehmet", 30);
yaslar.set("Ayşe", 28);

// JavaScript Object
let yaslar2 = {};
yaslar2["Ahmet"] = 25;
yaslar2["Mehmet"] = 30;
yaslar2["Ayşe"] = 28;
```

**Java'da:**
```java
// Java HashMap
HashMap<String, Integer> yaslar = new HashMap<>();
yaslar.put("Ahmet", 25);    // Hash: 12345, Kova: 9
yaslar.put("Mehmet", 30);   // Hash: 67890, Kova: 2
yaslar.put("Ayşe", 28);     // Hash: 11111, Kova: 7
```

**3. Veri Alma:**

**JavaScript'te:**
```javascript
// JavaScript Map
let yas = yaslar.get("Ahmet");  // 25
let yas2 = yaslar.get("Ali");   // undefined (yok)

// JavaScript Object
let yas3 = yaslar2["Ahmet"];    // 25
let yas4 = yaslar2["Ali"];      // undefined (yok)
```

**Java'da:**
```java
// Java HashMap
Integer yas = yaslar.get("Ahmet");  // 25
Integer yas2 = yaslar.get("Ali");   // null (yok)
```

**4. Veri Kontrolü:**

**JavaScript'te:**
```javascript
// JavaScript Map
let varMi = yaslar.has("Ahmet");  // true
let degerVarMi = yaslar.has("Ali"); // false

// JavaScript Object
let varMi2 = "Ahmet" in yaslar2;  // true
let degerVarMi2 = "Ali" in yaslar2; // false
```

**Java'da:**
```java
// Java HashMap
boolean varMi = yaslar.containsKey("Ahmet");  // true
boolean degerVarMi = yaslar.containsValue(25); // true
```

**5. Veri Silme:**

**JavaScript'te:**
```javascript
// JavaScript Map
let silinenYas = yaslar.delete("Ahmet");  // true döner
let silindi = yaslar.delete("Ali");       // false döner

// JavaScript Object
delete yaslar2["Ahmet"];  // true döner
delete yaslar2["Ali"];    // true döner (yoksa da true)
```

**Java'da:**
```java
// Java HashMap
Integer silinenYas = yaslar.remove("Ahmet");  // 25 döner
boolean silindi = yaslar.remove("Mehmet", 30); // true döner
```

**HashMap'in Avantajları - JavaScript vs Java:**

**JavaScript Map:**
- ✅ **Hızlı Erişim**: O(1) ortalama süre
- ✅ **Esnek Boyut**: Dinamik olarak büyür
- ✅ **Null Desteği**: Anahtar ve değer null olabilir
- ✅ **Basit Kullanım**: Kolay öğrenilir

**Java HashMap:**
- ✅ **Hızlı Erişim**: O(1) ortalama süre (JavaScript Map gibi)
- ✅ **Esnek Boyut**: Dinamik olarak büyür (JavaScript Map gibi)
- ✅ **Null Desteği**: Anahtar ve değer null olabilir (JavaScript Map gibi)
- ✅ **Thread-Safe Değil**: Daha hızlı (tek thread için)
- ✅ **Load Factor Kontrolü**: JavaScript'te yok
- ✅ **Kapasite Kontrolü**: JavaScript'te yok

**HashMap'in Dezavantajları - JavaScript vs Java:**

**JavaScript Map:**
- ❌ **Sıralama Yok**: Ekleme sırasını korumaz
- ❌ **Load Factor Kontrolü Yok**: Otomatik yönetilir
- ❌ **Kapasite Kontrolü Yok**: Otomatik yönetilir

**Java HashMap:**
- ❌ **Sıralama Yok**: Ekleme sırasını korumaz (JavaScript Map gibi)
- ❌ **Thread-Safe Değil**: Çoklu thread'de güvenli değil
- ❌ **Bellek Kullanımı**: Yüksek (bucket'lar için)
- ❌ **Hash Collision**: Aynı hash code'lar performansı düşürür

**Gerçek Hayat Örnekleri - JavaScript'ten Java'ya:**

**1. Sözlük Uygulaması:**

**JavaScript'te:**
```javascript
// JavaScript Map
let sozluk = new Map();
sozluk.set("kitap", "book");
sozluk.set("kalem", "pen");
sozluk.set("masa", "table");

// JavaScript Object
let sozluk2 = {
    kitap: "book",
    kalem: "pen",
    masa: "table"
};
```

**Java'da:**
```java
// Java HashMap
HashMap<String, String> sozluk = new HashMap<>();
sozluk.put("kitap", "book");
sozluk.put("kalem", "pen");
sozluk.put("masa", "table");
```

**2. Kullanıcı Profilleri:**

**JavaScript'te:**
```javascript
// JavaScript Map
let kullanicilar = new Map();
kullanicilar.set("ahmet123", {name: "Ahmet", email: "ahmet@email.com"});
kullanicilar.set("mehmet456", {name: "Mehmet", email: "mehmet@email.com"});

// JavaScript Object
let kullanicilar2 = {
    ahmet123: {name: "Ahmet", email: "ahmet@email.com"},
    mehmet456: {name: "Mehmet", email: "mehmet@email.com"}
};
```

**Java'da:**
```java
// Java HashMap
HashMap<String, User> kullanicilar = new HashMap<>();
kullanicilar.put("ahmet123", new User("Ahmet", "ahmet@email.com"));
kullanicilar.put("mehmet456", new User("Mehmet", "mehmet@email.com"));
```

**3. Önbellek Sistemi:**

**JavaScript'te:**
```javascript
// JavaScript Map
let cache = new Map();
cache.set("user:123", userData);
cache.set("product:456", productData);

// JavaScript Object
let cache2 = {};
cache2["user:123"] = userData;
cache2["product:456"] = productData;
```

**Java'da:**
```java
// Java HashMap
HashMap<String, Object> cache = new HashMap<>();
cache.put("user:123", userData);
cache.put("product:456", productData);
```

**4. Konfigürasyon Ayarları:**

**JavaScript'te:**
```javascript
// JavaScript Map
let ayarlar = new Map();
ayarlar.set("database.url", "jdbc:mysql://localhost:3306/db");
ayarlar.set("server.port", "8080");
ayarlar.set("log.level", "INFO");

// JavaScript Object
let ayarlar2 = {
    "database.url": "jdbc:mysql://localhost:3306/db",
    "server.port": "8080",
    "log.level": "INFO"
};
```

**Java'da:**
```java
// Java HashMap
HashMap<String, String> ayarlar = new HashMap<>();
ayarlar.put("database.url", "jdbc:mysql://localhost:3306/db");
ayarlar.put("server.port", "8080");
ayarlar.put("log.level", "INFO");
```

**Performans İpuçları - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript'te performans optimizasyonu sınırlı
let map = new Map();
// JavaScript engine otomatik olarak optimize eder
// Sen bunu kontrol edemezsin
```

**Java'da:**
```java
// 1. Başlangıç Kapasitesi
// Kaç eleman ekleyeceğini biliyorsan
HashMap<String, String> map = new HashMap<>(1000);

// 2. Load Factor
// Daha az rehashing için
HashMap<String, String> map2 = new HashMap<>(16, 0.5f);

// 3. Hash Code Kalitesi
// String için hashCode() iyi çalışır
// Custom class'lar için equals() ve hashCode() override et
```

**JavaScript'ten Java'ya Geçiş Özeti:**

| Özellik | JavaScript Map | Java HashMap |
|---------|----------------|--------------|
| **Oluşturma** | `new Map()` | `new HashMap<>()` |
| **Ekleme** | `map.set(key, value)` | `map.put(key, value)` |
| **Alma** | `map.get(key)` | `map.get(key)` |
| **Kontrol** | `map.has(key)` | `map.containsKey(key)` |
| **Silme** | `map.delete(key)` | `map.remove(key)` |
| **Boyut** | `map.size` | `map.size()` |
| **Load Factor** | ❌ Yok | ✅ Kontrol edilebilir |
| **Kapasite** | ❌ Yok | ✅ Kontrol edilebilir |
| **Thread Safety** | ❌ Yok | ❌ Yok (ConcurrentHashMap var) |

**Özet:**
- **JavaScript Map**: Basit, otomatik optimize edilmiş
- **Java HashMap**: Daha fazla kontrol, daha optimize edilmiş
- **Hash Fonksiyonu**: Her iki dilde de benzer şekilde çalışır
- **Bucket Sistemi**: Her iki dilde de benzer optimizasyon
- **Collision**: Her iki dilde de otomatik çözülür
- **Load Factor**: Sadece Java'da kontrol edilebilir
    Node<K,V> next; // For collision resolution
}
```

**Put Operation:**
1. Key'in hashCode() hesaplanır
2. Bucket index: `(n-1) & hash` (n = table length)
3. Bucket boşsa: yeni node eklenir
4. Bucket doluysa: collision resolution uygulanır

**Kod Örneği:**
```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", 1);
map.put("Python", 2);

// Internal process:
// 1. "Java".hashCode() = 2301506
// 2. bucketIndex = 2301506 % 16 = 2
// 3. table[2] = new Node("Java", 1)
```

#### 5. HashSet nasıl çalışır ve neden unique değerler tutar?

**Cevap:**

**JavaScript'ten Java'ya Geçiş - HashSet**

**JavaScript'te Ne Kullanıyordun?**
```javascript
// JavaScript Set - benzersiz değerler
let set = new Set();
set.add("Java");
set.add("Python");
set.add("Java"); // Eklenmez - duplicate
console.log(set); // Set(2) {"Java", "Python"}

// JavaScript Array - duplicate'lar var
let array = ["Java", "Python", "Java"];
console.log(array); // ["Java", "Python", "Java"]
```

**Java'da Ne Kullanacaksın?**
```java
// Java HashSet - JavaScript Set'e çok benzer
Set<String> set = new HashSet<>();
set.add("Java");
set.add("Python");
set.add("Java"); // Eklenmez - duplicate
System.out.println(set); // [Java, Python]
```

**HashSet Nedir?**
HashSet, Java'da benzersiz değerleri saklamak için kullanılan veri yapısıdır. JavaScript'teki Set'e çok benzer, ancak daha optimize edilmiş ve tip güvenli.

**Çalışma Prensibi - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript Set - engine tarafından optimize edilmiş
let set = new Set();
set.add("Java"); // Engine otomatik olarak duplicate kontrolü yapar
```

**Java'da:**
```java
// Java HashSet - HashMap kullanarak implement edilmiş
public class HashSet<E> {
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

**Unique Garantisi - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript Set - engine otomatik olarak unique garantisi verir
let set = new Set();
set.add("Java");   // true - yeni eleman
set.add("Java");   // false - duplicate (eklenmez)
```

**Java'da:**
1. **hashCode() ve equals()**: İki obje aynıysa aynı hash code'a sahip olmalı
2. **HashMap Behavior**: Aynı key için put() çağrıldığında eski value döner
3. **Return Value Check**: put() null dönerse yeni eleman, null dönmezse duplicate

**Kod Örneği - JavaScript'ten Java'ya:**

**JavaScript'te:**
```javascript
// JavaScript Set - basit kullanım
let set = new Set();
console.log(set.add("Java"));   // true - yeni eleman
console.log(set.add("Java"));   // false - duplicate
console.log(set.size);          // 1 - sadece bir eleman

// JavaScript Array - duplicate'lar var
let array = ["Java", "Python", "Java"];
console.log(array.length);      // 3 - duplicate'lar dahil
```

**Java'da:**
```java
// Java HashSet - JavaScript Set'e benzer
Set<String> set = new HashSet<>();
System.out.println(set.add("Java")); // true - yeni eleman
System.out.println(set.add("Java")); // false - duplicate
System.out.println(set.size());      // 1 - sadece bir eleman

// Java ArrayList - duplicate'lar var
List<String> list = new ArrayList<>();
list.add("Java");
list.add("Python");
list.add("Java");
System.out.println(list.size());     // 3 - duplicate'lar dahil
```

**Custom Object için hashCode() ve equals() - JavaScript vs Java:**

**JavaScript'te:**
```javascript
// JavaScript'te custom object'ler için özel bir şey yapmana gerek yok
let set = new Set();
set.add({name: "Ahmet", age: 25});
set.add({name: "Ahmet", age: 25}); // Farklı obje olarak kabul edilir
console.log(set.size); // 2 - farklı objeler
```

**Java'da:**
```java
// Java'da custom object'ler için hashCode() ve equals() override gerekli
class Person {
    String name;
    int age;
    
    // Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

// Kullanım
Set<Person> personSet = new HashSet<>();
personSet.add(new Person("Ahmet", 25));
personSet.add(new Person("Ahmet", 25)); // Eklenmez - duplicate
System.out.println(personSet.size()); // 1
```

**JavaScript'ten Java'ya Geçiş Özeti:**

| Özellik | JavaScript Set | Java HashSet |
|---------|----------------|--------------|
| **Oluşturma** | `new Set()` | `new HashSet<>()` |
| **Ekleme** | `set.add(item)` | `set.add(item)` |
| **Kontrol** | `set.has(item)` | `set.contains(item)` |
| **Silme** | `set.delete(item)` | `set.remove(item)` |
| **Boyut** | `set.size` | `set.size()` |
| **Custom Object** | Otomatik | hashCode() + equals() gerekli |
| **Null Desteği** | ✅ Var | ✅ Var |
| **Thread Safety** | ❌ Yok | ❌ Yok (ConcurrentHashMap var) |

**Özet:**
- **JavaScript Set**: Basit, otomatik unique garantisi
- **Java HashSet**: Daha fazla kontrol, custom object'ler için hashCode() + equals() gerekli
- **Unique Garantisi**: Her iki dilde de benzer şekilde çalışır
- **HashMap Kullanımı**: Java HashSet internal olarak HashMap kullanır
- **Custom Object**: JavaScript'te otomatik, Java'da manuel implementasyon gerekli

#### 6. Iterator ve ListIterator arasındaki fark nedir?

**Cevap:**

**JavaScript'ten Java'ya Geçiş - Iterator vs ListIterator**

**JavaScript'te Ne Kullanıyordun?**
```javascript
// JavaScript'te for...of loop
let array = ["Java", "Python", "JavaScript"];
for (let item of array) {
    console.log(item);
}

// JavaScript'te forEach
array.forEach((item, index) => {
    console.log(item, index);
});

// JavaScript'te for...in loop
for (let index in array) {
    console.log(array[index]);
}
```

**Java'da Ne Kullanacaksın?**
```java
// Java'da Iterator - JavaScript for...of gibi
List<String> list = new ArrayList<>();
list.add("Java");
list.add("Python");
list.add("JavaScript");

// Iterator - sadece ileri yönde
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    System.out.println(item);
}

// ListIterator - ileri ve geri yönde
ListIterator<String> listIterator = list.listIterator();
while (listIterator.hasNext()) {
    String item = listIterator.next();
    System.out.println(item);
}
```

**Temel Farklar:**

| Özellik | Iterator | ListIterator | JavaScript |
|---------|----------|--------------|------------|
| **Direction** | Forward only | Bidirectional | Forward only |
| **Collection Type** | Tüm Collection'lar | Sadece List | Array, Set, Map |
| **Methods** | hasNext(), next(), remove() | hasNext(), next(), hasPrevious(), previous(), add(), set() | for...of, forEach, for...in |
| **Index** | Yok | Var (nextIndex(), previousIndex()) | Var (for...in) |

**1. Iterator - JavaScript for...of gibi:**

**JavaScript'te:**
```javascript
// JavaScript'te for...of loop
let array = ["A", "B", "C"];
for (let item of array) {
    if (item === "B") {
        // JavaScript'te remove işlemi farklı
        let index = array.indexOf(item);
        array.splice(index, 1);
    }
}
```

**Java'da:**
```java
// Java'da Iterator - JavaScript for...of gibi
List<String> list = Arrays.asList("A", "B", "C");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
    if ("B".equals(element)) {
        iterator.remove(); // Safe removal
    }
}
```

**2. ListIterator - JavaScript'te Yok, Java'da Yeni:**

**JavaScript'te Böyle Bir Yapı Yok:**
```javascript
// JavaScript'te geri yönde gitmek için
let array = ["A", "B", "C"];
for (let i = array.length - 1; i >= 0; i--) {
    console.log(array[i]);
}
```

**Java'da:**
```java
// Java'da ListIterator - JavaScript'te olmayan özellik
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
ListIterator<String> listIterator = list.listIterator();

// Forward iteration - JavaScript for...of gibi
while (listIterator.hasNext()) {
    System.out.println(listIterator.next());
}

// Backward iteration - JavaScript'te yok
while (listIterator.hasPrevious()) {
    System.out.println(listIterator.previous());
}

// Index access - JavaScript for...in gibi
System.out.println("Next index: " + listIterator.nextIndex());
System.out.println("Previous index: " + listIterator.previousIndex());

// Add/Set operations - JavaScript'te farklı
listIterator.add("D"); // Yeni eleman ekle
listIterator.set("E"); // Mevcut elemanı değiştir
```

**JavaScript'ten Java'ya Geçiş Özeti:**

| Özellik | JavaScript | Java Iterator | Java ListIterator |
|---------|------------|---------------|-------------------|
| **Forward Loop** | `for...of` | `hasNext() + next()` | `hasNext() + next()` |
| **Backward Loop** | `for (let i = array.length - 1; i >= 0; i--)` | ❌ Yok | `hasPrevious() + previous()` |
| **Index Access** | `for...in` | ❌ Yok | `nextIndex() + previousIndex()` |
| **Remove** | `splice()` | `remove()` | `remove()` |
| **Add** | `push()`, `unshift()` | ❌ Yok | `add()` |
| **Set** | `array[index] = value` | ❌ Yok | `set()` |
| **Collection Type** | Array, Set, Map | Tüm Collection'lar | Sadece List |

**Özet:**
- **JavaScript**: Basit loop'lar, manuel index yönetimi
- **Java Iterator**: JavaScript for...of gibi, sadece ileri yönde
- **Java ListIterator**: JavaScript'te olmayan özellikler, ileri-geri yönde, index erişimi
- **Güvenli Silme**: Java'da Iterator ile güvenli silme, JavaScript'te manuel
- **Bidirectional**: Sadece Java ListIterator'da var
}

// Backward iteration
while (listIterator.hasPrevious()) {
    System.out.println(listIterator.previous());
}

// Modification during iteration
listIterator.add("D"); // Add at current position
listIterator.set("E"); // Replace last returned element
```

#### 7. Comparable ve Comparator interface'lerinin farkı nedir?

**Cevap:**

| Özellik | Comparable | Comparator |
|---------|------------|------------|
| **Package** | java.lang | java.util |
| **Method** | compareTo(T o) | compare(T o1, T o2) |
| **Implementation** | Class'ın kendisinde | Ayrı class veya lambda |
| **Natural Ordering** | Evet | Hayır |
| **Multiple Sorting** | Hayır | Evet |

**Comparable:**
```java
class Person implements Comparable<Person> {
    String name;
    int age;
    
    @Override
    public int compareTo(Person other) {
        return this.age - other.age; // Natural ordering by age
    }
}

// Usage
List<Person> people = Arrays.asList(new Person("John", 30), new Person("Jane", 25));
Collections.sort(people); // Uses natural ordering
```

**Comparator:**
```java
// Multiple sorting criteria
Comparator<Person> byName = (p1, p2) -> p1.name.compareTo(p2.name);
Comparator<Person> byAge = (p1, p2) -> Integer.compare(p1.age, p2.age);

// Usage
Collections.sort(people, byName); // Sort by name
Collections.sort(people, byAge);  // Sort by age

// Chaining comparators
Collections.sort(people, byAge.thenComparing(byName));
```

#### 8. Collections.sort() ve Arrays.sort() arasındaki fark nedir?

**Cevap:**

| Özellik | Collections.sort() | Arrays.sort() |
|---------|-------------------|---------------|
| **Input** | List | Array |
| **Algorithm** | Merge sort (stable) | Dual-pivot quicksort (unstable) |
| **Performance** | O(n log n) | O(n log n) average, O(n²) worst |
| **Memory** | O(n) extra space | O(log n) extra space |
| **Stability** | Stable | Unstable |

**Kod Örneği:**
```java
// Collections.sort() - List için
List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5));
Collections.sort(list);
System.out.println(list); // [1, 1, 3, 4, 5]

// Arrays.sort() - Array için
int[] array = {3, 1, 4, 1, 5};
Arrays.sort(array);
System.out.println(Arrays.toString(array)); // [1, 1, 3, 4, 5]

// Custom comparator
Collections.sort(list, Collections.reverseOrder());
Arrays.sort(array); // Primitive arrays don't support custom comparator
```

### Orta Seviye

#### 9. HashMap'te collision nasıl çözülür?

**Cevap:**
HashMap'te collision resolution iki yöntemle yapılır:

**1. Separate Chaining (Linked List):**
- Aynı bucket'a gelen elemanlar linked list olarak saklanır
- Java 8 öncesi tek yöntem
- Java 8+ da 8'den az eleman için kullanılır

**2. Tree (Red-Black Tree):**
- Java 8+ da 8+ eleman için Red-Black Tree kullanılır
- O(log n) lookup time
- Treeify threshold: 8, Untreeify threshold: 6

**Kod Örneği:**
```java
// Collision example
Map<String, Integer> map = new HashMap<>();
// Assume these keys hash to same bucket
map.put("Aa", 1); // hashCode: 2112
map.put("BB", 2); // hashCode: 2112 (collision!)

// Internal structure after collision:
// Bucket[0]: Node("Aa", 1) -> Node("BB", 2) -> null
// Or if > 8 elements: Red-Black Tree
```

**Performance Impact:**
- **No collision**: O(1)
- **With collision (list)**: O(k) where k = elements in bucket
- **With collision (tree)**: O(log k)

#### 10. ConcurrentHashMap'in HashMap'ten farkı nedir?

**Cevap:**

| Özellik | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| **Thread Safety** | No | Yes |
| **Null Keys/Values** | Allowed | Not allowed |
| **Locking** | No locking | Segment-based locking (Java 7), CAS (Java 8+) |
| **Performance** | Higher | Slightly lower |
| **Iterator** | Fail-fast | Weakly consistent |

**ConcurrentHashMap Features:**
```java
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();

// Thread-safe operations
concurrentMap.put("key1", 1);
concurrentMap.putIfAbsent("key2", 2);
concurrentMap.compute("key1", (k, v) -> v + 1);

// Atomic operations
concurrentMap.merge("key3", 1, Integer::sum);

// No null values allowed
// concurrentMap.put("key", null); // Throws NullPointerException
```

**Internal Structure (Java 8+):**
- **CAS (Compare-And-Swap)**: Lock-free operations
- **Synchronized blocks**: Only for tree operations
- **Volatile variables**: For visibility

#### 11. TreeMap ve HashMap arasındaki farklar nelerdir?

**Cevap:**

| Özellik | HashMap | TreeMap |
|---------|---------|---------|
| **Ordering** | No guarantee | Sorted (natural or custom) |
| **Performance** | O(1) average | O(log n) |
| **Null Keys** | One null allowed | Not allowed |
| **Implementation** | Hash table | Red-Black Tree |
| **Memory** | Lower | Higher |

**TreeMap Özellikleri:**
```java
// Natural ordering
TreeMap<String, Integer> treeMap = new TreeMap<>();
treeMap.put("Charlie", 3);
treeMap.put("Alice", 1);
treeMap.put("Bob", 2);
System.out.println(treeMap); // {Alice=1, Bob=2, Charlie=3}

// Custom ordering
TreeMap<String, Integer> customTreeMap = new TreeMap<>(Collections.reverseOrder());
customTreeMap.put("Alice", 1);
customTreeMap.put("Bob", 2);
System.out.println(customTreeMap); // {Bob=2, Alice=1}

// Range operations
SortedMap<String, Integer> subMap = treeMap.subMap("Alice", "Charlie");
System.out.println(subMap); // {Alice=1, Bob=2}
```

#### 12. LinkedHashMap'in özellikleri nelerdir?

**Cevap:**
LinkedHashMap, HashMap + insertion order veya access order korur.

**Özellikler:**
- **Insertion Order**: Varsayılan, elemanlar eklendiği sırada
- **Access Order**: LRU (Least Recently Used) cache için
- **Performance**: HashMap'ten biraz yavaş (pointer overhead)

**Kod Örneği:**
```java
// Insertion order (default)
LinkedHashMap<String, Integer> insertionOrder = new LinkedHashMap<>();
insertionOrder.put("First", 1);
insertionOrder.put("Second", 2);
insertionOrder.put("Third", 3);
System.out.println(insertionOrder); // {First=1, Second=2, Third=3}

// Access order (LRU cache)
LinkedHashMap<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
accessOrder.put("A", 1);
accessOrder.put("B", 2);
accessOrder.put("C", 3);
accessOrder.get("A"); // Move to end
System.out.println(accessOrder); // {B=2, C=3, A=1}

// LRU Cache implementation
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

#### 13. PriorityQueue nasıl çalışır?

**Cevap:**
PriorityQueue, heap veri yapısını kullanarak priority-based ordering sağlar.

**Özellikler:**
- **Heap Structure**: Min-heap (küçük değer yüksek priority)
- **Ordering**: Natural ordering veya custom comparator
- **Performance**: O(log n) insertion/deletion, O(1) peek
- **Not Thread-Safe**: Concurrent access için PriorityBlockingQueue

**Kod Örneği:**
```java
// Natural ordering (min-heap)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);
System.out.println(minHeap.poll()); // 1 (highest priority)

// Custom ordering (max-heap)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
maxHeap.offer(1);
maxHeap.offer(5);
maxHeap.offer(3);
System.out.println(maxHeap.poll()); // 5 (highest priority)

// Custom objects
class Task implements Comparable<Task> {
    String name;
    int priority;
    
    @Override
    public int compareTo(Task other) {
        return Integer.compare(this.priority, other.priority);
    }
}

PriorityQueue<Task> taskQueue = new PriorityQueue<>();
taskQueue.offer(new Task("Low", 3));
taskQueue.offer(new Task("High", 1));
taskQueue.offer(new Task("Medium", 2));
```

#### 14. WeakHashMap'in kullanım alanları nelerdir?

**Cevap:**
WeakHashMap, weak reference'lar kullanarak key'lerin garbage collect edilmesine izin verir.

**Özellikler:**
- **Weak References**: Key'ler strong reference olmazsa GC edilebilir
- **Automatic Cleanup**: Entry'ler otomatik olarak temizlenir
- **Use Cases**: Cache, listener patterns, metadata storage

**Kod Örneği:**
```java
// WeakHashMap example
WeakHashMap<String, String> weakMap = new WeakHashMap<>();
String key1 = new String("key1");
String key2 = new String("key2");

weakMap.put(key1, "value1");
weakMap.put(key2, "value2");
System.out.println(weakMap.size()); // 2

// Remove strong reference
key1 = null;
System.gc(); // Suggest garbage collection
Thread.sleep(1000);

System.out.println(weakMap.size()); // 1 (key1 entry removed)

// Cache implementation
class WeakCache<K, V> {
    private final WeakHashMap<K, V> cache = new WeakHashMap<>();
    
    public V get(K key) {
        return cache.get(key);
    }
    
    public void put(K key, V value) {
        cache.put(key, value);
    }
}
```

#### 15. Collections.synchronizedList() ve CopyOnWriteArrayList arasındaki fark nedir?

**Cevap:**

| Özellik | synchronizedList() | CopyOnWriteArrayList |
|---------|-------------------|---------------------|
| **Locking** | Method-level synchronization | No locking for reads |
| **Performance** | Slower (all operations locked) | Faster reads, slower writes |
| **Iterator** | Fail-fast (throws exception) | Snapshot iterator |
| **Memory** | Lower | Higher (copy on write) |
| **Use Case** | General synchronization | Read-heavy scenarios |

**synchronizedList:**
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

// All operations are synchronized
synchronized(syncList) {
    syncList.add("item");
    // Iterator must be in synchronized block
    Iterator<String> it = syncList.iterator();
    while (it.hasNext()) {
        System.out.println(it.next());
    }
}
```

**CopyOnWriteArrayList:**
```java
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("item1");
cowList.add("item2");

// Read operations are lock-free
for (String item : cowList) {
    System.out.println(item); // Safe iteration
    cowList.add("newItem"); // Won't affect current iteration
}

// Write operations create new array
cowList.add("item3"); // Creates new internal array
```

#### 16. Stream API ile Collections arasındaki ilişki nedir?

**Cevap:**
Stream API, Collection'lar üzerinde functional programming operasyonları sağlar.

**İlişki:**
- **Source**: Collection'lar stream'in kaynağıdır
- **Intermediate Operations**: Lazy evaluation, yeni stream döner
- **Terminal Operations**: Stream'i consume eder, sonuç döner

**Kod Örneği:**
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Stream operations
List<String> result = names.stream()
    .filter(name -> name.length() > 3)        // Intermediate
    .map(String::toUpperCase)                 // Intermediate
    .sorted()                                 // Intermediate
    .collect(Collectors.toList());           // Terminal

System.out.println(result); // [ALICE, CHARLIE, DAVID]

// Parallel processing
List<String> parallelResult = names.parallelStream()
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());

// Grouping and partitioning
Map<Integer, List<String>> grouped = names.stream()
    .collect(Collectors.groupingBy(String::length));

Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(name -> name.length() > 4));
```

---

## Thread Safety

### Temel Seviye

#### 1. Thread-safe olan ve olmayan Collection'ları listeleyin.

**Cevap:**

**Thread-Safe Collection'lar:**
- **Vector** (synchronized)
- **Hashtable** (synchronized)
- **Stack** (Vector'den extend eder)
- **ConcurrentHashMap** (Java 5+)
- **CopyOnWriteArrayList** (Java 5+)
- **CopyOnWriteArraySet** (Java 5+)
- **ConcurrentLinkedQueue** (Java 5+)
- **ConcurrentLinkedDeque** (Java 7+)
- **BlockingQueue implementations** (ArrayBlockingQueue, LinkedBlockingQueue, etc.)
- **Collections.synchronizedX()** wrapper'ları

**Thread-Safe Olmayan Collection'lar:**
- **ArrayList**
- **LinkedList**
- **HashMap**
- **TreeMap**
- **LinkedHashMap**
- **HashSet**
- **TreeSet**
- **LinkedHashSet**
- **PriorityQueue**

**Kod Örneği:**
```java
// Thread-safe
Vector<String> vector = new Vector<>();
Hashtable<String, String> hashtable = new Hashtable<>();
ConcurrentHashMap<String, String> concurrentMap = new ConcurrentHashMap<>();

// Thread-safe olmayan
ArrayList<String> arrayList = new ArrayList<>();
HashMap<String, String> hashMap = new HashMap<>();

// Synchronized wrapper
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
```

#### 2. Vector ve ArrayList arasındaki fark nedir?

**Cevap:**

| Özellik | Vector | ArrayList |
|---------|--------|-----------|
| **Thread Safety** | Thread-safe (synchronized) | Thread-safe değil |
| **Performance** | Daha yavaş (synchronization overhead) | Daha hızlı |
| **Growth Strategy** | 2x growth | 1.5x growth |
| **Legacy** | Java 1.0'dan beri | Java 1.2'den beri |
| **Enumeration** | Enumeration support | Iterator only |
| **Initial Capacity** | 10 (default) | 10 (default) |

**Detaylı Açıklama:**

**Vector:**
- Tüm public method'lar synchronized
- Legacy class, yeni projelerde kullanılmamalı
- Enumeration interface'ini destekler
- Capacity artırırken 2x büyür

**ArrayList:**
- Synchronization yok, daha performanslı
- Modern Java'da tercih edilen
- Iterator ve ListIterator destekler
- Capacity artırırken 1.5x büyür

**Kod Örneği:**
```java
// Vector - Thread-safe but slower
Vector<String> vector = new Vector<>();
vector.add("Item1");
vector.add("Item2");

// Synchronized access
synchronized(vector) {
    Enumeration<String> enumeration = vector.elements();
    while (enumeration.hasMoreElements()) {
        System.out.println(enumeration.nextElement());
    }
}

// ArrayList - Not thread-safe but faster
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("Item1");
arrayList.add("Item2");

// Need external synchronization for thread safety
synchronized(arrayList) {
    Iterator<String> iterator = arrayList.iterator();
    while (iterator.hasNext()) {
        System.out.println(iterator.next());
    }
}
```

#### 3. Hashtable ve HashMap arasındaki fark nedir?

**Cevap:**

| Özellik | Hashtable | HashMap |
|---------|-----------|---------|
| **Thread Safety** | Thread-safe (synchronized) | Thread-safe değil |
| **Null Keys/Values** | İzin vermez | Bir null key, birden fazla null value |
| **Performance** | Daha yavaş | Daha hızlı |
| **Legacy** | Java 1.0'dan beri | Java 1.2'den beri |
| **Iterator** | Enumeration ve Iterator | Iterator (fail-fast) |
| **Initial Capacity** | 11 (default) | 16 (default) |

**Detaylı Açıklama:**

**Hashtable:**
- Tüm public method'lar synchronized
- Null key/value'lara izin vermez
- Legacy class, ConcurrentHashMap tercih edilmeli
- Enumeration interface'ini destekler

**HashMap:**
- Synchronization yok, daha performanslı
- Null key/value'lara izin verir
- Modern Java'da tercih edilen
- Fail-fast iterator

**Kod Örneği:**
```java
// Hashtable - Thread-safe
Hashtable<String, Integer> hashtable = new Hashtable<>();
hashtable.put("key1", 1);
hashtable.put("key2", 2);
// hashtable.put(null, 3); // Throws NullPointerException
// hashtable.put("key3", null); // Throws NullPointerException

// HashMap - Not thread-safe
HashMap<String, Integer> hashMap = new HashMap<>();
hashMap.put("key1", 1);
hashMap.put("key2", 2);
hashMap.put(null, 3); // Allowed
hashMap.put("key3", null); // Allowed

// Thread-safe alternative
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("key1", 1);
// concurrentMap.put(null, 2); // Throws NullPointerException
```

#### 4. StringBuffer ve StringBuilder arasındaki fark nedir?

**Cevap:**

| Özellik | StringBuffer | StringBuilder |
|---------|--------------|---------------|
| **Thread Safety** | Thread-safe (synchronized) | Thread-safe değil |
| **Performance** | Daha yavaş | Daha hızlı |
| **Java Version** | Java 1.0'dan beri | Java 1.5'ten beri |
| **Use Case** | Multi-threaded environments | Single-threaded environments |
| **Memory** | Aynı | Aynı |

**Detaylı Açıklama:**

**StringBuffer:**
- Tüm public method'lar synchronized
- Multi-threaded environment'larda güvenli
- Performance overhead'i var
- Legacy class

**StringBuilder:**
- Synchronization yok, daha performanslı
- Single-threaded environment'larda tercih edilen
- StringBuffer'ın unsynchronized versiyonu
- Modern Java'da default choice

**Kod Örneği:**
```java
// StringBuffer - Thread-safe
StringBuffer stringBuffer = new StringBuffer();
stringBuffer.append("Hello");
stringBuffer.append(" ");
stringBuffer.append("World");
String result1 = stringBuffer.toString(); // "Hello World"

// StringBuilder - Not thread-safe but faster
StringBuilder stringBuilder = new StringBuilder();
stringBuilder.append("Hello");
stringBuilder.append(" ");
stringBuilder.append("World");
String result2 = stringBuilder.toString(); // "Hello World"

// Performance comparison
long start = System.currentTimeMillis();
StringBuffer sb1 = new StringBuffer();
for (int i = 0; i < 100000; i++) {
    sb1.append("test");
}
long end = System.currentTimeMillis();
System.out.println("StringBuffer time: " + (end - start) + "ms");

start = System.currentTimeMillis();
StringBuilder sb2 = new StringBuilder();
for (int i = 0; i < 100000; i++) {
    sb2.append("test");
}
end = System.currentTimeMillis();
System.out.println("StringBuilder time: " + (end - start) + "ms");
```

#### 5. synchronized keyword'ünün kullanımı nasıldır?

**Cevap:**
synchronized keyword'ü, Java'da thread synchronization sağlamak için kullanılır.

**Kullanım Şekilleri:**

1. **Method Level Synchronization**
2. **Block Level Synchronization**
3. **Static Method Synchronization**
4. **Object Reference Synchronization**

**Kod Örneği:**
```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    // Method level synchronization
    public synchronized void increment() {
        count++;
    }
    
    // Block level synchronization
    public void decrement() {
        synchronized(this) {
            count--;
        }
    }
    
    // Custom object synchronization
    public void reset() {
        synchronized(lock) {
            count = 0;
        }
    }
    
    // Static method synchronization
    public static synchronized void staticMethod() {
        // Synchronized on Class object
    }
    
    // Static block synchronization
    public static void staticBlock() {
        synchronized(Counter.class) {
            // Synchronized on Class object
        }
    }
}

// Usage
Counter counter = new Counter();

// Multiple threads accessing synchronized methods
Thread thread1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

Thread thread2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

thread1.start();
thread2.start();
thread1.join();
thread2.join();

System.out.println("Final count: " + counter.getCount()); // Should be 2000
```

### Orta Seviye
6. ConcurrentHashMap'in thread-safe olma prensibi nedir?
7. CopyOnWriteArrayList ne zaman kullanılır?
8. Collections.synchronizedMap() ve ConcurrentHashMap arasındaki fark nedir?
9. BlockingQueue interface'inin önemi nedir?
10. ThreadLocal'ın kullanım alanları nelerdir?
11. volatile keyword'ünün işlevi nedir?
12. AtomicInteger ve synchronized arasındaki fark nedir?

---

## Data Types

### Temel Seviye

#### 1. Java'da primitive ve non-primitive data type'ları listeleyin.

**Cevap:**

**Primitive Data Types:**
- **byte**: 8-bit, -128 to 127
- **short**: 16-bit, -32,768 to 32,767
- **int**: 32-bit, -2,147,483,648 to 2,147,483,647
- **long**: 64-bit, -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
- **float**: 32-bit IEEE 754 floating point
- **double**: 64-bit IEEE 754 floating point
- **char**: 16-bit Unicode character
- **boolean**: true or false (size not precisely defined)

**Non-Primitive Data Types (Reference Types):**
- **String**: Character sequence
- **Array**: Collection of elements
- **Class**: User-defined types
- **Interface**: Contract definition
- **Enum**: Enumeration type
- **Wrapper Classes**: Integer, Double, Boolean, etc.

**Kod Örneği:**
```java
// Primitive types
byte b = 100;
short s = 1000;
int i = 100000;
long l = 1000000000L;
float f = 3.14f;
double d = 3.14159265359;
char c = 'A';
boolean bool = true;

// Non-primitive types
String str = "Hello World";
int[] array = {1, 2, 3, 4, 5};
Integer wrapperInt = 42;
List<String> list = new ArrayList<>();
```

#### 2. int ve Integer arasındaki fark nedir?

**Cevap:**

| Özellik | int | Integer |
|---------|-----|---------|
| **Type** | Primitive | Wrapper class |
| **Memory** | 4 bytes | 16 bytes (object overhead) |
| **Default Value** | 0 | null |
| **Null Support** | No | Yes |
| **Performance** | Faster | Slower (object creation) |
| **Generic Support** | No | Yes |
| **Method Support** | No | Yes (parseInt, toString, etc.) |

**Detaylı Açıklama:**

**int (Primitive):**
- Stack'te saklanır
- Direct value storage
- No method calls
- Cannot be null
- Better performance

**Integer (Wrapper Class):**
- Heap'te saklanır
- Object reference
- Rich method set
- Can be null
- Autoboxing/unboxing overhead

**Kod Örneği:**
```java
// int - Primitive
int primitiveInt = 42;
int defaultInt; // Default value: 0
// int nullInt = null; // Compilation error

// Integer - Wrapper class
Integer wrapperInt = 42;
Integer nullInteger = null; // Allowed
Integer defaultWrapper; // Default value: null

// Autoboxing and Unboxing
int i = 100;
Integer integer = i; // Autoboxing: int -> Integer
int j = integer; // Unboxing: Integer -> int

// Method usage
String str = Integer.toString(42); // Static method
int parsed = Integer.parseInt("123"); // Static method
int max = Integer.max(10, 20); // Static method
```

#### 3. Autoboxing ve Unboxing nedir?

**Cevap:**
Autoboxing ve Unboxing, primitive types ile wrapper classes arasında otomatik dönüşüm sağlar.

**Autoboxing:**
- Primitive type'ı wrapper class'a otomatik dönüştürme
- Java 5'ten beri mevcut
- Compiler tarafından otomatik yapılır

**Unboxing:**
- Wrapper class'ı primitive type'a otomatik dönüştürme
- Null pointer exception riski
- Compiler tarafından otomatik yapılır

**Kod Örneği:**
```java
// Autoboxing examples
int primitiveInt = 42;
Integer wrapperInt = primitiveInt; // Autoboxing

List<Integer> numbers = new ArrayList<>();
numbers.add(1); // Autoboxing: int -> Integer
numbers.add(2); // Autoboxing: int -> Integer

// Unboxing examples
Integer wrapper = 100;
int primitive = wrapper; // Unboxing: Integer -> int

int sum = 0;
for (Integer num : numbers) {
    sum += num; // Unboxing: Integer -> int
}

// Null pointer exception risk
Integer nullInteger = null;
// int value = nullInteger; // Throws NullPointerException

// Manual boxing/unboxing (Java 5 öncesi)
Integer manualBox = Integer.valueOf(42); // Manual boxing
int manualUnbox = manualBox.intValue(); // Manual unboxing
```

#### 4. String, StringBuffer ve StringBuilder arasındaki farklar nelerdir?

**Cevap:**

| Özellik | String | StringBuffer | StringBuilder |
|---------|--------|--------------|---------------|
| **Mutability** | Immutable | Mutable | Mutable |
| **Thread Safety** | Thread-safe | Thread-safe (synchronized) | Not thread-safe |
| **Performance** | Slow for concatenation | Medium | Fast |
| **Memory** | Creates new objects | Modifies existing | Modifies existing |
| **Use Case** | Immutable strings | Multi-threaded concatenation | Single-threaded concatenation |

**Detaylı Açıklama:**

**String:**
- Immutable (değiştirilemez)
- Her değişiklik yeni String object oluşturur
- String pool'da saklanır
- Thread-safe (immutable olduğu için)

**StringBuffer:**
- Mutable (değiştirilebilir)
- Synchronized method'lar
- Multi-threaded environment'larda güvenli
- Legacy class

**StringBuilder:**
- Mutable (değiştirilebilir)
- Unsynchronized method'lar
- Single-threaded environment'larda hızlı
- Modern Java'da tercih edilen

**Kod Örneği:**
```java
// String - Immutable
String str1 = "Hello";
String str2 = str1 + " World"; // Creates new String object
System.out.println(str1); // "Hello" (unchanged)

// StringBuffer - Mutable and thread-safe
StringBuffer stringBuffer = new StringBuffer();
stringBuffer.append("Hello");
stringBuffer.append(" ");
stringBuffer.append("World");
String result1 = stringBuffer.toString(); // "Hello World"

// StringBuilder - Mutable and fast
StringBuilder stringBuilder = new StringBuilder();
stringBuilder.append("Hello");
stringBuilder.append(" ");
stringBuilder.append("World");
String result2 = stringBuilder.toString(); // "Hello World"

// Performance comparison
long start = System.currentTimeMillis();
String str = "";
for (int i = 0; i < 10000; i++) {
    str += "a"; // Creates new String each time
}
long end = System.currentTimeMillis();
System.out.println("String concatenation: " + (end - start) + "ms");

start = System.currentTimeMillis();
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append("a"); // Modifies existing StringBuilder
}
end = System.currentTimeMillis();
System.out.println("StringBuilder: " + (end - start) + "ms");
```

#### 5. float ve double arasındaki fark nedir?

**Cevap:**

| Özellik | float | double |
|---------|-------|--------|
| **Size** | 32-bit (4 bytes) | 64-bit (8 bytes) |
| **Precision** | 7 decimal digits | 15-17 decimal digits |
| **Range** | ±3.4×10³⁸ | ±1.7×10³⁰⁸ |
| **Default** | 0.0f | 0.0d |
| **Suffix** | f or F | d or D (optional) |
| **Performance** | Faster | Slower |
| **Memory** | Less | More |

**Detaylı Açıklama:**

**float:**
- Single precision floating point
- IEEE 754 standard
- 1 sign bit + 8 exponent bits + 23 mantissa bits
- Suitable for memory-constrained applications

**double:**
- Double precision floating point
- IEEE 754 standard
- 1 sign bit + 11 exponent bits + 52 mantissa bits
- Default choice for floating point calculations

**Kod Örneği:**
```java
// float examples
float f1 = 3.14f; // Must use 'f' suffix
float f2 = 3.14F; // Alternative syntax
float f3 = (float) 3.14; // Casting

// double examples
double d1 = 3.14159265359; // Default for decimal literals
double d2 = 3.14159265359d; // Explicit 'd' suffix
double d3 = 3.14159265359D; // Alternative syntax

// Precision comparison
float floatValue = 1.23456789f;
double doubleValue = 1.23456789;

System.out.println("float: " + floatValue);   // 1.2345679 (7 digits)
System.out.println("double: " + doubleValue); // 1.23456789 (15+ digits)

// Scientific notation
float scientificFloat = 1.23e4f; // 12300.0
double scientificDouble = 1.23e4; // 12300.0

// Special values
float positiveInfinity = Float.POSITIVE_INFINITY;
double negativeInfinity = Double.NEGATIVE_INFINITY;
float notANumber = Float.NaN;
```

#### 6. char ve String arasındaki fark nedir?

**Cevap:**

| Özellik | char | String |
|---------|------|--------|
| **Type** | Primitive | Object |
| **Size** | 16-bit (2 bytes) | Variable |
| **Content** | Single character | Sequence of characters |
| **Mutability** | Immutable | Immutable |
| **Memory** | Stack | Heap |
| **Default Value** | '\u0000' | null |
| **Operations** | Limited | Rich method set |

**Detaylı Açıklama:**

**char:**
- Single Unicode character
- 16-bit unsigned integer
- Range: 0 to 65,535
- Can represent any Unicode character

**String:**
- Sequence of characters
- Immutable object
- Rich method set for manipulation
- String pool optimization

**Kod Örneği:**
```java
// char examples
char c1 = 'A';
char c2 = 65; // ASCII value
char c3 = '\u0041'; // Unicode escape
char c4 = '\n'; // Escape sequence

// String examples
String str1 = "Hello";
String str2 = new String("Hello");
String str3 = String.valueOf('A'); // char to String

// char operations
char ch = 'A';
int asciiValue = ch; // 65
char nextChar = (char) (ch + 1); // 'B'
boolean isDigit = Character.isDigit(ch); // false
boolean isLetter = Character.isLetter(ch); // true

// String operations
String str = "Hello World";
int length = str.length(); // 11
char firstChar = str.charAt(0); // 'H'
String upper = str.toUpperCase(); // "HELLO WORLD"
String lower = str.toLowerCase(); // "hello world"
String substring = str.substring(0, 5); // "Hello"

// char array to String
char[] charArray = {'H', 'e', 'l', 'l', 'o'};
String fromCharArray = new String(charArray); // "Hello"

// String to char array
char[] toCharArray = str.toCharArray();
```

#### 7. boolean data type'ının boyutu nedir?

**Cevap:**
boolean data type'ının boyutu Java specification'da tam olarak tanımlanmamıştır.

**Boyut Bilgileri:**
- **JVM Implementation Dependent**: Farklı JVM'lerde farklı olabilir
- **Typical Size**: 1 byte (8 bits)
- **Array Elements**: 1 byte per element
- **Object Fields**: JVM optimization'a bağlı
- **Possible Values**: true, false

**Kod Örneği:**
```java
// boolean examples
boolean flag = true;
boolean isComplete = false;
boolean defaultBoolean; // Default value: false

// boolean operations
boolean a = true;
boolean b = false;
boolean and = a && b; // false
boolean or = a || b;  // true
boolean not = !a;     // false

// boolean in conditions
if (flag) {
    System.out.println("Flag is true");
} else {
    System.out.println("Flag is false");
}

// boolean array
boolean[] flags = new boolean[3]; // Default: [false, false, false]
flags[0] = true;
flags[1] = false;
flags[2] = true;

// Memory usage (approximate)
System.out.println("Boolean size: " + Byte.BYTES + " bytes"); // 1 byte
System.out.println("Boolean array size: " + flags.length + " bytes"); // 3 bytes
```

### Orta Seviye
8. Wrapper class'ların kullanım alanları nelerdir?
9. BigDecimal ne zaman kullanılır?
10. Optional class'ının amacı nedir?
11. Generic type'ların primitive type'larla çalışma şekli nasıldır?
12. Enum'ların memory'deki yerleşimi nasıldır?
13. Varargs (Variable Arguments) nasıl çalışır?
14. Type erasure nedir?

---

## Java 21 & Spring Boot 3.x Değişiklikleri

### Temel Seviye

#### 1. Java 21'deki yeni özellikler nelerdir?

**Cevap:**
Java 21, LTS (Long Term Support) sürümüdür ve birçok önemli özellik içerir.

**Ana Özellikler:**
- **Virtual Threads (Project Loom)**: Hafif thread'ler
- **Pattern Matching for switch**: Gelişmiş switch expressions
- **Record Patterns**: Record'lar için pattern matching
- **String Templates (Preview)**: String interpolation
- **Sequenced Collections**: Sıralı collection interface'leri
- **Foreign Function & Memory API**: Native code integration
- **Unnamed Patterns and Variables**: Gelişmiş pattern matching

**Kod Örneği:**
```java
// Virtual Threads
Thread virtualThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread");
});

// Pattern Matching for switch
String result = switch (obj) {
    case String s -> "String: " + s;
    case Integer i -> "Integer: " + i;
    case null -> "Null value";
    default -> "Unknown type";
};

// Record Patterns
record Point(int x, int y) {}
record Circle(Point center, int radius) {}

public static double area(Object shape) {
    return switch (shape) {
        case Circle(Point(var x, var y), var r) -> Math.PI * r * r;
        case Point(var x, var y) -> 0.0;
        default -> throw new IllegalArgumentException("Unknown shape");
    };
}

// Sequenced Collections
List<String> list = new ArrayList<>();
list.addFirst("First"); // New method
list.addLast("Last");   // New method
String first = list.getFirst(); // New method
String last = list.getLast();   // New method
```

#### 2. Virtual Threads nedir ve nasıl çalışır?

**Cevap:**
Virtual Threads, geleneksel platform thread'lerinden çok daha hafif olan thread'lerdir.

**Özellikler:**
- **Lightweight**: Çok az memory kullanır (KB seviyesinde)
- **Massive Scale**: Milyonlarca virtual thread oluşturulabilir
- **M:N Mapping**: M virtual thread, N platform thread
- **Blocking Operations**: Blocking I/O operations virtual thread'i bloklamaz
- **Compatibility**: Mevcut Java code ile uyumlu

**Çalışma Prensibi:**
- Virtual thread'ler platform thread'ler üzerinde çalışır
- Blocking operation'da virtual thread park edilir
- Platform thread başka virtual thread'e atanır
- I/O tamamlandığında virtual thread devam eder

**Kod Örneği:**
```java
// Virtual Thread creation
Thread virtualThread = Thread.ofVirtual()
    .name("virtual-thread-1")
    .start(() -> {
        System.out.println("Virtual thread: " + Thread.currentThread());
        try {
            Thread.sleep(1000); // Non-blocking for platform thread
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });

// Virtual Thread with ExecutorService
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1000000; i++) {
        final int taskId = i;
        executor.submit(() -> {
            System.out.println("Task " + taskId + " in thread: " + 
                Thread.currentThread());
            // Simulate I/O operation
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }
}

// Traditional vs Virtual Threads
// Traditional: Limited to ~1000-10000 threads
ExecutorService traditionalExecutor = Executors.newFixedThreadPool(100);

// Virtual: Can handle millions of threads
ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();
```

#### 3. Pattern Matching for switch'in faydaları nelerdir?

**Cevap:**
Pattern Matching for switch, type checking ve casting'i otomatikleştirir.

**Faydalar:**
- **Type Safety**: Compile-time type checking
- **Automatic Casting**: Manual casting gerekmez
- **Exhaustiveness**: Tüm case'lerin kapsandığından emin olur
- **Null Handling**: Null case'leri explicit olarak handle eder
- **Guard Clauses**: Ek koşullar eklenebilir

**Kod Örneği:**
```java
// Traditional approach
public static String processObject(Object obj) {
    if (obj instanceof String) {
        String s = (String) obj;
        return "String: " + s.toUpperCase();
    } else if (obj instanceof Integer) {
        Integer i = (Integer) obj;
        return "Integer: " + (i * 2);
    } else if (obj instanceof List) {
        List<?> list = (List<?>) obj;
        return "List with " + list.size() + " elements";
    } else {
        return "Unknown type";
    }
}

// Pattern Matching approach
public static String processObject(Object obj) {
    return switch (obj) {
        case String s -> "String: " + s.toUpperCase();
        case Integer i -> "Integer: " + (i * 2);
        case List<?> list -> "List with " + list.size() + " elements";
        case null -> "Null value";
        default -> "Unknown type";
    };
}

// Guard clauses
public static String processNumber(Number num) {
    return switch (num) {
        case Integer i when i > 0 -> "Positive integer: " + i;
        case Integer i when i < 0 -> "Negative integer: " + i;
        case Integer i -> "Zero: " + i;
        case Double d when d > 0 -> "Positive double: " + d;
        case Double d -> "Non-positive double: " + d;
        default -> "Other number: " + num;
    };
}

// Sealed classes with pattern matching
sealed interface Shape permits Circle, Rectangle, Triangle {
    double area();
}

record Circle(double radius) implements Shape {
    public double area() { return Math.PI * radius * radius; }
}

record Rectangle(double width, double height) implements Shape {
    public double area() { return width * height; }
}

record Triangle(double base, double height) implements Shape {
    public double area() { return 0.5 * base * height; }
}

public static String describeShape(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with radius " + c.radius();
        case Rectangle r -> "Rectangle " + r.width() + "x" + r.height();
        case Triangle t -> "Triangle with base " + t.base() + " and height " + t.height();
    };
}
```

#### 4. Record class'larının özellikleri nelerdir?

**Cevap:**
Record'lar, immutable data carrier class'larıdır ve boilerplate code'u azaltır.

**Özellikler:**
- **Immutable**: Tüm field'lar final
- **Auto-generated Methods**: equals(), hashCode(), toString()
- **Compact Syntax**: Minimal syntax ile class tanımı
- **Constructor**: Canonical constructor otomatik oluşturulur
- **Accessor Methods**: Field'lar için getter method'ları
- **Serializable**: Serializable interface'ini implement eder

**Kod Örneği:**
```java
// Traditional class
public class Person {
    private final String name;
    private final int age;
    private final String email;
    
    public Person(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getEmail() { return email; }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && 
               Objects.equals(name, person.name) && 
               Objects.equals(email, person.email);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age, email);
    }
    
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + ", email='" + email + "'}";
    }
}

// Record equivalent
public record Person(String name, int age, String email) {
    // All methods auto-generated
}

// Usage
Person person = new Person("John Doe", 30, "john@example.com");
System.out.println(person.name()); // Accessor method
System.out.println(person); // toString() output

// Record with validation
public record Person(String name, int age, String email) {
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
    }
    
    // Custom method
    public boolean isAdult() {
        return age >= 18;
    }
}

// Record with additional methods
public record Point(int x, int y) {
    public Point {
        // Compact constructor for validation
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be non-negative");
        }
    }
    
    public double distanceFromOrigin() {
        return Math.sqrt(x * x + y * y);
    }
    
    public Point translate(int dx, int dy) {
        return new Point(x + dx, y + dy);
    }
}
```

#### 5. Text Blocks'ın kullanım alanları nelerdir?

**Cevap:**
Text Blocks, çok satırlı string'leri daha okunabilir şekilde tanımlamayı sağlar.

**Kullanım Alanları:**
- **SQL Queries**: Çok satırlı SQL sorguları
- **JSON/XML**: Structured data
- **HTML Templates**: Web template'leri
- **Configuration**: YAML, properties dosyaları
- **Documentation**: Code documentation
- **Test Data**: Test case'leri için data

**Kod Örneği:**
```java
// Traditional string concatenation
String html = "<html>\n" +
              "  <body>\n" +
              "    <h1>Hello World</h1>\n" +
              "    <p>This is a paragraph</p>\n" +
              "  </body>\n" +
              "</html>";

// Text Block
String html = """
    <html>
      <body>
        <h1>Hello World</h1>
        <p>This is a paragraph</p>
      </body>
    </html>
    """;

// SQL Query
String sql = """
    SELECT u.id, u.name, u.email, p.title
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    WHERE u.active = true
    AND p.created_at > ?
    ORDER BY u.name, p.created_at DESC
    """;

// JSON
String json = """
    {
      "name": "John Doe",
      "age": 30,
      "email": "john@example.com",
      "address": {
        "street": "123 Main St",
        "city": "New York",
        "zipCode": "10001"
      }
    }
    """;

// Configuration
String config = """
    server:
      port: 8080
      context-path: /api
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/mydb
        username: root
        password: password
    """;

// Text Block with interpolation (Preview in Java 21)
String name = "John";
String email = "john@example.com";
String message = STR."""
    Hello \{name},
    
    Your email address is: \{email}
    
    Best regards,
    System
    """;
```

### Orta Seviye
6. Spring Boot 3.x'teki breaking change'ler nelerdir?
7. Jakarta EE migration'ının etkileri nelerdir?
8. Native Image support'u nasıl çalışır?
9. Spring Boot 3.x'teki yeni actuator endpoint'leri nelerdir?
10. GraalVM integration'ının faydaları nelerdir?
11. Spring Boot 3.x'teki yeni configuration properties'leri nelerdir?
12. Migration stratejisi nasıl planlanmalıdır?

---

## Spring Bean Scopes

### Temel Seviye

#### 1. Spring'deki bean scope'ları nelerdir?

**Cevap:**
Spring'de farklı bean scope'ları vardır ve her biri farklı lifecycle'a sahiptir.

**Bean Scope'ları:**
- **singleton**: Varsayılan scope, tek instance
- **prototype**: Her injection'da yeni instance
- **request**: HTTP request başına bir instance
- **session**: HTTP session başına bir instance
- **application**: ServletContext başına bir instance
- **websocket**: WebSocket session başına bir instance
- **thread**: Thread başına bir instance (custom scope)

**Kod Örneği:**
```java
// Singleton scope (default)
@Component
@Scope("singleton")
public class SingletonService {
    // Single instance for entire application
}

// Prototype scope
@Component
@Scope("prototype")
public class PrototypeService {
    // New instance for each injection
}

// Request scope
@Component
@Scope("request")
public class RequestService {
    // New instance for each HTTP request
}

// Session scope
@Component
@Scope("session")
public class SessionService {
    // New instance for each HTTP session
}

// Application scope
@Component
@Scope("application")
public class ApplicationService {
    // Single instance for entire servlet context
}

// WebSocket scope
@Component
@Scope("websocket")
public class WebSocketService {
    // New instance for each WebSocket session
}
```

#### 2. Singleton ve Prototype scope arasındaki fark nedir?

**Cevap:**

| Özellik | Singleton | Prototype |
|---------|-----------|-----------|
| **Instance Count** | Tek instance | Her injection'da yeni instance |
| **Memory Usage** | Düşük | Yüksek |
| **Thread Safety** | Dikkatli olunmalı | Her instance ayrı |
| **Performance** | Yüksek | Düşük |
| **State Management** | Shared state | Isolated state |
| **Lifecycle** | Application başlangıcında oluşturulur | Her injection'da oluşturulur |

**Detaylı Açıklama:**

**Singleton:**
- Application başlangıcında bir kez oluşturulur
- Tüm injection'larda aynı instance kullanılır
- Memory efficient
- Thread safety dikkat edilmeli
- Varsayılan scope

**Prototype:**
- Her injection'da yeni instance oluşturulur
- Her instance ayrı state'e sahip
- Memory overhead'i yüksek
- Thread safe (her instance ayrı)
- Explicit olarak belirtilmeli

**Kod Örneği:**
```java
// Singleton example
@Component
@Scope("singleton")
public class CounterService {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}

// Prototype example
@Component
@Scope("prototype")
public class PrototypeCounter {
    private int count = 0;
    
    public void increment() {
        count++; // No synchronization needed
    }
    
    public int getCount() {
        return count;
    }
}

// Usage
@RestController
public class TestController {
    
    @Autowired
    private CounterService singletonCounter; // Same instance
    
    @Autowired
    private PrototypeCounter prototypeCounter1; // New instance
    
    @Autowired
    private PrototypeCounter prototypeCounter2; // Another new instance
    
    @GetMapping("/test")
    public String test() {
        singletonCounter.increment();
        prototypeCounter1.increment();
        prototypeCounter2.increment();
        
        return String.format("Singleton: %d, Prototype1: %d, Prototype2: %d",
            singletonCounter.getCount(),
            prototypeCounter1.getCount(),
            prototypeCounter2.getCount());
    }
}
```

#### 3. Request scope ne zaman kullanılır?

**Cevap:**
Request scope, HTTP request başına bir instance oluşturur.

**Kullanım Alanları:**
- **Request-specific Data**: Request'e özel veri
- **User Context**: Kullanıcı bilgileri
- **Request Processing**: Request işleme sırasında state
- **Temporary Data**: Geçici veri saklama
- **Request Scoped Services**: Request'e özel servisler

**Kod Örneği:**
```java
// Request scope service
@Component
@Scope("request")
public class RequestContextService {
    private String requestId;
    private String userId;
    private long startTime;
    
    public RequestContextService() {
        this.requestId = UUID.randomUUID().toString();
        this.startTime = System.currentTimeMillis();
    }
    
    public String getRequestId() {
        return requestId;
    }
    
    public void setUserId(String userId) {
        this.userId = userId;
    }
    
    public String getUserId() {
        return userId;
    }
    
    public long getRequestDuration() {
        return System.currentTimeMillis() - startTime;
    }
}

// Usage in controller
@RestController
public class UserController {
    
    @Autowired
    private RequestContextService requestContext;
    
    @GetMapping("/user/profile")
    public ResponseEntity<UserProfile> getUserProfile() {
        // Set user context
        requestContext.setUserId("user123");
        
        // Process request
        UserProfile profile = processUserProfile();
        
        return ResponseEntity.ok(profile);
    }
    
    @GetMapping("/request-info")
    public ResponseEntity<Map<String, Object>> getRequestInfo() {
        Map<String, Object> info = new HashMap<>();
        info.put("requestId", requestContext.getRequestId());
        info.put("userId", requestContext.getUserId());
        info.put("duration", requestContext.getRequestDuration());
        
        return ResponseEntity.ok(info);
    }
}

// Request scope with proxy
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedService {
    // Service implementation
}
```

#### 4. Session scope'un kullanım alanları nelerdir?

**Cevap:**
Session scope, HTTP session başına bir instance oluşturur.

**Kullanım Alanları:**
- **User Session Data**: Kullanıcı session verileri
- **Shopping Cart**: Alışveriş sepeti
- **User Preferences**: Kullanıcı tercihleri
- **Session State**: Session state yönetimi
- **Multi-step Forms**: Çok adımlı formlar

**Kod Örneği:**
```java
// Session scope service
@Component
@Scope("session")
public class UserSessionService {
    private String userId;
    private String userName;
    private List<String> permissions;
    private Map<String, Object> sessionData;
    
    public UserSessionService() {
        this.permissions = new ArrayList<>();
        this.sessionData = new HashMap<>();
    }
    
    public void setUser(String userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }
    
    public String getUserId() {
        return userId;
    }
    
    public String getUserName() {
        return userName;
    }
    
    public void addPermission(String permission) {
        permissions.add(permission);
    }
    
    public boolean hasPermission(String permission) {
        return permissions.contains(permission);
    }
    
    public void setSessionData(String key, Object value) {
        sessionData.put(key, value);
    }
    
    public Object getSessionData(String key) {
        return sessionData.get(key);
    }
}

// Shopping cart example
@Component
@Scope("session")
public class ShoppingCart {
    private List<CartItem> items;
    private BigDecimal total;
    
    public ShoppingCart() {
        this.items = new ArrayList<>();
        this.total = BigDecimal.ZERO;
    }
    
    public void addItem(Product product, int quantity) {
        CartItem item = new CartItem(product, quantity);
        items.add(item);
        calculateTotal();
    }
    
    public void removeItem(Product product) {
        items.removeIf(item -> item.getProduct().getId().equals(product.getId()));
        calculateTotal();
    }
    
    public List<CartItem> getItems() {
        return new ArrayList<>(items);
    }
    
    public BigDecimal getTotal() {
        return total;
    }
    
    private void calculateTotal() {
        total = items.stream()
            .map(CartItem::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// Usage in controller
@RestController
public class CartController {
    
    @Autowired
    private ShoppingCart cart;
    
    @PostMapping("/cart/add")
    public ResponseEntity<String> addToCart(@RequestBody AddToCartRequest request) {
        cart.addItem(request.getProduct(), request.getQuantity());
        return ResponseEntity.ok("Item added to cart");
    }
    
    @GetMapping("/cart")
    public ResponseEntity<ShoppingCart> getCart() {
        return ResponseEntity.ok(cart);
    }
}
```

#### 5. Application scope nedir?

**Cevap:**
Application scope, ServletContext başına bir instance oluşturur.

**Özellikler:**
- **ServletContext Level**: ServletContext başına bir instance
- **Application-wide**: Tüm application'da paylaşılır
- **Lifecycle**: Application başlangıcında oluşturulur
- **Memory**: Singleton'a benzer memory kullanımı
- **Thread Safety**: Dikkatli olunmalı

**Kod Örneği:**
```java
// Application scope service
@Component
@Scope("application")
public class ApplicationConfigService {
    private Map<String, String> config;
    private List<String> supportedLanguages;
    private String applicationVersion;
    
    @PostConstruct
    public void init() {
        this.config = new HashMap<>();
        this.supportedLanguages = Arrays.asList("en", "tr", "fr", "de");
        this.applicationVersion = "1.0.0";
        
        // Load configuration
        loadConfiguration();
    }
    
    private void loadConfiguration() {
        config.put("app.name", "My Application");
        config.put("app.description", "Spring Boot Application");
        config.put("app.contact", "admin@example.com");
    }
    
    public String getConfig(String key) {
        return config.get(key);
    }
    
    public List<String> getSupportedLanguages() {
        return new ArrayList<>(supportedLanguages);
    }
    
    public String getApplicationVersion() {
        return applicationVersion;
    }
    
    public void updateConfig(String key, String value) {
        config.put(key, value);
    }
}

// Cache service example
@Component
@Scope("application")
public class ApplicationCacheService {
    private Map<String, Object> cache;
    private long lastCleanup;
    
    public ApplicationCacheService() {
        this.cache = new ConcurrentHashMap<>();
        this.lastCleanup = System.currentTimeMillis();
    }
    
    public void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public Object get(String key) {
        return cache.get(key);
    }
    
    public void remove(String key) {
        cache.remove(key);
    }
    
    public void clear() {
        cache.clear();
    }
    
    public int size() {
        return cache.size();
    }
    
    @Scheduled(fixedRate = 300000) // 5 minutes
    public void cleanup() {
        long now = System.currentTimeMillis();
        if (now - lastCleanup > 300000) {
            // Cleanup logic
            lastCleanup = now;
        }
    }
}

// Usage
@RestController
public class ConfigController {
    
    @Autowired
    private ApplicationConfigService configService;
    
    @Autowired
    private ApplicationCacheService cacheService;
    
    @GetMapping("/config")
    public ResponseEntity<Map<String, String>> getConfig() {
        Map<String, String> config = new HashMap<>();
        config.put("app.name", configService.getConfig("app.name"));
        config.put("version", configService.getApplicationVersion());
        config.put("languages", String.join(",", configService.getSupportedLanguages()));
        
        return ResponseEntity.ok(config);
    }
    
    @GetMapping("/cache/stats")
    public ResponseEntity<Map<String, Object>> getCacheStats() {
        Map<String, Object> stats = new HashMap<>();
        stats.put("size", cacheService.size());
        stats.put("lastCleanup", new Date());
        
        return ResponseEntity.ok(stats);
    }
}
```

### Orta Seviye
6. Custom scope nasıl oluşturulur?
7. Scope proxy'lerin kullanımı nasıldır?
8. Thread scope'un thread safety açısından önemi nedir?
9. WebSocket scope'un özellikleri nelerdir?
10. Scope'ların lifecycle'ı nasıl yönetilir?
11. @Scope annotation'ının parametreleri nelerdir?
12. Scope'ların memory leak'e neden olma durumları nelerdir?

---

## Spring Bean Annotations

### Temel Seviye

#### 1. @Component, @Service, @Repository ve @Controller arasındaki farklar nelerdir?

**Cevap:**
Bu dört annotation, Spring Framework'te bean tanımlamak için kullanılan temel stereotip annotation'lardır. Hepsi @Component annotation'ını meta-annotation olarak kullanır ve Spring container tarafından otomatik olarak tespit edilir.

**@Component:** En genel annotation'dır. Herhangi bir Spring-managed component için kullanılır. Spring'in component scanning mekanizması tarafından tespit edilir ve bean olarak kaydedilir.

**@Service:** İş mantığı (business logic) katmanındaki sınıflar için kullanılır. @Component'in özelleştirilmiş bir versiyonudur. Servis katmanındaki sınıfları işaretlemek için kullanılır ve kodun okunabilirliğini artırır.

**@Repository:** Veri erişim katmanındaki sınıflar için kullanılır. DAO (Data Access Object) sınıflarını işaretlemek için kullanılır. @Component'in özelleştirilmiş bir versiyonudur ve veri erişim katmanındaki sınıfları belirtir.

**@Controller:** Web katmanındaki sınıflar için kullanılır. MVC pattern'inde Controller rolündeki sınıfları işaretlemek için kullanılır. @Component'in özelleştirilmiş bir versiyonudur ve web isteklerini işleyen sınıfları belirtir.

**Fonksiyonel Farklar:**
- @Repository: Veri erişim katmanı, exception translation sağlar
- @Service: İş mantığı katmanı, transaction management için uygun
- @Controller: Web katmanı, HTTP isteklerini işler
- @Component: Genel amaçlı, herhangi bir katman için kullanılabilir

#### 2. @Autowired annotation'ının kullanım şekilleri nelerdir?

**Cevap:**
@Autowired annotation'ı, Spring Framework'te dependency injection yapmak için kullanılan temel annotation'dır. Farklı kullanım şekilleri vardır:

**Field Injection:** En yaygın kullanım şeklidir. Field'ların üzerine direkt olarak @Autowired yazılır. Spring container, uygun bean'i bulup field'a inject eder.

**Constructor Injection:** Constructor parametrelerine @Autowired yazılır. Spring 4.3'ten itibaren constructor'da tek parametre varsa @Autowired yazmaya gerek yoktur. En güvenli injection yöntemidir çünkü immutable object'ler oluşturulabilir.

**Setter Injection:** Setter methodlarına @Autowired yazılır. Spring container, setter method'u çağırarak dependency'yi inject eder.

**Method Injection:** Herhangi bir method'a @Autowired yazılabilir. Spring container, method'u çağırarak parametreleri inject eder.

**Array ve Collection Injection:** @Autowired ile array veya collection'lar inject edilebilir. Spring, uygun tipteki tüm bean'leri bulup collection'a ekler.

**Optional Injection:** @Autowired(required = false) ile optional injection yapılabilir. Bean bulunamazsa null değer inject edilir.

**Map Injection:** @Autowired ile Map<String, BeanType> şeklinde injection yapılabilir. Key olarak bean name, value olarak bean instance kullanılır.

#### 3. @Qualifier annotation'ının amacı nedir?

**Cevap:**
@Qualifier annotation'ı, aynı tipte birden fazla bean olduğunda hangi bean'in inject edileceğini belirtmek için kullanılır. Spring container'da aynı interface'i implement eden birden fazla bean olduğunda ambiguity (belirsizlik) oluşur.

**Temel Kullanım:** @Qualifier annotation'ı, bean'in adını veya özel bir qualifier değerini belirtir. Spring container, bu değere göre doğru bean'i seçer.

**Bean Name ile Kullanım:** @Qualifier("beanName") şeklinde kullanılır. Spring container, belirtilen isimdeki bean'i inject eder.

**Custom Qualifier:** @Qualifier annotation'ı ile custom qualifier'lar oluşturulabilir. Bu, daha anlamlı ve tip güvenli qualifier'lar oluşturmak için kullanılır.

**Multiple Qualifiers:** Birden fazla @Qualifier kullanılabilir. Spring container, tüm qualifier'ları karşılayan bean'i seçer.

**@Primary ile Birlikte Kullanım:** @Primary annotation'ı ile birlikte kullanıldığında, @Qualifier @Primary'yi override eder.

**Programmatic Qualifier:** @Qualifier annotation'ı programmatic olarak da kullanılabilir. Configuration class'larında @Bean method'larında kullanılır.

#### 4. @Primary annotation'ı ne zaman kullanılır?

**Cevap:**
@Primary annotation'ı, aynı tipte birden fazla bean olduğunda hangi bean'in öncelikli olarak inject edileceğini belirtmek için kullanılır. Spring container, @Primary ile işaretlenmiş bean'i diğerlerine tercih eder.

**Temel Kullanım:** @Primary annotation'ı, bean definition'ında kullanılır. Spring container, aynı tipte birden fazla bean olduğunda @Primary ile işaretlenmiş olanı seçer.

**Default Bean Belirleme:** Aynı interface'i implement eden birden fazla bean olduğunda, hangisinin default olarak kullanılacağını belirtir.

**@Qualifier ile Birlikte Kullanım:** @Qualifier annotation'ı @Primary'yi override eder. @Qualifier belirtilmişse, @Primary göz ardı edilir.

**Configuration Class'larda Kullanım:** @Configuration class'larında @Bean method'larında kullanılır. Bu method'dan dönen bean, o tipte primary bean olur.

**Multiple @Primary:** Aynı tipte birden fazla @Primary bean varsa, Spring exception fırlatır. Her tipte sadece bir @Primary bean olmalıdır.

**Inheritance:** @Primary annotation'ı, bean inheritance'ında da kullanılır. Parent bean @Primary ise, child bean'ler de primary olur.

#### 5. @Value annotation'ı ile property'ler nasıl inject edilir?

**Cevap:**
@Value annotation'ı, Spring Framework'te property değerlerini bean'lere inject etmek için kullanılır. External configuration dosyalarından, environment variable'lardan veya system property'lerinden değer alabilir.

**Temel Kullanım:** @Value annotation'ı, field'ların üzerine yazılır. Spring container, belirtilen property'yi bulup field'a inject eder.

**Property Placeholder:** @Value("${property.name}") şeklinde kullanılır. Spring, property.name key'ini arar ve değerini inject eder.

**Default Value:** @Value("${property.name:defaultValue}") şeklinde default değer belirtilebilir. Property bulunamazsa default değer kullanılır.

**SpEL (Spring Expression Language):** @Value("#{expression}") şeklinde SpEL kullanılabilir. Karmaşık expression'lar yazılabilir.

**Environment Variable:** @Value("${ENV_VAR}") şeklinde environment variable'lar inject edilebilir.

**System Property:** @Value("${system.property}") şeklinde system property'ler inject edilebilir.

**Type Conversion:** Spring, otomatik olarak type conversion yapar. String değerleri uygun tipe dönüştürür.

**Validation:** @Value ile inject edilen değerler, @Valid annotation'ı ile validate edilebilir.

### Orta Seviye

#### 6. @Configuration ve @Component arasındaki fark nedir?

**Cevap:**
@Configuration ve @Component annotation'ları arasında önemli farklar vardır, her ikisi de Spring container tarafından bean olarak yönetilir ancak farklı amaçlara hizmet ederler.

**@Component:** Genel amaçlı bir annotation'dır. Herhangi bir Spring-managed component için kullanılır. Component scanning ile otomatik olarak tespit edilir ve bean olarak kaydedilir.

**@Configuration:** Özel bir @Component türüdür. Configuration class'larını işaretlemek için kullanılır. @Configuration ile işaretlenmiş class'lar, Spring container'ın configuration metadata'sını sağlar.

**Temel Farklar:**
- @Configuration, @Component'in özelleştirilmiş bir versiyonudur
- @Configuration class'ları, @Bean method'ları içerebilir
- @Configuration class'ları, CGLIB proxy ile enhance edilir
- @Configuration class'ları, bean definition'ları sağlar

**CGLIB Enhancement:** @Configuration class'ları, Spring tarafından CGLIB ile enhance edilir. Bu, @Bean method'larının çağrıldığında aynı bean instance'ının döndürülmesini sağlar.

**Bean Definition:** @Configuration class'ları, @Bean method'ları ile bean definition'ları sağlar. @Component class'ları ise kendileri bean olur.

**Inter-bean References:** @Configuration class'larında, @Bean method'ları birbirini çağırabilir. Spring, bu çağrıları proxy ile intercept eder.

**Performance:** @Configuration class'ları, CGLIB enhancement nedeniyle @Component'ten daha yavaştır.

#### 7. @Bean annotation'ının @Component'ten farkı nedir?

**Cevap:**
@Bean ve @Component annotation'ları, Spring Framework'te bean tanımlamak için kullanılan farklı yaklaşımlardır. Her ikisi de Spring container'da bean oluşturur ancak farklı yöntemler kullanır.

**@Component:** Class-level annotation'dır. Sınıfın kendisini Spring bean'i olarak işaretler. Component scanning ile otomatik olarak tespit edilir.

**@Bean:** Method-level annotation'dır. @Configuration class'larında kullanılır ve method'un döndürdüğü object'i Spring bean'i olarak kaydeder.

**Temel Farklar:**
- @Component: Class-level, @Bean: Method-level
- @Component: Otomatik scanning, @Bean: Manuel tanımlama
- @Component: Sınıfın kendisi bean, @Bean: Method'un döndürdüğü object bean
- @Component: Constructor injection, @Bean: Method call

**Kullanım Senaryoları:**
- @Component: Kendi yazdığınız class'lar için
- @Bean: Third-party library'lerden gelen class'lar için
- @Bean: Karmaşık bean creation logic'i için
- @Bean: Conditional bean creation için

**Bean Creation Control:** @Bean annotation'ı ile bean creation process'i üzerinde daha fazla kontrol sağlanır. Method içinde karmaşık logic yazılabilir.

**Third-party Integration:** @Bean annotation'ı, third-party library'lerden gelen class'ları Spring bean'i yapmak için idealdir.

**Conditional Beans:** @Bean annotation'ı, @Conditional annotation'ları ile birlikte kullanılarak conditional bean creation sağlanır.

#### 8. @PostConstruct ve @PreDestroy annotation'larının işlevi nedir?

**Cevap:**
@PostConstruct ve @PreDestroy annotation'ları, Spring bean'lerinin yaşam döngüsünde önemli noktalarda çalışacak method'ları işaretlemek için kullanılır. Bu annotation'lar, JSR-250 standardının bir parçasıdır.

**@PostConstruct:** Bean'in initialization'ı tamamlandıktan sonra çalışacak method'u işaretler. Spring container, bean'i oluşturduktan ve tüm dependency'leri inject ettikten sonra bu method'u çağırır.

**@PreDestroy:** Bean'in destroy edilmeden önce çalışacak method'u işaretler. Spring container, bean'i destroy etmeden önce bu method'u çağırır.

**Yaşam Döngüsü Sırası:**
1. Bean instance oluşturulur
2. Dependency injection yapılır
3. @PostConstruct method'u çağrılır
4. Bean kullanıma hazır
5. @PreDestroy method'u çağrılır (container kapatılırken)
6. Bean destroy edilir

**@PostConstruct Kullanım Alanları:**
- Database connection'ları test etme
- Cache'leri initialize etme
- External service'lerle bağlantı kurma
- Bean'in kullanıma hazır olduğunu doğrulama

**@PreDestroy Kullanım Alanları:**
- Database connection'ları kapatma
- Cache'leri temizleme
- External service'lerle bağlantıları sonlandırma
- Resource'ları serbest bırakma

**Method Requirements:** Bu annotation'lar ile işaretlenmiş method'lar void döndürmeli, parametre almamalı ve exception fırlatmamalıdır.

**Multiple Methods:** Aynı bean'de birden fazla @PostConstruct veya @PreDestroy method'u olabilir. Spring, bunları sırayla çağırır.

#### 9. @Profile annotation'ının kullanım alanları nelerdir?

**Cevap:**
@Profile annotation'ı, Spring Framework'te farklı environment'larda farklı bean'lerin aktif olmasını sağlamak için kullanılır. Bu, development, test, production gibi farklı ortamlarda farklı konfigürasyonlar kullanmayı mümkün kılar.

**Temel Kullanım:** @Profile annotation'ı, bean'lerin hangi profile'larda aktif olacağını belirtir. Spring container, sadece aktif profile'a ait bean'leri oluşturur.

**Profile Activation:** Profile'lar, application.properties dosyasında veya environment variable'lar ile aktif edilir. Spring.active.profiles property'si ile belirtilir.

**Multiple Profiles:** Bir bean, birden fazla profile'da aktif olabilir. @Profile({"dev", "test"}) şeklinde kullanılır.

**Profile Expressions:** @Profile annotation'ı, SpEL expression'ları destekler. Karmaşık profile logic'i yazılabilir.

**Default Profile:** @Profile("default") ile default profile belirtilebilir. Hiçbir profile aktif değilse default profile kullanılır.

**Kullanım Alanları:**
- Database configuration'ları (dev, test, prod)
- External service endpoint'leri
- Logging configuration'ları
- Security configuration'ları
- Feature flag'ler

**Configuration Classes:** @Configuration class'ları da @Profile ile işaretlenebilir. Tüm class'ın bean'leri o profile'da aktif olur.

**Method-level Profiles:** @Bean method'ları da @Profile ile işaretlenebilir. Sadece o method'un bean'i o profile'da aktif olur.

**Profile Inheritance:** Profile'lar inherit edilebilir. Parent profile aktifse, child profile'lar da aktif olur.

#### 10. @Conditional annotation'ları nasıl çalışır?

**Cevap:**
@Conditional annotation'ları, Spring Framework'te conditional bean creation sağlamak için kullanılır. Bean'lerin oluşturulup oluşturulmayacağı, belirli koşullara bağlıdır.

**Temel Kullanım:** @Conditional annotation'ı, Condition interface'ini implement eden class'lar ile kullanılır. Bu class'lar, bean'in oluşturulup oluşturulmayacağına karar verir.

**Condition Interface:** Condition interface'i, matches method'u içerir. Bu method, ConditionContext ve AnnotatedTypeMetadata parametreleri alır ve boolean döndürür.

**Built-in Conditions:** Spring, birçok built-in condition sağlar:
- @ConditionalOnClass: Belirli class'ın classpath'te olup olmadığını kontrol eder
- @ConditionalOnMissingClass: Belirli class'ın classpath'te olmadığını kontrol eder
- @ConditionalOnBean: Belirli bean'in var olup olmadığını kontrol eder
- @ConditionalOnMissingBean: Belirli bean'in var olmadığını kontrol eder
- @ConditionalOnProperty: Belirli property'nin var olup olmadığını kontrol eder

**Custom Conditions:** Kendi condition'larınızı yazabilirsiniz. Condition interface'ini implement eden class'lar oluşturabilirsiniz.

**Multiple Conditions:** Birden fazla condition kullanılabilir. Tüm condition'lar true döndürmelidir.

**Condition Context:** ConditionContext, Spring container hakkında bilgi sağlar. Bean registry, environment, resource loader gibi bilgilere erişim sağlar.

**Annotated Type Metadata:** AnnotatedTypeMetadata, annotation'lar hakkında bilgi sağlar. Annotation attribute'larına erişim sağlar.

**Performance:** @Conditional annotation'ları, bean creation sırasında çalışır. Bu, startup time'ı etkileyebilir.

#### 11. @Lazy annotation'ının performans etkisi nedir?

**Cevap:**
@Lazy annotation'ı, Spring Framework'te lazy initialization sağlamak için kullanılır. Bean'ler, ilk kullanıldıklarında oluşturulur, application startup'ta değil.

**Temel Kullanım:** @Lazy annotation'ı, bean'lerin lazy olarak oluşturulmasını sağlar. Bean, ilk kez inject edildiğinde veya getBean() ile çağrıldığında oluşturulur.

**Startup Performance:** @Lazy annotation'ı, application startup time'ını azaltır. Tüm bean'ler startup'ta oluşturulmak yerine, ihtiyaç duyulduğunda oluşturulur.

**Memory Usage:** @Lazy annotation'ı, memory usage'ı optimize eder. Kullanılmayan bean'ler oluşturulmaz.

**Proxy Creation:** @Lazy bean'ler, proxy ile wrap edilir. Bu, bean'in lazy olarak oluşturulmasını sağlar.

**Circular Dependencies:** @Lazy annotation'ı, circular dependency problemlerini çözmek için kullanılabilir. Bean'lerden biri lazy olarak işaretlenirse, circular dependency çözülür.

**Configuration Classes:** @Configuration class'ları da @Lazy ile işaretlenebilir. Tüm class'ın bean'leri lazy olur.

**Method-level Lazy:** @Bean method'ları da @Lazy ile işaretlenebilir. Sadece o method'un bean'i lazy olur.

**Injection Points:** @Lazy annotation'ı, injection point'lerde de kullanılabilir. @Autowired ile birlikte kullanılır.

**Performance Trade-offs:** @Lazy annotation'ı, startup time'ı azaltır ancak runtime'da ilk kullanımda gecikme yaratır.

**Thread Safety:** @Lazy bean'ler, thread-safe olarak oluşturulur. Multiple thread'ler aynı anda bean'i oluşturmaya çalışırsa, sadece bir kez oluşturulur.

#### 12. @DependsOn annotation'ının kullanımı nasıldır?

**Cevap:**
@DependsOn annotation'ı, Spring Framework'te bean'ler arasında dependency relationship kurmak için kullanılır. Bir bean'in oluşturulmadan önce belirli bean'lerin oluşturulmasını garanti eder.

**Temel Kullanım:** @DependsOn annotation'ı, bean'in hangi bean'lerden önce oluşturulması gerektiğini belirtir. Spring container, dependency'leri sırayla oluşturur.

**Bean Creation Order:** @DependsOn annotation'ı, bean creation order'ını kontrol eder. Belirtilen bean'ler, bu bean'den önce oluşturulur.

**Multiple Dependencies:** Birden fazla bean dependency'si belirtilebilir. @DependsOn({"bean1", "bean2"}) şeklinde kullanılır.

**Circular Dependencies:** @DependsOn annotation'ı, circular dependency problemlerini çözmek için kullanılabilir. Bean'ler arasında explicit dependency relationship kurar.

**Initialization Order:** @DependsOn annotation'ı, bean'lerin initialization order'ını kontrol eder. @PostConstruct method'ları da bu sıraya göre çağrılır.

**Configuration Classes:** @Configuration class'ları da @DependsOn ile işaretlenebilir. Tüm class'ın bean'leri belirtilen dependency'leri bekler.

**Method-level Dependencies:** @Bean method'ları da @DependsOn ile işaretlenebilir. Sadece o method'un bean'i belirtilen dependency'leri bekler.

**Lazy Dependencies:** @DependsOn annotation'ı, lazy bean'lerle de çalışır. Lazy bean'ler, dependency'leri oluşturulduktan sonra oluşturulur.

**Performance Impact:** @DependsOn annotation'ı, bean creation order'ını kontrol ettiği için startup time'ı etkileyebilir.

**Use Cases:** @DependsOn annotation'ı, database initialization, cache setup, external service connection gibi durumlarda kullanılır.

---

## Spring Transactional

### Temel Seviye

#### 1. @Transactional annotation'ının amacı nedir?

**Cevap:**
@Transactional annotation'ı, Spring Framework'te declarative transaction management sağlamak için kullanılır. Bu annotation, method'ların veya class'ların transaction boundary'sini belirler ve Spring'in transaction management mekanizmasını aktif eder.

**Temel Amaç:** @Transactional annotation'ı, method'ların transaction içinde çalışmasını sağlar. Bu, database operation'larının atomic, consistent, isolated ve durable olmasını garanti eder.

**Declarative Approach:** @Transactional annotation'ı, declarative transaction management yaklaşımını kullanır. Programmatic transaction management'ten farklı olarak, transaction logic'i business logic'ten ayrılır.

**AOP Integration:** @Transactional annotation'ı, Spring AOP (Aspect-Oriented Programming) ile entegre çalışır. Spring, proxy oluşturarak transaction management'i sağlar.

**Method-level ve Class-level:** @Transactional annotation'ı, hem method level'da hem de class level'da kullanılabilir. Class level'da kullanıldığında, tüm public method'lar transaction içinde çalışır.

**Transaction Manager:** @Transactional annotation'ı, Spring'in transaction manager'ı ile çalışır. PlatformTransactionManager interface'ini implement eden class'lar kullanılır.

**Automatic Rollback:** @Transactional annotation'ı, runtime exception'lar durumunda otomatik rollback sağlar. Checked exception'lar için rollback yapılmaz.

**Performance:** @Transactional annotation'ı, database connection'ları optimize eder ve connection pooling'i etkin kullanır.

#### 2. Transaction'ın ACID özellikleri nelerdir?

**Cevap:**
ACID, transaction'ların sahip olması gereken dört temel özelliği ifade eder. Bu özellikler, database system'lerinde data integrity'yi sağlar.

**Atomicity (Atomiklik):** Transaction'ın tüm operation'ları ya tamamen başarılı olur ya da hiçbiri çalışmaz. Transaction'ın bir kısmı başarısız olursa, tüm transaction rollback edilir.

**Consistency (Tutarlılık):** Transaction'ın başlangıcında ve sonunda database tutarlı durumda olmalıdır. Transaction, database'in integrity constraint'lerini ihlal etmemelidir.

**Isolation (İzolasyon):** Eşzamanlı çalışan transaction'lar birbirini etkilememelidir. Her transaction, diğer transaction'ların intermediate state'lerini görmemelidir.

**Durability (Dayanıklılık):** Transaction commit edildikten sonra, yapılan değişiklikler kalıcı olmalıdır. System failure durumunda bile data kaybolmamalıdır.

**Atomicity Detayları:** Atomicity, "all or nothing" prensibini uygular. Transaction'ın herhangi bir adımı başarısız olursa, tüm transaction iptal edilir.

**Consistency Detayları:** Consistency, business rule'ların ve database constraint'lerin korunmasını sağlar. Transaction'lar, database'i geçersiz duruma getirmemelidir.

**Isolation Detayları:** Isolation, concurrency control sağlar. Farklı isolation level'ları vardır ve her biri farklı trade-off'lar sunar.

**Durability Detayları:** Durability, transaction'ın sonuçlarının kalıcı olmasını sağlar. WAL (Write-Ahead Logging) ve checkpointing gibi teknikler kullanılır.

#### 3. @Transactional'ın propagation davranışları nelerdir?

**Cevap:**
@Transactional annotation'ının propagation özelliği, mevcut bir transaction context'inde yeni bir transaction method'u çağrıldığında ne olacağını belirler. Spring, yedi farklı propagation davranışı sağlar.

**REQUIRED (Default):** Mevcut transaction varsa onu kullanır, yoksa yeni transaction oluşturur. En yaygın kullanılan propagation davranışıdır.

**REQUIRES_NEW:** Her zaman yeni transaction oluşturur. Mevcut transaction varsa suspend edilir ve yeni transaction başlatılır.

**SUPPORTS:** Mevcut transaction varsa onu kullanır, yoksa transaction olmadan çalışır. Transaction'ın varlığı opsiyoneldir.

**NOT_SUPPORTED:** Transaction'ı desteklemez. Mevcut transaction varsa suspend edilir ve method transaction olmadan çalışır.

**MANDATORY:** Mevcut transaction'ın varlığını zorunlu kılar. Transaction yoksa exception fırlatır.

**NEVER:** Transaction'ın varlığını yasaklar. Mevcut transaction varsa exception fırlatır.

**NESTED:** Nested transaction oluşturur. Mevcut transaction varsa savepoint oluşturur, yoksa yeni transaction başlatır.

**REQUIRED Kullanımı:** En güvenli ve yaygın kullanılan davranıştır. Çoğu business method için uygundur.

**REQUIRES_NEW Kullanımı:** Logging, auditing gibi işlemler için kullanılır. Ana transaction rollback olsa bile bu işlemler commit edilir.

**NESTED Kullanımı:** Partial rollback gereken durumlarda kullanılır. Sadece JDBC transaction manager'da desteklenir.

#### 4. Isolation level'ları nelerdir?

**Cevap:**
Isolation level'ları, eşzamanlı çalışan transaction'ların birbirini nasıl etkileyeceğini belirler. Her isolation level, farklı concurrency problem'lerini çözer ancak farklı performance trade-off'ları sunar.

**READ_UNCOMMITTED (En Düşük):** Transaction'lar, diğer transaction'ların henüz commit edilmemiş değişikliklerini okuyabilir. Dirty read, non-repeatable read ve phantom read problem'leri yaşanabilir.

**READ_COMMITTED:** Transaction'lar, sadece commit edilmiş değişiklikleri okuyabilir. Dirty read problemi çözülür ancak non-repeatable read ve phantom read problem'leri devam eder.

**REPEATABLE_READ:** Transaction'lar, aynı data'yı birden fazla kez okuduğunda aynı sonucu alır. Dirty read ve non-repeatable read problem'leri çözülür ancak phantom read problemi devam eder.

**SERIALIZABLE (En Yüksek):** Transaction'lar, serial olarak çalışıyormuş gibi davranır. Tüm concurrency problem'leri çözülür ancak performance en düşüktür.

**Dirty Read:** Bir transaction, diğer transaction'ın henüz commit edilmemiş değişikliklerini okur. Bu, geçersiz data okunmasına neden olabilir.

**Non-repeatable Read:** Bir transaction, aynı data'yı birden fazla kez okuduğunda farklı sonuçlar alır. Diğer transaction'lar araya girer ve data'yı değiştirir.

**Phantom Read:** Bir transaction, aynı query'yi birden fazla kez çalıştırdığında farklı sayıda row alır. Diğer transaction'lar yeni row'lar ekler veya mevcut row'ları siler.

**Performance Trade-offs:** Yüksek isolation level'ları, daha az concurrency problem'i yaşar ancak daha düşük performance sunar. Düşük isolation level'ları, daha yüksek performance sunar ancak daha fazla concurrency problem'i yaşar.

**Database Support:** Farklı database'ler, farklı isolation level'ları destekler. PostgreSQL, MySQL, Oracle gibi database'lerin farklı default isolation level'ları vardır.

#### 5. Rollback durumları nelerdir?

**Cevap:**
Rollback, transaction'ın iptal edilmesi ve tüm değişikliklerin geri alınması işlemidir. Spring Framework'te rollback, farklı durumlarda otomatik olarak veya manuel olarak tetiklenebilir.

**Otomatik Rollback:** Runtime exception'lar durumunda Spring otomatik olarak rollback yapar. Checked exception'lar için rollback yapılmaz.

**Manuel Rollback:** TransactionStatus object'i kullanılarak manuel olarak rollback yapılabilir. Programmatic transaction management'te kullanılır.

**Rollback Rules:** @Transactional annotation'ında rollbackFor ve noRollbackFor attribute'ları ile rollback kuralları belirlenebilir.

**Runtime Exception Rollback:** RuntimeException ve onun subclass'ları için otomatik rollback yapılır. Bu, Spring'in default davranışıdır.

**Checked Exception No Rollback:** Checked exception'lar için otomatik rollback yapılmaz. Bu exception'lar business logic'in bir parçası olarak kabul edilir.

**Custom Rollback Rules:** @Transactional(rollbackFor = CustomException.class) ile custom exception'lar için rollback belirlenebilir.

**No Rollback Rules:** @Transactional(noRollbackFor = BusinessException.class) ile belirli exception'lar için rollback yapılmaması sağlanabilir.

**Partial Rollback:** Nested transaction'larda partial rollback yapılabilir. Sadece nested transaction rollback edilir, parent transaction devam eder.

**Savepoint Rollback:** JDBC transaction manager'da savepoint'ler kullanılarak partial rollback yapılabilir.

**Transaction Status:** TransactionStatus object'i, transaction'ın durumu hakkında bilgi sağlar ve manuel rollback imkanı sunar.

**Rollback Timing:** Rollback, transaction'ın sonunda yapılır. Exception fırlatıldığında hemen rollback yapılmaz, transaction boundary'sinde yapılır.

### Orta Seviye

#### 6. Programmatic transaction management nasıl yapılır?

**Cevap:**
Programmatic transaction management, transaction'ların kod içinde manuel olarak yönetilmesi yaklaşımıdır. Declarative transaction management'ten farklı olarak, transaction logic'i business logic ile birlikte yazılır.

**TransactionTemplate:** Spring'in TransactionTemplate class'ı, programmatic transaction management için en yaygın kullanılan yöntemdir. Callback pattern kullanarak transaction'ları yönetir.

**PlatformTransactionManager:** PlatformTransactionManager interface'i, transaction'ları programmatic olarak yönetmek için kullanılır. begin(), commit(), rollback() method'ları sağlar.

**TransactionDefinition:** TransactionDefinition interface'i, transaction'ın özelliklerini (propagation, isolation, timeout) belirler.

**TransactionStatus:** TransactionStatus interface'i, transaction'ın durumu hakkında bilgi sağlar ve manuel rollback imkanı sunar.

**TransactionTemplate Kullanımı:** TransactionTemplate, execute() method'u ile callback alır. Callback içindeki kod transaction içinde çalışır.

**PlatformTransactionManager Kullanımı:** PlatformTransactionManager ile transaction'lar manuel olarak başlatılır, commit edilir veya rollback yapılır.

**Exception Handling:** Programmatic transaction management'te exception handling önemlidir. Try-catch block'ları kullanılarak rollback yapılır.

**Performance:** Programmatic transaction management, declarative'ten daha performanslıdır çünkü proxy overhead'i yoktur.

**Flexibility:** Programmatic transaction management, daha fazla flexibility sağlar. Karmaşık transaction logic'i yazılabilir.

**Use Cases:** Programmatic transaction management, conditional transaction logic, complex error handling, legacy system integration gibi durumlarda kullanılır.

#### 7. @Transactional'ın timeout özelliği nasıl çalışır?

**Cevap:**
@Transactional annotation'ının timeout özelliği, transaction'ın maksimum süresini belirler. Bu süre aşılırsa, transaction otomatik olarak rollback edilir.

**Timeout Tanımı:** Timeout, saniye cinsinden belirtilir. @Transactional(timeout = 30) şeklinde kullanılır.

**Default Timeout:** Timeout belirtilmezse, database'in default timeout değeri kullanılır. Bu değer genellikle sınırsızdır.

**Timeout Behavior:** Timeout süresi aşıldığında, Spring TransactionTimedOutException fırlatır ve transaction rollback edilir.

**Database Level Timeout:** Timeout, database level'da da uygulanabilir. Database connection timeout'u ile Spring timeout'u farklı olabilir.

**Nested Transaction Timeout:** Nested transaction'larda, parent transaction'ın timeout'u child transaction'ları da etkiler.

**Timeout Calculation:** Timeout, transaction başladığında hesaplanır. Method execution time'ı timeout'u aşarsa rollback yapılır.

**Performance Impact:** Timeout, long-running transaction'ları önler ve database resource'larını korur.

**Timeout vs Lock Timeout:** Transaction timeout ile database lock timeout farklıdır. Transaction timeout, tüm transaction'ı etkiler.

**Monitoring:** Timeout'lar, application monitoring'de önemli metrik'lerdir. Long-running transaction'lar performance problem'lerine işaret edebilir.

**Best Practices:** Timeout değerleri, business requirement'lara göre belirlenmelidir. Çok kısa timeout'lar false positive'lere neden olabilir.

#### 8. Nested transaction'lar nasıl yönetilir?

**Cevap:**
Nested transaction'lar, bir transaction içinde başka bir transaction oluşturma mekanizmasıdır. Spring, farklı propagation davranışları ile nested transaction'ları destekler.

**NESTED Propagation:** @Transactional(propagation = Propagation.NESTED) ile nested transaction oluşturulur. Mevcut transaction varsa savepoint oluşturur.

**Savepoint Mechanism:** Nested transaction'lar, savepoint'ler kullanarak çalışır. Savepoint, transaction'ın belirli bir noktasını işaretler.

**Partial Rollback:** Nested transaction rollback edildiğinde, sadece o transaction'ın değişiklikleri geri alınır. Parent transaction devam eder.

**JDBC Support:** Nested transaction'lar, sadece JDBC transaction manager'da desteklenir. JTA transaction manager'da desteklenmez.

**REQUIRES_NEW vs NESTED:** REQUIRES_NEW, yeni transaction oluşturur ve mevcut transaction'ı suspend eder. NESTED, savepoint oluşturur.

**Transaction Boundary:** Nested transaction'lar, parent transaction'ın boundary'si içinde çalışır. Parent transaction commit edilmeden nested transaction commit edilmez.

**Exception Handling:** Nested transaction'da exception fırlatıldığında, sadece o transaction rollback edilir. Parent transaction etkilenmez.

**Performance:** Nested transaction'lar, savepoint overhead'i nedeniyle biraz daha yavaştır.

**Use Cases:** Nested transaction'lar, optional operation'lar, audit logging, partial failure handling gibi durumlarda kullanılır.

**Limitations:** Nested transaction'lar, tüm database'lerde desteklenmez. MySQL, PostgreSQL gibi database'ler destekler.

#### 9. Transaction synchronization'ın işlevi nedir?

**Cevap:**
Transaction synchronization, transaction'ların yaşam döngüsünde belirli noktalarda callback'ler çalıştırma mekanizmasıdır. Bu, transaction'ların commit veya rollback edilmesi sırasında ek işlemler yapmayı sağlar.

**TransactionSynchronization Interface:** TransactionSynchronization interface'i, transaction callback'lerini tanımlar. beforeCommit(), afterCommit(), afterCompletion() method'ları sağlar.

**beforeCommit():** Transaction commit edilmeden önce çalışır. Bu method'da transaction'ı iptal etmek için exception fırlatılabilir.

**afterCommit():** Transaction başarıyla commit edildikten sonra çalışır. Bu method'da transaction'ı iptal etmek mümkün değildir.

**afterCompletion():** Transaction tamamlandıktan sonra (commit veya rollback) çalışır. Transaction'ın durumu (STATUS_COMMITTED veya STATUS_ROLLED_BACK) parametre olarak geçilir.

**Synchronization Registration:** TransactionSynchronizationManager.registerSynchronization() ile synchronization'lar kaydedilir.

**Resource Management:** Transaction synchronization, resource'ların transaction yaşam döngüsü ile senkronize edilmesini sağlar.

**Cache Invalidation:** Transaction commit edildikten sonra cache'lerin invalidate edilmesi için kullanılır.

**Event Publishing:** Transaction'ların başarılı olması durumunda event'lerin publish edilmesi için kullanılır.

**Cleanup Operations:** Transaction rollback edildiğinde cleanup operation'larının çalıştırılması için kullanılır.

**Ordering:** Birden fazla synchronization varsa, order property'si ile sıralama belirlenebilir.

**Thread Safety:** Transaction synchronization'lar, thread-safe olarak çalışır. Her thread'in kendi synchronization'ları vardır.

#### 10. @TransactionalEventListener'ın kullanımı nasıldır?

**Cevap:**
@TransactionalEventListener annotation'ı, transaction'ların yaşam döngüsünde event'lerin publish edilmesini sağlar. Bu, transaction'ların commit veya rollback edilmesi sırasında event-driven işlemler yapmayı mümkün kılar.

**Temel Kullanım:** @TransactionalEventListener annotation'ı, method'ları transaction event listener'ı olarak işaretler. Bu method'lar, transaction'ların belirli aşamalarında çalışır.

**Transaction Phase:** @TransactionalEventListener'da phase attribute'u ile event'in hangi transaction phase'inde çalışacağı belirlenir.

**Phase Options:** BEFORE_COMMIT, AFTER_COMMIT, AFTER_ROLLBACK, AFTER_COMPLETION phase'leri mevcuttur.

**BEFORE_COMMIT:** Transaction commit edilmeden önce çalışır. Bu phase'de transaction'ı iptal etmek için exception fırlatılabilir.

**AFTER_COMMIT:** Transaction başarıyla commit edildikten sonra çalışır. Bu phase'de transaction'ı iptal etmek mümkün değildir.

**AFTER_ROLLBACK:** Transaction rollback edildikten sonra çalışır. Bu phase'de rollback edilen transaction'ın bilgilerine erişim sağlanır.

**AFTER_COMPLETION:** Transaction tamamlandıktan sonra (commit veya rollback) çalışır. Transaction'ın durumu parametre olarak geçilir.

**Event Publishing:** @TransactionalEventListener, @EventListener ile birlikte kullanılır. Event'ler transaction context'inde publish edilir.

**Conditional Execution:** @TransactionalEventListener'da condition attribute'u ile event'in hangi koşullarda çalışacağı belirlenebilir.

**Fallback Execution:** @TransactionalEventListener'da fallbackExecution attribute'u ile transaction olmadığında da çalışması sağlanabilir.

**Use Cases:** @TransactionalEventListener, audit logging, cache invalidation, external service notification, email sending gibi durumlarda kullanılır.

**Performance:** @TransactionalEventListener, transaction'ların yaşam döngüsü ile entegre çalışır ve performanslıdır.

#### 11. Distributed transaction'lar nasıl yönetilir?

**Cevap:**
Distributed transaction'lar, birden fazla resource (database, message queue, external service) üzerinde çalışan transaction'lardır. Bu transaction'lar, ACID özelliklerini distributed environment'te sağlamak için özel mekanizmalar gerektirir.

**Two-Phase Commit (2PC):** 2PC, distributed transaction'ları yönetmek için kullanılan temel protokoldür. Prepare ve commit phase'lerinden oluşur.

**JTA (Java Transaction API):** JTA, Java'da distributed transaction'ları yönetmek için kullanılan API'dir. XA transaction'ları destekler.

**XA Transaction:** XA, distributed transaction standard'ıdır. XA resource'ları (XA datasource, XA connection) kullanılır.

**Transaction Manager:** Distributed transaction'lar için JTA transaction manager kullanılır. Atomikos, Bitronix gibi implementation'lar mevcuttur.

**Resource Manager:** Her resource (database, message queue) için XA resource manager gerekir. Bu, resource'ların 2PC protokolüne katılmasını sağlar.

**Prepare Phase:** Transaction manager, tüm resource'lara prepare mesajı gönderir. Resource'lar, transaction'ı commit edip edemeyeceklerini belirtir.

**Commit Phase:** Tüm resource'lar prepare'da başarılı olursa, transaction manager commit mesajı gönderir. Aksi takdirde rollback mesajı gönderir.

**Spring JTA Support:** Spring, JTA transaction'ları destekler. @Transactional annotation'ı JTA ile çalışır.

**Performance Issues:** Distributed transaction'lar, network latency ve coordination overhead nedeniyle yavaştır.

**Failure Handling:** Distributed transaction'larda failure handling karmaşıktır. Network failure, resource failure gibi durumlar ele alınmalıdır.

**Alternatives:** Distributed transaction'lar yerine saga pattern, event sourcing, eventual consistency gibi alternatifler kullanılabilir.

#### 12. Transaction template pattern'ının faydaları nelerdir?

**Cevap:**
Transaction template pattern, programmatic transaction management için kullanılan bir design pattern'dir. Bu pattern, transaction logic'ini business logic'ten ayırarak kod tekrarını azaltır ve transaction management'i standardize eder.

**Template Method Pattern:** Transaction template, template method pattern'ini kullanır. Transaction logic'i template'de, business logic'i callback'te bulunur.

**Code Reuse:** Transaction template, transaction management logic'ini tekrar kullanılabilir hale getirir. Her method'da aynı transaction logic'ini yazmaya gerek kalmaz.

**Consistency:** Transaction template, tüm transaction'ların tutarlı şekilde yönetilmesini sağlar. Aynı error handling, timeout, isolation level'ları kullanılır.

**Error Handling:** Transaction template, standardize edilmiş error handling sağlar. Exception'lar tutarlı şekilde handle edilir.

**Configuration:** Transaction template, transaction özelliklerini (timeout, isolation, propagation) merkezi olarak yapılandırır.

**Testing:** Transaction template, unit test'lerde mock'lanabilir. Business logic'i transaction logic'inden ayırarak test edilebilir hale getirir.

**Performance:** Transaction template, transaction management overhead'ini minimize eder. Proxy kullanmaz, direkt transaction manager kullanır.

**Flexibility:** Transaction template, farklı transaction özellikleri ile kullanılabilir. Her kullanım için farklı template instance'ı oluşturulabilir.

**Callback Pattern:** Transaction template, callback pattern kullanır. Business logic, callback olarak geçilir.

**Resource Management:** Transaction template, resource'ların (connection, transaction) doğru şekilde yönetilmesini sağlar.

**Use Cases:** Transaction template, complex transaction logic, conditional transaction, legacy system integration gibi durumlarda kullanılır.

---

## Java Release Features (8-21)

### Java 8

#### 1. Lambda expressions'ın syntax'ı nasıldır?

**Cevap:**
Lambda expressions, Java 8'de functional programming desteği sağlamak için tanıtılan önemli bir özelliktir. Anonymous function'ları kısa ve öz bir şekilde tanımlamayı mümkün kılar.

**Temel Syntax:** Lambda expression'lar, (parameter list) -> { body } formatında yazılır. Parameter list ve body kısımları farklı durumlarda farklı şekillerde yazılabilir.

**Parameter List:** Lambda expression'da parametreler parantez içinde belirtilir. Tek parametre varsa parantez opsiyoneldir. Parametre tipleri belirtilebilir veya compiler tarafından infer edilebilir.

**Arrow Operator:** -> operatörü, parametre listesini body'den ayırır. Bu operatör, lambda expression'ın temel karakteristiğidir.

**Body:** Lambda expression'ın body'si, tek expression veya statement block olabilir. Tek expression varsa return keyword'ü ve süslü parantezler opsiyoneldir.

**Functional Interface:** Lambda expression'lar, functional interface'ler ile çalışır. Functional interface, tek abstract method'a sahip interface'dir.

**Type Inference:** Java compiler, lambda expression'ların tiplerini context'ten çıkarabilir. Bu, daha temiz kod yazmayı sağlar.

**Variable Capture:** Lambda expression'lar, enclosing scope'taki effectively final variable'ları capture edebilir. Bu, closure functionality sağlar.

**Method References:** Lambda expression'lar, method reference'lar ile değiştirilebilir. Bu, daha okunabilir kod sağlar.

**Performance:** Lambda expression'lar, anonymous inner class'lardan daha performanslıdır. JVM optimizasyonları sayesinde overhead azalır.

**Use Cases:** Lambda expression'lar, collection processing, event handling, functional programming patterns gibi durumlarda kullanılır.

#### 2. Stream API'nin temel operasyonları nelerdir?

**Cevap:**
Stream API, Java 8'de collection'lar üzerinde functional-style operation'lar yapmayı sağlayan güçlü bir API'dir. Stream'ler, data processing pipeline'ları oluşturmayı mümkün kılar.

**Stream Creation:** Stream'ler, collection'lardan, array'lerden, generator function'lardan veya builder pattern ile oluşturulabilir. Collection.stream() method'u en yaygın kullanılan yöntemdir.

**Intermediate Operations:** Intermediate operation'lar, stream'i dönüştürür ve yeni stream döndürür. Bu operation'lar lazy'dir ve terminal operation çağrılana kadar çalışmaz.

**Terminal Operations:** Terminal operation'lar, stream'i consume eder ve sonuç üretir. Bu operation'lar çağrıldığında, tüm pipeline çalışır.

**Filter Operation:** filter() method'u, stream'deki element'leri predicate'e göre filtreler. Sadece predicate'i karşılayan element'ler geçer.

**Map Operation:** map() method'u, stream'deki her element'i başka bir forma dönüştürür. Transformation function'ı uygular.

**Reduce Operation:** reduce() method'u, stream'deki element'leri tek bir değere indirger. Accumulator function kullanır.

**Collect Operation:** collect() method'u, stream'deki element'leri collection'a toplar. Collector interface'i kullanır.

**ForEach Operation:** forEach() method'u, stream'deki her element için action uygular. Side effect'li operation'dır.

**Distinct Operation:** distinct() method'u, stream'deki duplicate element'leri kaldırır. equals() method'unu kullanır.

**Sorted Operation:** sorted() method'u, stream'deki element'leri sıralar. Natural ordering veya custom comparator kullanır.

**Limit/Skip Operations:** limit() ve skip() method'ları, stream'deki element sayısını kontrol eder.

**Parallel Streams:** Stream'ler, parallel() method'u ile parallel processing yapabilir. Multi-core CPU'lardan faydalanır.

#### 3. Optional class'ının kullanımı nasıldır?

**Cevap:**
Optional class'ı, Java 8'de null pointer exception'larını önlemek ve null safety sağlamak için tanıtılan container class'ıdır. Bir değerin var olup olmadığını belirtir.

**Temel Konsept:** Optional, bir değerin mevcut olup olmadığını wrap eder. Null değer yerine Optional.empty() kullanılır.

**Optional Creation:** Optional.of(), Optional.ofNullable(), Optional.empty() method'ları ile Optional instance'ları oluşturulur.

**of() Method:** Optional.of(value) method'u, null olmayan değerler için kullanılır. Null değer geçilirse NullPointerException fırlatır.

**ofNullable() Method:** Optional.ofNullable(value) method'u, null olabilen değerler için kullanılır. Null değer geçilirse empty Optional döndürür.

**empty() Method:** Optional.empty() method'u, boş Optional instance'ı döndürür.

**Value Access:** get(), orElse(), orElseGet(), orElseThrow() method'ları ile Optional'dan değer alınır.

**get() Method:** get() method'u, Optional'da değer varsa döndürür, yoksa NoSuchElementException fırlatır.

**orElse() Method:** orElse(defaultValue) method'u, Optional'da değer varsa döndürür, yoksa default değer döndürür.

**orElseGet() Method:** orElseGet(supplier) method'u, Optional'da değer varsa döndürür, yoksa supplier'dan değer alır.

**orElseThrow() Method:** orElseThrow() method'u, Optional'da değer varsa döndürür, yoksa exception fırlatır.

**Functional Operations:** map(), flatMap(), filter() method'ları ile Optional üzerinde functional operation'lar yapılır.

**map() Method:** map() method'u, Optional'da değer varsa transformation uygular, yoksa empty Optional döndürür.

**flatMap() Method:** flatMap() method'u, Optional'da değer varsa transformation uygular ve Optional döndürür, yoksa empty Optional döndürür.

**filter() Method:** filter() method'u, Optional'da değer varsa predicate'i test eder, yoksa empty Optional döndürür.

**ifPresent() Method:** ifPresent() method'u, Optional'da değer varsa action uygular, yoksa hiçbir şey yapmaz.

**Best Practices:** Optional, return type olarak kullanılmalı, field olarak kullanılmamalıdır. Null check'ler yerine Optional kullanılmalıdır.

#### 4. Method references'ın türleri nelerdir?

**Cevap:**
Method references, Java 8'de lambda expression'ların daha kısa ve okunabilir alternatifi olarak tanıtılan özelliktir. Mevcut method'ları referans etmeyi sağlar.

**Temel Syntax:** Method reference'lar, Class::method formatında yazılır. :: operatörü, method reference'ı belirtir.

**Static Method Reference:** Class::staticMethod formatında static method'lar referans edilir. Örnek: Math::max, String::valueOf.

**Instance Method Reference:** object::instanceMethod formatında instance method'lar referans edilir. Örnek: System.out::println.

**Instance Method Reference (Class):** Class::instanceMethod formatında instance method'lar referans edilir. İlk parametre olarak instance geçilir.

**Constructor Reference:** Class::new formatında constructor'lar referans edilir. Örnek: ArrayList::new, String::new.

**Array Constructor Reference:** Type[]::new formatında array constructor'lar referans edilir. Örnek: String[]::new.

**Static Method Reference Kullanımı:** Static method'lar, functional interface'lerde kullanılabilir. Method signature'ı uyumlu olmalıdır.

**Instance Method Reference Kullanımı:** Instance method'lar, object üzerinden referans edilir. Object'in method'u çağrılır.

**Class Instance Method Reference:** Class üzerinden instance method referans edilir. İlk parametre olarak instance geçilir.

**Constructor Reference Kullanımı:** Constructor'lar, Supplier functional interface'inde kullanılabilir. Factory pattern'de yararlıdır.

**Array Constructor Reference Kullanımı:** Array constructor'lar, IntFunction functional interface'inde kullanılabilir.

**Method Reference vs Lambda:** Method reference'lar, lambda expression'lardan daha okunabilirdir. Aynı functionality'yi sağlar.

**Performance:** Method reference'lar, lambda expression'larla aynı performance'a sahiptir. Compiler tarafından optimize edilir.

**Use Cases:** Method reference'lar, Stream API'de, functional interface'lerde, event handling'de kullanılır.

#### 5. Default methods'ın interface'lerdeki rolü nedir?

**Cevap:**
Default methods, Java 8'de interface'lere concrete method implementation'ları ekleme imkanı sağlayan özelliktir. Bu, interface'lerin backward compatibility'sini korurken yeni functionality eklemeyi mümkün kılar.

**Temel Konsept:** Default methods, interface'lerde default keyword'ü ile tanımlanan concrete method'lardır. Bu method'lar, implementing class'lar tarafından override edilebilir.

**Default Keyword:** default keyword'ü, interface method'larında concrete implementation belirtir. Bu, interface'lerin evolution'ını sağlar.

**Backward Compatibility:** Default methods, mevcut implementing class'ları bozmadan interface'lere yeni method'lar eklemeyi sağlar.

**Multiple Inheritance:** Default methods, Java'da multiple inheritance benzeri functionality sağlar. Bir class, birden fazla interface'den default method inherit edebilir.

**Diamond Problem:** Default methods, diamond problem'e neden olabilir. Aynı method birden fazla interface'de default implementation'a sahipse conflict oluşur.

**Conflict Resolution:** Diamond problem durumunda, implementing class'da method override edilmelidir. Aksi takdirde compile error oluşur.

**Super Keyword:** Default method'larda super keyword'ü kullanılarak parent interface'in method'u çağrılabilir.

**Static Methods:** Interface'lerde static method'lar da tanımlanabilir. Bu method'lar, interface name üzerinden çağrılır.

**Private Methods:** Java 9'da interface'lerde private method'lar da tanımlanabilir. Bu, default method'ların code reuse'ını sağlar.

**Functional Interface Evolution:** Default methods, functional interface'lerin evolution'ını sağlar. Yeni method'lar eklenebilir.

**Use Cases:** Default methods, Collection interface'inde, Stream API'de, Comparator interface'inde kullanılır.

**Best Practices:** Default methods, interface'lerin evolution'ı için kullanılmalıdır. Business logic için kullanılmamalıdır.

### Java 9-17

#### 6. Module system'ın faydaları nelerdir?

**Cevap:**
Java Module System (Project Jigsaw), Java 9'da tanıtılan önemli bir özelliktir. Java platform'unu modüler hale getirerek daha iyi encapsulation, dependency management ve performance sağlar.

**Temel Konsept:** Module system, Java application'larını modüllere böler. Her modül, kendi public API'sini tanımlar ve diğer modüllerle explicit dependency'ler kurar.

**module-info.java:** Her modül, module-info.java dosyası ile tanımlanır. Bu dosya, modülün adını, export ettiği package'ları ve ihtiyaç duyduğu modülleri belirtir.

**Encapsulation:** Module system, strong encapsulation sağlar. Internal package'lar, modül dışından erişilemez. Bu, API stability'yi artırır.

**Dependency Management:** Module system, explicit dependency management sağlar. Modüller, ihtiyaç duydukları diğer modülleri açıkça belirtir.

**JRE Modularization:** Java 9'da JRE, modüllere bölünmüştür. Bu, daha küçük runtime image'ları oluşturmayı mümkün kılar.

**JLink Tool:** jlink tool'u, custom runtime image'ları oluşturmayı sağlar. Sadece gerekli modüller dahil edilir.

**Performance Benefits:** Module system, startup time'ı azaltır ve memory usage'ı optimize eder. Lazy loading ve dead code elimination sağlar.

**Security:** Module system, security'yi artırır. Internal API'lara erişim kısıtlanır ve reflection kullanımı kontrol edilir.

**Backward Compatibility:** Module system, backward compatibility sağlar. Mevcut application'lar, module olmadan da çalışabilir.

**Migration:** Mevcut application'lar, gradual olarak module system'e migrate edilebilir. Automatic module'lar ile başlanabilir.

**Use Cases:** Module system, large application'lar, library'ler, microservices gibi durumlarda kullanılır.

**Best Practices:** Module system, clear API boundary'leri, minimal dependencies, proper encapsulation için kullanılmalıdır.

#### 7. var keyword'ünün kullanım alanları nelerdir?

**Cevap:**
var keyword'ü, Java 10'da tanıtılan Local Variable Type Inference özelliğidir. Compiler'ın variable type'ını otomatik olarak çıkarmasını sağlar.

**Temel Kullanım:** var keyword'ü, local variable'larda kullanılır. Compiler, variable'ın type'ını initializer'dan çıkarır.

**Type Inference:** var keyword'ü, type inference sağlar. Compiler, variable'ın type'ını context'ten belirler.

**Local Variables Only:** var keyword'ü, sadece local variable'larda kullanılabilir. Field'lar, method parameter'ları, return type'lar için kullanılamaz.

**Initializer Required:** var keyword'ü kullanıldığında, variable'ın initializer'ı olmalıdır. Compiler, type'ı initializer'dan çıkarır.

**Readability:** var keyword'ü, kodun okunabilirliğini artırabilir. Özellikle complex generic type'larda yararlıdır.

**Generic Types:** var keyword'ü, generic type'larda özellikle yararlıdır. Diamond operator ile birlikte kullanılır.

**Lambda Expressions:** var keyword'ü, lambda expression'larda kullanılabilir. Parameter type'ları infer edilir.

**Stream API:** var keyword'ü, Stream API'de yararlıdır. Complex stream operation'larında type'lar infer edilir.

**Limitations:** var keyword'ü, null initializer ile kullanılamaz. Compiler, type'ı çıkaramaz.

**Best Practices:** var keyword'ü, type'ın açık olduğu durumlarda kullanılmalıdır. Obscure type'lar için kullanılmamalıdır.

**Performance:** var keyword'ü, runtime performance'ı etkilemez. Sadece compile time'da type inference yapar.

**IDE Support:** Modern IDE'ler, var keyword'ü ile iyi entegrasyon sağlar. Type information'ı gösterir.

#### 8. Switch expressions'ın geleneksel switch'ten farkı nedir?

**Cevap:**
Switch expressions, Java 14'te tanıtılan ve Java 17'de stabilize edilen özelliktir. Geleneksel switch statement'larından farklı olarak expression olarak kullanılabilir ve değer döndürür.

**Expression vs Statement:** Switch expressions, expression olarak kullanılır ve değer döndürür. Geleneksel switch, statement olarak kullanılır.

**Arrow Syntax:** Switch expressions, -> operatörü ile yazılır. Bu, daha kısa ve okunabilir syntax sağlar.

**Multiple Values:** Switch expressions, case'lerde multiple value'lar destekler. Virgül ile ayrılmış değerler kullanılabilir.

**Yield Keyword:** Switch expressions, yield keyword'ü ile değer döndürür. Return keyword'ü kullanılmaz.

**Exhaustiveness:** Switch expressions, exhaustive olmalıdır. Tüm possible value'lar handle edilmelidir.

**Pattern Matching:** Switch expressions, pattern matching destekler. Type pattern'lar kullanılabilir.

**Null Handling:** Switch expressions, null value'ları handle edebilir. Null case'i eklenebilir.

**Performance:** Switch expressions, geleneksel switch'ten daha performanslıdır. Compiler optimizasyonları sağlar.

**Readability:** Switch expressions, daha okunabilir kod sağlar. Arrow syntax ve yield keyword'ü ile.

**Use Cases:** Switch expressions, value mapping, type checking, state machine gibi durumlarda kullanılır.

**Migration:** Geleneksel switch statement'lar, switch expression'lara migrate edilebilir.

**Best Practices:** Switch expressions, simple value mapping için kullanılmalıdır. Complex logic için geleneksel switch tercih edilmelidir.

#### 9. Text blocks'ın syntax'ı nasıldır?

**Cevap:**
Text blocks, Java 13'te preview olarak tanıtılan ve Java 15'te stabilize edilen özelliktir. Multi-line string'leri daha okunabilir şekilde yazmayı sağlar.

**Temel Syntax:** Text blocks, """ ile başlar ve """ ile biter. Üç tırnak işareti kullanılır.

**Multi-line Strings:** Text blocks, multi-line string'leri kolayca yazmayı sağlar. Escape character'lara gerek yoktur.

**Indentation:** Text blocks, indentation'ı preserve eder. Leading whitespace, common indentation olarak hesaplanır.

**Escape Sequences:** Text blocks, geleneksel escape sequence'ları destekler. \n, \t, \" gibi.

**New Escape Sequences:** Text blocks, yeni escape sequence'lar sağlar. \s (space), \ (line continuation) gibi.

**String Interpolation:** Text blocks, string interpolation desteklemez. String.format() veya concatenation kullanılır.

**Performance:** Text blocks, geleneksel string concatenation'dan daha performanslıdır. Compile time'da optimize edilir.

**Use Cases:** Text blocks, SQL query'ler, JSON, XML, HTML, configuration file'lar gibi durumlarda kullanılır.

**Readability:** Text blocks, kodun okunabilirliğini artırır. Multi-line content'ler daha temiz görünür.

**Migration:** Geleneksel string'ler, text block'lara migrate edilebilir.

**Best Practices:** Text blocks, multi-line content için kullanılmalıdır. Single line string'ler için geleneksel syntax tercih edilmelidir.

**IDE Support:** Modern IDE'ler, text block'lar için syntax highlighting ve formatting sağlar.

#### 10. Record class'larının özellikleri nelerdir?

**Cevap:**
Record class'ları, Java 14'te preview olarak tanıtılan ve Java 16'da stabilize edilen özelliktir. Immutable data carrier'ları için kısa ve öz syntax sağlar.

**Temel Konsept:** Record class'ları, immutable data class'ları için kısa syntax sağlar. Constructor, getter, equals, hashCode, toString method'ları otomatik olarak oluşturulur.

**Record Declaration:** Record class'ları, record keyword'ü ile tanımlanır. Component'ler parantez içinde belirtilir.

**Automatic Methods:** Record class'ları, otomatik olarak constructor, getter, equals, hashCode, toString method'ları oluşturur.

**Immutability:** Record class'ları, immutable'dır. Component'ler final'dır ve değiştirilemez.

**Compact Constructor:** Record class'ları, compact constructor destekler. Validation logic'i eklenebilir.

**Custom Methods:** Record class'ları, custom method'lar eklenebilir. Static method'lar, instance method'lar desteklenir.

**Inheritance:** Record class'ları, başka class'lardan inherit edemez. Sadece interface'ler implement edebilir.

**Serialization:** Record class'ları, Serializable interface'ini implement edebilir. Otomatik serialization sağlar.

**Performance:** Record class'ları, geleneksel class'lardan daha performanslıdır. JVM optimizasyonları sağlar.

**Use Cases:** Record class'ları, DTO'lar, value object'ler, data transfer object'ler gibi durumlarda kullanılır.

**Best Practices:** Record class'ları, immutable data için kullanılmalıdır. Mutable state için geleneksel class'lar tercih edilmelidir.

**IDE Support:** Modern IDE'ler, record class'ları için code generation ve refactoring sağlar.

### Java 18-21

#### 11. Virtual threads'ın geleneksel thread'lerden farkı nedir?

**Cevap:**
Virtual threads, Java 19'da preview olarak tanıtılan ve Java 21'de stabilize edilen Project Loom'un temel özelliğidir. Geleneksel platform thread'lerinden farklı olarak lightweight, scalable ve efficient thread'ler sağlar.

**Temel Konsept:** Virtual threads, JVM tarafından yönetilen lightweight thread'lerdir. Platform thread'lerinden farklı olarak, çok daha az memory kullanır ve çok daha hızlı oluşturulur.

**Memory Usage:** Virtual threads, geleneksel thread'lerden çok daha az memory kullanır. Platform thread'ler ~1MB memory kullanırken, virtual thread'ler sadece birkaç KB kullanır.

**Creation Cost:** Virtual thread'ler, platform thread'lerinden çok daha hızlı oluşturulur. Milyonlarca virtual thread oluşturulabilir.

**Scheduling:** Virtual threads, M:N scheduling model kullanır. Çok sayıda virtual thread, az sayıda platform thread üzerinde çalışır.

**Blocking Behavior:** Virtual threads, blocking operation'larda platform thread'i bloke etmez. Virtual thread suspend edilir ve platform thread başka virtual thread'ler için kullanılır.

**I/O Operations:** Virtual threads, I/O operation'larında özellikle etkilidir. Blocking I/O'da platform thread'i bloke etmez.

**Thread Pool:** Virtual threads, geleneksel thread pool'lara gerek kalmaz. Her request için yeni virtual thread oluşturulabilir.

**Compatibility:** Virtual threads, mevcut Java code ile uyumludur. Existing API'ler değişmeden kullanılabilir.

**Performance:** Virtual threads, high-throughput application'lar için özellikle etkilidir. I/O-bound workload'larda büyük performance artışı sağlar.

**Use Cases:** Virtual threads, web server'lar, microservices, reactive application'lar gibi durumlarda kullanılır.

**Best Practices:** Virtual threads, I/O-bound task'lar için kullanılmalıdır. CPU-intensive task'lar için platform thread'ler tercih edilmelidir.

**Migration:** Mevcut application'lar, virtual thread'lere gradual olarak migrate edilebilir.

#### 12. Pattern matching'in switch'teki kullanımı nasıldır?

**Cevap:**
Pattern matching for switch, Java 17'de preview olarak tanıtılan ve Java 21'de stabilize edilen özelliktir. Switch expression'larında type pattern'ları kullanarak daha güçlü ve güvenli kod yazmayı sağlar.

**Temel Konsept:** Pattern matching for switch, switch expression'larında type pattern'ları kullanır. Object'in type'ını kontrol eder ve otomatik casting yapar.

**Type Patterns:** Type pattern'lar, instanceof operator'ü ile benzer şekilde çalışır. Object'in type'ını kontrol eder ve variable'a bind eder.

**Automatic Casting:** Pattern matching, otomatik casting sağlar. Type pattern match edildiğinde, object otomatik olarak cast edilir.

**Exhaustiveness:** Pattern matching, exhaustive olmalıdır. Tüm possible type'lar handle edilmelidir.

**Null Handling:** Pattern matching, null value'ları handle edebilir. Null case'i eklenebilir.

**Guarded Patterns:** Pattern matching, guarded pattern'lar destekler. Pattern'a ek condition'lar eklenebilir.

**Nested Patterns:** Pattern matching, nested pattern'lar destekler. Complex object structure'ları match edilebilir.

**Record Patterns:** Pattern matching, record pattern'lar destekler. Record component'leri destructure edilebilir.

**Array Patterns:** Pattern matching, array pattern'lar destekler. Array element'leri match edilebilir.

**Performance:** Pattern matching, geleneksel instanceof check'lerden daha performanslıdır. Compiler optimizasyonları sağlar.

**Readability:** Pattern matching, daha okunabilir kod sağlar. Type checking ve casting tek statement'da yapılır.

**Use Cases:** Pattern matching, type checking, data processing, visitor pattern gibi durumlarda kullanılır.

**Best Practices:** Pattern matching, type-safe code için kullanılmalıdır. Complex type hierarchy'lerde özellikle yararlıdır.

#### 13. Sealed classes'ın amacı nedir?

**Cevap:**
Sealed classes, Java 15'te preview olarak tanıtılan ve Java 17'de stabilize edilen özelliktir. Class hierarchy'lerini kontrol etmeyi ve pattern matching ile birlikte type safety sağlamayı mümkün kılar.

**Temel Konsept:** Sealed classes, hangi class'ların extend edebileceğini veya implement edebileceğini kontrol eder. Class hierarchy'sini sınırlar.

**Sealed Declaration:** Sealed class'lar, sealed keyword'ü ile tanımlanır. permits clause ile hangi class'ların extend edebileceği belirtilir.

**Permits Clause:** permits clause, sealed class'ın hangi class'lar tarafından extend edilebileceğini belirtir. Compile time'da kontrol edilir.

**Final Classes:** Sealed class'ları extend eden class'lar, final, sealed veya non-sealed olmalıdır.

**Non-sealed Classes:** Non-sealed class'lar, sealed class'ları extend edebilir ancak kendileri sealed değildir.

**Pattern Matching:** Sealed classes, pattern matching ile birlikte kullanılır. Exhaustive pattern matching sağlar.

**Type Safety:** Sealed classes, type safety sağlar. Compile time'da tüm possible type'lar kontrol edilir.

**API Design:** Sealed classes, API design'da yararlıdır. Public API'lerin extension'ını kontrol eder.

**Use Cases:** Sealed classes, algebraic data type'lar, state machine'ler, visitor pattern gibi durumlarda kullanılır.

**Best Practices:** Sealed classes, controlled inheritance için kullanılmalıdır. Public API'lerde özellikle yararlıdır.

**Performance:** Sealed classes, runtime performance'ı etkilemez. Sadece compile time'da type checking sağlar.

**IDE Support:** Modern IDE'ler, sealed classes için pattern matching ve refactoring sağlar.

#### 14. Foreign Function & Memory API'nin kullanım alanları nelerdir?

**Cevap:**
Foreign Function & Memory API (Project Panama), Java 19'da preview olarak tanıtılan ve Java 21'de stabilize edilen özelliktir. Java'dan native code'u çağırmayı ve native memory'yi yönetmeyi sağlar.

**Temel Konsept:** Foreign Function & Memory API, Java'dan native library'leri çağırmayı ve native memory'yi yönetmeyi sağlar. JNI'den daha güvenli ve performanslıdır.

**Foreign Function Interface:** Foreign Function Interface, Java'dan native function'ları çağırmayı sağlar. C, C++, Rust gibi dillerden function'lar çağrılabilir.

**Memory Management:** Foreign Memory API, native memory'yi yönetmeyi sağlar. Off-heap memory allocation ve deallocation yapılabilir.

**Type Safety:** Foreign Function & Memory API, type safety sağlar. Compile time'da type checking yapılır.

**Performance:** Foreign Function & Memory API, JNI'den daha performanslıdır. Zero-cost abstraction sağlar.

**Memory Segments:** Memory segments, native memory'yi yönetmek için kullanılır. Bounded, unbounded, shared segment'ler desteklenir.

**Function Descriptors:** Function descriptor'lar, native function'ların signature'larını tanımlar. Parameter type'ları ve return type'ı belirtir.

**Symbol Lookup:** Symbol lookup, native library'lerdeki symbol'leri bulmayı sağlar. Function'lar, variable'lar lookup edilebilir.

**Use Cases:** Foreign Function & Memory API, system programming, performance-critical application'lar, legacy system integration gibi durumlarda kullanılır.

**Best Practices:** Foreign Function & Memory API, native library integration için kullanılmalıdır. Memory management dikkatli yapılmalıdır.

**Security:** Foreign Function & Memory API, güvenli native code execution sağlar. Memory corruption'ları önler.

**Migration:** JNI code'ları, Foreign Function & Memory API'ye migrate edilebilir.

---

## Java Access Modifiers

### Temel Seviye

#### 1. Java'daki access modifier'ları listeleyin.

**Cevap:**
Java'da dört temel access modifier bulunur. Bu modifier'lar, class'lar, method'lar, field'lar ve constructor'ların erişilebilirlik seviyesini belirler.

**public:** En geniş erişim seviyesidir. public olarak tanımlanan element'ler, her yerden erişilebilir. Aynı package içinde, farklı package'lardan, inheritance ile her yerden erişim mümkündür.

**private:** En kısıtlı erişim seviyesidir. private olarak tanımlanan element'ler, sadece aynı class içinden erişilebilir. Class dışından, inheritance ile bile erişim mümkün değildir.

**protected:** Orta seviye erişim sağlar. protected olarak tanımlanan element'ler, aynı package içinden ve inheritance ile farklı package'lardan erişilebilir. Aynı class içinden de erişim mümkündür.

**default (package-private):** Modifier belirtilmediğinde default erişim seviyesi kullanılır. default element'ler, sadece aynı package içinden erişilebilir. Farklı package'lardan erişim mümkün değildir.

**Access Level Hierarchy:** Erişim seviyeleri, en genişten en kısıtlıya doğru şu sırayla sıralanır: public > protected > default > private.

**Visibility Rules:** Her access modifier'ın kendine özgü visibility kuralları vardır. Bu kurallar, encapsulation ve information hiding prensiplerini destekler.

**Best Practices:** Access modifier'lar, minimum necessary access prensibine göre seçilmelidir. Mümkün olan en kısıtlı erişim seviyesi kullanılmalıdır.

**Security:** Access modifier'lar, application security'sini artırır. Internal implementation detail'ları gizler.

**Maintainability:** Access modifier'lar, code maintainability'yi artırır. API boundary'lerini net bir şekilde tanımlar.

**Performance:** Access modifier'lar, runtime performance'ı etkilemez. Sadece compile time'da erişim kontrolü yapar.

#### 2. public, private, protected ve default arasındaki farklar nelerdir?

**Cevap:**
Java'daki dört access modifier'ın her biri farklı erişim seviyeleri sağlar. Bu farklar, encapsulation ve information hiding prensiplerini destekler.

**public Modifier:** public modifier'ı, en geniş erişim seviyesini sağlar. public element'ler, her yerden erişilebilir. Aynı class, aynı package, farklı package, inheritance ile her yerden erişim mümkündür.

**private Modifier:** private modifier'ı, en kısıtlı erişim seviyesini sağlar. private element'ler, sadece aynı class içinden erişilebilir. Class dışından, inheritance ile bile erişim mümkün değildir.

**protected Modifier:** protected modifier'ı, orta seviye erişim sağlar. protected element'ler, aynı package içinden ve inheritance ile farklı package'lardan erişilebilir. Aynı class içinden de erişim mümkündür.

**default Modifier:** default modifier'ı, package-private erişim sağlar. default element'ler, sadece aynı package içinden erişilebilir. Farklı package'lardan erişim mümkün değildir.

**Erişim Matrisi:** 
- Aynı class: public, private, protected, default - hepsi erişilebilir
- Aynı package: public, protected, default - erişilebilir; private - erişilemez
- Farklı package (inheritance): public, protected - erişilebilir; private, default - erişilemez
- Farklı package (inheritance yok): public - erişilebilir; private, protected, default - erişilemez

**Inheritance Etkisi:** protected modifier'ı, inheritance ile farklı package'lardan erişim sağlar. Diğer modifier'lar inheritance'dan etkilenmez.

**Package Boundary:** default modifier'ı, package boundary'sini korur. Package dışından erişimi engeller.

**API Design:** public modifier'ı, public API'ler için kullanılır. private modifier'ı, internal implementation için kullanılır.

**Best Practices:** Access modifier'lar, minimum necessary access prensibine göre seçilmelidir. Mümkün olan en kısıtlı erişim seviyesi kullanılmalıdır.

#### 3. Class level'da hangi modifier'lar kullanılabilir?

**Cevap:**
Java'da class level'da kullanılabilecek access modifier'lar sınırlıdır. Class'ların erişim seviyesi, package structure ve inheritance ile ilgilidir.

**public Class:** public class'lar, her yerden erişilebilir. Farklı package'lardan import edilebilir ve kullanılabilir. Bir Java dosyasında sadece bir public class olabilir.

**default Class:** default class'lar, sadece aynı package içinden erişilebilir. Farklı package'lardan erişim mümkün değildir. Bir Java dosyasında birden fazla default class olabilir.

**private Class:** Class level'da private modifier kullanılamaz. private modifier, sadece nested class'larda kullanılabilir.

**protected Class:** Class level'da protected modifier kullanılamaz. protected modifier, sadece nested class'larda kullanılabilir.

**Nested Class'lar:** Nested class'larda tüm access modifier'lar kullanılabilir. public, private, protected, default modifier'lar nested class'larda geçerlidir.

**Abstract Class:** Abstract class'lar, public veya default olabilir. private veya protected abstract class'lar oluşturulamaz.

**Final Class:** Final class'lar, public veya default olabilir. private veya protected final class'lar oluşturulamaz.

**File Naming:** public class'ların dosya adı, class adı ile aynı olmalıdır. default class'ların dosya adı farklı olabilir.

**Package Structure:** Class'ların erişim seviyesi, package structure'ı etkiler. public class'lar, package dışından erişilebilir.

**Best Practices:** Class'lar, mümkün olan en kısıtlı erişim seviyesi ile tanımlanmalıdır. Sadece gerekli olduğunda public yapılmalıdır.

**API Design:** public class'lar, public API'nin bir parçasıdır. Backward compatibility korunmalıdır.

#### 4. Method ve field'larda access control nasıl çalışır?

**Cevap:**
Java'da method'lar ve field'lar için access control, encapsulation prensiplerini destekler. Her access modifier'ın kendine özgü erişim kuralları vardır.

**Method Access Control:** Method'ların erişim seviyesi, çağrılabilirliklerini belirler. public method'lar her yerden çağrılabilir, private method'lar sadece aynı class içinden çağrılabilir.

**Field Access Control:** Field'ların erişim seviyesi, değerlerine erişimi belirler. public field'lar her yerden erişilebilir, private field'lar sadece aynı class içinden erişilebilir.

**Getter/Setter Pattern:** private field'lar, getter ve setter method'lar ile erişilebilir hale getirilir. Bu, encapsulation sağlar.

**Inheritance Etkisi:** protected method'lar ve field'lar, inheritance ile farklı package'lardan erişilebilir. private method'lar ve field'lar inheritance'dan etkilenmez.

**Method Overriding:** Override edilen method'lar, parent class'taki method'dan daha kısıtlı olamaz. Daha geniş erişim seviyesi sağlayabilir.

**Field Hiding:** Field'lar override edilmez, hide edilir. Child class'taki field, parent class'taki field'ı hide eder.

**Static Method'lar:** Static method'lar, class level'da erişim kontrolüne tabidir. Instance oluşturmadan çağrılabilir.

**Static Field'lar:** Static field'lar, class level'da erişim kontrolüne tabidir. Instance oluşturmadan erişilebilir.

**Final Method'lar:** Final method'lar override edilemez. Access modifier'ı inheritance'ı etkilemez.

**Final Field'lar:** Final field'lar, değer atandıktan sonra değiştirilemez. Access modifier'ı değer atamasını etkilemez.

**Best Practices:** Method'lar ve field'lar, mümkün olan en kısıtlı erişim seviyesi ile tanımlanmalıdır. Encapsulation prensipleri uygulanmalıdır.

#### 5. Package-private'ın anlamı nedir?

**Cevap:**
Package-private, Java'da default access modifier'ın sağladığı erişim seviyesidir. Bu, package boundary'sini koruyan önemli bir encapsulation mekanizmasıdır.

**Temel Konsept:** Package-private, modifier belirtilmediğinde kullanılan default erişim seviyesidir. Bu element'ler, sadece aynı package içinden erişilebilir.

**Package Boundary:** Package-private element'ler, package boundary'sini korur. Package dışından erişimi engeller.

**Erişim Kuralları:** Package-private element'ler, aynı package içindeki tüm class'lardan erişilebilir. Farklı package'lardan erişim mümkün değildir.

**Inheritance Etkisi:** Package-private element'ler, inheritance ile farklı package'lardan erişilemez. Inheritance, package boundary'sini aşamaz.

**Class Level:** Package-private class'lar, sadece aynı package içinden import edilebilir ve kullanılabilir.

**Method Level:** Package-private method'lar, sadece aynı package içinden çağrılabilir. Farklı package'lardan çağrılamaz.

**Field Level:** Package-private field'lar, sadece aynı package içinden erişilebilir. Farklı package'lardan erişilemez.

**Constructor Level:** Package-private constructor'lar, sadece aynı package içinden çağrılabilir. Farklı package'lardan instance oluşturulamaz.

**Use Cases:** Package-private, internal implementation detail'ları gizlemek için kullanılır. Package içi utility'ler için uygundur.

**Best Practices:** Package-private, package içi element'ler için kullanılmalıdır. Public API'nin bir parçası olmamalıdır.

**Security:** Package-private, package level security sağlar. Internal API'lerin dışarıdan erişimini engeller.

**Maintainability:** Package-private, code maintainability'yi artırır. Package içi değişikliklerin dış etkisini azaltır.

### Orta Seviye

#### 6. Access modifier'ların inheritance'daki etkisi nedir?

**Cevap:**
Access modifier'lar, inheritance mekanizmasında önemli rol oynar. Child class'ların parent class'ların member'larına erişimini kontrol eder ve inheritance hierarchy'sinde erişim kurallarını belirler.

**Inheritance Erişim Kuralları:** Child class'lar, parent class'ların member'larına erişim yaparken access modifier'ları dikkate alır. private member'lar inheritance ile erişilemez, protected member'lar inheritance ile erişilebilir.

**private Member'lar:** private member'lar, inheritance ile erişilemez. Child class'lar, parent class'ların private member'larına doğrudan erişim yapamaz. Sadece public veya protected method'lar üzerinden erişim mümkündür.

**protected Member'lar:** protected member'lar, inheritance ile erişilebilir. Child class'lar, parent class'ların protected member'larına doğrudan erişim yapabilir. Bu, farklı package'lardan bile mümkündür.

**public Member'lar:** public member'lar, inheritance ile erişilebilir. Child class'lar, parent class'ların public member'larına doğrudan erişim yapabilir. Bu, her yerden mümkündür.

**default Member'lar:** default member'lar, inheritance ile erişilemez. Child class'lar, parent class'ların default member'larına sadece aynı package içinden erişim yapabilir.

**Method Overriding:** Override edilen method'lar, parent class'taki method'dan daha kısıtlı olamaz. Daha geniş erişim seviyesi sağlayabilir. private method'lar override edilemez.

**Field Hiding:** Field'lar override edilmez, hide edilir. Child class'taki field, parent class'taki field'ı hide eder. Access modifier'ı inheritance'ı etkilemez.

**Constructor Inheritance:** Constructor'lar inherit edilmez. Child class'lar, parent class'ların constructor'larını çağırabilir ancak inherit edemez.

**Abstract Method'lar:** Abstract method'lar, child class'larda implement edilmelidir. Access modifier'ı inheritance'ı etkilemez.

**Interface Implementation:** Interface'ler implement edilirken, tüm method'lar public olarak implement edilmelidir. Interface method'ları public'tir.

**Best Practices:** Inheritance'da access modifier'lar, encapsulation prensiplerini korumalıdır. Mümkün olan en kısıtlı erişim seviyesi kullanılmalıdır.

#### 7. Interface'lerde access modifier kullanımı nasıldır?

**Cevap:**
Interface'lerde access modifier kullanımı, Java'nın farklı versiyonlarında değişiklik göstermiştir. Java 8'den itibaren interface'lerde daha fazla flexibility sağlanmıştır.

**Interface Method'ları:** Interface method'ları, varsayılan olarak public abstract'tır. public ve abstract keyword'leri yazılmasa bile otomatik olarak eklenir.

**Java 8 Öncesi:** Java 8 öncesinde, interface'lerde sadece public abstract method'lar tanımlanabilirdi. private, protected, default method'lar desteklenmezdi.

**Java 8 Sonrası:** Java 8'den itibaren, interface'lerde default method'lar ve static method'lar tanımlanabilir. Bu method'lar public'tir.

**Default Method'lar:** Default method'lar, interface'lerde concrete implementation sağlar. Bu method'lar public'tir ve public keyword'ü yazılmasa bile otomatik olarak eklenir.

**Static Method'lar:** Static method'lar, interface'lerde tanımlanabilir. Bu method'lar public'tir ve interface name üzerinden çağrılır.

**Java 9 Sonrası:** Java 9'dan itibaren, interface'lerde private method'lar da tanımlanabilir. Bu method'lar, default method'ların code reuse'ını sağlar.

**Private Method'lar:** Private method'lar, interface'lerde sadece default method'lar tarafından çağrılabilir. Interface dışından erişilemez.

**Interface Field'ları:** Interface field'ları, varsayılan olarak public static final'dır. public, static, final keyword'leri yazılmasa bile otomatik olarak eklenir.

**Nested Interface'ler:** Nested interface'ler, tüm access modifier'ları kullanabilir. public, private, protected, default modifier'lar nested interface'lerde geçerlidir.

**Access Control:** Interface'lerde access control, implementation class'ları etkiler. Interface method'ları, implementing class'larda public olarak implement edilmelidir.

**Best Practices:** Interface'lerde access modifier'lar, API design'ı etkiler. Public API'ler için uygun erişim seviyeleri seçilmelidir.

**Backward Compatibility:** Interface'lerde access modifier değişiklikleri, backward compatibility'yi etkileyebilir. Dikkatli yapılmalıdır.

#### 8. Nested class'larda access control nasıl çalışır?

**Cevap:**
Nested class'larda access control, outer class ile inner class arasındaki erişim kurallarını belirler. Nested class'lar, outer class'ın member'larına erişim yapabilir ve kendi access modifier'larına sahip olabilir.

**Nested Class Türleri:** Java'da dört tür nested class vardır: static nested class, inner class, local class, anonymous class. Her birinin farklı access control kuralları vardır.

**Static Nested Class:** Static nested class'lar, outer class'ın static member'larına erişim yapabilir. Outer class'ın instance member'larına erişim yapamaz. Kendi access modifier'larına sahip olabilir.

**Inner Class:** Inner class'lar, outer class'ın tüm member'larına erişim yapabilir. Outer class'ın instance member'larına doğrudan erişim yapabilir. Kendi access modifier'larına sahip olabilir.

**Local Class:** Local class'lar, enclosing method'un local variable'larına erişim yapabilir. Sadece effectively final variable'lara erişim yapabilir. Access modifier'ları yoktur.

**Anonymous Class:** Anonymous class'lar, enclosing method'un local variable'larına erişim yapabilir. Sadece effectively final variable'lara erişim yapabilir. Access modifier'ları yoktur.

**Outer Class Erişimi:** Nested class'lar, outer class'ın member'larına erişim yaparken access modifier'ları dikkate alır. private member'lar bile erişilebilir.

**Access Modifier'lar:** Nested class'lar, tüm access modifier'ları kullanabilir. public, private, protected, default modifier'lar nested class'larda geçerlidir.

**Private Nested Class:** Private nested class'lar, sadece outer class içinden erişilebilir. Outer class dışından erişim mümkün değildir.

**Protected Nested Class:** Protected nested class'lar, aynı package içinden ve inheritance ile farklı package'lardan erişilebilir.

**Public Nested Class:** Public nested class'lar, her yerden erişilebilir. Outer class'ın access modifier'ından bağımsızdır.

**Best Practices:** Nested class'lar, mümkün olan en kısıtlı erişim seviyesi ile tanımlanmalıdır. Sadece gerekli olduğunda public yapılmalıdır.

**Use Cases:** Nested class'lar, helper class'lar, event handler'lar, iterator'lar gibi durumlarda kullanılır.

#### 9. Reflection ile access modifier'lar nasıl kontrol edilir?

**Cevap:**
Java Reflection API, access modifier'ları runtime'da kontrol etmeyi sağlar. Class, method, field, constructor'ların access modifier'larını programmatic olarak öğrenmek mümkündür.

**Modifier Class:** java.lang.reflect.Modifier class'ı, access modifier'ları kontrol etmek için kullanılır. Bu class, static method'lar sağlar.

**isPublic() Method:** Modifier.isPublic() method'u, bir member'ın public olup olmadığını kontrol eder. int modifier değeri alır ve boolean döndürür.

**isPrivate() Method:** Modifier.isPrivate() method'u, bir member'ın private olup olmadığını kontrol eder. int modifier değeri alır ve boolean döndürür.

**isProtected() Method:** Modifier.isProtected() method'u, bir member'ın protected olup olmadığını kontrol eder. int modifier değeri alır ve boolean döndürür.

**isPackagePrivate() Method:** Modifier.isPackagePrivate() method'u, bir member'ın package-private olup olmadığını kontrol eder. int modifier değeri alır ve boolean döndürür.

**getModifiers() Method:** Class, Method, Field, Constructor class'larının getModifiers() method'u, member'ın modifier'larını int olarak döndürür.

**toString() Method:** Modifier.toString() method'u, modifier'ları string olarak döndürür. Human-readable format sağlar.

**Access Control:** Reflection ile access modifier'lar kontrol edilebilir ancak değiştirilemez. Access modifier'lar, compile time'da belirlenir.

**Security Manager:** Security Manager, reflection kullanımını kontrol edebilir. Access modifier'ları bypass etmeyi engelleyebilir.

**Performance:** Reflection, normal method call'lardan daha yavaştır. Access modifier kontrolü de overhead yaratır.

**Use Cases:** Reflection, framework'ler, testing tool'ları, debugging tool'ları gibi durumlarda kullanılır.

**Best Practices:** Reflection, sadece gerekli olduğunda kullanılmalıdır. Access modifier kontrolü, security açısından dikkatli yapılmalıdır.

**Security Considerations:** Reflection, access modifier'ları bypass edebilir. Security açısından dikkatli kullanılmalıdır.

#### 10. Access modifier'ların performance etkisi var mıdır?

**Cevap:**
Access modifier'lar, Java'da compile time'da kontrol edilir ve runtime performance'ı doğrudan etkilemez. Ancak dolaylı olarak performance'ı etkileyebilir.

**Compile Time Control:** Access modifier'lar, compile time'da kontrol edilir. Runtime'da erişim kontrolü yapılmaz. Bu, performance overhead'i yaratmaz.

**JVM Optimizasyonları:** JVM, access modifier'ları dikkate alarak optimizasyonlar yapabilir. private method'lar, inline edilebilir.

**Method Inlining:** private method'lar, JVM tarafından inline edilebilir. Bu, method call overhead'ini azaltır.

**Field Access:** private field'lar, direct access ile erişilebilir. public field'lar da aynı şekilde erişilebilir. Performance farkı yoktur.

**Getter/Setter Overhead:** private field'lar, getter/setter method'lar üzerinden erişilirse method call overhead'i yaratır. Direct access daha hızlıdır.

**Reflection Overhead:** Reflection ile access modifier kontrolü, normal method call'lardan daha yavaştır. Bu, reflection'ın kendisinden kaynaklanır.

**JIT Compilation:** JIT compiler, access modifier'ları dikkate alarak optimizasyonlar yapabilir. private method'lar, daha agresif optimize edilebilir.

**Memory Layout:** Access modifier'lar, memory layout'ı etkilemez. Field'ların memory'deki sırası access modifier'larından bağımsızdır.

**Garbage Collection:** Access modifier'lar, garbage collection'ı etkilemez. Object'lerin erişilebilirliği access modifier'larından bağımsızdır.

**Best Practices:** Access modifier'lar, performance için değil, encapsulation için kullanılmalıdır. Performance optimization'ları access modifier'lar üzerinden yapılmamalıdır.

**Profiling:** Performance problem'leri, access modifier'lar üzerinden değil, algorithm ve data structure'lar üzerinden çözülmelidir.

**Modern JVM:** Modern JVM'ler, access modifier'ları dikkate alarak çok gelişmiş optimizasyonlar yapar. Performance farkları minimaldir.

---

## Java Garbage Collection

### Temel Seviye

#### 1. Garbage Collection'ın amacı nedir?

**Cevap:**
Garbage Collection (GC), Java'da otomatik memory management sağlayan önemli bir mekanizmadır. Programcıların manuel olarak memory allocation ve deallocation yapmasına gerek kalmadan, kullanılmayan object'lerin memory'den temizlenmesini sağlar.

**Temel Amaç:** Garbage Collection'ın temel amacı, heap memory'deki kullanılmayan object'leri otomatik olarak temizlemektir. Bu, memory leak'leri önler ve application'ın memory kullanımını optimize eder.

**Memory Management:** GC, heap memory'yi yönetir. Stack memory, GC tarafından yönetilmez. Heap'teki object'ler, GC tarafından temizlenir.

**Automatic Cleanup:** GC, kullanılmayan object'leri otomatik olarak tespit eder ve temizler. Programcı, manuel olarak memory temizliği yapmak zorunda değildir.

**Memory Leak Prevention:** GC, memory leak'leri önler. Kullanılmayan object'ler, memory'de kalarak memory leak'e neden olabilir. GC, bu object'leri temizler.

**Performance Optimization:** GC, memory kullanımını optimize eder. Kullanılmayan object'ler temizlenerek, yeni object'ler için memory alanı açılır.

**Object Lifecycle:** GC, object'lerin yaşam döngüsünü yönetir. Object'ler, reference'ları kalmadığında garbage olarak işaretlenir ve temizlenir.

**Heap Fragmentation:** GC, heap fragmentation'ı azaltır. Kullanılmayan object'ler temizlenerek, memory'de boşluklar oluşur ve bu boşluklar birleştirilir.

**Application Stability:** GC, application stability'sini artırır. Memory leak'ler, application'ın crash olmasına neden olabilir. GC, bu durumu önler.

**Developer Productivity:** GC, developer productivity'sini artırır. Programcılar, memory management ile uğraşmak zorunda kalmaz.

**Best Practices:** GC, doğru kullanıldığında application performance'ını artırır. Yanlış kullanım, performance problem'lerine neden olabilir.

#### 2. Java'da hangi GC algoritmaları bulunur?

**Cevap:**
Java'da farklı GC algoritmaları bulunur. Her algoritma, farklı use case'ler için optimize edilmiştir ve farklı performance karakteristikleri sunar.

**Serial GC:** Serial GC, tek thread ile çalışan basit bir GC algoritmasıdır. Small application'lar ve single-core system'ler için uygundur. Stop-the-world pause'ları yaratır.

**Parallel GC:** Parallel GC, multiple thread ile çalışan GC algoritmasıdır. Throughput'u artırmak için tasarlanmıştır. Stop-the-world pause'ları yaratır ancak daha hızlıdır.

**Concurrent Mark Sweep (CMS):** CMS, concurrent collection sağlayan GC algoritmasıdır. Application'ın çalışmasını durdurmadan garbage collection yapar. Low latency sağlar.

**G1GC (Garbage First):** G1GC, Java 7'de tanıtılan modern GC algoritmasıdır. Large heap'ler için optimize edilmiştir. Predictable pause time'lar sağlar.

**ZGC:** ZGC, Java 11'de tanıtılan ultra-low latency GC algoritmasıdır. Very large heap'ler için tasarlanmıştır. Sub-millisecond pause time'lar sağlar.

**Shenandoah GC:** Shenandoah GC, Java 12'de tanıtılan concurrent GC algoritmasıdır. Low latency ve high throughput sağlar. Large heap'ler için optimize edilmiştir.

**Epsilon GC:** Epsilon GC, Java 11'de tanıtılan no-op GC algoritmasıdır. Memory allocation yapar ancak garbage collection yapmaz. Testing ve benchmarking için kullanılır.

**Algorithm Selection:** GC algoritması seçimi, application'ın requirement'larına göre yapılır. Throughput, latency, memory usage gibi faktörler dikkate alınır.

**JVM Versions:** Farklı JVM versiyonları, farklı GC algoritmalarını destekler. Yeni versiyonlar, daha gelişmiş GC algoritmaları sunar.

**Best Practices:** GC algoritması seçimi, application profiling sonuçlarına göre yapılmalıdır. Default GC, çoğu durumda yeterlidir.

#### 3. Young Generation ve Old Generation arasındaki fark nedir?

**Cevap:**
Java heap memory, Young Generation ve Old Generation olmak üzere iki ana bölüme ayrılır. Bu ayrım, object'lerin yaşam sürelerine göre yapılır ve GC efficiency'sini artırır.

**Young Generation:** Young Generation, yeni oluşturulan object'lerin bulunduğu heap bölümüdür. Object'lerin çoğu kısa süre yaşar, bu nedenle Young Generation'da sık sık GC yapılır.

**Old Generation:** Old Generation, uzun süre yaşayan object'lerin bulunduğu heap bölümüdür. Object'ler, Young Generation'dan Old Generation'a promote edilir. Old Generation'da daha az sıklıkla GC yapılır.

**Object Lifecycle:** Object'ler, önce Young Generation'da oluşturulur. Belirli sayıda GC cycle'ından sonra Old Generation'a promote edilir.

**GC Frequency:** Young Generation'da sık sık GC yapılır çünkü object'lerin çoğu kısa süre yaşar. Old Generation'da daha az sıklıkla GC yapılır çünkü object'ler uzun süre yaşar.

**GC Algorithm:** Young Generation'da genellikle copying algorithm kullanılır. Old Generation'da genellikle mark-sweep-compact algorithm kullanılır.

**Memory Size:** Young Generation, genellikle heap'in küçük bir bölümünü kaplar. Old Generation, genellikle heap'in büyük bir bölümünü kaplar.

**Pause Time:** Young Generation GC, genellikle kısa pause time'lar yaratır. Old Generation GC, genellikle uzun pause time'lar yaratır.

**Object Promotion:** Object'ler, Young Generation'dan Old Generation'a promote edilir. Bu, object'in yaşam süresine göre belirlenir.

**Tuning:** Young Generation ve Old Generation'ın boyutları, GC tuning ile optimize edilebilir. Application'ın object allocation pattern'ına göre ayarlanır.

**Best Practices:** Young Generation ve Old Generation'ın boyutları, application profiling sonuçlarına göre optimize edilmelidir.

#### 4. Eden, Survivor ve Tenured space'lerin işlevi nedir?

**Cevap:**
Java heap memory, Young Generation ve Old Generation olmak üzere iki ana bölüme ayrılır. Young Generation, Eden, Survivor ve Tenured space'lerden oluşur. Her space'in kendine özgü işlevi vardır.

**Eden Space:** Eden Space, yeni oluşturulan object'lerin bulunduğu Young Generation'ın ilk bölümüdür. Object'ler, önce Eden Space'te oluşturulur. Eden Space dolduğunda, minor GC tetiklenir.

**Survivor Space:** Survivor Space, Young Generation'da GC cycle'ından sonra yaşayan object'lerin bulunduğu bölümdür. İki Survivor Space vardır: S0 ve S1. Object'ler, bu space'ler arasında taşınır.

**Tenured Space:** Tenured Space, Old Generation'ın bir parçasıdır. Uzun süre yaşayan object'ler, Survivor Space'ten Tenured Space'e promote edilir. Tenured Space'te major GC yapılır.

**Object Movement:** Object'ler, Eden Space'te oluşturulur. Eden Space dolduğunda, yaşayan object'ler Survivor Space'e taşınır. Belirli sayıda GC cycle'ından sonra Tenured Space'e promote edilir.

**Minor GC:** Minor GC, Young Generation'da yapılan GC'dir. Eden Space ve Survivor Space'lerde yapılır. Genellikle hızlıdır ve kısa pause time'lar yaratır.

**Major GC:** Major GC, Old Generation'da yapılan GC'dir. Tenured Space'te yapılır. Genellikle yavaştır ve uzun pause time'lar yaratır.

**Copying Algorithm:** Young Generation'da genellikle copying algorithm kullanılır. Object'ler, bir space'ten diğerine kopyalanır.

**Mark-Sweep-Compact:** Old Generation'da genellikle mark-sweep-compact algorithm kullanılır. Object'ler işaretlenir, temizlenir ve compact edilir.

**Tuning:** Eden, Survivor ve Tenured space'lerin boyutları, GC tuning ile optimize edilebilir. Application'ın object allocation pattern'ına göre ayarlanır.

**Best Practices:** Space boyutları, application profiling sonuçlarına göre optimize edilmelidir. Default değerler, çoğu durumda yeterlidir.

#### 5. System.gc() çağrısının etkisi nedir?

**Cevap:**
System.gc() method'u, JVM'e garbage collection yapması için suggestion verir. Ancak bu method'un çağrılması, GC'nin hemen çalışacağını garanti etmez ve genellikle önerilmez.

**Temel İşlev:** System.gc() method'u, JVM'e garbage collection yapması için suggestion verir. JVM, bu suggestion'ı dikkate alabilir veya göz ardı edebilir.

**Non-Guaranteed:** System.gc() method'u, GC'nin hemen çalışacağını garanti etmez. JVM, kendi algoritmasına göre GC'yi çalıştırır.

**Performance Impact:** System.gc() method'u, application performance'ını olumsuz etkileyebilir. GC, application'ın çalışmasını durdurabilir.

**Stop-the-World:** System.gc() method'u, stop-the-world pause'ları yaratabilir. Application, GC sırasında çalışamaz.

**Memory Pressure:** System.gc() method'u, memory pressure durumunda çağrılabilir. Ancak bu, genellikle geçici bir çözümdür.

**Debugging:** System.gc() method'u, debugging sırasında kullanılabilir. Memory leak'leri tespit etmek için yararlıdır.

**Best Practices:** System.gc() method'u, production code'da kullanılmamalıdır. JVM, kendi algoritmasına göre GC'yi yönetir.

**Alternative:** System.gc() method'u yerine, JVM tuning ile GC performance'ı optimize edilmelidir.

**JVM Flags:** -XX:+DisableExplicitGC flag'i ile System.gc() method'u devre dışı bırakılabilir.

**Monitoring:** System.gc() method'u, GC monitoring tool'ları ile izlenebilir. Performance impact'i ölçülebilir.

**Use Cases:** System.gc() method'u, sadece özel durumlarda kullanılmalıdır. Testing, debugging, memory analysis gibi durumlarda yararlıdır.

### Orta Seviye

#### 6. G1GC'nin özellikleri nelerdir?

**Cevap:**
G1GC (Garbage First Garbage Collector), Java 7'de tanıtılan modern bir GC algoritmasıdır. Large heap'ler için optimize edilmiş olup, predictable pause time'lar sağlar.

**Temel Özellikler:** G1GC, heap'i region'lara böler ve her region'ı ayrı ayrı yönetir. Bu, large heap'lerde daha iyi performance sağlar.

**Region-based:** G1GC, heap'i region'lara böler. Her region, 1MB ile 32MB arasında boyutlarda olabilir. Region'lar, Eden, Survivor, Old Generation olarak kategorize edilir.

**Predictable Pause Times:** G1GC, predictable pause time'lar sağlar. -XX:MaxGCPauseMillis parametresi ile maksimum pause time belirlenebilir.

**Concurrent Collection:** G1GC, concurrent collection sağlar. Application'ın çalışmasını durdurmadan garbage collection yapar.

**Incremental Collection:** G1GC, incremental collection yapar. Heap'in sadece bir kısmını temizler, tüm heap'i temizlemez.

**Mixed Collection:** G1GC, mixed collection yapar. Young Generation ve Old Generation'ı birlikte temizler.

**Remembered Sets:** G1GC, remembered set'ler kullanır. Region'lar arası reference'ları takip eder.

**SATB (Snapshot At The Beginning):** G1GC, SATB algorithm kullanır. Concurrent marking sırasında object graph'ın snapshot'ını alır.

**Humongous Objects:** G1GC, humongous object'leri özel olarak yönetir. Büyük object'ler, özel region'larda saklanır.

**Tuning Parameters:** G1GC, çok sayıda tuning parametresi sağlar. -XX:G1HeapRegionSize, -XX:MaxGCPauseMillis gibi.

**Use Cases:** G1GC, large heap'ler, low latency requirement'ları olan application'lar için uygundur.

**Best Practices:** G1GC, heap boyutu 6GB'dan büyük olan application'lar için önerilir.

#### 7. ZGC ve Shenandoah GC'nin avantajları nelerdir?

**Cevap:**
ZGC ve Shenandoah GC, Java'nın modern GC algoritmalarıdır. Ultra-low latency sağlayarak, very large heap'lerde bile sub-millisecond pause time'lar sunar.

**ZGC Özellikleri:** ZGC, Java 11'de tanıtılan ultra-low latency GC algoritmasıdır. Very large heap'ler için tasarlanmıştır.

**Sub-millisecond Pauses:** ZGC, sub-millisecond pause time'lar sağlar. Heap boyutu ne olursa olsun, pause time'lar 10ms'den azdır.

**Concurrent Collection:** ZGC, tamamen concurrent collection sağlar. Application'ın çalışmasını durdurmadan garbage collection yapar.

**Scalability:** ZGC, heap boyutundan bağımsız olarak çalışır. 8TB'ye kadar heap'ler desteklenir.

**Colored Pointers:** ZGC, colored pointer'lar kullanır. Pointer'ların metadata'sını pointer'ın kendisinde saklar.

**Load Barriers:** ZGC, load barrier'lar kullanır. Object access sırasında GC ile koordinasyon sağlar.

**Shenandoah GC Özellikleri:** Shenandoah GC, Java 12'de tanıtılan concurrent GC algoritmasıdır. Low latency ve high throughput sağlar.

**Concurrent Evacuation:** Shenandoah GC, concurrent evacuation sağlar. Object'ler, application çalışırken taşınır.

**Brooks Pointers:** Shenandoah GC, Brooks pointer'lar kullanır. Object'lerin forwarding pointer'larını saklar.

**Connection Matrix:** Shenandoah GC, connection matrix kullanır. Region'lar arası reference'ları takip eder.

**Use Cases:** ZGC ve Shenandoah GC, real-time application'lar, low latency requirement'ları olan system'ler için uygundur.

**Best Practices:** ZGC ve Shenandoah GC, heap boyutu 8GB'dan büyük olan application'lar için önerilir.

#### 8. Memory leak'lerin GC'ye etkisi nedir?

**Cevap:**
Memory leak'ler, GC'nin etkinliğini azaltır ve application performance'ını olumsuz etkiler. GC, memory leak'leri temizleyemez çünkü object'ler hala reference'lanıyordur.

**Temel Problem:** Memory leak'ler, object'lerin reference'ları kalmadığında bile memory'de kalmasıdır. GC, bu object'leri temizleyemez.

**GC Ineffectiveness:** Memory leak'ler, GC'nin etkinliğini azaltır. GC, sürekli çalışır ancak memory'yi temizleyemez.

**Heap Growth:** Memory leak'ler, heap'in sürekli büyümesine neden olur. GC, memory'yi temizleyemez ve heap büyümeye devam eder.

**OutOfMemoryError:** Memory leak'ler, OutOfMemoryError'a neden olabilir. Heap, maksimum boyuta ulaştığında application crash olur.

**GC Frequency:** Memory leak'ler, GC frequency'sini artırır. GC, sürekli çalışır ancak memory'yi temizleyemez.

**Performance Degradation:** Memory leak'ler, application performance'ını olumsuz etkiler. GC overhead'i artar.

**Common Causes:** Memory leak'lerin yaygın nedenleri: static reference'lar, listener'lar, cache'ler, thread'ler, native resource'lar.

**Detection:** Memory leak'ler, profiling tool'ları ile tespit edilebilir. Heap dump analysis yapılabilir.

**Prevention:** Memory leak'ler, good programming practice'ler ile önlenebilir. Reference'lar doğru yönetilmelidir.

**Best Practices:** Memory leak'leri önlemek için: weak reference'lar kullanılmalı, listener'lar unregister edilmeli, cache'ler temizlenmeli.

**Monitoring:** Memory leak'ler, monitoring tool'ları ile izlenebilir. Heap usage, GC frequency gibi metrik'ler takip edilmelidir.

#### 9. GC tuning parametreleri nelerdir?

**Cevap:**
GC tuning parametreleri, JVM'in garbage collection davranışını kontrol etmek için kullanılır. Bu parametreler, application'ın performance requirement'larına göre ayarlanır.

**Heap Size Parameters:** -Xms (initial heap size), -Xmx (maximum heap size) parametreleri heap boyutunu kontrol eder.

**Young Generation Parameters:** -XX:NewSize, -XX:MaxNewSize parametreleri Young Generation boyutunu kontrol eder.

**Survivor Space Parameters:** -XX:SurvivorRatio parametresi Survivor Space boyutunu kontrol eder.

**GC Algorithm Selection:** -XX:+UseSerialGC, -XX:+UseParallelGC, -XX:+UseG1GC gibi parametreler GC algoritmasını seçer.

**G1GC Parameters:** -XX:MaxGCPauseMillis, -XX:G1HeapRegionSize gibi parametreler G1GC'yi tune eder.

**Parallel GC Parameters:** -XX:ParallelGCThreads parametresi parallel GC thread sayısını kontrol eder.

**CMS Parameters:** -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction gibi parametreler CMS'yi tune eder.

**ZGC Parameters:** -XX:+UnlockExperimentalVMOptions -XX:+UseZGC gibi parametreler ZGC'yi aktif eder.

**Shenandoah Parameters:** -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC gibi parametreler Shenandoah GC'yi aktif eder.

**Logging Parameters:** -XX:+PrintGC, -XX:+PrintGCDetails gibi parametreler GC log'larını aktif eder.

**Best Practices:** GC tuning, application profiling sonuçlarına göre yapılmalıdır. Default parametreler, çoğu durumda yeterlidir.

**Monitoring:** GC tuning, monitoring tool'ları ile izlenmelidir. Performance impact'i ölçülmelidir.

#### 10. WeakReference, SoftReference ve PhantomReference arasındaki farklar nelerdir?

**Cevap:**
Java'da üç tür reference bulunur: WeakReference, SoftReference ve PhantomReference. Her biri, GC'nin object'leri temizleme davranışını farklı şekilde etkiler.

**Strong Reference:** Strong reference, normal object reference'ıdır. Object, strong reference'ı olduğu sürece GC tarafından temizlenmez.

**WeakReference:** WeakReference, weak reference sağlar. Object, sadece weak reference'ı varsa GC tarafından temizlenebilir.

**SoftReference:** SoftReference, soft reference sağlar. Object, memory pressure durumunda GC tarafından temizlenebilir.

**PhantomReference:** PhantomReference, phantom reference sağlar. Object, phantom reference'ı olduğu sürece GC tarafından temizlenmez.

**GC Behavior:** WeakReference, GC'nin object'i hemen temizlemesine izin verir. SoftReference, memory pressure durumunda temizlenir.

**Use Cases:** WeakReference, cache'ler, listener'lar için kullanılır. SoftReference, memory-sensitive cache'ler için kullanılır.

**ReferenceQueue:** WeakReference, SoftReference ve PhantomReference, ReferenceQueue ile birlikte kullanılabilir.

**Memory Management:** WeakReference ve SoftReference, memory management için kullanılır. PhantomReference, object finalization için kullanılır.

**Best Practices:** WeakReference, memory leak'leri önlemek için kullanılmalıdır. SoftReference, memory pressure durumunda temizlenebilir cache'ler için kullanılmalıdır.

**Performance:** WeakReference, SoftReference ve PhantomReference, GC performance'ını etkiler. Dikkatli kullanılmalıdır.

#### 11. Finalization'ın GC'ye etkisi nedir?

**Cevap:**
Finalization, Java'da object'lerin temizlenmesi sırasında çalışan mekanizmadır. Ancak finalization, GC performance'ını olumsuz etkiler ve genellikle önerilmez.

**Temel Konsept:** Finalization, object'in finalize() method'unun çağrılmasıdır. Object, GC tarafından temizlenmeden önce finalize() method'u çağrılır.

**GC Impact:** Finalization, GC performance'ını olumsuz etkiler. Object'ler, finalization queue'da bekler ve GC tarafından temizlenemez.

**Finalization Queue:** Object'ler, finalization queue'da bekler. Bu queue, GC tarafından yönetilir ve object'ler bu queue'da beklerken temizlenemez.

**Memory Leak Risk:** Finalization, memory leak risk'i yaratır. Object'ler, finalization queue'da beklerken memory'de kalır.

**Performance Degradation:** Finalization, application performance'ını olumsuz etkiler. GC overhead'i artar.

**Alternative:** Finalization yerine, try-with-resources, AutoCloseable interface'i kullanılmalıdır.

**Best Practices:** Finalization, sadece özel durumlarda kullanılmalıdır. Modern Java'da önerilmez.

**Deprecation:** Finalization, Java 9'da deprecated edilmiştir. Java 18'de kaldırılacaktır.

**Use Cases:** Finalization, native resource'ların temizlenmesi için kullanılabilir. Ancak modern alternatifler tercih edilmelidir.

**Monitoring:** Finalization, monitoring tool'ları ile izlenebilir. Performance impact'i ölçülebilir.

#### 12. GC log'ları nasıl analiz edilir?

**Cevap:**
GC log'ları, garbage collection davranışını analiz etmek için kullanılır. Bu log'lar, GC performance'ını optimize etmek için önemli bilgiler sağlar.

**GC Log Format:** GC log'ları, belirli bir format'ta yazılır. Timestamp, GC type, pause time, memory usage gibi bilgiler içerir.

**Log Parameters:** -XX:+PrintGC, -XX:+PrintGCDetails, -XX:+PrintGCTimeStamps gibi parametreler GC log'larını aktif eder.

**Log Analysis:** GC log'ları, manual olarak veya tool'lar ile analiz edilebilir. GCViewer, GCEasy gibi tool'lar kullanılabilir.

**Key Metrics:** GC log'larında önemli metrik'ler: pause time, throughput, memory usage, GC frequency.

**Pause Time Analysis:** Pause time'lar, application'ın responsiveness'ini etkiler. Yüksek pause time'lar, performance problem'lerine işaret eder.

**Throughput Analysis:** Throughput, application'ın iş yapma kapasitesini gösterir. Düşük throughput, GC overhead'inin yüksek olduğunu gösterir.

**Memory Usage Analysis:** Memory usage, heap'in nasıl kullanıldığını gösterir. Memory leak'ler, heap growth pattern'ları ile tespit edilebilir.

**GC Frequency Analysis:** GC frequency, GC'nin ne sıklıkla çalıştığını gösterir. Yüksek frequency, memory pressure'i gösterir.

**Best Practices:** GC log'ları, production environment'te aktif edilmelidir. Log rotation yapılmalıdır.

**Monitoring:** GC log'ları, monitoring system'ler ile entegre edilebilir. Alert'ler kurulabilir.

**Tuning:** GC log'ları, GC tuning için kullanılır. Performance problem'leri tespit edilir ve çözülür.

---

## Cron Jobs

### Temel Seviye

#### 1. Cron expression'ın syntax'ı nasıldır?

**Cevap:**
Cron expression, Unix/Linux sistemlerde zamanlanmış görevleri tanımlamak için kullanılan bir string format'ıdır. Spring Framework'te de aynı format kullanılır ve belirli zamanlarda method'ların çalıştırılmasını sağlar.

**Temel Format:** Cron expression, 6 veya 7 field'dan oluşur. Her field, zamanın farklı bir birimini temsil eder. Field'lar boşluk ile ayrılır.

**6 Field Format:** Saniye, Dakika, Saat, Gün, Ay, Haftanın Günü. Spring'de genellikle 6 field format kullanılır.

**7 Field Format:** Saniye, Dakika, Saat, Gün, Ay, Haftanın Günü, Yıl. 7 field format, yıl bilgisini de içerir.

**Field Açıklamaları:**
- Saniye (0-59): Dakikanın hangi saniyesinde çalışacağı
- Dakika (0-59): Saatin hangi dakikasında çalışacağı
- Saat (0-23): Günün hangi saatinde çalışacağı
- Gün (1-31): Ayın hangi gününde çalışacağı
- Ay (1-12): Yılın hangi ayında çalışacağı
- Haftanın Günü (0-7): Haftanın hangi gününde çalışacağı (0 ve 7 = Pazar)

**Özel Karakterler:**
- * (Yıldız): Her değer (her saniye, her dakika, vb.)
- ? (Soru işareti): Gün veya haftanın günü için kullanılır
- - (Tire): Aralık belirtir (1-5 = 1,2,3,4,5)
- , (Virgül): Liste belirtir (1,3,5 = 1, 3, 5)
- / (Slash): Adım belirtir (0/15 = 0,15,30,45)
- L (Last): Son değer (L = ayın son günü)
- W (Weekday): Hafta içi gün
- # (Hash): Haftanın belirli günü (#3 = ayın 3. Pazartesi)

**Örnekler:**
- "0 0 12 * * ?" = Her gün saat 12:00'de
- "0 15 10 ? * *" = Her gün saat 10:15'te
- "0 0 8-17 * * MON-FRI" = Hafta içi 8:00-17:00 arası her saat
- "0 0/30 8-17 * * MON-FRI" = Hafta içi 8:00-17:00 arası 30 dakikada bir

**Spring Kullanımı:** Spring'de @Scheduled annotation'ında cron parametresi olarak kullanılır.

**Timezone:** Cron expression, server'ın timezone'ına göre çalışır. Farklı timezone'lar için özel ayarlar gerekir.

**Validation:** Cron expression'lar, geçersiz format'ta olursa exception fırlatır.

**Best Practices:** Cron expression'lar, test edilmeli ve doğrulanmalıdır. Karmaşık expression'lar yerine basit olanlar tercih edilmelidir.

#### 2. @Scheduled annotation'ının parametreleri nelerdir?

**Cevap:**
@Scheduled annotation'ı, Spring Framework'te method'ları zamanlanmış görev olarak işaretlemek için kullanılır. Bu annotation'ın çeşitli parametreleri vardır ve farklı scheduling pattern'ları destekler.

**Temel Parametreler:** @Scheduled annotation'ının temel parametreleri: fixedRate, fixedDelay, cron, initialDelay, zone.

**fixedRate:** fixedRate parametresi, method'un belirli aralıklarla çalışmasını sağlar. Önceki execution'ın bitmesini beklemez. Milisaniye cinsinden belirtilir.

**fixedDelay:** fixedDelay parametresi, method'un önceki execution'ı bitirdikten belirli süre sonra çalışmasını sağlar. Önceki execution'ın bitmesini bekler. Milisaniye cinsinden belirtilir.

**cron:** cron parametresi, cron expression kullanarak method'un ne zaman çalışacağını belirtir. Unix cron format'ını kullanır.

**initialDelay:** initialDelay parametresi, method'un ilk kez ne kadar gecikme ile çalışacağını belirtir. Milisaniye cinsinden belirtilir.

**zone:** zone parametresi, method'un hangi timezone'da çalışacağını belirtir. Timezone ID'si kullanılır.

**String vs Long:** fixedRate, fixedDelay, initialDelay parametreleri hem string hem de long değer alabilir. String değerler, SpEL expression'ları destekler.

**SpEL Support:** String parametreler, Spring Expression Language (SpEL) destekler. Property'lerden değer alınabilir.

**Property Placeholder:** @Scheduled annotation'ında property placeholder'lar kullanılabilir. ${property.name} formatında.

**Multiple Scheduling:** Bir method'da birden fazla @Scheduled annotation'ı kullanılamaz. Farklı scheduling pattern'ları için farklı method'lar gerekir.

**Method Requirements:** @Scheduled annotation'ı ile işaretlenmiş method'lar void döndürmeli ve parametre almamalıdır.

**Best Practices:** @Scheduled annotation'ı, basit ve anlaşılır scheduling pattern'ları için kullanılmalıdır. Karmaşık logic'ler için TaskScheduler kullanılmalıdır.

**Performance:** @Scheduled annotation'ı, Spring'in TaskScheduler'ı ile çalışır. Thread pool yönetimi otomatiktir.

#### 3. Fixed rate ve fixed delay arasındaki fark nedir?

**Cevap:**
Fixed rate ve fixed delay, @Scheduled annotation'ının iki farklı scheduling parametresidir. Her ikisi de belirli aralıklarla method'ların çalışmasını sağlar ancak farklı davranış sergiler.

**Fixed Rate:** fixedRate parametresi, method'un belirli aralıklarla çalışmasını sağlar. Önceki execution'ın bitmesini beklemez. Her belirtilen sürede method çalışır.

**Fixed Delay:** fixedDelay parametresi, method'un önceki execution'ı bitirdikten belirli süre sonra çalışmasını sağlar. Önceki execution'ın bitmesini bekler.

**Timing Difference:** Fixed rate, absolute timing kullanır. Fixed delay, relative timing kullanır.

**Execution Overlap:** Fixed rate'de, önceki execution bitmeden yeni execution başlayabilir. Fixed delay'de, önceki execution bitmeden yeni execution başlamaz.

**Use Cases:** Fixed rate, regular interval'lar için uygundur. Fixed delay, execution'lar arasında minimum süre olması gereken durumlar için uygundur.

**Performance Impact:** Fixed rate, execution overlap nedeniyle resource kullanımını artırabilir. Fixed delay, resource kullanımını kontrol eder.

**Thread Pool:** Fixed rate, thread pool'u daha fazla kullanabilir. Fixed delay, thread pool'u daha az kullanır.

**Error Handling:** Fixed rate'de, bir execution hata verirse diğer execution'lar etkilenmez. Fixed delay'de, bir execution hata verirse delay süresi başlar.

**Monitoring:** Fixed rate, execution frequency'sini garanti eder. Fixed delay, execution'lar arası süreyi garanti eder.

**Best Practices:** Fixed rate, time-sensitive task'lar için kullanılmalıdır. Fixed delay, resource-intensive task'lar için kullanılmalıdır.

**Example:** Fixed rate 1000ms = her 1 saniyede çalışır. Fixed delay 1000ms = önceki execution bitince 1 saniye sonra çalışır.

#### 4. Cron job'ların thread pool'u nasıl yönetilir?

**Cevap:**
Spring Framework'te cron job'lar, TaskScheduler interface'i ile yönetilir. TaskScheduler, thread pool'u otomatik olarak yönetir ve scheduled task'ları execute eder.

**TaskScheduler Interface:** TaskScheduler, scheduled task'ları yönetmek için kullanılan interface'dir. Thread pool yönetimi bu interface'in sorumluluğundadır.

**Default Implementation:** Spring, TaskScheduler için default implementation sağlar. ThreadPoolTaskScheduler, en yaygın kullanılan implementation'dır.

**Thread Pool Configuration:** ThreadPoolTaskScheduler, thread pool'u yapılandırmak için çeşitli parametreler sağlar.

**Core Pool Size:** Core pool size, thread pool'da her zaman aktif olan thread sayısıdır. Default değer genellikle 1'dir.

**Max Pool Size:** Max pool size, thread pool'da maksimum thread sayısıdır. Default değer genellikle Integer.MAX_VALUE'dır.

**Queue Capacity:** Queue capacity, task'ların bekleyeceği queue'nun kapasitesidir. Default değer genellikle Integer.MAX_VALUE'dır.

**Keep Alive Time:** Keep alive time, idle thread'lerin ne kadar süre bekleyeceğidir. Default değer genellikle 60 saniyedir.

**Thread Name Prefix:** Thread name prefix, thread'lerin isimlerinin ön ekidir. Debugging için yararlıdır.

**Custom Configuration:** TaskScheduler, @Configuration class'larında custom olarak yapılandırılabilir.

**Bean Definition:** TaskScheduler, Spring bean olarak tanımlanır. @Bean annotation'ı ile yapılandırılır.

**Best Practices:** TaskScheduler, application'ın requirement'larına göre yapılandırılmalıdır. Default değerler, çoğu durumda yeterlidir.

**Monitoring:** TaskScheduler, thread pool metrik'leri ile izlenebilir. Active thread count, queue size gibi metrik'ler takip edilmelidir.

#### 5. @EnableScheduling annotation'ının işlevi nedir?

**Cevap:**
@EnableScheduling annotation'ı, Spring Framework'te scheduled task'ları aktif etmek için kullanılır. Bu annotation, Spring'in scheduling infrastructure'ını aktif eder.

**Temel İşlev:** @EnableScheduling annotation'ı, Spring'in scheduling support'ını aktif eder. Bu annotation olmadan @Scheduled annotation'ları çalışmaz.

**Configuration Class:** @EnableScheduling annotation'ı, @Configuration class'larında kullanılır. Genellikle main configuration class'ında kullanılır.

**Infrastructure Activation:** @EnableScheduling annotation'ı, Spring'in scheduling infrastructure'ını aktif eder. TaskScheduler, TaskExecutor gibi component'ler oluşturulur.

**Auto Configuration:** @EnableScheduling annotation'ı, Spring Boot'ta otomatik olarak aktif edilir. Spring Boot, scheduling'i otomatik olarak yapılandırır.

**TaskScheduler Bean:** @EnableScheduling annotation'ı, TaskScheduler bean'ini otomatik olarak oluşturur. Bu bean, scheduled task'ları execute eder.

**Thread Pool:** @EnableScheduling annotation'ı, default thread pool'u oluşturur. Bu thread pool, scheduled task'ları execute eder.

**Scanning:** @EnableScheduling annotation'ı, @Scheduled annotation'larını scan eder. Bu annotation'lar ile işaretlenmiş method'ları bulur.

**Registration:** @EnableScheduling annotation'ı, bulunan scheduled method'ları TaskScheduler'a register eder.

**Lifecycle Management:** @EnableScheduling annotation'ı, scheduled task'ların lifecycle'ını yönetir. Start, stop, pause gibi işlemler yapılabilir.

**Best Practices:** @EnableScheduling annotation'ı, sadece scheduled task'lar kullanıldığında aktif edilmelidir. Gereksiz yere aktif edilmemelidir.

**Performance:** @EnableScheduling annotation'ı, minimal overhead yaratır. Sadece gerekli infrastructure'ı aktif eder.

### Orta Seviye

#### 6. Distributed environment'da cron job'lar nasıl yönetilir?

**Cevap:**
Distributed environment'da cron job'lar, birden fazla instance'da çalışan application'larda önemli bir challenge'dır. Aynı job'ın birden fazla instance'da çalışmasını önlemek ve job'ların güvenilir şekilde execute edilmesini sağlamak gerekir.

**Temel Problem:** Distributed environment'da, aynı cron job birden fazla instance'da çalışabilir. Bu, duplicate execution'lara ve data inconsistency'ye neden olabilir.

**Leader Election:** Leader election, distributed environment'da cron job'ları yönetmek için kullanılan yaygın bir yaklaşımdır. Sadece leader instance, cron job'ları execute eder.

**Database Locking:** Database locking, cron job'ların sadece bir instance'da çalışmasını sağlamak için kullanılır. Database'de lock record'u oluşturulur.

**Redis Locking:** Redis locking, distributed lock'lar için kullanılır. Redis'in atomic operation'ları ile lock'lar yönetilir.

**Zookeeper:** Zookeeper, distributed coordination için kullanılır. Leader election ve distributed lock'lar sağlar.

**ShedLock:** ShedLock, Spring'de distributed locking için kullanılan library'dir. Database, Redis, Zookeeper gibi backend'ler destekler.

**Quartz Scheduler:** Quartz Scheduler, distributed scheduling için kullanılır. Clustering support sağlar.

**Spring Cloud:** Spring Cloud, distributed system'ler için kullanılır. Service discovery, configuration management sağlar.

**Message Queue:** Message queue'lar, cron job'ları distribute etmek için kullanılır. Job'lar queue'ya gönderilir ve consumer'lar tarafından işlenir.

**Best Practices:** Distributed environment'da cron job'lar, idempotent olmalıdır. Duplicate execution'lar zarar vermemelidir.

**Monitoring:** Distributed cron job'lar, monitoring system'ler ile izlenmelidir. Execution status, failure rate gibi metrik'ler takip edilmelidir.

#### 7. Cron job'ların error handling'i nasıl yapılır?

**Cevap:**
Cron job'ların error handling'i, application'ın stability'sini ve reliability'sini sağlamak için kritik öneme sahiptir. Hata durumlarında job'ların nasıl davranacağı ve hataların nasıl handle edileceği önemlidir.

**Temel Yaklaşım:** Cron job'ların error handling'i, try-catch block'ları ile yapılır. Exception'lar yakalanır ve uygun şekilde handle edilir.

**Exception Handling:** Cron job'lar, exception'ları yakalamalı ve handle etmelidir. Unhandled exception'lar, job'ın crash olmasına neden olabilir.

**Retry Mechanism:** Retry mechanism, geçici hatalar için kullanılır. Job'lar, belirli sayıda retry yapabilir.

**Exponential Backoff:** Exponential backoff, retry'lar arasında giderek artan süre bekler. System overload'ını önler.

**Circuit Breaker:** Circuit breaker, sürekli hata veren job'ları geçici olarak devre dışı bırakır. System stability'sini sağlar.

**Dead Letter Queue:** Dead letter queue, başarısız job'ları saklar. Manual intervention için kullanılır.

**Logging:** Cron job'lar, detaylı logging yapmalıdır. Error'lar, warning'ler, info mesajları log'lanmalıdır.

**Monitoring:** Cron job'lar, monitoring system'ler ile izlenmelidir. Error rate, success rate gibi metrik'ler takip edilmelidir.

**Alerting:** Cron job'lar, critical error'larda alert göndermelidir. Email, SMS, Slack gibi kanallar kullanılabilir.

**Best Practices:** Cron job'lar, idempotent olmalıdır. Retry'lar zarar vermemelidir.

**Health Check:** Cron job'lar, health check endpoint'leri sağlamalıdır. System status'u kontrol edilebilir.

#### 8. Dynamic scheduling nasıl implement edilir?

**Cevap:**
Dynamic scheduling, runtime'da cron job'ların schedule'ının değiştirilmesini sağlar. Bu, application'ın requirement'larına göre job'ların schedule'ının dinamik olarak ayarlanmasını mümkün kılar.

**Temel Konsept:** Dynamic scheduling, cron job'ların schedule'ının runtime'da değiştirilmesini sağlar. Static @Scheduled annotation'ları yerine dynamic scheduling kullanılır.

**TaskScheduler Interface:** TaskScheduler interface'i, dynamic scheduling için kullanılır. schedule() method'u ile job'lar schedule edilir.

**ScheduledFuture:** ScheduledFuture, scheduled task'ları temsil eder. Task'ları cancel etmek, schedule'ını değiştirmek için kullanılır.

**CronTrigger:** CronTrigger, cron expression'ları ile dynamic scheduling sağlar. Cron expression'lar runtime'da değiştirilebilir.

**FixedRateTrigger:** FixedRateTrigger, fixed rate scheduling sağlar. Rate değeri runtime'da değiştirilebilir.

**FixedDelayTrigger:** FixedDelayTrigger, fixed delay scheduling sağlar. Delay değeri runtime'da değiştirilebilir.

**Task Registration:** Dynamic scheduling'de, task'lar runtime'da register edilir. @Scheduled annotation'ları kullanılmaz.

**Task Cancellation:** Dynamic scheduling'de, task'lar runtime'da cancel edilebilir. ScheduledFuture.cancel() method'u kullanılır.

**Configuration Management:** Dynamic scheduling, configuration management ile entegre edilebilir. Schedule'lar configuration'dan okunabilir.

**Database Storage:** Dynamic scheduling, schedule'ları database'de saklayabilir. Runtime'da database'den okunabilir.

**Best Practices:** Dynamic scheduling, thread-safe olmalıdır. Concurrent access'ler handle edilmelidir.

**Monitoring:** Dynamic scheduling, monitoring system'ler ile izlenmelidir. Schedule değişiklikleri log'lanmalıdır.

#### 9. Cron job'ların monitoring'i nasıl yapılır?

**Cevap:**
Cron job'ların monitoring'i, application'ın health'ini ve performance'ını takip etmek için kritik öneme sahiptir. Job'ların execution status'u, performance metrik'leri ve error rate'leri izlenmelidir.

**Temel Metrik'ler:** Cron job'ların monitoring'inde önemli metrik'ler: execution count, success rate, failure rate, execution time, last execution time.

**Execution Status:** Cron job'ların execution status'u izlenmelidir. Success, failure, timeout gibi status'lar takip edilmelidir.

**Performance Metrics:** Cron job'ların performance metrik'leri izlenmelidir. Execution time, memory usage, CPU usage gibi metrik'ler takip edilmelidir.

**Error Tracking:** Cron job'ların error'ları izlenmelidir. Exception'lar, error message'ları, stack trace'ler log'lanmalıdır.

**Health Check:** Cron job'lar, health check endpoint'leri sağlamalıdır. System status'u kontrol edilebilir.

**Logging:** Cron job'lar, detaylı logging yapmalıdır. Execution start, end, error'lar log'lanmalıdır.

**Metrics Collection:** Cron job'lar, metrics collection system'ler ile entegre edilmelidir. Prometheus, Micrometer gibi tool'lar kullanılabilir.

**Alerting:** Cron job'lar, critical error'larda alert göndermelidir. Email, SMS, Slack gibi kanallar kullanılabilir.

**Dashboard:** Cron job'lar, monitoring dashboard'larında görüntülenmelidir. Grafana, Kibana gibi tool'lar kullanılabilir.

**Best Practices:** Cron job'lar, monitoring için standardize edilmiş format kullanmalıdır. Consistent logging, metrics sağlanmalıdır.

**Automation:** Cron job'lar, monitoring automation'ı sağlamalıdır. Auto-recovery, auto-scaling gibi özellikler eklenebilir.

#### 10. Timezone handling'i cron job'larda nasıl çalışır?

**Cevap:**
Timezone handling, cron job'larda önemli bir konudur. Job'ların farklı timezone'larda çalışması ve doğru zamanlarda execute edilmesi gerekir.

**Temel Konsept:** Cron job'lar, server'ın timezone'ına göre çalışır. Farklı timezone'lar için özel ayarlar gerekir.

**Server Timezone:** Cron job'lar, server'ın timezone'ına göre çalışır. Server'ın timezone'ı, job'ların execution time'ını etkiler.

**@Scheduled Zone:** @Scheduled annotation'ında zone parametresi ile timezone belirtilebilir. Timezone ID'si kullanılır.

**Cron Expression:** Cron expression'lar, timezone'a göre yorumlanır. Aynı expression, farklı timezone'larda farklı zamanlarda çalışır.

**UTC Standard:** UTC, timezone handling için standard olarak kullanılır. Tüm timezone'lar UTC'ye göre hesaplanır.

**Daylight Saving:** Daylight saving, timezone handling'i karmaşık hale getirir. Timezone değişiklikleri job'ları etkileyebilir.

**Java Time API:** Java 8'deki Time API, timezone handling için kullanılır. ZonedDateTime, LocalDateTime gibi class'lar kullanılabilir.

**Configuration:** Timezone, configuration'dan okunabilir. application.properties dosyasında belirtilebilir.

**Best Practices:** Cron job'lar, UTC timezone kullanmalıdır. Timezone conversion'ları doğru yapılmalıdır.

**Testing:** Timezone handling, test edilmelidir. Farklı timezone'larda test yapılmalıdır.

**Monitoring:** Timezone handling, monitoring system'ler ile izlenmelidir. Timezone değişiklikleri log'lanmalıdır.

---

## HTTP Stateful vs Stateless

### Temel Seviye

#### 1. Stateful ve Stateless arasındaki fark nedir?

**Cevap:**
Stateful ve Stateless, web application'ların client-server communication'ında kullandığı iki farklı yaklaşımdır. Her ikisi de farklı avantaj ve dezavantajlara sahiptir.

**Stateful Communication:** Stateful communication'da, server client'ın state'ini (durumunu) saklar. Her request, önceki request'lerin context'ini içerir. Server, client'ın session bilgilerini memory'de tutar.

**Stateless Communication:** Stateless communication'da, server client'ın state'ini saklamaz. Her request, kendi başına bağımsızdır. Server, client'ın önceki request'lerini hatırlamaz.

**State Management:** Stateful'da state server'da saklanır. Stateless'da state client'ta veya external storage'da saklanır.

**Session Handling:** Stateful'da session server'da yönetilir. Stateless'da session client'ta veya token'larda yönetilir.

**Scalability:** Stateful servisler, scalability açısından sınırlıdır. Stateless servisler, daha kolay scale edilebilir.

**Load Balancing:** Stateful servisler, sticky session gerektirir. Stateless servisler, herhangi bir server'a yönlendirilebilir.

**Memory Usage:** Stateful servisler, daha fazla memory kullanır. Stateless servisler, daha az memory kullanır.

**Fault Tolerance:** Stateful servisler, server failure durumunda state kaybeder. Stateless servisler, server failure'dan etkilenmez.

**Use Cases:** Stateful, real-time application'lar için uygundur. Stateless, RESTful API'ler için uygundur.

**Best Practices:** Modern web application'lar, stateless yaklaşımı tercih eder. Microservices architecture'da stateless servisler kullanılır.

#### 2. HTTP'nin stateless olmasının anlamı nedir?

**Cevap:**
HTTP'nin stateless olması, HTTP protocol'ünün temel karakteristiğidir. Bu, her HTTP request'inin kendi başına bağımsız olması ve server'ın önceki request'leri hatırlamaması anlamına gelir.

**Temel Konsept:** HTTP stateless'tır çünkü her request, kendi başına bağımsızdır. Server, client'ın önceki request'lerini hatırlamaz.

**Request Independence:** Her HTTP request, kendi başına bağımsızdır. Request'ler arasında bağımlılık yoktur.

**Server Memory:** HTTP server'lar, client state'ini saklamaz. Her request, gerekli tüm bilgileri içerir.

**Protocol Design:** HTTP'nin stateless olması, protocol'ün basitliğini sağlar. Server'lar, complex state management yapmak zorunda değildir.

**Scalability:** HTTP'nin stateless olması, scalability'yi artırır. Server'lar, load balancer'lar arasında kolayca dağıtılabilir.

**Caching:** HTTP'nin stateless olması, caching'i kolaylaştırır. Response'lar, client'tan bağımsız olarak cache'lenebilir.

**RESTful Design:** HTTP'nin stateless olması, RESTful API design'ını destekler. RESTful servisler, stateless olmalıdır.

**Session Management:** HTTP'nin stateless olması, session management'i karmaşık hale getirir. Cookie'ler, session ID'ler kullanılır.

**State Simulation:** HTTP'nin stateless olmasına rağmen, stateful behavior simulate edilebilir. Cookie'ler, session'lar kullanılır.

**Best Practices:** HTTP'nin stateless nature'ı, modern web application design'ında dikkate alınmalıdır.

#### 3. Session management nasıl yapılır?

**Cevap:**
Session management, HTTP'nin stateless nature'ına rağmen stateful behavior sağlamak için kullanılan mekanizmadır. Client'ın server ile olan interaction'ının state'ini yönetir.

**Temel Konsept:** Session management, client'ın server ile olan interaction'ının state'ini yönetir. Session, client'ın server'da saklanan state'idir.

**Session Creation:** Session, client'ın ilk request'inde oluşturulur. Server, unique session ID oluşturur ve client'a gönderir.

**Session ID:** Session ID, unique identifier'dır. Client, bu ID'yi her request'te gönderir. Server, bu ID ile session'ı bulur.

**Cookie Storage:** Session ID, genellikle cookie'de saklanır. Browser, cookie'yi otomatik olarak gönderir.

**Server Storage:** Session data, server'da saklanır. Memory, database veya external storage kullanılır.

**Session Lifecycle:** Session, oluşturulur, kullanılır ve sonlandırılır. Timeout, explicit logout ile sonlandırılabilir.

**Session Timeout:** Session, belirli süre sonra otomatik olarak sonlandırılır. Security için önemlidir.

**Session Security:** Session, güvenli şekilde yönetilmelidir. Session ID, tahmin edilemez olmalıdır.

**Session Clustering:** Session, multiple server'lar arasında paylaşılabilir. Database, Redis gibi external storage kullanılır.

**Best Practices:** Session management, security, performance, scalability açısından dikkatli yapılmalıdır.

**Alternatives:** Session management yerine, JWT token'lar, OAuth gibi stateless authentication kullanılabilir.

#### 4. Cookie'lerin stateful communication'daki rolü nedir?

**Cevap:**
Cookie'ler, HTTP'nin stateless nature'ına rağmen stateful communication sağlamak için kullanılan temel mekanizmadır. Client'ın server ile olan interaction'ının state'ini saklar.

**Temel Rol:** Cookie'ler, client'ın server ile olan interaction'ının state'ini saklar. Session ID, user preference'ları gibi bilgileri tutar.

**Session Management:** Cookie'ler, session management için kullanılır. Session ID, cookie'de saklanır ve her request'te gönderilir.

**Automatic Handling:** Browser'lar, cookie'leri otomatik olarak yönetir. Server'dan gelen cookie'ler otomatik olarak saklanır.

**Request Inclusion:** Browser'lar, cookie'leri her request'te otomatik olarak gönderir. Server, cookie'leri okuyarak client'ı tanır.

**Cookie Attributes:** Cookie'ler, çeşitli attribute'lara sahiptir. Domain, path, expires, secure, httpOnly gibi.

**Security:** Cookie'ler, güvenli şekilde yönetilmelidir. Secure, httpOnly attribute'ları kullanılmalıdır.

**Privacy:** Cookie'ler, privacy concern'leri yaratır. GDPR, CCPA gibi regulation'lar cookie kullanımını kısıtlar.

**Storage Limit:** Cookie'ler, storage limit'ine sahiptir. 4KB'a kadar data saklanabilir.

**Cross-Domain:** Cookie'ler, cross-domain request'lerde kısıtlamalara sahiptir. Same-origin policy uygulanır.

**Best Practices:** Cookie'ler, minimal data saklamalıdır. Sensitive data, cookie'de saklanmamalıdır.

**Alternatives:** Cookie'ler yerine, localStorage, sessionStorage, JWT token'lar kullanılabilir.

#### 5. RESTful servislerin stateless olmasının faydaları nelerdir?

**Cevap:**
RESTful servislerin stateless olması, modern web application'ların temel prensiplerinden biridir. Bu yaklaşım, çeşitli faydalar sağlar ve application'ın scalability'sini, maintainability'sini artırır.

**Temel Faydalar:** RESTful servislerin stateless olması, scalability, maintainability, reliability açısından faydalar sağlar.

**Scalability:** Stateless servisler, daha kolay scale edilebilir. Herhangi bir server'a yönlendirilebilir.

**Load Balancing:** Stateless servisler, load balancer'lar arasında kolayca dağıtılabilir. Sticky session gerektirmez.

**Fault Tolerance:** Stateless servisler, server failure'dan etkilenmez. State kaybı olmaz.

**Caching:** Stateless servisler, daha iyi cache'lenebilir. Response'lar, client'tan bağımsız olarak cache'lenebilir.

**Testing:** Stateless servisler, daha kolay test edilebilir. Her request, kendi başına test edilebilir.

**Debugging:** Stateless servisler, daha kolay debug edilebilir. Request'ler arasında bağımlılık yoktur.

**Performance:** Stateless servisler, daha iyi performance sağlar. Server memory kullanımı azalır.

**Maintainability:** Stateless servisler, daha kolay maintain edilebilir. State management complexity'si yoktur.

**Microservices:** Stateless servisler, microservices architecture'da ideal'dir. Service'ler arasında loose coupling sağlar.

**Best Practices:** RESTful servisler, stateless olmalıdır. State, client'ta veya external storage'da saklanmalıdır.

**Implementation:** Stateless servisler, JWT token'lar, OAuth gibi stateless authentication kullanmalıdır.

### Orta Seviye

#### 6. JWT token'ların stateless authentication'daki rolü nedir?

**Cevap:**
JWT (JSON Web Token) token'lar, stateless authentication'ın temel taşlarından biridir. Server'ın client state'ini saklamasına gerek kalmadan güvenli authentication sağlar.

**Temel Rol:** JWT token'lar, stateless authentication sağlar. Server, client state'ini saklamak zorunda değildir.

**Token Structure:** JWT token'lar, üç bölümden oluşur: header, payload, signature. Her bölüm, farklı bilgiler içerir.

**Self-Contained:** JWT token'lar, self-contained'dır. Token'ın kendisi, authentication bilgilerini içerir.

**Stateless Nature:** JWT token'lar, stateless'tır. Server, token'ı validate eder ancak state saklamaz.

**Digital Signature:** JWT token'lar, digital signature ile güvenli hale getirilir. Token'ın değiştirilmediği garanti edilir.

**Expiration:** JWT token'lar, expiration time'a sahiptir. Token, belirli süre sonra geçersiz olur.

**Refresh Token:** JWT token'lar, refresh token ile yenilenebilir. Access token'ın süresi uzatılabilir.

**Claims:** JWT token'lar, claims içerir. User ID, role, permission gibi bilgiler token'da saklanır.

**Cross-Domain:** JWT token'lar, cross-domain request'lerde kullanılabilir. CORS policy'leri ile yönetilir.

**Security:** JWT token'lar, güvenli şekilde yönetilmelidir. HTTPS, secure storage kullanılmalıdır.

**Best Practices:** JWT token'lar, minimal claim içermelidir. Sensitive data, token'da saklanmamalıdır.

**Alternatives:** JWT token'lar yerine, OAuth, SAML gibi authentication protocol'ları kullanılabilir.

#### 7. Session clustering nasıl yapılır?

**Cevap:**
Session clustering, multiple server'lar arasında session'ların paylaşılmasını sağlar. Bu, load balancing ve high availability için kritik öneme sahiptir.

**Temel Konsept:** Session clustering, multiple server'lar arasında session'ların paylaşılmasını sağlar. Client, herhangi bir server'a yönlendirilebilir.

**Load Balancing:** Session clustering, load balancing ile entegre çalışır. Client'lar, herhangi bir server'a yönlendirilebilir.

**Session Replication:** Session replication, session'ların multiple server'lar arasında replicate edilmesini sağlar. Her server, tüm session'ları saklar.

**Session Persistence:** Session persistence, session'ların external storage'da saklanmasını sağlar. Database, Redis gibi storage'lar kullanılır.

**Sticky Session:** Sticky session, client'ın her zaman aynı server'a yönlendirilmesini sağlar. Session clustering gerektirmez.

**Database Storage:** Session'lar, database'de saklanabilir. MySQL, PostgreSQL gibi database'ler kullanılır.

**Redis Storage:** Session'lar, Redis'te saklanabilir. Redis, high-performance session storage sağlar.

**Memcached Storage:** Session'lar, Memcached'te saklanabilir. Memcached, distributed caching sağlar.

**Hazelcast:** Hazelcast, distributed session management sağlar. In-memory data grid olarak çalışır.

**Spring Session:** Spring Session, session management'i abstract eder. Multiple backend'ler destekler.

**Best Practices:** Session clustering, performance, consistency, security açısından dikkatli yapılmalıdır.

**Monitoring:** Session clustering, monitoring system'ler ile izlenmelidir. Session count, hit rate gibi metrik'ler takip edilmelidir.

#### 8. Stateless servislerin scalability avantajları nelerdir?

**Cevap:**
Stateless servislerin scalability avantajları, modern web application'ların temel faydalarından biridir. Bu servisler, horizontal scaling'i kolaylaştırır ve performance'ı artırır.

**Temel Avantajlar:** Stateless servislerin scalability avantajları, horizontal scaling, load balancing, fault tolerance açısından faydalar sağlar.

**Horizontal Scaling:** Stateless servisler, horizontal scaling'i kolaylaştırır. Yeni server'lar eklenebilir.

**Load Balancing:** Stateless servisler, load balancing'i kolaylaştırır. Herhangi bir server'a yönlendirilebilir.

**Fault Tolerance:** Stateless servisler, fault tolerance sağlar. Server failure'dan etkilenmez.

**Auto Scaling:** Stateless servisler, auto scaling'i kolaylaştırır. Cloud platform'ları ile entegre çalışır.

**Resource Utilization:** Stateless servisler, resource utilization'ı optimize eder. Server'lar, daha verimli kullanılır.

**Cost Efficiency:** Stateless servisler, cost efficiency sağlar. Daha az server ile daha fazla load handle edilir.

**Performance:** Stateless servisler, daha iyi performance sağlar. Memory usage azalır.

**Maintenance:** Stateless servisler, maintenance'i kolaylaştırır. Server'lar, independent olarak update edilebilir.

**Deployment:** Stateless servisler, deployment'ı kolaylaştırır. Blue-green deployment, rolling update yapılabilir.

**Best Practices:** Stateless servisler, scalability için design edilmelidir. State, external storage'da saklanmalıdır.

**Monitoring:** Stateless servisler, monitoring system'ler ile izlenmelidir. Performance metrik'leri takip edilmelidir.

#### 9. Stateful servislerin kullanım alanları nelerdir?

**Cevap:**
Stateful servisler, belirli use case'lerde gerekli olabilir. Real-time application'lar, gaming, financial system'ler gibi durumlarda stateful behavior gerekebilir.

**Temel Kullanım Alanları:** Stateful servisler, real-time application'lar, gaming, financial system'ler gibi durumlarda kullanılır.

**Real-Time Applications:** Real-time application'lar, stateful servisler gerektirir. WebSocket, Server-Sent Events gibi teknolojiler kullanılır.

**Gaming:** Gaming application'lar, stateful servisler gerektirir. Game state, player position gibi bilgiler saklanır.

**Financial Systems:** Financial system'ler, stateful servisler gerektirir. Transaction state, account balance gibi bilgiler saklanır.

**E-Commerce:** E-commerce application'lar, stateful servisler gerektirir. Shopping cart, checkout process gibi bilgiler saklanır.

**Collaborative Tools:** Collaborative tool'lar, stateful servisler gerektirir. Document editing, real-time collaboration gibi bilgiler saklanır.

**IoT Applications:** IoT application'lar, stateful servisler gerektirir. Device state, sensor data gibi bilgiler saklanır.

**Chat Applications:** Chat application'lar, stateful servisler gerektirir. Message history, online status gibi bilgiler saklanır.

**Video Streaming:** Video streaming application'lar, stateful servisler gerektirir. Playback position, user preference'lar gibi bilgiler saklanır.

**Best Practices:** Stateful servisler, sadece gerekli olduğunda kullanılmalıdır. State, minimal tutulmalıdır.

**Alternatives:** Stateful servisler yerine, stateless servisler + external storage kullanılabilir.

**Monitoring:** Stateful servisler, monitoring system'ler ile izlenmelidir. State size, memory usage gibi metrik'ler takip edilmelidir.

#### 10. Microservices architecture'da state management nasıl yapılır?

**Cevap:**
Microservices architecture'da state management, distributed system'lerin temel challenge'larından biridir. Service'ler arasında state'in nasıl yönetileceği ve paylaşılacağı önemlidir.

**Temel Konsept:** Microservices architecture'da state management, service'ler arasında state'in nasıl yönetileceği ve paylaşılacağıdır.

**Stateless Services:** Microservices, stateless olmalıdır. State, external storage'da saklanmalıdır.

**Database per Service:** Her service, kendi database'ine sahip olmalıdır. Service'ler arasında database paylaşılmamalıdır.

**Event Sourcing:** Event sourcing, state'in event'ler olarak saklanmasını sağlar. State, event'lerden reconstruct edilir.

**CQRS:** CQRS (Command Query Responsibility Segregation), read ve write operation'ları ayırır. State, farklı şekillerde yönetilir.

**Saga Pattern:** Saga pattern, distributed transaction'ları yönetir. State, multiple service'ler arasında coordinate edilir.

**API Gateway:** API Gateway, client'ların state'ini yönetir. Authentication, authorization gibi bilgiler saklanır.

**Service Mesh:** Service mesh, service'ler arası communication'ı yönetir. State, service mesh'te saklanabilir.

**Message Queue:** Message queue'lar, service'ler arası state synchronization sağlar. Event'ler, queue'lar üzerinden gönderilir.

**Best Practices:** Microservices architecture'da state management, service boundary'leri dikkate alınmalıdır.

**Monitoring:** Microservices architecture'da state management, monitoring system'ler ile izlenmelidir. State consistency, data integrity gibi metrik'ler takip edilmelidir.

**Testing:** Microservices architecture'da state management, test edilmelidir. Integration test'ler, end-to-end test'ler yapılmalıdır.

---

## HTTP Request Methods

### Temel Seviye

#### 1. GET, POST, PUT, DELETE method'larının kullanım alanları nelerdir?

**Cevap:**
HTTP request method'ları, client'ın server'a ne tür bir işlem yapmak istediğini belirtir. Her method'un kendine özgü kullanım alanları ve semantic anlamları vardır.

**GET Method:** GET method'u, server'dan data retrieve etmek için kullanılır. Idempotent ve safe'tir. URL'de parametreler gönderilir. Cache'lenebilir.

**POST Method:** POST method'u, server'a data göndermek için kullanılır. Non-idempotent'tir. Request body'de data gönderilir. Cache'lenmez.

**PUT Method:** PUT method'u, server'da resource'u create veya update etmek için kullanılır. Idempotent'tir. Request body'de data gönderilir. Cache'lenmez.

**DELETE Method:** DELETE method'u, server'da resource'u silmek için kullanılır. Idempotent'tir. URL'de resource identifier gönderilir. Cache'lenmez.

**GET Kullanım Alanları:** Data retrieval, search, filtering, pagination gibi durumlarda kullanılır. RESTful API'lerde resource'ları getirmek için kullanılır.

**POST Kullanım Alanları:** Data creation, form submission, file upload, complex operations gibi durumlarda kullanılır. RESTful API'lerde resource oluşturmak için kullanılır.

**PUT Kullanım Alanları:** Resource creation, resource update, data replacement gibi durumlarda kullanılır. RESTful API'lerde resource'u tamamen değiştirmek için kullanılır.

**DELETE Kullanım Alanları:** Resource deletion, data removal gibi durumlarda kullanılır. RESTful API'lerde resource'u silmek için kullanılır.

**Best Practices:** HTTP method'ları, semantic anlamlarına uygun şekilde kullanılmalıdır. RESTful API design'ında doğru method'lar seçilmelidir.

**Security:** HTTP method'ları, security açısından farklı risk'ler taşır. GET method'u güvenli, POST method'u riskli olabilir.

#### 2. GET ve POST arasındaki farklar nelerdir?

**Cevap:**
GET ve POST, HTTP'nin en yaygın kullanılan iki method'udur. Her ikisi de farklı amaçlara hizmet eder ve farklı özelliklere sahiptir.

**Temel Farklar:** GET ve POST arasındaki temel farklar, data transmission, caching, security, idempotency açısından farklılık gösterir.

**Data Transmission:** GET method'unda data, URL'de query parameter olarak gönderilir. POST method'unda data, request body'de gönderilir.

**URL Length:** GET method'unda URL length limit'i vardır. POST method'unda request body size limit'i vardır.

**Caching:** GET method'u cache'lenebilir. POST method'u cache'lenmez.

**Security:** GET method'u güvenli değildir. POST method'u daha güvenlidir.

**Idempotency:** GET method'u idempotent'tir. POST method'u non-idempotent'tir.

**Safe:** GET method'u safe'tir. POST method'u safe değildir.

**Use Cases:** GET method'u, data retrieval için kullanılır. POST method'u, data submission için kullanılır.

**Browser Behavior:** GET method'u, browser'da bookmark edilebilir. POST method'u bookmark edilemez.

**Back Button:** GET method'u, browser'ın back button'u ile tekrar edilebilir. POST method'u tekrar edilemez.

**Best Practices:** GET method'u, data retrieval için kullanılmalıdır. POST method'u, data modification için kullanılmalıdır.

**RESTful API:** RESTful API'lerde GET method'u, resource'ları retrieve etmek için kullanılır. POST method'u, resource oluşturmak için kullanılır.

#### 3. PUT ve PATCH arasındaki fark nedir?

**Cevap:**
PUT ve PATCH, HTTP'nin resource update işlemleri için kullanılan iki method'udur. Her ikisi de farklı update stratejileri kullanır.

**Temel Fark:** PUT ve PATCH arasındaki temel fark, update stratejisidir. PUT, resource'u tamamen değiştirir. PATCH, resource'un sadece belirli kısımlarını değiştirir.

**PUT Method:** PUT method'u, resource'u tamamen değiştirir. Request body'de resource'un tüm data'sı gönderilir. Resource'un mevcut olmayan field'ları null olur.

**PATCH Method:** PATCH method'u, resource'un sadece belirli kısımlarını değiştirir. Request body'de sadece değiştirilecek field'lar gönderilir. Diğer field'lar değişmez.

**Idempotency:** PUT method'u idempotent'tir. PATCH method'u idempotent olmayabilir.

**Data Size:** PUT method'unda tüm resource data'sı gönderilir. PATCH method'unda sadece değiştirilecek data gönderilir.

**Use Cases:** PUT method'u, resource'u tamamen değiştirmek için kullanılır. PATCH method'u, resource'un belirli kısımlarını değiştirmek için kullanılır.

**RESTful API:** RESTful API'lerde PUT method'u, resource'u tamamen değiştirmek için kullanılır. PATCH method'u, partial update için kullanılır.

**Best Practices:** PUT method'u, resource'u tamamen değiştirmek için kullanılmalıdır. PATCH method'u, partial update için kullanılmalıdır.

**Performance:** PATCH method'u, daha az data gönderdiği için daha performanslıdır. PUT method'u, tüm data gönderdiği için daha yavaştır.

**Error Handling:** PUT method'unda, eksik field'lar null olur. PATCH method'unda, eksik field'lar değişmez.

**Validation:** PUT method'unda, tüm field'lar validate edilir. PATCH method'unda, sadece gönderilen field'lar validate edilir.

#### 4. HEAD ve OPTIONS method'larının işlevi nedir?

**Cevap:**
HEAD ve OPTIONS, HTTP'nin özel amaçlı method'larıdır. Her ikisi de farklı işlevlere sahiptir ve belirli durumlarda kullanılır.

**HEAD Method:** HEAD method'u, GET method'u ile aynı response'u döndürür ancak response body'si olmaz. Sadece header'lar döndürülür.

**OPTIONS Method:** OPTIONS method'u, server'ın desteklediği HTTP method'ları ve diğer özellikleri döndürür. CORS preflight request'lerde kullanılır.

**HEAD Kullanım Alanları:** HEAD method'u, resource'ın var olup olmadığını kontrol etmek, content length öğrenmek, last modified date öğrenmek için kullanılır.

**OPTIONS Kullanım Alanları:** OPTIONS method'u, CORS preflight request'lerde, server capability'lerini öğrenmek için kullanılır.

**Performance:** HEAD method'u, response body olmadığı için daha hızlıdır. OPTIONS method'u, server capability'lerini öğrenmek için kullanılır.

**Caching:** HEAD method'u, GET method'u ile aynı cache behavior'a sahiptir. OPTIONS method'u genellikle cache'lenmez.

**CORS:** OPTIONS method'u, CORS preflight request'lerde kullanılır. Browser, cross-origin request öncesi OPTIONS request gönderir.

**Best Practices:** HEAD method'u, resource metadata'sını öğrenmek için kullanılmalıdır. OPTIONS method'u, server capability'lerini öğrenmek için kullanılmalıdır.

**RESTful API:** RESTful API'lerde HEAD method'u, resource'ın var olup olmadığını kontrol etmek için kullanılır. OPTIONS method'u, API capability'lerini öğrenmek için kullanılır.

**Security:** HEAD method'u, GET method'u ile aynı security risk'lerine sahiptir. OPTIONS method'u, server information expose edebilir.

**Implementation:** HEAD method'u, GET method'u ile aynı şekilde implement edilir. OPTIONS method'u, server capability'lerini döndürür.

#### 5. HTTP status code'larının kategorileri nelerdir?

**Cevap:**
HTTP status code'ları, server'ın request'e verdiği response'un durumunu belirtir. Status code'lar, 3 haneli sayılardan oluşur ve farklı kategorilere ayrılır.

**1xx Informational:** 1xx status code'ları, informational response'ları belirtir. Request alındı ve işleniyor anlamına gelir.

**2xx Success:** 2xx status code'ları, successful response'ları belirtir. Request başarıyla işlendi anlamına gelir.

**3xx Redirection:** 3xx status code'ları, redirection response'ları belirtir. Request'in başka bir location'a yönlendirilmesi gerektiği anlamına gelir.

**4xx Client Error:** 4xx status code'ları, client error'ları belirtir. Request'te hata var anlamına gelir.

**5xx Server Error:** 5xx status code'ları, server error'ları belirtir. Server'da hata var anlamına gelir.

**Yaygın Status Code'lar:** 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error.

**200 OK:** Request başarıyla işlendi. GET, PUT, PATCH request'lerde kullanılır.

**201 Created:** Resource başarıyla oluşturuldu. POST request'lerde kullanılır.

**400 Bad Request:** Request'te syntax hatası var. Client'ın request'i düzeltmesi gerekir.

**401 Unauthorized:** Authentication gerekli. Client'ın authentication yapması gerekir.

**403 Forbidden:** Request geçerli ancak server işlemi reddediyor. Authorization problemi.

**404 Not Found:** Request edilen resource bulunamadı. URL yanlış olabilir.

**500 Internal Server Error:** Server'da beklenmeyen hata oluştu. Server'ın düzeltmesi gerekir.

**Best Practices:** HTTP status code'ları, semantic anlamlarına uygun şekilde kullanılmalıdır. RESTful API'lerde doğru status code'lar döndürülmelidir.

**Error Handling:** HTTP status code'ları, error handling'de önemli rol oynar. Client'lar, status code'lara göre farklı davranış sergiler.

### Orta Seviye

#### 6. Idempotent method'lar hangileridir?

**Cevap:**
Idempotent method'lar, aynı request'in birden fazla kez gönderilmesi durumunda aynı sonucu veren HTTP method'larıdır. Bu özellik, network reliability ve error handling açısından önemlidir.

**Idempotent Method'lar:** GET, PUT, DELETE, HEAD, OPTIONS method'ları idempotent'tir. POST ve PATCH method'ları idempotent değildir.

**GET Method:** GET method'u, resource'ı retrieve ettiği için idempotent'tir. Aynı request birden fazla kez gönderilse bile aynı sonucu verir.

**PUT Method:** PUT method'u, resource'u tamamen değiştirdiği için idempotent'tir. Aynı request birden fazla kez gönderilse bile aynı sonucu verir.

**DELETE Method:** DELETE method'u, resource'u sildiği için idempotent'tir. Aynı request birden fazla kez gönderilse bile aynı sonucu verir.

**HEAD Method:** HEAD method'u, GET method'u ile aynı davranışı sergilediği için idempotent'tir.

**OPTIONS Method:** OPTIONS method'u, server capability'lerini döndürdüğü için idempotent'tir.

**POST Method:** POST method'u, resource oluşturduğu için idempotent değildir. Aynı request birden fazla kez gönderilirse farklı sonuçlar verebilir.

**PATCH Method:** PATCH method'u, partial update yaptığı için idempotent olmayabilir. Update logic'ine bağlıdır.

**Network Reliability:** Idempotent method'lar, network failure durumlarında güvenli şekilde retry edilebilir.

**Error Handling:** Idempotent method'lar, error handling'de daha güvenlidir. Client'lar, request'i tekrar gönderebilir.

**Best Practices:** Idempotent method'lar, network reliability için önemlidir. POST method'u, idempotent olmayan işlemler için kullanılmalıdır.

**RESTful API:** RESTful API'lerde idempotent method'lar, resource operation'ları için kullanılmalıdır.

**Caching:** Idempotent method'lar, cache'lenebilir. Non-idempotent method'lar cache'lenmez.

#### 7. Safe method'ların özellikleri nelerdir?

**Cevap:**
Safe method'lar, server'da herhangi bir değişiklik yapmayan HTTP method'larıdır. Bu method'lar, resource'ları sadece retrieve eder ve server state'ini değiştirmez.

**Safe Method'lar:** GET, HEAD, OPTIONS method'ları safe'tir. POST, PUT, PATCH, DELETE method'ları safe değildir.

**GET Method:** GET method'u, resource'ı retrieve ettiği için safe'tir. Server state'ini değiştirmez.

**HEAD Method:** HEAD method'u, resource metadata'sını retrieve ettiği için safe'tir. Server state'ini değiştirmez.

**OPTIONS Method:** OPTIONS method'u, server capability'lerini döndürdüğü için safe'tir. Server state'ini değiştirmez.

**POST Method:** POST method'u, resource oluşturduğu için safe değildir. Server state'ini değiştirir.

**PUT Method:** PUT method'u, resource'u değiştirdiği için safe değildir. Server state'ini değiştirir.

**PATCH Method:** PATCH method'u, resource'u değiştirdiği için safe değildir. Server state'ini değiştirir.

**DELETE Method:** DELETE method'u, resource'u sildiği için safe değildir. Server state'ini değiştirir.

**Caching:** Safe method'lar, cache'lenebilir. Non-safe method'lar cache'lenmez.

**Browser Behavior:** Safe method'lar, browser'da bookmark edilebilir. Non-safe method'lar bookmark edilemez.

**Back Button:** Safe method'lar, browser'ın back button'u ile tekrar edilebilir. Non-safe method'lar tekrar edilemez.

**Security:** Safe method'lar, güvenli kabul edilir. Non-safe method'lar, security risk'i taşır.

**Best Practices:** Safe method'lar, data retrieval için kullanılmalıdır. Non-safe method'lar, data modification için kullanılmalıdır.

**RESTful API:** RESTful API'lerde safe method'lar, resource'ları retrieve etmek için kullanılmalıdır.

**Error Handling:** Safe method'lar, error handling'de daha güvenlidir. Client'lar, request'i tekrar gönderebilir.

#### 8. HTTP method'ların cache behavior'ı nasıldır?

**Cevap:**
HTTP method'ların cache behavior'ı, method'un safe olup olmadığına ve idempotent olup olmadığına bağlıdır. Cache'leme, performance ve network efficiency açısından önemlidir.

**Cacheable Method'lar:** GET, HEAD method'ları cache'lenebilir. POST, PUT, PATCH, DELETE method'ları cache'lenmez.

**GET Method:** GET method'u, safe ve idempotent olduğu için cache'lenebilir. Response'lar cache'lenir ve tekrar kullanılır.

**HEAD Method:** HEAD method'u, safe ve idempotent olduğu için cache'lenebilir. Response header'ları cache'lenir.

**POST Method:** POST method'u, safe ve idempotent olmadığı için cache'lenmez. Her request fresh olarak gönderilir.

**PUT Method:** PUT method'u, safe olmadığı için cache'lenmez. Her request fresh olarak gönderilir.

**PATCH Method:** PATCH method'u, safe olmadığı için cache'lenmez. Her request fresh olarak gönderilir.

**DELETE Method:** DELETE method'u, safe olmadığı için cache'lenmez. Her request fresh olarak gönderilir.

**Cache Headers:** Cache behavior, HTTP header'ları ile kontrol edilir. Cache-Control, Expires, ETag gibi header'lar kullanılır.

**Cache-Control:** Cache-Control header'ı, cache behavior'ını kontrol eder. max-age, no-cache, no-store gibi directive'ler kullanılır.

**ETag:** ETag header'ı, resource'ın version'ını belirtir. Conditional request'ler için kullanılır.

**Last-Modified:** Last-Modified header'ı, resource'ın son değiştirilme tarihini belirtir. Conditional request'ler için kullanılır.

**Conditional Requests:** If-Modified-Since, If-None-Match gibi header'lar ile conditional request'ler yapılır.

**Best Practices:** Cache'leme, performance için önemlidir. Safe method'lar cache'lenmeli, non-safe method'lar cache'lenmemelidir.

**RESTful API:** RESTful API'lerde cache'leme, GET request'ler için kullanılmalıdır.

**Security:** Cache'leme, security açısından dikkatli yapılmalıdır. Sensitive data cache'lenmemelidir.

#### 9. Custom HTTP method'lar nasıl implement edilir?

**Cevap:**
Custom HTTP method'lar, standard HTTP method'ları dışında özel amaçlı method'lar oluşturmak için kullanılır. Bu method'lar, specific use case'ler için tasarlanır.

**Temel Konsept:** Custom HTTP method'lar, HTTP specification'ında tanımlanmamış method'lardır. Server ve client tarafında implement edilmelidir.

**Method Naming:** Custom method'lar, descriptive name'ler almalıdır. VERB format'ında yazılmalıdır.

**Server Implementation:** Server tarafında custom method'lar, HTTP handler'lar ile implement edilir. Method name'i kontrol edilir.

**Client Implementation:** Client tarafında custom method'lar, HTTP client library'leri ile implement edilir. Method name'i request'te belirtilir.

**Use Cases:** Custom method'lar, specific operation'lar için kullanılır. SEARCH, LOCK, UNLOCK gibi method'lar örnek verilebilir.

**SEARCH Method:** SEARCH method'u, resource'ları aramak için kullanılır. GET method'undan farklı olarak complex query'ler destekler.

**LOCK Method:** LOCK method'u, resource'ları kilitlemek için kullanılır. Concurrent access'i kontrol eder.

**UNLOCK Method:** UNLOCK method'u, resource'ların kilidini açmak için kullanılır. LOCK method'u ile birlikte kullanılır.

**PURGE Method:** PURGE method'u, cache'leri temizlemek için kullanılır. Cache management için kullanılır.

**Best Practices:** Custom method'lar, sadece gerekli olduğunda kullanılmalıdır. Standard method'lar tercih edilmelidir.

**Compatibility:** Custom method'lar, client compatibility'yi etkileyebilir. Standard method'lar daha uyumludur.

**Documentation:** Custom method'lar, API documentation'da açık şekilde belirtilmelidir.

**Testing:** Custom method'lar, comprehensive testing gerektirir. Standard method'lar daha iyi test edilmiştir.

#### 10. HTTP method override nasıl çalışır?

**Cevap:**
HTTP method override, client'ların HTTP method'larını override etmesini sağlayan mekanizmadır. Bu, browser limitation'ları ve proxy restriction'ları aşmak için kullanılır.

**Temel Konsept:** HTTP method override, client'ların HTTP method'larını değiştirmesini sağlar. Header'lar veya form field'ları ile yapılır.

**X-HTTP-Method-Override Header:** X-HTTP-Method-Override header'ı, HTTP method'u override etmek için kullanılır. PUT, DELETE gibi method'lar için kullanılır.

**Form Field Override:** Form field'ları ile HTTP method override yapılabilir. _method field'ı kullanılır.

**Use Cases:** HTTP method override, browser limitation'ları aşmak için kullanılır. HTML form'ları sadece GET ve POST destekler.

**Browser Limitation:** Browser'lar, HTML form'larında sadece GET ve POST method'larını destekler. PUT, DELETE gibi method'lar desteklenmez.

**Proxy Restriction:** Proxy'ler, belirli HTTP method'larını bloke edebilir. Method override ile bu restriction'lar aşılabilir.

**RESTful API:** RESTful API'lerde method override, browser compatibility için kullanılır.

**Security Considerations:** HTTP method override, security risk'i yaratabilir. Method validation yapılmalıdır.

**Server Implementation:** Server tarafında method override, header'lar veya form field'ları kontrol edilerek implement edilir.

**Client Implementation:** Client tarafında method override, header'lar veya form field'ları set edilerek implement edilir.

**Best Practices:** HTTP method override, sadece gerekli olduğunda kullanılmalıdır. Native HTTP method'lar tercih edilmelidir.

**Validation:** HTTP method override, server tarafında validate edilmelidir. Malicious override'lar engellenmelidir.

**Documentation:** HTTP method override, API documentation'da belirtilmelidir. Client'lar bu özelliği bilmelidir.

---

## Apache Kafka

### Temel Seviye

#### 1. Kafka'nın temel bileşenleri nelerdir?

**Cevap:**
Apache Kafka, distributed streaming platform'unun temel bileşenleri, high-throughput, low-latency data streaming sağlamak için tasarlanmıştır. Her bileşen, farklı işlevlere sahiptir.

**Broker:** Kafka broker'ları, Kafka cluster'ının temel node'larıdır. Topic'leri store eder, producer'ların data göndermesini ve consumer'ların data okumasını sağlar.

**Topic:** Topic'ler, data'nın kategorize edildiği logical channel'lardır. Producer'lar topic'lere data gönderir, consumer'lar topic'lerden data okur.

**Partition:** Partition'lar, topic'lerin physical subdivision'larıdır. Her partition, ordered sequence of record'lar içerir. Parallelism ve scalability sağlar.

**Producer:** Producer'lar, Kafka topic'lerine data gönderen application'lardır. Data'yı serialize eder ve broker'lara gönderir.

**Consumer:** Consumer'lar, Kafka topic'lerinden data okuyan application'lardır. Data'yı deserialize eder ve process eder.

**Consumer Group:** Consumer group'lar, topic'leri birlikte consume eden consumer'ların collection'ıdır. Load balancing ve fault tolerance sağlar.

**Zookeeper:** Zookeeper, Kafka cluster'ının metadata'sını store eder. Broker coordination, configuration management, leader election yapar.

**Schema Registry:** Schema Registry, data schema'larını store eder. Data compatibility ve evolution sağlar.

**Kafka Connect:** Kafka Connect, external system'ler ile Kafka arasında data transfer sağlar. Source ve sink connector'ları sağlar.

**Kafka Streams:** Kafka Streams, Kafka'da real-time data processing yapmayı sağlar. Stream processing library'sidir.

**Best Practices:** Kafka bileşenleri, scalability ve fault tolerance için tasarlanmıştır. Proper configuration ve monitoring gerektirir.

**Use Cases:** Kafka, event streaming, log aggregation, real-time analytics, data integration gibi durumlarda kullanılır.

#### 2. Topic, Partition ve Offset kavramları nelerdir?

**Cevap:**
Topic, Partition ve Offset, Kafka'nın temel data organization kavramlarıdır. Bu kavramlar, data'nın nasıl organize edildiğini ve erişildiğini belirler.

**Topic:** Topic, data'nın kategorize edildiği logical channel'dır. Producer'lar topic'lere data gönderir, consumer'lar topic'lerden data okur. Topic'ler, named category'lerdir.

**Partition:** Partition, topic'in physical subdivision'ıdır. Her topic, bir veya daha fazla partition'a bölünebilir. Partition'lar, ordered sequence of record'lar içerir.

**Offset:** Offset, partition içindeki her record'ın unique identifier'ıdır. Monotonically increasing integer'dır. Consumer'lar, offset'leri kullanarak data okur.

**Topic Structure:** Topic'ler, partition'lara bölünür. Her partition, ordered sequence of record'lar içerir. Partition'lar, farklı broker'larda store edilebilir.

**Partition Benefits:** Partition'lar, parallelism sağlar. Multiple consumer'lar aynı anda farklı partition'ları okuyabilir. Scalability artar.

**Offset Management:** Offset'ler, consumer'lar tarafından manage edilir. Consumer'lar, hangi offset'ten okumaya devam edeceklerini belirtir.

**Ordering:** Partition içinde record'lar ordered'dır. Topic seviyesinde ordering yoktur. Partition seviyesinde ordering vardır.

**Replication:** Partition'lar, multiple broker'larda replicate edilebilir. Fault tolerance sağlar. Leader ve follower partition'lar vardır.

**Leader Election:** Her partition'ın bir leader'ı vardır. Leader, read ve write operation'ları handle eder. Follower'lar, leader'ı replicate eder.

**Consumer Group:** Consumer group'lar, partition'ları distribute eder. Her partition, sadece bir consumer tarafından okunur.

**Best Practices:** Topic'ler, logical separation için kullanılmalıdır. Partition'lar, parallelism için kullanılmalıdır. Offset'ler, consumer progress tracking için kullanılmalıdır.

**Use Cases:** Topic'ler, event categorization için kullanılır. Partition'lar, scalability için kullanılır. Offset'ler, data processing tracking için kullanılır.

#### 3. Producer ve Consumer'ın rolleri nelerdir?

**Cevap:**
Producer ve Consumer, Kafka'nın temel data flow bileşenleridir. Producer'lar data gönderir, Consumer'lar data okur. Her ikisi de farklı rollere sahiptir.

**Producer Role:** Producer'lar, Kafka topic'lerine data gönderen application'lardır. Data'yı serialize eder, partition'a gönderir. Data publishing yapar.

**Consumer Role:** Consumer'lar, Kafka topic'lerinden data okuyan application'lardır. Data'yı deserialize eder, process eder. Data consumption yapar.

**Producer Responsibilities:** Producer'lar, data serialization, partition selection, error handling, retry logic yapar. Data quality ve delivery guarantee sağlar.

**Consumer Responsibilities:** Consumer'lar, data deserialization, offset management, error handling, data processing yapar. Data processing ve business logic uygular.

**Data Flow:** Producer'lar data gönderir → Kafka broker'ları store eder → Consumer'lar data okur. Asynchronous data flow sağlar.

**Partition Selection:** Producer'lar, partition selection yapar. Round-robin, key-based, custom partitioner kullanabilir. Load balancing sağlar.

**Offset Management:** Consumer'lar, offset management yapar. Manual veya automatic offset commit yapabilir. Data processing progress track eder.

**Error Handling:** Producer'lar, send error'ları handle eder. Retry logic, dead letter queue kullanabilir. Consumer'lar, processing error'ları handle eder.

**Serialization:** Producer'lar, data'yı serialize eder. JSON, Avro, Protobuf gibi format'lar kullanabilir. Consumer'lar, data'yı deserialize eder.

**Batch Processing:** Producer'lar, batch sending yapabilir. Performance artırır. Consumer'lar, batch processing yapabilir. Throughput artırır.

**Best Practices:** Producer'lar, idempotent sending yapmalıdır. Consumer'lar, idempotent processing yapmalıdır. Error handling implement edilmelidir.

**Use Cases:** Producer'lar, event publishing, log streaming, data ingestion için kullanılır. Consumer'lar, event processing, analytics, data transformation için kullanılır.

**Monitoring:** Producer'lar, send rate, error rate monitor edilmelidir. Consumer'lar, processing rate, lag monitor edilmelidir.

#### 4. Kafka'nın traditional messaging system'lerden farkı nedir?

**Cevap:**
Kafka, traditional messaging system'lerden farklı olarak distributed streaming platform'dur. High-throughput, low-latency, fault-tolerant data streaming sağlar.

**Architecture Difference:** Kafka, distributed architecture kullanır. Traditional messaging system'ler, centralized architecture kullanır. Kafka, horizontal scaling sağlar.

**Data Storage:** Kafka, data'yı disk'te store eder. Traditional messaging system'ler, data'yı memory'de store eder. Kafka, data persistence sağlar.

**Throughput:** Kafka, very high throughput sağlar. Traditional messaging system'ler, limited throughput sağlar. Kafka, millions of messages per second handle edebilir.

**Latency:** Kafka, low latency sağlar. Traditional messaging system'ler, variable latency sağlar. Kafka, sub-millisecond latency sağlar.

**Durability:** Kafka, data durability sağlar. Traditional messaging system'ler, limited durability sağlar. Kafka, data'yı persistent store eder.

**Scalability:** Kafka, horizontal scaling sağlar. Traditional messaging system'ler, vertical scaling sağlar. Kafka, cluster'ı expand edebilir.

**Fault Tolerance:** Kafka, fault tolerance sağlar. Traditional messaging system'ler, limited fault tolerance sağlar. Kafka, replication ve leader election sağlar.

**Message Ordering:** Kafka, partition seviyesinde ordering sağlar. Traditional messaging system'ler, global ordering sağlar. Kafka, partition-based ordering sağlar.

**Consumer Model:** Kafka, pull-based consumer model kullanır. Traditional messaging system'ler, push-based model kullanır. Kafka, consumer control sağlar.

**Data Retention:** Kafka, configurable data retention sağlar. Traditional messaging system'ler, limited retention sağlar. Kafka, long-term storage sağlar.

**Use Cases:** Kafka, event streaming, log aggregation, real-time analytics için kullanılır. Traditional messaging system'ler, request-response, RPC için kullanılır.

**Best Practices:** Kafka, high-throughput, low-latency use case'ler için kullanılmalıdır. Traditional messaging system'ler, simple messaging için kullanılmalıdır.

**Migration:** Traditional messaging system'lerden Kafka'ya migration, architecture change gerektirir. Data model ve consumer logic değişir.

#### 5. Kafka Connect'in amacı nedir?

**Cevap:**
Kafka Connect, external system'ler ile Kafka arasında data transfer sağlayan framework'tür. Data integration, data pipeline oluşturmayı kolaylaştırır.

**Temel Amaç:** Kafka Connect, external system'ler ile Kafka arasında scalable, reliable data transfer sağlar. Data integration'ı standardize eder.

**Source Connector:** Source connector'lar, external system'lerden Kafka'ya data çeker. Database'ler, file system'ler, message queue'lar gibi system'lerden data alır.

**Sink Connector:** Sink connector'lar, Kafka'dan external system'lere data gönderir. Database'ler, data warehouse'lar, search engine'ler gibi system'lere data gönderir.

**Distributed Mode:** Kafka Connect, distributed mode'da çalışabilir. Multiple worker node'ları kullanır. Scalability ve fault tolerance sağlar.

**Standalone Mode:** Kafka Connect, standalone mode'da çalışabilir. Single process'te çalışır. Development ve testing için uygundur.

**Configuration:** Kafka Connect, configuration-based approach kullanır. JSON configuration file'ları ile connector'lar configure edilir.

**Transformations:** Kafka Connect, data transformation sağlar. Single Message Transform (SMT) kullanır. Data format conversion, filtering yapar.

**Error Handling:** Kafka Connect, error handling sağlar. Dead letter queue, retry logic kullanır. Data loss'u önler.

**Monitoring:** Kafka Connect, monitoring sağlar. Connector status, task status, error rate monitor edilir. JMX metrics sağlar.

**Use Cases:** Kafka Connect, data integration, ETL pipeline'lar, data synchronization için kullanılır. Database replication, log aggregation için kullanılır.

**Best Practices:** Kafka Connect, data integration için kullanılmalıdır. Custom connector'lar, specific use case'ler için yazılabilir.

**Performance:** Kafka Connect, high-throughput data transfer sağlar. Parallel processing, batch processing kullanır. Performance optimize edilir.

**Security:** Kafka Connect, security sağlar. Authentication, authorization, encryption destekler. Secure data transfer sağlar.

### Orta Seviye

#### 6. Kafka Streams'in kullanım alanları nelerdir?

**Cevap:**
Kafka Streams, Kafka'da real-time data processing yapmayı sağlayan stream processing library'sidir. Event-driven application'lar, real-time analytics, data transformation için kullanılır.

**Temel Konsept:** Kafka Streams, Kafka topic'lerinden data okuyarak real-time processing yapar. Processed data'yı başka topic'lere yazar. Stream processing sağlar.

**Event Processing:** Kafka Streams, event processing için kullanılır. Real-time event'leri process eder, business logic uygular. Event-driven architecture sağlar.

**Data Transformation:** Kafka Streams, data transformation için kullanılır. Data format conversion, data enrichment, data filtering yapar. ETL pipeline'lar oluşturur.

**Real-time Analytics:** Kafka Streams, real-time analytics için kullanılır. Aggregation, windowing, join operation'ları yapar. Real-time dashboard'lar oluşturur.

**Stream Processing:** Kafka Streams, stream processing için kullanılır. Continuous data processing, windowed operations, stateful processing sağlar.

**Microservices:** Kafka Streams, microservices architecture'da kullanılır. Service-to-service communication, event sourcing, CQRS pattern'ları implement eder.

**Data Pipeline:** Kafka Streams, data pipeline'lar oluşturur. Data ingestion, processing, transformation, output sağlar. End-to-end data flow oluşturur.

**Stateful Processing:** Kafka Streams, stateful processing sağlar. Local state store'lar kullanır. Aggregation, join, windowing operation'ları yapar.

**Windowing:** Kafka Streams, windowing operation'ları sağlar. Tumbling window, hopping window, session window kullanır. Time-based processing yapar.

**Join Operations:** Kafka Streams, join operation'ları sağlar. Stream-stream join, stream-table join yapar. Data correlation sağlar.

**Best Practices:** Kafka Streams, real-time processing için kullanılmalıdır. Stateful operation'lar dikkatli implement edilmelidir.

**Use Cases:** Kafka Streams, real-time analytics, fraud detection, recommendation system'ler, IoT data processing için kullanılır.

**Performance:** Kafka Streams, high-throughput, low-latency processing sağlar. Parallel processing, local state store'lar kullanır.

#### 7. Kafka'nın partitioning stratejileri nelerdir?

**Cevap:**
Kafka'nın partitioning stratejileri, data'nın partition'lara nasıl dağıtılacağını belirler. Farklı stratejiler, farklı use case'ler için optimize edilmiştir.

**Round-Robin Partitioning:** Round-robin partitioning, data'yı partition'lar arasında eşit olarak dağıtır. Load balancing sağlar. Ordering guarantee yoktur.

**Key-Based Partitioning:** Key-based partitioning, data'yı key'e göre partition'lara dağıtır. Aynı key'e sahip record'lar aynı partition'a gider. Ordering guarantee sağlar.

**Custom Partitioning:** Custom partitioning, custom partitioner kullanır. Business logic'e göre partition selection yapar. Flexible partitioning sağlar.

**Hash Partitioning:** Hash partitioning, key'in hash'ini kullanır. Consistent partitioning sağlar. Key distribution'ı optimize eder.

**Range Partitioning:** Range partitioning, key range'lerine göre partition selection yapar. Ordered data için uygundur. Range-based processing sağlar.

**Partition Selection:** Producer'lar, partition selection yapar. Partition number, key hash, custom logic kullanır. Load balancing sağlar.

**Key Selection:** Key selection, partitioning'i etkiler. Null key, round-robin partitioning kullanır. Non-null key, key-based partitioning kullanır.

**Partition Count:** Partition count, parallelism'i belirler. Daha fazla partition, daha fazla parallelism sağlar. Consumer scaling'i etkiler.

**Rebalancing:** Partition rebalancing, partition'ları consumer'lar arasında yeniden dağıtır. Consumer group membership değiştiğinde yapılır.

**Ordering:** Partitioning, ordering'i etkiler. Key-based partitioning, key seviyesinde ordering sağlar. Round-robin partitioning, ordering sağlamaz.

**Best Practices:** Partitioning stratejisi, use case'e göre seçilmelidir. Key-based partitioning, ordering gerektiren durumlarda kullanılmalıdır.

**Use Cases:** Round-robin partitioning, load balancing için kullanılır. Key-based partitioning, ordering için kullanılır. Custom partitioning, specific logic için kullanılır.

**Performance:** Partitioning stratejisi, performance'ı etkiler. Key-based partitioning, hot partition'lara neden olabilir. Round-robin partitioning, even distribution sağlar.

#### 8. Consumer group'ların çalışma prensibi nasıldır?

**Cevap:**
Consumer group'lar, Kafka'da load balancing ve fault tolerance sağlayan mekanizmadır. Multiple consumer'lar birlikte çalışarak topic'leri consume eder.

**Temel Konsept:** Consumer group, topic'leri birlikte consume eden consumer'ların collection'ıdır. Partition'ları consumer'lar arasında dağıtır. Load balancing sağlar.

**Partition Assignment:** Consumer group'lar, partition'ları consumer'lar arasında assign eder. Her partition, sadece bir consumer tarafından okunur. No overlap sağlar.

**Load Balancing:** Consumer group'lar, load balancing sağlar. Partition'lar consumer'lar arasında eşit olarak dağıtılır. Throughput artar.

**Fault Tolerance:** Consumer group'lar, fault tolerance sağlar. Consumer fail olduğunda, partition'lar diğer consumer'lara reassign edilir. No data loss sağlar.

**Rebalancing:** Consumer group'lar, rebalancing yapar. Consumer join/leave olduğunda, partition'lar yeniden assign edilir. Dynamic scaling sağlar.

**Coordinator:** Consumer group'lar, group coordinator kullanır. Rebalancing, offset management yapar. Group membership manage eder.

**Offset Management:** Consumer group'lar, offset management yapar. Group seviyesinde offset'ler store edilir. Consumer progress track edilir.

**Commit Strategy:** Consumer group'lar, offset commit strategy kullanır. Automatic veya manual commit yapabilir. Data processing guarantee sağlar.

**Group Membership:** Consumer group'lar, group membership manage eder. Consumer join/leave event'lerini handle eder. Dynamic membership sağlar.

**Partition Reassignment:** Consumer group'lar, partition reassignment yapar. Consumer fail olduğunda, partition'lar reassign edilir. Fault tolerance sağlar.

**Best Practices:** Consumer group'lar, load balancing için kullanılmalıdır. Partition count, consumer count'a göre optimize edilmelidir.

**Use Cases:** Consumer group'lar, parallel processing, fault tolerance, scaling için kullanılır. High-throughput consumption için kullanılır.

**Performance:** Consumer group'lar, performance'ı artırır. Parallel processing, load balancing sağlar. Throughput optimize eder.

**Monitoring:** Consumer group'lar, monitoring gerektirir. Consumer lag, partition assignment, rebalancing monitor edilmelidir.

#### 9. Kafka'nın durability ve reliability garantileri nelerdir?

**Cevap:**
Kafka'nın durability ve reliability garantileri, data'nın güvenli şekilde store edilmesini ve işlenmesini sağlar. Production environment'lerde kritik öneme sahiptir.

**Temel Garantiler:** Kafka, data durability, message ordering, delivery guarantee sağlar. Replication, acknowledgment mechanism kullanır.

**Durability:** Kafka, data durability sağlar. Data disk'te persistent store edilir. Broker fail olduğunda data kaybolmaz.

**Reliability:** Kafka, reliability sağlar. Replication, leader election, fault tolerance mekanizmaları kullanır. High availability sağlar.

**Replication:** Kafka, replication kullanır. Partition'lar multiple broker'larda replicate edilir. Data redundancy sağlar.

**Acknowledgment:** Kafka, acknowledgment mechanism kullanır. Producer'lar, write acknowledgment alır. Data delivery guarantee sağlar.

**In-Sync Replicas (ISR):** ISR, leader ile sync olan follower'ların listesidir. Data consistency için kullanılır. Write operation'ları ISR'ye göre yapılır.

**Min In-Sync Replicas:** Min ISR, minimum sync replica sayısını belirler. Write operation'ları bu sayıya göre yapılır. Data consistency sağlar.

**Leader Election:** Kafka, leader election yapar. Leader fail olduğunda, yeni leader seçilir. High availability sağlar.

**Message Ordering:** Kafka, partition seviyesinde message ordering sağlar. Aynı partition'daki message'lar ordered'dır.

**Delivery Semantics:** Kafka, delivery semantics sağlar. At-least-once, at-most-once, exactly-once delivery sağlar.

**Best Practices:** Durability ve reliability, production environment'lerde kritiktir. Replication factor, min ISR ayarlanmalıdır.

**Use Cases:** Durability ve reliability, financial system'ler, critical application'lar için önemlidir. Data loss'u önler.

**Performance:** Durability ve reliability, performance overhead yaratır. Write latency artar. Durability vs performance trade-off'u vardır.

**Monitoring:** Durability ve reliability, monitoring gerektirir. ISR size, follower lag, leader election monitor edilmelidir.

#### 10. Schema Registry'nin rolü nedir?

**Cevap:**
Schema Registry, Kafka'da data schema'larını store eden ve manage eden service'dir. Data compatibility, evolution, validation sağlar.

**Temel Rol:** Schema Registry, data schema'larını centralize eder. Schema versioning, compatibility checking, validation sağlar.

**Schema Storage:** Schema Registry, schema'ları store eder. Avro, JSON Schema, Protobuf format'larını destekler. Versioned schema storage sağlar.

**Schema Evolution:** Schema Registry, schema evolution sağlar. Backward compatibility, forward compatibility kontrol eder. Schema change'leri manage eder.

**Compatibility Checking:** Schema Registry, compatibility checking yapar. Schema change'lerin compatibility'sini kontrol eder. Breaking change'leri önler.

**Schema Validation:** Schema Registry, schema validation sağlar. Data'nın schema'ya uygunluğunu kontrol eder. Data quality sağlar.

**Versioning:** Schema Registry, schema versioning sağlar. Schema'ların version'larını track eder. Version history maintain eder.

**Client Integration:** Schema Registry, client integration sağlar. Producer'lar ve consumer'lar schema'ları kullanır. Serialization/deserialization sağlar.

**REST API:** Schema Registry, REST API sağlar. Schema'ları programmatically manage eder. Integration sağlar.

**Security:** Schema Registry, security sağlar. Authentication, authorization, encryption destekler. Secure schema management sağlar.

**Best Practices:** Schema Registry, data governance için kullanılmalıdır. Schema evolution, compatibility checking yapılmalıdır.

**Use Cases:** Schema Registry, data integration, API evolution, data quality için kullanılır. Microservices architecture'da önemlidir.

**Performance:** Schema Registry, performance overhead yaratır. Schema lookup, validation overhead'i vardır. Caching kullanılmalıdır.

**Monitoring:** Schema Registry, monitoring gerektirir. Schema usage, compatibility check, validation rate monitor edilmelidir.

#### 11. Kafka'nın performance tuning parametreleri nelerdir?

**Cevap:**
Kafka'nın performance tuning parametreleri, system performance'ını optimize etmek için kullanılır. Throughput, latency, resource usage optimize edilir.

**Temel Parametreler:** Kafka performance tuning, broker parametreleri, producer parametreleri, consumer parametreleri, topic parametreleri içerir.

**Broker Parameters:** Broker parametreleri, num.network.threads, num.io.threads, socket.send.buffer.bytes, socket.receive.buffer.bytes içerir.

**Producer Parameters:** Producer parametreleri, batch.size, linger.ms, compression.type, acks, retries içerir.

**Consumer Parameters:** Consumer parametreleri, fetch.min.bytes, fetch.max.wait.ms, max.partition.fetch.bytes, session.timeout.ms içerir.

**Topic Parameters:** Topic parametreleri, num.partitions, replication.factor, min.insync.replicas, retention.ms içerir.

**Network Tuning:** Network tuning, socket buffer size, network thread count, I/O thread count optimize eder.

**Memory Tuning:** Memory tuning, heap size, page cache, buffer size optimize eder. JVM tuning yapılır.

**Disk Tuning:** Disk tuning, disk I/O, file system, storage optimize eder. SSD kullanımı önerilir.

**Compression:** Compression, data size'ı azaltır. Throughput artar, CPU usage artar. Compression type seçilir.

**Batching:** Batching, message'ları batch'ler halinde gönderir. Throughput artar, latency artar. Batch size optimize edilir.

**Best Practices:** Performance tuning, workload'a göre yapılmalıdır. Benchmarking, monitoring yapılmalıdır.

**Use Cases:** Performance tuning, high-throughput, low-latency application'lar için önemlidir. Production environment'lerde kritiktir.

**Monitoring:** Performance tuning, monitoring gerektirir. Throughput, latency, resource usage monitor edilmelidir.

**Tools:** Performance tuning, JMX metrics, monitoring tool'ları ile yapılır. Custom dashboard'lar oluşturulur.

#### 12. Exactly-once semantics nasıl sağlanır?

**Cevap:**
Exactly-once semantics, Kafka'da message'ların tam olarak bir kez işlenmesini garanti eder. Duplicate processing'i önler, data consistency sağlar.

**Temel Konsept:** Exactly-once semantics, message'ların duplicate olmadan, loss olmadan işlenmesini sağlar. Idempotent processing gerektirir.

**Idempotent Producer:** Idempotent producer, duplicate message'ları önler. Producer ID, sequence number kullanır. Duplicate detection sağlar.

**Transactional API:** Transactional API, exactly-once semantics sağlar. Transaction boundary'leri kullanır. Atomic operation'lar sağlar.

**Consumer Group:** Consumer group, exactly-once semantics sağlar. Offset management, rebalancing handle eder. Duplicate processing önler.

**Kafka Streams:** Kafka Streams, exactly-once semantics sağlar. Stateful processing, transaction support sağlar. End-to-end guarantee sağlar.

**Two-Phase Commit:** Two-phase commit, exactly-once semantics sağlar. Prepare, commit phase'leri kullanır. Atomic operation'lar sağlar.

**Offset Management:** Offset management, exactly-once semantics sağlar. Offset commit, processing order kontrol eder. Duplicate processing önler.

**State Store:** State store, exactly-once semantics sağlar. Local state, transaction support sağlar. Consistency guarantee sağlar.

**Best Practices:** Exactly-once semantics, idempotent processing gerektirir. Transaction boundary'leri dikkatli define edilmelidir.

**Use Cases:** Exactly-once semantics, financial system'ler, critical application'lar için önemlidir. Data consistency gerektiren durumlarda kullanılır.

**Performance:** Exactly-once semantics, performance overhead yaratır. Transaction overhead, coordination overhead vardır.

**Monitoring:** Exactly-once semantics, monitoring gerektirir. Transaction success rate, duplicate detection rate monitor edilmelidir.

---

## Spring Async

### Temel Seviye

#### 1. @Async annotation'ının kullanımı nasıldır?

**Cevap:**
@Async annotation'ı, Spring Framework'te method'ları asynchronous olarak çalıştırmak için kullanılır. Bu annotation, method'ların background thread'lerde çalışmasını sağlar.

**Temel Kullanım:** @Async annotation'ı, method'ları asynchronous olarak işaretler. Method çağrıldığında, hemen return eder ve background'da çalışır.

**Method Level:** @Async annotation'ı, method level'da kullanılır. Sadece o method asynchronous olarak çalışır.

**Class Level:** @Async annotation'ı, class level'da kullanılabilir. Tüm public method'lar asynchronous olarak çalışır.

**Return Types:** @Async method'lar, void, CompletableFuture, ListenableFuture return edebilir. void method'lar fire-and-forget olarak çalışır.

**Configuration:** @Async annotation'ı, @EnableAsync ile aktif edilmelidir. AsyncConfigurer interface'i implement edilebilir.

**Thread Pool:** @Async method'lar, default thread pool'da çalışır. Custom thread pool configure edilebilir.

**Exception Handling:** @Async method'larda exception'lar, AsyncUncaughtExceptionHandler ile handle edilir.

**Self-Invocation:** @Async annotation'ı, self-invocation'da çalışmaz. Aynı class içinden çağrıldığında asynchronous olmaz.

**Best Practices:** @Async method'lar, long-running task'lar için kullanılmalıdır. CPU-intensive task'lar için uygun değildir.

**Use Cases:** @Async method'lar, email sending, file processing, external API call'ları için kullanılır.

**Performance:** @Async method'lar, response time'ı artırır. User experience'i iyileştirir.

**Monitoring:** @Async method'lar, monitoring gerektirir. Thread pool usage, execution time monitor edilmelidir.

#### 2. Asynchronous method'ların return type'ları nelerdir?

**Cevap:**
Asynchronous method'lar, farklı return type'ları kullanabilir. Her return type'ın kendine özgü kullanım alanları ve davranışları vardır.

**void Return Type:** void return type, fire-and-forget pattern için kullanılır. Method çağrıldığında hemen return eder, sonucu beklenmez.

**CompletableFuture Return Type:** CompletableFuture return type, Java 8'de tanıtılan modern async return type'ıdır. Future result'ı handle eder.

**ListenableFuture Return Type:** ListenableFuture return type, Spring'in async return type'ıdır. Callback-based result handling sağlar.

**Future Return Type:** Future return type, traditional async return type'ıdır. Blocking result retrieval sağlar.

**void Kullanımı:** void return type, sonucun önemli olmadığı durumlarda kullanılır. Email sending, logging gibi işlemler için uygundur.

**CompletableFuture Kullanımı:** CompletableFuture, modern async programming için kullanılır. Chain operation'lar, exception handling sağlar.

**ListenableFuture Kullanımı:** ListenableFuture, Spring ecosystem'inde kullanılır. Callback-based programming sağlar.

**Future Kullanımı:** Future, legacy code'lar için kullanılır. Blocking get() method'u ile result alınır.

**Exception Handling:** Farklı return type'lar, farklı exception handling sağlar. CompletableFuture, exception chaining sağlar.

**Chaining:** CompletableFuture, method chaining sağlar. thenApply(), thenCompose() gibi method'lar kullanılır.

**Best Practices:** Return type seçimi, use case'e göre yapılmalıdır. CompletableFuture, modern application'lar için önerilir.

**Use Cases:** void, fire-and-forget işlemler için kullanılır. CompletableFuture, complex async logic için kullanılır.

**Performance:** Return type seçimi, performance'ı etkiler. CompletableFuture, overhead yaratır.

#### 3. @EnableAsync annotation'ının işlevi nedir?

**Cevap:**
@EnableAsync annotation'ı, Spring Framework'te asynchronous processing'i aktif etmek için kullanılır. Bu annotation olmadan @Async method'lar çalışmaz.

**Temel İşlev:** @EnableAsync annotation'ı, Spring'in async processing mekanizmasını aktif eder. @Async annotation'ları çalışır hale getirir.

**Configuration Class:** @EnableAsync annotation'ı, @Configuration class'ında kullanılır. Application context'te async support sağlar.

**Proxy Creation:** @EnableAsync annotation'ı, AOP proxy'leri oluşturur. @Async method'lar proxy üzerinden çağrılır.

**Thread Pool:** @EnableAsync annotation'ı, default thread pool oluşturur. SimpleAsyncTaskExecutor kullanır.

**Custom Configuration:** @EnableAsync annotation'ı, AsyncConfigurer interface'i ile customize edilebilir. Custom executor, exception handler sağlar.

**Method Detection:** @EnableAsync annotation'ı, @Async annotation'ları detect eder. Method'ları async olarak işaretler.

**AOP Integration:** @EnableAsync annotation'ı, Spring AOP ile entegre çalışır. Method interception sağlar.

**Bean Creation:** @EnableAsync annotation'ı, async-related bean'leri oluşturur. TaskExecutor, AsyncUncaughtExceptionHandler sağlar.

**Best Practices:** @EnableAsync annotation'ı, application'ın root configuration'ında kullanılmalıdır.

**Use Cases:** @EnableAsync annotation'ı, async processing gerektiren application'larda kullanılır.

**Performance:** @EnableAsync annotation'ı, startup overhead yaratır. Proxy creation, thread pool creation yapar.

**Monitoring:** @EnableAsync annotation'ı, monitoring gerektirir. Thread pool usage, async method execution monitor edilmelidir.

#### 4. Async method'larda exception handling nasıl yapılır?

**Cevap:**
Async method'larda exception handling, synchronous method'lardan farklıdır. Exception'lar, calling thread'de değil, async thread'de oluşur.

**Temel Problem:** Async method'larda exception'lar, calling thread'de catch edilemez. Exception'lar, async thread'de oluşur.

**AsyncUncaughtExceptionHandler:** AsyncUncaughtExceptionHandler interface'i, async method'lardaki uncaught exception'ları handle eder.

**Custom Exception Handler:** Custom exception handler, AsyncConfigurer interface'i implement edilerek oluşturulur.

**CompletableFuture Exception Handling:** CompletableFuture, exception handling sağlar. exceptionally(), handle() method'ları kullanılır.

**ListenableFuture Exception Handling:** ListenableFuture, callback-based exception handling sağlar. Failure callback'leri kullanılır.

**Future Exception Handling:** Future, get() method'unda exception'ları fırlatır. ExecutionException wrap eder.

**Exception Propagation:** Async method'larda exception'lar, calling thread'e propagate edilmez. Exception handler'da handle edilmelidir.

**Logging:** Async method'larda exception'lar, logging yapılmalıdır. Exception handler'da log yazılır.

**Retry Logic:** Async method'larda retry logic, exception handler'da implement edilir. Failed task'lar retry edilir.

**Dead Letter Queue:** Async method'larda failed task'lar, dead letter queue'ya gönderilebilir. Error handling sağlar.

**Best Practices:** Async method'larda exception handling, comprehensive olmalıdır. All exception'lar handle edilmelidir.

**Use Cases:** Exception handling, async method'larda kritiktir. Email sending, file processing gibi işlemlerde önemlidir.

**Monitoring:** Exception handling, monitoring gerektirir. Exception rate, error pattern'lar monitor edilmelidir.

#### 5. CompletableFuture'ın kullanım alanları nelerdir?

**Cevap:**
CompletableFuture, Java 8'de tanıtılan modern async programming API'sidir. Complex async operation'ları handle etmek için kullanılır.

**Temel Kullanım:** CompletableFuture, async operation'ları represent eder. Future result'ı handle eder.

**Method Chaining:** CompletableFuture, method chaining sağlar. thenApply(), thenCompose(), thenCombine() gibi method'lar kullanılır.

**Exception Handling:** CompletableFuture, exception handling sağlar. exceptionally(), handle() method'ları kullanılır.

**Combining Futures:** CompletableFuture, multiple future'ları combine eder. thenCombine(), allOf(), anyOf() method'ları kullanılır.

**Async Execution:** CompletableFuture, async execution sağlar. supplyAsync(), runAsync() method'ları kullanılır.

**Callback-based:** CompletableFuture, callback-based programming sağlar. thenAccept(), thenRun() method'ları kullanılır.

**Timeout Handling:** CompletableFuture, timeout handling sağlar. orTimeout(), completeOnTimeout() method'ları kullanılır.

**Cancellation:** CompletableFuture, cancellation sağlar. cancel() method'u kullanılır.

**Best Practices:** CompletableFuture, complex async logic için kullanılmalıdır. Simple async işlemler için @Async yeterlidir.

**Use Cases:** CompletableFuture, parallel processing, async orchestration, complex async workflow'lar için kullanılır.

**Performance:** CompletableFuture, performance overhead yaratır. Memory usage, thread usage artar.

**Monitoring:** CompletableFuture, monitoring gerektirir. Execution time, completion rate monitor edilmelidir.

### Orta Seviye

#### 6. Custom async executor nasıl configure edilir?

**Cevap:**
Custom async executor, Spring'de @Async method'lar için özel thread pool configure etmek için kullanılır. Performance tuning ve resource management için önemlidir.

**Temel Konsept:** Custom async executor, AsyncConfigurer interface'i implement edilerek configure edilir. Thread pool, queue size, rejection policy ayarlanır.

**AsyncConfigurer Interface:** AsyncConfigurer interface'i, getAsyncExecutor() method'u ile custom executor sağlar. Thread pool configuration yapılır.

**ThreadPoolTaskExecutor:** ThreadPoolTaskExecutor, Spring'in async executor implementation'ıdır. Thread pool, queue, rejection policy configure edilir.

**Thread Pool Configuration:** Thread pool, core pool size, max pool size, queue capacity ayarlanır. Resource usage optimize edilir.

**Queue Configuration:** Queue configuration, task queue size, queue type belirlenir. Memory usage, task handling optimize edilir.

**Rejection Policy:** Rejection policy, queue full olduğunda ne yapılacağını belirler. AbortPolicy, CallerRunsPolicy, DiscardPolicy kullanılır.

**Thread Naming:** Thread naming, thread'lerin isimlendirilmesini sağlar. Debugging, monitoring için önemlidir.

**Keep Alive Time:** Keep alive time, idle thread'lerin ne kadar süre yaşayacağını belirler. Resource management için kullanılır.

**Best Practices:** Custom executor, workload'a göre configure edilmelidir. Monitoring, tuning yapılmalıdır.

**Use Cases:** Custom executor, high-throughput, low-latency application'lar için kullanılır. Resource optimization için önemlidir.

**Performance:** Custom executor, performance'ı optimize eder. Thread pool size, queue size ayarlanır.

**Monitoring:** Custom executor, monitoring gerektirir. Thread pool usage, queue size, rejection rate monitor edilmelidir.

#### 7. Async method'ların transaction behavior'ı nasıldır?

**Cevap:**
Async method'ların transaction behavior'ı, synchronous method'lardan farklıdır. Transaction context, async thread'e propagate edilmez.

**Temel Problem:** Async method'lar, farklı thread'de çalışır. Transaction context, thread-local storage'da saklanır. Context propagation olmaz.

**Transaction Context:** Transaction context, ThreadLocal'da saklanır. Async thread'de bu context mevcut değildir.

**@Transactional Async:** @Transactional annotation'ı, async method'larda çalışmaz. Transaction context propagate edilmez.

**Manual Transaction:** Async method'larda manual transaction management yapılmalıdır. TransactionTemplate, PlatformTransactionManager kullanılır.

**Transaction Propagation:** Transaction propagation, async method'larda çalışmaz. Parent transaction context'i async thread'de mevcut değildir.

**Isolation Level:** Isolation level, async method'larda farklı davranabilir. New transaction context oluşturulur.

**Rollback Behavior:** Rollback behavior, async method'larda farklıdır. Exception'lar, calling thread'de handle edilmez.

**Best Practices:** Async method'larda transaction management, dikkatli yapılmalıdır. Manual transaction management önerilir.

**Use Cases:** Async method'larda transaction, data consistency gerektiren durumlarda önemlidir.

**Performance:** Transaction management, async method'larda overhead yaratır. Transaction boundary'leri dikkatli define edilmelidir.

**Monitoring:** Transaction behavior, async method'larda monitor edilmelidir. Transaction success rate, rollback rate track edilmelidir.

#### 8. Async method'larda context propagation nasıl çalışır?

**Cevap:**
Async method'larda context propagation, Spring'in context management mekanizmasıdır. Security context, request context gibi context'lerin async thread'e aktarılmasını sağlar.

**Temel Konsept:** Context propagation, calling thread'deki context'in async thread'e aktarılmasıdır. Security, request, custom context'ler propagate edilir.

**Security Context:** Security context, authentication, authorization bilgilerini içerir. Async method'larda security context propagate edilir.

**Request Context:** Request context, HTTP request bilgilerini içerir. Async method'larda request context propagate edilir.

**Custom Context:** Custom context, application-specific context'lerdir. Async method'larda custom context propagate edilir.

**Context Propagation:** Context propagation, @Async method'larda otomatik olarak yapılır. Spring'in context management mekanizması kullanılır.

**Context Isolation:** Context isolation, async thread'de context'in isolated olmasını sağlar. Thread safety sağlar.

**Context Cleanup:** Context cleanup, async method tamamlandığında context'in temizlenmesini sağlar. Memory leak'leri önler.

**Best Practices:** Context propagation, security, performance açısından dikkatli yapılmalıdır. Gereksiz context propagation önlenmelidir.

**Use Cases:** Context propagation, security, logging, tracing için önemlidir. Async method'larda context bilgisi gerektiren durumlarda kullanılır.

**Performance:** Context propagation, performance overhead yaratır. Context size, propagation cost optimize edilmelidir.

**Monitoring:** Context propagation, monitoring gerektirir. Context size, propagation time monitor edilmelidir.

#### 9. Async method'ların monitoring'i nasıl yapılır?

**Cevap:**
Async method'ların monitoring'i, performance, error rate, resource usage'ı izlemek için kritiktir. Comprehensive monitoring, proactive issue detection sağlar.

**Temel Metrik'ler:** Async method monitoring, execution time, success rate, error rate, thread pool usage izler. Performance, reliability track edilir.

**Execution Time:** Execution time, async method'ların ne kadar sürede tamamlandığını gösterir. Performance bottleneck'leri gösterir.

**Success Rate:** Success rate, async method'ların başarı oranını gösterir. Error rate'in tersidir. Reliability gösterir.

**Error Rate:** Error rate, async method'ların hata oranını gösterir. Exception rate, failure rate track edilir.

**Thread Pool Usage:** Thread pool usage, thread pool'un ne kadar kullanıldığını gösterir. Resource utilization gösterir.

**Queue Size:** Queue size, task queue'unun ne kadar dolu olduğunu gösterir. Backlog, bottleneck gösterir.

**JMX Metrics:** JMX metrics, async method'ların runtime metrik'lerini expose eder. Monitoring tool'ları ile integrate edilir.

**Custom Metrics:** Custom metrics, application-specific metrik'lerdir. Business logic, performance metrik'leri track edilir.

**Logging:** Logging, async method'ların execution log'larını sağlar. Debugging, troubleshooting için kullanılır.

**Best Practices:** Async method monitoring, comprehensive olmalıdır. Proactive monitoring, reactive monitoring birlikte kullanılmalıdır.

**Use Cases:** Monitoring, performance optimization, issue detection, capacity planning için kullanılır. Production environment'lerde kritiktir.

**Tools:** Monitoring, Prometheus, Grafana, Micrometer gibi tool'lar ile yapılır. Custom dashboard'lar oluşturulur.

**Alerting:** Monitoring, alerting gerektirir. Threshold'lar set edilir. Critical issue'lar için notification sağlanır.

#### 10. Reactive programming ile async programming arasındaki fark nedir?

**Cevap:**
Reactive programming ile async programming, farklı programming paradigm'larıdır. Her ikisi de non-blocking operation'lar sağlar ancak farklı yaklaşımlar kullanır.

**Temel Fark:** Reactive programming, data stream'leri handle eder. Async programming, individual operation'ları handle eder.

**Reactive Programming:** Reactive programming, data stream'leri, backpressure, flow control sağlar. RxJava, Project Reactor kullanır.

**Async Programming:** Async programming, individual operation'ları non-blocking olarak handle eder. CompletableFuture, @Async kullanır.

**Data Flow:** Reactive programming, continuous data flow sağlar. Async programming, discrete operation'lar sağlar.

**Backpressure:** Reactive programming, backpressure sağlar. Async programming, backpressure sağlamaz.

**Flow Control:** Reactive programming, flow control sağlar. Async programming, flow control sağlamaz.

**Composition:** Reactive programming, stream composition sağlar. Async programming, operation composition sağlar.

**Error Handling:** Reactive programming, stream-level error handling sağlar. Async programming, operation-level error handling sağlar.

**Use Cases:** Reactive programming, real-time data processing için kullanılır. Async programming, I/O operation'ları için kullanılır.

**Performance:** Reactive programming, memory efficiency sağlar. Async programming, CPU efficiency sağlar.

**Best Practices:** Reactive programming, stream processing için kullanılmalıdır. Async programming, I/O operation'ları için kullanılmalıdır.

**Learning Curve:** Reactive programming, daha steep learning curve'a sahiptir. Async programming, daha kolay öğrenilir.

**Monitoring:** Reactive programming, stream monitoring gerektirir. Async programming, operation monitoring gerektirir.

---

## Kafka vs RabbitMQ

### Temel Seviye

#### 1. Kafka ve RabbitMQ'nun temel farkları nelerdir?

**Cevap:**
Kafka ve RabbitMQ, farklı messaging paradigm'ları kullanan iki farklı messaging system'dir. Her ikisi de farklı use case'ler için optimize edilmiştir.

**Temel Farklar:** Kafka ve RabbitMQ arasındaki temel farklar, architecture, data model, use case'ler açısından farklılık gösterir.

**Architecture:** Kafka, distributed streaming platform'dur. RabbitMQ, traditional message broker'dır. Kafka, horizontal scaling sağlar.

**Data Model:** Kafka, log-based data model kullanır. RabbitMQ, queue-based data model kullanır. Kafka, append-only log sağlar.

**Throughput:** Kafka, very high throughput sağlar. RabbitMQ, moderate throughput sağlar. Kafka, millions of messages per second handle edebilir.

**Latency:** Kafka, low latency sağlar. RabbitMQ, variable latency sağlar. Kafka, sub-millisecond latency sağlar.

**Durability:** Kafka, data durability sağlar. RabbitMQ, configurable durability sağlar. Kafka, disk-based storage kullanır.

**Message Ordering:** Kafka, partition seviyesinde ordering sağlar. RabbitMQ, queue seviyesinde ordering sağlar. Kafka, partition-based ordering sağlar.

**Consumer Model:** Kafka, pull-based consumer model kullanır. RabbitMQ, push-based model kullanır. Kafka, consumer control sağlar.

**Routing:** Kafka, simple routing sağlar. RabbitMQ, complex routing sağlar. RabbitMQ, exchange, binding kullanır.

**Use Cases:** Kafka, event streaming, log aggregation için kullanılır. RabbitMQ, request-response, RPC için kullanılır.

**Best Practices:** Kafka, high-throughput, low-latency use case'ler için kullanılmalıdır. RabbitMQ, complex routing, request-response için kullanılmalıdır.

**Performance:** Kafka, high-throughput, low-latency sağlar. RabbitMQ, moderate performance sağlar. Kafka, better performance sağlar.

**Complexity:** Kafka, moderate complexity sağlar. RabbitMQ, higher complexity sağlar. Kafka, simpler architecture sağlar.

#### 2. Message broker vs Event streaming platform farkı nedir?

**Cevap:**
Message broker ve Event streaming platform, farklı messaging paradigm'larıdır. Her ikisi de farklı use case'ler için tasarlanmıştır.

**Temel Fark:** Message broker, point-to-point messaging sağlar. Event streaming platform, publish-subscribe messaging sağlar.

**Message Broker:** Message broker, traditional messaging sağlar. Queue-based, request-response pattern'ları destekler. RabbitMQ örnektir.

**Event Streaming Platform:** Event streaming platform, event-driven architecture sağlar. Log-based, stream processing destekler. Kafka örnektir.

**Data Model:** Message broker, queue-based data model kullanır. Event streaming platform, log-based data model kullanır.

**Message Lifecycle:** Message broker, message'ları consume edildikten sonra siler. Event streaming platform, message'ları persistent store eder.

**Consumer Model:** Message broker, push-based consumer model kullanır. Event streaming platform, pull-based consumer model kullanır.

**Throughput:** Message broker, moderate throughput sağlar. Event streaming platform, very high throughput sağlar.

**Latency:** Message broker, variable latency sağlar. Event streaming platform, low latency sağlar.

**Durability:** Message broker, configurable durability sağlar. Event streaming platform, high durability sağlar.

**Use Cases:** Message broker, request-response, RPC için kullanılır. Event streaming platform, event streaming, log aggregation için kullanılır.

**Best Practices:** Message broker, traditional messaging için kullanılmalıdır. Event streaming platform, event-driven architecture için kullanılmalıdır.

**Performance:** Message broker, moderate performance sağlar. Event streaming platform, high performance sağlar.

**Complexity:** Message broker, higher complexity sağlar. Event streaming platform, moderate complexity sağlar.

#### 3. Kafka'nın throughput avantajı nereden gelir?

**Cevap:**
Kafka'nın throughput avantajı, architecture design'ından ve data model'inden gelir. Distributed, log-based architecture, high throughput sağlar.

**Temel Avantajlar:** Kafka'nın throughput avantajı, distributed architecture, log-based storage, batch processing, zero-copy optimization'dan gelir.

**Distributed Architecture:** Kafka, distributed architecture kullanır. Multiple broker'lar, parallel processing sağlar. Horizontal scaling sağlar.

**Log-based Storage:** Kafka, log-based storage kullanır. Append-only log, sequential write sağlar. Disk I/O optimize edilir.

**Batch Processing:** Kafka, batch processing kullanır. Multiple message'lar batch'ler halinde işlenir. Network overhead azalır.

**Zero-copy Optimization:** Kafka, zero-copy optimization kullanır. Data copy olmadan transfer edilir. CPU usage azalır.

**Partitioning:** Kafka, partitioning kullanır. Parallel processing sağlar. Load distribution optimize edilir.

**Compression:** Kafka, compression kullanır. Data size azalır. Network bandwidth optimize edilir.

**Memory Mapping:** Kafka, memory mapping kullanır. Disk I/O optimize edilir. Performance artar.

**Best Practices:** Kafka'nın throughput avantajı, proper configuration ile maximize edilir. Partition count, batch size optimize edilir.

**Use Cases:** Kafka'nın throughput avantajı, high-volume data processing için kullanılır. Event streaming, log aggregation için uygundur.

**Performance:** Kafka'nın throughput avantajı, millions of messages per second handle edebilir. Traditional messaging system'lerden çok daha yüksektir.

**Monitoring:** Kafka'nın throughput avantajı, monitoring ile track edilir. Throughput, latency metrik'leri izlenir.

#### 4. RabbitMQ'nun routing flexibility'si nasıldır?

**Cevap:**
RabbitMQ'nun routing flexibility'si, exchange, binding, routing key mekanizmalarından gelir. Complex routing pattern'ları destekler.

**Temel Mekanizmalar:** RabbitMQ'nun routing flexibility'si, exchange types, binding rules, routing keys'den gelir. Flexible message routing sağlar.

**Exchange Types:** RabbitMQ, farklı exchange type'ları sağlar. Direct, topic, fanout, headers exchange'ler kullanılır.

**Direct Exchange:** Direct exchange, exact routing key match kullanır. Simple routing sağlar. Point-to-point messaging için uygundur.

**Topic Exchange:** Topic exchange, pattern-based routing kullanır. Wildcard pattern'lar kullanılır. Complex routing sağlar.

**Fanout Exchange:** Fanout exchange, broadcast routing kullanır. Tüm queue'lara message gönderir. Publish-subscribe pattern için uygundur.

**Headers Exchange:** Headers exchange, header-based routing kullanır. Message header'larına göre routing yapar. Complex routing sağlar.

**Binding Rules:** RabbitMQ, binding rules kullanır. Exchange'ler queue'lara bind edilir. Routing logic define edilir.

**Routing Keys:** RabbitMQ, routing keys kullanır. Message'lar routing key'e göre route edilir. Flexible routing sağlar.

**Best Practices:** RabbitMQ'nun routing flexibility'si, proper exchange design ile maximize edilir. Exchange type'ları use case'e göre seçilir.

**Use Cases:** RabbitMQ'nun routing flexibility'si, complex messaging pattern'ları için kullanılır. Workflow, orchestration için uygundur.

**Performance:** RabbitMQ'nun routing flexibility'si, performance overhead yaratır. Complex routing, CPU usage artırır.

**Monitoring:** RabbitMQ'nun routing flexibility'si, monitoring ile track edilir. Routing performance, queue depth izlenir.

#### 5. Hangisi daha kolay setup edilir?

**Cevap:**
Setup kolaylığı açısından, RabbitMQ genellikle Kafka'dan daha kolay setup edilir. Ancak bu, use case'e ve requirement'lara bağlıdır.

**Setup Kolaylığı:** Setup kolaylığı, installation, configuration, operation complexity açısından değerlendirilir.

**RabbitMQ Setup:** RabbitMQ, daha kolay setup edilir. Single node installation, simple configuration sağlar. Docker, package manager ile kolayca install edilir.

**Kafka Setup:** Kafka, daha complex setup gerektirir. Cluster setup, Zookeeper dependency, configuration complexity sağlar. Multiple node'lar gerekir.

**Installation:** RabbitMQ, single package ile install edilir. Kafka, multiple component'ler gerekir. Zookeeper, Kafka broker'ları gerekir.

**Configuration:** RabbitMQ, simple configuration sağlar. Kafka, complex configuration gerektirir. Multiple configuration file'ları gerekir.

**Operation:** RabbitMQ, simple operation sağlar. Kafka, complex operation gerektirir. Cluster management, partition management gerekir.

**Best Practices:** Setup kolaylığı, use case'e göre değerlendirilmelidir. Simple use case'ler için RabbitMQ, complex use case'ler için Kafka kullanılmalıdır.

**Use Cases:** Setup kolaylığı, development, testing için önemlidir. Production environment'lerde complexity kabul edilebilir.

**Performance:** Setup kolaylığı, performance'ı etkilemez. Her iki system de high performance sağlar.

**Monitoring:** Setup kolaylığı, monitoring complexity'yi etkiler. Kafka, daha complex monitoring gerektirir.

**Maintenance:** Setup kolaylığı, maintenance complexity'yi etkiler. Kafka, daha complex maintenance gerektirir.

**Learning Curve:** Setup kolaylığı, learning curve'u etkiler. RabbitMQ, daha kolay öğrenilir. Kafka, daha steep learning curve'a sahiptir.

### Orta Seviye

#### 6. Kafka ve RabbitMQ'nun use case'leri nelerdir?

**Cevap:**
Kafka ve RabbitMQ, farklı use case'ler için optimize edilmiştir. Her ikisi de farklı messaging pattern'ları ve requirement'ları destekler.

**Kafka Use Cases:** Kafka, high-throughput, low-latency, event streaming use case'leri için optimize edilmiştir.

**Event Streaming:** Kafka, event streaming için kullanılır. Real-time event processing, event sourcing, CQRS pattern'ları destekler.

**Log Aggregation:** Kafka, log aggregation için kullanılır. Application log'ları, system log'ları centralize eder. Log analysis sağlar.

**Real-time Analytics:** Kafka, real-time analytics için kullanılır. Stream processing, data pipeline'lar oluşturur. Real-time dashboard'lar sağlar.

**Data Integration:** Kafka, data integration için kullanılır. ETL pipeline'lar, data synchronization sağlar. Data lake, data warehouse integration.

**IoT Data Processing:** Kafka, IoT data processing için kullanılır. Sensor data, device data handle eder. High-volume data processing sağlar.

**RabbitMQ Use Cases:** RabbitMQ, complex routing, request-response, workflow use case'leri için optimize edilmiştir.

**Request-Response:** RabbitMQ, request-response pattern'ları için kullanılır. RPC, API gateway, service communication sağlar.

**Workflow Orchestration:** RabbitMQ, workflow orchestration için kullanılır. Business process, workflow management sağlar.

**Task Queue:** RabbitMQ, task queue için kullanılır. Background job processing, task distribution sağlar.

**Message Routing:** RabbitMQ, complex message routing için kullanılır. Exchange, binding, routing key kullanır. Flexible routing sağlar.

**Best Practices:** Use case seçimi, requirement'lara göre yapılmalıdır. Kafka, event streaming için kullanılmalıdır. RabbitMQ, complex routing için kullanılmalıdır.

**Performance:** Use case seçimi, performance'ı etkiler. Kafka, high-throughput için optimize edilmiştir. RabbitMQ, complex routing için optimize edilmiştir.

**Monitoring:** Use case seçimi, monitoring complexity'yi etkiler. Kafka, stream monitoring gerektirir. RabbitMQ, queue monitoring gerektirir.

#### 7. Message ordering garantileri nasıldır?

**Cevap:**
Message ordering garantileri, Kafka ve RabbitMQ'da farklı şekillerde sağlanır. Her ikisi de farklı ordering guarantee'leri sunar.

**Kafka Ordering:** Kafka, partition seviyesinde message ordering sağlar. Global ordering sağlamaz, partition-based ordering sağlar.

**Partition-based Ordering:** Kafka, partition içinde message'lar ordered'dır. Aynı partition'daki message'lar, gönderilme sırasında işlenir.

**Key-based Ordering:** Kafka, key-based ordering sağlar. Aynı key'e sahip message'lar, aynı partition'a gider. Key seviyesinde ordering sağlar.

**Global Ordering:** Kafka, global ordering sağlamaz. Tüm message'lar için global ordering yoktur. Partition seviyesinde ordering vardır.

**RabbitMQ Ordering:** RabbitMQ, queue seviyesinde message ordering sağlar. Global ordering sağlamaz, queue-based ordering sağlar.

**Queue-based Ordering:** RabbitMQ, queue içinde message'lar ordered'dır. Aynı queue'daki message'lar, gönderilme sırasında işlenir.

**Exchange Ordering:** RabbitMQ, exchange seviyesinde ordering sağlamaz. Exchange'ler, message'ları queue'lara dağıtır. Ordering queue seviyesinde sağlanır.

**Consumer Ordering:** RabbitMQ, consumer seviyesinde ordering sağlar. Aynı consumer, message'ları sırayla işler.

**Best Practices:** Message ordering, use case'e göre değerlendirilmelidir. Kafka, partition-based ordering için kullanılmalıdır. RabbitMQ, queue-based ordering için kullanılmalıdır.

**Use Cases:** Message ordering, critical application'lar için önemlidir. Financial system'ler, order processing için kritiktir.

**Performance:** Message ordering, performance'ı etkiler. Strict ordering, throughput'u azaltır. Loose ordering, throughput'u artırır.

**Monitoring:** Message ordering, monitoring gerektirir. Ordering violation'lar, performance impact monitor edilmelidir.

#### 8. Durability ve persistence farkları nelerdir?

**Cevap:**
Durability ve persistence, Kafka ve RabbitMQ'da farklı şekillerde implement edilir. Her ikisi de farklı durability guarantee'leri sunar.

**Kafka Durability:** Kafka, high durability sağlar. Disk-based storage, replication kullanır. Data loss'u önler.

**Disk-based Storage:** Kafka, disk'te data store eder. Memory'de sadece cache eder. Persistent storage sağlar.

**Replication:** Kafka, replication kullanır. Multiple broker'larda data replicate edilir. Fault tolerance sağlar.

**Acknowledgment:** Kafka, acknowledgment mechanism kullanır. Producer'lar, write acknowledgment alır. Data delivery guarantee sağlar.

**RabbitMQ Durability:** RabbitMQ, configurable durability sağlar. Memory-based, disk-based storage seçenekleri sunar.

**Configurable Durability:** RabbitMQ, durability configurable'dır. Queue, message durability ayarlanabilir. Memory, disk storage seçilebilir.

**Queue Durability:** RabbitMQ, queue durability sağlar. Queue'lar disk'te store edilebilir. Broker restart'ta queue'lar korunur.

**Message Durability:** RabbitMQ, message durability sağlar. Message'lar disk'te store edilebilir. Broker restart'ta message'lar korunur.

**Best Practices:** Durability, use case'e göre ayarlanmalıdır. Critical data için high durability, non-critical data için low durability kullanılmalıdır.

**Use Cases:** Durability, data loss'u önlemek için önemlidir. Financial system'ler, critical application'lar için kritiktir.

**Performance:** Durability, performance'ı etkiler. High durability, write latency artırır. Low durability, write latency azaltır.

**Monitoring:** Durability, monitoring gerektirir. Data loss rate, durability compliance monitor edilmelidir.

#### 9. Scaling stratejileri nasıldır?

**Cevap:**
Scaling stratejileri, Kafka ve RabbitMQ'da farklı şekillerde implement edilir. Her ikisi de farklı scaling approach'ları kullanır.

**Kafka Scaling:** Kafka, horizontal scaling sağlar. Distributed architecture, partition-based scaling kullanır.

**Horizontal Scaling:** Kafka, horizontal scaling sağlar. Yeni broker'lar eklenerek cluster expand edilir. Linear scaling sağlar.

**Partition-based Scaling:** Kafka, partition-based scaling kullanır. Partition'lar, farklı broker'larda store edilir. Parallel processing sağlar.

**Consumer Scaling:** Kafka, consumer scaling sağlar. Consumer group'lar, partition'ları distribute eder. Load balancing sağlar.

**RabbitMQ Scaling:** RabbitMQ, vertical scaling sağlar. Single node performance, resource optimization kullanır.

**Vertical Scaling:** RabbitMQ, vertical scaling sağlar. CPU, memory, disk resource'ları artırılır. Single node performance optimize edilir.

**Cluster Scaling:** RabbitMQ, cluster scaling sağlar. Multiple node'lar, queue'ları distribute eder. Load distribution sağlar.

**Queue Scaling:** RabbitMQ, queue scaling sağlar. Queue'lar, farklı node'larda store edilir. Parallel processing sağlar.

**Best Practices:** Scaling stratejisi, use case'e göre seçilmelidir. Kafka, horizontal scaling için kullanılmalıdır. RabbitMQ, vertical scaling için kullanılmalıdır.

**Use Cases:** Scaling, high-volume data processing için önemlidir. Event streaming, real-time analytics için kritiktir.

**Performance:** Scaling stratejisi, performance'ı etkiler. Horizontal scaling, throughput artırır. Vertical scaling, latency azaltır.

**Monitoring:** Scaling, monitoring gerektirir. Resource usage, performance metrik'leri izlenmelidir.

#### 10. Monitoring ve operational complexity nasıldır?

**Cevap:**
Monitoring ve operational complexity, Kafka ve RabbitMQ'da farklı seviyelerdedir. Her ikisi de farklı monitoring requirement'ları ve operational complexity'ye sahiptir.

**Kafka Monitoring:** Kafka, moderate monitoring complexity sağlar. Distributed system monitoring, partition monitoring gerektirir.

**Distributed Monitoring:** Kafka, distributed system monitoring gerektirir. Multiple broker'lar, partition'lar monitor edilir. Cluster health track edilir.

**Partition Monitoring:** Kafka, partition monitoring gerektirir. Partition count, replication factor, ISR size monitor edilir.

**Consumer Monitoring:** Kafka, consumer monitoring gerektirir. Consumer lag, offset commit rate, processing rate monitor edilir.

**RabbitMQ Monitoring:** RabbitMQ, higher monitoring complexity sağlar. Queue monitoring, exchange monitoring, connection monitoring gerektirir.

**Queue Monitoring:** RabbitMQ, queue monitoring gerektirir. Queue depth, message rate, consumer count monitor edilir.

**Exchange Monitoring:** RabbitMQ, exchange monitoring gerektirir. Exchange type, binding count, routing performance monitor edilir.

**Connection Monitoring:** RabbitMQ, connection monitoring gerektirir. Connection count, connection rate, connection health monitor edilir.

**Best Practices:** Monitoring, comprehensive olmalıdır. Proactive monitoring, reactive monitoring birlikte kullanılmalıdır.

**Use Cases:** Monitoring, production environment'lerde kritiktir. Performance optimization, issue detection için önemlidir.

**Performance:** Monitoring, performance overhead yaratır. Monitoring tool'ları, system resource'larını kullanır.

**Tools:** Monitoring, Prometheus, Grafana, Kafka Manager, RabbitMQ Management UI gibi tool'lar ile yapılır.

---

## Java Concurrency and Multithreading

### Temel Seviye

#### 1. Thread ve Process arasındaki fark nedir?

**Cevap:**
Thread ve Process, Java'da concurrency sağlamak için kullanılan iki farklı execution unit'idir. Her ikisi de farklı seviyelerde isolation ve resource sharing sağlar.

**Temel Farklar:** Thread ve Process arasındaki temel farklar, memory space, resource sharing, communication, creation cost açısından farklılık gösterir.

**Process:** Process, operating system tarafından yönetilen execution unit'idir. Kendi memory space'ine sahiptir. Independent execution sağlar.

**Thread:** Thread, process içinde çalışan execution unit'idir. Process'in memory space'ini paylaşır. Lightweight execution sağlar.

**Memory Space:** Process, kendi memory space'ine sahiptir. Thread, process'in memory space'ini paylaşır. Memory isolation farklıdır.

**Resource Sharing:** Process, resource'ları paylaşmaz. Thread, process'in resource'larını paylaşır. Resource sharing farklıdır.

**Communication:** Process, IPC (Inter-Process Communication) kullanır. Thread, shared memory kullanır. Communication mechanism farklıdır.

**Creation Cost:** Process, yüksek creation cost'a sahiptir. Thread, düşük creation cost'a sahiptir. Creation overhead farklıdır.

**Context Switching:** Process, yüksek context switching cost'a sahiptir. Thread, düşük context switching cost'a sahiptir. Switching overhead farklıdır.

**Use Cases:** Process, isolation gerektiren durumlarda kullanılır. Thread, parallelism gerektiren durumlarda kullanılır.

**Best Practices:** Process, security, isolation için kullanılmalıdır. Thread, performance, parallelism için kullanılmalıdır.

**Performance:** Process, yüksek overhead'e sahiptir. Thread, düşük overhead'e sahiptir. Performance farklıdır.

**Security:** Process, yüksek security sağlar. Thread, düşük security sağlar. Security level farklıdır.

#### 2. Runnable ve Callable interface'lerinin farkı nedir?

**Cevap:**
Runnable ve Callable, Java'da thread'lerde task'ları execute etmek için kullanılan iki farklı interface'dir. Her ikisi de farklı return type'ları ve exception handling sağlar.

**Temel Farklar:** Runnable ve Callable arasındaki temel farklar, return type, exception handling, use case'ler açısından farklılık gösterir.

**Runnable Interface:** Runnable interface, void return type'a sahiptir. Exception throw edemez. Simple task execution sağlar.

**Callable Interface:** Callable interface, generic return type'a sahiptir. Exception throw edebilir. Complex task execution sağlar.

**Return Type:** Runnable, void return type'a sahiptir. Callable, generic return type'a sahiptir. Return value farklıdır.

**Exception Handling:** Runnable, exception throw edemez. Callable, exception throw edebilir. Exception handling farklıdır.

**Use Cases:** Runnable, simple task'lar için kullanılır. Callable, complex task'lar için kullanılır. Task complexity farklıdır.

**ExecutorService:** Runnable, ExecutorService'de execute() method'u ile kullanılır. Callable, submit() method'u ile kullanılır. Execution method farklıdır.

**Future:** Runnable, Future return etmez. Callable, Future return eder. Future handling farklıdır.

**Best Practices:** Runnable, simple task'lar için kullanılmalıdır. Callable, complex task'lar için kullanılmalıdır.

**Performance:** Runnable, düşük overhead'e sahiptir. Callable, yüksek overhead'e sahiptir. Performance farklıdır.

**Error Handling:** Runnable, error handling yapamaz. Callable, error handling yapabilir. Error handling farklıdır.

**Thread Creation:** Runnable, Thread constructor'ında kullanılır. Callable, ExecutorService'de kullanılır. Usage pattern farklıdır.

#### 3. Thread lifecycle'ı nasıldır?

**Cevap:**
Thread lifecycle, Java'da thread'lerin yaşam döngüsünü belirler. Thread'ler, farklı state'lerde bulunabilir ve bu state'ler arasında geçiş yapabilir.

**Temel State'ler:** Thread lifecycle, NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED state'lerinden oluşur.

**NEW State:** NEW state, thread'in oluşturulduğu ancak henüz start() method'u çağrılmadığı state'dir. Thread henüz çalışmaya başlamamıştır.

**RUNNABLE State:** RUNNABLE state, thread'in çalışmaya hazır olduğu state'dir. Thread, CPU time bekliyor veya çalışıyor olabilir.

**BLOCKED State:** BLOCKED state, thread'in synchronized block veya method için beklediği state'dir. Thread, lock bekliyor.

**WAITING State:** WAITING state, thread'in wait() method'u ile beklediği state'dir. Thread, notify() veya notifyAll() bekliyor.

**TIMED_WAITING State:** TIMED_WAITING state, thread'in sleep() veya wait(timeout) method'u ile beklediği state'dir. Thread, timeout bekliyor.

**TERMINATED State:** TERMINATED state, thread'in çalışmasının tamamlandığı state'dir. Thread, run() method'unu tamamlamıştır.

**State Transitions:** Thread'ler, state'ler arasında geçiş yapabilir. State transition'lar, method call'ları ile tetiklenir.

**Best Practices:** Thread lifecycle, proper thread management için önemlidir. State'ler monitor edilmelidir.

**Use Cases:** Thread lifecycle, thread management, debugging için kullanılır. Thread state'leri track edilir.

**Performance:** Thread lifecycle, performance'ı etkiler. State transition'lar overhead yaratır.

**Monitoring:** Thread lifecycle, monitoring gerektirir. Thread state'leri, execution time monitor edilmelidir.

#### 4. synchronized keyword'ünün kullanımı nasıldır?

**Cevap:**
synchronized keyword'ü, Java'da thread safety sağlamak için kullanılır. Critical section'ları protect eder ve race condition'ları önler.

**Temel Kullanım:** synchronized keyword'ü, method'lar, block'lar, static method'lar için kullanılır. Thread safety sağlar.

**Method Synchronization:** synchronized method'lar, tüm method'u synchronize eder. Method'a aynı anda sadece bir thread erişebilir.

**Block Synchronization:** synchronized block'lar, belirli kod bloklarını synchronize eder. Fine-grained synchronization sağlar.

**Static Synchronization:** static synchronized method'lar, class level'da synchronization sağlar. Tüm instance'lar için geçerlidir.

**Lock Object:** synchronized keyword'ü, implicit lock sağlar. Object'in monitor'ını kullanır. Lock acquisition, release otomatiktir.

**Reentrant Lock:** synchronized keyword'ü, reentrant lock sağlar. Aynı thread, aynı lock'ı multiple kez acquire edebilir.

**Best Practices:** synchronized keyword'ü, minimal scope'ta kullanılmalıdır. Performance overhead'i minimize edilmelidir.

**Use Cases:** synchronized keyword'ü, shared resource access, critical section protection için kullanılır.

**Performance:** synchronized keyword'ü, performance overhead yaratır. Lock contention, performance'ı etkiler.

**Alternatives:** synchronized keyword'ü yerine, ReentrantLock, ReadWriteLock gibi alternatifler kullanılabilir.

**Monitoring:** synchronized keyword'ü, monitoring gerektirir. Lock contention, deadlock monitor edilmelidir.

#### 5. wait(), notify() ve notifyAll() method'larının işlevi nedir?

**Cevap:**
wait(), notify() ve notifyAll() method'ları, Java'da thread coordination için kullanılır. Thread'ler arası communication sağlar.

**Temel İşlevler:** wait(), notify() ve notifyAll() method'ları, thread'lerin coordination'ını sağlar. Producer-consumer pattern'ları implement eder.

**wait() Method:** wait() method'u, thread'i wait state'ine geçirir. Thread, notify() veya notifyAll() bekler. Lock'ı release eder.

**notify() Method:** notify() method'u, wait state'inde olan bir thread'i uyandırır. Random thread seçilir. Lock acquisition gerekir.

**notifyAll() Method:** notifyAll() method'u, wait state'inde olan tüm thread'leri uyandırır. Tüm thread'ler compete eder. Lock acquisition gerekir.

**Synchronization:** wait(), notify(), notifyAll() method'ları, synchronized context'te kullanılmalıdır. IllegalMonitorStateException fırlatır.

**Lock Release:** wait() method'u, lock'ı release eder. notify() method'u, lock'ı release etmez. Lock management önemlidir.

**Spurious Wakeup:** wait() method'u, spurious wakeup yaşayabilir. Condition check yapılmalıdır. While loop kullanılmalıdır.

**Best Practices:** wait(), notify(), notifyAll() method'ları, proper synchronization ile kullanılmalıdır. Condition check yapılmalıdır.

**Use Cases:** wait(), notify(), notifyAll() method'ları, producer-consumer, reader-writer pattern'ları için kullanılır.

**Performance:** wait(), notify(), notifyAll() method'ları, performance overhead yaratır. Context switching, lock contention etkiler.

**Alternatives:** wait(), notify(), notifyAll() method'ları yerine, Condition, CountDownLatch gibi alternatifler kullanılabilir.

**Monitoring:** wait(), notify(), notifyAll() method'ları, monitoring gerektirir. Thread state, lock contention monitor edilmelidir.

### Orta Seviye

#### 6. ExecutorService'in kullanım alanları nelerdir?

**Cevap:**
ExecutorService, Java'da thread management ve task execution için kullanılan interface'dir. Thread pool'ları manage eder ve task'ları execute eder.

**Temel Kullanım:** ExecutorService, thread pool'ları manage eder. Task'ları submit eder, execute eder, shutdown eder. Thread lifecycle management sağlar.

**Task Submission:** ExecutorService, task'ları submit eder. execute(), submit(), invokeAll(), invokeAny() method'ları kullanılır.

**Thread Pool Management:** ExecutorService, thread pool'ları manage eder. Thread creation, destruction, reuse sağlar. Resource management optimize eder.

**Future Handling:** ExecutorService, Future object'leri return eder. Task result'ları, exception'ları handle eder. Asynchronous execution sağlar.

**Shutdown Management:** ExecutorService, shutdown management sağlar. shutdown(), shutdownNow(), awaitTermination() method'ları kullanılır.

**Best Practices:** ExecutorService, proper shutdown ile kullanılmalıdır. Resource leak'leri önlenmelidir.

**Use Cases:** ExecutorService, parallel processing, background task execution, async operation'lar için kullanılır.

**Performance:** ExecutorService, performance'ı optimize eder. Thread reuse, resource management sağlar.

**Monitoring:** ExecutorService, monitoring gerektirir. Thread pool usage, task execution rate monitor edilmelidir.

#### 7. Future ve CompletableFuture arasındaki fark nedir?

**Cevap:**
Future ve CompletableFuture, Java'da asynchronous operation'ları handle etmek için kullanılan interface'lerdir. Her ikisi de farklı functionality ve use case'ler sunar.

**Temel Farklar:** Future ve CompletableFuture arasındaki temel farklar, functionality, composition, exception handling açısından farklılık gösterir.

**Future Interface:** Future interface, basic asynchronous operation sağlar. get(), isDone(), cancel() method'ları kullanılır. Simple async handling sağlar.

**CompletableFuture Class:** CompletableFuture class, advanced asynchronous operation sağlar. Method chaining, composition, exception handling sağlar.

**Functionality:** Future, basic functionality sağlar. CompletableFuture, advanced functionality sağlar. Feature set farklıdır.

**Composition:** Future, composition sağlamaz. CompletableFuture, method chaining sağlar. thenApply(), thenCompose() gibi method'lar kullanılır.

**Exception Handling:** Future, basic exception handling sağlar. CompletableFuture, advanced exception handling sağlar. exceptionally(), handle() method'ları kullanılır.

**Use Cases:** Future, simple async operation'lar için kullanılır. CompletableFuture, complex async logic için kullanılır.

**Best Practices:** Future, simple use case'ler için kullanılmalıdır. CompletableFuture, complex use case'ler için kullanılmalıdır.

**Performance:** Future, düşük overhead'e sahiptir. CompletableFuture, yüksek overhead'e sahiptir. Performance farklıdır.

**Monitoring:** Future, basic monitoring sağlar. CompletableFuture, advanced monitoring sağlar. Monitoring capability farklıdır.

#### 8. Thread pool'ların türleri nelerdir?

**Cevap:**
Thread pool'lar, Java'da thread management için kullanılan farklı türlerde pool'larıdır. Her tür, farklı use case'ler için optimize edilmiştir.

**Temel Türler:** Thread pool'lar, FixedThreadPool, CachedThreadPool, SingleThreadPool, ScheduledThreadPool türlerinden oluşur.

**FixedThreadPool:** FixedThreadPool, fixed size thread pool sağlar. Core pool size, max pool size aynıdır. Stable thread count sağlar.

**CachedThreadPool:** CachedThreadPool, dynamic size thread pool sağlar. Thread'ler create edilir, idle thread'ler terminate edilir. Flexible thread count sağlar.

**SingleThreadPool:** SingleThreadPool, single thread pool sağlar. Sadece bir thread çalışır. Sequential execution sağlar.

**ScheduledThreadPool:** ScheduledThreadPool, scheduled task execution sağlar. Delayed execution, periodic execution sağlar. Timer functionality sağlar.

**Use Cases:** Thread pool türleri, farklı use case'ler için kullanılır. Workload'a göre seçilmelidir.

**Best Practices:** Thread pool türleri, use case'e göre seçilmelidir. Performance, resource usage optimize edilmelidir.

**Performance:** Thread pool türleri, farklı performance karakteristikleri sağlar. Workload'a göre optimize edilir.

**Monitoring:** Thread pool türleri, monitoring gerektirir. Thread count, task execution rate monitor edilmelidir.

#### 9. Deadlock nedir ve nasıl önlenir?

**Cevap:**
Deadlock, Java'da thread'lerin birbirini beklediği durumdur. Thread'ler, resource'ları acquire etmek için birbirini bekler ve hiçbiri ilerleyemez.

**Temel Konsept:** Deadlock, circular wait condition'dır. Thread'ler, resource'ları circular order'da acquire etmeye çalışır. System deadlock'a girer.

**Deadlock Conditions:** Deadlock, dört condition gerektirir: mutual exclusion, hold and wait, no preemption, circular wait.

**Mutual Exclusion:** Mutual exclusion, resource'ların aynı anda sadece bir thread tarafından kullanılabilmesidir. Shared resource'lar exclusive'dir.

**Hold and Wait:** Hold and Wait, thread'lerin resource'ları hold ederken başka resource'ları beklemeleridir. Resource'ları release etmez.

**No Preemption:** No Preemption, resource'ların thread'lerden zorla alınamamasıdır. Resource'lar sadece thread tarafından release edilir.

**Circular Wait:** Circular Wait, thread'lerin circular order'da resource'ları beklemeleridir. T1→T2→T3→T1 şeklinde wait cycle oluşur.

**Deadlock Prevention:** Deadlock prevention, deadlock condition'larından birini kaldırarak deadlock'u önler. Resource ordering, timeout kullanılır.

**Resource Ordering:** Resource ordering, resource'ları consistent order'da acquire etmeyi sağlar. Circular wait condition'ını kaldırır.

**Timeout:** Timeout, resource acquisition'a timeout ekler. Deadlock durumunda thread'ler timeout ile recover eder.

**Best Practices:** Deadlock prevention, proper resource ordering ile yapılmalıdır. Timeout, monitoring kullanılmalıdır.

**Use Cases:** Deadlock prevention, critical system'ler için önemlidir. Financial system'ler, real-time system'ler için kritiktir.

**Performance:** Deadlock prevention, performance overhead yaratır. Resource ordering, timeout overhead'i vardır.

**Monitoring:** Deadlock prevention, monitoring gerektirir. Deadlock detection, prevention rate monitor edilmelidir.

#### 10. Race condition'lar nasıl çözülür?

**Cevap:**
Race condition'lar, Java'da thread'lerin shared resource'lara aynı anda erişmesi durumunda oluşur. Thread safety mechanism'ları ile çözülür.

**Temel Konsept:** Race condition, thread'lerin shared resource'lara aynı anda erişmesi durumunda oluşur. Data inconsistency, unpredictable behavior yaratır.

**Race Condition Causes:** Race condition, shared resource access, non-atomic operation'lar, lack of synchronization'dan oluşur.

**Shared Resource Access:** Shared resource access, multiple thread'lerin aynı resource'a erişmesi durumunda oluşur. Data corruption yaratır.

**Non-atomic Operations:** Non-atomic operation'lar, multiple step operation'ların atomic olmaması durumunda oluşur. Intermediate state'ler görülebilir.

**Lack of Synchronization:** Lack of synchronization, thread'lerin coordinate olmaması durumunda oluşur. Concurrent access control yoktur.

**Race Condition Solutions:** Race condition'lar, synchronization, atomic operation'lar, immutable object'ler ile çözülür.

**Synchronization:** Synchronization, synchronized keyword, ReentrantLock kullanır. Critical section'ları protect eder. Thread safety sağlar.

**Atomic Operations:** Atomic operation'lar, AtomicInteger, AtomicReference kullanır. Lock-free thread safety sağlar. Performance optimize eder.

**Immutable Objects:** Immutable object'ler, state değişmezliği sağlar. Thread safety sağlar. Race condition'ları önler.

**Best Practices:** Race condition'lar, proper synchronization ile çözülmelidir. Atomic operation'lar, immutable object'ler kullanılmalıdır.

**Use Cases:** Race condition'lar, shared resource access gerektiren durumlarda önemlidir. Counter, cache, shared state için kritiktir.

**Performance:** Race condition çözümleri, performance overhead yaratır. Synchronization, atomic operation overhead'i vardır.

**Monitoring:** Race condition'lar, monitoring gerektirir. Data consistency, thread safety monitor edilmelidir.

#### 11. Atomic operations'ın faydaları nelerdir?

**Cevap:**
Atomic operations, Java'da lock-free thread safety sağlar. Performance optimize eder ve deadlock'ları önler.

**Temel Faydalar:** Atomic operations, lock-free thread safety, performance optimization, deadlock prevention sağlar.

**Lock-free Thread Safety:** Atomic operations, lock kullanmadan thread safety sağlar. CAS (Compare-And-Swap) operation'ları kullanır. Lock contention yoktur.

**Performance Optimization:** Atomic operations, synchronization'dan daha performanslıdır. Lock overhead'i yoktur. CPU cycle'ları optimize eder.

**Deadlock Prevention:** Atomic operations, deadlock'ları önler. Lock kullanmadığı için deadlock risk'i yoktur. System stability sağlar.

**Memory Visibility:** Atomic operations, memory visibility sağlar. Thread'ler arası data consistency sağlar. Volatile semantics sağlar.

**Use Cases:** Atomic operations, counter, flag, reference update için kullanılır. High-performance concurrent programming için uygundur.

**Best Practices:** Atomic operations, simple operation'lar için kullanılmalıdır. Complex logic için synchronization tercih edilmelidir.

**Performance:** Atomic operations, high performance sağlar. Lock-free implementation, CPU efficiency artırır.

**Monitoring:** Atomic operations, monitoring gerektirir. Operation success rate, contention monitor edilmelidir.

#### 12. Fork/Join framework'ün kullanımı nasıldır?

**Cevap:**
Fork/Join framework, Java'da parallel processing için kullanılan framework'tür. Divide-and-conquer algorithm'ları implement eder.

**Temel Konsept:** Fork/Join framework, task'ları divide eder, parallel execute eder, join eder. Recursive task processing sağlar.

**Fork Operation:** Fork operation, task'ı smaller task'lara böler. Recursive decomposition sağlar. Parallel execution enable eder.

**Join Operation:** Join operation, task result'larını combine eder. Result aggregation sağlar. Final result oluşturur.

**Work Stealing:** Work stealing, idle thread'lerin busy thread'lerden task almasıdır. Load balancing sağlar. CPU utilization optimize eder.

**RecursiveTask:** RecursiveTask, result return eden recursive task'lar için kullanılır. compute() method'u implement edilir.

**RecursiveAction:** RecursiveAction, result return etmeyen recursive task'lar için kullanılır. compute() method'u implement edilir.

**ForkJoinPool:** ForkJoinPool, Fork/Join task'ları execute eder. Work stealing algorithm kullanır. CPU core count'a göre thread'ler oluşturur.

**Best Practices:** Fork/Join framework, CPU-intensive task'lar için kullanılmalıdır. I/O-intensive task'lar için uygun değildir.

**Use Cases:** Fork/Join framework, sorting, searching, mathematical computation için kullanılır. Parallel algorithm'lar için uygundur.

**Performance:** Fork/Join framework, high performance sağlar. Work stealing, CPU utilization optimize eder.

**Monitoring:** Fork/Join framework, monitoring gerektirir. Task execution time, thread utilization monitor edilmelidir.

---

## Spring Framework Temelleri

### Temel Seviye

#### 1. Inversion of Control (IoC) nedir?

**Cevap:**
Inversion of Control (IoC), Spring Framework'ün temel prensibidir. Object'lerin dependency'lerinin external container tarafından yönetilmesini sağlar.

**Temel Konsept:** IoC, object'lerin dependency'lerini kendilerinin oluşturmak yerine, external container'ın sağlamasını sağlar. Control'ün inversion'ı gerçekleşir.

**Traditional Approach:** Traditional approach'ta, object'ler kendi dependency'lerini oluşturur. Tight coupling oluşur. Test edilmesi zordur.

**IoC Approach:** IoC approach'ta, object'ler dependency'lerini external container'dan alır. Loose coupling oluşur. Test edilmesi kolaydır.

**Dependency Injection:** Dependency Injection, IoC'ın implementation'ıdır. Object'lerin dependency'leri inject edilir. Constructor, setter, field injection kullanılır.

**Benefits:** IoC, loose coupling, testability, maintainability sağlar. Object'ler arası dependency'ler azalır.

**Best Practices:** IoC, interface'ler üzerinden kullanılmalıdır. Concrete class'lara bağımlılık azaltılmalıdır.

**Use Cases:** IoC, enterprise application'larda kullanılır. Service layer, repository layer için uygundur.

**Performance:** IoC, minimal performance overhead yaratır. Container initialization, object creation overhead'i vardır.

**Monitoring:** IoC, monitoring gerektirir. Bean creation, dependency resolution monitor edilmelidir.

#### 2. Dependency Injection'ın türleri nelerdir?

**Cevap:**
Dependency Injection, Spring'de üç farklı türde yapılabilir. Her tür, farklı use case'ler için optimize edilmiştir.

**Temel Türler:** Dependency Injection, Constructor Injection, Setter Injection, Field Injection türlerinden oluşur.

**Constructor Injection:** Constructor Injection, dependency'leri constructor üzerinden inject eder. Immutable dependency'ler için uygundur. Recommended approach'tır.

**Setter Injection:** Setter Injection, dependency'leri setter method'lar üzerinden inject eder. Optional dependency'ler için uygundur. Flexible injection sağlar.

**Field Injection:** Field Injection, dependency'leri field'lar üzerinden inject eder. @Autowired annotation kullanılır. Simple injection sağlar.

**Constructor Injection Benefits:** Constructor Injection, immutable dependency'ler sağlar. Null safety sağlar. Test edilmesi kolaydır.

**Setter Injection Benefits:** Setter Injection, optional dependency'ler sağlar. Flexible configuration sağlar. Default value'lar set edilebilir.

**Field Injection Benefits:** Field Injection, simple injection sağlar. Boilerplate code azalır. Quick development sağlar.

**Best Practices:** Constructor Injection, primary choice olmalıdır. Setter Injection, optional dependency'ler için kullanılmalıdır. Field Injection, avoid edilmelidir.

**Use Cases:** Constructor Injection, required dependency'ler için kullanılır. Setter Injection, optional dependency'ler için kullanılır.

**Performance:** Dependency Injection türleri, similar performance sağlar. Container overhead'i aynıdır.

**Monitoring:** Dependency Injection, monitoring gerektirir. Injection success rate, dependency resolution monitor edilmelidir.

#### 3. ApplicationContext ve BeanFactory arasındaki fark nedir?

**Cevap:**
ApplicationContext ve BeanFactory, Spring'de bean management için kullanılan iki farklı interface'dir. Her ikisi de farklı functionality ve use case'ler sunar.

**Temel Farklar:** ApplicationContext ve BeanFactory arasındaki temel farklar, functionality, features, use case'ler açısından farklılık gösterir.

**BeanFactory Interface:** BeanFactory, basic bean management sağlar. Bean creation, retrieval, lifecycle management sağlar. Lightweight container'dır.

**ApplicationContext Interface:** ApplicationContext, BeanFactory'yi extend eder. Advanced features sağlar. Enterprise-ready container'dır.

**Functionality:** BeanFactory, basic functionality sağlar. ApplicationContext, advanced functionality sağlar. Feature set farklıdır.

**Features:** BeanFactory, basic features sağlar. ApplicationContext, AOP, event publishing, internationalization sağlar. Enterprise features sağlar.

**Use Cases:** BeanFactory, lightweight application'lar için kullanılır. ApplicationContext, enterprise application'lar için kullanılır.

**Best Practices:** ApplicationContext, primary choice olmalıdır. BeanFactory, resource-constrained environment'ler için kullanılmalıdır.

**Performance:** BeanFactory, düşük overhead'e sahiptir. ApplicationContext, yüksek overhead'e sahiptir. Performance farklıdır.

**Monitoring:** ApplicationContext, advanced monitoring sağlar. BeanFactory, basic monitoring sağlar. Monitoring capability farklıdır.

#### 4. Spring container'ın lifecycle'ı nasıldır?

**Cevap:**
Spring container'ın lifecycle'ı, container'ın initialization'dan destruction'a kadar olan sürecini belirler. Bean'lerin yaşam döngüsü ile entegre çalışır.

**Temel Aşamalar:** Spring container lifecycle, initialization, configuration, bean creation, bean initialization, bean destruction aşamalarından oluşur.

**Initialization:** Initialization, container'ın başlatılması aşamasıdır. Configuration file'ları load edilir. Bean definition'ları parse edilir.

**Configuration:** Configuration, bean definition'ları'nın configure edilmesi aşamasıdır. Dependency'ler resolve edilir. Bean scope'ları belirlenir.

**Bean Creation:** Bean creation, bean'lerin instantiate edilmesi aşamasıdır. Constructor'lar çağrılır. Object'ler oluşturulur.

**Bean Initialization:** Bean initialization, bean'lerin initialize edilmesi aşamasıdır. @PostConstruct method'ları çağrılır. Initialization logic çalışır.

**Bean Destruction:** Bean destruction, bean'lerin destroy edilmesi aşamasıdır. @PreDestroy method'ları çağrılır. Cleanup logic çalışır.

**Lifecycle Callbacks:** Lifecycle callbacks, bean lifecycle'ında çalışan method'lardır. @PostConstruct, @PreDestroy annotation'ları kullanılır.

**Best Practices:** Spring container lifecycle, proper resource management için önemlidir. Cleanup logic implement edilmelidir.

**Use Cases:** Spring container lifecycle, resource management, initialization logic için kullanılır.

**Performance:** Spring container lifecycle, performance'ı etkiler. Initialization time, memory usage optimize edilmelidir.

**Monitoring:** Spring container lifecycle, monitoring gerektirir. Initialization time, bean creation rate monitor edilmelidir.

#### 5. @Configuration class'larının rolü nedir?

**Cevap:**
@Configuration class'ları, Spring'de bean definition'ları için kullanılır. Programmatic bean configuration sağlar ve @Bean method'ları içerir.

**Temel Rol:** @Configuration class'ları, bean definition'ları'nı programmatic olarak tanımlar. @Bean method'ları ile bean'ler oluşturulur.

**@Configuration Annotation:** @Configuration annotation'ı, class'ı configuration class olarak işaretler. Spring tarafından scan edilir. Bean definition'ları sağlar.

**@Bean Method'ları:** @Bean method'ları, bean'lerin oluşturulmasını sağlar. Method name, bean name olur. Return type, bean type olur.

**Bean Dependencies:** @Configuration class'ları, bean dependency'lerini handle eder. Method call'ları ile dependency'ler inject edilir.

**Conditional Configuration:** @Configuration class'ları, conditional configuration sağlar. @Conditional annotation'ları kullanılır. Environment-based configuration sağlar.

**Best Practices:** @Configuration class'ları, clear bean definition'ları için kullanılmalıdır. Method name'ler descriptive olmalıdır.

**Use Cases:** @Configuration class'ları, complex bean configuration, conditional configuration için kullanılır.

**Performance:** @Configuration class'ları, minimal performance overhead yaratır. Bean creation overhead'i vardır.

**Monitoring:** @Configuration class'ları, monitoring gerektirir. Bean creation success rate, configuration loading monitor edilmelidir.

### Orta Seviye

#### 6. Spring AOP'ın temel kavramları nelerdir?

**Cevap:**
Spring AOP (Aspect-Oriented Programming), cross-cutting concern'ları handle etmek için kullanılır. Logging, security, transaction gibi concern'ları modularize eder.

**Temel Kavramlar:** Spring AOP, Aspect, Pointcut, Advice, Join Point, Target Object kavramlarından oluşur.

**Aspect:** Aspect, cross-cutting concern'ı implement eden class'dır. @Aspect annotation ile işaretlenir. Business logic'ten ayrıdır.

**Pointcut:** Pointcut, advice'ın hangi join point'lerde çalışacağını belirler. Expression language kullanır. Method, class level'da tanımlanabilir.

**Advice:** Advice, aspect'in ne zaman çalışacağını belirler. @Before, @After, @Around gibi annotation'lar kullanılır. Join point'lerde çalışır.

**Join Point:** Join Point, advice'ın çalışabileceği point'lerdir. Method execution, object instantiation gibi point'ler. Spring AOP'da sadece method execution desteklenir.

**Target Object:** Target Object, advice'ın uygulandığı object'dir. Proxied object'dir. Original object'in proxy'si oluşturulur.

**Best Practices:** Spring AOP, cross-cutting concern'lar için kullanılmalıdır. Business logic'te kullanılmamalıdır.

**Use Cases:** Spring AOP, logging, security, transaction, caching için kullanılır.

**Performance:** Spring AOP, performance overhead yaratır. Proxy creation, method interception overhead'i vardır.

**Monitoring:** Spring AOP, monitoring gerektirir. Advice execution time, pointcut matching rate monitor edilmelidir.

#### 7. Aspect, Pointcut ve Advice arasındaki ilişki nedir?

**Cevap:**
Aspect, Pointcut ve Advice, Spring AOP'ın temel bileşenleridir. Her biri farklı rol oynar ve birlikte cross-cutting concern'ları implement eder.

**Temel İlişki:** Aspect, Pointcut ve Advice arasındaki ilişki, cross-cutting concern'ın implement edilmesi için gerekli olan bileşenlerin koordinasyonudur.

**Aspect Role:** Aspect, cross-cutting concern'ı implement eden class'dır. Pointcut'ları ve Advice'ları içerir. Modular concern sağlar.

**Pointcut Role:** Pointcut, advice'ın hangi join point'lerde çalışacağını belirler. Expression language kullanır. Join point selection sağlar.

**Advice Role:** Advice, aspect'in ne zaman çalışacağını belirler. Join point'lerde çalışır. Cross-cutting behavior sağlar.

**Coordination:** Aspect, Pointcut ve Advice birlikte çalışır. Pointcut, join point'leri select eder. Advice, selected point'lerde çalışır.

**Best Practices:** Aspect, Pointcut ve Advice, clear separation of concern için kullanılmalıdır. Reusable aspect'ler oluşturulmalıdır.

**Use Cases:** Aspect, Pointcut ve Advice, logging, security, transaction için kullanılır.

**Performance:** Aspect, Pointcut ve Advice, performance overhead yaratır. Pointcut matching, advice execution overhead'i vardır.

**Monitoring:** Aspect, Pointcut ve Advice, monitoring gerektirir. Pointcut matching rate, advice execution time monitor edilmelidir.

#### 8. Spring Events'ın kullanım alanları nelerdir?

**Cevap:**
Spring Events, application'da event-driven architecture sağlar. Loose coupling, asynchronous processing, decoupled communication sağlar.

**Temel Kullanım:** Spring Events, event publishing, event listening sağlar. @EventListener annotation kullanılır. Event-driven communication sağlar.

**Event Publishing:** Event publishing, event'lerin publish edilmesini sağlar. ApplicationEventPublisher kullanılır. Event'ler broadcast edilir.

**Event Listening:** Event listening, event'lerin dinlenmesini sağlar. @EventListener annotation kullanılır. Event'ler handle edilir.

**Event Types:** Event types, custom event class'larıdır. ApplicationEvent'den extend edilir. Event data'sı encapsulate edilir.

**Asynchronous Events:** Asynchronous events, @Async annotation ile sağlanır. Event'ler async olarak process edilir. Non-blocking processing sağlar.

**Best Practices:** Spring Events, loose coupling için kullanılmalıdır. Event'ler lightweight olmalıdır.

**Use Cases:** Spring Events, notification, audit logging, integration için kullanılır.

**Performance:** Spring Events, performance overhead yaratır. Event publishing, listener execution overhead'i vardır.

**Monitoring:** Spring Events, monitoring gerektirir. Event publishing rate, listener execution time monitor edilmelidir.

#### 9. Property placeholder'ların resolution'ı nasıl çalışır?

**Cevap:**
Property placeholder'lar, Spring'de external configuration'ları inject etmek için kullanılır. ${property.name} syntax'ı kullanılır.

**Temel Çalışma:** Property placeholder'lar, PropertyPlaceholderConfigurer veya @Value annotation ile çalışır. External property'ler inject edilir.

**PropertyPlaceholderConfigurer:** PropertyPlaceholderConfigurer, property file'larından property'leri load eder. Bean definition'larında kullanılır.

**@Value Annotation:** @Value annotation, property'leri field'lara inject eder. ${property.name} syntax'ı kullanılır. Type conversion sağlar.

**Property Sources:** Property sources, property'lerin nereden geldiğini belirler. File, environment variable, system property kullanılır.

**Resolution Order:** Resolution order, property'lerin hangi sırayla resolve edileceğini belirler. Priority-based resolution sağlar.

**Best Practices:** Property placeholder'lar, external configuration için kullanılmalıdır. Default value'lar sağlanmalıdır.

**Use Cases:** Property placeholder'lar, database configuration, external service URL'leri için kullanılır.

**Performance:** Property placeholder'lar, minimal performance overhead yaratır. Property resolution overhead'i vardır.

**Monitoring:** Property placeholder'lar, monitoring gerektirir. Property resolution success rate, missing property'ler monitor edilmelidir.

#### 10. Spring Expression Language (SpEL) nasıl kullanılır?

**Cevap:**
Spring Expression Language (SpEL), Spring'de expression evaluation için kullanılır. #{expression} syntax'ı kullanılır.

**Temel Kullanım:** SpEL, expression'ları evaluate eder. Bean reference, method call, property access sağlar. Dynamic expression evaluation sağlar.

**Expression Syntax:** Expression syntax, #{expression} format'ında yazılır. Bean reference, method call, property access kullanılır.

**Bean Reference:** Bean reference, #{beanName} syntax'ı ile yapılır. Bean'ler reference edilir. Dependency injection sağlar.

**Method Call:** Method call, #{beanName.methodName()} syntax'ı ile yapılır. Method'lar çağrılır. Dynamic method invocation sağlar.

**Property Access:** Property access, #{beanName.property} syntax'ı ile yapılır. Property'ler access edilir. Nested property access sağlar.

**Best Practices:** SpEL, dynamic expression evaluation için kullanılmalıdır. Complex expression'lar avoid edilmelidir.

**Use Cases:** SpEL, conditional bean creation, dynamic property injection için kullanılır.

**Performance:** SpEL, performance overhead yaratır. Expression parsing, evaluation overhead'i vardır.

**Monitoring:** SpEL, monitoring gerektirir. Expression evaluation time, success rate monitor edilmelidir.

---

## Spring Security

### Temel Seviye

#### 1. Spring Security'nin temel bileşenleri nelerdir?

**Cevap:**
Spring Security, Java application'ları için comprehensive security framework'üdür. Authentication, authorization, protection against common attacks sağlar.

**Temel Bileşenler:** Spring Security, SecurityFilterChain, AuthenticationManager, UserDetailsService, PasswordEncoder bileşenlerinden oluşur.

**SecurityFilterChain:** SecurityFilterChain, HTTP request'lerin security filter'larından geçmesini sağlar. Request processing pipeline'ı oluşturur.

**AuthenticationManager:** AuthenticationManager, authentication process'ini manage eder. Authentication provider'ları coordinate eder. Authentication result döner.

**UserDetailsService:** UserDetailsService, user information'ları load eder. UserDetails object döner. Database, LDAP gibi source'lardan user bilgileri alır.

**PasswordEncoder:** PasswordEncoder, password'ları encode/decode eder. BCrypt, Argon2 gibi algorithm'lar kullanır. Password security sağlar.

**Best Practices:** Spring Security, defense in depth principle'ına uygun kullanılmalıdır. Multiple security layer'ları implement edilmelidir.

**Use Cases:** Spring Security, web application, REST API, microservice security için kullanılır.

**Performance:** Spring Security, performance overhead yaratır. Filter chain, authentication overhead'i vardır.

**Monitoring:** Spring Security, monitoring gerektirir. Authentication success rate, authorization failure rate monitor edilmelidir.

#### 2. Authentication ve Authorization arasındaki fark nedir?

**Cevap:**
Authentication ve Authorization, security'nin iki farklı aspect'idir. Authentication, identity verification'dır. Authorization, permission checking'dir.

**Authentication:** Authentication, user'ın kim olduğunu verify eder. Username/password, token, certificate kullanır. Identity verification sağlar.

**Authorization:** Authorization, user'ın ne yapabileceğini kontrol eder. Role, permission, resource-based authorization sağlar. Access control sağlar.

**Temel Fark:** Authentication, "Who are you?" sorusunu cevaplar. Authorization, "What can you do?" sorusunu cevaplar.

**Authentication Flow:** Authentication flow, login process'idir. Credential validation, session creation sağlar. User identity establish eder.

**Authorization Flow:** Authorization flow, access control process'idir. Permission checking, resource access sağlar. User action'ları kontrol eder.

**Best Practices:** Authentication ve Authorization, separate concern'lar olarak treat edilmelidir. Clear separation sağlanmalıdır.

**Use Cases:** Authentication, login, session management için kullanılır. Authorization, resource access, API endpoint protection için kullanılır.

**Performance:** Authentication, login time overhead yaratır. Authorization, request time overhead yaratır.

**Monitoring:** Authentication, login success rate monitor edilmelidir. Authorization, access denial rate monitor edilmelidir.

#### 3. @PreAuthorize ve @PostAuthorize annotation'larının kullanımı nasıldır?

**Cevap:**
@PreAuthorize ve @PostAuthorize, Spring Security'de method level security sağlar. SpEL expression'ları kullanır. Fine-grained access control sağlar.

**@PreAuthorize:** @PreAuthorize, method execution'dan önce authorization check yapar. SpEL expression evaluate eder. Method access'i kontrol eder.

**@PostAuthorize:** @PostAuthorize, method execution'dan sonra authorization check yapar. Return value'ya göre authorization yapar. Result access'i kontrol eder.

**SpEL Expressions:** SpEL expressions, authorization logic'ini define eder. hasRole(), hasAuthority(), authentication.name gibi expression'lar kullanılır.

**hasRole():** hasRole(), user'ın belirli role'e sahip olup olmadığını kontrol eder. ROLE_ prefix'i otomatik eklenir. Role-based authorization sağlar.

**hasAuthority():** hasAuthority(), user'ın belirli authority'e sahip olup olmadığını kontrol eder. Exact authority name kullanılır. Authority-based authorization sağlar.

**Best Practices:** @PreAuthorize ve @PostAuthorize, method level security için kullanılmalıdır. Complex authorization logic'te kullanılmalıdır.

**Use Cases:** @PreAuthorize ve @PostAuthorize, service method'ları, repository method'ları için kullanılır.

**Performance:** @PreAuthorize ve @PostAuthorize, method execution overhead yaratır. SpEL evaluation overhead'i vardır.

**Monitoring:** @PreAuthorize ve @PostAuthorize, monitoring gerektirir. Authorization failure rate, method access rate monitor edilmelidir.

#### 4. CSRF protection nasıl çalışır?

**Cevap:**
CSRF (Cross-Site Request Forgery) protection, malicious website'lerin user'ın behalf'inde request göndermesini engeller. Token-based protection sağlar.

**Temel Çalışma:** CSRF protection, CSRF token generate eder. Form submission'da token validate eder. Token mismatch'te request'i reject eder.

**CSRF Token:** CSRF token, unique, unpredictable token'dır. Session'da store edilir. Form'da hidden field olarak gönderilir.

**Token Validation:** Token validation, request'teki token'ın session'daki token ile match edip etmediğini kontrol eder. Match etmezse request reject edilir.

**Same-Origin Policy:** Same-Origin Policy, browser'ın same origin'den gelen request'leri allow etmesini sağlar. Cross-origin request'leri block eder.

**Best Practices:** CSRF protection, state-changing operation'lar için kullanılmalıdır. GET request'ler için gerekli değildir.

**Use Cases:** CSRF protection, form submission, API endpoint'leri için kullanılır.

**Performance:** CSRF protection, minimal performance overhead yaratır. Token generation, validation overhead'i vardır.

**Monitoring:** CSRF protection, monitoring gerektirir. CSRF attack attempt'leri, token validation failure rate monitor edilmelidir.

#### 5. Password encoding'ın önemi nedir?

**Cevap:**
Password encoding, password'ların secure şekilde store edilmesini sağlar. Plain text password'lar store edilmez. Hash'lenmiş password'lar store edilir.

**Temel Önem:** Password encoding, password security'nin temelidir. Database breach'te bile password'lar protect edilir. User privacy sağlar.

**Hash Algorithm:** Hash algorithm, password'ları one-way hash'e çevirir. BCrypt, Argon2, PBKDF2 gibi algorithm'lar kullanılır. Reverse engineering imkansızdır.

**Salt:** Salt, password hash'ine eklenen random data'dır. Rainbow table attack'ları engeller. Unique hash'ler oluşturur.

**BCrypt:** BCrypt, adaptive hashing algorithm'ıdır. Work factor ile hash strength ayarlanabilir. Time-consuming hash generation sağlar.

**Best Practices:** Password encoding, strong algorithm'lar kullanmalıdır. Salt kullanılmalıdır. Work factor yeterli olmalıdır.

**Use Cases:** Password encoding, user registration, password change için kullanılır.

**Performance:** Password encoding, CPU intensive operation'dır. Hash generation time overhead'i vardır.

**Monitoring:** Password encoding, monitoring gerektirir. Hash generation time, encoding success rate monitor edilmelidir.

### Orta Seviye

#### 6. OAuth2 ve JWT integration'ı nasıl yapılır?

**Cevap:**
OAuth2 ve JWT integration, modern authentication ve authorization sağlar. Stateless authentication, token-based security sağlar.

**OAuth2 Flow:** OAuth2 flow, authorization code, client credentials, resource owner password gibi flow'ları destekler. Authorization server ile communication sağlar.

**JWT Token:** JWT token, self-contained token'dır. Header, payload, signature'dan oluşur. Stateless authentication sağlar.

**Token Structure:** Token structure, header (algorithm), payload (claims), signature (verification) bölümlerinden oluşur. Base64 encoded format'ta store edilir.

**Claims:** Claims, token'da bulunan information'lardır. iss (issuer), exp (expiration), sub (subject) gibi standard claim'ler vardır.

**Token Validation:** Token validation, signature verification, expiration check, issuer validation yapar. Token integrity sağlar.

**Best Practices:** OAuth2 ve JWT, stateless authentication için kullanılmalıdır. Token expiration time yeterli olmalıdır.

**Use Cases:** OAuth2 ve JWT, microservice authentication, API security için kullanılır.

**Performance:** OAuth2 ve JWT, minimal performance overhead yaratır. Token validation overhead'i vardır.

**Monitoring:** OAuth2 ve JWT, monitoring gerektirir. Token validation success rate, token expiration rate monitor edilmelidir.

#### 7. Method level security nasıl implement edilir?

**Cevap:**
Method level security, individual method'lar için fine-grained access control sağlar. @PreAuthorize, @PostAuthorize, @Secured annotation'ları kullanılır.

**@PreAuthorize:** @PreAuthorize, method execution'dan önce authorization check yapar. SpEL expression kullanır. Method access'i kontrol eder.

**@PostAuthorize:** @PostAuthorize, method execution'dan sonra authorization check yapar. Return value'ya göre authorization yapar. Result access'i kontrol eder.

**@Secured:** @Secured, role-based authorization sağlar. Simple role check yapar. Method access'i kontrol eder.

**SpEL Integration:** SpEL integration, complex authorization logic sağlar. hasRole(), hasAuthority(), authentication.name gibi expression'lar kullanılır.

**AOP Integration:** AOP integration, method level security'yi implement eder. Proxy-based interception sağlar. Transparent security sağlar.

**Best Practices:** Method level security, business logic method'ları için kullanılmalıdır. Clear authorization rule'ları define edilmelidir.

**Use Cases:** Method level security, service method'ları, repository method'ları için kullanılır.

**Performance:** Method level security, method execution overhead yaratır. Authorization check overhead'i vardır.

**Monitoring:** Method level security, monitoring gerektirir. Authorization failure rate, method access rate monitor edilmelidir.

#### 8. Custom authentication provider nasıl oluşturulur?

**Cevap:**
Custom authentication provider, custom authentication logic implement eder. AuthenticationProvider interface'ini implement eder. Custom authentication mechanism sağlar.

**AuthenticationProvider Interface:** AuthenticationProvider interface, authenticate() ve supports() method'larını implement etmeyi gerektirir. Authentication logic sağlar.

**authenticate() Method:** authenticate() method, authentication process'ini implement eder. Authentication object alır, Authentication result döner. Authentication logic sağlar.

**supports() Method:** supports() method, provider'ın hangi authentication type'ını support ettiğini belirler. Class type check yapar. Provider selection sağlar.

**Custom Authentication:** Custom authentication, custom credential validation sağlar. Database, LDAP, external service integration sağlar. Flexible authentication sağlar.

**Provider Registration:** Provider registration, AuthenticationManagerBuilder'a provider register edilir. Authentication manager'a provider eklenir. Provider chain oluşturulur.

**Best Practices:** Custom authentication provider, specific authentication requirement'lar için kullanılmalıdır. Clear authentication logic implement edilmelidir.

**Use Cases:** Custom authentication provider, multi-factor authentication, biometric authentication için kullanılır.

**Performance:** Custom authentication provider, authentication time overhead yaratır. Custom validation overhead'i vardır.

**Monitoring:** Custom authentication provider, monitoring gerektirir. Authentication success rate, provider usage rate monitor edilmelidir.

#### 9. Session management stratejileri nelerdir?

**Cevap:**
Session management stratejileri, user session'larının nasıl manage edileceğini belirler. Stateless, stateful, hybrid approach'lar kullanılır.

**Stateless Strategy:** Stateless strategy, session state'i server'da store etmez. JWT token'lar kullanılır. Scalability sağlar.

**Stateful Strategy:** Stateful strategy, session state'i server'da store eder. Session store kullanılır. Session control sağlar.

**Hybrid Strategy:** Hybrid strategy, stateless ve stateful approach'ları combine eder. Critical state server'da, non-critical state client'da store edilir.

**Session Store:** Session store, Redis, database, memory gibi store'lar kullanılır. Session data persistence sağlar. Session sharing sağlar.

**Session Timeout:** Session timeout, session'ın ne kadar süre valid olacağını belirler. Security ve usability balance sağlar.

**Best Practices:** Session management, application requirement'larına göre seçilmelidir. Security ve performance balance sağlanmalıdır.

**Use Cases:** Session management, web application, mobile application için kullanılır.

**Performance:** Session management, performance overhead yaratır. Session store access, session validation overhead'i vardır.

**Monitoring:** Session management, monitoring gerektirir. Session creation rate, session timeout rate monitor edilmelidir.

#### 10. Security context'in thread safety'si nasıl sağlanır?

**Cevap:**
Security context'in thread safety'si, multi-threaded environment'da security context'in güvenli şekilde access edilmesini sağlar. ThreadLocal, SecurityContextHolder kullanılır.

**ThreadLocal Storage:** ThreadLocal storage, her thread için separate security context sağlar. Thread isolation sağlar. Thread safety sağlar.

**SecurityContextHolder:** SecurityContextHolder, security context'i manage eder. ThreadLocal, Global, InheritableThreadLocal strategy'leri destekler. Context management sağlar.

**Context Propagation:** Context propagation, security context'in thread'ler arası propagate edilmesini sağlar. Async method'larda context transfer sağlar.

**Context Clearing:** Context clearing, security context'in thread'den temizlenmesini sağlar. Memory leak prevention sağlar. Resource cleanup sağlar.

**Best Practices:** Security context, thread-safe şekilde access edilmelidir. Context propagation doğru implement edilmelidir.

**Use Cases:** Security context, multi-threaded application, async processing için kullanılır.

**Performance:** Security context, minimal performance overhead yaratır. ThreadLocal access overhead'i vardır.

**Monitoring:** Security context, monitoring gerektirir. Context access rate, context propagation success rate monitor edilmelidir.

---

## Spring Data JPA

### Temel Seviye

#### 1. JPA ve Hibernate arasındaki ilişki nedir?

**Cevap:**
JPA (Java Persistence API) ve Hibernate, Java'da object-relational mapping (ORM) sağlar. JPA specification'dır, Hibernate implementation'dır.

**Temel İlişki:** JPA, Java EE specification'ıdır. Hibernate, JPA specification'ının en popüler implementation'ıdır. JPA interface'leri, Hibernate concrete class'ları sağlar.

**JPA Specification:** JPA specification, ORM için standard interface'leri define eder. @Entity, @Table, @Id gibi annotation'ları sağlar. Database independence sağlar.

**Hibernate Implementation:** Hibernate implementation, JPA specification'ını implement eder. EntityManager, SessionFactory gibi concrete class'ları sağlar. Advanced ORM features sağlar.

**EntityManager:** EntityManager, JPA'nın core interface'idir. Entity lifecycle management sağlar. CRUD operation'ları sağlar.

**SessionFactory:** SessionFactory, Hibernate'in core class'ıdır. Session creation sağlar. Database connection management sağlar.

**Best Practices:** JPA annotation'ları kullanılmalıdır. Hibernate-specific feature'lar avoid edilmelidir. Database independence sağlanmalıdır.

**Use Cases:** JPA ve Hibernate, enterprise application'lar, web application'lar için kullanılır.

**Performance:** JPA ve Hibernate, performance overhead yaratır. Object-relational mapping overhead'i vardır.

**Monitoring:** JPA ve Hibernate, monitoring gerektirir. Query execution time, connection pool usage monitor edilmelidir.

#### 2. Entity, Repository ve Service katmanlarının rolleri nelerdir?

**Cevap:**
Entity, Repository ve Service katmanları, layered architecture'ın temel bileşenleridir. Her katman farklı responsibility'ye sahiptir.

**Entity Katmanı:** Entity katmanı, database table'larını represent eder. @Entity annotation ile işaretlenir. Data model sağlar.

**Repository Katmanı:** Repository katmanı, data access logic'ini encapsulate eder. JpaRepository interface'ini extend eder. Data persistence sağlar.

**Service Katmanı:** Service katmanı, business logic'i implement eder. Repository'leri kullanır. Business rule'ları sağlar.

**Temel Roller:** Entity, data representation sağlar. Repository, data access sağlar. Service, business logic sağlar.

**Entity Responsibilities:** Entity, data structure define eder. Validation rule'ları sağlar. Database mapping sağlar.

**Repository Responsibilities:** Repository, CRUD operation'ları sağlar. Custom query'ler implement eder. Data access abstraction sağlar.

**Service Responsibilities:** Service, business logic implement eder. Transaction management sağlar. Business rule'ları enforce eder.

**Best Practices:** Entity, Repository, Service katmanları, clear separation of concern için kullanılmalıdır. Dependency direction doğru olmalıdır.

**Use Cases:** Entity, Repository, Service katmanları, enterprise application'lar için kullanılır.

**Performance:** Entity, Repository, Service katmanları, performance overhead yaratır. Layer interaction overhead'i vardır.

**Monitoring:** Entity, Repository, Service katmanları, monitoring gerektirir. Layer interaction time, business logic execution time monitor edilmelidir.

#### 3. @Entity, @Table, @Id annotation'larının kullanımı nasıldır?

**Cevap:**
@Entity, @Table, @Id annotation'ları, JPA'da entity mapping için kullanılır. Database table'ları ile Java class'ları arasında mapping sağlar.

**@Entity Annotation:** @Entity annotation, class'ın JPA entity olduğunu belirtir. EntityManager tarafından manage edilir. Database table'a map edilir.

**@Table Annotation:** @Table annotation, entity'nin hangi table'a map edileceğini belirtir. Table name, schema, catalog specify edilebilir. Default table name class name'dir.

**@Id Annotation:** @Id annotation, entity'nin primary key field'ını belirtir. Unique identifier sağlar. Database primary key constraint oluşturur.

**Temel Kullanım:** @Entity, class level'da kullanılır. @Table, class level'da kullanılır. @Id, field level'da kullanılır.

**@GeneratedValue:** @GeneratedValue annotation, primary key'in nasıl generate edileceğini belirtir. AUTO, IDENTITY, SEQUENCE, TABLE strategy'leri destekler.

**@Column:** @Column annotation, field'ın hangi column'a map edileceğini belirtir. Column name, length, nullable gibi property'ler specify edilebilir.

**Best Practices:** @Entity, @Table, @Id annotation'ları, clear mapping için kullanılmalıdır. Default value'lar kullanılmalıdır.

**Use Cases:** @Entity, @Table, @Id annotation'ları, data model definition için kullanılır.

**Performance:** @Entity, @Table, @Id annotation'ları, minimal performance overhead yaratır. Metadata processing overhead'i vardır.

**Monitoring:** @Entity, @Table, @Id annotation'ları, monitoring gerektirir. Entity creation time, mapping success rate monitor edilmelidir.

#### 4. JPA relationship'lerin türleri nelerdir?

**Cevap:**
JPA relationship'leri, entity'ler arasındaki ilişkileri define eder. One-to-One, One-to-Many, Many-to-One, Many-to-Many türleri vardır.

**One-to-One:** One-to-One relationship, bir entity'nin başka bir entity ile unique ilişkisi olduğunu belirtir. @OneToOne annotation kullanılır.

**One-to-Many:** One-to-Many relationship, bir entity'nin birden fazla entity ile ilişkisi olduğunu belirtir. @OneToMany annotation kullanılır.

**Many-to-One:** Many-to-One relationship, birden fazla entity'nin bir entity ile ilişkisi olduğunu belirtir. @ManyToOne annotation kullanılır.

**Many-to-Many:** Many-to-Many relationship, birden fazla entity'nin birden fazla entity ile ilişkisi olduğunu belirtir. @ManyToMany annotation kullanılır.

**@JoinColumn:** @JoinColumn annotation, foreign key column'ını specify eder. Relationship mapping sağlar. Database constraint oluşturur.

**@JoinTable:** @JoinTable annotation, many-to-many relationship'ler için join table specify eder. Intermediate table oluşturur.

**Best Practices:** JPA relationship'leri, clear relationship definition için kullanılmalıdır. Appropriate annotation'lar seçilmelidir.

**Use Cases:** JPA relationship'leri, complex data model'ler için kullanılır.

**Performance:** JPA relationship'leri, performance overhead yaratır. Join query overhead'i vardır.

**Monitoring:** JPA relationship'leri, monitoring gerektirir. Relationship loading time, join query performance monitor edilmelidir.

#### 5. Lazy vs Eager loading arasındaki fark nedir?

**Cevap:**
Lazy ve Eager loading, JPA'da relationship'lerin ne zaman load edileceğini belirler. Performance ve memory usage trade-off'u sağlar.

**Lazy Loading:** Lazy loading, relationship'lerin ihtiyaç duyulduğunda load edilmesini sağlar. On-demand loading sağlar. Memory efficient'tir.

**Eager Loading:** Eager loading, relationship'lerin entity load edilirken birlikte load edilmesini sağlar. Immediate loading sağlar. Query efficient'tir.

**Temel Fark:** Lazy loading, memory efficient'tir. Eager loading, query efficient'tir. Performance trade-off'u vardır.

**@LazyCollection:** @LazyCollection annotation, collection'ların lazy loading'ini kontrol eder. EXTRA, TRUE, FALSE value'ları destekler.

**N+1 Problem:** N+1 problem, lazy loading'de ortaya çıkar. Her entity için ayrı query çalışır. Performance problem yaratır.

**Fetch Join:** Fetch join, N+1 problem'ini çözer. Single query ile relationship'leri load eder. Eager loading sağlar.

**Best Practices:** Lazy loading, default olarak kullanılmalıdır. Eager loading, specific use case'ler için kullanılmalıdır.

**Use Cases:** Lazy loading, memory-constrained environment'lar için kullanılır. Eager loading, query-optimized environment'lar için kullanılır.

**Performance:** Lazy loading, memory efficient'tir. Eager loading, query efficient'tir.

**Monitoring:** Lazy loading, monitoring gerektirir. Lazy loading time, N+1 query count monitor edilmelidir.

### Orta Seviye

#### 6. Custom query'ler nasıl yazılır?

**Cevap:**
Custom query'ler, JPA'da complex data access requirement'ları için kullanılır. @Query annotation, JPQL, native SQL kullanılır.

**@Query Annotation:** @Query annotation, custom query'leri define eder. JPQL veya native SQL kullanılır. Method name convention'ına gerek yoktur.

**JPQL:** JPQL (Java Persistence Query Language), object-oriented query language'dır. Entity'ler üzerinde query yazılır. Database independent'tır.

**Native SQL:** Native SQL, database-specific SQL query'leridir. Performance optimization sağlar. Database dependent'tır.

**Query Parameters:** Query parameters, @Param annotation ile bind edilir. Type-safe parameter binding sağlar. SQL injection prevention sağlar.

**Named Query:** Named query, @NamedQuery annotation ile define edilir. Reusable query'ler sağlar. Performance optimization sağlar.

**Best Practices:** Custom query'ler, complex requirement'lar için kullanılmalıdır. JPQL tercih edilmelidir. Native SQL sadece gerekli olduğunda kullanılmalıdır.

**Use Cases:** Custom query'ler, complex reporting, data analysis için kullanılır.

**Performance:** Custom query'ler, performance optimization sağlar. Query execution time optimize edilir.

**Monitoring:** Custom query'ler, monitoring gerektirir. Query execution time, query success rate monitor edilmelidir.

#### 7. @Query annotation'ının kullanımı nasıldır?

**Cevap:**
@Query annotation, Spring Data JPA'da custom query'leri define etmek için kullanılır. JPQL, native SQL, SpEL expression'ları destekler.

**Temel Kullanım:** @Query annotation, method level'da kullanılır. Query string specify edilir. Method name convention'ına gerek yoktur.

**JPQL Query:** JPQL query, object-oriented query language kullanır. Entity'ler üzerinde query yazılır. Database independent'tır.

**Native Query:** Native query, database-specific SQL kullanır. nativeQuery = true parameter'i kullanılır. Performance optimization sağlar.

**Parameter Binding:** Parameter binding, @Param annotation ile yapılır. :parameterName syntax'ı kullanılır. Type-safe binding sağlar.

**SpEL Integration:** SpEL integration, #{#entityName} gibi expression'lar kullanır. Dynamic query generation sağlar. Entity name injection sağlar.

**Best Practices:** @Query annotation, complex query'ler için kullanılmalıdır. JPQL tercih edilmelidir. Parameter binding güvenli olmalıdır.

**Use Cases:** @Query annotation, complex data access, reporting query'leri için kullanılır.

**Performance:** @Query annotation, performance optimization sağlar. Query execution time optimize edilir.

**Monitoring:** @Query annotation, monitoring gerektirir. Query execution time, parameter binding success rate monitor edilmelidir.

#### 8. Pagination ve Sorting nasıl implement edilir?

**Cevap:**
Pagination ve Sorting, JPA'da large dataset'lerin efficient şekilde handle edilmesini sağlar. Pageable, Sort interface'leri kullanılır.

**Pageable Interface:** Pageable interface, pagination parameter'larını encapsulate eder. Page number, page size, sort criteria sağlar.

**Sort Interface:** Sort interface, sorting criteria'larını encapsulate eder. Field name, direction (ASC, DESC) sağlar.

**Page Interface:** Page interface, paginated result'ları encapsulate eder. Content, total elements, total pages sağlar.

**Repository Method:** Repository method, Pageable parameter alır. Page<T> return type kullanır. Automatic pagination sağlar.

**Custom Pagination:** Custom pagination, @Query annotation ile implement edilir. Manual pagination logic sağlar. Complex pagination sağlar.

**Best Practices:** Pagination ve Sorting, large dataset'ler için kullanılmalıdır. Page size optimize edilmelidir. Sort field'ları index'lenmiş olmalıdır.

**Use Cases:** Pagination ve Sorting, data listing, search result'ları için kullanılır.

**Performance:** Pagination ve Sorting, performance optimization sağlar. Memory usage optimize edilir.

**Monitoring:** Pagination ve Sorting, monitoring gerektirir. Page load time, sort performance monitor edilmelidir.

#### 9. Transaction management JPA'da nasıl çalışır?

**Cevap:**
Transaction management, JPA'da data consistency sağlar. @Transactional annotation, transaction boundary'lerini define eder. ACID property'leri sağlar.

**@Transactional Annotation:** @Transactional annotation, method'ları transaction boundary'si olarak işaretler. Automatic transaction management sağlar.

**Transaction Propagation:** Transaction propagation, transaction'ların nasıl propagate edileceğini belirler. REQUIRED, REQUIRES_NEW, SUPPORTS gibi option'lar vardır.

**Transaction Isolation:** Transaction isolation, transaction'lar arası isolation level'ını belirler. READ_COMMITTED, REPEATABLE_READ gibi level'lar vardır.

**Rollback Rules:** Rollback rules, hangi exception'larda rollback yapılacağını belirler. Default olarak RuntimeException'larda rollback yapılır.

**EntityManager:** EntityManager, transaction context'inde çalışır. Entity state management sağlar. Database synchronization sağlar.

**Best Practices:** Transaction management, business logic method'ları için kullanılmalıdır. Short transaction'lar tercih edilmelidir.

**Use Cases:** Transaction management, data modification, business operation'lar için kullanılır.

**Performance:** Transaction management, performance overhead yaratır. Transaction boundary overhead'i vardır.

**Monitoring:** Transaction management, monitoring gerektirir. Transaction duration, rollback rate monitor edilmelidir.

#### 10. N+1 problem'i nedir ve nasıl çözülür?

**Cevap:**
N+1 problem, JPA'da lazy loading'de ortaya çıkan performance problem'idir. N entity için N+1 query çalışır. Performance degradation yaratır.

**Problem Tanımı:** N+1 problem, parent entity load edildikten sonra, her child entity için ayrı query çalışmasıdır. Query count exponential olarak artar.

**Lazy Loading:** Lazy loading, N+1 problem'inin temel nedenidir. Relationship'ler on-demand load edilir. Her access'te yeni query çalışır.

**Çözüm Yöntemleri:** N+1 problem'i, fetch join, @EntityGraph, batch fetching ile çözülür. Single query ile relationship'ler load edilir.

**Fetch Join:** Fetch join, JPQL'de JOIN FETCH kullanır. Single query ile relationship'leri load eder. Eager loading sağlar.

**@EntityGraph:** @EntityGraph annotation, entity graph'ı define eder. Relationship loading strategy sağlar. Performance optimization sağlar.

**Batch Fetching:** Batch fetching, @BatchSize annotation kullanır. Multiple entity'leri batch'te load eder. Query count'u azaltır.

**Best Practices:** N+1 problem'i, fetch join ile çözülmelidir. Lazy loading dikkatli kullanılmalıdır. Query count monitor edilmelidir.

**Use Cases:** N+1 problem'i, complex relationship'li entity'ler için ortaya çıkar.

**Performance:** N+1 problem'i, performance degradation yaratır. Query count exponential olarak artar.

**Monitoring:** N+1 problem'i, monitoring gerektirir. Query count, relationship loading time monitor edilmelidir.

#### 11. Caching stratejileri nelerdir?

**Cevap:**
Caching stratejileri, JPA'da performance optimization sağlar. First-level cache, second-level cache, query cache kullanılır.

**First-Level Cache:** First-level cache, EntityManager scope'unda çalışır. Session cache olarak da bilinir. Automatic cache sağlar.

**Second-Level Cache:** Second-level cache, application scope'unda çalışır. Shared cache sağlar. Multiple EntityManager'lar arasında paylaşılır.

**Query Cache:** Query cache, query result'larını cache eder. Identical query'ler için cache'den result döner. Performance optimization sağlar.

**Cache Configuration:** Cache configuration, @Cacheable, @CacheEvict annotation'ları ile yapılır. Cache strategy, expiration time sağlar.

**Cache Provider:** Cache provider, EhCache, Hazelcast, Redis gibi provider'lar kullanılır. Cache implementation sağlar.

**Best Practices:** Caching stratejileri, read-heavy application'lar için kullanılmalıdır. Cache invalidation strategy sağlanmalıdır.

**Use Cases:** Caching stratejileri, frequently accessed data için kullanılır.

**Performance:** Caching stratejileri, performance optimization sağlar. Database access count azalır.

**Monitoring:** Caching stratejileri, monitoring gerektirir. Cache hit rate, cache miss rate monitor edilmelidir.

#### 12. Database migration'lar nasıl yönetilir?

**Cevap:**
Database migration'lar, database schema değişikliklerinin version control'ünü sağlar. Flyway, Liquibase gibi tool'lar kullanılır.

**Migration Tool:** Migration tool, database schema version'larını manage eder. Version control sağlar. Rollback capability sağlar.

**Flyway:** Flyway, database migration tool'ıdır. SQL script'leri version'lar. Automatic migration sağlar.

**Liquibase:** Liquibase, database migration tool'ıdır. XML, YAML format'ları destekler. Cross-database compatibility sağlar.

**Migration Script:** Migration script, database schema değişikliklerini define eder. Version number, description sağlar. Idempotent olmalıdır.

**Version Control:** Version control, migration script'lerinin version'larını track eder. Git, SVN gibi tool'lar kullanılır.

**Best Practices:** Database migration'lar, version control'de tutulmalıdır. Idempotent script'ler yazılmalıdır. Rollback plan'ı hazırlanmalıdır.

**Use Cases:** Database migration'lar, schema evolution, data migration için kullanılır.

**Performance:** Database migration'lar, performance overhead yaratır. Schema change overhead'i vardır.

**Monitoring:** Database migration'lar, monitoring gerektirir. Migration success rate, migration time monitor edilmelidir.

---

## Spring Boot Auto Configuration

### Temel Seviye

#### 1. Auto Configuration'ın amacı nedir?

**Cevap:**
Auto Configuration, Spring Boot'da application'ların minimal configuration ile çalışmasını sağlar. Convention over configuration principle'ına uygun çalışır.

**Temel Amaç:** Auto Configuration, developer'ların boilerplate configuration'ları yazmasını engeller. Sensible default'lar sağlar. Rapid development sağlar.

**Convention over Configuration:** Convention over Configuration, standard convention'ları kullanarak configuration'ları minimize eder. Custom configuration sadece gerekli olduğunda yapılır.

**Sensible Defaults:** Sensible defaults, common use case'ler için optimal configuration'lar sağlar. Database connection, web server, security gibi configuration'lar otomatik yapılır.

**Conditional Configuration:** Conditional configuration, classpath'te bulunan dependency'lere göre configuration'ları enable eder. Conditional annotation'lar kullanılır.

**Best Practices:** Auto Configuration, rapid prototyping için kullanılmalıdır. Production'da gerekli configuration'lar explicit olarak yapılmalıdır.

**Use Cases:** Auto Configuration, web application'lar, REST API'lar, microservice'ler için kullanılır.

**Performance:** Auto Configuration, startup time'ı optimize eder. Lazy loading, conditional configuration sağlar.

**Monitoring:** Auto Configuration, monitoring gerektirir. Configuration loading time, conditional evaluation success rate monitor edilmelidir.

#### 2. @EnableAutoConfiguration annotation'ının işlevi nedir?

**Cevap:**
@EnableAutoConfiguration annotation, Spring Boot'da auto configuration'ı enable eder. @SpringBootApplication annotation'ının içinde bulunur. Auto configuration mechanism'ini başlatır.

**Temel İşlev:** @EnableAutoConfiguration, classpath'te bulunan dependency'lere göre configuration'ları otomatik olarak enable eder. Conditional evaluation yapar.

**Auto Configuration Classes:** Auto configuration classes, @Configuration annotation ile işaretlenmiş class'lardır. Conditional annotation'lar ile kontrol edilir.

**Conditional Evaluation:** Conditional evaluation, @ConditionalOnClass, @ConditionalOnMissingBean gibi annotation'lar ile yapılır. Condition'lar evaluate edilir.

**Configuration Loading:** Configuration loading, spring.factories file'ından auto configuration class'ları load eder. Priority-based loading sağlar.

**Best Practices:** @EnableAutoConfiguration, main application class'ında kullanılmalıdır. Custom auto configuration'lar için ayrı annotation'lar oluşturulmalıdır.

**Use Cases:** @EnableAutoConfiguration, Spring Boot application'ları için kullanılır.

**Performance:** @EnableAutoConfiguration, startup time'ı optimize eder. Conditional evaluation overhead'i vardır.

**Monitoring:** @EnableAutoConfiguration, monitoring gerektirir. Auto configuration loading time, conditional evaluation success rate monitor edilmelidir.

#### 3. Starter dependency'lerin rolü nedir?

**Cevap:**
Starter dependency'ler, Spring Boot'da related dependency'leri gruplandırır. Single dependency ile multiple library'leri include eder. Dependency management sağlar.

**Temel Rol:** Starter dependency'ler, common use case'ler için gerekli dependency'leri paketler. Version compatibility sağlar. Dependency conflict'leri önler.

**Dependency Grouping:** Dependency grouping, related library'leri tek dependency altında toplar. spring-boot-starter-web, spring-boot-starter-data-jpa gibi starter'lar vardır.

**Version Management:** Version management, Spring Boot parent POM'u ile yapılır. Compatible version'lar otomatik olarak seçilir. Version conflict'leri önlenir.

**Auto Configuration:** Auto configuration, starter dependency'ler ile birlikte gelir. Starter'lar auto configuration class'larını include eder. Automatic configuration sağlar.

**Best Practices:** Starter dependency'ler, Spring Boot'da tercih edilmelidir. Custom starter'lar oluşturulabilir. Version compatibility sağlanmalıdır.

**Use Cases:** Starter dependency'ler, web application'lar, data access, security için kullanılır.

**Performance:** Starter dependency'ler, dependency resolution time'ı optimize eder. Version conflict resolution overhead'i vardır.

**Monitoring:** Starter dependency'ler, monitoring gerektirir. Dependency resolution time, version conflict rate monitor edilmelidir.

#### 4. application.properties vs application.yml farkı nedir?

**Cevap:**
application.properties ve application.yml, Spring Boot'da configuration file format'larıdır. Her ikisi de aynı configuration'ları sağlar, farklı syntax kullanır.

**Temel Fark:** application.properties, key-value format'ında yazılır. application.yml, YAML format'ında yazılır. Syntax farkı vardır.

**Properties Format:** Properties format, key=value syntax'ı kullanır. Nested property'ler için dot notation kullanılır. Simple format'tır.

**YAML Format:** YAML format, hierarchical structure sağlar. Indentation-based syntax kullanır. Human-readable format'tır.

**Nested Properties:** Nested properties, properties'te dot notation ile yazılır. YAML'de indentation ile yazılır. Structure farkı vardır.

**Comments:** Comments, properties'te # ile yazılır. YAML'de # ile yazılır. Comment support aynıdır.

**Best Practices:** YAML format, complex configuration'lar için tercih edilmelidir. Properties format, simple configuration'lar için kullanılabilir.

**Use Cases:** application.properties, simple configuration'lar için kullanılır. application.yml, complex configuration'lar için kullanılır.

**Performance:** Her iki format da aynı performance'a sahiptir. Configuration loading time aynıdır.

**Monitoring:** Her iki format da aynı monitoring gerektirir. Configuration loading success rate monitor edilmelidir.

#### 5. Profile'ların kullanım alanları nelerdir?

**Cevap:**
Profile'lar, Spring Boot'da environment-specific configuration'ları sağlar. Development, test, production gibi environment'lar için farklı configuration'lar kullanılır.

**Temel Kullanım:** Profile'lar, @Profile annotation ile method'ları işaretler. @ActiveProfiles annotation ile profile'lar activate edilir. Environment-specific configuration sağlar.

**Environment Separation:** Environment separation, development, test, production environment'ları için farklı configuration'lar sağlar. Database URL, logging level gibi configuration'lar değişir.

**Configuration Files:** Configuration files, application-{profile}.properties format'ında yazılır. Profile-specific configuration'lar sağlar. Override mechanism sağlar.

**Profile Activation:** Profile activation, --spring.profiles.active parameter ile yapılır. Environment variable, system property ile de yapılabilir. Runtime activation sağlar.

**Default Profile:** Default profile, profile belirtilmediğinde kullanılır. application.properties default configuration sağlar. Fallback mechanism sağlar.

**Best Practices:** Profile'lar, environment-specific configuration'lar için kullanılmalıdır. Clear profile naming convention kullanılmalıdır.

**Use Cases:** Profile'lar, multi-environment deployment, testing için kullanılır.

**Performance:** Profile'lar, minimal performance overhead yaratır. Profile evaluation overhead'i vardır.

**Monitoring:** Profile'lar, monitoring gerektirir. Profile activation success rate, configuration loading time monitor edilmelidir.

### Orta Seviye

#### 6. Custom auto configuration nasıl oluşturulur?

**Cevap:**
Custom auto configuration, Spring Boot'da custom configuration'ları otomatik olarak enable etmek için kullanılır. @Configuration, @Conditional annotation'ları kullanılır.

**Temel Oluşturma:** Custom auto configuration, @Configuration class'ı oluşturarak yapılır. @Conditional annotation'lar ile condition'lar define edilir. Auto configuration logic sağlar.

**@Configuration Class:** @Configuration class, auto configuration logic'ini içerir. @Bean method'ları ile bean'ler define edilir. Configuration sağlar.

**@Conditional Annotation:** @Conditional annotation, condition'ları define eder. @ConditionalOnClass, @ConditionalOnMissingBean gibi annotation'lar kullanılır. Conditional evaluation sağlar.

**spring.factories:** spring.factories file'ı, auto configuration class'larını register eder. org.springframework.boot.autoconfigure.EnableAutoConfiguration key'i kullanılır. Auto configuration registration sağlar.

**@AutoConfigureOrder:** @AutoConfigureOrder annotation, auto configuration'ların sırasını belirler. Priority-based configuration sağlar. Order management sağlar.

**Best Practices:** Custom auto configuration, reusable configuration'lar için kullanılmalıdır. Clear condition'lar define edilmelidir.

**Use Cases:** Custom auto configuration, custom starter'lar, library integration için kullanılır.

**Performance:** Custom auto configuration, startup time'ı optimize eder. Conditional evaluation overhead'i vardır.

**Monitoring:** Custom auto configuration, monitoring gerektirir. Auto configuration loading time, conditional evaluation success rate monitor edilmelidir.

#### 7. Conditional annotation'ları nasıl çalışır?

**Cevap:**
Conditional annotation'lar, Spring Boot'da condition'ları evaluate ederek configuration'ların enable edilip edilmeyeceğini belirler. @Conditional interface'ini implement eder.

**Temel Çalışma:** Conditional annotation'lar, condition'ları evaluate eder. Condition true ise configuration enable edilir. Condition false ise configuration skip edilir.

**@ConditionalOnClass:** @ConditionalOnClass, classpath'te belirli class'ın bulunup bulunmadığını kontrol eder. Class availability check yapar. Dependency-based configuration sağlar.

**@ConditionalOnMissingBean:** @ConditionalOnMissingBean, belirli bean'in mevcut olup olmadığını kontrol eder. Bean existence check yapar. Override prevention sağlar.

**@ConditionalOnProperty:** @ConditionalOnProperty, property'nin belirli value'ya sahip olup olmadığını kontrol eder. Property value check yapar. Configuration-based activation sağlar.

**Custom Condition:** Custom condition, @Conditional interface'ini implement eder. matches() method'u ile custom logic sağlar. Complex condition'lar define eder.

**Best Practices:** Conditional annotation'lar, clear condition'lar için kullanılmalıdır. Performance impact consider edilmelidir.

**Use Cases:** Conditional annotation'lar, environment-specific configuration, feature toggle için kullanılır.

**Performance:** Conditional annotation'lar, evaluation overhead yaratır. Condition complexity performance'ı etkiler.

**Monitoring:** Conditional annotation'lar, monitoring gerektirir. Condition evaluation time, success rate monitor edilmelidir.

#### 8. Configuration properties'lerin binding'i nasıl yapılır?

**Cevap:**
Configuration properties binding, Spring Boot'da external configuration'ları Java object'lerine bind etmek için kullanılır. @ConfigurationProperties annotation kullanılır.

**Temel Binding:** Configuration properties binding, @ConfigurationProperties annotation ile yapılır. Property'ler Java object'lerine otomatik olarak bind edilir. Type conversion sağlar.

**@ConfigurationProperties:** @ConfigurationProperties annotation, class'ı configuration properties olarak işaretler. Property prefix specify edilir. Automatic binding sağlar.

**Property Prefix:** Property prefix, property'lerin hangi prefix ile başlayacağını belirtir. Hierarchical property structure sağlar. Namespace separation sağlar.

**Type Conversion:** Type conversion, property value'larını Java type'larına convert eder. String, Integer, Boolean gibi type'lar desteklenir. Automatic conversion sağlar.

**Validation:** Validation, @Valid annotation ile yapılır. Bean validation annotation'ları kullanılır. Configuration validation sağlar.

**Best Practices:** Configuration properties, type-safe configuration için kullanılmalıdır. Clear property naming convention kullanılmalıdır.

**Use Cases:** Configuration properties, external configuration, environment-specific setting'ler için kullanılır.

**Performance:** Configuration properties, minimal performance overhead yaratır. Property binding overhead'i vardır.

**Monitoring:** Configuration properties, monitoring gerektirir. Property binding success rate, validation failure rate monitor edilmelidir.

#### 9. Auto configuration'ın override edilmesi nasıl yapılır?

**Cevap:**
Auto configuration override, Spring Boot'da default auto configuration'ları custom configuration'larla değiştirmek için kullanılır. @Primary, @ConditionalOnMissingBean annotation'ları kullanılır.

**Temel Override:** Auto configuration override, custom @Configuration class'ları ile yapılır. @Primary annotation ile priority sağlanır. Default configuration'lar değiştirilir.

**@Primary Annotation:** @Primary annotation, bean'in primary bean olduğunu belirtir. Multiple bean'ler arasında priority sağlar. Default bean selection sağlar.

**@ConditionalOnMissingBean:** @ConditionalOnMissingBean annotation, belirli bean'in mevcut olmadığında configuration'ın enable edilmesini sağlar. Override prevention sağlar.

**@AutoConfigureBefore:** @AutoConfigureBefore annotation, auto configuration'ın belirli configuration'dan önce çalışmasını sağlar. Order management sağlar.

**@AutoConfigureAfter:** @AutoConfigureAfter annotation, auto configuration'ın belirli configuration'dan sonra çalışmasını sağlar. Dependency-based ordering sağlar.

**Best Practices:** Auto configuration override, specific requirement'lar için kullanılmalıdır. Clear override strategy sağlanmalıdır.

**Use Cases:** Auto configuration override, custom behavior, third-party integration için kullanılır.

**Performance:** Auto configuration override, minimal performance overhead yaratır. Configuration evaluation overhead'i vardır.

**Monitoring:** Auto configuration override, monitoring gerektirir. Override success rate, configuration loading time monitor edilmelidir.

#### 10. Spring Boot actuator'ın monitoring özellikleri nelerdir?

**Cevap:**
Spring Boot Actuator, application'ların monitoring ve management özelliklerini sağlar. Health check, metrics, endpoint'ler sağlar. Production-ready monitoring sağlar.

**Temel Özellikler:** Spring Boot Actuator, health check, metrics, info, environment endpoint'lerini sağlar. REST API ile monitoring sağlar. JMX support sağlar.

**Health Check:** Health check, application'ın sağlık durumunu kontrol eder. Database, disk space, custom health indicator'lar sağlar. UP/DOWN status döner.

**Metrics:** Metrics, application'ın performance metric'lerini sağlar. JVM metrics, HTTP request metrics, custom metrics sağlar. Micrometer integration sağlar.

**Info Endpoint:** Info endpoint, application'ın bilgilerini sağlar. Version, build info, custom info sağlar. Application metadata sağlar.

**Environment Endpoint:** Environment endpoint, application'ın environment bilgilerini sağlar. Configuration properties, system properties sağlar. Configuration inspection sağlar.

**Best Practices:** Spring Boot Actuator, production environment'lar için kullanılmalıdır. Security configuration sağlanmalıdır.

**Use Cases:** Spring Boot Actuator, application monitoring, health checking, performance monitoring için kullanılır.

**Performance:** Spring Boot Actuator, minimal performance overhead yaratır. Endpoint access overhead'i vardır.

**Monitoring:** Spring Boot Actuator, monitoring gerektirir. Endpoint response time, health check success rate monitor edilmelidir.

---

## RESTful Web Services

### Temel Seviye

#### 1. REST'in temel prensipleri nelerdir?

**Cevap:**
REST (Representational State Transfer), web service'ler için architectural style'dır. HTTP protocol'ünü kullanarak stateless, client-server communication sağlar.

**Temel Prensipler:** REST, stateless, client-server, cacheable, uniform interface, layered system, code on demand prensiplerine dayanır.

**Stateless:** Stateless, her request'in kendi context'ini taşıması gerektiğini belirtir. Server'da client state store edilmez. Scalability sağlar.

**Client-Server:** Client-Server, client ve server'ın independent olarak geliştirilebileceğini belirtir. Separation of concern sağlar. Technology independence sağlar.

**Cacheable:** Cacheable, response'ların cache'lenebileceğini belirtir. Performance optimization sağlar. Network traffic azalır.

**Uniform Interface:** Uniform Interface, standard HTTP method'ları kullanır. GET, POST, PUT, DELETE method'ları resource operation'ları için kullanılır.

**Resource-Based:** Resource-based, her şeyin resource olarak treat edilmesini belirtir. URL'ler resource'ları identify eder. Resource representation sağlar.

**Best Practices:** REST, HTTP standard'larına uygun kullanılmalıdır. Proper HTTP status code'ları kullanılmalıdır.

**Use Cases:** REST, web API'lar, microservice communication için kullanılır.

**Performance:** REST, HTTP caching ile performance optimization sağlar. Stateless nature scalability sağlar.

**Monitoring:** REST, HTTP monitoring tool'ları ile monitor edilir. Response time, status code distribution monitor edilmelidir.

#### 2. @RestController ve @Controller arasındaki fark nedir?

**Cevap:**
@RestController ve @Controller, Spring MVC'de farklı controller type'larıdır. @RestController, @Controller + @ResponseBody combination'ıdır.

**Temel Fark:** @Controller, view rendering için kullanılır. @RestController, JSON/XML response döner. Response type farkı vardır.

**@Controller:** @Controller, traditional MVC controller'dır. View name return eder. ViewResolver ile view render edilir. HTML response sağlar.

**@RestController:** @RestController, REST API controller'dır. Object return eder. HttpMessageConverter ile JSON/XML'e convert edilir. Data response sağlar.

**@ResponseBody:** @ResponseBody annotation, method'ların return value'sunu HTTP response body'sine yazar. JSON/XML serialization sağlar.

**View Resolution:** View resolution, @Controller'da ViewResolver ile yapılır. @RestController'da gerekli değildir. Direct response döner.

**Content Type:** Content type, @Controller'da HTML döner. @RestController'da JSON/XML döner. Content-Type header farklıdır.

**Best Practices:** @RestController, REST API'lar için kullanılmalıdır. @Controller, web page'ler için kullanılmalıdır.

**Use Cases:** @RestController, REST API endpoint'leri için kullanılır. @Controller, web page controller'ları için kullanılır.

**Performance:** @RestController, JSON serialization overhead'i vardır. @Controller, view rendering overhead'i vardır.

**Monitoring:** Her iki controller type da monitoring gerektirir. Response time, error rate monitor edilmelidir.

#### 3. HTTP status code'larının kullanımı nasıldır?

**Cevap:**
HTTP status code'ları, HTTP response'un durumunu belirtir. 1xx, 2xx, 3xx, 4xx, 5xx kategorilerinde organize edilir. Client'a response durumu hakkında bilgi sağlar.

**Temel Kategoriler:** HTTP status code'ları, informational, success, redirection, client error, server error kategorilerinde organize edilir.

**2xx Success:** 2xx status code'ları, successful operation'ları belirtir. 200 OK, 201 Created, 204 No Content gibi code'lar vardır. Success response sağlar.

**4xx Client Error:** 4xx status code'ları, client error'ları belirtir. 400 Bad Request, 401 Unauthorized, 404 Not Found gibi code'lar vardır. Client error sağlar.

**5xx Server Error:** 5xx status code'ları, server error'ları belirtir. 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable gibi code'lar vardır. Server error sağlar.

**REST API Usage:** REST API'da, proper status code'lar kullanılmalıdır. Resource creation'da 201, successful retrieval'da 200, deletion'da 204 kullanılır.

**Error Handling:** Error handling, appropriate status code'lar ile yapılır. Validation error'da 400, authentication error'da 401 kullanılır.

**Best Practices:** HTTP status code'ları, standard convention'lara uygun kullanılmalıdır. Consistent status code usage sağlanmalıdır.

**Use Cases:** HTTP status code'ları, API response, error handling için kullanılır.

**Performance:** HTTP status code'ları, minimal performance overhead yaratır. Response header overhead'i vardır.

**Monitoring:** HTTP status code'ları, monitoring gerektirir. Status code distribution, error rate monitor edilmelidir.

#### 4. Request mapping annotation'larının kullanımı nasıldır?

**Cevap:**
Request mapping annotation'ları, Spring MVC'de HTTP request'leri method'lara map etmek için kullanılır. @RequestMapping, @GetMapping, @PostMapping gibi annotation'lar vardır.

**Temel Kullanım:** Request mapping annotation'ları, URL pattern'leri method'lara bind eder. HTTP method, path, parameter'lar specify edilebilir. Request routing sağlar.

**@RequestMapping:** @RequestMapping, generic request mapping annotation'ıdır. HTTP method, path, parameter'lar specify edilebilir. Flexible mapping sağlar.

**@GetMapping:** @GetMapping, GET request'leri için kullanılır. @RequestMapping(method = RequestMethod.GET) kısayoludur. Read operation'lar için kullanılır.

**@PostMapping:** @PostMapping, POST request'leri için kullanılır. @RequestMapping(method = RequestMethod.POST) kısayoludur. Create operation'lar için kullanılır.

**@PutMapping:** @PutMapping, PUT request'leri için kullanılır. @RequestMapping(method = RequestMethod.PUT) kısayoludur. Update operation'lar için kullanılır.

**@DeleteMapping:** @DeleteMapping, DELETE request'leri için kullanılır. @RequestMapping(method = RequestMethod.DELETE) kısayoludur. Delete operation'lar için kullanılır.

**Path Variables:** Path variables, @PathVariable annotation ile alınır. URL'deki dynamic segment'ler extract edilir. Resource identification sağlar.

**Request Parameters:** Request parameters, @RequestParam annotation ile alınır. Query parameter'lar extract edilir. Optional parameter'lar sağlar.

**Best Practices:** Request mapping annotation'ları, RESTful URL pattern'leri için kullanılmalıdır. HTTP method'ları doğru kullanılmalıdır.

**Use Cases:** Request mapping annotation'ları, REST API endpoint'leri, web controller'ları için kullanılır.

**Performance:** Request mapping annotation'ları, minimal performance overhead yaratır. URL matching overhead'i vardır.

**Monitoring:** Request mapping annotation'ları, monitoring gerektirir. Endpoint access rate, response time monitor edilmelidir.

#### 5. Request/Response body'lerin serialization'ı nasıl çalışır?

**Cevap:**
Request/Response body serialization, Java object'lerini JSON/XML format'ına convert etme işlemidir. HttpMessageConverter interface'i kullanılır. Automatic conversion sağlar.

**Temel Çalışma:** Serialization, HttpMessageConverter'lar ile yapılır. Jackson, Gson gibi library'ler kullanılır. Object'ler JSON/XML'e convert edilir.

**HttpMessageConverter:** HttpMessageConverter, HTTP message'ları Java object'lerine convert eder. Request body'den object'e, object'ten response body'ye conversion sağlar.

**Jackson Integration:** Jackson integration, JSON serialization/deserialization sağlar. ObjectMapper kullanılır. Annotation-based configuration sağlar.

**@RequestBody:** @RequestBody annotation, request body'yi Java object'ine convert eder. JSON/XML'den object'e deserialization sağlar. Automatic binding sağlar.

**@ResponseBody:** @ResponseBody annotation, Java object'ini response body'ye convert eder. Object'ten JSON/XML'e serialization sağlar. Automatic conversion sağlar.

**Content Negotiation:** Content negotiation, client'ın accept header'ına göre response format'ını belirler. JSON, XML format'ları destekler. Flexible response sağlar.

**Best Practices:** Serialization, proper annotation'lar ile yapılmalıdır. Error handling sağlanmalıdır. Performance consider edilmelidir.

**Use Cases:** Serialization, REST API, data exchange için kullanılır.

**Performance:** Serialization, CPU intensive operation'dır. Object conversion overhead'i vardır.

**Monitoring:** Serialization, monitoring gerektirir. Serialization time, error rate monitor edilmelidir.

### Orta Seviye

#### 6. Content negotiation nasıl yapılır?

**Cevap:**
Content negotiation, client'ın request ettiği content type'a göre response format'ını belirleme işlemidir. Accept header, Content-Type header kullanılır.

**Temel Çalışma:** Content negotiation, HttpMessageConverter'lar ile yapılır. Accept header'ına göre appropriate converter seçilir. Multiple format support sağlar.

**Accept Header:** Accept header, client'ın hangi content type'ları kabul ettiğini belirtir. application/json, application/xml gibi type'lar specify edilir.

**Content-Type Header:** Content-Type header, request/response body'nin content type'ını belirtir. MIME type specify edilir. Format identification sağlar.

**HttpMessageConverter:** HttpMessageConverter, content type'a göre appropriate converter seçer. JSON, XML, plain text format'ları destekler. Automatic selection sağlar.

**@RequestMapping:** @RequestMapping, produces ve consumes attribute'ları ile content type specify eder. Specific format requirement'ları sağlar.

**Best Practices:** Content negotiation, standard MIME type'lar kullanmalıdır. Fallback mechanism sağlanmalıdır.

**Use Cases:** Content negotiation, multi-format API'lar için kullanılır.

**Performance:** Content negotiation, minimal performance overhead yaratır. Converter selection overhead'i vardır.

**Monitoring:** Content negotiation, monitoring gerektirir. Format distribution, conversion success rate monitor edilmelidir.

#### 7. API versioning stratejileri nelerdir?

**Cevap:**
API versioning, API'ların farklı version'larını manage etme stratejisidir. URL path, header, query parameter gibi method'lar kullanılır. Backward compatibility sağlar.

**Temel Stratejiler:** API versioning, URL path versioning, header versioning, query parameter versioning stratejileri kullanılır. Her stratejinin avantaj/dezavantaj'ları vardır.

**URL Path Versioning:** URL path versioning, /api/v1/users gibi URL'lerde version specify eder. Clear version identification sağlar. URL structure değişir.

**Header Versioning:** Header versioning, Accept header'da version specify eder. URL structure korunur. Header-based versioning sağlar.

**Query Parameter Versioning:** Query parameter versioning, ?version=1 gibi query parameter'da version specify eder. URL structure korunur. Parameter-based versioning sağlar.

**Version Management:** Version management, multiple version'ları parallel olarak support eder. Deprecation strategy sağlar. Migration path sağlar.

**Best Practices:** API versioning, clear versioning strategy kullanmalıdır. Deprecation timeline sağlanmalıdır.

**Use Cases:** API versioning, long-term API maintenance için kullanılır.

**Performance:** API versioning, minimal performance overhead yaratır. Version routing overhead'i vardır.

**Monitoring:** API versioning, monitoring gerektirir. Version usage distribution, deprecation rate monitor edilmelidir.

#### 8. Error handling ve exception mapping nasıl yapılır?

**Cevap:**
Error handling ve exception mapping, Spring MVC'de exception'ları HTTP response'lara map etme işlemidir. @ControllerAdvice, @ExceptionHandler annotation'ları kullanılır.

**Temel Çalışma:** Error handling, @ControllerAdvice class'ları ile global exception handling sağlar. @ExceptionHandler method'ları ile specific exception'lar handle edilir.

**@ControllerAdvice:** @ControllerAdvice annotation, global exception handling sağlar. Multiple controller'lar için exception handling sağlar. Centralized error handling sağlar.

**@ExceptionHandler:** @ExceptionHandler annotation, specific exception type'ları handle eder. Exception'ı HTTP response'a map eder. Custom error response sağlar.

**ResponseEntity:** ResponseEntity, HTTP status code ve body ile response oluşturur. Error response structure sağlar. Consistent error format sağlar.

**Error Response:** Error response, error code, message, timestamp gibi field'lar içerir. Standardized error format sağlar. Client-friendly error sağlar.

**Best Practices:** Error handling, consistent error format kullanmalıdır. Security information expose edilmemelidir.

**Use Cases:** Error handling, API error management için kullanılır.

**Performance:** Error handling, minimal performance overhead yaratır. Exception processing overhead'i vardır.

**Monitoring:** Error handling, monitoring gerektirir. Error rate, exception type distribution monitor edilmelidir.

#### 9. API documentation nasıl oluşturulur?

**Cevap:**
API documentation, API'ların kullanımını açıklayan dokümantasyondur. Swagger/OpenAPI, SpringDoc gibi tool'lar kullanılır. Interactive documentation sağlar.

**Temel Oluşturma:** API documentation, annotation'lar ile generate edilir. @ApiOperation, @ApiParam gibi annotation'lar kullanılır. Automatic documentation generation sağlar.

**Swagger/OpenAPI:** Swagger/OpenAPI, API documentation standard'ıdır. JSON/YAML format'ında API specification sağlar. Interactive documentation sağlar.

**SpringDoc:** SpringDoc, Spring Boot ile Swagger integration sağlar. Automatic API documentation generation sağlar. OpenAPI 3.0 support sağlar.

**@ApiOperation:** @ApiOperation annotation, API endpoint'lerinin açıklamasını sağlar. Method description, parameter'lar specify edilir. Documentation enhancement sağlar.

**@ApiParam:** @ApiParam annotation, parameter'ların açıklamasını sağlar. Parameter description, example value'lar specify edilir. Parameter documentation sağlar.

**Best Practices:** API documentation, clear ve comprehensive olmalıdır. Example'lar sağlanmalıdır.

**Use Cases:** API documentation, API consumer'lar için kullanılır.

**Performance:** API documentation, minimal performance overhead yaratır. Documentation generation overhead'i vardır.

**Monitoring:** API documentation, monitoring gerektirir. Documentation access rate, API usage rate monitor edilmelidir.

#### 10. Rate limiting nasıl implement edilir?

**Cevap:**
Rate limiting, API'ların kullanımını sınırlama işlemidir. Request rate'ini kontrol eder. Abuse prevention, resource protection sağlar.

**Temel Implementasyon:** Rate limiting, interceptor, filter, annotation-based approach'lar kullanılır. Request count, time window kontrol edilir. Throttling sağlar.

**Token Bucket:** Token bucket algorithm, rate limiting için kullanılır. Token'lar bucket'da store edilir. Request'ler token consume eder. Burst traffic handle eder.

**Sliding Window:** Sliding window algorithm, time window'da request count'u track eder. Moving window ile rate limit sağlar. Smooth rate limiting sağlar.

**Redis Integration:** Redis integration, distributed rate limiting sağlar. Multiple instance'lar arasında rate limit share edilir. Scalable rate limiting sağlar.

**@RateLimit:** @RateLimit annotation, method level rate limiting sağlar. Custom rate limit rule'ları define eder. Fine-grained control sağlar.

**Best Practices:** Rate limiting, appropriate rate limit value'ları kullanmalıdır. Error response sağlanmalıdır.

**Use Cases:** Rate limiting, API protection, abuse prevention için kullanılır.

**Performance:** Rate limiting, minimal performance overhead yaratır. Rate limit check overhead'i vardır.

**Monitoring:** Rate limiting, monitoring gerektirir. Rate limit hit rate, throttled request count monitor edilmelidir.

---

## Microservices

### Temel Seviye
1. Microservices architecture'ın avantajları nelerdir?
2. Service discovery'ın amacı nedir?
3. API Gateway'in rolü nedir?
4. Circuit breaker pattern'ının işlevi nedir?
5. Distributed logging nasıl yapılır?

### Orta Seviye
6. Service mesh'in faydaları nelerdir?
7. Distributed transaction'lar nasıl yönetilir?
8. Event-driven architecture'ın özellikleri nelerdir?
9. Microservices'lerin testing stratejileri nelerdir?
10. Container orchestration'ın önemi nedir?

---

## Design Patterns

### Temel Seviye
1. Singleton pattern'ının Spring'deki implementasyonu nasıldır?
2. Factory pattern'ının kullanım alanları nelerdir?
3. Observer pattern'ının Spring Events'daki kullanımı nasıldır?
4. Strategy pattern'ının dependency injection'daki rolü nedir?
5. Template method pattern'ının Spring'deki örnekleri nelerdir?

### Orta Seviye
6. Decorator pattern'ının AOP'daki kullanımı nasıldır?
7. Proxy pattern'ının Spring'deki implementasyonu nasıldır?
8. Command pattern'ının Spring MVC'daki kullanımı nasıldır?
9. Adapter pattern'ının legacy system integration'daki rolü nedir?
10. Builder pattern'ının configuration'da kullanımı nasıldır?

---

## Database & ORM

### Temel Seviye
1. ACID properties'lerin anlamı nedir?
2. Normalization'ın faydaları nelerdir?
3. Index'lerin performans etkisi nasıldır?
4. Connection pooling'ın amacı nedir?
5. Transaction isolation level'ları nelerdir?

### Orta Seviye
6. Database sharding stratejileri nelerdir?
7. Read replica'ların kullanım alanları nelerdir?
8. Database migration'ların best practice'leri nelerdir?
9. Query optimization teknikleri nelerdir?
10. Database monitoring ve profiling nasıl yapılır?

---

## Testing

### Temel Seviye
1. Unit test, Integration test ve E2E test arasındaki farklar nelerdir?
2. @SpringBootTest annotation'ının kullanımı nasıldır?
3. Mockito'nun temel kullanımı nasıldır?
4. Test slicing'ın faydaları nelerdir?
5. @MockBean ve @SpyBean arasındaki fark nedir?

### Orta Seviye
6. TestContainers'ın kullanım alanları nelerdir?
7. Contract testing'in amacı nedir?
8. Performance testing stratejileri nelerdir?
9. Test data management nasıl yapılır?
10. Continuous integration'da testing nasıl entegre edilir?

---

## Performance & Optimization

### Temel Seviye
1. JVM tuning parametreleri nelerdir?
2. Memory profiling nasıl yapılır?
3. Database query optimization teknikleri nelerdir?
4. Caching stratejileri nelerdir?
5. Connection pooling'ın performans etkisi nasıldır?

### Orta Seviye
6. Application profiling araçları nelerdir?
7. Load balancing stratejileri nelerdir?
8. CDN kullanımının faydaları nelerdir?
9. Asynchronous processing'in performans etkisi nasıldır?
10. Monitoring ve alerting nasıl kurulur?

---

## DevOps & Deployment

### Temel Seviye
1. Docker containerization'ın faydaları nelerdir?
2. CI/CD pipeline'ının aşamaları nelerdir?
3. Environment configuration management nasıl yapılır?
4. Logging stratejileri nelerdir?
5. Health check'lerin önemi nedir?

### Orta Seviye
6. Kubernetes orchestration'ın avantajları nelerdir?
7. Blue-green deployment stratejisi nasıl uygulanır?
8. Infrastructure as Code'ın faydaları nelerdir?
9. Monitoring ve observability nasıl sağlanır?
10. Disaster recovery plan'ı nasıl oluşturulur?

---

## Sonuç

Bu mülakat soruları, Java Spring Boot geliştiricilerinin karşılaşabileceği temel ve orta seviye konuları kapsamaktadır. Her konu başlığı altında hem teorik bilgi hem de pratik uygulama becerilerini test eden sorular bulunmaktadır. Mülakat sürecinde bu sorulara verilen cevaplar, adayın teknik derinliğini ve pratik deneyimini değerlendirmek için kullanılabilir.
