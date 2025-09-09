# React Test Stratejileri ve Araçları

## İçindekiler
1. [Test Nedir ve Neden Önemlidir?](#test-nedir-ve-neden-önemlidir)
2. [React Test Türleri](#react-test-türleri)
3. [Test Araçları ve Kütüphaneleri](#test-araçları-ve-kütüphaneleri)
4. [Unit Testing](#unit-testing)
5. [Integration Testing](#integration-testing)
6. [End-to-End Testing](#end-to-end-testing)
7. [Test Best Practices](#test-best-practices)
8. [Test Coverage ve Metrikleri](#test-coverage-ve-metrikleri)
9. [Mock ve Stub Kullanımı](#mock-ve-stub-kullanımı)
10. [Test Driven Development (TDD)](#test-driven-development-tdd)

---

## Test Nedir ve Neden Önemlidir?

### Test Tanımı
Test, yazılım geliştirme sürecinde kodun doğru çalışıp çalışmadığını, beklenen davranışları sergileyip sergilemediğini kontrol etme sürecidir. React uygulamalarında test, bileşenlerin, hook'ların ve uygulama mantığının doğru şekilde çalıştığını doğrulamak için kritik öneme sahiptir.

### Testin Faydaları
- **Hata Tespiti**: Kod değişikliklerinden kaynaklanan hataları erken tespit eder
- **Güven**: Yeni özellikler eklerken mevcut işlevselliğin bozulmadığından emin olur
- **Dokümantasyon**: Testler, kodun nasıl çalışması gerektiğini gösteren canlı dokümantasyon görevi görür
- **Refactoring Güvenliği**: Kodu yeniden düzenlerken güvenli bir ortam sağlar
- **Kalite Artışı**: Test yazma süreci, daha temiz ve modüler kod yazmaya teşvik eder

### Test Piramidi
```
        /\
       /  \
      / E2E \     (Az sayıda, yavaş, pahalı)
     /______\
    /        \
   /Integration\  (Orta sayıda, orta hız, orta maliyet)
  /____________\
 /              \
/    Unit Tests  \  (Çok sayıda, hızlı, ucuz)
/________________\
```

---

## React Test Türleri

### 1. Unit Testing (Birim Testleri)
**Tanım**: En küçük test edilebilir kod parçalarını (fonksiyonlar, hook'lar, utility fonksiyonları) test eder.

**Ne Zaman Kullanılır**:
- Utility fonksiyonları test ederken
- Custom hook'ları test ederken
- Pure fonksiyonları test ederken
- Business logic'i test ederken

**Özellikler**:
- Hızlı çalışır
- Bağımlılıkları yoktur
- İzole edilmiş testler
- Kolay debug edilir

### 2. Integration Testing (Entegrasyon Testleri)
**Tanım**: Birden fazla bileşenin birlikte çalışmasını test eder.

**Ne Zaman Kullanılır**:
- Bileşenler arası etkileşimleri test ederken
- API çağrıları ile bileşen etkileşimlerini test ederken
- Form işlemlerini test ederken
- State management ile bileşen etkileşimlerini test ederken

**Özellikler**:
- Gerçek kullanım senaryolarını test eder
- Bileşenler arası bağımlılıkları kontrol eder
- Daha gerçekçi test ortamı

### 3. End-to-End Testing (E2E Testleri)
**Tanım**: Uygulamanın baştan sona kullanıcı perspektifinden test edilmesi.

**Ne Zaman Kullanılır**:
- Kritik kullanıcı akışlarını test ederken
- Tam uygulama işlevselliğini doğrularken
- Production benzeri ortamda test yaparken

**Özellikler**:
- En gerçekçi test türü
- Yavaş ve pahalı
- Az sayıda yazılır
- Kritik path'leri kapsar

---

## Test Araçları ve Kütüphaneleri

### 1. Jest
**Tanım**: Facebook tarafından geliştirilen JavaScript test framework'ü.

**Özellikler**:
- Zero configuration
- Built-in mocking
- Code coverage
- Snapshot testing
- Parallel test execution

**Kurulum**:
```bash
npm install --save-dev jest @types/jest
```

**Temel Kullanım**:
```javascript
// math.test.js
function add(a, b) {
  return a + b;
}

test('adds 1 + 2 to equal 3', () => {
  expect(add(1, 2)).toBe(3);
});
```

### 2. React Testing Library
**Tanım**: React bileşenlerini test etmek için özel olarak tasarlanmış kütüphane.

**Özellikler**:
- User-centric testing
- Accessibility-first approach
- Minimal implementation details
- Real DOM testing

**Kurulum**:
```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

**Temel Kullanım**:
```javascript
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button with text', () => {
  render(<Button>Click me</Button>);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});
```

### 3. Enzyme (Deprecated)
**Tanım**: Airbnb tarafından geliştirilen React test utility'si (artık önerilmiyor).

**Neden Önerilmiyor**:
- Implementation details'e odaklanır
- Accessibility'yi göz ardı eder
- React Testing Library'ye göre daha karmaşık

### 4. Cypress
**Tanım**: End-to-end test framework'ü.

**Özellikler**:
- Real browser testing
- Time travel debugging
- Automatic waiting
- Screenshot ve video recording

**Kurulum**:
```bash
npm install --save-dev cypress
```

### 5. Playwright
**Tanım**: Microsoft tarafından geliştirilen modern E2E test framework'ü.

**Özellikler**:
- Multi-browser support
- Fast execution
- Built-in waiting
- Network interception

---

## Unit Testing

### Utility Fonksiyonları Test Etme

```javascript
// utils/helpers.js
export const formatCurrency = (amount, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency
  }).format(amount);
};

export const calculateDiscount = (price, discountPercent) => {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new Error('Discount must be between 0 and 100');
  }
  return price * (1 - discountPercent / 100);
};
```

```javascript
// utils/helpers.test.js
import { formatCurrency, calculateDiscount } from './helpers';

describe('formatCurrency', () => {
  test('formats USD currency correctly', () => {
    expect(formatCurrency(100)).toBe('$100.00');
    expect(formatCurrency(99.99)).toBe('$99.99');
  });

  test('formats EUR currency correctly', () => {
    expect(formatCurrency(100, 'EUR')).toBe('€100.00');
  });
});

describe('calculateDiscount', () => {
  test('calculates discount correctly', () => {
    expect(calculateDiscount(100, 10)).toBe(90);
    expect(calculateDiscount(50, 20)).toBe(40);
  });

  test('throws error for invalid discount', () => {
    expect(() => calculateDiscount(100, -10)).toThrow('Discount must be between 0 and 100');
    expect(() => calculateDiscount(100, 110)).toThrow('Discount must be between 0 and 100');
  });
});
```

### Custom Hook Test Etme

```javascript
// hooks/useCounter.js
import { useState, useCallback } from 'react';

export const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  return { count, increment, decrement, reset };
};
```

```javascript
// hooks/useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  test('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  test('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  test('should increment count', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  test('should decrement count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  test('should reset count', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

---

## Integration Testing

### Bileşen Entegrasyon Testleri

```javascript
// components/UserProfile.jsx
import React, { useState, useEffect } from 'react';

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error('User not found');
        }
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (userId) {
      fetchUser();
    }
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Role: {user.role}</p>
    </div>
  );
};

export default UserProfile;
```

```javascript
// components/UserProfile.test.jsx
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from './UserProfile';

// Mock fetch
global.fetch = jest.fn();

describe('UserProfile Integration Tests', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  test('displays user information after successful fetch', async () => {
    const mockUser = {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      role: 'admin'
    };

    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser,
    });

    render(<UserProfile userId={1} />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('Email: john@example.com')).toBeInTheDocument();
      expect(screen.getByText('Role: admin')).toBeInTheDocument();
    });

    expect(fetch).toHaveBeenCalledWith('/api/users/1');
  });

  test('displays error message when fetch fails', async () => {
    fetch.mockRejectedValueOnce(new Error('User not found'));

    render(<UserProfile userId={999} />);

    await waitFor(() => {
      expect(screen.getByText('Error: User not found')).toBeInTheDocument();
    });
  });

  test('displays loading state initially', () => {
    fetch.mockImplementation(() => new Promise(() => {})); // Never resolves

    render(<UserProfile userId={1} />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

### Form Entegrasyon Testleri

```javascript
// components/LoginForm.jsx
import React, { useState } from 'react';

const LoginForm = ({ onSubmit }) => {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };

  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }
    
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validateForm();
    
    if (Object.keys(newErrors).length === 0) {
      onSubmit(formData);
    } else {
      setErrors(newErrors);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
};

export default LoginForm;
```

```javascript
// components/LoginForm.test.jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginForm from './LoginForm';

describe('LoginForm Integration Tests', () => {
  test('submits form with valid data', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();

    render(<LoginForm onSubmit={mockOnSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(mockOnSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });

  test('shows validation errors for invalid data', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();

    render(<LoginForm onSubmit={mockOnSubmit} />);

    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText('Email is required')).toBeInTheDocument();
    expect(screen.getByText('Password is required')).toBeInTheDocument();
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  test('shows error for invalid email format', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();

    render(<LoginForm onSubmit={mockOnSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'invalid-email');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText('Email is invalid')).toBeInTheDocument();
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  test('clears error when user starts typing', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();

    render(<LoginForm onSubmit={mockOnSubmit} />);

    // Submit empty form to show errors
    await user.click(screen.getByRole('button', { name: /login/i }));
    expect(screen.getByText('Email is required')).toBeInTheDocument();

    // Start typing in email field
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    
    // Error should be cleared
    expect(screen.queryByText('Email is required')).not.toBeInTheDocument();
  });
});
```

---

## End-to-End Testing

### Cypress ile E2E Test

```javascript
// cypress/e2e/login.cy.js
describe('Login Flow', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('should login successfully with valid credentials', () => {
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="login-button"]').click();
    
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="user-menu"]').should('contain', 'Welcome, Test User');
  });

  it('should show error for invalid credentials', () => {
    cy.get('[data-testid="email-input"]').type('invalid@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="error-message"]').should('contain', 'Invalid credentials');
    cy.url().should('include', '/login');
  });

  it('should validate form fields', () => {
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="email-error"]').should('contain', 'Email is required');
    cy.get('[data-testid="password-error"]').should('contain', 'Password is required');
  });
});
```

### Playwright ile E2E Test

```javascript
// tests/e2e/shopping-cart.spec.js
import { test, expect } from '@playwright/test';

test.describe('Shopping Cart', () => {
  test('should add product to cart and proceed to checkout', async ({ page }) => {
    await page.goto('/products');
    
    // Add first product to cart
    await page.click('[data-testid="product-card"]:first-child [data-testid="add-to-cart"]');
    
    // Verify cart count updates
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');
    
    // Go to cart
    await page.click('[data-testid="cart-icon"]');
    
    // Verify product is in cart
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(1);
    
    // Proceed to checkout
    await page.click('[data-testid="checkout-button"]');
    
    // Verify we're on checkout page
    await expect(page).toHaveURL('/checkout');
    await expect(page.locator('h1')).toHaveText('Checkout');
  });

  test('should update quantity and remove items from cart', async ({ page }) => {
    await page.goto('/cart');
    
    // Add multiple items
    await page.click('[data-testid="quantity-increase"]');
    await expect(page.locator('[data-testid="quantity-display"]')).toHaveText('2');
    
    // Remove item
    await page.click('[data-testid="remove-item"]');
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(0);
    await expect(page.locator('[data-testid="empty-cart-message"]')).toBeVisible();
  });
});
```

---

## Test Best Practices

### 1. Test İsimlendirme
```javascript
// ❌ Kötü
test('test 1', () => {});
test('should work', () => {});

// ✅ İyi
test('should display error message when email is invalid', () => {});
test('should increment counter when increment button is clicked', () => {});
```

### 2. AAA Pattern (Arrange, Act, Assert)
```javascript
test('should calculate total price with tax', () => {
  // Arrange
  const price = 100;
  const taxRate = 0.1;
  
  // Act
  const total = calculateTotalWithTax(price, taxRate);
  
  // Assert
  expect(total).toBe(110);
});
```

### 3. Test Isolation
```javascript
// Her test bağımsız olmalı
describe('UserService', () => {
  beforeEach(() => {
    // Her test öncesi temiz başlangıç
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Her test sonrası temizlik
    cleanup();
  });
});
```

### 4. Meaningful Assertions
```javascript
// ❌ Kötü
expect(result).toBeTruthy();

// ✅ İyi
expect(result).toBe(true);
expect(result).toHaveLength(3);
expect(result).toContain('expected value');
```

### 5. Test Data Management
```javascript
// Test data'yı ayrı dosyalarda tut
// __tests__/fixtures/userData.js
export const mockUser = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com'
};

export const mockUsers = [
  mockUser,
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
];
```

---

## Test Coverage ve Metrikleri

### Coverage Türleri
1. **Line Coverage**: Kod satırlarının yüzde kaçının test edildiği
2. **Branch Coverage**: Koşullu dalların yüzde kaçının test edildiği
3. **Function Coverage**: Fonksiyonların yüzde kaçının test edildiği
4. **Statement Coverage**: İfadelerin yüzde kaçının test edildiği

### Jest ile Coverage
```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "collectCoverageFrom": [
      "src/**/*.{js,jsx}",
      "!src/**/*.test.{js,jsx}",
      "!src/index.js"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

### Coverage Raporu
```bash
npm run test:coverage
```

---

## Mock ve Stub Kullanımı

### Jest Mock'ları
```javascript
// API service mock
const mockApiService = {
  getUser: jest.fn(),
  updateUser: jest.fn(),
  deleteUser: jest.fn()
};

// Mock implementation
mockApiService.getUser.mockResolvedValue({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com'
});

// Mock module
jest.mock('../services/apiService', () => mockApiService);
```

### React Testing Library ile Mock
```javascript
// Mock fetch
global.fetch = jest.fn();

beforeEach(() => {
  fetch.mockClear();
});

test('should fetch and display user data', async () => {
  fetch.mockResolvedValueOnce({
    ok: true,
    json: async () => ({ name: 'John Doe' })
  });

  render(<UserProfile userId={1} />);
  
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});
```

### Mock Functions
```javascript
// Custom hook test with mock
const mockSetState = jest.fn();
jest.mock('react', () => ({
  ...jest.requireActual('react'),
  useState: jest.fn(() => [null, mockSetState])
}));

test('should call setState when data changes', () => {
  const { result } = renderHook(() => useCustomHook());
  
  act(() => {
    result.current.updateData('new data');
  });
  
  expect(mockSetState).toHaveBeenCalledWith('new data');
});
```

---

## Test Driven Development (TDD)

### TDD Döngüsü
1. **Red**: Başarısız test yaz
2. **Green**: Testi geçecek minimum kodu yaz
3. **Refactor**: Kodu iyileştir, testler hala geçmeli

### TDD Örneği
```javascript
// 1. Test yaz (Red)
test('should format phone number correctly', () => {
  expect(formatPhoneNumber('1234567890')).toBe('(123) 456-7890');
});

// 2. Minimum kodu yaz (Green)
function formatPhoneNumber(phone) {
  return `(${phone.slice(0, 3)}) ${phone.slice(3, 6)}-${phone.slice(6)}`;
}

// 3. Refactor
function formatPhoneNumber(phone) {
  const cleaned = phone.replace(/\D/g, '');
  const match = cleaned.match(/^(\d{3})(\d{3})(\d{4})$/);
  return match ? `(${match[1]}) ${match[2]}-${match[3]}` : phone;
}
```

### TDD Avantajları
- Daha iyi kod tasarımı
- Kapsamlı test coverage
- Güvenli refactoring
- Daha az bug
- Canlı dokümantasyon

---

## Sonuç

React test stratejileri, modern web uygulamalarının kalitesini ve güvenilirliğini artırmak için kritik öneme sahiptir. Doğru test araçlarını seçmek, uygun test türlerini kullanmak ve best practice'leri takip etmek, sürdürülebilir ve güvenilir uygulamalar geliştirmenin temelini oluşturur.

### Önemli Noktalar
- Test piramidini takip edin (Unit > Integration > E2E)
- React Testing Library'yi tercih edin
- User-centric testler yazın
- Meaningful test isimleri kullanın
- Coverage metriklerini takip edin
- Mock'ları doğru kullanın
- TDD yaklaşımını benimseyin

### Kaynaklar
- [React Testing Library Docs](https://testing-library.com/docs/react-testing-library/intro/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Cypress Documentation](https://docs.cypress.io/)
- [Playwright Documentation](https://playwright.dev/)
- [Testing Best Practices](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
