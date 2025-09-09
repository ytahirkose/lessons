## BÖLÜM 2: TYPESCRIPT - TİP GÜVENLİ JAVASCRIPT

**TypeScript Nedir?**

TypeScript, Microsoft tarafından geliştirilen, JavaScript'in üst kümesi olan açık kaynaklı bir programlama dilidir. TypeScript, JavaScript'e statik tip kontrolü, sınıf tabanlı nesne yönelimli programlama ve diğer modern programlama özelliklerini ekler. TypeScript kodu, JavaScript'e derlenir ve herhangi bir JavaScript çalıştırma ortamında çalışabilir.

**TypeScript Ne Demek?**

TypeScript, "JavaScript + Types" anlamına gelir. JavaScript'in dinamik tip sisteminin yarattığı sorunları çözmek için geliştirilmiştir. TypeScript, JavaScript'in tüm özelliklerini destekler ve üzerine statik tip sistemi ekler.

**Neden İhtiyaç Var?**

JavaScript'in dinamik tip sistemi, büyük projelerde ciddi sorunlar yaratır:

1. **Runtime Hatalar**: Tip hataları sadece çalışma zamanında keşfedilir
2. **Kod Organizasyonu**: Büyük projelerde kod organizasyonu zorlaşır
3. **Refactoring Zorluğu**: Kod yeniden düzenleme riskli hale gelir
4. **Takım Çalışması**: Farklı geliştiriciler arasında iletişim sorunları
5. **IDE Desteği**: Gelişmiş IDE özellikleri sınırlı kalır

**Ne İşe Yarar?**

TypeScript, JavaScript geliştirmeyi daha güvenli ve verimli hale getirir:

1. **Erken Hata Tespiti**: Hataları derleme zamanında yakalar
2. **Daha İyi IDE Desteği**: IntelliSense, otomatik tamamlama, refactoring
3. **Kod Dokümantasyonu**: Tipler kodun kendini belgeler
4. **Güvenli Refactoring**: Kod değişikliklerinde güvenlik
5. **Takım Çalışması**: Daha iyi işbirliği ve iletişim

**TypeScript'in Temel Özellikleri:**

1. **Static Type Checking - Statik Tip Kontrolü**: Derleme zamanında tip kontrolü
```typescript
// TypeScript'te tip kontrolü
function greet(name: string): string {
    return `Hello, ${name}!`;
}

// Hata: Argument of type 'number' is not assignable to parameter of type 'string'
// greet(123); // TypeScript hatası

// Doğru kullanım
greet("John"); // "Hello, John!"
```

2. **Object-Oriented Programming - Nesne Yönelimli Programlama**: Sınıf tabanlı programlama
```typescript
// TypeScript'te sınıf tabanlı OOP
class Person {
    private name: string;
    private age: number;
    
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
    
    public greet(): string {
        return `Hello, I'm ${this.name} and I'm ${this.age} years old`;
    }
    
    public getAge(): number {
        return this.age;
    }
}

const person = new Person("John", 30);
console.log(person.greet()); // "Hello, I'm John and I'm 30 years old"
```

3. **Interface Support - Arayüz Desteği**: Yapısal tip tanımlama
```typescript
// TypeScript'te interface
interface User {
    id: number;
    name: string;
    email: string;
    isActive: boolean;
}

interface UserService {
    getUser(id: number): User;
    createUser(userData: Omit<User, 'id'>): User;
    updateUser(id: number, updates: Partial<User>): User;
}

// Interface implementasyonu
class DatabaseUserService implements UserService {
    getUser(id: number): User {
        // Database'den kullanıcı getir
        return { id, name: "John", email: "john@example.com", isActive: true };
    }
    
    createUser(userData: Omit<User, 'id'>): User {
        // Yeni kullanıcı oluştur
        return { id: Math.random(), ...userData };
    }
    
    updateUser(id: number, updates: Partial<User>): User {
        // Kullanıcıyı güncelle
        return { id, name: "Updated", email: "updated@example.com", isActive: true, ...updates };
    }
}
```

4. **Generic Types - Genel Tip Desteği**: Yeniden kullanılabilir tip tanımları
```typescript
// TypeScript'te generic types
interface Repository<T> {
    findById(id: number): T | null;
    findAll(): T[];
    save(entity: T): T;
    delete(id: number): boolean;
}

// Generic class
class UserRepository implements Repository<User> {
    private users: User[] = [];
    
    findById(id: number): User | null {
        return this.users.find(user => user.id === id) || null;
    }
    
    findAll(): User[] {
        return [...this.users];
    }
    
    save(user: User): User {
        const existingIndex = this.users.findIndex(u => u.id === user.id);
        if (existingIndex >= 0) {
            this.users[existingIndex] = user;
        } else {
            this.users.push(user);
        }
        return user;
    }
    
    delete(id: number): boolean {
        const index = this.users.findIndex(user => user.id === id);
        if (index >= 0) {
            this.users.splice(index, 1);
            return true;
        }
        return false;
    }
}

// Generic function
function createArray<T>(length: number, value: T): T[] {
    return Array(length).fill(value);
}

const numbers = createArray<number>(5, 0); // [0, 0, 0, 0, 0]
const strings = createArray<string>(3, "hello"); // ["hello", "hello", "hello"]
```

5. **Decorators - Dekoratör Desteği**: Metaprogramming özellikleri
```typescript
// TypeScript'te decorators
function Log(target: any, propertyName: string, descriptor: PropertyDescriptor) {
    const method = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyName} with arguments:`, args);
        const result = method.apply(this, args);
        console.log(`${propertyName} returned:`, result);
        return result;
    };
}

class Calculator {
    @Log
    add(a: number, b: number): number {
        return a + b;
    }
    
    @Log
    multiply(a: number, b: number): number {
        return a * b;
    }
}

const calc = new Calculator();
calc.add(5, 3); // Logs: Calling add with arguments: [5, 3] and add returned: 8
```

**TypeScript'in Faydaları:**

1. **Early Error Detection - Erken Hata Tespiti**: Hataları derleme zamanında yakalama
```typescript
// JavaScript'te runtime hatası
function calculateArea(width, height) {
    return width * height;
}

// calculateArea("5", "10"); // "510" (string concatenation)

// TypeScript'te compile-time hatası
function calculateArea(width: number, height: number): number {
    return width * height;
}

// calculateArea("5", "10"); // TypeScript hatası: Argument of type 'string' is not assignable to parameter of type 'number'
```

2. **Better Code Documentation - Daha İyi Kod Dokümantasyonu**: Kodun kendini belgelemesi
```typescript
// TypeScript'te self-documenting code
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
    timestamp: Date;
}

interface User {
    id: number;
    name: string;
    email: string;
    profile: {
        avatar: string;
        bio: string;
        preferences: {
            theme: 'light' | 'dark';
            language: string;
            notifications: boolean;
        };
    };
}

// API fonksiyonu
async function fetchUser(id: number): Promise<ApiResponse<User>> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}
```

3. **Enhanced IDE Support - Gelişmiş IDE Desteği**: IntelliSense, otomatik tamamlama
```typescript
// TypeScript'te IDE desteği
interface Product {
    id: number;
    name: string;
    price: number;
    category: string;
    inStock: boolean;
}

const product: Product = {
    id: 1,
    name: "Laptop",
    price: 999.99,
    category: "Electronics",
    inStock: true
};

// IDE otomatik tamamlama önerir
product. // IDE: id, name, price, category, inStock önerir

