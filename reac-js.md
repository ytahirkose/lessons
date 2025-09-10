# React.js ve JavaScript Mülakat Soruları

## JavaScript Orta Seviye Sorular

### 1. Hoisting ve Temporal Dead Zone
**Soru:** Aşağıdaki kodun çıktısı nedir ve neden?

```javascript
console.log(a); // ?
console.log(b); // ?
console.log(c); // ?

var a = 1;
let b = 2;
const c = 3;
```

**Cevap:** 
- `a`: undefined (var hoisted ama değer atanmamış)
- `b`: ReferenceError (let temporal dead zone'da)
- `c`: ReferenceError (const temporal dead zone'da)

### 2. this Binding
**Soru:** Aşağıdaki kodun çıktısı nedir?

```javascript
const obj = {
  name: 'John',
  greet: function() {
    console.log(this.name);
  },
  greetArrow: () => {
    console.log(this.name);
  }
};

obj.greet(); // ?
obj.greetArrow(); // ?

const greet = obj.greet;
greet(); // ?
```

**Cevap:** 
- `obj.greet()`: "John" (this = obj)
- `obj.greetArrow()`: undefined (arrow function'da this = window/global)
- `greet()`: undefined (this = window/global)

### 3. Closures ve Module Pattern
**Soru:** Module pattern'i kullanarak private variable'lar nasıl oluşturulur?

```javascript
const counter = (function() {
  let count = 0; // private variable
  
  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
})();

console.log(counter.getCount()); // 0
console.log(counter.increment()); // 1
console.log(counter.count); // undefined (private)
```

### 4. Array Methods ve Functional Programming
**Soru:** Array'de duplicate'ları nasıl kaldırırsınız? Performans açısından en iyi yöntem hangisidir?

```javascript
// Yöntem 1: Set kullanarak
const unique1 = [...new Set(array)];

// Yöntem 2: filter ve indexOf
const unique2 = array.filter((item, index) => array.indexOf(item) === index);

// Yöntem 3: reduce kullanarak
const unique3 = array.reduce((acc, current) => {
  if (!acc.includes(current)) {
    acc.push(current);
  }
  return acc;
}, []);

// En performanslı: Set (O(n))
```

### 5. Promises ve Async/Await
**Soru:** Promise.all, Promise.allSettled, Promise.race arasındaki farklar nelerdir?

```javascript
const promises = [
  Promise.resolve(1),
  Promise.reject('Error'),
  Promise.resolve(3)
];

// Promise.all - hepsi başarılı olmalı
Promise.all(promises)
  .then(results => console.log(results))
  .catch(error => console.log(error)); // "Error"

// Promise.allSettled - hepsini bekle, başarılı/başarısız fark etmez
Promise.allSettled(promises)
  .then(results => console.log(results));
  // [{status: 'fulfilled', value: 1}, {status: 'rejected', reason: 'Error'}, ...]

// Promise.race - ilk tamamlanan
Promise.race(promises)
  .then(result => console.log(result)) // 1
  .catch(error => console.log(error));
```

### 6. Destructuring ve Spread Operator
**Soru:** Nested object destructuring nasıl yapılır? Default values nasıl verilir?

```javascript
const user = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  },
  hobbies: ['reading', 'coding']
};

// Nested destructuring
const { 
  name, 
  age, 
  address: { city, country },
  hobbies: [firstHobby, secondHobby],
  email = 'no-email@example.com' // default value
} = user;

// Spread operator
const newUser = { ...user, age: 31 };
const newHobbies = [...user.hobbies, 'gaming'];
```

## React Orta Seviye Sorular

### 1. Component Lifecycle ve useEffect
**Soru:** useEffect'in dependency array'i nasıl çalışır? Cleanup function ne zaman çalışır?

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // Component mount olduğunda çalışır
    console.log('Component mounted');
    
    // Cleanup function
    return () => {
      console.log('Component will unmount');
    };
  }, []); // Empty dependency array
  
  useEffect(() => {
    // userId değiştiğinde çalışır
    fetchUser(userId).then(setUser);
  }, [userId]); // userId dependency
  
  useEffect(() => {
    // Her render'da çalışır
    document.title = user ? `${user.name}'s Profile` : 'Profile';
  }); // No dependency array
  
  return <div>{user?.name}</div>;
}
```

### 2. State Management ve useState
**Soru:** useState ile object state nasıl güncellenir? Functional update ne zaman kullanılır?

```javascript
function Counter() {
  const [state, setState] = useState({ count: 0, name: 'Counter' });
  
  // Yanlış - state mutation
  const incrementWrong = () => {
    state.count++; // Bu çalışmaz!
    setState(state);
  };
  
  // Doğru - spread operator
  const incrementCorrect = () => {
    setState({ ...state, count: state.count + 1 });
  };
  
  // Functional update - önceki state'e bağlı olduğunda
  const incrementFunctional = () => {
    setState(prevState => ({
      ...prevState,
      count: prevState.count + 1
    }));
  };
  
  return (
    <div>
      <p>{state.name}: {state.count}</p>
      <button onClick={incrementCorrect}>Increment</button>
    </div>
  );
}
```

### 3. Props ve PropTypes
**Soru:** PropTypes ile type checking nasıl yapılır? Default props nasıl tanımlanır?

```javascript
import PropTypes from 'prop-types';

