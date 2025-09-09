## BÖLÜM 3: REACT JS - MODERN WEB GELİŞTİRME

# React Referans Dokümantasyonu - react@19.1

Bu bölüm, React ile çalışmak için detaylı referans dokümantasyonu sağlar. React'e giriş için lütfen Öğrenme bölümünü ziyaret edin.

React referans dokümantasyonu işlevsel alt bölümlere ayrılmıştır:

## React

Programatik React özellikleri:

### Hooks - Kancalar
Bileşenlerinizden farklı React özelliklerini kullanın.

#### useActionState
Form eylemlerini ve durumlarını yönetmek için kullanılan hook.

```jsx
import { useActionState } from 'react';

function MyComponent() {
  const [state, formAction, isPending] = useActionState(
    async (prevState, formData) => {
      // Form işleme mantığı
      return { message: 'Form gönderildi!' };
    },
    { message: '' }
  );

  return (
    <form action={formAction}>
      <input name="email" type="email" />
      <button disabled={isPending}>
        {isPending ? 'Gönderiliyor...' : 'Gönder'}
      </button>
      {state.message && <p>{state.message}</p>}
    </form>
  );
}
```

#### useCallback
Fonksiyonları memoize etmek için kullanılan hook.

```jsx
import { useCallback, useState } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

#### useContext
Context değerini okumak için kullanılan hook.

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function ThemedButton() {
  const theme = useContext(ThemeContext);
  
  return (
    <button style={{ 
      background: theme === 'dark' ? '#333' : '#fff',
      color: theme === 'dark' ? '#fff' : '#333'
    }}>
      Themed Button
    </button>
  );
}
```

#### useDebugValue
Özel hook'larda debug değerleri göstermek için kullanılan hook.

```jsx
import { useDebugValue, useState } from 'react';

function useCustomHook() {
  const [value, setValue] = useState(0);
  
  useDebugValue(value > 0 ? 'Pozitif' : 'Negatif veya Sıfır');
  
  return [value, setValue];
}
```

#### useDeferredValue
Değer güncellemelerini ertelemek için kullanılan hook.

```jsx
import { useDeferredValue, useState, useMemo } from 'react';

function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);
  
  const results = useMemo(() => {
    return expensiveSearch(deferredQuery);
  }, [deferredQuery]);
  
  return <div>{results}</div>;
}
```

#### useEffect
Yan etkileri yönetmek için kullanılan hook.

```jsx
import { useEffect, useState } from 'react';

function MyComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Component mount olduğunda çalışır
    fetchData().then(setData);
    
    // Cleanup function
    return () => {
      console.log('Component unmount oldu');
    };
  }, []); // Boş dependency array = sadece mount'ta çalış

  return <div>{data ? data.title : 'Yükleniyor...'}</div>;
}
```

#### useId
Benzersiz ID'ler oluşturmak için kullanılan hook.

```jsx
import { useId } from 'react';

function MyComponent() {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>Email:</label>
      <input id={id} type="email" />
    </div>
  );
}
```

#### useImperativeHandle
Ref'i özelleştirmek için kullanılan hook.

```jsx
import { useImperativeHandle, forwardRef, useRef } from 'react';

const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = '';
    }
  }));

  return <input ref={inputRef} {...props} />;
});
```

#### useInsertionEffect
DOM'a stil eklemeden önce çalışan hook.

```jsx
import { useInsertionEffect } from 'react';

function MyComponent() {
  useInsertionEffect(() => {
    // Stil ekleme işlemleri
    const style = document.createElement('style');
    style.textContent = '.my-class { color: red; }';
    document.head.appendChild(style);
    
    return () => {
      document.head.removeChild(style);
    };
  });

  return <div className="my-class">Styled content</div>;
}
```

#### useLayoutEffect
DOM güncellemelerinden hemen sonra senkron olarak çalışan hook.

```jsx
import { useLayoutEffect, useRef, useState } from 'react';

function MyComponent() {
  const [width, setWidth] = useState(0);
  const ref = useRef();

  useLayoutEffect(() => {
    setWidth(ref.current.offsetWidth);
  });

  return <div ref={ref}>Width: {width}px</div>;
}
```

#### useMemo
Hesaplanmış değerleri memoize etmek için kullanılan hook.

```jsx
import { useMemo, useState } from 'react';

function ExpensiveComponent({ items }) {
  const [filter, setFilter] = useState('');

  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  return (
    <div>
      <input 
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filtrele..."
      />
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### useOptimistic
Optimistic güncellemeler için kullanılan hook.

```jsx
import { useOptimistic, useState } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, pending: true }]
  );

  const addTodo = async (text) => {
    const newTodo = { id: Date.now(), text };
    addOptimisticTodo(newTodo);
    
    try {
      await saveTodo(newTodo);
      setTodos(prev => [...prev, newTodo]);
    } catch (error) {
      // Hata durumunda optimistic güncellemeyi geri al
    }
  };

  return (
    <div>
      {optimisticTodos.map(todo => (
        <div key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
          {todo.text}
        </div>
      ))}
    </div>
  );
}
```

#### useReducer
Karmaşık state yönetimi için kullanılan hook.

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

#### useRef
Mutable ref objesi oluşturmak için kullanılan hook.

```jsx
import { useRef, useEffect } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  const countRef = useRef(0);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  const handleClick = () => {
    countRef.current += 1;
    console.log('Click count:', countRef.current);
  };

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

#### useState
State yönetimi için kullanılan hook.

```jsx
import { useState } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <input 
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="İsminizi girin"
      />
      <p>Merhaba, {name}!</p>
    </div>
  );
}
```

#### useSyncExternalStore
Harici store'lara senkron erişim için kullanılan hook.

```jsx
import { useSyncExternalStore } from 'react';

function useOnlineStatus() {
  return useSyncExternalStore(
    (listener) => {
      window.addEventListener('online', listener);
      window.addEventListener('offline', listener);
      return () => {
        window.removeEventListener('online', listener);
        window.removeEventListener('offline', listener);
      };
    },
    () => navigator.onLine,
    () => true // Server-side fallback
  );
}

function MyComponent() {
  const isOnline = useOnlineStatus();
  
  return <div>Status: {isOnline ? 'Online' : 'Offline'}</div>;
}
```

#### useTransition
State güncellemelerini non-blocking yapmak için kullanılan hook.

```jsx
import { useTransition, useState } from 'react';

function MyComponent() {
  const [isPending, startTransition] = useTransition();
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);

  const handleChange = (e) => {
    setInput(e.target.value);
    
    startTransition(() => {
      // Büyük liste güncellemesi
      const newList = generateLargeList(e.target.value);
      setList(newList);
    });
  };

  return (
    <div>
      <input value={input} onChange={handleChange} />
      {isPending && <div>Güncelleniyor...</div>}
      <ul>
        {list.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

### Components - Bileşenler
JSX'te kullanabileceğiniz yerleşik bileşenler.

#### Fragment (<>) - Parça
Birden fazla element döndürmek için kullanılan bileşen.

```jsx
import { Fragment } from 'react';

function MyComponent() {
  return (
    <Fragment>
      <h1>Başlık</h1>
      <p>Paragraf</p>
    </Fragment>
  );
}

// Kısa syntax
function MyComponent() {
  return (
    <>
      <h1>Başlık</h1>
      <p>Paragraf</p>
    </>
  );
}
```

#### Profiler
Performans ölçümü için kullanılan bileşen.

```jsx
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  console.log('Render:', {
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime
  });
}

function MyComponent() {
  return (
    <Profiler id="MyComponent" onRender={onRenderCallback}>
      <div>Profiled content</div>
    </Profiler>
  );
}
```

#### StrictMode
Geliştirme sırasında potansiyel sorunları tespit etmek için kullanılan bileşen.

```jsx
import { StrictMode } from 'react';

function App() {
  return (
    <StrictMode>
      <MyComponent />
    </StrictMode>
  );
}
```

#### Suspense
Lazy loading ve async işlemler için kullanılan bileşen.

```jsx
import { Suspense, lazy } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function MyComponent() {
  return (
    <Suspense fallback={<div>Yükleniyor...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

#### Activity (Deneysel)
Activity bileşeni - React'ın en son deneysel sürümünde mevcut.

```jsx
import { Activity } from 'react';

function MyComponent() {
  return (
    <Activity>
      <div>Activity content</div>
    </Activity>
  );
}
```

#### ViewTransition (Deneysel)
View transition bileşeni - React'ın en son deneysel sürümünde mevcut.

```jsx
import { ViewTransition } from 'react';

function MyComponent() {
  return (
    <ViewTransition>
      <div>View transition content</div>
    </ViewTransition>
  );
}
```

### APIs - API'ler
Bileşenleri tanımlamak için yararlı API'ler.

#### act
Test ortamında güncellemeleri flush etmek için kullanılan API.

```jsx
import { act } from 'react';

// Test içinde kullanım
act(() => {
  // State güncellemeleri
  setState(newValue);
});
```

#### cache
Fonksiyon sonuçlarını cache'lemek için kullanılan API.

```jsx
import { cache } from 'react';

const fetchUser = cache(async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
});

function UserProfile({ userId }) {
  const user = fetchUser(userId);
  return <div>{user.name}</div>;
}
```

#### captureOwnerStack
Hata ayıklama için owner stack'i yakalamak için kullanılan API.

```jsx
import { captureOwnerStack } from 'react';

function MyComponent() {
  const stack = captureOwnerStack();
  console.log('Owner stack:', stack);
  
  return <div>My Component</div>;
}
```

#### createContext
Context oluşturmak için kullanılan API.

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed Button</button>;
}
```

#### lazy
Bileşenleri lazy load etmek için kullanılan API.

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

#### memo
Bileşenleri memoize etmek için kullanılan API.

```jsx
import { memo } from 'react';

const MyComponent = memo(function MyComponent({ name, age }) {
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
    </div>
  );
});
```

#### startTransition
State güncellemelerini non-blocking yapmak için kullanılan API.

```jsx
import { startTransition, useState } from 'react';

function MyComponent() {
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);

  const handleChange = (e) => {
    setInput(e.target.value);
    
    startTransition(() => {
      setList(generateLargeList(e.target.value));
    });
  };

  return (
    <div>
      <input value={input} onChange={handleChange} />
      <ul>
        {list.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

#### use
Promise ve context değerlerini okumak için kullanılan API.

```jsx
import { use } from 'react';

function MyComponent({ userPromise }) {
  const user = use(userPromise);
  return <div>Hello, {user.name}!</div>;
}
```

#### experimental_taintObjectReference (Deneysel)
Object reference'ları taint etmek için kullanılan API - React'ın en son deneysel sürümünde mevcut.

```jsx
import { experimental_taintObjectReference } from 'react';

// Güvenlik için object reference'ı taint et
experimental_taintObjectReference('Secret data', secretObject);
```

#### experimental_taintUniqueValue (Deneysel)
Unique değerleri taint etmek için kullanılan API - React'ın en son deneysel sürümünde mevcut.

```jsx
import { experimental_taintUniqueValue } from 'react';

// Güvenlik için unique değeri taint et
experimental_taintUniqueValue('Secret value', secretValue);
```

#### unstable_addTransitionType (Deneysel)
Transition tipi eklemek için kullanılan API - React'ın en son deneysel sürümünde mevcut.

```jsx
import { unstable_addTransitionType } from 'react';

// Transition tipi ekle
unstable_addTransitionType('custom-transition');
```

## React DOM

React-dom, yalnızca web uygulamaları (tarayıcı DOM ortamında çalışan) için desteklenen özellikleri içerir. Bu bölüm şu şekilde ayrılmıştır:

### Hooks - Kancalar
Tarayıcı DOM ortamında çalışan web uygulamaları için kancalar.

#### useFormStatus
Form durumunu okumak için kullanılan hook.

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Gönderiliyor...' : 'Gönder'}
    </button>
  );
}
```

### Components - Bileşenler
React, tarayıcının yerleşik HTML ve SVG bileşenlerinin tümünü destekler.

#### Common (örn. div)
Yaygın HTML elementleri.

```jsx
function MyComponent() {
  return (
    <div>
      <h1>Başlık</h1>
      <p>Paragraf</p>
      <span>Span</span>
      <a href="/">Link</a>
    </div>
  );
}
```

#### form
Form elementi.

```jsx
function MyForm() {
  const handleSubmit = (e) => {
    e.preventDefault();
    // Form işleme
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" required />
      <button type="submit">Gönder</button>
    </form>
  );
}
```

#### input
Input elementi.

```jsx
function MyInput() {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
      placeholder="Bir şeyler yazın..."
    />
  );
}
```

#### option
Select seçenekleri.

```jsx
function MySelect() {
  const [selected, setSelected] = useState('');

  return (
    <select value={selected} onChange={(e) => setSelected(e.target.value)}>
      <option value="">Seçiniz</option>
      <option value="option1">Seçenek 1</option>
      <option value="option2">Seçenek 2</option>
    </select>
  );
}
```

#### progress
İlerleme çubuğu.

```jsx
function MyProgress() {
  const [progress, setProgress] = useState(0);

  return (
    <div>
      <progress value={progress} max={100} />
      <p>{progress}% tamamlandı</p>
    </div>
  );
}
```

#### select
Seçim kutusu.

```jsx
function MySelect() {
  const [value, setValue] = useState('');

  return (
    <select value={value} onChange={(e) => setValue(e.target.value)}>
      <option value="value1">Değer 1</option>
      <option value="value2">Değer 2</option>
    </select>
  );
}
```

#### textarea
Çok satırlı metin alanı.

```jsx
function MyTextarea() {
  const [text, setText] = useState('');

  return (
    <textarea
      value={text}
      onChange={(e) => setText(e.target.value)}
      placeholder="Metninizi yazın..."
      rows={4}
    />
  );
}
```

#### link
Link elementi.

```jsx
function MyLink() {
  return (
    <link rel="stylesheet" href="/styles.css" />
  );
}
```

#### meta
Meta elementi.

```jsx
function MyMeta() {
  return (
    <meta name="description" content="Sayfa açıklaması" />
  );
}
```

#### script
Script elementi.

```jsx
function MyScript() {
  return (
    <script src="/script.js" />
  );
}
```

#### style
Style elementi.

```jsx
function MyStyle() {
  return (
    <style>{`
      .my-class {
        color: red;
        font-size: 16px;
      }
    `}</style>
  );
}
```

#### title
Title elementi.

```jsx
function MyTitle() {
  return <title>Sayfa Başlığı</title>;
}
```

### APIs - API'ler
`react-dom` paketi, yalnızca web uygulamalarında desteklenen yöntemler içerir.

#### createPortal
Portal oluşturmak için kullanılan API.

```jsx
import { createPortal } from 'react-dom';

function Modal({ children, isOpen }) {
  if (!isOpen) return null;

  return createPortal(
    <div className="modal">
      {children}
    </div>,
    document.body
  );
}
```

#### flushSync
Güncellemeleri senkron olarak flush etmek için kullanılan API.

```jsx
import { flushSync } from 'react-dom';

