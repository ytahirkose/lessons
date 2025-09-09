## BÖLÜM 4: REACT NATIVE - MOBİL UYGULAMA GELİŞTİRME

# React Native Dokümantasyonu - react-native@0.81

Bu bölüm, React Native ile çalışmak için detaylı dokümantasyon sağlar. React Native yolculuğunuzun başlangıcına hoş geldiniz! Başlangıç talimatları arıyorsanız, bunlar kendi bölümlerine taşındı. Dokümantasyon, Native Components, React ve daha fazlasına giriş için okumaya devam edin!

## Giriş

React Native'i birçok farklı türde insan kullanır: gelişmiş iOS geliştiricilerinden React yeni başlayanlarına, kariyerlerinde ilk kez programlama yapmaya başlayan insanlara kadar. Bu dokümantasyon, deneyim seviyesi veya geçmişi ne olursa olsun tüm öğrenenler için yazılmıştır.

### Bu dokümantasyonu nasıl kullanabilirsiniz

Buradan başlayabilir ve bu dokümantasyonu bir kitap gibi doğrusal olarak okuyabilirsiniz; veya ihtiyacınız olan belirli bölümleri okuyabilirsiniz. React'e zaten aşina mısınız? O bölümü atlayabilirsiniz—veya hafif bir hatırlatma için okuyabilirsiniz.

### Önkoşullar

React Native ile çalışmak için JavaScript temellerini anlamanız gerekecek. JavaScript'e yeni başlıyorsanız veya hatırlamaya ihtiyacınız varsa, Mozilla Developer Network'te dalabilir veya tazeleyebilirsiniz.

> **Bilgi**: React, Android veya iOS geliştirme konusunda önceden bilgi sahibi olmadığınızı varsaymaya çalışsak da, bunlar hevesli React Native geliştiricisi için değerli çalışma konularıdır. Mantıklı olduğu yerlerde, daha derinlemesine giden kaynaklara ve makalelere bağlantılar verdik.

### İnteraktif örnekler

Bu giriş, tarayıcınızda hemen başlamanızı sağlar ve şu gibi interaktif örneklerle:

```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Try editing me!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  text: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});

export default App;
```

Yukarıdaki bir Snack Player'dır. Expo tarafından oluşturulan, React Native projelerini gömme ve çalıştırma ve bunların Android ve iOS gibi platformlarda nasıl render edildiğini paylaşma için kullanışlı bir araçtır. Kod canlı ve düzenlenebilir, böylece doğrudan tarayıcınızda oynayabilirsiniz. Yukarıdaki "Try editing me!" metnini "Hello, world!" olarak değiştirmeyi deneyin!

> **İpucu**: İsteğe bağlı olarak, yerel bir geliştirme ortamı kurmak istiyorsanız, ortamınızı yerel makinenizde kurma rehberimizi takip edebilir ve kod örneklerini projenize yapıştırabilirsiniz. (Web geliştiricisiyseniz, mobil tarayıcı testi için zaten yerel bir ortam kurmuş olabilirsiniz!)

### Geliştirici Notları

Birçok farklı geliştirme geçmişinden insanlar React Native öğreniyor. Web'den Android'e, iOS'a ve daha fazlasına kadar bir dizi teknoloji deneyiminiz olabilir. Tüm geçmişlerden geliştiriciler için yazmaya çalışıyoruz. Bazen şu gibi bir platforma özel açıklamalar sağlıyoruz:

#### Android
Android geliştiricileri bu kavrama aşina olabilir.

#### iOS
iOS geliştiricileri bu kavrama aşina olabilir.

#### Web
Web geliştiricileri bu kavrama aşina olabilir.

### Formatlama

Menü yolları kalın yazılır ve alt menülerde gezinmek için caret'ler kullanır. Örnek: **Android Studio > Preferences**

---

Artık bu rehberin nasıl çalıştığını bildiğinize göre, React Native'in temelini tanıma zamanı: Native Components.

## Temel Bilgiler

### Core Components ve Native Components

React Native, native platform bileşenlerini JavaScript'te kullanmanızı sağlar. Bu bileşenler, iOS ve Android'deki native UI bileşenlerine karşılık gelir.

#### Temel Bileşenler

```jsx
import React from 'react';
import { 
  View, 
  Text, 
  Image, 
  ScrollView, 
  TextInput, 
  TouchableOpacity,
  StyleSheet 
} from 'react-native';

function BasicComponents() {
  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>React Native Temel Bileşenler</Text>
      </View>
      
      <View style={styles.content}>
        <Text style={styles.text}>Bu bir Text bileşenidir</Text>
        
        <Image 
          source={{ uri: 'https://reactnative.dev/img/tiny_logo.png' }}
          style={styles.image}
        />
        
        <TextInput 
          style={styles.input}
          placeholder="Bir şeyler yazın..."
          multiline
        />
        
        <TouchableOpacity style={styles.button}>
          <Text style={styles.buttonText}>Tıkla</Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    padding: 20,
    backgroundColor: '#f0f0f0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  content: {
    padding: 20,
  },
  text: {
    fontSize: 16,
    marginBottom: 10,
  },
  image: {
    width: 100,
    height: 100,
    marginBottom: 10,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    padding: 10,
    marginBottom: 10,
    borderRadius: 5,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 5,
    alignItems: 'center',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default BasicComponents;
```

### React Temelleri

React Native, React'in temel prensiplerini kullanır. İşte React Native'de React temellerinin nasıl uygulandığı:

#### State ve Props

```jsx
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Props kullanımı
function Greeting({ name, age }) {
  return (
    <View style={styles.greeting}>
      <Text style={styles.text}>Merhaba, {name}!</Text>
      <Text style={styles.text}>Yaşınız: {age}</Text>
    </View>
  );
}

// State kullanımı
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <View style={styles.counter}>
      <Text style={styles.countText}>Sayaç: {count}</Text>
      <View style={styles.buttonContainer}>
        <TouchableOpacity style={styles.button} onPress={decrement}>
          <Text style={styles.buttonText}>-</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.button} onPress={increment}>
          <Text style={styles.buttonText}>+</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

function App() {
  return (
    <View style={styles.container}>
      <Greeting name="Ahmet" age={25} />
      <Counter />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  greeting: {
    marginBottom: 30,
    alignItems: 'center',
  },
  text: {
    fontSize: 18,
    marginBottom: 5,
  },
  counter: {
    alignItems: 'center',
  },
  countText: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    width: 50,
    height: 50,
    borderRadius: 25,
    justifyContent: 'center',
    alignItems: 'center',
  },
  buttonText: {
    color: '#fff',
    fontSize: 24,
    fontWeight: 'bold',
  },
});

export default App;
```

### Text Input İşleme

React Native'de metin girişi işleme:

```jsx
import React, { useState } from 'react';
import { 
  View, 
  Text, 
  TextInput, 
  TouchableOpacity, 
  Alert,
  StyleSheet 
} from 'react-native';

function TextInputExample() {
  const [text, setText] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = () => {
    if (!text.trim()) {
      Alert.alert('Hata', 'Lütfen bir metin girin');
      return;
    }
    
    if (!email.includes('@')) {
      Alert.alert('Hata', 'Geçerli bir email adresi girin');
      return;
    }
    
    Alert.alert('Başarılı', `Metin: ${text}\nEmail: ${email}`);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Text Input Örneği</Text>
      
      <TextInput
        style={styles.input}
        placeholder="Bir metin girin..."
        value={text}
        onChangeText={setText}
        multiline
        numberOfLines={3}
      />
      
      <TextInput
        style={styles.input}
        placeholder="Email adresiniz"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
        autoCorrect={false}
      />
      
      <TextInput
        style={styles.input}
        placeholder="Şifreniz"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        autoCapitalize="none"
      />
      
      <TouchableOpacity style={styles.button} onPress={handleSubmit}>
        <Text style={styles.buttonText}>Gönder</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 15,
    marginBottom: 15,
    fontSize: 16,
    backgroundColor: '#f9f9f9',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 10,
  },
  buttonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default TextInputExample;
```

### ScrollView Kullanımı

ScrollView ile kaydırılabilir içerik:

```jsx
import React from 'react';
import { ScrollView, View, Text, Image, StyleSheet } from 'react-native';

function ScrollViewExample() {
  const items = Array.from({ length: 20 }, (_, i) => ({
    id: i + 1,
    title: `Öğe ${i + 1}`,
    description: `Bu ${i + 1}. öğenin açıklamasıdır.`,
  }));

  return (
    <ScrollView 
      style={styles.container}
      showsVerticalScrollIndicator={true}
      contentContainerStyle={styles.contentContainer}
    >
      <Text style={styles.header}>ScrollView Örneği</Text>
      
      {items.map((item) => (
        <View key={item.id} style={styles.item}>
          <Image 
            source={{ uri: `https://picsum.photos/100/100?random=${item.id}` }}
            style={styles.image}
          />
          <View style={styles.textContainer}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.description}>{item.description}</Text>
          </View>
        </View>
      ))}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  contentContainer: {
    padding: 20,
  },
  header: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333',
  },
  item: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    padding: 15,
    marginBottom: 10,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.1,
    shadowRadius: 3.84,
    elevation: 5,
  },
  image: {
    width: 60,
    height: 60,
    borderRadius: 30,
    marginRight: 15,
  },
  textContainer: {
    flex: 1,
    justifyContent: 'center',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 5,
    color: '#333',
  },
  description: {
    fontSize: 14,
    color: '#666',
  },
});

export default ScrollViewExample;
```

### List View Kullanımı

FlatList ile performanslı liste görünümü:

```jsx
import React, { useState, useEffect } from 'react';
import { 
  FlatList, 
  View, 
  Text, 
  TouchableOpacity, 
  RefreshControl,
  StyleSheet 
} from 'react-native';

function ListViewExample() {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    setLoading(true);
    // Simüle edilmiş API çağrısı
    setTimeout(() => {
      const newData = Array.from({ length: 50 }, (_, i) => ({
        id: i + 1,
        title: `Öğe ${i + 1}`,
        subtitle: `Alt başlık ${i + 1}`,
        description: `Bu ${i + 1}. öğenin detaylı açıklamasıdır.`,
      }));
      setData(newData);
      setLoading(false);
    }, 1000);
  };

  const onRefresh = async () => {
    setRefreshing(true);
    await loadData();
    setRefreshing(false);
  };

  const renderItem = ({ item }) => (
    <TouchableOpacity style={styles.item}>
      <View style={styles.itemContent}>
        <Text style={styles.title}>{item.title}</Text>
        <Text style={styles.subtitle}>{item.subtitle}</Text>
        <Text style={styles.description}>{item.description}</Text>
      </View>
    </TouchableOpacity>
  );

  const renderHeader = () => (
    <View style={styles.header}>
      <Text style={styles.headerText}>FlatList Örneği</Text>
    </View>
  );

  const renderFooter = () => (
    <View style={styles.footer}>
      <Text style={styles.footerText}>Toplam {data.length} öğe</Text>
    </View>
  );

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <Text style={styles.loadingText}>Yükleniyor...</Text>
      </View>
    );
  }

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      ListHeaderComponent={renderHeader}
      ListFooterComponent={renderFooter}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
      showsVerticalScrollIndicator={true}
      contentContainerStyle={styles.listContainer}
    />
  );
}

const styles = StyleSheet.create({
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: {
    fontSize: 18,
    color: '#666',
  },
  listContainer: {
    padding: 10,
  },
  header: {
    padding: 20,
    backgroundColor: '#f0f0f0',
    borderRadius: 10,
    marginBottom: 10,
  },
  headerText: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    color: '#333',
  },
  item: {
    backgroundColor: '#fff',
    padding: 15,
    marginBottom: 10,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.1,
    shadowRadius: 3.84,
    elevation: 5,
  },
  itemContent: {
    flex: 1,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 5,
    color: '#333',
  },
  subtitle: {
    fontSize: 14,
    color: '#007AFF',
    marginBottom: 5,
  },
  description: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
  footer: {
    padding: 20,
    alignItems: 'center',
  },
  footerText: {
    fontSize: 16,
    color: '#666',
  },
});

export default ListViewExample;
```

### Sorun Giderme

React Native'de yaygın sorunlar ve çözümleri:

#### 1. Metro Bundler Sorunları

```bash
# Metro cache'i temizle
npx react-native start --reset-cache

# Node modules'ü temizle ve yeniden yükle
rm -rf node_modules
npm install

# iOS için pod'ları temizle
cd ios && pod deintegrate && pod install
```

#### 2. Platform Spesifik Kod

```jsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: Platform.OS === 'ios' ? 20 : 0,
    backgroundColor: Platform.OS === 'ios' ? '#fff' : '#f0f0f0',
  },
  text: {
    fontSize: Platform.OS === 'ios' ? 16 : 14,
    fontFamily: Platform.OS === 'ios' ? 'System' : 'Roboto',
  },
});

// Platform spesifik bileşen
function PlatformSpecificComponent() {
  if (Platform.OS === 'ios') {
    return <IOSComponent />;
  } else if (Platform.OS === 'android') {
    return <AndroidComponent />;
  }
  return <DefaultComponent />;
}
```

#### 3. Performans Optimizasyonu

```jsx
import React, { memo, useMemo, useCallback } from 'react';
import { FlatList, View, Text, StyleSheet } from 'react-native';

// Memoized bileşen
const ListItem = memo(({ item, onPress }) => {
  return (
    <TouchableOpacity style={styles.item} onPress={() => onPress(item)}>
      <Text style={styles.text}>{item.title}</Text>
    </TouchableOpacity>
  );
});

function OptimizedList({ data }) {
  // Memoized render function
  const renderItem = useCallback(({ item }) => (
    <ListItem item={item} onPress={handleItemPress} />
  ), []);

  // Memoized key extractor
  const keyExtractor = useCallback((item) => item.id.toString(), []);

  // Memoized item press handler
  const handleItemPress = useCallback((item) => {
    console.log('Item pressed:', item.id);
  }, []);

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={10}
    />
  );
}
```

## React Native Best Practices ve Mimari

### Sıfırdan Proje Yapısı

#### 1. Proje Klasör Yapısı

```
MyReactNativeApp/
├── src/
│   ├── components/          # Yeniden kullanılabilir bileşenler
│   │   ├── common/         # Genel bileşenler
│   │   ├── forms/          # Form bileşenleri
│   │   └── ui/             # UI bileşenleri
│   ├── screens/            # Ekran bileşenleri
│   │   ├── auth/           # Kimlik doğrulama ekranları
│   │   ├── home/           # Ana sayfa ekranları
│   │   └── profile/        # Profil ekranları
│   ├── navigation/         # Navigasyon yapılandırması
│   ├── services/           # API servisleri
│   ├── store/              # State management
│   ├── utils/              # Yardımcı fonksiyonlar
│   ├── constants/          # Sabitler
│   ├── hooks/              # Custom hooks
│   ├── types/              # TypeScript tipleri
│   └── assets/             # Görseller, fontlar
├── android/                # Android native kod
├── ios/                    # iOS native kod
├── __tests__/              # Test dosyaları
├── .env                    # Environment variables
├── package.json
├── metro.config.js
├── babel.config.js
└── tsconfig.json
```

#### 2. Temel Proje Kurulumu

```bash
# Yeni React Native projesi oluştur
npx react-native@latest init MyReactNativeApp

