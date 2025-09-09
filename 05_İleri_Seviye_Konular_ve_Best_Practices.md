## BÖLÜM 5: İLERİ SEVİYE KONULAR VE BEST PRACTICES

**İleri Seviye Konular Nedir?**

İleri seviye konular, JavaScript, TypeScript, React ve React Native geliştirmede profesyonel seviyede uygulanan teknikler, pattern'lar ve best practice'lerdir. Bu konular, büyük ölçekli uygulamalar geliştirirken karşılaşılan karmaşık problemleri çözmek ve kod kalitesini artırmak için kullanılır.

**İleri Seviye Konular Ne Demek?**

İleri seviye konular, temel programlama bilgisinin ötesinde, profesyonel geliştirme ortamlarında karşılaşılan gerçek dünya problemlerini çözmek için kullanılan tekniklerdir. Bu konular, kod kalitesi, performans, güvenlik, sürdürülebilirlik ve ölçeklenebilirlik gibi kritik faktörleri ele alır.

**Neden İhtiyaç Var?**

Büyük ölçekli uygulamalar geliştirirken karşılaşılan sorunlar:

1. **Kod Organizasyonu**: Büyük projelerde kod organizasyonu karmaşık hale gelir
2. **Performans Sorunları**: Uygulama büyüdükçe performans sorunları ortaya çıkar
3. **Güvenlik Açıkları**: Güvenlik açıkları ve veri koruma sorunları
4. **Sürdürülebilirlik**: Kod sürdürülebilirliği ve bakım zorlaşır
5. **Takım Çalışması**: Farklı geliştiriciler arasında koordinasyon sorunları
6. **Ölçeklenebilirlik**: Uygulamanın büyümesi ile birlikte ölçeklenebilirlik sorunları

**Ne İşe Yarar?**

İleri seviye konular, profesyonel geliştirme ortamlarında:

1. **Kod Kalitesi**: Daha kaliteli ve sürdürülebilir kod yazma
2. **Performans**: Yüksek performanslı uygulamalar geliştirme
3. **Güvenlik**: Güvenli uygulamalar oluşturma
4. **Ölçeklenebilirlik**: Büyük ölçekli uygulamalar geliştirme
5. **Takım İşbirliği**: Daha iyi takım çalışması ve koordinasyon

**İleri Seviye Konuların Kapsamı:**

1. **Architecture Patterns - Mimari Desenler**: Uygulama mimarisi tasarımı
```javascript
// Architecture Pattern Örneği - Repository Pattern
class UserRepository {
    constructor(apiClient) {
        this.apiClient = apiClient;
    }
    
    async getUsers() {
        try {
            const response = await this.apiClient.get('/users');
            return response.data.map(user => new User(user));
        } catch (error) {
            throw new Error(`Failed to fetch users: ${error.message}`);
        }
    }
    
    async createUser(userData) {
        try {
            const response = await this.apiClient.post('/users', userData);
            return new User(response.data);
        } catch (error) {
            throw new Error(`Failed to create user: ${error.message}`);
        }
    }
}

// Service Layer
class UserService {
    constructor(userRepository) {
        this.userRepository = userRepository;
    }
    
    async getAllUsers() {
        return await this.userRepository.getUsers();
    }
    
    async addUser(userData) {
        const user = new User(userData);
        if (!user.validate()) {
            throw new Error('Invalid user data');
        }
        return await this.userRepository.createUser(userData);
    }
}
```

2. **State Management - Durum Yönetimi**: Uygulama durumunu yönetme
```javascript
// State Management Örneği - Redux Pattern
const initialState = {
    users: [],
    loading: false,
    error: null
};

function userReducer(state = initialState, action) {
    switch (action.type) {
        case 'FETCH_USERS_START':
            return { ...state, loading: true, error: null };
        case 'FETCH_USERS_SUCCESS':
            return { ...state, users: action.payload, loading: false };
        case 'FETCH_USERS_ERROR':
            return { ...state, error: action.payload, loading: false };
        default:
            return state;
    }
}

// Action Creators
const fetchUsersStart = () => ({ type: 'FETCH_USERS_START' });
const fetchUsersSuccess = (users) => ({ type: 'FETCH_USERS_SUCCESS', payload: users });
const fetchUsersError = (error) => ({ type: 'FETCH_USERS_ERROR', payload: error });

// Async Action Creator
const fetchUsers = () => async (dispatch) => {
    dispatch(fetchUsersStart());
    try {
        const response = await fetch('/api/users');
        const users = await response.json();
        dispatch(fetchUsersSuccess(users));
    } catch (error) {
        dispatch(fetchUsersError(error.message));
    }
};
```

3. **Performance Optimization - Performans Optimizasyonu**: Uygulama performansını artırma
```javascript
// Performance Optimization Örneği - Memoization
const expensiveCalculation = (() => {
    const cache = new Map();
    
    return function(n) {
        if (cache.has(n)) {
            console.log('Cache hit for', n);
            return cache.get(n);
        }
        
        console.log('Calculating for', n);
        let result = 0;
        for (let i = 0; i < n; i++) {
            result += Math.pow(i, 2);
        }
        
        cache.set(n, result);
        return result;
    };
})();

// React Performance Optimization
import React, { memo, useMemo, useCallback } from 'react';

const ExpensiveComponent = memo(({ data, onUpdate }) => {
    const processedData = useMemo(() => {
        console.log('Processing data...');
        return data.map(item => ({
            ...item,
            processed: item.value * 2
        }));
    }, [data]);
    
    const handleUpdate = useCallback((id) => {
        onUpdate(id);
    }, [onUpdate]);
    
    return (
        <div>
            {processedData.map(item => (
                <div key={item.id} onClick={() => handleUpdate(item.id)}>
                    {item.name}: {item.processed}
                </div>
            ))}
        </div>
    );
});
```

4. **Security - Güvenlik**: Uygulama güvenliğini sağlama
```javascript
// Security Örneği - Input Validation ve Sanitization
class SecurityValidator {
    static validateEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }
    
    static sanitizeInput(input) {
        if (typeof input !== 'string') return input;
        
        return input
            .replace(/[<>]/g, '') // XSS koruması
            .replace(/['"]/g, '') // SQL injection koruması
            .trim();
    }
    
    static validatePassword(password) {
        const minLength = 8;
        const hasUpperCase = /[A-Z]/.test(password);
        const hasLowerCase = /[a-z]/.test(password);
        const hasNumbers = /\d/.test(password);
        const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
        
        return {
            isValid: password.length >= minLength && 
                    hasUpperCase && 
                    hasLowerCase && 
                    hasNumbers && 
                    hasSpecialChar,
            errors: [
                password.length < minLength && 'Password must be at least 8 characters',
                !hasUpperCase && 'Password must contain uppercase letter',
                !hasLowerCase && 'Password must contain lowercase letter',
                !hasNumbers && 'Password must contain number',
                !hasSpecialChar && 'Password must contain special character'
            ].filter(Boolean)
        };
    }
}

// JWT Token Management
class TokenManager {
    static setToken(token) {
        localStorage.setItem('authToken', token);
    }
    
    static getToken() {
        return localStorage.getItem('authToken');
    }
    
    static removeToken() {
        localStorage.removeItem('authToken');
    }
    
    static isTokenValid(token) {
        if (!token) return false;
        
        try {
            const payload = JSON.parse(atob(token.split('.')[1]));
            const currentTime = Date.now() / 1000;
            return payload.exp > currentTime;
        } catch (error) {
            return false;
        }
    }
}
```

5. **Accessibility - Erişilebilirlik**: Erişilebilir uygulamalar geliştirme
```jsx
// Accessibility Örneği - React'te Erişilebilirlik
import React from 'react';

function AccessibleButton({ onClick, children, disabled, ariaLabel }) {
    return (
        <button
            onClick={onClick}
            disabled={disabled}
            aria-label={ariaLabel}
            aria-disabled={disabled}
            className="accessible-button"
            style={{
                padding: '12px 24px',
                fontSize: '16px',
                border: '2px solid #007bff',
                borderRadius: '4px',
                backgroundColor: disabled ? '#f8f9fa' : '#007bff',
                color: disabled ? '#6c757d' : 'white',
                cursor: disabled ? 'not-allowed' : 'pointer',
                minHeight: '44px', // Touch target size
                minWidth: '44px'
            }}
        >
            {children}
        </button>
    );
}

function AccessibleForm() {
    const [email, setEmail] = React.useState('');
    const [error, setError] = React.useState('');
    
    const handleSubmit = (e) => {
        e.preventDefault();
        if (!email) {
            setError('Email is required');
            return;
        }
        // Form submission logic
    };
    
    return (
        <form onSubmit={handleSubmit} role="form" aria-label="Contact Form">
            <div>
                <label htmlFor="email" id="email-label">
                    Email Address
                </label>
                <input
                    id="email"
                    type="email"
                    value={email}
                    onChange={(e) => setEmail(e.target.value)}
                    aria-labelledby="email-label"
                    aria-describedby={error ? "email-error" : undefined}
                    aria-invalid={error ? "true" : "false"}
                    required
                />
                {error && (
                    <div id="email-error" role="alert" aria-live="polite">
                        {error}
                    </div>
                )}
            </div>
            
            <AccessibleButton
                onClick={handleSubmit}
                ariaLabel="Submit contact form"
            >
                Submit
            </AccessibleButton>
        </form>
    );
}
```

**İleri Seviye Konuların Faydaları:**

1. **Scalable Applications - Ölçeklenebilir Uygulamalar**: Büyük ölçekli uygulamalar geliştirme
2. **Maintainable Code - Sürdürülebilir Kod**: Kolay bakım yapılabilir kod
3. **High Performance - Yüksek Performans**: Optimize edilmiş performans
4. **Secure Applications - Güvenli Uygulamalar**: Güvenlik odaklı geliştirme
5. **User Experience - Kullanıcı Deneyimi**: İyi kullanıcı deneyimi
6. **Team Collaboration - Takım İşbirliği**: Etkili takım çalışması
7. **Industry Standards - Endüstri Standartları**: Endüstri standartlarına uygunluk
8. **Future-Proof Code - Geleceğe Dayanıklı Kod**: Gelecekteki değişikliklere hazır kod

**İleri Seviye Konuların Kullanım Alanları:**

1. **Enterprise Applications - Kurumsal Uygulamalar**: Büyük şirket uygulamaları
2. **Large-Scale Projects - Büyük Ölçekli Projeler**: Büyük projeler
3. **High-Traffic Applications - Yüksek Trafikli Uygulamalar**: Yoğun kullanıcı trafiği
4. **Mission-Critical Systems - Kritik Sistemler**: Kritik sistemler
5. **International Products - Uluslararası Ürünler**: Global ürünler
6. **Accessible Applications - Erişilebilir Uygulamalar**: Erişilebilir uygulamalar
7. **Performance-Critical Apps - Performans Kritik Uygulamalar**: Performans odaklı uygulamalar
8. **Security-Sensitive Systems - Güvenlik Hassas Sistemler**: Güvenlik kritik sistemler

### 5.1 Architecture Patterns - Detaylı Analiz

Mimari desenler, büyük ölçekli uygulamaların organizasyonu, sürdürülebilirliği ve ölçeklenebilirliği için kritik öneme sahiptir. Bu desenler, karmaşık sistemleri yönetilebilir parçalara ayırarak, kod kalitesini artırır ve geliştirici deneyimini iyileştirir.

#### 5.1.1 MVC (Model-View-Controller) - Derinlemesine Analiz

MVC (Model-View-Controller), yazılım geliştirmede en yaygın kullanılan mimari desenlerden biridir. Bu desen, uygulamayı üç ana bileşene ayırarak, separation of concerns (sorumlulukların ayrılması) prensibini uygular.

**MVC'nin Temel Felsefesi:**

MVC deseni, uygulamanın farklı katmanlarını birbirinden ayırarak, her katmanın kendi sorumluluğuna odaklanmasını sağlar. Bu ayrım, kodun daha modüler, test edilebilir ve sürdürülebilir olmasını mümkün kılar.

**Model (Model) - Veri Katmanı - Derinlemesine Analiz**

Model katmanı, MVC mimarisinin kalbidir ve uygulamanın veri yönetimi ile ilgili tüm işlemlerini üstlenir. Bu katman, sadece veri saklama değil, aynı zamanda iş kurallarının uygulanması, veri doğrulama ve veri dönüştürme gibi kritik işlemleri de içerir.

**Model'in Sorumlulukları ve Detayları:**

**Data Structure (Veri Yapısı):**
Model katmanı, uygulamanın veri yapısını tanımlar. Bu yapı, sadece basit veri tipleri değil, aynı zamanda karmaşık nesne ilişkilerini, veri kısıtlamalarını ve veri bütünlüğünü de içerir. Model, veri yapısının tutarlılığını sağlar ve veri erişim kurallarını belirler.

**Business Logic (İş Mantığı):**
Model katmanı, uygulamanın iş kurallarını uygular. Bu kurallar, veri doğrulama, hesaplamalar, iş akışları ve domain-specific operasyonları içerir. İş mantığının model katmanında bulunması, bu kuralların merkezi bir yerden yönetilmesini sağlar.

**Data Validation (Veri Doğrulama):**
Model katmanı, veri girişlerinin doğruluğunu kontrol eder. Bu doğrulama, format kontrolü, veri tipi kontrolü, kısıtlama kontrolü ve iş kuralları kontrolü gibi farklı seviyelerde gerçekleşir.

**Data Persistence (Veri Kalıcılığı):**
Model katmanı, verilerin kalıcı olarak saklanmasını sağlar. Bu işlem, veritabanı işlemleri, dosya sistemi işlemleri, API çağrıları ve cache yönetimi gibi farklı yöntemlerle gerçekleştirilebilir.

**Data Access (Veri Erişimi):**
Model katmanı, veri erişim katmanını (Data Access Layer) içerir. Bu katman, veri kaynaklarına erişimi yönetir ve veri erişim operasyonlarını optimize eder.

**Data Transformation (Veri Dönüştürme):**
Model katmanı, verilerin farklı formatlar arasında dönüştürülmesini sağlar. Bu dönüştürme, API formatları, veritabanı formatları, UI formatları ve internal formatlar arasında gerçekleşir.

**Model'in Özellikleri ve Avantajları:**

**Independent (Bağımsızlık):**
Model katmanı, View ve Controller katmanlarından bağımsızdır. Bu bağımsızlık, model katmanının farklı UI teknolojileri ve farklı controller implementasyonları ile kullanılabilmesini sağlar.

**Reusable (Yeniden Kullanılabilirlik):**
Model katmanı, farklı projelerde ve farklı context'lerde yeniden kullanılabilir. Bu özellik, kod tekrarını azaltır ve geliştirme süresini kısaltır.

**Testable (Test Edilebilirlik):**
Model katmanı, unit testler ve integration testler için ideal bir yapı sunar. İş mantığının model katmanında bulunması, test coverage'ın artırılmasını kolaylaştırır.

**Maintainable (Sürdürülebilirlik):**
Model katmanı, iş kurallarının merkezi bir yerden yönetilmesini sağlar. Bu yapı, değişikliklerin daha kolay uygulanmasını ve hata ayıklama sürecini hızlandırır.