function MyComponent() {
  const handleClick = () => {
    flushSync(() => {
      setState(newValue);
    });
    // DOM güncellemesi hemen tamamlandı
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

#### preconnect
DNS lookup ve TCP handshake'i önceden yapmak için kullanılan API.

```jsx
import { preconnect } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    preconnect('https://api.example.com');
  }, []);

  return <div>My Component</div>;
}
```

#### prefetchDNS
DNS lookup'u önceden yapmak için kullanılan API.

```jsx
import { prefetchDNS } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    prefetchDNS('https://api.example.com');
  }, []);

  return <div>My Component</div>;
}
```

#### preinit
Script'i önceden başlatmak için kullanılan API.

```jsx
import { preinit } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    preinit('https://example.com/script.js');
  }, []);

  return <div>My Component</div>;
}
```

#### preinitModule
ES module'i önceden başlatmak için kullanılan API.

```jsx
import { preinitModule } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    preinitModule('https://example.com/module.js');
  }, []);

  return <div>My Component</div>;
}
```

#### preload
Kaynağı önceden yüklemek için kullanılan API.

```jsx
import { preload } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    preload('https://example.com/image.jpg', { as: 'image' });
  }, []);

  return <div>My Component</div>;
}
```

#### preloadModule
ES module'i önceden yüklemek için kullanılan API.

```jsx
import { preloadModule } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    preloadModule('https://example.com/module.js');
  }, []);

  return <div>My Component</div>;
}
```

### Client APIs - İstemci API'leri
`react-dom/client` API'leri, React bileşenlerini istemcide (tarayıcıda) render etmenizi sağlar.

#### createRoot
Root oluşturmak için kullanılan API.

```jsx
import { createRoot } from 'react-dom/client';

const container = document.getElementById('root');
const root = createRoot(container);

root.render(<App />);
```

#### hydrateRoot
Server-side rendered içeriği hydrate etmek için kullanılan API.

```jsx
import { hydrateRoot } from 'react-dom/client';

const container = document.getElementById('root');
hydrateRoot(container, <App />);
```

### Server APIs - Sunucu API'leri
`react-dom/server` API'leri, React bileşenlerini sunucuda HTML'e render etmenizi sağlar.

#### renderToPipeableStream
Pipeable stream'e render etmek için kullanılan API.

```jsx
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />);
pipe(response);
```

#### renderToReadableStream
Readable stream'e render etmek için kullanılan API.

```jsx
import { renderToReadableStream } from 'react-dom/server';

const stream = await renderToReadableStream(<App />);
return new Response(stream);
```

#### renderToStaticMarkup
Static markup'e render etmek için kullanılan API.

```jsx
import { renderToStaticMarkup } from 'react-dom/server';

const html = renderToStaticMarkup(<App />);
```

#### renderToString
String'e render etmek için kullanılan API.

```jsx
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
```

### Static APIs - Statik API'ler

#### prerender
Statik render için kullanılan API.

```jsx
import { prerender } from 'react-dom/server';

await prerender(<App />);
```

#### prerenderToNodeStream
Node stream'e prerender için kullanılan API.

```jsx
import { prerenderToNodeStream } from 'react-dom/server';

const stream = prerenderToNodeStream(<App />);
```

## React Compiler

React Compiler, React bileşenlerinizi ve değerlerinizi otomatik olarak memoize eden build-time optimizasyon aracıdır:

### Configuration - Yapılandırma
React Compiler için yapılandırma seçenekleri.

#### compilationMode
Derleme modunu belirler.

```jsx
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      compilationMode: 'annotation' // veya 'infer'
    }]
  ]
};
```

#### gating
Derleme gating'i.

```jsx
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      gating: true
    }]
  ]
};
```

#### logger
Logger yapılandırması.

```jsx
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      logger: {
        level: 'info'
      }
    }]
  ]
};
```

#### panicThreshold
Panic threshold'u.

```jsx
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      panicThreshold: 0.8
    }]
  ]
};
```

#### target
Hedef platform.

```jsx
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      target: 'react'
    }]
  ]
};
```

### Directives - Direktifler
Derlemeyi kontrol etmek için fonksiyon seviyesi direktifler.

#### "use memo"
Memoization'ı zorla.

```jsx
function MyComponent({ items }) {
  "use memo";
  
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);

  return <div>{expensiveValue}</div>;
}
```

#### "use no memo"
Memoization'ı devre dışı bırak.

```jsx
function MyComponent({ items }) {
  "use no memo";
  
  // Bu fonksiyon memoize edilmeyecek
  const expensiveValue = items.reduce((sum, item) => sum + item.value, 0);

  return <div>{expensiveValue}</div>;
}
```

### Compiling Libraries - Kütüphaneleri Derleme
Önceden derlenmiş kütüphane kodu gönderme rehberi.

```jsx
// Kütüphane geliştiricileri için
export function MyLibraryComponent() {
  // React Compiler ile derlenmiş kod
  return <div>Library Component</div>;
}
```

## Rules of React - React Kuralları

React'ın, kalıpları anlaşılması kolay ve yüksek kaliteli uygulamalar üreten bir şekilde ifade etmek için deyimleri — veya kuralları — vardır:

### Components and Hooks must be pure - Bileşenler ve Kancalar saf olmalıdır
Saf olmak, kodunuzun anlaşılmasını, hata ayıklanmasını kolaylaştırır ve React'ın bileşenlerinizi ve kancalarınızı doğru şekilde otomatik olarak optimize etmesine olanak tanır.

```jsx
// ✅ Saf fonksiyon
function MyComponent({ name }) {
  return <div>Hello, {name}!</div>;
}

// ❌ Saf olmayan fonksiyon
function MyComponent({ name }) {
  console.log('Render edildi'); // Yan etki
  return <div>Hello, {name}!</div>;
}
```

### React calls Components and Hooks - React, Bileşenleri ve Kancaları çağırır
React, kullanıcı deneyimini optimize etmek için gerekli olduğunda bileşenleri ve kancaları render etmekten sorumludur.

```jsx
// React bu bileşeni gerektiğinde çağırır
function MyComponent() {
  const [count, setCount] = useState(0); // React bu kancayı çağırır
  
  return <div>Count: {count}</div>;
}
```

### Rules of Hooks - Kancaların Kuralları
Kancalar JavaScript fonksiyonları kullanılarak tanımlanır, ancak nerede çağrılabilecekleri konusunda kısıtlamaları olan özel bir tür yeniden kullanılabilir UI mantığını temsil ederler.

```jsx
// ✅ Doğru kullanım
function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  return <div>Count: {count}</div>;
}

// ❌ Yanlış kullanım
function MyComponent() {
  if (someCondition) {
    const [count, setCount] = useState(0); // Koşullu kanca çağrısı
  }
  
  return <div>My Component</div>;
}
```

## React Server Components

### Server Components - Sunucu Bileşenleri
Sunucuda çalışan React bileşenleri.

```jsx
// Server Component
async function ServerComponent() {
  const data = await fetch('https://api.example.com/data');
  const json = await data.json();
  
  return <div>{json.title}</div>;
}
```

### Server Functions - Sunucu Fonksiyonları
Sunucuda çalışan fonksiyonlar.

```jsx
// Server Function
async function serverAction(formData) {
  'use server';
  
  const name = formData.get('name');
  // Veritabanı işlemi
  return { success: true };
}
```

### Directives - Direktifler

#### 'use client'
İstemci tarafında çalışacak bileşeni işaretler.

```jsx
'use client';

import { useState } from 'react';

function ClientComponent() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

#### 'use server'
Sunucu tarafında çalışacak fonksiyonu işaretler.

```jsx
async function serverAction(formData) {
  'use server';
  
  const name = formData.get('name');
  // Sunucu işlemi
  return { success: true };
}
```

## Legacy APIs - Eski API'ler

`react` paketinden export edilen, ancak yeni yazılan kodlarda kullanılması önerilmeyen API'ler.

### Legacy React APIs - Eski React API'leri

#### Children
Children utility fonksiyonları.

```jsx
import { Children } from 'react';

function MyComponent({ children }) {
  const childrenArray = Children.toArray(children);
  
  return (
    <div>
      {childrenArray.map((child, index) => (
        <div key={index}>{child}</div>
      ))}
    </div>
  );
}
```

#### cloneElement
Element klonlamak için kullanılan API.

```jsx
import { cloneElement } from 'react';

function MyComponent({ children }) {
  return (
    <div>
      {Children.map(children, child => 
        cloneElement(child, { className: 'cloned' })
      )}
    </div>
  );
}
```

#### Component
Class component base class'ı.

```jsx
import { Component } from 'react';

class MyComponent extends Component {
  render() {
    return <div>Class Component</div>;
  }
}
```

#### createElement
Element oluşturmak için kullanılan API.

```jsx
import { createElement } from 'react';

function MyComponent() {
  return createElement('div', { className: 'my-class' }, 'Hello World');
}
```

#### createRef
Ref oluşturmak için kullanılan API.

```jsx
import { createRef, Component } from 'react';

class MyComponent extends Component {
  constructor(props) {
    super(props);
    this.inputRef = createRef();
  }
  
  render() {
    return <input ref={this.inputRef} />;
  }
}
```

#### forwardRef
Ref forwarding için kullanılan API.

```jsx
import { forwardRef } from 'react';

const MyComponent = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});
```

#### isValidElement
Element geçerliliğini kontrol etmek için kullanılan API.

```jsx
import { isValidElement } from 'react';

function MyComponent({ children }) {
  if (isValidElement(children)) {
    return <div>Valid element: {children}</div>;
  }
  
  return <div>Not a valid element</div>;
}
```

#### PureComponent
Pure class component base class'ı.

```jsx
import { PureComponent } from 'react';

class MyComponent extends PureComponent {
  render() {
    return <div>Pure Component</div>;
  }
}
```

---

**React Nedir?**

React, Facebook (Meta) tarafından geliştirilen, kullanıcı arayüzü oluşturmak için kullanılan açık kaynaklı bir JavaScript kütüphanesidir. React, component tabanlı mimarisi, Virtual DOM teknolojisi ve tek yönlü veri akışı ile modern web uygulamaları geliştirmeyi kolaylaştırır. React, özellikle büyük ve karmaşık uygulamalarda performans ve sürdürülebilirlik sağlar.

**React Ne Demek?**

React, "React to changes" (değişikliklere tepki ver) anlamına gelir. React, kullanıcı arayüzünde meydana gelen değişikliklere otomatik olarak tepki veren, component tabanlı bir kütüphanedir. React'in temel felsefesi, UI'ı state değişikliklerine göre otomatik olarak güncellemektir.

**Neden İhtiyaç Var?**

Web geliştirmede geleneksel yaklaşımların sorunları:

1. **DOM Manipülasyonu**: jQuery ile manuel DOM manipülasyonu yavaş ve hataya açık
2. **State Yönetimi**: Büyük uygulamalarda state yönetimi karmaşık hale gelir
3. **Kod Organizasyonu**: Kod organizasyonu ve sürdürülebilirlik sorunları
4. **Performans**: Büyük uygulamalarda performans sorunları
5. **Developer Experience**: Geliştirici deneyimi sınırlı kalır

**Ne İşe Yarar?**

React, modern web geliştirmeyi daha kolay ve verimli hale getirir:

1. **Component-Based Development**: Yeniden kullanılabilir bileşenler
2. **Virtual DOM**: Yüksek performanslı DOM güncellemeleri
3. **Declarative UI**: Declarative programlama ile UI tanımlama
4. **Unidirectional Data Flow**: Öngörülebilir veri akışı
5. **Rich Ecosystem**: Zengin ekosistem ve araç desteği

**React'in Temel Özellikleri:**

1. **Component-Based Architecture - Bileşen Tabanlı Mimari**: Yeniden kullanılabilir bileşenler
```jsx
// React'te component tabanlı mimari
function Button({ text, onClick, variant = "primary" }) {
    return (
        <button 
            className={`btn btn-${variant}`}
            onClick={onClick}
        >
            {text}
        </button>
    );
}

function App() {
    const handleClick = () => {
        console.log("Button clicked!");
    };

    return (
        <div>
            <Button text="Click Me" onClick={handleClick} />
            <Button text="Secondary" onClick={handleClick} variant="secondary" />
        </div>
    );
}
```

2. **Virtual DOM - Sanal DOM**: Yüksek performanslı DOM güncellemeleri
```jsx
// React'te Virtual DOM
function Counter() {
    const [count, setCount] = useState(0);

    const increment = () => {
        setCount(count + 1); // Virtual DOM otomatik günceller
    };

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>Increment</button>
        </div>
    );
}

// Virtual DOM sadece değişen kısımları günceller
// Gerçek DOM'da sadece <p>Count: {count}</p> güncellenir
```

3. **Unidirectional Data Flow - Tek Yönlü Veri Akışı**: Öngörülebilir veri akışı
```jsx
// React'te unidirectional data flow
function ParentComponent() {
    const [data, setData] = useState("Hello from parent");

    const updateData = (newData) => {
        setData(newData); // Veri sadece parent'tan child'a akar
    };

    return (
        <div>
            <ChildComponent data={data} onUpdate={updateData} />
        </div>
    );
}

function ChildComponent({ data, onUpdate }) {
    return (
        <div>
            <p>{data}</p>
            <button onClick={() => onUpdate("Updated from child")}>
                Update Parent
            </button>
        </div>
    );
}
```

4. **JSX - JavaScript XML**: HTML benzeri syntax
```jsx
// React'te JSX
function UserProfile({ user }) {
    return (
        <div className="user-profile">
            <img src={user.avatar} alt={user.name} />
            <h2>{user.name}</h2>
            <p>{user.email}</p>
            <div className="user-stats">
                <span>Posts: {user.postCount}</span>
                <span>Followers: {user.followerCount}</span>
            </div>
        </div>
    );
}

// JSX JavaScript'e derlenir
// React.createElement("div", { className: "user-profile" }, ...)
```

5. **Hooks - Fonksiyonel Component'ler için State Yönetimi**: Modern state yönetimi
```jsx
// React'te hooks
import { useState, useEffect, useContext } from 'react';

function UserDashboard() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        // Component mount olduğunda çalışır
        fetchUsers();
    }, []);

    const fetchUsers = async () => {
        try {
            setLoading(true);
            const response = await fetch('/api/users');
            const data = await response.json();
            setUsers(data);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    };

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;

    return (
        <div>
            {users.map(user => (
                <UserCard key={user.id} user={user} />
            ))}
        </div>
    );
}
```

**React'in Faydaları:**

1. **Reusable Components - Yeniden Kullanılabilir Bileşenler**: Kod tekrarını önler
```jsx
// Yeniden kullanılabilir component
function Card({ title, content, actions }) {
    return (
        <div className="card">
            <div className="card-header">
                <h3>{title}</h3>
            </div>
            <div className="card-body">
                {content}
            </div>
            {actions && (
                <div className="card-footer">
                    {actions}
                </div>
            )}
        </div>
    );
}

// Farklı yerlerde kullanım
function App() {
    return (
        <div>
            <Card 
                title="User Profile" 
                content={<UserProfile />}
                actions={<button>Edit</button>}
            />
            <Card 
                title="Settings" 
                content={<SettingsForm />}
            />
        </div>
    );
}
```

2. **Performance - Yüksek Performans**: Virtual DOM ile optimize edilmiş güncellemeler
```jsx
// React'te performans optimizasyonu
import { memo, useMemo, useCallback } from 'react';