function UserCard({ name, age, email, isActive, hobbies }) {
  return (
    <div className={`user-card ${isActive ? 'active' : 'inactive'}`}>
      <h3>{name}</h3>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string.isRequired,
  isActive: PropTypes.bool,
  hobbies: PropTypes.arrayOf(PropTypes.string)
};

UserCard.defaultProps = {
  isActive: true,
  hobbies: []
};
```

### 4. Event Handling ve Synthetic Events
**Soru:** React'te event handling nasıl çalışır? Event delegation nedir?

```javascript
function Form() {
  const [formData, setFormData] = useState({ name: '', email: '' });
  
  // Synthetic event
  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  // Event delegation
  const handleFormSubmit = (e) => {
    e.preventDefault(); // Prevent default form submission
    console.log('Form submitted:', formData);
  };
  
  // Event pooling (React 16'da kaldırıldı)
  const handleClick = (e) => {
    // e.persist() - React 16'da gerekliydi
    setTimeout(() => {
      console.log(e.target); // Artık çalışır
    }, 100);
  };
  
  return (
    <form onSubmit={handleFormSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleInputChange}
        placeholder="Name"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleInputChange}
        placeholder="Email"
      />
      <button type="submit" onClick={handleClick}>
        Submit
      </button>
    </form>
  );
}
```

### 5. Conditional Rendering ve Lists
**Soru:** React'te conditional rendering yöntemleri nelerdir? Key prop neden önemlidir?

```javascript
function UserList({ users, showInactive }) {
  // Conditional rendering yöntemleri
  const renderUsers = () => {
    if (users.length === 0) {
      return <p>No users found</p>;
    }
    
    return users
      .filter(user => showInactive || user.isActive)
      .map(user => (
        <UserItem key={user.id} user={user} />
      ));
  };
  
  return (
    <div>
      {/* Ternary operator */}
      {users.length > 0 ? (
        <ul>{renderUsers()}</ul>
      ) : (
        <p>No users available</p>
      )}
      
      {/* Logical AND */}
      {users.length > 0 && <p>Total: {users.length} users</p>}
      
      {/* Switch-like pattern */}
      {(() => {
        switch (users.length) {
          case 0:
            return <p>No users</p>;
          case 1:
            return <p>1 user</p>;
          default:
            return <p>{users.length} users</p>;
        }
      })()}
    </div>
  );
}

// Key prop'un önemi
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id} // Unique, stable key
          todo={todo} 
        />
      ))}
    </ul>
  );
}
```

### 6. Forms ve Controlled Components
**Soru:** Controlled vs Uncontrolled components arasındaki fark nedir?

```javascript
// Controlled Component
function ControlledForm() {
  const [value, setValue] = useState('');
  
  const handleChange = (e) => {
    setValue(e.target.value);
  };
  
  return (
    <input
      type="text"
      value={value}
      onChange={handleChange}
    />
  );
}