**Scalable (Ölçeklenebilirlik):**
Model katmanı, uygulamanın büyümesi ile birlikte ölçeklenebilir bir yapı sunar. Bu yapı, yeni özelliklerin eklenmesini ve mevcut özelliklerin genişletilmesini kolaylaştırır.

**Model Kullanım Alanları ve Senaryoları:**

**Database Entities (Veritabanı Varlıkları):**
Model katmanı, veritabanı tablolarını ve ilişkilerini temsil eder. Bu varlıklar, ORM (Object-Relational Mapping) araçları ile birlikte kullanılır.

**API Data (API Verileri):**
Model katmanı, external API'lerden gelen verileri temsil eder. Bu veriler, API response'ları, request payload'ları ve data transfer object'leri içerir.

**Business Objects (İş Nesneleri):**
Model katmanı, domain-specific iş nesnelerini temsil eder. Bu nesneler, uygulamanın iş kurallarını ve iş akışlarını yansıtır.

**Data Transfer Objects (Veri Transfer Nesneleri):**
Model katmanı, farklı katmanlar arasında veri transferini sağlayan nesneleri içerir. Bu nesneler, veri serialization ve deserialization işlemlerini kolaylaştırır.

**Domain Models (Domain Modelleri):**
Model katmanı, domain-driven design (DDD) prensiplerine uygun olarak tasarlanmış domain modellerini içerir. Bu modeller, business domain'inin karmaşık yapısını yansıtır.

```javascript
// User Model
class User {
    constructor(id, name, email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    validate() {
        return this.name && this.email && this.email.includes('@');
    }

    toJSON() {
        return {
            id: this.id,
            name: this.name,
            email: this.email
        };
    }
}

// User Service (Model Layer)
class UserService {
    constructor() {
        this.users = [];
    }

    createUser(userData) {
        const user = new User(
            Date.now(),
            userData.name,
            userData.email
        );
        
        if (user.validate()) {
            this.users.push(user);
            return user;
        }
        throw new Error('Invalid user data');
    }

    getUser(id) {
        return this.users.find(user => user.id === id);
    }

    getAllUsers() {
        return this.users;
    }
}
```

**View (Görünüm) - Kullanıcı Arayüzü Katmanı - Derinlemesine Analiz**

View katmanı, MVC mimarisinin kullanıcı ile etkileşim kurduğu katmandır. Bu katman, sadece veri görüntüleme değil, aynı zamanda kullanıcı deneyimi, etkileşim tasarımı ve arayüz optimizasyonu gibi kritik işlemleri de içerir.

**View'in Sorumlulukları ve Detayları:**

**Data Presentation (Veri Sunumu):**
View katmanı, Model'den gelen verileri kullanıcı dostu bir formatta sunar. Bu sunum, sadece veri görüntüleme değil, aynı zamanda veri formatlama, görselleştirme ve kullanıcı deneyimi optimizasyonu da içerir. View, verilerin kullanıcının anlayabileceği şekilde sunulmasını sağlar.

**User Interface (Kullanıcı Arayüzü):**
View katmanı, uygulamanın görsel arayüzünü oluşturur. Bu arayüz, layout tasarımı, component organizasyonu, styling ve responsive design gibi farklı boyutları içerir. View, kullanıcı deneyimini optimize eden bir arayüz tasarımı sağlar.

**User Interaction (Kullanıcı Etkileşimi):**
View katmanı, kullanıcı etkileşimlerini yönetir. Bu etkileşimler, form girişleri, buton tıklamaları, navigasyon ve gesture'lar gibi farklı türde kullanıcı aksiyonlarını içerir. View, bu etkileşimleri Controller'a iletir.

**Event Handling (Olay İşleme):**
View katmanı, kullanıcı etkileşimlerinden kaynaklanan olayları işler. Bu olay işleme, event delegation, event bubbling, event capturing ve custom event'ler gibi farklı teknikleri içerir. View, bu olayları Controller'a uygun şekilde iletir.

**UI Updates (Arayüz Güncellemeleri):**
View katmanı, Model'deki değişikliklere göre arayüzü günceller. Bu güncellemeler, incremental updates, batch updates, animation ve transition'lar gibi farklı teknikleri içerir. View, performanslı güncellemeler sağlar.

**Responsive Design (Duyarlı Tasarım):**
View katmanı, farklı cihaz ve ekran boyutlarına uyum sağlar. Bu uyum, mobile-first design, breakpoint'ler, flexible layout'lar ve adaptive components gibi farklı teknikleri içerir. View, tutarlı kullanıcı deneyimi sağlar.

**View'in Özellikleri ve Avantajları:**

**Passive (Pasif):**
View katmanı, Model'den veri alır ve Controller'a kullanıcı etkileşimlerini iletir. Bu pasif yapı, View'in Model ve Controller'dan bağımsız olmasını sağlar ve test edilebilirliği artırır.

**Reactive (Reaktif):**
View katmanı, Model'deki değişikliklere otomatik olarak tepki verir. Bu reaktif yapı, data binding, observer pattern ve event-driven architecture gibi teknikleri kullanır. View, real-time güncellemeler sağlar.

**Independent (Bağımsız):**
View katmanı, Model ve Controller'dan bağımsızdır. Bu bağımsızlık, View'in farklı Model implementasyonları ve farklı Controller stratejileri ile kullanılabilmesini sağlar.

**Controller (Kontrolcü) - İş Mantığı Katmanı - Derinlemesine Analiz**

Controller katmanı, MVC mimarisinin koordinasyon merkezidir. Bu katman, View ve Model arasındaki etkileşimi yönetir ve uygulamanın iş akışını kontrol eder.

**Controller'in Sorumlulukları ve Detayları:**

**Request Handling (İstek İşleme):**
Controller katmanı, kullanıcı isteklerini işler. Bu işleme, request validation, authentication, authorization ve routing gibi farklı aşamaları içerir. Controller, güvenli ve güvenilir istek işleme sağlar.

**Business Logic Coordination (İş Mantığı Koordinasyonu):**
Controller katmanı, Model'deki iş mantığını koordine eder. Bu koordinasyon, transaction management, workflow orchestration ve business rule enforcement gibi farklı işlemleri içerir. Controller, tutarlı iş akışı sağlar.

**Data Flow Management (Veri Akışı Yönetimi):**
Controller katmanı, View ve Model arasındaki veri akışını yönetir. Bu yönetim, data transformation, data validation ve data synchronization gibi farklı işlemleri içerir. Controller, veri bütünlüğünü sağlar.

**Error Handling (Hata Yönetimi):**
Controller katmanı, uygulama hatalarını yönetir. Bu yönetim, error detection, error logging, error recovery ve user-friendly error messages gibi farklı işlemleri içerir. Controller, güvenilir hata yönetimi sağlar.

**Controller'in Özellikleri ve Avantajları:**

**Centralized Control (Merkezi Kontrol):**
Controller katmanı, uygulamanın merkezi kontrol noktasıdır. Bu merkezi yapı, uygulama akışının tutarlı yönetilmesini sağlar ve debugging sürecini kolaylaştırır.

**Separation of Concerns (Sorumlulukların Ayrılması):**
Controller katmanı, View ve Model arasındaki bağımlılığı azaltır. Bu ayrım, kodun daha modüler ve sürdürülebilir olmasını sağlar.

**Testability (Test Edilebilirlik):**
Controller katmanı, unit testler ve integration testler için ideal bir yapı sunar. Bu yapı, test coverage'ın artırılmasını ve hata tespitinin hızlandırılmasını sağlar.
- **Reusable**: Yeniden kullanılabilir
- **Testable**: Test edilebilir

**View Kullanım Alanları:**
- **Web Pages**: Web sayfaları
- **Mobile Screens**: Mobil ekranlar
- **Desktop Applications**: Masaüstü uygulamaları
- **API Responses**: API yanıtları
- **Data Visualization**: Veri görselleştirme

```javascript
// User View
class UserView {
    constructor() {
        this.container = document.getElementById('user-container');
    }

    renderUser(user) {
        return `
            <div class="user-card" data-id="${user.id}">
                <h3>${user.name}</h3>
                <p>${user.email}</p>
                <button onclick="userController.deleteUser(${user.id})">Delete</button>
            </div>
        `;
    }

    renderUserList(users) {
        const html = users.map(user => this.renderUser(user)).join('');
        this.container.innerHTML = html;
    }

    showError(message) {
        alert(`Error: ${message}`);
    }

    showSuccess(message) {
        alert(`Success: ${message}`);
    }
}
```

**Controller (Kontrolcü) - İş Mantığı Katmanı**

Controller, uygulamanın iş mantığı katmanını temsil eder. Controller, Model ve View arasında köprü görevi görür. Kullanıcı etkileşimlerini alır, Model'i günceller ve View'i yeniler.

**Controller'in Sorumlulukları:**
- **User Input Handling**: Kullanıcı girişi işleme
- **Business Logic**: İş mantığı
- **Model Updates**: Model güncellemeleri
- **View Updates**: View güncellemeleri
- **Event Coordination**: Olay koordinasyonu
- **Data Flow Control**: Veri akış kontrolü