const ExpensiveComponent = memo(({ data, onUpdate }) => {
    const processedData = useMemo(() => {
        // Sadece data değiştiğinde hesaplanır
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

3. **Developer Experience - Geliştirici Deneyimi**: Gelişmiş araç desteği
```jsx
// React Developer Tools ile debugging
function UserList({ users }) {
    console.log('UserList rendered with:', users);
    
    return (
        <div>
            {users.map(user => (
                <UserItem key={user.id} user={user} />
            ))}
        </div>
    );
}

// React DevTools ile:
// - Component tree görüntüleme
// - Props ve state inceleme
// - Performance profiling
// - Hook debugging
```

**React'in Kullanım Alanları:**

1. **Single Page Applications (SPA) - Tek Sayfa Uygulamaları**: Modern web uygulamaları
2. **Progressive Web Apps (PWA) - İlerleyici Web Uygulamaları**: Mobil benzeri deneyim
3. **E-commerce Platforms - E-ticaret Platformları**: Online mağazalar
4. **Social Media Applications - Sosyal Medya Uygulamaları**: Facebook, Instagram benzeri
5. **Dashboard Applications - Kontrol Paneli Uygulamaları**: Admin panelleri
6. **Content Management Systems - İçerik Yönetim Sistemleri**: CMS uygulamaları
7. **Real-time Applications - Gerçek Zamanlı Uygulamalar**: Chat, live updates
8. **Mobile Applications - Mobil Uygulamalar**: React Native ile cross-platform

### 3.1 React'in Tarihçesi ve Evrimi - Detaylı Analiz

React'in tarihçesi, modern web geliştirmenin en önemli dönüm noktalarından biridir. Bu kütüphane, sadece bir JavaScript kütüphanesi değil, aynı zamanda web geliştirme paradigmasının köklü bir değişimini temsil eder. React'in evrimi, web uygulamalarının karmaşıklığının artması ve geleneksel DOM manipülasyonu yaklaşımlarının sınırlarının aşılması sürecini yansıtır.

**React'in Doğuşu - Neden Gerekli Oldu? - Derinlemesine Analiz**

React'in doğuşu, web geliştirmenin tarihsel sınırlarının aşılması ihtiyacından kaynaklanır. 2010'lu yılların başında, web uygulamaları giderek karmaşıklaşıyor ve geleneksel DOM manipülasyonu yaklaşımları, büyük projelerde ciddi sorunlar yaratıyordu. Bu sorunlar, sadece teknik zorluklar değil, aynı zamanda geliştirici deneyimi, kod kalitesi ve proje sürdürülebilirliği açısından kritik problemlerdi.

**Web Geliştirmenin Tarihsel Sınırları:**

Web geliştirme, 1990'lardan 2010'lara kadar geleneksel DOM manipülasyonu yaklaşımlarına dayanıyordu. Bu yaklaşımlar, küçük projeler için yeterli olurken, büyük ve karmaşık uygulamalar için ciddi zorluklar yaratıyordu. Özellikle Facebook gibi büyük şirketler, bu sınırlarla karşılaştıklarında yeni çözümler aramaya başladılar.

**Geleneksel Yaklaşımların Sorunları:**

**jQuery DOM Manipülasyonu Sorunları:**
jQuery, DOM manipülasyonunu kolaylaştırmış olsa da, büyük uygulamalarda ciddi performans ve sürdürülebilirlik sorunları yaratıyordu. Manuel DOM manipülasyonu, state yönetimi ve event handling gibi konularda karmaşıklık artıyordu.

**Backbone.js MVC Pattern Karmaşıklığı:**
Backbone.js, MVC pattern'ini JavaScript'e getirmiş olsa da, büyük uygulamalarda model-view-controller yapısı karmaşık hale geliyordu. Özellikle data binding ve state synchronization konularında zorluklar yaşanıyordu.

**AngularJS Two-way Data Binding Sorunları:**
AngularJS, two-way data binding ile devrim yaratmış olsa da, performans sorunları ve karmaşıklık yaratıyordu. Özellikle büyük uygulamalarda digest cycle'ları performans sorunlarına neden oluyordu.

**Code Maintainability Sorunları:**
Büyük uygulamalarda kod sürdürülebilirliği ciddi bir sorun haline gelmişti. Farklı geliştiriciler, farklı kodlama stilleri kullanıyor ve bu durum kod tutarlılığını bozuyordu.

**Performance Issues:**
DOM manipülasyonu, özellikle büyük uygulamalarda ciddi performans sorunları yaratıyordu. Her değişiklik, DOM'da manuel güncellemeler gerektiriyordu.

**Facebook'un Stratejik Yaklaşımı:**

Facebook, web uygulamalarındaki bu sorunları fark etmiş ve bu sorunları çözmek için stratejik bir yaklaşım benimsemiştir. Facebook'un amacı, web geliştirmeyi tamamen değiştirmek değil, onu geliştirmek ve güçlendirmektir. Bu yaklaşım, React'in "component-based architecture" olarak tasarlanmasına neden olmuştur.

**Jordan Walke'nin Vizyonu:**

Jordan Walke, React projesinin başında, XHP deneyiminden yararlanarak, JavaScript için modern bir component sistemi tasarlamıştır. Walke'nin vizyonu, web geliştirmenin esnekliğini korurken, performans ve geliştirici deneyimini artırmaktır.

**React'in Tasarım Prensipleri:**

**Component-Based Architecture:**
React, component tabanlı mimariyi benimser. Bu prensip, UI'ı küçük, yeniden kullanılabilir parçalara böler ve kod organizasyonunu iyileştirir.

**Virtual DOM:**
React, Virtual DOM teknolojisini kullanır. Bu prensip, DOM manipülasyonunu optimize eder ve performansı artırır.

**Declarative Programming:**
React, declarative programlama yaklaşımını benimser. Bu prensip, UI'ın nasıl görünmesi gerektiğini tanımlar, nasıl güncelleneceğini değil.

**Unidirectional Data Flow:**
React, tek yönlü veri akışını benimser. Bu prensip, veri akışını öngörülebilir hale getirir ve debugging'i kolaylaştırır.

**Composition over Inheritance:**
React, composition'ı inheritance'a tercih eder. Bu prensip, kod tekrarını azaltır ve esnekliği artırır.

**Web Geliştirmede Geleneksel Yaklaşımların Sorunları:**

1. **jQuery DOM Manipülasyonu**: Manuel DOM manipülasyonu yavaş ve hataya açık
```javascript
// jQuery ile DOM manipülasyonu
$('#user-list').empty();
users.forEach(function(user) {
    var userElement = $('<div class="user">' + user.name + '</div>');
    $('#user-list').append(userElement);
});
```

2. **Backbone.js MVC Pattern**: Karmaşık model-view-controller yapısı
3. **AngularJS Two-way Data Binding**: Performans sorunları ve karmaşıklık
4. **Code Maintainability**: Büyük uygulamalarda kod sürdürülebilirliği sorunları
5. **Performance Issues**: DOM manipülasyonu performans sorunları

**Facebook ve Jordan Walke - React'in Yaratıcısı**:

**Jordan Walke Kimdir?**
- **Facebook'ta Yazılım Mühendisi**: 2000'li yıllardan beri Facebook'ta
- **XHP Geliştiricisi**: PHP için XHP (XML in PHP) kütüphanesinin geliştiricisi
- **Facebook Ads**: Facebook'un reklam sisteminde çalışıyordu
- **Performance Issues**: Facebook'un web uygulamalarındaki performans sorunları

**Jordan Walke'nin Deneyimi:**
```php
// XHP (XML in PHP) - React'in öncüsü
class :ui:button extends :x:element {
    attribute string text;
    attribute string variant = "primary";
    
    protected function render() {
        return <button class={"btn btn-{$this->getAttribute('variant')}"}>
            {$this->getAttribute('text')}
        </button>;
    }
}

// Kullanım
$button = <ui:button text="Click Me" variant="secondary" />;
```

**XHP (XML in PHP) - React'in Öncüsü**:

**2009 - XHP'nin Geliştirilmesi:**
- **Facebook'ta PHP için XHP**: Component tabanlı PHP kütüphanesi
- **Component-based**: PHP'de component tabanlı programlama
- **Type Safety**: PHP'de tip güvenliği
- **Server-side Rendering**: Sunucu tarafında render etme
- **Jordan Walke**: XHP'nin ana geliştiricisi

**XHP'nin Özellikleri:**
```php
// XHP ile component tabanlı programlama
class :ui:user-profile extends :x:element {
    attribute User user;
    
    protected function render() {
        $user = $this->getAttribute('user');
        return <div class="user-profile">
            <img src={$user->avatar} alt={$user->name} />
            <h2>{$user->name}</h2>
            <p>{$user->email}</p>
        </div>;
    }
}
```

**2011 - Facebook'un Web Sorunları**:

**Performance Issues:**
- **Facebook'un Web Uygulamaları**: Büyük ve karmaşık uygulamalar
- **DOM Manipulation**: jQuery ile DOM manipülasyonu yavaştı
- **Code Maintainability**: Kod sürdürülebilirliği sorunları
- **Developer Experience**: Geliştirici deneyimi sorunları

**Geleneksel Yaklaşımın Sorunları:**
```javascript
// jQuery ile karmaşık DOM manipülasyonu
function updateUserList(users) {
    $('#user-list').empty();
    users.forEach(function(user) {
        var userElement = $('<div class="user">');
        userElement.append('<h3>' + user.name + '</h3>');
        userElement.append('<p>' + user.email + '</p>');
        userElement.append('<button class="edit-btn" data-id="' + user.id + '">Edit</button>');
        $('#user-list').append(userElement);
    });
    
    // Event listener'ları yeniden ekle
    $('.edit-btn').off('click').on('click', function() {
        var userId = $(this).data('id');
        editUser(userId);
    });
}
```

**2012 - React Projesinin Başlangıcı**:

**Jordan Walke'nin Vizyonu:**
- **XHP Inspiration**: XHP'den ilham alındı
- **JavaScript Port**: XHP'nin JavaScript'e port edilmesi
- **Component Architecture**: Component tabanlı mimari
- **Virtual DOM**: Sanal DOM kullanımı

**İlk React Prototipi:**
```javascript
// İlk React prototipi (2012)
var UserProfile = React.createClass({
    render: function() {
        return React.createElement('div', { className: 'user-profile' },
            React.createElement('img', { src: this.props.user.avatar }),
            React.createElement('h2', null, this.props.user.name),
            React.createElement('p', null, this.props.user.email)
        );
    }
});

// Kullanım
React.render(
    React.createElement(UserProfile, { user: userData }),
    document.getElementById('app')
);
```

**2013 - React'in Doğuşu**:

**Mayıs 2013 - JSConf US:**
- **React İlk Kez Tanıtıldı**: JSConf US'te Jordan Walke tarafından
- **"Rethinking Best Practices"**: Konuşma başlığı
- **Open Source**: Açık kaynak olarak yayınlandı
- **Facebook Internal**: Facebook'un iç projelerinde kullanıldı

**JSConf US Konuşması:**
```javascript
// Jordan Walke'nin JSConf US'teki örneği
var CommentBox = React.createClass({
    getInitialState: function() {
        return { data: [] };
    },
    
    componentDidMount: function() {
        // Component mount olduğunda çalışır
        this.loadCommentsFromServer();
    },
    
    loadCommentsFromServer: function() {
        $.ajax({
            url: this.props.url,
            dataType: 'json',
            success: function(data) {
                this.setState({ data: data });
            }.bind(this)
        });
    },
    
    render: function() {
        return React.createElement('div', { className: 'commentBox' },
            React.createElement('h1', null, 'Comments'),
            React.createElement(CommentList, { data: this.state.data }),
            React.createElement(CommentForm, { onCommentSubmit: this.handleCommentSubmit })
        );
    }
});
```

**React'in Tasarım Felsefesi:**

1. **Component-Based**: Component tabanlı mimari
2. **Virtual DOM**: Sanal DOM kullanımı
3. **Unidirectional Data Flow**: Tek yönlü veri akışı
4. **Declarative**: Declarative programlama
5. **Learn Once, Write Anywhere**: Bir kez öğren, her yerde yaz

**2013 - React 0.3 - İlk Public Sürüm**:

**Temel Özellikler:**
- **Basic Components**: Temel component sistemi
- **JSX**: JavaScript XML syntax
- **Props ve State**: Props ve state yönetimi
- **Virtual DOM**: İlk Virtual DOM implementasyonu

**React 0.3 Örneği:**
```javascript
// React 0.3'te ilk örnek
var HelloWorld = React.createClass({
    render: function() {
        return React.createElement('div', null, 'Hello, World!');
    }
});

// JSX ile
var HelloWorld = React.createClass({
    render: function() {
        return <div>Hello, World!</div>;
    }
});

// Kullanım
React.render(<HelloWorld />, document.getElementById('app'));
```

**2014 - React 0.12 - Performans İyileştirmeleri**:

**Yeni Özellikler:**
- **Performance Improvements**: Performans iyileştirmeleri
- **Better JSX**: Gelişmiş JSX desteği
- **PropTypes**: Prop validation sistemi
- **React.Children**: Children manipulation utilities

**PropTypes Örneği:**
```javascript
// React 0.12'de PropTypes
var UserProfile = React.createClass({
    propTypes: {
        user: React.PropTypes.object.isRequired,
        showEmail: React.PropTypes.bool
    },
    
    getDefaultProps: function() {
        return {
            showEmail: true
        };
    },
    
    render: function() {
        return (
            <div className="user-profile">
                <h2>{this.props.user.name}</h2>
                {this.props.showEmail && <p>{this.props.user.email}</p>}
            </div>
        );
    }
});
```

**2015 - React 0.13 - ES6 Desteği**:

**Yeni Özellikler:**
- **ES6 Classes**: ES6 sınıf desteği
- **createClass Deprecation**: createClass kullanımı uyarısı
- **Better Performance**: Performans iyileştirmeleri
- **TypeScript Support**: TypeScript desteği

**ES6 Classes Örneği:**
```javascript
// React 0.13'te ES6 classes
class UserProfile extends React.Component {
    constructor(props) {
        super(props);
        this.state = { isEditing: false };
    }
    
    handleEdit = () => {
        this.setState({ isEditing: true });
    }
    
    render() {
        return (
            <div className="user-profile">
                <h2>{this.props.user.name}</h2>
                {this.state.isEditing ? (
                    <EditForm user={this.props.user} />
                ) : (
                    <button onClick={this.handleEdit}>Edit</button>
                )}
            </div>
        );
    }
}
```

**2015 - React 0.14 - Stateless Functional Components**:

**Yeni Özellikler:**
- **Stateless Functional Components**: Stateless functional component'ler
- **ReactDOM Separation**: ReactDOM ayrımı
- **Performance Improvements**: Performans iyileştirmeleri

**Stateless Functional Components Örneği:**
```javascript
// React 0.14'te stateless functional components
function UserCard({ user, onEdit }) {
    return (
        <div className="user-card">
            <h3>{user.name}</h3>
            <p>{user.email}</p>
            <button onClick={() => onEdit(user.id)}>Edit</button>
        </div>
    );
}

// ReactDOM ayrımı
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(<App />, document.getElementById('root'));
```
- **PropTypes Package**: PropTypes paketi
- **Better Testing**: Daha iyi test desteği

**2016 - React 15**:
- **Error Boundaries**: Hata sınırları
- **React DevTools**: React geliştirici araçları
- **Fiber Architecture**: Fiber mimarisi (experimental)
- **Better Performance**: Performans iyileştirmeleri

**2017 - React 16**:
- **Fiber Architecture**: Yeni reconciliation algoritması
- **Error Boundaries**: Hata yakalama ve yönetimi
- **Portals**: DOM tree dışında render etme
- **Fragments**: Wrapper element olmadan gruplama
- **Custom DOM Attributes**: Özel HTML özellikleri
- **setState with Function**: Fonksiyonel setState
- **ComponentDidCatch**: Hata yakalama lifecycle'ı

**2018 - React 16.3**:
- **New Lifecycle Methods**: getDerivedStateFromProps, getSnapshotBeforeUpdate
- **Context API**: Stable context API
- **createRef**: Ref oluşturma API'si
- **forwardRef**: Ref forwarding

**2018 - React 16.6**:
- **React.memo**: Functional component memoization
- **React.lazy**: Code splitting için lazy loading
- **Suspense**: Asenkron component loading
- **getDerivedStateFromError**: Error boundary için lifecycle

**2019 - React 16.8**:
- **Hooks**: Functional components için state ve lifecycle
- **useState**: State yönetimi
- **useEffect**: Side effects yönetimi
- **useContext**: Context kullanımı
- **useReducer**: Complex state yönetimi
- **useCallback**: Function memoization
- **useMemo**: Value memoization
- **useRef**: Mutable ref object
- **useImperativeHandle**: Custom ref value
- **useLayoutEffect**: Synchronous effect
- **useDebugValue**: Custom hook debugging

**2020 - React 17**:
- **New JSX Transform**: Babel transform değişiklikleri
- **Event Delegation Changes**: Event handling optimizasyonları
- **Effect Cleanup Timing**: useEffect cleanup timing değişiklikleri
- **No Breaking Changes**: Büyük API değişiklikleri yok

**2022 - React 18**:
- **Concurrent Features**: Concurrent rendering
- **Automatic Batching**: Otomatik state batching
- **Suspense Improvements**: Gelişmiş Suspense desteği
- **useId**: Unique ID generation
- **useDeferredValue**: Deferred value updates
- **useTransition**: Transition state management
- **useSyncExternalStore**: External store synchronization
- **useInsertionEffect**: CSS-in-JS için effect
- **createRoot**: Yeni root API
- **hydrateRoot**: SSR hydration API

**2024 - React 19**:
- **Server Components**: Sunucu tarafında render edilen component'ler
- **Actions API**: Form actions ve server actions
- **use() Hook**: Promise ve context için hook
- **useOptimistic**: Optimistic updates
- **useFormStatus**: Form durumu yönetimi
- **useFormState**: Form state yönetimi
- **Document Metadata**: Head yönetimi
- **Resource Preloading**: Kaynak ön yükleme
- **Custom Elements Support**: Web Components desteği
- **Improved DevTools**: Gelişmiş geliştirici araçları
- **Better TypeScript Support**: Gelişmiş TypeScript desteği
- **Performance Improvements**: Performans iyileştirmeleri

**2025 - React 19.1+**:
- **Improved Concurrent Rendering**: Daha akıcı kullanıcı deneyimi
- **Automatic Hydration**: Otomatik hydration optimizasyonları
- **Enhanced Server Components**: Gelişmiş sunucu component'leri
- **React Compiler Integration**: React Compiler ile entegrasyon
- **WebAssembly Support**: WebAssembly ile performans optimizasyonu
- **Better Error Boundaries**: Gelişmiş hata sınırları
- **Improved Suspense**: Gelişmiş Suspense desteği
- **Enhanced DevTools**: Daha gelişmiş geliştirici araçları
- **Stricter TypeScript Integration**: Daha sıkı TypeScript entegrasyonu
- **Performance Monitoring**: Performans izleme araçları

**İlk Çözümler**:
- **Virtual DOM**: Gerçek DOM manipülasyonunu minimize ederek performansı artırdı
- **Component-Based Architecture**: Yeniden kullanılabilir UI bileşenleri
- **Unidirectional Data Flow**: Veri akışının tek yönlü olması ile hata ayıklamayı kolaylaştırdı
- **Declarative UI**: Imperative DOM manipülasyonu yerine declarative yaklaşım
- **Server-Side Rendering**: Sunucu tarafında render etme
- **Code Splitting**: Kod bölme ve lazy loading
- **Error Boundaries**: Hata yakalama ve yönetimi
- **Performance Optimization**: Performans optimizasyonu

### 3.2 React Temel Kavramlar - Detaylı Analiz

React'in temel kavramları, modern component-based web geliştirmenin temelini oluşturur. Bu kavramlar, sadece syntax öğrenmek değil, aynı zamanda React'in felsefesini ve tasarım prensiplerini anlamak için kritik öneme sahiptir. React'in temel kavramları, functional programming, component composition ve declarative programming gibi modern programlama paradigmalarını harmanlayarak, güçlü ve esnek bir geliştirme deneyimi sunar.

**React Temel Kavramlarının Felsefesi:**

React'in temel kavramları, "component-based thinking" prensibini benimser. Bu prensip, UI'ı küçük, bağımsız ve yeniden kullanılabilir parçalara böler. Bu yaklaşım, kod organizasyonunu iyileştirir, test edilebilirliği artırır ve geliştirici deneyimini optimize eder.

**React Temel Kavramlarının Kategorileri:**

**Component Architecture (Bileşen Mimarisi):**
Component'ler, React'in temel yapı taşlarıdır. Bu mimari, UI'ı küçük, bağımsız ve yeniden kullanılabilir parçalara böler.

**State Management (Durum Yönetimi):**
State management, component'lerin iç durumunu ve veri akışını yönetir. Bu konu, useState, useEffect ve custom hooks gibi konuları kapsar.

**Props System (Props Sistemi):**
Props sistemi, component'ler arası veri iletişimini sağlar. Bu sistem, parent-child communication ve data flow'u yönetir.

**Event Handling (Olay Yönetimi):**
Event handling, kullanıcı etkileşimlerini yönetir. Bu konu, synthetic events, event delegation ve event handling patterns gibi konuları kapsar.

**Lifecycle Management (Yaşam Döngüsü Yönetimi):**
Lifecycle management, component'lerin yaşam döngüsünü yönetir. Bu konu, mounting, updating ve unmounting gibi konuları kapsar.

**Component Architecture - Derinlemesine Analiz:**

**Component'lerin Felsefesi:**
Component'ler, React'in temel felsefesini yansıtır. Her component, belirli bir sorumluluğa sahip olmalı ve tek bir işi iyi yapmalıdır. Bu prensip, Single Responsibility Principle'ı yansıtır.

**Component Composition:**
Component composition, küçük component'leri birleştirerek büyük component'ler oluşturma prensibidir. Bu yaklaşım, inheritance'a tercih edilir ve kod tekrarını azaltır.

**Component Reusability:**
Component reusability, component'lerin farklı yerlerde kullanılabilmesi prensibidir. Bu özellik, kod tekrarını azaltır ve tutarlılığı artırır.

**Component Isolation:**
Component isolation, component'lerin birbirinden bağımsız olması prensibidir. Bu özellik, test edilebilirliği artırır ve debugging'i kolaylaştırır.

**State Management - Derinlemesine Analiz:**

**State'in Felsefesi:**
State, component'in iç durumunu temsil eder. State, component'in davranışını ve görünümünü etkiler. State management, React'in en kritik konularından biridir.

**State Immutability:**
State immutability, state'in değiştirilemez olması prensibidir. Bu prensip, state güncellemelerinin öngörülebilir olmasını sağlar.

**State Lifting:**
State lifting, state'i component hierarchy'de yukarı taşıma prensibidir. Bu prensip, state'i paylaşan component'ler arasında veri akışını sağlar.

**State Normalization:**
State normalization, state'in normalize edilmiş formda tutulması prensibidir. Bu prensip, state yönetimini kolaylaştırır.

**Props System - Derinlemesine Analiz:**

**Props'in Felsefesi:**
Props, component'ler arası veri iletişimini sağlar. Props, parent component'ten child component'e veri geçirme mekanizmasıdır.

**Props Validation:**
Props validation, props'ların doğru tipte ve formatta olmasını sağlar. Bu özellik, runtime hatalarını önler.

**Props Drilling:**
Props drilling, props'ların çok sayıda component'ten geçirilmesi sorunudur. Bu sorun, Context API veya state management kütüphaneleri ile çözülür.

**Props Composition:**
Props composition, props'ları birleştirerek yeni props'lar oluşturma prensibidir. Bu prensip, props'ların yeniden kullanılabilirliğini artırır.

**Event Handling - Derinlemesine Analiz:**

**Synthetic Events:**
Synthetic events, React'in cross-browser uyumlu event sistemi. Bu sistem, farklı tarayıcılarda tutarlı event handling sağlar.

**Event Delegation:**
Event delegation, event'lerin parent element'te dinlenmesi prensibidir. Bu prensip, performansı artırır ve memory kullanımını azaltır.

**Event Handling Patterns:**
Event handling patterns, event'lerin nasıl handle edileceğini belirler. Bu pattern'ler, callback functions, event handlers ve event binding gibi konuları kapsar.

**Lifecycle Management - Derinlemesine Analiz:**

**Component Lifecycle:**
Component lifecycle, component'in yaşam döngüsünü temsil eder. Bu döngü, mounting, updating ve unmounting aşamalarını içerir.

**Lifecycle Methods:**
Lifecycle methods, component'in yaşam döngüsünde çalışan method'lardır. Bu method'lar, component'in davranışını kontrol eder.

**Effect Hooks:**
Effect hooks, side effect'leri yönetir. Bu hook'lar, component'in yaşam döngüsünde çalışır.

**React Temel Kavramlarının Avantajları:**

**Code Organization:**
React'in temel kavramları, kod organizasyonunu iyileştirir. Component-based architecture, kodun modüler olmasını sağlar.

**Reusability:**
React'in temel kavramları, kod yeniden kullanılabilirliğini artırır. Component'ler, farklı yerlerde kullanılabilir.

**Testability:**
React'in temel kavramları, test edilebilirliği artırır. Component'ler, bağımsız olarak test edilebilir.

**Maintainability:**
React'in temel kavramları, kod sürdürülebilirliğini artırır. Component-based architecture, kodun anlaşılabilir olmasını sağlar.

**Developer Experience:**
React'in temel kavramları, geliştirici deneyimini iyileştirir. Declarative programming, kod yazmayı kolaylaştırır.

**React Temel Kavramlarının Kullanım Senaryoları:**

**Small to Medium Applications:**
React'in temel kavramları, küçük ve orta ölçekli uygulamalar için idealdir. Bu uygulamalarda, component-based architecture yeterlidir.

**Component Libraries:**
React'in temel kavramları, component kütüphaneleri oluşturmak için idealdir. Bu kütüphaneler, yeniden kullanılabilir component'ler sağlar.

**Prototype Development:**
React'in temel kavramları, prototip geliştirme için idealdir. Bu süreçte, hızlı geliştirme önemlidir.

**Learning and Education:**
React'in temel kavramları, öğrenme ve eğitim için idealdir. Bu kavramlar, modern web geliştirmenin temelini oluşturur.

#### 3.2.1 Component'ler - Derinlemesine Analiz

React'te her şey component'lerden oluşur. Component'ler, UI'ın yeniden kullanılabilir parçalarıdır ve modern web geliştirmenin temel yapı taşlarıdır. Component'ler, sadece UI elementleri değil, aynı zamanda uygulama mantığının ve state yönetiminin merkezi noktalarıdır.

**Function Component'ler**:

```jsx
// Temel function component
function Welcome(props) {
    return <h1>Hello, {props.name}!</h1>;
}

// Arrow function component
const Welcome = (props) => {
    return <h1>Hello, {props.name}!</h1>;
};

// Destructuring props
const Welcome = ({ name, age }) => {
    return (
        <div>
            <h1>Hello, {name}!</h1>
            <p>You are {age} years old.</p>
        </div>
    );
};

// Default props
const Welcome = ({ name = "Guest", age = 0 }) => {
    return (
        <div>
            <h1>Hello, {name}!</h1>
            <p>You are {age} years old.</p>
        </div>
    );
};

// Component kullanımı
function App() {
    return (
        <div>
            <Welcome name="John" age={30} />
            <Welcome name="Jane" age={25} />
            <Welcome /> {/* Default props kullanılır */}
        </div>
    );
}
```

**Class Component'ler**:

```jsx
// Temel class component
class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}!</h1>;
    }
}

// Class component with state
class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        };
    }

    increment = () => {
        this.setState({ count: this.state.count + 1 });
    }

    decrement = () => {
        this.setState({ count: this.state.count - 1 });
    }

    render() {
        return (
            <div>
                <h2>Count: {this.state.count}</h2>
                <button onClick={this.increment}>+</button>
                <button onClick={this.decrement}>-</button>
            </div>
        );
    }
}

// Class component with lifecycle methods
class UserProfile extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            user: null,
            loading: true
        };
    }

    componentDidMount() {
        // Component mount olduğunda çalışır
        this.fetchUser();
    }

    componentDidUpdate(prevProps) {
        // Props değiştiğinde çalışır
        if (prevProps.userId !== this.props.userId) {
            this.fetchUser();
        }
    }

    componentWillUnmount() {
        // Component unmount olmadan önce çalışır
        // Cleanup işlemleri
    }

    fetchUser = async () => {
        try {
            const response = await fetch(`/api/users/${this.props.userId}`);
            const user = await response.json();
            this.setState({ user, loading: false });
        } catch (error) {
            this.setState({ loading: false });
        }
    }

    render() {
        if (this.state.loading) {
            return <div>Loading...</div>;
        }

        return (
            <div>
                <h1>{this.state.user.name}</h1>
                <p>{this.state.user.email}</p>
            </div>
        );
    }
}
```

#### 3.2.2 Props ve State

**Props (Properties)**:

```jsx
// Props passing
function Parent() {
    const user = {
        name: "John",
        age: 30,
        email: "john@example.com"
    };

    return <Child user={user} isActive={true} />;
}

function Child({ user, isActive }) {
    return (
        <div className={isActive ? "active" : "inactive"}>
            <h2>{user.name}</h2>
            <p>Age: {user.age}</p>
            <p>Email: {user.email}</p>
        </div>
    );
}

// Props validation with PropTypes
import PropTypes from 'prop-types';

function UserCard({ name, age, email, isActive }) {
    return (
        <div className={`user-card ${isActive ? 'active' : 'inactive'}`}>
            <h3>{name}</h3>
            <p>Age: {age}</p>
            <p>Email: {email}</p>
        </div>
    );
}

UserCard.propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number.isRequired,
    email: PropTypes.string.isRequired,
    isActive: PropTypes.bool
};

UserCard.defaultProps = {
    isActive: false
};

// Children props
function Card({ children, title }) {
    return (
        <div className="card">
            {title && <h3 className="card-title">{title}</h3>}
            <div className="card-content">
                {children}
            </div>
        </div>
    );
}

function App() {
    return (
        <Card title="User Information">
            <p>This is the card content.</p>
            <button>Click me</button>
        </Card>
    );
}
```

**State Management**:

```jsx
// useState Hook
import React, { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('');

    const increment = () => {
        setCount(count + 1);
    };

    const decrement = () => {
        setCount(count - 1);
    };

    const reset = () => {
        setCount(0);
    };

    return (
        <div>
            <h2>Count: {count}</h2>
            <input 
                value={name}
                onChange={(e) => setName(e.target.value)}
                placeholder="Enter your name"
            />
            <p>Hello, {name}!</p>
            <button onClick={increment}>+</button>
            <button onClick={decrement}>-</button>
            <button onClick={reset}>Reset</button>
        </div>
    );
}

// Multiple state variables
function UserForm() {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        age: 0
    });

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prevState => ({
            ...prevState,
            [name]: value
        }));
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        console.log('Form submitted:', formData);
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                name="name"
                value={formData.name}
                onChange={handleChange}
                placeholder="Name"
            />
            <input
                name="email"
                type="email"
                value={formData.email}
                onChange={handleChange}
                placeholder="Email"
            />
            <input
                name="age"
                type="number"
                value={formData.age}
                onChange={handleChange}
                placeholder="Age"
            />
            <button type="submit">Submit</button>
        </form>
    );
}

// State with objects and arrays
function TodoList() {
    const [todos, setTodos] = useState([]);
    const [newTodo, setNewTodo] = useState('');

    const addTodo = () => {
        if (newTodo.trim()) {
            setTodos(prevTodos => [
                ...prevTodos,
                {
                    id: Date.now(),
                    text: newTodo,
                    completed: false
                }
            ]);
            setNewTodo('');
        }
    };

    const toggleTodo = (id) => {
        setTodos(prevTodos =>
            prevTodos.map(todo =>
                todo.id === id
                    ? { ...todo, completed: !todo.completed }
                    : todo
            )
        );
    };

    const deleteTodo = (id) => {
        setTodos(prevTodos =>
            prevTodos.filter(todo => todo.id !== id)
        );
    };

    return (
        <div>
            <div>
                <input
                    value={newTodo}
                    onChange={(e) => setNewTodo(e.target.value)}
                    placeholder="Add a new todo"
                />
                <button onClick={addTodo}>Add Todo</button>
            </div>
            <ul>
                {todos.map(todo => (
                    <li key={todo.id}>
                        <span
                            style={{
                                textDecoration: todo.completed ? 'line-through' : 'none'
                            }}
                            onClick={() => toggleTodo(todo.id)}
                        >
                            {todo.text}
                        </span>
                        <button onClick={() => deleteTodo(todo.id)}>Delete</button>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

#### 3.2.3 Event Handling

```jsx
// Temel event handling
function Button() {
    const handleClick = () => {
        console.log('Button clicked!');
    };

    return <button onClick={handleClick}>Click me</button>;
}

// Event handling with parameters
function TodoItem({ todo, onToggle, onDelete }) {
    const handleToggle = () => {
        onToggle(todo.id);
    };

    const handleDelete = () => {
        onDelete(todo.id);
    };

    return (
        <li>
            <span onClick={handleToggle}>
                {todo.text}
            </span>
            <button onClick={handleDelete}>Delete</button>
        </li>
    );
}

// Form event handling
function ContactForm() {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        message: ''
    });

    const handleInputChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        console.log('Form submitted:', formData);
        // Form submission logic
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                name="name"
                value={formData.name}
                onChange={handleInputChange}
                placeholder="Your name"
            />
            <input
                name="email"
                type="email"
                value={formData.email}
                onChange={handleInputChange}
                placeholder="Your email"
            />
            <textarea
                name="message"
                value={formData.message}
                onChange={handleInputChange}
                placeholder="Your message"
            />
            <button type="submit">Send Message</button>
        </form>
    );
}

// Synthetic events
function EventExample() {
    const handleMouseEnter = (e) => {
        console.log('Mouse entered:', e.target);
    };

    const handleMouseLeave = (e) => {
        console.log('Mouse left:', e.target);
    };

    const handleKeyDown = (e) => {
        if (e.key === 'Enter') {
            console.log('Enter key pressed');
        }
    };

    return (
        <div>
            <div
                onMouseEnter={handleMouseEnter}
                onMouseLeave={handleMouseLeave}
                style={{ padding: '20px', border: '1px solid #ccc' }}
            >
                Hover over me
            </div>
            <input
                onKeyDown={handleKeyDown}
                placeholder="Press Enter"
            />
        </div>
    );
}
```

#### 3.2.4 React Hooks

Hooks, function component'lerde state ve lifecycle özelliklerini kullanmamızı sağlar:

**useState Hook**:

```jsx
import React, { useState } from 'react';

// Temel useState kullanımı
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>+</button>
            <button onClick={() => setCount(count - 1)}>-</button>
        </div>
    );
}