// Uncontrolled Component
function UncontrolledForm() {
  const inputRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(inputRef.current.value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={inputRef}
        type="text"
        defaultValue="default value"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 7. Context API ve State Lifting
**Soru:** Context API ne zaman kullanılır? Provider pattern nasıl implement edilir?

```javascript
// Context oluşturma
const ThemeContext = createContext();

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Kullanım
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={theme}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}
```

### 8. useRef ve DOM Manipulation
**Soru:** useRef ne zaman kullanılır? Forward ref nasıl çalışır?

```javascript
// useRef ile DOM manipulation
function FocusInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

// Forward ref
const FancyButton = forwardRef((props, ref) => (
  <button ref={ref} className="fancy-button">
    {props.children}
  </button>
));

// useRef ile previous value tracking
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### 9. Custom Hooks
**Soru:** Custom hook nasıl yazılır? Hangi durumlarda kullanılır?

```javascript
// Custom hook - API data fetching
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
}

// Custom hook - local storage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });
  
  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}

// Kullanım
function UserProfile() {
  const { data: user, loading, error } = useApi('/api/user');
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div className={theme}>
      <h1>{user.name}</h1>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}
```

### 10. Performance Optimization
**Soru:** React'te performance optimization yöntemleri nelerdir?

```javascript
// React.memo - component memoization
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  console.log('ExpensiveComponent rendered');
  return (
    <div>
      {data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
});

// useMemo - expensive calculations
function ExpensiveCalculation({ items }) {
  const expensiveValue = useMemo(() => {
    console.log('Calculating expensive value');
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total: {expensiveValue}</div>;
}

// useCallback - function memoization
function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);
  
  // Bu function her render'da yeniden oluşturulur
  const handleClick = () => {
    setCount(count + 1);
  };
  
  // Bu function sadece count değiştiğinde yeniden oluşturulur
  const handleClickMemoized = useCallback(() => {
    setCount(count + 1);
  }, [count]);
  
  // Bu function hiç yeniden oluşturulmaz
  const handleClickStable = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  return (
    <div>
      <ExpensiveComponent 
        data={items} 
        onUpdate={handleClickStable} 
      />
    </div>
  );
}
```

## React İleri Seviye Sorular

### 1. Memory Management ve Garbage Collection
**Soru:** JavaScript'te WeakMap ve WeakSet'in normal Map ve Set'ten farkları nelerdir? Memory leak'leri nasıl önlerler?

**Cevap:** WeakMap ve WeakSet, referansları "weak" tutar, yani garbage collector'ın objeleri temizlemesine engel olmaz. Normal Map/Set'te referanslar güçlü tutulur ve objeler silinmez.

```javascript
// Memory leak örneği
const map = new Map();
let obj = { data: 'important' };
map.set(obj, 'metadata');
obj = null; // Obje hala Map'te referans edildiği için silinmez

// WeakMap ile çözüm
const weakMap = new WeakMap();
let obj2 = { data: 'important' };
weakMap.set(obj2, 'metadata');
obj2 = null; // Obje artık garbage collect edilebilir
```

### 2. Event Loop ve Microtask Queue
**Soru:** Aşağıdaki kodun çıktısı nedir ve neden?

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
```

**Cevap:** 1, 4, 3, 2
- Call stack: 1, 4
- Microtask queue: 3 (Promise)
- Macrotask queue: 2 (setTimeout)

### 3. Closures ve Lexical Scoping
**Soru:** Aşağıdaki kodun çıktısı nedir?

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);
}
```

**Cevap:** İlk döngü: 3, 3, 3 (var function-scoped)
İkinci döngü: 0, 1, 2 (let block-scoped)

### 4. Prototype Chain ve Inheritance
**Soru:** JavaScript'te multiple inheritance nasıl implement edilir? Mixin pattern'i açıklayın.

```javascript
// Mixin pattern
const CanFly = {
  fly() {
    console.log(`${this.name} is flying`);
  }
};

const CanSwim = {
  swim() {
    console.log(`${this.name} is swimming`);
  }
};

function Duck(name) {
  this.name = name;
}

// Mixin'leri ekle
Object.assign(Duck.prototype, CanFly, CanSwim);