// Tip güvenliği
product.name = "Desktop"; // ✅ Doğru
// product.name = 123; // ❌ TypeScript hatası
```

**TypeScript'in Kullanım Alanları:**

1. **Large-Scale Applications - Büyük Ölçekli Uygulamalar**: Enterprise projeler
2. **Enterprise Development - Kurumsal Geliştirme**: Büyük şirketlerde kullanım
3. **Team Projects - Takım Projeleri**: Çoklu geliştirici projeleri
4. **Library Development - Kütüphane Geliştirme**: NPM paketleri
5. **Modern Web Development - Modern Web Geliştirme**: React, Vue, Angular
6. **Node.js Applications - Node.js Uygulamaları**: Backend geliştirme
7. **React/Angular/Vue Projects - Modern Framework Projeleri**: Frontend framework'leri

### 2.1 TypeScript'in Tarihçesi ve Amacı - Detaylı Analiz

TypeScript'in doğuşu, modern web geliştirmenin en önemli dönüm noktalarından biridir. Bu dil, sadece bir programlama dili değil, aynı zamanda JavaScript ekosisteminin evrimini temsil eden bir paradigma değişimidir. TypeScript'in tarihçesi, web geliştirmenin karmaşıklığının artması ve büyük ölçekli uygulamaların ihtiyaçlarının karşılanması sürecini yansıtır.

**TypeScript'in Doğuşu - Neden Gerekli Oldu? - Derinlemesine Analiz**

TypeScript'in doğuşu, JavaScript'in sınırlarının aşılması ihtiyacından kaynaklanır. 2010'lu yılların başında, web uygulamaları giderek karmaşıklaşıyor ve JavaScript'in dinamik tip sistemi, büyük projelerde ciddi sorunlar yaratıyordu. Bu sorunlar, sadece teknik zorluklar değil, aynı zamanda geliştirici deneyimi, kod kalitesi ve proje sürdürülebilirliği açısından kritik problemlerdi.

**JavaScript'in Tarihsel Sınırları:**

JavaScript, 1995 yılında Brendan Eich tarafından 10 günde geliştirilmiş bir dildir. Bu hızlı geliştirme süreci, dilin temel tasarımında bazı sınırlamalar yaratmıştır. Özellikle dinamik tip sistemi, küçük projeler için esneklik sağlarken, büyük projeler için ciddi zorluklar yaratmıştır.

**Büyük Projelerde Karşılaşılan Sorunlar:**

**Runtime Hata Problemi:**
JavaScript'in dinamik tip sistemi, tip hatalarının sadece çalışma zamanında keşfedilmesine neden olur. Bu durum, özellikle büyük projelerde ciddi güvenlik ve güvenilirlik sorunları yaratır. Geliştiriciler, kod yazarken tip hatalarını fark edemez ve bu hatalar production ortamında ortaya çıkar.

**Kod Organizasyonu Zorluğu:**
JavaScript'in esnek yapısı, büyük projelerde kod organizasyonunu zorlaştırır. Farklı geliştiriciler, farklı kodlama stilleri kullanabilir ve bu durum kod tutarlılığını bozar. Ayrıca, JavaScript'in modül sistemi eksikliği, kod organizasyonunu daha da zorlaştırır.

**Refactoring Riskleri:**
JavaScript'te kod yeniden düzenleme (refactoring) işlemi, yüksek risk taşır. Tip güvenliği olmadığı için, bir değişiklik yapıldığında, bu değişikliğin etkilediği tüm yerlerin manuel olarak kontrol edilmesi gerekir. Bu durum, refactoring sürecini yavaşlatır ve hata riskini artırır.

**Takım Çalışması Sorunları:**
JavaScript'in esnek yapısı, takım çalışmasında iletişim sorunları yaratır. Farklı geliştiriciler, aynı API'yi farklı şekillerde kullanabilir ve bu durum uyumsuzluklara neden olur. Ayrıca, kod dokümantasyonu eksikliği, yeni takım üyelerinin projeye katılmasını zorlaştırır.

**IDE Desteği Sınırları:**
JavaScript'in dinamik yapısı, IDE'lerin gelişmiş özellikler sunmasını sınırlar. IntelliSense, otomatik tamamlama ve refactoring gibi özellikler, tip bilgisi olmadığı için sınırlı kalır. Bu durum, geliştirici verimliliğini düşürür.

**Microsoft'un Stratejik Yaklaşımı:**

Microsoft, JavaScript'in bu sınırlarını fark etmiş ve bu sorunları çözmek için stratejik bir yaklaşım benimsemiştir. Microsoft'un amacı, JavaScript'i tamamen değiştirmek değil, onu geliştirmek ve güçlendirmektir. Bu yaklaşım, TypeScript'in "JavaScript superset" olarak tasarlanmasına neden olmuştur.

**Anders Hejlsberg'in Vizyonu:**

Anders Hejlsberg, TypeScript projesinin başında, C# ve Delphi deneyiminden yararlanarak, JavaScript için modern bir tip sistemi tasarlamıştır. Hejlsberg'in vizyonu, JavaScript'in esnekliğini korurken, tip güvenliği ve geliştirici deneyimini artırmaktır.

**TypeScript'in Tasarım Prensipleri:**

**JavaScript Superset:**
TypeScript, JavaScript'in tam bir üst kümesidir. Bu prensip, mevcut JavaScript kodunun TypeScript'te çalışmasını sağlar ve geçiş sürecini kolaylaştırır.

**Gradual Typing:**
TypeScript, aşamalı tip eklemeyi destekler. Bu prensip, geliştiricilerin projelerini aşamalı olarak TypeScript'e geçirmesini mümkün kılar.

**Structural Typing:**
TypeScript, yapısal tip sistemini benimser. Bu prensip, nominal typing'den farklı olarak, tip uyumluluğunu yapısal özelliklere göre belirler.

**Type Inference:**
TypeScript, otomatik tip çıkarımı sağlar. Bu prensip, geliştiricilerin her yerde tip belirtmesini gerektirmez.

**Compile-time Checking:**
TypeScript, derleme zamanında tip kontrolü yapar. Bu prensip, runtime hatalarını önler ve geliştirici deneyimini iyileştirir.

**JavaScript'in Sorunları (2010 Öncesi):**

1. **Runtime Hatalar**: Tip hataları sadece çalışma zamanında keşfedilir
```javascript
// JavaScript'te runtime hatası
function calculateTotal(price, tax) {
    return price + (price * tax);
}