// Functional update
function Counter() {
    const [count, setCount] = useState(0);

    const increment = () => {
        setCount(prevCount => prevCount + 1);
    };

    const decrement = () => {
        setCount(prevCount => prevCount - 1);
    };

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>+</button>
            <button onClick={decrement}>-</button>
        </div>
    );
}

// Lazy initial state
function ExpensiveComponent() {
    const [data, setData] = useState(() => {
        // Bu sadece ilk render'da çalışır
        return expensiveCalculation();
    });

    return <div>{data}</div>;
}
```

**useEffect Hook**:

```jsx
import React, { useState, useEffect } from 'react';

// Temel useEffect
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        // Component mount olduğunda çalışır
        fetchUser();
    }, []); // Empty dependency array

    useEffect(() => {
        // userId değiştiğinde çalışır
        if (userId) {
            fetchUser();
        }
    }, [userId]); // userId dependency

    const fetchUser = async () => {
        setLoading(true);
        try {
            const response = await fetch(`/api/users/${userId}`);
            const userData = await response.json();
            setUser(userData);
        } catch (error) {
            console.error('Error fetching user:', error);
        } finally {
            setLoading(false);
        }
    };

    if (loading) return <div>Loading...</div>;
    if (!user) return <div>User not found</div>;

    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
        </div>
    );
}