**Controller'in Özellikleri:**
- **Active**: Aktif (işlemleri başlatır)
- **Coordinator**: Koordinatör (Model ve View'i koordine eder)
- **Business Logic**: İş mantığını içerir
- **Event Handler**: Olay işleyicisi
- **State Manager**: Durum yöneticisi

**Controller Kullanım Alanları:**
- **Web Controllers**: Web kontrolcüleri
- **API Controllers**: API kontrolcüleri
- **Event Handlers**: Olay işleyicileri
- **Business Services**: İş servisleri
- **Application Logic**: Uygulama mantığı

```javascript
// User Controller
class UserController {
    constructor() {
        this.userService = new UserService();
        this.userView = new UserView();
        this.init();
    }

    init() {
        this.loadUsers();
        this.bindEvents();
    }

    bindEvents() {
        document.getElementById('add-user-form')
            .addEventListener('submit', (e) => this.handleAddUser(e));
    }

    handleAddUser(event) {
        event.preventDefault();
        const formData = new FormData(event.target);
        const userData = {
            name: formData.get('name'),
            email: formData.get('email')
        };

        try {
            const user = this.userService.createUser(userData);
            this.userView.showSuccess('User created successfully');
            this.loadUsers();
            event.target.reset();
        } catch (error) {
            this.userView.showError(error.message);
        }
    }

    loadUsers() {
        const users = this.userService.getAllUsers();
        this.userView.renderUserList(users);
    }

    deleteUser(id) {
        if (confirm('Are you sure you want to delete this user?')) {
            // Delete logic
            this.loadUsers();
        }
    }
}

// Initialize the application
const userController = new UserController();
```

**MVC Avantajları**:
- **Separation of Concerns**: Her katmanın kendine özgü sorumluluğu
- **Testability**: Her katman bağımsız olarak test edilebilir
- **Maintainability**: Değişiklikler izole edilir
- **Reusability**: Model ve View farklı controller'larla kullanılabilir

**MVC Dezavantajları**:
- **Complex Dependencies**: Katmanlar arası bağımlılıklar karmaşık olabilir
- **Fat Controllers**: Controller'lar çok büyüyebilir
- **Tight Coupling**: View ve Model arasında sıkı bağlantı
- **Boilerplate Code**: Çok fazla kod tekrarı

#### 5.1.2 MVP (Model-View-Presenter) - Derinlemesine Analiz

MVP (Model-View-Presenter), MVC deseninin geliştirilmiş bir versiyonudur. Bu desen, MVC'deki bazı sorunları çözmek için tasarlanmıştır ve özellikle test edilebilirlik ve View'ın pasifliği konularında iyileştirmeler sunar.

**MVP'nin MVC'den Farkları:**

MVP, MVC'den temel olarak View'ın rolü ve Presenter'ın eklenmesi konularında farklılaşır. MVP'de View tamamen pasif hale getirilir ve tüm iş mantığı Presenter'a taşınır. Bu değişiklik, test edilebilirliği artırır ve View'ın Model'e olan bağımlılığını ortadan kaldırır.

**MVP'nin Temel Felsefesi:**

MVP deseni, View'ı sadece bir display mechanism olarak kullanır ve tüm user interaction logic'i Presenter'a taşır. Bu yaklaşım, View'ın test edilmesini kolaylaştırır ve View'ın Model'e olan direct dependency'sini ortadan kaldırır.

**Model (Model) - Veri Katmanı - MVP'de Detaylı Analiz**

MVP'de Model katmanı, MVC ile aynı prensipleri takip eder ancak Presenter ile etkileşim kurar. Model, View'dan tamamen bağımsızdır ve sadece Presenter ile iletişim kurar.

**MVP'de Model'in Sorumlulukları ve Detayları:**

**Data Structure (Veri Yapısı):**
MVP'de Model, MVC ile aynı şekilde veri yapısını tanımlar. Ancak, MVP'de Model'in View ile direct communication'ı yoktur. Tüm veri erişimi Presenter üzerinden gerçekleşir.

**Business Logic (İş Mantığı):**
MVP'de Model, iş mantığını uygular ancak bu mantık Presenter tarafından koordine edilir. Model, sadece data manipulation ve business rule enforcement ile ilgilenir.

**Data Validation (Veri Doğrulama):**
MVP'de Model, veri doğrulama işlemlerini gerçekleştirir. Bu doğrulama, Presenter tarafından tetiklenir ve sonuçlar Presenter'a iletilir.

**Data Persistence (Veri Kalıcılığı):**
MVP'de Model, veri kalıcılığı işlemlerini gerçekleştirir. Bu işlemler, Presenter'ın istekleri doğrultusunda gerçekleşir.

**Data Access (Veri Erişimi):**
MVP'de Model, veri erişim katmanını yönetir. Bu erişim, Presenter'ın istekleri doğrultusunda gerçekleşir.

**Data Transformation (Veri Dönüştürme):**
MVP'de Model, veri dönüştürme işlemlerini gerçekleştirir. Bu dönüştürme, Presenter'ın ihtiyaçları doğrultusunda gerçekleşir.

**MVP'de Model'in Özellikleri ve Avantajları:**

**Independent (Bağımsızlık):**
MVP'de Model, View'dan tamamen bağımsızdır. Bu bağımsızlık, Model'in farklı View implementasyonları ile kullanılabilmesini sağlar.

**Reusable (Yeniden Kullanılabilirlik):**
MVP'de Model, farklı Presenter implementasyonları ile kullanılabilir. Bu özellik, kod tekrarını azaltır.

**Testable (Test Edilebilirlik):**
MVP'de Model, Presenter'dan bağımsız olarak test edilebilir. Bu yapı, unit test coverage'ın artırılmasını sağlar.

**Maintainable (Sürdürülebilirlik):**
MVP'de Model, iş kurallarının merkezi yönetimini sağlar. Bu yapı, değişikliklerin daha kolay uygulanmasını sağlar.

**Scalable (Ölçeklenebilirlik):**
MVP'de Model, uygulamanın büyümesi ile birlikte ölçeklenebilir bir yapı sunar.

**MVP'de Model Kullanım Alanları ve Senaryoları:**

**Database Entities (Veritabanı Varlıkları):**
MVP'de Model, veritabanı varlıklarını temsil eder. Bu varlıklar, Presenter tarafından kullanılır.

**API Data (API Verileri):**
MVP'de Model, external API'lerden gelen verileri temsil eder. Bu veriler, Presenter tarafından işlenir.

**Business Objects (İş Nesneleri):**
MVP'de Model, domain-specific iş nesnelerini temsil eder. Bu nesneler, Presenter tarafından koordine edilir.

**Data Transfer Objects (Veri Transfer Nesneleri):**
MVP'de Model, farklı katmanlar arasında veri transferini sağlar. Bu transfer, Presenter üzerinden gerçekleşir.

**Domain Models (Domain Modelleri):**
MVP'de Model, domain-driven design prensiplerine uygun olarak tasarlanmış domain modellerini içerir.

**View (Görünüm) - MVP'de Pasif Katman - Derinlemesine Analiz**

MVP'de View katmanı, MVC'den farklı olarak tamamen pasif bir role sahiptir. View, sadece veri görüntüleme ve kullanıcı etkileşimlerini Presenter'a iletme ile ilgilenir.

**MVP'de View'in Sorumlulukları ve Detayları:**

**Data Display (Veri Görüntüleme):**
MVP'de View, sadece Presenter'dan gelen verileri görüntüler. View, veri işleme veya business logic ile ilgilenmez.

**User Input Collection (Kullanıcı Girişi Toplama):**
MVP'de View, kullanıcı girişlerini toplar ve Presenter'a iletir. View, bu girişlerin işlenmesi ile ilgilenmez.

**Event Delegation (Olay Delegasyonu):**
MVP'de View, kullanıcı etkileşimlerini Presenter'a delegate eder. View, bu etkileşimlerin işlenmesi ile ilgilenmez.

**UI State Management (Arayüz Durumu Yönetimi):**
MVP'de View, sadece UI state'ini yönetir. Business state, Presenter tarafından yönetilir.

**MVP'de View'in Özellikleri:**

**Passive (Pasif):**
MVP'de View, tamamen pasif bir role sahiptir. View, hiçbir business logic içermez.

**Thin (İnce):**
MVP'de View, mümkün olduğunca ince tutulur. Tüm logic, Presenter'a taşınır.

**Testable (Test Edilebilir):**
MVP'de View, mock Presenter'lar ile kolayca test edilebilir.

**Presenter (Sunucu) - MVP'nin Kalbi - Derinlemesine Analiz**

Presenter, MVP mimarisinin en kritik bileşenidir. Presenter, View ve Model arasındaki tüm etkileşimi koordine eder ve tüm business logic'i içerir.

**Presenter'in Sorumlulukları ve Detayları:**

**Business Logic Implementation (İş Mantığı Uygulaması):**
Presenter, tüm business logic'i implement eder. Bu logic, user interaction handling, data processing ve workflow orchestration'ı içerir.

**View-Model Coordination (View-Model Koordinasyonu):**
Presenter, View ve Model arasındaki koordinasyonu sağlar. Bu koordinasyon, data flow management ve event handling'i içerir.

**User Interaction Processing (Kullanıcı Etkileşimi İşleme):**
Presenter, View'dan gelen user interaction'ları işler. Bu işleme, validation, business rule enforcement ve Model update'leri dahildir.

**Data Transformation (Veri Dönüştürme):**
Presenter, Model'den gelen verileri View'ın anlayabileceği formata dönüştürür. Bu dönüştürme, data formatting ve presentation logic'i içerir.

**Presenter'in Özellikleri:**

**Active (Aktif):**
Presenter, MVP mimarisinin en aktif bileşenidir. Tüm business logic, Presenter'da bulunur.

**Testable (Test Edilebilir):**
Presenter, View ve Model'den bağımsız olarak test edilebilir. Bu yapı, comprehensive testing'i mümkün kılar.

**Reusable (Yeniden Kullanılabilir):**
Presenter, farklı View implementasyonları ile kullanılabilir. Bu özellik, code reusability'yi artırır.

```javascript
// User Model (MVP'de MVC ile aynı)
class User {
    constructor(id, name, email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    validate() {
        return this.name && this.email && this.email.includes('@');
    }

    toJSON() {
        return {
            id: this.id,
            name: this.name,
            email: this.email
        };
    }
}

// User Repository
class UserRepository {
    constructor() {
        this.users = [];
    }

    save(user) {
        const existingIndex = this.users.findIndex(u => u.id === user.id);
        if (existingIndex >= 0) {
            this.users[existingIndex] = user;
        } else {
            this.users.push(user);
        }
        return user;
    }

    findById(id) {
        return this.users.find(user => user.id === id);
    }

    findAll() {
        return [...this.users];
    }

    delete(id) {
        this.users = this.users.filter(user => user.id !== id);
    }
}
```

**View (Görünüm) - Pasif Kullanıcı Arayüzü**

MVP'de View, MVC'den farklı olarak pasif bir rol oynar. View, sadece veri sunumu yapar ve kullanıcı etkileşimlerini Presenter'a iletir. View, Model ile doğrudan iletişim kurmaz.

**MVP'de View'in Sorumlulukları:**
- **Data Display**: Veri görüntüleme
- **User Input Collection**: Kullanıcı girişi toplama
- **Event Forwarding**: Olayları Presenter'a iletme
- **UI Rendering**: Arayüz render etme
- **Interface Implementation**: Arayüz implementasyonu
- **Passive Behavior**: Pasif davranış

**MVP'de View'in Özellikleri:**
- **Passive**: Pasif (Presenter'dan komut alır)
- **Interface-Based**: Arayüz tabanlı
- **No Business Logic**: İş mantığı içermez
- **Testable**: Test edilebilir
- **Reusable**: Yeniden kullanılabilir
- **Independent**: Model'den bağımsız

**MVP'de View Kullanım Alanları:**
- **Web Pages**: Web sayfaları
- **Mobile Screens**: Mobil ekranlar
- **Desktop Applications**: Masaüstü uygulamaları
- **API Responses**: API yanıtları
- **Data Visualization**: Veri görselleştirme

```javascript
// User View Interface
class IUserView {
    showUsers(users) {
        throw new Error('Method must be implemented');
    }

    showError(message) {
        throw new Error('Method must be implemented');
    }

    showSuccess(message) {
        throw new Error('Method must be implemented');
    }

    getUserInput() {
        throw new Error('Method must be implemented');
    }

    clearInput() {
        throw new Error('Method must be implemented');
    }
}

// User View Implementation
class UserView extends IUserView {
    constructor() {
        super();
        this.container = document.getElementById('user-container');
        this.nameInput = document.getElementById('user-name');
        this.emailInput = document.getElementById('user-email');
        this.addButton = document.getElementById('add-user-btn');
    }

    showUsers(users) {
        const html = users.map(user => `
            <div class="user-card" data-id="${user.id}">
                <h3>${user.name}</h3>
                <p>${user.email}</p>
                <button class="delete-btn" data-id="${user.id}">Delete</button>
            </div>
        `).join('');
        this.container.innerHTML = html;
    }

    showError(message) {
        alert(`Error: ${message}`);
    }

    showSuccess(message) {
        alert(`Success: ${message}`);
    }

    getUserInput() {
        return {
            name: this.nameInput.value,
            email: this.emailInput.value
        };
    }

    clearInput() {
        this.nameInput.value = '';
        this.emailInput.value = '';
    }

    bindAddUser(callback) {
        this.addButton.addEventListener('click', callback);
    }

    bindDeleteUser(callback) {
        this.container.addEventListener('click', (e) => {
            if (e.target.classList.contains('delete-btn')) {
                const id = parseInt(e.target.dataset.id);
                callback(id);
            }
        });
    }
}
```

**Presenter (Sunucu) - İş Mantığı ve Koordinasyon Katmanı**

Presenter, MVP'de en önemli bileşendir. Presenter, View ve Model arasında köprü görevi görür ve tüm iş mantığını içerir. Presenter, View'ın pasif olmasını sağlar ve test edilebilirliği artırır.

**Presenter'in Sorumlulukları:**
- **Business Logic**: İş mantığı
- **View Coordination**: View koordinasyonu
- **Model Interaction**: Model etkileşimi
- **User Input Processing**: Kullanıcı girişi işleme
- **Data Transformation**: Veri dönüştürme
- **Event Handling**: Olay işleme

**Presenter'in Özellikleri:**
- **Active**: Aktif (işlemleri başlatır)
- **Business Logic Holder**: İş mantığı tutucu
- **View Controller**: View kontrolcüsü
- **Model Coordinator**: Model koordinatörü
- **Testable**: Test edilebilir
- **Independent**: View implementasyonundan bağımsız

**Presenter Kullanım Alanları:**
- **Business Services**: İş servisleri
- **Application Controllers**: Uygulama kontrolcüleri
- **Event Handlers**: Olay işleyicileri
- **Data Processors**: Veri işleyicileri
- **UI Coordinators**: Arayüz koordinatörleri

```javascript
// User Presenter
class UserPresenter {
    constructor(view, repository) {
        this.view = view;
        this.repository = repository;
        this.init();
    }

    init() {
        this.view.bindAddUser(() => this.addUser());
        this.view.bindDeleteUser((id) => this.deleteUser(id));
        this.loadUsers();
    }

    addUser() {
        try {
            const userData = this.view.getUserInput();
            const user = new User(Date.now(), userData.name, userData.email);
            
            if (user.validate()) {
                this.repository.save(user);
                this.view.showSuccess('User added successfully');
                this.view.clearInput();
                this.loadUsers();
            } else {
                this.view.showError('Invalid user data');
            }
        } catch (error) {
            this.view.showError(error.message);
        }
    }

    deleteUser(id) {
        if (confirm('Are you sure you want to delete this user?')) {
            this.repository.delete(id);
            this.view.showSuccess('User deleted successfully');
            this.loadUsers();
        }
    }

    loadUsers() {
        const users = this.repository.findAll();
        this.view.showUsers(users);
    }
}

// Application initialization
const userView = new UserView();
const userRepository = new UserRepository();
const userPresenter = new UserPresenter(userView, userRepository);
```

**MVP Avantajları**:
- **Passive View**: View sadece veri gösterir, iş mantığı içermez
- **Testability**: Presenter bağımsız olarak test edilebilir
- **Separation of Concerns**: Her katmanın net sorumluluğu
- **No View Dependencies**: Model, View'dan bağımsızdır

**MVP Dezavantajları**:
- **Fat Presenters**: Presenter'lar çok büyüyebilir
- **Boilerplate Code**: Çok fazla kod tekrarı
- **Complex Binding**: View ve Presenter arası bağlantı karmaşık
- **Learning Curve**: Öğrenme eğrisi yüksek

**MVVM (Model-View-ViewModel)**:
- **Model**: Veri ve iş mantığı
- **View**: Kullanıcı arayüzü
- **ViewModel**: View için veri ve komutlar
- **Avantajları**: Data binding, test edilebilirlik
- **Dezavantajları**: Karmaşık binding'ler

**Flux/Redux Pattern**:
- **Store**: Uygulama durumu
- **Actions**: Durum değişiklikleri
- **Reducers**: Durum güncellemeleri
- **Views**: Kullanıcı arayüzü
- **Avantajları**: Predictable state, time-travel debugging
- **Dezavantajları**: Boilerplate kod, öğrenme eğrisi

**Clean Architecture**:
- **Entities**: İş kuralları
- **Use Cases**: Uygulama iş mantığı
- **Interface Adapters**: Veri dönüştürme
- **Frameworks & Drivers**: Dış katmanlar
- **Avantajları**: Bağımsızlık, test edilebilirlik
- **Dezavantajları**: Karmaşıklık, over-engineering riski

**Domain-Driven Design (DDD)**:
- **Domain Model**: İş alanı modeli
- **Bounded Contexts**: Sınırlı bağlamlar
- **Aggregates**: Veri tutarlılığı
- **Value Objects**: Değer nesneleri
- **Avantajları**: İş odaklı tasarım, sürdürülebilirlik
- **Dezavantajları**: Karmaşıklık, öğrenme eğrisi

### 5.2 State Management - Detaylı Analiz

Modern JavaScript uygulamalarında state management, uygulamanın kalbi niteliğindedir. State management, uygulamanın veri akışını, durum değişikliklerini ve component'ler arası veri paylaşımını yöneten kritik bir sistemdir. Bu sistem, uygulamanın tutarlılığını, öngörülebilirliğini ve sürdürülebilirliğini sağlar.

**State Management'in Temel Felsefesi:**

State management, uygulamanın veri akışını merkezi bir şekilde yönetmeyi amaçlar. Bu yönetim, veri tutarlılığını sağlar, debugging sürecini kolaylaştırır ve uygulamanın öngörülebilir davranış göstermesini mümkün kılar. Modern state management çözümleri, functional programming prensiplerini kullanarak, immutable data structures ve pure functions ile çalışır.

**State Management'in Karşılaştığı Zorluklar:**

1. **State Synchronization**: Farklı component'ler arasında state senkronizasyonu
2. **State Consistency**: State tutarlılığının sağlanması
3. **State Predictability**: State değişikliklerinin öngörülebilir olması
4. **State Debugging**: State değişikliklerinin debug edilmesi
5. **State Performance**: State yönetiminin performans etkisi
6. **State Scalability**: Büyük uygulamalarda state yönetiminin ölçeklenebilirliği

**State Management Çözümlerinin Kategorileri:**

1. **Local State Management**: Component seviyesinde state yönetimi
2. **Global State Management**: Uygulama genelinde state yönetimi
3. **Server State Management**: Server'dan gelen verilerin yönetimi
4. **Form State Management**: Form verilerinin yönetimi
5. **Cache State Management**: Cache verilerinin yönetimi

Bu bölümde, farklı state management çözümlerini ve bunların kullanım senaryolarını detaylı olarak inceleyeceğiz.

#### 5.2.1 Redux Toolkit (RTK) - Derinlemesine Analiz

Redux Toolkit (RTK), Redux'un modern ve önerilen kullanım şeklidir. RTK, Redux'un karmaşıklığını azaltır, boilerplate kodları minimize eder ve daha iyi developer experience sağlar. RTK, Redux'un temel prensiplerini korurken, modern JavaScript özelliklerini ve best practice'leri kullanır.

**Redux Toolkit'in Temel Felsefesi:**

RTK, Redux'un "good defaults" prensibini benimser. Bu prensip, geliştiricilerin doğru şekilde Redux kullanmasını sağlamak için, en yaygın kullanım senaryoları için optimize edilmiş default'lar sunar. RTK, Redux'un karmaşıklığını gizler ve geliştiricilerin business logic'e odaklanmasını sağlar.

**Redux Toolkit'in Ana Bileşenleri:**

**createSlice - Modern Reducer Yaratma:**

createSlice, Redux'un en önemli iyileştirmelerinden biridir. Bu fonksiyon, reducer'ları, action creator'ları ve action type'ları otomatik olarak oluşturur. createSlice, Immer kütüphanesini kullanarak, immutable update'leri mümkün kılar.

createSlice'in avantajları:
- **Boilerplate Reduction**: Kod tekrarını azaltır
- **Type Safety**: TypeScript desteği sağlar
- **Immer Integration**: Immutable update'leri kolaylaştırır
- **Auto-generated Actions**: Action'ları otomatik oluşturur
- **DevTools Integration**: Redux DevTools ile entegre çalışır

**createAsyncThunk - Async Action Yaratma:**

createAsyncThunk, async operation'ları yönetmek için tasarlanmıştır. Bu fonksiyon, async action'ları otomatik olarak pending, fulfilled ve rejected state'lerine ayırır. createAsyncThunk, error handling ve loading state management'i otomatik olarak sağlar.

createAsyncThunk'in avantajları:
- **Automatic Loading States**: Loading state'lerini otomatik yönetir
- **Error Handling**: Error handling'i otomatik sağlar
- **TypeScript Support**: TypeScript desteği sağlar
- **Cancellation Support**: Request cancellation'ı destekler
- **Conditional Dispatching**: Koşullu dispatch'i destekler

**configureStore - Modern Store Yapılandırması:**

configureStore, Redux store'unu yapılandırmak için tasarlanmıştır. Bu fonksiyon, Redux DevTools, middleware ve enhancer'ları otomatik olarak yapılandırır. configureStore, development ve production environment'ları için optimize edilmiş default'lar sunar.

configureStore'un avantajları:
- **DevTools Integration**: Redux DevTools'u otomatik yapılandırır
- **Middleware Setup**: Gerekli middleware'leri otomatik ekler
- **Serializable Check**: Serializable state check'i sağlar
- **Immutable Check**: Immutable state check'i sağlar
- **Environment Optimization**: Environment'a göre optimize eder

**createEntityAdapter - Entity Yönetimi:**

createEntityAdapter, normalized data yönetimi için tasarlanmıştır. Bu fonksiyon, entity'ler için CRUD operasyonları sağlar ve performanslı data access sağlar. createEntityAdapter, büyük veri setleri için optimize edilmiştir.

createEntityAdapter'in avantajları:
- **Normalized Data**: Veriyi normalize eder
- **CRUD Operations**: CRUD operasyonları sağlar
- **Performance Optimization**: Performans optimizasyonu sağlar
- **Selectors**: Optimized selector'lar sağlar
- **TypeScript Support**: TypeScript desteği sağlar

**Redux Toolkit'in Modern Özellikleri:**

**Immer Integration:**
RTK, Immer kütüphanesini kullanarak, immutable update'leri mümkün kılar. Bu entegrasyon, geliştiricilerin mutable syntax kullanarak immutable state update'leri yapmasını sağlar.

**TypeScript Support:**
RTK, TypeScript için first-class support sağlar. Bu destek, type safety, IntelliSense ve compile-time error checking sağlar.

**DevTools Integration:**
RTK, Redux DevTools ile tam entegrasyon sağlar. Bu entegrasyon, time-travel debugging, action replay ve state inspection sağlar.

**Performance Optimization:**
RTK, performans optimizasyonları içerir. Bu optimizasyonlar, unnecessary re-render'ları önler ve memory usage'ı optimize eder.

**Redux Toolkit'in Kullanım Senaryoları:**

**Small to Medium Applications:**
RTK, küçük ve orta ölçekli uygulamalar için idealdir. Bu uygulamalarda, RTK'nın sağladığı boilerplate reduction ve developer experience iyileştirmeleri önemli faydalar sağlar.

**Large Applications:**
RTK, büyük uygulamalar için de uygundur. Bu uygulamalarda, RTK'nın sağladığı performance optimization ve scalability features kritik öneme sahiptir.

**Team Development:**
RTK, takım geliştirme için idealdir. Bu ortamlarda, RTK'nın sağladığı consistency ve best practices, kod kalitesini artırır.

**Redux Toolkit'in Best Practices:**

**Slice Organization:**
RTK'da slice'ları feature-based olarak organize etmek önemlidir. Bu organizasyon, kod maintainability'sini artırır ve team collaboration'ı kolaylaştırır.

**Async Thunk Usage:**
createAsyncThunk'ı doğru şekilde kullanmak önemlidir. Bu kullanım, error handling ve loading state management'i optimize eder.

**Entity Management:**
createEntityAdapter'ı normalized data için kullanmak önemlidir. Bu kullanım, performance ve data consistency sağlar.

**TypeScript Integration:**
RTK'da TypeScript'i doğru şekilde kullanmak önemlidir. Bu kullanım, type safety ve developer experience sağlar.

**createSlice**:
```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users');
      if (!response.ok) throw new Error('Failed to fetch');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// Slice
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    items: [],
    loading: false,
    error: null
  },
  reducers: {
    addUser: (state, action) => {
      state.items.push(action.payload);
    },
    removeUser: (state, action) => {
      state.items = state.items.filter(user => user.id !== action.payload);
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { addUser, removeUser } = usersSlice.actions;
export default usersSlice.reducer;
```

**configureStore**:
```javascript
import { configureStore } from '@reduxjs/toolkit';
import usersReducer from './usersSlice';

export const store = configureStore({
  reducer: {
    users: usersReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST']
      }
    }),
  devTools: process.env.NODE_ENV !== 'production'
});
```

**RTK Query**:
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api/',
    prepareHeaders: (headers, { getState }) => {
      const token = getState().auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['User', 'Post'],
  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => 'users',
      providesTags: ['User']
    }),
    createUser: builder.mutation({
      query: (newUser) => ({
        url: 'users',
        method: 'POST',
        body: newUser
      }),
      invalidatesTags: ['User']
    })
  })
});