const duck = new Duck('Donald');
duck.fly(); // Donald is flying
duck.swim(); // Donald is swimming
```

### 5. Proxy ve Reflect API
**Soru:** Proxy kullanarak bir objenin property'lerine erişimi nasıl intercept edersiniz?

```javascript
const target = { name: 'John', age: 30 };
const proxy = new Proxy(target, {
  get(target, prop, receiver) {
    console.log(`Accessing property: ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`Setting property: ${prop} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  }
});

proxy.name; // "Accessing property: name"
proxy.age = 31; // "Setting property: age = 31"
```

## React.js İleri Seviye Sorular

### 1. Fiber Architecture ve Reconciliation
**Soru:** React Fiber'ın reconciliation algoritması nasıl çalışır? Time slicing nedir?

**Cevap:** Fiber, React'ın yeni reconciliation engine'idir. Work'ü küçük parçalara böler ve priority'ye göre sıralar. Time slicing ile UI thread'i bloklamadan çalışır.

```javascript
// Fiber node yapısı
const fiberNode = {
  type: 'div',
  key: 'unique-key',
  child: null,
  sibling: null,
  return: null,
  alternate: null, // previous fiber
  effectTag: 'UPDATE',
  memoizedProps: {},
  memoizedState: null
};
```

### 2. Concurrent Features
**Soru:** React 18'deki concurrent features nelerdir? Suspense ve startTransition nasıl çalışır?

```javascript
import { startTransition, useDeferredValue, useTransition } from 'react';

function SearchResults({ query }) {
  const [isPending, startTransition] = useTransition();
  const deferredQuery = useDeferredValue(query);
  
  const results = useMemo(() => {
    return expensiveSearch(deferredQuery);
  }, [deferredQuery]);
  
  return (
    <div>
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </div>
  );
}
```

### 3. Custom Hooks ve State Management
**Soru:** useReducer ile karmaşık state yönetimi nasıl yapılır? Middleware pattern'i nasıl implement edilir?

```javascript
// Reducer middleware
function loggerMiddleware(reducer) {
  return (state, action) => {
    console.log('Previous state:', state);
    console.log('Action:', action);
    const newState = reducer(state, action);
    console.log('New state:', newState);
    return newState;
  };
}

// Custom hook with middleware
function useReducerWithMiddleware(reducer, initialState, middlewares = []) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  const enhancedDispatch = useCallback((action) => {
    let currentDispatch = dispatch;
    
    middlewares.reverse().forEach(middleware => {
      currentDispatch = middleware(currentDispatch);
    });
    
    currentDispatch(action);
  }, [middlewares]);
  
  return [state, enhancedDispatch];
}
```

### 4. Performance Optimization
**Soru:** React.memo, useMemo, useCallback arasındaki farklar nelerdir? Ne zaman kullanılmalıdır?

```javascript
// React.memo - component level memoization
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  return <div>{data.map(item => <Item key={item.id} data={item} />)}</div>;
}, (prevProps, nextProps) => {
  // Custom comparison function
  return prevProps.data.length === nextProps.data.length;
});

// useMemo - expensive calculations
const ExpensiveCalculation = ({ items }) => {
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total: {expensiveValue}</div>;
};

// useCallback - function memoization
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  return <ChildComponent onClick={handleClick} />;
};
```

### 5. Error Boundaries ve Error Handling
**Soru:** Error boundary'lerin sınırları nelerdir? Async error'ları nasıl handle edilir?

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Error reporting service'e gönder
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    
    return this.props.children;
  }
}

// Async error handling
function useAsyncError() {
  const [error, setError] = useState(null);
  
  const throwError = useCallback((error) => {
    setError(error);
  }, []);
  
  if (error) {
    throw error;
  }
  
  return throwError;
}
```

### 6. Server-Side Rendering (SSR) ve Hydration
**Soru:** SSR'da hydration mismatch'leri nasıl önlenir? Selective hydration nedir?

```javascript
// Hydration-safe component
function ClientOnlyComponent({ children }) {
  const [hasMounted, setHasMounted] = useState(false);
  
  useEffect(() => {
    setHasMounted(true);
  }, []);
  
  if (!hasMounted) {
    return null; // SSR'da render edilmez
  }
  
  return children;
}

// Selective hydration
function App() {
  return (
    <div>
      <Header /> {/* Critical, immediate hydration */}
      <Suspense fallback={<Skeleton />}>
        <LazyComponent /> {/* Deferred hydration */}
      </Suspense>
    </div>
  );
}
```

### 7. Advanced Patterns
**Soru:** Compound component pattern'i nasıl implement edilir? Render props vs HOC vs Hooks karşılaştırması?