// calculateTotal("100", "0.1"); // "1000.1" (string concatenation)
// calculateTotal(100, "0.1"); // "1000.1" (string concatenation)
// calculateTotal(100, 0.1); // 110 (doğru)
```

2. **Kod Organizasyonu**: Büyük projelerde kod organizasyonu zorlaşır
3. **Refactoring Zorluğu**: Kod yeniden düzenleme riskli hale gelir
4. **Takım Çalışması**: Farklı geliştiriciler arasında iletişim sorunları
5. **IDE Desteği**: Gelişmiş IDE özellikleri sınırlı kalır

**Microsoft ve Anders Hejlsberg - TypeScript'in Yaratıcısı**:

**Anders Hejlsberg Kimdir?**
- **Danimarkalı Yazılım Mühendisi**: 1960 doğumlu
- **Microsoft'ta Çalışıyor**: 1996'dan beri Microsoft'ta
- **C# Tasarımcısı**: C# programlama dilinin baş tasarımcısı
- **Delphi Tasarımcısı**: Borland'da Delphi'nin tasarımcısı
- **Turbo Pascal**: Borland'da Turbo Pascal'ın geliştiricisi

**Anders Hejlsberg'in Deneyimi:**
```csharp
// C# deneyimi TypeScript'e yansıdı
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    
    public User(int id, string name, string email)
    {
        Id = id;
        Name = name;
        Email = email;
    }
}
```

**2010 - TypeScript Projesinin Başlangıcı**:

**Microsoft Research'te Başlangıç:**
- **Microsoft Research**: TypeScript projesi Microsoft Research'te başladı
- **JavaScript Sorunları**: Büyük JavaScript projelerindeki tip güvenliği sorunları
- **C# Deneyimi**: Anders Hejlsberg'in C# deneyiminden yararlanma
- **Visual Studio Entegrasyonu**: Microsoft'un IDE'si ile entegrasyon hedefi

**Proje Hedefleri:**
1. **JavaScript Superset**: JavaScript'in üst kümesi olarak tasarlama
2. **Gradual Typing**: Aşamalı tip ekleme
3. **Structural Typing**: Yapısal tip sistemi
4. **Type Inference**: Otomatik tip çıkarımı
5. **Compile-time Checking**: Derleme zamanı tip kontrolü

**2012 - TypeScript'in Doğuşu**:

**Ekim 2012 - TypeScript 0.8:**
- **İlk Sürüm**: TypeScript 0.8 sürümü yayınlandı
- **Açık Kaynak**: Apache 2.0 lisansı ile açık kaynak yapıldı
- **GitHub**: GitHub'da yayınlandı
- **Topluluk Desteği**: Hızlı topluluk benimsemesi

**İlk TypeScript Örneği:**
```typescript
// TypeScript 0.8'de ilk örnek
class Greeter {
    greeting: string;
    