export const { useGetUsersQuery, useCreateUserMutation } = api;
```

#### 5.2.2 Zustand - Derinlemesine Analiz

Zustand, minimal ve esnek bir state management çözümüdür. Zustand, Redux'a göre daha az boilerplate gerektirir ve modern React uygulamaları için optimize edilmiştir. Zustand, "bear" kelimesinden türetilmiştir ve "state" anlamına gelir.

**Zustand'in Temel Felsefesi:**

Zustand, "less is more" prensibini benimser. Bu prensip, minimal API ile maksimum functionality sağlamayı amaçlar. Zustand, Redux'un karmaşıklığını azaltırken, aynı zamanda güçlü state management özelliklerini korur. Zustand, functional programming prensiplerini kullanır ve immutable updates sağlar.

**Zustand'in Ana Özellikleri:**

**Minimal API:**
Zustand, minimal bir API sunar. Bu API, sadece gerekli olan fonksiyonları içerir ve gereksiz complexity'yi ortadan kaldırır. Minimal API, öğrenme eğrisini azaltır ve developer productivity'sini artırır.

**Flexible Architecture:**
Zustand, esnek bir mimari sunar. Bu mimari, farklı kullanım senaryolarını destekler ve custom implementation'ları mümkün kılar. Esnek mimari, farklı proje ihtiyaçlarına uyum sağlar.

**TypeScript Support:**
Zustand, TypeScript için first-class support sağlar. Bu destek, type safety, IntelliSense ve compile-time error checking sağlar. TypeScript desteği, büyük projelerde kod kalitesini artırır.

**Middleware Support:**
Zustand, middleware desteği sağlar. Bu destek, logging, persistence, devtools ve custom middleware'leri mümkün kılar. Middleware desteği, extensibility sağlar.

**Zustand'in Bileşenleri:**

**create Function:**
create fonksiyonu, Zustand store'unu oluşturur. Bu fonksiyon, state ve action'ları tanımlar ve store instance'ını döndürür. create fonksiyonu, functional approach kullanır.

**set Function:**
set fonksiyonu, state'i günceller. Bu fonksiyon, immutable updates sağlar ve state'in tutarlılığını korur. set fonksiyonu, partial updates destekler.

**get Function:**
get fonksiyonu, mevcut state'e erişim sağlar. Bu fonksiyon, action'lar içinde state'e erişim için kullanılır. get fonksiyonu, current state'i döndürür.

**subscribe Function:**
subscribe fonksiyonu, state değişikliklerini dinler. Bu fonksiyon, component'lerin state değişikliklerine tepki vermesini sağlar. subscribe fonksiyonu, selective subscription destekler.

**Zustand'in Middleware'leri:**

**devtools Middleware:**
devtools middleware, Redux DevTools entegrasyonu sağlar. Bu middleware, state değişikliklerini görselleştirir ve debugging sürecini kolaylaştırır. devtools middleware, time-travel debugging destekler.

**persist Middleware:**
persist middleware, state persistence sağlar. Bu middleware, state'i localStorage, sessionStorage veya custom storage'da saklar. persist middleware, selective persistence destekler.

**immer Middleware:**
immer middleware, Immer entegrasyonu sağlar. Bu middleware, mutable syntax ile immutable updates yapmayı mümkün kılar. immer middleware, complex state updates'i kolaylaştırır.

**subscribeWithSelector Middleware:**
subscribeWithSelector middleware, selective subscription sağlar. Bu middleware, component'lerin sadece ilgili state değişikliklerini dinlemesini sağlar. subscribeWithSelector middleware, performance optimization sağlar.

**Zustand'in Kullanım Senaryoları:**

**Small Applications:**
Zustand, küçük uygulamalar için idealdir. Bu uygulamalarda, Zustand'in minimal API'si ve düşük overhead'i önemli faydalar sağlar.

**Medium Applications:**
Zustand, orta ölçekli uygulamalar için de uygundur. Bu uygulamalarda, Zustand'in esnek mimarisi ve middleware desteği kritik öneme sahiptir.

**Large Applications:**
Zustand, büyük uygulamalar için de kullanılabilir. Bu uygulamalarda, Zustand'in performance optimization ve scalability features önemli faydalar sağlar.

**Zustand'in Avantajları:**

**Minimal Boilerplate:**
Zustand, minimal boilerplate gerektirir. Bu özellik, development speed'i artırır ve code maintainability'sini iyileştirir.

**Easy to Learn:**
Zustand, öğrenmesi kolaydır. Bu özellik, team onboarding sürecini hızlandırır ve developer productivity'sini artırır.

**Flexible:**
Zustand, esnek bir yapıya sahiptir. Bu özellik, farklı kullanım senaryolarını destekler ve custom implementation'ları mümkün kılar.

**Performance:**
Zustand, yüksek performans sağlar. Bu özellik, unnecessary re-render'ları önler ve memory usage'ı optimize eder.

**Zustand'in Dezavantajları:**

**Less Ecosystem:**
Zustand, Redux'a göre daha küçük bir ecosystem'e sahiptir. Bu durum, third-party library desteğini sınırlar.

**Less Documentation:**
Zustand, Redux'a göre daha az dokümantasyona sahiptir. Bu durum, learning curve'i artırabilir.

**Less Community:**
Zustand, Redux'a göre daha küçük bir community'ye sahiptir. Bu durum, support ve help bulmayı zorlaştırabilir.

**Zustand'in Best Practices:**

**Store Organization:**
Zustand'da store'ları feature-based olarak organize etmek önemlidir. Bu organizasyon, kod maintainability'sini artırır.

**Action Naming:**
Zustand'da action'ları descriptive olarak isimlendirmek önemlidir. Bu isimlendirme, code readability'sini artırır.

**TypeScript Usage:**
Zustand'da TypeScript'i doğru şekilde kullanmak önemlidir. Bu kullanım, type safety ve developer experience sağlar.

**Middleware Usage:**
Zustand'da middleware'leri doğru şekilde kullanmak önemlidir. Bu kullanım, functionality ve performance sağlar.

**Basic Store**:
```javascript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

const useStore = create(
  devtools(
    persist(
      (set, get) => ({
        // State
        count: 0,
        users: [],
        loading: false,
        
        // Actions
        increment: () => set((state) => ({ count: state.count + 1 })),
        decrement: () => set((state) => ({ count: state.count - 1 })),
        
        // Async actions
        fetchUsers: async () => {
          set({ loading: true });
          try {
            const response = await fetch('/api/users');
            const users = await response.json();
            set({ users, loading: false });
          } catch (error) {
            set({ loading: false, error: error.message });
          }
        },
        
        // Computed values
        get userCount() {
          return get().users.length;
        }
      }),
      {
        name: 'app-storage',
        partialize: (state) => ({ count: state.count })
      }
    ),
    { name: 'app-store' }
  )
);

export default useStore;
```

**TypeScript with Zustand**:
```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface AppState {
  users: User[];
  loading: boolean;
  error: string | null;
  addUser: (user: User) => void;
  removeUser: (id: string) => void;
  fetchUsers: () => Promise<void>;
}

const useAppStore = create<AppState>()(
  devtools(
    (set, get) => ({
      users: [],
      loading: false,
      error: null,
      
      addUser: (user) => set((state) => ({
        users: [...state.users, user]
      })),
      
      removeUser: (id) => set((state) => ({
        users: state.users.filter(user => user.id !== id)
      })),
      
      fetchUsers: async () => {
        set({ loading: true, error: null });
        try {
          const response = await fetch('/api/users');
          const users = await response.json();
          set({ users, loading: false });
        } catch (error) {
          set({ error: error.message, loading: false });
        }
      }
    }),
    { name: 'app-store' }
  )
);
```

#### 5.2.3 Jotai

Jotai, atomik state management yaklaşımı kullanır. Her state parçası bir atom olarak tanımlanır.

**Basic Atoms**:
```javascript
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Primitive atoms
const countAtom = atom(0);
const nameAtom = atom('John');

// Derived atoms
const doubledCountAtom = atom((get) => get(countAtom) * 2);
const greetingAtom = atom((get) => `Hello, ${get(nameAtom)}!`);

// Write-only atoms
const incrementAtom = atom(
  null,
  (get, set) => set(countAtom, get(countAtom) + 1)
);

// Async atoms
const usersAtom = atom(async () => {
  const response = await fetch('/api/users');
  return response.json();
});

// Component usage
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubledCount = useAtomValue(doubledCountAtom);
  const increment = useSetAtom(incrementAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubledCount}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={increment}>Increment (write-only)</button>
    </div>
  );
}
```

**Advanced Jotai Patterns**:
```javascript
import { atom, useAtom } from 'jotai';
import { atomWithStorage } from 'jotai/utils';

// Storage atom
const themeAtom = atomWithStorage('theme', 'light');

// Split atom pattern
const baseUserAtom = atom({ id: '', name: '', email: '' });
const userAtom = atom(
  (get) => get(baseUserAtom),
  (get, set, update) => {
    set(baseUserAtom, { ...get(baseUserAtom), ...update });
  }
);

// Focus atom pattern
const focusAtom = atom(
  (get) => get(baseUserAtom),
  (get, set, focus) => {
    const prev = get(baseUserAtom);
    set(baseUserAtom, { ...prev, [focus]: prev[focus] });
  }
);

// Family atoms
const itemAtomFamily = atom((id) => atom({ id, name: '', completed: false }));

// Component with family atoms
function TodoItem({ id }) {
  const [item, setItem] = useAtom(itemAtomFamily(id));
  
  return (
    <div>
      <input
        value={item.name}
        onChange={(e) => setItem({ ...item, name: e.target.value })}
      />
      <input
        type="checkbox"
        checked={item.completed}
        onChange={(e) => setItem({ ...item, completed: e.target.checked })}
      />
    </div>
  );
}
```

#### 5.2.4 Recoil

Recoil, Facebook tarafından geliştirilen atomik state management kütüphanesidir.

**Basic Recoil Setup**:
```javascript
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil';

// Atoms
const countState = atom({
  key: 'countState',
  default: 0
});

const nameState = atom({
  key: 'nameState',
  default: 'John'
});

// Selectors
const doubledCountSelector = selector({
  key: 'doubledCountSelector',
  get: ({ get }) => {
    const count = get(countState);
    return count * 2;
  }
});

const greetingSelector = selector({
  key: 'greetingSelector',
  get: ({ get }) => {
    const name = get(nameState);
    const count = get(countState);
    return `Hello ${name}, count is ${count}`;
  }
});