# Gerekli paketleri yükle
cd MyReactNativeApp
npm install @react-navigation/native @react-navigation/stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context
npm install @reduxjs/toolkit react-redux
npm install axios
npm install react-native-vector-icons
npm install react-native-async-storage
npm install react-native-config

# iOS için pod'ları yükle
cd ios && pod install && cd ..
```

#### 3. Environment Yapılandırması

```javascript
// .env
API_BASE_URL=https://api.example.com
API_KEY=your_api_key_here
APP_VERSION=1.0.0
DEBUG_MODE=true
```

```javascript
// src/config/environment.js
import Config from 'react-native-config';

export const ENV = {
  API_BASE_URL: Config.API_BASE_URL || 'https://api.example.com',
  API_KEY: Config.API_KEY || '',
  APP_VERSION: Config.APP_VERSION || '1.0.0',
  DEBUG_MODE: Config.DEBUG_MODE === 'true',
  IS_DEV: __DEV__,
};
```

### State Management (Redux Toolkit)

```javascript
// src/store/index.js
import { configureStore } from '@reduxjs/toolkit';
import authSlice from './slices/authSlice';
import userSlice from './slices/userSlice';

export const store = configureStore({
  reducer: {
    auth: authSlice,
    user: userSlice,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST'],
      },
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```javascript
// src/store/slices/authSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { authService } from '../../services/authService';

export const loginUser = createAsyncThunk(
  'auth/loginUser',
  async (credentials, { rejectWithValue }) => {
    try {
      const response = await authService.login(credentials);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    token: null,
    loading: false,
    error: null,
  },
  reducers: {
    logout: (state) => {
      state.user = null;
      state.token = null;
      state.error = null;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});

export const { logout, clearError } = authSlice.actions;
export default authSlice.reducer;
```

### Navigation Yapılandırması

```javascript
// src/navigation/AppNavigator.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { useSelector } from 'react-redux';

// Screens
import LoginScreen from '../screens/auth/LoginScreen';
import HomeScreen from '../screens/home/HomeScreen';
import ProfileScreen from '../screens/profile/ProfileScreen';
import SettingsScreen from '../screens/settings/SettingsScreen';

// Icons
import Icon from 'react-native-vector-icons/MaterialIcons';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

function AuthStack() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="Login" component={LoginScreen} />
    </Stack.Navigator>
  );
}

function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;

          if (route.name === 'Home') {
            iconName = 'home';
          } else if (route.name === 'Profile') {
            iconName = 'person';
          } else if (route.name === 'Settings') {
            iconName = 'settings';
          }

          return <Icon name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
}

function MainStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen 
        name="MainTabs" 
        component={MainTabs} 
        options={{ headerShown: false }}
      />
    </Stack.Navigator>
  );
}

export default function AppNavigator() {
  const { token } = useSelector((state) => state.auth);

  return (
    <NavigationContainer>
      {token ? <MainStack /> : <AuthStack />}
    </NavigationContainer>
  );
}
```

### API Service Katmanı

```javascript
// src/services/apiService.js
import axios from 'axios';
import { ENV } from '../config/environment';
import { store } from '../store';
import { logout } from '../store/slices/authSlice';

const apiClient = axios.create({
  baseURL: ENV.API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    const state = store.getState();
    const token = state.auth.token;
    
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error.response?.status === 401) {
      store.dispatch(logout());
    }
    
    return Promise.reject(error);
  }
);

export default apiClient;
```

```javascript
// src/services/authService.js
import apiClient from './apiService';

export const authService = {
  login: async (credentials) => {
    const response = await apiClient.post('/auth/login', credentials);
    return response.data;
  },
  
  register: async (userData) => {
    const response = await apiClient.post('/auth/register', userData);
    return response.data;
  },
  
  logout: async () => {
    const response = await apiClient.post('/auth/logout');
    return response.data;
  },
  
  refreshToken: async () => {
    const response = await apiClient.post('/auth/refresh');
    return response.data;
  },
};
```

### Custom Hooks

```javascript
// src/hooks/useApi.js
import { useState, useEffect } from 'react';

export function useApi(apiFunction, dependencies = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const execute = async (...args) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await apiFunction(...args);
      setData(result);
      return result;
    } catch (err) {
      setError(err);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (dependencies.length > 0) {
      execute();
    }
  }, dependencies);

  return { data, loading, error, execute };
}
```

```javascript
// src/hooks/useAuth.js
import { useSelector, useDispatch } from 'react-redux';
import { loginUser, logout, clearError } from '../store/slices/authSlice';

export function useAuth() {
  const dispatch = useDispatch();
  const { user, token, loading, error } = useSelector((state) => state.auth);

  const login = (credentials) => {
    return dispatch(loginUser(credentials));
  };

  const logoutUser = () => {
    dispatch(logout());
  };

  const clearAuthError = () => {
    dispatch(clearError());
  };

  return {
    user,
    token,
    loading,
    error,
    isAuthenticated: !!token,
    login,
    logout: logoutUser,
    clearError: clearAuthError,
  };
}
```

### Utility Fonksiyonlar

```javascript
// src/utils/validation.js
export const validation = {
  email: (email) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  },
  
  password: (password) => {
    return password.length >= 8;
  },
  
  phone: (phone) => {
    const phoneRegex = /^[0-9]{10,11}$/;
    return phoneRegex.test(phone.replace(/\s/g, ''));
  },
  
  required: (value) => {
    return value && value.trim().length > 0;
  },
};
```

```javascript
// src/utils/storage.js
import AsyncStorage from '@react-native-async-storage/async-storage';

export const storage = {
  setItem: async (key, value) => {
    try {
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Storage setItem error:', error);
    }
  },
  
  getItem: async (key) => {
    try {
      const value = await AsyncStorage.getItem(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('Storage getItem error:', error);
      return null;
    }
  },
  
  removeItem: async (key) => {
    try {
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Storage removeItem error:', error);
    }
  },
  
  clear: async () => {
    try {
      await AsyncStorage.clear();
    } catch (error) {
      console.error('Storage clear error:', error);
    }
  },
};
```

### Önerilen Paketler

#### Temel Paketler
```bash
# Navigation
npm install @react-navigation/native @react-navigation/stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context

# State Management
npm install @reduxjs/toolkit react-redux
npm install redux-persist

# HTTP Client
npm install axios

# Storage
npm install @react-native-async-storage/async-storage

# Icons
npm install react-native-vector-icons

# Environment
npm install react-native-config

# Forms
npm install react-hook-form
npm install yup @hookform/resolvers/yup

# UI Components
npm install react-native-elements
npm install react-native-paper

# Image Handling
npm install react-native-image-picker
npm install react-native-image-crop-picker

# Notifications
npm install @react-native-firebase/messaging

# Analytics
npm install @react-native-firebase/analytics

# Crash Reporting
npm install @react-native-firebase/crashlytics

# Testing
npm install --save-dev @testing-library/react-native
npm install --save-dev jest
npm install --save-dev detox
```

#### Gelişmiş Paketler
```bash
# Animation
npm install react-native-reanimated
npm install react-native-gesture-handler

# Maps
npm install react-native-maps

# Charts
npm install react-native-chart-kit

# PDF
npm install react-native-pdf

# Camera
npm install react-native-camera

# Barcode Scanner
npm install react-native-barcode-scanner

# Biometric
npm install react-native-biometrics

# Push Notifications
npm install @react-native-firebase/messaging

# Deep Linking
npm install react-native-deep-linking

# Share
npm install react-native-share

# File System
npm install react-native-fs

# Network Info
npm install @react-native-community/netinfo

# Device Info
npm install react-native-device-info
```

### Performans Optimizasyonu

```javascript
// src/components/OptimizedList.js
import React, { memo, useMemo, useCallback } from 'react';
import { FlatList, View, Text, StyleSheet } from 'react-native';

const ListItem = memo(({ item, onPress }) => {
  return (
    <TouchableOpacity style={styles.item} onPress={() => onPress(item)}>
      <Text style={styles.text}>{item.title}</Text>
    </TouchableOpacity>
  );
});

function OptimizedList({ data }) {
  const renderItem = useCallback(({ item }) => (
    <ListItem item={item} onPress={handleItemPress} />
  ), []);

  const keyExtractor = useCallback((item) => item.id.toString(), []);

  const handleItemPress = useCallback((item) => {
    console.log('Item pressed:', item.id);
  }, []);

  const getItemLayout = useCallback((data, index) => ({
    length: 60,
    offset: 60 * index,
    index,
  }), []);

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={10}
      updateCellsBatchingPeriod={50}
    />
  );
}
```

### Test Stratejisi

```javascript
// __tests__/components/Button.test.js
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import Button from '../../src/components/common/Button';

describe('Button Component', () => {
  it('renders correctly', () => {
    const { getByText } = render(
      <Button title="Test Button" onPress={() => {}} />
    );
    
    expect(getByText('Test Button')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const mockOnPress = jest.fn();
    const { getByText } = render(
      <Button title="Test Button" onPress={mockOnPress} />
    );
    
    fireEvent.press(getByText('Test Button'));
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });
});
```

### Güvenlik Best Practices

```javascript
// src/utils/security.js
import { Platform } from 'react-native';

export const security = {
  // Sensitive data'ları güvenli şekilde sakla
  storeSecureData: async (key, value) => {
    if (Platform.OS === 'ios') {
      // Keychain kullan
      await Keychain.setInternetCredentials(key, key, value);
    } else {
      // Android Keystore kullan
      await SecureStore.setItemAsync(key, value);
    }
  },
  
  // SSL Pinning
  validateCertificate: (certificate) => {
    // Certificate validation logic
    return true;
  },
  
  // Code Obfuscation için
  obfuscateString: (str) => {
    return btoa(str);
  },
};
```

Bu kapsamlı React Native dokümantasyonu ve best practices rehberi, sıfırdan profesyonel bir React Native projesi kurmanız için gereken tüm bilgileri içermektedir. [React Native resmi dokümantasyonu](https://reactnative.dev/docs/getting-started) temel alınarak hazırlanmıştır.

---

**React Native Nedir?**

React Native, Facebook (Meta) tarafından geliştirilen, JavaScript ve React kullanarak native mobil uygulamalar oluşturmak için kullanılan açık kaynaklı bir framework'tür. React Native, tek bir kod tabanı ile hem iOS hem de Android platformları için uygulama geliştirmeyi sağlar. React Native, native performansı JavaScript'in esnekliği ile birleştirir.

**React Native Ne Demek?**

React Native, "React for Native" anlamına gelir. React'in web geliştirmedeki başarısını mobil platformlara taşıyan bir framework'tür. React Native, JavaScript ile yazılan kodları native iOS ve Android uygulamalarına dönüştürür.

**Neden İhtiyaç Var?**

Mobil uygulama geliştirmede geleneksel yaklaşımların sorunları:

1. **Native Development**: iOS ve Android için ayrı ayrı geliştirme gerekiyor
```swift
// iOS (Swift) - Native development
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        label.text = "Hello from iOS!"
    }
}
```

```java
// Android (Java) - Native development
public class MainActivity extends AppCompatActivity {
    private TextView textView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        textView = findViewById(R.id.textView);
        textView.setText("Hello from Android!");
    }
}
```

2. **Hybrid Apps**: Cordova/PhoneGap ile performans sorunları
3. **Cross-Platform**: Xamarin ile karmaşık geliştirme süreci
4. **Code Duplication**: Aynı işlevsellik için iki kez kod yazma
5. **Development Speed**: Geliştirme hızı ve maliyet sorunları

**Ne İşe Yarar?**

React Native, mobil uygulama geliştirmeyi daha kolay ve verimli hale getirir:

1. **Single Codebase**: Tek kod tabanı ile iki platform
2. **Native Performance**: Gerçek native performans
3. **JavaScript Ecosystem**: Mevcut JavaScript bilgisi ve kütüphaneleri
4. **Hot Reloading**: Anında kod güncelleme
5. **Cost Effective**: Maliyet etkin geliştirme

**React Native'in Temel Özellikleri:**

1. **Cross-Platform Development - Çapraz Platform Geliştirme**: Tek kod, iki platform
```jsx
// React Native - Tek kod, iki platform
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

function App() {
    return (
        <View style={styles.container}>
            <Text style={styles.text}>Hello from React Native!</Text>
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    text: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
});

export default App;
```

2. **Native Performance - Native Performans**: Gerçek native performans
```jsx
// React Native'de native performans
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, StyleSheet } from 'react-native';

function UserList() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetchUsers();
    }, []);

    const fetchUsers = async () => {
        try {
            const response = await fetch('https://api.example.com/users');
            const data = await response.json();
            setUsers(data);
        } catch (error) {
            console.error('Error fetching users:', error);
        } finally {
            setLoading(false);
        }
    };

    const renderUser = ({ item }) => (
        <View style={styles.userItem}>
            <Text style={styles.userName}>{item.name}</Text>
            <Text style={styles.userEmail}>{item.email}</Text>
        </View>
    );

    if (loading) {
        return <Text>Loading...</Text>;
    }

    return (
        <FlatList
            data={users}
            renderItem={renderUser}
            keyExtractor={item => item.id.toString()}
            style={styles.list}
        />
    );
}

const styles = StyleSheet.create({
    list: {
        flex: 1,
    },
    userItem: {
        padding: 15,
        borderBottomWidth: 1,
        borderBottomColor: '#eee',
    },
    userName: {
        fontSize: 16,
        fontWeight: 'bold',
    },
    userEmail: {
        fontSize: 14,
        color: '#666',
    },
});
```

3. **JavaScript Ecosystem - JavaScript Ekosistemi**: Mevcut kütüphaneler
```jsx
// React Native'de JavaScript ekosistemi
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, Alert } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import NetInfo from '@react-native-community/netinfo';

function App() {
    const [isConnected, setIsConnected] = useState(true);

    useEffect(() => {
        const unsubscribe = NetInfo.addEventListener(state => {
            setIsConnected(state.isConnected);
        });

        return () => unsubscribe();
    }, []);

    const saveData = async () => {
        try {
            await AsyncStorage.setItem('userData', JSON.stringify({
                name: 'John Doe',
                email: 'john@example.com'
            }));
            Alert.alert('Success', 'Data saved successfully!');
        } catch (error) {
            Alert.alert('Error', 'Failed to save data');
        }
    };

    return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
            <Text>Network Status: {isConnected ? 'Connected' : 'Disconnected'}</Text>
            <TouchableOpacity onPress={saveData} style={{ marginTop: 20 }}>
                <Text>Save Data</Text>
            </TouchableOpacity>
        </View>
    );
}
```

4. **Hot Reloading - Sıcak Yeniden Yükleme**: Anında kod güncelleme
```jsx
// React Native'de hot reloading
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

function Counter() {
    const [count, setCount] = useState(0);

    return (
        <View style={styles.container}>
            <Text style={styles.count}>{count}</Text>
            <TouchableOpacity 
                style={styles.button}
                onPress={() => setCount(count + 1)}
            >
                <Text style={styles.buttonText}>Increment</Text>
            </TouchableOpacity>
        </View>
    );
}

// Hot reloading ile kod değişiklikleri anında görünür
// Geliştirme sırasında uygulamayı yeniden başlatmaya gerek yok
```

5. **Platform-Specific Code - Platform Özel Kod**: Platform farklılıkları
```jsx
// React Native'de platform özel kod
import React from 'react';
import { View, Text, Platform, StyleSheet } from 'react-native';

function PlatformSpecificComponent() {
    return (
        <View style={styles.container}>
            <Text style={styles.text}>
                {Platform.OS === 'ios' ? 'Hello from iOS!' : 'Hello from Android!'}
            </Text>
            
            {Platform.OS === 'ios' && (
                <Text style={styles.iosText}>This text only appears on iOS</Text>
            )}
            
            {Platform.OS === 'android' && (
                <Text style={styles.androidText}>This text only appears on Android</Text>
            )}
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    text: {
        fontSize: 18,
        marginBottom: 20,
    },
    iosText: {
        color: '#007AFF',
        fontSize: 16,
    },
    androidText: {
        color: '#4CAF50',
        fontSize: 16,
    },
});
```

**React Native'in Faydaları:**

1. **Single Codebase - Tek Kod Tabanı**: Kod tekrarını önler
```jsx
// Tek kod tabanı ile iki platform
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

function UserProfile({ user }) {
    return (
        <View style={styles.container}>
            <Text style={styles.name}>{user.name}</Text>
            <Text style={styles.email}>{user.email}</Text>
            <Text style={styles.phone}>{user.phone}</Text>
        </View>
    );
}

// Bu component hem iOS hem de Android'de çalışır
// Kod tekrarı yok, sürdürülebilirlik yüksek
```

2. **Faster Development - Hızlı Geliştirme**: Geliştirme süresini kısaltır
```jsx
// React Native ile hızlı geliştirme
import React, { useState } from 'react';
import { View, TextInput, Button, Alert } from 'react-native';

function LoginForm() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleLogin = () => {
        if (email && password) {
            Alert.alert('Success', 'Login successful!');
        } else {
            Alert.alert('Error', 'Please fill all fields');
        }
    };

    return (
        <View style={{ padding: 20 }}>
            <TextInput
                placeholder="Email"
                value={email}
                onChangeText={setEmail}
                style={{ borderWidth: 1, padding: 10, marginBottom: 10 }}
            />
            <TextInput
                placeholder="Password"
                value={password}
                onChangeText={setPassword}
                secureTextEntry
                style={{ borderWidth: 1, padding: 10, marginBottom: 20 }}
            />
            <Button title="Login" onPress={handleLogin} />
        </View>
    );
}

// Hızlı prototipleme ve geliştirme
// Hot reloading ile anında test
```

3. **Cost Effective - Maliyet Etkin**: Geliştirme maliyetini düşürür
```jsx
// React Native ile maliyet etkin geliştirme
// Tek geliştirici, iki platform
// Tek kod tabanı, iki uygulama
// Daha az test, daha az bakım
```

**React Native'in Kullanım Alanları:**

1. **Mobile Applications - Mobil Uygulamalar**: Native mobil uygulamalar
2. **Cross-Platform Apps - Çapraz Platform Uygulamaları**: iOS ve Android
3. **Prototype Development - Prototip Geliştirme**: Hızlı prototipleme
4. **MVP Development - Minimum Viable Product Geliştirme**: Hızlı MVP
5. **Enterprise Applications - Kurumsal Uygulamalar**: Büyük şirket uygulamaları
6. **E-commerce Apps - E-ticaret Uygulamaları**: Online mağaza uygulamaları
7. **Social Media Apps - Sosyal Medya Uygulamaları**: Sosyal platform uygulamaları
8. **Gaming Applications - Oyun Uygulamaları**: Mobil oyun uygulamaları

### 4.1 React Native'in Tarihçesi

**React Native'in Doğuşu - Neden Gerekli Oldu?**

Mobil uygulama geliştirmede geleneksel yaklaşımların sorunları:

1. **Native Development**: iOS ve Android için ayrı ayrı geliştirme gerekiyor
```swift
// iOS (Swift) - Native development
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        label.text = "Hello from iOS!"
    }
}
```

```java
// Android (Java) - Native development
public class MainActivity extends AppCompatActivity {
    private TextView textView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        textView = findViewById(R.id.textView);
        textView.setText("Hello from Android!");
    }
}
```

2. **Hybrid Apps**: Cordova/PhoneGap ile performans sorunları
3. **Cross-Platform**: Xamarin ile karmaşık geliştirme süreci
4. **Code Duplication**: Aynı işlevsellik için iki kez kod yazma
5. **Development Speed**: Geliştirme hızı ve maliyet sorunları

**Facebook'un Mobil Sorunları**:

**2012 - Facebook'un Mobil Stratejisi:**
- **HTML5 vs Native**: Facebook'un HTML5 uygulaması başarısız oldu
- **Native Development**: iOS ve Android için ayrı ayrı geliştirme gerekiyordu
- **Developer Resources**: Geliştirici kaynaklarının ikiye bölünmesi
- **Code Duplication**: Kod tekrarı ve sürdürülebilirlik sorunları

**Facebook'un HTML5 Deneyimi:**
```html
<!-- Facebook'un HTML5 uygulaması (2012) -->
<!DOCTYPE html>
<html>
<head>
    <title>Facebook Mobile</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <div id="app">
        <!-- HTML5 uygulaması -->
    </div>
    <script src="facebook-mobile.js"></script>
</body>
</html>
```

**Facebook'un Mobil Stratejisi:**
- **2012**: Facebook, HTML5 uygulamasını terk etti
- **Native Apps**: iOS ve Android için native uygulamalar geliştirdi
- **Performance Issues**: Native uygulamalarda da performans sorunları
- **Development Speed**: Geliştirme hızı sorunları
- **Code Sharing**: Kod paylaşımı ihtiyacı

**React Native Projesinin Başlangıcı**:

**2013 - Facebook'ta React Native Projesi:**
- **Christopher Chedeau**: React Native'in ana geliştiricisi
- **React Inspiration**: React'in web'deki başarısından ilham alındı
- **Mobile Web**: Mobil web uygulamalarının sınırları
- **Native Performance**: Native performans ihtiyacı

**Christopher Chedeau Kimdir?**
- **Facebook'ta Yazılım Mühendisi**: React Native'in ana geliştiricisi
- **React Contributor**: React ekibinin üyesi
- **Mobile Development**: Mobil geliştirme deneyimi
- **Performance Expert**: Performans optimizasyonu uzmanı

**React Native'in İlham Kaynağı:**
```jsx
// React'in web'deki başarısı
function UserProfile({ user }) {
    return (
        <div className="user-profile">
            <img src={user.avatar} alt={user.name} />
            <h2>{user.name}</h2>
            <p>{user.email}</p>
        </div>
    );
}

// React Native ile mobil platformlara taşınması
function UserProfile({ user }) {
    return (
        <View style={styles.container}>
            <Image source={{ uri: user.avatar }} style={styles.avatar} />
            <Text style={styles.name}>{user.name}</Text>
            <Text style={styles.email}>{user.email}</Text>
        </View>
    );
}
```

**2014 - React Native Geliştirme**:

**Internal Development:**
- **Facebook'un İç Projeleri**: Facebook'un iç projelerinde geliştirildi
- **Ads Manager**: Facebook'un Ads Manager uygulamasında test edildi
- **Performance Testing**: Performans testleri yapıldı
- **Bridge Architecture**: JavaScript-Native bridge mimarisi geliştirildi

**Bridge Architecture:**
```javascript
// React Native Bridge Architecture
// JavaScript Thread
const user = { name: 'John', email: 'john@example.com' };

// Bridge Communication
NativeModules.UserModule.saveUser(user);

// Native Thread (iOS/Android)
// Native code receives the data and processes it
```

**2015 - React Native'in Doğuşu**:

**Ocak 2015 - React Conf:**
- **React Native İlk Kez Tanıtıldı**: React Conf'te Christopher Chedeau tarafından
- **"React Native: Bringing modern web techniques to mobile"**: Konuşma başlığı
- **Open Source**: Açık kaynak olarak yayınlandı
- **iOS Support**: İlk iOS desteği
- **Facebook Internal**: Facebook'un iç uygulamalarında kullanıldı

**React Conf Konuşması:**
```jsx
// Christopher Chedeau'nun React Conf'teki örneği
import React, { Component } from 'react';
import { View, Text, StyleSheet } from 'react-native';

class HelloWorld extends Component {
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.text}>Hello, World!</Text>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    text: {
        fontSize: 20,
        textAlign: 'center',
    },
});
```

**React Native'in Tasarım Felsefesi:**

1. **Learn Once, Write Anywhere**: Bir kez öğren, her yerde yaz
2. **Native Performance**: Native performans
3. **JavaScript Ecosystem**: JavaScript ekosistemi ile uyumluluk
4. **Hot Reloading**: Geliştirme sırasında anında güncelleme
5. **Code Sharing**: Kod paylaşımı

**2015 - React Native 0.1-0.20 - İlk Sürümler**:

**Temel Özellikler:**
- **Temel Component'ler**: View, Text, Image, ScrollView
- **iOS Support**: İlk iOS desteği
- **Basic Styling**: StyleSheet ve Flexbox
- **Hot Reloading**: Geliştirme sırasında anında güncelleme
- **React Native CLI**: Command line interface

**React Native 0.1 Örneği:**
```jsx
// React Native 0.1'de ilk örnek
import React, { Component } from 'react';
import { View, Text, StyleSheet } from 'react-native';

class HelloWorld extends Component {
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.text}>Hello, React Native!</Text>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    text: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
});

export default HelloWorld;
```

**2015 - React Native 0.21-0.40 - Android Desteği**:

**Yeni Özellikler:**
- **Android Support**: Android platform desteği
- **Navigator Sistemi**: Navigation component'i
- **AsyncStorage**: Yerel veri saklama
- **NetInfo**: Ağ durumu bilgisi
- **Platform-specific Code**: Platform özel kod desteği

**Android Desteği Örneği:**
```jsx
// React Native 0.21'de Android desteği
import React, { Component } from 'react';
import { View, Text, Platform, StyleSheet } from 'react-native';

class PlatformSpecific extends Component {
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.text}>
                    {Platform.OS === 'ios' ? 'Hello from iOS!' : 'Hello from Android!'}
                </Text>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    text: {
        fontSize: 18,
        textAlign: 'center',
    },
});
```

**Navigator Sistemi Örneği:**
```jsx
// React Native 0.21'de Navigator
import React, { Component } from 'react';
import { Navigator, View, Text } from 'react-native';

class App extends Component {
    renderScene(route, navigator) {
        switch (route.name) {
            case 'Home':
                return <HomeView navigator={navigator} />;
            case 'Profile':
                return <ProfileView navigator={navigator} />;
            default:
                return <HomeView navigator={navigator} />;
        }
    }

    render() {
        return (
            <Navigator
                initialRoute={{ name: 'Home' }}
                renderScene={this.renderScene}
            />
        );
    }
}
```

**2016 - React Native 0.41-0.60 - Metro ve Flipper**:

**Yeni Özellikler:**
- **Metro Bundler**: Metro bundler
- **Flipper Integration**: Flipper entegrasyonu
- **Hermes JavaScript Engine**: Hermes JavaScript engine
- **New Architecture Preview**: Yeni mimari önizleme

**Metro Bundler:**
```javascript
// Metro bundler configuration
// metro.config.js
module.exports = {
    transformer: {
        getTransformOptions: async () => ({
            transform: {
                experimentalImportSupport: false,
                inlineRequires: true,
            },
        }),
    },
};
```

**Flipper Integration:**
```jsx
// React Native'de Flipper entegrasyonu
import { Flipper } from 'react-native-flipper';

function App() {
    return (
        <View>
            <Flipper />
            {/* App content */}
        </View>
    );
}
```
- **TurboModules**: Turbo modüller

**2017 - React Native 0.61-0.70**:
- **Auto-linking**: Otomatik bağlama
- **CocoaPods Integration**: CocoaPods entegrasyonu
- **Fast Refresh**: Hızlı yenileme
- **Hermes Improvements**: Hermes iyileştirmeleri
- **Better Performance**: Performans iyileştirmeleri

**2018 - React Native 0.71+**:
- **New Architecture Rollout**: Fabric renderer ve TurboModules
- **Fabric Renderer**: Yeni rendering sistemi
- **TurboModules**: Native modül sistemi
- **JSI (JavaScript Interface)**: JavaScript ve native kod arası iletişim
- **Codegen**: Otomatik native kod üretimi

**2019 - React Native 0.75+**:
- **Improved New Architecture**: Fabric ve TurboModules stabilizasyonu
- **Better Performance**: Render performansı iyileştirmeleri
- **Enhanced DevTools**: Gelişmiş geliştirici araçları
- **Better TypeScript Support**: Gelişmiş TypeScript desteği
- **Improved Metro**: Metro bundler iyileştirmeleri

**2020 - React Native 0.78+**:
- **React 19 Support**: React 19 özelliklerinin mobil platformlara entegrasyonu
- **Legacy Architecture Deprecation**: Eski mimarinin kaldırılması
- **Stable JavaScript API**: Kararlı JavaScript API'si
- **TypeScript First**: Varsayılan TypeScript desteği
- **Expo Atlas Integration**: Bundle boyutu analiz aracı entegrasyonu
- **WebAssembly Support**: WebAssembly ile performans optimizasyonu
- **Enhanced New Architecture**: Gelişmiş yeni mimari
- **Better Memory Management**: Gelişmiş bellek yönetimi
- **Improved Hot Reloading**: Gelişmiş hot reloading
- **Cross-Platform Desktop Support**: Masaüstü platform desteği

**React Native'in Başarı Hikayesi**:
- **Facebook**: Facebook'un mobil uygulamalarında kullanım
- **Instagram**: Instagram'ın React Native'e geçişi
- **Airbnb**: Airbnb'nin React Native deneyimi
- **Uber**: Uber'in React Native kullanımı
- **Walmart**: Walmart'ın React Native uygulamaları
- **Tesla**: Tesla'nın React Native uygulamaları
- **Discord**: Discord'un React Native uygulamaları
- **Pinterest**: Pinterest'in React Native uygulamaları

**İlk Çözümler**:
- **Cross-Platform Development**: Tek kod tabanı ile iOS ve Android uygulamaları
- **Native Performance**: JavaScript bridge ile native performans
- **JavaScript Ecosystem**: Mevcut JavaScript kütüphanelerini kullanabilme
- **Hot Reloading**: Geliştirme sırasında anında değişiklik görme
- **Code Sharing**: Web ve mobil arasında kod paylaşımı
- **Developer Experience**: Geliştirici deneyimi iyileştirmeleri
- **Rapid Development**: Hızlı geliştirme süreci
- **Maintenance**: Kolay bakım ve güncelleme

### 4.2 React Native Temel Kavramlar

#### 4.2.1 Core Components

React Native'de temel UI component'leri vardır. Bu component'ler native platform component'lerine dönüştürülür.

**View Component**:

```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

// Temel View kullanımı
function BasicView() {
    return (
        <View style={styles.container}>
            <Text>Hello World!</Text>
        </View>
    );
}

// Nested Views
function NestedViews() {
    return (
        <View style={styles.container}>
            <View style={styles.header}>
                <Text style={styles.headerText}>Header</Text>
            </View>
            <View style={styles.content}>
                <Text style={styles.contentText}>Content</Text>
            </View>
            <View style={styles.footer}>
                <Text style={styles.footerText}>Footer</Text>
            </View>
        </View>
    );
}

// View with flexbox
function FlexboxView() {
    return (
        <View style={styles.flexContainer}>
            <View style={[styles.box, { backgroundColor: 'red' }]} />
            <View style={[styles.box, { backgroundColor: 'green' }]} />
            <View style={[styles.box, { backgroundColor: 'blue' }]} />
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#fff',
        alignItems: 'center',
        justifyContent: 'center',
    },
    header: {
        height: 60,
        backgroundColor: '#f0f0f0',
        justifyContent: 'center',
        alignItems: 'center',
    },
    content: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    footer: {
        height: 60,
        backgroundColor: '#f0f0f0',
        justifyContent: 'center',
        alignItems: 'center',
    },
    flexContainer: {
        flex: 1,
        flexDirection: 'row',
        justifyContent: 'space-around',
        alignItems: 'center',
    },
    box: {
        width: 50,
        height: 50,
    },
});
```

**Text Component**:

```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

// Temel Text kullanımı
function BasicText() {
    return (
        <View style={styles.container}>
            <Text>Hello World!</Text>
            <Text style={styles.boldText}>Bold Text</Text>
            <Text style={styles.italicText}>Italic Text</Text>
            <Text style={styles.coloredText}>Colored Text</Text>
        </View>
    );
}

// Nested Text
function NestedText() {
    return (
        <View style={styles.container}>
            <Text style={styles.paragraph}>
                This is a paragraph with{' '}
                <Text style={styles.bold}>bold text</Text> and{' '}
                <Text style={styles.italic}>italic text</Text>.
            </Text>
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        padding: 20,
    },
    boldText: {
        fontWeight: 'bold',
        fontSize: 18,
    },
    italicText: {
        fontStyle: 'italic',
        fontSize: 16,
    },
    coloredText: {
        color: 'blue',
        fontSize: 16,
    },
    paragraph: {
        fontSize: 16,
        lineHeight: 24,
        textAlign: 'center',
    },
    bold: {
        fontWeight: 'bold',
    },
    italic: {
        fontStyle: 'italic',
    },
});
```

### 4.3 React Native Versiyonları

**React Native 0.1 (2015)**:
- **Temel component'ler**: View, Text, Image, ScrollView
- **iOS support**: İlk iOS desteği
- **Basic styling**: StyleSheet ve Flexbox
- **Hot reloading**: Geliştirme sırasında anında güncelleme
- **React Native CLI**: Command line interface

**React Native 0.2 (2015)**:
- **Android support**: Android platform desteği
- **Navigator sistemi**: Navigation component'i
- **AsyncStorage**: Yerel veri saklama
- **NetInfo**: Ağ durumu bilgisi
- **Platform-specific code**: Platform özel kod desteği

**React Native 0.3 (2015)**:
- **TouchableOpacity**: Dokunulabilir component'ler
- **TouchableHighlight**: Vurgulanabilir component'ler
- **Alert**: Alert dialog'ları
- **Modal**: Modal component'i
- **ActivityIndicator**: Yükleme göstergesi

**React Native 0.4 (2015)**:
- **ListView**: Liste component'i
- **Switch**: Switch component'i
- **Slider**: Slider component'i
- **Picker**: Picker component'i
- **ProgressBarAndroid**: Android progress bar

**React Native 0.5 (2015)**:
- **TextInput**: Metin girişi component'i
- **Button**: Buton component'i
- **RefreshControl**: Pull-to-refresh
- **StatusBar**: Status bar kontrolü
- **KeyboardAvoidingView**: Klavye kaçınma

**React Native 0.6 (2015)**:
- **FlatList**: Performanslı liste component'i
- **SectionList**: Bölümlü liste component'i
- **VirtualizedList**: Sanal liste component'i
- **ImageBackground**: Arka plan resmi
- **SafeAreaView**: Güvenli alan görünümü

**React Native 0.7 (2015)**:
- **Animated**: Animasyon API'si
- **LayoutAnimation**: Layout animasyonları
- **PanResponder**: Dokunma yanıtlayıcısı
- **Dimensions**: Ekran boyutları
- **PixelRatio**: Pixel oranı

**React Native 0.8 (2015)**:
- **CameraRoll**: Kamera roll erişimi
- **ImagePicker**: Resim seçici
- **Linking**: Deep linking
- **Share**: Paylaşım API'si
- **Clipboard**: Pano API'si

**React Native 0.9 (2015)**:
- **WebView**: Web görünümü
- **MapView**: Harita görünümü
- **Geolocation**: Konum servisleri
- **PushNotification**: Push bildirimleri
- **DeviceInfo**: Cihaz bilgileri

**React Native 0.10 (2015)**:
- **SwipeableRow**: Kaydırılabilir satır
- **DrawerLayoutAndroid**: Android drawer
- **ToolbarAndroid**: Android toolbar
- **ViewPagerAndroid**: Android view pager
- **DatePickerAndroid**: Android tarih seçici

**React Native 0.11 (2015)**:
- **TimePickerAndroid**: Android saat seçici
- **ToastAndroid**: Android toast
- **PermissionsAndroid**: Android izinler
- **BackHandler**: Geri tuşu işleme
- **AppState**: Uygulama durumu

**React Native 0.12 (2015)**:
- **Vibration**: Titreşim API'si
- **Sound**: Ses API'si
- **Video**: Video component'i
- **BlurView**: Bulanık görünüm
- **LinearGradient**: Doğrusal gradyan

**React Native 0.13 (2015)**:
- **RadialGradient**: Radyal gradyan
- **MaskedView**: Maskelenmiş görünüm
- **Svg**: SVG desteği
- **Lottie**: Lottie animasyonları
- **Reanimated**: Gelişmiş animasyonlar

**React Native 0.14 (2015)**:
- **Fast Refresh**: Hızlı yenileme
- **Flipper integration**: Flipper entegrasyonu
- **Hermes engine**: Hermes JavaScript engine
- **New Architecture preview**: Yeni mimari önizleme
- **TurboModules**: Turbo modüller

**React Native 0.15 (2015)**:
- **Fabric renderer**: Fabric renderer
- **JSI**: JavaScript Interface
- **Codegen**: Otomatik kod üretimi
- **Performance improvements**: Performans iyileştirmeleri
- **Better debugging**: Daha iyi hata ayıklama

**React Native 0.16 (2015)**:
- **Auto-linking**: Otomatik bağlama
- **CocoaPods integration**: CocoaPods entegrasyonu
- **Metro bundler**: Metro bundler
- **Better TypeScript support**: Gelişmiş TypeScript desteği
- **Improved error messages**: Daha iyi hata mesajları

**React Native 0.17 (2015)**:
- **New Architecture rollout**: Yeni mimari yayılımı
- **Fabric stabilizasyonu**: Fabric stabilizasyonu
- **TurboModules stabilizasyonu**: TurboModules stabilizasyonu
- **Better performance**: Daha iyi performans
- **Enhanced DevTools**: Gelişmiş geliştirici araçları

**React Native 0.18 (2015)**:
- **Legacy architecture deprecation**: Eski mimari kullanımdan kaldırma
- **Stable JavaScript API**: Kararlı JavaScript API'si
- **TypeScript first**: TypeScript öncelikli
- **Better memory management**: Daha iyi bellek yönetimi
- **Improved hot reloading**: Gelişmiş hot reloading

**React Native 0.19 (2015)**:
- **Cross-platform desktop support**: Çapraz platform masaüstü desteği
- **WebAssembly support**: WebAssembly desteği
- **Enhanced new architecture**: Gelişmiş yeni mimari
- **Better error handling**: Daha iyi hata yönetimi
- **Performance monitoring**: Performans izleme

**React Native 0.20 (2015)**:
- **React 19 support**: React 19 desteği
- **Expo Atlas integration**: Expo Atlas entegrasyonu
- **Bundle size optimization**: Paket boyutu optimizasyonu
- **Better testing tools**: Daha iyi test araçları
- **Enhanced documentation**: Gelişmiş dokümantasyon

**React Native 0.21-0.40**:
- Android desteği iyileştirmeleri
- Performance optimizasyonları
- Yeni component'ler

**React Native 0.41-0.60**:
- Metro bundler
- Flipper integration
- Hermes JavaScript engine
- New Architecture (Fabric/TurboModules) hazırlıkları

**React Native 0.61-0.70**:
- Auto-linking
- CocoaPods integration
- Fast Refresh
- Hermes iyileştirmeleri

**React Native 0.71+ (2023-2024)**:
- **New Architecture Rollout**: Fabric renderer ve TurboModules
- **Fabric Renderer**: Yeni rendering sistemi
- **TurboModules**: Native modül sistemi
- **JSI (JavaScript Interface)**: JavaScript ve native kod arası iletişim
- **Codegen**: Otomatik native kod üretimi

**React Native 0.75+ (2024-2025)**:
- **Improved New Architecture**: Fabric ve TurboModules stabilizasyonu
- **Better Performance**: Render performansı iyileştirmeleri
- **Enhanced DevTools**: Gelişmiş geliştirici araçları
- **Better TypeScript Support**: Gelişmiş TypeScript desteği
- **Improved Metro**: Metro bundler iyileştirmeleri

**React Native 0.78+ (2025)**:
- **React 19 Support**: React 19 özelliklerinin mobil platformlara entegrasyonu
- **Legacy Architecture Deprecation**: Eski mimarinin kaldırılması
- **Stable JavaScript API**: Kararlı JavaScript API'si
- **TypeScript First**: Varsayılan TypeScript desteği
- **Expo Atlas Integration**: Bundle boyutu analiz aracı entegrasyonu
- **WebAssembly Support**: WebAssembly ile performans optimizasyonu
- **Enhanced New Architecture**: Gelişmiş yeni mimari
- **Better Memory Management**: Gelişmiş bellek yönetimi
- **Improved Hot Reloading**: Gelişmiş hot reloading
- **Cross-Platform Desktop Support**: Masaüstü platform desteği

### 4.4 React Native Kurulum ve Geliştirme Ortamı

#### 4.4.1 Geliştirme Ortamı Kurulumu

**Gerekli Araçlar:**

1. **Node.js (v18 veya üzeri)**
```bash
# Node.js versiyonunu kontrol et
node --version
npm --version

# nvm ile Node.js yönetimi
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
```

2. **React Native CLI**
```bash
# React Native CLI'yi global olarak yükle
npm install -g @react-native-community/cli

# Versiyonu kontrol et
npx react-native --version
```

3. **Expo CLI (Opsiyonel)**
```bash
# Expo CLI'yi yükle
npm install -g @expo/cli

# Expo versiyonunu kontrol et
expo --version
```

**Android Geliştirme Ortamı:**

1. **Android Studio Kurulumu**
```bash
# Android Studio'yu indir ve kur
# https://developer.android.com/studio

# Android SDK'yı yapılandır
# Android Studio > SDK Manager > SDK Platforms
# Android 13 (API 33) veya üzeri seç

# Environment variables ayarla
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

2. **Android Emulator**
```bash
# Emulator listesini görüntüle
emulator -list-avds

# Emulator'ü başlat
emulator -avd Pixel_4_API_33
```

**iOS Geliştirme Ortamı (macOS):**

1. **Xcode Kurulumu**
```bash
# Xcode'u App Store'dan indir
# Xcode Command Line Tools'u yükle
xcode-select --install

# iOS Simulator'ü başlat
open -a Simulator
```

#### 4.4.2 Yeni Proje Oluşturma

**React Native CLI ile Proje Oluşturma:**
```bash
# Yeni React Native projesi oluştur
npx react-native init MyReactNativeApp

# Proje dizinine git
cd MyReactNativeApp

# iOS için bağımlılıkları yükle (macOS)
cd ios && pod install && cd ..

# Android için build
npx react-native run-android

# iOS için build (macOS)
npx react-native run-ios
```

**Expo ile Proje Oluşturma:**
```bash
# Expo projesi oluştur
npx create-expo-app MyExpoApp

# Proje dizinine git
cd MyExpoApp

# Geliştirme sunucusunu başlat
npx expo start

# Expo Go uygulaması ile QR kodu tarat
```

**Proje Yapısı:**
```
MyReactNativeApp/
├── android/                 # Android native kodları
├── ios/                     # iOS native kodları
├── src/                     # Kaynak kodlar
│   ├── components/          # React Native component'leri
│   ├── screens/             # Ekran component'leri
│   ├── navigation/          # Navigation yapılandırması
│   ├── services/            # API servisleri
│   ├── utils/               # Yardımcı fonksiyonlar
│   └── styles/              # Stil dosyaları
├── App.js                   # Ana uygulama dosyası
├── index.js                 # Giriş noktası
├── package.json             # Proje bağımlılıkları
└── metro.config.js          # Metro bundler yapılandırması
```

### 4.5 Navigation (Navigasyon)

#### 4.5.1 React Navigation Kurulumu

```bash
# React Navigation'ı yükle
npm install @react-navigation/native

# Navigation bağımlılıklarını yükle
npm install react-native-screens react-native-safe-area-context

# iOS için ek bağımlılık
cd ios && pod install && cd ..
```

#### 4.5.2 Stack Navigator

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import HomeScreen from './src/screens/HomeScreen';
import ProfileScreen from './src/screens/ProfileScreen';
import SettingsScreen from './src/screens/SettingsScreen';

const Stack = createStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: {
            backgroundColor: '#2196F3',
          },
          headerTintColor: '#fff',
          headerTitleStyle: {
            fontWeight: 'bold',
          },
        }}
      >
        <Stack.Screen 
          name="Home" 
          component={HomeScreen}
          options={{ title: 'Ana Sayfa' }}
        />
        <Stack.Screen 
          name="Profile" 
          component={ProfileScreen}
          options={{ title: 'Profil' }}
        />
        <Stack.Screen 
          name="Settings" 
          component={SettingsScreen}
          options={{ title: 'Ayarlar' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

export default App;
```

```jsx
// src/screens/HomeScreen.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

function HomeScreen({ navigation }) {
  const navigateToProfile = () => {
    navigation.navigate('Profile', { 
      userId: 123,
      userName: 'John Doe' 
    });
  };

  const navigateToSettings = () => {
    navigation.navigate('Settings');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Ana Sayfa</Text>
      
      <TouchableOpacity 
        style={styles.button}
        onPress={navigateToProfile}
      >
        <Text style={styles.buttonText}>Profile Git</Text>
      </TouchableOpacity>
      
      <TouchableOpacity 
        style={styles.button}
        onPress={navigateToSettings}
      >
        <Text style={styles.buttonText}>Ayarlara Git</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    marginVertical: 10,
    minWidth: 200,
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default HomeScreen;
```

```jsx
// src/screens/ProfileScreen.js
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

function ProfileScreen({ route, navigation }) {
  const { userId, userName } = route.params;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Profil Sayfası</Text>
      <Text style={styles.info}>Kullanıcı ID: {userId}</Text>
      <Text style={styles.info}>Kullanıcı Adı: {userName}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  info: {
    fontSize: 16,
    marginVertical: 10,
  },
});

export default ProfileScreen;
```

#### 4.5.3 Tab Navigator

```bash
# Tab Navigator'ı yükle
npm install @react-navigation/bottom-tabs
```

```jsx
// src/navigation/TabNavigator.js
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Ionicons } from '@expo/vector-icons';
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';

const Tab = createBottomTabNavigator();

function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;

          if (route.name === 'Home') {
            iconName = focused ? 'home' : 'home-outline';
          } else if (route.name === 'Profile') {
            iconName = focused ? 'person' : 'person-outline';
          } else if (route.name === 'Settings') {
            iconName = focused ? 'settings' : 'settings-outline';
          }

          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#2196F3',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen 
        name="Home" 
        component={HomeScreen}
        options={{ title: 'Ana Sayfa' }}
      />
      <Tab.Screen 
        name="Profile" 
        component={ProfileScreen}
        options={{ title: 'Profil' }}
      />
      <Tab.Screen 
        name="Settings" 
        component={SettingsScreen}
        options={{ title: 'Ayarlar' }}
      />
    </Tab.Navigator>
  );
}

export default TabNavigator;
```

#### 4.5.4 Drawer Navigator

```bash
# Drawer Navigator'ı yükle
npm install @react-navigation/drawer
npm install react-native-gesture-handler react-native-reanimated
```

```jsx
// src/navigation/DrawerNavigator.js
import React from 'react';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { Ionicons } from '@expo/vector-icons';
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';

const Drawer = createDrawerNavigator();

function DrawerNavigator() {
  return (
    <Drawer.Navigator
      screenOptions={({ route }) => ({
        drawerIcon: ({ focused, color, size }) => {
          let iconName;

          if (route.name === 'Home') {
            iconName = focused ? 'home' : 'home-outline';
          } else if (route.name === 'Profile') {
            iconName = focused ? 'person' : 'person-outline';
          } else if (route.name === 'Settings') {
            iconName = focused ? 'settings' : 'settings-outline';
          }

          return <Ionicons name={iconName} size={size} color={color} />;
        },
        drawerActiveTintColor: '#2196F3',
        drawerInactiveTintColor: 'gray',
      })}
    >
      <Drawer.Screen 
        name="Home" 
        component={HomeScreen}
        options={{ title: 'Ana Sayfa' }}
      />
      <Drawer.Screen 
        name="Profile" 
        component={ProfileScreen}
        options={{ title: 'Profil' }}
      />
      <Drawer.Screen 
        name="Settings" 
        component={SettingsScreen}
        options={{ title: 'Ayarlar' }}
      />
    </Drawer.Navigator>
  );
}

export default DrawerNavigator;
```

### 4.6 State Management (Durum Yönetimi)

#### 4.6.1 Context API

```jsx
// src/context/UserContext.js
import React, { createContext, useContext, useReducer } from 'react';

// Context oluştur
const UserContext = createContext();

// Action types
const USER_ACTIONS = {
  LOGIN: 'LOGIN',
  LOGOUT: 'LOGOUT',
  UPDATE_PROFILE: 'UPDATE_PROFILE',
};

// Initial state
const initialState = {
  user: null,
  isAuthenticated: false,
  loading: false,
};

// Reducer
function userReducer(state, action) {
  switch (action.type) {
    case USER_ACTIONS.LOGIN:
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true,
        loading: false,
      };
    case USER_ACTIONS.LOGOUT:
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        loading: false,
      };
    case USER_ACTIONS.UPDATE_PROFILE:
      return {
        ...state,
        user: { ...state.user, ...action.payload },
      };
    default:
      return state;
  }
}

// Provider component
export function UserProvider({ children }) {
  const [state, dispatch] = useReducer(userReducer, initialState);

  const login = (userData) => {
    dispatch({ type: USER_ACTIONS.LOGIN, payload: userData });
  };

  const logout = () => {
    dispatch({ type: USER_ACTIONS.LOGOUT });
  };

  const updateProfile = (profileData) => {
    dispatch({ type: USER_ACTIONS.UPDATE_PROFILE, payload: profileData });
  };

  const value = {
    ...state,
    login,
    logout,
    updateProfile,
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook
export function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}
```

```jsx
// App.js - Context Provider ile sarmalama
import React from 'react';
import { UserProvider } from './src/context/UserContext';
import NavigationContainer from './src/navigation/NavigationContainer';

function App() {
  return (
    <UserProvider>
      <NavigationContainer />
    </UserProvider>
  );
}

export default App;
```

```jsx
// src/screens/LoginScreen.js
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useUser } from '../context/UserContext';

function LoginScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { login } = useUser();

  const handleLogin = () => {
    if (email && password) {
      // Simüle edilmiş login
      const userData = {
        id: 1,
        email: email,
        name: 'John Doe',
        avatar: 'https://example.com/avatar.jpg',
      };
      
      login(userData);
      navigation.navigate('Home');
    } else {
      Alert.alert('Hata', 'Lütfen tüm alanları doldurun');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Giriş Yap</Text>
      
      <TextInput
        style={styles.input}
        placeholder="E-posta"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />
      
      <TextInput
        style={styles.input}
        placeholder="Şifre"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      
      <TouchableOpacity style={styles.button} onPress={handleLogin}>
        <Text style={styles.buttonText}>Giriş Yap</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 15,
    borderRadius: 8,
    marginBottom: 15,
    fontSize: 16,
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    marginTop: 10,
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default LoginScreen;
```

#### 4.6.2 Redux Toolkit

```bash
# Redux Toolkit'ı yükle
npm install @reduxjs/toolkit react-redux
```

```jsx
// src/store/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk for login
export const loginUser = createAsyncThunk(
  'user/login',
  async (credentials, { rejectWithValue }) => {
    try {
      // API call simulation
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(credentials),
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }
      
      const userData = await response.json();
      return userData;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    user: null,
    isAuthenticated: false,
    loading: false,
    error: null,
  },
  reducers: {
    logout: (state) => {
      state.user = null;
      state.isAuthenticated = false;
      state.error = null;
    },
    updateProfile: (state, action) => {
      if (state.user) {
        state.user = { ...state.user, ...action.payload };
      }
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
        state.isAuthenticated = true;
        state.error = null;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});

export const { logout, updateProfile, clearError } = userSlice.actions;
export default userSlice.reducer;
```

```jsx
// src/store/index.js
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```jsx
// App.js - Redux Provider ile sarmalama
import React from 'react';
import { Provider } from 'react-redux';
import { store } from './src/store';
import NavigationContainer from './src/navigation/NavigationContainer';

function App() {
  return (
    <Provider store={store}>
      <NavigationContainer />
    </Provider>
  );
}

export default App;
```

```jsx
// src/screens/LoginScreen.js - Redux kullanımı
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useDispatch, useSelector } from 'react-redux';
import { loginUser } from '../store/userSlice';

function LoginScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const dispatch = useDispatch();
  const { loading, error } = useSelector((state) => state.user);

  const handleLogin = () => {
    if (email && password) {
      dispatch(loginUser({ email, password }))
        .unwrap()
        .then(() => {
          navigation.navigate('Home');
        })
        .catch((error) => {
          Alert.alert('Hata', error);
        });
    } else {
      Alert.alert('Hata', 'Lütfen tüm alanları doldurun');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Giriş Yap</Text>
      
      {error && <Text style={styles.errorText}>{error}</Text>}
      
      <TextInput
        style={styles.input}
        placeholder="E-posta"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />
      
      <TextInput
        style={styles.input}
        placeholder="Şifre"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      
      <TouchableOpacity 
        style={[styles.button, loading && styles.buttonDisabled]} 
        onPress={handleLogin}
        disabled={loading}
      >
        <Text style={styles.buttonText}>
          {loading ? 'Giriş yapılıyor...' : 'Giriş Yap'}
        </Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 15,
    borderRadius: 8,
    marginBottom: 15,
    fontSize: 16,
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    marginTop: 10,
  },
  buttonDisabled: {
    backgroundColor: '#ccc',
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
  errorText: {
    color: 'red',
    textAlign: 'center',
    marginBottom: 15,
  },
});

export default LoginScreen;
```

### 4.7 API ve Veri Yönetimi

#### 4.7.1 Fetch API

```jsx
// src/services/api.js
const BASE_URL = 'https://jsonplaceholder.typicode.com';

class ApiService {
  async request(endpoint, options = {}) {
    const url = `${BASE_URL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
      ...options,
    };

    try {
      const response = await fetch(url, config);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const data = await response.json();
      return data;
    } catch (error) {
      console.error('API request failed:', error);
      throw error;
    }
  }

  // GET request
  async get(endpoint) {
    return this.request(endpoint, { method: 'GET' });
  }

  // POST request
  async post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  // PUT request
  async put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  // DELETE request
  async delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

export const apiService = new ApiService();
```

```jsx
// src/services/userService.js
import { apiService } from './api';

export const userService = {
  // Tüm kullanıcıları getir
  async getUsers() {
    return apiService.get('/users');
  },

  // Belirli bir kullanıcıyı getir
  async getUser(id) {
    return apiService.get(`/users/${id}`);
  },

  // Yeni kullanıcı oluştur
  async createUser(userData) {
    return apiService.post('/users', userData);
  },

  // Kullanıcıyı güncelle
  async updateUser(id, userData) {
    return apiService.put(`/users/${id}`, userData);
  },

  // Kullanıcıyı sil
  async deleteUser(id) {
    return apiService.delete(`/users/${id}`);
  },
};
```

```jsx
// src/screens/UserListScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
  Alert,
} from 'react-native';
import { userService } from '../services/userService';

function UserListScreen({ navigation }) {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    try {
      setLoading(true);
      setError(null);
      const userData = await userService.getUsers();
      setUsers(userData);
    } catch (err) {
      setError('Kullanıcılar yüklenirken hata oluştu');
      console.error('Error fetching users:', err);
    } finally {
      setLoading(false);
    }
  };

  const handleUserPress = (user) => {
    navigation.navigate('UserDetail', { userId: user.id });
  };

  const renderUser = ({ item }) => (
    <TouchableOpacity
      style={styles.userItem}
      onPress={() => handleUserPress(item)}
    >
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userEmail}>{item.email}</Text>
      <Text style={styles.userPhone}>{item.phone}</Text>
    </TouchableOpacity>
  );

  if (loading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#2196F3" />
        <Text style={styles.loadingText}>Kullanıcılar yükleniyor...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.errorText}>{error}</Text>
        <TouchableOpacity style={styles.retryButton} onPress={fetchUsers}>
          <Text style={styles.retryButtonText}>Tekrar Dene</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={users}
        renderItem={renderUser}
        keyExtractor={(item) => item.id.toString()}
        refreshing={loading}
        onRefresh={fetchUsers}
        style={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  list: {
    flex: 1,
  },
  userItem: {
    backgroundColor: 'white',
    padding: 15,
    marginVertical: 5,
    marginHorizontal: 10,
    borderRadius: 8,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  userName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  userEmail: {
    fontSize: 14,
    color: '#666',
    marginTop: 5,
  },
  userPhone: {
    fontSize: 14,
    color: '#666',
    marginTop: 2,
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorText: {
    fontSize: 16,
    color: 'red',
    textAlign: 'center',
    marginBottom: 20,
  },
  retryButton: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default UserListScreen;
```

#### 4.7.2 AsyncStorage

```bash
# AsyncStorage'ı yükle
npm install @react-native-async-storage/async-storage
```

```jsx
// src/services/storageService.js
import AsyncStorage from '@react-native-async-storage/async-storage';

class StorageService {
  // Veri kaydet
  async setItem(key, value) {
    try {
      const jsonValue = JSON.stringify(value);
      await AsyncStorage.setItem(key, jsonValue);
      return true;
    } catch (error) {
      console.error('Error saving data:', error);
      return false;
    }
  }

  // Veri getir
  async getItem(key) {
    try {
      const jsonValue = await AsyncStorage.getItem(key);
      return jsonValue != null ? JSON.parse(jsonValue) : null;
    } catch (error) {
      console.error('Error reading data:', error);
      return null;
    }
  }

  // Veri sil
  async removeItem(key) {
    try {
      await AsyncStorage.removeItem(key);
      return true;
    } catch (error) {
      console.error('Error removing data:', error);
      return false;
    }
  }

  // Tüm verileri temizle
  async clear() {
    try {
      await AsyncStorage.clear();
      return true;
    } catch (error) {
      console.error('Error clearing storage:', error);
      return false;
    }
  }

  // Tüm anahtarları getir
  async getAllKeys() {
    try {
      return await AsyncStorage.getAllKeys();
    } catch (error) {
      console.error('Error getting keys:', error);
      return [];
    }
  }

  // Birden fazla veri getir
  async getMultiple(keys) {
    try {
      const values = await AsyncStorage.multiGet(keys);
      return values.map(([key, value]) => [
        key,
        value ? JSON.parse(value) : null,
      ]);
    } catch (error) {
      console.error('Error getting multiple data:', error);
      return [];
    }
  }

  // Birden fazla veri kaydet
  async setMultiple(keyValuePairs) {
    try {
      const pairs = keyValuePairs.map(([key, value]) => [
        key,
        JSON.stringify(value),
      ]);
      await AsyncStorage.multiSet(pairs);
      return true;
    } catch (error) {
      console.error('Error setting multiple data:', error);
      return false;
    }
  }
}

export const storageService = new StorageService();
```

```jsx
// src/hooks/useAsyncStorage.js
import { useState, useEffect } from 'react';
import { storageService } from '../services/storageService';

export function useAsyncStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(initialValue);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadStoredValue();
  }, [key]);

  const loadStoredValue = async () => {
    try {
      const item = await storageService.getItem(key);
      if (item !== null) {
        setStoredValue(item);
      }
    } catch (error) {
      console.error('Error loading stored value:', error);
    } finally {
      setLoading(false);
    }
  };

  const setValue = async (value) => {
    try {
      setStoredValue(value);
      await storageService.setItem(key, value);
    } catch (error) {
      console.error('Error setting value:', error);
    }
  };

  const removeValue = async () => {
    try {
      setStoredValue(initialValue);
      await storageService.removeItem(key);
    } catch (error) {
      console.error('Error removing value:', error);
    }
  };

  return [storedValue, setValue, removeValue, loading];
}
```

```jsx
// src/screens/SettingsScreen.js
import React, { useState } from 'react';
import {
  View,
  Text,
  Switch,
  TouchableOpacity,
  StyleSheet,
  Alert,
} from 'react-native';
import { useAsyncStorage } from '../hooks/useAsyncStorage';

function SettingsScreen() {
  const [notifications, setNotifications] = useAsyncStorage('notifications', true);
  const [darkMode, setDarkMode] = useAsyncStorage('darkMode', false);
  const [language, setLanguage] = useAsyncStorage('language', 'tr');

  const handleClearCache = () => {
    Alert.alert(
      'Önbelleği Temizle',
      'Tüm önbellek verileri silinecek. Emin misiniz?',
      [
        { text: 'İptal', style: 'cancel' },
        {
          text: 'Temizle',
          style: 'destructive',
          onPress: () => {
            // Önbellek temizleme işlemi
            Alert.alert('Başarılı', 'Önbellek temizlendi');
          },
        },
      ]
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Ayarlar</Text>

      <View style={styles.settingItem}>
        <Text style={styles.settingLabel}>Bildirimler</Text>
        <Switch
          value={notifications}
          onValueChange={setNotifications}
          trackColor={{ false: '#767577', true: '#81b0ff' }}
          thumbColor={notifications ? '#2196F3' : '#f4f3f4'}
        />
      </View>

      <View style={styles.settingItem}>
        <Text style={styles.settingLabel}>Karanlık Mod</Text>
        <Switch
          value={darkMode}
          onValueChange={setDarkMode}
          trackColor={{ false: '#767577', true: '#81b0ff' }}
          thumbColor={darkMode ? '#2196F3' : '#f4f3f4'}
        />
      </View>

      <View style={styles.settingItem}>
        <Text style={styles.settingLabel}>Dil: {language}</Text>
        <TouchableOpacity
          style={styles.languageButton}
          onPress={() => {
            const newLanguage = language === 'tr' ? 'en' : 'tr';
            setLanguage(newLanguage);
          }}
        >
          <Text style={styles.languageButtonText}>Değiştir</Text>
        </TouchableOpacity>
      </View>

      <TouchableOpacity style={styles.clearButton} onPress={handleClearCache}>
        <Text style={styles.clearButtonText}>Önbelleği Temizle</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
    textAlign: 'center',
  },
  settingItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 15,
    marginBottom: 10,
    borderRadius: 8,
    elevation: 2,
  },
  settingLabel: {
    fontSize: 16,
    color: '#333',
  },
  languageButton: {
    backgroundColor: '#2196F3',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 5,
  },
  languageButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  clearButton: {
    backgroundColor: '#f44336',
    padding: 15,
    borderRadius: 8,
    marginTop: 30,
  },
  clearButtonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default SettingsScreen;
```

### 4.8 Native Modüller ve Kütüphaneler

#### 4.8.1 Popüler Kütüphaneler

**Vector Icons:**
```bash
# React Native Vector Icons'ı yükle
npm install react-native-vector-icons

# iOS için ek kurulum
cd ios && pod install && cd ..
```

```jsx
// src/components/IconButton.js
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';

function IconButton({ iconName, title, onPress, color = '#2196F3' }) {
  return (
    <TouchableOpacity style={styles.button} onPress={onPress}>
      <Icon name={iconName} size={24} color={color} />
      <Text style={[styles.text, { color }]}>{title}</Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  button: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 10,
    borderRadius: 8,
    backgroundColor: '#f0f0f0',
    marginVertical: 5,
  },
  text: {
    marginLeft: 10,
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default IconButton;
```

**Image Picker:**
```bash
# React Native Image Picker'ı yükle
npm install react-native-image-picker

# iOS için ek kurulum
cd ios && pod install && cd ..
```

```jsx
// src/components/ImagePicker.js
import React, { useState } from 'react';
import { View, Image, TouchableOpacity, Text, StyleSheet, Alert } from 'react-native';
import { launchImageLibrary, launchCamera } from 'react-native-image-picker';

function ImagePicker({ onImageSelected }) {
  const [selectedImage, setSelectedImage] = useState(null);

  const showImagePicker = () => {
    Alert.alert(
      'Resim Seç',
      'Resmi nereden seçmek istiyorsunuz?',
      [
        { text: 'İptal', style: 'cancel' },
        { text: 'Kamera', onPress: openCamera },
        { text: 'Galeri', onPress: openGallery },
      ]
    );
  };

  const openCamera = () => {
    const options = {
      mediaType: 'photo',
      quality: 0.8,
      maxWidth: 1000,
      maxHeight: 1000,
    };

    launchCamera(options, (response) => {
      if (response.didCancel || response.error) {
        return;
      }
      
      if (response.assets && response.assets[0]) {
        const image = response.assets[0];
        setSelectedImage(image);
        onImageSelected && onImageSelected(image);
      }
    });
  };

  const openGallery = () => {
    const options = {
      mediaType: 'photo',
      quality: 0.8,
      maxWidth: 1000,
      maxHeight: 1000,
    };

    launchImageLibrary(options, (response) => {
      if (response.didCancel || response.error) {
        return;
      }
      
      if (response.assets && response.assets[0]) {
        const image = response.assets[0];
        setSelectedImage(image);
        onImageSelected && onImageSelected(image);
      }
    });
  };

  return (
    <View style={styles.container}>
      {selectedImage ? (
        <View style={styles.imageContainer}>
          <Image source={{ uri: selectedImage.uri }} style={styles.image} />
          <TouchableOpacity style={styles.changeButton} onPress={showImagePicker}>
            <Text style={styles.changeButtonText}>Değiştir</Text>
          </TouchableOpacity>
        </View>
      ) : (
        <TouchableOpacity style={styles.selectButton} onPress={showImagePicker}>
          <Text style={styles.selectButtonText}>Resim Seç</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
    marginVertical: 20,
  },
  imageContainer: {
    alignItems: 'center',
  },
  image: {
    width: 200,
    height: 200,
    borderRadius: 100,
    marginBottom: 10,
  },
  selectButton: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    minWidth: 150,
  },
  selectButtonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
  changeButton: {
    backgroundColor: '#4CAF50',
    padding: 10,
    borderRadius: 5,
  },
  changeButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
});

export default ImagePicker;
```

**Permissions:**
```bash
# React Native Permissions'ı yükle
npm install react-native-permissions

# iOS için ek kurulum
cd ios && pod install && cd ..
```

```jsx
// src/services/permissionService.js
import { Platform, Alert } from 'react-native';
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

class PermissionService {
  // Kamera izni kontrol et ve iste
  async requestCameraPermission() {
    const permission = Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.CAMERA 
      : PERMISSIONS.ANDROID.CAMERA;

    const result = await check(permission);
    
    if (result === RESULTS.GRANTED) {
      return true;
    }
    
    if (result === RESULTS.DENIED) {
      const requestResult = await request(permission);
      return requestResult === RESULTS.GRANTED;
    }
    
    if (result === RESULTS.BLOCKED) {
      Alert.alert(
        'İzin Gerekli',
        'Kamera izni ayarlardan etkinleştirilmelidir.',
        [{ text: 'Tamam' }]
      );
      return false;
    }
    
    return false;
  }

  // Galeri izni kontrol et ve iste
  async requestPhotoLibraryPermission() {
    const permission = Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.PHOTO_LIBRARY 
      : PERMISSIONS.ANDROID.READ_EXTERNAL_STORAGE;

    const result = await check(permission);
    
    if (result === RESULTS.GRANTED) {
      return true;
    }
    
    if (result === RESULTS.DENIED) {
      const requestResult = await request(permission);
      return requestResult === RESULTS.GRANTED;
    }
    
    if (result === RESULTS.BLOCKED) {
      Alert.alert(
        'İzin Gerekli',
        'Galeri izni ayarlardan etkinleştirilmelidir.',
        [{ text: 'Tamam' }]
      );
      return false;
    }
    
    return false;
  }

  // Konum izni kontrol et ve iste
  async requestLocationPermission() {
    const permission = Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.LOCATION_WHEN_IN_USE 
      : PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION;

    const result = await check(permission);
    
    if (result === RESULTS.GRANTED) {
      return true;
    }
    
    if (result === RESULTS.DENIED) {
      const requestResult = await request(permission);
      return requestResult === RESULTS.GRANTED;
    }
    
    if (result === RESULTS.BLOCKED) {
      Alert.alert(
        'İzin Gerekli',
        'Konum izni ayarlardan etkinleştirilmelidir.',
        [{ text: 'Tamam' }]
      );
      return false;
    }
    
    return false;
  }
}

export const permissionService = new PermissionService();
```

#### 4.8.2 Native Modül Geliştirme

**Android Native Modül:**
```java
// android/app/src/main/java/com/yourapp/MyNativeModule.java
package com.yourapp;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;

public class MyNativeModule extends ReactContextBaseJavaModule {
    private ReactApplicationContext reactContext;

    public MyNativeModule(ReactApplicationContext reactContext) {
        super(reactContext);
        this.reactContext = reactContext;
    }

    @Override
    public String getName() {
        return "MyNativeModule";
    }

    @ReactMethod
    public void getDeviceInfo(Promise promise) {
        try {
            WritableMap deviceInfo = Arguments.createMap();
            deviceInfo.putString("model", android.os.Build.MODEL);
            deviceInfo.putString("version", android.os.Build.VERSION.RELEASE);
            deviceInfo.putString("manufacturer", android.os.Build.MANUFACTURER);
            
            promise.resolve(deviceInfo);
        } catch (Exception e) {
            promise.reject("ERROR", e.getMessage());
        }
    }

    @ReactMethod
    public void showToast(String message) {
        android.widget.Toast.makeText(
            reactContext,
            message,
            android.widget.Toast.LENGTH_LONG
        ).show();
    }
}
```

```java
// android/app/src/main/java/com/yourapp/MyNativeModulePackage.java
package com.yourapp;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MyNativeModulePackage implements ReactPackage {
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new MyNativeModule(reactContext));
        return modules;
    }
}
```

**iOS Native Modül:**
```objc
// ios/MyNativeModule.h
#import <React/RCTBridgeModule.h>

@interface MyNativeModule : NSObject <RCTBridgeModule>

@end
```

```objc
// ios/MyNativeModule.m
#import "MyNativeModule.h"
#import <UIKit/UIKit.h>

@implementation MyNativeModule

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(getDeviceInfo:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
    @try {
        NSDictionary *deviceInfo = @{
            @"model": [[UIDevice currentDevice] model],
            @"version": [[UIDevice currentDevice] systemVersion],
            @"name": [[UIDevice currentDevice] name]
        };
        resolve(deviceInfo);
    }
    @catch (NSException *exception) {
        reject(@"ERROR", exception.reason, nil);
    }
}

RCT_EXPORT_METHOD(showToast:(NSString *)message)
{
    dispatch_async(dispatch_get_main_queue(), ^{
        UIAlertController *alert = [UIAlertController 
            alertControllerWithTitle:@"Toast" 
            message:message 
            preferredStyle:UIAlertControllerStyleAlert];
        
        UIAlertAction *okAction = [UIAlertAction 
            actionWithTitle:@"OK" 
            style:UIAlertActionStyleDefault 
            handler:nil];
        
        [alert addAction:okAction];
        
        UIViewController *rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
        [rootViewController presentViewController:alert animated:YES completion:nil];
    });
}

@end
```

**JavaScript Interface:**
```jsx
// src/services/nativeModuleService.js
import { NativeModules } from 'react-native';

const { MyNativeModule } = NativeModules;

class NativeModuleService {
  // Cihaz bilgilerini getir
  async getDeviceInfo() {
    try {
      const deviceInfo = await MyNativeModule.getDeviceInfo();
      return deviceInfo;
    } catch (error) {
      console.error('Error getting device info:', error);
      throw error;
    }
  }

  // Toast mesajı göster
  showToast(message) {
    try {
      MyNativeModule.showToast(message);
    } catch (error) {
      console.error('Error showing toast:', error);
    }
  }
}

export const nativeModuleService = new NativeModuleService();
```

```jsx
// src/screens/DeviceInfoScreen.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { nativeModuleService } from '../services/nativeModuleService';

function DeviceInfoScreen() {
  const [deviceInfo, setDeviceInfo] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadDeviceInfo();
  }, []);

  const loadDeviceInfo = async () => {
    try {
      setLoading(true);
      const info = await nativeModuleService.getDeviceInfo();
      setDeviceInfo(info);
    } catch (error) {
      console.error('Error loading device info:', error);
    } finally {
      setLoading(false);
    }
  };

  const showToast = () => {
    nativeModuleService.showToast('Merhaba React Native!');
  };

  if (loading) {
    return (
      <View style={styles.container}>
        <Text style={styles.loadingText}>Cihaz bilgileri yükleniyor...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Cihaz Bilgileri</Text>
      
      {deviceInfo && (
        <View style={styles.infoContainer}>
          <Text style={styles.infoText}>Model: {deviceInfo.model}</Text>
          <Text style={styles.infoText}>Versiyon: {deviceInfo.version}</Text>
          <Text style={styles.infoText}>Üretici: {deviceInfo.manufacturer}</Text>
        </View>
      )}
      
      <TouchableOpacity style={styles.button} onPress={showToast}>
        <Text style={styles.buttonText}>Toast Göster</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
  },
  loadingText: {
    fontSize: 16,
    color: '#666',
  },
  infoContainer: {
    backgroundColor: '#f0f0f0',
    padding: 20,
    borderRadius: 8,
    marginBottom: 30,
    minWidth: 250,
  },
  infoText: {
    fontSize: 16,
    marginVertical: 5,
    color: '#333',
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    minWidth: 150,
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default DeviceInfoScreen;
```

### 4.9 Styling ve UI

#### 4.9.1 StyleSheet ve Flexbox

```jsx
// src/components/Card.js
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';

function Card({ title, description, onPress, style }) {
  return (
    <TouchableOpacity 
      style={[styles.card, style]} 
      onPress={onPress}
      activeOpacity={0.7}
    >
      <View style={styles.header}>
        <Text style={styles.title}>{title}</Text>
      </View>
      <View style={styles.content}>
        <Text style={styles.description}>{description}</Text>
      </View>
      <View style={styles.footer}>
        <Text style={styles.footerText}>Detayları Gör</Text>
      </View>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 12,
    marginVertical: 8,
    marginHorizontal: 16,
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  header: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  content: {
    padding: 16,
    flex: 1,
  },
  description: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
  footer: {
    padding: 16,
    borderTopWidth: 1,
    borderTopColor: '#f0f0f0',
    alignItems: 'flex-end',
  },
  footerText: {
    fontSize: 14,
    color: '#2196F3',
    fontWeight: '600',
  },
});

export default Card;
```

```jsx
// src/screens/FlexboxExampleScreen.js
import React from 'react';
import { View, Text, StyleSheet, ScrollView } from 'react-native';
import Card from '../components/Card';

function FlexboxExampleScreen() {
  const cards = [
    { id: 1, title: 'Kart 1', description: 'Bu birinci kartın açıklamasıdır.' },
    { id: 2, title: 'Kart 2', description: 'Bu ikinci kartın açıklamasıdır.' },
    { id: 3, title: 'Kart 3', description: 'Bu üçüncü kartın açıklamasıdır.' },
  ];

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Flexbox Örnekleri</Text>
      </View>
      
      {/* Flex Direction Row */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Flex Direction: Row</Text>
        <View style={styles.rowContainer}>
          <View style={[styles.box, { backgroundColor: '#FF6B6B' }]}>
            <Text style={styles.boxText}>1</Text>
          </View>
          <View style={[styles.box, { backgroundColor: '#4ECDC4' }]}>
            <Text style={styles.boxText}>2</Text>
          </View>
          <View style={[styles.box, { backgroundColor: '#45B7D1' }]}>
            <Text style={styles.boxText}>3</Text>
          </View>
        </View>
      </View>

      {/* Justify Content */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Justify Content: Space Between</Text>
        <View style={styles.justifyContainer}>
          <View style={[styles.box, { backgroundColor: '#96CEB4' }]}>
            <Text style={styles.boxText}>A</Text>
          </View>
          <View style={[styles.box, { backgroundColor: '#FFEAA7' }]}>
            <Text style={styles.boxText}>B</Text>
          </View>
          <View style={[styles.box, { backgroundColor: '#DDA0DD' }]}>
            <Text style={styles.boxText}>C</Text>
          </View>
        </View>
      </View>

      {/* Align Items */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Align Items: Center</Text>
        <View style={styles.alignContainer}>
          <View style={[styles.box, { backgroundColor: '#FF7675' }]}>
            <Text style={styles.boxText}>X</Text>
          </View>
          <View style={[styles.box, { backgroundColor: '#74B9FF' }]}>
            <Text style={styles.boxText}>Y</Text>
          </View>
          <View style={[styles.box, { backgroundColor: '#A29BFE' }]}>
            <Text style={styles.boxText}>Z</Text>
          </View>
        </View>
      </View>

      {/* Cards List */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Kartlar</Text>
        {cards.map((card) => (
          <Card
            key={card.id}
            title={card.title}
            description={card.description}
            onPress={() => console.log('Card pressed:', card.id)}
          />
        ))}
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    backgroundColor: '#2196F3',
    padding: 20,
    alignItems: 'center',
  },
  headerTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'white',
  },
  section: {
    marginVertical: 20,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginHorizontal: 16,
    marginBottom: 10,
    color: '#333',
  },
  rowContainer: {
    flexDirection: 'row',
    paddingHorizontal: 16,
  },
  justifyContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingHorizontal: 16,
  },
  alignContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 16,
  },
  box: {
    width: 60,
    height: 60,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 8,
    marginHorizontal: 5,
  },
  boxText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default FlexboxExampleScreen;
```

#### 4.9.2 Responsive Tasarım

```jsx
// src/utils/dimensions.js
import { Dimensions, PixelRatio } from 'react-native';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

// Responsive font size
export const responsiveFontSize = (size) => {
  const scale = SCREEN_WIDTH / 320;
  const newSize = size * scale;
  return Math.round(PixelRatio.roundToNearestPixel(newSize));
};

// Responsive width
export const responsiveWidth = (percentage) => {
  return (SCREEN_WIDTH * percentage) / 100;
};

// Responsive height
export const responsiveHeight = (percentage) => {
  return (SCREEN_HEIGHT * percentage) / 100;
};

// Device type detection
export const isTablet = () => {
  const pixelDensity = PixelRatio.get();
  const adjustedWidth = SCREEN_WIDTH * pixelDensity;
  const adjustedHeight = SCREEN_HEIGHT * pixelDensity;
  
  if (pixelDensity < 2 && (adjustedWidth >= 1000 || adjustedHeight >= 1000)) {
    return true;
  } else if (pixelDensity === 2 && (adjustedWidth >= 1920 || adjustedHeight >= 1920)) {
    return true;
  } else {
    return false;
  }
};

// Screen dimensions
export const screenDimensions = {
  width: SCREEN_WIDTH,
  height: SCREEN_HEIGHT,
  isTablet: isTablet(),
};
```

```jsx
// src/components/ResponsiveCard.js
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { responsiveFontSize, responsiveWidth, responsiveHeight, screenDimensions } from '../utils/dimensions';

function ResponsiveCard({ title, description, onPress }) {
  const isTablet = screenDimensions.isTablet;
  
  return (
    <TouchableOpacity 
      style={[
        styles.card, 
        isTablet && styles.cardTablet
      ]} 
      onPress={onPress}
    >
      <View style={styles.content}>
        <Text style={[
          styles.title,
          isTablet && styles.titleTablet
        ]}>
          {title}
        </Text>
        <Text style={[
          styles.description,
          isTablet && styles.descriptionTablet
        ]}>
          {description}
        </Text>
      </View>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: responsiveFontSize(8),
    marginVertical: responsiveHeight(1),
    marginHorizontal: responsiveWidth(4),
    padding: responsiveWidth(4),
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  cardTablet: {
    marginHorizontal: responsiveWidth(2),
    padding: responsiveWidth(3),
  },
  content: {
    flex: 1,
  },
  title: {
    fontSize: responsiveFontSize(16),
    fontWeight: 'bold',
    color: '#333',
    marginBottom: responsiveHeight(1),
  },
  titleTablet: {
    fontSize: responsiveFontSize(20),
  },
  description: {
    fontSize: responsiveFontSize(14),
    color: '#666',
    lineHeight: responsiveFontSize(20),
  },
  descriptionTablet: {
    fontSize: responsiveFontSize(16),
    lineHeight: responsiveFontSize(24),
  },
});

export default ResponsiveCard;
```

#### 4.9.3 UI Kütüphaneleri

**NativeBase:**
```bash
# NativeBase'ı yükle
npm install native-base react-native-svg
cd ios && pod install && cd ..
```

```jsx
// src/screens/NativeBaseExampleScreen.js
import React, { useState } from 'react';
import {
  Box,
  VStack,
  HStack,
  Text,
  Button,
  Input,
  FormControl,
  Select,
  CheckIcon,
  Switch,
  Slider,
  Progress,
  Spinner,
  AlertDialog,
  Modal,
  useDisclose,
} from 'native-base';

function NativeBaseExampleScreen() {
  const [name, setName] = useState('');
  const [service, setService] = useState('');
  const [isEnabled, setIsEnabled] = useState(false);
  const [sliderValue, setSliderValue] = useState(50);
  const { isOpen, onOpen, onClose } = useDisclose();

  return (
    <Box flex={1} bg="gray.50" p={4}>
      <VStack space={4} flex={1}>
        <Text fontSize="2xl" fontWeight="bold" color="primary.500">
          NativeBase Örnekleri
        </Text>

        {/* Form Controls */}
        <FormControl>
          <FormControl.Label>İsim</FormControl.Label>
          <Input
            placeholder="İsminizi girin"
            value={name}
            onChangeText={setName}
          />
        </FormControl>

        <FormControl>
          <FormControl.Label>Servis Seçin</FormControl.Label>
          <Select
            selectedValue={service}
            minWidth="200"
            accessibilityLabel="Servis seçin"
            placeholder="Servis seçin"
            _selectedItem={{
              bg: "teal.600",
              endIcon: <CheckIcon size="5" />
            }}
            mt={1}
            onValueChange={setService}
          >
            <Select.Item label="UX Research" value="ux" />
            <Select.Item label="Web Development" value="web" />
            <Select.Item label="Cross Platform" value="cross" />
            <Select.Item label="UI Designing" value="ui" />
            <Select.Item label="Backend Development" value="backend" />
          </Select>
        </FormControl>

        {/* Switch */}
        <HStack space={4} alignItems="center">
          <Text>Bildirimler</Text>
          <Switch
            isChecked={isEnabled}
            onToggle={setIsEnabled}
            colorScheme="primary"
          />
        </HStack>

        {/* Slider */}
        <VStack space={2}>
          <Text>Slider Değeri: {sliderValue}</Text>
          <Slider
            value={sliderValue}
            onChange={setSliderValue}
            colorScheme="primary"
          >
            <Slider.Track>
              <Slider.FilledTrack />
            </Slider.Track>
            <Slider.Thumb />
          </Slider>
        </VStack>

        {/* Progress */}
        <VStack space={2}>
          <Text>Progress Bar</Text>
          <Progress value={sliderValue} colorScheme="primary" />
        </VStack>

        {/* Buttons */}
        <HStack space={4}>
          <Button colorScheme="primary" onPress={onOpen}>
            Modal Aç
          </Button>
          <Button colorScheme="secondary" variant="outline">
            İkincil Buton
          </Button>
        </HStack>

        {/* Spinner */}
        <HStack space={4} alignItems="center">
          <Text>Yükleniyor...</Text>
          <Spinner color="primary.500" />
        </HStack>
      </VStack>

      {/* Modal */}
      <Modal isOpen={isOpen} onClose={onClose}>
        <Modal.Content maxWidth="400px">
          <Modal.CloseButton />
          <Modal.Header>Modal Başlığı</Modal.Header>
          <Modal.Body>
            <Text>
              Bu bir NativeBase modal örneğidir. Modal içeriği burada görüntülenir.
            </Text>
          </Modal.Body>
          <Modal.Footer>
            <Button.Group space={2}>
              <Button variant="ghost" colorScheme="blueGray" onPress={onClose}>
                İptal
              </Button>
              <Button onPress={onClose}>
                Tamam
              </Button>
            </Button.Group>
          </Modal.Footer>
        </Modal.Content>
      </Modal>
    </Box>
  );
}

export default NativeBaseExampleScreen;
```

### 4.10 Performance Optimizasyonu

#### 4.10.1 FlatList Optimizasyonu

```jsx
// src/components/OptimizedFlatList.js
import React, { useCallback, useMemo } from 'react';
import { FlatList, View, Text, StyleSheet, Image } from 'react-native';

function OptimizedFlatList({ data }) {
  // Memoized render item
  const renderItem = useCallback(({ item, index }) => (
    <ListItem item={item} index={index} />
  ), []);

  // Memoized key extractor
  const keyExtractor = useCallback((item) => item.id.toString(), []);

  // Memoized get item layout
  const getItemLayout = useCallback((data, index) => ({
    length: 80,
    offset: 80 * index,
    index,
  }), []);

  // Memoized list footer
  const ListFooterComponent = useMemo(() => (
    <View style={styles.footer}>
      <Text style={styles.footerText}>Liste sonu</Text>
    </View>
  ), []);

  // Memoized list header
  const ListHeaderComponent = useMemo(() => (
    <View style={styles.header}>
      <Text style={styles.headerText}>Liste başlığı</Text>
    </View>
  ), []);

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      ListHeaderComponent={ListHeaderComponent}
      ListFooterComponent={ListFooterComponent}
      // Performance optimizations
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      updateCellsBatchingPeriod={50}
      initialNumToRender={10}
      windowSize={10}
      // Pull to refresh
      refreshing={false}
      onRefresh={() => {}}
      // Infinite scroll
      onEndReached={() => {}}
      onEndReachedThreshold={0.5}
    />
  );
}

// Memoized list item component
const ListItem = React.memo(({ item, index }) => (
  <View style={styles.listItem}>
    <Image source={{ uri: item.image }} style={styles.image} />
    <View style={styles.content}>
      <Text style={styles.title}>{item.title}</Text>
      <Text style={styles.subtitle}>{item.subtitle}</Text>
    </View>
  </View>
));

const styles = StyleSheet.create({
  listItem: {
    flexDirection: 'row',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
    height: 80,
  },
  image: {
    width: 50,
    height: 50,
    borderRadius: 25,
    marginRight: 15,
  },
  content: {
    flex: 1,
    justifyContent: 'center',
  },
  title: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    marginTop: 2,
  },
  header: {
    padding: 20,
    backgroundColor: '#f0f0f0',
    alignItems: 'center',
  },
  headerText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  footer: {
    padding: 20,
    alignItems: 'center',
  },
  footerText: {
    fontSize: 14,
    color: '#666',
  },
});

export default OptimizedFlatList;
```

#### 4.10.2 Image Optimizasyonu

```jsx
// src/components/OptimizedImage.js
import React, { useState, useCallback } from 'react';
import { Image, View, StyleSheet, ActivityIndicator } from 'react-native';

function OptimizedImage({ 
  source, 
  style, 
  placeholder = null,
  resizeMode = 'cover',
  ...props 
}) {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  const handleLoadStart = useCallback(() => {
    setLoading(true);
    setError(false);
  }, []);

  const handleLoadEnd = useCallback(() => {
    setLoading(false);
  }, []);

  const handleError = useCallback(() => {
    setLoading(false);
    setError(true);
  }, []);

  return (
    <View style={[styles.container, style]}>
      <Image
        source={source}
        style={[styles.image, style]}
        resizeMode={resizeMode}
        onLoadStart={handleLoadStart}
        onLoadEnd={handleLoadEnd}
        onError={handleError}
        {...props}
      />
      
      {loading && (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="small" color="#2196F3" />
        </View>
      )}
      
      {error && (
        <View style={styles.errorContainer}>
          {placeholder || (
            <View style={styles.defaultPlaceholder}>
              <Text style={styles.errorText}>Resim yüklenemedi</Text>
            </View>
          )}
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    position: 'relative',
  },
  image: {
    width: '100%',
    height: '100%',
  },
  loadingContainer: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
  },
  errorContainer: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
  },
  defaultPlaceholder: {
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#e0e0e0',
    borderRadius: 4,
  },
  errorText: {
    color: '#666',
    fontSize: 12,
    textAlign: 'center',
  },
});

export default OptimizedImage;
```

#### 4.10.3 Memory Management

```jsx
// src/hooks/useMemoryOptimization.js
import { useEffect, useRef, useCallback } from 'react';

export function useMemoryOptimization() {
  const timers = useRef(new Set());
  const listeners = useRef(new Set());

  // Timer cleanup
  const addTimer = useCallback((timer) => {
    timers.current.add(timer);
    return timer;
  }, []);

  const clearTimer = useCallback((timer) => {
    if (timer) {
      clearTimeout(timer);
      clearInterval(timer);
      timers.current.delete(timer);
    }
  }, []);

  // Event listener cleanup
  const addListener = useCallback((listener) => {
    listeners.current.add(listener);
    return listener;
  }, []);

  const removeListener = useCallback((listener) => {
    if (listener && listener.remove) {
      listener.remove();
      listeners.current.delete(listener);
    }
  }, []);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      // Clear all timers
      timers.current.forEach(clearTimer);
      timers.current.clear();

      // Remove all listeners
      listeners.current.forEach(removeListener);
      listeners.current.clear();
    };
  }, [clearTimer, removeListener]);

  return {
    addTimer,
    clearTimer,
    addListener,
    removeListener,
  };
}
```

```jsx
// src/screens/MemoryOptimizedScreen.js
import React, { useState, useEffect, useCallback } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, Alert } from 'react-native';
import { useMemoryOptimization } from '../hooks/useMemoryOptimization';

function MemoryOptimizedScreen() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState([]);
  const { addTimer, clearTimer, addListener, removeListener } = useMemoryOptimization();

  // Optimized data fetching
  const fetchData = useCallback(async () => {
    try {
      // Simulate API call
      const response = await fetch('https://jsonplaceholder.typicode.com/posts');
      const result = await response.json();
      setData(result.slice(0, 10)); // Limit data to prevent memory issues
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  }, []);

  // Timer example
  const startTimer = useCallback(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);
    
    addTimer(timer);
    
    // Auto stop after 10 seconds
    const stopTimer = setTimeout(() => {
      clearTimer(timer);
      Alert.alert('Timer', 'Timer durduruldu');
    }, 10000);
    
    addTimer(stopTimer);
  }, [addTimer, clearTimer]);

  // Event listener example
  useEffect(() => {
    const handleAppStateChange = (nextAppState) => {
      console.log('App state changed to:', nextAppState);
    };

    const listener = addListener({
      remove: () => {
        // AppState.removeEventListener('change', handleAppStateChange);
      }
    });

    return () => {
      removeListener(listener);
    };
  }, [addListener, removeListener]);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Memory Optimized Screen</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Timer: {count}</Text>
        <TouchableOpacity style={styles.button} onPress={startTimer}>
          <Text style={styles.buttonText}>Timer Başlat</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Data Count: {data.length}</Text>
        <TouchableOpacity style={styles.button} onPress={fetchData}>
          <Text style={styles.buttonText}>Veri Yükle</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Memory Tips:</Text>
        <Text style={styles.tip}>• Büyük listeler için FlatList kullanın</Text>
        <Text style={styles.tip}>• Gereksiz re-render'ları önleyin</Text>
        <Text style={styles.tip}>• Timer'ları ve listener'ları temizleyin</Text>
        <Text style={styles.tip}>• Resimleri optimize edin</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 8,
    marginBottom: 20,
    elevation: 2,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  tip: {
    fontSize: 14,
    color: '#666',
    marginVertical: 2,
  },
});

export default MemoryOptimizedScreen;
```

### 4.11 Testing

#### 4.11.1 Jest Testing

```bash
# Jest ve React Native Testing Library'yi yükle
npm install --save-dev @testing-library/react-native @testing-library/jest-native
```

```jsx
// src/components/__tests__/Button.test.js
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import Button from '../Button';

describe('Button Component', () => {
  it('renders correctly with title', () => {
    const { getByText } = render(<Button title="Test Button" />);
    expect(getByText('Test Button')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const mockOnPress = jest.fn();
    const { getByText } = render(
      <Button title="Test Button" onPress={mockOnPress} />
    );
    
    fireEvent.press(getByText('Test Button'));
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });

  it('applies custom styles', () => {
    const customStyle = { backgroundColor: 'red' };
    const { getByTestId } = render(
      <Button title="Test Button" style={customStyle} testID="button" />
    );
    
    const button = getByTestId('button');
    expect(button.props.style).toContainEqual(customStyle);
  });

  it('shows loading state', () => {
    const { getByText } = render(
      <Button title="Test Button" loading={true} />
    );
    
    expect(getByText('Yükleniyor...')).toBeTruthy();
  });
});
```

```jsx
// src/components/Button.js
import React from 'react';
import { TouchableOpacity, Text, StyleSheet, ActivityIndicator } from 'react-native';

function Button({ title, onPress, style, loading = false, testID }) {
  return (
    <TouchableOpacity
      style={[styles.button, style]}
      onPress={onPress}
      disabled={loading}
      testID={testID}
    >
      {loading ? (
        <ActivityIndicator color="white" />
      ) : (
        <Text style={styles.buttonText}>{title}</Text>
      )}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default Button;
```

#### 4.11.2 Component Testing

```jsx
// src/screens/__tests__/LoginScreen.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';
import LoginScreen from '../LoginScreen';

// Mock navigation
const mockNavigate = jest.fn();
const mockNavigation = {
  navigate: mockNavigate,
};

// Mock user context
jest.mock('../../context/UserContext', () => ({
  useUser: () => ({
    login: jest.fn(),
  }),
}));

const renderWithNavigation = (component) => {
  return render(
    <NavigationContainer>
      {component}
    </NavigationContainer>
  );
};

describe('LoginScreen', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders login form correctly', () => {
    const { getByPlaceholderText, getByText } = renderWithNavigation(
      <LoginScreen navigation={mockNavigation} />
    );

    expect(getByPlaceholderText('E-posta')).toBeTruthy();
    expect(getByPlaceholderText('Şifre')).toBeTruthy();
    expect(getByText('Giriş Yap')).toBeTruthy();
  });

  it('shows error when fields are empty', async () => {
    const { getByText } = renderWithNavigation(
      <LoginScreen navigation={mockNavigation} />
    );

    fireEvent.press(getByText('Giriş Yap'));

    await waitFor(() => {
      expect(getByText('Lütfen tüm alanları doldurun')).toBeTruthy();
    });
  });

  it('navigates to home on successful login', async () => {
    const { getByPlaceholderText, getByText } = renderWithNavigation(
      <LoginScreen navigation={mockNavigation} />
    );

    fireEvent.changeText(getByPlaceholderText('E-posta'), 'test@example.com');
    fireEvent.changeText(getByPlaceholderText('Şifre'), 'password123');
    fireEvent.press(getByText('Giriş Yap'));

    await waitFor(() => {
      expect(mockNavigate).toHaveBeenCalledWith('Home');
    });
  });
});
```

#### 4.11.3 API Testing

```jsx
// src/services/__tests__/apiService.test.js
import { apiService } from '../api';

// Mock fetch
global.fetch = jest.fn();

describe('ApiService', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  it('makes GET request successfully', async () => {
    const mockData = { id: 1, name: 'Test User' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData,
    });

    const result = await apiService.get('/users/1');
    
    expect(fetch).toHaveBeenCalledWith(
      'https://jsonplaceholder.typicode.com/users/1',
      expect.objectContaining({
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
        },
      })
    );
    expect(result).toEqual(mockData);
  });

  it('handles POST request', async () => {
    const mockData = { name: 'New User', email: 'new@example.com' };
    const responseData = { id: 2, ...mockData };
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => responseData,
    });

    const result = await apiService.post('/users', mockData);
    
    expect(fetch).toHaveBeenCalledWith(
      'https://jsonplaceholder.typicode.com/users',
      expect.objectContaining({
        method: 'POST',
        body: JSON.stringify(mockData),
      })
    );
    expect(result).toEqual(responseData);
  });

  it('handles API errors', async () => {
    fetch.mockRejectedValueOnce(new Error('Network error'));

    await expect(apiService.get('/users')).rejects.toThrow('Network error');
  });

  it('handles HTTP errors', async () => {
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
    });

    await expect(apiService.get('/users/999')).rejects.toThrow(
      'HTTP error! status: 404'
    );
  });
});
```

#### 4.11.4 E2E Testing with Detox

```bash
# Detox'u yükle
npm install --save-dev detox
```

```javascript
// e2e/firstTest.e2e.js
describe('Example', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should show welcome screen', async () => {
    await expect(element(by.id('welcome'))).toBeVisible();
  });

  it('should navigate to login screen', async () => {
    await element(by.id('login-button')).tap();
    await expect(element(by.id('login-screen'))).toBeVisible();
  });

  it('should login successfully', async () => {
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-submit')).tap();
    
    await expect(element(by.id('home-screen'))).toBeVisible();
  });

  it('should show error for invalid credentials', async () => {
    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');
    await element(by.id('login-submit')).tap();
    
    await expect(element(by.id('error-message'))).toBeVisible();
  });
});
```

```javascript
// .detoxrc.js
module.exports = {
  testRunner: 'jest',
  runnerConfig: 'e2e/config.json',
  configurations: {
    'ios.sim.debug': {
      binaryPath: 'ios/build/Build/Products/Debug-iphonesimulator/MyApp.app',
      build: 'xcodebuild -workspace ios/MyApp.xcworkspace -scheme MyApp -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build',
      type: 'ios.simulator',
      device: {
        type: 'iPhone 14',
      },
    },
    'android.emu.debug': {
      binaryPath: 'android/app/build/outputs/apk/debug/app-debug.apk',
      build: 'cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug',
      type: 'android.emulator',
      device: {
        avdName: 'Pixel_4_API_33',
      },
    },
  },
};
```

### 4.12 Build ve Deployment

#### 4.12.1 Android Build

```bash
# Android APK build
cd android
./gradlew assembleRelease

# Android Bundle build (Google Play Store için)
./gradlew bundleRelease
```

```gradle
// android/app/build.gradle
android {
    compileSdkVersion 33
    buildToolsVersion "33.0.0"

    defaultConfig {
        applicationId "com.yourapp"
        minSdkVersion 21
        targetSdkVersion 33
        versionCode 1
        versionName "1.0"
    }

    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled enableProguardInReleaseBuilds
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        }
    }
}
```

```properties
# android/gradle.properties
MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=*****
MYAPP_RELEASE_KEY_PASSWORD=*****
```

#### 4.12.2 iOS Build

```bash
# iOS build
cd ios
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -destination generic/platform=iOS -archivePath MyApp.xcarchive archive
```

```xml
<!-- ios/MyApp/Info.plist -->
<dict>
    <key>CFBundleDisplayName</key>
    <string>My App</string>
    <key>CFBundleIdentifier</key>
    <string>com.yourapp</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
</dict>
```

#### 4.12.3 Code Signing

```bash
# Android keystore oluştur
keytool -genkeypair -v -storetype PKCS12 -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

# iOS certificates ve provisioning profiles
# Xcode > Preferences > Accounts > Apple ID > Manage Certificates
```

#### 4.12.4 App Store ve Google Play Store Deployment

```bash
# Google Play Store için bundle yükle
cd android
./gradlew bundleRelease

# APK'yı Google Play Console'a yükle
# https://play.google.com/console

# iOS App Store için archive yükle
# Xcode > Product > Archive > Distribute App
```

```json
// app.json (Expo için)
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.yourapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

### 4.13 Debugging ve Development Tools

#### 4.13.1 Flipper

```bash
# Flipper'ı yükle
npm install --save-dev react-native-flipper

# iOS için ek kurulum
cd ios && pod install && cd ..
```

```jsx
// src/utils/flipper.js
import { Platform } from 'react-native';

if (__DEV__ && Platform.OS === 'ios') {
  import('react-native-flipper').then((Flipper) => {
    Flipper.addPlugin({
      getId() {
        return 'ReactNative';
      },
      onConnect(connection) {
        connection.send('connected', { message: 'React Native connected to Flipper' });
      },
      onDisconnect() {
        console.log('Disconnected from Flipper');
      },
      runInBackground() {
        return true;
      },
    });
  });
}
```

#### 4.13.2 React Native Debugger

```bash
# React Native Debugger'ı yükle
# https://github.com/jhen0409/react-native-debugger

# macOS
brew install --cask react-native-debugger

# Windows/Linux
# GitHub'dan indir: https://github.com/jhen0409/react-native-debugger/releases
```

```jsx
// src/utils/debugger.js
import { Platform } from 'react-native';

if (__DEV__) {
  // Redux DevTools
  if (Platform.OS === 'web') {
    window.__REDUX_DEVTOOLS_EXTENSION__ = window.__REDUX_DEVTOOLS_EXTENSION__ || (() => (next) => (reducer, initialState, enhancer) => {
      const store = next(reducer, initialState, enhancer);
      const devTools = window.__REDUX_DEVTOOLS_EXTENSION__.connect({
        name: 'React Native App',
      });
      devTools.init(store.getState());
      store.subscribe(() => {
        devTools.send('ACTION', store.getState());
      });
      return store;
    });
  }
}
```

#### 4.13.3 Chrome DevTools

```jsx
// src/utils/chromeDevTools.js
if (__DEV__) {
  // Chrome DevTools için
  const originalConsole = console;
  global.console = {
    ...originalConsole,
    log: (...args) => {
      originalConsole.log('[LOG]', ...args);
    },
    warn: (...args) => {
      originalConsole.warn('[WARN]', ...args);
    },
    error: (...args) => {
      originalConsole.error('[ERROR]', ...args);
    },
  };
}
```

#### 4.13.4 Error Handling

```jsx
// src/utils/errorHandler.js
import { Alert } from 'react-native';

class ErrorHandler {
  static handle(error, context = '') {
    console.error(`Error in ${context}:`, error);
    
    // Error reporting service'e gönder
    this.reportError(error, context);
    
    // Kullanıcıya göster
    this.showUserFriendlyError(error);
  }

  static reportError(error, context) {
    // Crashlytics, Sentry gibi servislere gönder
    if (__DEV__) {
      console.log('Error reported:', { error, context });
    }
  }

  static showUserFriendlyError(error) {
    let message = 'Beklenmeyen bir hata oluştu.';
    
    if (error.message) {
      if (error.message.includes('Network')) {
        message = 'İnternet bağlantınızı kontrol edin.';
      } else if (error.message.includes('Timeout')) {
        message = 'İşlem zaman aşımına uğradı.';
      }
    }

    Alert.alert('Hata', message, [
      { text: 'Tamam', style: 'default' },
    ]);
  }
}

export default ErrorHandler;
```

```jsx
// src/components/ErrorBoundary.js
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import ErrorHandler from '../utils/errorHandler';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    ErrorHandler.handle(error, 'ErrorBoundary');
  }

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.container}>
          <Text style={styles.title}>Bir hata oluştu</Text>
          <Text style={styles.message}>
            Uygulamada beklenmeyen bir hata oluştu. Lütfen uygulamayı yeniden başlatın.
          </Text>
          <TouchableOpacity
            style={styles.button}
            onPress={() => this.setState({ hasError: false, error: null })}
          >
            <Text style={styles.buttonText}>Tekrar Dene</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 20,
  },
  message: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    marginBottom: 30,
    lineHeight: 24,
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    minWidth: 150,
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default ErrorBoundary;
```

#### 4.13.5 Performance Monitoring

```jsx
// src/utils/performanceMonitor.js
import { Platform } from 'react-native';

class PerformanceMonitor {
  static startTiming(label) {
    if (__DEV__) {
      console.time(label);
    }
  }

  static endTiming(label) {
    if (__DEV__) {
      console.timeEnd(label);
    }
  }

  static measureRender(componentName, renderFunction) {
    if (__DEV__) {
      const start = performance.now();
      const result = renderFunction();
      const end = performance.now();
      console.log(`${componentName} render time: ${end - start}ms`);
      return result;
    }
    return renderFunction();
  }

  static logMemoryUsage() {
    if (__DEV__ && Platform.OS === 'android') {
      // Android memory usage
      const memInfo = require('react-native').NativeModules.DeviceInfo;
      if (memInfo) {
        memInfo.getUsedMemory().then((memory) => {
          console.log('Memory usage:', memory);
        });
      }
    }
  }
}

export default PerformanceMonitor;
```

```jsx
// App.js - ErrorBoundary ile sarmalama
import React from 'react';
import ErrorBoundary from './src/components/ErrorBoundary';
import NavigationContainer from './src/navigation/NavigationContainer';

function App() {
  return (
    <ErrorBoundary>
      <NavigationContainer />
    </ErrorBoundary>
  );
}

export default App;
```