    constructor(message: string) {
        this.greeting = message;
    }
    
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

**2013 - TypeScript 0.9 - Gelişmiş Tip Sistemi**:

**Yeni Özellikler:**
- **Generic Constraints**: Generic tip kısıtlamaları
- **Union Types**: Birleşim tipleri
- **Intersection Types**: Kesişim tipleri
- **Type Aliases**: Tip takma adları

**Generic Constraints Örneği:**
```typescript
// TypeScript 0.9'da generic constraints
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

loggingIdentity("hello"); // ✅ string has length property
loggingIdentity([1, 2, 3]); // ✅ array has length property
// loggingIdentity(123); // ❌ number doesn't have length property
```

**Union Types Örneği:**
```typescript
// TypeScript 0.9'da union types
function formatId(id: string | number): string {
    if (typeof id === "string") {
        return id.toUpperCase();
    } else {
        return id.toString();
    }
}

formatId("abc123"); // "ABC123"
formatId(123); // "123"
```

**2014 - TypeScript 1.0 - Production Ready**:

**Stabil API:**
- **Kararlı API**: Kararlı API yayınlandı
- **Production Ready**: Üretim ortamında kullanıma hazır
- **Visual Studio 2013**: Visual Studio 2013'te tam destek
- **Node.js Desteği**: Node.js projelerinde kullanım

**TypeScript 1.0 Özellikleri:**
```typescript
// TypeScript 1.0'da production-ready özellikler
interface User {
    id: number;
    name: string;
    email: string;
}

class UserService {
    private users: User[] = [];
    
    addUser(user: User): void {
        this.users.push(user);
    }
    
    getUser(id: number): User | undefined {
        return this.users.find(user => user.id === id);
    }
}

const userService = new UserService();
userService.addUser({ id: 1, name: "John", email: "john@example.com" });
```

**2015 - TypeScript 1.5 - ES6 Desteği**:

**ES6 Özellikleri:**
- **ES6 Modules**: ES6 modül desteği
- **Decorators**: Dekoratör desteği (experimental)
- **Destructuring**: Yapı sökme desteği
- **Spread Operator**: Spread operatör desteği

**ES6 Modules Örneği:**
```typescript
// TypeScript 1.5'te ES6 modules
// user.ts
export interface User {
    id: number;
    name: string;
}

export class UserService {
    private users: User[] = [];
    
    addUser(user: User): void {
        this.users.push(user);
    }
}

// app.ts
import { User, UserService } from './user';

const userService = new UserService();
userService.addUser({ id: 1, name: "John" });
```

**2016 - TypeScript 2.0 - Güçlü Tip Sistemi**:

**Yeni Tip Özellikleri:**
- **Nullable Types**: Nullable tip sistemi
- **Control Flow Analysis**: Kontrol akışı analizi
- **Tagged Union Types**: Etiketli birleşim tipleri
- **Never Type**: Never tipi
- **Readonly Properties**: Salt okunur özellikler

**Nullable Types Örneği:**
```typescript
// TypeScript 2.0'da nullable types
function processUser(user: User | null): string {
    if (user === null) {
        return "No user provided";
    }
    return `Processing user: ${user.name}`;
}

processUser({ id: 1, name: "John" }); // "Processing user: John"
processUser(null); // "No user provided"
```

**Never Type Örneği:**
```typescript
// TypeScript 2.0'da never type
function throwError(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {
        // Infinite loop
    }
}
```

**2017 - TypeScript 2.4 - Modern JavaScript Desteği**:

**Yeni Özellikler:**
- **Dynamic Import**: Dinamik import desteği
- **String Enums**: String enum desteği
- **Weak Type Detection**: Zayıf tip tespiti
- **Stricter Generics**: Daha sıkı generic kontrolü

**Dynamic Import Örneği:**
```typescript
// TypeScript 2.4'te dynamic import
async function loadModule() {
    const module = await import('./user');
    return module.UserService;
}

// String Enums
enum Color {
    Red = "red",
    Green = "green",
    Blue = "blue"
}

const color: Color = Color.Red; // "red"
```

**2018 - TypeScript 3.0 - Proje Yönetimi**:

**Yeni Özellikler:**
- **Project References**: Proje referansları
- **Tuples in Rest Parameters**: Rest parametrelerde tuple desteği
- **Unknown Type**: Unknown tipi
- **Definite Assignment Assertions**: Kesin atama onayları

**Project References Örneği:**
```json
// tsconfig.json
{
    "compilerOptions": {
        "composite": true,
        "declaration": true
    },
    "references": [
        { "path": "./packages/core" },
        { "path": "./packages/ui" }
    ]
}
```

**Unknown Type Örneği:**
```typescript
// TypeScript 3.0'da unknown type
function processData(data: unknown): string {
    if (typeof data === "string") {
        return data.toUpperCase();
    } else if (typeof data === "number") {
        return data.toString();
    } else {
        return "Unknown data type";
    }
}
```

**TypeScript'in Tasarım Felsefesi:**

1. **JavaScript Superset**: JavaScript'in üst kümesi olarak tasarlandı
2. **Gradual Typing**: Aşamalı tip ekleme
3. **Structural Typing**: Yapısal tip sistemi
4. **Type Inference**: Otomatik tip çıkarımı
5. **Compile-time Checking**: Derleme zamanı tip kontrolü

**TypeScript'in Başarısı:**

TypeScript, JavaScript ekosisteminde büyük başarı elde etti:

- **GitHub'da En Popüler Diller**: En popüler programlama dillerinden biri
- **Büyük Şirketlerde Kullanım**: Google, Microsoft, Facebook, Netflix
- **Modern Framework'lerde Destek**: React, Vue, Angular, Svelte
- **Node.js'te Yaygın Kullanım**: Backend geliştirmede standart
- **Enterprise Projelerde Tercih**: Büyük ölçekli projelerde kullanım

**2019 - TypeScript 3.7 - Modern JavaScript Desteği**:

**Yeni Özellikler:**
- **Optional Chaining**: Opsiyonel zincirleme
- **Nullish Coalescing**: Nullish birleştirme
- **Assertion Functions**: Onay fonksiyonları
- **Uncalled Function Checks**: Çağrılmayan fonksiyon kontrolleri

**Optional Chaining Örneği:**
```typescript
// TypeScript 3.7'de optional chaining
interface User {
    name: string;
    address?: {
        street?: string;
        city?: string;
        country?: string;
    };
    profile?: {
        avatar?: string;
        bio?: string;
    };
}

function getUserCity(user: User): string | undefined {
    // Güvenli erişim
    return user.address?.city;
}

function getUserAvatar(user: User): string | undefined {
    // Nested optional chaining
    return user.profile?.avatar;
}

// Array optional chaining
function getFirstUser(users: User[]): string | undefined {
    return users?.[0]?.name;
}
```

**Nullish Coalescing Örneği:**
```typescript
// TypeScript 3.7'de nullish coalescing
function createUser(name: string, age?: number, city?: string) {
    return {
        name,
        age: age ?? 18, // null veya undefined ise 18 kullan
        city: city ?? "Unknown" // null veya undefined ise "Unknown" kullan
    };
}

// || vs ?? farkı
const value1 = 0 || "default"; // "default" (0 falsy)
const value2 = 0 ?? "default"; // 0 (0 nullish değil)

const value3 = "" || "default"; // "default" ("" falsy)
const value4 = "" ?? "default"; // "" ("" nullish değil)
```

**2020 - TypeScript 4.0 - Tuple ve Performans İyileştirmeleri**:

**Yeni Özellikler:**
- **Variadic Tuple Types**: Değişken tuple tipleri
- **Labeled Tuple Elements**: Etiketli tuple elementleri
- **Class Property Inference**: Sınıf özellik çıkarımı
- **Short-circuiting Assignment**: Kısa devre atama

**Variadic Tuple Types Örneği:**
```typescript
// TypeScript 4.0'da variadic tuple types
type Head<T extends readonly unknown[]> = T extends readonly [infer H, ...unknown[]] ? H : never;
type Tail<T extends readonly unknown[]> = T extends readonly [unknown, ...infer T] ? T : [];

type HeadType = Head<[string, number, boolean]>; // string
type TailType = Tail<[string, number, boolean]>; // [number, boolean]

// Function overloading with variadic tuples
function concat<T extends readonly unknown[], U extends readonly unknown[]>(
    t: T,
    u: U
): [...T, ...U] {
    return [...t, ...u];
}

const result = concat([1, 2], ["a", "b"]); // [1, 2, "a", "b"]
```

**Labeled Tuple Elements Örneği:**
```typescript
// TypeScript 4.0'da labeled tuple elements
type Point = [x: number, y: number];
type User = [id: number, name: string, email: string];

function createPoint(x: number, y: number): Point {
    return [x, y];
}

function createUser(id: number, name: string, email: string): User {
    return [id, name, email];
}

const point: Point = createPoint(10, 20);
const user: User = createUser(1, "John", "john@example.com");
```

**2021 - TypeScript 4.5 - Template Literal Types ve Async İyileştirmeleri**:

**Yeni Özellikler:**
- **Template Literal Types**: Şablon literal tipleri
- **Awaited Type**: Awaited tipi
- **Node.js ESM Support**: Node.js ESM desteği
- **Import Assertions**: Import onayları

**Template Literal Types Örneği:**
```typescript
// TypeScript 4.5'te template literal types
type EventName<T extends string> = `on${Capitalize<T>}`;
type MouseEvent = EventName<"click" | "hover">; // "onClick" | "onHover"

type CSSProperty<T extends string> = `--${T}`;
type CSSVar = CSSProperty<"primary" | "secondary">; // "--primary" | "--secondary"

// API endpoint types
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type ApiEndpoint<T extends string> = `/api/${T}`;
type UserEndpoint = ApiEndpoint<"users" | "posts">; // "/api/users" | "/api/posts"

// Database table names
type TableName<T extends string> = `${T}_table`;
type UserTable = TableName<"user" | "post">; // "user_table" | "post_table"
```

**Awaited Type Örneği:**
```typescript
// TypeScript 4.5'te awaited type
type Awaited<T> = T extends Promise<infer U> ? U : T;

async function fetchUser(id: number): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

type UserPromise = Promise<User>;
type UserType = Awaited<UserPromise>; // User

// Nested promises
type NestedPromise = Promise<Promise<User>>;
type FlattenedUser = Awaited<NestedPromise>; // User
```

**2022 - TypeScript 4.9 - Satisfies Operatörü ve Performans**:

**Yeni Özellikler:**
- **Satisfies Operator**: Satisfies operatörü
- **Unlisted Property Narrowing**: Listelenmemiş özellik daraltma
- **File-watching Performance**: Dosya izleme performansı
- **Better Error Messages**: Daha iyi hata mesajları

**Satisfies Operator Örneği:**
```typescript
// TypeScript 4.9'da satisfies operator
interface Config {
    apiUrl: string;
    timeout: number;
    retries: number;
}

// satisfies ile tip kontrolü
const config = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    retries: 3
} satisfies Config;

// config'in tipi Config olarak korunur
console.log(config.apiUrl); // string
console.log(config.timeout); // number

// satisfies vs as farkı
const config1 = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    retries: 3
} satisfies Config; // Tip kontrolü yapılır

const config2 = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    retries: 3
} as Config; // Tip kontrolü yapılmaz
```

**2023 - TypeScript 5.0 - Decorators ve Modern Özellikler**:

**Yeni Özellikler:**
- **Decorators (Stage 3)**: Dekoratörler (Stage 3)
- **const Type Parameters**: const tip parametreleri
- **extends Constraints on infer**: infer üzerinde extends kısıtlamaları
- **Resolution Mode Flags**: Çözümleme mod bayrakları

**Decorators (Stage 3) Örneği:**
```typescript
// TypeScript 5.0'da decorators (Stage 3)
function Log(target: any, context: ClassFieldDecoratorContext) {
    return function (this: any, value: any) {
        console.log(`Setting ${String(context.name)} to ${value}`);
        return value;
    };
}

function Validate(target: any, context: ClassMethodDecoratorContext) {
    return function (this: any, ...args: any[]) {
        console.log(`Calling ${String(context.name)} with args:`, args);
        const result = target.apply(this, args);
        console.log(`${String(context.name)} returned:`, result);
        return result;
    };
}

class UserService {
    @Log
    private name: string = "";
    
    @Validate
    setName(name: string): void {
        this.name = name;
    }
}
```

**const Type Parameters Örneği:**
```typescript
// TypeScript 5.0'da const type parameters
function createArray<T extends readonly unknown[]>(...items: T): T {
    return items;
}

const array1 = createArray(1, 2, 3); // readonly [1, 2, 3]
const array2 = createArray("a", "b", "c"); // readonly ["a", "b", "c"]

// const assertion ile
const array3 = [1, 2, 3] as const; // readonly [1, 2, 3]
```

**2024-2025 - TypeScript 5.1-5.8 - Gelecek ve Performans**:

**Yeni Özellikler:**
- **Golang Rewrite**: Derleyicinin Go ile yeniden yazılması (experimental)
- **Performance Improvements**: Performans iyileştirmeleri
- **Better Type Inference**: Daha iyi tip çıkarımı
- **Enhanced Error Messages**: Gelişmiş hata mesajları

**Performans İyileştirmeleri:**
```typescript
// TypeScript 5.1+ performans iyileştirmeleri
// Daha hızlı tip kontrolü
// Daha iyi memory kullanımı
// Daha hızlı build süreleri

// Better type inference
function processData<T>(data: T): T {
    // Daha iyi tip çıkarımı
    return data;
}

const result = processData({ name: "John", age: 30 });
// result tipi: { name: string; age: number }
```

**TypeScript'in Geleceği:**

TypeScript'in gelecekteki yönü:

1. **Performans**: Daha hızlı derleme süreleri
2. **Tip Sistemi**: Daha güçlü tip sistemi
3. **JavaScript Uyumluluğu**: JavaScript ile daha iyi uyumluluk
4. **Araç Desteği**: Daha iyi IDE ve araç desteği
5. **Ekosistem**: Daha geniş ekosistem desteği

**İlk Çözümler**:
- **Statik Tip Kontrolü**: Derleme zamanında tip hatalarını yakalama
- **IntelliSense Desteği**: IDE'lerde gelişmiş kod tamamlama
- **Refactoring Güvenliği**: Kod değişikliklerinde tip güvenliği
- **Büyük Projelerde Kod Organizasyonu**: Modüler ve sürdürülebilir kod yapısı
- **JavaScript Ecosystem**: Mevcut JavaScript ekosistemi ile uyumluluk
- **Gradual Adoption**: Aşamalı benimseme
- **Tooling Support**: Gelişmiş araç desteği

### 2.2 TypeScript Temel Kavramlar - Detaylı Analiz

TypeScript'in temel kavramları, modern tip güvenli programlamanın temelini oluşturur. Bu kavramlar, sadece syntax öğrenmek değil, aynı zamanda tip sisteminin felsefesini ve tasarım prensiplerini anlamak için kritik öneme sahiptir. TypeScript'in tip sistemi, functional programming, object-oriented programming ve modern JavaScript özelliklerini harmanlayarak, güçlü ve esnek bir programlama deneyimi sunar.

#### 2.2.1 Tip Sistemi (Type System) - Derinlemesine Analiz

TypeScript'in tip sistemi, modern programlama dillerinin en gelişmiş tip sistemlerinden biridir. Bu sistem, sadece tip kontrolü yapmakla kalmaz, aynı zamanda kod organizasyonu, refactoring güvenliği ve geliştirici deneyimi açısından kritik faydalar sağlar.

**Tip Sisteminin Felsefesi:**

TypeScript'in tip sistemi, "structural typing" prensibini benimser. Bu prensip, nominal typing'den farklı olarak, tip uyumluluğunu yapısal özelliklere göre belirler. Bu yaklaşım, JavaScript'in esnek yapısıyla uyumlu olurken, tip güvenliği sağlar.

**Tip Sisteminin Temel Prensipleri:**

**Type Safety (Tip Güvenliği):**
TypeScript'in tip sistemi, derleme zamanında tip hatalarını yakalar. Bu özellik, runtime hatalarını önler ve kod güvenilirliğini artırır. Tip güvenliği, özellikle büyük projelerde kritik öneme sahiptir.

**Type Inference (Tip Çıkarımı):**
TypeScript, otomatik tip çıkarımı yapar. Bu özellik, geliştiricilerin her yerde tip belirtmesini gerektirmez. Tip çıkarımı, kod yazma hızını artırır ve kod okunabilirliğini iyileştirir.

**Structural Typing (Yapısal Tip Sistemi):**
TypeScript, yapısal tip sistemini kullanır. Bu sistem, tip uyumluluğunu yapısal özelliklere göre belirler. Bu yaklaşım, JavaScript'in esnek yapısıyla uyumlu olurken, tip güvenliği sağlar.

**Gradual Typing (Aşamalı Tip Ekleme):**
TypeScript, aşamalı tip eklemeyi destekler. Bu özellik, geliştiricilerin projelerini aşamalı olarak TypeScript'e geçirmesini mümkün kılar. Aşamalı tip ekleme, geçiş sürecini kolaylaştırır.

**Tip Sisteminin Kategorileri:**

**Primitive Types (İlkel Tipler):**
İlkel tipler, JavaScript'in temel veri tiplerini temsil eder. Bu tipler, string, number, boolean, null, undefined, bigint ve symbol'dir. İlkel tipler, TypeScript'in tip sisteminin temelini oluşturur.

**Object Types (Nesne Tipleri):**
Nesne tipleri, karmaşık veri yapılarını temsil eder. Bu tipler, interface'ler, class'lar, array'ler ve function'lar gibi farklı yapıları içerir. Nesne tipleri, TypeScript'in güçlü tip sisteminin kalbidir.

**Union Types (Birleşim Tipleri):**
Birleşim tipleri, birden fazla tipin birleşimini temsil eder. Bu tipler, "|" operatörü ile oluşturulur. Birleşim tipleri, esnek tip tanımlamaları sağlar.

**Intersection Types (Kesişim Tipleri):**
Kesişim tipleri, birden fazla tipin kesişimini temsil eder. Bu tipler, "&" operatörü ile oluşturulur. Kesişim tipleri, tip composition'ı sağlar.

**Generic Types (Generic Tipler):**
Generic tipler, tip parametreleri kullanarak yeniden kullanılabilir tip tanımlamaları sağlar. Bu tipler, tip güvenliği ile birlikte kod tekrarını azaltır.

**Conditional Types (Koşullu Tipler):**
Koşullu tipler, tip seviyesinde koşullar sağlar. Bu tipler, "extends" ve "?" operatörleri ile oluşturulur. Koşullu tipler, tip seviyesinde programlama yapılmasını sağlar.

**Mapped Types (Eşleştirilmiş Tipler):**
Eşleştirilmiş tipler, mevcut tiplerden yeni tipler oluşturur. Bu tipler, "in" operatörü ile oluşturulur. Eşleştirilmiş tipler, tip transformation'ları sağlar.

**Template Literal Types (Şablon Literal Tipleri):**
Şablon literal tipleri, string template'leri tip seviyesinde destekler. Bu tipler, string manipulation'ları için çok etkilidir.

**Tip Sisteminin Avantajları:**

**Compile-time Error Detection:**
TypeScript'in tip sistemi, hataları derleme zamanında yakalar. Bu özellik, runtime hatalarını önler ve kod güvenilirliğini artırır.

**Better IDE Support:**
TypeScript'in tip sistemi, IDE'lerin gelişmiş özellikler sunmasını sağlar. IntelliSense, otomatik tamamlama ve refactoring gibi özellikler, tip bilgisi sayesinde çok daha etkili çalışır.

**Code Documentation:**
TypeScript'in tip sistemi, kodun kendini belgelemesini sağlar. Tip tanımlamaları, kodun ne yaptığını açıklar ve dokümantasyon ihtiyacını azaltır.

**Refactoring Safety:**
TypeScript'in tip sistemi, refactoring işlemlerini güvenli hale getirir. Tip kontrolü, değişikliklerin etkilediği tüm yerleri otomatik olarak tespit eder.

**Team Collaboration:**
TypeScript'in tip sistemi, takım çalışmasını iyileştirir. Tip tanımlamaları, API'lerin nasıl kullanılacağını açık bir şekilde belirtir.

**Tip Sisteminin Kullanım Senaryoları:**

**Large-scale Applications:**
TypeScript'in tip sistemi, büyük ölçekli uygulamalar için idealdir. Bu uygulamalarda, tip güvenliği ve kod organizasyonu kritik öneme sahiptir.

**Team Development:**
TypeScript'in tip sistemi, takım geliştirme için idealdir. Bu ortamlarda, tip tanımlamaları iletişimi kolaylaştırır ve kod kalitesini artırır.

**Library Development:**
TypeScript'in tip sistemi, kütüphane geliştirme için idealdir. Bu projelerde, tip tanımlamaları API dokümantasyonu sağlar.

**Legacy Code Migration:**
TypeScript'in tip sistemi, legacy kod migrasyonu için idealdir. Aşamalı tip ekleme, geçiş sürecini kolaylaştırır.

**Primitive Types (İlkel Tipler)**:

```typescript
// Temel tipler
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let data: null = null;
let value: undefined = undefined;

// Literal types
let direction: "up" | "down" | "left" | "right" = "up";
let status: 200 | 404 | 500 = 200;

// Template literal types
type EventName<T extends string> = `on${Capitalize<T>}`;
type MouseEvent = EventName<"click" | "hover">; // "onClick" | "onHover"

// BigInt
let bigNumber: bigint = 123456789012345678901234567890n;

// Symbol
let sym: symbol = Symbol("id");
```

**Object Types (Nesne Tipleri)**:

```typescript
// Object type annotation
let person: {
    name: string;
    age: number;
    email?: string; // Optional property
    readonly id: number; // Readonly property
} = {
    name: "John",
    age: 30,
    id: 1
};

// Index signatures
let dictionary: { [key: string]: number } = {
    "apple": 1,
    "banana": 2
};

// Function types
let add: (a: number, b: number) => number = (a, b) => a + b;

// Array types
let numbers: number[] = [1, 2, 3, 4, 5];
let names: Array<string> = ["John", "Jane", "Bob"];

// Tuple types
let coordinate: [number, number] = [10, 20];
let personInfo: [string, number, boolean] = ["John", 30, true];

// Readonly arrays
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // Error: Property 'push' does not exist
```

**Union ve Intersection Types**:

```typescript
// Union types
let id: string | number = "123";
id = 456; // Also valid

// Union with literal types
let theme: "light" | "dark" = "light";

// Intersection types
interface Person {
    name: string;
    age: number;
}

interface Employee {
    employeeId: string;
    department: string;
}

type PersonEmployee = Person & Employee;

let personEmployee: PersonEmployee = {
    name: "John",
    age: 30,
    employeeId: "EMP001",
    department: "IT"
};

// Union vs Intersection
type StringOrNumber = string | number; // Can be either string OR number
type StringAndNumber = string & number; // Never (impossible type)
```

#### 2.2.2 Interface'ler

Interface'ler, nesnelerin yapısını tanımlamak için kullanılır:

```typescript
// Temel interface
interface User {
    id: number;
    name: string;
    email: string;
    age?: number; // Optional property
    readonly createdAt: Date; // Readonly property
}

// Interface implementation
class UserService implements User {
    id: number;
    name: string;
    email: string;
    readonly createdAt: Date;

    constructor(id: number, name: string, email: string) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.createdAt = new Date();
    }
}

// Interface extension
interface AdminUser extends User {
    permissions: string[];
    isActive: boolean;
}

// Interface merging
interface Window {
    title: string;
}

interface Window {
    version: string;
}

// Now Window has both title and version properties

// Function interfaces
interface SearchFunction {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunction = function(source: string, subString: string): boolean {
    return source.indexOf(subString) > -1;
};

// Indexable interfaces
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray = ["Bob", "Fred"];
let myStr: string = myArray[0];
```

#### 2.2.3 Class'lar

TypeScript'te class'lar ES6 class'larına ek olarak tip güvenliği sağlar:

```typescript
// Temel class
class Animal {
    private name: string; // Private property
    protected age: number; // Protected property
    public species: string; // Public property (default)

    constructor(name: string, age: number, species: string) {
        this.name = name;
        this.age = age;
        this.species = species;
    }

    // Public method
    public getName(): string {
        return this.name;
    }

    // Protected method
    protected getAge(): number {
        return this.age;
    }

    // Private method
    private getSpecies(): string {
        return this.species;
    }

    // Static method
    static createDefault(): Animal {
        return new Animal("Unknown", 0, "Unknown");
    }
}

// Class inheritance
class Dog extends Animal {
    private breed: string;

    constructor(name: string, age: number, breed: string) {
        super(name, age, "Canine");
        this.breed = breed;
    }

    // Override method
    public getName(): string {
        return `Dog: ${super.getName()}`;
    }

    // New method
    public getBreed(): string {
        return this.breed;
    }

    // Access protected property
    public getDogAge(): number {
        return this.getAge(); // Can access protected method
    }
}

// Abstract classes
abstract class Shape {
    protected color: string;

    constructor(color: string) {
        this.color = color;
    }

    // Abstract method
    abstract getArea(): number;

    // Concrete method
    public getColor(): string {
        return this.color;
    }
}

class Circle extends Shape {
    private radius: number;

    constructor(color: string, radius: number) {
        super(color);
        this.radius = radius;
    }

    public getArea(): number {
        return Math.PI * this.radius * this.radius;
    }
}

// Access modifiers
class AccessExample {
    public publicProp: string = "public";
    private privateProp: string = "private";
    protected protectedProp: string = "protected";
    readonly readonlyProp: string = "readonly";

    public publicMethod(): void {
        console.log("Public method");
    }

    private privateMethod(): void {
        console.log("Private method");
    }

    protected protectedMethod(): void {
        console.log("Protected method");
    }
}
```

#### 2.2.4 Generics

Generics, tip güvenliğini korurken kodun yeniden kullanılabilirliğini artırır:

```typescript
// Temel generic function
function identity<T>(arg: T): T {
    return arg;
}

let output1 = identity<string>("hello"); // output1 is of type string
let output2 = identity<number>(42); // output2 is of type number
let output3 = identity("hello"); // Type inference: output3 is of type string

// Generic interface
interface GenericIdentityFn<T> {
    (arg: T): T;
}

let myIdentity: GenericIdentityFn<number> = identity;

// Generic class
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };

// Generic constraints
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length); // Now we know it has a .length property
    return arg;
}

loggingIdentity("hello"); // OK
loggingIdentity([1, 2, 3]); // OK
// loggingIdentity(123); // Error: Argument of type 'number' is not assignable

// Using type parameters in generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };
getProperty(x, "a"); // OK
getProperty(x, "m"); // Error: Argument of type '"m"' is not assignable

// Generic utility types
interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

// Required - makes all properties required
type RequiredUser = Required<PartialUser>;
// { id: number; name: string; email: string; age: number; }

// Pick - select specific properties
type UserBasicInfo = Pick<User, "id" | "name">;
// { id: number; name: string; }

// Omit - exclude specific properties
type UserWithoutId = Omit<User, "id">;
// { name: string; email: string; age: number; }

// Record - create object type with specific keys and values
type UserRoles = Record<string, string[]>;
// { [key: string]: string[]; }
```

#### 2.2.5 Enum'lar

Enum'lar, sabit değerlerin kümesini tanımlamak için kullanılır:

```typescript
// Numeric enum
enum Direction {
    Up,    // 0
    Down,  // 1
    Left,  // 2
    Right  // 3
}

let direction: Direction = Direction.Up;
console.log(direction); // 0

// String enum
enum Color {
    Red = "red",
    Green = "green",
    Blue = "blue"
}

let color: Color = Color.Red;
console.log(color); // "red"

// Mixed enum
enum Status {
    Pending = 0,
    Approved = "approved",
    Rejected = "rejected"
}

// Const enum (inlined at compile time)
const enum Size {
    Small = "S",
    Medium = "M",
    Large = "L"
}

let size: Size = Size.Medium;

// Enum with computed values
enum FileAccess {
    None,
    Read = 1 << 1,
    Write = 1 << 2,
    ReadWrite = Read | Write,
    G = "123".length
}

// Reverse mapping (only for numeric enums)
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"

// Ambient enum
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

#### 2.2.6 Type Guards ve Type Narrowing

Type guards, tip güvenliğini sağlamak için kullanılır:

```typescript
// typeof type guard
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

// instanceof type guard
class Bird {
    fly() {
        console.log("Flying");
    }
    layEggs() {
        console.log("Laying eggs");
    }
}

class Fish {
    swim() {
        console.log("Swimming");
    }
    layEggs() {
        console.log("Laying eggs");
    }
}

function getRandomPet(): Fish | Bird {
    return Math.random() > 0.5 ? new Fish() : new Bird();
}

let pet = getRandomPet();

if (pet instanceof Fish) {
    pet.swim(); // OK
}
if (pet instanceof Bird) {
    pet.fly(); // OK
}

// User-defined type guards
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}