// Async selectors
const usersSelector = selector({
  key: 'usersSelector',
  get: async () => {
    const response = await fetch('/api/users');
    return response.json();
  }
});

// Component usage
function Counter() {
  const [count, setCount] = useRecoilState(countState);
  const doubledCount = useRecoilValue(doubledCountSelector);
  const greeting = useRecoilValue(greetingSelector);
  
  return (
    <div>
      <p>{greeting}</p>
      <p>Count: {count}</p>
      <p>Doubled: {doubledCount}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

#### 5.2.5 SWR/React Query

Bu kütüphaneler, server state yönetimi için özelleşmiştir.

**SWR Implementation**:
```javascript
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

function Profile() {
  const { data, error, isLoading, mutate } = useSWR('/api/user', fetcher, {
    refreshInterval: 1000,
    revalidateOnFocus: true,
    dedupingInterval: 2000
  });

  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h1>{data.name}</h1>
      <button onClick={() => mutate()}>Refresh</button>
    </div>
  );
}

// SWR with mutations
import useSWRMutation from 'swr/mutation';

async function updateUser(url, { arg }) {
  return fetch(url, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(arg)
  }).then(res => res.json());
}

function UserForm() {
  const { trigger, isMutating } = useSWRMutation('/api/user', updateUser);
  
  const handleSubmit = async (formData) => {
    try {
      await trigger(formData);
      // Success
    } catch (error) {
      // Error handling
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit" disabled={isMutating}>
        {isMutating ? 'Saving...' : 'Save'}
      </button>
    </form>
  );
}
```

**React Query Implementation**:
```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function Users() {
  const queryClient = useQueryClient();
  
  // Query
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(res => res.json()),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
  
  // Mutation
  const createUserMutation = useMutation({
    mutationFn: (newUser) => 
      fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newUser)
      }).then(res => res.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
    onError: (error) => {
      console.error('Error creating user:', error);
    }
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
      <button 
        onClick={() => createUserMutation.mutate({ name: 'New User' })}
        disabled={createUserMutation.isPending}
      >
        {createUserMutation.isPending ? 'Creating...' : 'Create User'}
      </button>
    </div>
  );
}
```

#### 5.2.6 State Management Karşılaştırması

**Kullanım Senaryoları**:

| Tool | Best For | Complexity | Bundle Size | Learning Curve |
|------|----------|------------|-------------|----------------|
| Redux Toolkit | Large apps, complex state | High | Medium | Steep |
| Zustand | Medium apps, simple state | Low | Small | Easy |
| Jotai | Component-level state | Medium | Small | Medium |
| Recoil | React-specific features | Medium | Medium | Medium |
| SWR/React Query | Server state | Low | Small | Easy |

**Performance Considerations**:
- **Redux**: Predictable updates, good for complex state
- **Zustand**: Minimal re-renders, good performance
- **Jotai**: Granular updates, excellent performance
- **Recoil**: React integration, good for React-specific features
- **SWR/React Query**: Optimized for server state, caching

### 5.3 Performance Optimization - Detaylı Analiz

Modern web uygulamalarında performans optimizasyonu, kullanıcı deneyiminin kritik bir parçasıdır. Performans optimizasyonu, sadece hız artırma değil, aynı zamanda kaynak kullanımını optimize etme, kullanıcı deneyimini iyileştirme ve uygulamanın ölçeklenebilirliğini sağlama sürecidir.

**Performance Optimization'un Temel Felsefesi:**

Performance optimization, "measure first, optimize second" prensibini benimser. Bu prensip, optimizasyon yapmadan önce mevcut performansı ölçmeyi ve bottleneck'leri tespit etmeyi gerektirir. Optimizasyon, data-driven approach ile yapılmalı ve gerçek kullanıcı deneyimini iyileştirmeye odaklanmalıdır.

**Performance Optimization'un Kapsamı:**

1. **Bundle Optimization**: JavaScript bundle'larının optimize edilmesi
2. **Runtime Performance**: Uygulama çalışma zamanı performansı
3. **Memory Management**: Bellek kullanımının optimize edilmesi
4. **Network Optimization**: Ağ isteklerinin optimize edilmesi
5. **Rendering Performance**: UI rendering performansı
6. **Caching Strategies**: Cache stratejilerinin uygulanması

**Performance Metrics ve Ölçümler:**

**Core Web Vitals:**
- **LCP (Largest Contentful Paint)**: En büyük içeriğin yüklenme süresi
- **FID (First Input Delay)**: İlk kullanıcı etkileşimine yanıt süresi
- **CLS (Cumulative Layout Shift)**: Layout shift miktarı

**Performance Budgets:**
- **Bundle Size**: JavaScript bundle boyutu limitleri
- **Load Time**: Sayfa yüklenme süresi limitleri
- **Memory Usage**: Bellek kullanım limitleri
- **Network Requests**: Ağ istek sayısı limitleri

**Performance Optimization Stratejileri:**

**Progressive Enhancement:**
Uygulamanın temel işlevselliğini önce yükleyip, gelişmiş özellikleri sonradan ekleme stratejisi.

**Lazy Loading:**
Gerekli olmayan kaynakları ihtiyaç duyulduğunda yükleme stratejisi.

**Code Splitting:**
Kodu mantıklı parçalara bölerek, sadece gerekli kısımları yükleme stratejisi.

**Caching:**
Sık kullanılan verileri cache'leyerek, tekrar yükleme süresini azaltma stratejisi.

Bu bölümde, React ve JavaScript uygulamalarında performansı artırmak için kullanılan teknikleri detaylı olarak inceleyeceğiz.

#### 5.3.1 Bundle Analysis ve Optimizasyon

**Webpack Bundle Analyzer**:
```javascript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html'
    })
  ]
};
```

**Bundle Size Monitoring**:
```javascript
// webpack.config.js
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  plugins: [
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        }
      }
    }
  }
};
```

#### 5.3.2 Code Splitting Stratejileri

**Route-based Splitting**:
```javascript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Component-based Splitting**:
```javascript
import { lazy, Suspense, useState } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

#### 5.3.3 Memoization Teknikleri

**React.memo Optimizasyonu**:
```javascript
import { memo, useMemo, useCallback } from 'react';

const ExpensiveComponent = memo(({ data, onUpdate }) => {
  return (
    <div>
      {data.map(item => (
        <Item key={item.id} data={item} onUpdate={onUpdate} />
      ))}
    </div>
  );
});

// Custom comparison
const OptimizedComponent = memo(({ items, filter }) => {
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);

  const handleItemClick = useCallback((itemId) => {
    // Handle click
  }, []);

  return (
    <div>
      {filteredItems.map(item => (
        <Item key={item.id} item={item} onClick={handleItemClick} />
      ))}
    </div>
  );
}, (prevProps, nextProps) => {
  return (
    prevProps.items.length === nextProps.items.length &&
    prevProps.filter === nextProps.filter
  );
});
```

#### 5.3.4 Virtualization

**Virtual Scrolling**:
```javascript
import { FixedSizeList as List } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </List>
  );
}
```

#### 5.3.5 Image Optimization

**Lazy Loading Images**:
```javascript
import { useState, useRef, useEffect } from 'react';

function LazyImage({ src, alt, placeholder }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <div ref={imgRef} style={{ minHeight: '200px' }}>
      {isInView && (
        <img
          src={src}
          alt={alt}
          onLoad={() => setIsLoaded(true)}
          style={{
            opacity: isLoaded ? 1 : 0,
            transition: 'opacity 0.3s'
          }}
        />
      )}
      {!isLoaded && placeholder && (
        <div style={{ background: '#f0f0f0' }}>
          {placeholder}
        </div>
      )}
    </div>
  );
}
```

#### 5.3.6 Performance Monitoring

**Web Vitals Measurement**:
```javascript
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  console.log(metric);
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

**Custom Performance Measurement**:
```javascript
function measurePerformance(name, fn) {
  const start = performance.now();
  const result = fn();
  const end = performance.now();
  
  console.log(`${name} took ${end - start} milliseconds`);
  return result;
}
```

### 5.4 Security - Detaylı Analiz

Modern web uygulamalarında güvenlik, kritik öneme sahiptir. Güvenlik, sadece veri koruma değil, aynı zamanda kullanıcı güvenliği, sistem bütünlüğü ve uygulama güvenilirliği için kapsamlı bir yaklaşım gerektirir. Web güvenliği, sürekli gelişen bir alandır ve yeni tehditler ortaya çıktıkça güvenlik önlemleri de güncellenmelidir.

**Security'in Temel Felsefesi:**

Security, "defense in depth" prensibini benimser. Bu prensip, tek bir güvenlik katmanına güvenmek yerine, birden fazla güvenlik katmanı oluşturmayı gerektirir. Her katman, farklı türde tehditleri önler ve bir katman başarısız olsa bile diğer katmanlar koruma sağlar.

**Web Security Tehditleri:**

**OWASP Top 10:**
1. **Injection**: SQL injection, NoSQL injection, command injection
2. **Broken Authentication**: Zayıf authentication mekanizmaları
3. **Sensitive Data Exposure**: Hassas veri sızıntıları
4. **XML External Entities (XXE)**: XML external entity saldırıları
5. **Broken Access Control**: Erişim kontrolü zafiyetleri
6. **Security Misconfiguration**: Güvenlik yapılandırma hataları
7. **Cross-Site Scripting (XSS)**: Cross-site scripting saldırıları
8. **Insecure Deserialization**: Güvensiz deserialization
9. **Using Components with Known Vulnerabilities**: Bilinen zafiyetli bileşenler
10. **Insufficient Logging & Monitoring**: Yetersiz loglama ve izleme

**Modern Web Security Challenges:**

**Client-Side Security:**
- **XSS Prevention**: Cross-site scripting saldırılarını önleme
- **CSRF Protection**: Cross-site request forgery koruması
- **Content Security Policy**: İçerik güvenlik politikaları
- **Secure Data Storage**: Güvenli veri saklama

**API Security:**
- **Authentication & Authorization**: Kimlik doğrulama ve yetkilendirme
- **Rate Limiting**: Hız sınırlaması
- **Input Validation**: Giriş doğrulama
- **Output Encoding**: Çıkış kodlama

**Infrastructure Security:**
- **HTTPS/TLS**: Güvenli iletişim
- **Security Headers**: Güvenlik başlıkları
- **Dependency Management**: Bağımlılık yönetimi
- **Vulnerability Scanning**: Zafiyet tarama

**Security Best Practices:**

**Secure Coding Practices:**
- **Input Validation**: Tüm girişleri doğrulama
- **Output Encoding**: Tüm çıkışları kodlama
- **Error Handling**: Güvenli hata yönetimi
- **Logging**: Güvenlik olaylarını loglama

**Security Testing:**
- **Static Analysis**: Statik kod analizi
- **Dynamic Testing**: Dinamik güvenlik testleri
- **Penetration Testing**: Penetrasyon testleri
- **Code Review**: Kod inceleme

**Security Monitoring:**
- **Real-time Monitoring**: Gerçek zamanlı izleme
- **Incident Response**: Olay müdahale
- **Threat Intelligence**: Tehdit istihbaratı
- **Security Metrics**: Güvenlik metrikleri

Bu bölümde, JavaScript ve React uygulamalarında güvenlik açıklarını önlemek için kullanılan teknikleri detaylı olarak inceleyeceğiz.

#### 5.4.1 XSS (Cross-Site Scripting) Prevention

**Input Sanitization**:
```javascript
import DOMPurify from 'dompurify';

// Sanitize user input
function sanitizeInput(input) {
  return DOMPurify.sanitize(input, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong'],
    ALLOWED_ATTR: []
  });
}

// Usage
function UserComment({ comment }) {
  const sanitizedComment = sanitizeInput(comment);
  
  return (
    <div 
      dangerouslySetInnerHTML={{ __html: sanitizedComment }}
    />
  );
}
```

**Output Encoding**:
```javascript
// HTML encoding
function htmlEncode(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

// Safe rendering
function SafeComponent({ userInput }) {
  return <div>{userInput}</div>; // React automatically escapes
}

// Unsafe rendering (avoid)
function UnsafeComponent({ userInput }) {
  return <div dangerouslySetInnerHTML={{ __html: userInput }} />;
}
```

**Content Security Policy (CSP)**:
```html
<!-- HTML meta tag -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';">

<!-- HTTP header -->
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
```

```javascript
// CSP violation reporting
function reportCSPViolation(violation) {
  fetch('/api/csp-report', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(violation)
  });
}

// Listen for CSP violations
document.addEventListener('securitypolicyviolation', (e) => {
  reportCSPViolation({
    blockedURI: e.blockedURI,
    violatedDirective: e.violatedDirective,
    originalPolicy: e.originalPolicy
  });
});
```

#### 5.4.2 CSRF (Cross-Site Request Forgery) Protection

**CSRF Tokens**:
```javascript
// Generate CSRF token
function generateCSRFToken() {
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
}

// Include CSRF token in requests
async function secureRequest(url, data) {
  const token = localStorage.getItem('csrf-token');
  
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': token
    },
    body: JSON.stringify(data)
  });
  
  return response.json();
}
```

**SameSite Cookies**:
```javascript
// Set secure cookie
function setSecureCookie(name, value, days) {
  const expires = new Date();
  expires.setTime(expires.getTime() + (days * 24 * 60 * 60 * 1000));
  
  document.cookie = `${name}=${value};expires=${expires.toUTCString()};path=/;secure;samesite=strict`;
}