// Cleanup function
function Timer() {
    const [seconds, setSeconds] = useState(0);

    useEffect(() => {
        const interval = setInterval(() => {
            setSeconds(prevSeconds => prevSeconds + 1);
        }, 1000);

        // Cleanup function
        return () => {
            clearInterval(interval);
        };
    }, []);

    return <div>Seconds: {seconds}</div>;
}
```

**useContext Hook**:

```jsx
import React, { createContext, useContext, useState } from 'react';

// Context oluşturma
const ThemeContext = createContext();

// Provider component
function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');

    const toggleTheme = () => {
        setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
    };

    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

// Consumer component
function ThemedButton() {
    const { theme, toggleTheme } = useContext(ThemeContext);

    return (
        <button
            onClick={toggleTheme}
            style={{
                backgroundColor: theme === 'light' ? '#fff' : '#333',
                color: theme === 'light' ? '#333' : '#fff'
            }}
        >
            Toggle Theme
        </button>
    );
}
```

**useReducer Hook**:

```jsx
import React, { useReducer } from 'react';

// Reducer function
function todoReducer(state, action) {
    switch (action.type) {
        case 'ADD_TODO':
            return [...state, {
                id: Date.now(),
                text: action.text,
                completed: false
            }];
        case 'TOGGLE_TODO':
            return state.map(todo =>
                todo.id === action.id
                    ? { ...todo, completed: !todo.completed }
                    : todo
            );
        case 'DELETE_TODO':
            return state.filter(todo => todo.id !== action.id);
        default:
            return state;
    }
}

function TodoApp() {
    const [todos, dispatch] = useReducer(todoReducer, []);
    const [inputValue, setInputValue] = useState('');

    const addTodo = () => {
        if (inputValue.trim()) {
            dispatch({ type: 'ADD_TODO', text: inputValue });
            setInputValue('');
        }
    };

    return (
        <div>
            <input
                value={inputValue}
                onChange={(e) => setInputValue(e.target.value)}
                placeholder="Add a todo"
            />
            <button onClick={addTodo}>Add</button>
            <ul>
                {todos.map(todo => (
                    <li key={todo.id}>{todo.text}</li>
                ))}
            </ul>
        </div>
    );
}
```

**Custom Hooks**:

```jsx
// Custom hook for API calls
function useApi(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        let cancelled = false;

        const fetchData = async () => {
            try {
                setLoading(true);
                setError(null);
                const response = await fetch(url);
                const result = await response.json();
                
                if (!cancelled) {
                    setData(result);
                }
            } catch (err) {
                if (!cancelled) {
                    setError(err.message);
                }
            } finally {
                if (!cancelled) {
                    setLoading(false);
                }
            }
        };

        fetchData();

        return () => {
            cancelled = true;
        };
    }, [url]);

    return { data, loading, error };
}

// Custom hook for local storage
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
            console.error('Error saving to localStorage:', error);
        }
    };

    return [storedValue, setValue];
}
```

### 3.3 React Versiyonları

**React 0.3 (2013)**:
- **Temel component sistemi**: İlk component architecture
- **JSX syntax**: JavaScript XML syntax tanıtımı
- **Props ve state yönetimi**: Temel veri akışı
- **Virtual DOM**: İlk Virtual DOM implementasyonu
- **React.createClass**: Component oluşturma API'si
- **React.render**: Component render etme

**React 0.4 (2013)**:
- **Key prop optimizasyonu**: List rendering optimizasyonu
- **React.Children utilities**: Children manipulation utilities
- **Improved error handling**: Gelişmiş hata yönetimi
- **Performance improvements**: Performans iyileştirmeleri

**React 0.5 (2013)**:
- **Refs sistemi**: Component referansları
- **React.findDOMNode**: DOM node bulma
- **Event system improvements**: Olay sistemi iyileştirmeleri
- **Better browser support**: Daha iyi tarayıcı desteği

**React 0.6 (2013)**:
- **Lifecycle methods**: componentDidMount, componentWillUnmount
- **React.cloneElement**: Element klonlama
- **Improved JSX**: JSX iyileştirmeleri
- **Better debugging**: Daha iyi hata ayıklama

**React 0.7 (2013)**:
- **Context API (experimental)**: İlk context implementasyonu
- **React.PropTypes**: Prop validation sistemi
- **React.addons**: Addon sistemi
- **Performance optimizations**: Performans optimizasyonları

**React 0.8 (2013)**:
- **React.createFactory**: Factory pattern
- **Better memory management**: Bellek yönetimi iyileştirmeleri
- **Improved reconciliation**: Reconciliation algoritması iyileştirmeleri
- **React.Children.map**: Children iteration

**React 0.9 (2013)**:
- **React.unmountComponentAtNode**: Component unmounting
- **React.renderToString**: Server-side rendering
- **React.renderToStaticMarkup**: Static markup rendering
- **Better error boundaries**: Hata sınırları iyileştirmeleri

**React 0.10 (2013)**:
- **React.isValidElement**: Element validation
- **React.DOM**: DOM element factory
- **Improved event system**: Olay sistemi iyileştirmeleri
- **Better performance**: Performans iyileştirmeleri

**React 0.11 (2013)**:
- **React.Children.only**: Single child validation
- **React.Children.count**: Children counting
- **Improved JSX transform**: JSX transform iyileştirmeleri
- **Better debugging tools**: Hata ayıklama araçları

**React 0.12 (2014)**:
- **React.Children.toArray**: Children to array conversion
- **Improved PropTypes**: PropTypes iyileştirmeleri
- **Better error messages**: Daha iyi hata mesajları
- **Performance improvements**: Performans iyileştirmeleri

**React 0.13 (2015)**:
- **ES6 Classes support**: ES6 sınıf desteği
- **React.createClass deprecation warning**: createClass kullanımı uyarısı
- **Improved performance**: Performans iyileştirmeleri
- **Better TypeScript support**: TypeScript desteği iyileştirmeleri

**React 0.14 (2015)**:
- Stateless functional components
- ReactDOM separation
- PropTypes sistemi

**React 15 (2016)**:
- Error boundaries
- React DevTools
- Fiber architecture (experimental)

**React 16 (2017)**:
- **Fiber Architecture**: Yeni reconciliation algoritması
- **Error Boundaries**: Hata yakalama ve yönetimi
- **Portals**: DOM tree dışında render etme
- **Fragments**: Wrapper element olmadan gruplama
- **Custom DOM Attributes**: Özel HTML özellikleri
- **setState with Function**: Fonksiyonel setState
- **ComponentDidCatch**: Hata yakalama lifecycle'ı

**React 16.3 (2018)**:
- **New Lifecycle Methods**: getDerivedStateFromProps, getSnapshotBeforeUpdate
- **Context API**: Stable context API
- **createRef**: Ref oluşturma API'si
- **forwardRef**: Ref forwarding

**React 16.6 (2018)**:
- **React.memo**: Functional component memoization
- **React.lazy**: Code splitting için lazy loading
- **Suspense**: Asenkron component loading
- **getDerivedStateFromError**: Error boundary için lifecycle

**React 16.8 (2019)**:
- **Hooks**: Functional components için state ve lifecycle
- **useState**: State yönetimi
- **useEffect**: Side effects yönetimi
- **useContext**: Context kullanımı
- **useReducer**: Complex state yönetimi
- **useCallback**: Function memoization
- **useMemo**: Value memoization
- **useRef**: Mutable ref object
- **useImperativeHandle**: Custom ref value
- **useLayoutEffect**: Synchronous effect
- **useDebugValue**: Custom hook debugging

**React 16.9 (2019)**:
- **React.Profiler**: Performance profiling
- **Async act**: Asenkron testing utilities
- **Deprecation Warnings**: UNSAFE_ lifecycle warnings

**React 16.13 (2020)**:
- **Concurrent Mode**: Experimental concurrent features
- **Suspense for Data Fetching**: Data fetching için Suspense

**React 17 (2020)**:
- **New JSX Transform**: Babel transform değişiklikleri
- **Event Delegation Changes**: Event handling optimizasyonları
- **Effect Cleanup Timing**: useEffect cleanup timing değişiklikleri

**React 18 (2022)**:
- **Concurrent Features**: Concurrent rendering
- **Automatic Batching**: Otomatik state batching
- **Suspense Improvements**: Gelişmiş Suspense desteği
- **useId**: Unique ID generation
- **useDeferredValue**: Deferred value updates
- **useTransition**: Transition state management
- **useSyncExternalStore**: External store synchronization
- **useInsertionEffect**: CSS-in-JS için effect
- **createRoot**: Yeni root API
- **hydrateRoot**: SSR hydration API

**React 19 (2024)**:
- **Server Components**: Sunucu tarafında render edilen component'ler
- **Actions API**: Form actions ve server actions
- **use() Hook**: Promise ve context için hook
- **useOptimistic**: Optimistic updates
- **useFormStatus**: Form durumu yönetimi
- **useFormState**: Form state yönetimi
- **Document Metadata**: Head yönetimi
- **Resource Preloading**: Kaynak ön yükleme
- **Custom Elements Support**: Web Components desteği
- **Improved DevTools**: Gelişmiş geliştirici araçları
- **Better TypeScript Support**: Gelişmiş TypeScript desteği
- **Performance Improvements**: Performans iyileştirmeleri

**React 19.1+ (2025)**:
- **Improved Concurrent Rendering**: Daha akıcı kullanıcı deneyimi
- **Automatic Hydration**: Otomatik hydration optimizasyonları
- **Enhanced Server Components**: Gelişmiş sunucu component'leri
- **React Compiler Integration**: React Compiler ile entegrasyon
- **WebAssembly Support**: WebAssembly ile performans optimizasyonu
- **Better Error Boundaries**: Gelişmiş hata sınırları
- **Improved Suspense**: Gelişmiş Suspense desteği
- **Enhanced DevTools**: Daha gelişmiş geliştirici araçları
- **Stricter TypeScript Integration**: Daha sıkı TypeScript entegrasyonu
- **Performance Monitoring**: Performans izleme araçları

- **Custom Hooks**: Özel hook'lar

**Performance Optimization**:
- **React.memo**: Component memoization
- **useMemo**: Expensive calculation memoization
- **useCallback**: Function memoization
- **Code Splitting**: Kod bölme
- **Lazy Loading**: Gecikmeli yükleme
- **Virtual Scrolling**: Büyük listeler için optimizasyon
- **Bundle Optimization**: Paket boyutu optimizasyonu

**Testing**:
- **Jest**: Test framework
- **React Testing Library**: Component testing
- **Enzyme**: Component testing (legacy)
- **Unit Testing**: Birim testleri
- **Integration Testing**: Entegrasyon testleri
- **Snapshot Testing**: Snapshot testleri

### 3.4 React İleri Seviye Konuları

**Advanced Patterns**:
- **Compound Components**: Bileşik component pattern'i
- **Render Props**: Render prop pattern'i
- **Higher-Order Components**: HOC pattern'i
- **Custom Hooks**: Özel hook pattern'leri
- **Context Pattern**: Context kullanım pattern'leri
- **Provider Pattern**: Provider pattern'i

**State Management Patterns**:
- **Flux Pattern**: Unidirectional data flow
- **Redux Pattern**: Predictable state updates
- **MobX Pattern**: Observable state
- **Zustand Pattern**: Simple state management
- **Jotai Pattern**: Atomic state management

**Performance Patterns**:
- **Memoization**: Memoization teknikleri
- **Virtualization**: Virtual scrolling
- **Code Splitting**: Route-based splitting
- **Bundle Analysis**: Bundle boyutu analizi
- **Lazy Loading**: Component lazy loading

**Testing Patterns**:
- **Test-Driven Development**: TDD yaklaşımı
- **Behavior-Driven Development**: BDD yaklaşımı
- **Mocking**: Mock teknikleri
- **Integration Testing**: Entegrasyon test stratejileri
- **E2E Testing**: End-to-end test stratejileri

### 3.5 React Kurulum ve Geliştirme Ortamı

#### 3.5.1 Create React App ile Kurulum

```bash
# Create React App ile yeni proje oluşturma
npx create-react-app my-react-app

# Proje dizinine git
cd my-react-app

# Geliştirme sunucusunu başlat
npm start

# Production build
npm run build

# Test çalıştır
npm test
```

**Proje Yapısı:**
```
my-react-app/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── manifest.json
├── src/
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   ├── services/
│   ├── utils/
│   ├── styles/
│   ├── App.js
│   ├── index.js
│   └── setupTests.js
├── package.json
└── README.md
```

#### 3.5.2 Vite ile Kurulum

```bash
# Vite ile React projesi oluşturma
npm create vite@latest my-react-app -- --template react

# Proje dizinine git
cd my-react-app

# Bağımlılıkları yükle
npm install

# Geliştirme sunucusunu başlat
npm run dev

# Production build
npm run build

# Preview production build
npm run preview
```

**Vite Konfigürasyonu:**
```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  }
})
```

#### 3.5.3 Webpack Konfigürasyonu

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.[contenthash].js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader'
        }
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new MiniCssExtractPlugin({
      filename: 'styles.[contenthash].css'
    })
  ],
  resolve: {
    extensions: ['.js', '.jsx']
  }
};
```

#### 3.5.4 Babel Konfigürasyonu