if (isFish(pet)) {
    pet.swim(); // OK
} else {
    pet.fly(); // OK
}

// Discriminated unions
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;

function area(s: Shape): number {
    switch (s.kind) {
        case "square":
            return s.size * s.size;
        case "rectangle":
            return s.width * s.height;
        case "circle":
            return Math.PI * s.radius * s.radius;
        default:
            const _exhaustiveCheck: never = s;
            return _exhaustiveCheck;
    }
}

// In operator type guard
interface A {
    x: number;
}

interface B {
    y: string;
}

function doStuff(q: A | B) {
    if ("x" in q) {
        // q is A
        console.log(q.x);
    } else {
        // q is B
        console.log(q.y);
    }
}
```

#### 2.2.7 Advanced Types

TypeScript'in gelişmiş tip özellikleri:

```typescript
// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;

type Example1 = NonNullable<string | null>; // string
type Example2 = NonNullable<number | undefined>; // number

// Mapped types
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

type Partial<T> = {
    [P in keyof T]?: T[P];
};

// Template literal types
type EventName<T extends string> = `on${Capitalize<T>}`;
type MouseEvent = EventName<"click" | "hover">; // "onClick" | "onHover"

// Keyof and typeof
const colors = {
    red: "#ff0000",
    green: "#00ff00",
    blue: "#0000ff"
} as const;