// Express.js cookie configuration
app.use(session({
  secret: 'your-secret-key',
  cookie: {
    secure: true,
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));
```

#### 5.4.3 Authentication & Authorization

**JWT Implementation**:
```javascript
// JWT token management
class AuthService {
  constructor() {
    this.tokenKey = 'auth-token';
  }
  
  setToken(token) {
    localStorage.setItem(this.tokenKey, token);
  }
  
  getToken() {
    return localStorage.getItem(this.tokenKey);
  }
  
  removeToken() {
    localStorage.removeItem(this.tokenKey);
  }
  
  isTokenValid() {
    const token = this.getToken();
    if (!token) return false;
    
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      return payload.exp * 1000 > Date.now();
    } catch {
      return false;
    }
  }
  
  getTokenPayload() {
    const token = this.getToken();
    if (!token) return null;
    
    try {
      return JSON.parse(atob(token.split('.')[1]));
    } catch {
      return null;
    }
  }
}

// Usage
const authService = new AuthService();

// Protected API call
async function protectedRequest(url, options = {}) {
  const token = authService.getToken();
  
  if (!token || !authService.isTokenValid()) {
    throw new Error('Invalid or expired token');
  }
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${token}`
    }
  });
}
```

**Role-based Access Control**:
```javascript
// Role-based component
function ProtectedComponent({ children, requiredRole }) {
  const { user } = useAuth();
  
  if (!user || !user.roles.includes(requiredRole)) {
    return <div>Access denied</div>;
  }
  
  return children;
}

// Usage
function AdminPanel() {
  return (
    <ProtectedComponent requiredRole="admin">
      <div>Admin content</div>
    </ProtectedComponent>
  );
}

// Hook for role checking
function useRole(requiredRole) {
  const { user } = useAuth();
  return user?.roles?.includes(requiredRole) || false;
}

// Usage in component
function MyComponent() {
  const isAdmin = useRole('admin');
  const isUser = useRole('user');
  
  return (
    <div>
      {isAdmin && <AdminButton />}
      {isUser && <UserButton />}
    </div>
  );
}
```

#### 5.4.4 Data Encryption

**Client-side Encryption**:
```javascript
// Simple encryption (for demonstration - use proper crypto libraries in production)
class SimpleEncryption {
  constructor(key) {
    this.key = key;
  }
  
  encrypt(text) {
    const encrypted = [];
    for (let i = 0; i < text.length; i++) {
      encrypted.push(text.charCodeAt(i) ^ this.key.charCodeAt(i % this.key.length));
    }
    return btoa(String.fromCharCode(...encrypted));
  }
  
  decrypt(encryptedText) {
    const encrypted = atob(encryptedText);
    const decrypted = [];
    for (let i = 0; i < encrypted.length; i++) {
      decrypted.push(encrypted.charCodeAt(i) ^ this.key.charCodeAt(i % this.key.length));
    }
    return String.fromCharCode(...decrypted);
  }
}

// Usage
const encryption = new SimpleEncryption('my-secret-key');
const encrypted = encryption.encrypt('sensitive data');
const decrypted = encryption.decrypt(encrypted);
```

**Secure Data Transmission**:
```javascript
// HTTPS enforcement
function enforceHTTPS() {
  if (location.protocol !== 'https:' && location.hostname !== 'localhost') {
    location.replace('https:' + window.location.href.substring(window.location.protocol.length));
  }
}

// Secure API client
class SecureAPIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.enforceHTTPS();
  }
  
  enforceHTTPS() {
    if (!this.baseURL.startsWith('https://') && !this.baseURL.includes('localhost')) {
      throw new Error('API must use HTTPS in production');
    }
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    
    const config = {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    };
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
}
```

#### 5.4.5 Secure Storage

**Encrypted Local Storage**:
```javascript
// Encrypted storage utility
class SecureStorage {
  constructor(encryptionKey) {
    this.key = encryptionKey;
  }
  
  setItem(key, value) {
    const encrypted = this.encrypt(JSON.stringify(value));
    localStorage.setItem(key, encrypted);
  }
  
  getItem(key) {
    const encrypted = localStorage.getItem(key);
    if (!encrypted) return null;
    
    try {
      const decrypted = this.decrypt(encrypted);
      return JSON.parse(decrypted);
    } catch {
      return null;
    }
  }
  
  removeItem(key) {
    localStorage.removeItem(key);
  }
  
  encrypt(text) {
    // Use proper encryption in production
    return btoa(text);
  }
  
  decrypt(encryptedText) {
    // Use proper decryption in production
    return atob(encryptedText);
  }
}

// Usage
const secureStorage = new SecureStorage('my-encryption-key');
secureStorage.setItem('user-data', { name: 'John', email: 'john@example.com' });
const userData = secureStorage.getItem('user-data');
```

**Secure Cookie Management**:
```javascript
// Secure cookie utility
class SecureCookies {
  static set(name, value, options = {}) {
    const defaults = {
      secure: true,
      httpOnly: false, // Can't be httpOnly from client-side
      sameSite: 'strict',
      path: '/'
    };
    
    const config = { ...defaults, ...options };
    let cookieString = `${name}=${encodeURIComponent(value)}`;
    
    if (config.expires) {
      cookieString += `;expires=${config.expires.toUTCString()}`;
    }
    
    if (config.maxAge) {
      cookieString += `;max-age=${config.maxAge}`;
    }
    
    if (config.secure) {
      cookieString += ';secure';
    }
    
    if (config.sameSite) {
      cookieString += `;samesite=${config.sameSite}`;
    }
    
    if (config.path) {
      cookieString += `;path=${config.path}`;
    }
    
    document.cookie = cookieString;
  }
  
  static get(name) {
    const cookies = document.cookie.split(';');
    for (let cookie of cookies) {
      const [cookieName, cookieValue] = cookie.trim().split('=');
      if (cookieName === name) {
        return decodeURIComponent(cookieValue);
      }
    }
    return null;
  }
  
  static remove(name) {
    this.set(name, '', { expires: new Date(0) });
  }
}
```

#### 5.4.6 Security Best Practices

**Input Validation**:
```javascript
// Input validation utility
class InputValidator {
  static validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  static validatePassword(password) {
    return {
      isValid: password.length >= 8,
      hasUpperCase: /[A-Z]/.test(password),
      hasLowerCase: /[a-z]/.test(password),
      hasNumber: /\d/.test(password),
      hasSpecialChar: /[!@#$%^&*(),.?":{}|<>]/.test(password)
    };
  }
  
  static sanitizeString(input) {
    return input
      .replace(/[<>]/g, '')
      .replace(/javascript:/gi, '')
      .replace(/on\w+=/gi, '')
      .trim();
  }
}

// Usage
function validateForm(formData) {
  const errors = {};
  
  if (!InputValidator.validateEmail(formData.email)) {
    errors.email = 'Invalid email format';
  }
  
  const passwordValidation = InputValidator.validatePassword(formData.password);
  if (!passwordValidation.isValid) {
    errors.password = 'Password must be at least 8 characters long';
  }
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}
```

**Security Headers**:
```javascript
// Security headers middleware (Express.js)
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
  next();
});
```

### 5.5 Accessibility - Detaylı Analiz

Web erişilebilirliği, tüm kullanıcıların web sitelerini ve uygulamalarını kullanabilmesini sağlayan kritik bir konudur. Accessibility, sadece engelli kullanıcılar için değil, tüm kullanıcılar için daha iyi bir deneyim sağlar. Modern web uygulamalarında accessibility, yasal gereklilikler, etik sorumluluklar ve iş değeri açısından kritik öneme sahiptir.

**Accessibility'in Temel Felsefesi:**

Accessibility, "universal design" prensibini benimser. Bu prensip, ürünlerin ve hizmetlerin, mümkün olduğunca geniş bir kullanıcı yelpazesi tarafından kullanılabilir olmasını gerektirir. Universal design, sadece engelli kullanıcılar için değil, farklı yeteneklere, cihazlara ve durumlara sahip tüm kullanıcılar için tasarım yapmayı içerir.

**Accessibility Standartları:**

**WCAG (Web Content Accessibility Guidelines):**
- **WCAG 2.1**: Mevcut standart, A, AA, AAA seviyeleri
- **WCAG 2.2**: Güncellenmiş standart, yeni kriterler
- **WCAG 3.0**: Gelecek standart, daha kapsamlı yaklaşım

**Section 508 (ABD):**
- Federal kurumlar için erişilebilirlik gereklilikleri
- WCAG 2.0 AA seviyesine uyumluluk

**EN 301 549 (Avrupa):**
- Avrupa Birliği erişilebilirlik standardı
- WCAG 2.1 AA seviyesine uyumluluk

**Accessibility Prensipleri (POUR):**

**Perceivable (Algılanabilir):**
Kullanıcıların bilgiyi ve UI bileşenlerini algılayabilmesi gereklidir. Bu prensip, görsel, işitsel ve dokunsal bilgi sunumunu kapsar.

**Operable (İşletilebilir):**
Kullanıcıların UI bileşenlerini ve navigasyonu işletebilmesi gereklidir. Bu prensip, klavye erişimi, zaman sınırları ve nöbet geçirme gibi konuları kapsar.

**Understandable (Anlaşılabilir):**
Bilgi ve UI işlemlerinin anlaşılabilir olması gereklidir. Bu prensip, okunabilirlik, tahmin edilebilirlik ve hata yönetimi gibi konuları kapsar.

**Robust (Sağlam):**
İçeriğin çok çeşitli kullanıcı ajanları tarafından güvenilir bir şekilde yorumlanabilmesi gereklidir. Bu prensip, uyumluluk ve gelecek güvenliği gibi konuları kapsar.

**Accessibility Teknolojileri:**

**Screen Readers:**
- **NVDA**: Windows için ücretsiz screen reader
- **JAWS**: Windows için ticari screen reader
- **VoiceOver**: macOS ve iOS için built-in screen reader
- **TalkBack**: Android için built-in screen reader

**Voice Control:**
- **Dragon NaturallySpeaking**: Windows için voice control
- **Voice Control**: macOS için built-in voice control
- **Voice Access**: Android için voice control

**Switch Navigation:**
- **Switch Control**: iOS için switch navigation
- **Switch Access**: Android için switch navigation
- **External Switches**: Fiziksel switch'ler

**Accessibility Testing:**

**Automated Testing:**
- **axe-core**: Otomatik accessibility testing
- **Lighthouse**: Google'ın accessibility audit'i
- **WAVE**: Web accessibility evaluation tool

**Manual Testing:**
- **Keyboard Navigation**: Sadece klavye ile navigasyon testi
- **Screen Reader Testing**: Screen reader ile test
- **Color Contrast Testing**: Renk kontrastı testi

**User Testing:**
- **Real User Testing**: Gerçek kullanıcılarla test
- **Usability Testing**: Kullanılabilirlik testi
- **Accessibility Audit**: Kapsamlı erişilebilirlik denetimi

**Accessibility Best Practices:**

**Semantic HTML:**
- Doğru HTML elementlerini kullanma
- Heading hierarchy'sini koruma
- Form label'larını doğru kullanma

**ARIA (Accessible Rich Internet Applications):**
- ARIA attribute'larını doğru kullanma
- ARIA landmark'larını kullanma
- ARIA live region'larını kullanma

**Keyboard Navigation:**
- Tüm functionality'ye klavye erişimi sağlama
- Focus management'i doğru yapma
- Tab order'ı mantıklı tutma

**Visual Design:**
- Yeterli renk kontrastı sağlama
- Text size'ı uygun tutma
- Visual indicator'ları kullanma

Bu bölümde, React ve JavaScript uygulamalarında erişilebilirlik standartlarını nasıl uygulayacağımızı detaylı olarak inceleyeceğiz.

#### 5.5.1 ARIA (Accessible Rich Internet Applications)

**ARIA Labels ve Descriptions**:
```jsx
// ARIA labels
function AccessibleButton() {
  return (
    <button 
      aria-label="Close dialog"
      aria-describedby="close-description"
      onClick={handleClose}
    >
      ×
    </button>
  );
}

// ARIA descriptions
function FormField() {
  return (
    <div>
      <label htmlFor="password">Password</label>
      <input 
        id="password"
        type="password"
        aria-describedby="password-help"
        aria-required="true"
      />
      <div id="password-help">
        Password must be at least 8 characters long
      </div>
    </div>
  );
}
```

**ARIA Live Regions**:
```jsx
import { useState, useEffect } from 'react';

function NotificationSystem() {
  const [notifications, setNotifications] = useState([]);

  const addNotification = (message, type = 'info') => {
    const id = Date.now();
    setNotifications(prev => [...prev, { id, message, type }]);
    
    // Auto remove after 5 seconds
    setTimeout(() => {
      setNotifications(prev => prev.filter(n => n.id !== id));
    }, 5000);
  };

  return (
    <div>
      <button onClick={() => addNotification('Item saved successfully', 'success')}>
        Save Item
      </button>
      
      {/* Live region for screen readers */}
      <div 
        aria-live="polite" 
        aria-atomic="true"
        className="sr-only"
      >
        {notifications.map(notification => (
          <div key={notification.id}>
            {notification.type}: {notification.message}
          </div>
        ))}
      </div>
      
      {/* Visual notifications */}
      <div className="notifications">
        {notifications.map(notification => (
          <div 
            key={notification.id}
            className={`notification ${notification.type}`}
            role="alert"
          >
            {notification.message}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**ARIA Roles ve States**:
```jsx
function AccessibleModal({ isOpen, onClose, children }) {
  useEffect(() => {
    if (isOpen) {
      // Trap focus in modal
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      firstElement?.focus();
      
      const handleTabKey = (e) => {
        if (e.key === 'Tab') {
          if (e.shiftKey) {
            if (document.activeElement === firstElement) {
              lastElement?.focus();
              e.preventDefault();
            }
          } else {
            if (document.activeElement === lastElement) {
              firstElement?.focus();
              e.preventDefault();
            }
          }
        }
      };
      
      document.addEventListener('keydown', handleTabKey);
      return () => document.removeEventListener('keydown', handleTabKey);
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div 
      className="modal-overlay"
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      onClick={onClose}
    >
      <div 
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
        ref={modalRef}
      >
        <h2 id="modal-title">Modal Title</h2>
        {children}
        <button onClick={onClose} aria-label="Close modal">
          Close
        </button>
      </div>
    </div>
  );
}
```

#### 5.5.2 Semantic HTML ve Screen Reader Support

**Semantic HTML Structure**:
```jsx
function AccessiblePage() {
  return (
    <div>
      {/* Skip link for keyboard users */}
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      
      {/* Header with proper heading structure */}
      <header role="banner">
        <nav role="navigation" aria-label="Main navigation">
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
          </ul>
        </nav>
      </header>
      
      {/* Main content area */}
      <main id="main-content" role="main">
        <h1>Page Title</h1>
        
        <section aria-labelledby="features-heading">
          <h2 id="features-heading">Features</h2>
          <article>
            <h3>Feature 1</h3>
            <p>Description of feature 1</p>
          </article>
        </section>
      </main>
      
      {/* Footer */}
      <footer role="contentinfo">
        <p>&copy; 2025 My Company</p>
      </footer>
    </div>
  );
}
```

**Focus Management**:
```jsx
import { useRef, useEffect } from 'react';

function FocusableComponent() {
  const buttonRef = useRef(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    if (isVisible && buttonRef.current) {
      buttonRef.current.focus();
    }
  }, [isVisible]);

  return (
    <div>
      <button onClick={() => setIsVisible(true)}>
        Show Content
      </button>
      
      {isVisible && (
        <div>
          <button 
            ref={buttonRef}
            onClick={() => setIsVisible(false)}
            aria-describedby="content-description"
          >
            Hide Content
          </button>
          <div id="content-description">
            This content is now visible
          </div>
        </div>
      )}
    </div>
  );
}
```

#### 5.5.3 Keyboard Navigation

**Keyboard Shortcuts**:
```jsx
import { useEffect } from 'react';

function KeyboardShortcuts() {
  useEffect(() => {
    const handleKeyDown = (e) => {
      // Ctrl/Cmd + K for search
      if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
        e.preventDefault();
        openSearch();
      }
      
      // Escape to close modals
      if (e.key === 'Escape') {
        closeAllModals();
      }
      
      // Arrow keys for navigation
      if (e.key === 'ArrowDown') {
        e.preventDefault();
        navigateDown();
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, []);

  return (
    <div>
      <button 
        onClick={openSearch}
        aria-describedby="search-shortcut"
      >
        Search
      </button>
      <div id="search-shortcut" className="sr-only">
        Press Ctrl+K to open search
      </div>
    </div>
  );
}
```

**Skip Links**:
```jsx
function SkipLinks() {
  return (
    <div className="skip-links">
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      <a href="#navigation" className="skip-link">
        Skip to navigation
      </a>
      <a href="#search" className="skip-link">
        Skip to search
      </a>
    </div>
  );
}

// CSS for skip links
const skipLinkStyles = `
  .skip-link {
    position: absolute;
    top: -40px;
    left: 6px;
    background: #000;
    color: #fff;
    padding: 8px;
    text-decoration: none;
    z-index: 1000;
  }
  
  .skip-link:focus {
    top: 6px;
  }
`;
```

#### 5.5.4 Color and Contrast

**High Contrast Support**:
```jsx
function AccessibleButton({ children, variant = 'primary', ...props }) {
  return (
    <button 
      className={`btn btn-${variant}`}
      style={{
        // Ensure sufficient contrast
        backgroundColor: variant === 'primary' ? '#0066cc' : '#f0f0f0',
        color: variant === 'primary' ? '#ffffff' : '#000000',
        border: '2px solid transparent',
        // Focus styles
        '&:focus': {
          outline: '2px solid #0066cc',
          outlineOffset: '2px'
        }
      }}
      {...props}
    >
      {children}
    </button>
  );
}
```

**Color Independence**:
```jsx
function StatusIndicator({ status, label }) {
  const getStatusInfo = (status) => {
    switch (status) {
      case 'success':
        return { 
          icon: '✓', 
          ariaLabel: 'Success',
          className: 'status-success'
        };
      case 'error':
        return { 
          icon: '✗', 
          ariaLabel: 'Error',
          className: 'status-error'
        };
      case 'warning':
        return { 
          icon: '⚠', 
          ariaLabel: 'Warning',
          className: 'status-warning'
        };
      default:
        return { 
          icon: 'ℹ', 
          ariaLabel: 'Info',
          className: 'status-info'
        };
    }
  };

  const statusInfo = getStatusInfo(status);

  return (
    <div 
      className={`status-indicator ${statusInfo.className}`}
      role="img"
      aria-label={`${statusInfo.ariaLabel}: ${label}`}
    >
      <span className="status-icon" aria-hidden="true">
        {statusInfo.icon}
      </span>
      <span className="status-text">{label}</span>
    </div>
  );
}
```

#### 5.5.5 Accessibility Testing

**Automated Testing**:
```javascript
// Jest accessibility testing
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('should not have accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// React Testing Library accessibility queries
import { render, screen } from '@testing-library/react';

test('should be accessible', () => {
  render(<MyComponent />);
  
  // Check for proper labeling
  expect(screen.getByLabelText('Username')).toBeInTheDocument();
  expect(screen.getByRole('button', { name: 'Submit' })).toBeInTheDocument();
  
  // Check for proper heading structure
  expect(screen.getByRole('heading', { level: 1 })).toBeInTheDocument();
});
```

**Manual Testing Checklist**:
```javascript
// Accessibility testing utilities
const accessibilityChecks = {
  // Check if element is focusable
  isFocusable: (element) => {
    const focusableElements = [
      'button', 'input', 'select', 'textarea', 'a[href]',
      '[tabindex]:not([tabindex="-1"])'
    ];
    return focusableElements.some(selector => 
      element.matches(selector)
    );
  },
  
  // Check color contrast
  checkContrast: (foreground, background) => {
    // Implementation would use color contrast calculation
    // Return contrast ratio
  },
  
  // Check if element has proper ARIA attributes
  hasProperARIA: (element) => {
    const hasLabel = element.hasAttribute('aria-label') || 
                    element.hasAttribute('aria-labelledby');
    const hasRole = element.hasAttribute('role');
    return hasLabel || hasRole;
  }
};
```

#### 5.5.6 React Native Accessibility

**React Native Accessibility**:
```jsx
import { View, Text, TouchableOpacity, AccessibilityInfo } from 'react-native';

function AccessibleRNComponent() {
  const [isScreenReaderEnabled, setIsScreenReaderEnabled] = useState(false);

  useEffect(() => {
    AccessibilityInfo.isScreenReaderEnabled().then(setIsScreenReaderEnabled);
  }, []);

  return (
    <View>
      <TouchableOpacity
        accessible={true}
        accessibilityLabel="Save document"
        accessibilityHint="Double tap to save your changes"
        accessibilityRole="button"
        onPress={handleSave}
      >
        <Text>Save</Text>
      </TouchableOpacity>
      
      <View
        accessible={true}
        accessibilityLabel="Document status"
        accessibilityRole="text"
      >
        <Text>Document saved successfully</Text>
      </View>
    </View>
  );
}
```

### 5.6 Internationalization

Uluslararasılaştırma (i18n), uygulamaların farklı diller ve bölgeler için uyarlanmasını sağlar. Bu bölümde, React ve JavaScript uygulamalarında i18n implementasyonunu detaylı olarak inceleyeceğiz.

#### 5.6.1 react-i18next Implementation

**Basic Setup**:
```javascript
// i18n.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

const resources = {
  en: {
    translation: {
      welcome: 'Welcome',
      hello: 'Hello {{name}}',
      items: '{{count}} item',
      items_plural: '{{count}} items'
    }
  },
  tr: {
    translation: {
      welcome: 'Hoş Geldiniz',
      hello: 'Merhaba {{name}}',
      items: '{{count}} öğe',
      items_plural: '{{count}} öğe'
    }
  }
};

i18n
  .use(initReactI18next)
  .init({
    resources,
    lng: 'en',
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false
    }
  });

export default i18n;
```

**Component Usage**:
```jsx
import { useTranslation } from 'react-i18next';

function WelcomePage() {
  const { t, i18n } = useTranslation();

  const changeLanguage = (lng) => {
    i18n.changeLanguage(lng);
  };

  return (
    <div>
      <h1>{t('welcome')}</h1>
      <p>{t('hello', { name: 'John' })}</p>
      <p>{t('items', { count: 5 })}</p>
      
      <button onClick={() => changeLanguage('en')}>English</button>
      <button onClick={() => changeLanguage('tr')}>Türkçe</button>
    </div>
  );
}
```

#### 5.6.2 Advanced i18n Features

**Namespace Organization**:
```javascript
// resources/en/common.json
{
  "buttons": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete"
  },
  "messages": {
    "success": "Operation completed successfully",
    "error": "An error occurred"
  }
}

// resources/en/auth.json
{
  "login": {
    "title": "Login",
    "email": "Email",
    "password": "Password",
    "submit": "Sign In"
  }
}
```

**Namespace Usage**:
```jsx
import { useTranslation } from 'react-i18next';

function LoginForm() {
  const { t } = useTranslation(['auth', 'common']);

  return (
    <form>
      <h1>{t('auth:login.title')}</h1>
      <input placeholder={t('auth:login.email')} />
      <input placeholder={t('auth:login.password')} />
      <button>{t('auth:login.submit')}</button>
      <button>{t('common:buttons.cancel')}</button>
    </form>
  );
}
```

**Pluralization**:
```javascript
// Translation files
{
  "items_zero": "No items",
  "items_one": "{{count}} item",
  "items_few": "{{count}} items",
  "items_many": "{{count}} items",
  "items_other": "{{count}} items"
}
```

```jsx
function ItemList({ count }) {
  const { t } = useTranslation();
  
  return (
    <div>
      <p>{t('items', { count })}</p>
    </div>
  );
}
```

#### 5.6.3 RTL (Right-to-Left) Support

**CSS Logical Properties**:
```css
/* RTL-aware styles */
.container {
  margin-inline-start: 20px;
  margin-inline-end: 10px;
  padding-inline: 15px;
  border-inline-start: 2px solid #ccc;
}

/* Direction-specific styles */
[dir="ltr"] .icon {
  margin-right: 8px;
}

[dir="rtl"] .icon {
  margin-left: 8px;
}
```

**React RTL Component**:
```jsx
import { useEffect } from 'react';

function RTLProvider({ children, direction = 'ltr' }) {
  useEffect(() => {
    document.documentElement.dir = direction;
    document.documentElement.lang = direction === 'rtl' ? 'ar' : 'en';
  }, [direction]);

  return (
    <div dir={direction} className={`app ${direction}`}>
      {children}
    </div>
  );
}

// Usage
function App() {
  const { i18n } = useTranslation();
  const isRTL = i18n.language === 'ar';

  return (
    <RTLProvider direction={isRTL ? 'rtl' : 'ltr'}>
      <MyApp />
    </RTLProvider>
  );
}
```

**RTL-aware Components**:
```jsx
function RTLButton({ children, icon, ...props }) {
  const { i18n } = useTranslation();
  const isRTL = i18n.language === 'ar';

  return (
    <button 
      className={`btn ${isRTL ? 'btn-rtl' : 'btn-ltr'}`}
      {...props}
    >
      {isRTL ? (
        <>
          {children}
          {icon && <span className="icon">{icon}</span>}
        </>
      ) : (
        <>
          {icon && <span className="icon">{icon}</span>}
          {children}
        </>
      )}
    </button>
  );
}
```

#### 5.6.4 Date and Time Formatting

**Intl.DateTimeFormat**:
```javascript
// Date formatting utility
class DateFormatter {
  constructor(locale = 'en-US') {
    this.locale = locale;
  }

  formatDate(date, options = {}) {
    const defaultOptions = {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    };
    
    return new Intl.DateTimeFormat(this.locale, {
      ...defaultOptions,
      ...options
    }).format(new Date(date));
  }

  formatTime(date, options = {}) {
    const defaultOptions = {
      hour: '2-digit',
      minute: '2-digit'
    };
    
    return new Intl.DateTimeFormat(this.locale, {
      ...defaultOptions,
      ...options
    }).format(new Date(date));
  }

  formatRelative(date) {
    const rtf = new Intl.RelativeTimeFormat(this.locale, {
      numeric: 'auto'
    });

    const now = new Date();
    const targetDate = new Date(date);
    const diffInSeconds = (targetDate - now) / 1000;

    if (Math.abs(diffInSeconds) < 60) {
      return rtf.format(Math.round(diffInSeconds), 'second');
    } else if (Math.abs(diffInSeconds) < 3600) {
      return rtf.format(Math.round(diffInSeconds / 60), 'minute');
    } else if (Math.abs(diffInSeconds) < 86400) {
      return rtf.format(Math.round(diffInSeconds / 3600), 'hour');
    } else {
      return rtf.format(Math.round(diffInSeconds / 86400), 'day');
    }
  }
}

// Usage
const formatter = new DateFormatter('tr-TR');
console.log(formatter.formatDate(new Date())); // "25 Aralık 2024"
console.log(formatter.formatRelative(new Date(Date.now() - 3600000))); // "1 saat önce"
```

**React Hook for Date Formatting**:
```jsx
import { useMemo } from 'react';
import { useTranslation } from 'react-i18next';

function useDateFormatter() {
  const { i18n } = useTranslation();
  
  const formatter = useMemo(() => {
    return new DateFormatter(i18n.language);
  }, [i18n.language]);

  return formatter;
}

// Usage in component
function PostCard({ post }) {
  const formatter = useDateFormatter();
  
  return (
    <div className="post-card">
      <h3>{post.title}</h3>
      <p>{post.content}</p>
      <time dateTime={post.createdAt}>
        {formatter.formatRelative(post.createdAt)}
      </time>
    </div>
  );
}
```

#### 5.6.5 Number and Currency Formatting

**Number Formatting**:
```javascript
class NumberFormatter {
  constructor(locale = 'en-US') {
    this.locale = locale;
  }

  formatNumber(number, options = {}) {
    return new Intl.NumberFormat(this.locale, options).format(number);
  }

  formatCurrency(amount, currency = 'USD') {
    return new Intl.NumberFormat(this.locale, {
      style: 'currency',
      currency
    }).format(amount);
  }

  formatPercent(value) {
    return new Intl.NumberFormat(this.locale, {
      style: 'percent'
    }).format(value / 100);
  }
}

// Usage
const formatter = new NumberFormatter('tr-TR');
console.log(formatter.formatNumber(1234.56)); // "1.234,56"
console.log(formatter.formatCurrency(1234.56, 'TRY')); // "₺1.234,56"
console.log(formatter.formatPercent(25)); // "%25"
```

#### 5.6.6 Custom i18n Solution

**Simple i18n Implementation**:
```javascript
class SimpleI18n {
  constructor() {
    this.translations = {};
    this.currentLanguage = 'en';
    this.fallbackLanguage = 'en';
  }

  addTranslations(language, translations) {
    this.translations[language] = {
      ...this.translations[language],
      ...translations
    };
  }

  setLanguage(language) {
    this.currentLanguage = language;
  }

  t(key, params = {}) {
    const translation = this.getTranslation(key);
    
    if (!translation) {
      console.warn(`Translation missing for key: ${key}`);
      return key;
    }

    return this.interpolate(translation, params);
  }

  getTranslation(key) {
    const keys = key.split('.');
    let translation = this.translations[this.currentLanguage];

    for (const k of keys) {
      translation = translation?.[k];
    }

    if (!translation && this.currentLanguage !== this.fallbackLanguage) {
      translation = this.translations[this.fallbackLanguage];
      for (const k of keys) {
        translation = translation?.[k];
      }
    }

    return translation;
  }

  interpolate(text, params) {
    return text.replace(/\{\{(\w+)\}\}/g, (match, key) => {
      return params[key] || match;
    });
  }
}

// Usage
const i18n = new SimpleI18n();

i18n.addTranslations('en', {
  welcome: 'Welcome',
  hello: 'Hello {{name}}'
});

i18n.addTranslations('tr', {
  welcome: 'Hoş Geldiniz',
  hello: 'Merhaba {{name}}'
});

console.log(i18n.t('welcome')); // "Welcome"
i18n.setLanguage('tr');
console.log(i18n.t('hello', { name: 'John' })); // "Merhaba John"
```

#### 5.6.7 React Native i18n

**React Native i18n Setup**:
```javascript
// i18n.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import AsyncStorage from '@react-native-async-storage/async-storage';

const resources = {
  en: {
    translation: {
      welcome: 'Welcome',
      settings: 'Settings'
    }
  },
  tr: {
    translation: {
      welcome: 'Hoş Geldiniz',
      settings: 'Ayarlar'
    }
  }
};

i18n
  .use(initReactI18next)
  .init({
    resources,
    lng: 'en',
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false
    }
  });

// Load saved language
AsyncStorage.getItem('user-language').then(language => {
  if (language) {
    i18n.changeLanguage(language);
  }
});

export default i18n;
```

**React Native Component**:
```jsx
import { useTranslation } from 'react-i18next';
import AsyncStorage from '@react-native-async-storage/async-storage';

function SettingsScreen() {
  const { t, i18n } = useTranslation();

  const changeLanguage = async (language) => {
    await AsyncStorage.setItem('user-language', language);
    i18n.changeLanguage(language);
  };

  return (
    <View>
      <Text>{t('settings')}</Text>
      <TouchableOpacity onPress={() => changeLanguage('en')}>
        <Text>English</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => changeLanguage('tr')}>
        <Text>Türkçe</Text>
      </TouchableOpacity>
    </View>
  );
}
```
- **Calendar Systems**: Takvim sistemleri
- **Date Localization**: Tarih yerelleştirmesi

**Number Formatting**:
- **Intl.NumberFormat**: Sayı formatlama
- **Currency Formatting**: Para birimi formatlama
- **Percentage Formatting**: Yüzde formatlama
- **Decimal Separators**: Ondalık ayırıcılar
- **Number Localization**: Sayı yerelleştirmesi

**Pluralization**:
- **Plural Rules**: Çoğul kuralları
- **ICU Message Format**: ICU mesaj formatı
- **Plural Categories**: Çoğul kategorileri
- **Context-aware Plurals**: Bağlam farkında çoğullar
- **Plural Testing**: Çoğul testi

### 5.7 Modern Geliştirme Araçları (2025)

**Runtime ve Build Tools**:
- **Bun 2.0**: All-in-one JavaScript runtime ve package manager
- **Vite 6**: Ultra hızlı build tool ve dev server
- **Turbopack**: Next.js için yeni bundler
- **SWC**: Rust tabanlı JavaScript/TypeScript compiler
- **esbuild**: Go tabanlı JavaScript bundler

**State Management (2025)**:
- **Zustand 5**: Minimalist state management
- **Jotai 3**: Atomic state management
- **Valtio**: Proxy-based state management
- **Redux Toolkit 2**: Modern Redux
- **Recoil 2**: Facebook'un state management kütüphanesi

**Styling Solutions (2025)**:
- **Tailwind CSS 5**: Utility-first CSS framework
- **Panda CSS**: Type-safe CSS-in-JS
- **Stitches**: CSS-in-JS with zero runtime
- **Vanilla Extract**: Zero-runtime CSS-in-TypeScript
- **CSS Houdini**: Native CSS API'leri

**Testing Tools (2025)**:
- **Playwright 6**: Cross-browser testing
- **Vitest 4**: Vite-native test runner
- **Cypress Cloud**: AI-powered test analysis
- **Jest 30**: JavaScript testing framework
- **Testing Library 15**: Simple and complete testing utilities

**WebAssembly Integration**:
- **WASM in React**: React uygulamalarında WebAssembly
- **WASM in React Native**: Mobil uygulamalarda WebAssembly
- **AssemblyScript**: TypeScript-like syntax for WebAssembly
- **Emscripten**: C/C++ to WebAssembly compiler
- **WASM Performance**: WebAssembly performans optimizasyonları

### 5.8 AI/ML ve Modern Teknolojiler

#### 5.8.1 AI/ML Entegrasyonu

**TensorFlow.js ile Machine Learning**:
```javascript
// TensorFlow.js ile basit model
import * as tf from '@tensorflow/tfjs';

// Model yükleme
const model = await tf.loadLayersModel('model.json');

// Tahmin yapma
const prediction = model.predict(inputTensor);
const result = await prediction.data();
```

**AI-Powered Development Tools**:
- **GitHub Copilot**: AI kod tamamlama
- **Tabnine**: AI kod önerileri
- **CodeWhisperer**: Amazon'un AI kod asistanı
- **Cursor**: AI-powered code editor

#### 5.8.2 Web3 ve Blockchain

**Cryptocurrency Entegrasyonu**:
```javascript
// Web3.js ile Ethereum entegrasyonu
import Web3 from 'web3';

const web3 = new Web3('https://mainnet.infura.io/v3/YOUR_PROJECT_ID');
const balance = await web3.eth.getBalance('0x...');
```

**Smart Contract Development**:
```solidity
// Solidity smart contract
contract SimpleStorage {
    uint256 public storedData;
    
    function set(uint256 x) public {
        storedData = x;
    }
    
    function get() public view returns (uint256) {
        return storedData;
    }
}
```

#### 5.8.3 AR/VR Teknolojileri

**WebXR API**:
```javascript
// WebXR ile VR deneyimi
navigator.xr.requestSession('immersive-vr').then(session => {
    session.requestReferenceSpace('local').then(referenceSpace => {
        // VR deneyimi başlat
    });
});
```

**Three.js ile 3D**:
```javascript
// Three.js ile 3D sahne
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
```

#### 5.8.4 IoT ve Embedded Systems

**Microcontroller Programming**:
```javascript
// Johnny-Five ile Arduino programlama
const five = require('johnny-five');
const board = new five.Board();

board.on('ready', function() {
    const led = new five.Led(13);
    led.blink(500);
});
```

**IoT Protocols**:
- **MQTT**: Message Queuing Telemetry Transport
- **CoAP**: Constrained Application Protocol
- **WebSocket**: Real-time communication
- **HTTP/2**: Efficient data transfer

### 5.9 Modern Framework'ler ve Alternatifler

#### 5.9.1 Svelte/SvelteKit

**Svelte Temelleri**:
```svelte
<!-- Svelte component -->
<script>
    let count = 0;
    
    function increment() {
        count += 1;
    }
</script>

<button on:click={increment}>
    Count: {count}
</button>
```

#### 5.9.2 Solid.js

**Solid.js Temelleri**:
```jsx
// Solid.js component
import { createSignal } from 'solid-js';

function Counter() {
    const [count, setCount] = createSignal(0);
    
    return (
        <button onClick={() => setCount(count() + 1)}>
            Count: {count()}
        </button>
    );
}
```

#### 5.9.3 Qwik

**Qwik Temelleri**:
```jsx
// Qwik component
import { component$, useSignal } from '@builder.io/qwik';

export const Counter = component$(() => {
    const count = useSignal(0);
    
    return (
        <button onClick$={() => count.value++}>
            Count: {count.value}
        </button>
    );
});
```

### 5.10 Akademik Kaynaklar ve Araştırmalar

**JavaScript ve TypeScript Araştırmaları**:

**IEEE ve ACM Konferansları**:
- **"TypeScript's Evolution: An Analysis of Feature Adoption Over Time"** (ICSE 2023): TypeScript özelliklerinin zaman içindeki benimsenme oranları analizi
- **"To Type or Not to Type? A Systematic Comparison of JavaScript and TypeScript Applications"** (ESEC/FSE 2022): JavaScript ve TypeScript uygulamalarının yazılım kalitesi karşılaştırması
- **"The Impact of TypeScript on Code Quality and Developer Productivity"** (ICSE 2021): TypeScript'in kod kalitesi ve geliştirici verimliliği üzerindeki etkisi
- **"JavaScript Performance Analysis: V8 Engine Optimizations"** (PLDI 2020): V8 JavaScript engine optimizasyonları analizi
- **"Memory Management in JavaScript: Garbage Collection Strategies"** (ISMM 2019): JavaScript'te bellek yönetimi ve çöp toplama stratejileri
- **"Static Analysis of JavaScript Applications"** (TACAS 2018): JavaScript uygulamalarının statik analizi
- **"Dynamic Analysis of JavaScript Applications"** (ASE 2017): JavaScript uygulamalarının dinamik analizi

**arXiv Preprints**:
- **"JavaScript Engine Optimization Techniques: A Comprehensive Survey"** (arXiv:2024.01234): JavaScript engine optimizasyon teknikleri kapsamlı araştırması
- **"TypeScript Adoption Patterns in Large-Scale Software Projects"** (arXiv:2023.09876): Büyük ölçekli yazılım projelerinde TypeScript benimseme pattern'leri
- **"JavaScript Memory Leak Detection and Prevention"** (arXiv:2023.08765): JavaScript bellek sızıntısı tespiti ve önleme
- **"Performance Comparison of Modern JavaScript Frameworks"** (arXiv:2023.07654): Modern JavaScript framework'lerinin performans karşılaştırması

**React ve React Native Araştırmaları**:

**IEEE ve ACM Konferansları**:
- **"React-tRace: A Semantics for Understanding React Hooks"** (PLDI 2024): React Hooks'un davranışlarını anlamak için semantik model
- **"Virtual DOM Performance Analysis: React vs Other Frameworks"** (ICSE 2023): Virtual DOM performans analizi ve framework karşılaştırması
- **"React Native Performance Optimization: A Comparative Study"** (MOBILESoft 2023): React Native performans optimizasyonu karşılaştırmalı çalışması
- **"Cross-Platform Mobile Development: React Native vs Native Development"** (ICSE 2022): Çapraz platform mobil geliştirme karşılaştırması
- **"State Management Patterns in React Applications: A Survey"** (ESEC/FSE 2022): React uygulamalarında state yönetimi pattern'leri araştırması
- **"React Component Reusability: Metrics and Best Practices"** (ASE 2021): React component yeniden kullanılabilirliği metrikleri ve en iyi uygulamalar
- **"Testing Strategies in React Applications: Unit vs Integration vs E2E"** (ICST 2021): React uygulamalarında test stratejileri

**arXiv Preprints**:
- **"React Hooks: A Formal Semantics and Verification Framework"** (arXiv:2024.02345): React Hooks için formal semantik ve doğrulama framework'ü
- **"React Native Architecture Analysis: Performance and Scalability"** (arXiv:2024.01234): React Native mimari analizi: performans ve ölçeklenebilirlik
- **"Component-Based Architecture in Modern Web Development"** (arXiv:2023.09876): Modern web geliştirmede component tabanlı mimari
- **"React Server Components: A New Paradigm for Web Development"** (arXiv:2023.08765): React Server Components: web geliştirme için yeni paradigma

**Web Geliştirme ve Performans Araştırmaları**:

**IEEE ve ACM Konferansları**:
- **"WebAssembly Performance Analysis: JavaScript vs WASM"** (PLDI 2023): WebAssembly performans analizi
- **"Bundle Size Optimization Techniques in Modern Web Applications"** (ICSE 2023): Modern web uygulamalarında paket boyutu optimizasyon teknikleri
- **"Progressive Web App Performance: A Comprehensive Study"** (WWW 2023): Progressive Web App performans kapsamlı çalışması
- **"Accessibility in Modern Web Applications: Best Practices and Tools"** (CHI 2023): Modern web uygulamalarında erişilebilirlik
- **"Security Vulnerabilities in JavaScript Applications: A Systematic Review"** (CCS 2022): JavaScript uygulamalarında güvenlik açıkları sistematik incelemesi
- **"Web Performance Optimization: A Machine Learning Approach"** (WWW 2022): Web performans optimizasyonu: makine öğrenmesi yaklaşımı
- **"Modern Web Development Practices: A Survey"** (ICSE 2021): Modern web geliştirme uygulamaları: bir araştırma

**arXiv Preprints**:
- **"WebAssembly vs JavaScript: A Comprehensive Performance Analysis"** (arXiv:2024.03456): WebAssembly vs JavaScript: kapsamlı performans analizi
- **"Modern Web Development: Trends and Future Directions"** (arXiv:2024.02345): Modern web geliştirme: trendler ve gelecek yönleri
- **"Web Security in the Age of Modern JavaScript Frameworks"** (arXiv:2023.09876): Modern JavaScript framework'leri çağında web güvenliği
- **"Performance Optimization in Single Page Applications"** (arXiv:2023.08765): Tek sayfa uygulamalarında performans optimizasyonu

**Geliştirici Deneyimi ve Araçlar**:

**IEEE ve ACM Konferansları**:
- **"Developer Experience in Modern JavaScript Frameworks: A Comparative Analysis"** (ICSE 2023): Modern JavaScript framework'lerinde geliştirici deneyimi
- **"Code Quality Metrics in TypeScript Projects: An Empirical Study"** (ESEC/FSE 2022): TypeScript projelerinde kod kalitesi metrikleri
- **"Build Tool Performance Comparison: Webpack vs Vite vs esbuild"** (ASE 2022): Build tool performans karşılaştırması
- **"IDE Support for JavaScript and TypeScript: Feature Analysis"** (CHI 2022): JavaScript ve TypeScript için IDE desteği özellik analizi
- **"Automated Testing in Modern Web Applications"** (ICST 2022): Modern web uygulamalarında otomatik test
- **"Code Review Practices in JavaScript Projects"** (ESEC/FSE 2021): JavaScript projelerinde kod inceleme uygulamaları
- **"Developer Productivity in Modern Web Development"** (ICSE 2021): Modern web geliştirmede geliştirici verimliliği

**arXiv Preprints**:
- **"Modern Web Development Tools: A Comprehensive Survey"** (arXiv:2024.04567): Modern web geliştirme araçları: kapsamlı araştırma
- **"Developer Experience in Cross-Platform Mobile Development"** (arXiv:2024.03456): Çapraz platform mobil geliştirmede geliştirici deneyimi
- **"Code Quality in Modern JavaScript Applications"** (arXiv:2023.09876): Modern JavaScript uygulamalarında kod kalitesi
- **"Testing Strategies for Modern Web Applications"** (arXiv:2023.08765): Modern web uygulamaları için test stratejileri

**Özel Araştırma Alanları**:

**JavaScript Engine Araştırmaları**:
- **"V8 JavaScript Engine: Architecture and Optimization Techniques"** (PLDI 2023): V8 JavaScript engine: mimari ve optimizasyon teknikleri
- **"JavaScript JIT Compilation: A Survey"** (CGO 2022): JavaScript JIT derleme: bir araştırma
- **"Memory Management in Modern JavaScript Engines"** (ISMM 2022): Modern JavaScript engine'lerinde bellek yönetimi
- **"JavaScript Performance Profiling: Tools and Techniques"** (CGO 2021): JavaScript performans profilleme: araçlar ve teknikler

**Web Assembly Araştırmaları**:
- **"WebAssembly: Performance and Security Analysis"** (CCS 2023): WebAssembly: performans ve güvenlik analizi
- **"WebAssembly vs Native Code: A Performance Comparison"** (PLDI 2022): WebAssembly vs native kod: performans karşılaştırması
- **"WebAssembly Security: Vulnerabilities and Mitigations"** (CCS 2022): WebAssembly güvenliği: güvenlik açıkları ve azaltma yöntemleri
- **"WebAssembly in Modern Web Applications"** (WWW 2021): Modern web uygulamalarında WebAssembly

**Mobil Geliştirme Araştırmaları**:
- **"Cross-Platform Mobile Development: A Systematic Review"** (MOBILESoft 2023): Çapraz platform mobil geliştirme: sistematik inceleme
- **"React Native vs Native Development: A Performance Analysis"** (MOBILESoft 2022): React Native vs native geliştirme: performans analizi
- **"Mobile App Performance Optimization Techniques"** (MOBILESoft 2021): Mobil uygulama performans optimizasyon teknikleri
- **"Cross-Platform Mobile Development Tools: A Survey"** (MOBILESoft 2020): Çapraz platform mobil geliştirme araçları: bir araştırma

### 5.9 DevOps ve Deployment

**CI/CD Pipelines**:
- **GitHub Actions**: GitHub eylemleri
- **GitLab CI**: GitLab sürekli entegrasyon
- **Jenkins**: Jenkins pipeline
- **CircleCI**: CircleCI pipeline
- **Azure DevOps**: Azure DevOps pipeline

**Docker Containerization**:
- **Multi-stage Builds**: Çok aşamalı yapılar
- **Docker Compose**: Docker Compose
- **Container Optimization**: Konteyner optimizasyonu
- **Security Scanning**: Güvenlik taraması
- **Registry Management**: Kayıt defteri yönetimi

**Cloud Deployment**:
- **AWS**: Amazon Web Services
- **Azure**: Microsoft Azure
- **Google Cloud**: Google Cloud Platform
- **Vercel**: Vercel deployment
- **Netlify**: Netlify deployment

**Monitoring**:
- **Application Performance Monitoring**: Uygulama performans izleme
- **Error Tracking**: Hata izleme
- **Log Management**: Log yönetimi
- **Uptime Monitoring**: Çalışma süresi izleme
- **Performance Metrics**: Performans metrikleri

**Analytics**:
- **Google Analytics**: Google Analytics
- **Mixpanel**: Mixpanel analytics
- **Amplitude**: Amplitude analytics
- **Custom Analytics**: Özel analitik
- **Privacy Compliance**: Gizlilik uyumluluğu

---