```javascript
// Compound Component Pattern
const Tabs = ({ children, defaultTab }) => {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
};

const TabList = ({ children }) => (
  <div className="tab-list">{children}</div>
);

const Tab = ({ id, children }) => {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      className={activeTab === id ? 'active' : ''}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
};

const TabPanels = ({ children }) => {
  const { activeTab } = useContext(TabsContext);
  return <div className="tab-panels">{children[activeTab]}</div>;
};

// Usage
<Tabs defaultTab={0}>
  <TabList>
    <Tab id={0}>Tab 1</Tab>
    <Tab id={1}>Tab 2</Tab>
  </TabList>
  <TabPanels>
    <div>Panel 1</div>
    <div>Panel 2</div>
  </TabPanels>
</Tabs>
```

### 8. Testing Strategies
**Soru:** React component'lerini nasıl test edersiniz? Integration test vs Unit test?

```javascript
// Testing with React Testing Library
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('should handle async operations', async () => {
  const user = userEvent.setup();
  render(<AsyncComponent />);
  
  const button = screen.getByRole('button', { name: /load data/i });
  await user.click(button);
  
  await waitFor(() => {
    expect(screen.getByText('Data loaded')).toBeInTheDocument();
  });
});

// Custom render with providers
const customRender = (ui, options) => {
  return render(ui, {
    wrapper: ({ children }) => (
      <ThemeProvider>
        <Router>{children}</Router>
      </ThemeProvider>
    ),
    ...options
  });
};
```

### 9. Bundle Optimization ve Code Splitting
**Soru:** Webpack bundle'ını nasıl optimize edersiniz? Dynamic imports nasıl kullanılır?

```javascript
// Dynamic import with React.lazy
const LazyComponent = React.lazy(() => import('./LazyComponent'));

// Preloading
const LazyComponent = React.lazy(() => 
  import('./LazyComponent').then(module => {
    // Preload related resources
    return module;
  })
);

// Route-based code splitting
const routes = [
  {
    path: '/',
    component: React.lazy(() => import('./Home'))
  },
  {
    path: '/about',
    component: React.lazy(() => import('./About'))
  }
];
```

### 10. Advanced State Management
**Soru:** Zustand vs Redux vs Context API karşılaştırması. Ne zaman hangisini kullanmalı?

```javascript
// Zustand store
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

const useStore = create(
  devtools(
    persist(
      (set, get) => ({
        count: 0,
        increment: () => set(state => ({ count: state.count + 1 })),
        decrement: () => set(state => ({ count: state.count - 1 })),
        reset: () => set({ count: 0 })
      }),
      { name: 'counter-storage' }
    )
  )
);

// Custom hook for complex state
function useComplexState(initialState) {
  const [state, setState] = useState(initialState);
  
  const updateState = useCallback((updates) => {
    setState(prev => ({ ...prev, ...updates }));
  }, []);
  
  const resetState = useCallback(() => {
    setState(initialState);
  }, [initialState]);
  
  return [state, updateState, resetState];
}
```

## JavaScript Engine ve V8 Optimizasyonları

### 1. Hidden Classes ve Inline Caching
**Soru:** V8 engine'inde hidden classes nasıl çalışır? Property access optimization nedir?

```javascript
// Hidden class optimization
function Point(x, y) {
  this.x = x;
  this.y = y;
}

// Bu pattern V8 tarafından optimize edilir
const p1 = new Point(1, 2);
const p2 = new Point(3, 4);

// Property'leri aynı sırada ekle
p1.z = 3;
p2.z = 5; // Aynı hidden class kullanılır
```

### 2. Micro-optimizations
**Soru:** JavaScript'te performans için hangi micro-optimization'ları yapabilirsiniz?

```javascript
// 1. Object property access optimization
const obj = { a: 1, b: 2, c: 3 };
// Kötü: obj['a'] + obj['b']
// İyi: obj.a + obj.b

// 2. Array operations
const arr = [1, 2, 3, 4, 5];
// Kötü: arr.map(x => x * 2).filter(x => x > 4)
// İyi: arr.reduce((acc, x) => { const doubled = x * 2; return doubled > 4 ? [...acc, doubled] : acc; }, [])

// 3. Function declarations vs expressions
// Kötü: const fn = function() { return 'hello'; }
// İyi: function fn() { return 'hello'; }
```

## React Internals ve Advanced Concepts

### 1. React Scheduler
**Soru:** React'ın scheduling algoritması nasıl çalışır? Priority levels nelerdir?