```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      targets: {
        browsers: ['> 1%', 'last 2 versions']
      }
    }],
    ['@babel/preset-react', {
      runtime: 'automatic'
    }]
  ],
  plugins: [
    '@babel/plugin-proposal-class-properties',
    '@babel/plugin-syntax-dynamic-import'
  ]
};
```

#### 3.5.5 ESLint ve Prettier Kurulumu

```bash
# ESLint ve Prettier kurulumu
npm install --save-dev eslint prettier eslint-config-prettier eslint-plugin-prettier
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

```javascript
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'prettier'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 12,
    sourceType: 'module'
  },
  plugins: [
    'react',
    '@typescript-eslint',
    'prettier'
  ],
  rules: {
    'prettier/prettier': 'error',
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off'
  },
  settings: {
    react: {
      version: 'detect'
    }
  }
};
```

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

### 3.6 Routing (React Router)

#### 3.6.1 React Router Kurulumu

```bash
# React Router kurulumu
npm install react-router-dom

# TypeScript desteği için
npm install @types/react-router-dom
```

#### 3.6.2 Temel Routing

```jsx
// App.js
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import Contact from './pages/Contact';
import UserProfile from './pages/UserProfile';

function App() {
  return (
    <Router>
      <div className="App">
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/contact">Contact</Link>
            </li>
          </ul>
        </nav>

        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
          <Route path="/user/:id" element={<UserProfile />} />
          <Route path="*" element={<div>404 - Page Not Found</div>} />
        </Routes>
      </div>
    </Router>
  );
}

export default App;
```

```jsx
// pages/Home.js
import React from 'react';
import { Link } from 'react-router-dom';

function Home() {
  return (
    <div>
      <h1>Home Page</h1>
      <p>Welcome to our website!</p>
      <Link to="/about">Learn more about us</Link>
    </div>
  );
}

export default Home;
```

```jsx
// pages/UserProfile.js
import React from 'react';
import { useParams, useNavigate } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  const navigate = useNavigate();

  const handleGoBack = () => {
    navigate(-1); // Geri git
  };

  const handleGoHome = () => {
    navigate('/'); // Ana sayfaya git
  };

  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {id}</p>
      <button onClick={handleGoBack}>Go Back</button>
      <button onClick={handleGoHome}>Go Home</button>
    </div>
  );
}

export default UserProfile;
```

#### 3.6.3 Nested Routes

```jsx
// App.js - Nested Routes
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link, Outlet } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/dashboard">Dashboard</Link>
        <Link to="/dashboard/profile">Profile</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </nav>
      <main>
        <Outlet />
      </main>
    </div>
  );
}

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet />
    </div>
  );
}

function Profile() {
  return <div>Profile Page</div>;
}

function Settings() {
  return <div>Settings Page</div>;
}

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<div>Home Page</div>} />
          <Route path="dashboard" element={<Dashboard />}>
            <Route path="profile" element={<Profile />} />
            <Route path="settings" element={<Settings />} />
          </Route>
        </Route>
      </Routes>
    </Router>
  );
}

export default App;
```

#### 3.6.4 Protected Routes

```jsx
// components/ProtectedRoute.js
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

export default ProtectedRoute;
```

```jsx
// hooks/useAuth.js
import { useState, useEffect } from 'react';

export function useAuth() {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Token kontrolü
    const token = localStorage.getItem('token');
    if (token) {
      // Token doğrulama
      validateToken(token);
    } else {
      setLoading(false);
    }
  }, []);

  const validateToken = async (token) => {
    try {
      const response = await fetch('/api/validate-token', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      });
      
      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
        setIsAuthenticated(true);
      } else {
        localStorage.removeItem('token');
      }
    } catch (error) {
      console.error('Token validation failed:', error);
      localStorage.removeItem('token');
    } finally {
      setLoading(false);
    }
  };

  const login = async (email, password) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, password })
      });

      if (response.ok) {
        const { token, user } = await response.json();
        localStorage.setItem('token', token);
        setUser(user);
        setIsAuthenticated(true);
        return true;
      }
      return false;
    } catch (error) {
      console.error('Login failed:', error);
      return false;
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
    setIsAuthenticated(false);
  };

  return {
    isAuthenticated,
    user,
    loading,
    login,
    logout
  };
}
```

```jsx
// App.js - Protected Routes kullanımı
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import ProtectedRoute from './components/ProtectedRoute';
import Login from './pages/Login';
import Dashboard from './pages/Dashboard';
import Profile from './pages/Profile';

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/dashboard" element={
          <ProtectedRoute>
            <Dashboard />
          </ProtectedRoute>
        } />
        <Route path="/profile" element={
          <ProtectedRoute>
            <Profile />
          </ProtectedRoute>
        } />
      </Routes>
    </Router>
  );
}

export default App;
```

### 3.7 State Management - Detaylı Analiz

State Management, React uygulamalarının en kritik konularından biridir. Bu konu, sadece veri yönetimi değil, aynı zamanda uygulama mimarisinin, performansının ve sürdürülebilirliğinin temelini oluşturur. Modern React uygulamalarında state management, component'ler arası veri akışını, global state yönetimini ve side effect'leri koordine eder.

**State Management'in Felsefesi:**

State Management, "single source of truth" prensibini benimser. Bu prensip, uygulamanın state'inin tek bir yerde tutulmasını ve tüm component'lerin bu state'ten veri almasını sağlar. Bu yaklaşım, veri tutarlılığını garanti eder ve debugging'i kolaylaştırır.

**State Management'in Kategorileri:**

**Local State Management (Yerel Durum Yönetimi):**
Local state, component'in kendi iç durumunu yönetir. Bu state, useState hook'u ile yönetilir ve sadece o component'e özeldir.

**Global State Management (Küresel Durum Yönetimi):**
Global state, uygulamanın genel durumunu yönetir. Bu state, Context API, Redux, Zustand gibi kütüphaneler ile yönetilir.

**Server State Management (Sunucu Durum Yönetimi):**
Server state, sunucudan gelen verilerin yönetimini sağlar. Bu state, React Query, SWR gibi kütüphaneler ile yönetilir.

**Form State Management (Form Durum Yönetimi):**
Form state, form verilerinin yönetimini sağlar. Bu state, React Hook Form, Formik gibi kütüphaneler ile yönetilir.

**State Management'in Temel Prensipleri:**

**Immutability (Değiştirilemezlik):**
State, değiştirilemez olmalıdır. Bu prensip, state güncellemelerinin öngörülebilir olmasını sağlar.

**Predictability (Öngörülebilirlik):**
State güncellemeleri, öngörülebilir olmalıdır. Bu prensip, debugging'i kolaylaştırır.

**Single Source of Truth (Tek Gerçek Kaynak):**
State, tek bir yerde tutulmalıdır. Bu prensip, veri tutarlılığını garanti eder.

**Unidirectional Data Flow (Tek Yönlü Veri Akışı):**
Veri akışı, tek yönlü olmalıdır. Bu prensip, veri akışını öngörülebilir hale getirir.

**State Management'in Avantajları:**

**Data Consistency:**
State management, veri tutarlılığını sağlar. Tüm component'ler, aynı state'ten veri alır.

**Predictable Updates:**
State management, öngörülebilir güncellemeler sağlar. State değişiklikleri, belirli pattern'lerde gerçekleşir.

**Debugging:**
State management, debugging'i kolaylaştırır. State değişiklikleri, takip edilebilir.

**Testing:**
State management, test edilebilirliği artırır. State logic'i, component'lerden ayrı test edilebilir.

**Performance:**
State management, performansı optimize eder. Gereksiz re-render'lar önlenir.

**State Management'in Kullanım Senaryoları:**

**Large Applications:**
State management, büyük uygulamalar için kritik öneme sahiptir. Bu uygulamalarda, state yönetimi karmaşık hale gelir.

**Complex Data Flow:**
State management, karmaşık veri akışı olan uygulamalar için idealdir. Bu uygulamalarda, component'ler arası veri iletişimi önemlidir.

**Real-time Applications:**
State management, gerçek zamanlı uygulamalar için idealdir. Bu uygulamalarda, state güncellemeleri sık gerçekleşir.

**Team Development:**
State management, takım geliştirme için idealdir. Bu ortamlarda, state yapısı standartlaştırılır.

#### 3.7.1 Redux Toolkit - Derinlemesine Analiz

Redux Toolkit, modern Redux geliştirmenin standart yaklaşımıdır. Bu kütüphane, Redux'un karmaşıklığını azaltır ve geliştirici deneyimini iyileştirir. Redux Toolkit, boilerplate kod'u azaltır ve best practice'leri otomatik olarak uygular.

```bash
# Redux Toolkit kurulumu
npm install @reduxjs/toolkit react-redux
```

```jsx
// store/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk for fetching users
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users');
      if (!response.ok) {
        throw new Error('Failed to fetch users');
      }
      const data = await response.json();
      return data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// Async thunk for creating user