type ColorKeys = keyof typeof colors; // "red" | "green" | "blue"
type ColorValues = typeof colors[ColorKeys]; // "#ff0000" | "#00ff00" | "#0000ff"

// Indexed access types
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number
type I1 = Person["age" | "name"]; // string | number
type I2 = Person[keyof Person]; // string | number | boolean

// Utility types
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

// Partial
type PartialTodo = Partial<Todo>;
// { title?: string; description?: string; completed?: boolean; }

// Required
type RequiredTodo = Required<PartialTodo>;
// { title: string; description: string; completed: boolean; }

// Pick
type TodoPreview = Pick<Todo, "title" | "completed">;
// { title: string; completed: boolean; }

// Omit
type TodoInfo = Omit<Todo, "completed">;
// { title: string; description: string; }

// Record
type PageInfo = Record<"home" | "about" | "contact", { title: string; url: string }>;
// { home: { title: string; url: string }; about: { title: string; url: string }; contact: { title: string; url: string }; }

// Exclude
type T0 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // "c"

// Extract
type T2 = Extract<"a" | "b" | "c", "a" | "f">; // "a"
type T3 = Extract<string | number | (() => void), Function>; // () => void

// NonNullable
type T4 = NonNullable<string | number | undefined>; // string | number

// Parameters
type T5 = Parameters<(s: string, n: number) => void>; // [string, number]