```javascript
// React scheduler priorities
const ImmediatePriority = 1;
const UserBlockingPriority = 2;
const NormalPriority = 3;
const LowPriority = 4;
const IdlePriority = 5;

// Custom scheduler
function scheduleCallback(priorityLevel, callback, options) {
  // React'ın internal scheduler'ı
  return unstable_scheduleCallback(priorityLevel, callback, options);
}
```

### 2. React DevTools Profiler
**Soru:** React DevTools Profiler'da hangi metrikleri takip edersiniz?

- **Render Count**: Component'in kaç kez render edildiği
- **Render Duration**: Render süresi
- **Commit Duration**: DOM'a commit süresi
- **Self Time**: Component'in kendi render süresi
- **Base Time**: Children olmadan render süresi

### 3. React Fiber Debugging
**Soru:** React Fiber tree'sini nasıl debug edersiniz?

```javascript
// Fiber debugging utilities
function getFiberFromElement(element) {
  const key = Object.keys(element).find(key => 
    key.startsWith('__reactInternalInstance$') || 
    key.startsWith('__reactFiber$')
  );
  return element[key];
}

// Fiber tree traversal
function traverseFiber(fiber, callback) {
  callback(fiber);
  if (fiber.child) {
    traverseFiber(fiber.child, callback);
  }
  if (fiber.sibling) {
    traverseFiber(fiber.sibling, callback);
  }
}
```

## Yaygın Mülakat Soruları

### JavaScript Temel Sorular

#### 1. var, let, const Farkları
**Soru:** var, let ve const arasındaki farklar nelerdir?

```javascript
// var - function scoped, hoisted, redeclarable
var name = 'John';
var name = 'Jane'; // OK

// let - block scoped, hoisted ama temporal dead zone, not redeclarable
let age = 25;
// let age = 30; // SyntaxError

// const - block scoped, must be initialized, not reassignable
const city = 'Istanbul';
// city = 'Ankara'; // TypeError
```

#### 2. Arrow Functions vs Regular Functions
**Soru:** Arrow function'ların regular function'lardan farkları nelerdir?

```javascript
// Regular function
function regularFunction() {
  console.log(this); // Function'ın kendi this'i
  return 'Hello';
}

// Arrow function
const arrowFunction = () => {
  console.log(this); // Lexical this (parent scope)
  return 'Hello';
};

// this binding farkı
const obj = {
  name: 'John',
  regular: function() {
    console.log(this.name); // 'John'
  },
  arrow: () => {
    console.log(this.name); // undefined
  }
};
```

#### 3. Closures
**Soru:** Closure nedir? Nasıl çalışır?

```javascript
function outerFunction(x) {
  // Outer function'ın scope'u
  return function innerFunction(y) {
    // Inner function outer'ın variable'larına erişebilir
    return x + y;
  };
}

const add5 = outerFunction(5);
console.log(add5(3)); // 8

// Practical example
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.getCount()); // 1
```

#### 4. Array Methods
**Soru:** map, filter, reduce arasındaki farklar nelerdir?

```javascript
const numbers = [1, 2, 3, 4, 5];

// map - her elementi transform eder
const doubled = numbers.map(n => n * 2); // [2, 4, 6, 8, 10]

// filter - koşula uyan elementleri filtreler
const evens = numbers.filter(n => n % 2 === 0); // [2, 4]

// reduce - array'i tek değere indirger
const sum = numbers.reduce((acc, n) => acc + n, 0); // 15

// Chaining
const result = numbers
  .filter(n => n > 2)
  .map(n => n * 2)
  .reduce((acc, n) => acc + n, 0); // 24
```

### React Temel Sorular

#### 1. JSX Nedir?
**Soru:** JSX nedir? Nasıl çalışır?

```javascript
// JSX
const element = <h1>Hello, {name}!</h1>;

// Babel tarafından şuna dönüştürülür
const element = React.createElement('h1', null, 'Hello, ', name, '!');

// JSX expressions
const isLoggedIn = true;
const element = (
  <div>
    <h1>Welcome!</h1>
    {isLoggedIn ? <p>You are logged in</p> : <p>Please log in</p>}
  </div>
);
```

#### 2. Component Nedir?
**Soru:** React component nedir? Functional vs Class component farkları?

```javascript
// Functional Component (Modern)
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// Arrow Function Component
const Welcome = (props) => <h1>Hello, {props.name}</h1>;

// Class Component (Legacy)
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// Kullanım
<Welcome name="Sara" />
```