export const createUser = createAsyncThunk(
  'users/createUser',
  async (userData, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(userData)
      });
      
      if (!response.ok) {
        throw new Error('Failed to create user');
      }
      
      const data = await response.json();
      return data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: false,
    error: null
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    updateUser: (state, action) => {
      const { id, updates } = action.payload;
      const user = state.users.find(user => user.id === id);
      if (user) {
        Object.assign(user, updates);
      }
    },
    deleteUser: (state, action) => {
      state.users = state.users.filter(user => user.id !== action.payload);
    }
  },
  extraReducers: (builder) => {
    builder
      // Fetch users
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      // Create user
      .addCase(createUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(createUser.fulfilled, (state, action) => {
        state.loading = false;
        state.users.push(action.payload);
      })
      .addCase(createUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { clearError, updateUser, deleteUser } = userSlice.actions;
export default userSlice.reducer;
```

```jsx
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    users: userReducer
  }
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```jsx
// App.js - Redux Provider
import React from 'react';
import { Provider } from 'react-redux';
import { store } from './store';
import UserList from './components/UserList';

function App() {
  return (
    <Provider store={store}>
      <div className="App">
        <UserList />
      </div>
    </Provider>
  );
}

export default App;
```

```jsx
// components/UserList.js
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchUsers, createUser, deleteUser } from '../store/userSlice';

function UserList() {
  const { users, loading, error } = useSelector((state) => state.users);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchUsers());
  }, [dispatch]);

  const handleCreateUser = () => {
    const newUser = {
      name: 'New User',
      email: 'newuser@example.com'
    };
    dispatch(createUser(newUser));
  };

  const handleDeleteUser = (id) => {
    dispatch(deleteUser(id));
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <button onClick={handleCreateUser}>Add User</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button onClick={() => handleDeleteUser(user.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

#### 3.7.2 Zustand

```bash
# Zustand kurulumu
npm install zustand
```

```jsx
// store/userStore.js
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

const useUserStore = create(
  devtools(
    (set, get) => ({
      users: [],
      loading: false,
      error: null,

      // Actions
      fetchUsers: async () => {
        set({ loading: true, error: null });
        try {
          const response = await fetch('/api/users');
          if (!response.ok) {
            throw new Error('Failed to fetch users');
          }
          const users = await response.json();
          set({ users, loading: false });
        } catch (error) {
          set({ error: error.message, loading: false });
        }
      },

      createUser: async (userData) => {
        set({ loading: true, error: null });
        try {
          const response = await fetch('/api/users', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(userData)
          });
          
          if (!response.ok) {
            throw new Error('Failed to create user');
          }
          
          const newUser = await response.json();
          set((state) => ({
            users: [...state.users, newUser],
            loading: false
          }));
        } catch (error) {
          set({ error: error.message, loading: false });
        }
      },

      updateUser: (id, updates) => {
        set((state) => ({
          users: state.users.map(user =>
            user.id === id ? { ...user, ...updates } : user
          )
        }));
      },

      deleteUser: (id) => {
        set((state) => ({
          users: state.users.filter(user => user.id !== id)
        }));
      },

      clearError: () => set({ error: null })
    }),
    {
      name: 'user-store'
    }
  )
);

export default useUserStore;
```

```jsx
// components/UserList.js - Zustand kullanımı
import React, { useEffect } from 'react';
import useUserStore from '../store/userStore';

function UserList() {
  const {
    users,
    loading,
    error,
    fetchUsers,
    createUser,
    deleteUser,
    clearError
  } = useUserStore();

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  const handleCreateUser = () => {
    const newUser = {
      name: 'New User',
      email: 'newuser@example.com'
    };
    createUser(newUser);
  };

  const handleDeleteUser = (id) => {
    deleteUser(id);
  };

  if (loading) return <div>Loading...</div>;
  if (error) {
    return (
      <div>
        <div>Error: {error}</div>
        <button onClick={clearError}>Clear Error</button>
      </div>
    );
  }

  return (
    <div>
      <button onClick={handleCreateUser}>Add User</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button onClick={() => handleDeleteUser(user.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

### 3.8 API ve Veri Yönetimi

#### 3.8.1 React Query (TanStack Query)

```bash
# React Query kurulumu
npm install @tanstack/react-query
```

```jsx
// App.js - QueryClient Provider
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import UserList from './components/UserList';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 3,
      refetchOnWindowFocus: false
    }
  }
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <UserList />
      </div>
    </QueryClientProvider>
  );
}

export default App;
```

```jsx
// hooks/useUsers.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// API functions
const fetchUsers = async () => {
  const response = await fetch('/api/users');
  if (!response.ok) {
    throw new Error('Failed to fetch users');
  }
  return response.json();
};

const createUser = async (userData) => {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(userData)
  });
  
  if (!response.ok) {
    throw new Error('Failed to create user');
  }
  
  return response.json();
};

const updateUser = async ({ id, updates }) => {
  const response = await fetch(`/api/users/${id}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(updates)
  });
  
  if (!response.ok) {
    throw new Error('Failed to update user');
  }
  
  return response.json();
};

const deleteUser = async (id) => {
  const response = await fetch(`/api/users/${id}`, {
    method: 'DELETE'
  });
  
  if (!response.ok) {
    throw new Error('Failed to delete user');
  }
  
  return id;
};

// Custom hooks
export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: updateUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: deleteUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
}
```

```jsx
// components/UserList.js - React Query kullanımı
import React from 'react';
import { useUsers, useCreateUser, useDeleteUser } from '../hooks/useUsers';

function UserList() {
  const { data: users, isLoading, error } = useUsers();
  const createUserMutation = useCreateUser();
  const deleteUserMutation = useDeleteUser();

  const handleCreateUser = () => {
    const newUser = {
      name: 'New User',
      email: 'newuser@example.com'
    };
    createUserMutation.mutate(newUser);
  };

  const handleDeleteUser = (id) => {
    deleteUserMutation.mutate(id);
  };

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button 
        onClick={handleCreateUser}
        disabled={createUserMutation.isPending}
      >
        {createUserMutation.isPending ? 'Creating...' : 'Add User'}
      </button>
      <ul>
        {users?.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button 
              onClick={() => handleDeleteUser(user.id)}
              disabled={deleteUserMutation.isPending}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

#### 3.8.2 SWR

```bash
# SWR kurulumu
npm install swr
```

```jsx
// hooks/useUsers.js - SWR
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

export function useUsers() {
  const { data, error, isLoading, mutate } = useSWR('/api/users', fetcher, {
    revalidateOnFocus: false,
    revalidateOnReconnect: true,
    dedupingInterval: 60000 // 1 minute
  });

  return {
    users: data,
    isLoading,
    isError: error,
    mutate
  };
}

export function useUser(id) {
  const { data, error, isLoading } = useSWR(
    id ? `/api/users/${id}` : null,
    fetcher
  );

  return {
    user: data,
    isLoading,
    isError: error
  };
}
```

```jsx
// components/UserList.js - SWR kullanımı
import React from 'react';
import { useUsers } from '../hooks/useUsers';

function UserList() {
  const { users, isLoading, isError, mutate } = useUsers();

  const handleRefresh = () => {
    mutate(); // Revalidate data
  };

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error loading users</div>;

  return (
    <div>
      <button onClick={handleRefresh}>Refresh</button>
      <ul>
        {users?.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

### 3.9 Styling ve CSS

#### 3.9.1 CSS Modules

```css
/* UserCard.module.css */
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 8px;
}

.content {
  color: #666;
  line-height: 1.5;
}

.button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 8px;
}

.button:hover {
  background-color: #0056b3;
}
```

```jsx
// components/UserCard.js - CSS Modules
import React from 'react';
import styles from './UserCard.module.css';

function UserCard({ user, onEdit, onDelete }) {
  return (
    <div className={styles.card}>
      <h3 className={styles.title}>{user.name}</h3>
      <p className={styles.content}>{user.email}</p>
      <button className={styles.button} onClick={() => onEdit(user.id)}>
        Edit
      </button>
      <button className={styles.button} onClick={() => onDelete(user.id)}>
        Delete
      </button>
    </div>
  );
}

export default UserCard;
```

#### 3.9.2 Styled Components

```bash
# Styled Components kurulumu
npm install styled-components
npm install --save-dev @types/styled-components
```

```jsx
// components/StyledUserCard.js
import React from 'react';
import styled from 'styled-components';

const Card = styled.div`
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;

  &:hover {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  }
`;

const Title = styled.h3`
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 8px;
`;

const Content = styled.p`
  color: #666;
  line-height: 1.5;
`;

const Button = styled.button`
  background-color: ${props => props.variant === 'danger' ? '#dc3545' : '#007bff'};
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 8px;
  margin-right: 8px;

  &:hover {
    background-color: ${props => props.variant === 'danger' ? '#c82333' : '#0056b3'};
  }

  &:disabled {
    background-color: #6c757d;
    cursor: not-allowed;
  }
`;

function StyledUserCard({ user, onEdit, onDelete }) {
  return (
    <Card>
      <Title>{user.name}</Title>
      <Content>{user.email}</Content>
      <Button onClick={() => onEdit(user.id)}>
        Edit
      </Button>
      <Button variant="danger" onClick={() => onDelete(user.id)}>
        Delete
      </Button>
    </Card>
  );
}

export default StyledUserCard;
```

#### 3.9.3 Tailwind CSS

```bash
# Tailwind CSS kurulumu
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```css
/* tailwind.config.js */
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```jsx
// components/TailwindUserCard.js
import React from 'react';

function TailwindUserCard({ user, onEdit, onDelete }) {
  return (
    <div className="border border-gray-300 rounded-lg p-4 m-2 shadow-md hover:shadow-lg transition-shadow">
      <h3 className="text-lg font-bold text-gray-800 mb-2">{user.name}</h3>
      <p className="text-gray-600 leading-relaxed">{user.email}</p>
      <div className="mt-4 space-x-2">
        <button 
          className="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded transition-colors"
          onClick={() => onEdit(user.id)}
        >
          Edit
        </button>
        <button 
          className="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded transition-colors"
          onClick={() => onDelete(user.id)}
        >
          Delete
        </button>
      </div>
    </div>
  );
}

export default TailwindUserCard;
```

### 3.10 Performance Optimizasyonu - Detaylı Analiz

Performance Optimization, React uygulamalarının en kritik konularından biridir. Bu konu, sadece hız optimizasyonu değil, aynı zamanda kullanıcı deneyiminin, ölçeklenebilirliğin ve sürdürülebilirliğin temelini oluşturur. Modern React uygulamalarında performance optimization, render optimizasyonu, memory management ve bundle optimization gibi konuları kapsar.

**Performance Optimization'un Felsefesi:**

Performance Optimization, "measure first, optimize second" prensibini benimser. Bu prensip, önce performans sorunlarının tespit edilmesini, sonra optimize edilmesini sağlar. Bu yaklaşım, premature optimization'ı önler ve gerçek performans sorunlarına odaklanır.

**Performance Optimization'un Kategorileri:**

**Render Optimization (Render Optimizasyonu):**
Render optimization, component'lerin gereksiz re-render'larını önler. Bu optimizasyon, React.memo, useMemo, useCallback gibi teknikler ile sağlanır.

**Memory Management (Bellek Yönetimi):**
Memory management, memory leak'lerini önler ve memory kullanımını optimize eder. Bu yönetim, cleanup function'ları ve proper dependency management ile sağlanır.

**Bundle Optimization (Paket Optimizasyonu):**
Bundle optimization, JavaScript bundle'ının boyutunu azaltır. Bu optimizasyon, code splitting, tree shaking ve lazy loading ile sağlanır.

**Network Optimization (Ağ Optimizasyonu):**
Network optimization, network request'lerini optimize eder. Bu optimizasyon, caching, prefetching ve request batching ile sağlanır.

**Performance Optimization'un Temel Prensipleri:**

**Measure First:**
Performans optimizasyonu, önce ölçüm yapılmasını gerektirir. Bu prensip, gerçek performans sorunlarının tespit edilmesini sağlar.

**Optimize Critical Path:**
Performans optimizasyonu, critical path'e odaklanır. Bu prensip, en önemli performans sorunlarının çözülmesini sağlar.

**Avoid Premature Optimization:**
Performans optimizasyonu, premature optimization'ı önler. Bu prensip, gereksiz optimizasyonları önler.

**Profile and Monitor:**
Performans optimizasyonu, sürekli profiling ve monitoring gerektirir. Bu prensip, performans regresyonlarının tespit edilmesini sağlar.

**Performance Optimization'un Avantajları:**

**Better User Experience:**
Performance optimization, daha iyi kullanıcı deneyimi sağlar. Hızlı uygulamalar, kullanıcı memnuniyetini artırır.

**Reduced Bounce Rate:**
Performance optimization, bounce rate'i azaltır. Hızlı uygulamalar, kullanıcıların sitede kalma süresini artırır.

**Better SEO:**
Performance optimization, SEO'yu iyileştirir. Google, hızlı siteleri önceliklendirir.

**Lower Server Costs:**
Performance optimization, server maliyetlerini azaltır. Optimize edilmiş uygulamalar, daha az server kaynağı kullanır.

**Better Scalability:**
Performance optimization, ölçeklenebilirliği artırır. Optimize edilmiş uygulamalar, daha fazla kullanıcıyı destekler.

**Performance Optimization'un Kullanım Senaryoları:**

**Large Applications:**
Performance optimization, büyük uygulamalar için kritik öneme sahiptir. Bu uygulamalarda, performans sorunları daha belirgin olur.

**High Traffic Applications:**
Performance optimization, yüksek trafikli uygulamalar için idealdir. Bu uygulamalarda, performans optimizasyonu kritik öneme sahiptir.

**Mobile Applications:**
Performance optimization, mobil uygulamalar için idealdir. Mobil cihazlarda, performans optimizasyonu daha önemlidir.

**Real-time Applications:**
Performance optimization, gerçek zamanlı uygulamalar için idealdir. Bu uygulamalarda, düşük latency kritik öneme sahiptir.

#### 3.10.1 React.memo - Derinlemesine Analiz

React.memo, React'in en önemli performans optimizasyon araçlarından biridir. Bu higher-order component, component'lerin gereksiz re-render'larını önler ve performansı artırır. React.memo, shallow comparison yapar ve props değişmediği sürece component'i re-render etmez.

```jsx
// components/UserCard.js - React.memo ile optimizasyon
import React, { memo } from 'react';

const UserCard = memo(({ user, onEdit, onDelete }) => {
  console.log('UserCard rendered for:', user.name);
  
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Edit</button>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
});

// Custom comparison function
const UserCardWithCustomComparison = memo(
  ({ user, onEdit, onDelete }) => {
    return (
      <div className="user-card">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
        <button onClick={() => onEdit(user.id)}>Edit</button>
        <button onClick={() => onDelete(user.id)}>Delete</button>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Sadece user objesi değiştiğinde re-render et
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name &&
      prevProps.user.email === nextProps.user.email
    );
  }
);

export default UserCard;
```

#### 3.10.2 useMemo ve useCallback

```jsx
// components/UserList.js - useMemo ve useCallback optimizasyonu
import React, { useState, useMemo, useCallback } from 'react';

function UserList({ users, onEdit, onDelete }) {
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');

  // useMemo ile expensive calculation'ı memoize et
  const filteredAndSortedUsers = useMemo(() => {
    console.log('Filtering and sorting users...');
    
    let filtered = users.filter(user =>
      user.name.toLowerCase().includes(filter.toLowerCase()) ||
      user.email.toLowerCase().includes(filter.toLowerCase())
    );

    return filtered.sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else if (sortBy === 'email') {
        return a.email.localeCompare(b.email);
      }
      return 0;
    });
  }, [users, filter, sortBy]);

  // useCallback ile function'ları memoize et
  const handleEdit = useCallback((id) => {
    onEdit(id);
  }, [onEdit]);

  const handleDelete = useCallback((id) => {
    onDelete(id);
  }, [onDelete]);

  // useMemo ile expensive component'i memoize et
  const userStats = useMemo(() => {
    console.log('Calculating user stats...');
    
    return {
      total: users.length,
      filtered: filteredAndSortedUsers.length,
      averageAge: users.reduce((sum, user) => sum + (user.age || 0), 0) / users.length
    };
  }, [users, filteredAndSortedUsers]);

  return (
    <div>
      <div className="filters">
        <input
          type="text"
          placeholder="Filter users..."
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
        />
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="email">Sort by Email</option>
        </select>
      </div>

      <div className="stats">
        <p>Total: {userStats.total}</p>
        <p>Filtered: {userStats.filtered}</p>
        <p>Average Age: {userStats.averageAge.toFixed(1)}</p>
      </div>

      <div className="user-list">
        {filteredAndSortedUsers.map(user => (
          <UserCard
            key={user.id}
            user={user}
            onEdit={handleEdit}
            onDelete={handleDelete}
          />
        ))}
      </div>
    </div>
  );
}

export default UserList;
```

#### 3.10.3 Code Splitting ve Lazy Loading

```jsx
// App.js - Code Splitting
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