// ReturnType
type T6 = ReturnType<() => string>; // string
type T7 = ReturnType<(s: string) => void>; // void
```

### 2.3 TypeScript Versiyonları

**TypeScript 0.8 (2012)**:
- **Temel tip sistemi**: Primitive types (string, number, boolean)
- **Interface'ler**: Object type definitions
- **Class'lar**: ES6 class syntax support
- **Enum'lar**: Enumerated types
- **Function types**: Function signature typing
- **Array types**: Array type annotations
- **Basic generics**: Simple generic support

**TypeScript 0.9 (2013)**:
- **Generic constraints**: Generic type constraints
- **Union types**: Basic union type support
- **Intersection types**: Type intersection
- **Type aliases**: Type alias declarations
- **String literal types**: String literal types
- **Numeric literal types**: Numeric literal types

**TypeScript 1.0 (2014)**:
- **Advanced generics**: Complex generic scenarios
- **Union types**: Full union type support
- **Intersection types**: Complete intersection types
- **Type guards**: Type narrowing with type guards
- **Discriminated unions**: Tagged union types
- **Mapped types**: Basic mapped type support
- **Conditional types**: Simple conditional types

**TypeScript 1.1 (2014)**:
- **Performance improvements**: Compiler performance
- **Better error messages**: Improved error reporting
- **Module resolution**: Enhanced module resolution
- **Declaration files**: Better .d.ts support

**TypeScript 1.2 (2014)**:
- **Decorator'lar**: Decorator support (experimental)
- **Namespaces**: Namespace declarations
- **Module augmentation**: Module augmentation
- **Declaration merging**: Declaration merging
- **Ambient declarations**: Ambient module declarations

**TypeScript 1.3 (2014)**:
- **Protected members**: Protected class members
- **Tuple types**: Tuple type support
- **ES6 modules**: ES6 module support
- **let/const**: let and const support

**TypeScript 1.4 (2015)**:
- **Union types improvements**: Better union type inference
- **Type guards**: User-defined type guards
- **Template strings**: Template string types
- **let/const**: Full let/const support

**TypeScript 1.5 (2015)**:
- **ES6 modules**: Full ES6 module support
- **Destructuring**: Destructuring support
- **Spread operator**: Spread operator support
- **for...of**: for...of loop support

**TypeScript 1.6 (2015)**:
- **JSX support**: JSX syntax support
- **React support**: React JSX support
- **Class expressions**: Class expressions
- **Abstract classes**: Abstract class support

**TypeScript 1.7 (2015)**:
- **Async/await**: Async/await support
- **ES6 target**: ES6 compilation target
- **Polymorphic this**: Polymorphic this types
- **Module resolution**: Enhanced module resolution

**TypeScript 1.8 (2016)**:
- **String literal types**: String literal types
- **Numeric literal types**: Numeric literal types
- **Union types**: Improved union types
- **Control flow analysis**: Better control flow analysis

**TypeScript 2.0 (2016)**:
- Nullable types
- Control flow analysis
- Tagged union types
- Never type
- Readonly properties

**TypeScript 2.1-2.9**:
- Mapped types
- Conditional types
- Keyof operator
- Partial, Required, Pick, Record utility types
- Strict mode options

**TypeScript 3.0 (2018)**:
- Project references
- Tuples in rest parameters
- Unknown type
- Definite assignment assertions

**TypeScript 3.1-3.9**:
- Mapped types improvements
- Template literal types
- Recursive conditional types
- Assertion functions
- Optional chaining
- Nullish coalescing

**TypeScript 4.0 (2020)**:
- Variadic tuple types
- Labeled tuple elements
- Class property inference
- Short-circuiting assignment operators

**TypeScript 4.1-4.9**:
- Template literal types improvements
- Key remapping in mapped types
- Recursive conditional types
- Indexed access types
- Template literal types inference

**TypeScript 5.0 (2023)**:
- Decorators (stage 3)
- const type parameters
- extends constraints on infer type variables
- Resolution mode flags

**TypeScript 5.1-5.8 (2024-2025)**:
- **Improved Type Inference**: Gelişmiş tip çıkarımı
- **Better Performance**: Derleme performansı iyileştirmeleri
- **Enhanced Error Messages**: Daha açıklayıcı hata mesajları
- **New Utility Types**: Yeni yardımcı tipler
- **Improved Module Resolution**: Gelişmiş modül çözümleme
- **Golang Rewrite**: Derleyicinin Go ile yeniden yazılması (experimental)
- **Faster Compilation**: Daha hızlı derleme süreçleri
- **Better Memory Usage**: Bellek kullanımı optimizasyonları
- **Enhanced JSX Support**: Gelişmiş JSX desteği
- **Stricter Type Checking**: Daha sıkı tip kontrolü


### 2.4 İleri Seviye TypeScript Konuları - Detaylı Analiz

İleri seviye TypeScript konuları, modern tip güvenli programlamanın en karmaşık ve güçlü özelliklerini içerir. Bu konular, sadece syntax öğrenmek değil, aynı zamanda tip sisteminin derinliklerini anlamak ve karmaşık tip senaryolarını çözmek için kritik öneme sahiptir. İleri seviye TypeScript, functional programming, type-level programming ve advanced type manipulation gibi konuları kapsar.

**İleri Seviye TypeScript'in Felsefesi:**

İleri seviye TypeScript, "type-level programming" prensibini benimser. Bu prensip, tip seviyesinde programlama yaparak, compile-time'da karmaşık hesaplamalar ve dönüşümler gerçekleştirmeyi amaçlar. Bu yaklaşım, runtime overhead'i azaltır ve tip güvenliğini maksimize eder.

**İleri Seviye TypeScript'in Kategorileri:**

**Advanced Types (Gelişmiş Tipler):**
Gelişmiş tipler, TypeScript'in en karmaşık tip özelliklerini içerir. Bu tipler, conditional types, mapped types, template literal types ve recursive types gibi farklı kategorileri kapsar.

**Type Guards ve Assertions (Tip Koruyucuları ve Onayları):**
Tip koruyucuları ve onayları, tip narrowing ve type assertion işlemlerini yönetir. Bu özellikler, runtime'da tip kontrolü ve compile-time'da tip onayı sağlar.

**Generic Programming (Generic Programlama):**
Generic programlama, tip parametreleri kullanarak yeniden kullanılabilir kod yazmayı sağlar. Bu konu, generic constraints, variance ve higher-kinded types gibi konuları kapsar.

**Type-level Programming (Tip Seviyesi Programlama):**
Tip seviyesi programlama, tip seviyesinde hesaplamalar ve dönüşümler yapmayı sağlar. Bu konu, conditional types, mapped types ve template literal types gibi konuları kapsar.

**Advanced Types - Derinlemesine Analiz:**

**Distributive Conditional Types (Dağıtımsal Koşullu Tipler):**
Dağıtımsal koşullu tipler, union type'lar üzerinde koşullu tip işlemlerini yapar. Bu özellik, tip seviyesinde karmaşık dönüşümler gerçekleştirmeyi sağlar.

**Infer Keyword (Infer Anahtar Kelimesi):**
Infer anahtar kelimesi, conditional types içinde tip çıkarımı yapar. Bu özellik, tip seviyesinde pattern matching işlemlerini gerçekleştirmeyi sağlar.

**Template Literal Types (Şablon Literal Tipleri):**
Şablon literal tipleri, string template'leri tip seviyesinde destekler. Bu özellik, string manipulation'ları için çok etkilidir.

**Recursive Types (Özyinelemeli Tipler):**
Özyinelemeli tipler, kendini referans eden tip tanımlamaları sağlar. Bu özellik, karmaşık veri yapılarını modellemeyi sağlar.

**Branded Types (Markalı Tipler):**
Markalı tipler, nominal typing benzeri özellikler sağlar. Bu özellik, tip güvenliğini artırır ve tip karışıklığını önler.

**Nominal Typing (İsimsel Tip Sistemi):**
İsimsel tip sistemi, structural typing'e alternatif olarak nominal typing sağlar. Bu özellik, tip güvenliğini artırır.

**Type Guards ve Assertions - Derinlemesine Analiz:**

**Type Guards (Tip Koruyucuları):**
Tip koruyucuları, runtime'da tip kontrolü yapar. Bu özellik, type narrowing işlemlerini gerçekleştirmeyi sağlar.

**Type Assertions (Tip Onayları):**
Tip onayları, compile-time'da tip onayı yapar. Bu özellik, tip güvenliğini sağlar.

**User-defined Type Guards (Kullanıcı Tanımlı Tip Koruyucuları):**
Kullanıcı tanımlı tip koruyucuları, özel tip kontrol fonksiyonları sağlar. Bu özellik, karmaşık tip kontrol senaryolarını destekler.

**Type Predicates (Tip Yüklemeleri):**
Tip yüklemeleri, boolean döndüren tip kontrol fonksiyonları sağlar. Bu özellik, type narrowing işlemlerini gerçekleştirmeyi sağlar.

**Generic Programming - Derinlemesine Analiz:**

**Generic Constraints (Generic Kısıtlamalar):**
Generic kısıtlamalar, generic type parametrelerini sınırlar. Bu özellik, tip güvenliğini artırır.

**Variance (Varyans):**
Varyans, generic type'ların alt tip ilişkilerini belirler. Bu konu, covariance, contravariance ve invariance gibi kavramları kapsar.

**Higher-kinded Types (Yüksek Dereceli Tipler):**
Yüksek dereceli tipler, generic'lerin generic'lerini destekler. Bu özellik, çok karmaşık tip yapılarını destekler.

**Type-level Programming - Derinlemesine Analiz:**

**Conditional Types (Koşullu Tipler):**
Koşullu tipler, tip seviyesinde koşullar sağlar. Bu özellik, tip seviyesinde programlama yapılmasını sağlar.

**Mapped Types (Eşleştirilmiş Tipler):**
Eşleştirilmiş tipler, mevcut tiplerden yeni tipler oluşturur. Bu özellik, tip transformation'ları sağlar.

**Template Literal Types (Şablon Literal Tipleri):**
Şablon literal tipleri, string template'leri tip seviyesinde destekler. Bu özellik, string manipulation'ları için çok etkilidir.

**İleri Seviye TypeScript'in Kullanım Senaryoları:**

**Complex Type Transformations:**
İleri seviye TypeScript, karmaşık tip dönüşümlerini gerçekleştirmek için kullanılır. Bu dönüşümler, API response'ları, form data'ları ve state management gibi konularda kritik öneme sahiptir.

**Type-safe APIs:**
İleri seviye TypeScript, tip güvenli API'ler oluşturmak için kullanılır. Bu API'ler, compile-time'da tip kontrolü sağlar.

**Advanced Generic Libraries:**
İleri seviye TypeScript, gelişmiş generic kütüphaneler oluşturmak için kullanılır. Bu kütüphaneler, tip güvenliği ile birlikte esneklik sağlar.

**Type-level Computations:**
İleri seviye TypeScript, tip seviyesinde hesaplamalar yapmak için kullanılır. Bu hesaplamalar, compile-time'da gerçekleştirilir.

**Type Guards ve Assertions - Derinlemesine Analiz:**
- **Type Guards**: `typeof`, `instanceof`, `in` operatörleri
- **User-Defined Type Guards**: Özel tip koruyucuları
- **Type Assertions**: `as` ve `<>` sözdizimi
- **Assertion Functions**: `asserts` anahtar kelimesi

**Declaration Merging**:
- **Interface Merging**: Interface'lerin birleştirilmesi
- **Namespace Merging**: Namespace'lerin birleştirilmesi
- **Module Augmentation**: Modül genişletme
- **Global Augmentation**: Global tipleri genişletme

**Performance ve Optimization**:
- **Incremental Compilation**: Artımlı derleme
- **Project References**: Proje referansları
- **Build Performance**: Derleme performansı
- **Memory Usage**: Bellek kullanımı optimizasyonu

## BÖLÜM 3: REACT JS - MODERN WEB GELİŞTİRME