#### 3. Props ve State
**Soru:** Props ve State arasındaki fark nedir?

```javascript
// Props - parent'tan child'a geçirilen data
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.email}</p>
    </div>
  );
}

// State - component'in kendi internal data'sı
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

#### 4. Event Handling
**Soru:** React'te event handling nasıl yapılır?

```javascript
function Button() {
  const [count, setCount] = useState(0);
  
  // Event handler
  const handleClick = (event) => {
    event.preventDefault();
    setCount(count + 1);
  };
  
  // Inline event handler
  const handleMouseOver = () => {
    console.log('Mouse over button');
  };
  
  return (
    <button 
      onClick={handleClick}
      onMouseOver={handleMouseOver}
    >
      Clicked {count} times
    </button>
  );
}
```

#### 5. Conditional Rendering
**Soru:** React'te conditional rendering nasıl yapılır?

```javascript
function UserGreeting({ isLoggedIn, username }) {
  // If statement
  if (isLoggedIn) {
    return <h1>Welcome back, {username}!</h1>;
  }
  return <h1>Please sign up.</h1>;
}

// Ternary operator
function LoginButton({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? (
        <button>Logout</button>
      ) : (
        <button>Login</button>
      )}
    </div>
  );
}

// Logical AND operator
function Mailbox({ unreadMessages }) {
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 && (
        <h2>You have {unreadMessages.length} unread messages.</h2>
      )}
    </div>
  );
}
```

#### 6. Lists ve Keys
**Soru:** React'te list rendering nasıl yapılır? Key prop neden önemlidir?

```javascript
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

// Key'in önemi
function NumberList({ numbers }) {
  // Yanlış - index kullanmak
  const wrongList = numbers.map((number, index) => (
    <li key={index}>{number}</li>
  ));
  
  // Doğru - unique, stable key
  const correctList = numbers.map(number => (
    <li key={number.id}>{number.value}</li>
  ));
  
  return <ul>{correctList}</ul>;
}
```

### Orta Seviye Yaygın Sorular

#### 1. useEffect Hook
**Soru:** useEffect ne zaman kullanılır? Dependency array nasıl çalışır?

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // Component mount olduğunda çalışır
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  // userId değiştiğinde çalışır
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId]);
  
  // Cleanup function
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Timer tick');
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  if (loading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}
```

#### 2. Custom Hooks
**Soru:** Custom hook nasıl yazılır? Ne zaman kullanılır?

```javascript
// Custom hook - form handling
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  
  const handleChange = (event) => {
    const { name, value } = event.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  const reset = () => setValues(initialValues);
  
  return { values, handleChange, reset };
}

// Kullanım
function ContactForm() {
  const { values, handleChange, reset } = useForm({
    name: '',
    email: '',
    message: ''
  });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(values);
    reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={values.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        value={values.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <textarea
        name="message"
        value={values.message}
        onChange={handleChange}
        placeholder="Message"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

#### 3. Context API
**Soru:** Context API ne zaman kullanılır? Provider pattern nasıl çalışır?

```javascript
// Context oluşturma
const UserContext = createContext();

// Provider component
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // User data fetch
    fetchUser().then(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);
  
  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);
  
  const value = {
    user,
    loading,
    login,
    logout
  };
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook
function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

// Kullanım
function App() {
  return (
    <UserProvider>
      <Header />
      <Main />
    </UserProvider>
  );
}

function Header() {
  const { user, logout } = useUser();
  
  return (
    <header>
      {user ? (
        <div>
          Welcome, {user.name}!
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <div>Please login</div>
      )}
    </header>
  );
}
```

## Sonuç

Bu sorular, React.js ve JavaScript'in tüm seviyelerini kapsar:

**Temel Seviye:** JavaScript ve React'in temel kavramları
**Orta Seviye:** Hooks, Context API, Custom Hooks, Performance
**İleri Seviye:** Fiber Architecture, Concurrent Features, Advanced Patterns

**Önemli Notlar:**
- Bu sorular sadece teorik bilgi değil, pratik deneyim gerektirir
- Her soru için kod örnekleri ve gerçek kullanım senaryoları verilmiştir
- Mülakat sırasında bu konuları derinlemesine açıklayabilmek önemlidir
- Güncel React ve JavaScript özelliklerini takip etmek gereklidir
- Pratik projeler yaparak bu konuları pekiştirmek önemlidir