// Lazy load components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));
const UserProfile = lazy(() => import('./pages/UserProfile'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

// Loading component
const LoadingSpinner = () => (
  <div className="loading-spinner">
    <div className="spinner"></div>
    <p>Loading...</p>
  </div>
);

function App() {
  return (
    <Router>
      <div className="App">
        <Suspense fallback={<LoadingSpinner />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/contact" element={<Contact />} />
            <Route path="/user/:id" element={<UserProfile />} />
            <Route path="/dashboard" element={<Dashboard />} />
          </Routes>
        </Suspense>
      </div>
    </Router>
  );
}

export default App;
```

```jsx
// components/LazyComponent.js - Dynamic import
import React, { useState, lazy, Suspense } from 'react';

// Lazy load component
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function LazyComponent() {
  const [showHeavy, setShowHeavy] = useState(false);

  return (
    <div>
      <button onClick={() => setShowHeavy(true)}>
        Load Heavy Component
      </button>
      
      {showHeavy && (
        <Suspense fallback={<div>Loading heavy component...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}

export default LazyComponent;
```

#### 3.10.4 Virtual Scrolling

```bash
# React Window kurulumu
npm install react-window react-window-infinite-loader
```

```jsx
// components/VirtualizedUserList.js
import React, { useMemo } from 'react';
import { FixedSizeList as List } from 'react-window';

function VirtualizedUserList({ users }) {
  // Row component
  const UserRow = ({ index, style }) => {
    const user = users[index];
    
    return (
      <div style={style} className="user-row">
        <div className="user-info">
          <h4>{user.name}</h4>
          <p>{user.email}</p>
        </div>
        <div className="user-actions">
          <button>Edit</button>
          <button>Delete</button>
        </div>
      </div>
    );
  };

  return (
    <List
      height={600} // Container height
      itemCount={users.length} // Total number of items
      itemSize={80} // Height of each item
      width="100%"
    >
      {UserRow}
    </List>
  );
}

export default VirtualizedUserList;
```

```jsx
// components/InfiniteVirtualizedList.js
import React, { useState, useEffect } from 'react';
import { FixedSizeList as List } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';

function InfiniteVirtualizedList() {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);

  // Simulate API call
  const loadMoreItems = async (startIndex, stopIndex) => {
    setLoading(true);
    
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    const newItems = Array.from({ length: 20 }, (_, index) => ({
      id: startIndex + index,
      name: `User ${startIndex + index}`,
      email: `user${startIndex + index}@example.com`
    }));
    
    setItems(prevItems => [...prevItems, ...newItems]);
    setLoading(false);
  };

  const isItemLoaded = (index) => !!items[index];

  const Item = ({ index, style }) => {
    const item = items[index];
    
    if (!item) {
      return (
        <div style={style} className="loading-row">
          Loading...
        </div>
      );
    }

    return (
      <div style={style} className="user-row">
        <h4>{item.name}</h4>
        <p>{item.email}</p>
      </div>
    );
  };

  return (
    <InfiniteLoader
      isItemLoaded={isItemLoaded}
      itemCount={1000} // Total expected items
      loadMoreItems={loadMoreItems}
    >
      {({ onItemsRendered, ref }) => (
        <List
          ref={ref}
          height={600}
          itemCount={items.length}
          itemSize={80}
          onItemsRendered={onItemsRendered}
        >
          {Item}
        </List>
      )}
    </InfiniteLoader>
  );
}

export default InfiniteVirtualizedList;
```

### 3.11 Testing - Detaylı Analiz

Testing, React uygulamalarının en kritik konularından biridir. Bu konu, sadece hata tespiti değil, aynı zamanda kod kalitesinin, güvenilirliğin ve sürdürülebilirliğin temelini oluşturur. Modern React uygulamalarında testing, unit testing, integration testing, end-to-end testing ve visual regression testing gibi konuları kapsar.

**Testing'in Felsefesi:**

Testing, "test-driven development" prensibini benimser. Bu prensip, test'lerin kod yazılmadan önce yazılmasını ve kod'un test'leri geçecek şekilde yazılmasını sağlar. Bu yaklaşım, kod kalitesini artırır ve hata oranını azaltır.

**Testing'in Kategorileri:**

**Unit Testing (Birim Testleri):**
Unit testing, individual function'ların ve component'lerin test edilmesini sağlar. Bu test'ler, küçük parçaların doğru çalıştığını garanti eder.

**Integration Testing (Entegrasyon Testleri):**
Integration testing, farklı component'lerin birlikte çalışmasını test eder. Bu test'ler, component'ler arası etkileşimlerin doğru çalıştığını garanti eder.

**End-to-End Testing (Uçtan Uca Testler):**
End-to-end testing, tüm uygulamanın kullanıcı perspektifinden test edilmesini sağlar. Bu test'ler, gerçek kullanıcı senaryolarını simüle eder.

**Visual Regression Testing (Görsel Gerileme Testleri):**
Visual regression testing, UI'ın görsel değişikliklerini test eder. Bu test'ler, beklenmeyen görsel değişiklikleri tespit eder.

**Testing'in Temel Prensipleri:**

**Test Pyramid:**
Testing, test pyramid prensibini benimser. Bu prensip, çok sayıda unit test, orta sayıda integration test ve az sayıda end-to-end test yazılmasını sağlar.

**AAA Pattern:**
Testing, Arrange-Act-Assert pattern'ini benimser. Bu pattern, test'lerin yapılandırılmasını, eylemin gerçekleştirilmesini ve sonucun doğrulanmasını sağlar.

**Test Isolation:**
Testing, test isolation prensibini benimser. Bu prensip, test'lerin birbirinden bağımsız olmasını sağlar.

**Test Coverage:**
Testing, test coverage prensibini benimser. Bu prensip, kod'un ne kadarının test edildiğini ölçer.

**Testing'in Avantajları:**

**Bug Prevention:**
Testing, bug'ların önlenmesini sağlar. Test'ler, hataları erken tespit eder.

**Code Quality:**
Testing, kod kalitesini artırır. Test edilebilir kod, genellikle daha iyi tasarlanmış kod'dur.

**Refactoring Safety:**
Testing, refactoring güvenliğini sağlar. Test'ler, refactoring sonrası kod'un hala çalıştığını garanti eder.

**Documentation:**
Testing, kod dokümantasyonu sağlar. Test'ler, kod'un nasıl kullanılacağını gösterir.

**Confidence:**
Testing, geliştirici güvenini artırır. Test'ler, kod'un doğru çalıştığından emin olmayı sağlar.

**Testing'in Kullanım Senaryoları:**

**Large Applications:**
Testing, büyük uygulamalar için kritik öneme sahiptir. Bu uygulamalarda, hata tespiti daha zor olur.

**Team Development:**
Testing, takım geliştirme için idealdir. Test'ler, farklı geliştiricilerin kod'unu doğrular.

**Critical Applications:**
Testing, kritik uygulamalar için idealdir. Bu uygulamalarda, hata toleransı düşüktür.

**Long-term Projects:**
Testing, uzun vadeli projeler için idealdir. Test'ler, kod'un sürdürülebilirliğini sağlar.

#### 3.11.1 Jest - Derinlemesine Analiz

Jest, React uygulamalarının en popüler testing framework'üdür. Bu framework, unit testing, integration testing ve mocking gibi özellikler sağlar. Jest, Facebook tarafından geliştirilmiş ve React ekosisteminin standart testing aracıdır. ve React Testing Library

```bash
# Testing kütüphaneleri kurulumu
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event
npm install --save-dev jest-environment-jsdom
```

```jsx
// components/Button.js
import React from 'react';

function Button({ children, onClick, disabled = false, variant = 'primary' }) {
  return (
    <button
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

export default Button;
```

```jsx
// components/Button.test.js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';
import Button from './Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    
    const button = screen.getByRole('button', { name: /click me/i });
    expect(button).toBeInTheDocument();
  });

  test('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  test('applies correct variant class', () => {
    render(<Button variant="secondary">Click me</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('btn-secondary');
  });

  test('does not call onClick when disabled', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick} disabled>Click me</Button>);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

```jsx
// components/UserForm.js
import React, { useState } from 'react';

function UserForm({ onSubmit, initialData = {} }) {
  const [formData, setFormData] = useState({
    name: initialData.name || '',
    email: initialData.email || '',
    age: initialData.age || ''
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
    
    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.age || formData.age < 0) {
      newErrors.age = 'Age must be a positive number';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (validateForm()) {
      onSubmit(formData);
    }
  };

  return (
    <form onSubmit={handleSubmit} data-testid="user-form">
      <div>
        <label htmlFor="name">Name:</label>
        <input
          id="name"
          name="name"
          type="text"
          value={formData.name}
          onChange={handleChange}
          data-testid="name-input"
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>

      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          data-testid="email-input"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <label htmlFor="age">Age:</label>
        <input
          id="age"
          name="age"
          type="number"
          value={formData.age}
          onChange={handleChange}
          data-testid="age-input"
        />
        {errors.age && <span className="error">{errors.age}</span>}
      </div>

      <button type="submit" data-testid="submit-button">
        Submit
      </button>
    </form>
  );
}

export default UserForm;
```

```jsx
// components/UserForm.test.js
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';
import UserForm from './UserForm';

describe('UserForm Component', () => {
  test('renders form fields', () => {
    render(<UserForm onSubmit={jest.fn()} />);
    
    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/age/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  test('submits form with valid data', async () => {
    const handleSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<UserForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByTestId('name-input'), 'John Doe');
    await user.type(screen.getByTestId('email-input'), 'john@example.com');
    await user.type(screen.getByTestId('age-input'), '30');
    
    await user.click(screen.getByTestId('submit-button'));
    
    expect(handleSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com',
      age: '30'
    });
  });

  test('shows validation errors for empty fields', async () => {
    const user = userEvent.setup();
    
    render(<UserForm onSubmit={jest.fn()} />);
    
    await user.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(screen.getByText('Name is required')).toBeInTheDocument();
      expect(screen.getByText('Email is required')).toBeInTheDocument();
      expect(screen.getByText('Age must be a positive number')).toBeInTheDocument();
    });
  });

  test('shows error for invalid email', async () => {
    const user = userEvent.setup();
    
    render(<UserForm onSubmit={jest.fn()} />);
    
    await user.type(screen.getByTestId('email-input'), 'invalid-email');
    await user.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(screen.getByText('Email is invalid')).toBeInTheDocument();
    });
  });

  test('clears error when user starts typing', async () => {
    const user = userEvent.setup();
    
    render(<UserForm onSubmit={jest.fn()} />);
    
    // Submit empty form to show errors
    await user.click(screen.getByTestId('submit-button'));
    
    await waitFor(() => {
      expect(screen.getByText('Name is required')).toBeInTheDocument();
    });
    
    // Start typing in name field
    await user.type(screen.getByTestId('name-input'), 'J');
    
    await waitFor(() => {
      expect(screen.queryByText('Name is required')).not.toBeInTheDocument();
    });
  });

  test('populates form with initial data', () => {
    const initialData = {
      name: 'Jane Doe',
      email: 'jane@example.com',
      age: '25'
    };
    
    render(<UserForm onSubmit={jest.fn()} initialData={initialData} />);
    
    expect(screen.getByTestId('name-input')).toHaveValue('Jane Doe');
    expect(screen.getByTestId('email-input')).toHaveValue('jane@example.com');
    expect(screen.getByTestId('age-input')).toHaveValue('25');
  });
});
```

#### 3.11.2 Integration Testing

```jsx
// components/UserList.test.js - Integration test
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';
import UserList from './UserList';

// Mock API
const mockUsers = [
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
];

// Mock fetch
global.fetch = jest.fn(() =>
  Promise.resolve({
    ok: true,
    json: () => Promise.resolve(mockUsers)
  })
);

describe('UserList Integration Tests', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  test('loads and displays users', async () => {
    render(<UserList />);
    
    // Wait for users to load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('Jane Smith')).toBeInTheDocument();
    });
    
    expect(fetch).toHaveBeenCalledWith('/api/users');
  });

  test('filters users by name', async () => {
    const user = userEvent.setup();
    
    render(<UserList />);
    
    // Wait for users to load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    // Filter by name
    const filterInput = screen.getByPlaceholderText(/filter users/i);
    await user.type(filterInput, 'John');
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.queryByText('Jane Smith')).not.toBeInTheDocument();
  });

  test('sorts users by name', async () => {
    const user = userEvent.setup();
    
    render(<UserList />);
    
    // Wait for users to load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    // Sort by name
    const sortSelect = screen.getByDisplayValue(/sort by name/i);
    await user.selectOptions(sortSelect, 'name');
    
    const userRows = screen.getAllByTestId(/user-row/i);
    expect(userRows[0]).toHaveTextContent('Jane Smith'); // Jane comes before John alphabetically
    expect(userRows[1]).toHaveTextContent('John Doe');
  });
});
```

### 3.12 Build ve Deployment

#### 3.12.1 Production Build

```bash
# Production build
npm run build

# Build analizi
npm install --save-dev webpack-bundle-analyzer
npm run build -- --analyze
```

```javascript
// webpack.config.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'js/[name].[contenthash].js',
    chunkFilename: 'js/[name].[contenthash].chunk.js',
    clean: true,
    publicPath: '/'
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // Remove console.log in production
            drop_debugger: true
          }
        }
      }),
      new CssMinimizerPlugin()
    ],
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', { modules: false }],
              ['@babel/preset-react', { runtime: 'automatic' }]
            ]
          }
        }
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'images/[name].[contenthash][ext]'
        }
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true
      }
    }),
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash].css',
      chunkFilename: 'css/[name].[contenthash].chunk.css'
    })
  ]
};
```

#### 3.12.2 Environment Variables

```bash
# .env.development
REACT_APP_API_URL=http://localhost:3001/api
REACT_APP_DEBUG=true
REACT_APP_VERSION=1.0.0
```

```bash
# .env.production
REACT_APP_API_URL=https://api.myapp.com
REACT_APP_DEBUG=false
REACT_APP_VERSION=1.0.0
```

```jsx
// config/api.js
const config = {
  apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3001/api',
  debug: process.env.REACT_APP_DEBUG === 'true',
  version: process.env.REACT_APP_VERSION || '1.0.0'
};

export default config;
```

#### 3.12.3 Docker Deployment

```dockerfile
# Dockerfile
# Multi-stage build
FROM node:18-alpine as build

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the app
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built app to nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # Gzip compression
        gzip on;
        gzip_vary on;
        gzip_min_length 1024;
        gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Handle client-side routing
        location / {
            try_files $uri $uri/ /index.html;
        }

        # API proxy
        location /api/ {
            proxy_pass http://backend:3001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build: .
    ports:
      - "80:80"
    depends_on:
      - backend
    environment:
      - REACT_APP_API_URL=http://localhost/api

  backend:
    image: myapp-backend:latest
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/myapp

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 3.13 Advanced Patterns

#### 3.13.1 Higher-Order Components (HOC)

```jsx
// hoc/withAuth.js
import React from 'react';
import { useAuth } from '../hooks/useAuth';

function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated, loading } = useAuth();

    if (loading) {
      return <div>Loading...</div>;
    }

    if (!isAuthenticated) {
      return <div>Please log in to access this page.</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

export default withAuth;
```

```jsx
// hoc/withLoading.js
import React from 'react';

function withLoading(WrappedComponent) {
  return function LoadingComponent({ loading, ...props }) {
    if (loading) {
      return (
        <div className="loading-container">
          <div className="spinner"></div>
          <p>Loading...</p>
        </div>
      );
    }

    return <WrappedComponent {...props} />;
  };
}

export default withLoading;
```

```jsx
// components/UserProfile.js - HOC kullanımı
import React from 'react';
import withAuth from '../hoc/withAuth';
import withLoading from '../hoc/withLoading';

function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// HOC'ları compose et
export default withAuth(withLoading(UserProfile));
```

#### 3.13.2 Render Props Pattern

```jsx
// components/DataProvider.js
import React, { useState, useEffect } from 'react';

function DataProvider({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return children({ data, loading, error });
}

export default DataProvider;
```

```jsx
// components/UserList.js - Render Props kullanımı
import React from 'react';
import DataProvider from './DataProvider';

function UserList() {
  return (
    <DataProvider url="/api/users">
      {({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error}</div>;
        if (!data) return <div>No data</div>;

        return (
          <ul>
            {data.map(user => (
              <li key={user.id}>
                {user.name} - {user.email}
              </li>
            ))}
          </ul>
        );
      }}
    </DataProvider>
  );
}

export default UserList;
```

#### 3.13.3 Compound Components

```jsx
// components/Modal.js
import React, { createContext, useContext, useState } from 'react';

const ModalContext = createContext();

function Modal({ children, isOpen, onClose }) {
  const [isVisible, setIsVisible] = useState(isOpen);

  const handleClose = () => {
    setIsVisible(false);
    onClose?.();
  };

  if (!isVisible) return null;

  return (
    <ModalContext.Provider value={{ handleClose }}>
      <div className="modal-overlay" onClick={handleClose}>
        <div className="modal-content" onClick={(e) => e.stopPropagation()}>
          {children}
        </div>
      </div>
    </ModalContext.Provider>
  );
}

function ModalHeader({ children }) {
  const { handleClose } = useContext(ModalContext);
  
  return (
    <div className="modal-header">
      <h2>{children}</h2>
      <button onClick={handleClose} className="close-button">×</button>
    </div>
  );
}

function ModalBody({ children }) {
  return <div className="modal-body">{children}</div>;
}

function ModalFooter({ children }) {
  return <div className="modal-footer">{children}</div>;
}

// Compound component
Modal.Header = ModalHeader;
Modal.Body = ModalBody;
Modal.Footer = ModalFooter;

export default Modal;
```

```jsx
// components/UserModal.js - Compound Component kullanımı
import React, { useState } from 'react';
import Modal from './Modal';
import UserForm from './UserForm';

function UserModal({ isOpen, onClose, user, onSubmit }) {
  return (
    <Modal isOpen={isOpen} onClose={onClose}>
      <Modal.Header>
        {user ? 'Edit User' : 'Add New User'}
      </Modal.Header>
      
      <Modal.Body>
        <UserForm
          initialData={user}
          onSubmit={onSubmit}
        />
      </Modal.Body>
      
      <Modal.Footer>
        <button onClick={onClose}>Cancel</button>
      </Modal.Footer>
    </Modal>
  );
}

export default UserModal;
```

### 3.14 Error Handling ve Debugging

#### 3.14.1 Error Boundaries

```jsx
// components/ErrorBoundary.js
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error: error,
      errorInfo: errorInfo
    });

    // Log error to monitoring service
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Send to error reporting service
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <div className="error-boundary">
          <h2>Something went wrong.</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

```jsx
// App.js - Error Boundary kullanımı
import React from 'react';
import ErrorBoundary from './components/ErrorBoundary';
import UserList from './components/UserList';

function App() {
  const handleError = (error, errorInfo) => {
    // Send to error reporting service
    console.error('Application error:', error, errorInfo);
  };

  return (
    <div className="App">
      <ErrorBoundary onError={handleError}>
        <UserList />
      </ErrorBoundary>
    </div>
  );
}

export default App;
```

#### 3.14.2 React DevTools

```jsx
// utils/devTools.js
import { useEffect } from 'react';

// Development-only debugging utilities
export const useDevTools = (componentName, props, state) => {
  useEffect(() => {
    if (process.env.NODE_ENV === 'development') {
      console.group(`🔍 ${componentName} Debug Info`);
      console.log('Props:', props);
      console.log('State:', state);
      console.groupEnd();
    }
  }, [componentName, props, state]);
};

// Performance monitoring
export const measurePerformance = (componentName, renderFn) => {
  if (process.env.NODE_ENV === 'development') {
    const start = performance.now();
    const result = renderFn();
    const end = performance.now();
    
    console.log(`${componentName} render time: ${end - start}ms`);
    return result;
  }
  
  return renderFn();
};

// Memory usage monitoring
export const logMemoryUsage = (label) => {
  if (process.env.NODE_ENV === 'development' && performance.memory) {
    const memory = performance.memory;
    console.log(`${label} Memory Usage:`, {
      used: `${Math.round(memory.usedJSHeapSize / 1048576)} MB`,
      total: `${Math.round(memory.totalJSHeapSize / 1048576)} MB`,
      limit: `${Math.round(memory.jsHeapSizeLimit / 1048576)} MB`
    });
  }
};
```

```jsx
// components/UserList.js - DevTools kullanımı
import React, { useState, useEffect } from 'react';
import { useDevTools, measurePerformance, logMemoryUsage } from '../utils/devTools';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  // DevTools debugging
  useDevTools('UserList', { users, loading }, { users, loading });

  useEffect(() => {
    logMemoryUsage('UserList Mount');
    
    const fetchUsers = async () => {
      try {
        const response = await fetch('/api/users');
        const data = await response.json();
        setUsers(data);
      } catch (error) {
        console.error('Error fetching users:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  const renderUsers = () => {
    return measurePerformance('UserList Render', () => (
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    ));
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Users</h1>
      {renderUsers()}
    </div>
  );
}

export default UserList;
```

#### 3.14.3 Custom Error Handling Hook

```jsx
// hooks/useErrorHandler.js
import { useState, useCallback } from 'react';

export function useErrorHandler() {
  const [error, setError] = useState(null);

  const handleError = useCallback((error) => {
    console.error('Error handled:', error);
    setError(error);
    
    // Send to error reporting service
    if (process.env.NODE_ENV === 'production') {
      // Send to Sentry, LogRocket, etc.
      console.log('Sending error to monitoring service:', error);
    }
  }, []);

  const clearError = useCallback(() => {
    setError(null);
  }, []);

  return { error, handleError, clearError };
}
```

```jsx
// components/ErrorDisplay.js
import React from 'react';
import { useErrorHandler } from '../hooks/useErrorHandler';

function ErrorDisplay() {
  const { error, clearError } = useErrorHandler();

  if (!error) return null;

  return (
    <div className="error-display">
      <h3>An error occurred:</h3>
      <p>{error.message}</p>
      <button onClick={clearError}>Dismiss</button>
    </div>
  );
}

export default ErrorDisplay;
```
