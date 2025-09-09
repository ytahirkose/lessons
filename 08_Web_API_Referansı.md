# Web API Referansı - Mozilla Developer Network

## Giriş

Web API'leri, web tarayıcıları tarafından sağlanan ve geliştiricilerin web uygulamalarında çeşitli işlevleri gerçekleştirmelerine olanak tanıyan programlama arayüzleridir. Bu API'ler JavaScript ile kullanılır ve web uygulamalarının daha zengin, etkileşimli ve güçlü olmasını sağlar.

## Web API'lerinin Temel Özellikleri

- **Nesne Tabanlı Yapı**: Web API'leri JavaScript nesneleri aracılığıyla çalışır
- **Güvenlik Mekanizmaları**: Bazı API'ler HTTPS gerektirir ve kullanıcı izni ister
- **Tarayıcı Uyumluluğu**: Farklı tarayıcılarda farklı destek seviyeleri
- **Asenkron İşlemler**: Çoğu API asenkron çalışır ve Promise döndürür

---

## A - Bölümü

### AbortController API
İşlemleri iptal etmek için kullanılan API - Asenkron işlemleri kontrol etme.

## AbortController Nedir?

AbortController, asenkron işlemleri (fetch, setTimeout, vb.) iptal etmek için kullanılan bir API'dir. Özellikle uzun süren işlemlerde kullanıcı deneyimini iyileştirmek için kritik öneme sahiptir.

## Neden Kullanılır?

### 1. **Kullanıcı Deneyimi**
```javascript
// ❌ Kötü: İptal edilemeyen uzun işlem
function longRunningTask() {
  return fetch('/api/slow-endpoint')
    .then(response => response.json())
    .then(data => {
      // 10 saniye süren işlem
      return processLargeData(data);
    });
}

// Kullanıcı sayfayı terk etse bile işlem devam eder
```

```javascript
// ✅ İyi: İptal edilebilir işlem
const controller = new AbortController();
const signal = controller.signal;

function cancellableTask() {
  return fetch('/api/slow-endpoint', { signal })
    .then(response => response.json())
    .then(data => {
      if (signal.aborted) {
        throw new Error('İşlem iptal edildi');
      }
      return processLargeData(data);
    });
}

// Kullanıcı iptal edebilir
document.getElementById('cancelBtn').onclick = () => {
  controller.abort();
};
```

### 2. **Kaynak Yönetimi**
```javascript
// Çoklu istekleri yönet
const controllers = new Map();

function startRequest(id, url) {
  const controller = new AbortController();
  controllers.set(id, controller);
  
  return fetch(url, { signal: controller.signal })
    .then(response => response.json())
    .finally(() => {
      controllers.delete(id);
    });
}

function cancelRequest(id) {
  const controller = controllers.get(id);
  if (controller) {
    controller.abort();
    controllers.delete(id);
  }
}
```

## Pratik Kullanım Örnekleri

### 1. **Fetch İsteklerini İptal Etme**
```javascript
class RequestManager {
  constructor() {
    this.controllers = new Map();
  }
  
  async makeRequest(url, options = {}) {
    const controller = new AbortController();
    const requestId = Date.now().toString();
    
    this.controllers.set(requestId, controller);
    
    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal
      });
      
      return await response.json();
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('İstek iptal edildi');
        return null;
      }
      throw error;
    } finally {
      this.controllers.delete(requestId);
    }
  }
  
  cancelRequest(requestId) {
    const controller = this.controllers.get(requestId);
    if (controller) {
      controller.abort();
    }
  }
  
  cancelAllRequests() {
    this.controllers.forEach(controller => controller.abort());
    this.controllers.clear();
  }
}

// Kullanım
const requestManager = new RequestManager();

// İstek başlat
const requestId = await requestManager.makeRequest('/api/data');

// İstek iptal et
requestManager.cancelRequest(requestId);
```

### 2. **Timeout ile Otomatik İptal**
```javascript
function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  
  // Timeout ayarla
  const timeoutId = setTimeout(() => {
    controller.abort();
  }, timeout);
  
  return fetch(url, { signal: controller.signal })
    .then(response => {
      clearTimeout(timeoutId);
      return response.json();
    })
    .catch(error => {
      clearTimeout(timeoutId);
      if (error.name === 'AbortError') {
        throw new Error('İstek zaman aşımına uğradı');
      }
      throw error;
    });
}

// Kullanım
fetchWithTimeout('/api/data', 3000)
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### 3. **WebSocket Bağlantılarını İptal Etme**
```javascript
class WebSocketManager {
  constructor() {
    this.controllers = new Map();
  }
  
  connect(url, options = {}) {
    const controller = new AbortController();
    const connectionId = Date.now().toString();
    
    this.controllers.set(connectionId, controller);
    
    const ws = new WebSocket(url);
    
    // Abort signal dinle
    controller.signal.addEventListener('abort', () => {
      ws.close();
      this.controllers.delete(connectionId);
    });
    
    return { ws, connectionId };
  }
  
  disconnect(connectionId) {
    const controller = this.controllers.get(connectionId);
    if (controller) {
      controller.abort();
    }
  }
}
```

### 4. **Promise Zincirlerini İptal Etme**
```javascript
function cancellablePromise(executor) {
  let controller = new AbortController();
  
  const promise = new Promise((resolve, reject) => {
    const signal = controller.signal;
    
    // Abort event dinle
    signal.addEventListener('abort', () => {
      reject(new Error('Promise iptal edildi'));
    });
    
    // Executor'ı çalıştır
    executor(resolve, reject, signal);
  });
  
  // Abort fonksiyonunu promise'e ekle
  promise.abort = () => controller.abort();
  
  return promise;
}

// Kullanım
const promise = cancellablePromise((resolve, reject, signal) => {
  setTimeout(() => {
    if (signal.aborted) return;
    resolve('İşlem tamamlandı');
  }, 5000);
});

// İptal et
setTimeout(() => {
  promise.abort();
}, 2000);
```

## Alternatifler

### 1. **Promise.race() ile Timeout**
```javascript
function timeoutPromise(ms) {
  return new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), ms);
  });
}

function fetchWithRaceTimeout(url, timeout = 5000) {
  return Promise.race([
    fetch(url),
    timeoutPromise(timeout)
  ]);
}
```

### 2. **Custom Cancellation Token**
```javascript
class CancellationToken {
  constructor() {
    this.cancelled = false;
    this.listeners = [];
  }
  
  cancel() {
    this.cancelled = true;
    this.listeners.forEach(listener => listener());
  }
  
  onCancel(callback) {
    this.listeners.push(callback);
  }
  
  throwIfCancelled() {
    if (this.cancelled) {
      throw new Error('İşlem iptal edildi');
    }
  }
}
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Uzun süren fetch istekleri
- Kullanıcı etkileşimli işlemler
- Dosya yükleme/indirme
- WebSocket bağlantıları
- Arama işlemleri
- Real-time veri akışları

### ❌ **Kullanmayın:**
- Kısa süren işlemler
- Kritik sistem işlemleri
- Tek seferlik işlemler

## Kullanmazsak Ne Olur?

### 1. **Kötü Kullanıcı Deneyimi**
- İptal edilemeyen işlemler
- Gereksiz kaynak kullanımı
- Sayfa performans sorunları

### 2. **Kaynak İsrafı**
- Kullanılmayan network istekleri
- Bellek sızıntıları
- CPU döngü israfı

### 3. **Güvenlik Sorunları**
- Kontrolsüz veri akışı
- Potansiyel DoS saldırıları

## Modern Alternatifler

### 1. **AbortSignal.timeout()**
```javascript
// Modern timeout API
const controller = AbortSignal.timeout(5000);

fetch('/api/data', { signal: controller })
  .then(response => response.json())
  .catch(error => {
    if (error.name === 'TimeoutError') {
      console.log('İstek zaman aşımına uğradı');
    }
  });
```

### 2. **AbortSignal.any()**
```javascript
// Birden fazla signal'ı birleştir
const signal1 = new AbortController().signal;
const signal2 = new AbortController().signal;
const combinedSignal = AbortSignal.any([signal1, signal2]);

fetch('/api/data', { signal: combinedSignal });
```

## Özet

**AbortController API**, modern web uygulamalarında **kritik öneme sahip** bir teknolojidir. Asenkron işlemleri kontrol ederek:

- ✅ Kullanıcı deneyimini iyileştirir
- ✅ Kaynak kullanımını optimize eder
- ✅ Performansı artırır
- ✅ Güvenliği sağlar

**Kullanın** - özellikle uzun süren işlemler, kullanıcı etkileşimli uygulamalar ve kaynak yönetimi gereken durumlar için.

```javascript
// Basit AbortController örneği
const controller = new AbortController();
const signal = controller.signal;

fetch('/api/data', { signal })
  .then(response => response.json())
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('İstek iptal edildi');
    }
  });

// İsteği iptal et
controller.abort();
```

### Animation API
Web animasyonları oluşturmak için kullanılan API - Modern CSS animasyonları.

## Animation API Nedir?

Web Animations API, CSS animasyonları ve transitions'ları JavaScript ile kontrol etmek için kullanılan modern bir API'dir. CSS animasyonlarından daha güçlü ve esnek bir kontrol sağlar.

## Neden Kullanılır?

### 1. **JavaScript Kontrolü**
```javascript
// ❌ CSS ile sınırlı kontrol
.element {
  animation: slideIn 1s ease-in-out;
}

@keyframes slideIn {
  from { transform: translateX(-100px); }
  to { transform: translateX(0); }
}

// Animasyonu durdurmak, hızlandırmak zor
```

```javascript
// ✅ JavaScript ile tam kontrol
const element = document.getElementById('myElement');
const animation = element.animate([
  { transform: 'translateX(-100px)', opacity: 0 },
  { transform: 'translateX(0)', opacity: 1 }
], {
  duration: 1000,
  easing: 'ease-in-out',
  fill: 'forwards'
});

// Tam kontrol
animation.pause();
animation.play();
animation.reverse();
animation.currentTime = 500; // 500ms'ye atla
```

### 2. **Dinamik Animasyonlar**
```javascript
// Kullanıcı etkileşimine göre animasyon
function createDynamicAnimation(element, direction) {
  const keyframes = direction === 'left' 
    ? [
        { transform: 'translateX(0px)' },
        { transform: 'translateX(-100px)' }
      ]
    : [
        { transform: 'translateX(0px)' },
        { transform: 'translateX(100px)' }
      ];

  return element.animate(keyframes, {
    duration: 300,
    easing: 'ease-in-out',
    fill: 'forwards'
  });
}

// Kullanım
document.addEventListener('keydown', (e) => {
  if (e.key === 'ArrowLeft') {
    createDynamicAnimation(element, 'left');
  } else if (e.key === 'ArrowRight') {
    createDynamicAnimation(element, 'right');
  }
});
```

## Pratik Kullanım Örnekleri

### 1. **Temel Animasyonlar**
```javascript
class AnimationManager {
  constructor() {
    this.animations = new Map();
  }
  
  // Fade in animasyonu
  fadeIn(element, duration = 500) {
    const animation = element.animate([
      { opacity: 0 },
      { opacity: 1 }
    ], {
      duration,
      fill: 'forwards',
      easing: 'ease-in-out'
    });
    
    this.animations.set(element, animation);
    return animation;
  }
  
  // Fade out animasyonu
  fadeOut(element, duration = 500) {
    const animation = element.animate([
      { opacity: 1 },
      { opacity: 0 }
    ], {
      duration,
      fill: 'forwards',
      easing: 'ease-in-out'
    });
    
    this.animations.set(element, animation);
    return animation;
  }
  
  // Slide animasyonu
  slide(element, direction, distance = 100, duration = 300) {
    const startTransform = direction === 'left' 
      ? `translateX(${distance}px)`
      : direction === 'right'
      ? `translateX(-${distance}px)`
      : direction === 'up'
      ? `translateY(${distance}px)`
      : `translateY(-${distance}px)`;
    
    const endTransform = 'translateX(0px) translateY(0px)';
    
    const animation = element.animate([
      { transform: startTransform },
      { transform: endTransform }
    ], {
      duration,
      fill: 'forwards',
      easing: 'ease-out'
    });
    
    this.animations.set(element, animation);
    return animation;
  }
  
  // Scale animasyonu
  scale(element, fromScale = 0, toScale = 1, duration = 300) {
    const animation = element.animate([
      { transform: `scale(${fromScale})` },
      { transform: `scale(${toScale})` }
    ], {
      duration,
      fill: 'forwards',
      easing: 'ease-out'
    });
    
    this.animations.set(element, animation);
    return animation;
  }
  
  // Animasyonu durdur
  stop(element) {
    const animation = this.animations.get(element);
    if (animation) {
      animation.cancel();
      this.animations.delete(element);
    }
  }
  
  // Tüm animasyonları durdur
  stopAll() {
    this.animations.forEach(animation => animation.cancel());
    this.animations.clear();
  }
}

// Kullanım
const animator = new AnimationManager();
const element = document.getElementById('myElement');

// Animasyonları sırayla çalıştır
animator.fadeIn(element, 500)
  .addEventListener('finish', () => {
    animator.slide(element, 'up', 50, 300);
  });
```

### 2. **Karmaşık Animasyonlar**
```javascript
// Çoklu özellik animasyonu
function complexAnimation(element) {
  return element.animate([
    {
      transform: 'translateX(0px) scale(1) rotate(0deg)',
      opacity: 1,
      backgroundColor: '#ff0000'
    },
    {
      transform: 'translateX(100px) scale(1.2) rotate(180deg)',
      opacity: 0.5,
      backgroundColor: '#00ff00'
    },
    {
      transform: 'translateX(200px) scale(0.8) rotate(360deg)',
      opacity: 1,
      backgroundColor: '#0000ff'
    }
  ], {
    duration: 2000,
    iterations: Infinity,
    direction: 'alternate',
    easing: 'ease-in-out'
  });
}

// Staggered animasyon (sıralı)
function staggeredAnimation(elements, delay = 100) {
  elements.forEach((element, index) => {
    element.animate([
      { transform: 'translateY(20px)', opacity: 0 },
      { transform: 'translateY(0)', opacity: 1 }
    ], {
      duration: 500,
      delay: index * delay,
      fill: 'forwards',
      easing: 'ease-out'
    });
  });
}

// Kullanım
const elements = document.querySelectorAll('.item');
staggeredAnimation(Array.from(elements), 150);
```

### 3. **Scroll-triggered Animasyonlar**
```javascript
class ScrollAnimations {
  constructor() {
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      { threshold: 0.1 }
    );
    this.animatedElements = new Set();
  }
  
  observe(element, animationType = 'fadeIn') {
    element.dataset.animationType = animationType;
    this.observer.observe(element);
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting && !this.animatedElements.has(entry.target)) {
        this.animateElement(entry.target);
        this.animatedElements.add(entry.target);
      }
    });
  }
  
  animateElement(element) {
    const animationType = element.dataset.animationType;
    
    switch (animationType) {
      case 'fadeIn':
        this.fadeIn(element);
        break;
      case 'slideUp':
        this.slideUp(element);
        break;
      case 'scale':
        this.scale(element);
        break;
      default:
        this.fadeIn(element);
    }
  }
  
  fadeIn(element) {
    element.animate([
      { opacity: 0 },
      { opacity: 1 }
    ], {
      duration: 800,
      fill: 'forwards',
      easing: 'ease-out'
    });
  }
  
  slideUp(element) {
    element.animate([
      { transform: 'translateY(50px)', opacity: 0 },
      { transform: 'translateY(0)', opacity: 1 }
    ], {
      duration: 600,
      fill: 'forwards',
      easing: 'ease-out'
    });
  }
  
  scale(element) {
    element.animate([
      { transform: 'scale(0.8)', opacity: 0 },
      { transform: 'scale(1)', opacity: 1 }
    ], {
      duration: 500,
      fill: 'forwards',
      easing: 'ease-out'
    });
  }
}

// Kullanım
const scrollAnimations = new ScrollAnimations();

document.querySelectorAll('.animate-on-scroll').forEach(element => {
  scrollAnimations.observe(element, 'slideUp');
});
```

### 4. **Morphing Animasyonlar**
```javascript
// Şekil değiştirme animasyonu
function morphShape(element, fromShape, toShape, duration = 1000) {
  return element.animate([
    { clipPath: fromShape },
    { clipPath: toShape }
  ], {
    duration,
    fill: 'forwards',
    easing: 'ease-in-out'
  });
}

// Kullanım
const shape = document.getElementById('morphing-shape');
const circle = 'circle(50% at 50% 50%)';
const square = 'polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)';
const triangle = 'polygon(50% 0%, 0% 100%, 100% 100%)';

// Şekiller arası geçiş
morphShape(shape, circle, square, 1000)
  .addEventListener('finish', () => {
    morphShape(shape, square, triangle, 1000);
  });
```

## Alternatifler

### 1. **CSS Animations**
```css
/* CSS ile basit animasyonlar */
@keyframes slideIn {
  from { transform: translateX(-100px); }
  to { transform: translateX(0); }
}

.element {
  animation: slideIn 1s ease-in-out;
}
```

### 2. **CSS Transitions**
```css
/* Hover efektleri için */
.button {
  transition: all 0.3s ease;
}

.button:hover {
  transform: scale(1.1);
  background-color: #007AFF;
}
```

### 3. **GSAP (GreenSock)**
```javascript
// Daha güçlü animasyon kütüphanesi
gsap.to('.element', {
  duration: 1,
  x: 100,
  rotation: 360,
  ease: 'bounce'
});
```

### 4. **Framer Motion (React)**
```jsx
// React için animasyon kütüphanesi
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.5 }}
>
  Content
</motion.div>
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- JavaScript kontrolü gereken animasyonlar
- Dinamik animasyonlar
- Karmaşık animasyon dizileri
- Scroll-triggered animasyonlar
- Kullanıcı etkileşimli animasyonlar
- Performans kritik animasyonlar

### ❌ **Kullanmayın:**
- Basit CSS animasyonları yeterliyse
- Statik animasyonlar
- Hover efektleri
- Basit geçişler

## Kullanmazsak Ne Olur?

### 1. **Sınırlı Kontrol**
- CSS animasyonları durdurulamaz
- Dinamik değişiklikler zor
- JavaScript entegrasyonu karmaşık

### 2. **Performans Sorunları**
- CSS animasyonları optimize edilemez
- Bellek yönetimi zor
- Çoklu animasyon kontrolü yok

### 3. **Kullanıcı Deneyimi**
- Animasyonlar kesintiye uğrayabilir
- Responsive animasyonlar zor
- Accessibility desteği sınırlı

## Modern Alternatifler

### 1. **CSS Custom Properties ile Dinamik Animasyonlar**
```css
:root {
  --animation-duration: 1s;
  --animation-delay: 0s;
}

.element {
  animation: slideIn var(--animation-duration) ease-in-out var(--animation-delay);
}
```

```javascript
// JavaScript ile CSS değişkenlerini kontrol et
element.style.setProperty('--animation-duration', '2s');
element.style.setProperty('--animation-delay', '0.5s');
```

### 2. **Web Animations API + CSS**
```javascript
// CSS keyframes'leri JavaScript'te kullan
const keyframes = [
  { transform: 'translateX(0px)' },
  { transform: 'translateX(100px)' }
];

const animation = element.animate(keyframes, {
  duration: 1000,
  easing: 'ease-in-out'
});
```

## Özet

**Animation API**, modern web uygulamalarında **güçlü animasyon kontrolü** sağlar. JavaScript ile tam kontrol ederek:

- ✅ Dinamik animasyonlar yaratır
- ✅ Performansı optimize eder
- ✅ Kullanıcı etkileşimini geliştirir
- ✅ Karmaşık animasyon dizileri yönetir

**Kullanın** - özellikle JavaScript kontrolü gereken, dinamik ve karmaşık animasyonlar için.

```javascript
// Basit Animation API örneği
const element = document.getElementById('myElement');
const animation = element.animate([
  { transform: 'translateX(0px)' },
  { transform: 'translateX(100px)' }
], {
  duration: 1000,
  iterations: Infinity
});
```

### AudioContext API
Web ses işleme için kullanılan API - Web Audio işleme.

## AudioContext Nedir?

AudioContext API, web tarayıcılarında profesyonel seviyede ses işleme yapmak için kullanılan güçlü bir API'dir. Ses sinyallerini oluşturma, işleme, analiz etme ve çalma işlemlerini gerçekleştirir.

## Neden Kullanılır?

### 1. **Profesyonel Ses İşleme**
```javascript
// ❌ Basit HTML5 Audio sınırlı
const audio = new Audio('sound.mp3');
audio.play(); // Sadece çalma, işleme yok
```

```javascript
// ✅ AudioContext ile tam kontrol
const audioContext = new AudioContext();
const oscillator = audioContext.createOscillator();
const gainNode = audioContext.createGain();
const filterNode = audioContext.createBiquadFilter();

// Ses işleme zinciri
oscillator.connect(filterNode);
filterNode.connect(gainNode);
gainNode.connect(audioContext.destination);

// Parametreleri kontrol et
oscillator.frequency.setValueAtTime(440, audioContext.currentTime);
filterNode.frequency.setValueAtTime(1000, audioContext.currentTime);
gainNode.gain.setValueAtTime(0.5, audioContext.currentTime);

oscillator.start();
```

### 2. **Real-time Ses Analizi**
```javascript
// Mikrofon girişini analiz et
async function analyzeMicrophone() {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  const audioContext = new AudioContext();
  const source = audioContext.createMediaStreamSource(stream);
  const analyser = audioContext.createAnalyser();
  
  source.connect(analyser);
  
  // Frekans analizi
  const frequencyData = new Uint8Array(analyser.frequencyBinCount);
  
  function analyze() {
    analyser.getByteFrequencyData(frequencyData);
    console.log('Frekans verisi:', frequencyData);
    requestAnimationFrame(analyze);
  }
  
  analyze();
}
```

## Pratik Kullanım Örnekleri

### 1. **Ses Sentezi**
```javascript
class AudioSynthesizer {
  constructor() {
    this.audioContext = new AudioContext();
    this.oscillators = new Map();
  }
  
  // Basit ton üret
  playTone(frequency, duration = 1, type = 'sine') {
    const oscillator = this.audioContext.createOscillator();
    const gainNode = this.audioContext.createGain();
    
    oscillator.type = type;
    oscillator.frequency.setValueAtTime(frequency, this.audioContext.currentTime);
    
    // Fade in/out efekti
    gainNode.gain.setValueAtTime(0, this.audioContext.currentTime);
    gainNode.gain.linearRampToValueAtTime(0.3, this.audioContext.currentTime + 0.1);
    gainNode.gain.linearRampToValueAtTime(0, this.audioContext.currentTime + duration);
    
    oscillator.connect(gainNode);
    gainNode.connect(this.audioContext.destination);
    
    oscillator.start();
    oscillator.stop(this.audioContext.currentTime + duration);
  }
  
  // Akor çal
  playChord(frequencies, duration = 2) {
    frequencies.forEach(freq => {
      this.playTone(freq, duration);
    });
  }
  
  // Melodi çal
  playMelody(notes, tempo = 120) {
    const noteDuration = 60 / tempo; // saniye cinsinden
    
    notes.forEach((note, index) => {
      const startTime = this.audioContext.currentTime + (index * noteDuration);
      const oscillator = this.audioContext.createOscillator();
      const gainNode = this.audioContext.createGain();
      
      oscillator.frequency.setValueAtTime(note.frequency, startTime);
      gainNode.gain.setValueAtTime(0, startTime);
      gainNode.gain.linearRampToValueAtTime(0.3, startTime + 0.1);
      gainNode.gain.linearRampToValueAtTime(0, startTime + noteDuration);
      
      oscillator.connect(gainNode);
      gainNode.connect(this.audioContext.destination);
      
      oscillator.start(startTime);
      oscillator.stop(startTime + noteDuration);
    });
  }
}

// Kullanım
const synth = new AudioSynthesizer();

// Tek ton
synth.playTone(440, 1); // A4 notası

// Akor
synth.playChord([261.63, 329.63, 392.00]); // C major

// Melodi
const melody = [
  { frequency: 261.63 }, // C
  { frequency: 293.66 }, // D
  { frequency: 329.63 }, // E
  { frequency: 349.23 }, // F
  { frequency: 392.00 }, // G
  { frequency: 440.00 }, // A
  { frequency: 493.88 }, // B
  { frequency: 523.25 }  // C
];
synth.playMelody(melody, 120);
```

### 2. **Ses Efektleri**
```javascript
class AudioEffects {
  constructor() {
    this.audioContext = new AudioContext();
  }
  
  // Reverb efekti
  createReverb(source, roomSize = 0.5, dampening = 0.5) {
    const convolver = this.audioContext.createConvolver();
    const gainNode = this.audioContext.createGain();
    
    // Basit reverb impulse response
    const length = this.audioContext.sampleRate * roomSize;
    const impulse = this.audioContext.createBuffer(2, length, this.audioContext.sampleRate);
    
    for (let channel = 0; channel < 2; channel++) {
      const channelData = impulse.getChannelData(channel);
      for (let i = 0; i < length; i++) {
        channelData[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, dampening);
      }
    }
    
    convolver.buffer = impulse;
    gainNode.gain.setValueAtTime(0.3, this.audioContext.currentTime);
    
    source.connect(convolver);
    convolver.connect(gainNode);
    gainNode.connect(this.audioContext.destination);
    
    return { convolver, gainNode };
  }
  
  // Delay efekti
  createDelay(source, delayTime = 0.5, feedback = 0.3) {
    const delay = this.audioContext.createDelay();
    const feedbackGain = this.audioContext.createGain();
    const outputGain = this.audioContext.createGain();
    
    delay.delayTime.setValueAtTime(delayTime, this.audioContext.currentTime);
    feedbackGain.gain.setValueAtTime(feedback, this.audioContext.currentTime);
    outputGain.gain.setValueAtTime(0.7, this.audioContext.currentTime);
    
    source.connect(delay);
    delay.connect(feedbackGain);
    feedbackGain.connect(delay);
    delay.connect(outputGain);
    outputGain.connect(this.audioContext.destination);
    
    return { delay, feedbackGain, outputGain };
  }
  
  // Distortion efekti
  createDistortion(source, amount = 50) {
    const distortion = this.audioContext.createWaveShaper();
    const gainNode = this.audioContext.createGain();
    
    // Distortion curve
    const samples = 44100;
    const curve = new Float32Array(samples);
    const deg = Math.PI / 180;
    
    for (let i = 0; i < samples; i++) {
      const x = (i * 2) / samples - 1;
      curve[i] = ((3 + amount) * x * 20 * deg) / (Math.PI + amount * Math.abs(x));
    }
    
    distortion.curve = curve;
    distortion.oversample = '4x';
    gainNode.gain.setValueAtTime(0.5, this.audioContext.currentTime);
    
    source.connect(distortion);
    distortion.connect(gainNode);
    gainNode.connect(this.audioContext.destination);
    
    return { distortion, gainNode };
  }
}

// Kullanım
const effects = new AudioEffects();
const oscillator = effects.audioContext.createOscillator();
oscillator.frequency.setValueAtTime(440, effects.audioContext.currentTime);

// Efektleri uygula
const reverb = effects.createReverb(oscillator, 0.8, 0.3);
const delay = effects.createDelay(oscillator, 0.3, 0.2);
const distortion = effects.createDistortion(oscillator, 30);

oscillator.start();
```

### 3. **Ses Analizi ve Görselleştirme**
```javascript
class AudioAnalyzer {
  constructor() {
    this.audioContext = new AudioContext();
    this.analyser = this.audioContext.createAnalyser();
    this.analyser.fftSize = 2048;
    this.analyser.smoothingTimeConstant = 0.8;
  }
  
  // Mikrofon girişini analiz et
  async startMicrophoneAnalysis() {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const source = this.audioContext.createMediaStreamSource(stream);
    source.connect(this.analyser);
    
    return this.analyser;
  }
  
  // Frekans analizi
  getFrequencyData() {
    const bufferLength = this.analyser.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    this.analyser.getByteFrequencyData(dataArray);
    return dataArray;
  }
  
  // Zaman domain analizi
  getTimeDomainData() {
    const bufferLength = this.analyser.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    this.analyser.getByteTimeDomainData(dataArray);
    return dataArray;
  }
  
  // Ses seviyesi
  getVolume() {
    const dataArray = this.getTimeDomainData();
    let sum = 0;
    for (let i = 0; i < dataArray.length; i++) {
      sum += Math.abs(dataArray[i] - 128);
    }
    return sum / dataArray.length;
  }
  
  // Frekans spektrumu
  getFrequencySpectrum() {
    const dataArray = this.getFrequencyData();
    const spectrum = [];
    
    // Frekans bantlarına böl
    const bands = 8;
    const bandSize = Math.floor(dataArray.length / bands);
    
    for (let i = 0; i < bands; i++) {
      let sum = 0;
      for (let j = 0; j < bandSize; j++) {
        sum += dataArray[i * bandSize + j];
      }
      spectrum.push(sum / bandSize);
    }
    
    return spectrum;
  }
}

// Kullanım
const analyzer = new AudioAnalyzer();

// Mikrofon analizi başlat
analyzer.startMicrophoneAnalysis().then(() => {
  // Real-time analiz
  function analyze() {
    const volume = analyzer.getVolume();
    const spectrum = analyzer.getFrequencySpectrum();
    
    console.log('Ses seviyesi:', volume);
    console.log('Frekans spektrumu:', spectrum);
    
    // Görselleştirme
    updateVisualization(volume, spectrum);
    
    requestAnimationFrame(analyze);
  }
  
  analyze();
});

function updateVisualization(volume, spectrum) {
  // Canvas'a çiz
  const canvas = document.getElementById('visualizer');
  const ctx = canvas.getContext('2d');
  
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // Ses seviyesi çubuğu
  ctx.fillStyle = `hsl(${volume * 2}, 100%, 50%)`;
  ctx.fillRect(0, 0, volume * 2, 20);
  
  // Frekans spektrumu
  const barWidth = canvas.width / spectrum.length;
  spectrum.forEach((value, index) => {
    const barHeight = (value / 255) * canvas.height;
    ctx.fillStyle = `hsl(${index * 45}, 100%, 50%)`;
    ctx.fillRect(index * barWidth, canvas.height - barHeight, barWidth - 2, barHeight);
  });
}
```

### 4. **Ses Kayıt ve Oynatma**
```javascript
class AudioRecorder {
  constructor() {
    this.audioContext = new AudioContext();
    this.mediaRecorder = null;
    this.audioChunks = [];
  }
  
  // Kayıt başlat
  async startRecording() {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    this.mediaRecorder = new MediaRecorder(stream);
    this.audioChunks = [];
    
    this.mediaRecorder.ondataavailable = (event) => {
      this.audioChunks.push(event.data);
    };
    
    this.mediaRecorder.start();
  }
  
  // Kayıt durdur
  stopRecording() {
    return new Promise((resolve) => {
      this.mediaRecorder.onstop = () => {
        const audioBlob = new Blob(this.audioChunks, { type: 'audio/wav' });
        const audioUrl = URL.createObjectURL(audioBlob);
        resolve(audioUrl);
      };
      
      this.mediaRecorder.stop();
    });
  }
  
  // Kayıt oynat
  playRecording(audioUrl) {
    const audio = new Audio(audioUrl);
    audio.play();
  }
  
  // Kayıt indir
  downloadRecording(audioUrl, filename = 'recording.wav') {
    const a = document.createElement('a');
    a.href = audioUrl;
    a.download = filename;
    a.click();
  }
}

// Kullanım
const recorder = new AudioRecorder();

document.getElementById('startBtn').onclick = () => {
  recorder.startRecording();
};

document.getElementById('stopBtn').onclick = async () => {
  const audioUrl = await recorder.stopRecording();
  recorder.playRecording(audioUrl);
  
  // İndirme butonu ekle
  const downloadBtn = document.createElement('button');
  downloadBtn.textContent = 'İndir';
  downloadBtn.onclick = () => recorder.downloadRecording(audioUrl);
  document.body.appendChild(downloadBtn);
};
```

## Alternatifler

### 1. **HTML5 Audio**
```javascript
// Basit ses çalma
const audio = new Audio('sound.mp3');
audio.play();
```

### 2. **Web Audio API Kütüphaneleri**
```javascript
// Tone.js kütüphanesi
import * as Tone from 'tone';

const synth = new Tone.Synth().toDestination();
synth.triggerAttackRelease('C4', '8n');
```

### 3. **Howler.js**
```javascript
// Ses yönetimi kütüphanesi
import { Howl } from 'howler';

const sound = new Howl({
  src: ['sound.mp3'],
  volume: 0.5
});

sound.play();
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Profesyonel ses işleme
- Real-time ses analizi
- Ses sentezi
- Ses efektleri
- Müzik uygulamaları
- Oyun ses sistemleri
- Ses görselleştirme

### ❌ **Kullanmayın:**
- Basit ses çalma
- Statik ses dosyaları
- Basit müzik çalma
- Tek ses efektleri

## Kullanmazsak Ne Olur?

### 1. **Sınırlı Ses Kontrolü**
- HTML5 Audio ile sınırlı kontrol
- Real-time işleme yok
- Ses analizi yok

### 2. **Performans Sorunları**
- Çoklu ses yönetimi zor
- Bellek kullanımı optimize edilemez
- CPU kullanımı yüksek

### 3. **Kullanıcı Deneyimi**
- Ses kalitesi düşük
- Gecikme sorunları
- Etkileşim sınırlı

## Modern Alternatifler

### 1. **Web Audio API + WebAssembly**
```javascript
// WASM ile ses işleme
const wasmModule = await WebAssembly.instantiateStreaming(
  fetch('audio-processor.wasm')
);

const processor = wasmModule.instance.exports.processAudio;
```

### 2. **Audio Worklets**
```javascript
// Arka planda ses işleme
class AudioProcessor extends AudioWorkletProcessor {
  process(inputs, outputs, parameters) {
    // Ses işleme mantığı
    return true;
  }
}

registerProcessor('audio-processor', AudioProcessor);
```

## Özet

**AudioContext API**, modern web uygulamalarında **profesyonel ses işleme** sağlar. Web Audio işleme ile:

- ✅ Real-time ses analizi yapar
- ✅ Ses sentezi ve efektleri uygular
- ✅ Performansı optimize eder
- ✅ Profesyonel ses kalitesi sağlar

**Kullanın** - özellikle müzik uygulamaları, oyun ses sistemleri, ses analizi ve real-time ses işleme için.

```javascript
// Basit AudioContext örneği
const audioContext = new AudioContext();
const oscillator = audioContext.createOscillator();
const gainNode = audioContext.createGain();

oscillator.connect(gainNode);
gainNode.connect(audioContext.destination);
oscillator.start();
```

### AudioWorklet API
Ses işleme için özel işçi iş parçacıkları oluşturur - Arka planda ses işleme.

## AudioWorklet Nedir?

AudioWorklet API, ses işleme işlemlerini ana thread'den ayırarak arka planda çalıştırmak için kullanılan modern bir API'dir. Web Workers'a benzer şekilde çalışır ancak ses işleme için özelleştirilmiştir.

## Neden Kullanılır?

### 1. **Performans Optimizasyonu**
```javascript
// ❌ Ana thread'de ses işleme - UI donabilir
function processAudioOnMainThread(audioData) {
  // Ağır ses işleme
  for (let i = 0; i < audioData.length; i++) {
    audioData[i] = audioData[i] * 0.5; // Basit işlem
  }
  // UI donabilir
}
```

```javascript
// ✅ AudioWorklet ile arka planda işleme
// audio-processor.js (AudioWorklet)
class AudioProcessor extends AudioWorkletProcessor {
  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];
    
    if (input.length > 0) {
      for (let channel = 0; channel < input.length; channel++) {
        const inputChannel = input[channel];
        const outputChannel = output[channel];
        
        for (let i = 0; i < inputChannel.length; i++) {
          // Ağır ses işleme - UI donmaz
          outputChannel[i] = inputChannel[i] * 0.5;
        }
      }
    }
    
    return true; // Sürekli çalış
  }
}

registerProcessor('audio-processor', AudioProcessor);
```

### 2. **Real-time Ses İşleme**
```javascript
// Ana thread
const audioContext = new AudioContext();
await audioContext.audioWorklet.addModule('audio-processor.js');
const audioWorkletNode = new AudioWorkletNode(audioContext, 'audio-processor');

// Mikrofon girişini bağla
navigator.mediaDevices.getUserMedia({ audio: true })
  .then(stream => {
    const source = audioContext.createMediaStreamSource(stream);
    source.connect(audioWorkletNode);
    audioWorkletNode.connect(audioContext.destination);
  });
```

## Pratik Kullanım Örnekleri

### 1. **Ses Filtreleme**
```javascript
// audio-filter.js (AudioWorklet)
class AudioFilter extends AudioWorkletProcessor {
  constructor(options) {
    super();
    this.cutoff = options.processorOptions.cutoff || 1000;
    this.resonance = options.processorOptions.resonance || 1;
    this.type = options.processorOptions.type || 'lowpass';
    
    // Filtre katsayıları
    this.calculateCoefficients();
  }
  
  calculateCoefficients() {
    const omega = 2 * Math.PI * this.cutoff / sampleRate;
    const cosOmega = Math.cos(omega);
    const sinOmega = Math.sin(omega);
    const alpha = sinOmega / (2 * this.resonance);
    
    switch (this.type) {
      case 'lowpass':
        this.b0 = (1 - cosOmega) / 2;
        this.b1 = 1 - cosOmega;
        this.b2 = (1 - cosOmega) / 2;
        this.a0 = 1 + alpha;
        this.a1 = -2 * cosOmega;
        this.a2 = 1 - alpha;
        break;
      case 'highpass':
        this.b0 = (1 + cosOmega) / 2;
        this.b1 = -(1 + cosOmega);
        this.b2 = (1 + cosOmega) / 2;
        this.a0 = 1 + alpha;
        this.a1 = -2 * cosOmega;
        this.a2 = 1 - alpha;
        break;
    }
  }
  
  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];
    
    if (input.length > 0) {
      for (let channel = 0; channel < input.length; channel++) {
        const inputChannel = input[channel];
        const outputChannel = output[channel];
        
        // Filtre uygula
        for (let i = 0; i < inputChannel.length; i++) {
          const sample = inputChannel[i];
          const filtered = (this.b0 * sample + this.b1 * this.x1 + this.b2 * this.x2 - 
                          this.a1 * this.y1 - this.a2 * this.y2) / this.a0;
          
          outputChannel[i] = filtered;
          
          // Geçmiş değerleri güncelle
          this.x2 = this.x1;
          this.x1 = sample;
          this.y2 = this.y1;
          this.y1 = filtered;
        }
      }
    }
    
    return true;
  }
}

registerProcessor('audio-filter', AudioFilter);
```

### 2. **Ses Kompresyonu**
```javascript
// audio-compressor.js (AudioWorklet)
class AudioCompressor extends AudioWorkletProcessor {
  constructor(options) {
    super();
    this.threshold = options.processorOptions.threshold || -20;
    this.ratio = options.processorOptions.ratio || 4;
    this.attack = options.processorOptions.attack || 0.003;
    this.release = options.processorOptions.release || 0.1;
    
    this.envelope = 0;
    this.gain = 1;
  }
  
  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];
    
    if (input.length > 0) {
      for (let channel = 0; channel < input.length; channel++) {
        const inputChannel = input[channel];
        const outputChannel = output[channel];
        
        for (let i = 0; i < inputChannel.length; i++) {
          const sample = inputChannel[i];
          const absSample = Math.abs(sample);
          
          // Envelope follower
          if (absSample > this.envelope) {
            this.envelope += (absSample - this.envelope) * this.attack;
          } else {
            this.envelope += (absSample - this.envelope) * this.release;
          }
          
          // Kompresyon hesapla
          const db = 20 * Math.log10(this.envelope);
          if (db > this.threshold) {
            const overThreshold = db - this.threshold;
            const compressedDb = this.threshold + (overThreshold / this.ratio);
            this.gain = Math.pow(10, (compressedDb - db) / 20);
          } else {
            this.gain = 1;
          }
          
          outputChannel[i] = sample * this.gain;
        }
      }
    }
    
    return true;
  }
}

registerProcessor('audio-compressor', AudioCompressor);
```

### 3. **Ses Analizi**
```javascript
// audio-analyzer.js (AudioWorklet)
class AudioAnalyzer extends AudioWorkletProcessor {
  constructor() {
    super();
    this.fftSize = 2048;
    this.buffer = new Float32Array(this.fftSize);
    this.bufferIndex = 0;
    this.analysisInterval = 100; // ms
    this.lastAnalysis = 0;
  }
  
  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];
    
    if (input.length > 0) {
      // Girişi çıkışa kopyala
      for (let channel = 0; channel < input.length; channel++) {
        output[channel].set(input[channel]);
      }
      
      // Analiz için buffer'a ekle
      const inputChannel = input[0];
      for (let i = 0; i < inputChannel.length; i++) {
        this.buffer[this.bufferIndex] = inputChannel[i];
        this.bufferIndex = (this.bufferIndex + 1) % this.fftSize;
      }
      
      // Periyodik analiz
      const now = currentTime * 1000;
      if (now - this.lastAnalysis > this.analysisInterval) {
        this.analyzeAudio();
        this.lastAnalysis = now;
      }
    }
    
    return true;
  }
  
  analyzeAudio() {
    // Basit spektrum analizi
    let sum = 0;
    let max = 0;
    
    for (let i = 0; i < this.buffer.length; i++) {
      const sample = Math.abs(this.buffer[i]);
      sum += sample;
      max = Math.max(max, sample);
    }
    
    const rms = Math.sqrt(sum / this.buffer.length);
    const peak = max;
    
    // Ana thread'e gönder
    this.port.postMessage({
      type: 'analysis',
      rms: rms,
      peak: peak,
      timestamp: currentTime
    });
  }
}

registerProcessor('audio-analyzer', AudioAnalyzer);
```

### 4. **Ses Sentezi**
```javascript
// audio-synthesizer.js (AudioWorklet)
class AudioSynthesizer extends AudioWorkletProcessor {
  constructor(options) {
    super();
    this.sampleRate = sampleRate;
    this.phase = 0;
    this.frequency = 440;
    this.waveform = 'sine';
    this.amplitude = 0.5;
    
    // Port mesajlarını dinle
    this.port.onmessage = (event) => {
      const { type, data } = event.data;
      
      switch (type) {
        case 'frequency':
          this.frequency = data;
          break;
        case 'waveform':
          this.waveform = data;
          break;
        case 'amplitude':
          this.amplitude = data;
          break;
      }
    };
  }
  
  process(inputs, outputs, parameters) {
    const output = outputs[0];
    
    if (output.length > 0) {
      const outputChannel = output[0];
      const phaseIncrement = (2 * Math.PI * this.frequency) / this.sampleRate;
      
      for (let i = 0; i < outputChannel.length; i++) {
        let sample = 0;
        
        switch (this.waveform) {
          case 'sine':
            sample = Math.sin(this.phase);
            break;
          case 'square':
            sample = Math.sin(this.phase) > 0 ? 1 : -1;
            break;
          case 'sawtooth':
            sample = 2 * (this.phase / (2 * Math.PI)) - 1;
            break;
          case 'triangle':
            sample = 2 * Math.abs(2 * (this.phase / (2 * Math.PI)) - 1) - 1;
            break;
        }
        
        outputChannel[i] = sample * this.amplitude;
        this.phase += phaseIncrement;
        
        // Phase wrap
        if (this.phase >= 2 * Math.PI) {
          this.phase -= 2 * Math.PI;
        }
      }
    }
    
    return true;
  }
}

registerProcessor('audio-synthesizer', AudioSynthesizer);
```

## Alternatifler

### 1. **Ana Thread'de Ses İşleme**
```javascript
// Basit ama performans sorunlu
function processAudio(audioData) {
  // Ağır işlemler UI'ı dondurabilir
  return audioData.map(sample => sample * 0.5);
}
```

### 2. **Web Workers**
```javascript
// Ses işleme için uygun değil
const worker = new Worker('audio-worker.js');
worker.postMessage(audioData);
```

### 3. **OfflineAudioContext**
```javascript
// Offline işleme için
const offlineContext = new OfflineAudioContext(2, 44100, 44100);
const source = offlineContext.createBufferSource();
// ... işleme
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Real-time ses işleme
- Ağır ses hesaplamaları
- Ses efektleri
- Ses analizi
- Ses sentezi
- Performans kritik uygulamalar

### ❌ **Kullanmayın:**
- Basit ses çalma
- Offline ses işleme
- Tek seferlik işlemler
- Basit ses manipülasyonu

## Kullanmazsak Ne Olur?

### 1. **Performans Sorunları**
- Ana thread donabilir
- UI yanıt vermeyebilir
- Ses gecikmeleri olabilir

### 2. **Kullanıcı Deneyimi**
- Uygulama donabilir
- Ses kesintileri
- Responsive olmayan arayüz

### 3. **Sınırlı İşlem Kapasitesi**
- Karmaşık ses işleme yapılamaz
- Real-time analiz zor
- Çoklu ses kanalı yönetimi zor

## Modern Alternatifler

### 1. **WebAssembly + AudioWorklet**
```javascript
// WASM ile ses işleme
class WASMAudioProcessor extends AudioWorkletProcessor {
  constructor() {
    super();
    this.wasmModule = null;
    this.initWASM();
  }
  
  async initWASM() {
    const wasmModule = await WebAssembly.instantiateStreaming(
      fetch('audio-processor.wasm')
    );
    this.wasmModule = wasmModule;
  }
  
  process(inputs, outputs, parameters) {
    if (this.wasmModule) {
      // WASM ile ses işleme
      const result = this.wasmModule.instance.exports.processAudio(
        inputs[0][0], outputs[0][0]
      );
    }
    return true;
  }
}
```

### 2. **SharedArrayBuffer + AudioWorklet**
```javascript
// Paylaşılan bellek ile veri aktarımı
class SharedAudioProcessor extends AudioWorkletProcessor {
  constructor() {
    super();
    this.sharedBuffer = new SharedArrayBuffer(1024 * 4);
    this.sharedArray = new Float32Array(this.sharedBuffer);
  }
  
  process(inputs, outputs, parameters) {
    // Paylaşılan bellek kullan
    const input = inputs[0][0];
    for (let i = 0; i < input.length; i++) {
      this.sharedArray[i] = input[i];
    }
    
    return true;
  }
}
```

## Özet

**AudioWorklet API**, modern web uygulamalarında **performanslı ses işleme** sağlar. Arka planda çalışarak:

- ✅ Ana thread'i bloke etmez
- ✅ Real-time ses işleme yapar
- ✅ Performansı optimize eder
- ✅ Karmaşık ses algoritmaları çalıştırır

**Kullanın** - özellikle real-time ses işleme, ses efektleri, ses analizi ve performans kritik ses uygulamaları için.

```javascript
// Basit AudioWorklet örneği
// Ana thread
const audioContext = new AudioContext();
await audioContext.audioWorklet.addModule('audio-processor.js');
const audioWorkletNode = new AudioWorkletNode(audioContext, 'audio-processor');
```

---

## B - Bölümü

### Background Fetch API
Arka planda dosya indirme işlemleri için API - Offline-first uygulamalar.

## Background Fetch Nedir?

Background Fetch API, kullanıcı uygulamayı kullanmıyorken bile arka planda dosya indirme işlemlerini gerçekleştirmek için kullanılan bir API'dir. Özellikle offline-first uygulamalar ve büyük dosya indirmeleri için kritik öneme sahiptir.

## Neden Kullanılır?

### 1. **Offline-First Uygulamalar**
```javascript
// ❌ Normal fetch - kullanıcı uygulamayı kullanmalı
async function downloadContent() {
  const response = await fetch('/api/large-content');
  const data = await response.json();
  // Kullanıcı uygulamayı terk ederse indirme durur
}
```

```javascript
// ✅ Background Fetch - arka planda devam eder
if ('serviceWorker' in navigator && 'BackgroundFetchManager' in window) {
  const registration = await navigator.serviceWorker.ready();
  const bgFetch = await registration.backgroundFetch.fetch('content-download', [
    '/api/large-content',
    '/api/images',
    '/api/videos'
  ]);
  
  // Kullanıcı uygulamayı terk etse bile indirme devam eder
}
```

### 2. **Büyük Dosya İndirmeleri**
```javascript
// Çoklu dosya indirme
async function downloadMultipleFiles() {
  const registration = await navigator.serviceWorker.ready();
  
  const files = [
    '/api/dataset1.json',
    '/api/dataset2.json',
    '/api/dataset3.json',
    '/api/images/photo1.jpg',
    '/api/images/photo2.jpg'
  ];
  
  const bgFetch = await registration.backgroundFetch.fetch('dataset-download', files);
  
  // İndirme durumunu takip et
  bgFetch.addEventListener('progress', () => {
    console.log('İndirme ilerlemesi:', bgFetch.downloaded, '/', bgFetch.total);
  });
}
```

## Pratik Kullanım Örnekleri

### 1. **Temel Background Fetch**
```javascript
class BackgroundDownloader {
  constructor() {
    this.registration = null;
    this.activeDownloads = new Map();
  }
  
  async init() {
    if ('serviceWorker' in navigator && 'BackgroundFetchManager' in window) {
      this.registration = await navigator.serviceWorker.ready();
      return true;
    }
    return false;
  }
  
  async startDownload(id, urls, options = {}) {
    if (!this.registration) {
      throw new Error('Background Fetch desteklenmiyor');
    }
    
    const bgFetch = await this.registration.backgroundFetch.fetch(id, urls, {
      title: options.title || 'İndiriliyor...',
      icons: options.icons || [],
      downloadTotal: options.downloadTotal || urls.length
    });
    
    this.activeDownloads.set(id, bgFetch);
    
    // Event listener'ları ekle
    bgFetch.addEventListener('progress', () => {
      this.onProgress(id, bgFetch);
    });
    
    bgFetch.addEventListener('abort', () => {
      this.onAbort(id, bgFetch);
    });
    
    bgFetch.addEventListener('complete', () => {
      this.onComplete(id, bgFetch);
    });
    
    return bgFetch;
  }
  
  onProgress(id, bgFetch) {
    const progress = {
      downloaded: bgFetch.downloaded,
      total: bgFetch.total,
      percentage: (bgFetch.downloaded / bgFetch.total) * 100
    };
    
    console.log(`İndirme ${id}:`, progress);
    
    // UI güncelle
    this.updateProgressUI(id, progress);
  }
  
  onAbort(id, bgFetch) {
    console.log(`İndirme ${id} iptal edildi`);
    this.activeDownloads.delete(id);
  }
  
  onComplete(id, bgFetch) {
    console.log(`İndirme ${id} tamamlandı`);
    this.activeDownloads.delete(id);
    
    // İndirilen dosyaları işle
    this.processDownloadedFiles(id, bgFetch);
  }
  
  updateProgressUI(id, progress) {
    const progressElement = document.getElementById(`progress-${id}`);
    if (progressElement) {
      progressElement.value = progress.percentage;
      progressElement.textContent = `${Math.round(progress.percentage)}%`;
    }
  }
  
  async processDownloadedFiles(id, bgFetch) {
    // İndirilen dosyaları cache'e ekle
    const cache = await caches.open('downloads');
    
    for (const [url, response] of bgFetch.matchAll()) {
      await cache.put(url, response);
    }
    
    console.log(`İndirme ${id} cache'e eklendi`);
  }
  
  async cancelDownload(id) {
    const bgFetch = this.activeDownloads.get(id);
    if (bgFetch) {
      await bgFetch.abort();
    }
  }
  
  getActiveDownloads() {
    return Array.from(this.activeDownloads.keys());
  }
}

// Kullanım
const downloader = new BackgroundDownloader();

// İndiriciyi başlat
downloader.init().then(supported => {
  if (supported) {
    console.log('Background Fetch destekleniyor');
  } else {
    console.log('Background Fetch desteklenmiyor');
  }
});

// İndirme başlat
document.getElementById('downloadBtn').onclick = async () => {
  const urls = [
    '/api/large-file1.zip',
    '/api/large-file2.zip',
    '/api/large-file3.zip'
  ];
  
  await downloader.startDownload('large-files', urls, {
    title: 'Büyük dosyalar indiriliyor...',
    downloadTotal: urls.length
  });
};
```

### 2. **Service Worker ile Background Fetch**
```javascript
// service-worker.js
self.addEventListener('backgroundfetch', event => {
  console.log('Background fetch başladı:', event.registration.id);
  
  event.waitUntil(handleBackgroundFetch(event));
});

async function handleBackgroundFetch(event) {
  const { registration } = event;
  
  // İndirme tamamlandığında
  if (registration.result === 'success') {
    console.log('İndirme başarılı:', registration.id);
    
    // Dosyaları cache'e ekle
    const cache = await caches.open('background-downloads');
    
    for (const [url, response] of registration.matchAll()) {
      await cache.put(url, response);
    }
    
    // Kullanıcıya bildirim gönder
    self.registration.showNotification('İndirme Tamamlandı', {
      body: `${registration.id} indirmesi tamamlandı`,
      icon: '/icon-192.png',
      tag: 'download-complete'
    });
  } else if (registration.result === 'failure') {
    console.log('İndirme başarısız:', registration.id);
    
    // Hata bildirimi
    self.registration.showNotification('İndirme Başarısız', {
      body: `${registration.id} indirmesi başarısız oldu`,
      icon: '/icon-192.png',
      tag: 'download-failed'
    });
  }
}

// Background fetch iptal edildiğinde
self.addEventListener('backgroundfetchabort', event => {
  console.log('Background fetch iptal edildi:', event.registration.id);
});

// Background fetch tamamlandığında
self.addEventListener('backgroundfetchcomplete', event => {
  console.log('Background fetch tamamlandı:', event.registration.id);
});
```

### 3. **İndirme Yönetimi**
```javascript
class DownloadManager {
  constructor() {
    this.downloads = new Map();
    this.registration = null;
  }
  
  async init() {
    if ('serviceWorker' in navigator && 'BackgroundFetchManager' in window) {
      this.registration = await navigator.serviceWorker.ready();
      await this.loadExistingDownloads();
      return true;
    }
    return false;
  }
  
  async loadExistingDownloads() {
    const registrations = await this.registration.backgroundFetch.getAll();
    
    for (const registration of registrations) {
      this.downloads.set(registration.id, {
        registration,
        status: registration.result || 'downloading',
        progress: {
          downloaded: registration.downloaded,
          total: registration.total
        }
      });
    }
  }
  
  async startDownload(id, urls, options = {}) {
    const bgFetch = await this.registration.backgroundFetch.fetch(id, urls, {
      title: options.title || 'İndiriliyor...',
      icons: options.icons || [],
      downloadTotal: options.downloadTotal || urls.length
    });
    
    this.downloads.set(id, {
      registration: bgFetch,
      status: 'downloading',
      progress: { downloaded: 0, total: bgFetch.total }
    });
    
    return bgFetch;
  }
  
  async pauseDownload(id) {
    const download = this.downloads.get(id);
    if (download && download.status === 'downloading') {
      await download.registration.abort();
      download.status = 'paused';
    }
  }
  
  async resumeDownload(id) {
    const download = this.downloads.get(id);
    if (download && download.status === 'paused') {
      // Yeni indirme başlat
      const urls = Array.from(download.registration.matchAll()).map(([url]) => url);
      await this.startDownload(id, urls);
    }
  }
  
  async cancelDownload(id) {
    const download = this.downloads.get(id);
    if (download) {
      await download.registration.abort();
      this.downloads.delete(id);
    }
  }
  
  getDownloadStatus(id) {
    return this.downloads.get(id);
  }
  
  getAllDownloads() {
    return Array.from(this.downloads.entries()).map(([id, download]) => ({
      id,
      status: download.status,
      progress: download.progress
    }));
  }
  
  async clearCompletedDownloads() {
    for (const [id, download] of this.downloads.entries()) {
      if (download.status === 'success' || download.status === 'failure') {
        await download.registration.abort();
        this.downloads.delete(id);
      }
    }
  }
}

// Kullanım
const downloadManager = new DownloadManager();

// İndirme yöneticisini başlat
downloadManager.init().then(supported => {
  if (supported) {
    console.log('Download Manager hazır');
    
    // Mevcut indirmeleri göster
    const downloads = downloadManager.getAllDownloads();
    console.log('Mevcut indirmeler:', downloads);
  }
});

// İndirme başlat
document.getElementById('startDownload').onclick = async () => {
  const urls = [
    '/api/content1.json',
    '/api/content2.json',
    '/api/content3.json'
  ];
  
  await downloadManager.startDownload('content-download', urls, {
    title: 'İçerik indiriliyor...'
  });
};

// İndirme durumunu kontrol et
document.getElementById('checkStatus').onclick = () => {
  const status = downloadManager.getDownloadStatus('content-download');
  console.log('İndirme durumu:', status);
};
```

### 4. **Offline-First Uygulama**
```javascript
class OfflineFirstApp {
  constructor() {
    this.downloadManager = new DownloadManager();
    this.cacheManager = new CacheManager();
  }
  
  async init() {
    // Service Worker'ı kaydet
    await this.registerServiceWorker();
    
    // Background Fetch'i başlat
    const supported = await this.downloadManager.init();
    if (supported) {
      console.log('Offline-first uygulama hazır');
    }
    
    // Offline durumunu kontrol et
    this.checkOfflineStatus();
  }
  
  async registerServiceWorker() {
    if ('serviceWorker' in navigator) {
      const registration = await navigator.serviceWorker.register('/sw.js');
      console.log('Service Worker kaydedildi:', registration);
    }
  }
  
  async preloadContent() {
    // Önemli içerikleri önceden indir
    const criticalContent = [
      '/api/user-profile',
      '/api/settings',
      '/api/offline-data'
    ];
    
    await this.downloadManager.startDownload('critical-content', criticalContent, {
      title: 'Kritik içerik indiriliyor...'
    });
  }
  
  async syncWhenOnline() {
    // Online olduğunda senkronizasyon
    if (navigator.onLine) {
      const pendingDownloads = [
        '/api/sync-data',
        '/api/updates'
      ];
      
      await this.downloadManager.startDownload('sync-download', pendingDownloads, {
        title: 'Senkronizasyon...'
      });
    }
  }
  
  checkOfflineStatus() {
    if (navigator.onLine) {
      this.syncWhenOnline();
    } else {
      console.log('Offline modda çalışıyor');
    }
    
    // Online/offline event listener'ları
    window.addEventListener('online', () => {
      console.log('Online oldu');
      this.syncWhenOnline();
    });
    
    window.addEventListener('offline', () => {
      console.log('Offline oldu');
    });
  }
  
  async getCachedData(url) {
    // Cache'den veri al
    const cache = await caches.open('offline-cache');
    const response = await cache.match(url);
    
    if (response) {
      return await response.json();
    }
    
    // Cache'de yoksa network'ten al
    if (navigator.onLine) {
      const response = await fetch(url);
      const data = await response.json();
      
      // Cache'e ekle
      cache.put(url, response.clone());
      
      return data;
    }
    
    throw new Error('Veri bulunamadı ve offline');
  }
}

// Kullanım
const app = new OfflineFirstApp();
app.init();
```

## Alternatifler

### 1. **Normal Fetch**
```javascript
// Basit fetch - offline çalışmaz
async function fetchData(url) {
  const response = await fetch(url);
  return await response.json();
}
```

### 2. **Service Worker + Cache**
```javascript
// Manuel cache yönetimi
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        return response || fetch(event.request);
      })
  );
});
```

### 3. **IndexedDB**
```javascript
// Büyük veri saklama
const db = await openDB('offline-db', 1);
await db.put('content', data, 'key');
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Offline-first uygulamalar
- Büyük dosya indirmeleri
- İçerik önceden yükleme
- Senkronizasyon
- PWA uygulamaları
- Medya indirme uygulamaları

### ❌ **Kullanmayın:**
- Basit web uygulamaları
- Küçük dosya indirmeleri
- Real-time veri
- Tek seferlik işlemler

## Kullanmazsak Ne Olur?

### 1. **Offline Deneyimi Yok**
- İnternet bağlantısı olmadan çalışmaz
- Kullanıcı deneyimi kötü
- Veri kaybı riski

### 2. **Performans Sorunları**
- Her seferinde indirme
- Yavaş yükleme
- Gereksiz bandwidth kullanımı

### 3. **Kullanıcı Memnuniyetsizliği**
- Bekleme süreleri
- Kesintili deneyim
- Veri maliyeti

## Modern Alternatifler

### 1. **Workbox**
```javascript
// Google'ın PWA kütüphanesi
import { BackgroundSyncPlugin } from 'workbox-background-sync';
import { CacheFirst } from 'workbox-strategies';

const bgSyncPlugin = new BackgroundSyncPlugin('download-queue');
const cacheFirst = new CacheFirst({
  cacheName: 'downloads-cache',
  plugins: [bgSyncPlugin]
});
```

### 2. **PWA Builder**
```javascript
// Microsoft'un PWA aracı
// Otomatik background fetch yapılandırması
```

## Özet

**Background Fetch API**, modern web uygulamalarında **offline-first deneyim** sağlar. Arka planda indirme ile:

- ✅ Offline çalışma sağlar
- ✅ Kullanıcı deneyimini iyileştirir
- ✅ Performansı optimize eder
- ✅ PWA özelliklerini destekler

**Kullanın** - özellikle offline-first uygulamalar, büyük dosya indirmeleri ve PWA uygulamaları için.

```javascript
// Basit Background Fetch örneği
if ('serviceWorker' in navigator && 'BackgroundFetchManager' in window) {
  const registration = await navigator.serviceWorker.ready();
  const bgFetch = await registration.backgroundFetch.fetch('my-fetch', [
    '/api/data1',
    '/api/data2'
  ]);
}
```

### Background Sync API
Arka planda veri senkronizasyonu için API - Offline veri senkronizasyonu.

## Background Sync Nedir?

Background Sync API, kullanıcı offline olduğunda gerçekleştirilemeyen işlemleri, kullanıcı tekrar online olduğunda arka planda otomatik olarak gerçekleştirmek için kullanılan bir API'dir. Özellikle offline-first uygulamalar için kritik öneme sahiptir.

## Neden Kullanılır?

### 1. **Offline Veri Senkronizasyonu**
```javascript
// ❌ Normal fetch - offline'da çalışmaz
async function saveData(data) {
  try {
    const response = await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    return await response.json();
  } catch (error) {
    // Offline'da hata - veri kaybolur
    console.error('Veri kaydedilemedi:', error);
  }
}
```

```javascript
// ✅ Background Sync - offline'da kuyruğa alır
async function saveDataWithSync(data) {
  try {
    const response = await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    return await response.json();
  } catch (error) {
    // Offline'da - Background Sync'e kaydet
    await saveToSyncQueue('save-data', data);
    console.log('Veri senkronizasyon kuyruğuna eklendi');
  }
}

async function saveToSyncQueue(tag, data) {
  const registration = await navigator.serviceWorker.ready;
  await registration.sync.register(tag);
  
  // Veriyi IndexedDB'ye kaydet
  const db = await openDB('sync-queue', 1);
  await db.put('sync-queue', { tag, data, timestamp: Date.now() });
}
```

### 2. **Otomatik Senkronizasyon**
```javascript
// Service Worker içinde
self.addEventListener('sync', event => {
  if (event.tag === 'save-data') {
    event.waitUntil(processSyncQueue());
  }
});

async function processSyncQueue() {
  const db = await openDB('sync-queue', 1);
  const transactions = await db.getAll('sync-queue');
  
  for (const transaction of transactions) {
    try {
      await fetch('/api/save', {
        method: 'POST',
        body: JSON.stringify(transaction.data)
      });
      
      // Başarılı olursa kuyruktan sil
      await db.delete('sync-queue', transaction.id);
    } catch (error) {
      console.error('Senkronizasyon hatası:', error);
    }
  }
}
```

## Pratik Kullanım Örnekleri

### 1. **Temel Background Sync**
```javascript
class BackgroundSyncManager {
  constructor() {
    this.registration = null;
    this.db = null;
  }
  
  async init() {
    if ('serviceWorker' in navigator && 'sync' in window.ServiceWorkerRegistration.prototype) {
      this.registration = await navigator.serviceWorker.ready;
      this.db = await this.initDB();
      return true;
    }
    return false;
  }
  
  async initDB() {
    return await openDB('sync-queue', 1, {
      upgrade(db) {
        if (!db.objectStoreNames.contains('sync-queue')) {
          db.createObjectStore('sync-queue', { keyPath: 'id', autoIncrement: true });
        }
      }
    });
  }
  
  async addToSyncQueue(tag, data, options = {}) {
    const syncData = {
      tag,
      data,
      timestamp: Date.now(),
      retryCount: 0,
      maxRetries: options.maxRetries || 3,
      priority: options.priority || 'normal'
    };
    
    // IndexedDB'ye kaydet
    await this.db.put('sync-queue', syncData);
    
    // Background Sync'i kaydet
    await this.registration.sync.register(tag);
    
    console.log(`Senkronizasyon kuyruğuna eklendi: ${tag}`);
  }
  
  async getSyncQueue() {
    return await this.db.getAll('sync-queue');
  }
  
  async removeFromSyncQueue(id) {
    await this.db.delete('sync-queue', id);
  }
  
  async clearSyncQueue() {
    await this.db.clear('sync-queue');
  }
}

// Kullanım
const syncManager = new BackgroundSyncManager();

// Senkronizasyon yöneticisini başlat
syncManager.init().then(supported => {
  if (supported) {
    console.log('Background Sync destekleniyor');
  } else {
    console.log('Background Sync desteklenmiyor');
  }
});

// Veri kaydetme fonksiyonu
async function saveData(data) {
  try {
    const response = await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    
    if (response.ok) {
      console.log('Veri başarıyla kaydedildi');
      return await response.json();
    } else {
      throw new Error('Sunucu hatası');
    }
  } catch (error) {
    // Offline veya hata durumunda sync kuyruğuna ekle
    await syncManager.addToSyncQueue('save-data', data, {
      maxRetries: 5,
      priority: 'high'
    });
    
    console.log('Veri senkronizasyon kuyruğuna eklendi');
  }
}
```

### 2. **Service Worker ile Background Sync**
```javascript
// service-worker.js
class SyncProcessor {
  constructor() {
    this.db = null;
    this.initDB();
  }
  
  async initDB() {
    this.db = await openDB('sync-queue', 1, {
      upgrade(db) {
        if (!db.objectStoreNames.contains('sync-queue')) {
          db.createObjectStore('sync-queue', { keyPath: 'id', autoIncrement: true });
        }
      }
    });
  }
  
  async processSyncQueue(tag) {
    const transactions = await this.db.getAll('sync-queue');
    const filteredTransactions = transactions.filter(t => t.tag === tag);
    
    for (const transaction of filteredTransactions) {
      try {
        await this.processTransaction(transaction);
        await this.db.delete('sync-queue', transaction.id);
        console.log(`Senkronizasyon başarılı: ${transaction.tag}`);
      } catch (error) {
        console.error(`Senkronizasyon hatası: ${transaction.tag}`, error);
        await this.handleSyncError(transaction, error);
      }
    }
  }
  
  async processTransaction(transaction) {
    switch (transaction.tag) {
      case 'save-data':
        await this.saveData(transaction.data);
        break;
      case 'update-data':
        await this.updateData(transaction.data);
        break;
      case 'delete-data':
        await this.deleteData(transaction.data);
        break;
      default:
        throw new Error(`Bilinmeyen sync tag: ${transaction.tag}`);
    }
  }
  
  async saveData(data) {
    const response = await fetch('/api/save', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`Save failed: ${response.status}`);
    }
    
    return await response.json();
  }
  
  async updateData(data) {
    const response = await fetch('/api/update', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`Update failed: ${response.status}`);
    }
    
    return await response.json();
  }
  
  async deleteData(data) {
    const response = await fetch('/api/delete', {
      method: 'DELETE',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`Delete failed: ${response.status}`);
    }
  }
  
  async handleSyncError(transaction, error) {
    transaction.retryCount = (transaction.retryCount || 0) + 1;
    
    if (transaction.retryCount < transaction.maxRetries) {
      // Tekrar dene
      await this.db.put('sync-queue', transaction);
      console.log(`Senkronizasyon tekrar denenecek: ${transaction.tag}`);
    } else {
      // Maksimum deneme sayısına ulaşıldı
      await this.db.delete('sync-queue', transaction.id);
      console.error(`Senkronizasyon başarısız: ${transaction.tag}`);
      
      // Kullanıcıya bildirim gönder
      self.registration.showNotification('Senkronizasyon Hatası', {
        body: `${transaction.tag} işlemi başarısız oldu`,
        icon: '/icon-192.png',
        tag: 'sync-error'
      });
    }
  }
}

const syncProcessor = new SyncProcessor();

// Sync event listener
self.addEventListener('sync', event => {
  console.log('Background sync başladı:', event.tag);
  
  event.waitUntil(syncProcessor.processSyncQueue(event.tag));
});

// Online/offline event listener'ları
self.addEventListener('online', () => {
  console.log('Online oldu - senkronizasyon başlatılıyor');
});

self.addEventListener('offline', () => {
  console.log('Offline oldu');
});
```

### 3. **Gelişmiş Senkronizasyon Yönetimi**
```javascript
class AdvancedSyncManager {
  constructor() {
    this.syncManager = new BackgroundSyncManager();
    this.syncStrategies = new Map();
    this.conflictResolver = new ConflictResolver();
  }
  
  async init() {
    await this.syncManager.init();
    this.setupSyncStrategies();
  }
  
  setupSyncStrategies() {
    // Farklı veri tipleri için farklı stratejiler
    this.syncStrategies.set('user-profile', {
      priority: 'high',
      maxRetries: 5,
      retryDelay: 1000,
      conflictResolution: 'server-wins'
    });
    
    this.syncStrategies.set('user-settings', {
      priority: 'high',
      maxRetries: 3,
      retryDelay: 2000,
      conflictResolution: 'client-wins'
    });
    
    this.syncStrategies.set('user-data', {
      priority: 'normal',
      maxRetries: 3,
      retryDelay: 5000,
      conflictResolution: 'merge'
    });
  }
  
  async syncData(type, data, options = {}) {
    const strategy = this.syncStrategies.get(type) || {
      priority: 'normal',
      maxRetries: 3,
      retryDelay: 5000,
      conflictResolution: 'server-wins'
    };
    
    try {
      const response = await fetch(`/api/${type}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      
      if (response.ok) {
        return await response.json();
      } else if (response.status === 409) {
        // Conflict - çakışma çözümü
        return await this.resolveConflict(type, data, response);
      } else {
        throw new Error(`HTTP ${response.status}`);
      }
    } catch (error) {
      // Offline veya hata durumunda sync kuyruğuna ekle
      await this.syncManager.addToSyncQueue(`sync-${type}`, {
        type,
        data,
        strategy,
        timestamp: Date.now()
      }, strategy);
      
      console.log(`${type} senkronizasyon kuyruğuna eklendi`);
    }
  }
  
  async resolveConflict(type, clientData, serverResponse) {
    const strategy = this.syncStrategies.get(type);
    
    switch (strategy.conflictResolution) {
      case 'server-wins':
        return await serverResponse.json();
      case 'client-wins':
        return clientData;
      case 'merge':
        return await this.conflictResolver.merge(clientData, await serverResponse.json());
      default:
        throw new Error('Çakışma çözülemedi');
    }
  }
  
  async getSyncStatus() {
    const queue = await this.syncManager.getSyncQueue();
    return {
      total: queue.length,
      byType: this.groupByType(queue),
      byPriority: this.groupByPriority(queue)
    };
  }
  
  groupByType(queue) {
    const groups = {};
    queue.forEach(item => {
      const type = item.data.type;
      groups[type] = (groups[type] || 0) + 1;
    });
    return groups;
  }
  
  groupByPriority(queue) {
    const groups = { high: 0, normal: 0, low: 0 };
    queue.forEach(item => {
      const priority = item.data.strategy?.priority || 'normal';
      groups[priority]++;
    });
    return groups;
  }
}

class ConflictResolver {
  async merge(clientData, serverData) {
    // Basit merge stratejisi
    const merged = { ...serverData };
    
    for (const [key, value] of Object.entries(clientData)) {
      if (value !== null && value !== undefined) {
        if (Array.isArray(value)) {
          merged[key] = [...(merged[key] || []), ...value];
        } else if (typeof value === 'object') {
          merged[key] = { ...(merged[key] || {}), ...value };
        } else {
          merged[key] = value;
        }
      }
    }
    
    return merged;
  }
}

// Kullanım
const advancedSyncManager = new AdvancedSyncManager();
await advancedSyncManager.init();

// Veri senkronizasyonu
await advancedSyncManager.syncData('user-profile', {
  name: 'John Doe',
  email: 'john@example.com'
});

// Senkronizasyon durumu
const status = await advancedSyncManager.getSyncStatus();
console.log('Senkronizasyon durumu:', status);
```

### 4. **Offline-First Form Yönetimi**
```javascript
class OfflineFormManager {
  constructor() {
    this.syncManager = new BackgroundSyncManager();
    this.forms = new Map();
  }
  
  async init() {
    await this.syncManager.init();
    this.setupFormHandlers();
  }
  
  setupFormHandlers() {
    // Tüm formları dinle
    document.addEventListener('submit', (event) => {
      event.preventDefault();
      this.handleFormSubmit(event.target);
    });
  }
  
  async handleFormSubmit(form) {
    const formData = new FormData(form);
    const data = Object.fromEntries(formData.entries());
    
    // Form ID'si oluştur
    const formId = this.generateFormId(form);
    
    try {
      // Online'da gönder
      const response = await fetch(form.action, {
        method: form.method || 'POST',
        body: formData
      });
      
      if (response.ok) {
        console.log('Form başarıyla gönderildi');
        this.showSuccessMessage(form);
        this.clearForm(form);
      } else {
        throw new Error(`HTTP ${response.status}`);
      }
    } catch (error) {
      // Offline veya hata durumunda
      await this.saveFormOffline(formId, form, data);
      this.showOfflineMessage(form);
    }
  }
  
  async saveFormOffline(formId, form, data) {
    const offlineData = {
      formId,
      action: form.action,
      method: form.method || 'POST',
      data,
      timestamp: Date.now()
    };
    
    await this.syncManager.addToSyncQueue('submit-form', offlineData, {
      maxRetries: 5,
      priority: 'high'
    });
    
    // Local storage'a da kaydet
    localStorage.setItem(`form-${formId}`, JSON.stringify(offlineData));
  }
  
  generateFormId(form) {
    return `${form.action}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  showSuccessMessage(form) {
    const message = document.createElement('div');
    message.className = 'success-message';
    message.textContent = 'Form başarıyla gönderildi!';
    form.appendChild(message);
    
    setTimeout(() => {
      message.remove();
    }, 3000);
  }
  
  showOfflineMessage(form) {
    const message = document.createElement('div');
    message.className = 'offline-message';
    message.textContent = 'Form offline olarak kaydedildi. Online olduğunda gönderilecek.';
    form.appendChild(message);
    
    setTimeout(() => {
      message.remove();
    }, 5000);
  }
  
  clearForm(form) {
    form.reset();
  }
  
  async getOfflineForms() {
    const forms = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key.startsWith('form-')) {
        const formData = JSON.parse(localStorage.getItem(key));
        forms.push(formData);
      }
    }
    return forms;
  }
}

// Kullanım
const formManager = new OfflineFormManager();
await formManager.init();

// Offline formları listele
const offlineForms = await formManager.getOfflineForms();
console.log('Offline formlar:', offlineForms);
```

## Alternatifler

### 1. **Normal Fetch**
```javascript
// Basit fetch - offline çalışmaz
async function submitForm(data) {
  const response = await fetch('/api/submit', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  return await response.json();
}
```

### 2. **IndexedDB + Manual Sync**
```javascript
// Manuel senkronizasyon
async function saveOffline(data) {
  const db = await openDB('offline-data', 1);
  await db.put('offline-data', data);
}

async function syncWhenOnline() {
  if (navigator.onLine) {
    const db = await openDB('offline-data', 1);
    const data = await db.getAll('offline-data');
    
    for (const item of data) {
      await fetch('/api/sync', {
        method: 'POST',
        body: JSON.stringify(item)
      });
    }
  }
}
```

### 3. **Service Worker + Cache**
```javascript
// Cache tabanlı senkronizasyon
self.addEventListener('fetch', event => {
  if (event.request.method === 'POST') {
    event.respondWith(
      fetch(event.request)
        .catch(() => {
          // Offline'da cache'e kaydet
          return caches.open('offline-requests')
            .then(cache => cache.put(event.request, new Response('Offline')));
        })
    );
  }
});
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Offline-first uygulamalar
- Form gönderimi
- Veri senkronizasyonu
- PWA uygulamaları
- Mobil uygulamalar
- Kritik veri işlemleri

### ❌ **Kullanmayın:**
- Basit web uygulamaları
- Real-time veri
- Tek seferlik işlemler
- Offline çalışmayan uygulamalar

## Kullanmazsak Ne Olur?

### 1. **Veri Kaybı**
- Offline'da veri kaybolur
- Kullanıcı deneyimi kötü
- Güvenilirlik düşük

### 2. **Senkronizasyon Sorunları**
- Veri tutarsızlığı
- Çakışma problemleri
- Manuel senkronizasyon gerekir

### 3. **Kullanıcı Memnuniyetsizliği**
- Veri kaybı korkusu
- Güven sorunu
- Uygulama terk etme

## Modern Alternatifler

### 1. **Workbox Background Sync**
```javascript
// Google'ın PWA kütüphanesi
import { BackgroundSyncPlugin } from 'workbox-background-sync';

const bgSyncPlugin = new BackgroundSyncPlugin('form-queue', {
  maxRetentionTime: 24 * 60 // 24 saat
});

// Fetch stratejisine ekle
const networkFirst = new NetworkFirst({
  cacheName: 'forms-cache',
  plugins: [bgSyncPlugin]
});
```

### 2. **PouchDB + CouchDB**
```javascript
// Offline-first veritabanı
const db = new PouchDB('local-db');
const remoteDb = new PouchDB('http://localhost:5984/remote-db');

// Senkronizasyon
db.sync(remoteDb, {
  live: true,
  retry: true
});
```

## Özet

**Background Sync API**, modern web uygulamalarında **güvenilir veri senkronizasyonu** sağlar. Offline veri yönetimi ile:

- ✅ Veri kaybını önler
- ✅ Otomatik senkronizasyon yapar
- ✅ Kullanıcı deneyimini iyileştirir
- ✅ Offline-first uygulamalar destekler

**Kullanın** - özellikle offline-first uygulamalar, form yönetimi ve kritik veri işlemleri için.

```javascript
// Basit Background Sync örneği
// Service Worker içinde
self.addEventListener('sync', event => {
  if (event.tag === 'background-sync') {
    event.waitUntil(doBackgroundSync());
  }
});
```

### Badging API
Uygulama simgelerine rozet eklemek için API - PWA bildirim rozetleri.

## Badging API Nedir?

Badging API, PWA (Progressive Web App) uygulamalarının simgelerine sayısal rozetler eklemek için kullanılan bir API'dir. Kullanıcılara okunmamış mesaj sayısı, bildirim sayısı gibi bilgileri görsel olarak iletmek için kullanılır.

## Neden Kullanılır?

### 1. **Kullanıcı Bildirimi**
```javascript
// ❌ Rozet olmadan - kullanıcı bilgilendirilmez
function showNotification() {
  // Sadece bildirim göster
  new Notification('Yeni mesajınız var');
  // Kullanıcı kaç mesaj olduğunu bilmez
}
```

```javascript
// ✅ Badging API ile - görsel bilgi
async function showNotificationWithBadge() {
  // Bildirim göster
  new Notification('Yeni mesajınız var');
  
  // Rozet ekle
  if ('setAppBadge' in navigator) {
    const unreadCount = await getUnreadMessageCount();
    await navigator.setAppBadge(unreadCount);
  }
}
```

### 2. **PWA Deneyimi**
```javascript
// PWA uygulamasında rozet yönetimi
class BadgeManager {
  constructor() {
    this.isSupported = 'setAppBadge' in navigator;
  }
  
  async setBadge(count) {
    if (this.isSupported && count > 0) {
      await navigator.setAppBadge(count);
    }
  }
  
  async clearBadge() {
    if (this.isSupported) {
      await navigator.clearAppBadge();
    }
  }
  
  async updateBadge(newCount) {
    if (newCount === 0) {
      await this.clearBadge();
    } else {
      await this.setBadge(newCount);
    }
  }
}

// Kullanım
const badgeManager = new BadgeManager();
await badgeManager.setBadge(5); // 5 rozet
```

## Pratik Kullanım Örnekleri

### 1. **Mesajlaşma Uygulaması**
```javascript
class MessagingApp {
  constructor() {
    this.badgeManager = new BadgeManager();
    this.unreadMessages = 0;
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    // Yeni mesaj geldiğinde
    this.addEventListener('newMessage', (event) => {
      this.handleNewMessage(event.detail);
    });
    
    // Mesaj okunduğunda
    this.addEventListener('messageRead', (event) => {
      this.handleMessageRead(event.detail);
    });
    
    // Uygulama aktif olduğunda
    document.addEventListener('visibilitychange', () => {
      if (!document.hidden) {
        this.clearBadge();
      }
    });
  }
  
  async handleNewMessage(message) {
    this.unreadMessages++;
    
    // Rozet güncelle
    await this.badgeManager.setBadge(this.unreadMessages);
    
    // Bildirim göster
    if (Notification.permission === 'granted') {
      new Notification('Yeni Mesaj', {
        body: message.content,
        icon: '/icon-192.png',
        tag: 'new-message'
      });
    }
  }
  
  async handleMessageRead(messageId) {
    this.unreadMessages = Math.max(0, this.unreadMessages - 1);
    
    // Rozet güncelle
    await this.badgeManager.updateBadge(this.unreadMessages);
  }
  
  async clearBadge() {
    this.unreadMessages = 0;
    await this.badgeManager.clearBadge();
  }
  
  async getUnreadCount() {
    // Sunucudan okunmamış mesaj sayısını al
    try {
      const response = await fetch('/api/unread-count');
      const data = await response.json();
      this.unreadMessages = data.count;
      await this.badgeManager.updateBadge(this.unreadMessages);
    } catch (error) {
      console.error('Okunmamış mesaj sayısı alınamadı:', error);
    }
  }
}

// Kullanım
const messagingApp = new MessagingApp();

// Uygulama başlatıldığında
messagingApp.getUnreadCount();
```

### 2. **E-posta Uygulaması**
```javascript
class EmailApp {
  constructor() {
    this.badgeManager = new BadgeManager();
    this.unreadEmails = 0;
    this.setupEmailSync();
  }
  
  async setupEmailSync() {
    // E-posta senkronizasyonu
    setInterval(async () => {
      await this.syncEmails();
    }, 30000); // 30 saniyede bir
    
    // İlk yükleme
    await this.syncEmails();
  }
  
  async syncEmails() {
    try {
      const response = await fetch('/api/emails/unread');
      const emails = await response.json();
      
      this.unreadEmails = emails.length;
      await this.badgeManager.updateBadge(this.unreadEmails);
      
      // Yeni e-postalar varsa bildirim göster
      if (emails.length > 0) {
        this.showEmailNotification(emails[0]);
      }
    } catch (error) {
      console.error('E-posta senkronizasyonu hatası:', error);
    }
  }
  
  showEmailNotification(email) {
    if (Notification.permission === 'granted') {
      new Notification(`Yeni E-posta: ${email.subject}`, {
        body: email.sender,
        icon: '/icon-192.png',
        tag: 'new-email'
      });
    }
  }
  
  async markAsRead(emailId) {
    try {
      await fetch(`/api/emails/${emailId}/read`, {
        method: 'PUT'
      });
      
      this.unreadEmails = Math.max(0, this.unreadEmails - 1);
      await this.badgeManager.updateBadge(this.unreadEmails);
    } catch (error) {
      console.error('E-posta okundu olarak işaretlenemedi:', error);
    }
  }
  
  async markAllAsRead() {
    try {
      await fetch('/api/emails/mark-all-read', {
        method: 'PUT'
      });
      
      this.unreadEmails = 0;
      await this.badgeManager.clearBadge();
    } catch (error) {
      console.error('Tüm e-postalar okundu olarak işaretlenemedi:', error);
    }
  }
}

// Kullanım
const emailApp = new EmailApp();
```

### 3. **Sosyal Medya Uygulaması**
```javascript
class SocialMediaApp {
  constructor() {
    this.badgeManager = new BadgeManager();
    this.notifications = {
      likes: 0,
      comments: 0,
      follows: 0,
      mentions: 0
    };
    this.setupRealTimeUpdates();
  }
  
  setupRealTimeUpdates() {
    // WebSocket bağlantısı
    this.socket = new WebSocket('wss://api.example.com/notifications');
    
    this.socket.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      this.handleNotification(notification);
    };
  }
  
  handleNotification(notification) {
    switch (notification.type) {
      case 'like':
        this.notifications.likes++;
        break;
      case 'comment':
        this.notifications.comments++;
        break;
      case 'follow':
        this.notifications.follows++;
        break;
      case 'mention':
        this.notifications.mentions++;
        break;
    }
    
    this.updateBadge();
    this.showNotification(notification);
  }
  
  updateBadge() {
    const totalNotifications = Object.values(this.notifications)
      .reduce((sum, count) => sum + count, 0);
    
    this.badgeManager.updateBadge(totalNotifications);
  }
  
  showNotification(notification) {
    if (Notification.permission === 'granted') {
      const title = this.getNotificationTitle(notification.type);
      const body = this.getNotificationBody(notification);
      
      new Notification(title, {
        body,
        icon: '/icon-192.png',
        tag: notification.type
      });
    }
  }
  
  getNotificationTitle(type) {
    const titles = {
      like: 'Beğeni Aldınız',
      comment: 'Yorum Yapıldı',
      follow: 'Yeni Takipçi',
      mention: 'Bahsedildiniz'
    };
    return titles[type] || 'Yeni Bildirim';
  }
  
  getNotificationBody(notification) {
    return `${notification.user} ${notification.action}`;
  }
  
  async clearNotifications(type = null) {
    if (type) {
      this.notifications[type] = 0;
    } else {
      // Tüm bildirimleri temizle
      Object.keys(this.notifications).forEach(key => {
        this.notifications[key] = 0;
      });
    }
    
    this.updateBadge();
  }
}

// Kullanım
const socialApp = new SocialMediaApp();
```

### 4. **Görev Yönetimi Uygulaması**
```javascript
class TaskManager {
  constructor() {
    this.badgeManager = new BadgeManager();
    this.pendingTasks = 0;
    this.overdueTasks = 0;
    this.setupTaskSync();
  }
  
  async setupTaskSync() {
    // Görevleri senkronize et
    await this.syncTasks();
    
    // Periyodik güncelleme
    setInterval(async () => {
      await this.syncTasks();
    }, 60000); // 1 dakikada bir
  }
  
  async syncTasks() {
    try {
      const response = await fetch('/api/tasks');
      const tasks = await response.json();
      
      this.pendingTasks = tasks.filter(task => 
        task.status === 'pending' && !task.overdue
      ).length;
      
      this.overdueTasks = tasks.filter(task => 
        task.overdue
      ).length;
      
      // Öncelik: geciken görevler
      const badgeCount = this.overdueTasks > 0 ? 
        this.overdueTasks : this.pendingTasks;
      
      await this.badgeManager.updateBadge(badgeCount);
      
      // Geciken görevler varsa bildirim göster
      if (this.overdueTasks > 0) {
        this.showOverdueNotification();
      }
    } catch (error) {
      console.error('Görev senkronizasyonu hatası:', error);
    }
  }
  
  showOverdueNotification() {
    if (Notification.permission === 'granted') {
      new Notification('Geciken Görevler', {
        body: `${this.overdueTasks} görev gecikti`,
        icon: '/icon-192.png',
        tag: 'overdue-tasks'
      });
    }
  }
  
  async completeTask(taskId) {
    try {
      await fetch(`/api/tasks/${taskId}/complete`, {
        method: 'PUT'
      });
      
      // Görev sayısını güncelle
      this.pendingTasks = Math.max(0, this.pendingTasks - 1);
      await this.badgeManager.updateBadge(this.pendingTasks);
    } catch (error) {
      console.error('Görev tamamlanamadı:', error);
    }
  }
  
  async addTask(task) {
    try {
      await fetch('/api/tasks', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(task)
      });
      
      this.pendingTasks++;
      await this.badgeManager.updateBadge(this.pendingTasks);
    } catch (error) {
      console.error('Görev eklenemedi:', error);
    }
  }
}

// Kullanım
const taskManager = new TaskManager();
```

## Alternatifler

### 1. **Document Title**
```javascript
// Basit title güncelleme
function updateTitle(count) {
  if (count > 0) {
    document.title = `(${count}) Uygulama Adı`;
  } else {
    document.title = 'Uygulama Adı';
  }
}
```

### 2. **Favicon Değiştirme**
```javascript
// Favicon ile rozet
function updateFavicon(count) {
  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');
  
  // Favicon çiz
  ctx.fillStyle = '#ff0000';
  ctx.fillRect(0, 0, 32, 32);
  
  if (count > 0) {
    ctx.fillStyle = '#ffffff';
    ctx.font = '16px Arial';
    ctx.fillText(count.toString(), 8, 20);
  }
  
  // Favicon'u güncelle
  const link = document.querySelector('link[rel="icon"]');
  link.href = canvas.toDataURL();
}
```

### 3. **Notification API**
```javascript
// Sadece bildirim
function showNotification(title, body) {
  if (Notification.permission === 'granted') {
    new Notification(title, { body });
  }
}
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- PWA uygulamaları
- Mesajlaşma uygulamaları
- E-posta uygulamaları
- Sosyal medya uygulamaları
- Görev yönetimi uygulamaları
- Bildirim gerektiren uygulamalar

### ❌ **Kullanmayın:**
- Basit web siteleri
- Rozet gerektirmeyen uygulamalar
- Tek sayfalık uygulamalar
- Offline uygulamalar

## Kullanmazsak Ne Olur?

### 1. **Kullanıcı Bilgilendirilmez**
- Okunmamış içerik sayısı bilinmez
- Kullanıcı deneyimi kötü
- Etkileşim azalır

### 2. **PWA Deneyimi Eksik**
- Native app deneyimi yok
- Kullanıcı memnuniyetsizliği
- Uygulama terk etme

### 3. **Bildirim Karmaşası**
- Sadece bildirim yeterli değil
- Görsel geri bildirim eksik
- Kullanıcı karışıklığı

## Modern Alternatifler

### 1. **Web App Manifest**
```json
{
  "name": "Uygulama Adı",
  "short_name": "Uygulama",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ],
  "display": "standalone",
  "start_url": "/"
}
```

### 2. **Service Worker + Notifications**
```javascript
// Service Worker içinde
self.addEventListener('notificationclick', event => {
  event.notification.close();
  
  // Uygulamayı aç
  event.waitUntil(
    clients.openWindow('/')
  );
});
```

### 3. **Push API**
```javascript
// Push bildirimleri
navigator.serviceWorker.ready.then(registration => {
  return registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: 'your-vapid-key'
  });
});
```

## Özet

**Badging API**, modern PWA uygulamalarında **görsel bildirim** sağlar. Uygulama rozetleri ile:

- ✅ Kullanıcıyı bilgilendirir
- ✅ PWA deneyimini iyileştirir
- ✅ Etkileşimi artırır
- ✅ Native app deneyimi sağlar

**Kullanın** - özellikle PWA uygulamaları, mesajlaşma, e-posta ve bildirim gerektiren uygulamalar için.

```javascript
// Basit Badging API örneği
if ('setAppBadge' in navigator) {
  await navigator.setAppBadge(5); // 5 rozet göster
}

if ('clearAppBadge' in navigator) {
  await navigator.clearAppBadge(); // Rozeti temizle
}
```

### Battery Status API
Cihazın pil durumunu sorgulamak için API - Pil yönetimi ve optimizasyon.

## Battery Status API Nedir?

Battery Status API, web uygulamalarının cihazın pil durumunu sorgulamasına olanak tanıyan bir API'dir. Pil seviyesi, şarj durumu ve tahmini kalan süre gibi bilgileri sağlar. Özellikle mobil cihazlarda performans optimizasyonu için kritik öneme sahiptir.

## Neden Kullanılır?

### 1. **Performans Optimizasyonu**
```javascript
// ❌ Pil durumu bilinmeden - gereksiz kaynak kullanımı
function startHeavyTask() {
  // Ağır işlemler - pil tükenebilir
  setInterval(() => {
    processLargeData();
    updateUI();
  }, 100);
}
```

```javascript
// ✅ Pil durumuna göre optimizasyon
async function startOptimizedTask() {
  const battery = await navigator.getBattery();
  
  if (battery.level < 0.2) {
    // Düşük pil - hafif işlemler
    setInterval(() => {
      processLightData();
    }, 1000);
  } else if (battery.level < 0.5) {
    // Orta pil - normal işlemler
    setInterval(() => {
      processNormalData();
    }, 500);
  } else {
    // Yüksek pil - ağır işlemler
    setInterval(() => {
      processHeavyData();
    }, 100);
  }
}
```

### 2. **Kullanıcı Deneyimi**
```javascript
// Pil durumuna göre kullanıcı bilgilendirme
async function setupBatteryAwareness() {
  const battery = await navigator.getBattery();
  
  // Pil durumu değişikliklerini dinle
  battery.addEventListener('levelchange', () => {
    updateBatteryIndicator(battery.level);
  });
  
  battery.addEventListener('chargingchange', () => {
    updateChargingStatus(battery.charging);
  });
  
  // İlk durumu göster
  updateBatteryIndicator(battery.level);
  updateChargingStatus(battery.charging);
}

function updateBatteryIndicator(level) {
  const percentage = Math.round(level * 100);
  const indicator = document.getElementById('battery-indicator');
  
  indicator.textContent = `${percentage}%`;
  indicator.className = level < 0.2 ? 'low-battery' : 
                       level < 0.5 ? 'medium-battery' : 'high-battery';
}
```

## Pratik Kullanım Örnekleri

### 1. **Pil Yönetimi Sistemi**
```javascript
class BatteryManager {
  constructor() {
    this.battery = null;
    this.optimizationLevel = 'normal';
    this.listeners = new Map();
  }
  
  async init() {
    if ('getBattery' in navigator) {
      this.battery = await navigator.getBattery();
      this.setupEventListeners();
      this.updateOptimizationLevel();
      return true;
    }
    return false;
  }
  
  setupEventListeners() {
    this.battery.addEventListener('levelchange', () => {
      this.updateOptimizationLevel();
      this.notifyListeners('levelchange', this.battery.level);
    });
    
    this.battery.addEventListener('chargingchange', () => {
      this.notifyListeners('chargingchange', this.battery.charging);
    });
    
    this.battery.addEventListener('chargingtimechange', () => {
      this.notifyListeners('chargingtimechange', this.battery.chargingTime);
    });
    
    this.battery.addEventListener('dischargingtimechange', () => {
      this.notifyListeners('dischargingtimechange', this.battery.dischargingTime);
    });
  }
  
  updateOptimizationLevel() {
    const level = this.battery.level;
    
    if (level < 0.1) {
      this.optimizationLevel = 'critical';
    } else if (level < 0.2) {
      this.optimizationLevel = 'low';
    } else if (level < 0.5) {
      this.optimizationLevel = 'medium';
    } else {
      this.optimizationLevel = 'normal';
    }
    
    this.notifyListeners('optimizationchange', this.optimizationLevel);
  }
  
  addListener(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }
  
  notifyListeners(event, data) {
    if (this.listeners.has(event)) {
      this.listeners.get(event).forEach(callback => callback(data));
    }
  }
  
  getBatteryInfo() {
    return {
      level: this.battery.level,
      charging: this.battery.charging,
      chargingTime: this.battery.chargingTime,
      dischargingTime: this.battery.dischargingTime,
      optimizationLevel: this.optimizationLevel
    };
  }
  
  shouldOptimize() {
    return this.optimizationLevel !== 'normal';
  }
  
  getOptimizationRecommendations() {
    const recommendations = [];
    
    switch (this.optimizationLevel) {
      case 'critical':
        recommendations.push('Ağır işlemleri durdurun');
        recommendations.push('Animasyonları kapatın');
        recommendations.push('Arka plan güncellemelerini durdurun');
        break;
      case 'low':
        recommendations.push('Gereksiz işlemleri azaltın');
        recommendations.push('Animasyon sıklığını düşürün');
        break;
      case 'medium':
        recommendations.push('Performansı izleyin');
        break;
    }
    
    return recommendations;
  }
}

// Kullanım
const batteryManager = new BatteryManager();

batteryManager.init().then(supported => {
  if (supported) {
    console.log('Battery Manager hazır');
    
    // Pil durumu değişikliklerini dinle
    batteryManager.addListener('optimizationchange', (level) => {
      console.log(`Optimizasyon seviyesi: ${level}`);
      
      if (level === 'critical') {
        showBatteryWarning();
      }
    });
  }
});
```

### 2. **Performans Optimizasyonu**
```javascript
class PerformanceOptimizer {
  constructor() {
    this.batteryManager = new BatteryManager();
    this.optimizations = new Map();
    this.setupOptimizations();
  }
  
  async init() {
    await this.batteryManager.init();
    this.batteryManager.addListener('optimizationchange', (level) => {
      this.applyOptimizations(level);
    });
  }
  
  setupOptimizations() {
    // Animasyon optimizasyonları
    this.optimizations.set('animations', {
      normal: { duration: 300, easing: 'ease-in-out' },
      medium: { duration: 200, easing: 'ease' },
      low: { duration: 100, easing: 'linear' },
      critical: { duration: 0, easing: 'none' }
    });
    
    // Güncelleme sıklığı optimizasyonları
    this.optimizations.set('updates', {
      normal: 100,    // 100ms
      medium: 500,    // 500ms
      low: 1000,      // 1s
      critical: 5000  // 5s
    });
    
    // Ağ istekleri optimizasyonları
    this.optimizations.set('requests', {
      normal: true,   // Tüm istekler
      medium: true,   // Tüm istekler
      low: false,     // Sadece kritik istekler
      critical: false // Hiç istek yok
    });
  }
  
  applyOptimizations(level) {
    // Animasyon optimizasyonları
    this.optimizeAnimations(level);
    
    // Güncelleme sıklığı optimizasyonları
    this.optimizeUpdates(level);
    
    // Ağ istekleri optimizasyonları
    this.optimizeRequests(level);
    
    // UI optimizasyonları
    this.optimizeUI(level);
  }
  
  optimizeAnimations(level) {
    const animationConfig = this.optimizations.get('animations')[level];
    
    // CSS değişkenlerini güncelle
    document.documentElement.style.setProperty(
      '--animation-duration', 
      `${animationConfig.duration}ms`
    );
    document.documentElement.style.setProperty(
      '--animation-easing', 
      animationConfig.easing
    );
    
    // JavaScript animasyonlarını güncelle
    this.updateAnimationSettings(animationConfig);
  }
  
  optimizeUpdates(level) {
    const updateInterval = this.optimizations.get('updates')[level];
    
    // Mevcut güncellemeleri durdur
    if (this.updateTimer) {
      clearInterval(this.updateTimer);
    }
    
    // Yeni güncelleme sıklığı ayarla
    if (updateInterval > 0) {
      this.updateTimer = setInterval(() => {
        this.performUpdate();
      }, updateInterval);
    }
  }
  
  optimizeRequests(level) {
    const allowRequests = this.optimizations.get('requests')[level];
    
    // Ağ isteklerini kontrol et
    this.setRequestPolicy(allowRequests);
  }
  
  optimizeUI(level) {
    const body = document.body;
    
    // Pil seviyesine göre UI sınıfları
    body.className = body.className.replace(/battery-\w+/g, '');
    body.classList.add(`battery-${level}`);
    
    // Düşük pil uyarısı
    if (level === 'critical' || level === 'low') {
      this.showBatteryWarning(level);
    } else {
      this.hideBatteryWarning();
    }
  }
  
  showBatteryWarning(level) {
    let warning = document.getElementById('battery-warning');
    
    if (!warning) {
      warning = document.createElement('div');
      warning.id = 'battery-warning';
      warning.className = 'battery-warning';
      document.body.appendChild(warning);
    }
    
    const messages = {
      low: 'Düşük pil seviyesi - performans azaltıldı',
      critical: 'Kritik pil seviyesi - ağır işlemler durduruldu'
    };
    
    warning.textContent = messages[level];
    warning.style.display = 'block';
  }
  
  hideBatteryWarning() {
    const warning = document.getElementById('battery-warning');
    if (warning) {
      warning.style.display = 'none';
    }
  }
}

// Kullanım
const optimizer = new PerformanceOptimizer();
await optimizer.init();
```

### 3. **Akıllı İçerik Yönetimi**
```javascript
class SmartContentManager {
  constructor() {
    this.batteryManager = new BatteryManager();
    this.contentLevel = 'full';
    this.setupContentOptimizations();
  }
  
  async init() {
    await this.batteryManager.init();
    this.batteryManager.addListener('optimizationchange', (level) => {
      this.adjustContentLevel(level);
    });
  }
  
  setupContentOptimizations() {
    this.contentLevels = {
      normal: {
        images: 'high',
        videos: 'high',
        animations: 'full',
        updates: 'real-time'
      },
      medium: {
        images: 'medium',
        videos: 'medium',
        animations: 'reduced',
        updates: 'frequent'
      },
      low: {
        images: 'low',
        videos: 'low',
        animations: 'minimal',
        updates: 'periodic'
      },
      critical: {
        images: 'none',
        videos: 'none',
        animations: 'none',
        updates: 'minimal'
      }
    };
  }
  
  adjustContentLevel(batteryLevel) {
    this.contentLevel = batteryLevel;
    const config = this.contentLevels[batteryLevel];
    
    // Görsel içerik optimizasyonu
    this.optimizeImages(config.images);
    this.optimizeVideos(config.videos);
    this.optimizeAnimations(config.animations);
    this.optimizeUpdates(config.updates);
  }
  
  optimizeImages(quality) {
    const images = document.querySelectorAll('img');
    
    images.forEach(img => {
      switch (quality) {
        case 'high':
          img.src = img.dataset.highRes || img.src;
          break;
        case 'medium':
          img.src = img.dataset.mediumRes || img.src;
          break;
        case 'low':
          img.src = img.dataset.lowRes || img.src;
          break;
        case 'none':
          img.style.display = 'none';
          break;
      }
    });
  }
  
  optimizeVideos(quality) {
    const videos = document.querySelectorAll('video');
    
    videos.forEach(video => {
      switch (quality) {
        case 'high':
          video.playbackRate = 1;
          video.muted = false;
          break;
        case 'medium':
          video.playbackRate = 0.8;
          video.muted = false;
          break;
        case 'low':
          video.playbackRate = 0.5;
          video.muted = true;
          break;
        case 'none':
          video.pause();
          video.style.display = 'none';
          break;
      }
    });
  }
  
  optimizeAnimations(level) {
    const animatedElements = document.querySelectorAll('[data-animated]');
    
    animatedElements.forEach(element => {
      switch (level) {
        case 'full':
          element.style.animationPlayState = 'running';
          break;
        case 'reduced':
          element.style.animationDuration = '0.5s';
          break;
        case 'minimal':
          element.style.animationDuration = '0.2s';
          break;
        case 'none':
          element.style.animationPlayState = 'paused';
          break;
      }
    });
  }
  
  optimizeUpdates(frequency) {
    // Güncelleme sıklığını ayarla
    const intervals = {
      'real-time': 100,
      'frequent': 500,
      'periodic': 2000,
      'minimal': 10000
    };
    
    const interval = intervals[frequency];
    
    if (this.updateTimer) {
      clearInterval(this.updateTimer);
    }
    
    if (interval > 0) {
      this.updateTimer = setInterval(() => {
        this.updateContent();
      }, interval);
    }
  }
  
  updateContent() {
    // İçerik güncelleme mantığı
    console.log('İçerik güncellendi');
  }
}

// Kullanım
const contentManager = new SmartContentManager();
await contentManager.init();
```

### 4. **Pil Durumu Monitörü**
```javascript
class BatteryMonitor {
  constructor() {
    this.batteryManager = new BatteryManager();
    this.stats = {
      sessions: [],
      currentSession: null
    };
    this.setupMonitoring();
  }
  
  async init() {
    await this.batteryManager.init();
    this.startSession();
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    this.batteryManager.addListener('levelchange', (level) => {
      this.recordBatteryLevel(level);
    });
    
    this.batteryManager.addListener('chargingchange', (charging) => {
      this.recordChargingStatus(charging);
    });
  }
  
  startSession() {
    this.stats.currentSession = {
      startTime: Date.now(),
      batteryLevels: [],
      chargingEvents: [],
      optimizations: []
    };
  }
  
  recordBatteryLevel(level) {
    if (this.stats.currentSession) {
      this.stats.currentSession.batteryLevels.push({
        level,
        timestamp: Date.now()
      });
    }
  }
  
  recordChargingStatus(charging) {
    if (this.stats.currentSession) {
      this.stats.currentSession.chargingEvents.push({
        charging,
        timestamp: Date.now()
      });
    }
  }
  
  recordOptimization(level) {
    if (this.stats.currentSession) {
      this.stats.currentSession.optimizations.push({
        level,
        timestamp: Date.now()
      });
    }
  }
  
  endSession() {
    if (this.stats.currentSession) {
      this.stats.currentSession.endTime = Date.now();
      this.stats.sessions.push(this.stats.currentSession);
      this.stats.currentSession = null;
    }
  }
  
  getBatteryStats() {
    const current = this.stats.currentSession;
    const battery = this.batteryManager.getBatteryInfo();
    
    return {
      current: {
        level: battery.level,
        charging: battery.charging,
        optimizationLevel: battery.optimizationLevel,
        sessionDuration: current ? Date.now() - current.startTime : 0
      },
      history: this.stats.sessions,
      recommendations: this.batteryManager.getOptimizationRecommendations()
    };
  }
  
  exportStats() {
    const stats = this.getBatteryStats();
    const dataStr = JSON.stringify(stats, null, 2);
    const dataBlob = new Blob([dataStr], { type: 'application/json' });
    
    const link = document.createElement('a');
    link.href = URL.createObjectURL(dataBlob);
    link.download = `battery-stats-${Date.now()}.json`;
    link.click();
  }
}

// Kullanım
const monitor = new BatteryMonitor();
await monitor.init();

// İstatistikleri görüntüle
setInterval(() => {
  const stats = monitor.getBatteryStats();
  console.log('Pil istatistikleri:', stats);
}, 30000);
```

## Alternatifler

### 1. **Manuel Optimizasyon**
```javascript
// Kullanıcı tercihlerine göre optimizasyon
function setPerformanceMode(mode) {
  switch (mode) {
    case 'high':
      // Yüksek performans
      break;
    case 'balanced':
      // Dengeli performans
      break;
    case 'power-save':
      // Güç tasarrufu
      break;
  }
}
```

### 2. **Network Information API**
```javascript
// Ağ durumuna göre optimizasyon
if ('connection' in navigator) {
  const connection = navigator.connection;
  
  if (connection.effectiveType === 'slow-2g') {
    // Yavaş ağ - içerik azalt
  }
}
```

### 3. **Device Memory API**
```javascript
// Bellek durumuna göre optimizasyon
if ('deviceMemory' in navigator) {
  const memory = navigator.deviceMemory;
  
  if (memory < 4) {
    // Düşük bellek - optimizasyon uygula
  }
}
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Mobil web uygulamaları
- Performans kritik uygulamalar
- Uzun süreli kullanım uygulamaları
- Oyun uygulamaları
- Medya uygulamaları
- PWA uygulamaları

### ❌ **Kullanmayın:**
- Basit web siteleri
- Kısa süreli kullanım uygulamaları
- Masaüstü odaklı uygulamalar
- Pil optimizasyonu gerektirmeyen uygulamalar

## Kullanmazsak Ne Olur?

### 1. **Performans Sorunları**
- Gereksiz kaynak kullanımı
- Pil tükenmesi
- Uygulama donması

### 2. **Kullanıcı Deneyimi**
- Yavaş performans
- Beklenmeyen kapanmalar
- Memnuniyetsizlik

### 3. **Kaynak İsrafı**
- Aşırı CPU kullanımı
- Gereksiz ağ istekleri
- Bellek sızıntıları

## Modern Alternatifler

### 1. **Performance Observer API**
```javascript
// Performans izleme
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 100) {
      // Yavaş işlem - optimizasyon uygula
    }
  }
});

observer.observe({ entryTypes: ['measure'] });
```

### 2. **Intersection Observer API**
```javascript
// Görünürlük tabanlı optimizasyon
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Görünür - yükle
    } else {
      // Görünmez - durdur
    }
  });
});
```

### 3. **Page Visibility API**
```javascript
// Sayfa görünürlüğü
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Sayfa gizli - işlemleri durdur
  } else {
    // Sayfa görünür - işlemleri başlat
  }
});
```

## Özet

**Battery Status API**, modern web uygulamalarında **akıllı pil yönetimi** sağlar. Pil durumuna göre optimizasyon ile:

- ✅ Performansı optimize eder
- ✅ Pil ömrünü uzatır
- ✅ Kullanıcı deneyimini iyileştirir
- ✅ Kaynak kullanımını optimize eder

**Kullanın** - özellikle mobil uygulamalar, performans kritik uygulamalar ve uzun süreli kullanım uygulamaları için.

```javascript
// Basit Battery Status API örneği
navigator.getBattery().then(battery => {
  console.log(`Pil seviyesi: ${battery.level * 100}%`);
  console.log(`Şarj durumu: ${battery.charging ? 'Şarj oluyor' : 'Şarj olmuyor'}`);
});
```

### Beacon API
Küçük veri paketlerini arka planda göndermek için API - Güvenilir veri gönderimi.

## Beacon API Nedir?

Beacon API, web uygulamalarının küçük veri paketlerini arka planda güvenilir bir şekilde göndermesine olanak tanıyan bir API'dir. Özellikle analitik veriler, kullanıcı etkileşimleri ve sayfa kapatma olayları için kritik öneme sahiptir.

## Neden Kullanılır?

### 1. **Güvenilir Veri Gönderimi**
```javascript
// ❌ Normal fetch - sayfa kapatılırken başarısız olabilir
function sendAnalytics(data) {
  fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify(data)
  }).catch(error => {
    // Sayfa kapatılırken hata - veri kaybolur
    console.error('Analytics gönderilemedi:', error);
  });
}

// Sayfa kapatılırken
window.addEventListener('beforeunload', () => {
  sendAnalytics({ event: 'page_exit' }); // Başarısız olabilir
});
```

```javascript
// ✅ Beacon API - güvenilir gönderim
function sendAnalytics(data) {
  const success = navigator.sendBeacon('/api/analytics', JSON.stringify(data));
  
  if (success) {
    console.log('Analytics başarıyla gönderildi');
  } else {
    console.log('Analytics gönderilemedi');
  }
}

// Sayfa kapatılırken
window.addEventListener('beforeunload', () => {
  sendAnalytics({ event: 'page_exit' }); // Güvenilir gönderim
});
```

### 2. **Performans Optimizasyonu**
```javascript
// Beacon API ile performanslı veri gönderimi
class AnalyticsManager {
  constructor() {
    this.queue = [];
    this.batchSize = 10;
    this.flushInterval = 5000; // 5 saniye
    this.setupFlushing();
  }
  
  track(event, data = {}) {
    const eventData = {
      event,
      data,
      timestamp: Date.now(),
      url: window.location.href,
      userAgent: navigator.userAgent
    };
    
    this.queue.push(eventData);
    
    // Batch boyutuna ulaştıysa gönder
    if (this.queue.length >= this.batchSize) {
      this.flush();
    }
  }
  
  setupFlushing() {
    // Periyodik gönderim
    setInterval(() => {
      if (this.queue.length > 0) {
        this.flush();
      }
    }, this.flushInterval);
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.flush();
    });
    
    // Sayfa gizlendiğinde
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        this.flush();
      }
    });
  }
  
  flush() {
    if (this.queue.length === 0) return;
    
    const data = this.queue.splice(0); // Tüm veriyi al ve temizle
    
    const success = navigator.sendBeacon('/api/analytics/batch', JSON.stringify({
      events: data,
      batchId: Date.now()
    }));
    
    if (!success) {
      // Başarısız olursa tekrar kuyruğa ekle
      this.queue.unshift(...data);
    }
  }
}

// Kullanım
const analytics = new AnalyticsManager();
analytics.track('page_view');
analytics.track('button_click', { button: 'submit' });
```

## Pratik Kullanım Örnekleri

### 1. **Analitik Sistemi**
```javascript
class WebAnalytics {
  constructor(config = {}) {
    this.endpoint = config.endpoint || '/api/analytics';
    this.sessionId = this.generateSessionId();
    this.userId = config.userId || this.generateUserId();
    this.events = [];
    this.setupTracking();
  }
  
  generateSessionId() {
    return 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  generateUserId() {
    let userId = localStorage.getItem('analytics_user_id');
    if (!userId) {
      userId = 'user_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
      localStorage.setItem('analytics_user_id', userId);
    }
    return userId;
  }
  
  setupTracking() {
    // Sayfa görüntüleme
    this.track('page_view', {
      url: window.location.href,
      referrer: document.referrer,
      title: document.title
    });
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.track('page_exit', {
        timeOnPage: Date.now() - this.pageStartTime
      });
      this.flush();
    });
    
    // Sayfa gizlendiğinde
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        this.track('page_hidden');
        this.flush();
      } else {
        this.track('page_visible');
      }
    });
    
    // Hata takibi
    window.addEventListener('error', (event) => {
      this.track('javascript_error', {
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno
      });
    });
    
    // Promise hataları
    window.addEventListener('unhandledrejection', (event) => {
      this.track('promise_rejection', {
        reason: event.reason
      });
    });
  }
  
  track(event, data = {}) {
    const eventData = {
      event,
      data,
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
      url: window.location.href,
      userAgent: navigator.userAgent,
      screen: {
        width: screen.width,
        height: screen.height
      },
      viewport: {
        width: window.innerWidth,
        height: window.innerHeight
      }
    };
    
    this.events.push(eventData);
    
    // Kritik olayları hemen gönder
    if (this.isCriticalEvent(event)) {
      this.sendEvent(eventData);
    }
  }
  
  isCriticalEvent(event) {
    const criticalEvents = ['page_exit', 'javascript_error', 'promise_rejection'];
    return criticalEvents.includes(event);
  }
  
  sendEvent(eventData) {
    const success = navigator.sendBeacon(this.endpoint, JSON.stringify(eventData));
    
    if (success) {
      // Başarılı gönderim - olayı listeden çıkar
      const index = this.events.indexOf(eventData);
      if (index > -1) {
        this.events.splice(index, 1);
      }
    }
    
    return success;
  }
  
  flush() {
    if (this.events.length === 0) return;
    
    const events = this.events.splice(0); // Tüm olayları al ve temizle
    
    const success = navigator.sendBeacon(this.endpoint + '/batch', JSON.stringify({
      events,
      batchId: Date.now()
    }));
    
    if (!success) {
      // Başarısız olursa tekrar kuyruğa ekle
      this.events.unshift(...events);
    }
  }
  
  // Kullanıcı etkileşimleri
  trackClick(element, data = {}) {
    this.track('click', {
      element: element.tagName,
      id: element.id,
      className: element.className,
      text: element.textContent?.substring(0, 100),
      ...data
    });
  }
  
  trackFormSubmit(form, data = {}) {
    this.track('form_submit', {
      formId: form.id,
      formAction: form.action,
      formMethod: form.method,
      fieldCount: form.elements.length,
      ...data
    });
  }
  
  trackScroll(depth) {
    this.track('scroll', {
      depth: Math.round(depth * 100) / 100
    });
  }
}

// Kullanım
const analytics = new WebAnalytics({
  endpoint: '/api/analytics',
  userId: 'user123'
});

// Manuel olay takibi
analytics.track('button_click', { button: 'subscribe' });
analytics.track('video_play', { videoId: 'video123', duration: 30 });

// Otomatik etkileşim takibi
document.addEventListener('click', (event) => {
  analytics.trackClick(event.target);
});

document.addEventListener('submit', (event) => {
  analytics.trackFormSubmit(event.target);
});

// Scroll takibi
let scrollDepth = 0;
window.addEventListener('scroll', () => {
  const newDepth = window.scrollY / (document.body.scrollHeight - window.innerHeight);
  if (newDepth > scrollDepth + 0.1) { // Her %10'da bir
    scrollDepth = newDepth;
    analytics.trackScroll(scrollDepth);
  }
});
```

### 2. **Performans İzleme**
```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = [];
    this.setupMonitoring();
  }
  
  setupMonitoring() {
    // Sayfa yükleme performansı
    window.addEventListener('load', () => {
      this.measurePageLoad();
    });
    
    // Core Web Vitals
    this.measureCoreWebVitals();
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.flush();
    });
  }
  
  measurePageLoad() {
    const navigation = performance.getEntriesByType('navigation')[0];
    
    const metrics = {
      type: 'page_load',
      timestamp: Date.now(),
      url: window.location.href,
      metrics: {
        // Sayfa yükleme süreleri
        domContentLoaded: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
        loadComplete: navigation.loadEventEnd - navigation.loadEventStart,
        firstByte: navigation.responseStart - navigation.requestStart,
        domInteractive: navigation.domInteractive - navigation.navigationStart,
        
        // Kaynak yükleme süreleri
        totalTime: navigation.loadEventEnd - navigation.navigationStart,
        
        // Ağ bilgileri
        transferSize: navigation.transferSize,
        encodedBodySize: navigation.encodedBodySize,
        decodedBodySize: navigation.decodedBodySize
      }
    };
    
    this.recordMetric(metrics);
  }
  
  measureCoreWebVitals() {
    // Largest Contentful Paint (LCP)
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1];
      
      this.recordMetric({
        type: 'lcp',
        timestamp: Date.now(),
        value: lastEntry.startTime,
        url: window.location.href
      });
    }).observe({ entryTypes: ['largest-contentful-paint'] });
    
    // First Input Delay (FID)
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach(entry => {
        this.recordMetric({
          type: 'fid',
          timestamp: Date.now(),
          value: entry.processingStart - entry.startTime,
          url: window.location.href
        });
      });
    }).observe({ entryTypes: ['first-input'] });
    
    // Cumulative Layout Shift (CLS)
    let clsValue = 0;
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach(entry => {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      });
      
      this.recordMetric({
        type: 'cls',
        timestamp: Date.now(),
        value: clsValue,
        url: window.location.href
      });
    }).observe({ entryTypes: ['layout-shift'] });
  }
  
  recordMetric(metric) {
    this.metrics.push(metric);
    
    // Kritik metrikleri hemen gönder
    if (this.isCriticalMetric(metric.type)) {
      this.sendMetric(metric);
    }
  }
  
  isCriticalMetric(type) {
    const criticalMetrics = ['lcp', 'fid', 'cls'];
    return criticalMetrics.includes(type);
  }
  
  sendMetric(metric) {
    const success = navigator.sendBeacon('/api/performance', JSON.stringify(metric));
    
    if (success) {
      // Başarılı gönderim - metrikleri listeden çıkar
      const index = this.metrics.indexOf(metric);
      if (index > -1) {
        this.metrics.splice(index, 1);
      }
    }
    
    return success;
  }
  
  flush() {
    if (this.metrics.length === 0) return;
    
    const metrics = this.metrics.splice(0);
    
    const success = navigator.sendBeacon('/api/performance/batch', JSON.stringify({
      metrics,
      batchId: Date.now()
    }));
    
    if (!success) {
      this.metrics.unshift(...metrics);
    }
  }
}

// Kullanım
const performanceMonitor = new PerformanceMonitor();
```

### 3. **Kullanıcı Davranış Analizi**
```javascript
class UserBehaviorTracker {
  constructor() {
    this.behaviorData = [];
    this.sessionStart = Date.now();
    this.setupTracking();
  }
  
  setupTracking() {
    // Mouse hareketleri
    this.trackMouseMovements();
    
    // Klavye etkileşimleri
    this.trackKeyboardEvents();
    
    // Sayfa görünürlüğü
    this.trackVisibility();
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.flush();
    });
  }
  
  trackMouseMovements() {
    let mouseData = [];
    let lastSend = Date.now();
    
    document.addEventListener('mousemove', (event) => {
      mouseData.push({
        x: event.clientX,
        y: event.clientY,
        timestamp: Date.now()
      });
      
      // Her 5 saniyede bir gönder
      if (Date.now() - lastSend > 5000) {
        this.recordBehavior('mouse_movement', mouseData);
        mouseData = [];
        lastSend = Date.now();
      }
    });
  }
  
  trackKeyboardEvents() {
    document.addEventListener('keydown', (event) => {
      this.recordBehavior('keydown', {
        key: event.key,
        code: event.code,
        ctrlKey: event.ctrlKey,
        shiftKey: event.shiftKey,
        altKey: event.altKey,
        timestamp: Date.now()
      });
    });
  }
  
  trackVisibility() {
    document.addEventListener('visibilitychange', () => {
      this.recordBehavior('visibility_change', {
        hidden: document.hidden,
        timestamp: Date.now()
      });
    });
  }
  
  recordBehavior(type, data) {
    const behaviorData = {
      type,
      data,
      timestamp: Date.now(),
      sessionId: this.sessionStart,
      url: window.location.href
    };
    
    this.behaviorData.push(behaviorData);
    
    // Kritik davranışları hemen gönder
    if (this.isCriticalBehavior(type)) {
      this.sendBehavior(behaviorData);
    }
  }
  
  isCriticalBehavior(type) {
    const criticalBehaviors = ['visibility_change'];
    return criticalBehaviors.includes(type);
  }
  
  sendBehavior(behaviorData) {
    const success = navigator.sendBeacon('/api/behavior', JSON.stringify(behaviorData));
    
    if (success) {
      const index = this.behaviorData.indexOf(behaviorData);
      if (index > -1) {
        this.behaviorData.splice(index, 1);
      }
    }
    
    return success;
  }
  
  flush() {
    if (this.behaviorData.length === 0) return;
    
    const behaviors = this.behaviorData.splice(0);
    
    const success = navigator.sendBeacon('/api/behavior/batch', JSON.stringify({
      behaviors,
      batchId: Date.now()
    }));
    
    if (!success) {
      this.behaviorData.unshift(...behaviors);
    }
  }
}

// Kullanım
const behaviorTracker = new UserBehaviorTracker();
```

### 4. **Hata Raporlama Sistemi**
```javascript
class ErrorReporter {
  constructor(config = {}) {
    this.endpoint = config.endpoint || '/api/errors';
    this.environment = config.environment || 'production';
    this.userId = config.userId;
    this.errors = [];
    this.setupErrorHandling();
  }
  
  setupErrorHandling() {
    // JavaScript hataları
    window.addEventListener('error', (event) => {
      this.reportError({
        type: 'javascript_error',
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack,
        timestamp: Date.now()
      });
    });
    
    // Promise reddetmeleri
    window.addEventListener('unhandledrejection', (event) => {
      this.reportError({
        type: 'promise_rejection',
        reason: event.reason,
        stack: event.reason?.stack,
        timestamp: Date.now()
      });
    });
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.flush();
    });
  }
  
  reportError(errorData) {
    const error = {
      ...errorData,
      environment: this.environment,
      userId: this.userId,
      url: window.location.href,
      userAgent: navigator.userAgent,
      timestamp: Date.now()
    };
    
    this.errors.push(error);
    
    // Kritik hataları hemen gönder
    if (this.isCriticalError(error)) {
      this.sendError(error);
    }
  }
  
  isCriticalError(error) {
    const criticalTypes = ['javascript_error', 'promise_rejection'];
    return criticalTypes.includes(error.type);
  }
  
  sendError(errorData) {
    const success = navigator.sendBeacon(this.endpoint, JSON.stringify(errorData));
    
    if (success) {
      const index = this.errors.indexOf(errorData);
      if (index > -1) {
        this.errors.splice(index, 1);
      }
    }
    
    return success;
  }
  
  flush() {
    if (this.errors.length === 0) return;
    
    const errors = this.errors.splice(0);
    
    const success = navigator.sendBeacon(this.endpoint + '/batch', JSON.stringify({
      errors,
      batchId: Date.now()
    }));
    
    if (!success) {
      this.errors.unshift(...errors);
    }
  }
  
  // Manuel hata raporlama
  reportCustomError(message, data = {}) {
    this.reportError({
      type: 'custom_error',
      message,
      data,
      timestamp: Date.now()
    });
  }
}

// Kullanım
const errorReporter = new ErrorReporter({
  endpoint: '/api/errors',
  environment: 'production',
  userId: 'user123'
});

// Manuel hata raporlama
try {
  // Risky operation
  riskyOperation();
} catch (error) {
  errorReporter.reportCustomError('Risky operation failed', {
    operation: 'riskyOperation',
    error: error.message
  });
}
```

## Alternatifler

### 1. **Normal Fetch**
```javascript
// Basit fetch - güvenilir değil
function sendData(data) {
  fetch('/api/data', {
    method: 'POST',
    body: JSON.stringify(data)
  }).catch(error => {
    console.error('Veri gönderilemedi:', error);
  });
}
```

### 2. **XMLHttpRequest**
```javascript
// XMLHttpRequest - eski yöntem
function sendDataXHR(data) {
  const xhr = new XMLHttpRequest();
  xhr.open('POST', '/api/data');
  xhr.send(JSON.stringify(data));
}
```

### 3. **Service Worker**
```javascript
// Service Worker ile arka plan gönderimi
self.addEventListener('sync', event => {
  if (event.tag === 'background-sync') {
    event.waitUntil(sendQueuedData());
  }
});
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Analitik veri gönderimi
- Performans metrikleri
- Hata raporlama
- Kullanıcı davranış analizi
- Sayfa kapatma olayları
- Kritik veri gönderimi

### ❌ **Kullanmayın:**
- Büyük veri paketleri
- Real-time veri
- Kritik olmayan veriler
- Tek seferlik işlemler

## Kullanmazsak Ne Olur?

### 1. **Veri Kaybı**
- Sayfa kapatılırken veri kaybolur
- Analitik veriler eksik
- Hata raporları ulaşmaz

### 2. **Performans Sorunları**
- Sayfa kapatma gecikmeleri
- Kullanıcı deneyimi kötü
- Tarayıcı donması

### 3. **Güvenilirlik Sorunları**
- Veri gönderimi başarısız
- Analitik eksiklikleri
- Hata takibi yapılamaz

## Modern Alternatifler

### 1. **Fetch API + AbortController**
```javascript
// Modern fetch ile iptal kontrolü
const controller = new AbortController();
fetch('/api/data', {
  method: 'POST',
  body: JSON.stringify(data),
  signal: controller.signal
}).catch(error => {
  if (error.name === 'AbortError') {
    // İptal edildi
  }
});
```

### 2. **Web Streams API**
```javascript
// Stream tabanlı veri gönderimi
const stream = new ReadableStream({
  start(controller) {
    controller.enqueue(data);
    controller.close();
  }
});

fetch('/api/data', {
  method: 'POST',
  body: stream
});
```

### 3. **Background Sync API**
```javascript
// Arka plan senkronizasyonu
navigator.serviceWorker.ready.then(registration => {
  registration.sync.register('data-sync');
});
```

## Özet

**Beacon API**, modern web uygulamalarında **güvenilir veri gönderimi** sağlar. Arka plan gönderimi ile:

- ✅ Veri kaybını önler
- ✅ Performansı optimize eder
- ✅ Güvenilir gönderim yapar
- ✅ Sayfa kapatma olaylarını destekler

**Kullanın** - özellikle analitik, performans izleme, hata raporlama ve kritik veri gönderimi için.

```javascript
// Basit Beacon API örneği
navigator.sendBeacon('/api/analytics', JSON.stringify({
  event: 'page_view',
  timestamp: Date.now()
}));
```

### Broadcast Channel API
Aynı kökendeki farklı bağlamlar arasında mesajlaşma için API - Çoklu sekme iletişimi.

## Broadcast Channel API Nedir?

Broadcast Channel API, aynı kökendeki (origin) farklı bağlamlar (sekmeler, pencereler, iframe'ler, worker'lar) arasında mesajlaşma yapmak için kullanılan bir API'dir. Özellikle çoklu sekme uygulamaları ve gerçek zamanlı senkronizasyon için kritik öneme sahiptir.

## Neden Kullanılır?

### 1. **Çoklu Sekme Senkronizasyonu**
```javascript
// ❌ LocalStorage ile sınırlı senkronizasyon
function updateData(data) {
  localStorage.setItem('app_data', JSON.stringify(data));
  // Diğer sekmeler değişikliği fark etmez
}
```

```javascript
// ✅ Broadcast Channel ile gerçek zamanlı senkronizasyon
const channel = new BroadcastChannel('app-sync');

function updateData(data) {
  localStorage.setItem('app_data', JSON.stringify(data));
  
  // Tüm sekmelere bildir
  channel.postMessage({
    type: 'data_updated',
    data: data
  });
}

// Diğer sekmelerde değişiklikleri dinle
channel.onmessage = (event) => {
  if (event.data.type === 'data_updated') {
    // UI'ı güncelle
    updateUI(event.data.data);
  }
};
```

### 2. **Gerçek Zamanlı İletişim**
```javascript
// Çoklu sekme uygulaması
class MultiTabApp {
  constructor() {
    this.channel = new BroadcastChannel('app-channel');
    this.setupCommunication();
  }
  
  setupCommunication() {
    // Mesajları dinle
    this.channel.onmessage = (event) => {
      this.handleMessage(event.data);
    };
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.channel.postMessage({
        type: 'tab_closed',
        tabId: this.tabId
      });
    });
  }
  
  handleMessage(data) {
    switch (data.type) {
      case 'user_login':
        this.handleUserLogin(data.user);
        break;
      case 'data_updated':
        this.handleDataUpdate(data.data);
        break;
      case 'notification':
        this.showNotification(data.message);
        break;
    }
  }
  
  broadcastUserLogin(user) {
    this.channel.postMessage({
      type: 'user_login',
      user: user
    });
  }
  
  broadcastDataUpdate(data) {
    this.channel.postMessage({
      type: 'data_updated',
      data: data
    });
  }
}
```

## Pratik Kullanım Örnekleri

### 1. **Çoklu Sekme Uygulaması**
```javascript
class MultiTabManager {
  constructor() {
    this.channel = new BroadcastChannel('multi-tab-app');
    this.tabId = this.generateTabId();
    this.activeTabs = new Set();
    this.setupCommunication();
  }
  
  generateTabId() {
    return 'tab_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  setupCommunication() {
    // Mesajları dinle
    this.channel.onmessage = (event) => {
      this.handleMessage(event.data);
    };
    
    // Sayfa yüklendiğinde
    this.broadcastTabOpened();
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.broadcastTabClosed();
    });
    
    // Sayfa gizlendiğinde
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        this.broadcastTabHidden();
      } else {
        this.broadcastTabVisible();
      }
    });
  }
  
  handleMessage(data) {
    switch (data.type) {
      case 'tab_opened':
        this.handleTabOpened(data);
        break;
      case 'tab_closed':
        this.handleTabClosed(data);
        break;
      case 'tab_hidden':
        this.handleTabHidden(data);
        break;
      case 'tab_visible':
        this.handleTabVisible(data);
        break;
      case 'data_sync':
        this.handleDataSync(data);
        break;
      case 'user_action':
        this.handleUserAction(data);
        break;
    }
  }
  
  broadcastTabOpened() {
    this.channel.postMessage({
      type: 'tab_opened',
      tabId: this.tabId,
      url: window.location.href,
      timestamp: Date.now()
    });
  }
  
  broadcastTabClosed() {
    this.channel.postMessage({
      type: 'tab_closed',
      tabId: this.tabId,
      timestamp: Date.now()
    });
  }
  
  broadcastTabHidden() {
    this.channel.postMessage({
      type: 'tab_hidden',
      tabId: this.tabId,
      timestamp: Date.now()
    });
  }
  
  broadcastTabVisible() {
    this.channel.postMessage({
      type: 'tab_visible',
      tabId: this.tabId,
      timestamp: Date.now()
    });
  }
  
  broadcastDataSync(data) {
    this.channel.postMessage({
      type: 'data_sync',
      tabId: this.tabId,
      data: data,
      timestamp: Date.now()
    });
  }
  
  broadcastUserAction(action, data = {}) {
    this.channel.postMessage({
      type: 'user_action',
      tabId: this.tabId,
      action: action,
      data: data,
      timestamp: Date.now()
    });
  }
  
  handleTabOpened(data) {
    if (data.tabId !== this.tabId) {
      this.activeTabs.add(data.tabId);
      this.updateTabCounter();
      console.log(`Yeni sekme açıldı: ${data.tabId}`);
    }
  }
  
  handleTabClosed(data) {
    this.activeTabs.delete(data.tabId);
    this.updateTabCounter();
    console.log(`Sekme kapatıldı: ${data.tabId}`);
  }
  
  handleTabHidden(data) {
    console.log(`Sekme gizlendi: ${data.tabId}`);
  }
  
  handleTabVisible(data) {
    console.log(`Sekme görünür oldu: ${data.tabId}`);
  }
  
  handleDataSync(data) {
    if (data.tabId !== this.tabId) {
      // Diğer sekmeden gelen veri senkronizasyonu
      this.syncData(data.data);
    }
  }
  
  handleUserAction(data) {
    if (data.tabId !== this.tabId) {
      // Diğer sekmede yapılan kullanıcı eylemi
      this.handleRemoteUserAction(data.action, data.data);
    }
  }
  
  updateTabCounter() {
    const counter = document.getElementById('tab-counter');
    if (counter) {
      counter.textContent = `Aktif Sekmeler: ${this.activeTabs.size + 1}`;
    }
  }
  
  syncData(data) {
    // Veri senkronizasyonu
    console.log('Veri senkronize edildi:', data);
  }
  
  handleRemoteUserAction(action, data) {
    // Uzak kullanıcı eylemi
    console.log(`Uzak eylem: ${action}`, data);
  }
}

// Kullanım
const multiTabManager = new MultiTabManager();

// Kullanıcı eylemi yayınla
document.getElementById('save-btn').addEventListener('click', () => {
  multiTabManager.broadcastUserAction('save', { 
    data: 'saved data' 
  });
});
```

### 2. **Gerçek Zamanlı Chat Uygulaması**
```javascript
class RealTimeChat {
  constructor() {
    this.channel = new BroadcastChannel('chat-app');
    this.messages = [];
    this.setupChat();
  }
  
  setupChat() {
    // Mesajları dinle
    this.channel.onmessage = (event) => {
      this.handleMessage(event.data);
    };
    
    // Form gönderimi
    document.getElementById('chat-form').addEventListener('submit', (e) => {
      e.preventDefault();
      this.sendMessage();
    });
  }
  
  sendMessage() {
    const input = document.getElementById('message-input');
    const message = input.value.trim();
    
    if (message) {
      const messageData = {
        type: 'chat_message',
        message: message,
        sender: this.getCurrentUser(),
        timestamp: Date.now()
      };
      
      // Tüm sekmelere gönder
      this.channel.postMessage(messageData);
      
      // Kendi sekmesine ekle
      this.addMessage(messageData);
      
      input.value = '';
    }
  }
  
  handleMessage(data) {
    if (data.type === 'chat_message') {
      this.addMessage(data);
    }
  }
  
  addMessage(messageData) {
    this.messages.push(messageData);
    this.renderMessages();
  }
  
  renderMessages() {
    const container = document.getElementById('messages-container');
    container.innerHTML = '';
    
    this.messages.forEach(message => {
      const messageElement = document.createElement('div');
      messageElement.className = 'message';
      messageElement.innerHTML = `
        <strong>${message.sender}:</strong> ${message.message}
        <small>${new Date(message.timestamp).toLocaleTimeString()}</small>
      `;
      container.appendChild(messageElement);
    });
    
    // En alta kaydır
    container.scrollTop = container.scrollHeight;
  }
  
  getCurrentUser() {
    return localStorage.getItem('chat_user') || 'Anonim';
  }
}

// Kullanım
const chat = new RealTimeChat();
```

### 3. **Veri Senkronizasyon Sistemi**
```javascript
class DataSyncManager {
  constructor() {
    this.channel = new BroadcastChannel('data-sync');
    this.data = {};
    this.setupSync();
  }
  
  setupSync() {
    // Mesajları dinle
    this.channel.onmessage = (event) => {
      this.handleSyncMessage(event.data);
    };
    
    // Sayfa yüklendiğinde
    this.loadData();
    
    // Sayfa kapatılırken
    window.addEventListener('beforeunload', () => {
      this.saveData();
    });
  }
  
  handleSyncMessage(data) {
    switch (data.type) {
      case 'data_update':
        this.handleDataUpdate(data);
        break;
      case 'data_request':
        this.handleDataRequest(data);
        break;
      case 'data_response':
        this.handleDataResponse(data);
        break;
    }
  }
  
  updateData(key, value) {
    this.data[key] = value;
    
    // Tüm sekmelere bildir
    this.channel.postMessage({
      type: 'data_update',
      key: key,
      value: value,
      timestamp: Date.now()
    });
    
    // LocalStorage'a kaydet
    this.saveData();
  }
  
  handleDataUpdate(data) {
    this.data[data.key] = data.value;
    this.updateUI(data.key, data.value);
  }
  
  handleDataRequest(data) {
    // Veri isteği geldiğinde yanıtla
    this.channel.postMessage({
      type: 'data_response',
      data: this.data,
      timestamp: Date.now()
    });
  }
  
  handleDataResponse(data) {
    // Veri yanıtı geldiğinde güncelle
    this.data = { ...this.data, ...data.data };
    this.updateAllUI();
  }
  
  requestDataSync() {
    // Tüm sekmelerden veri iste
    this.channel.postMessage({
      type: 'data_request',
      timestamp: Date.now()
    });
  }
  
  loadData() {
    const savedData = localStorage.getItem('app_data');
    if (savedData) {
      this.data = JSON.parse(savedData);
      this.updateAllUI();
    }
  }
  
  saveData() {
    localStorage.setItem('app_data', JSON.stringify(this.data));
  }
  
  updateUI(key, value) {
    const element = document.getElementById(key);
    if (element) {
      element.textContent = value;
    }
  }
  
  updateAllUI() {
    Object.keys(this.data).forEach(key => {
      this.updateUI(key, this.data[key]);
    });
  }
}

// Kullanım
const dataSync = new DataSyncManager();

// Veri güncelle
document.getElementById('update-btn').addEventListener('click', () => {
  const newValue = document.getElementById('input-field').value;
  dataSync.updateData('user_input', newValue);
});

// Veri senkronizasyonu iste
document.getElementById('sync-btn').addEventListener('click', () => {
  dataSync.requestDataSync();
});
```

### 4. **Bildirim Sistemi**
```javascript
class NotificationManager {
  constructor() {
    this.channel = new BroadcastChannel('notifications');
    this.notifications = [];
    this.setupNotifications();
  }
  
  setupNotifications() {
    // Mesajları dinle
    this.channel.onmessage = (event) => {
      this.handleNotification(event.data);
    };
    
    // Sayfa görünürlüğü
    document.addEventListener('visibilitychange', () => {
      if (!document.hidden) {
        this.clearNotifications();
      }
    });
  }
  
  handleNotification(data) {
    switch (data.type) {
      case 'new_notification':
        this.addNotification(data.notification);
        break;
      case 'clear_notifications':
        this.clearNotifications();
        break;
    }
  }
  
  showNotification(message, type = 'info') {
    const notification = {
      id: Date.now(),
      message: message,
      type: type,
      timestamp: Date.now()
    };
    
    // Kendi sekmesine ekle
    this.addNotification(notification);
    
    // Tüm sekmelere bildir
    this.channel.postMessage({
      type: 'new_notification',
      notification: notification
    });
  }
  
  addNotification(notification) {
    this.notifications.push(notification);
    this.renderNotifications();
  }
  
  renderNotifications() {
    const container = document.getElementById('notifications-container');
    container.innerHTML = '';
    
    this.notifications.forEach(notification => {
      const notificationElement = document.createElement('div');
      notificationElement.className = `notification ${notification.type}`;
      notificationElement.innerHTML = `
        <span>${notification.message}</span>
        <button onclick="this.parentElement.remove()">×</button>
      `;
      container.appendChild(notificationElement);
    });
  }
  
  clearNotifications() {
    this.notifications = [];
    this.renderNotifications();
    
    // Tüm sekmelere bildir
    this.channel.postMessage({
      type: 'clear_notifications'
    });
  }
}

// Kullanım
const notificationManager = new NotificationManager();

// Bildirim göster
document.getElementById('show-notification').addEventListener('click', () => {
  notificationManager.showNotification('Yeni mesajınız var!', 'success');
});
```

## Alternatifler

### 1. **LocalStorage Events**
```javascript
// LocalStorage ile basit senkronizasyon
function updateData(data) {
  localStorage.setItem('app_data', JSON.stringify(data));
}

window.addEventListener('storage', (event) => {
  if (event.key === 'app_data') {
    // Veri değişti
    const newData = JSON.parse(event.newValue);
    updateUI(newData);
  }
});
```

### 2. **SharedArrayBuffer**
```javascript
// Paylaşılan bellek (sınırlı destek)
const sharedBuffer = new SharedArrayBuffer(1024);
const sharedArray = new Int32Array(sharedBuffer);
```

### 3. **WebSocket**
```javascript
// WebSocket ile gerçek zamanlı iletişim
const socket = new WebSocket('wss://api.example.com/ws');
socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  handleMessage(data);
};
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Çoklu sekme uygulamaları
- Gerçek zamanlı senkronizasyon
- Bildirim sistemleri
- Chat uygulamaları
- Veri paylaşımı
- Kullanıcı durumu senkronizasyonu

### ❌ **Kullanmayın:**
- Tek sekme uygulamaları
- Basit web siteleri
- Offline uygulamalar
- Güvenlik kritik uygulamalar

## Kullanmazsak Ne Olur?

### 1. **Senkronizasyon Sorunları**
- Sekmeler arası veri tutarsızlığı
- Kullanıcı deneyimi kötü
- Veri kaybı riski

### 2. **Performans Sorunları**
- Gereksiz veri tekrarı
- Bellek kullanımı yüksek
- Ağ trafiği artışı

### 3. **Kullanıcı Deneyimi**
- Tutarsız durumlar
- Kafa karışıklığı
- Memnuniyetsizlik

## Modern Alternatifler

### 1. **Service Worker + MessageChannel**
```javascript
// Service Worker ile iletişim
navigator.serviceWorker.ready.then(registration => {
  const messageChannel = new MessageChannel();
  registration.active.postMessage('Hello', [messageChannel.port2]);
});
```

### 2. **WebRTC DataChannel**
```javascript
// WebRTC ile peer-to-peer iletişim
const peerConnection = new RTCPeerConnection();
const dataChannel = peerConnection.createDataChannel('messages');
```

### 3. **WebSocket + Server**
```javascript
// Sunucu tabanlı gerçek zamanlı iletişim
const socket = new WebSocket('wss://api.example.com/ws');
socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  handleMessage(data);
};
```

## Özet

**Broadcast Channel API**, modern web uygulamalarında **çoklu sekme iletişimi** sağlar. Gerçek zamanlı senkronizasyon ile:

- ✅ Sekmeler arası iletişim yapar
- ✅ Veri senkronizasyonu sağlar
- ✅ Gerçek zamanlı güncellemeler yapar
- ✅ Kullanıcı deneyimini iyileştirir

**Kullanın** - özellikle çoklu sekme uygulamaları, gerçek zamanlı senkronizasyon ve bildirim sistemleri için.

```javascript
// Basit Broadcast Channel API örneği
const channel = new BroadcastChannel('my-channel');

// Mesaj gönder
channel.postMessage('Merhaba dünya!');

// Mesaj al
channel.onmessage = event => {
  console.log('Alınan mesaj:', event.data);
};
```

---

## C - Bölümü

### Canvas API

**Canvas API Nedir?**
Canvas API, HTML5'in bir parçası olan ve web sayfalarında 2D grafik çizimi yapmaya olanak sağlayan güçlü bir JavaScript API'sidir. Canvas elementi, piksel tabanlı bir çizim yüzeyi sağlar ve JavaScript ile programatik olarak kontrol edilebilir.

**Neden Kullanılır?**
- **Grafik Çizimi**: Daireler, kareler, çizgiler ve karmaşık şekiller çizebilir
- **Animasyonlar**: Frame-by-frame animasyonlar oluşturabilir
- **Görsel Efektler**: Gradient'lar, gölgeler, blur efektleri uygulayabilir
- **Oyun Geliştirme**: 2D oyunlar için grafik motoru olarak kullanılabilir
- **Veri Görselleştirme**: Grafikler, çizelgeler ve infografikler oluşturabilir
- **Resim İşleme**: Mevcut resimleri manipüle edebilir ve filtreler uygulayabilir

**Temel Kullanım Örnekleri:**

```javascript
// Canvas elementi ve context alımı
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// Temel şekil çizimi
ctx.beginPath();
ctx.arc(100, 100, 50, 0, 2 * Math.PI);
ctx.fillStyle = 'blue';
ctx.fill();
ctx.strokeStyle = 'red';
ctx.lineWidth = 3;
ctx.stroke();

// Metin çizimi
ctx.font = '20px Arial';
ctx.fillStyle = 'black';
ctx.fillText('Merhaba Canvas!', 50, 150);

// Gradient oluşturma
const gradient = ctx.createLinearGradient(0, 0, 200, 0);
gradient.addColorStop(0, 'red');
gradient.addColorStop(1, 'blue');
ctx.fillStyle = gradient;
ctx.fillRect(50, 200, 200, 100);

// Resim çizimi
const img = new Image();
img.onload = function() {
  ctx.drawImage(img, 0, 0, 100, 100);
};
img.src = 'image.jpg';

// Animasyon örneği
function animate() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // Hareket eden daire
  const time = Date.now() * 0.001;
  const x = 100 + Math.sin(time) * 50;
  const y = 100 + Math.cos(time) * 50;
  
  ctx.beginPath();
  ctx.arc(x, y, 20, 0, 2 * Math.PI);
  ctx.fillStyle = 'green';
  ctx.fill();
  
  requestAnimationFrame(animate);
}
animate();
```

**Gelişmiş Özellikler:**

```javascript
// Transform işlemleri
ctx.save(); // Mevcut durumu kaydet
ctx.translate(100, 100); // Koordinat sistemini taşı
ctx.rotate(Math.PI / 4); // 45 derece döndür
ctx.scale(2, 2); // 2x büyüt
ctx.fillRect(-25, -25, 50, 50); // Döndürülmüş kare
ctx.restore(); // Önceki durumu geri yükle

// Compositing işlemleri
ctx.globalAlpha = 0.5; // Şeffaflık
ctx.globalCompositeOperation = 'multiply'; // Karışım modu

// Path işlemleri
ctx.beginPath();
ctx.moveTo(50, 50);
ctx.lineTo(100, 100);
ctx.quadraticCurveTo(150, 50, 200, 100);
ctx.bezierCurveTo(250, 50, 300, 150, 350, 100);
ctx.stroke();

// Clipping
ctx.beginPath();
ctx.arc(100, 100, 50, 0, 2 * Math.PI);
ctx.clip(); // Sadece bu alan içinde çizim yapılabilir
```

**Alternatifler:**
- **SVG**: Vektör tabanlı grafikler için
- **WebGL**: 3D grafikler ve gelişmiş efektler için
- **CSS**: Basit animasyonlar ve efektler için
- **D3.js**: Veri görselleştirme için
- **Three.js**: 3D grafikler için
- **Fabric.js**: Canvas üzerinde interaktif objeler için

**Ne Zaman Kullanılmalı?**
- ✅ Piksel tabanlı grafik çizimi gerektiğinde
- ✅ Karmaşık animasyonlar yapılacağında
- ✅ Oyun geliştirme projelerinde
- ✅ Veri görselleştirme uygulamalarında
- ✅ Resim işleme ve filtreleme gerektiğinde
- ✅ Gerçek zamanlı grafik güncellemeleri yapılacağında

**Ne Zaman Kullanılmamalı?**
- ❌ Basit CSS animasyonları yeterli olduğunda
- ❌ Vektör tabanlı grafikler gerektiğinde (SVG daha uygun)
- ❌ 3D grafikler gerektiğinde (WebGL daha uygun)
- ❌ Performans kritik olmadığında
- ❌ Erişilebilirlik öncelikli olduğunda

**Kullanılmazsa Ne Olur?**
- **Sınırlı Görsel Deneyim**: Sadece HTML/CSS ile sınırlı kalınır
- **Animasyon Eksikliği**: Karmaşık animasyonlar yapılamaz
- **Oyun Geliştirme Zorluğu**: 2D oyunlar geliştirilemez
- **Veri Görselleştirme Eksikliği**: Dinamik grafikler oluşturulamaz
- **Resim İşleme Yetersizliği**: Client-side resim manipülasyonu yapılamaz

**Modern Alternatifler:**
- **OffscreenCanvas**: Web Worker'larda canvas işleme
- **WebGL2**: Daha gelişmiş 3D grafikler
- **WebGPU**: Yeni nesil grafik API'si
- **CSS Houdini**: Programatik CSS animasyonları
- **WebAssembly**: Yüksek performanslı grafik işleme

### Channel Messaging API

**Channel Messaging API Nedir?**
Channel Messaging API, farklı JavaScript bağlamları (contexts) arasında güvenli ve yapılandırılmış mesajlaşma sağlayan bir API'dir. MessageChannel ve MessagePort objeleri kullanarak iki yönlü iletişim kanalları oluşturur.

**Neden Kullanılır?**
- **Güvenli İletişim**: Farklı origin'ler arasında güvenli mesajlaşma
- **Web Worker İletişimi**: Ana thread ile worker'lar arasında veri transferi
- **Iframe İletişimi**: Parent ve child iframe'ler arasında mesajlaşma
- **Service Worker İletişimi**: Service worker ile ana uygulama arasında iletişim
- **Structured Cloning**: Karmaşık objelerin güvenli transferi
- **Bidirectional Communication**: İki yönlü mesajlaşma desteği

**Temel Kullanım Örnekleri:**

```javascript
// Temel MessageChannel oluşturma
const channel = new MessageChannel();
const port1 = channel.port1;
const port2 = channel.port2;

// Port1 mesaj dinleyicisi
port1.onmessage = event => {
  console.log('Port1\'den gelen:', event.data);
};

// Port2'den mesaj gönderme
port2.postMessage('Port2\'den mesaj');

// Port2 mesaj dinleyicisi
port2.onmessage = event => {
  console.log('Port2\'den gelen:', event.data);
  // Yanıt gönder
  port2.postMessage('Port2\'den yanıt: ' + event.data);
};

// Port1'den mesaj gönderme
port1.postMessage('Port1\'den mesaj');
```

**Web Worker ile Kullanım:**

```javascript
// Ana thread'de
const worker = new Worker('worker.js');
const channel = new MessageChannel();

// Worker'a port gönder
worker.postMessage({ port: channel.port2 }, [channel.port2]);

// Ana thread'de mesaj dinle
channel.port1.onmessage = event => {
  console.log('Worker\'dan gelen:', event.data);
};

// Worker'a mesaj gönder
channel.port1.postMessage('Ana thread\'den mesaj');

// worker.js dosyasında
self.onmessage = function(event) {
  const port = event.data.port;
  
  // Port üzerinden mesaj dinle
  port.onmessage = function(e) {
    console.log('Ana thread\'den gelen:', e.data);
    // Yanıt gönder
    port.postMessage('Worker\'dan yanıt: ' + e.data);
  };
  
  // İlk mesajı gönder
  port.postMessage('Worker\'dan ilk mesaj');
};
```

**Iframe ile Kullanım:**

```javascript
// Parent window'da
const iframe = document.getElementById('myIframe');
const channel = new MessageChannel();

// Iframe'e port gönder
iframe.contentWindow.postMessage({ port: channel.port2 }, '*', [channel.port2]);

// Parent'ta mesaj dinle
channel.port1.onmessage = event => {
  console.log('Iframe\'den gelen:', event.data);
};

// Iframe'e mesaj gönder
channel.port1.postMessage('Parent\'tan mesaj');

// Iframe içinde
window.addEventListener('message', function(event) {
  const port = event.data.port;
  
  port.onmessage = function(e) {
    console.log('Parent\'tan gelen:', e.data);
    port.postMessage('Iframe\'den yanıt: ' + e.data);
  };
  
  port.postMessage('Iframe\'den ilk mesaj');
});
```

**Service Worker ile Kullanım:**

```javascript
// Ana uygulamada
navigator.serviceWorker.ready.then(registration => {
  const channel = new MessageChannel();
  
  // Service worker'a port gönder
  registration.active.postMessage({ port: channel.port2 }, [channel.port2]);
  
  // Ana uygulamada mesaj dinle
  channel.port1.onmessage = event => {
    console.log('Service Worker\'dan gelen:', event.data);
  };
  
  // Service worker'a mesaj gönder
  channel.port1.postMessage('Ana uygulamadan mesaj');
});

// Service worker'da
self.addEventListener('message', function(event) {
  const port = event.data.port;
  
  port.onmessage = function(e) {
    console.log('Ana uygulamadan gelen:', e.data);
    port.postMessage('Service Worker\'dan yanıt: ' + e.data);
  };
  
  port.postMessage('Service Worker\'dan ilk mesaj');
});
```

**Gelişmiş Özellikler:**

```javascript
// Karmaşık obje transferi
const channel = new MessageChannel();
const port1 = channel.port1;
const port2 = channel.port2;

// Structured cloning ile karmaşık obje gönderme
const complexObject = {
  name: 'Test',
  data: new ArrayBuffer(1024),
  nested: {
    array: [1, 2, 3],
    date: new Date()
  }
};

port1.onmessage = event => {
  console.log('Alınan obje:', event.data);
  console.log('ArrayBuffer boyutu:', event.data.data.byteLength);
};

port2.postMessage(complexObject);

// Port kapatma
port1.close();
port2.close();

// Port durumu kontrolü
if (port1.readyState === 'open') {
  port1.postMessage('Port açık');
} else {
  console.log('Port kapalı');
}
```

**Alternatifler:**
- **postMessage API**: Basit mesajlaşma için
- **Broadcast Channel API**: Çoklu sekme iletişimi için
- **SharedArrayBuffer**: Paylaşılan bellek için
- **WebSocket**: Sunucu ile iletişim için
- **Server-Sent Events**: Tek yönlü sunucu mesajları için

**Ne Zaman Kullanılmalı?**
- ✅ Web Worker'lar ile iletişim gerektiğinde
- ✅ Iframe'ler arası güvenli iletişim gerektiğinde
- ✅ Service Worker ile iletişim gerektiğinde
- ✅ Karmaşık objelerin transferi gerektiğinde
- ✅ İki yönlü iletişim gerektiğinde
- ✅ Güvenli mesajlaşma gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit mesajlaşma yeterli olduğunda
- ❌ Tek yönlü iletişim yeterli olduğunda
- ❌ Sunucu ile iletişim gerektiğinde
- ❌ Gerçek zamanlı iletişim gerektiğinde
- ❌ Büyük veri transferi gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Sınırlı İletişim**: Sadece basit postMessage kullanılabilir
- **Güvenlik Riskleri**: Güvenli iletişim sağlanamaz
- **Karmaşık Veri Transferi Zorluğu**: Karmaşık objeler transfer edilemez
- **Worker İletişimi Eksikliği**: Web Worker'larla etkili iletişim kurulamaz
- **Iframe İletişimi Sınırlılığı**: Iframe'ler arası güvenli iletişim sağlanamaz

**Modern Alternatifler:**
- **Broadcast Channel API**: Çoklu sekme iletişimi
- **SharedArrayBuffer**: Paylaşılan bellek
- **WebAssembly**: Yüksek performanslı hesaplama
- **WebRTC DataChannel**: P2P iletişim
- **Web Streams API**: Büyük veri akışları

### Clipboard API

**Clipboard API Nedir?**
Clipboard API, web uygulamalarının sistem panosuna (clipboard) güvenli bir şekilde erişmesini ve veri yazmasını sağlayan modern bir web API'sidir. Metin, resim ve diğer veri türlerini panoya kopyalama ve okuma işlemlerini destekler.

**Neden Kullanılır?**
- **Metin Kopyalama**: Kullanıcıların metinleri kolayca kopyalamasını sağlar
- **Resim Kopyalama**: Resimleri panoya kopyalama ve yapıştırma
- **Veri Transferi**: Uygulamalar arası veri transferi
- **Kullanıcı Deneyimi**: Kopyala-yapıştır işlemlerini kolaylaştırır
- **Güvenlik**: Güvenli pano erişimi sağlar
- **Çoklu Format Desteği**: Farklı veri türlerini destekler

**Temel Kullanım Örnekleri:**

```javascript
// Metin kopyalama
async function copyText() {
  try {
    await navigator.clipboard.writeText('Kopyalanacak metin');
    console.log('Metin panoya kopyalandı');
  } catch (err) {
    console.error('Kopyalama hatası:', err);
  }
}

// Metin okuma
async function readText() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('Panodaki metin:', text);
    return text;
  } catch (err) {
    console.error('Okuma hatası:', err);
  }
}

// Resim kopyalama
async function copyImage(blob) {
  try {
    const clipboardItem = new ClipboardItem({
      'image/png': blob
    });
    await navigator.clipboard.write([clipboardItem]);
    console.log('Resim panoya kopyalandı');
  } catch (err) {
    console.error('Resim kopyalama hatası:', err);
  }
}

// Resim okuma
async function readImage() {
  try {
    const clipboardItems = await navigator.clipboard.read();
    for (const clipboardItem of clipboardItems) {
      for (const type of clipboardItem.types) {
        if (type.startsWith('image/')) {
          const blob = await clipboardItem.getType(type);
          return blob;
        }
      }
    }
  } catch (err) {
    console.error('Resim okuma hatası:', err);
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Çoklu format desteği
async function copyMultipleFormats() {
  const text = 'Kopyalanacak metin';
  const html = '<p>HTML içeriği</p>';
  
  try {
    const clipboardItem = new ClipboardItem({
      'text/plain': new Blob([text], { type: 'text/plain' }),
      'text/html': new Blob([html], { type: 'text/html' })
    });
    
    await navigator.clipboard.write([clipboardItem]);
    console.log('Çoklu format kopyalandı');
  } catch (err) {
    console.error('Çoklu format kopyalama hatası:', err);
  }
}

// Canvas'tan resim kopyalama
async function copyCanvasImage(canvas) {
  try {
    canvas.toBlob(async (blob) => {
      const clipboardItem = new ClipboardItem({
        'image/png': blob
      });
      await navigator.clipboard.write([clipboardItem]);
      console.log('Canvas resmi kopyalandı');
    });
  } catch (err) {
    console.error('Canvas kopyalama hatası:', err);
  }
}

// Pano izleme
function watchClipboard() {
  navigator.clipboard.addEventListener('clipboardchange', (event) => {
    console.log('Pano değişti:', event);
  });
}

// Güvenlik kontrolü
async function checkClipboardPermission() {
  try {
    const permission = await navigator.permissions.query({ name: 'clipboard-read' });
    console.log('Pano okuma izni:', permission.state);
    
    if (permission.state === 'granted') {
      const text = await navigator.clipboard.readText();
      console.log('Panodaki metin:', text);
    }
  } catch (err) {
    console.error('İzin kontrolü hatası:', err);
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Kopyala butonu
document.getElementById('copyBtn').addEventListener('click', async () => {
  const textToCopy = document.getElementById('textInput').value;
  
  try {
    await navigator.clipboard.writeText(textToCopy);
    showNotification('Metin kopyalandı!');
  } catch (err) {
    showNotification('Kopyalama başarısız!');
  }
});

// Yapıştır butonu
document.getElementById('pasteBtn').addEventListener('click', async () => {
  try {
    const text = await navigator.clipboard.readText();
    document.getElementById('textInput').value = text;
    showNotification('Metin yapıştırıldı!');
  } catch (err) {
    showNotification('Yapıştırma başarısız!');
  }
});

// Resim yapıştırma
document.getElementById('imageInput').addEventListener('paste', async (event) => {
  try {
    const clipboardItems = await navigator.clipboard.read();
    
    for (const clipboardItem of clipboardItems) {
      for (const type of clipboardItem.types) {
        if (type.startsWith('image/')) {
          const blob = await clipboardItem.getType(type);
          const imageUrl = URL.createObjectURL(blob);
          
          const img = document.createElement('img');
          img.src = imageUrl;
          document.getElementById('imageContainer').appendChild(img);
        }
      }
    }
  } catch (err) {
    console.error('Resim yapıştırma hatası:', err);
  }
});

// Bildirim gösterme
function showNotification(message) {
  const notification = document.createElement('div');
  notification.textContent = message;
  notification.style.cssText = `
    position: fixed;
    top: 20px;
    right: 20px;
    background: #4CAF50;
    color: white;
    padding: 10px 20px;
    border-radius: 4px;
    z-index: 1000;
  `;
  
  document.body.appendChild(notification);
  
  setTimeout(() => {
    document.body.removeChild(notification);
  }, 3000);
}
```

**Alternatifler:**
- **execCommand('copy')**: Eski tarayıcılar için (deprecated)
- **Selection API**: Metin seçimi ve kopyalama için
- **File API**: Dosya yükleme ve işleme için
- **Drag and Drop API**: Sürükle-bırak işlemleri için
- **Web Share API**: Paylaşım işlemleri için

**Ne Zaman Kullanılmalı?**
- ✅ Kullanıcı deneyimini iyileştirmek için
- ✅ Metin kopyalama/yapıştırma gerektiğinde
- ✅ Resim kopyalama/yapıştırma gerektiğinde
- ✅ Uygulamalar arası veri transferi gerektiğinde
- ✅ Modern tarayıcı desteği gerektiğinde
- ✅ Güvenli pano erişimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ HTTPS olmayan sayfalarda
- ❌ Kullanıcı izni olmadan
- ❌ Hassas veri işleme gerektiğinde
- ❌ Offline uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Sınırlı Kullanıcı Deneyimi**: Kopyala-yapıştır işlemleri zorlaşır
- **Manuel İşlemler**: Kullanıcılar manuel olarak kopyalama yapmak zorunda kalır
- **Veri Transferi Zorluğu**: Uygulamalar arası veri transferi zorlaşır
- **Resim İşleme Eksikliği**: Resim kopyalama/yapıştırma yapılamaz
- **Modern Özellik Eksikliği**: Modern web uygulaması özellikleri kullanılamaz

**Modern Alternatifler:**
- **Web Share API**: Native paylaşım özellikleri
- **File System Access API**: Dosya sistemi erişimi
- **Web Streams API**: Büyük veri akışları
- **WebAssembly**: Yüksek performanslı veri işleme
- **Web Workers**: Arka planda veri işleme

### Compression Streams API

**Compression Streams API Nedir?**
Compression Streams API, web uygulamalarında veri sıkıştırma ve açma işlemlerini gerçekleştirmek için kullanılan modern bir web API'sidir. Stream tabanlı yaklaşım ile büyük veri setlerini etkili bir şekilde sıkıştırabilir ve açabilir.

**Neden Kullanılır?**
- **Veri Sıkıştırma**: Büyük veri setlerini sıkıştırarak boyutu küçültür
- **Bant Genişliği Tasarrufu**: Ağ üzerinden gönderilen veri miktarını azaltır
- **Depolama Optimizasyonu**: Yerel depolamada alan tasarrufu sağlar
- **Performans İyileştirme**: Veri transfer sürelerini kısaltır
- **Stream İşleme**: Büyük veri setlerini parça parça işler
- **Çoklu Format Desteği**: Gzip, deflate, brotli gibi formatları destekler

**Temel Kullanım Örnekleri:**

```javascript
// Gzip sıkıştırma
async function compressData(data) {
  const stream = new CompressionStream('gzip');
  const writer = stream.writable.getWriter();
  const reader = stream.readable.getReader();
  
  // Veriyi yaz
  await writer.write(new TextEncoder().encode(data));
  await writer.close();
  
  // Sıkıştırılmış veriyi oku
  const chunks = [];
  let done = false;
  
  while (!done) {
    const { value, done: readerDone } = await reader.read();
    done = readerDone;
    if (value) {
      chunks.push(value);
    }
  }
  
  return new Uint8Array(chunks.reduce((acc, chunk) => [...acc, ...chunk], []));
}

// Gzip açma
async function decompressData(compressedData) {
  const stream = new DecompressionStream('gzip');
  const writer = stream.writable.getWriter();
  const reader = stream.readable.getReader();
  
  // Sıkıştırılmış veriyi yaz
  await writer.write(compressedData);
  await writer.close();
  
  // Açılmış veriyi oku
  const chunks = [];
  let done = false;
  
  while (!done) {
    const { value, done: readerDone } = await reader.read();
    done = readerDone;
    if (value) {
      chunks.push(value);
    }
  }
  
  return new TextDecoder().decode(new Uint8Array(chunks.reduce((acc, chunk) => [...acc, ...chunk], [])));
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Farklı sıkıştırma formatları
async function compressWithFormat(data, format) {
  const stream = new CompressionStream(format);
  const writer = stream.writable.getWriter();
  const reader = stream.readable.getReader();
  
  await writer.write(new TextEncoder().encode(data));
  await writer.close();
  
  const chunks = [];
  let done = false;
  
  while (!done) {
    const { value, done: readerDone } = await reader.read();
    done = readerDone;
    if (value) {
      chunks.push(value);
    }
  }
  
  return new Uint8Array(chunks.reduce((acc, chunk) => [...acc, ...chunk], []));
}

// Dosya sıkıştırma
async function compressFile(file) {
  const stream = new CompressionStream('gzip');
  const writer = stream.writable.getWriter();
  const reader = stream.readable.getReader();
  
  // Dosyayı parça parça oku ve sıkıştır
  const fileReader = file.stream().getReader();
  let done = false;
  
  while (!done) {
    const { value, done: readerDone } = await fileReader.read();
    done = readerDone;
    if (value) {
      await writer.write(value);
    }
  }
  
  await writer.close();
  
  // Sıkıştırılmış veriyi topla
  const chunks = [];
  let streamDone = false;
  
  while (!streamDone) {
    const { value, done: streamReaderDone } = await reader.read();
    streamDone = streamReaderDone;
    if (value) {
      chunks.push(value);
    }
  }
  
  return new Blob(chunks, { type: 'application/gzip' });
}

// Büyük veri seti sıkıştırma
async function compressLargeDataset(dataArray) {
  const stream = new CompressionStream('brotli');
  const writer = stream.writable.getWriter();
  const reader = stream.readable.getReader();
  
  // Veri dizisini JSON olarak sıkıştır
  const jsonData = JSON.stringify(dataArray);
  await writer.write(new TextEncoder().encode(jsonData));
  await writer.close();
  
  // Sıkıştırılmış veriyi oku
  const chunks = [];
  let done = false;
  
  while (!done) {
    const { value, done: readerDone } = await reader.read();
    done = readerDone;
    if (value) {
      chunks.push(value);
    }
  }
  
  return new Uint8Array(chunks.reduce((acc, chunk) => [...acc, ...chunk], []));
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// API yanıtlarını sıkıştırma
async function compressApiResponse(response) {
  const data = await response.text();
  const compressed = await compressData(data);
  
  // Sıkıştırılmış veriyi localStorage'a kaydet
  const base64 = btoa(String.fromCharCode(...compressed));
  localStorage.setItem('compressed_data', base64);
  
  return compressed;
}

// Sıkıştırılmış veriyi açma
async function decompressApiResponse() {
  const base64 = localStorage.getItem('compressed_data');
  if (!base64) return null;
  
  const compressed = new Uint8Array(atob(base64).split('').map(c => c.charCodeAt(0)));
  return await decompressData(compressed);
}

// Veri transferi optimizasyonu
async function sendCompressedData(url, data) {
  const compressed = await compressData(JSON.stringify(data));
  
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/gzip',
      'Content-Encoding': 'gzip'
    },
    body: compressed
  });
  
  return response;
}

// Dosya yükleme optimizasyonu
async function uploadCompressedFile(file) {
  const compressed = await compressFile(file);
  
  const formData = new FormData();
  formData.append('file', compressed, file.name + '.gz');
  
  const response = await fetch('/upload', {
    method: 'POST',
    body: formData
  });
  
  return response;
}
```

**Performans Karşılaştırması:**

```javascript
// Sıkıştırma performansı testi
async function performanceTest() {
  const testData = 'A'.repeat(1000000); // 1MB test verisi
  const formats = ['gzip', 'deflate', 'brotli'];
  
  for (const format of formats) {
    const start = performance.now();
    const compressed = await compressWithFormat(testData, format);
    const end = performance.now();
    
    const compressionRatio = (1 - compressed.length / testData.length) * 100;
    
    console.log(`${format}:`);
    console.log(`  Sıkıştırma süresi: ${end - start}ms`);
    console.log(`  Sıkıştırma oranı: ${compressionRatio.toFixed(2)}%`);
    console.log(`  Boyut: ${compressed.length} bytes`);
  }
}
```

**Alternatifler:**
- **pako.js**: JavaScript sıkıştırma kütüphanesi
- **fflate**: Hızlı sıkıştırma kütüphanesi
- **lz-string**: String sıkıştırma kütüphanesi
- **Server-side compression**: Sunucu tarafında sıkıştırma
- **CDN compression**: İçerik dağıtım ağı sıkıştırması

**Ne Zaman Kullanılmalı?**
- ✅ Büyük veri setleri işlendiğinde
- ✅ Ağ bant genişliği sınırlı olduğunda
- ✅ Depolama alanı optimize edilmesi gerektiğinde
- ✅ API yanıtları sıkıştırılması gerektiğinde
- ✅ Dosya transferi optimize edilmesi gerektiğinde
- ✅ Modern tarayıcı desteği gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Küçük veri setleri işlendiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ CPU kaynakları sınırlı olduğunda
- ❌ Gerçek zamanlı işlem gerektiğinde
- ❌ Zaten sıkıştırılmış veri işlendiğinde

**Kullanılmazsa Ne Olur?**
- **Yüksek Bant Genişliği Kullanımı**: Daha fazla ağ trafiği oluşur
- **Yavaş Veri Transferi**: Büyük dosyalar yavaş yüklenir
- **Depolama İsrafı**: Daha fazla yerel depolama alanı kullanılır
- **API Performans Sorunları**: Büyük API yanıtları yavaş gelir
- **Kullanıcı Deneyimi Sorunları**: Yavaş yükleme süreleri

**Modern Alternatifler:**
- **WebAssembly**: Yüksek performanslı sıkıştırma
- **Web Workers**: Arka planda sıkıştırma işlemi
- **Service Workers**: Cache sıkıştırması
- **HTTP/2 Server Push**: Sunucu tarafında optimizasyon
- **Web Streams API**: Büyük veri akışları

### Cookies API

**Cookies API Nedir?**
Cookies API, web uygulamalarının HTTP cookie'lerini yönetmesini sağlayan geleneksel web API'sidir. Kullanıcı oturum bilgileri, tercihler, kimlik doğrulama token'ları ve diğer küçük veri parçalarını tarayıcıda saklamak için kullanılır. Her cookie bir domain ve path ile sınırlandırılır ve belirli bir süre sonra otomatik olarak silinebilir.

**Neden Kullanılır?**
- **Oturum Yönetimi**: Kullanıcı oturum bilgilerini saklama
- **Kimlik Doğrulama**: Login token'ları ve kimlik bilgileri
- **Kullanıcı Tercihleri**: Tema, dil, ayarlar gibi tercihler
- **Takip ve Analitik**: Kullanıcı davranış analizi
- **E-ticaret**: Sepet bilgileri ve alışveriş geçmişi
- **Kişiselleştirme**: Kullanıcı deneyimi özelleştirme
- **Cross-Site İletişim**: Farklı domain'ler arası veri paylaşımı

**Temel Kullanım Örnekleri:**

```javascript
// Cookie oluştur
document.cookie = "username=john_doe; expires=Thu, 18 Dec 2024 12:00:00 UTC; path=/";

// Güvenli cookie
document.cookie = "sessionId=abc123; secure; httponly; samesite=strict";

// Domain ile cookie
document.cookie = "theme=dark; domain=.example.com; path=/";

// Cookie oku
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

const username = getCookie('username');
console.log('Kullanıcı adı:', username);

// Cookie sil
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";

// Tüm cookie'leri listele
function getAllCookies() {
  return document.cookie.split(';').reduce((cookies, cookie) => {
    const [name, value] = cookie.trim().split('=');
    cookies[name] = value;
    return cookies;
  }, {});
}

const allCookies = getAllCookies();
console.log('Tüm cookie\'ler:', allCookies);

// Cookie varlığını kontrol et
function hasCookie(name) {
  return document.cookie.indexOf(`${name}=`) > -1;
}

if (hasCookie('username')) {
  console.log('Kullanıcı giriş yapmış');
} else {
  console.log('Kullanıcı giriş yapmamış');
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Cookie yöneticisi
class CookieManager {
  constructor() {
    this.defaultOptions = {
      expires: 7, // 7 gün
      path: '/',
      domain: window.location.hostname,
      secure: window.location.protocol === 'https:',
      samesite: 'lax'
    };
  }
  
  // Cookie oluştur
  set(name, value, options = {}) {
    const config = { ...this.defaultOptions, ...options };
    let cookieString = `${name}=${encodeURIComponent(value)}`;
    
    // Expires
    if (config.expires) {
      const date = new Date();
      date.setTime(date.getTime() + (config.expires * 24 * 60 * 60 * 1000));
      cookieString += `; expires=${date.toUTCString()}`;
    }
    
    // Path
    if (config.path) {
      cookieString += `; path=${config.path}`;
    }
    
    // Domain
    if (config.domain) {
      cookieString += `; domain=${config.domain}`;
    }
    
    // Secure
    if (config.secure) {
      cookieString += `; secure`;
    }
    
    // HttpOnly
    if (config.httponly) {
      cookieString += `; httponly`;
    }
    
    // SameSite
    if (config.samesite) {
      cookieString += `; samesite=${config.samesite}`;
    }
    
    document.cookie = cookieString;
    console.log('Cookie oluşturuldu:', name);
  }
  
  // Cookie oku
  get(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) {
      return decodeURIComponent(parts.pop().split(';').shift());
    }
    return null;
  }
  
  // Cookie sil
  remove(name, options = {}) {
    const config = { ...this.defaultOptions, ...options };
    let cookieString = `${name}=; expires=Thu, 01 Jan 1970 00:00:00 UTC`;
    
    if (config.path) {
      cookieString += `; path=${config.path}`;
    }
    
    if (config.domain) {
      cookieString += `; domain=${config.domain}`;
    }
    
    document.cookie = cookieString;
    console.log('Cookie silindi:', name);
  }
  
  // Tüm cookie'leri al
  getAll() {
    return document.cookie.split(';').reduce((cookies, cookie) => {
      const [name, value] = cookie.trim().split('=');
      if (name && value) {
        cookies[name] = decodeURIComponent(value);
      }
      return cookies;
    }, {});
  }
  
  // Cookie varlığını kontrol et
  has(name) {
    return document.cookie.indexOf(`${name}=`) > -1;
  }
  
  // Cookie'leri temizle
  clear() {
    const cookies = this.getAll();
    Object.keys(cookies).forEach(name => {
      this.remove(name);
    });
    console.log('Tüm cookie\'ler temizlendi');
  }
  
  // Cookie boyutunu hesapla
  getSize() {
    return document.cookie.length;
  }
  
  // Cookie sayısını al
  getCount() {
    return document.cookie.split(';').filter(cookie => cookie.trim()).length;
  }
}

// Oturum yöneticisi
class SessionManager {
  constructor() {
    this.cookieManager = new CookieManager();
    this.sessionKey = 'session_data';
    this.userKey = 'user_data';
  }
  
  // Oturum başlat
  startSession(userData) {
    const sessionData = {
      id: this.generateSessionId(),
      startTime: Date.now(),
      userAgent: navigator.userAgent,
      ip: 'unknown' // Bu bilgi server'dan gelir
    };
    
    this.cookieManager.set(this.sessionKey, JSON.stringify(sessionData), {
      expires: 1, // 1 gün
      secure: true,
      httponly: true,
      samesite: 'strict'
    });
    
    this.cookieManager.set(this.userKey, JSON.stringify(userData), {
      expires: 1,
      secure: true,
      samesite: 'strict'
    });
    
    console.log('Oturum başlatıldı:', sessionData.id);
  }
  
  // Oturum bilgilerini al
  getSession() {
    const sessionData = this.cookieManager.get(this.sessionKey);
    return sessionData ? JSON.parse(sessionData) : null;
  }
  
  // Kullanıcı bilgilerini al
  getUser() {
    const userData = this.cookieManager.get(this.userKey);
    return userData ? JSON.parse(userData) : null;
  }
  
  // Oturum süresini kontrol et
  isSessionValid() {
    const session = this.getSession();
    if (!session) return false;
    
    const now = Date.now();
    const sessionAge = now - session.startTime;
    const maxAge = 24 * 60 * 60 * 1000; // 24 saat
    
    return sessionAge < maxAge;
  }
  
  // Oturumu yenile
  refreshSession() {
    const user = this.getUser();
    if (user) {
      this.startSession(user);
      console.log('Oturum yenilendi');
    }
  }
  
  // Oturumu sonlandır
  endSession() {
    this.cookieManager.remove(this.sessionKey);
    this.cookieManager.remove(this.userKey);
    console.log('Oturum sonlandırıldı');
  }
  
  // Session ID oluştur
  generateSessionId() {
    return 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
}

// Kullanıcı tercihleri yöneticisi
class UserPreferencesManager {
  constructor() {
    this.cookieManager = new CookieManager();
    this.preferencesKey = 'user_preferences';
  }
  
  // Tercihleri kaydet
  savePreferences(preferences) {
    this.cookieManager.set(this.preferencesKey, JSON.stringify(preferences), {
      expires: 365, // 1 yıl
      secure: true,
      samesite: 'lax'
    });
    
    console.log('Tercihler kaydedildi');
  }
  
  // Tercihleri yükle
  loadPreferences() {
    const preferences = this.cookieManager.get(this.preferencesKey);
    return preferences ? JSON.parse(preferences) : this.getDefaultPreferences();
  }
  
  // Varsayılan tercihler
  getDefaultPreferences() {
    return {
      theme: 'light',
      language: 'tr',
      fontSize: 'medium',
      notifications: true,
      autoSave: true,
      darkMode: false
    };
  }
  
  // Tercih güncelle
  updatePreference(key, value) {
    const preferences = this.loadPreferences();
    preferences[key] = value;
    this.savePreferences(preferences);
    
    console.log('Tercih güncellendi:', key, value);
  }
  
  // Tercih al
  getPreference(key) {
    const preferences = this.loadPreferences();
    return preferences[key];
  }
  
  // Tercihleri sıfırla
  resetPreferences() {
    this.cookieManager.remove(this.preferencesKey);
    console.log('Tercihler sıfırlandı');
  }
}

// E-ticaret sepet yöneticisi
class ShoppingCartManager {
  constructor() {
    this.cookieManager = new CookieManager();
    this.cartKey = 'shopping_cart';
  }
  
  // Sepete ürün ekle
  addToCart(product) {
    const cart = this.getCart();
    const existingItem = cart.find(item => item.id === product.id);
    
    if (existingItem) {
      existingItem.quantity += 1;
    } else {
      cart.push({
        id: product.id,
        name: product.name,
        price: product.price,
        quantity: 1,
        image: product.image
      });
    }
    
    this.saveCart(cart);
    console.log('Ürün sepete eklendi:', product.name);
  }
  
  // Sepetten ürün çıkar
  removeFromCart(productId) {
    const cart = this.getCart();
    const filteredCart = cart.filter(item => item.id !== productId);
    this.saveCart(filteredCart);
    
    console.log('Ürün sepetten çıkarıldı:', productId);
  }
  
  // Ürün miktarını güncelle
  updateQuantity(productId, quantity) {
    const cart = this.getCart();
    const item = cart.find(item => item.id === productId);
    
    if (item) {
      if (quantity <= 0) {
        this.removeFromCart(productId);
      } else {
        item.quantity = quantity;
        this.saveCart(cart);
      }
    }
  }
  
  // Sepeti al
  getCart() {
    const cart = this.cookieManager.get(this.cartKey);
    return cart ? JSON.parse(cart) : [];
  }
  
  // Sepeti kaydet
  saveCart(cart) {
    this.cookieManager.set(this.cartKey, JSON.stringify(cart), {
      expires: 30, // 30 gün
      secure: true,
      samesite: 'lax'
    });
  }
  
  // Sepet toplamını hesapla
  getCartTotal() {
    const cart = this.getCart();
    return cart.reduce((total, item) => total + (item.price * item.quantity), 0);
  }
  
  // Sepet ürün sayısını al
  getCartItemCount() {
    const cart = this.getCart();
    return cart.reduce((count, item) => count + item.quantity, 0);
  }
  
  // Sepeti temizle
  clearCart() {
    this.cookieManager.remove(this.cartKey);
    console.log('Sepet temizlendi');
  }
}
```

**Alternatifler:**
- **Local Storage**: Kalıcı veri depolama
- **Session Storage**: Oturum bazlı depolama
- **IndexedDB**: Büyük veri depolama
- **Cookie Store API**: Modern cookie yönetimi
- **Web Storage**: Basit key-value depolama

**Ne Zaman Kullanılmalı?**
- ✅ Oturum yönetimi gerektiğinde
- ✅ Kimlik doğrulama gerektiğinde
- ✅ Cross-site veri paylaşımı gerektiğinde
- ✅ Server-side okuma gerektiğinde
- ✅ Küçük veri parçaları gerektiğinde
- ✅ Otomatik süre sonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Büyük veri setleri gerektiğinde
- ❌ Güvenlik kritik veri gerektiğinde
- ❌ Karmaşık veri yapıları gerektiğinde
- ❌ Binary veri gerektiğinde
- ❌ Client-side sadece veri gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Oturum Yönetimi Eksikliği**: Kullanıcı oturumları saklanamaz
- **Kimlik Doğrulama Eksikliği**: Login bilgileri korunamaz
- **Cross-Site İletişim Eksikliği**: Domain'ler arası veri paylaşımı olmaz
- **Server-Side Erişim Eksikliği**: Server'dan veri okunamaz
- **Otomatik Süre Sonu Eksikliği**: Veriler manuel olarak silinmeli

**Modern Alternatifler:**
- **Cookie Store API**: Modern cookie yönetimi
- **Local Storage**: Kalıcı veri depolama
- **Session Storage**: Oturum bazlı depolama
- **IndexedDB**: Büyük veri depolama
- **Web Locks API**: Veri kilitleme

### Console API

**Console API Nedir?**
Console API, web uygulamalarında hata ayıklama, loglama ve geliştirici araçları için kullanılan temel bir web API'sidir. Tarayıcı konsoluna mesaj gönderme, veri görselleştirme ve performans ölçümü gibi işlemleri destekler.

**Neden Kullanılır?**
- **Hata Ayıklama**: Kod hatalarını tespit etme ve çözme
- **Loglama**: Uygulama durumunu izleme ve kaydetme
- **Performans Ölçümü**: Kod performansını analiz etme
- **Veri Görselleştirme**: Karmaşık veri yapılarını görselleştirme
- **Geliştirme Araçları**: Geliştirme sürecini kolaylaştırma
- **Kullanıcı Deneyimi**: Hata mesajları ve bilgilendirme

**Temel Kullanım Örnekleri:**

```javascript
// Temel loglama
console.log('Bilgi mesajı');
console.info('Bilgilendirme mesajı');
console.warn('Uyarı mesajı');
console.error('Hata mesajı');
console.debug('Debug mesajı');

// Obje loglama
const user = { name: 'Ali', age: 25, city: 'İstanbul' };
console.log('Kullanıcı bilgileri:', user);

// Tablo görselleştirme
const users = [
  { name: 'Ali', age: 25, city: 'İstanbul' },
  { name: 'Ayşe', age: 30, city: 'Ankara' },
  { name: 'Mehmet', age: 28, city: 'İzmir' }
];
console.table(users);

// Grup loglama
console.group('Kullanıcı İşlemleri');
console.log('Kullanıcı giriş yaptı');
console.log('Profil güncellendi');
console.log('Ayarlar kaydedildi');
console.groupEnd();

// Koşullu loglama
const isDebug = true;
console.assert(isDebug, 'Debug modu aktif değil');
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Performans ölçümü
console.time('API Çağrısı');
fetch('/api/users')
  .then(response => response.json())
  .then(data => {
    console.timeEnd('API Çağrısı');
    console.log('API yanıtı:', data);
  });

// Çoklu performans ölçümü
console.time('Toplam İşlem');
console.time('Veri Yükleme');
// Veri yükleme işlemi
console.timeEnd('Veri Yükleme');

console.time('Veri İşleme');
// Veri işleme işlemi
console.timeEnd('Veri İşleme');
console.timeEnd('Toplam İşlem');

// Stil uygulama
console.log('%cÖnemli Mesaj', 'color: red; font-size: 20px; font-weight: bold;');
console.log('%cBaşarılı İşlem', 'color: green; background: yellow; padding: 5px;');

// Trace (çağrı yığını)
function functionA() {
  functionB();
}

function functionB() {
  functionC();
}

function functionC() {
  console.trace('Fonksiyon çağrı yığını');
}

functionA();

// Count (sayac)
for (let i = 0; i < 5; i++) {
  console.count('Döngü sayacı');
}

// CountReset
console.countReset('Döngü sayacı');

// Dir (obje detayları)
const complexObject = {
  name: 'Test',
  nested: {
    array: [1, 2, 3],
    func: function() { return 'test'; }
  }
};
console.dir(complexObject);

// Dirxml (DOM elementi)
const element = document.getElementById('myElement');
console.dirxml(element);
```

**Pratik Kullanım Senaryoları:**

```javascript
// Hata yakalama ve loglama
try {
  // Riskli işlem
  const result = riskyOperation();
  console.log('İşlem başarılı:', result);
} catch (error) {
  console.error('Hata oluştu:', error);
  console.error('Hata detayları:', {
    message: error.message,
    stack: error.stack,
    timestamp: new Date().toISOString()
  });
}

// API istekleri loglama
function logApiRequest(url, method, data) {
  console.group(`API İsteği: ${method} ${url}`);
  console.log('İstek verisi:', data);
  console.time('İstek süresi');
  
  return fetch(url, {
    method,
    body: JSON.stringify(data),
    headers: { 'Content-Type': 'application/json' }
  })
  .then(response => {
    console.timeEnd('İstek süresi');
    console.log('Yanıt durumu:', response.status);
    return response.json();
  })
  .then(result => {
    console.log('Yanıt verisi:', result);
    console.groupEnd();
    return result;
  })
  .catch(error => {
    console.timeEnd('İstek süresi');
    console.error('API hatası:', error);
    console.groupEnd();
    throw error;
  });
}

// Kullanıcı etkileşimleri loglama
function logUserInteraction(action, details) {
  console.log(`%cKullanıcı Etkileşimi: ${action}`, 
    'color: blue; font-weight: bold;');
  console.log('Detaylar:', details);
  console.log('Zaman:', new Date().toLocaleString());
}

// Performans izleme
function performanceMonitor() {
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.log(`%cPerformans: ${entry.name}`, 
        'color: purple; font-weight: bold;');
      console.log('Süre:', entry.duration + 'ms');
      console.log('Başlangıç:', entry.startTime);
    }
  });
  
  observer.observe({ entryTypes: ['measure', 'navigation'] });
}

// Veri analizi
function analyzeData(data) {
  console.group('Veri Analizi');
  
  console.log('Toplam kayıt sayısı:', data.length);
  console.log('Veri türü:', typeof data);
  
  if (Array.isArray(data)) {
    console.log('Dizi uzunluğu:', data.length);
    console.log('İlk eleman:', data[0]);
    console.log('Son eleman:', data[data.length - 1]);
  }
  
  if (typeof data === 'object' && data !== null) {
    console.log('Obje anahtarları:', Object.keys(data));
    console.log('Obje değerleri:', Object.values(data));
  }
  
  console.groupEnd();
}
```

**Geliştirme Araçları:**

```javascript
// Debug helper fonksiyonları
const debug = {
  log: (message, data) => {
    if (process.env.NODE_ENV === 'development') {
      console.log(`[DEBUG] ${message}`, data);
    }
  },
  
  warn: (message, data) => {
    if (process.env.NODE_ENV === 'development') {
      console.warn(`[WARN] ${message}`, data);
    }
  },
  
  error: (message, error) => {
    console.error(`[ERROR] ${message}`, error);
  },
  
  group: (title, callback) => {
    if (process.env.NODE_ENV === 'development') {
      console.group(title);
      callback();
      console.groupEnd();
    }
  }
};

// Kullanım örneği
debug.log('Kullanıcı giriş yaptı', { userId: 123, timestamp: Date.now() });
debug.group('API İsteği', () => {
  debug.log('İstek gönderiliyor', { url: '/api/users' });
  debug.log('Yanıt alındı', { status: 200 });
});

// Performans profili
function profileFunction(fn, name) {
  return function(...args) {
    console.time(name);
    const result = fn.apply(this, args);
    console.timeEnd(name);
    return result;
  };
}

// Kullanım örneği
const expensiveFunction = profileFunction((n) => {
  let sum = 0;
  for (let i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
}, 'Expensive Function');

expensiveFunction(1000000);
```

**Alternatifler:**
- **Custom Logger**: Özel loglama sistemi
- **Winston**: Node.js loglama kütüphanesi
- **Log4js**: JavaScript loglama kütüphanesi
- **Sentry**: Hata takip servisi
- **Bugsnag**: Hata izleme servisi

**Ne Zaman Kullanılmalı?**
- ✅ Hata ayıklama gerektiğinde
- ✅ Geliştirme sürecinde
- ✅ Performans analizi gerektiğinde
- ✅ Veri görselleştirme gerektiğinde
- ✅ Kullanıcı etkileşimlerini izleme gerektiğinde
- ✅ API isteklerini loglama gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Production ortamında (performans nedeniyle)
- ❌ Hassas veri loglama gerektiğinde
- ❌ Büyük veri setleri loglama gerektiğinde
- ❌ Gerçek zamanlı uygulamalarda
- ❌ Mobil uygulamalarda (battery drain)

**Kullanılmazsa Ne Olur?**
- **Hata Ayıklama Zorluğu**: Kod hatalarını tespit etmek zorlaşır
- **Performans Sorunları**: Performans sorunları tespit edilemez
- **Geliştirme Yavaşlığı**: Geliştirme süreci yavaşlar
- **Kullanıcı Deneyimi Sorunları**: Hata mesajları görüntülenemez
- **Veri Analizi Eksikliği**: Veri yapıları analiz edilemez

**Modern Alternatifler:**
- **Performance API**: Detaylı performans ölçümü
- **Error Reporting Services**: Profesyonel hata takibi
- **Analytics Tools**: Kullanıcı davranış analizi
- **Monitoring Tools**: Uygulama izleme araçları
- **Debugging Tools**: Gelişmiş hata ayıklama araçları

### Contact Picker API

**Contact Picker API Nedir?**
Contact Picker API, web uygulamalarının kullanıcının cihazındaki kişi listesine güvenli bir şekilde erişmesini sağlayan modern bir web API'sidir. Kullanıcıların kişi bilgilerini seçmesine ve uygulamalarda kullanmasına olanak tanır.

**Neden Kullanılır?**
- **Kişi Seçimi**: Kullanıcıların kişi listesinden seçim yapmasını sağlar
- **Güvenli Erişim**: Kişi bilgilerine güvenli erişim sağlar
- **Kullanıcı Deneyimi**: Manuel kişi girişini kolaylaştırır
- **Uygulama Entegrasyonu**: Kişi bilgilerini uygulamalarda kullanma
- **Paylaşım Kolaylığı**: Kişi bilgilerini paylaşmayı kolaylaştırır
- **Mobil Uyumluluk**: Mobil cihazlarda native deneyim sağlar

**Temel Kullanım Örnekleri:**

```javascript
// Temel kişi seçimi
async function selectContacts() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email']);
      console.log('Seçilen kişiler:', contacts);
      return contacts;
    } catch (error) {
      console.error('Kişi seçimi hatası:', error);
    }
  } else {
    console.log('Contact Picker API desteklenmiyor');
  }
}

// Çoklu alan seçimi
async function selectMultipleFields() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select([
        'name',
        'email',
        'tel',
        'address'
      ]);
      
      contacts.forEach(contact => {
        console.log('Kişi:', contact.name);
        console.log('E-posta:', contact.email);
        console.log('Telefon:', contact.tel);
        console.log('Adres:', contact.address);
      });
      
      return contacts;
    } catch (error) {
      console.error('Kişi seçimi hatası:', error);
    }
  }
}

// Tek kişi seçimi
async function selectSingleContact() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email'], {
        multiple: false
      });
      
      if (contacts.length > 0) {
        const contact = contacts[0];
        console.log('Seçilen kişi:', contact.name);
        console.log('E-posta:', contact.email);
        return contact;
      }
    } catch (error) {
      console.error('Kişi seçimi hatası:', error);
    }
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Kişi seçimi ile form doldurma
async function fillFormWithContact() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email', 'tel']);
      
      if (contacts.length > 0) {
        const contact = contacts[0];
        
        // Form alanlarını doldur
        document.getElementById('name').value = contact.name || '';
        document.getElementById('email').value = contact.email || '';
        document.getElementById('phone').value = contact.tel || '';
        
        console.log('Form kişi bilgileri ile dolduruldu');
      }
    } catch (error) {
      console.error('Form doldurma hatası:', error);
    }
  }
}

// Kişi paylaşımı
async function shareContact() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email', 'tel']);
      
      if (contacts.length > 0) {
        const contact = contacts[0];
        
        // Paylaşım verisi hazırla
        const shareData = {
          title: 'Kişi Bilgisi',
          text: `${contact.name} - ${contact.email} - ${contact.tel}`,
          url: window.location.href
        };
        
        // Web Share API ile paylaş
        if (navigator.share) {
          await navigator.share(shareData);
        } else {
          // Fallback: Clipboard API
          await navigator.clipboard.writeText(shareData.text);
          alert('Kişi bilgileri panoya kopyalandı');
        }
      }
    } catch (error) {
      console.error('Kişi paylaşım hatası:', error);
    }
  }
}

// Kişi listesi oluşturma
async function createContactList() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email'], {
        multiple: true
      });
      
      // Kişi listesi HTML'i oluştur
      const contactList = document.getElementById('contactList');
      contactList.innerHTML = '';
      
      contacts.forEach(contact => {
        const li = document.createElement('li');
        li.innerHTML = `
          <div class="contact-item">
            <h3>${contact.name || 'İsimsiz'}</h3>
            <p>${contact.email || 'E-posta yok'}</p>
            <button onclick="sendEmail('${contact.email}')">E-posta Gönder</button>
          </div>
        `;
        contactList.appendChild(li);
      });
      
      console.log(`${contacts.length} kişi seçildi`);
    } catch (error) {
      console.error('Kişi listesi oluşturma hatası:', error);
    }
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// E-posta gönderme uygulaması
async function selectEmailRecipients() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email'], {
        multiple: true
      });
      
      const recipients = contacts
        .filter(contact => contact.email)
        .map(contact => contact.email);
      
      if (recipients.length > 0) {
        // E-posta formunu doldur
        document.getElementById('to').value = recipients.join(', ');
        document.getElementById('recipientCount').textContent = 
          `${recipients.length} alıcı seçildi`;
      }
    } catch (error) {
      console.error('Alıcı seçimi hatası:', error);
    }
  }
}

// Toplantı planlama
async function selectMeetingParticipants() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email', 'tel']);
      
      const participants = contacts.map(contact => ({
        name: contact.name,
        email: contact.email,
        phone: contact.tel
      }));
      
      // Toplantı detaylarını güncelle
      updateMeetingDetails(participants);
      
    } catch (error) {
      console.error('Katılımcı seçimi hatası:', error);
    }
  }
}

// Kişi arama ve filtreleme
async function searchContacts() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email', 'tel'], {
        multiple: true
      });
      
      const searchTerm = document.getElementById('searchInput').value.toLowerCase();
      
      const filteredContacts = contacts.filter(contact => 
        contact.name?.toLowerCase().includes(searchTerm) ||
        contact.email?.toLowerCase().includes(searchTerm) ||
        contact.tel?.includes(searchTerm)
      );
      
      displayContacts(filteredContacts);
      
    } catch (error) {
      console.error('Kişi arama hatası:', error);
    }
  }
}

// Kişi bilgilerini güncelleme
async function updateContactInfo() {
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email', 'tel']);
      
      if (contacts.length > 0) {
        const contact = contacts[0];
        
        // Mevcut bilgileri göster
        document.getElementById('currentName').textContent = contact.name || 'Bilinmiyor';
        document.getElementById('currentEmail').textContent = contact.email || 'Bilinmiyor';
        document.getElementById('currentPhone').textContent = contact.tel || 'Bilinmiyor';
        
        // Güncelleme formunu aktif et
        document.getElementById('updateForm').style.display = 'block';
        
      }
    } catch (error) {
      console.error('Kişi bilgisi güncelleme hatası:', error);
    }
  }
}
```

**Güvenlik ve İzin Yönetimi:**

```javascript
// İzin kontrolü
async function checkContactPermission() {
  try {
    const permission = await navigator.permissions.query({ name: 'contacts' });
    
    switch (permission.state) {
      case 'granted':
        console.log('Kişi erişim izni verildi');
        return true;
      case 'denied':
        console.log('Kişi erişim izni reddedildi');
        return false;
      case 'prompt':
        console.log('Kişi erişim izni istenecek');
        return null;
    }
  } catch (error) {
    console.error('İzin kontrolü hatası:', error);
    return false;
  }
}

// Güvenli kişi seçimi
async function secureContactSelection() {
  // İzin kontrolü
  const hasPermission = await checkContactPermission();
  
  if (hasPermission === false) {
    alert('Kişi erişim izni gerekli');
    return;
  }
  
  if ('contacts' in navigator) {
    try {
      const contacts = await navigator.contacts.select(['name', 'email']);
      
      // Hassas bilgileri maskele
      const maskedContacts = contacts.map(contact => ({
        name: contact.name,
        email: contact.email ? 
          contact.email.replace(/(.{2}).*(@.*)/, '$1***$2') : 
          'E-posta yok'
      }));
      
      return maskedContacts;
    } catch (error) {
      console.error('Güvenli kişi seçimi hatası:', error);
    }
  }
}
```

**Alternatifler:**
- **Manuel Giriş**: Kullanıcının manuel olarak kişi bilgisi girmesi
- **QR Kod**: Kişi bilgilerini QR kod ile paylaşma
- **NFC**: Yakın alan iletişimi ile kişi paylaşımı
- **E-posta İçe Aktarma**: E-posta adreslerini manuel olarak girme
- **CSV İçe Aktarma**: Dosya yükleme ile kişi listesi

**Ne Zaman Kullanılmalı?**
- ✅ Kişi seçimi gerektiğinde
- ✅ E-posta gönderme uygulamalarında
- ✅ Toplantı planlama uygulamalarında
- ✅ Kişi paylaşımı gerektiğinde
- ✅ Mobil uygulamalarda
- ✅ Modern tarayıcı desteği gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Desktop uygulamalarda
- ❌ Hassas kişi bilgileri gerektiğinde
- ❌ Offline uygulamalarda
- ❌ Kullanıcı izni olmadan

**Kullanılmazsa Ne Olur?**
- **Manuel Giriş Gereksinimi**: Kullanıcılar manuel olarak kişi bilgisi girmek zorunda kalır
- **Kullanıcı Deneyimi Sorunları**: Kişi seçimi zorlaşır
- **Hata Riski**: Manuel girişte hata riski artar
- **Zaman Kaybı**: Kişi bilgisi girişi zaman alır
- **Mobil Uyumsuzluk**: Mobil cihazlarda native deneyim sağlanamaz

**Modern Alternatifler:**
- **Web Share API**: Native paylaşım özellikleri
- **QR Code API**: QR kod ile kişi paylaşımı
- **NFC API**: Yakın alan iletişimi
- **File System Access API**: Dosya tabanlı kişi yönetimi
- **WebRTC**: Gerçek zamanlı kişi paylaşımı

### Cookie Store API

**Cookie Store API Nedir?**
Cookie Store API, web uygulamalarında çerez (cookie) yönetimi için kullanılan modern bir web API'sidir. Çerezleri asenkron olarak oluşturma, okuma, güncelleme ve silme işlemlerini destekler. Geleneksel `document.cookie` API'sine göre daha güvenli ve kullanımı kolaydır.

**Neden Kullanılır?**
- **Asenkron İşlemler**: Çerez işlemlerini asenkron olarak gerçekleştirir
- **Güvenlik**: Güvenli çerez yönetimi sağlar
- **Kolay Kullanım**: Basit ve anlaşılır API
- **Event Desteği**: Çerez değişikliklerini dinleme
- **Service Worker Desteği**: Service worker'larda çerez yönetimi
- **Modern Özellikler**: SameSite, Secure gibi modern özellikler

**Temel Kullanım Örnekleri:**

```javascript
// Temel çerez işlemleri
async function basicCookieOperations() {
  try {
    // Çerez ayarla
    await cookieStore.set('username', 'john_doe');
    console.log('Çerez ayarlandı');
    
    // Çerez oku
    const username = await cookieStore.get('username');
    console.log('Kullanıcı adı:', username.value);
    
    // Çerez güncelle
    await cookieStore.set('username', 'jane_doe');
    console.log('Çerez güncellendi');
    
    // Çerez sil
    await cookieStore.delete('username');
    console.log('Çerez silindi');
    
  } catch (error) {
    console.error('Çerez işlemi hatası:', error);
  }
}

// Çerez seçenekleri ile ayarlama
async function setCookieWithOptions() {
  try {
    await cookieStore.set('session', 'abc123', {
      expires: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 saat
      domain: '.example.com',
      path: '/',
      secure: true,
      sameSite: 'strict'
    });
    console.log('Güvenli çerez ayarlandı');
  } catch (error) {
    console.error('Çerez ayarlama hatası:', error);
  }
}

// Çoklu çerez işlemleri
async function multipleCookieOperations() {
  try {
    // Birden fazla çerez ayarla
    await Promise.all([
      cookieStore.set('theme', 'dark'),
      cookieStore.set('language', 'tr'),
      cookieStore.set('notifications', 'enabled')
    ]);
    console.log('Çoklu çerezler ayarlandı');
    
    // Tüm çerezleri oku
    const allCookies = await cookieStore.getAll();
    console.log('Tüm çerezler:', allCookies);
    
  } catch (error) {
    console.error('Çoklu çerez işlemi hatası:', error);
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Çerez değişikliklerini dinleme
function watchCookieChanges() {
  cookieStore.addEventListener('change', (event) => {
    console.log('Çerez değişikliği:', event);
    
    event.changed.forEach(cookie => {
      console.log('Değişen çerez:', cookie.name, cookie.value);
    });
    
    event.deleted.forEach(cookie => {
      console.log('Silinen çerez:', cookie.name);
    });
  });
}

// Kullanıcı tercihlerini yönetme
class UserPreferences {
  constructor() {
    this.preferences = {
      theme: 'light',
      language: 'tr',
      notifications: true,
      fontSize: 'medium'
    };
  }
  
  async loadPreferences() {
    try {
      const cookies = await cookieStore.getAll();
      
      cookies.forEach(cookie => {
        if (this.preferences.hasOwnProperty(cookie.name)) {
          this.preferences[cookie.name] = cookie.value;
        }
      });
      
      console.log('Tercihler yüklendi:', this.preferences);
      return this.preferences;
    } catch (error) {
      console.error('Tercih yükleme hatası:', error);
    }
  }
  
  async savePreference(key, value) {
    try {
      await cookieStore.set(key, value, {
        expires: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000), // 1 yıl
        secure: true,
        sameSite: 'lax'
      });
      
      this.preferences[key] = value;
      console.log(`Tercih kaydedildi: ${key} = ${value}`);
    } catch (error) {
      console.error('Tercih kaydetme hatası:', error);
    }
  }
  
  async clearAllPreferences() {
    try {
      const cookies = await cookieStore.getAll();
      
      await Promise.all(
        cookies.map(cookie => cookieStore.delete(cookie.name))
      );
      
      this.preferences = {
        theme: 'light',
        language: 'tr',
        notifications: true,
        fontSize: 'medium'
      };
      
      console.log('Tüm tercihler temizlendi');
    } catch (error) {
      console.error('Tercih temizleme hatası:', error);
    }
  }
}

// Session yönetimi
class SessionManager {
  constructor() {
    this.sessionId = null;
    this.sessionData = {};
  }
  
  async createSession(userData) {
    try {
      this.sessionId = this.generateSessionId();
      this.sessionData = userData;
      
      await cookieStore.set('sessionId', this.sessionId, {
        expires: new Date(Date.now() + 2 * 60 * 60 * 1000), // 2 saat
        secure: true,
        sameSite: 'strict',
        httpOnly: true
      });
      
      // Session verilerini ayrı çerezde sakla
      await cookieStore.set('sessionData', JSON.stringify(this.sessionData), {
        expires: new Date(Date.now() + 2 * 60 * 60 * 1000),
        secure: true,
        sameSite: 'strict'
      });
      
      console.log('Session oluşturuldu:', this.sessionId);
      return this.sessionId;
    } catch (error) {
      console.error('Session oluşturma hatası:', error);
    }
  }
  
  async getSession() {
    try {
      const sessionId = await cookieStore.get('sessionId');
      const sessionData = await cookieStore.get('sessionData');
      
      if (sessionId && sessionData) {
        this.sessionId = sessionId.value;
        this.sessionData = JSON.parse(sessionData.value);
        return this.sessionData;
      }
      
      return null;
    } catch (error) {
      console.error('Session okuma hatası:', error);
      return null;
    }
  }
  
  async destroySession() {
    try {
      await Promise.all([
        cookieStore.delete('sessionId'),
        cookieStore.delete('sessionData')
      ]);
      
      this.sessionId = null;
      this.sessionData = {};
      console.log('Session sonlandırıldı');
    } catch (error) {
      console.error('Session sonlandırma hatası:', error);
    }
  }
  
  generateSessionId() {
    return 'session_' + Math.random().toString(36).substr(2, 9);
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// E-ticaret sepet yönetimi
class ShoppingCart {
  constructor() {
    this.cartItems = [];
  }
  
  async loadCart() {
    try {
      const cartData = await cookieStore.get('cart');
      
      if (cartData) {
        this.cartItems = JSON.parse(cartData.value);
        console.log('Sepet yüklendi:', this.cartItems);
      }
      
      return this.cartItems;
    } catch (error) {
      console.error('Sepet yükleme hatası:', error);
    }
  }
  
  async addToCart(product) {
    try {
      const existingItem = this.cartItems.find(item => item.id === product.id);
      
      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        this.cartItems.push({ ...product, quantity: 1 });
      }
      
      await cookieStore.set('cart', JSON.stringify(this.cartItems), {
        expires: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000), // 30 gün
        secure: true,
        sameSite: 'lax'
      });
      
      console.log('Ürün sepete eklendi:', product.name);
    } catch (error) {
      console.error('Sepete ekleme hatası:', error);
    }
  }
  
  async removeFromCart(productId) {
    try {
      this.cartItems = this.cartItems.filter(item => item.id !== productId);
      
      await cookieStore.set('cart', JSON.stringify(this.cartItems), {
        expires: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
        secure: true,
        sameSite: 'lax'
      });
      
      console.log('Ürün sepetten çıkarıldı:', productId);
    } catch (error) {
      console.error('Sepetten çıkarma hatası:', error);
    }
  }
  
  async clearCart() {
    try {
      await cookieStore.delete('cart');
      this.cartItems = [];
      console.log('Sepet temizlendi');
    } catch (error) {
      console.error('Sepet temizleme hatası:', error);
    }
  }
}

// A/B test yönetimi
class ABTestManager {
  constructor() {
    this.tests = {};
  }
  
  async getTestVariant(testName) {
    try {
      const testCookie = await cookieStore.get(`ab_test_${testName}`);
      
      if (testCookie) {
        return testCookie.value;
      }
      
      // Yeni test variantı oluştur
      const variant = Math.random() < 0.5 ? 'A' : 'B';
      
      await cookieStore.set(`ab_test_${testName}`, variant, {
        expires: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 gün
        secure: true,
        sameSite: 'lax'
      });
      
      console.log(`A/B test variantı: ${testName} = ${variant}`);
      return variant;
    } catch (error) {
      console.error('A/B test hatası:', error);
      return 'A'; // Varsayılan variant
    }
  }
  
  async trackTestEvent(testName, event) {
    try {
      const variant = await this.getTestVariant(testName);
      
      // Test olayını kaydet
      const eventData = {
        test: testName,
        variant: variant,
        event: event,
        timestamp: Date.now()
      };
      
      // Analytics servisine gönder
      await this.sendAnalytics(eventData);
      
      console.log('Test olayı kaydedildi:', eventData);
    } catch (error) {
      console.error('Test olayı kaydetme hatası:', error);
    }
  }
  
  async sendAnalytics(data) {
    // Analytics servisine veri gönderme
    console.log('Analytics verisi:', data);
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli çerez yönetimi
class SecureCookieManager {
  constructor() {
    this.defaultOptions = {
      secure: true,
      sameSite: 'strict',
      httpOnly: false // JavaScript'ten erişilebilir olması için
    };
  }
  
  async setSecureCookie(name, value, options = {}) {
    try {
      const cookieOptions = {
        ...this.defaultOptions,
        ...options,
        expires: options.expires || new Date(Date.now() + 24 * 60 * 60 * 1000)
      };
      
      await cookieStore.set(name, value, cookieOptions);
      console.log(`Güvenli çerez ayarlandı: ${name}`);
    } catch (error) {
      console.error('Güvenli çerez ayarlama hatası:', error);
    }
  }
  
  async getSecureCookie(name) {
    try {
      const cookie = await cookieStore.get(name);
      return cookie ? cookie.value : null;
    } catch (error) {
      console.error('Güvenli çerez okuma hatası:', error);
      return null;
    }
  }
  
  async deleteSecureCookie(name) {
    try {
      await cookieStore.delete(name);
      console.log(`Güvenli çerez silindi: ${name}`);
    } catch (error) {
      console.error('Güvenli çerez silme hatası:', error);
    }
  }
  
  // Çerez boyutunu kontrol et
  async checkCookieSize(name, value) {
    const cookieString = `${name}=${value}`;
    
    if (cookieString.length > 4096) {
      console.warn('Çerez boyutu çok büyük:', cookieString.length);
      return false;
    }
    
    return true;
  }
  
  // Çerez sayısını kontrol et
  async checkCookieCount() {
    try {
      const cookies = await cookieStore.getAll();
      
      if (cookies.length > 50) {
        console.warn('Çok fazla çerez var:', cookies.length);
        return false;
      }
      
      return true;
    } catch (error) {
      console.error('Çerez sayısı kontrolü hatası:', error);
      return false;
    }
  }
}
```

**Alternatifler:**
- **document.cookie**: Geleneksel çerez API'si
- **localStorage**: Yerel depolama
- **sessionStorage**: Oturum depolama
- **IndexedDB**: Büyük veri depolama
- **Web Storage API**: Genel depolama API'si

**Ne Zaman Kullanılmalı?**
- ✅ Çerez yönetimi gerektiğinde
- ✅ Kullanıcı tercihlerini saklama gerektiğinde
- ✅ Session yönetimi gerektiğinde
- ✅ A/B test yapma gerektiğinde
- ✅ Modern tarayıcı desteği gerektiğinde
- ✅ Güvenli çerez yönetimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Büyük veri depolama gerektiğinde
- ❌ Offline uygulamalarda
- ❌ Hassas veri depolama gerektiğinde
- ❌ Çok fazla çerez gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Geleneksel API Kullanımı**: `document.cookie` ile sınırlı kalınır
- **Senkron İşlemler**: Çerez işlemleri senkron olarak yapılır
- **Güvenlik Riskleri**: Güvenli çerez yönetimi sağlanamaz
- **Event Desteği Eksikliği**: Çerez değişiklikleri dinlenemez
- **Service Worker Uyumsuzluğu**: Service worker'larda çerez yönetimi zorlaşır

**Modern Alternatifler:**
- **Web Storage API**: Genel depolama çözümleri
- **IndexedDB**: Büyük veri depolama
- **Web Locks API**: Çerez kilitleme
- **Storage Access API**: Çerez erişim kontrolü
- **Partitioned Cookies**: Bölümlenmiş çerezler

### Credential Management API

**Credential Management API Nedir?**
Credential Management API, web uygulamalarında kullanıcı kimlik bilgilerini güvenli bir şekilde yönetmek için kullanılan modern bir web API'sidir. Şifreler, biyometrik veriler ve diğer kimlik doğrulama bilgilerini saklama, alma ve yönetme işlemlerini destekler.

**Neden Kullanılır?**
- **Güvenli Kimlik Yönetimi**: Kimlik bilgilerini güvenli şekilde saklar
- **Otomatik Doldurma**: Tarayıcı şifre yöneticisi ile entegrasyon
- **Biyometrik Desteği**: Parmak izi, yüz tanıma gibi biyometrik doğrulama
- **Kullanıcı Deneyimi**: Otomatik giriş ve şifre yönetimi
- **Güvenlik**: Şifrelerin güvenli saklanması ve yönetimi
- **Modern Doğrulama**: WebAuthn ve FIDO2 desteği

**Temel Kullanım Örnekleri:**

```javascript
// Temel kimlik bilgisi kaydetme
async function storeCredentials() {
  try {
    const credential = new PasswordCredential({
      id: 'user@example.com',
      password: 'password123',
      name: 'John Doe',
      iconURL: 'https://example.com/avatar.jpg'
    });
    
    const storedCredential = await navigator.credentials.store(credential);
    console.log('Kimlik bilgisi kaydedildi:', storedCredential);
  } catch (error) {
    console.error('Kimlik bilgisi kaydetme hatası:', error);
  }
}

// Kimlik bilgisi alma
async function getCredentials() {
  try {
    const credential = await navigator.credentials.get({
      password: true,
      federated: {
        providers: ['https://accounts.google.com']
      }
    });
    
    if (credential) {
      console.log('Kimlik bilgisi alındı:', credential);
      return credential;
    }
  } catch (error) {
    console.error('Kimlik bilgisi alma hatası:', error);
  }
}

// Kimlik bilgisi silme
async function deleteCredentials() {
  try {
    await navigator.credentials.preventSilentAccess();
    console.log('Sessiz erişim engellendi');
  } catch (error) {
    console.error('Kimlik bilgisi silme hatası:', error);
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// WebAuthn ile biyometrik doğrulama
class BiometricAuth {
  constructor() {
    this.publicKeyCredential = null;
  }
  
  async registerBiometric(userInfo) {
    try {
      const publicKeyCredentialCreationOptions = {
        challenge: new Uint8Array(32),
        rp: {
          name: "Example Corp",
          id: "example.com",
        },
        user: {
          id: new TextEncoder().encode(userInfo.id),
          name: userInfo.name,
          displayName: userInfo.displayName,
        },
        pubKeyCredParams: [
          {
            type: "public-key",
            alg: -7, // ES256
          },
        ],
        authenticatorSelection: {
          authenticatorAttachment: "platform",
          userVerification: "required",
        },
        timeout: 60000,
        attestation: "direct"
      };
      
      const credential = await navigator.credentials.create({
        publicKey: publicKeyCredentialCreationOptions
      });
      
      this.publicKeyCredential = credential;
      console.log('Biyometrik kimlik oluşturuldu:', credential);
      return credential;
    } catch (error) {
      console.error('Biyometrik kayıt hatası:', error);
    }
  }
  
  async authenticateBiometric() {
    try {
      const publicKeyCredentialRequestOptions = {
        challenge: new Uint8Array(32),
        allowCredentials: [{
          id: this.publicKeyCredential.rawId,
          type: 'public-key',
          transports: ['internal']
        }],
        timeout: 60000,
        userVerification: "required"
      };
      
      const assertion = await navigator.credentials.get({
        publicKey: publicKeyCredentialRequestOptions
      });
      
      console.log('Biyometrik doğrulama başarılı:', assertion);
      return assertion;
    } catch (error) {
      console.error('Biyometrik doğrulama hatası:', error);
    }
  }
}

// Federated kimlik doğrulama
class FederatedAuth {
  constructor() {
    this.providers = [
      'https://accounts.google.com',
      'https://www.facebook.com',
      'https://github.com'
    ];
  }
  
  async getFederatedCredentials() {
    try {
      const credential = await navigator.credentials.get({
        federated: {
          providers: this.providers
        }
      });
      
      if (credential) {
        console.log('Federated kimlik alındı:', credential);
        return credential;
      }
    } catch (error) {
      console.error('Federated kimlik hatası:', error);
    }
  }
  
  async storeFederatedCredentials(provider, userInfo) {
    try {
      const credential = new FederatedCredential({
        id: userInfo.id,
        name: userInfo.name,
        iconURL: userInfo.avatar,
        provider: provider
      });
      
      const storedCredential = await navigator.credentials.store(credential);
      console.log('Federated kimlik kaydedildi:', storedCredential);
      return storedCredential;
    } catch (error) {
      console.error('Federated kimlik kaydetme hatası:', error);
    }
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Otomatik giriş sistemi
class AutoLogin {
  constructor() {
    this.isAutoLoginEnabled = false;
  }
  
  async enableAutoLogin() {
    try {
      const credential = await navigator.credentials.get({
        password: true,
        mediation: 'silent'
      });
      
      if (credential) {
        await this.performLogin(credential);
        this.isAutoLoginEnabled = true;
        console.log('Otomatik giriş etkinleştirildi');
      }
    } catch (error) {
      console.error('Otomatik giriş hatası:', error);
    }
  }
  
  async performLogin(credential) {
    try {
      const formData = new FormData();
      formData.append('username', credential.id);
      formData.append('password', credential.password);
      
      const response = await fetch('/login', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        console.log('Giriş başarılı');
        return true;
      }
    } catch (error) {
      console.error('Giriş hatası:', error);
    }
  }
  
  async disableAutoLogin() {
    try {
      await navigator.credentials.preventSilentAccess();
      this.isAutoLoginEnabled = false;
      console.log('Otomatik giriş devre dışı bırakıldı');
    } catch (error) {
      console.error('Otomatik giriş devre dışı bırakma hatası:', error);
    }
  }
}

// Şifre yöneticisi entegrasyonu
class PasswordManager {
  constructor() {
    this.savedPasswords = [];
  }
  
  async savePassword(username, password, siteInfo) {
    try {
      const credential = new PasswordCredential({
        id: username,
        password: password,
        name: siteInfo.name,
        iconURL: siteInfo.icon
      });
      
      const storedCredential = await navigator.credentials.store(credential);
      this.savedPasswords.push(storedCredential);
      console.log('Şifre kaydedildi:', username);
      return storedCredential;
    } catch (error) {
      console.error('Şifre kaydetme hatası:', error);
    }
  }
  
  async getSavedPasswords() {
    try {
      const credential = await navigator.credentials.get({
        password: true,
        mediation: 'optional'
      });
      
      if (credential) {
        console.log('Kaydedilmiş şifre alındı:', credential);
        return credential;
      }
    } catch (error) {
      console.error('Şifre alma hatası:', error);
    }
  }
  
  async updatePassword(username, newPassword) {
    try {
      const credential = new PasswordCredential({
        id: username,
        password: newPassword
      });
      
      const updatedCredential = await navigator.credentials.store(credential);
      console.log('Şifre güncellendi:', username);
      return updatedCredential;
    } catch (error) {
      console.error('Şifre güncelleme hatası:', error);
    }
  }
}

// Çok faktörlü kimlik doğrulama
class MultiFactorAuth {
  constructor() {
    this.factors = ['password', 'biometric', 'totp'];
  }
  
  async authenticateWithPassword(username, password) {
    try {
      const credential = new PasswordCredential({
        id: username,
        password: password
      });
      
      const storedCredential = await navigator.credentials.store(credential);
      console.log('Şifre doğrulaması başarılı');
      return storedCredential;
    } catch (error) {
      console.error('Şifre doğrulama hatası:', error);
    }
  }
  
  async authenticateWithBiometric() {
    try {
      const biometricAuth = new BiometricAuth();
      const assertion = await biometricAuth.authenticateBiometric();
      
      if (assertion) {
        console.log('Biyometrik doğrulama başarılı');
        return assertion;
      }
    } catch (error) {
      console.error('Biyometrik doğrulama hatası:', error);
    }
  }
  
  async authenticateWithTOTP(totpCode) {
    try {
      // TOTP doğrulama mantığı
      const isValid = await this.validateTOTP(totpCode);
      
      if (isValid) {
        console.log('TOTP doğrulaması başarılı');
        return true;
      }
    } catch (error) {
      console.error('TOTP doğrulama hatası:', error);
    }
  }
  
  async validateTOTP(code) {
    // TOTP doğrulama implementasyonu
    return code.length === 6 && /^\d+$/.test(code);
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli kimlik yönetimi
class SecureCredentialManager {
  constructor() {
    this.securityLevel = 'high';
  }
  
  async storeSecureCredential(credentialData) {
    try {
      // Güvenlik kontrolleri
      if (!this.validateCredentialData(credentialData)) {
        throw new Error('Geçersiz kimlik bilgisi');
      }
      
      const credential = new PasswordCredential({
        id: credentialData.username,
        password: credentialData.password,
        name: credentialData.name,
        iconURL: credentialData.avatar
      });
      
      const storedCredential = await navigator.credentials.store(credential);
      console.log('Güvenli kimlik bilgisi kaydedildi');
      return storedCredential;
    } catch (error) {
      console.error('Güvenli kimlik kaydetme hatası:', error);
    }
  }
  
  validateCredentialData(data) {
    // Kimlik bilgisi doğrulama
    if (!data.username || !data.password) {
      return false;
    }
    
    if (data.password.length < 8) {
      return false;
    }
    
    return true;
  }
  
  async getSecureCredentials() {
    try {
      const credential = await navigator.credentials.get({
        password: true,
        mediation: 'required'
      });
      
      if (credential) {
        // Güvenlik loglama
        this.logSecurityEvent('credential_access', credential.id);
        return credential;
      }
    } catch (error) {
      console.error('Güvenli kimlik alma hatası:', error);
    }
  }
  
  logSecurityEvent(event, userId) {
    console.log(`Güvenlik olayı: ${event} - Kullanıcı: ${userId} - Zaman: ${new Date().toISOString()}`);
  }
  
  async clearAllCredentials() {
    try {
      await navigator.credentials.preventSilentAccess();
      console.log('Tüm kimlik bilgileri temizlendi');
    } catch (error) {
      console.error('Kimlik bilgisi temizleme hatası:', error);
    }
  }
}
```

**Alternatifler:**
- **Manuel Giriş**: Kullanıcının manuel olarak kimlik bilgisi girmesi
- **Session Yönetimi**: Sunucu tarafında oturum yönetimi
- **JWT Tokens**: JSON Web Token tabanlı kimlik doğrulama
- **OAuth 2.0**: Üçüncü taraf kimlik doğrulama
- **SAML**: Güvenlik Assertion Markup Language

**Ne Zaman Kullanılmalı?**
- ✅ Güvenli kimlik yönetimi gerektiğinde
- ✅ Otomatik giriş özelliği gerektiğinde
- ✅ Biyometrik doğrulama gerektiğinde
- ✅ Modern tarayıcı desteği gerektiğinde
- ✅ Kullanıcı deneyimi öncelikli olduğunda
- ✅ WebAuthn desteği gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Hassas güvenlik gereksinimleri olduğunda
- ❌ Offline uygulamalarda
- ❌ Kullanıcı gizliliği öncelikli olduğunda
- ❌ Çok faktörlü doğrulama gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Manuel Giriş Gereksinimi**: Kullanıcılar manuel olarak kimlik bilgisi girmek zorunda kalır
- **Güvenlik Riskleri**: Kimlik bilgileri güvenli şekilde saklanamaz
- **Kullanıcı Deneyimi Sorunları**: Otomatik giriş yapılamaz
- **Biyometrik Desteği Eksikliği**: Modern doğrulama yöntemleri kullanılamaz
- **Şifre Yöneticisi Uyumsuzluğu**: Tarayıcı şifre yöneticisi ile entegrasyon sağlanamaz

**Modern Alternatifler:**
- **WebAuthn**: Modern kimlik doğrulama standardı
- **FIDO2**: FIDO Alliance kimlik doğrulama
- **Passkeys**: Apple'ın kimlik doğrulama sistemi
- **WebOTP API**: SMS tabanlı doğrulama
- **Web Crypto API**: Kriptografik işlemler

---

## D - Bölümü

### Device Memory API

**Device Memory API Nedir?**
Device Memory API, web uygulamalarının cihazın toplam RAM miktarını öğrenmesini sağlayan modern bir web API'sidir. Bu bilgiyi kullanarak uygulamalar, cihazın bellek kapasitesine göre performans optimizasyonları yapabilir ve kullanıcı deneyimini iyileştirebilir.

**Neden Kullanılır?**
- **Performans Optimizasyonu**: Cihaz bellek kapasitesine göre uygulama davranışını ayarlama
- **Kaynak Yönetimi**: Bellek kullanımını optimize etme
- **Kullanıcı Deneyimi**: Düşük bellekli cihazlarda daha hafif deneyim sunma
- **Adaptif Uygulama**: Cihaz özelliklerine göre uygulama özelliklerini ayarlama
- **Battery Optimization**: Pil tüketimini optimize etme
- **Memory Management**: Bellek yönetimi stratejileri geliştirme

**Temel Kullanım Örnekleri:**

```javascript
// Temel bellek bilgisi alma
function getDeviceMemory() {
  if ('deviceMemory' in navigator) {
    const memory = navigator.deviceMemory;
    console.log(`Cihaz belleği: ${memory} GB`);
    return memory;
  } else {
    console.log('Device Memory API desteklenmiyor');
    return null;
  }
}

// Bellek kapasitesine göre uygulama ayarları
function configureAppBasedOnMemory() {
  const memory = getDeviceMemory();
  
  if (memory) {
    if (memory >= 8) {
      // Yüksek bellekli cihaz - tüm özellikleri etkinleştir
      enableAdvancedFeatures();
      setHighQualitySettings();
    } else if (memory >= 4) {
      // Orta bellekli cihaz - bazı özellikleri sınırla
      enableBasicFeatures();
      setMediumQualitySettings();
    } else {
      // Düşük bellekli cihaz - minimal özellikler
      enableMinimalFeatures();
      setLowQualitySettings();
    }
  }
}

// Bellek durumu kontrolü
function checkMemoryStatus() {
  const memory = getDeviceMemory();
  
  if (memory) {
    const status = memory >= 4 ? 'Yeterli' : 'Sınırlı';
    console.log(`Bellek durumu: ${status} (${memory} GB)`);
    return status;
  }
  
  return 'Bilinmiyor';
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Adaptif uygulama yapılandırması
class AdaptiveAppConfig {
  constructor() {
    this.memory = this.getDeviceMemory();
    this.config = this.generateConfig();
  }
  
  getDeviceMemory() {
    return 'deviceMemory' in navigator ? navigator.deviceMemory : null;
  }
  
  generateConfig() {
    const memory = this.memory;
    
    if (!memory) {
      return this.getDefaultConfig();
    }
    
    if (memory >= 8) {
      return {
        maxConcurrentRequests: 10,
        imageQuality: 'high',
        enableAnimations: true,
        enableWebGL: true,
        maxCacheSize: 100, // MB
        enableBackgroundSync: true
      };
    } else if (memory >= 4) {
      return {
        maxConcurrentRequests: 5,
        imageQuality: 'medium',
        enableAnimations: true,
        enableWebGL: false,
        maxCacheSize: 50,
        enableBackgroundSync: false
      };
    } else {
      return {
        maxConcurrentRequests: 2,
        imageQuality: 'low',
        enableAnimations: false,
        enableWebGL: false,
        maxCacheSize: 20,
        enableBackgroundSync: false
      };
    }
  }
  
  getDefaultConfig() {
    return {
      maxConcurrentRequests: 3,
      imageQuality: 'medium',
      enableAnimations: false,
      enableWebGL: false,
      maxCacheSize: 30,
      enableBackgroundSync: false
    };
  }
  
  applyConfig() {
    // Uygulama yapılandırmasını uygula
    console.log('Uygulama yapılandırması:', this.config);
    
    // CSS değişkenleri ile tema ayarları
    document.documentElement.style.setProperty('--image-quality', this.config.imageQuality);
    document.documentElement.style.setProperty('--enable-animations', this.config.enableAnimations);
    
    return this.config;
  }
}

// Bellek tabanlı özellik yönetimi
class MemoryBasedFeatureManager {
  constructor() {
    this.memory = this.getDeviceMemory();
    this.features = this.initializeFeatures();
  }
  
  getDeviceMemory() {
    return 'deviceMemory' in navigator ? navigator.deviceMemory : null;
  }
  
  initializeFeatures() {
    const memory = this.memory;
    
    return {
      imageOptimization: memory >= 2,
      videoStreaming: memory >= 4,
      realTimeUpdates: memory >= 4,
      backgroundProcessing: memory >= 6,
      advancedAnimations: memory >= 8,
      webGL: memory >= 4,
      serviceWorker: memory >= 2
    };
  }
  
  isFeatureEnabled(featureName) {
    return this.features[featureName] || false;
  }
  
  enableFeature(featureName) {
    if (this.isFeatureEnabled(featureName)) {
      console.log(`Özellik etkinleştirildi: ${featureName}`);
      return true;
    } else {
      console.log(`Özellik devre dışı: ${featureName} (Yetersiz bellek)`);
      return false;
    }
  }
  
  getAvailableFeatures() {
    return Object.keys(this.features).filter(feature => this.features[feature]);
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Görsel içerik optimizasyonu
class ImageOptimizer {
  constructor() {
    this.memory = this.getDeviceMemory();
    this.qualitySettings = this.getQualitySettings();
  }
  
  getDeviceMemory() {
    return 'deviceMemory' in navigator ? navigator.deviceMemory : null;
  }
  
  getQualitySettings() {
    const memory = this.memory;
    
    if (memory >= 8) {
      return {
        maxWidth: 1920,
        maxHeight: 1080,
        quality: 0.9,
        format: 'webp'
      };
    } else if (memory >= 4) {
      return {
        maxWidth: 1280,
        maxHeight: 720,
        quality: 0.7,
        format: 'jpeg'
      };
    } else {
      return {
        maxWidth: 800,
        maxHeight: 600,
        quality: 0.5,
        format: 'jpeg'
      };
    }
  }
  
  optimizeImage(imageUrl) {
    const settings = this.qualitySettings;
    
    // Görsel optimizasyon parametreleri
    const optimizedUrl = `${imageUrl}?w=${settings.maxWidth}&h=${settings.maxHeight}&q=${settings.quality}&f=${settings.format}`;
    
    console.log(`Görsel optimize edildi: ${optimizedUrl}`);
    return optimizedUrl;
  }
  
  shouldPreloadImages() {
    return this.memory >= 4;
  }
  
  getMaxConcurrentImages() {
    const memory = this.memory;
    
    if (memory >= 8) return 10;
    if (memory >= 4) return 5;
    return 2;
  }
}

// Bellek tabanlı cache yönetimi
class MemoryBasedCache {
  constructor() {
    this.memory = this.getDeviceMemory();
    this.maxCacheSize = this.getMaxCacheSize();
    this.cache = new Map();
  }
  
  getDeviceMemory() {
    return 'deviceMemory' in navigator ? navigator.deviceMemory : null;
  }
  
  getMaxCacheSize() {
    const memory = this.memory;
    
    if (memory >= 8) return 100; // MB
    if (memory >= 4) return 50;
    if (memory >= 2) return 25;
    return 10;
  }
  
  set(key, value) {
    const currentSize = this.getCacheSize();
    const itemSize = this.getItemSize(value);
    
    if (currentSize + itemSize > this.maxCacheSize) {
      this.evictOldestItems();
    }
    
    this.cache.set(key, {
      value: value,
      timestamp: Date.now(),
      size: itemSize
    });
    
    console.log(`Cache'e eklendi: ${key} (${itemSize} MB)`);
  }
  
  get(key) {
    const item = this.cache.get(key);
    
    if (item) {
      // LRU güncelleme
      item.timestamp = Date.now();
      return item.value;
    }
    
    return null;
  }
  
  getCacheSize() {
    let totalSize = 0;
    for (const [key, item] of this.cache) {
      totalSize += item.size;
    }
    return totalSize;
  }
  
  getItemSize(value) {
    // Basit boyut hesaplama
    return JSON.stringify(value).length / 1024 / 1024; // MB
  }
  
  evictOldestItems() {
    const items = Array.from(this.cache.entries());
    items.sort((a, b) => a[1].timestamp - b[1].timestamp);
    
    // En eski %20'yi sil
    const toDelete = Math.ceil(items.length * 0.2);
    
    for (let i = 0; i < toDelete; i++) {
      this.cache.delete(items[i][0]);
    }
    
    console.log(`${toDelete} öğe cache'den silindi`);
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli bellek yönetimi
class SecureMemoryManager {
  constructor() {
    this.memory = this.getDeviceMemory();
    this.securityLevel = this.determineSecurityLevel();
  }
  
  getDeviceMemory() {
    return 'deviceMemory' in navigator ? navigator.deviceMemory : null;
  }
  
  determineSecurityLevel() {
    const memory = this.memory;
    
    if (memory >= 8) {
      return 'high'; // Yüksek güvenlik, tüm özellikler
    } else if (memory >= 4) {
      return 'medium'; // Orta güvenlik, sınırlı özellikler
    } else {
      return 'low'; // Düşük güvenlik, minimal özellikler
    }
  }
  
  shouldEnableEncryption() {
    return this.securityLevel === 'high';
  }
  
  shouldEnableCompression() {
    return this.memory < 4;
  }
  
  getMaxDataSize() {
    const memory = this.memory;
    
    if (memory >= 8) return 50; // MB
    if (memory >= 4) return 20;
    return 5;
  }
  
  validateDataSize(data) {
    const maxSize = this.getMaxDataSize();
    const dataSize = this.getDataSize(data);
    
    if (dataSize > maxSize) {
      console.warn(`Veri boyutu çok büyük: ${dataSize} MB > ${maxSize} MB`);
      return false;
    }
    
    return true;
  }
  
  getDataSize(data) {
    return JSON.stringify(data).length / 1024 / 1024; // MB
  }
}
```

**Alternatifler:**
- **Performance API**: Performans ölçümü
- **Navigator API**: Tarayıcı bilgileri
- **Hardware Concurrency**: CPU çekirdek sayısı
- **Connection API**: Ağ bağlantı bilgileri
- **Battery API**: Pil durumu bilgileri

**Ne Zaman Kullanılmalı?**
- ✅ Performans optimizasyonu gerektiğinde
- ✅ Adaptif uygulama geliştirme gerektiğinde
- ✅ Bellek yönetimi gerektiğinde
- ✅ Kullanıcı deneyimi iyileştirme gerektiğinde
- ✅ Kaynak optimizasyonu gerektiğinde
- ✅ Modern tarayıcı desteği gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Hassas güvenlik gereksinimleri olduğunda
- ❌ Offline uygulamalarda
- ❌ Kullanıcı gizliliği öncelikli olduğunda
- ❌ Basit uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Performans Sorunları**: Düşük bellekli cihazlarda yavaşlık
- **Kullanıcı Deneyimi Sorunları**: Uygulama donmaları ve hatalar
- **Kaynak İsrafı**: Gereksiz bellek kullanımı
- **Battery Drain**: Pil tüketimi artışı
- **Uyumsuzluk**: Farklı cihazlarda tutarsız deneyim

**Modern Alternatifler:**
- **Performance Observer API**: Detaylı performans izleme
- **Memory API**: Bellek kullanım izleme
- **Resource Hints API**: Kaynak optimizasyonu
- **Intersection Observer API**: Görünürlük tabanlı yükleme
- **Web Workers**: Arka planda işlem yapma

### Device Orientation Events

**Device Orientation Events Nedir?**
Device Orientation Events, web uygulamalarının cihazın yönelim değişikliklerini algılamasını sağlayan bir web API'sidir. Cihazın döndürülmesi, eğilmesi ve yatay/dikey konum değişikliklerini takip eder. Mobil cihazlarda özellikle yararlıdır.

**Neden Kullanılır?**
- **Cihaz Yönelim Algılama**: Cihazın döndürülmesini ve eğilmesini algılama
- **Mobil Uygulama Geliştirme**: Mobil cihazlarda yönelim tabanlı özellikler
- **Oyun Geliştirme**: Cihaz hareketlerini oyun kontrolleri olarak kullanma
- **AR/VR Uygulamaları**: Artırılmış ve sanal gerçeklik uygulamaları
- **Kullanıcı Deneyimi**: Cihaz yönelimine göre arayüz ayarlama
- **Sensör Entegrasyonu**: Cihaz sensörlerini web uygulamalarında kullanma

**Temel Kullanım Örnekleri:**

```javascript
// Temel yönelim algılama
function setupOrientationListener() {
  if (window.DeviceOrientationEvent) {
    window.addEventListener('deviceorientation', handleOrientation);
    console.log('Device Orientation Events destekleniyor');
  } else {
    console.log('Device Orientation Events desteklenmiyor');
  }
}

function handleOrientation(event) {
  const alpha = event.alpha; // Z ekseni etrafında döndürme (0-360)
  const beta = event.beta;   // X ekseni etrafında eğilme (-180-180)
  const gamma = event.gamma; // Y ekseni etrafında eğilme (-90-90)
  
  console.log(`Alpha: ${alpha.toFixed(2)}°`);
  console.log(`Beta: ${beta.toFixed(2)}°`);
  console.log(`Gamma: ${gamma.toFixed(2)}°`);
  
  // Yönelim bilgilerini kullan
  updateUI(alpha, beta, gamma);
}

// Hareket algılama
function setupMotionListener() {
  if (window.DeviceMotionEvent) {
    window.addEventListener('devicemotion', handleMotion);
    console.log('Device Motion Events destekleniyor');
  } else {
    console.log('Device Motion Events desteklenmiyor');
  }
}

function handleMotion(event) {
  const acceleration = event.acceleration;
  const accelerationIncludingGravity = event.accelerationIncludingGravity;
  const rotationRate = event.rotationRate;
  
  console.log('Hızlanma:', acceleration);
  console.log('Yerçekimi dahil hızlanma:', accelerationIncludingGravity);
  console.log('Döndürme hızı:', rotationRate);
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Yönelim tabanlı oyun kontrolü
class OrientationGameController {
  constructor() {
    this.alpha = 0;
    this.beta = 0;
    this.gamma = 0;
    this.isEnabled = false;
    this.callbacks = {
      onOrientationChange: null,
      onMotion: null
    };
  }
  
  enable() {
    if (window.DeviceOrientationEvent) {
      window.addEventListener('deviceorientation', this.handleOrientation.bind(this));
      this.isEnabled = true;
      console.log('Yönelim kontrolü etkinleştirildi');
    }
  }
  
  disable() {
    if (this.isEnabled) {
      window.removeEventListener('deviceorientation', this.handleOrientation.bind(this));
      this.isEnabled = false;
      console.log('Yönelim kontrolü devre dışı bırakıldı');
    }
  }
  
  handleOrientation(event) {
    this.alpha = event.alpha;
    this.beta = event.beta;
    this.gamma = event.gamma;
    
    if (this.callbacks.onOrientationChange) {
      this.callbacks.onOrientationChange({
        alpha: this.alpha,
        beta: this.beta,
        gamma: this.gamma
      });
    }
  }
  
  onOrientationChange(callback) {
    this.callbacks.onOrientationChange = callback;
  }
  
  // Yönelim değerlerini normalize et
  normalizeOrientation() {
    return {
      alpha: this.alpha / 360, // 0-1 arası
      beta: (this.beta + 180) / 360, // 0-1 arası
      gamma: (this.gamma + 90) / 180 // 0-1 arası
    };
  }
}

// Hareket tabanlı etkileşim
class MotionInteraction {
  constructor() {
    this.acceleration = { x: 0, y: 0, z: 0 };
    this.rotationRate = { alpha: 0, beta: 0, gamma: 0 };
    this.isEnabled = false;
    this.threshold = 0.5; // Hareket eşiği
  }
  
  enable() {
    if (window.DeviceMotionEvent) {
      window.addEventListener('devicemotion', this.handleMotion.bind(this));
      this.isEnabled = true;
      console.log('Hareket etkileşimi etkinleştirildi');
    }
  }
  
  disable() {
    if (this.isEnabled) {
      window.removeEventListener('devicemotion', this.handleMotion.bind(this));
      this.isEnabled = false;
      console.log('Hareket etkileşimi devre dışı bırakıldı');
    }
  }
  
  handleMotion(event) {
    this.acceleration = event.acceleration;
    this.rotationRate = event.rotationRate;
    
    // Hareket algılama
    this.detectMotion();
  }
  
  detectMotion() {
    const { x, y, z } = this.acceleration;
    const magnitude = Math.sqrt(x * x + y * y + z * z);
    
    if (magnitude > this.threshold) {
      console.log('Hareket algılandı:', magnitude);
      this.onMotionDetected(magnitude);
    }
  }
  
  onMotionDetected(magnitude) {
    // Hareket algılandığında yapılacak işlemler
    console.log('Hareket büyüklüğü:', magnitude);
  }
  
  // Sallama hareketi algılama
  detectShake() {
    const { x, y, z } = this.acceleration;
    const totalAcceleration = Math.abs(x) + Math.abs(y) + Math.abs(z);
    
    if (totalAcceleration > 15) { // Sallama eşiği
      console.log('Sallama hareketi algılandı');
      return true;
    }
    
    return false;
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Cihaz yönelimine göre arayüz ayarlama
class OrientationBasedUI {
  constructor() {
    this.currentOrientation = 'portrait';
    this.orientationThreshold = 45; // Derece
  }
  
  init() {
    if (window.DeviceOrientationEvent) {
      window.addEventListener('deviceorientation', this.handleOrientation.bind(this));
    }
    
    // Ekran yönelim değişikliklerini de dinle
    window.addEventListener('orientationchange', this.handleScreenOrientation.bind(this));
  }
  
  handleOrientation(event) {
    const beta = event.beta;
    const gamma = event.gamma;
    
    // Yönelim belirleme
    if (Math.abs(beta) < this.orientationThreshold) {
      if (Math.abs(gamma) < this.orientationThreshold) {
        this.currentOrientation = 'portrait';
      } else if (gamma > 0) {
        this.currentOrientation = 'landscape-left';
      } else {
        this.currentOrientation = 'landscape-right';
      }
    } else if (beta > 0) {
      this.currentOrientation = 'portrait-upside-down';
    }
    
    this.updateUI();
  }
  
  handleScreenOrientation() {
    const orientation = screen.orientation.angle;
    console.log('Ekran yönelim açısı:', orientation);
    this.updateUI();
  }
  
  updateUI() {
    const body = document.body;
    
    // CSS sınıflarını güncelle
    body.className = body.className.replace(/orientation-\w+/g, '');
    body.classList.add(`orientation-${this.currentOrientation}`);
    
    // Arayüz elementlerini güncelle
    this.adjustLayout();
  }
  
  adjustLayout() {
    const container = document.getElementById('main-container');
    
    if (this.currentOrientation.includes('landscape')) {
      container.style.flexDirection = 'row';
      container.style.height = '100vh';
    } else {
      container.style.flexDirection = 'column';
      container.style.height = 'auto';
    }
  }
}

// AR/VR uygulaması için yönelim takibi
class ARVROrientationTracker {
  constructor() {
    this.orientation = { alpha: 0, beta: 0, gamma: 0 };
    this.isTracking = false;
    this.callbacks = {
      onOrientationUpdate: null,
      onCalibrationComplete: null
    };
  }
  
  startTracking() {
    if (window.DeviceOrientationEvent) {
      // İzin iste
      if (typeof DeviceOrientationEvent.requestPermission === 'function') {
        DeviceOrientationEvent.requestPermission()
          .then(response => {
            if (response === 'granted') {
              this.enableTracking();
            }
          });
      } else {
        this.enableTracking();
      }
    }
  }
  
  enableTracking() {
    window.addEventListener('deviceorientation', this.handleOrientation.bind(this));
    this.isTracking = true;
    console.log('AR/VR yönelim takibi başlatıldı');
  }
  
  handleOrientation(event) {
    this.orientation = {
      alpha: event.alpha,
      beta: event.beta,
      gamma: event.gamma
    };
    
    if (this.callbacks.onOrientationUpdate) {
      this.callbacks.onOrientationUpdate(this.orientation);
    }
  }
  
  onOrientationUpdate(callback) {
    this.callbacks.onOrientationUpdate = callback;
  }
  
  // Kalibrasyon
  calibrate() {
    console.log('Yönelim kalibrasyonu başlatılıyor...');
    
    // 3 saniye bekle ve ortalama al
    setTimeout(() => {
      const avgOrientation = this.getAverageOrientation();
      this.setCalibrationOffset(avgOrientation);
      
      if (this.callbacks.onCalibrationComplete) {
        this.callbacks.onCalibrationComplete(avgOrientation);
      }
    }, 3000);
  }
  
  getAverageOrientation() {
    // Basit ortalama hesaplama
    return {
      alpha: this.orientation.alpha,
      beta: this.orientation.beta,
      gamma: this.orientation.gamma
    };
  }
  
  setCalibrationOffset(offset) {
    this.calibrationOffset = offset;
    console.log('Kalibrasyon tamamlandı:', offset);
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli yönelim yönetimi
class SecureOrientationManager {
  constructor() {
    this.isPermissionGranted = false;
    this.orientationData = null;
    this.privacyMode = false;
  }
  
  async requestPermission() {
    try {
      if (typeof DeviceOrientationEvent.requestPermission === 'function') {
        const response = await DeviceOrientationEvent.requestPermission();
        this.isPermissionGranted = response === 'granted';
        
        if (this.isPermissionGranted) {
          this.startOrientationTracking();
        }
        
        return this.isPermissionGranted;
      } else {
        // Eski tarayıcılar için otomatik izin
        this.isPermissionGranted = true;
        this.startOrientationTracking();
        return true;
      }
    } catch (error) {
      console.error('Yönelim izni hatası:', error);
      return false;
    }
  }
  
  startOrientationTracking() {
    if (this.isPermissionGranted) {
      window.addEventListener('deviceorientation', this.handleOrientation.bind(this));
      console.log('Yönelim takibi başlatıldı');
    }
  }
  
  handleOrientation(event) {
    if (this.privacyMode) {
      return; // Gizlilik modunda veri toplama
    }
    
    this.orientationData = {
      alpha: event.alpha,
      beta: event.beta,
      gamma: event.gamma,
      timestamp: Date.now()
    };
    
    // Veri doğrulama
    if (this.validateOrientationData(this.orientationData)) {
      this.processOrientationData(this.orientationData);
    }
  }
  
  validateOrientationData(data) {
    // Veri doğrulama
    if (data.alpha < 0 || data.alpha > 360) return false;
    if (data.beta < -180 || data.beta > 180) return false;
    if (data.gamma < -90 || data.gamma > 90) return false;
    
    return true;
  }
  
  processOrientationData(data) {
    // Güvenli veri işleme
    console.log('Yönelim verisi işlendi:', data);
  }
  
  enablePrivacyMode() {
    this.privacyMode = true;
    console.log('Gizlilik modu etkinleştirildi');
  }
  
  disablePrivacyMode() {
    this.privacyMode = false;
    console.log('Gizlilik modu devre dışı bırakıldı');
  }
  
  clearOrientationData() {
    this.orientationData = null;
    console.log('Yönelim verisi temizlendi');
  }
}
```

**Alternatifler:**
- **Screen Orientation API**: Ekran yönelim kontrolü
- **Resize Observer API**: Boyut değişiklik algılama
- **Intersection Observer API**: Görünürlük algılama
- **Media Queries**: CSS tabanlı yönelim algılama
- **Touch Events**: Dokunma tabanlı etkileşim

**Ne Zaman Kullanılmalı?**
- ✅ Mobil uygulama geliştirme gerektiğinde
- ✅ Oyun geliştirme gerektiğinde
- ✅ AR/VR uygulamaları geliştirme gerektiğinde
- ✅ Cihaz hareketlerini algılama gerektiğinde
- ✅ Yönelim tabanlı arayüz gerektiğinde
- ✅ Sensör tabanlı özellikler gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Desktop uygulamalarda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Gizlilik öncelikli uygulamalarda
- ❌ Basit web sitelerinde
- ❌ Performans kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Sınırlı Mobil Deneyim**: Mobil cihazlarda yönelim tabanlı özellikler kullanılamaz
- **Oyun Kontrolü Eksikliği**: Cihaz hareketleri oyun kontrolleri olarak kullanılamaz
- **AR/VR Uyumsuzluğu**: Artırılmış gerçeklik uygulamaları geliştirilemez
- **Kullanıcı Deneyimi Sorunları**: Cihaz yönelimine göre arayüz ayarlanamaz
- **Sensör Entegrasyonu Eksikliği**: Cihaz sensörleri web uygulamalarında kullanılamaz

**Modern Alternatifler:**
- **Screen Orientation API**: Ekran yönelim kontrolü
- **Device Motion API**: Hareket algılama
- **WebXR API**: Sanal ve artırılmış gerçeklik
- **WebRTC**: Gerçek zamanlı iletişim
- **WebAssembly**: Yüksek performanslı hesaplama

### Document Object Model (DOM)

**Document Object Model (DOM) Nedir?**
Document Object Model (DOM), HTML ve XML belgelerinin yapısını temsil eden bir programlama arayüzüdür. Web sayfalarının içeriğini, yapısını ve stilini dinamik olarak değiştirmek için kullanılır. DOM, belgeyi bir ağaç yapısı olarak temsil eder ve her element bir düğüm (node) olarak işlenir.

**Neden Kullanılır?**
- **Dinamik İçerik**: Web sayfalarının içeriğini dinamik olarak değiştirme
- **Etkileşim**: Kullanıcı etkileşimlerini yönetme
- **Stil Değişiklikleri**: CSS stillerini dinamik olarak uygulama
- **Form Yönetimi**: Form verilerini işleme ve doğrulama
- **Event Handling**: Olayları (click, hover, vb.) yönetme
- **AJAX Entegrasyonu**: Sunucudan gelen verileri sayfaya entegre etme

**Temel Kullanım Örnekleri:**

```javascript
// Element seçimi
function selectElements() {
  // ID ile seçim
  const elementById = document.getElementById('myId');
  
  // Sınıf ile seçim
  const elementsByClass = document.getElementsByClassName('myClass');
  
  // Etiket ile seçim
  const elementsByTag = document.getElementsByTagName('div');
  
  // CSS seçici ile seçim
  const elementBySelector = document.querySelector('.myClass');
  const elementsBySelectorAll = document.querySelectorAll('.myClass');
  
  console.log('Seçilen elementler:', {
    byId: elementById,
    byClass: elementsByClass,
    byTag: elementsByTag,
    bySelector: elementBySelector,
    bySelectorAll: elementsBySelectorAll
  });
}

// Element oluşturma ve ekleme
function createAndAddElement() {
  // Yeni element oluştur
  const newElement = document.createElement('div');
  newElement.textContent = 'Yeni element';
  newElement.className = 'new-element';
  newElement.id = 'new-element-id';
  
  // Element özelliklerini ayarla
  newElement.setAttribute('data-custom', 'value');
  newElement.style.color = 'blue';
  newElement.style.fontSize = '16px';
  
  // Element'i sayfaya ekle
  document.body.appendChild(newElement);
  
  // Veya belirli bir element'e ekle
  const container = document.getElementById('container');
  if (container) {
    container.appendChild(newElement);
  }
}

// Event listener ekleme
function addEventListeners() {
  const element = document.getElementById('myButton');
  
  if (element) {
    // Click event
    element.addEventListener('click', (event) => {
      console.log('Element tıklandı:', event.target);
    });
    
    // Hover event
    element.addEventListener('mouseenter', () => {
      element.style.backgroundColor = 'lightblue';
    });
    
    element.addEventListener('mouseleave', () => {
      element.style.backgroundColor = '';
    });
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// DOM manipülasyon sınıfı
class DOMManager {
  constructor() {
    this.elements = new Map();
    this.eventListeners = new Map();
  }
  
  // Element seçimi ve cache'leme
  select(selector) {
    if (this.elements.has(selector)) {
      return this.elements.get(selector);
    }
    
    const element = document.querySelector(selector);
    if (element) {
      this.elements.set(selector, element);
    }
    
    return element;
  }
  
  // Çoklu element seçimi
  selectAll(selector) {
    return document.querySelectorAll(selector);
  }
  
  // Element oluşturma
  create(tagName, options = {}) {
    const element = document.createElement(tagName);
    
    // Özellikleri ayarla
    if (options.className) {
      element.className = options.className;
    }
    
    if (options.id) {
      element.id = options.id;
    }
    
    if (options.textContent) {
      element.textContent = options.textContent;
    }
    
    if (options.innerHTML) {
      element.innerHTML = options.innerHTML;
    }
    
    if (options.attributes) {
      Object.entries(options.attributes).forEach(([key, value]) => {
        element.setAttribute(key, value);
      });
    }
    
    if (options.styles) {
      Object.assign(element.style, options.styles);
    }
    
    return element;
  }
  
  // Element ekleme
  append(parentSelector, child) {
    const parent = this.select(parentSelector);
    if (parent) {
      parent.appendChild(child);
    }
  }
  
  // Element kaldırma
  remove(selector) {
    const element = this.select(selector);
    if (element && element.parentNode) {
      element.parentNode.removeChild(element);
      this.elements.delete(selector);
    }
  }
  
  // Event listener ekleme
  on(selector, event, handler) {
    const element = this.select(selector);
    if (element) {
      element.addEventListener(event, handler);
      
      // Event listener'ı kaydet
      const key = `${selector}-${event}`;
      if (!this.eventListeners.has(key)) {
        this.eventListeners.set(key, []);
      }
      this.eventListeners.get(key).push(handler);
    }
  }
  
  // Event listener kaldırma
  off(selector, event, handler) {
    const element = this.select(selector);
    if (element) {
      element.removeEventListener(event, handler);
    }
  }
}

// Form yönetimi
class FormManager {
  constructor(formSelector) {
    this.form = document.querySelector(formSelector);
    this.fields = new Map();
    this.validators = new Map();
  }
  
  // Form alanlarını kaydet
  registerField(name, selector) {
    const field = document.querySelector(selector);
    if (field) {
      this.fields.set(name, field);
    }
  }
  
  // Validator ekle
  addValidator(fieldName, validator) {
    this.validators.set(fieldName, validator);
  }
  
  // Form doğrulama
  validate() {
    let isValid = true;
    const errors = [];
    
    this.validators.forEach((validator, fieldName) => {
      const field = this.fields.get(fieldName);
      if (field) {
        const result = validator(field.value);
        if (!result.isValid) {
          isValid = false;
          errors.push({ field: fieldName, message: result.message });
        }
      }
    });
    
    return { isValid, errors };
  }
  
  // Form verilerini al
  getFormData() {
    const data = {};
    this.fields.forEach((field, name) => {
      data[name] = field.value;
    });
    return data;
  }
  
  // Form'u temizle
  clear() {
    this.fields.forEach(field => {
      field.value = '';
    });
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Dinamik liste yönetimi
class DynamicList {
  constructor(containerSelector) {
    this.container = document.querySelector(containerSelector);
    this.items = [];
  }
  
  addItem(itemData) {
    const item = this.createListItem(itemData);
    this.container.appendChild(item);
    this.items.push(itemData);
  }
  
  createListItem(data) {
    const li = document.createElement('li');
    li.className = 'list-item';
    li.innerHTML = `
      <span class="item-title">${data.title}</span>
      <span class="item-description">${data.description}</span>
      <button class="delete-btn" data-id="${data.id}">Sil</button>
    `;
    
    // Silme butonu event listener
    const deleteBtn = li.querySelector('.delete-btn');
    deleteBtn.addEventListener('click', () => {
      this.removeItem(data.id);
    });
    
    return li;
  }
  
  removeItem(id) {
    const item = this.container.querySelector(`[data-id="${id}"]`);
    if (item) {
      item.parentNode.removeChild(item);
      this.items = this.items.filter(item => item.id !== id);
    }
  }
  
  clear() {
    this.container.innerHTML = '';
    this.items = [];
  }
}

// Modal yönetimi
class ModalManager {
  constructor() {
    this.modals = new Map();
    this.activeModal = null;
  }
  
  createModal(id, options = {}) {
    const modal = document.createElement('div');
    modal.className = 'modal';
    modal.id = id;
    modal.innerHTML = `
      <div class="modal-content">
        <span class="close">&times;</span>
        <div class="modal-header">
          <h2>${options.title || 'Modal'}</h2>
        </div>
        <div class="modal-body">
          ${options.content || ''}
        </div>
        <div class="modal-footer">
          ${options.footer || ''}
        </div>
      </div>
    `;
    
    // Kapatma butonu
    const closeBtn = modal.querySelector('.close');
    closeBtn.addEventListener('click', () => {
      this.closeModal(id);
    });
    
    // Dışarı tıklama ile kapatma
    modal.addEventListener('click', (e) => {
      if (e.target === modal) {
        this.closeModal(id);
      }
    });
    
    this.modals.set(id, modal);
    return modal;
  }
  
  showModal(id) {
    const modal = this.modals.get(id);
    if (modal) {
      document.body.appendChild(modal);
      modal.style.display = 'block';
      this.activeModal = id;
    }
  }
  
  closeModal(id) {
    const modal = this.modals.get(id);
    if (modal) {
      modal.style.display = 'none';
      if (modal.parentNode) {
        modal.parentNode.removeChild(modal);
      }
      this.activeModal = null;
    }
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli DOM manipülasyonu
class SecureDOMManager {
  constructor() {
    this.sanitizer = new DOMSanitizer();
  }
  
  // Güvenli HTML ekleme
  safeInnerHTML(element, html) {
    const sanitizedHTML = this.sanitizer.sanitize(html);
    element.innerHTML = sanitizedHTML;
  }
  
  // XSS koruması
  escapeHTML(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }
  
  // Event listener güvenliği
  safeAddEventListener(element, event, handler) {
    const safeHandler = (e) => {
      try {
        handler(e);
      } catch (error) {
        console.error('Event handler hatası:', error);
      }
    };
    
    element.addEventListener(event, safeHandler);
  }
}

// DOM sanitizer
class DOMSanitizer {
  constructor() {
    this.allowedTags = ['p', 'div', 'span', 'strong', 'em', 'br', 'ul', 'ol', 'li'];
  }
  
  sanitize(html) {
    const temp = document.createElement('div');
    temp.innerHTML = html;
    
    // Tehlikeli tag'ları kaldır
    const dangerousTags = ['script', 'iframe', 'object', 'embed'];
    dangerousTags.forEach(tag => {
      const elements = temp.querySelectorAll(tag);
      elements.forEach(el => el.remove());
    });
    
    return temp.innerHTML;
  }
}
```

**Alternatifler:**
- **Virtual DOM**: React, Vue gibi framework'ler
- **Shadow DOM**: Web Components
- **Template Literals**: String tabanlı HTML oluşturma
- **DocumentFragment**: Performanslı DOM manipülasyonu
- **InnerHTML**: Basit HTML değiştirme

**Ne Zaman Kullanılmalı?**
- ✅ Dinamik içerik gerektiğinde
- ✅ Kullanıcı etkileşimi gerektiğinde
- ✅ Form yönetimi gerektiğinde
- ✅ Event handling gerektiğinde
- ✅ AJAX entegrasyonu gerektiğinde
- ✅ Stil değişiklikleri gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Statik içerik gerektiğinde
- ❌ Performans kritik uygulamalarda
- ❌ Büyük veri setleri gerektiğinde
- ❌ Server-side rendering gerektiğinde
- ❌ Güvenlik kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Statik İçerik**: Web sayfaları statik kalır
- **Etkileşim Eksikliği**: Kullanıcı etkileşimleri yönetilemez
- **Dinamik Özellikler**: Dinamik içerik ve özellikler eklenemez
- **Form Yönetimi**: Form verileri işlenemez
- **Event Handling**: Olaylar yönetilemez

**Modern Alternatifler:**
- **Virtual DOM**: React, Vue, Angular
- **Shadow DOM**: Web Components
- **Template Literals**: Modern JavaScript
- **DocumentFragment**: Performanslı manipülasyon
- **Web Components**: Yeniden kullanılabilir bileşenler

---

## E - Bölümü

### Encoding API

**Encoding API Nedir?**
Encoding API, web uygulamalarında metin kodlama ve çözme işlemlerini gerçekleştirmek için kullanılan modern bir web API'sidir. TextEncoder ve TextDecoder sınıfları ile farklı karakter kodlamaları arasında dönüşüm yapabilir. Özellikle UTF-8, UTF-16 ve diğer karakter kodlamaları ile çalışır.

**Neden Kullanılır?**
- **Metin Kodlama**: Metinleri farklı karakter kodlamalarına dönüştürme
- **Veri Transferi**: Ağ üzerinden veri gönderirken kodlama
- **Dosya İşleme**: Dosya okuma/yazma işlemlerinde kodlama
- **API İletişimi**: Sunucu ile iletişimde veri formatı dönüşümü
- **Blob İşleme**: Binary veri ile metin arasında dönüşüm
- **WebSocket İletişimi**: Gerçek zamanlı iletişimde veri kodlama

**Temel Kullanım Örnekleri:**

```javascript
// Temel UTF-8 kodlama ve çözme
function basicEncoding() {
  // UTF-8 kodlama
  const encoder = new TextEncoder();
  const encoded = encoder.encode('Merhaba dünya');
  console.log('Kodlanmış veri:', encoded);
  
  // UTF-8 çözme
  const decoder = new TextDecoder();
  const decoded = decoder.decode(encoded);
  console.log('Çözülmüş metin:', decoded); // "Merhaba dünya"
}

// Farklı karakter kodlamaları
function differentEncodings() {
  const text = 'Türkçe karakterler: ğüşıöçĞÜŞİÖÇ';
  
  // UTF-8 kodlama
  const utf8Encoder = new TextEncoder();
  const utf8Encoded = utf8Encoder.encode(text);
  console.log('UTF-8 kodlanmış:', utf8Encoded);
  
  // UTF-8 çözme
  const utf8Decoder = new TextDecoder('utf-8');
  const utf8Decoded = utf8Decoder.decode(utf8Encoded);
  console.log('UTF-8 çözülmüş:', utf8Decoded);
  
  // UTF-16 kodlama
  const utf16Decoder = new TextDecoder('utf-16');
  const utf16Encoded = new Uint8Array([255, 254, 84, 0, 252, 0, 114, 0, 107, 0]);
  const utf16Decoded = utf16Decoder.decode(utf16Encoded);
  console.log('UTF-16 çözülmüş:', utf16Decoded);
}

// Hata yönetimi ile kodlama
function encodingWithErrorHandling() {
  const decoder = new TextDecoder('utf-8', {
    fatal: true, // Hatalı karakterlerde hata fırlat
    ignoreBOM: false // BOM karakterini yok sayma
  });
  
  try {
    // Geçerli UTF-8 veri
    const validData = new Uint8Array([72, 101, 108, 108, 111]); // "Hello"
    const decoded = decoder.decode(validData);
    console.log('Başarılı çözme:', decoded);
  } catch (error) {
    console.error('Kodlama hatası:', error);
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Gelişmiş kodlama yöneticisi
class EncodingManager {
  constructor() {
    this.encoders = new Map();
    this.decoders = new Map();
    this.supportedEncodings = ['utf-8', 'utf-16', 'iso-8859-1', 'windows-1252'];
  }
  
  // Encoder al veya oluştur
  getEncoder(encoding = 'utf-8') {
    if (!this.encoders.has(encoding)) {
      this.encoders.set(encoding, new TextEncoder());
    }
    return this.encoders.get(encoding);
  }
  
  // Decoder al veya oluştur
  getDecoder(encoding = 'utf-8') {
    if (!this.decoders.has(encoding)) {
      this.decoders.set(encoding, new TextDecoder(encoding));
    }
    return this.decoders.get(encoding);
  }
  
  // Metin kodlama
  encode(text, encoding = 'utf-8') {
    const encoder = this.getEncoder(encoding);
    return encoder.encode(text);
  }
  
  // Veri çözme
  decode(data, encoding = 'utf-8') {
    const decoder = this.getDecoder(encoding);
    return decoder.decode(data);
  }
  
  // Kodlama tespiti
  detectEncoding(data) {
    // Basit BOM kontrolü
    if (data.length >= 3) {
      if (data[0] === 0xEF && data[1] === 0xBB && data[2] === 0xBF) {
        return 'utf-8';
      }
    }
    
    if (data.length >= 2) {
      if (data[0] === 0xFF && data[1] === 0xFE) {
        return 'utf-16le';
      }
      if (data[0] === 0xFE && data[1] === 0xFF) {
        return 'utf-16be';
      }
    }
    
    return 'utf-8'; // Varsayılan
  }
  
  // Güvenli kodlama
  safeEncode(text, encoding = 'utf-8') {
    try {
      return this.encode(text, encoding);
    } catch (error) {
      console.error('Kodlama hatası:', error);
      return new Uint8Array(0);
    }
  }
  
  // Güvenli çözme
  safeDecode(data, encoding = 'utf-8') {
    try {
      return this.decode(data, encoding);
    } catch (error) {
      console.error('Çözme hatası:', error);
      return '';
    }
  }
}

// Dosya kodlama yöneticisi
class FileEncodingManager {
  constructor() {
    this.encodingManager = new EncodingManager();
  }
  
  // Dosya okuma ve kodlama
  async readFileAsText(file, encoding = 'utf-8') {
    try {
      const arrayBuffer = await file.arrayBuffer();
      const uint8Array = new Uint8Array(arrayBuffer);
      
      // Kodlama tespiti
      const detectedEncoding = this.encodingManager.detectEncoding(uint8Array);
      
      // Metin çözme
      const text = this.encodingManager.decode(uint8Array, detectedEncoding);
      return { text, encoding: detectedEncoding };
    } catch (error) {
      console.error('Dosya okuma hatası:', error);
      return { text: '', encoding: 'utf-8' };
    }
  }
  
  // Metin dosya oluşturma
  createTextFile(text, filename, encoding = 'utf-8') {
    try {
      const encoded = this.encodingManager.encode(text, encoding);
      const blob = new Blob([encoded], { type: 'text/plain' });
      
      // Dosya indirme
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = filename;
      a.click();
      URL.revokeObjectURL(url);
    } catch (error) {
      console.error('Dosya oluşturma hatası:', error);
    }
  }
  
  // Çoklu kodlama desteği
  convertEncoding(text, fromEncoding, toEncoding) {
    try {
      // Önce mevcut kodlamadan çöz
      const decoded = this.encodingManager.decode(
        this.encodingManager.encode(text, fromEncoding),
        fromEncoding
      );
      
      // Yeni kodlamaya dönüştür
      const encoded = this.encodingManager.encode(decoded, toEncoding);
      return this.encodingManager.decode(encoded, toEncoding);
    } catch (error) {
      console.error('Kodlama dönüşüm hatası:', error);
      return text;
    }
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// API veri işleme
class APIDataProcessor {
  constructor() {
    this.encodingManager = new EncodingManager();
  }
  
  // API yanıtını işle
  async processAPIResponse(response) {
    try {
      const arrayBuffer = await response.arrayBuffer();
      const uint8Array = new Uint8Array(arrayBuffer);
      
      // Kodlama tespiti
      const encoding = this.encodingManager.detectEncoding(uint8Array);
      
      // Metin çözme
      const text = this.encodingManager.decode(uint8Array, encoding);
      
      // JSON parse
      const data = JSON.parse(text);
      return { data, encoding };
    } catch (error) {
      console.error('API yanıt işleme hatası:', error);
      return { data: null, encoding: 'utf-8' };
    }
  }
  
  // API isteği gönder
  async sendAPIRequest(url, data, encoding = 'utf-8') {
    try {
      const jsonString = JSON.stringify(data);
      const encoded = this.encodingManager.encode(jsonString, encoding);
      
      const response = await fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json; charset=utf-8',
          'Content-Length': encoded.length.toString()
        },
        body: encoded
      });
      
      return await this.processAPIResponse(response);
    } catch (error) {
      console.error('API istek hatası:', error);
      return { data: null, encoding: 'utf-8' };
    }
  }
}

// WebSocket veri işleme
class WebSocketDataHandler {
  constructor() {
    this.encodingManager = new EncodingManager();
  }
  
  // WebSocket mesaj gönder
  sendMessage(websocket, message, encoding = 'utf-8') {
    try {
      const encoded = this.encodingManager.encode(message, encoding);
      websocket.send(encoded);
    } catch (error) {
      console.error('WebSocket mesaj gönderme hatası:', error);
    }
  }
  
  // WebSocket mesaj al
  handleMessage(event, encoding = 'utf-8') {
    try {
      if (event.data instanceof ArrayBuffer) {
        const uint8Array = new Uint8Array(event.data);
        const message = this.encodingManager.decode(uint8Array, encoding);
        return message;
      } else if (typeof event.data === 'string') {
        return event.data;
      }
    } catch (error) {
      console.error('WebSocket mesaj işleme hatası:', error);
    }
    
    return '';
  }
  
  // Binary veri gönder
  sendBinaryData(websocket, data) {
    try {
      if (typeof data === 'string') {
        const encoded = this.encodingManager.encode(data, 'utf-8');
        websocket.send(encoded);
      } else if (data instanceof Uint8Array) {
        websocket.send(data);
      }
    } catch (error) {
      console.error('Binary veri gönderme hatası:', error);
    }
  }
}

// Form veri işleme
class FormDataProcessor {
  constructor() {
    this.encodingManager = new EncodingManager();
  }
  
  // Form verilerini işle
  processFormData(formData) {
    const processedData = {};
    
    for (const [key, value] of formData.entries()) {
      if (typeof value === 'string') {
        // Metin verilerini güvenli şekilde işle
        processedData[key] = this.encodingManager.safeDecode(
          this.encodingManager.encode(value, 'utf-8'),
          'utf-8'
        );
      } else if (value instanceof File) {
        // Dosya verilerini işle
        processedData[key] = {
          name: value.name,
          size: value.size,
          type: value.type
        };
      }
    }
    
    return processedData;
  }
  
  // Form verilerini kodla
  encodeFormData(data, encoding = 'utf-8') {
    const encodedData = {};
    
    Object.entries(data).forEach(([key, value]) => {
      if (typeof value === 'string') {
        encodedData[key] = this.encodingManager.encode(value, encoding);
      } else {
        encodedData[key] = value;
      }
    });
    
    return encodedData;
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli kodlama yöneticisi
class SecureEncodingManager {
  constructor() {
    this.encodingManager = new EncodingManager();
    this.maxTextLength = 1000000; // 1MB
    this.allowedEncodings = ['utf-8', 'utf-16', 'iso-8859-1'];
  }
  
  // Güvenli metin kodlama
  secureEncode(text, encoding = 'utf-8') {
    // Kodlama kontrolü
    if (!this.allowedEncodings.includes(encoding)) {
      throw new Error(`İzin verilmeyen kodlama: ${encoding}`);
    }
    
    // Metin uzunluk kontrolü
    if (text.length > this.maxTextLength) {
      throw new Error('Metin çok uzun');
    }
    
    // Güvenli kodlama
    return this.encodingManager.safeEncode(text, encoding);
  }
  
  // Güvenli veri çözme
  secureDecode(data, encoding = 'utf-8') {
    // Kodlama kontrolü
    if (!this.allowedEncodings.includes(encoding)) {
      throw new Error(`İzin verilmeyen kodlama: ${encoding}`);
    }
    
    // Veri boyut kontrolü
    if (data.length > this.maxTextLength) {
      throw new Error('Veri çok büyük');
    }
    
    // Güvenli çözme
    return this.encodingManager.safeDecode(data, encoding);
  }
  
  // XSS koruması
  sanitizeText(text) {
    // HTML karakterlerini escape et
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }
  
  // Güvenli HTML çözme
  safeDecodeHTML(encodedHTML, encoding = 'utf-8') {
    const decoded = this.secureDecode(encodedHTML, encoding);
    return this.sanitizeText(decoded);
  }
}
```

**Alternatifler:**
- **btoa/atob**: Base64 kodlama/çözme
- **encodeURIComponent/decodeURIComponent**: URL kodlama
- **escape/unescape**: Eski JavaScript kodlama (deprecated)
- **Buffer**: Node.js buffer API'si
- **iconv-lite**: Node.js karakter kodlama kütüphanesi

**Ne Zaman Kullanılmalı?**
- ✅ Metin kodlama/çözme gerektiğinde
- ✅ Dosya işleme gerektiğinde
- ✅ API iletişimi gerektiğinde
- ✅ WebSocket iletişimi gerektiğinde
- ✅ Çoklu karakter kodlaması gerektiğinde
- ✅ Binary veri ile metin dönüşümü gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit string işlemleri gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Performans kritik uygulamalarda
- ❌ Güvenlik kritik uygulamalarda
- ❌ Büyük veri setleri gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Kodlama Sorunları**: Farklı karakter kodlamaları işlenemez
- **Veri Kaybı**: Karakter kodlama hataları oluşur
- **API Uyumsuzluğu**: Sunucu ile iletişim sorunları
- **Dosya İşleme Eksikliği**: Dosya okuma/yazma sorunları
- **WebSocket Sorunları**: Gerçek zamanlı iletişim sorunları

**Modern Alternatifler:**
- **Streams API**: Büyük veri akışları
- **File API**: Dosya işleme
- **Blob API**: Binary veri işleme
- **ArrayBuffer**: Binary veri manipülasyonu
- **WebAssembly**: Yüksek performanslı veri işleme

### Encrypted Media Extensions (EME) API

**Encrypted Media Extensions (EME) API Nedir?**
Encrypted Media Extensions (EME) API, web uygulamalarında şifrelenmiş medya içeriğini oynatmak için kullanılan modern bir web API'sidir. DRM (Digital Rights Management) korumalı video ve ses içeriklerini güvenli bir şekilde oynatmayı sağlar. Widevine, PlayReady, FairPlay gibi DRM sistemleri ile çalışır.

**Neden Kullanılır?**
- **DRM Korumalı İçerik**: Şifrelenmiş medya içeriklerini oynatma
- **Telif Hakkı Koruması**: İçerik sahiplerinin haklarını koruma
- **Premium İçerik**: Ücretli video/ses içeriklerini güvenli oynatma
- **Streaming Hizmetleri**: Netflix, Amazon Prime gibi platformlar için
- **Güvenli Oynatma**: İçeriğin kopyalanmasını engelleme
- **Lisans Yönetimi**: Medya lisanslarını yönetme

**Temel Kullanım Örnekleri:**

```javascript
// Temel EME kurulumu
async function setupEME() {
  if ('requestMediaKeySystemAccess' in navigator) {
    try {
      const keySystemConfig = {
        initDataTypes: ['cenc'],
        audioCapabilities: [{
          contentType: 'audio/mp4; codecs="mp4a.40.2"'
        }],
        videoCapabilities: [{
          contentType: 'video/mp4; codecs="avc1.42E01E"'
        }]
      };
      
      const keySystemAccess = await navigator.requestMediaKeySystemAccess(
        'com.widevine.alpha', 
        [keySystemConfig]
      );
      
      const mediaKeys = await keySystemAccess.createMediaKeys();
      console.log('EME başarıyla kuruldu:', mediaKeys);
      
      return mediaKeys;
    } catch (error) {
      console.error('EME kurulum hatası:', error);
    }
  } else {
    console.log('EME desteklenmiyor');
  }
}

// Video element ile EME entegrasyonu
async function setupEncryptedVideo() {
  const video = document.getElementById('encrypted-video');
  const mediaKeys = await setupEME();
  
  if (mediaKeys && video) {
    // MediaKeys'i video elementine ata
    await video.setMediaKeys(mediaKeys);
    
    // Lisans sunucusu URL'si
    const licenseServerUrl = 'https://license-server.example.com/';
    
    // Key session oluştur
    const keySession = mediaKeys.createSession();
    
    // Lisans isteği
    keySession.addEventListener('message', async (event) => {
      try {
        const licenseResponse = await fetch(licenseServerUrl, {
          method: 'POST',
          body: event.message
        });
        
        const license = await licenseResponse.arrayBuffer();
        await keySession.update(license);
      } catch (error) {
        console.error('Lisans güncelleme hatası:', error);
      }
    });
    
    // Video oynatma
    video.src = 'encrypted-video.mp4';
    video.play();
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// EME yöneticisi sınıfı
class EMEManager {
  constructor() {
    this.mediaKeys = null;
    this.keySession = null;
    this.licenseServerUrl = null;
    this.isInitialized = false;
  }
  
  // EME başlatma
  async initialize(drmSystem = 'com.widevine.alpha', licenseServerUrl) {
    try {
      this.licenseServerUrl = licenseServerUrl;
      
      const keySystemConfig = {
        initDataTypes: ['cenc', 'keyids'],
        audioCapabilities: [
          { contentType: 'audio/mp4; codecs="mp4a.40.2"' },
          { contentType: 'audio/webm; codecs="opus"' }
        ],
        videoCapabilities: [
          { contentType: 'video/mp4; codecs="avc1.42E01E"' },
          { contentType: 'video/webm; codecs="vp9"' }
        ],
        persistentState: 'required',
        sessionTypes: ['temporary', 'persistent-license']
      };
      
      const keySystemAccess = await navigator.requestMediaKeySystemAccess(
        drmSystem, 
        [keySystemConfig]
      );
      
      this.mediaKeys = await keySystemAccess.createMediaKeys();
      this.isInitialized = true;
      
      console.log('EME başarıyla başlatıldı');
      return true;
    } catch (error) {
      console.error('EME başlatma hatası:', error);
      return false;
    }
  }
  
  // Video element için EME kurulumu
  async setupVideo(videoElement) {
    if (!this.isInitialized || !this.mediaKeys) {
      throw new Error('EME başlatılmamış');
    }
    
    try {
      await videoElement.setMediaKeys(this.mediaKeys);
      this.keySession = this.mediaKeys.createSession();
      
      // Event listener'ları ekle
      this.setupEventListeners();
      
      console.log('Video EME ile yapılandırıldı');
      return true;
    } catch (error) {
      console.error('Video EME kurulum hatası:', error);
      return false;
    }
  }
  
  // Event listener'ları kurma
  setupEventListeners() {
    if (!this.keySession) return;
    
    // Lisans mesajı
    this.keySession.addEventListener('message', (event) => {
      this.handleLicenseMessage(event);
    });
    
    // Lisans güncellendi
    this.keySession.addEventListener('keystatuseschange', (event) => {
      this.handleKeyStatusChange(event);
    });
    
    // Hata yönetimi
    this.keySession.addEventListener('error', (event) => {
      console.error('Key session hatası:', event);
    });
  }
  
  // Lisans mesajını işleme
  async handleLicenseMessage(event) {
    try {
      if (!this.licenseServerUrl) {
        throw new Error('Lisans sunucusu URL\'si belirtilmemiş');
      }
      
      const response = await fetch(this.licenseServerUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/octet-stream'
        },
        body: event.message
      });
      
      if (!response.ok) {
        throw new Error(`Lisans sunucusu hatası: ${response.status}`);
      }
      
      const license = await response.arrayBuffer();
      await this.keySession.update(license);
      
      console.log('Lisans başarıyla güncellendi');
    } catch (error) {
      console.error('Lisans güncelleme hatası:', error);
    }
  }
  
  // Anahtar durumu değişikliği
  handleKeyStatusChange(event) {
    const keyStatuses = event.target.keyStatuses;
    
    for (const [keyId, status] of keyStatuses.entries()) {
      console.log(`Anahtar ${keyId} durumu: ${status}`);
      
      switch (status) {
        case 'usable':
          console.log('Anahtar kullanılabilir');
          break;
        case 'expired':
          console.log('Anahtar süresi dolmuş');
          break;
        case 'output-restricted':
          console.log('Çıktı kısıtlı');
          break;
        case 'output-downscaled':
          console.log('Çıktı düşük kalite');
          break;
        case 'status-pending':
          console.log('Anahtar durumu bekleniyor');
          break;
        case 'internal-error':
          console.log('İç hata');
          break;
      }
    }
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Streaming uygulaması için EME
class StreamingApp {
  constructor() {
    this.emeManager = new EMEManager();
    this.currentVideo = null;
    this.licenseServerUrl = 'https://license.example.com/';
  }
  
  // Uygulama başlatma
  async initialize() {
    const success = await this.emeManager.initialize(
      'com.widevine.alpha',
      this.licenseServerUrl
    );
    
    if (success) {
      console.log('Streaming uygulaması başlatıldı');
      this.setupVideoPlayer();
    } else {
      console.error('Streaming uygulaması başlatılamadı');
    }
  }
  
  // Video oynatıcı kurulumu
  setupVideoPlayer() {
    const video = document.getElementById('video-player');
    
    if (video) {
      this.currentVideo = video;
      
      // EME kurulumu
      this.emeManager.setupVideo(video);
      
      // Video event listener'ları
      video.addEventListener('encrypted', (event) => {
        this.handleEncryptedEvent(event);
      });
      
      video.addEventListener('error', (event) => {
        this.handleVideoError(event);
      });
    }
  }
  
  // Şifrelenmiş içerik olayı
  async handleEncryptedEvent(event) {
    try {
      console.log('Şifrelenmiş içerik algılandı:', event.initData);
      
      // Key session oluştur
      const keySession = this.emeManager.mediaKeys.createSession();
      
      // Lisans isteği
      await keySession.generateRequest(event.initDataType, event.initData);
      
      console.log('Lisans isteği gönderildi');
    } catch (error) {
      console.error('Şifrelenmiş içerik işleme hatası:', error);
    }
  }
  
  // Video hata yönetimi
  handleVideoError(event) {
    const error = event.target.error;
    
    if (error) {
      switch (error.code) {
        case MediaError.MEDIA_ERR_ABORTED:
          console.log('Video oynatma iptal edildi');
          break;
        case MediaError.MEDIA_ERR_NETWORK:
          console.log('Ağ hatası');
          break;
        case MediaError.MEDIA_ERR_DECODE:
          console.log('Dekodlama hatası');
          break;
        case MediaError.MEDIA_ERR_SRC_NOT_SUPPORTED:
          console.log('Desteklenmeyen format');
          break;
        default:
          console.log('Bilinmeyen hata');
      }
    }
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli EME yöneticisi
class SecureEMEManager {
  constructor() {
    this.emeManager = new EMEManager();
    this.securityLevel = 'high';
    this.allowedDRMSystems = ['com.widevine.alpha'];
    this.licenseCache = new Map();
  }
  
  // Güvenli EME başlatma
  async secureInitialize(drmSystem, licenseServerUrl) {
    // DRM sistemi kontrolü
    if (!this.allowedDRMSystems.includes(drmSystem)) {
      throw new Error(`İzin verilmeyen DRM sistemi: ${drmSystem}`);
    }
    
    // Lisans sunucusu güvenlik kontrolü
    if (!this.isSecureLicenseServer(licenseServerUrl)) {
      throw new Error('Güvenli olmayan lisans sunucusu');
    }
    
    return await this.emeManager.initialize(drmSystem, licenseServerUrl);
  }
  
  // Lisans sunucusu güvenlik kontrolü
  isSecureLicenseServer(url) {
    try {
      const urlObj = new URL(url);
      
      // HTTPS kontrolü
      if (urlObj.protocol !== 'https:') {
        return false;
      }
      
      // Güvenli domain kontrolü
      const secureDomains = ['license.example.com', 'drm.example.com'];
      return secureDomains.includes(urlObj.hostname);
    } catch (error) {
      return false;
    }
  }
  
  // Güvenlik loglama
  logSecurityEvent(event, details) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      event: event,
      details: details,
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    console.log('Güvenlik olayı:', logEntry);
  }
}
```

**Alternatifler:**
- **HLS.js**: HTTP Live Streaming
- **Dash.js**: Dynamic Adaptive Streaming
- **Video.js**: Video oynatıcı kütüphanesi
- **Shaka Player**: Google'ın medya oynatıcısı
- **JW Player**: Ticari video oynatıcı

**Ne Zaman Kullanılmalı?**
- ✅ DRM korumalı içerik gerektiğinde
- ✅ Premium video/ses içerikleri gerektiğinde
- ✅ Streaming hizmetleri geliştirme gerektiğinde
- ✅ Telif hakkı koruması gerektiğinde
- ✅ Güvenli medya oynatma gerektiğinde
- ✅ Lisans yönetimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Açık kaynak içerik gerektiğinde
- ❌ Basit video oynatma gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Performans kritik uygulamalarda
- ❌ Gizlilik öncelikli uygulamalarda

**Kullanılmazsa Ne Olur?**
- **DRM Koruması Eksikliği**: Şifrelenmiş içerik oynatılamaz
- **Telif Hakkı Sorunları**: İçerik koruması sağlanamaz
- **Premium İçerik Eksikliği**: Ücretli içerikler sunulamaz
- **Streaming Sınırlılığı**: Profesyonel streaming hizmetleri geliştirilemez
- **Lisans Yönetimi Eksikliği**: Medya lisansları yönetilemez

**Modern Alternatifler:**
- **WebRTC**: Gerçek zamanlı medya iletişimi
- **Media Source Extensions**: Gelişmiş medya oynatma
- **Web Audio API**: Ses işleme
- **WebCodecs API**: Video/audio kodlama
- **WebAssembly**: Yüksek performanslı medya işleme

---

## F - Bölümü

### Fetch API

**Fetch API Nedir?**
Fetch API, web uygulamalarında ağ istekleri yapmak için kullanılan modern bir web API'sidir. XMLHttpRequest'in yerini alan, Promise tabanlı bir API'dir. HTTP istekleri göndermek ve yanıtları almak için kullanılır. Daha temiz, daha güçlü ve daha esnek bir arayüz sunar.

**Neden Kullanılır?**
- **HTTP İstekleri**: GET, POST, PUT, DELETE gibi HTTP metodları
- **Promise Tabanlı**: Modern JavaScript async/await ile uyumlu
- **Stream Desteği**: Büyük veri akışlarını işleme
- **Request/Response Kontrolü**: Detaylı istek ve yanıt yönetimi
- **CORS Desteği**: Cross-origin istekleri
- **Modern Tarayıcı Desteği**: Tüm modern tarayıcılarda desteklenir

**Temel Kullanım Örnekleri:**

```javascript
// Temel GET isteği
async function basicGetRequest() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    console.log('Veri alındı:', data);
    return data;
  } catch (error) {
    console.error('GET isteği hatası:', error);
  }
}

// POST isteği ile veri gönderme
async function postData() {
  try {
    const postData = {
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    };
    
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer your-token-here'
      },
      body: JSON.stringify(postData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const result = await response.json();
    console.log('Kullanıcı oluşturuldu:', result);
    return result;
  } catch (error) {
    console.error('POST isteği hatası:', error);
  }
}

// Farklı veri formatları
async function differentDataFormats() {
  // JSON veri
  const jsonResponse = await fetch('/api/json-data');
  const jsonData = await jsonResponse.json();
  
  // Text veri
  const textResponse = await fetch('/api/text-data');
  const textData = await textResponse.text();
  
  // Blob veri (resim, dosya)
  const blobResponse = await fetch('/api/image.jpg');
  const blobData = await blobResponse.blob();
  
  // ArrayBuffer veri
  const bufferResponse = await fetch('/api/binary-data');
  const bufferData = await bufferResponse.arrayBuffer();
  
  console.log('Farklı formatlar:', {
    json: jsonData,
    text: textData,
    blob: blobData,
    buffer: bufferData
  });
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Fetch yöneticisi sınıfı
class FetchManager {
  constructor(baseURL = '', defaultHeaders = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...defaultHeaders
    };
    this.timeout = 10000; // 10 saniye
  }
  
  // Temel istek metodu
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      timeout: this.timeout,
      headers: { ...this.defaultHeaders, ...options.headers },
      ...options
    };
    
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), this.timeout);
      
      const response = await fetch(url, {
        ...config,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return response;
    } catch (error) {
      if (error.name === 'AbortError') {
        throw new Error('İstek zaman aşımına uğradı');
      }
      throw error;
    }
  }
  
  // GET isteği
  async get(endpoint, options = {}) {
    const response = await this.request(endpoint, {
      method: 'GET',
      ...options
    });
    return response.json();
  }
  
  // POST isteği
  async post(endpoint, data, options = {}) {
    const response = await this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
      ...options
    });
    return response.json();
  }
  
  // PUT isteği
  async put(endpoint, data, options = {}) {
    const response = await this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
      ...options
    });
    return response.json();
  }
  
  // DELETE isteği
  async delete(endpoint, options = {}) {
    const response = await this.request(endpoint, {
      method: 'DELETE',
      ...options
    });
    return response.json();
  }
  
  // Dosya yükleme
  async uploadFile(endpoint, file, options = {}) {
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await this.request(endpoint, {
      method: 'POST',
      body: formData,
      headers: {
        // Content-Type'ı FormData otomatik ayarlar
        ...this.defaultHeaders,
        ...options.headers
      },
      ...options
    });
    
    return response.json();
  }
  
  // Çoklu istek
  async all(requests) {
    try {
      const responses = await Promise.all(requests);
      return responses;
    } catch (error) {
      console.error('Çoklu istek hatası:', error);
      throw error;
    }
  }
}

// Retry mekanizması ile Fetch
class RetryFetch {
  constructor(maxRetries = 3, delay = 1000) {
    this.maxRetries = maxRetries;
    this.delay = delay;
  }
  
  async fetchWithRetry(url, options = {}) {
    let lastError;
    
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        const response = await fetch(url, options);
        
        if (!response.ok && response.status >= 500) {
          throw new Error(`Server error: ${response.status}`);
        }
        
        return response;
      } catch (error) {
        lastError = error;
        console.log(`İstek ${attempt}/${this.maxRetries} başarısız:`, error.message);
        
        if (attempt < this.maxRetries) {
          await this.delay(attempt * this.delay);
        }
      }
    }
    
    throw lastError;
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// API servis sınıfı
class APIService {
  constructor() {
    this.fetchManager = new FetchManager('https://api.example.com', {
      'Authorization': `Bearer ${this.getToken()}`
    });
  }
  
  getToken() {
    return localStorage.getItem('authToken') || '';
  }
  
  // Kullanıcı işlemleri
  async getUsers() {
    return await this.fetchManager.get('/users');
  }
  
  async getUser(id) {
    return await this.fetchManager.get(`/users/${id}`);
  }
  
  async createUser(userData) {
    return await this.fetchManager.post('/users', userData);
  }
  
  async updateUser(id, userData) {
    return await this.fetchManager.put(`/users/${id}`, userData);
  }
  
  async deleteUser(id) {
    return await this.fetchManager.delete(`/users/${id}`);
  }
  
  // Dosya işlemleri
  async uploadAvatar(userId, file) {
    return await this.fetchManager.uploadFile(`/users/${userId}/avatar`, file);
  }
  
  // Çoklu veri çekme
  async getUserDashboard(userId) {
    const requests = [
      this.fetchManager.get(`/users/${userId}`),
      this.fetchManager.get(`/users/${userId}/posts`),
      this.fetchManager.get(`/users/${userId}/followers`)
    ];
    
    const [user, posts, followers] = await this.fetchManager.all(requests);
    
    return {
      user,
      posts,
      followers
    };
  }
}

// Real-time veri güncelleme
class RealTimeDataManager {
  constructor(apiService) {
    this.apiService = apiService;
    this.pollingInterval = 5000; // 5 saniye
    this.isPolling = false;
    this.pollingId = null;
  }
  
  startPolling(endpoint, callback) {
    if (this.isPolling) {
      this.stopPolling();
    }
    
    this.isPolling = true;
    this.pollingId = setInterval(async () => {
      try {
        const data = await this.apiService.fetchManager.get(endpoint);
        callback(data);
      } catch (error) {
        console.error('Polling hatası:', error);
      }
    }, this.pollingInterval);
  }
  
  stopPolling() {
    if (this.pollingId) {
      clearInterval(this.pollingId);
      this.pollingId = null;
      this.isPolling = false;
    }
  }
}

// Offline desteği ile Fetch
class OfflineFetchManager {
  constructor() {
    this.cache = new Map();
    this.isOnline = navigator.onLine;
    this.setupOnlineOfflineListeners();
  }
  
  setupOnlineOfflineListeners() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.processQueuedRequests();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }
  
  async fetchWithOfflineSupport(url, options = {}) {
    if (this.isOnline) {
      try {
        const response = await fetch(url, options);
        
        // Başarılı istekleri cache'le
        if (response.ok) {
          this.cache.set(url, {
            data: await response.clone().json(),
            timestamp: Date.now()
          });
        }
        
        return response;
      } catch (error) {
        // Ağ hatası durumunda cache'den döndür
        return this.getFromCache(url);
      }
    } else {
      // Offline durumda cache'den döndür
      return this.getFromCache(url);
    }
  }
  
  getFromCache(url) {
    const cached = this.cache.get(url);
    
    if (cached && Date.now() - cached.timestamp < 300000) { // 5 dakika
      return new Response(JSON.stringify(cached.data), {
        status: 200,
        headers: { 'Content-Type': 'application/json' }
      });
    }
    
    throw new Error('Offline ve cache\'de veri yok');
  }
  
  processQueuedRequests() {
    // Offline sırasında kuyruğa alınan istekleri işle
    console.log('Çevrimiçi duruma geçildi, kuyruktaki istekler işleniyor...');
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli Fetch yöneticisi
class SecureFetchManager {
  constructor() {
    this.csrfToken = this.getCSRFToken();
    this.allowedOrigins = ['https://api.example.com', 'https://secure.example.com'];
  }
  
  getCSRFToken() {
    return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content') || '';
  }
  
  // Güvenli istek
  async secureRequest(url, options = {}) {
    // URL güvenlik kontrolü
    if (!this.isAllowedOrigin(url)) {
      throw new Error('İzin verilmeyen origin');
    }
    
    // CSRF token ekle
    const secureOptions = {
      ...options,
      headers: {
        'X-CSRF-Token': this.csrfToken,
        'X-Requested-With': 'XMLHttpRequest',
        ...options.headers
      }
    };
    
    try {
      const response = await fetch(url, secureOptions);
      
      // Güvenlik header'larını kontrol et
      this.validateSecurityHeaders(response);
      
      return response;
    } catch (error) {
      this.logSecurityEvent('fetch_error', { url, error: error.message });
      throw error;
    }
  }
  
  isAllowedOrigin(url) {
    try {
      const urlObj = new URL(url);
      return this.allowedOrigins.includes(urlObj.origin);
    } catch (error) {
      return false;
    }
  }
  
  validateSecurityHeaders(response) {
    const requiredHeaders = [
      'X-Content-Type-Options',
      'X-Frame-Options',
      'X-XSS-Protection'
    ];
    
    const missingHeaders = requiredHeaders.filter(header => 
      !response.headers.has(header)
    );
    
    if (missingHeaders.length > 0) {
      console.warn('Eksik güvenlik header\'ları:', missingHeaders);
    }
  }
  
  logSecurityEvent(event, details) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      event,
      details,
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    console.log('Güvenlik olayı:', logEntry);
  }
  
  // Rate limiting
  async rateLimitedRequest(url, options = {}, maxRequests = 10, windowMs = 60000) {
    const key = `rate_limit_${url}`;
    const now = Date.now();
    const requests = JSON.parse(localStorage.getItem(key) || '[]');
    
    // Eski istekleri temizle
    const recentRequests = requests.filter(time => now - time < windowMs);
    
    if (recentRequests.length >= maxRequests) {
      throw new Error('Rate limit aşıldı');
    }
    
    // İsteği kaydet
    recentRequests.push(now);
    localStorage.setItem(key, JSON.stringify(recentRequests));
    
    return await this.secureRequest(url, options);
  }
}
```

**Alternatifler:**
- **XMLHttpRequest**: Eski tarayıcı desteği
- **Axios**: Popüler HTTP kütüphanesi
- **jQuery.ajax**: jQuery tabanlı istekler
- **SuperAgent**: Minimalist HTTP kütüphanesi
- **Ky**: Küçük ve modern HTTP kütüphanesi

**Ne Zaman Kullanılmalı?**
- ✅ Modern tarayıcı desteği gerektiğinde
- ✅ Promise tabanlı kod gerektiğinde
- ✅ HTTP istekleri gerektiğinde
- ✅ RESTful API iletişimi gerektiğinde
- ✅ Stream veri işleme gerektiğinde
- ✅ CORS istekleri gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ IE11 desteği gerektiğinde
- ❌ Karmaşık HTTP özellikleri gerektiğinde
- ❌ Upload progress gerektiğinde
- ❌ Timeout kontrolü gerektiğinde

**Kullanılmazsa Ne Olur?**
- **HTTP İstekleri Eksikliği**: Ağ istekleri yapılamaz
- **API İletişimi Sorunları**: Sunucu ile iletişim kurulamaz
- **Veri Çekme Eksikliği**: Uzak veriler alınamaz
- **Modern JavaScript Uyumsuzluğu**: Promise tabanlı kod yazılamaz
- **CORS Sorunları**: Cross-origin istekler yapılamaz

**Modern Alternatifler:**
- **Axios**: Gelişmiş HTTP kütüphanesi
- **Ky**: Minimalist Fetch wrapper
- **SuperAgent**: Esnek HTTP kütüphanesi
- **WebSocket**: Gerçek zamanlı iletişim
- **Server-Sent Events**: Tek yönlü gerçek zamanlı iletişim

### File API

**File API Nedir?**
File API, web uygulamalarında dosya işlemleri yapmak için kullanılan modern bir web API'sidir. Kullanıcıların dosya seçmesini, dosyaları okumasını, dosya bilgilerini almasını ve dosya içeriğini işlemesini sağlar. FileReader, File, Blob gibi sınıflar ile çalışır.

**Neden Kullanılır?**
- **Dosya Seçimi**: Kullanıcıların dosya seçmesini sağlama
- **Dosya Okuma**: Dosya içeriğini okuma ve işleme
- **Dosya Bilgileri**: Dosya adı, boyutu, türü gibi bilgileri alma
- **Dosya Yükleme**: Dosyaları sunucuya yükleme
- **Dosya Önizleme**: Resim, video gibi dosyaları önizleme
- **Dosya Validasyonu**: Dosya türü ve boyut kontrolü

**Temel Kullanım Örnekleri:**

```javascript
// Dosya seçimi ve bilgi alma
function handleFileSelection() {
  const fileInput = document.getElementById('fileInput');
  
  fileInput.addEventListener('change', (event) => {
    const files = event.target.files;
    
    for (let i = 0; i < files.length; i++) {
      const file = files[i];
      console.log('Dosya bilgileri:', {
        name: file.name,
        size: file.size,
        type: file.type,
        lastModified: new Date(file.lastModified)
      });
      
      // Dosya işleme
      processFile(file);
    }
  });
}

// Dosya okuma
function readFile(file) {
  const reader = new FileReader();
  
  // Text dosyası okuma
  reader.onload = (event) => {
    console.log('Dosya içeriği:', event.target.result);
  };
  
  reader.onerror = (event) => {
    console.error('Dosya okuma hatası:', event.target.error);
  };
  
  // Dosya türüne göre okuma yöntemi
  if (file.type.startsWith('text/')) {
    reader.readAsText(file);
  } else if (file.type.startsWith('image/')) {
    reader.readAsDataURL(file);
  } else {
    reader.readAsArrayBuffer(file);
  }
}

// Dosya önizleme
function previewFile(file) {
  if (file.type.startsWith('image/')) {
    const reader = new FileReader();
    reader.onload = (event) => {
      const img = document.createElement('img');
      img.src = event.target.result;
      img.style.maxWidth = '300px';
      document.getElementById('preview').appendChild(img);
    };
    reader.readAsDataURL(file);
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Dosya yöneticisi sınıfı
class FileManager {
  constructor() {
    this.maxFileSize = 10 * 1024 * 1024; // 10MB
    this.allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'text/plain', 'application/pdf'];
    this.files = new Map();
  }
  
  // Dosya validasyonu
  validateFile(file) {
    const errors = [];
    
    // Boyut kontrolü
    if (file.size > this.maxFileSize) {
      errors.push(`Dosya çok büyük. Maksimum ${this.maxFileSize / 1024 / 1024}MB olmalı.`);
    }
    
    // Tür kontrolü
    if (!this.allowedTypes.includes(file.type)) {
      errors.push(`Desteklenmeyen dosya türü: ${file.type}`);
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
  
  // Dosya ekleme
  addFile(file) {
    const validation = this.validateFile(file);
    
    if (!validation.isValid) {
      console.error('Dosya validasyon hatası:', validation.errors);
      return false;
    }
    
    const fileId = this.generateFileId();
    this.files.set(fileId, {
      file: file,
      id: fileId,
      name: file.name,
      size: file.size,
      type: file.type,
      addedAt: new Date()
    });
    
    console.log('Dosya eklendi:', fileId);
    return fileId;
  }
  
  // Dosya ID oluşturma
  generateFileId() {
    return 'file_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Dosya okuma
  async readFile(fileId, readType = 'text') {
    const fileData = this.files.get(fileId);
    
    if (!fileData) {
      throw new Error('Dosya bulunamadı');
    }
    
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      
      reader.onload = (event) => {
        resolve(event.target.result);
      };
      
      reader.onerror = (event) => {
        reject(event.target.error);
      };
      
      switch (readType) {
        case 'text':
          reader.readAsText(fileData.file);
          break;
        case 'dataURL':
          reader.readAsDataURL(fileData.file);
          break;
        case 'arrayBuffer':
          reader.readAsArrayBuffer(fileData.file);
          break;
        case 'binaryString':
          reader.readAsBinaryString(fileData.file);
          break;
        default:
          reject(new Error('Geçersiz okuma türü'));
      }
    });
  }
  
  // Dosya silme
  removeFile(fileId) {
    if (this.files.has(fileId)) {
      this.files.delete(fileId);
      console.log('Dosya silindi:', fileId);
      return true;
    }
    return false;
  }
  
  // Tüm dosyaları listele
  listFiles() {
    return Array.from(this.files.values());
  }
  
  // Dosya boyutunu formatla
  formatFileSize(bytes) {
    if (bytes === 0) return '0 Bytes';
    
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Resim işleme
class ImageProcessor {
  constructor() {
    this.fileManager = new FileManager();
    this.maxWidth = 1920;
    this.maxHeight = 1080;
    this.quality = 0.8;
  }
  
  // Resim boyutlandırma
  async resizeImage(fileId, width, height) {
    const fileData = this.fileManager.files.get(fileId);
    
    if (!fileData || !fileData.type.startsWith('image/')) {
      throw new Error('Geçerli bir resim dosyası değil');
    }
    
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      
      reader.onload = (event) => {
        const img = new Image();
        
        img.onload = () => {
          const canvas = document.createElement('canvas');
          const ctx = canvas.getContext('2d');
          
          // Boyutları hesapla
          const { newWidth, newHeight } = this.calculateDimensions(
            img.width, img.height, width, height
          );
          
          canvas.width = newWidth;
          canvas.height = newHeight;
          
          // Resmi çiz
          ctx.drawImage(img, 0, 0, newWidth, newHeight);
          
          // Canvas'ı blob'a dönüştür
          canvas.toBlob((blob) => {
            resolve(blob);
          }, fileData.type, this.quality);
        };
        
        img.onerror = () => {
          reject(new Error('Resim yüklenemedi'));
        };
        
        img.src = event.target.result;
      };
      
      reader.onerror = () => {
        reject(new Error('Dosya okunamadı'));
      };
      
      reader.readAsDataURL(fileData.file);
    });
  }
  
  // Boyut hesaplama
  calculateDimensions(originalWidth, originalHeight, maxWidth, maxHeight) {
    let newWidth = originalWidth;
    let newHeight = originalHeight;
    
    if (originalWidth > maxWidth) {
      newWidth = maxWidth;
      newHeight = (originalHeight * maxWidth) / originalWidth;
    }
    
    if (newHeight > maxHeight) {
      newHeight = maxHeight;
      newWidth = (originalWidth * maxHeight) / originalHeight;
    }
    
    return { newWidth, newHeight };
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli dosya yöneticisi
class SecureFileManager {
  constructor() {
    this.fileManager = new FileManager();
    this.virusScanEnabled = true;
    this.encryptionEnabled = false;
  }
  
  // Güvenli dosya ekleme
  async addSecureFile(file) {
    // Dosya validasyonu
    const validation = this.fileManager.validateFile(file);
    if (!validation.isValid) {
      throw new Error(`Dosya güvenlik hatası: ${validation.errors.join(', ')}`);
    }
    
    // Virüs taraması (simülasyon)
    if (this.virusScanEnabled) {
      const isClean = await this.scanForVirus(file);
      if (!isClean) {
        throw new Error('Dosya güvenlik riski içeriyor');
      }
    }
    
    return this.fileManager.addFile(file);
  }
  
  // Virüs taraması (simülasyon)
  async scanForVirus(file) {
    // Gerçek uygulamada virüs tarama servisi kullanın
    return new Promise((resolve) => {
      setTimeout(() => {
        // Basit dosya türü kontrolü
        const suspiciousTypes = ['application/x-executable', 'application/x-msdownload'];
        resolve(!suspiciousTypes.includes(file.type));
      }, 1000);
    });
  }
}
```

**Alternatifler:**
- **FormData**: Dosya yükleme için
- **Blob API**: Binary veri işleme
- **ArrayBuffer**: Binary veri manipülasyonu
- **Canvas API**: Resim işleme
- **Web Workers**: Büyük dosya işleme

**Ne Zaman Kullanılmalı?**
- ✅ Dosya yükleme gerektiğinde
- ✅ Dosya okuma gerektiğinde
- ✅ Dosya önizleme gerektiğinde
- ✅ Dosya validasyonu gerektiğinde
- ✅ Dosya işleme gerektiğinde
- ✅ Kullanıcı dosya seçimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Büyük dosya işleme gerektiğinde
- ❌ Server-side dosya işleme gerektiğinde
- ❌ Güvenlik kritik uygulamalarda
- ❌ Performans kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Dosya Yükleme Eksikliği**: Kullanıcılar dosya yükleyemez
- **Dosya İşleme Eksikliği**: Dosya içeriği işlenemez
- **Dosya Önizleme Eksikliği**: Dosyalar önizlenemez
- **Dosya Validasyonu Eksikliği**: Dosya güvenliği sağlanamaz
- **Kullanıcı Deneyimi Sorunları**: Dosya işlemleri yapılamaz

**Modern Alternatifler:**
- **File System Access API**: Yerel dosya sistemi erişimi
- **Web Streams API**: Büyük dosya akışları
- **Web Workers**: Arka planda dosya işleme
- **IndexedDB**: Dosya depolama
- **WebAssembly**: Yüksek performanslı dosya işleme

### File System Access API

**File System Access API Nedir?**
File System Access API, web uygulamalarının kullanıcının yerel dosya sistemine doğrudan erişmesini sağlayan modern bir web API'sidir. Kullanıcıların dosya seçmesini, dosya okumasını, dosya yazmasını ve dosya sistemi ile etkileşim kurmasını sağlar.

**Neden Kullanılır?**
- **Yerel Dosya Erişimi**: Kullanıcının dosya sistemine doğrudan erişim
- **Dosya Okuma/Yazma**: Dosyaları okuma ve yazma işlemleri
- **Dosya Seçimi**: Kullanıcı dostu dosya seçim arayüzü
- **Dosya Yönetimi**: Dosya oluşturma, silme, taşıma işlemleri
- **Dizin Erişimi**: Klasör yapısına erişim
- **Güvenli Erişim**: Kullanıcı izni ile güvenli dosya erişimi

**Temel Kullanım Örnekleri:**

```javascript
// Dosya seçme ve okuma
async function openAndReadFile() {
  try {
    if (!('showOpenFilePicker' in window)) {
      throw new Error('File System Access API desteklenmiyor');
    }
    
    const [fileHandle] = await window.showOpenFilePicker({
      types: [
        {
          description: 'Text dosyaları',
          accept: {
            'text/plain': ['.txt', '.md', '.js', '.html', '.css']
          }
        }
      ],
      excludeAcceptAllOption: false,
      multiple: false
    });
    
    const file = await fileHandle.getFile();
    const contents = await file.text();
    
    console.log('Dosya adı:', file.name);
    console.log('Dosya boyutu:', file.size);
    console.log('Dosya içeriği:', contents);
    
    return { fileHandle, contents };
  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('Kullanıcı dosya seçimini iptal etti');
    } else {
      console.error('Dosya okuma hatası:', error);
    }
  }
}

// Dosya kaydetme
async function saveFile(content, suggestedName = 'untitled.txt') {
  try {
    if (!('showSaveFilePicker' in window)) {
      throw new Error('File System Access API desteklenmiyor');
    }
    
    const fileHandle = await window.showSaveFilePicker({
      suggestedName: suggestedName,
      types: [
        {
          description: 'Text dosyaları',
          accept: {
            'text/plain': ['.txt']
          }
        }
      ]
    });
    
    const writable = await fileHandle.createWritable();
    await writable.write(content);
    await writable.close();
    
    console.log('Dosya kaydedildi:', fileHandle.name);
    return fileHandle;
  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('Kullanıcı kaydetme işlemini iptal etti');
    } else {
      console.error('Dosya kaydetme hatası:', error);
    }
  }
}

// Çoklu dosya seçimi
async function selectMultipleFiles() {
  try {
    const fileHandles = await window.showOpenFilePicker({
      types: [
        {
          description: 'Resim dosyaları',
          accept: {
            'image/*': ['.png', '.jpg', '.jpeg', '.gif', '.webp']
          }
        }
      ],
      multiple: true
    });
    
    const files = await Promise.all(
      fileHandles.map(async (fileHandle) => {
        const file = await fileHandle.getFile();
        return {
          handle: fileHandle,
          file: file,
          name: file.name,
          size: file.size,
          type: file.type
        };
      })
    );
    
    console.log('Seçilen dosyalar:', files);
    return files;
  } catch (error) {
    console.error('Çoklu dosya seçimi hatası:', error);
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Dosya sistemi yöneticisi
class FileSystemManager {
  constructor() {
    this.fileHandles = new Map();
    this.directoryHandles = new Map();
    this.isSupported = 'showOpenFilePicker' in window;
  }
  
  // Dosya açma
  async openFile(options = {}) {
    if (!this.isSupported) {
      throw new Error('File System Access API desteklenmiyor');
    }
    
    const defaultOptions = {
      types: [
        {
          description: 'Tüm dosyalar',
          accept: {
            '*/*': []
          }
        }
      ],
      excludeAcceptAllOption: false,
      multiple: false
    };
    
    const config = { ...defaultOptions, ...options };
    const [fileHandle] = await window.showOpenFilePicker(config);
    
    const file = await fileHandle.getFile();
    const fileId = this.generateFileId();
    
    this.fileHandles.set(fileId, {
      handle: fileHandle,
      file: file,
      id: fileId,
      name: file.name,
      size: file.size,
      type: file.type,
      lastModified: file.lastModified
    });
    
    return { fileId, fileHandle, file };
  }
  
  // Dosya kaydetme
  async saveFile(content, options = {}) {
    if (!this.isSupported) {
      throw new Error('File System Access API desteklenmiyor');
    }
    
    const defaultOptions = {
      suggestedName: 'untitled.txt',
      types: [
        {
          description: 'Text dosyaları',
          accept: {
            'text/plain': ['.txt']
          }
        }
      ]
    };
    
    const config = { ...defaultOptions, ...options };
    const fileHandle = await window.showSaveFilePicker(config);
    
    const writable = await fileHandle.createWritable();
    await writable.write(content);
    await writable.close();
    
    const fileId = this.generateFileId();
    this.fileHandles.set(fileId, {
      handle: fileHandle,
      id: fileId,
      name: fileHandle.name,
      content: content
    });
    
    return { fileId, fileHandle };
  }
  
  // Dosya ID oluşturma
  generateFileId() {
    return 'file_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
}
```

**Alternatifler:**
- **File API**: Geleneksel dosya işleme
- **IndexedDB**: Dosya depolama
- **Web Storage**: Basit veri depolama
- **Blob API**: Binary veri işleme
- **Download API**: Dosya indirme

**Ne Zaman Kullanılmalı?**
- ✅ Yerel dosya sistemi erişimi gerektiğinde
- ✅ Dosya editörü uygulamaları gerektiğinde
- ✅ Dosya yöneticisi uygulamaları gerektiğinde
- ✅ Modern tarayıcı desteği gerektiğinde
- ✅ Güvenli dosya işleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik kritik uygulamalarda
- ❌ Server-side dosya işleme gerektiğinde
- ❌ Basit dosya yükleme gerektiğinde
- ❌ Cross-origin dosya erişimi gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Yerel Dosya Erişimi Eksikliği**: Kullanıcının dosya sistemine erişim sağlanamaz
- **Dosya Editörü Sınırlılığı**: Gelişmiş dosya editörü uygulamaları geliştirilemez
- **Dosya Yönetimi Eksikliği**: Dosya yöneticisi uygulamaları geliştirilemez
- **Kullanıcı Deneyimi Sorunları**: Geleneksel dosya seçimi kullanılır

**Modern Alternatifler:**
- **File API**: Geleneksel dosya işleme
- **Web Streams API**: Büyük dosya akışları
- **Web Workers**: Arka planda dosya işleme
- **IndexedDB**: Dosya depolama
- **WebAssembly**: Yüksek performanslı dosya işleme

### Fullscreen API

**Fullscreen API Nedir?**
Fullscreen API, web uygulamalarının tam ekran moduna geçmesini ve tam ekran modundan çıkmasını sağlayan modern bir web API'sidir. Kullanıcı deneyimini iyileştirmek için video oynatma, oyun, sunum gibi uygulamalarda kullanılır. Tarayıcı arayüzünü gizleyerek tam ekran deneyim sunar.

**Neden Kullanılır?**
- **Tam Ekran Deneyim**: Kullanıcıya tam ekran deneyim sunma
- **Video Oynatma**: Video oynatıcılarda tam ekran modu
- **Oyun Geliştirme**: Oyunlarda immersive deneyim
- **Sunum Uygulamaları**: Sunumlarda dikkat dağıtıcı unsurları gizleme
- **Medya İçeriği**: Resim, video galerilerinde tam ekran görüntüleme
- **Kullanıcı Deneyimi**: Daha iyi kullanıcı deneyimi sağlama

**Temel Kullanım Örnekleri:**

```javascript
// Tam ekran moduna geç
async function enterFullscreen() {
  try {
    if (document.documentElement.requestFullscreen) {
      await document.documentElement.requestFullscreen();
      console.log('Tam ekran moduna geçildi');
    } else {
      console.log('Fullscreen API desteklenmiyor');
    }
  } catch (error) {
    console.error('Tam ekran hatası:', error);
  }
}

// Tam ekran modundan çık
function exitFullscreen() {
  if (document.fullscreenElement) {
    document.exitFullscreen();
    console.log('Tam ekran modundan çıkıldı');
  }
}

// Tam ekran durumu kontrolü
function checkFullscreenStatus() {
  if (document.fullscreenElement) {
    console.log('Şu anda tam ekran modunda');
    return true;
  } else {
    console.log('Tam ekran modunda değil');
    return false;
  }
}

// Tam ekran değişikliklerini dinle
function setupFullscreenListeners() {
  document.addEventListener('fullscreenchange', () => {
    if (document.fullscreenElement) {
      console.log('Tam ekran moduna geçildi');
      document.body.classList.add('fullscreen');
    } else {
      console.log('Tam ekran modundan çıkıldı');
      document.body.classList.remove('fullscreen');
    }
  });
  
  document.addEventListener('fullscreenerror', (event) => {
    console.error('Tam ekran hatası:', event);
  });
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Tam ekran yöneticisi
class FullscreenManager {
  constructor() {
    this.isSupported = this.checkSupport();
    this.isFullscreen = false;
    this.callbacks = {
      onEnter: null,
      onExit: null,
      onError: null
    };
    
    this.setupEventListeners();
  }
  
  // Destek kontrolü
  checkSupport() {
    return !!(
      document.documentElement.requestFullscreen ||
      document.documentElement.webkitRequestFullscreen ||
      document.documentElement.mozRequestFullScreen ||
      document.documentElement.msRequestFullscreen
    );
  }
  
  // Tam ekran moduna geç
  async enterFullscreen(element = document.documentElement) {
    if (!this.isSupported) {
      throw new Error('Fullscreen API desteklenmiyor');
    }
    
    try {
      if (element.requestFullscreen) {
        await element.requestFullscreen();
      } else if (element.webkitRequestFullscreen) {
        await element.webkitRequestFullscreen();
      } else if (element.mozRequestFullScreen) {
        await element.mozRequestFullScreen();
      } else if (element.msRequestFullscreen) {
        await element.msRequestFullscreen();
      }
      
      this.isFullscreen = true;
      
      if (this.callbacks.onEnter) {
        this.callbacks.onEnter();
      }
      
      console.log('Tam ekran moduna geçildi');
    } catch (error) {
      console.error('Tam ekran hatası:', error);
      
      if (this.callbacks.onError) {
        this.callbacks.onError(error);
      }
      
      throw error;
    }
  }
  
  // Tam ekran modundan çık
  async exitFullscreen() {
    if (!this.isFullscreen) {
      return;
    }
    
    try {
      if (document.exitFullscreen) {
        await document.exitFullscreen();
      } else if (document.webkitExitFullscreen) {
        await document.webkitExitFullscreen();
      } else if (document.mozCancelFullScreen) {
        await document.mozCancelFullScreen();
      } else if (document.msExitFullscreen) {
        await document.msExitFullscreen();
      }
      
      this.isFullscreen = false;
      
      if (this.callbacks.onExit) {
        this.callbacks.onExit();
      }
      
      console.log('Tam ekran modundan çıkıldı');
    } catch (error) {
      console.error('Tam ekran çıkış hatası:', error);
      throw error;
    }
  }
  
  // Tam ekran durumunu değiştir
  async toggleFullscreen(element = document.documentElement) {
    if (this.isFullscreen) {
      await this.exitFullscreen();
    } else {
      await this.enterFullscreen(element);
    }
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    document.addEventListener('fullscreenchange', () => {
      this.isFullscreen = !!document.fullscreenElement;
      
      if (this.isFullscreen) {
        if (this.callbacks.onEnter) {
          this.callbacks.onEnter();
        }
      } else {
        if (this.callbacks.onExit) {
          this.callbacks.onExit();
        }
      }
    });
    
    document.addEventListener('fullscreenerror', (event) => {
      console.error('Tam ekran hatası:', event);
      
      if (this.callbacks.onError) {
        this.callbacks.onError(event);
      }
    });
  }
  
  // Callback'leri ayarla
  onEnter(callback) {
    this.callbacks.onEnter = callback;
  }
  
  onExit(callback) {
    this.callbacks.onExit = callback;
  }
  
  onError(callback) {
    this.callbacks.onError = callback;
  }
}
```

**Alternatifler:**
- **CSS Fullscreen**: CSS ile tam ekran simülasyonu
- **Video Fullscreen**: Video element'inin kendi tam ekran özelliği
- **Canvas Fullscreen**: Canvas element'i için özel tam ekran
- **WebGL Fullscreen**: WebGL uygulamaları için tam ekran
- **Mobile Fullscreen**: Mobil cihazlar için özel tam ekran

**Ne Zaman Kullanılmalı?**
- ✅ Video oynatma gerektiğinde
- ✅ Oyun geliştirme gerektiğinde
- ✅ Sunum uygulamaları gerektiğinde
- ✅ Medya galerileri gerektiğinde
- ✅ Immersive deneyim gerektiğinde
- ✅ Dikkat dağıtıcı unsurları gizleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik kritik uygulamalarda
- ❌ Mobil uygulamalarda
- ❌ Basit web sitelerinde
- ❌ Kullanıcı deneyimi bozulacağında

**Kullanılmazsa Ne Olur?**
- **Tam Ekran Deneyimi Eksikliği**: Kullanıcılar tam ekran deneyim yaşayamaz
- **Video Oynatma Sınırlılığı**: Video oynatıcılarda tam ekran özelliği olmaz
- **Oyun Deneyimi Sınırlılığı**: Oyunlarda immersive deneyim sağlanamaz
- **Sunum Sınırlılığı**: Sunumlarda dikkat dağıtıcı unsurlar gizlenemez
- **Kullanıcı Deneyimi Sorunları**: Daha az engaging deneyim sunulur

**Modern Alternatifler:**
- **Picture-in-Picture API**: Video için küçük pencere
- **WebXR API**: Sanal ve artırılmış gerçeklik
- **WebGL**: 3D grafik uygulamaları
- **Web Audio API**: Ses tabanlı uygulamalar
- **WebAssembly**: Yüksek performanslı uygulamalar

---

## G - Bölümü

### Gamepad API

**Gamepad API Nedir?**
Gamepad API, web uygulamalarının oyun kumandaları (gamepad) ile etkileşim kurmasını sağlayan modern bir web API'sidir. Kullanıcıların oyun kumandalarını web oyunlarında, simülasyonlarda ve interaktif uygulamalarda kullanmasını sağlar. Buton basma, analog stick hareketleri ve titreşim gibi özellikleri destekler.

**Neden Kullanılır?**
- **Oyun Geliştirme**: Web oyunlarında oyun kumandası desteği
- **Simülasyon Uygulamaları**: Uçuş simülatörleri, araç simülatörleri
- **Interaktif Uygulamalar**: Oyun kumandası ile kontrol edilebilen uygulamalar
- **Erişilebilirlik**: Alternatif giriş yöntemi sağlama
- **Kullanıcı Deneyimi**: Daha iyi oyun deneyimi sunma
- **Çoklu Giriş Desteği**: Klavye ve fareye alternatif

**Temel Kullanım Örnekleri:**

```javascript
// Oyun kumandası bağlantı olaylarını dinle
window.addEventListener('gamepadconnected', event => {
  const gamepad = event.gamepad;
  console.log(`Oyun kumandası bağlandı: ${gamepad.id}`);
  console.log(`Buton sayısı: ${gamepad.buttons.length}`);
  console.log(`Analog stick sayısı: ${gamepad.axes.length}`);
});

window.addEventListener('gamepaddisconnected', event => {
  const gamepad = event.gamepad;
  console.log(`Oyun kumandası bağlantısı kesildi: ${gamepad.id}`);
});

// Oyun kumandası durumunu kontrol et
function checkGamepad() {
  const gamepads = navigator.getGamepads();
  
  for (const gamepad of gamepads) {
    if (gamepad) {
      // Buton durumlarını kontrol et
      gamepad.buttons.forEach((button, index) => {
        if (button.pressed) {
          console.log(`Buton ${index} basıldı`);
        }
      });
      
      // Analog stick durumlarını kontrol et
      gamepad.axes.forEach((axis, index) => {
        if (Math.abs(axis) > 0.1) {
          console.log(`Analog stick ${index}: ${axis.toFixed(2)}`);
        }
      });
    }
  }
  
  requestAnimationFrame(checkGamepad);
}

// Oyun kumandası kontrolünü başlat
function startGamepadControl() {
  if (navigator.getGamepads) {
    checkGamepad();
  } else {
    console.log('Gamepad API desteklenmiyor');
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Oyun kumandası yöneticisi
class GamepadManager {
  constructor() {
    this.gamepads = new Map();
    this.isSupported = 'getGamepads' in navigator;
    this.callbacks = {
      onConnect: null,
      onDisconnect: null,
      onButtonPress: null,
      onAxisMove: null
    };
    
    this.setupEventListeners();
    this.startPolling();
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    if (!this.isSupported) return;
    
    window.addEventListener('gamepadconnected', (event) => {
      const gamepad = event.gamepad;
      this.gamepads.set(gamepad.index, {
        gamepad: gamepad,
        connected: true,
        lastButtonStates: new Array(gamepad.buttons.length).fill(false),
        lastAxisStates: new Array(gamepad.axes.length).fill(0)
      });
      
      console.log(`Oyun kumandası bağlandı: ${gamepad.id}`);
      
      if (this.callbacks.onConnect) {
        this.callbacks.onConnect(gamepad);
      }
    });
    
    window.addEventListener('gamepaddisconnected', (event) => {
      const gamepad = event.gamepad;
      this.gamepads.delete(gamepad.index);
      
      console.log(`Oyun kumandası bağlantısı kesildi: ${gamepad.id}`);
      
      if (this.callbacks.onDisconnect) {
        this.callbacks.onDisconnect(gamepad);
      }
    });
  }
  
  // Oyun kumandası durumunu kontrol et
  startPolling() {
    if (!this.isSupported) return;
    
    const pollGamepads = () => {
      const gamepads = navigator.getGamepads();
      
      for (const gamepad of gamepads) {
        if (gamepad && this.gamepads.has(gamepad.index)) {
          this.updateGamepadState(gamepad);
        }
      }
      
      requestAnimationFrame(pollGamepads);
    };
    
    pollGamepads();
  }
  
  // Oyun kumandası durumunu güncelle
  updateGamepadState(gamepad) {
    const gamepadData = this.gamepads.get(gamepad.index);
    
    // Buton durumlarını kontrol et
    gamepad.buttons.forEach((button, index) => {
      const wasPressed = gamepadData.lastButtonStates[index];
      const isPressed = button.pressed;
      
      if (isPressed && !wasPressed) {
        console.log(`Buton ${index} basıldı`);
        
        if (this.callbacks.onButtonPress) {
          this.callbacks.onButtonPress(gamepad.index, index, button);
        }
      }
      
      gamepadData.lastButtonStates[index] = isPressed;
    });
    
    // Analog stick durumlarını kontrol et
    gamepad.axes.forEach((axis, index) => {
      const lastValue = gamepadData.lastAxisStates[index];
      const currentValue = axis;
      
      if (Math.abs(currentValue - lastValue) > 0.1) {
        console.log(`Analog stick ${index}: ${currentValue.toFixed(2)}`);
        
        if (this.callbacks.onAxisMove) {
          this.callbacks.onAxisMove(gamepad.index, index, currentValue);
        }
      }
      
      gamepadData.lastAxisStates[index] = currentValue;
    });
  }
  
  // Callback'leri ayarla
  onConnect(callback) {
    this.callbacks.onConnect = callback;
  }
  
  onDisconnect(callback) {
    this.callbacks.onDisconnect = callback;
  }
  
  onButtonPress(callback) {
    this.callbacks.onButtonPress = callback;
  }
  
  onAxisMove(callback) {
    this.callbacks.onAxisMove = callback;
  }
  
  // Bağlı oyun kumandalarını listele
  getConnectedGamepads() {
    return Array.from(this.gamepads.values());
  }
  
  // Oyun kumandası bilgilerini al
  getGamepadInfo(index) {
    const gamepadData = this.gamepads.get(index);
    return gamepadData ? gamepadData.gamepad : null;
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Oyun kumandası ile karakter kontrolü
class CharacterController {
  constructor(character) {
    this.character = character;
    this.gamepadManager = new GamepadManager();
    this.setupGamepadControls();
  }
  
  setupGamepadControls() {
    // Buton basma olayları
    this.gamepadManager.onButtonPress((gamepadIndex, buttonIndex, button) => {
      switch (buttonIndex) {
        case 0: // A butonu
          this.character.jump();
          break;
        case 1: // B butonu
          this.character.attack();
          break;
        case 2: // X butonu
          this.character.special();
          break;
        case 3: // Y butonu
          this.character.interact();
          break;
      }
    });
    
    // Analog stick hareketleri
    this.gamepadManager.onAxisMove((gamepadIndex, axisIndex, value) => {
      switch (axisIndex) {
        case 0: // Sol analog stick X ekseni
          this.character.moveHorizontal(value);
          break;
        case 1: // Sol analog stick Y ekseni
          this.character.moveVertical(value);
          break;
        case 2: // Sağ analog stick X ekseni
          this.character.lookHorizontal(value);
          break;
        case 3: // Sağ analog stick Y ekseni
          this.character.lookVertical(value);
          break;
      }
    });
  }
}

// Oyun kumandası ile araç kontrolü
class VehicleController {
  constructor(vehicle) {
    this.vehicle = vehicle;
    this.gamepadManager = new GamepadManager();
    this.setupVehicleControls();
  }
  
  setupVehicleControls() {
    // Buton kontrolleri
    this.gamepadManager.onButtonPress((gamepadIndex, buttonIndex, button) => {
      switch (buttonIndex) {
        case 0: // A butonu - Fren
          this.vehicle.brake();
          break;
        case 1: // B butonu - El freni
          this.vehicle.handbrake();
          break;
        case 2: // X butonu - Vites değiştir
          this.vehicle.shiftGear();
          break;
        case 3: // Y butonu - Korna
          this.vehicle.horn();
          break;
      }
    });
    
    // Analog stick kontrolleri
    this.gamepadManager.onAxisMove((gamepadIndex, axisIndex, value) => {
      switch (axisIndex) {
        case 0: // Sol analog stick - Direksiyon
          this.vehicle.steer(value);
          break;
        case 1: // Sol analog stick - Gaz/Fren
          if (value > 0) {
            this.vehicle.accelerate(value);
          } else {
            this.vehicle.brake(-value);
          }
          break;
        case 2: // Sağ analog stick - Kamera
          this.vehicle.rotateCamera(value);
          break;
        case 3: // Sağ analog stick - Kamera
          this.vehicle.tiltCamera(value);
          break;
      }
    });
  }
}

// Oyun kumandası ile menü kontrolü
class MenuController {
  constructor(menu) {
    this.menu = menu;
    this.gamepadManager = new GamepadManager();
    this.currentSelection = 0;
    this.setupMenuControls();
  }
  
  setupMenuControls() {
    // Buton kontrolleri
    this.gamepadManager.onButtonPress((gamepadIndex, buttonIndex, button) => {
      switch (buttonIndex) {
        case 0: // A butonu - Seç
          this.menu.select(this.currentSelection);
          break;
        case 1: // B butonu - Geri
          this.menu.goBack();
          break;
        case 2: // X butonu - Yardım
          this.menu.showHelp();
          break;
        case 3: // Y butonu - Ayarlar
          this.menu.showSettings();
          break;
      }
    });
    
    // Analog stick kontrolleri
    this.gamepadManager.onAxisMove((gamepadIndex, axisIndex, value) => {
      if (axisIndex === 1 && Math.abs(value) > 0.5) { // Sol analog stick Y ekseni
        if (value > 0.5) {
          this.menu.moveUp();
        } else if (value < -0.5) {
          this.menu.moveDown();
        }
      }
    });
  }
}
```

**Güvenlik ve Best Practices:**

```javascript
// Güvenli oyun kumandası yöneticisi
class SecureGamepadManager {
  constructor() {
    this.gamepadManager = new GamepadManager();
    this.allowedButtons = [0, 1, 2, 3]; // Sadece temel butonlar
    this.allowedAxes = [0, 1]; // Sadece temel analog stick'ler
    this.isEnabled = false;
  }
  
  // Güvenli buton kontrolü
  onButtonPress(callback) {
    this.gamepadManager.onButtonPress((gamepadIndex, buttonIndex, button) => {
      if (this.isEnabled && this.allowedButtons.includes(buttonIndex)) {
        callback(gamepadIndex, buttonIndex, button);
      }
    });
  }
  
  // Güvenli analog stick kontrolü
  onAxisMove(callback) {
    this.gamepadManager.onAxisMove((gamepadIndex, axisIndex, value) => {
      if (this.isEnabled && this.allowedAxes.includes(axisIndex)) {
        callback(gamepadIndex, axisIndex, value);
      }
    });
  }
  
  // Oyun kumandası kontrolünü etkinleştir
  enable() {
    this.isEnabled = true;
  }
  
  // Oyun kumandası kontrolünü devre dışı bırak
  disable() {
    this.isEnabled = false;
  }
  
  // İzin verilen butonları ayarla
  setAllowedButtons(buttons) {
    this.allowedButtons = buttons;
  }
  
  // İzin verilen analog stick'leri ayarla
  setAllowedAxes(axes) {
    this.allowedAxes = axes;
  }
}
```

**Alternatifler:**
- **Keyboard API**: Klavye girişi
- **Mouse API**: Fare girişi
- **Touch Events**: Dokunmatik giriş
- **WebXR API**: Sanal gerçeklik kontrolleri
- **Web Bluetooth API**: Bluetooth cihazları

**Ne Zaman Kullanılmalı?**
- ✅ Web oyunları geliştirirken
- ✅ Simülasyon uygulamaları gerektiğinde
- ✅ Interaktif uygulamalar gerektiğinde
- ✅ Erişilebilirlik gerektiğinde
- ✅ Çoklu giriş yöntemi gerektiğinde
- ✅ Oyun deneyimi iyileştirme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit web sitelerinde
- ❌ Mobil uygulamalarda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik kritik uygulamalarda
- ❌ Oyun kumandası olmayan cihazlarda

**Kullanılmazsa Ne Olur?**
- **Oyun Deneyimi Sınırlılığı**: Web oyunlarında oyun kumandası desteği olmaz
- **Simülasyon Sınırlılığı**: Simülasyon uygulamalarında kontrol sınırlılığı
- **Erişilebilirlik Eksikliği**: Alternatif giriş yöntemi sağlanamaz
- **Kullanıcı Deneyimi Sorunları**: Daha az engaging deneyim sunulur
- **Çoklu Giriş Eksikliği**: Sadece klavye ve fare kullanılır

**Modern Alternatifler:**
- **WebXR API**: Sanal ve artırılmış gerçeklik
- **Web Bluetooth API**: Bluetooth cihazları
- **WebUSB API**: USB cihazları
- **WebHID API**: HID cihazları
- **Web Serial API**: Seri port cihazları

### Geolocation API

**Geolocation API Nedir?**
Geolocation API, web uygulamalarının kullanıcının coğrafi konumunu almasını sağlayan modern bir web API'sidir. GPS, Wi-Fi, hücresel ağ ve IP adresi gibi çeşitli yöntemlerle konum bilgisi sağlar. Harita uygulamaları, konum tabanlı servisler ve kişiselleştirilmiş içerik sunumu için kullanılır.

**Neden Kullanılır?**
- **Konum Tabanlı Servisler**: Kullanıcının konumuna göre içerik sunma
- **Harita Uygulamaları**: Navigasyon ve harita görüntüleme
- **Sosyal Medya**: Konum paylaşımı ve check-in özellikleri
- **E-ticaret**: Yakındaki mağazaları bulma
- **Hava Durumu**: Yerel hava durumu bilgisi
- **Güvenlik**: Konum tabanlı güvenlik önlemleri

**Temel Kullanım Örnekleri:**

```javascript
// Konum bilgisini al
if ('geolocation' in navigator) {
  navigator.geolocation.getCurrentPosition(
    position => {
      console.log(`Enlem: ${position.coords.latitude}`);
      console.log(`Boylam: ${position.coords.longitude}`);
      console.log(`Doğruluk: ${position.coords.accuracy}`);
      console.log(`Yükseklik: ${position.coords.altitude}`);
      console.log(`Hız: ${position.coords.speed}`);
      console.log(`Yön: ${position.coords.heading}`);
    },
    error => {
      switch (error.code) {
        case error.PERMISSION_DENIED:
          console.error('Konum erişimi reddedildi');
          break;
        case error.POSITION_UNAVAILABLE:
          console.error('Konum bilgisi mevcut değil');
          break;
        case error.TIMEOUT:
          console.error('Konum alma zaman aşımı');
          break;
      }
    },
    {
      enableHighAccuracy: true,
      timeout: 10000,
      maximumAge: 60000
    }
  );
}

// Konum değişikliklerini izle
const watchId = navigator.geolocation.watchPosition(
  position => {
    console.log('Yeni konum:', position.coords);
    updateMap(position.coords);
  },
  error => {
    console.error('Konum izleme hatası:', error);
  },
  {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 30000
  }
);

// Konum izlemeyi durdur
function stopWatching() {
  navigator.geolocation.clearWatch(watchId);
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Konum yöneticisi
class LocationManager {
  constructor() {
    this.isSupported = 'geolocation' in navigator;
    this.currentPosition = null;
    this.watchId = null;
    this.callbacks = {
      onLocationUpdate: null,
      onError: null,
      onPermissionChange: null
    };
    
    this.setupPermissionListener();
  }
  
  // Konum izni kontrolü
  async checkPermission() {
    if (!this.isSupported) {
      throw new Error('Geolocation API desteklenmiyor');
    }
    
    try {
      const permission = await navigator.permissions.query({ name: 'geolocation' });
      return permission.state;
    } catch (error) {
      console.warn('İzin kontrolü yapılamadı:', error);
      return 'unknown';
    }
  }
  
  // Konum bilgisini al
  async getCurrentLocation(options = {}) {
    if (!this.isSupported) {
      throw new Error('Geolocation API desteklenmiyor');
    }
    
    const defaultOptions = {
      enableHighAccuracy: true,
      timeout: 10000,
      maximumAge: 60000
    };
    
    const config = { ...defaultOptions, ...options };
    
    return new Promise((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.currentPosition = position;
          resolve(position);
        },
        (error) => {
          if (this.callbacks.onError) {
            this.callbacks.onError(error);
          }
          reject(error);
        },
        config
      );
    });
  }
  
  // Konum değişikliklerini izle
  startWatching(options = {}) {
    if (!this.isSupported) {
      throw new Error('Geolocation API desteklenmiyor');
    }
    
    const defaultOptions = {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 30000
    };
    
    const config = { ...defaultOptions, ...options };
    
    this.watchId = navigator.geolocation.watchPosition(
      (position) => {
        this.currentPosition = position;
        
        if (this.callbacks.onLocationUpdate) {
          this.callbacks.onLocationUpdate(position);
        }
      },
      (error) => {
        if (this.callbacks.onError) {
          this.callbacks.onError(error);
        }
      },
      config
    );
    
    return this.watchId;
  }
  
  // Konum izlemeyi durdur
  stopWatching() {
    if (this.watchId) {
      navigator.geolocation.clearWatch(this.watchId);
      this.watchId = null;
    }
  }
  
  // İki konum arasındaki mesafeyi hesapla
  calculateDistance(lat1, lon1, lat2, lon2) {
    const R = 6371; // Dünya yarıçapı (km)
    const dLat = this.toRadians(lat2 - lat1);
    const dLon = this.toRadians(lon2 - lon1);
    
    const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
              Math.cos(this.toRadians(lat1)) * Math.cos(this.toRadians(lat2)) *
              Math.sin(dLon / 2) * Math.sin(dLon / 2);
    
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const distance = R * c;
    
    return distance;
  }
  
  // Dereceyi radyana çevir
  toRadians(degrees) {
    return degrees * (Math.PI / 180);
  }
}
```

**Alternatifler:**
- **IP Geolocation**: IP adresi ile konum tespiti
- **Wi-Fi Geolocation**: Wi-Fi ağları ile konum tespiti
- **Cell Tower Geolocation**: Hücresel ağ ile konum tespiti
- **Bluetooth Geolocation**: Bluetooth cihazları ile konum tespiti
- **Manual Location**: Kullanıcı tarafından manuel konum girişi

**Ne Zaman Kullanılmalı?**
- ✅ Konum tabanlı servisler gerektiğinde
- ✅ Harita uygulamaları gerektiğinde
- ✅ Navigasyon gerektiğinde
- ✅ Sosyal medya uygulamaları gerektiğinde
- ✅ E-ticaret uygulamaları gerektiğinde
- ✅ Güvenlik uygulamaları gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Gizlilik kritik uygulamalarda
- ❌ Konum gerektirmeyen uygulamalarda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik riski oluşturacağında
- ❌ Kullanıcı izni olmadığında

**Kullanılmazsa Ne Olur?**
- **Konum Tabanlı Servisler Eksikliği**: Kullanıcının konumuna göre içerik sunulamaz
- **Harita Uygulamaları Sınırlılığı**: Navigasyon ve harita özellikleri sınırlı olur
- **Sosyal Medya Sınırlılığı**: Konum paylaşımı ve check-in özellikleri olmaz
- **E-ticaret Sınırlılığı**: Yakındaki mağazalar bulunamaz
- **Kişiselleştirme Eksikliği**: Konum tabanlı kişiselleştirme sağlanamaz

**Modern Alternatifler:**
- **Web Bluetooth API**: Bluetooth cihazları ile konum
- **WebUSB API**: USB cihazları ile konum
- **WebHID API**: HID cihazları ile konum
- **Web Serial API**: Seri port cihazları ile konum
- **WebXR API**: Sanal gerçeklik ile konum

### Geometry Interfaces

**Geometry Interfaces Nedir?**
Geometry Interfaces, web uygulamalarında geometrik hesaplamalar ve DOM elementlerinin konum, boyut ve sınır bilgilerini almak için kullanılan modern web API'leridir. DOMRect, DOMRectReadOnly, DOMPoint, DOMQuad gibi arayüzlerle 2D geometrik işlemler yapılmasını sağlar. Element konumlandırma, animasyonlar ve layout hesaplamaları için kullanılır.

**Neden Kullanılır?**
- **Element Konumlandırma**: DOM elementlerinin konum ve boyut bilgilerini alma
- **Layout Hesaplamaları**: Sayfa düzeni ve element yerleşimi hesaplama
- **Animasyonlar**: Element animasyonları için geometrik veriler
- **Collision Detection**: Element çarpışma tespiti
- **Responsive Design**: Responsive tasarım hesaplamaları
- **Performance Optimization**: Layout performansı optimizasyonu

**Temel Kullanım Örnekleri:**

```javascript
// DOMRect oluştur
const rect = new DOMRect(10, 20, 100, 50);
console.log(`X: ${rect.x}, Y: ${rect.y}`);
console.log(`Genişlik: ${rect.width}, Yükseklik: ${rect.height}`);
console.log(`Sol üst köşe: ${rect.left}, ${rect.top}`);
console.log(`Sağ alt köşe: ${rect.right}, ${rect.bottom}`);

// Element'in sınırlarını al
const element = document.getElementById('myElement');
const bounds = element.getBoundingClientRect();
console.log('Element sınırları:', bounds);

// DOMPoint oluştur
const point = new DOMPoint(50, 75);
console.log(`Nokta: (${point.x}, ${point.y})`);

// DOMQuad oluştur
const quad = new DOMQuad(
  new DOMPoint(0, 0),    // p1
  new DOMPoint(100, 0),  // p2
  new DOMPoint(100, 50), // p3
  new DOMPoint(0, 50)    // p4
);

console.log('Dörtgen köşeleri:', quad.p1, quad.p2, quad.p3, quad.p4);
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Geometri yöneticisi
class GeometryManager {
  constructor() {
    this.rects = new Map();
    this.points = new Map();
    this.quads = new Map();
  }
  
  // Element geometrisini al
  getElementGeometry(element) {
    const rect = element.getBoundingClientRect();
    const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
    const scrollLeft = window.pageXOffset || document.documentElement.scrollLeft;
    
    const geometry = {
      clientRect: rect,
      absoluteRect: new DOMRect(
        rect.left + scrollLeft,
        rect.top + scrollTop,
        rect.width,
        rect.height
      ),
      center: new DOMPoint(
        rect.left + rect.width / 2,
        rect.top + rect.height / 2
      ),
      corners: {
        topLeft: new DOMPoint(rect.left, rect.top),
        topRight: new DOMPoint(rect.right, rect.top),
        bottomLeft: new DOMPoint(rect.left, rect.bottom),
        bottomRight: new DOMPoint(rect.right, rect.bottom)
      }
    };
    
    return geometry;
  }
  
  // İki rect arasındaki mesafeyi hesapla
  calculateDistance(rect1, rect2) {
    const center1 = new DOMPoint(
      rect1.left + rect1.width / 2,
      rect1.top + rect1.height / 2
    );
    
    const center2 = new DOMPoint(
      rect2.left + rect2.width / 2,
      rect2.top + rect2.height / 2
    );
    
    const dx = center2.x - center1.x;
    const dy = center2.y - center1.y;
    
    return Math.sqrt(dx * dx + dy * dy);
  }
  
  // Rect'lerin kesişip kesişmediğini kontrol et
  isIntersecting(rect1, rect2) {
    return !(rect1.right < rect2.left || 
             rect2.right < rect1.left || 
             rect1.bottom < rect2.top || 
             rect2.bottom < rect1.top);
  }
  
  // Rect'lerin birleşimini hesapla
  union(rect1, rect2) {
    const left = Math.min(rect1.left, rect2.left);
    const top = Math.min(rect1.top, rect2.top);
    const right = Math.max(rect1.right, rect2.right);
    const bottom = Math.max(rect1.bottom, rect2.bottom);
    
    return new DOMRect(left, top, right - left, bottom - top);
  }
  
  // Rect'lerin kesişimini hesapla
  intersection(rect1, rect2) {
    if (!this.isIntersecting(rect1, rect2)) {
      return new DOMRect(0, 0, 0, 0);
    }
    
    const left = Math.max(rect1.left, rect2.left);
    const top = Math.max(rect1.top, rect2.top);
    const right = Math.min(rect1.right, rect2.right);
    const bottom = Math.min(rect1.bottom, rect2.bottom);
    
    return new DOMRect(left, top, right - left, bottom - top);
  }
}
```

**Alternatifler:**
- **CSS Transforms**: CSS transform özellikleri
- **CSS Grid**: CSS Grid layout
- **CSS Flexbox**: CSS Flexbox layout
- **Canvas API**: Canvas ile geometrik çizimler
- **SVG**: SVG ile geometrik şekiller

**Ne Zaman Kullanılmalı?**
- ✅ Element konumlandırma gerektiğinde
- ✅ Layout hesaplamaları gerektiğinde
- ✅ Animasyonlar gerektiğinde
- ✅ Collision detection gerektiğinde
- ✅ Responsive design gerektiğinde
- ✅ Performance optimization gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit CSS ile çözülebilecek durumlarda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Performans kritik olmayan durumlarda
- ❌ Karmaşık 3D geometri gerektiğinde
- ❌ WebGL gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Element Konumlandırma Sınırlılığı**: DOM elementlerinin konum bilgileri alınamaz
- **Layout Hesaplamaları Eksikliği**: Sayfa düzeni hesaplamaları yapılamaz
- **Animasyon Sınırlılığı**: Element animasyonları sınırlı olur
- **Collision Detection Eksikliği**: Element çarpışma tespiti yapılamaz
- **Responsive Design Sınırlılığı**: Responsive tasarım hesaplamaları yapılamaz

**Modern Alternatifler:**
- **CSS Houdini**: CSS özelliklerini JavaScript ile kontrol
- **Web Animations API**: Gelişmiş animasyonlar
- **Intersection Observer API**: Element görünürlük tespiti
- **Resize Observer API**: Element boyut değişiklik tespiti
- **WebGL**: 3D grafik ve geometri

---

## H - Bölümü

### HTML DOM API

**HTML DOM API Nedir?**
HTML DOM API, web uygulamalarının HTML belgelerini programatik olarak manipüle etmesini sağlayan temel web API'sidir. HTML elementlerini oluşturma, değiştirme, silme ve etkileşim kurma işlemlerini gerçekleştirir. Web sayfalarının dinamik içerik oluşturma, form işleme ve kullanıcı etkileşimlerini yönetme için kullanılır.

**Neden Kullanılır?**
- **Dinamik İçerik**: HTML içeriğini dinamik olarak oluşturma ve değiştirme
- **Form İşleme**: Form elementlerini oluşturma ve işleme
- **Event Handling**: Kullanıcı etkileşimlerini yönetme
- **DOM Manipulation**: Sayfa yapısını programatik olarak değiştirme
- **Element Selection**: HTML elementlerini seçme ve erişme
- **Attribute Management**: Element özelliklerini yönetme

**Temel Kullanım Örnekleri:**

```javascript
// HTML elementleri oluşturma
const div = document.createElement('div');
div.className = 'container';
div.id = 'main-container';
div.innerHTML = '<p>İçerik</p>';

// Element özelliklerini ayarlama
div.setAttribute('data-value', '123');
div.style.backgroundColor = 'lightblue';
div.style.padding = '20px';

// Element'i sayfaya ekleme
document.body.appendChild(div);

// Form elementleri oluşturma
const form = document.createElement('form');
form.method = 'POST';
form.action = '/submit';

const input = document.createElement('input');
input.type = 'text';
input.name = 'username';
input.placeholder = 'Kullanıcı adı';
input.required = true;

const button = document.createElement('button');
button.type = 'submit';
button.textContent = 'Gönder';

form.appendChild(input);
form.appendChild(button);
document.body.appendChild(form);

// Element seçme
const container = document.getElementById('main-container');
const allDivs = document.querySelectorAll('div');
const firstInput = document.querySelector('input[type="text"]');

// Event listener ekleme
button.addEventListener('click', (event) => {
  event.preventDefault();
  console.log('Form gönderildi:', input.value);
});

// Element içeriğini değiştirme
container.innerHTML = '<h1>Yeni Başlık</h1><p>Yeni içerik</p>';

// Element'i kaldırma
container.remove();
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// HTML DOM yöneticisi
class HTMLDOMManager {
  constructor() {
    this.elements = new Map();
    this.eventListeners = new Map();
  }
  
  // Element oluştur
  createElement(tagName, options = {}) {
    const element = document.createElement(tagName);
    
    // Özellikleri ayarla
    if (options.className) {
      element.className = options.className;
    }
    
    if (options.id) {
      element.id = options.id;
    }
    
    if (options.textContent) {
      element.textContent = options.textContent;
    }
    
    if (options.innerHTML) {
      element.innerHTML = options.innerHTML;
    }
    
    if (options.attributes) {
      Object.entries(options.attributes).forEach(([key, value]) => {
        element.setAttribute(key, value);
      });
    }
    
    if (options.styles) {
      Object.assign(element.style, options.styles);
    }
    
    // Event listener'ları ekle
    if (options.events) {
      Object.entries(options.events).forEach(([event, handler]) => {
        element.addEventListener(event, handler);
      });
    }
    
    // Element'i kaydet
    if (options.id) {
      this.elements.set(options.id, element);
    }
    
    return element;
  }
  
  // Element seç
  selectElement(selector) {
    if (selector.startsWith('#')) {
      return document.getElementById(selector.slice(1));
    } else if (selector.startsWith('.')) {
      return document.querySelector(selector);
    } else {
      return document.querySelector(selector);
    }
  }
  
  // Element'i güncelle
  updateElement(element, updates) {
    if (typeof element === 'string') {
      element = this.selectElement(element);
    }
    
    if (!element) return;
    
    if (updates.textContent !== undefined) {
      element.textContent = updates.textContent;
    }
    
    if (updates.innerHTML !== undefined) {
      element.innerHTML = updates.innerHTML;
    }
    
    if (updates.className !== undefined) {
      element.className = updates.className;
    }
    
    if (updates.styles) {
      Object.assign(element.style, updates.styles);
    }
  }
  
  // Form verilerini al
  getFormData(form) {
    if (typeof form === 'string') {
      form = this.selectElement(form);
    }
    
    if (!form) return {};
    
    const formData = new FormData(form);
    const data = {};
    
    for (const [key, value] of formData.entries()) {
      data[key] = value;
    }
    
    return data;
  }
}
```

**Alternatifler:**
- **jQuery**: DOM manipülasyonu için kütüphane
- **React**: Virtual DOM ile component tabanlı
- **Vue.js**: Reactive DOM manipülasyonu
- **Angular**: Framework tabanlı DOM yönetimi
- **Lit**: Web Components tabanlı

**Ne Zaman Kullanılmalı?**
- ✅ Dinamik içerik oluşturma gerektiğinde
- ✅ Form işleme gerektiğinde
- ✅ Event handling gerektiğinde
- ✅ DOM manipülasyonu gerektiğinde
- ✅ Element seçimi gerektiğinde
- ✅ Attribute yönetimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Modern framework kullanıldığında
- ❌ Server-side rendering gerektiğinde
- ❌ Karmaşık state management gerektiğinde
- ❌ Performance kritik uygulamalarda
- ❌ Büyük ölçekli uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Dinamik İçerik Eksikliği**: HTML içeriği dinamik olarak oluşturulamaz
- **Form İşleme Sınırlılığı**: Form elementleri işlenemez
- **Event Handling Eksikliği**: Kullanıcı etkileşimleri yönetilemez
- **DOM Manipülasyonu Sınırlılığı**: Sayfa yapısı değiştirilemez
- **Element Erişimi Sınırlılığı**: HTML elementlerine erişim sağlanamaz

**Modern Alternatifler:**
- **Web Components**: Custom element oluşturma
- **Shadow DOM**: Encapsulated DOM
- **Template Element**: HTML template kullanımı
- **DocumentFragment**: Performanslı DOM manipülasyonu
- **MutationObserver**: DOM değişiklik izleme

### History API

**History API Nedir?**
History API, web uygulamalarının tarayıcı geçmişini programatik olarak yönetmesini sağlayan modern bir web API'sidir. Sayfa geçmişine yeni girişler ekleme, mevcut girişleri değiştirme ve kullanıcının geri/ileri butonlarıyla etkileşim kurmasını sağlar. Single Page Application (SPA) geliştirme ve sayfa geçmişi yönetimi için kullanılır.

**Neden Kullanılır?**
- **SPA Navigasyon**: Single Page Application'larda sayfa geçmişi yönetimi
- **URL Yönetimi**: URL'leri programatik olarak değiştirme
- **Geri/İleri Butonları**: Tarayıcı geri/ileri butonlarını destekleme
- **State Management**: Sayfa durumunu geçmişte saklama
- **Bookmarking**: Sayfaları yer imlerine ekleme
- **SEO Optimization**: Arama motoru optimizasyonu

**Temel Kullanım Örnekleri:**

```javascript
// Yeni sayfa ekle
history.pushState({page: 'home'}, 'Ana Sayfa', '/home');

// Sayfa değiştir
history.replaceState({page: 'about'}, 'Hakkında', '/about');

// Geri git
history.back();

// İleri git
history.forward();

// Belirli bir sayfa sayısı kadar git
history.go(-2); // 2 sayfa geri
history.go(1);  // 1 sayfa ileri

// Geçmiş değişikliklerini dinle
window.addEventListener('popstate', event => {
  console.log('Geçmiş değişti:', event.state);
  console.log('Mevcut URL:', window.location.href);
  
  // Sayfa durumunu güncelle
  if (event.state) {
    updatePageContent(event.state);
  }
});

// Sayfa durumunu güncelle
function updatePageContent(state) {
  const content = document.getElementById('content');
  
  switch (state.page) {
    case 'home':
      content.innerHTML = '<h1>Ana Sayfa</h1><p>Hoş geldiniz!</p>';
      break;
    case 'about':
      content.innerHTML = '<h1>Hakkında</h1><p>Hakkımızda bilgiler</p>';
      break;
    case 'contact':
      content.innerHTML = '<h1>İletişim</h1><p>İletişim bilgileri</p>';
      break;
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// History yöneticisi
class HistoryManager {
  constructor() {
    this.routes = new Map();
    this.currentState = null;
    this.setupEventListeners();
  }
  
  // Route ekle
  addRoute(path, handler, title) {
    this.routes.set(path, { handler, title });
  }
  
  // Sayfa geçmişine ekle
  navigateTo(path, state = {}) {
    const route = this.routes.get(path);
    
    if (route) {
      const fullState = {
        path: path,
        title: route.title,
        timestamp: Date.now(),
        ...state
      };
      
      history.pushState(fullState, route.title, path);
      this.currentState = fullState;
      
      // Sayfa içeriğini güncelle
      route.handler(fullState);
      
      // Sayfa başlığını güncelle
      document.title = route.title;
      
      console.log('Sayfa değişti:', path);
    } else {
      console.error('Route bulunamadı:', path);
    }
  }
  
  // Mevcut geçmiş girişini değiştir
  replaceState(path, state = {}) {
    const route = this.routes.get(path);
    
    if (route) {
      const fullState = {
        path: path,
        title: route.title,
        timestamp: Date.now(),
        ...state
      };
      
      history.replaceState(fullState, route.title, path);
      this.currentState = fullState;
      
      // Sayfa içeriğini güncelle
      route.handler(fullState);
      
      // Sayfa başlığını güncelle
      document.title = route.title;
      
      console.log('Sayfa değiştirildi:', path);
    }
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    window.addEventListener('popstate', (event) => {
      if (event.state) {
        this.currentState = event.state;
        const route = this.routes.get(event.state.path);
        
        if (route) {
          route.handler(event.state);
          document.title = route.title;
        }
      }
    });
  }
  
  // Mevcut durumu al
  getCurrentState() {
    return this.currentState;
  }
  
  // Geçmiş uzunluğunu al
  getHistoryLength() {
    return history.length;
  }
}
```

**Alternatifler:**
- **Hash Routing**: URL hash kullanarak routing
- **React Router**: React için routing kütüphanesi
- **Vue Router**: Vue.js için routing kütüphanesi
- **Angular Router**: Angular için routing
- **Page.js**: Minimal routing kütüphanesi

**Ne Zaman Kullanılmalı?**
- ✅ Single Page Application geliştirirken
- ✅ URL yönetimi gerektiğinde
- ✅ Geri/ileri buton desteği gerektiğinde
- ✅ State management gerektiğinde
- ✅ Bookmarking desteği gerektiğinde
- ✅ SEO optimization gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit web sitelerinde
- ❌ Multi-page application'larda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Server-side rendering gerektiğinde
- ❌ Karmaşık routing gerektiğinde

**Kullanılmazsa Ne Olur?**
- **SPA Navigasyon Eksikliği**: Single Page Application'larda sayfa geçmişi yönetilemez
- **URL Yönetimi Sınırlılığı**: URL'ler programatik olarak değiştirilemez
- **Geri/İleri Buton Sınırlılığı**: Tarayıcı geri/ileri butonları çalışmaz
- **State Management Eksikliği**: Sayfa durumu geçmişte saklanamaz
- **Bookmarking Sınırlılığı**: Sayfalar yer imlerine eklenemez

**Modern Alternatifler:**
- **React Router**: React için routing
- **Vue Router**: Vue.js için routing
- **Angular Router**: Angular için routing
- **Next.js Router**: Next.js için routing
- **SvelteKit**: Svelte için routing

---

## I - Bölümü

### IndexedDB API

**IndexedDB API Nedir?**
IndexedDB API, web uygulamalarının tarayıcıda büyük miktarlarda yapılandırılmış veriyi depolamasını sağlayan modern bir web API'sidir. NoSQL tabanlı, asenkron ve transaction destekli bir veritabanı sistemidir. Offline uygulamalar, büyük veri setleri ve karmaşık veri yapıları için kullanılır.

**Neden Kullanılır?**
- **Büyük Veri Depolama**: Milyonlarca kayıt depolama
- **Offline Uygulamalar**: İnternet bağlantısı olmadan çalışma
- **Karmaşık Veri Yapıları**: Object, Array, Blob gibi veri türleri
- **Transaction Desteği**: Veri tutarlılığı sağlama
- **Indexing**: Hızlı veri arama
- **Asenkron İşlemler**: UI'ı bloklamadan veri işleme

**Temel Kullanım Örnekleri:**

```javascript
// Veritabanı açma
const request = indexedDB.open('MyDatabase', 1);

// Veritabanı yükseltme
request.onupgradeneeded = event => {
  const db = event.target.result;
  
  // Object store oluştur
  const store = db.createObjectStore('users', {keyPath: 'id'});
  
  // Index oluştur
  store.createIndex('name', 'name', {unique: false});
  store.createIndex('email', 'email', {unique: true});
};

// Veritabanı açma başarılı
request.onsuccess = event => {
  const db = event.target.result;
  
  // Transaction oluştur
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  
  // Veri ekle
  store.add({id: 1, name: 'John', email: 'john@example.com'});
  
  // Veri oku
  const getRequest = store.get(1);
  getRequest.onsuccess = () => {
    console.log('Kullanıcı:', getRequest.result);
  };
  
  // Veri güncelle
  store.put({id: 1, name: 'John Doe', email: 'john.doe@example.com'});
  
  // Veri sil
  store.delete(1);
};

// Veritabanı açma hatası
request.onerror = event => {
  console.error('Veritabanı açma hatası:', event.target.error);
};
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// IndexedDB yöneticisi
class IndexedDBManager {
  constructor(dbName, version) {
    this.dbName = dbName;
    this.version = version;
    this.db = null;
    this.stores = new Map();
  }
  
  // Veritabanını aç
  async open() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onupgradeneeded = (event) => {
        this.db = event.target.result;
        this.createStores();
      };
      
      request.onsuccess = (event) => {
        this.db = event.target.result;
        resolve(this.db);
      };
      
      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }
  
  // Object store oluştur
  createStore(storeName, options = {}) {
    if (!this.db) {
      throw new Error('Veritabanı açılmamış');
    }
    
    const store = this.db.createObjectStore(storeName, options);
    this.stores.set(storeName, store);
    
    return store;
  }
  
  // Index oluştur
  createIndex(storeName, indexName, keyPath, options = {}) {
    const store = this.stores.get(storeName);
    if (store) {
      store.createIndex(indexName, keyPath, options);
    }
  }
  
  // Veri ekle
  async add(storeName, data) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.add(data);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Veri oku
  async get(storeName, key) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.get(key);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Veri güncelle
  async put(storeName, data) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.put(data);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Veri sil
  async delete(storeName, key) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.delete(key);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Tüm verileri al
  async getAll(storeName) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.getAll();
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Index ile arama
  async getByIndex(storeName, indexName, value) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    const index = store.index(indexName);
    
    return new Promise((resolve, reject) => {
      const request = index.get(value);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Veri sayısını al
  async count(storeName) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.count();
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Veritabanını temizle
  async clear(storeName) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.clear();
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Veritabanını kapat
  close() {
    if (this.db) {
      this.db.close();
      this.db = null;
    }
  }
}
```

**Pratik Kullanım Senaryoları:**

```javascript
// Kullanıcı yönetimi
class UserManager {
  constructor() {
    this.dbManager = new IndexedDBManager('UserDatabase', 1);
    this.setupDatabase();
  }
  
  async setupDatabase() {
    await this.dbManager.open();
    
    // Users store oluştur
    this.dbManager.createStore('users', {keyPath: 'id'});
    this.dbManager.createIndex('users', 'email', 'email', {unique: true});
    this.dbManager.createIndex('users', 'name', 'name', {unique: false});
  }
  
  // Kullanıcı ekle
  async addUser(userData) {
    try {
      const user = {
        id: Date.now(),
        name: userData.name,
        email: userData.email,
        createdAt: new Date(),
        ...userData
      };
      
      await this.dbManager.add('users', user);
      console.log('Kullanıcı eklendi:', user);
      return user;
    } catch (error) {
      console.error('Kullanıcı ekleme hatası:', error);
      throw error;
    }
  }
  
  // Kullanıcı getir
  async getUser(id) {
    try {
      const user = await this.dbManager.get('users', id);
      return user;
    } catch (error) {
      console.error('Kullanıcı getirme hatası:', error);
      throw error;
    }
  }
  
  // E-posta ile kullanıcı bul
  async getUserByEmail(email) {
    try {
      const user = await this.dbManager.getByIndex('users', 'email', email);
      return user;
    } catch (error) {
      console.error('E-posta ile kullanıcı bulma hatası:', error);
      throw error;
    }
  }
  
  // Tüm kullanıcıları getir
  async getAllUsers() {
    try {
      const users = await this.dbManager.getAll('users');
      return users;
    } catch (error) {
      console.error('Kullanıcıları getirme hatası:', error);
      throw error;
    }
  }
  
  // Kullanıcı güncelle
  async updateUser(id, userData) {
    try {
      const existingUser = await this.dbManager.get('users', id);
      if (!existingUser) {
        throw new Error('Kullanıcı bulunamadı');
      }
      
      const updatedUser = {
        ...existingUser,
        ...userData,
        updatedAt: new Date()
      };
      
      await this.dbManager.put('users', updatedUser);
      console.log('Kullanıcı güncellendi:', updatedUser);
      return updatedUser;
    } catch (error) {
      console.error('Kullanıcı güncelleme hatası:', error);
      throw error;
    }
  }
  
  // Kullanıcı sil
  async deleteUser(id) {
    try {
      await this.dbManager.delete('users', id);
      console.log('Kullanıcı silindi:', id);
    } catch (error) {
      console.error('Kullanıcı silme hatası:', error);
      throw error;
    }
  }
}

// Offline veri senkronizasyonu
class OfflineSyncManager {
  constructor() {
    this.dbManager = new IndexedDBManager('OfflineDatabase', 1);
    this.syncQueue = [];
    this.setupDatabase();
  }
  
  async setupDatabase() {
    await this.dbManager.open();
    
    // Sync queue store oluştur
    this.dbManager.createStore('syncQueue', {keyPath: 'id'});
    this.dbManager.createIndex('syncQueue', 'status', 'status', {unique: false});
    this.dbManager.createIndex('syncQueue', 'timestamp', 'timestamp', {unique: false});
  }
  
  // Offline işlem ekle
  async addToSyncQueue(operation, data) {
    try {
      const syncItem = {
        id: Date.now() + Math.random(),
        operation: operation, // 'create', 'update', 'delete'
        data: data,
        status: 'pending',
        timestamp: new Date()
      };
      
      await this.dbManager.add('syncQueue', syncItem);
      this.syncQueue.push(syncItem);
      
      console.log('Sync queue\'ya eklendi:', syncItem);
    } catch (error) {
      console.error('Sync queue ekleme hatası:', error);
      throw error;
    }
  }
  
  // Bekleyen işlemleri senkronize et
  async syncPendingOperations() {
    try {
      const pendingItems = await this.dbManager.getByIndex('syncQueue', 'status', 'pending');
      
      for (const item of pendingItems) {
        try {
          await this.syncOperation(item);
          
          // Başarılı işlemleri işaretle
          item.status = 'completed';
          await this.dbManager.put('syncQueue', item);
          
        } catch (error) {
          // Başarısız işlemleri işaretle
          item.status = 'failed';
          item.error = error.message;
          await this.dbManager.put('syncQueue', item);
        }
      }
      
      console.log('Senkronizasyon tamamlandı');
    } catch (error) {
      console.error('Senkronizasyon hatası:', error);
      throw error;
    }
  }
  
  // İşlemi senkronize et
  async syncOperation(item) {
    // Burada gerçek API çağrıları yapılır
    console.log('Senkronize ediliyor:', item);
    
    // Simüle edilmiş API çağrısı
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}
```

**Alternatifler:**
- **Web Storage**: Basit key-value depolama
- **WebSQL**: SQL tabanlı depolama (deprecated)
- **LocalStorage**: Basit veri depolama
- **SessionStorage**: Oturum bazlı depolama
- **Cache API**: HTTP cache depolama

**Ne Zaman Kullanılmalı?**
- ✅ Büyük veri setleri gerektiğinde
- ✅ Offline uygulamalar gerektiğinde
- ✅ Karmaşık veri yapıları gerektiğinde
- ✅ Transaction desteği gerektiğinde
- ✅ Hızlı veri arama gerektiğinde
- ✅ Asenkron veri işleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit veri depolama gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Senkron veri işleme gerektiğinde
- ❌ Küçük veri setleri gerektiğinde
- ❌ Basit key-value depolama gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Büyük Veri Depolama Eksikliği**: Milyonlarca kayıt depolanamaz
- **Offline Uygulama Sınırlılığı**: İnternet bağlantısı olmadan çalışılamaz
- **Karmaşık Veri Yapısı Sınırlılığı**: Object, Array gibi veri türleri depolanamaz
- **Transaction Desteği Eksikliği**: Veri tutarlılığı sağlanamaz
- **Hızlı Arama Eksikliği**: Indexing ile hızlı arama yapılamaz

**Modern Alternatifler:**
- **Web Storage**: Basit key-value depolama
- **Cache API**: HTTP cache depolama
- **Web Locks API**: Veri kilitleme
- **Storage Access API**: Cross-origin depolama
- **Origin Private File System**: Dosya sistemi erişimi

### Intersection Observer API

**Intersection Observer API Nedir?**
Intersection Observer API, web uygulamalarının DOM elementlerinin görünürlük durumunu asenkron olarak gözlemlemesini sağlayan modern bir web API'sidir. Elementlerin viewport içinde görünüp görünmediğini, ne kadar görünür olduğunu ve hangi oranda görünür olduğunu tespit eder. Lazy loading, infinite scroll, animasyon tetikleme ve performans optimizasyonu için kullanılır.

**Neden Kullanılır?**
- **Lazy Loading**: Görünür olan içerikleri yükleme
- **Infinite Scroll**: Sayfa sonuna gelindiğinde içerik yükleme
- **Animasyon Tetikleme**: Element görünür olduğunda animasyon başlatma
- **Performance Optimization**: Görünmeyen elementleri optimize etme
- **Analytics**: Element görünürlük istatistikleri
- **Ad Tracking**: Reklam görünürlük takibi

**Temel Kullanım Örnekleri:**

```javascript
// Intersection Observer oluştur
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element görünür:', entry.target);
      console.log('Görünürlük oranı:', entry.intersectionRatio);
      console.log('Görünür alan:', entry.intersectionRect);
    } else {
      console.log('Element gizli:', entry.target);
    }
  });
}, {
  root: null, // Viewport
  rootMargin: '0px', // Root margin
  threshold: 0.1 // %10 görünür olduğunda tetikle
});

// Element'i gözlemle
const element = document.getElementById('myElement');
observer.observe(element);

// Gözlemlemeyi durdur
observer.unobserve(element);

// Tüm gözlemleri durdur
observer.disconnect();

// Gelişmiş konfigürasyon
const advancedObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const target = entry.target;
    
    if (entry.isIntersecting) {
      target.classList.add('visible');
      target.classList.remove('hidden');
    } else {
      target.classList.add('hidden');
      target.classList.remove('visible');
    }
  });
}, {
  root: document.getElementById('scrollContainer'),
  rootMargin: '50px 0px',
  threshold: [0, 0.25, 0.5, 0.75, 1.0]
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Intersection Observer yöneticisi
class IntersectionObserverManager {
  constructor() {
    this.observers = new Map();
    this.callbacks = new Map();
  }
  
  // Observer oluştur
  createObserver(name, options = {}) {
    const defaultOptions = {
      root: null,
      rootMargin: '0px',
      threshold: 0.1
    };
    
    const config = { ...defaultOptions, ...options };
    
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        const callback = this.callbacks.get(name);
        if (callback) {
          callback(entry);
        }
      });
    }, config);
    
    this.observers.set(name, observer);
    return observer;
  }
  
  // Callback ekle
  addCallback(name, callback) {
    this.callbacks.set(name, callback);
  }
  
  // Element gözlemle
  observe(name, element) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.observe(element);
    }
  }
  
  // Element gözlemlemeyi durdur
  unobserve(name, element) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.unobserve(element);
    }
  }
  
  // Observer'ı durdur
  disconnect(name) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.disconnect();
      this.observers.delete(name);
      this.callbacks.delete(name);
    }
  }
}

// Lazy loading yöneticisi
class LazyLoadingManager {
  constructor() {
    this.observerManager = new IntersectionObserverManager();
    this.setupLazyLoading();
  }
  
  setupLazyLoading() {
    // Lazy loading observer oluştur
    this.observerManager.createObserver('lazy', {
      rootMargin: '50px 0px',
      threshold: 0.1
    });
    
    // Lazy loading callback
    this.observerManager.addCallback('lazy', (entry) => {
      if (entry.isIntersecting) {
        this.loadContent(entry.target);
        this.observerManager.unobserve('lazy', entry.target);
      }
    });
  }
  
  // Lazy loading için element ekle
  addLazyElement(element, src, alt = '') {
    element.setAttribute('data-src', src);
    element.setAttribute('data-alt', alt);
    element.classList.add('lazy');
    
    // Placeholder ekle
    element.style.backgroundColor = '#f0f0f0';
    element.style.minHeight = '200px';
    
    this.observerManager.observe('lazy', element);
  }
  
  // İçerik yükle
  loadContent(element) {
    const src = element.getAttribute('data-src');
    const alt = element.getAttribute('data-alt');
    
    if (element.tagName === 'IMG') {
      element.src = src;
      element.alt = alt;
    } else if (element.tagName === 'VIDEO') {
      element.src = src;
    } else {
      // Diğer element türleri için
      element.style.backgroundImage = `url(${src})`;
    }
    
    element.classList.remove('lazy');
    element.classList.add('loaded');
    
    console.log('İçerik yüklendi:', src);
  }
}
```

**Alternatifler:**
- **Scroll Events**: Scroll olayları ile görünürlük tespiti
- **getBoundingClientRect**: Element pozisyon hesaplama
- **ResizeObserver**: Element boyut değişiklik tespiti
- **MutationObserver**: DOM değişiklik tespiti
- **Performance Observer**: Performans metrikleri

**Ne Zaman Kullanılmalı?**
- ✅ Lazy loading gerektiğinde
- ✅ Infinite scroll gerektiğinde
- ✅ Animasyon tetikleme gerektiğinde
- ✅ Performance optimization gerektiğinde
- ✅ Analytics tracking gerektiğinde
- ✅ Ad tracking gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit görünürlük tespiti gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Senkron görünürlük tespiti gerektiğinde
- ❌ Karmaşık hesaplamalar gerektiğinde
- ❌ Real-time tracking gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Lazy Loading Eksikliği**: Tüm içerikler sayfa yüklenirken yüklenir
- **Infinite Scroll Sınırlılığı**: Sayfa sonuna gelindiğinde içerik yüklenemez
- **Animasyon Tetikleme Eksikliği**: Element görünür olduğunda animasyon başlamaz
- **Performance Sorunları**: Görünmeyen elementler optimize edilemez
- **Analytics Eksikliği**: Element görünürlük istatistikleri alınamaz

**Modern Alternatifler:**
- **ResizeObserver**: Element boyut değişiklik tespiti
- **MutationObserver**: DOM değişiklik tespiti
- **Performance Observer**: Performans metrikleri
- **Web Animations API**: Gelişmiş animasyonlar
- **CSS Containment**: CSS ile performans optimizasyonu

---

## L - Bölümü

### Local Storage API

**Local Storage API Nedir?**
Local Storage API, web uygulamalarının tarayıcıda kalıcı veri depolamasını sağlayan modern bir web API'sidir. Key-value tabanlı basit veri depolama sistemi sunar. Kullanıcı tercihleri, uygulama ayarları, geçici veriler ve offline veri depolama için kullanılır. Veriler tarayıcı kapatılsa bile kalıcı olarak saklanır.

**Neden Kullanılır?**
- **Kalıcı Veri Depolama**: Tarayıcı kapatılsa bile veri saklama
- **Kullanıcı Tercihleri**: Tema, dil, ayarlar gibi tercihleri saklama
- **Offline Veri**: İnternet bağlantısı olmadan veri erişimi
- **Basit Veri Yapısı**: Key-value tabanlı kolay kullanım
- **Performans**: Hızlı veri erişimi
- **Cross-Tab Sync**: Aynı origin'deki tüm sekmeler arasında senkronizasyon

**Temel Kullanım Örnekleri:**

```javascript
// Veri kaydet
localStorage.setItem('username', 'john_doe');
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'tr');
localStorage.setItem('lastVisit', new Date().toISOString());

// Veri oku
const username = localStorage.getItem('username');
const theme = localStorage.getItem('theme');
const language = localStorage.getItem('language');
const lastVisit = localStorage.getItem('lastVisit');

console.log('Kullanıcı:', username);
console.log('Tema:', theme);
console.log('Dil:', language);
console.log('Son ziyaret:', lastVisit);

// Veri sil
localStorage.removeItem('username');

// Tüm verileri temizle
localStorage.clear();

// Veri varlığını kontrol et
if (localStorage.getItem('username')) {
  console.log('Kullanıcı giriş yapmış');
} else {
  console.log('Kullanıcı giriş yapmamış');
}

// Local Storage boyutunu kontrol et
function getLocalStorageSize() {
  let total = 0;
  for (let key in localStorage) {
    if (localStorage.hasOwnProperty(key)) {
      total += localStorage[key].length + key.length;
    }
  }
  return total;
}

console.log('Local Storage boyutu:', getLocalStorageSize(), 'bytes');

// Local Storage değişikliklerini dinle
window.addEventListener('storage', (event) => {
  console.log('Storage değişti:', event.key, event.newValue, event.oldValue);
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Local Storage yöneticisi
class LocalStorageManager {
  constructor() {
    this.prefix = 'app_';
    this.maxSize = 5 * 1024 * 1024; // 5MB
    this.setupEventListeners();
  }
  
  // Veri kaydet
  setItem(key, value, options = {}) {
    try {
      const fullKey = this.prefix + key;
      const data = {
        value: value,
        timestamp: Date.now(),
        expires: options.expires ? Date.now() + options.expires : null,
        ...options
      };
      
      const serializedData = JSON.stringify(data);
      
      // Boyut kontrolü
      if (this.getStorageSize() + serializedData.length > this.maxSize) {
        throw new Error('Local Storage dolu');
      }
      
      localStorage.setItem(fullKey, serializedData);
      console.log('Veri kaydedildi:', key);
    } catch (error) {
      console.error('Veri kaydetme hatası:', error);
      throw error;
    }
  }
  
  // Veri oku
  getItem(key, defaultValue = null) {
    try {
      const fullKey = this.prefix + key;
      const item = localStorage.getItem(fullKey);
      
      if (!item) {
        return defaultValue;
      }
      
      const data = JSON.parse(item);
      
      // Süre kontrolü
      if (data.expires && Date.now() > data.expires) {
        this.removeItem(key);
        return defaultValue;
      }
      
      return data.value;
    } catch (error) {
      console.error('Veri okuma hatası:', error);
      return defaultValue;
    }
  }
  
  // Veri sil
  removeItem(key) {
    const fullKey = this.prefix + key;
    localStorage.removeItem(fullKey);
    console.log('Veri silindi:', key);
  }
  
  // Tüm verileri temizle
  clear() {
    const keys = Object.keys(localStorage);
    keys.forEach(key => {
      if (key.startsWith(this.prefix)) {
        localStorage.removeItem(key);
      }
    });
    console.log('Tüm veriler temizlendi');
  }
  
  // Veri varlığını kontrol et
  hasItem(key) {
    const fullKey = this.prefix + key;
    return localStorage.getItem(fullKey) !== null;
  }
  
  // Tüm anahtarları listele
  getAllKeys() {
    const keys = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key && key.startsWith(this.prefix)) {
        keys.push(key.substring(this.prefix.length));
      }
    }
    return keys;
  }
  
  // Storage boyutunu hesapla
  getStorageSize() {
    let total = 0;
    for (let key in localStorage) {
      if (localStorage.hasOwnProperty(key) && key.startsWith(this.prefix)) {
        total += localStorage[key].length + key.length;
      }
    }
    return total;
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    window.addEventListener('storage', (event) => {
      if (event.key && event.key.startsWith(this.prefix)) {
        const key = event.key.substring(this.prefix.length);
        this.onStorageChange(key, event.newValue, event.oldValue);
      }
    });
  }
  
  // Storage değişiklik callback'i
  onStorageChange(key, newValue, oldValue) {
    console.log('Storage değişti:', key, newValue, oldValue);
  }
}

// Kullanıcı tercihleri yöneticisi
class UserPreferencesManager {
  constructor() {
    this.storage = new LocalStorageManager();
    this.preferences = this.loadPreferences();
  }
  
  // Tercihleri yükle
  loadPreferences() {
    return {
      theme: this.storage.getItem('theme', 'light'),
      language: this.storage.getItem('language', 'tr'),
      fontSize: this.storage.getItem('fontSize', 'medium'),
      notifications: this.storage.getItem('notifications', true),
      autoSave: this.storage.getItem('autoSave', true),
      darkMode: this.storage.getItem('darkMode', false)
    };
  }
  
  // Tercih güncelle
  updatePreference(key, value) {
    this.preferences[key] = value;
    this.storage.setItem(key, value);
    this.applyPreference(key, value);
  }
  
  // Tercihi uygula
  applyPreference(key, value) {
    switch (key) {
      case 'theme':
        document.body.className = value;
        break;
      case 'language':
        document.documentElement.lang = value;
        break;
      case 'fontSize':
        document.body.style.fontSize = this.getFontSize(value);
        break;
      case 'darkMode':
        document.body.classList.toggle('dark-mode', value);
        break;
    }
  }
  
  // Font boyutunu al
  getFontSize(size) {
    const sizes = {
      small: '14px',
      medium: '16px',
      large: '18px',
      xlarge: '20px'
    };
    return sizes[size] || sizes.medium;
  }
  
  // Tüm tercihleri uygula
  applyAllPreferences() {
    Object.entries(this.preferences).forEach(([key, value]) => {
      this.applyPreference(key, value);
    });
  }
  
  // Tercihleri sıfırla
  resetPreferences() {
    this.storage.clear();
    this.preferences = this.loadPreferences();
    this.applyAllPreferences();
  }
}

// Offline veri yöneticisi
class OfflineDataManager {
  constructor() {
    this.storage = new LocalStorageManager();
    this.syncQueue = [];
  }
  
  // Offline veri kaydet
  saveOfflineData(key, data) {
    try {
      const offlineData = {
        data: data,
        timestamp: Date.now(),
        synced: false
      };
      
      this.storage.setItem(`offline_${key}`, offlineData);
      this.addToSyncQueue(key, 'save', data);
      
      console.log('Offline veri kaydedildi:', key);
    } catch (error) {
      console.error('Offline veri kaydetme hatası:', error);
      throw error;
    }
  }
  
  // Offline veri oku
  getOfflineData(key) {
    const offlineData = this.storage.getItem(`offline_${key}`);
    return offlineData ? offlineData.data : null;
  }
  
  // Sync queue'ya ekle
  addToSyncQueue(key, operation, data) {
    const syncItem = {
      key: key,
      operation: operation,
      data: data,
      timestamp: Date.now()
    };
    
    this.syncQueue.push(syncItem);
    this.storage.setItem('syncQueue', this.syncQueue);
  }
  
  // Bekleyen verileri senkronize et
  async syncPendingData() {
    const queue = this.storage.getItem('syncQueue', []);
    
    for (const item of queue) {
      try {
        await this.syncData(item);
        this.markAsSynced(item.key);
      } catch (error) {
        console.error('Senkronizasyon hatası:', error);
      }
    }
  }
  
  // Veriyi senkronize et
  async syncData(item) {
    // Burada gerçek API çağrıları yapılır
    console.log('Senkronize ediliyor:', item);
    
    // Simüle edilmiş API çağrısı
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  // Senkronize edildi olarak işaretle
  markAsSynced(key) {
    const offlineData = this.storage.getItem(`offline_${key}`);
    if (offlineData) {
      offlineData.synced = true;
      this.storage.setItem(`offline_${key}`, offlineData);
    }
  }
}
```

**Alternatifler:**
- **Session Storage**: Oturum bazlı depolama
- **IndexedDB**: Büyük veri depolama
- **Web SQL**: SQL tabanlı depolama (deprecated)
- **Cookies**: HTTP cookie'leri
- **Cache API**: HTTP cache depolama

**Ne Zaman Kullanılmalı?**
- ✅ Kullanıcı tercihleri gerektiğinde
- ✅ Basit veri depolama gerektiğinde
- ✅ Offline veri gerektiğinde
- ✅ Cross-tab senkronizasyon gerektiğinde
- ✅ Kalıcı veri gerektiğinde
- ✅ Hızlı veri erişimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Büyük veri setleri gerektiğinde
- ❌ Karmaşık veri yapıları gerektiğinde
- ❌ Güvenlik kritik veri gerektiğinde
- ❌ Binary veri gerektiğinde
- ❌ Transaction desteği gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Kalıcı Veri Depolama Eksikliği**: Veriler tarayıcı kapatıldığında kaybolur
- **Kullanıcı Tercihleri Eksikliği**: Kullanıcı ayarları saklanamaz
- **Offline Veri Eksikliği**: İnternet bağlantısı olmadan veri erişilemez
- **Cross-Tab Sync Eksikliği**: Sekmeler arası veri paylaşımı olmaz
- **Performans Sorunları**: Her seferinde server'dan veri çekilir

**Modern Alternatifler:**
- **IndexedDB**: Büyük veri depolama
- **Web Storage**: Basit key-value depolama
- **Cache API**: HTTP cache depolama
- **Web Locks API**: Veri kilitleme
- **Storage Access API**: Cross-origin depolama

---

## M - Bölümü

### Media Capture and Streams API

**Media Capture and Streams API Nedir?**
Media Capture and Streams API, web uygulamalarının kullanıcının kamera, mikrofon ve ekran gibi medya cihazlarına erişmesini sağlayan modern bir web API'sidir. Gerçek zamanlı medya akışları oluşturma, video/audio kaydetme, canlı yayın yapma ve medya işleme için kullanılır. WebRTC, video konferans, canlı yayın ve medya uygulamalarının temelini oluşturur.

**Neden Kullanılır?**
- **Kamera Erişimi**: Web kamerasından video akışı alma
- **Mikrofon Erişimi**: Ses kaydetme ve işleme
- **Ekran Paylaşımı**: Ekran görüntüsü ve ekran kaydı
- **Canlı Yayın**: Gerçek zamanlı video/audio yayını
- **Video Konferans**: WebRTC tabanlı iletişim
- **Medya İşleme**: Video/audio düzenleme ve filtreleme
- **Güvenlik**: Kullanıcı izni tabanlı erişim

**Temel Kullanım Örnekleri:**

```javascript
// Kamera erişimi
navigator.mediaDevices.getUserMedia({video: true, audio: true})
  .then(stream => {
    const video = document.getElementById('video');
    video.srcObject = stream;
  })
  .catch(err => console.error('Kamera erişim hatası:', err));

// Sadece video
navigator.mediaDevices.getUserMedia({video: true})
  .then(stream => {
    const video = document.getElementById('video');
    video.srcObject = stream;
  });

// Sadece ses
navigator.mediaDevices.getUserMedia({audio: true})
  .then(stream => {
    const audio = document.getElementById('audio');
    audio.srcObject = stream;
  });

// Yüksek kalite video
navigator.mediaDevices.getUserMedia({
  video: {
    width: { ideal: 1920 },
    height: { ideal: 1080 },
    frameRate: { ideal: 30 }
  },
  audio: {
    echoCancellation: true,
    noiseSuppression: true
  }
})
.then(stream => {
  const video = document.getElementById('video');
  video.srcObject = stream;
});

// Medya cihazlarını listele
navigator.mediaDevices.enumerateDevices()
  .then(devices => {
    devices.forEach(device => {
      console.log(`${device.kind}: ${device.label} (${device.deviceId})`);
    });
  });

// Akışı durdur
function stopStream(stream) {
  stream.getTracks().forEach(track => {
    track.stop();
  });
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Medya yöneticisi
class MediaManager {
  constructor() {
    this.currentStream = null;
    this.recorder = null;
    this.recordedChunks = [];
    this.isSupported = 'mediaDevices' in navigator;
  }
  
  // Kamera erişimi
  async getCamera(options = {}) {
    if (!this.isSupported) {
      throw new Error('Media Devices API desteklenmiyor');
    }
    
    try {
      const defaultOptions = {
        video: {
          width: { ideal: 1280 },
          height: { ideal: 720 },
          frameRate: { ideal: 30 }
        },
        audio: true
      };
      
      const config = { ...defaultOptions, ...options };
      this.currentStream = await navigator.mediaDevices.getUserMedia(config);
      
      console.log('Kamera erişimi başarılı');
      return this.currentStream;
    } catch (error) {
      console.error('Kamera erişim hatası:', error);
      throw error;
    }
  }
  
  // Mikrofon erişimi
  async getMicrophone(options = {}) {
    if (!this.isSupported) {
      throw new Error('Media Devices API desteklenmiyor');
    }
    
    try {
      const defaultOptions = {
        audio: {
          echoCancellation: true,
          noiseSuppression: true,
          autoGainControl: true
        }
      };
      
      const config = { ...defaultOptions, ...options };
      this.currentStream = await navigator.mediaDevices.getUserMedia(config);
      
      console.log('Mikrofon erişimi başarılı');
      return this.currentStream;
    } catch (error) {
      console.error('Mikrofon erişim hatası:', error);
      throw error;
    }
  }
  
  // Ekran paylaşımı
  async getScreenShare(options = {}) {
    if (!this.isSupported) {
      throw new Error('Media Devices API desteklenmiyor');
    }
    
    try {
      const defaultOptions = {
        video: {
          mediaSource: 'screen'
        },
        audio: false
      };
      
      const config = { ...defaultOptions, ...options };
      this.currentStream = await navigator.mediaDevices.getDisplayMedia(config);
      
      console.log('Ekran paylaşımı başarılı');
      return this.currentStream;
    } catch (error) {
      console.error('Ekran paylaşım hatası:', error);
      throw error;
    }
  }
  
  // Medya cihazlarını listele
  async getDevices() {
    if (!this.isSupported) {
      throw new Error('Media Devices API desteklenmiyor');
    }
    
    try {
      const devices = await navigator.mediaDevices.enumerateDevices();
      
      const deviceList = {
        videoInputs: devices.filter(d => d.kind === 'videoinput'),
        audioInputs: devices.filter(d => d.kind === 'audioinput'),
        audioOutputs: devices.filter(d => d.kind === 'audiooutput')
      };
      
      return deviceList;
    } catch (error) {
      console.error('Cihaz listesi hatası:', error);
      throw error;
    }
  }
  
  // Akışı durdur
  stopStream() {
    if (this.currentStream) {
      this.currentStream.getTracks().forEach(track => {
        track.stop();
      });
      this.currentStream = null;
      console.log('Medya akışı durduruldu');
    }
  }
  
  // Video kaydetme
  startRecording() {
    if (!this.currentStream) {
      throw new Error('Aktif medya akışı yok');
    }
    
    this.recordedChunks = [];
    this.recorder = new MediaRecorder(this.currentStream);
    
    this.recorder.ondataavailable = (event) => {
      if (event.data.size > 0) {
        this.recordedChunks.push(event.data);
      }
    };
    
    this.recorder.onstop = () => {
      const blob = new Blob(this.recordedChunks, { type: 'video/webm' });
      this.downloadRecording(blob);
    };
    
    this.recorder.start();
    console.log('Kayıt başladı');
  }
  
  // Kaydı durdur
  stopRecording() {
    if (this.recorder && this.recorder.state === 'recording') {
      this.recorder.stop();
      console.log('Kayıt durduruldu');
    }
  }
  
  // Kaydı indir
  downloadRecording(blob) {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `recording_${Date.now()}.webm`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }
}

// Video konferans yöneticisi
class VideoConferenceManager {
  constructor() {
    this.mediaManager = new MediaManager();
    this.peerConnections = new Map();
    this.localStream = null;
  }
  
  // Konferans başlat
  async startConference() {
    try {
      // Yerel akışı al
      this.localStream = await this.mediaManager.getCamera({
        video: true,
        audio: true
      });
      
      // Yerel video elementini güncelle
      const localVideo = document.getElementById('localVideo');
      localVideo.srcObject = this.localStream;
      
      console.log('Konferans başlatıldı');
    } catch (error) {
      console.error('Konferans başlatma hatası:', error);
      throw error;
    }
  }
  
  // Konferansı durdur
  stopConference() {
    this.mediaManager.stopStream();
    this.peerConnections.forEach(pc => pc.close());
    this.peerConnections.clear();
    console.log('Konferans durduruldu');
  }
  
  // Medya ayarlarını değiştir
  async changeMediaSettings(settings) {
    if (this.localStream) {
      const videoTrack = this.localStream.getVideoTracks()[0];
      const audioTrack = this.localStream.getAudioTracks()[0];
      
      if (videoTrack && settings.video) {
        await videoTrack.applyConstraints(settings.video);
      }
      
      if (audioTrack && settings.audio) {
        await audioTrack.applyConstraints(settings.audio);
      }
      
      console.log('Medya ayarları güncellendi');
    }
  }
}

// Canlı yayın yöneticisi
class LiveStreamManager {
  constructor() {
    this.mediaManager = new MediaManager();
    this.isStreaming = false;
    this.streamUrl = null;
  }
  
  // Canlı yayın başlat
  async startLiveStream(streamUrl) {
    try {
      const stream = await this.mediaManager.getCamera({
        video: {
          width: { ideal: 1920 },
          height: { ideal: 1080 },
          frameRate: { ideal: 30 }
        },
        audio: true
      });
      
      // Burada gerçek streaming servisine bağlantı kurulur
      // Örnek: WebRTC, RTMP, HLS
      this.streamUrl = streamUrl;
      this.isStreaming = true;
      
      console.log('Canlı yayın başlatıldı:', streamUrl);
    } catch (error) {
      console.error('Canlı yayın başlatma hatası:', error);
      throw error;
    }
  }
  
  // Canlı yayını durdur
  stopLiveStream() {
    this.mediaManager.stopStream();
    this.isStreaming = false;
    this.streamUrl = null;
    console.log('Canlı yayın durduruldu');
  }
  
  // Yayın durumunu kontrol et
  getStreamStatus() {
    return {
      isStreaming: this.isStreaming,
      streamUrl: this.streamUrl,
      hasStream: !!this.mediaManager.currentStream
    };
  }
}
```

**Alternatifler:**
- **WebRTC**: Gerçek zamanlı iletişim
- **Web Audio API**: Ses işleme
- **Canvas API**: Video işleme
- **File API**: Medya dosya işleme
- **Media Source Extensions**: Medya akışı

**Ne Zaman Kullanılmalı?**
- ✅ Video konferans uygulamaları gerektiğinde
- ✅ Canlı yayın yapılması gerektiğinde
- ✅ Medya kaydetme gerektiğinde
- ✅ Ekran paylaşımı gerektiğinde
- ✅ WebRTC uygulamaları gerektiğinde
- ✅ Medya işleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Sadece statik medya gerektiğinde
- ❌ Güvenlik kritik uygulamalarda
- ❌ Offline uygulamalarda
- ❌ Basit medya oynatma gerektiğinde
- ❌ Performans kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Kamera Erişimi Eksikliği**: Web kamerası kullanılamaz
- **Mikrofon Erişimi Eksikliği**: Ses kaydedilemez
- **Canlı Yayın Eksikliği**: Gerçek zamanlı yayın yapılamaz
- **Video Konferans Eksikliği**: WebRTC iletişimi olmaz
- **Ekran Paylaşımı Eksikliği**: Ekran görüntüsü paylaşılamaz

**Modern Alternatifler:**
- **WebRTC**: Gerçek zamanlı iletişim
- **Web Audio API**: Ses işleme
- **Media Source Extensions**: Medya akışı
- **WebCodecs API**: Medya kodlama
- **WebAssembly**: Performans kritik işlemler

### Media Session API

**Media Session API Nedir?**
Media Session API, web uygulamalarının medya oturumlarını yönetmesini sağlayan modern bir web API'sidir. Medya oynatma bilgilerini (başlık, sanatçı, albüm, kapak resmi) sistem seviyesinde paylaşır ve medya kontrollerini (oynat, duraklat, ileri, geri) yönetir. Kullanıcının cihazında medya bildirimleri, kilit ekran kontrolleri ve medya merkezi entegrasyonu sağlar.

**Neden Kullanılır?**
- **Medya Bilgileri**: Şarkı, video bilgilerini sistem seviyesinde paylaşma
- **Medya Kontrolleri**: Oynat, duraklat, ileri, geri kontrolleri
- **Bildirim Entegrasyonu**: Medya bildirimleri ve kilit ekran kontrolleri
- **Sistem Entegrasyonu**: İşletim sistemi medya merkezi entegrasyonu
- **Kullanıcı Deneyimi**: Tutarlı medya deneyimi
- **Arka Plan Oynatma**: Uygulama arka plandayken medya kontrolü

**Temel Kullanım Örnekleri:**

```javascript
if ('mediaSession' in navigator) {
  // Medya bilgilerini ayarla
  navigator.mediaSession.metadata = new MediaMetadata({
    title: 'Şarkı Adı',
    artist: 'Sanatçı Adı',
    album: 'Albüm Adı',
    artwork: [
      {src: 'cover-small.jpg', sizes: '96x96', type: 'image/jpeg'},
      {src: 'cover-medium.jpg', sizes: '128x128', type: 'image/jpeg'},
      {src: 'cover-large.jpg', sizes: '192x192', type: 'image/jpeg'},
      {src: 'cover-xl.jpg', sizes: '512x512', type: 'image/jpeg'}
    ]
  });
  
  // Medya kontrollerini ayarla
  navigator.mediaSession.setActionHandler('play', () => {
    console.log('Oynat');
    // Oynatma mantığı
  });
  
  navigator.mediaSession.setActionHandler('pause', () => {
    console.log('Duraklat');
    // Duraklatma mantığı
  });
  
  navigator.mediaSession.setActionHandler('previoustrack', () => {
    console.log('Önceki şarkı');
    // Önceki şarkı mantığı
  });
  
  navigator.mediaSession.setActionHandler('nexttrack', () => {
    console.log('Sonraki şarkı');
    // Sonraki şarkı mantığı
  });
  
  // Oynatma durumunu güncelle
  navigator.mediaSession.playbackState = 'playing';
}

// Medya pozisyonunu güncelle
if ('setPositionState' in navigator.mediaSession) {
  navigator.mediaSession.setPositionState({
    duration: 180, // 3 dakika
    playbackRate: 1.0,
    position: 30 // 30 saniye
  });
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Medya oturumu yöneticisi
class MediaSessionManager {
  constructor() {
    this.isSupported = 'mediaSession' in navigator;
    this.currentTrack = null;
    this.playbackState = 'none';
    this.positionState = null;
    
    if (this.isSupported) {
      this.setupActionHandlers();
    }
  }
  
  // Medya bilgilerini ayarla
  setMetadata(track) {
    if (!this.isSupported) return;
    
    this.currentTrack = track;
    
    navigator.mediaSession.metadata = new MediaMetadata({
      title: track.title,
      artist: track.artist,
      album: track.album,
      artwork: track.artwork || this.getDefaultArtwork()
    });
    
    console.log('Medya bilgileri güncellendi:', track.title);
  }
  
  // Varsayılan kapak resmi
  getDefaultArtwork() {
    return [
      {src: '/images/default-cover-96.jpg', sizes: '96x96', type: 'image/jpeg'},
      {src: '/images/default-cover-128.jpg', sizes: '128x128', type: 'image/jpeg'},
      {src: '/images/default-cover-192.jpg', sizes: '192x192', type: 'image/jpeg'},
      {src: '/images/default-cover-512.jpg', sizes: '512x512', type: 'image/jpeg'}
    ];
  }
  
  // Oynatma durumunu güncelle
  setPlaybackState(state) {
    if (!this.isSupported) return;
    
    this.playbackState = state;
    navigator.mediaSession.playbackState = state;
    
    console.log('Oynatma durumu:', state);
  }
  
  // Pozisyon durumunu güncelle
  setPositionState(duration, position, playbackRate = 1.0) {
    if (!this.isSupported || !('setPositionState' in navigator.mediaSession)) return;
    
    this.positionState = { duration, position, playbackRate };
    
    navigator.mediaSession.setPositionState({
      duration: duration,
      position: position,
      playbackRate: playbackRate
    });
    
    console.log('Pozisyon güncellendi:', position, '/', duration);
  }
  
  // Action handler'ları kur
  setupActionHandlers() {
    // Oynat
    navigator.mediaSession.setActionHandler('play', () => {
      this.onPlay();
    });
    
    // Duraklat
    navigator.mediaSession.setActionHandler('pause', () => {
      this.onPause();
    });
    
    // Önceki şarkı
    navigator.mediaSession.setActionHandler('previoustrack', () => {
      this.onPreviousTrack();
    });
    
    // Sonraki şarkı
    navigator.mediaSession.setActionHandler('nexttrack', () => {
      this.onNextTrack();
    });
    
    // Seek
    navigator.mediaSession.setActionHandler('seekto', (details) => {
      this.onSeekTo(details.seekTime);
    });
    
    // Seek forward
    navigator.mediaSession.setActionHandler('seekforward', (details) => {
      this.onSeekForward(details.seekOffset || 10);
    });
    
    // Seek backward
    navigator.mediaSession.setActionHandler('seekbackward', (details) => {
      this.onSeekBackward(details.seekOffset || 10);
    });
    
    // Stop
    navigator.mediaSession.setActionHandler('stop', () => {
      this.onStop();
    });
  }
  
  // Event handler'lar
  onPlay() {
    console.log('Oynat');
    this.setPlaybackState('playing');
    // Oynatma mantığı
  }
  
  onPause() {
    console.log('Duraklat');
    this.setPlaybackState('paused');
    // Duraklatma mantığı
  }
  
  onPreviousTrack() {
    console.log('Önceki şarkı');
    // Önceki şarkı mantığı
  }
  
  onNextTrack() {
    console.log('Sonraki şarkı');
    // Sonraki şarkı mantığı
  }
  
  onSeekTo(time) {
    console.log('Seek to:', time);
    // Seek mantığı
  }
  
  onSeekForward(offset) {
    console.log('Seek forward:', offset);
    // İleri seek mantığı
  }
  
  onSeekBackward(offset) {
    console.log('Seek backward:', offset);
    // Geri seek mantığı
  }
  
  onStop() {
    console.log('Durdur');
    this.setPlaybackState('none');
    // Durdurma mantığı
  }
  
  // Medya oturumunu temizle
  clearSession() {
    if (!this.isSupported) return;
    
    navigator.mediaSession.metadata = null;
    navigator.mediaSession.playbackState = 'none';
    this.currentTrack = null;
    this.positionState = null;
    
    console.log('Medya oturumu temizlendi');
  }
}

// Müzik çalar yöneticisi
class MusicPlayerManager {
  constructor() {
    this.mediaSession = new MediaSessionManager();
    this.playlist = [];
    this.currentIndex = 0;
    this.isPlaying = false;
    this.currentTime = 0;
    this.duration = 0;
  }
  
  // Playlist ekle
  addToPlaylist(tracks) {
    this.playlist = [...this.playlist, ...tracks];
    console.log('Playlist güncellendi:', this.playlist.length, 'şarkı');
  }
  
  // Şarkı oynat
  playTrack(index) {
    if (index < 0 || index >= this.playlist.length) return;
    
    this.currentIndex = index;
    const track = this.playlist[index];
    
    // Medya bilgilerini ayarla
    this.mediaSession.setMetadata(track);
    this.mediaSession.setPlaybackState('playing');
    
    // Şarkıyı oynat
    this.isPlaying = true;
    this.duration = track.duration || 180;
    this.currentTime = 0;
    
    console.log('Şarkı oynatılıyor:', track.title);
  }
  
  // Oynatma durumunu değiştir
  togglePlayback() {
    if (this.isPlaying) {
      this.pause();
    } else {
      this.play();
    }
  }
  
  // Oynat
  play() {
    this.isPlaying = true;
    this.mediaSession.setPlaybackState('playing');
    console.log('Oynatılıyor');
  }
  
  // Duraklat
  pause() {
    this.isPlaying = false;
    this.mediaSession.setPlaybackState('paused');
    console.log('Duraklatıldı');
  }
  
  // Önceki şarkı
  previousTrack() {
    if (this.currentIndex > 0) {
      this.playTrack(this.currentIndex - 1);
    }
  }
  
  // Sonraki şarkı
  nextTrack() {
    if (this.currentIndex < this.playlist.length - 1) {
      this.playTrack(this.currentIndex + 1);
    }
  }
  
  // Pozisyon güncelle
  updatePosition(currentTime) {
    this.currentTime = currentTime;
    this.mediaSession.setPositionState(this.duration, currentTime);
  }
  
  // Mevcut şarkı bilgilerini al
  getCurrentTrack() {
    return this.playlist[this.currentIndex] || null;
  }
  
  // Playlist bilgilerini al
  getPlaylistInfo() {
    return {
      total: this.playlist.length,
      current: this.currentIndex + 1,
      isPlaying: this.isPlaying,
      currentTrack: this.getCurrentTrack()
    };
  }
}

// Video oynatıcı yöneticisi
class VideoPlayerManager {
  constructor() {
    this.mediaSession = new MediaSessionManager();
    this.videoElement = null;
    this.isPlaying = false;
  }
  
  // Video elementini bağla
  attachVideo(videoElement) {
    this.videoElement = videoElement;
    this.setupVideoListeners();
  }
  
  // Video listener'ları kur
  setupVideoListeners() {
    if (!this.videoElement) return;
    
    this.videoElement.addEventListener('play', () => {
      this.mediaSession.setPlaybackState('playing');
      this.isPlaying = true;
    });
    
    this.videoElement.addEventListener('pause', () => {
      this.mediaSession.setPlaybackState('paused');
      this.isPlaying = false;
    });
    
    this.videoElement.addEventListener('timeupdate', () => {
      this.updatePosition();
    });
    
    this.videoElement.addEventListener('loadedmetadata', () => {
      this.updatePosition();
    });
  }
  
  // Video bilgilerini ayarla
  setVideoMetadata(video) {
    this.mediaSession.setMetadata({
      title: video.title,
      artist: video.artist || 'Bilinmeyen',
      album: video.series || 'Bilinmeyen',
      artwork: video.thumbnail ? [{src: video.thumbnail, sizes: '512x512', type: 'image/jpeg'}] : undefined
    });
  }
  
  // Pozisyon güncelle
  updatePosition() {
    if (!this.videoElement) return;
    
    const duration = this.videoElement.duration;
    const currentTime = this.videoElement.currentTime;
    const playbackRate = this.videoElement.playbackRate;
    
    this.mediaSession.setPositionState(duration, currentTime, playbackRate);
  }
  
  // Video oynat
  play() {
    if (this.videoElement) {
      this.videoElement.play();
    }
  }
  
  // Video duraklat
  pause() {
    if (this.videoElement) {
      this.videoElement.pause();
    }
  }
  
  // Video seek
  seekTo(time) {
    if (this.videoElement) {
      this.videoElement.currentTime = time;
    }
  }
}
```

**Alternatifler:**
- **Web Audio API**: Ses işleme
- **HTML5 Audio/Video**: Basit medya oynatma
- **WebRTC**: Gerçek zamanlı medya
- **Media Source Extensions**: Medya akışı
- **WebCodecs API**: Medya kodlama

**Ne Zaman Kullanılmalı?**
- ✅ Müzik çalar uygulamaları gerektiğinde
- ✅ Video oynatıcı uygulamaları gerektiğinde
- ✅ Podcast uygulamaları gerektiğinde
- ✅ Medya bildirimleri gerektiğinde
- ✅ Kilit ekran kontrolleri gerektiğinde
- ✅ Sistem entegrasyonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit medya oynatma gerektiğinde
- ❌ Sistem entegrasyonu gerektiğinde
- ❌ Offline uygulamalarda
- ❌ Güvenlik kritik uygulamalarda
- ❌ Performans kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Medya Bilgileri Eksikliği**: Sistem seviyesinde medya bilgileri görünmez
- **Medya Kontrolleri Eksikliği**: Kilit ekran kontrolleri olmaz
- **Bildirim Eksikliği**: Medya bildirimleri görünmez
- **Sistem Entegrasyonu Eksikliği**: İşletim sistemi medya merkezi entegrasyonu olmaz
- **Kullanıcı Deneyimi Eksikliği**: Tutarsız medya deneyimi

**Modern Alternatifler:**
- **Web Audio API**: Ses işleme
- **Media Source Extensions**: Medya akışı
- **WebCodecs API**: Medya kodlama
- **WebRTC**: Gerçek zamanlı medya
- **WebAssembly**: Performans kritik işlemler

### Media Source Extensions (MSE) API

**Media Source Extensions (MSE) API Nedir?**
Media Source Extensions (MSE) API, web uygulamalarının JavaScript ile medya akışlarını oluşturmasını ve yönetmesini sağlayan modern bir web API'sidir. Gerçek zamanlı medya streaming, adaptive bitrate streaming, canlı yayın ve dinamik medya içerik oluşturma için kullanılır. HTML5 video ve audio elementlerini genişleterek daha gelişmiş medya işleme yetenekleri sunar.

**Neden Kullanılır?**
- **Adaptive Streaming**: Dinamik bitrate değişimi
- **Canlı Yayın**: Gerçek zamanlı medya streaming
- **Medya Birleştirme**: Farklı medya parçalarını birleştirme
- **Dinamik İçerik**: JavaScript ile medya oluşturma
- **Performans Optimizasyonu**: Verimli medya işleme
- **Özel Codec Desteği**: Farklı video/audio formatları
- **Buffer Yönetimi**: Akıllı medya tamponlama

**Temel Kullanım Örnekleri:**

```javascript
const video = document.getElementById('video');
const mediaSource = new MediaSource();
video.src = URL.createObjectURL(mediaSource);

mediaSource.addEventListener('sourceopen', () => {
  const sourceBuffer = mediaSource.addSourceBuffer('video/mp4; codecs="avc1.42E01E"');
  
  // Video verisi ekle
  fetch('video-segment.mp4')
    .then(response => response.arrayBuffer())
    .then(data => {
      sourceBuffer.appendBuffer(data);
    });
});

// Medya kaynağı durumunu kontrol et
console.log('MediaSource readyState:', mediaSource.readyState);

// Source buffer durumunu kontrol et
sourceBuffer.addEventListener('updateend', () => {
  console.log('Source buffer güncellendi');
});

// Hata yönetimi
mediaSource.addEventListener('error', (event) => {
  console.error('MediaSource hatası:', event);
});

// Medya kaynağını sonlandır
mediaSource.endOfStream();
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Medya akış yöneticisi
class MediaStreamManager {
  constructor(videoElement) {
    this.video = videoElement;
    this.mediaSource = null;
    this.sourceBuffers = new Map();
    this.isReady = false;
    this.qualityLevels = [];
    this.currentQuality = 0;
  }
  
  // Medya kaynağını başlat
  initialize() {
    this.mediaSource = new MediaSource();
    this.video.src = URL.createObjectURL(this.mediaSource);
    
    this.mediaSource.addEventListener('sourceopen', () => {
      this.isReady = true;
      console.log('MediaSource hazır');
    });
    
    this.mediaSource.addEventListener('error', (event) => {
      console.error('MediaSource hatası:', event);
    });
  }
  
  // Source buffer ekle
  addSourceBuffer(mimeType, quality = 'default') {
    if (!this.isReady) {
      throw new Error('MediaSource hazır değil');
    }
    
    const sourceBuffer = this.mediaSource.addSourceBuffer(mimeType);
    this.sourceBuffers.set(quality, sourceBuffer);
    
    sourceBuffer.addEventListener('updateend', () => {
      console.log('Source buffer güncellendi:', quality);
    });
    
    sourceBuffer.addEventListener('error', (event) => {
      console.error('Source buffer hatası:', quality, event);
    });
    
    return sourceBuffer;
  }
  
  // Medya segmenti ekle
  appendSegment(data, quality = 'default') {
    const sourceBuffer = this.sourceBuffers.get(quality);
    if (!sourceBuffer) {
      throw new Error('Source buffer bulunamadı:', quality);
    }
    
    if (sourceBuffer.updating) {
      sourceBuffer.addEventListener('updateend', () => {
        sourceBuffer.appendBuffer(data);
      }, { once: true });
    } else {
      sourceBuffer.appendBuffer(data);
    }
  }
  
  // Kalite seviyesi değiştir
  changeQuality(quality) {
    if (this.qualityLevels.includes(quality)) {
      this.currentQuality = quality;
      console.log('Kalite değiştirildi:', quality);
    }
  }
  
  // Medya kaynağını sonlandır
  endStream() {
    if (this.mediaSource && this.mediaSource.readyState === 'open') {
      this.mediaSource.endOfStream();
      console.log('Medya akışı sonlandırıldı');
    }
  }
  
  // Temizlik
  cleanup() {
    this.sourceBuffers.clear();
    if (this.mediaSource) {
      URL.revokeObjectURL(this.video.src);
    }
  }
}

// Adaptive streaming yöneticisi
class AdaptiveStreamingManager {
  constructor(videoElement) {
    this.video = videoElement;
    this.streamManager = new MediaStreamManager(videoElement);
    this.qualityLevels = [
      { quality: 'low', bitrate: 500000, resolution: '480p' },
      { quality: 'medium', bitrate: 1000000, resolution: '720p' },
      { quality: 'high', bitrate: 2000000, resolution: '1080p' }
    ];
    this.currentQuality = 'medium';
    this.bandwidth = 0;
    this.bufferHealth = 0;
    
    this.initialize();
  }
  
  // Başlat
  initialize() {
    this.streamManager.initialize();
    
    // Kalite seviyeleri için source buffer'lar oluştur
    this.qualityLevels.forEach(level => {
      this.streamManager.addSourceBuffer(
        'video/mp4; codecs="avc1.42E01E"',
        level.quality
      );
    });
    
    this.setupEventListeners();
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    this.video.addEventListener('waiting', () => {
      this.handleBufferUnderrun();
    });
    
    this.video.addEventListener('canplay', () => {
      this.handleBufferRecovery();
    });
    
    // Bant genişliği ölçümü
    setInterval(() => {
      this.measureBandwidth();
    }, 5000);
  }
  
  // Medya segmenti yükle
  async loadSegment(segmentUrl, quality = null) {
    const targetQuality = quality || this.currentQuality;
    
    try {
      const response = await fetch(segmentUrl);
      const data = await response.arrayBuffer();
      
      this.streamManager.appendSegment(data, targetQuality);
      
      console.log('Segment yüklendi:', segmentUrl, targetQuality);
    } catch (error) {
      console.error('Segment yükleme hatası:', error);
      this.handleSegmentError();
    }
  }
  
  // Bant genişliği ölç
  measureBandwidth() {
    const startTime = performance.now();
    
    fetch('/test-segment.mp4')
      .then(response => response.arrayBuffer())
      .then(data => {
        const endTime = performance.now();
        const duration = endTime - startTime;
        const size = data.byteLength;
        
        this.bandwidth = (size * 8) / (duration / 1000); // bps
        this.adjustQuality();
      });
  }
  
  // Kalite ayarla
  adjustQuality() {
    let newQuality = this.currentQuality;
    
    if (this.bandwidth < 500000) {
      newQuality = 'low';
    } else if (this.bandwidth < 1500000) {
      newQuality = 'medium';
    } else {
      newQuality = 'high';
    }
    
    if (newQuality !== this.currentQuality) {
      this.currentQuality = newQuality;
      console.log('Kalite otomatik ayarlandı:', newQuality);
    }
  }
  
  // Buffer underrun işle
  handleBufferUnderrun() {
    console.log('Buffer underrun - kalite düşürülüyor');
    this.currentQuality = 'low';
  }
  
  // Buffer recovery işle
  handleBufferRecovery() {
    console.log('Buffer recovery - kalite artırılabilir');
    this.adjustQuality();
  }
  
  // Segment hatası işle
  handleSegmentError() {
    console.log('Segment hatası - kalite düşürülüyor');
    this.currentQuality = 'low';
  }
}

// Canlı yayın yöneticisi
class LiveStreamingManager {
  constructor(videoElement) {
    this.video = videoElement;
    this.streamManager = new MediaStreamManager(videoElement);
    this.segmentQueue = [];
    this.isStreaming = false;
    this.segmentDuration = 2; // 2 saniye
    this.maxBufferSize = 10; // 10 segment
    
    this.initialize();
  }
  
  // Başlat
  initialize() {
    this.streamManager.initialize();
    this.streamManager.addSourceBuffer('video/mp4; codecs="avc1.42E01E"');
  }
  
  // Canlı yayın başlat
  startLiveStream(streamUrl) {
    this.isStreaming = true;
    this.fetchLiveSegments(streamUrl);
    console.log('Canlı yayın başlatıldı');
  }
  
  // Canlı segmentleri al
  async fetchLiveSegments(streamUrl) {
    while (this.isStreaming) {
      try {
        const segmentUrl = `${streamUrl}/segment_${Date.now()}.mp4`;
        const response = await fetch(segmentUrl);
        
        if (response.ok) {
          const data = await response.arrayBuffer();
          this.addLiveSegment(data);
        }
        
        // Segment süresi kadar bekle
        await new Promise(resolve => setTimeout(resolve, this.segmentDuration * 1000));
      } catch (error) {
        console.error('Canlı segment hatası:', error);
        await new Promise(resolve => setTimeout(resolve, 1000));
      }
    }
  }
  
  // Canlı segment ekle
  addLiveSegment(data) {
    this.segmentQueue.push(data);
    
    // Buffer boyutunu kontrol et
    if (this.segmentQueue.length > this.maxBufferSize) {
      this.segmentQueue.shift(); // En eski segmenti çıkar
    }
    
    // Segment'i ekle
    this.streamManager.appendSegment(data);
  }
  
  // Canlı yayını durdur
  stopLiveStream() {
    this.isStreaming = false;
    this.streamManager.endStream();
    console.log('Canlı yayın durduruldu');
  }
  
  // Buffer durumunu kontrol et
  getBufferStatus() {
    return {
      queueSize: this.segmentQueue.length,
      maxBufferSize: this.maxBufferSize,
      isStreaming: this.isStreaming
    };
  }
}

// Medya birleştirme yöneticisi
class MediaConcatenationManager {
  constructor(videoElement) {
    this.video = videoElement;
    this.streamManager = new MediaStreamManager(videoElement);
    this.mediaSegments = [];
    this.currentIndex = 0;
    
    this.initialize();
  }
  
  // Başlat
  initialize() {
    this.streamManager.initialize();
    this.streamManager.addSourceBuffer('video/mp4; codecs="avc1.42E01E"');
  }
  
  // Medya segmenti ekle
  addMediaSegment(segmentData) {
    this.mediaSegments.push(segmentData);
    console.log('Medya segmenti eklendi:', this.mediaSegments.length);
  }
  
  // Medya segmentlerini birleştir
  concatenateSegments() {
    if (this.mediaSegments.length === 0) {
      throw new Error('Birleştirilecek segment yok');
    }
    
    // Segmentleri sırayla ekle
    this.mediaSegments.forEach((segment, index) => {
      setTimeout(() => {
        this.streamManager.appendSegment(segment);
        console.log('Segment birleştirildi:', index + 1);
      }, index * 100);
    });
  }
  
  // Medya segmentlerini temizle
  clearSegments() {
    this.mediaSegments = [];
    this.currentIndex = 0;
    console.log('Medya segmentleri temizlendi');
  }
}
```

**Alternatifler:**
- **HTML5 Video/Audio**: Basit medya oynatma
- **WebRTC**: Gerçek zamanlı medya
- **Web Audio API**: Ses işleme
- **Canvas API**: Video işleme
- **WebCodecs API**: Medya kodlama

**Ne Zaman Kullanılmalı?**
- ✅ Adaptive streaming gerektiğinde
- ✅ Canlı yayın yapılması gerektiğinde
- ✅ Dinamik medya içerik gerektiğinde
- ✅ Özel codec desteği gerektiğinde
- ✅ Buffer yönetimi gerektiğinde
- ✅ Medya birleştirme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit medya oynatma gerektiğinde
- ❌ Statik medya dosyaları gerektiğinde
- ❌ Düşük performans gerektiğinde
- ❌ Basit uygulamalarda
- ❌ Offline uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Adaptive Streaming Eksikliği**: Dinamik kalite değişimi olmaz
- **Canlı Yayın Eksikliği**: Gerçek zamanlı streaming yapılamaz
- **Dinamik İçerik Eksikliği**: JavaScript ile medya oluşturulamaz
- **Buffer Yönetimi Eksikliği**: Akıllı tamponlama olmaz
- **Özel Codec Eksikliği**: Sınırlı format desteği

**Modern Alternatifler:**
- **WebCodecs API**: Medya kodlama
- **WebRTC**: Gerçek zamanlı medya
- **Web Audio API**: Ses işleme
- **WebAssembly**: Performans kritik işlemler
- **Web Workers**: Arka planda medya işleme

### Mutation Observer API

**Mutation Observer API Nedir?**
Mutation Observer API, web uygulamalarının DOM değişikliklerini asenkron olarak gözlemlemesini sağlayan modern bir web API'sidir. DOM elementlerinin eklenmesi, silinmesi, özellik değişiklikleri ve metin içerik değişikliklerini izler. Performanslı ve etkili DOM değişiklik takibi için kullanılır. Eski Mutation Events API'sinin yerini alan modern bir çözümdür.

**Neden Kullanılır?**
- **DOM Değişiklik Takibi**: Element ekleme, silme, değiştirme
- **Performans Optimizasyonu**: Etkili DOM izleme
- **Dinamik İçerik**: AJAX, SPA uygulamalarında değişiklik takibi
- **Üçüncü Parti Entegrasyon**: Harici kütüphanelerle uyumluluk
- **Otomatik Güncelleme**: DOM değişikliklerine otomatik tepki
- **Hata Ayıklama**: DOM değişikliklerini izleme
- **Accessibility**: Erişilebilirlik özelliklerini takip etme

**Temel Kullanım Örnekleri:**

```javascript
const observer = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    if (mutation.type === 'childList') {
      console.log('DOM yapısı değişti');
      mutation.addedNodes.forEach(node => {
        console.log('Eklenen node:', node);
      });
      mutation.removedNodes.forEach(node => {
        console.log('Silinen node:', node);
      });
    }
    
    if (mutation.type === 'attributes') {
      console.log('Özellik değişti:', mutation.attributeName);
      console.log('Eski değer:', mutation.oldValue);
      console.log('Yeni değer:', mutation.target.getAttribute(mutation.attributeName));
    }
    
    if (mutation.type === 'characterData') {
      console.log('Metin içerik değişti:', mutation.target.textContent);
    }
  });
});

// Gözlemlemeyi başlat
observer.observe(document.body, {
  childList: true,
  attributes: true,
  characterData: true,
  subtree: true,
  attributeOldValue: true,
  characterDataOldValue: true
});

// Gözlemlemeyi durdur
observer.disconnect();

// Belirli bir element'i gözlemle
const targetElement = document.getElementById('content');
observer.observe(targetElement, {
  childList: true,
  attributes: ['class', 'id'],
  subtree: false
});

// Sadece belirli değişiklikleri izle
observer.observe(document.body, {
  childList: true,
  attributes: false,
  characterData: false,
  subtree: true
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// DOM değişiklik yöneticisi
class DOMChangeManager {
  constructor() {
    this.observers = new Map();
    this.callbacks = new Map();
    this.isSupported = 'MutationObserver' in window;
  }
  
  // Gözlemci oluştur
  createObserver(name, options = {}) {
    if (!this.isSupported) {
      throw new Error('MutationObserver desteklenmiyor');
    }
    
    const defaultOptions = {
      childList: true,
      attributes: true,
      characterData: true,
      subtree: true,
      attributeOldValue: true,
      characterDataOldValue: true
    };
    
    const config = { ...defaultOptions, ...options };
    
    const observer = new MutationObserver((mutations) => {
      const callback = this.callbacks.get(name);
      if (callback) {
        callback(mutations);
      }
    });
    
    this.observers.set(name, observer);
    return observer;
  }
  
  // Callback ekle
  addCallback(name, callback) {
    this.callbacks.set(name, callback);
  }
  
  // Element gözlemle
  observe(name, element, options = {}) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.observe(element, options);
    }
  }
  
  // Gözlemlemeyi durdur
  disconnect(name) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.disconnect();
      this.observers.delete(name);
      this.callbacks.delete(name);
    }
  }
  
  // Tüm gözlemcileri durdur
  disconnectAll() {
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
    this.callbacks.clear();
  }
}

// Dinamik içerik yöneticisi
class DynamicContentManager {
  constructor() {
    this.domManager = new DOMChangeManager();
    this.contentHandlers = new Map();
    this.setupContentObserver();
  }
  
  // İçerik gözlemcisi kur
  setupContentObserver() {
    this.domManager.createObserver('content', {
      childList: true,
      subtree: true
    });
    
    this.domManager.addCallback('content', (mutations) => {
      mutations.forEach(mutation => {
        if (mutation.type === 'childList') {
          mutation.addedNodes.forEach(node => {
            if (node.nodeType === Node.ELEMENT_NODE) {
              this.handleNewElement(node);
            }
          });
        }
      });
    });
    
    // Tüm sayfayı gözlemle
    this.domManager.observe('content', document.body);
  }
  
  // Yeni element işle
  handleNewElement(element) {
    // Element türüne göre işle
    if (element.tagName === 'IMG') {
      this.handleImage(element);
    } else if (element.tagName === 'VIDEO') {
      this.handleVideo(element);
    } else if (element.classList.contains('lazy')) {
      this.handleLazyElement(element);
    }
    
    // Özel handler'ları çalıştır
    this.contentHandlers.forEach(handler => {
      if (handler.selector && element.matches(handler.selector)) {
        handler.callback(element);
      }
    });
  }
  
  // Resim işle
  handleImage(img) {
    // Lazy loading
    if (img.dataset.src) {
      img.src = img.dataset.src;
      img.removeAttribute('data-src');
    }
    
    // Hata yönetimi
    img.addEventListener('error', () => {
      img.src = '/images/placeholder.jpg';
    });
  }
  
  // Video işle
  handleVideo(video) {
    // Otomatik oynatma
    if (video.dataset.autoplay === 'true') {
      video.play();
    }
    
    // Kontroller
    if (video.dataset.controls === 'true') {
      video.controls = true;
    }
  }
  
  // Lazy element işle
  handleLazyElement(element) {
    // Intersection Observer ile lazy loading
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.loadLazyContent(entry.target);
          observer.unobserve(entry.target);
        }
      });
    });
    
    observer.observe(element);
  }
  
  // Lazy içerik yükle
  loadLazyContent(element) {
    if (element.dataset.src) {
      element.src = element.dataset.src;
      element.removeAttribute('data-src');
    }
    
    element.classList.remove('lazy');
    element.classList.add('loaded');
  }
  
  // Özel handler ekle
  addContentHandler(selector, callback) {
    this.contentHandlers.set(selector, { selector, callback });
  }
}

// Form değişiklik yöneticisi
class FormChangeManager {
  constructor() {
    this.domManager = new DOMChangeManager();
    this.formHandlers = new Map();
    this.setupFormObserver();
  }
  
  // Form gözlemcisi kur
  setupFormObserver() {
    this.domManager.createObserver('form', {
      childList: true,
      attributes: true,
      subtree: true,
      attributeOldValue: true
    });
    
    this.domManager.addCallback('form', (mutations) => {
      mutations.forEach(mutation => {
        if (mutation.type === 'childList') {
          mutation.addedNodes.forEach(node => {
            if (node.nodeType === Node.ELEMENT_NODE) {
              this.handleFormElement(node);
            }
          });
        }
        
        if (mutation.type === 'attributes') {
          this.handleAttributeChange(mutation);
        }
      });
    });
    
    // Tüm formları gözlemle
    document.querySelectorAll('form').forEach(form => {
      this.domManager.observe('form', form);
    });
  }
  
  // Form element işle
  handleFormElement(element) {
    if (element.tagName === 'INPUT') {
      this.handleInput(element);
    } else if (element.tagName === 'SELECT') {
      this.handleSelect(element);
    } else if (element.tagName === 'TEXTAREA') {
      this.handleTextarea(element);
    }
  }
  
  // Input işle
  handleInput(input) {
    // Validation
    if (input.required) {
      this.addValidation(input);
    }
    
    // Real-time validation
    input.addEventListener('input', () => {
      this.validateInput(input);
    });
  }
  
  // Select işle
  handleSelect(select) {
    // Change event
    select.addEventListener('change', () => {
      this.handleSelectChange(select);
    });
  }
  
  // Textarea işle
  handleTextarea(textarea) {
    // Auto-resize
    textarea.addEventListener('input', () => {
      this.autoResize(textarea);
    });
  }
  
  // Özellik değişikliği işle
  handleAttributeChange(mutation) {
    if (mutation.attributeName === 'disabled') {
      this.handleDisabledChange(mutation.target);
    } else if (mutation.attributeName === 'required') {
      this.handleRequiredChange(mutation.target);
    }
  }
  
  // Validation ekle
  addValidation(input) {
    input.addEventListener('blur', () => {
      this.validateInput(input);
    });
  }
  
  // Input doğrula
  validateInput(input) {
    const isValid = input.checkValidity();
    
    if (isValid) {
      input.classList.remove('invalid');
      input.classList.add('valid');
    } else {
      input.classList.remove('valid');
      input.classList.add('invalid');
    }
  }
  
  // Select değişikliği işle
  handleSelectChange(select) {
    console.log('Select değişti:', select.value);
  }
  
  // Auto-resize
  autoResize(textarea) {
    textarea.style.height = 'auto';
    textarea.style.height = textarea.scrollHeight + 'px';
  }
  
  // Disabled değişikliği işle
  handleDisabledChange(element) {
    if (element.disabled) {
      element.classList.add('disabled');
    } else {
      element.classList.remove('disabled');
    }
  }
  
  // Required değişikliği işle
  handleRequiredChange(element) {
    if (element.required) {
      element.classList.add('required');
    } else {
      element.classList.remove('required');
    }
  }
}

// Accessibility yöneticisi
class AccessibilityManager {
  constructor() {
    this.domManager = new DOMChangeManager();
    this.setupAccessibilityObserver();
  }
  
  // Erişilebilirlik gözlemcisi kur
  setupAccessibilityObserver() {
    this.domManager.createObserver('accessibility', {
      childList: true,
      attributes: true,
      subtree: true
    });
    
    this.domManager.addCallback('accessibility', (mutations) => {
      mutations.forEach(mutation => {
        if (mutation.type === 'childList') {
          mutation.addedNodes.forEach(node => {
            if (node.nodeType === Node.ELEMENT_NODE) {
              this.handleAccessibilityElement(node);
            }
          });
        }
        
        if (mutation.type === 'attributes') {
          this.handleAccessibilityAttribute(mutation);
        }
      });
    });
    
    // Tüm sayfayı gözlemle
    this.domManager.observe('accessibility', document.body);
  }
  
  // Erişilebilirlik element işle
  handleAccessibilityElement(element) {
    // ARIA attributes
    if (!element.getAttribute('aria-label') && element.title) {
      element.setAttribute('aria-label', element.title);
    }
    
    // Focus management
    if (element.tabIndex === -1) {
      element.setAttribute('aria-hidden', 'true');
    }
    
    // Role attributes
    if (element.tagName === 'BUTTON' && !element.getAttribute('role')) {
      element.setAttribute('role', 'button');
    }
  }
  
  // Erişilebilirlik özellik değişikliği işle
  handleAccessibilityAttribute(mutation) {
    if (mutation.attributeName === 'aria-hidden') {
      this.handleAriaHidden(mutation.target);
    } else if (mutation.attributeName === 'aria-expanded') {
      this.handleAriaExpanded(mutation.target);
    }
  }
  
  // ARIA hidden işle
  handleAriaHidden(element) {
    if (element.getAttribute('aria-hidden') === 'true') {
      element.setAttribute('tabindex', '-1');
    } else {
      element.removeAttribute('tabindex');
    }
  }
  
  // ARIA expanded işle
  handleAriaExpanded(element) {
    const isExpanded = element.getAttribute('aria-expanded') === 'true';
    
    if (isExpanded) {
      element.classList.add('expanded');
    } else {
      element.classList.remove('expanded');
    }
  }
}
```

**Alternatifler:**
- **Event Listeners**: Basit event dinleme
- **Intersection Observer**: Görünürlük takibi
- **Resize Observer**: Boyut değişiklik takibi
- **Performance Observer**: Performans takibi
- **Custom Events**: Özel event sistemi

**Ne Zaman Kullanılmalı?**
- ✅ DOM değişiklik takibi gerektiğinde
- ✅ Dinamik içerik uygulamalarında
- ✅ SPA uygulamalarında
- ✅ Üçüncü parti entegrasyonlarda
- ✅ Accessibility özelliklerinde
- ✅ Form validation gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit DOM işlemlerinde
- ❌ Performans kritik uygulamalarda
- ❌ Çok sık değişen DOM'larda
- ❌ Basit event listener'lar yeterli olduğunda
- ❌ Offline uygulamalarda

**Kullanılmazsa Ne Olur?**
- **DOM Değişiklik Takibi Eksikliği**: DOM değişiklikleri izlenemez
- **Dinamik İçerik Eksikliği**: AJAX içerik güncellemeleri takip edilemez
- **Performance Sorunları**: Eski Mutation Events kullanılmalı
- **Accessibility Eksikliği**: Erişilebilirlik özellikleri takip edilemez
- **Form Validation Eksikliği**: Form değişiklikleri izlenemez

**Modern Alternatifler:**
- **Intersection Observer**: Görünürlük takibi
- **Resize Observer**: Boyut değişiklik takibi
- **Performance Observer**: Performans takibi
- **Custom Events**: Özel event sistemi
- **Web Components**: Modüler DOM yapısı
```

---

## N - Bölümü

### Navigation API

**Navigation API Nedir?**
Navigation API, web uygulamalarının programatik navigasyon işlemlerini yönetmesini sağlayan modern bir web API'sidir. Sayfa geçişleri, URL değişiklikleri, geri/ileri navigasyon ve programatik sayfa yönlendirmelerini kontrol eder. SPA (Single Page Application) uygulamalarında client-side routing ve modern web uygulamalarında navigasyon yönetimi için kullanılır.

**Neden Kullanılır?**
- **Programatik Navigasyon**: JavaScript ile sayfa yönlendirme
- **SPA Routing**: Client-side routing yönetimi
- **URL Yönetimi**: URL değişikliklerini kontrol etme
- **Geri/İleri Kontrolü**: Browser geçmiş yönetimi
- **Navigasyon Takibi**: Sayfa geçişlerini izleme
- **Modern Web Apps**: PWA ve modern uygulamalarda navigasyon
- **User Experience**: Akıcı navigasyon deneyimi

**Temel Kullanım Örnekleri:**

```javascript
if ('navigation' in window) {
  // Navigasyon olaylarını dinle
  navigation.addEventListener('navigate', event => {
    console.log('Navigasyon olayı:', event.destination.url);
    console.log('Navigasyon tipi:', event.navigationType);
    console.log('Form submission:', event.formData);
  });
  
  // Navigasyon başlat
  navigation.navigate('/new-page', {
    state: { data: 'example' },
    replace: false
  });
  
  // Geri navigasyon
  navigation.back();
  
  // İleri navigasyon
  navigation.forward();
  
  // Belirli bir sayfaya git
  navigation.navigate('/specific-page');
  
  // Mevcut sayfayı değiştir
  navigation.navigate('/replacement-page', {
    replace: true
  });
  
  // Navigasyon durumunu kontrol et
  console.log('Can go back:', navigation.canGoBack);
  console.log('Can go forward:', navigation.canGoForward);
  
  // Navigasyon geçmişini al
  console.log('Navigation entries:', navigation.entries());
}

// Navigasyon olay türleri
navigation.addEventListener('navigate', event => {
  switch (event.navigationType) {
    case 'push':
      console.log('Yeni sayfa eklendi');
      break;
    case 'replace':
      console.log('Mevcut sayfa değiştirildi');
      break;
    case 'reload':
      console.log('Sayfa yenilendi');
      break;
    case 'traverse':
      console.log('Geçmiş navigasyonu');
      break;
  }
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Navigasyon yöneticisi
class NavigationManager {
  constructor() {
    this.isSupported = 'navigation' in window;
    this.routes = new Map();
    this.middleware = [];
    this.currentRoute = null;
    
    if (this.isSupported) {
      this.setupNavigation();
    }
  }
  
  // Navigasyon kurulumu
  setupNavigation() {
    navigation.addEventListener('navigate', (event) => {
      this.handleNavigation(event);
    });
    
    // Sayfa yüklendiğinde mevcut route'u ayarla
    this.currentRoute = window.location.pathname;
  }
  
  // Route ekle
  addRoute(path, handler, options = {}) {
    this.routes.set(path, {
      handler: handler,
      options: options,
      middleware: options.middleware || []
    });
  }
  
  // Middleware ekle
  addMiddleware(middleware) {
    this.middleware.push(middleware);
  }
  
  // Navigasyon işle
  async handleNavigation(event) {
    const url = new URL(event.destination.url);
    const path = url.pathname;
    
    // Middleware'leri çalıştır
    for (const middleware of this.middleware) {
      const result = await middleware(event);
      if (result === false) {
        event.preventDefault();
        return;
      }
    }
    
    // Route handler'ı bul
    const route = this.findRoute(path);
    
    if (route) {
      // Route middleware'lerini çalıştır
      for (const middleware of route.middleware) {
        const result = await middleware(event);
        if (result === false) {
          event.preventDefault();
          return;
        }
      }
      
      // Route handler'ı çalıştır
      await route.handler(event);
      this.currentRoute = path;
    } else {
      console.warn('Route bulunamadı:', path);
    }
  }
  
  // Route bul
  findRoute(path) {
    // Exact match
    if (this.routes.has(path)) {
      return this.routes.get(path);
    }
    
    // Pattern match
    for (const [routePath, route] of this.routes) {
      if (this.matchRoute(routePath, path)) {
        return route;
      }
    }
    
    return null;
  }
  
  // Route pattern eşleştirme
  matchRoute(pattern, path) {
    const patternParts = pattern.split('/');
    const pathParts = path.split('/');
    
    if (patternParts.length !== pathParts.length) {
      return false;
    }
    
    for (let i = 0; i < patternParts.length; i++) {
      const patternPart = patternParts[i];
      const pathPart = pathParts[i];
      
      if (patternPart.startsWith(':')) {
        // Parameter
        continue;
      } else if (patternPart !== pathPart) {
        return false;
      }
    }
    
    return true;
  }
  
  // Programatik navigasyon
  navigate(path, options = {}) {
    if (!this.isSupported) {
      window.location.href = path;
      return;
    }
    
    navigation.navigate(path, options);
  }
  
  // Geri git
  back() {
    if (this.isSupported && navigation.canGoBack) {
      navigation.back();
    } else {
      window.history.back();
    }
  }
  
  // İleri git
  forward() {
    if (this.isSupported && navigation.canGoForward) {
      navigation.forward();
    } else {
      window.history.forward();
    }
  }
  
  // Mevcut route'u al
  getCurrentRoute() {
    return this.currentRoute;
  }
  
  // Route parametrelerini al
  getRouteParams(pattern, path) {
    const patternParts = pattern.split('/');
    const pathParts = path.split('/');
    const params = {};
    
    for (let i = 0; i < patternParts.length; i++) {
      const patternPart = patternParts[i];
      const pathPart = pathParts[i];
      
      if (patternPart.startsWith(':')) {
        const paramName = patternPart.slice(1);
        params[paramName] = pathPart;
      }
    }
    
    return params;
  }
}

// SPA Router
class SPARouter {
  constructor() {
    this.navigationManager = new NavigationManager();
    this.components = new Map();
    this.layouts = new Map();
    this.currentComponent = null;
    this.setupRouter();
  }
  
  // Router kurulumu
  setupRouter() {
    // Layout middleware
    this.navigationManager.addMiddleware(async (event) => {
      const url = new URL(event.destination.url);
      const layout = this.getLayout(url.pathname);
      
      if (layout) {
        await this.loadLayout(layout);
      }
      
      return true;
    });
    
    // Component middleware
    this.navigationManager.addMiddleware(async (event) => {
      const url = new URL(event.destination.url);
      const component = this.getComponent(url.pathname);
      
      if (component) {
        await this.loadComponent(component, url);
      }
      
      return true;
    });
  }
  
  // Route tanımla
  route(path, component, options = {}) {
    this.navigationManager.addRoute(path, async (event) => {
      await this.renderComponent(component, event);
    }, options);
    
    this.components.set(path, component);
  }
  
  // Layout tanımla
  layout(path, layout) {
    this.layouts.set(path, layout);
  }
  
  // Layout al
  getLayout(path) {
    for (const [layoutPath, layout] of this.layouts) {
      if (path.startsWith(layoutPath)) {
        return layout;
      }
    }
    return null;
  }
  
  // Component al
  getComponent(path) {
    return this.components.get(path);
  }
  
  // Layout yükle
  async loadLayout(layout) {
    const layoutElement = document.getElementById('layout');
    if (layoutElement) {
      layoutElement.innerHTML = await layout.render();
    }
  }
  
  // Component yükle
  async loadComponent(component, url) {
    const contentElement = document.getElementById('content');
    if (contentElement) {
      const params = this.navigationManager.getRouteParams(
        this.getRoutePattern(url.pathname),
        url.pathname
      );
      
      contentElement.innerHTML = await component.render(params);
      this.currentComponent = component;
    }
  }
  
  // Component render et
  async renderComponent(component, event) {
    const url = new URL(event.destination.url);
    const params = this.navigationManager.getRouteParams(
      this.getRoutePattern(url.pathname),
      url.pathname
    );
    
    // Component lifecycle
    if (this.currentComponent && this.currentComponent.destroy) {
      await this.currentComponent.destroy();
    }
    
    if (component.init) {
      await component.init(params);
    }
    
    if (component.render) {
      const content = await component.render(params);
      document.getElementById('content').innerHTML = content;
    }
    
    this.currentComponent = component;
  }
  
  // Route pattern al
  getRoutePattern(path) {
    for (const routePath of this.components.keys()) {
      if (this.navigationManager.matchRoute(routePath, path)) {
        return routePath;
      }
    }
    return path;
  }
  
  // Navigasyon başlat
  start() {
    // İlk sayfa yükleme
    const currentPath = window.location.pathname;
    this.navigationManager.navigate(currentPath);
  }
}

// Navigasyon guard'ları
class NavigationGuards {
  constructor() {
    this.beforeGuards = [];
    this.afterGuards = [];
  }
  
  // Before guard ekle
  beforeEach(guard) {
    this.beforeGuards.push(guard);
  }
  
  // After guard ekle
  afterEach(guard) {
    this.afterGuards.push(guard);
  }
  
  // Before guard'ları çalıştır
  async runBeforeGuards(to, from) {
    for (const guard of this.beforeGuards) {
      const result = await guard(to, from);
      if (result === false) {
        return false;
      }
    }
    return true;
  }
  
  // After guard'ları çalıştır
  async runAfterGuards(to, from) {
    for (const guard of this.afterGuards) {
      await guard(to, from);
    }
  }
}

// Navigasyon geçmişi yöneticisi
class NavigationHistoryManager {
  constructor() {
    this.history = [];
    this.maxHistorySize = 50;
    this.setupHistoryTracking();
  }
  
  // Geçmiş takibi kur
  setupHistoryTracking() {
    if ('navigation' in window) {
      navigation.addEventListener('navigate', (event) => {
        this.addToHistory(event);
      });
    }
  }
  
  // Geçmişe ekle
  addToHistory(event) {
    const historyItem = {
      url: event.destination.url,
      timestamp: Date.now(),
      type: event.navigationType,
      state: event.destination.state
    };
    
    this.history.push(historyItem);
    
    // Geçmiş boyutunu sınırla
    if (this.history.length > this.maxHistorySize) {
      this.history.shift();
    }
  }
  
  // Geçmişi al
  getHistory() {
    return [...this.history];
  }
  
  // Geçmişi temizle
  clearHistory() {
    this.history = [];
  }
  
  // Belirli bir URL'yi geçmişte ara
  findInHistory(url) {
    return this.history.find(item => item.url === url);
  }
}
```

**Alternatifler:**
- **History API**: Browser geçmiş yönetimi
- **Location API**: URL yönetimi
- **Hash Routing**: Hash tabanlı routing
- **Custom Router**: Özel routing çözümleri
- **Framework Routers**: React Router, Vue Router

**Ne Zaman Kullanılmalı?**
- ✅ SPA uygulamaları gerektiğinde
- ✅ Client-side routing gerektiğinde
- ✅ Programatik navigasyon gerektiğinde
- ✅ Modern web uygulamalarında
- ✅ PWA uygulamalarında
- ✅ Dinamik sayfa yönlendirme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit web sitelerinde
- ❌ Server-side rendering gerektiğinde
- ❌ SEO kritik uygulamalarda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Statik sayfa yapılarında

**Kullanılmazsa Ne Olur?**
- **SPA Routing Eksikliği**: Client-side routing yapılamaz
- **Programatik Navigasyon Eksikliği**: JavaScript ile sayfa yönlendirme olmaz
- **Modern UX Eksikliği**: Akıcı navigasyon deneyimi olmaz
- **PWA Entegrasyonu Eksikliği**: Modern web app özellikleri eksik kalır
- **Dinamik İçerik Eksikliği**: Dinamik sayfa yükleme olmaz

**Modern Alternatifler:**
- **History API**: Browser geçmiş yönetimi
- **Location API**: URL yönetimi
- **Custom Router**: Özel routing çözümleri
- **Framework Routers**: React Router, Vue Router
- **Web Components**: Modüler routing

### Network Information API

**Network Information API Nedir?**
Network Information API, web uygulamalarının kullanıcının ağ bağlantısı hakkında bilgi almasını sağlayan modern bir web API'sidir. Bağlantı türü, hız, gecikme ve maliyet bilgilerini sağlar. Uygulamaların ağ durumuna göre davranışlarını optimize etmesini ve kullanıcı deneyimini iyileştirmesini sağlar. Özellikle mobil uygulamalarda ve düşük bant genişliği durumlarında önemlidir.

**Neden Kullanılır?**
- **Ağ Durumu Takibi**: Bağlantı türü ve kalitesi izleme
- **Performans Optimizasyonu**: Ağ durumuna göre içerik ayarlama
- **Kullanıcı Deneyimi**: Ağ durumuna uygun deneyim sunma
- **Veri Tasarrufu**: Düşük bant genişliğinde veri kullanımını azaltma
- **Offline Desteği**: Bağlantı durumuna göre offline mod
- **Adaptive Loading**: Ağ durumuna göre içerik yükleme
- **Maliyet Optimizasyonu**: Pahalı bağlantılarda veri kullanımını sınırlama

**Temel Kullanım Örnekleri:**

```javascript
if ('connection' in navigator) {
  const connection = navigator.connection;
  
  // Temel ağ bilgileri
  console.log(`Bağlantı tipi: ${connection.effectiveType}`);
  console.log(`İndirme hızı: ${connection.downlink} Mbps`);
  console.log(`Gecikme: ${connection.rtt} ms`);
  console.log(`Maliyet: ${connection.saveData ? 'Düşük' : 'Normal'}`);
  
  // Ağ durumu değişikliklerini dinle
  connection.addEventListener('change', () => {
    console.log('Ağ durumu değişti');
    console.log('Yeni bağlantı tipi:', connection.effectiveType);
    console.log('Yeni indirme hızı:', connection.downlink);
    console.log('Yeni gecikme:', connection.rtt);
  });
  
  // Ağ durumuna göre davranış
  if (connection.effectiveType === 'slow-2g' || connection.effectiveType === '2g') {
    console.log('Yavaş bağlantı - düşük kalite içerik yükle');
  } else if (connection.effectiveType === '3g') {
    console.log('Orta bağlantı - orta kalite içerik yükle');
  } else if (connection.effectiveType === '4g') {
    console.log('Hızlı bağlantı - yüksek kalite içerik yükle');
  }
  
  // Veri tasarrufu kontrolü
  if (connection.saveData) {
    console.log('Veri tasarrufu modu aktif - sıkıştırılmış içerik yükle');
  }
  
  // Bağlantı türü kontrolü
  if (connection.type === 'cellular') {
    console.log('Mobil bağlantı - veri kullanımını sınırla');
  } else if (connection.type === 'wifi') {
    console.log('WiFi bağlantı - tam kalite içerik yükle');
  }
}
```

**Alternatifler:**
- **Online/Offline Events**: Basit bağlantı durumu
- **Performance API**: Ağ performans ölçümü
- **Service Workers**: Offline desteği
- **Custom Network Detection**: Özel ağ algılama
- **Third-party Libraries**: Harici ağ kütüphaneleri

**Ne Zaman Kullanılmalı?**
- ✅ Mobil uygulamalarda
- ✅ Düşük bant genişliği durumlarında
- ✅ Veri tasarrufu gerektiğinde
- ✅ Adaptive içerik gerektiğinde
- ✅ Offline desteği gerektiğinde
- ✅ Performans optimizasyonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit web sitelerinde
- ❌ Sadece desktop uygulamalarda
- ❌ Sabit ağ bağlantılarında
- ❌ Offline uygulamalarda
- ❌ Eski tarayıcı desteği gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Adaptive Loading Eksikliği**: Ağ durumuna göre içerik ayarlanamaz
- **Veri Tasarrufu Eksikliği**: Gereksiz veri kullanımı
- **Kullanıcı Deneyimi Sorunları**: Yavaş bağlantılarda kötü deneyim
- **Offline Desteği Eksikliği**: Bağlantı kesildiğinde uygulama çalışmaz
- **Performans Sorunları**: Ağ durumuna göre optimizasyon yapılamaz

**Modern Alternatifler:**
- **Service Workers**: Offline desteği
- **Performance API**: Ağ performans ölçümü
- **Web Workers**: Arka planda ağ işlemleri
- **Custom Network Detection**: Özel ağ algılama
- **Third-party Libraries**: Harici ağ kütüphaneleri

### Notifications API

**Notifications API Nedir?**
Notifications API, web uygulamalarının kullanıcıya sistem seviyesinde bildirimler göndermesini sağlayan modern bir web API'sidir. Tarayıcı dışında, işletim sistemi bildirim sistemi üzerinden kullanıcıya mesajlar gönderir. PWA uygulamalarında, gerçek zamanlı uygulamalarda ve kullanıcı etkileşimi gerektiren durumlarda kullanılır. Kullanıcı izni tabanlı güvenli bir bildirim sistemi sunar.

**Neden Kullanılır?**
- **Sistem Bildirimleri**: Tarayıcı dışında bildirim gönderme
- **Kullanıcı Etkileşimi**: Önemli olaylar için kullanıcıyı bilgilendirme
- **PWA Desteği**: Progressive Web App özellikleri
- **Gerçek Zamanlı Uygulamalar**: Anlık bildirimler
- **Offline Bildirimler**: Uygulama kapalıyken bildirim
- **Kullanıcı Deneyimi**: Etkileşimli bildirim deneyimi
- **Engagement**: Kullanıcı katılımını artırma

**Temel Kullanım Örnekleri:**

```javascript
// Bildirim izni iste
if ('Notification' in window) {
  const permission = await Notification.requestPermission();
  
  if (permission === 'granted') {
    new Notification('Başlık', {
      body: 'Bildirim içeriği',
      icon: 'icon.png',
      badge: 'badge.png',
      tag: 'unique-tag',
      requireInteraction: true,
      silent: false,
      vibrate: [200, 100, 200]
    });
  }
}

// Bildirim izni kontrolü
if (Notification.permission === 'granted') {
  console.log('Bildirim izni verilmiş');
} else if (Notification.permission === 'denied') {
  console.log('Bildirim izni reddedilmiş');
} else {
  console.log('Bildirim izni henüz istenmemiş');
}

// Bildirim olaylarını dinle
const notification = new Notification('Başlık', {
  body: 'Bildirim içeriği',
  icon: 'icon.png'
});

notification.onclick = () => {
  console.log('Bildirime tıklandı');
  window.focus();
  notification.close();
};

notification.onshow = () => {
  console.log('Bildirim gösterildi');
};

notification.onclose = () => {
  console.log('Bildirim kapatıldı');
};

notification.onerror = () => {
  console.log('Bildirim hatası');
};

// Bildirim kapatma
notification.close();

// Bildirim ayarları
const notification = new Notification('Başlık', {
  body: 'Bildirim içeriği',
  icon: 'icon.png',
  badge: 'badge.png',
  tag: 'unique-tag',
  data: { url: '/specific-page' },
  actions: [
    {
      action: 'view',
      title: 'Görüntüle',
      icon: 'view-icon.png'
    },
    {
      action: 'dismiss',
      title: 'Kapat',
      icon: 'dismiss-icon.png'
    }
  ],
  requireInteraction: true,
  silent: false,
  vibrate: [200, 100, 200],
  timestamp: Date.now()
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Bildirim yöneticisi
class NotificationManager {
  constructor() {
    this.isSupported = 'Notification' in window;
    this.permission = this.isSupported ? Notification.permission : 'denied';
    this.notifications = new Map();
    this.setupEventListeners();
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    if (this.isSupported) {
      // Service Worker ile bildirim olaylarını dinle
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.addEventListener('message', (event) => {
          this.handleServiceWorkerMessage(event);
        });
      }
    }
  }
  
  // İzin iste
  async requestPermission() {
    if (!this.isSupported) {
      throw new Error('Notifications API desteklenmiyor');
    }
    
    if (this.permission === 'granted') {
      return true;
    }
    
    this.permission = await Notification.requestPermission();
    return this.permission === 'granted';
  }
  
  // Bildirim gönder
  async sendNotification(title, options = {}) {
    if (!this.isSupported) {
      throw new Error('Notifications API desteklenmiyor');
    }
    
    if (this.permission !== 'granted') {
      const granted = await this.requestPermission();
      if (!granted) {
        throw new Error('Bildirim izni verilmedi');
      }
    }
    
    const defaultOptions = {
      body: '',
      icon: '/images/icon-192.png',
      badge: '/images/badge-72.png',
      tag: `notification-${Date.now()}`,
      requireInteraction: false,
      silent: false,
      vibrate: [200, 100, 200],
      timestamp: Date.now()
    };
    
    const config = { ...defaultOptions, ...options };
    
    const notification = new Notification(title, config);
    
    // Bildirim olaylarını dinle
    this.setupNotificationEvents(notification);
    
    // Bildirimi kaydet
    this.notifications.set(config.tag, notification);
    
    return notification;
  }
  
  // Bildirim olaylarını kur
  setupNotificationEvents(notification) {
    notification.onclick = (event) => {
      this.handleNotificationClick(event);
    };
    
    notification.onshow = (event) => {
      this.handleNotificationShow(event);
    };
    
    notification.onclose = (event) => {
      this.handleNotificationClose(event);
    };
    
    notification.onerror = (event) => {
      this.handleNotificationError(event);
    };
  }
  
  // Bildirim tıklama işle
  handleNotificationClick(event) {
    const notification = event.target;
    
    // Bildirim verilerini kontrol et
    if (notification.data && notification.data.url) {
      window.open(notification.data.url, '_blank');
    }
    
    // Uygulamayı odakla
    window.focus();
    
    // Bildirimi kapat
    notification.close();
    
    console.log('Bildirime tıklandı:', notification.tag);
  }
  
  // Bildirim gösterim işle
  handleNotificationShow(event) {
    const notification = event.target;
    console.log('Bildirim gösterildi:', notification.tag);
  }
  
  // Bildirim kapatma işle
  handleNotificationClose(event) {
    const notification = event.target;
    this.notifications.delete(notification.tag);
    console.log('Bildirim kapatıldı:', notification.tag);
  }
  
  // Bildirim hata işle
  handleNotificationError(event) {
    const notification = event.target;
    console.error('Bildirim hatası:', notification.tag, event);
  }
  
  // Service Worker mesaj işle
  handleServiceWorkerMessage(event) {
    if (event.data.type === 'NOTIFICATION_CLICK') {
      this.handleServiceWorkerNotificationClick(event.data);
    }
  }
  
  // Service Worker bildirim tıklama işle
  handleServiceWorkerNotificationClick(data) {
    if (data.url) {
      window.open(data.url, '_blank');
    }
    
    window.focus();
  }
  
  // Bildirim kapat
  closeNotification(tag) {
    const notification = this.notifications.get(tag);
    if (notification) {
      notification.close();
      this.notifications.delete(tag);
    }
  }
  
  // Tüm bildirimleri kapat
  closeAllNotifications() {
    this.notifications.forEach(notification => {
      notification.close();
    });
    this.notifications.clear();
  }
  
  // Bildirim durumunu kontrol et
  isPermissionGranted() {
    return this.permission === 'granted';
  }
  
  // Aktif bildirim sayısını al
  getActiveNotificationCount() {
    return this.notifications.size;
  }
}

// Bildirim şablonları yöneticisi
class NotificationTemplateManager {
  constructor() {
    this.templates = new Map();
    this.setupDefaultTemplates();
  }
  
  // Varsayılan şablonları kur
  setupDefaultTemplates() {
    // Başarı bildirimi
    this.templates.set('success', {
      icon: '/images/success-icon.png',
      badge: '/images/success-badge.png',
      vibrate: [200, 100, 200],
      requireInteraction: false,
      silent: false
    });
    
    // Hata bildirimi
    this.templates.set('error', {
      icon: '/images/error-icon.png',
      badge: '/images/error-badge.png',
      vibrate: [500, 200, 500],
      requireInteraction: true,
      silent: false
    });
    
    // Uyarı bildirimi
    this.templates.set('warning', {
      icon: '/images/warning-icon.png',
      badge: '/images/warning-badge.png',
      vibrate: [300, 100, 300],
      requireInteraction: false,
      silent: false
    });
    
    // Bilgi bildirimi
    this.templates.set('info', {
      icon: '/images/info-icon.png',
      badge: '/images/info-badge.png',
      vibrate: [200, 100, 200],
      requireInteraction: false,
      silent: false
    });
  }
  
  // Şablon ekle
  addTemplate(name, template) {
    this.templates.set(name, template);
  }
  
  // Şablon al
  getTemplate(name) {
    return this.templates.get(name);
  }
  
  // Şablon ile bildirim gönder
  async sendTemplateNotification(templateName, title, body, options = {}) {
    const template = this.getTemplate(templateName);
    if (!template) {
      throw new Error(`Şablon bulunamadı: ${templateName}`);
    }
    
    const notificationOptions = {
      ...template,
      body: body,
      ...options
    };
    
    const notificationManager = new NotificationManager();
    return await notificationManager.sendNotification(title, notificationOptions);
  }
}

// Zamanlanmış bildirim yöneticisi
class ScheduledNotificationManager {
  constructor() {
    this.scheduledNotifications = new Map();
    this.notificationManager = new NotificationManager();
  }
  
  // Zamanlanmış bildirim ekle
  scheduleNotification(id, title, options, delay) {
    const timeoutId = setTimeout(async () => {
      try {
        await this.notificationManager.sendNotification(title, options);
        this.scheduledNotifications.delete(id);
      } catch (error) {
        console.error('Zamanlanmış bildirim hatası:', error);
      }
    }, delay);
    
    this.scheduledNotifications.set(id, {
      timeoutId: timeoutId,
      title: title,
      options: options,
      scheduledTime: Date.now() + delay
    });
    
    console.log('Bildirim zamanlandı:', id, delay + 'ms sonra');
  }
  
  // Zamanlanmış bildirimi iptal et
  cancelScheduledNotification(id) {
    const scheduled = this.scheduledNotifications.get(id);
    if (scheduled) {
      clearTimeout(scheduled.timeoutId);
      this.scheduledNotifications.delete(id);
      console.log('Zamanlanmış bildirim iptal edildi:', id);
    }
  }
  
  // Tüm zamanlanmış bildirimleri iptal et
  cancelAllScheduledNotifications() {
    this.scheduledNotifications.forEach(scheduled => {
      clearTimeout(scheduled.timeoutId);
    });
    this.scheduledNotifications.clear();
    console.log('Tüm zamanlanmış bildirimler iptal edildi');
  }
  
  // Zamanlanmış bildirimleri listele
  getScheduledNotifications() {
    return Array.from(this.scheduledNotifications.entries()).map(([id, scheduled]) => ({
      id: id,
      title: scheduled.title,
      scheduledTime: scheduled.scheduledTime,
      remainingTime: scheduled.scheduledTime - Date.now()
    }));
  }
}

// Bildirim istatistikleri yöneticisi
class NotificationAnalyticsManager {
  constructor() {
    this.stats = {
      sent: 0,
      clicked: 0,
      closed: 0,
      errors: 0,
      permissionGranted: 0,
      permissionDenied: 0
    };
    
    this.notificationManager = new NotificationManager();
    this.setupAnalytics();
  }
  
  // Analitik kurulumu
  setupAnalytics() {
    // Bildirim olaylarını dinle
    this.notificationManager.onclick = () => {
      this.stats.clicked++;
    };
    
    this.notificationManager.onclose = () => {
      this.stats.closed++;
    };
    
    this.notificationManager.onerror = () => {
      this.stats.errors++;
    };
  }
  
  // Bildirim gönderildi
  onNotificationSent() {
    this.stats.sent++;
  }
  
  // İzin verildi
  onPermissionGranted() {
    this.stats.permissionGranted++;
  }
  
  // İzin reddedildi
  onPermissionDenied() {
    this.stats.permissionDenied++;
  }
  
  // İstatistikleri al
  getStats() {
    return { ...this.stats };
  }
  
  // İstatistikleri sıfırla
  resetStats() {
    this.stats = {
      sent: 0,
      clicked: 0,
      closed: 0,
      errors: 0,
      permissionGranted: 0,
      permissionDenied: 0
    };
  }
  
  // Tıklama oranını hesapla
  getClickRate() {
    return this.stats.sent > 0 ? (this.stats.clicked / this.stats.sent) * 100 : 0;
  }
  
  // Kapatma oranını hesapla
  getCloseRate() {
    return this.stats.sent > 0 ? (this.stats.closed / this.stats.sent) * 100 : 0;
  }
  
  // Hata oranını hesapla
  getErrorRate() {
    return this.stats.sent > 0 ? (this.stats.errors / this.stats.sent) * 100 : 0;
  }
}
```

**Alternatifler:**
- **Alert/Confirm**: Basit tarayıcı bildirimleri
- **Toast Notifications**: Sayfa içi bildirimler
- **Push API**: Server'dan bildirim gönderme
- **Service Workers**: Arka plan bildirimleri
- **Custom Notification Systems**: Özel bildirim sistemleri

**Ne Zaman Kullanılmalı?**
- ✅ PWA uygulamalarında
- ✅ Gerçek zamanlı uygulamalarda
- ✅ Kullanıcı etkileşimi gerektiğinde
- ✅ Offline bildirim gerektiğinde
- ✅ Sistem seviyesinde bildirim gerektiğinde
- ✅ Engagement artırma gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Spam bildirimler için
- ❌ Gereksiz bildirimler için
- ❌ Kullanıcı izni olmadan
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Sistem Bildirimleri Eksikliği**: Tarayıcı dışında bildirim gönderilemez
- **PWA Özellikleri Eksikliği**: Modern web app özellikleri eksik kalır
- **Kullanıcı Etkileşimi Eksikliği**: Önemli olaylar bildirilemez
- **Offline Bildirim Eksikliği**: Uygulama kapalıyken bildirim olmaz
- **Engagement Eksikliği**: Kullanıcı katılımı azalır

**Modern Alternatifler:**
- **Push API**: Server'dan bildirim gönderme
- **Service Workers**: Arka plan bildirimleri
- **Web Push Protocol**: Standart push bildirim protokolü
- **Custom Notification Systems**: Özel bildirim sistemleri
- **Third-party Services**: Harici bildirim servisleri

---

## P - Bölümü

### Page Visibility API

**Page Visibility API Nedir?**
Page Visibility API, web uygulamalarının sayfa görünürlük durumunu algılamasını sağlayan modern bir web API'sidir. Kullanıcının sayfayı başka bir sekmeye geçirdiğinde, tarayıcıyı minimize ettiğinde veya başka bir uygulamaya geçtiğinde bu durumu algılar. Performans optimizasyonu, kaynak yönetimi ve kullanıcı deneyimi iyileştirmesi için kritik öneme sahiptir. Sayfa gizli olduğunda gereksiz işlemleri durdurur, görünür olduğunda devam ettirir.

**Neden Kullanılır?**
- **Performans Optimizasyonu**: Gizli sayfalarda gereksiz işlemleri durdurma
- **Kaynak Yönetimi**: CPU, bellek ve ağ kaynaklarını koruma
- **Battery Life**: Pil ömrünü uzatma
- **User Experience**: Kullanıcı deneyimini iyileştirme
- **Analytics**: Sayfa görünürlük istatistikleri
- **Media Control**: Video/audio oynatmayı kontrol etme
- **Background Tasks**: Arka plan görevlerini yönetme

**Temel Kullanım Örnekleri:**

```javascript
// Sayfa görünürlük değişikliklerini dinle
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    console.log('Sayfa gizli');
    // Gereksiz işlemleri durdur
    pauseAnimations();
    stopVideo();
    clearInterval(timer);
  } else {
    console.log('Sayfa görünür');
    // İşlemleri devam ettir
    resumeAnimations();
    playVideo();
    startTimer();
  }
});

// Sayfa görünürlük durumunu kontrol et
if (document.hidden) {
  console.log('Sayfa şu anda gizli');
} else {
  console.log('Sayfa şu anda görünür');
}

// Visibility state kontrolü
if (document.visibilityState === 'visible') {
  console.log('Sayfa tamamen görünür');
} else if (document.visibilityState === 'hidden') {
  console.log('Sayfa gizli');
} else if (document.visibilityState === 'prerender') {
  console.log('Sayfa önceden render ediliyor');
}

// Sayfa görünürlük durumu değişikliklerini takip et
let visibilityChangeCount = 0;
document.addEventListener('visibilitychange', () => {
  visibilityChangeCount++;
  console.log(`Görünürlük değişikliği #${visibilityChangeCount}`);
  console.log('Yeni durum:', document.visibilityState);
});

// Sayfa görünürlük süresini hesapla
let visibilityStartTime = Date.now();
let totalVisibleTime = 0;

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Sayfa gizlendi, görünür kalma süresini hesapla
    totalVisibleTime += Date.now() - visibilityStartTime;
    console.log('Toplam görünür kalma süresi:', totalVisibleTime + 'ms');
  } else {
    // Sayfa görünür oldu, başlangıç zamanını kaydet
    visibilityStartTime = Date.now();
  }
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Sayfa görünürlük yöneticisi
class PageVisibilityManager {
  constructor() {
    this.isSupported = 'visibilityState' in document;
    this.visibilityState = this.isSupported ? document.visibilityState : 'visible';
    this.listeners = new Map();
    this.timers = new Map();
    this.animations = new Map();
    this.mediaElements = new Map();
    this.visibilityStartTime = Date.now();
    this.totalVisibleTime = 0;
    this.visibilityChanges = 0;
    
    if (this.isSupported) {
      this.setupEventListeners();
    }
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    document.addEventListener('visibilitychange', () => {
      this.handleVisibilityChange();
    });
  }
  
  // Görünürlük değişikliğini işle
  handleVisibilityChange() {
    const newState = document.visibilityState;
    const oldState = this.visibilityState;
    
    this.visibilityState = newState;
    this.visibilityChanges++;
    
    if (newState === 'hidden') {
      this.onPageHidden();
    } else if (newState === 'visible') {
      this.onPageVisible();
    }
    
    // Listener'ları çağır
    this.notifyListeners(newState, oldState);
    
    console.log(`Görünürlük değişti: ${oldState} -> ${newState}`);
  }
  
  // Sayfa gizlendiğinde
  onPageHidden() {
    // Görünür kalma süresini hesapla
    this.totalVisibleTime += Date.now() - this.visibilityStartTime;
    
    // Timer'ları durdur
    this.pauseTimers();
    
    // Animasyonları durdur
    this.pauseAnimations();
    
    // Medya elementlerini durdur
    this.pauseMediaElements();
    
    // Arka plan görevlerini başlat
    this.startBackgroundTasks();
  }
  
  // Sayfa görünür olduğunda
  onPageVisible() {
    // Başlangıç zamanını kaydet
    this.visibilityStartTime = Date.now();
    
    // Timer'ları devam ettir
    this.resumeTimers();
    
    // Animasyonları devam ettir
    this.resumeAnimations();
    
    // Medya elementlerini devam ettir
    this.resumeMediaElements();
    
    // Arka plan görevlerini durdur
    this.stopBackgroundTasks();
  }
  
  // Timer yönetimi
  addTimer(name, callback, interval) {
    const timer = setInterval(callback, interval);
    this.timers.set(name, timer);
    return timer;
  }
  
  removeTimer(name) {
    const timer = this.timers.get(name);
    if (timer) {
      clearInterval(timer);
      this.timers.delete(name);
    }
  }
  
  pauseTimers() {
    this.timers.forEach((timer, name) => {
      clearInterval(timer);
      console.log('Timer durduruldu:', name);
    });
  }
  
  resumeTimers() {
    // Timer'ları yeniden başlat (bu örnekte basit)
    console.log('Timer\'lar devam ettirildi');
  }
  
  // Animasyon yönetimi
  addAnimation(name, element, animation) {
    this.animations.set(name, {
      element: element,
      animation: animation,
      isPaused: false
    });
  }
  
  pauseAnimations() {
    this.animations.forEach((anim, name) => {
      if (!anim.isPaused) {
        anim.element.style.animationPlayState = 'paused';
        anim.isPaused = true;
        console.log('Animasyon durduruldu:', name);
      }
    });
  }
  
  resumeAnimations() {
    this.animations.forEach((anim, name) => {
      if (anim.isPaused) {
        anim.element.style.animationPlayState = 'running';
        anim.isPaused = false;
        console.log('Animasyon devam ettirildi:', name);
      }
    });
  }
  
  // Medya element yönetimi
  addMediaElement(name, element) {
    this.mediaElements.set(name, {
      element: element,
      wasPlaying: false
    });
  }
  
  pauseMediaElements() {
    this.mediaElements.forEach((media, name) => {
      if (!media.element.paused) {
        media.wasPlaying = true;
        media.element.pause();
        console.log('Medya durduruldu:', name);
      }
    });
  }
  
  resumeMediaElements() {
    this.mediaElements.forEach((media, name) => {
      if (media.wasPlaying) {
        media.element.play();
        media.wasPlaying = false;
        console.log('Medya devam ettirildi:', name);
      }
    });
  }
  
  // Arka plan görevleri
  startBackgroundTasks() {
    console.log('Arka plan görevleri başlatıldı');
    // Veri senkronizasyonu, cache temizleme vb.
  }
  
  stopBackgroundTasks() {
    console.log('Arka plan görevleri durduruldu');
  }
  
  // Listener yönetimi
  addListener(name, callback) {
    this.listeners.set(name, callback);
  }
  
  removeListener(name) {
    this.listeners.delete(name);
  }
  
  notifyListeners(newState, oldState) {
    this.listeners.forEach((callback, name) => {
      try {
        callback(newState, oldState);
      } catch (error) {
        console.error('Listener hatası:', name, error);
      }
    });
  }
  
  // İstatistikler
  getStats() {
    return {
      visibilityState: this.visibilityState,
      totalVisibleTime: this.totalVisibleTime,
      visibilityChanges: this.visibilityChanges,
      currentSessionTime: Date.now() - this.visibilityStartTime
    };
  }
  
  // Görünürlük durumunu kontrol et
  isVisible() {
    return this.visibilityState === 'visible';
  }
  
  isHidden() {
    return this.visibilityState === 'hidden';
  }
  
  isPrerender() {
    return this.visibilityState === 'prerender';
  }
}

// Performans izleyici
class PerformanceMonitor {
  constructor() {
    this.visibilityManager = new PageVisibilityManager();
    this.metrics = {
      visibleTime: 0,
      hiddenTime: 0,
      visibilityChanges: 0,
      startTime: Date.now()
    };
    
    this.setupMonitoring();
  }
  
  setupMonitoring() {
    this.visibilityManager.addListener('performance', (newState, oldState) => {
      this.trackVisibilityChange(newState, oldState);
    });
  }
  
  trackVisibilityChange(newState, oldState) {
    const now = Date.now();
    
    if (oldState === 'visible' && newState === 'hidden') {
      this.metrics.visibleTime += now - this.lastChangeTime;
    } else if (oldState === 'hidden' && newState === 'visible') {
      this.metrics.hiddenTime += now - this.lastChangeTime;
    }
    
    this.lastChangeTime = now;
    this.metrics.visibilityChanges++;
  }
  
  getMetrics() {
    const totalTime = Date.now() - this.metrics.startTime;
    const visiblePercentage = (this.metrics.visibleTime / totalTime) * 100;
    const hiddenPercentage = (this.metrics.hiddenTime / totalTime) * 100;
    
    return {
      ...this.metrics,
      totalTime: totalTime,
      visiblePercentage: visiblePercentage,
      hiddenPercentage: hiddenPercentage
    };
  }
}

// Otomatik kaynak yöneticisi
class ResourceManager {
  constructor() {
    this.visibilityManager = new PageVisibilityManager();
    this.resources = new Map();
    this.setupResourceManagement();
  }
  
  setupResourceManagement() {
    this.visibilityManager.addListener('resource', (newState) => {
      if (newState === 'hidden') {
        this.optimizeForBackground();
      } else if (newState === 'visible') {
        this.optimizeForForeground();
      }
    });
  }
  
  addResource(name, resource) {
    this.resources.set(name, {
      resource: resource,
      type: resource.type || 'unknown',
      priority: resource.priority || 'normal'
    });
  }
  
  optimizeForBackground() {
    this.resources.forEach((res, name) => {
      if (res.type === 'animation' || res.type === 'video') {
        res.resource.pause();
        console.log('Kaynak durduruldu (arka plan):', name);
      } else if (res.type === 'polling' && res.priority === 'low') {
        res.resource.stop();
        console.log('Düşük öncelikli kaynak durduruldu:', name);
      }
    });
  }
  
  optimizeForForeground() {
    this.resources.forEach((res, name) => {
      if (res.type === 'animation' || res.type === 'video') {
        res.resource.play();
        console.log('Kaynak devam ettirildi (ön plan):', name);
      } else if (res.type === 'polling') {
        res.resource.start();
        console.log('Polling kaynağı başlatıldı:', name);
      }
    });
  }
}
```

**Alternatifler:**
- **Focus/Blur Events**: Window focus/blur olayları
- **Intersection Observer**: Element görünürlük takibi
- **Resize Observer**: Pencere boyut değişiklikleri
- **Custom Visibility Detection**: Özel görünürlük algılama
- **Third-party Libraries**: Harici görünürlük kütüphaneleri

**Ne Zaman Kullanılmalı?**
- ✅ Performans optimizasyonu gerektiğinde
- ✅ Kaynak yönetimi gerektiğinde
- ✅ Medya oynatma kontrolü gerektiğinde
- ✅ Animasyon kontrolü gerektiğinde
- ✅ Arka plan görevleri gerektiğinde
- ✅ Analytics ve metrik toplama gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit uygulamalarda
- ❌ Görünürlük takibi gerektirmediğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Sadece focus/blur yeterli olduğunda

**Kullanılmazsa Ne Olur?**
- **Performans Sorunları**: Gereksiz işlemler devam eder
- **Kaynak İsrafı**: CPU, bellek ve ağ kaynakları israf edilir
- **Battery Drain**: Pil ömrü kısalır
- **User Experience**: Kullanıcı deneyimi kötüleşir
- **Analytics Eksikliği**: Görünürlük metrikleri toplanamaz

**Modern Alternatifler:**
- **Intersection Observer API**: Element görünürlük takibi
- **Resize Observer API**: Boyut değişiklik takibi
- **Performance Observer API**: Performans metrikleri
- **Custom Visibility Systems**: Özel görünürlük sistemleri
- **Third-party Analytics**: Harici analitik servisleri

### Payment Request API

**Payment Request API Nedir?**
Payment Request API, web uygulamalarında güvenli ve standartlaştırılmış ödeme işlemleri gerçekleştirmek için kullanılan modern bir web API'sidir. Kullanıcıların ödeme bilgilerini güvenli bir şekilde girmesini ve işlemesini sağlar. Tarayıcı seviyesinde ödeme formları oluşturur, kullanıcı deneyimini iyileştirir ve ödeme güvenliğini artırır. E-ticaret uygulamalarında, abonelik servislerinde ve dijital ürün satışlarında kritik öneme sahiptir.

**Neden Kullanılır?**
- **Güvenli Ödeme**: Tarayıcı seviyesinde güvenli ödeme işlemi
- **Kullanıcı Deneyimi**: Standartlaştırılmış ödeme arayüzü
- **Hızlı Ödeme**: Kayıtlı ödeme bilgileri ile hızlı işlem
- **Çoklu Ödeme Yöntemi**: Farklı ödeme yöntemlerini destekleme
- **Mobil Uyumluluk**: Mobil cihazlarda optimize edilmiş deneyim
- **PCI Compliance**: PCI DSS uyumluluğu
- **Fraud Prevention**: Dolandırıcılık önleme

**Temel Kullanım Örnekleri:**

```javascript
// Temel ödeme isteği
if ('PaymentRequest' in window) {
  const paymentRequest = new PaymentRequest(
    [{
      supportedMethods: 'basic-card',
      data: {
        supportedNetworks: ['visa', 'mastercard', 'amex']
      }
    }],
    {
      total: {
        label: 'Toplam',
        amount: {currency: 'TRY', value: '100.00'}
      }
    }
  );
  
  paymentRequest.show();
}

// Detaylı ödeme isteği
const paymentRequest = new PaymentRequest(
  [{
    supportedMethods: 'basic-card',
    data: {
      supportedNetworks: ['visa', 'mastercard', 'amex'],
      supportedTypes: ['credit', 'debit']
    }
  }],
  {
    total: {
      label: 'Toplam',
      amount: {currency: 'TRY', value: '100.00'}
    },
    displayItems: [
      {
        label: 'Ürün',
        amount: {currency: 'TRY', value: '80.00'}
      },
      {
        label: 'KDV',
        amount: {currency: 'TRY', value: '20.00'}
      }
    ],
    shippingOptions: [
      {
        id: 'standard',
        label: 'Standart Kargo',
        amount: {currency: 'TRY', value: '0.00'},
        selected: true
      }
    ]
  },
  {
    requestPayerName: true,
    requestPayerEmail: true,
    requestPayerPhone: true,
    requestShipping: true
  }
);

// Ödeme işlemi
paymentRequest.show()
  .then(paymentResponse => {
    console.log('Ödeme başarılı:', paymentResponse);
    return paymentResponse.complete('success');
  })
  .catch(error => {
    console.error('Ödeme hatası:', error);
  });

// Ödeme yöntemi kontrolü
if (PaymentRequest.canMakePayment) {
  PaymentRequest.canMakePayment()
    .then(result => {
      if (result) {
        console.log('Ödeme destekleniyor');
      } else {
        console.log('Ödeme desteklenmiyor');
      }
    });
}

// Ödeme yöneticisi
class PaymentManager {
  constructor() {
    this.isSupported = 'PaymentRequest' in window;
    this.supportedMethods = [];
    this.defaultCurrency = 'TRY';
    this.taxRate = 0.18; // %18 KDV
  }
  
  // Desteklenen ödeme yöntemlerini kontrol et
  async checkSupport() {
    if (!this.isSupported) {
      throw new Error('Payment Request API desteklenmiyor');
    }
    
    const methods = ['basic-card', 'https://apple.com/apple-pay', 'https://google.com/pay'];
    
    for (const method of methods) {
      const canMakePayment = await PaymentRequest.canMakePayment([{
        supportedMethods: method
      }], {
        total: {
          label: 'Test',
          amount: {currency: this.defaultCurrency, value: '1.00'}
        }
      });
      
      if (canMakePayment) {
        this.supportedMethods.push(method);
      }
    }
    
    return this.supportedMethods;
  }
  
  // Ödeme isteği oluştur
  createPaymentRequest(items, options = {}) {
    const total = this.calculateTotal(items);
    const displayItems = this.createDisplayItems(items);
    
    const paymentDetails = {
      total: {
        label: 'Toplam',
        amount: total
      },
      displayItems: displayItems
    };
    
    const paymentRequest = new PaymentRequest(
      this.supportedMethods.map(method => ({
        supportedMethods: method,
        data: this.getMethodData(method)
      })),
      paymentDetails,
      {
        requestPayerName: options.requestPayerName || true,
        requestPayerEmail: options.requestPayerEmail || true,
        requestPayerPhone: options.requestPayerPhone || false,
        requestShipping: options.shipping || false
      }
    );
    
    return paymentRequest;
  }
  
  // Toplam tutarı hesapla
  calculateTotal(items) {
    const subtotal = items.reduce((sum, item) => {
      return sum + parseFloat(item.amount.value);
    }, 0);
    
    const tax = subtotal * this.taxRate;
    const total = subtotal + tax;
    
    return {
      currency: this.defaultCurrency,
      value: total.toFixed(2)
    };
  }
  
  // Görüntüleme öğelerini oluştur
  createDisplayItems(items) {
    const displayItems = items.map(item => ({
      label: item.label,
      amount: {
        currency: this.defaultCurrency,
        value: item.amount.value
      }
    }));
    
    const subtotal = items.reduce((sum, item) => {
      return sum + parseFloat(item.amount.value);
    }, 0);
    
    const tax = subtotal * this.taxRate;
    
    displayItems.push({
      label: 'KDV',
      amount: {
        currency: this.defaultCurrency,
        value: tax.toFixed(2)
      }
    });
    
    return displayItems;
  }
  
  // Ödeme yöntemi verilerini al
  getMethodData(method) {
    switch (method) {
      case 'basic-card':
        return {
          supportedNetworks: ['visa', 'mastercard', 'amex'],
          supportedTypes: ['credit', 'debit']
        };
      case 'https://apple.com/apple-pay':
        return {
          merchantIdentifier: 'merchant.com.example',
          supportedNetworks: ['visa', 'mastercard', 'amex'],
          merchantCapabilities: ['supports3DS'],
          countryCode: 'TR',
          currencyCode: 'TRY'
        };
      case 'https://google.com/pay':
        return {
          environment: 'TEST',
          apiVersion: 2,
          apiVersionMinor: 0,
          allowedPaymentMethods: [{
            type: 'CARD',
            parameters: {
              allowedAuthMethods: ['PAN_ONLY', 'CRYPTOGRAM_3DS'],
              allowedCardNetworks: ['VISA', 'MASTERCARD', 'AMEX']
            }
          }]
        };
      default:
        return {};
    }
  }
  
  // Ödeme işle
  async processPayment(paymentRequest) {
    try {
      const paymentResponse = await paymentRequest.show();
      
      const paymentData = {
        methodName: paymentResponse.methodName,
        details: paymentResponse.details,
        payerName: paymentResponse.payerName,
        payerEmail: paymentResponse.payerEmail,
        payerPhone: paymentResponse.payerPhone,
        shippingAddress: paymentResponse.shippingAddress,
        shippingOption: paymentResponse.shippingOption
      };
      
      const result = await this.sendPaymentToServer(paymentData);
      
      if (result.success) {
        await paymentResponse.complete('success');
        return { success: true, data: result.data };
      } else {
        await paymentResponse.complete('fail');
        return { success: false, error: result.error };
      }
    } catch (error) {
      if (error.name === 'AbortError') {
        return { success: false, error: 'Ödeme iptal edildi' };
      } else {
        return { success: false, error: error.message };
      }
    }
  }
  
  // Server'a ödeme bilgilerini gönder
  async sendPaymentToServer(paymentData) {
    try {
      const response = await fetch('/api/payment/process', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(paymentData)
      });
      
      return await response.json();
    } catch (error) {
      return { success: false, error: 'Server hatası' };
    }
  }
}
```

**Alternatifler:**
- **Traditional Forms**: Geleneksel ödeme formları
- **Third-party Payment**: Harici ödeme servisleri
- **Stripe Elements**: Stripe ödeme bileşenleri
- **PayPal SDK**: PayPal entegrasyonu
- **Custom Payment Forms**: Özel ödeme formları

**Ne Zaman Kullanılmalı?**
- ✅ E-ticaret uygulamalarında
- ✅ Abonelik servislerinde
- ✅ Dijital ürün satışlarında
- ✅ Mobil uygulamalarda
- ✅ Güvenli ödeme gerektiğinde
- ✅ Hızlı ödeme deneyimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Özel ödeme akışı gerektiğinde
- ❌ Çok karmaşık ödeme mantığı gerektiğinde
- ❌ Offline ödeme gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Güvenlik Riskleri**: Ödeme bilgileri güvenli olmayabilir
- **Kullanıcı Deneyimi**: Karmaşık ödeme formları
- **Mobil Uyumsuzluk**: Mobil cihazlarda zor kullanım
- **PCI Compliance**: PCI DSS uyumluluğu eksikliği
- **Fraud Risk**: Dolandırıcılık riski artar

**Modern Alternatifler:**
- **Stripe Elements**: Modern ödeme bileşenleri
- **PayPal SDK**: Güvenilir ödeme servisi
- **Apple Pay/Google Pay**: Mobil ödeme çözümleri
- **Cryptocurrency Payments**: Kripto para ödemeleri
- **BNPL Services**: Buy Now Pay Later servisleri

### Performance API

**Performance API Nedir?**
Performance API, web uygulamalarının performansını ölçmek, analiz etmek ve optimize etmek için kullanılan kapsamlı bir web API'sidir. Sayfa yükleme süreleri, kaynak yükleme performansı, kullanıcı etkileşim süreleri ve genel uygulama performansı hakkında detaylı bilgi sağlar. Web Vitals, Core Web Vitals ve performans optimizasyonu için kritik öneme sahiptir. Geliştiricilerin performans sorunlarını tespit etmesini ve çözmesini sağlar.

**Neden Kullanılır?**
- **Performans Ölçümü**: Detaylı performans metrikleri
- **Optimizasyon**: Performans sorunlarını tespit etme
- **Web Vitals**: Core Web Vitals metrikleri
- **Kullanıcı Deneyimi**: Kullanıcı deneyimi analizi
- **Monitoring**: Gerçek zamanlı performans izleme
- **Analytics**: Performans analitikleri
- **Debugging**: Performans hata ayıklama

**Temel Kullanım Örnekleri:**

```javascript
// Performans işareti
performance.mark('start');

// İşlem yap
setTimeout(() => {
  performance.mark('end');
  performance.measure('operation', 'start', 'end');
  
  const measure = performance.getEntriesByName('operation')[0];
  console.log(`İşlem süresi: ${measure.duration} ms`);
}, 1000);

// Sayfa yükleme performansı
window.addEventListener('load', () => {
  const navigation = performance.getEntriesByType('navigation')[0];
  console.log('Sayfa yükleme süresi:', navigation.loadEventEnd - navigation.loadEventStart);
  console.log('DOM hazır süresi:', navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart);
  console.log('İlk byte süresi:', navigation.responseStart - navigation.requestStart);
});

// Kaynak yükleme performansı
const resources = performance.getEntriesByType('resource');
resources.forEach(resource => {
  console.log(`${resource.name}: ${resource.duration}ms`);
});

// Performans gözlemcisi
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach(entry => {
    console.log('Performans girişi:', entry);
  });
});

observer.observe({ entryTypes: ['measure', 'navigation', 'resource'] });

// Timing API kullanımı
const timing = performance.timing;
console.log('DNS arama süresi:', timing.domainLookupEnd - timing.domainLookupStart);
console.log('TCP bağlantı süresi:', timing.connectEnd - timing.connectStart);
console.log('İstek süresi:', timing.responseEnd - timing.requestStart);
console.log('DOM işleme süresi:', timing.domComplete - timing.domLoading);

// Memory API
if ('memory' in performance) {
  console.log('Kullanılan bellek:', performance.memory.usedJSHeapSize);
  console.log('Toplam bellek:', performance.memory.totalJSHeapSize);
  console.log('Bellek limiti:', performance.memory.jsHeapSizeLimit);
}

// High Resolution Time
const startTime = performance.now();
// İşlem yap
const endTime = performance.now();
console.log(`İşlem süresi: ${endTime - startTime} ms`);

// User Timing API
performance.mark('custom-start');
// Özel işlem
performance.mark('custom-end');
performance.measure('custom-operation', 'custom-start', 'custom-end');

// Performans girişlerini temizle
performance.clearMarks();
performance.clearMeasures();
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// Performans yöneticisi
class PerformanceManager {
  constructor() {
    this.observers = new Map();
    this.metrics = new Map();
    this.isSupported = 'performance' in window;
    this.setupDefaultObservers();
  }
  
  // Varsayılan gözlemcileri kur
  setupDefaultObservers() {
    if (!this.isSupported) return;
    
    // Navigation timing
    this.observeNavigation();
    
    // Resource timing
    this.observeResources();
    
    // User timing
    this.observeUserTiming();
    
    // Paint timing
    this.observePaint();
    
    // Layout shift
    this.observeLayoutShift();
  }
  
  // Navigation timing gözlemi
  observeNavigation() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        this.analyzeNavigationTiming(entry);
      });
    });
    
    observer.observe({ entryTypes: ['navigation'] });
    this.observers.set('navigation', observer);
  }
  
  // Navigation timing analizi
  analyzeNavigationTiming(entry) {
    const metrics = {
      dns: entry.domainLookupEnd - entry.domainLookupStart,
      tcp: entry.connectEnd - entry.connectStart,
      request: entry.responseEnd - entry.requestStart,
      response: entry.responseEnd - entry.responseStart,
      dom: entry.domComplete - entry.domLoading,
      load: entry.loadEventEnd - entry.loadEventStart,
      total: entry.loadEventEnd - entry.navigationStart
    };
    
    this.metrics.set('navigation', metrics);
    console.log('Navigation metrics:', metrics);
  }
  
  // Resource timing gözlemi
  observeResources() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        this.analyzeResourceTiming(entry);
      });
    });
    
    observer.observe({ entryTypes: ['resource'] });
    this.observers.set('resource', observer);
  }
  
  // Resource timing analizi
  analyzeResourceTiming(entry) {
    const metrics = {
      name: entry.name,
      duration: entry.duration,
      size: entry.transferSize,
      type: entry.initiatorType,
      dns: entry.domainLookupEnd - entry.domainLookupStart,
      tcp: entry.connectEnd - entry.connectStart,
      request: entry.responseEnd - entry.requestStart
    };
    
    console.log('Resource metrics:', metrics);
    
    // Yavaş kaynakları tespit et
    if (entry.duration > 1000) {
      console.warn('Yavaş kaynak:', entry.name, entry.duration + 'ms');
    }
  }
  
  // User timing gözlemi
  observeUserTiming() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        this.analyzeUserTiming(entry);
      });
    });
    
    observer.observe({ entryTypes: ['measure', 'mark'] });
    this.observers.set('user-timing', observer);
  }
  
  // User timing analizi
  analyzeUserTiming(entry) {
    if (entry.entryType === 'measure') {
      console.log('Ölçüm:', entry.name, entry.duration + 'ms');
      
      // Yavaş işlemleri tespit et
      if (entry.duration > 100) {
        console.warn('Yavaş işlem:', entry.name, entry.duration + 'ms');
      }
    }
  }
  
  // Paint timing gözlemi
  observePaint() {
    if (!('PerformanceObserver' in window)) return;
    
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        this.analyzePaintTiming(entry);
      });
    });
    
    observer.observe({ entryTypes: ['paint'] });
    this.observers.set('paint', observer);
  }
  
  // Paint timing analizi
  analyzePaintTiming(entry) {
    const metrics = {
      name: entry.name,
      startTime: entry.startTime
    };
    
    this.metrics.set(entry.name, metrics);
    console.log('Paint timing:', metrics);
    
    // Core Web Vitals - FCP
    if (entry.name === 'first-contentful-paint') {
      console.log('First Contentful Paint:', entry.startTime + 'ms');
    }
  }
  
  // Layout shift gözlemi
  observeLayoutShift() {
    if (!('PerformanceObserver' in window)) return;
    
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        this.analyzeLayoutShift(entry);
      });
    });
    
    observer.observe({ entryTypes: ['layout-shift'] });
    this.observers.set('layout-shift', observer);
  }
  
  // Layout shift analizi
  analyzeLayoutShift(entry) {
    if (!entry.hadRecentInput) {
      const cls = entry.value;
      console.log('Layout Shift:', cls);
      
      // Core Web Vitals - CLS
      if (cls > 0.1) {
        console.warn('Yüksek Layout Shift:', cls);
      }
    }
  }
  
  // Özel performans işareti
  mark(name) {
    if (this.isSupported) {
      performance.mark(name);
    }
  }
  
  // Özel performans ölçümü
  measure(name, startMark, endMark) {
    if (this.isSupported) {
      performance.measure(name, startMark, endMark);
    }
  }
  
  // Performans girişlerini al
  getEntries(type) {
    if (!this.isSupported) return [];
    return performance.getEntriesByType(type);
  }
  
  // Performans girişlerini temizle
  clearEntries(type) {
    if (!this.isSupported) return;
    
    if (type === 'marks') {
      performance.clearMarks();
    } else if (type === 'measures') {
      performance.clearMeasures();
    } else if (type === 'all') {
      performance.clearMarks();
      performance.clearMeasures();
    }
  }
  
  // Bellek kullanımını al
  getMemoryUsage() {
    if (!this.isSupported || !('memory' in performance)) {
      return null;
    }
    
    return {
      used: performance.memory.usedJSHeapSize,
      total: performance.memory.totalJSHeapSize,
      limit: performance.memory.jsHeapSizeLimit
    };
  }
  
  // Core Web Vitals hesapla
  calculateCoreWebVitals() {
    const vitals = {};
    
    // FCP (First Contentful Paint)
    const fcpEntry = performance.getEntriesByName('first-contentful-paint')[0];
    if (fcpEntry) {
      vitals.fcp = fcpEntry.startTime;
    }
    
    // LCP (Largest Contentful Paint)
    const lcpEntries = performance.getEntriesByType('largest-contentful-paint');
    if (lcpEntries.length > 0) {
      vitals.lcp = lcpEntries[lcpEntries.length - 1].startTime;
    }
    
    // CLS (Cumulative Layout Shift)
    const clsEntries = performance.getEntriesByType('layout-shift');
    vitals.cls = clsEntries.reduce((sum, entry) => {
      return sum + (entry.hadRecentInput ? 0 : entry.value);
    }, 0);
    
    // FID (First Input Delay)
    const fidEntries = performance.getEntriesByType('first-input');
    if (fidEntries.length > 0) {
      vitals.fid = fidEntries[0].processingStart - fidEntries[0].startTime;
    }
    
    return vitals;
  }
  
  // Performans raporu oluştur
  generateReport() {
    const report = {
      timestamp: Date.now(),
      navigation: this.metrics.get('navigation'),
      memory: this.getMemoryUsage(),
      coreWebVitals: this.calculateCoreWebVitals(),
      resources: this.getEntries('resource'),
      measures: this.getEntries('measure')
    };
    
    return report;
  }
  
  // Gözlemcileri temizle
  cleanup() {
    this.observers.forEach(observer => {
      observer.disconnect();
    });
    this.observers.clear();
  }
}

// Performans izleyici
class PerformanceTracker {
  constructor() {
    this.manager = new PerformanceManager();
    this.timers = new Map();
    this.counters = new Map();
  }
  
  // Zamanlayıcı başlat
  startTimer(name) {
    this.timers.set(name, performance.now());
  }
  
  // Zamanlayıcı durdur
  endTimer(name) {
    const startTime = this.timers.get(name);
    if (startTime) {
      const duration = performance.now() - startTime;
      this.timers.delete(name);
      return duration;
    }
    return null;
  }
  
  // Sayaç artır
  incrementCounter(name) {
    const current = this.counters.get(name) || 0;
    this.counters.set(name, current + 1);
  }
  
  // Sayaç değerini al
  getCounter(name) {
    return this.counters.get(name) || 0;
  }
  
  // Performans raporu oluştur
  getReport() {
    return {
      ...this.manager.generateReport(),
      timers: Object.fromEntries(this.timers),
      counters: Object.fromEntries(this.counters)
    };
  }
}

// Performans analiz aracı
class PerformanceAnalyzer {
  constructor() {
    this.manager = new PerformanceManager();
    this.thresholds = {
      fcp: 1800, // 1.8s
      lcp: 2500, // 2.5s
      fid: 100,  // 100ms
      cls: 0.1   // 0.1
    };
  }
  
  // Performans analizi yap
  analyze() {
    const vitals = this.manager.calculateCoreWebVitals();
    const analysis = {
      score: 0,
      issues: [],
      recommendations: []
    };
    
    // FCP analizi
    if (vitals.fcp) {
      if (vitals.fcp > this.thresholds.fcp) {
        analysis.issues.push('FCP çok yavaş: ' + vitals.fcp + 'ms');
        analysis.recommendations.push('Sayfa yükleme hızını optimize edin');
      } else {
        analysis.score += 25;
      }
    }
    
    // LCP analizi
    if (vitals.lcp) {
      if (vitals.lcp > this.thresholds.lcp) {
        analysis.issues.push('LCP çok yavaş: ' + vitals.lcp + 'ms');
        analysis.recommendations.push('En büyük içerik yükleme hızını optimize edin');
      } else {
        analysis.score += 25;
      }
    }
    
    // FID analizi
    if (vitals.fid) {
      if (vitals.fid > this.thresholds.fid) {
        analysis.issues.push('FID çok yavaş: ' + vitals.fid + 'ms');
        analysis.recommendations.push('JavaScript yükleme hızını optimize edin');
      } else {
        analysis.score += 25;
      }
    }
    
    // CLS analizi
    if (vitals.cls) {
      if (vitals.cls > this.thresholds.cls) {
        analysis.issues.push('CLS çok yüksek: ' + vitals.cls);
        analysis.recommendations.push('Layout shift sorunlarını çözün');
      } else {
        analysis.score += 25;
      }
    }
    
    return analysis;
  }
}
```

**Alternatifler:**
- **Custom Timing**: Özel zamanlama çözümleri
- **Third-party Analytics**: Harici analitik servisleri
- **Browser DevTools**: Tarayıcı geliştirici araçları
- **Lighthouse**: Google Lighthouse
- **WebPageTest**: Web sayfası test araçları

**Ne Zaman Kullanılmalı?**
- ✅ Performans optimizasyonu gerektiğinde
- ✅ Web Vitals takibi gerektiğinde
- ✅ Kullanıcı deneyimi analizi gerektiğinde
- ✅ Performans monitoring gerektiğinde
- ✅ Debugging gerektiğinde
- ✅ Analytics gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit uygulamalarda
- ❌ Performans takibi gerektirmediğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Sadece temel timing gerektiğinde

**Kullanılmazsa Ne Olur?**
- **Performans Sorunları**: Performans sorunları tespit edilemez
- **Kullanıcı Deneyimi**: Kullanıcı deneyimi kötüleşir
- **SEO Etkisi**: Arama motoru sıralaması düşer
- **Conversion Loss**: Dönüşüm oranları azalır
- **Monitoring Eksikliği**: Performans izleme yapılamaz

**Modern Alternatifler:**
- **Web Vitals API**: Core Web Vitals metrikleri
- **Performance Observer API**: Performans gözlemi
- **User Timing API**: Kullanıcı zamanlama
- **Resource Timing API**: Kaynak zamanlama
- **Navigation Timing API**: Navigasyon zamanlama

### Permissions API

**Permissions API Nedir?**
Permissions API, web uygulamalarının kullanıcı izinlerini sorgulamasını, izin durumlarını takip etmesini ve izin değişikliklerini dinlemesini sağlayan modern bir web API'sidir. Kullanıcı deneyimini iyileştirmek, izin yönetimini optimize etmek ve güvenlik açıklarını önlemek için kritik öneme sahiptir. Geolocation, camera, microphone, notifications gibi hassas API'lerin izin durumlarını kontrol eder ve kullanıcıya uygun deneyim sunar.

**Neden Kullanılır?**
- **İzin Yönetimi**: Kullanıcı izinlerini sorgulama ve takip etme
- **Kullanıcı Deneyimi**: İzin durumuna göre uygun arayüz sunma
- **Güvenlik**: İzinsiz API erişimini önleme
- **Privacy**: Kullanıcı gizliliğini koruma
- **Performance**: Gereksiz izin isteklerini önleme
- **Analytics**: İzin kullanım istatistikleri
- **Compliance**: Veri koruma düzenlemelerine uyum

**Temel Kullanım Örnekleri:**

```javascript
// Konum izni sorgula
const permission = await navigator.permissions.query({name: 'geolocation'});
console.log(`Konum izni: ${permission.state}`);

permission.addEventListener('change', () => {
  console.log(`İzin durumu değişti: ${permission.state}`);
});

// Kamera izni sorgula
const cameraPermission = await navigator.permissions.query({name: 'camera'});
console.log(`Kamera izni: ${cameraPermission.state}`);

// Mikrofon izni sorgula
const microphonePermission = await navigator.permissions.query({name: 'microphone'});
console.log(`Mikrofon izni: ${microphonePermission.state}`);

// Bildirim izni sorgula
const notificationPermission = await navigator.permissions.query({name: 'notifications'});
console.log(`Bildirim izni: ${notificationPermission.state}`);

// İzin durumları
if (permission.state === 'granted') {
  console.log('İzin verilmiş');
} else if (permission.state === 'denied') {
  console.log('İzin reddedilmiş');
} else if (permission.state === 'prompt') {
  console.log('İzin istenmemiş');
}

// İzin değişikliklerini dinle
permission.addEventListener('change', () => {
  const newState = permission.state;
  console.log('Yeni izin durumu:', newState);
  
  if (newState === 'granted') {
    // İzin verildi, özelliği etkinleştir
    enableFeature();
  } else if (newState === 'denied') {
    // İzin reddedildi, alternatif göster
    showAlternative();
  }
});

// Çoklu izin sorgulama
async function checkMultiplePermissions() {
  const permissions = [
    {name: 'geolocation'},
    {name: 'camera'},
    {name: 'microphone'},
    {name: 'notifications'}
  ];
  
  const results = await Promise.all(
    permissions.map(permission => 
      navigator.permissions.query(permission)
    )
  );
  
  results.forEach((result, index) => {
    console.log(`${permissions[index].name}: ${result.state}`);
  });
}

// İzin durumuna göre davranış
async function handlePermissionBasedFeature() {
  const permission = await navigator.permissions.query({name: 'geolocation'});
  
  switch (permission.state) {
    case 'granted':
      // İzin var, özelliği kullan
      getCurrentLocation();
      break;
    case 'denied':
      // İzin reddedilmiş, alternatif göster
      showLocationAlternative();
      break;
    case 'prompt':
      // İzin istenmemiş, kullanıcıya sor
      requestLocationPermission();
      break;
  }
}
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// İzin yöneticisi
class PermissionManager {
  constructor() {
    this.isSupported = 'permissions' in navigator;
    this.permissions = new Map();
    this.listeners = new Map();
    this.cache = new Map();
    this.cacheTimeout = 5 * 60 * 1000; // 5 dakika
  }
  
  // İzin sorgula
  async query(permissionName, options = {}) {
    if (!this.isSupported) {
      throw new Error('Permissions API desteklenmiyor');
    }
    
    // Cache kontrolü
    const cacheKey = `${permissionName}_${JSON.stringify(options)}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.permission;
    }
    
    try {
      const permission = await navigator.permissions.query({
        name: permissionName,
        ...options
      });
      
      // Cache'e kaydet
      this.cache.set(cacheKey, {
        permission: permission,
        timestamp: Date.now()
      });
      
      // İzin durumunu kaydet
      this.permissions.set(permissionName, permission);
      
      // Event listener ekle
      this.setupPermissionListener(permissionName, permission);
      
      return permission;
    } catch (error) {
      console.error('İzin sorgulama hatası:', error);
      throw error;
    }
  }
  
  // İzin listener'ı kur
  setupPermissionListener(permissionName, permission) {
    if (this.listeners.has(permissionName)) {
      return; // Zaten dinleniyor
    }
    
    const listener = (event) => {
      this.handlePermissionChange(permissionName, event.target.state);
    };
    
    permission.addEventListener('change', listener);
    this.listeners.set(permissionName, listener);
  }
  
  // İzin değişikliğini işle
  handlePermissionChange(permissionName, newState) {
    console.log(`İzin değişti: ${permissionName} -> ${newState}`);
    
    // Cache'i temizle
    this.clearCache(permissionName);
    
    // Event dispatch
    this.dispatchPermissionChange(permissionName, newState);
  }
  
  // İzin değişiklik event'i gönder
  dispatchPermissionChange(permissionName, newState) {
    const event = new CustomEvent('permissionchange', {
      detail: {
        permission: permissionName,
        state: newState
      }
    });
    
    window.dispatchEvent(event);
  }
  
  // İzin durumunu al
  getPermissionState(permissionName) {
    const permission = this.permissions.get(permissionName);
    return permission ? permission.state : null;
  }
  
  // İzin verilmiş mi kontrol et
  isGranted(permissionName) {
    return this.getPermissionState(permissionName) === 'granted';
  }
  
  // İzin reddedilmiş mi kontrol et
  isDenied(permissionName) {
    return this.getPermissionState(permissionName) === 'denied';
  }
  
  // İzin istenmemiş mi kontrol et
  isPrompt(permissionName) {
    return this.getPermissionState(permissionName) === 'prompt';
  }
  
  // Cache'i temizle
  clearCache(permissionName) {
    for (const [key, value] of this.cache.entries()) {
      if (key.startsWith(permissionName)) {
        this.cache.delete(key);
      }
    }
  }
  
  // Tüm cache'i temizle
  clearAllCache() {
    this.cache.clear();
  }
  
  // İzin istatistikleri
  getPermissionStats() {
    const stats = {
      total: this.permissions.size,
      granted: 0,
      denied: 0,
      prompt: 0
    };
    
    for (const permission of this.permissions.values()) {
      switch (permission.state) {
        case 'granted':
          stats.granted++;
          break;
        case 'denied':
          stats.denied++;
          break;
        case 'prompt':
          stats.prompt++;
          break;
      }
    }
    
    return stats;
  }
  
  // Temizlik
  cleanup() {
    this.listeners.forEach((listener, permissionName) => {
      const permission = this.permissions.get(permissionName);
      if (permission) {
        permission.removeEventListener('change', listener);
      }
    });
    
    this.listeners.clear();
    this.permissions.clear();
    this.cache.clear();
  }
}

// İzin tabanlı özellik yöneticisi
class PermissionBasedFeatureManager {
  constructor() {
    this.permissionManager = new PermissionManager();
    this.features = new Map();
    this.setupGlobalListeners();
  }
  
  // Global listener'ları kur
  setupGlobalListeners() {
    window.addEventListener('permissionchange', (event) => {
      this.handlePermissionChange(event.detail.permission, event.detail.state);
    });
  }
  
  // Özellik kaydet
  registerFeature(name, permissionName, options = {}) {
    this.features.set(name, {
      permissionName: permissionName,
      options: options,
      enabled: false,
      callbacks: {
        onGranted: options.onGranted || (() => {}),
        onDenied: options.onDenied || (() => {}),
        onPrompt: options.onPrompt || (() => {})
      }
    });
    
    // İzin durumunu kontrol et
    this.checkFeaturePermission(name);
  }
  
  // Özellik izin durumunu kontrol et
  async checkFeaturePermission(featureName) {
    const feature = this.features.get(featureName);
    if (!feature) return;
    
    try {
      const permission = await this.permissionManager.query(feature.permissionName);
      this.updateFeatureState(featureName, permission.state);
    } catch (error) {
      console.error('Özellik izin kontrolü hatası:', error);
    }
  }
  
  // Özellik durumunu güncelle
  updateFeatureState(featureName, permissionState) {
    const feature = this.features.get(featureName);
    if (!feature) return;
    
    feature.enabled = permissionState === 'granted';
    
    // Callback'leri çağır
    switch (permissionState) {
      case 'granted':
        feature.callbacks.onGranted();
        break;
      case 'denied':
        feature.callbacks.onDenied();
        break;
      case 'prompt':
        feature.callbacks.onPrompt();
        break;
    }
  }
  
  // İzin değişikliğini işle
  handlePermissionChange(permissionName, newState) {
    // Bu izni kullanan özellikleri bul
    for (const [featureName, feature] of this.features.entries()) {
      if (feature.permissionName === permissionName) {
        this.updateFeatureState(featureName, newState);
      }
    }
  }
  
  // Özellik etkin mi kontrol et
  isFeatureEnabled(featureName) {
    const feature = this.features.get(featureName);
    return feature ? feature.enabled : false;
  }
  
  // Özellik kullan
  async useFeature(featureName) {
    const feature = this.features.get(featureName);
    if (!feature) {
      throw new Error('Özellik bulunamadı: ' + featureName);
    }
    
    if (!feature.enabled) {
      throw new Error('Özellik etkin değil: ' + featureName);
    }
    
    // Özellik kullanım mantığı
    return feature.options.handler ? feature.options.handler() : null;
  }
}

// İzin analiz aracı
class PermissionAnalyzer {
  constructor() {
    this.permissionManager = new PermissionManager();
    this.analytics = {
      permissionRequests: 0,
      permissionGrants: 0,
      permissionDenials: 0,
      permissionChanges: 0
    };
    
    this.setupAnalytics();
  }
  
  // Analitik kurulumu
  setupAnalytics() {
    window.addEventListener('permissionchange', () => {
      this.analytics.permissionChanges++;
    });
  }
  
  // İzin isteği kaydet
  recordPermissionRequest(permissionName) {
    this.analytics.permissionRequests++;
    console.log('İzin isteği:', permissionName);
  }
  
  // İzin verilme kaydet
  recordPermissionGrant(permissionName) {
    this.analytics.permissionGrants++;
    console.log('İzin verildi:', permissionName);
  }
  
  // İzin reddedilme kaydet
  recordPermissionDenial(permissionName) {
    this.analytics.permissionDenials++;
    console.log('İzin reddedildi:', permissionName);
  }
  
  // Analitik raporu al
  getAnalytics() {
    return {
      ...this.analytics,
      grantRate: this.analytics.permissionRequests > 0 ? 
        (this.analytics.permissionGrants / this.analytics.permissionRequests) * 100 : 0,
      denialRate: this.analytics.permissionRequests > 0 ? 
        (this.analytics.permissionDenials / this.analytics.permissionRequests) * 100 : 0
    };
  }
  
  // İzin kullanım önerileri
  getRecommendations() {
    const recommendations = [];
    const stats = this.permissionManager.getPermissionStats();
    
    if (stats.denied > stats.granted) {
      recommendations.push('Çok fazla izin reddediliyor. İzin isteklerini optimize edin.');
    }
    
    if (stats.prompt > 0) {
      recommendations.push('Bekleyen izin istekleri var. Kullanıcı deneyimini iyileştirin.');
    }
    
    return recommendations;
  }
}
```

**Alternatifler:**
- **Manual Permission Handling**: Manuel izin yönetimi
- **Feature Detection**: Özellik algılama
- **User Agent Detection**: Tarayıcı algılama
- **Third-party Libraries**: Harici izin kütüphaneleri
- **Custom Permission Systems**: Özel izin sistemleri

**Ne Zaman Kullanılmalı?**
- ✅ Hassas API'ler kullanırken
- ✅ Kullanıcı deneyimi optimize ederken
- ✅ İzin yönetimi gerektiğinde
- ✅ Privacy compliance gerektiğinde
- ✅ Performance optimization gerektiğinde
- ✅ Analytics gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit uygulamalarda
- ❌ İzin yönetimi gerektirmediğinde
- ❌ Sadece feature detection yeterli olduğunda

**Kullanılmazsa Ne Olur?**
- **Kullanıcı Deneyimi**: Kötü kullanıcı deneyimi
- **Güvenlik Riskleri**: İzinsiz API erişimi
- **Privacy Issues**: Gizlilik sorunları
- **Performance Issues**: Gereksiz izin istekleri
- **Compliance Issues**: Veri koruma uyumsuzluğu

**Modern Alternatifler:**
- **Permission Policy**: İzin politikaları
- **Feature Policy**: Özellik politikaları
- **Trusted Types**: Güvenilir tipler
- **CSP (Content Security Policy)**: İçerik güvenlik politikası
- **Third-party Permission Services**: Harici izin servisleri

### Pointer Events API

**Pointer Events API Nedir?**
Pointer Events API, web uygulamalarının farklı işaretçi türlerini (mouse, touch, pen, stylus) tek bir tutarlı arayüzle yönetmesini sağlayan modern bir web API'sidir. Mouse, dokunmatik ekran ve kalem girişlerini birleştirerek, çoklu cihaz desteği ve gelişmiş kullanıcı etkileşimi sağlar. Basınç, eğim, döndürme gibi gelişmiş özellikleri destekler ve modern web uygulamalarında dokunmatik deneyimi optimize eder.

**Neden Kullanılır?**
- **Çoklu Cihaz Desteği**: Mouse, touch, pen desteği
- **Gelişmiş Etkileşim**: Basınç, eğim, döndürme algılama
- **Tutarlı API**: Tek arayüzle tüm işaretçi türleri
- **Performans**: Optimize edilmiş olay işleme
- **Accessibility**: Erişilebilirlik desteği
- **Modern UX**: Gelişmiş kullanıcı deneyimi
- **Cross-Platform**: Platform bağımsız etkileşim

**Temel Kullanım Örnekleri:**

```javascript
// Pointer down olayı
element.addEventListener('pointerdown', event => {
  console.log(`İşaretçi tipi: ${event.pointerType}`);
  console.log(`Basınç: ${event.pressure}`);
  console.log(`Eğim X: ${event.tiltX}, Eğim Y: ${event.tiltY}`);
  console.log(`Döndürme: ${event.twist}`);
  console.log(`İşaretçi ID: ${event.pointerId}`);
});

// Pointer move olayı
element.addEventListener('pointermove', event => {
  console.log(`X: ${event.clientX}, Y: ${event.clientY}`);
  console.log(`Basınç: ${event.pressure}`);
  console.log(`Genişlik: ${event.width}, Yükseklik: ${event.height}`);
});

// Pointer up olayı
element.addEventListener('pointerup', event => {
  console.log('İşaretçi kaldırıldı');
  console.log(`Süre: ${event.timeStamp}`);
});

// Pointer cancel olayı
element.addEventListener('pointercancel', event => {
  console.log('İşaretçi iptal edildi');
});

// Pointer enter/leave olayları
element.addEventListener('pointerenter', event => {
  console.log('İşaretçi elemente girdi');
});

element.addEventListener('pointerleave', event => {
  console.log('İşaretçi elementten çıktı');
});

// Pointer over/out olayları
element.addEventListener('pointerover', event => {
  console.log('İşaretçi element üzerinde');
});

element.addEventListener('pointerout', event => {
  console.log('İşaretçi element dışında');
});

// İşaretçi türü kontrolü
element.addEventListener('pointerdown', event => {
  switch (event.pointerType) {
    case 'mouse':
      console.log('Mouse kullanılıyor');
      break;
    case 'touch':
      console.log('Dokunmatik kullanılıyor');
      break;
    case 'pen':
      console.log('Kalem kullanılıyor');
      break;
    default:
      console.log('Bilinmeyen işaretçi türü');
  }
});

// Basınç algılama
element.addEventListener('pointermove', event => {
  if (event.pressure > 0.5) {
    console.log('Güçlü basınç');
  } else if (event.pressure > 0.1) {
    console.log('Orta basınç');
  } else {
    console.log('Hafif basınç');
  }
});

// Çoklu işaretçi desteği
const activePointers = new Map();

element.addEventListener('pointerdown', event => {
  activePointers.set(event.pointerId, {
    type: event.pointerType,
    startX: event.clientX,
    startY: event.clientY,
    pressure: event.pressure
  });
  
  console.log(`Aktif işaretçi sayısı: ${activePointers.size}`);
});

element.addEventListener('pointerup', event => {
  activePointers.delete(event.pointerId);
  console.log(`Aktif işaretçi sayısı: ${activePointers.size}`);
});
```

**Gelişmiş Kullanım Örnekleri:**

```javascript
// İşaretçi yöneticisi
class PointerManager {
  constructor(element) {
    this.element = element;
    this.activePointers = new Map();
    this.gestures = new Map();
    this.isSupported = 'PointerEvent' in window;
    this.setupEventListeners();
  }
  
  // Event listener'ları kur
  setupEventListeners() {
    if (!this.isSupported) return;
    
    this.element.addEventListener('pointerdown', (e) => this.handlePointerDown(e));
    this.element.addEventListener('pointermove', (e) => this.handlePointerMove(e));
    this.element.addEventListener('pointerup', (e) => this.handlePointerUp(e));
    this.element.addEventListener('pointercancel', (e) => this.handlePointerCancel(e));
  }
  
  // Pointer down işle
  handlePointerDown(event) {
    const pointer = {
      id: event.pointerId,
      type: event.pointerType,
      startX: event.clientX,
      startY: event.clientY,
      currentX: event.clientX,
      currentY: event.clientY,
      pressure: event.pressure,
      tiltX: event.tiltX,
      tiltY: event.tiltY,
      twist: event.twist,
      width: event.width,
      height: event.height,
      startTime: event.timeStamp,
      isPrimary: event.isPrimary
    };
    
    this.activePointers.set(event.pointerId, pointer);
    
    // Gesture başlat
    this.startGesture(event.pointerId, pointer);
    
    console.log('Pointer down:', pointer);
  }
  
  // Pointer move işle
  handlePointerMove(event) {
    const pointer = this.activePointers.get(event.pointerId);
    if (!pointer) return;
    
    // Pointer bilgilerini güncelle
    pointer.currentX = event.clientX;
    pointer.currentY = event.clientY;
    pointer.pressure = event.pressure;
    pointer.tiltX = event.tiltX;
    pointer.tiltY = event.tiltY;
    pointer.twist = event.twist;
    
    // Gesture güncelle
    this.updateGesture(event.pointerId, pointer);
    
    // Basınç değişikliği algıla
    this.detectPressureChange(pointer);
  }
  
  // Pointer up işle
  handlePointerUp(event) {
    const pointer = this.activePointers.get(event.pointerId);
    if (!pointer) return;
    
    // Gesture tamamla
    this.completeGesture(event.pointerId, pointer);
    
    // Pointer'ı temizle
    this.activePointers.delete(event.pointerId);
    
    console.log('Pointer up:', pointer);
  }
  
  // Pointer cancel işle
  handlePointerCancel(event) {
    const pointer = this.activePointers.get(event.pointerId);
    if (!pointer) return;
    
    // Gesture iptal et
    this.cancelGesture(event.pointerId, pointer);
    
    // Pointer'ı temizle
    this.activePointers.delete(event.pointerId);
    
    console.log('Pointer cancel:', pointer);
  }
  
  // Gesture başlat
  startGesture(pointerId, pointer) {
    const gesture = {
      id: pointerId,
      type: this.detectGestureType(pointer),
      startTime: pointer.startTime,
      startX: pointer.startX,
      startY: pointer.startY,
      currentX: pointer.currentX,
      currentY: pointer.currentY,
      pressure: pointer.pressure,
      isActive: true
    };
    
    this.gestures.set(pointerId, gesture);
  }
  
  // Gesture güncelle
  updateGesture(pointerId, pointer) {
    const gesture = this.gestures.get(pointerId);
    if (!gesture) return;
    
    gesture.currentX = pointer.currentX;
    gesture.currentY = pointer.currentY;
    gesture.pressure = pointer.pressure;
    
    // Gesture türünü yeniden değerlendir
    const newType = this.detectGestureType(pointer);
    if (newType !== gesture.type) {
      gesture.type = newType;
    }
  }
  
  // Gesture tamamla
  completeGesture(pointerId, pointer) {
    const gesture = this.gestures.get(pointerId);
    if (!gesture) return;
    
    gesture.isActive = false;
    gesture.endTime = pointer.startTime;
    gesture.duration = gesture.endTime - gesture.startTime;
    gesture.distance = this.calculateDistance(
      gesture.startX, gesture.startY,
      gesture.currentX, gesture.currentY
    );
    
    // Gesture olayı gönder
    this.dispatchGestureEvent(gesture);
    
    this.gestures.delete(pointerId);
  }
  
  // Gesture iptal et
  cancelGesture(pointerId, pointer) {
    const gesture = this.gestures.get(pointerId);
    if (!gesture) return;
    
    gesture.isActive = false;
    gesture.cancelled = true;
    
    this.gestures.delete(pointerId);
  }
  
  // Gesture türü algıla
  detectGestureType(pointer) {
    const deltaX = Math.abs(pointer.currentX - pointer.startX);
    const deltaY = Math.abs(pointer.currentY - pointer.startY);
    
    if (deltaX < 5 && deltaY < 5) {
      return 'tap';
    } else if (deltaX > deltaY) {
      return 'swipe-horizontal';
    } else if (deltaY > deltaX) {
      return 'swipe-vertical';
    } else {
      return 'drag';
    }
  }
  
  // Basınç değişikliği algıla
  detectPressureChange(pointer) {
    if (pointer.pressure > 0.8) {
      console.log('Güçlü basınç algılandı');
    } else if (pointer.pressure < 0.2) {
      console.log('Hafif basınç algılandı');
    }
  }
  
  // Mesafe hesapla
  calculateDistance(x1, y1, x2, y2) {
    return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
  }
  
  // Gesture olayı gönder
  dispatchGestureEvent(gesture) {
    const event = new CustomEvent('gesture', {
      detail: gesture
    });
    
    this.element.dispatchEvent(event);
  }
  
  // Aktif işaretçi sayısını al
  getActivePointerCount() {
    return this.activePointers.size;
  }
  
  // İşaretçi türü sayısını al
  getPointerTypeCount() {
    const types = {};
    for (const pointer of this.activePointers.values()) {
      types[pointer.type] = (types[pointer.type] || 0) + 1;
    }
    return types;
  }
  
  // Temizlik
  cleanup() {
    this.activePointers.clear();
    this.gestures.clear();
  }
}

// Çoklu dokunmatik yöneticisi
class MultiTouchManager {
  constructor(element) {
    this.element = element;
    this.pointerManager = new PointerManager(element);
    this.touches = new Map();
    this.gestures = new Map();
    this.setupMultiTouch();
  }
  
  // Çoklu dokunmatik kurulumu
  setupMultiTouch() {
    this.element.addEventListener('gesture', (event) => {
      this.handleGesture(event.detail);
    });
  }
  
  // Gesture işle
  handleGesture(gesture) {
    if (gesture.type === 'tap') {
      this.handleTap(gesture);
    } else if (gesture.type === 'swipe-horizontal') {
      this.handleSwipeHorizontal(gesture);
    } else if (gesture.type === 'swipe-vertical') {
      this.handleSwipeVertical(gesture);
    } else if (gesture.type === 'drag') {
      this.handleDrag(gesture);
    }
  }
  
  // Tap işle
  handleTap(gesture) {
    console.log('Tap algılandı:', gesture);
    
    // Çift tap kontrolü
    this.checkDoubleTap(gesture);
  }
  
  // Çift tap kontrolü
  checkDoubleTap(gesture) {
    const now = Date.now();
    const lastTap = this.lastTapTime || 0;
    
    if (now - lastTap < 300) {
      console.log('Çift tap algılandı');
      this.dispatchEvent('doubletap', gesture);
    }
    
    this.lastTapTime = now;
  }
  
  // Yatay kaydırma işle
  handleSwipeHorizontal(gesture) {
    const direction = gesture.currentX > gesture.startX ? 'right' : 'left';
    console.log(`Yatay kaydırma: ${direction}`);
    
    this.dispatchEvent('swipe-horizontal', {
      ...gesture,
      direction: direction
    });
  }
  
  // Dikey kaydırma işle
  handleSwipeVertical(gesture) {
    const direction = gesture.currentY > gesture.startY ? 'down' : 'up';
    console.log(`Dikey kaydırma: ${direction}`);
    
    this.dispatchEvent('swipe-vertical', {
      ...gesture,
      direction: direction
    });
  }
  
  // Sürükleme işle
  handleDrag(gesture) {
    console.log('Sürükleme algılandı:', gesture);
    
    this.dispatchEvent('drag', gesture);
  }
  
  // Event gönder
  dispatchEvent(type, data) {
    const event = new CustomEvent(type, {
      detail: data
    });
    
    this.element.dispatchEvent(event);
  }
}

// Kalem yöneticisi
class PenManager {
  constructor(element) {
    this.element = element;
    this.pointerManager = new PointerManager(element);
    this.penData = new Map();
    this.setupPen();
  }
  
  // Kalem kurulumu
  setupPen() {
    this.element.addEventListener('pointerdown', (event) => {
      if (event.pointerType === 'pen') {
        this.handlePenDown(event);
      }
    });
    
    this.element.addEventListener('pointermove', (event) => {
      if (event.pointerType === 'pen') {
        this.handlePenMove(event);
      }
    });
    
    this.element.addEventListener('pointerup', (event) => {
      if (event.pointerType === 'pen') {
        this.handlePenUp(event);
      }
    });
  }
  
  // Kalem down işle
  handlePenDown(event) {
    const penData = {
      id: event.pointerId,
      pressure: event.pressure,
      tiltX: event.tiltX,
      tiltY: event.tiltY,
      twist: event.twist,
      width: event.width,
      height: event.height,
      startTime: event.timeStamp
    };
    
    this.penData.set(event.pointerId, penData);
    
    console.log('Kalem down:', penData);
  }
  
  // Kalem move işle
  handlePenMove(event) {
    const penData = this.penData.get(event.pointerId);
    if (!penData) return;
    
    // Kalem verilerini güncelle
    penData.pressure = event.pressure;
    penData.tiltX = event.tiltX;
    penData.tiltY = event.tiltY;
    penData.twist = event.twist;
    
    // Kalem özelliklerini analiz et
    this.analyzePenProperties(penData);
  }
  
  // Kalem up işle
  handlePenUp(event) {
    const penData = this.penData.get(event.pointerId);
    if (!penData) return;
    
    penData.endTime = event.timeStamp;
    penData.duration = penData.endTime - penData.startTime;
    
    console.log('Kalem up:', penData);
    
    this.penData.delete(event.pointerId);
  }
  
  // Kalem özelliklerini analiz et
  analyzePenProperties(penData) {
    // Basınç analizi
    if (penData.pressure > 0.8) {
      console.log('Güçlü kalem basıncı');
    }
    
    // Eğim analizi
    if (Math.abs(penData.tiltX) > 30 || Math.abs(penData.tiltY) > 30) {
      console.log('Kalem eğimi algılandı');
    }
    
    // Döndürme analizi
    if (Math.abs(penData.twist) > 45) {
      console.log('Kalem döndürme algılandı');
    }
  }
}
```

**Alternatifler:**
- **Mouse Events**: Geleneksel mouse olayları
- **Touch Events**: Dokunmatik olayları
- **Gesture Events**: Hareket olayları
- **Custom Event Systems**: Özel olay sistemleri
- **Third-party Libraries**: Harici dokunmatik kütüphaneleri

**Ne Zaman Kullanılmalı?**
- ✅ Çoklu cihaz desteği gerektiğinde
- ✅ Gelişmiş dokunmatik deneyim gerektiğinde
- ✅ Kalem/stylus desteği gerektiğinde
- ✅ Basınç algılama gerektiğinde
- ✅ Modern UX gerektiğinde
- ✅ Cross-platform uygulamalarda

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Sadece mouse desteği yeterli olduğunda
- ❌ Basit etkileşim gerektiğinde
- ❌ Performans kritik olduğunda

**Kullanılmazsa Ne Olur?**
- **Cihaz Uyumsuzluğu**: Farklı cihazlarda tutarsız deneyim
- **Gelişmiş Özellik Eksikliği**: Basınç, eğim algılanamaz
- **Kullanıcı Deneyimi**: Kötü dokunmatik deneyim
- **Accessibility**: Erişilebilirlik sorunları
- **Modern UX Eksikliği**: Güncel kullanıcı deneyimi eksikliği

**Modern Alternatifler:**
- **Touch Events**: Dokunmatik olayları
- **Mouse Events**: Mouse olayları
- **Gesture Recognition**: Hareket tanıma
- **Custom Input Systems**: Özel giriş sistemleri
- **Third-party Input Libraries**: Harici giriş kütüphaneleri

### Push API

**Push API Nedir?**
Push API, web uygulamalarının kullanıcılara server'dan push bildirimleri göndermesini sağlayan modern bir web API'sidir. Service Worker'lar ile birlikte çalışarak, uygulama kapalı olsa bile kullanıcılara bildirim gönderebilir. PWA uygulamalarında, gerçek zamanlı uygulamalarda ve kullanıcı engagement'ını artırmak için kritik öneme sahiptir. Web Push Protocol kullanarak güvenli ve güvenilir bildirim sistemi sağlar.

**Neden Kullanılır?**
- **Server Push**: Server'dan bildirim gönderme
- **Offline Bildirimler**: Uygulama kapalıyken bildirim
- **Real-time Updates**: Gerçek zamanlı güncellemeler
- **User Engagement**: Kullanıcı katılımını artırma
- **PWA Desteği**: Progressive Web App özellikleri
- **Cross-Platform**: Platform bağımsız bildirim
- **Scalable**: Ölçeklenebilir bildirim sistemi

**Temel Kullanım Örnekleri:**

```javascript
// Service Worker içinde
self.addEventListener('push', event => {
  const options = {
    body: event.data.text(),
    icon: 'icon.png',
    badge: 'badge.png',
    tag: 'notification-tag',
    requireInteraction: true,
    actions: [
      {
        action: 'view',
        title: 'Görüntüle',
        icon: 'view-icon.png'
      },
      {
        action: 'dismiss',
        title: 'Kapat',
        icon: 'dismiss-icon.png'
      }
    ]
  };
  
  event.waitUntil(
    self.registration.showNotification('Yeni Bildirim', options)
  );
});

// Push aboneliği oluştur
async function subscribeToPush() {
  if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
    throw new Error('Push API desteklenmiyor');
  }
  
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: 'BEl62iUYgUivxIkv69yViEuiBIa40HI8F7V7V-s2b9U'
  });
  
  console.log('Push aboneliği oluşturuldu:', subscription);
  return subscription;
}

// Push aboneliğini iptal et
async function unsubscribeFromPush() {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.getSubscription();
  
  if (subscription) {
    await subscription.unsubscribe();
    console.log('Push aboneliği iptal edildi');
  }
}

// Mevcut aboneliği kontrol et
async function checkPushSubscription() {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.getSubscription();
  
  if (subscription) {
    console.log('Aktif push aboneliği var');
    return subscription;
  } else {
    console.log('Push aboneliği yok');
    return null;
  }
}

// Bildirim tıklama olayı
self.addEventListener('notificationclick', event => {
  event.notification.close();
  
  if (event.action === 'view') {
    // Bildirimi görüntüle
    event.waitUntil(
      clients.openWindow('/notification-detail')
    );
  } else if (event.action === 'dismiss') {
    // Bildirimi kapat
    console.log('Bildirim kapatıldı');
  } else {
    // Bildirime tıklandı
    event.waitUntil(
      clients.openWindow('/')
    );
  }
});

// Bildirim kapatma olayı
self.addEventListener('notificationclose', event => {
  console.log('Bildirim kapatıldı:', event.notification.tag);
});

// Push mesajı işle
self.addEventListener('push', event => {
  let data = {};
  
  if (event.data) {
    try {
      data = event.data.json();
    } catch (e) {
      data = { body: event.data.text() };
    }
  }
  
  const options = {
    body: data.body || 'Yeni bildirim',
    icon: data.icon || '/icon-192.png',
    badge: data.badge || '/badge-72.png',
    tag: data.tag || 'default-tag',
    data: data.data || {},
    requireInteraction: data.requireInteraction || false,
    silent: data.silent || false,
    vibrate: data.vibrate || [200, 100, 200],
    timestamp: Date.now()
  };
  
  event.waitUntil(
    self.registration.showNotification(data.title || 'Bildirim', options)
  );
});

// Push yöneticisi
class PushManager {
  constructor() {
    this.isSupported = 'serviceWorker' in navigator && 'PushManager' in window;
    this.registration = null;
    this.subscription = null;
    this.serverKey = null;
  }
  
  // Push yöneticisini başlat
  async initialize(serverKey) {
    if (!this.isSupported) {
      throw new Error('Push API desteklenmiyor');
    }
    
    this.serverKey = serverKey;
    this.registration = await navigator.serviceWorker.ready;
    this.subscription = await this.registration.pushManager.getSubscription();
    
    return this.subscription;
  }
  
  // Push aboneliği oluştur
  async subscribe() {
    if (!this.registration) {
      throw new Error('Push yöneticisi başlatılmamış');
    }
    
    try {
      this.subscription = await this.registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.serverKey
      });
      
      console.log('Push aboneliği oluşturuldu');
      return this.subscription;
    } catch (error) {
      console.error('Push aboneliği oluşturma hatası:', error);
      throw error;
    }
  }
  
  // Push aboneliğini iptal et
  async unsubscribe() {
    if (!this.subscription) {
      return false;
    }
    
    try {
      const result = await this.subscription.unsubscribe();
      if (result) {
        this.subscription = null;
        console.log('Push aboneliği iptal edildi');
      }
      return result;
    } catch (error) {
      console.error('Push aboneliği iptal etme hatası:', error);
      throw error;
    }
  }
  
  // Abonelik durumunu kontrol et
  isSubscribed() {
    return this.subscription !== null;
  }
  
  // Abonelik bilgilerini al
  getSubscriptionInfo() {
    if (!this.subscription) {
      return null;
    }
    
    return {
      endpoint: this.subscription.endpoint,
      keys: this.subscription.getKey ? {
        p256dh: this.subscription.getKey('p256dh'),
        auth: this.subscription.getKey('auth')
      } : null
    };
  }
  
  // Server'a abonelik bilgilerini gönder
  async sendSubscriptionToServer() {
    if (!this.subscription) {
      throw new Error('Push aboneliği yok');
    }
    
    const subscriptionInfo = this.getSubscriptionInfo();
    
    try {
      const response = await fetch('/api/push/subscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(subscriptionInfo)
      });
      
      if (response.ok) {
        console.log('Abonelik bilgileri server\'a gönderildi');
        return true;
      } else {
        throw new Error('Server hatası: ' + response.status);
      }
    } catch (error) {
      console.error('Abonelik bilgileri gönderme hatası:', error);
      throw error;
    }
  }
}
```

**Alternatifler:**
- **Notifications API**: Yerel bildirimler
- **Service Workers**: Arka plan bildirimleri
- **WebSocket**: Gerçek zamanlı bildirimler
- **Server-Sent Events**: Server'dan bildirimler
- **Third-party Services**: Harici bildirim servisleri

**Ne Zaman Kullanılmalı?**
- ✅ PWA uygulamalarında
- ✅ Gerçek zamanlı uygulamalarda
- ✅ Kullanıcı engagement gerektiğinde
- ✅ Offline bildirim gerektiğinde
- ✅ Server'dan bildirim gerektiğinde
- ✅ Cross-platform bildirim gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit uygulamalarda
- ❌ Sadece yerel bildirim yeterli olduğunda
- ❌ Privacy hassas uygulamalarda

**Kullanılmazsa Ne Olur?**
- **Engagement Eksikliği**: Kullanıcı katılımı azalır
- **Real-time Updates Eksikliği**: Gerçek zamanlı güncellemeler olmaz
- **Offline Communication Eksikliği**: Offline iletişim olmaz
- **PWA Features Eksikliği**: PWA özellikleri eksik kalır
- **User Retention Eksikliği**: Kullanıcı tutma oranı düşer

**Modern Alternatifler:**
- **Web Push Protocol**: Standart push protokolü
- **Firebase Cloud Messaging**: Google'ın push servisi
- **OneSignal**: Harici push servisi
- **Pusher**: Gerçek zamanlı bildirim servisi
- **WebSocket**: Gerçek zamanlı iletişim

---

## R - Bölümü

### Resize Observer API

**Resize Observer API Nedir?**
Resize Observer API, web uygulamalarının DOM elementlerinin boyut değişikliklerini asenkron olarak gözlemlemesini sağlayan modern bir web API'sidir. Element boyutları değiştiğinde callback fonksiyonları tetikler ve performanslı bir şekilde boyut değişikliklerini takip eder. Responsive tasarım, layout hesaplamaları, performans optimizasyonu ve dinamik içerik yönetimi için kritik öneme sahiptir. Mutation Observer'dan daha performanslı ve özel olarak boyut değişiklikleri için optimize edilmiştir.

**Neden Kullanılır?**
- **Responsive Design**: Dinamik boyut değişikliklerini takip etme
- **Layout Calculations**: Layout hesaplamalarını optimize etme
- **Performance**: Performanslı boyut takibi
- **Dynamic Content**: Dinamik içerik yönetimi
- **Viewport Changes**: Viewport değişikliklerini algılama
- **Element Monitoring**: Element boyutlarını izleme
- **Optimization**: Layout thrashing'i önleme

**Temel Kullanım Örnekleri:**

```javascript
// Temel Resize Observer
const resizeObserver = new ResizeObserver(entries => {
  entries.forEach(entry => {
    console.log(`Yeni boyut: ${entry.contentRect.width}x${entry.contentRect.height}`);
    console.log(`Border box: ${entry.borderBoxSize[0].inlineSize}x${entry.borderBoxSize[0].blockSize}`);
    console.log(`Content box: ${entry.contentBoxSize[0].inlineSize}x${entry.contentBoxSize[0].blockSize}`);
  });
});

resizeObserver.observe(document.getElementById('myElement'));

// Çoklu element gözlemi
const elements = document.querySelectorAll('.resizable');
elements.forEach(element => {
  resizeObserver.observe(element);
});

// Boyut değişikliği işleme
const resizeObserver = new ResizeObserver(entries => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    
    // Responsive davranış
    if (width < 768) {
      element.classList.add('mobile');
      element.classList.remove('desktop');
    } else {
      element.classList.add('desktop');
      element.classList.remove('mobile');
    }
    
    // Layout güncelleme
    updateLayout(width, height);
  });
});

// Gözlemlemeyi durdur
resizeObserver.unobserve(element);

// Tüm gözlemleri durdur
resizeObserver.disconnect();

// Viewport boyut değişiklikleri
const viewportObserver = new ResizeObserver(entries => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    
    // Viewport değişikliği işle
    document.documentElement.style.setProperty('--viewport-width', width + 'px');
    document.documentElement.style.setProperty('--viewport-height', height + 'px');
    
    // Breakpoint kontrolü
    updateBreakpoint(width);
  });
});

viewportObserver.observe(document.documentElement);

// Resize Observer yöneticisi
class ResizeObserverManager {
  constructor() {
    this.observers = new Map();
    this.callbacks = new Map();
    this.isSupported = 'ResizeObserver' in window;
  }
  
  // Element gözlemle
  observe(element, callback, options = {}) {
    if (!this.isSupported) {
      console.warn('ResizeObserver desteklenmiyor');
      return null;
    }
    
    const observerId = this.generateObserverId();
    const observer = new ResizeObserver((entries) => {
      this.handleResize(observerId, entries);
    });
    
    this.callbacks.set(observerId, callback);
    this.observers.set(observerId, {
      observer: observer,
      element: element,
      options: options
    });
    
    observer.observe(element, options);
    return observerId;
  }
  
  // Gözlemi durdur
  unobserve(observerId) {
    const observerData = this.observers.get(observerId);
    if (observerData) {
      observerData.observer.unobserve(observerData.element);
      this.observers.delete(observerId);
      this.callbacks.delete(observerId);
    }
  }
  
  // Tüm gözlemleri durdur
  disconnect() {
    this.observers.forEach((observerData) => {
      observerData.observer.disconnect();
    });
    this.observers.clear();
    this.callbacks.clear();
  }
  
  // Resize olayını işle
  handleResize(observerId, entries) {
    const callback = this.callbacks.get(observerId);
    if (callback) {
      try {
        callback(entries);
      } catch (error) {
        console.error('Resize callback hatası:', error);
      }
    }
  }
  
  // Observer ID oluştur
  generateObserverId() {
    return 'observer_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
}
```

**Alternatifler:**
- **Mutation Observer**: DOM değişiklik takibi
- **Intersection Observer**: Görünürlük takibi
- **Window Resize Events**: Pencere boyut değişiklikleri
- **Media Queries**: CSS medya sorguları
- **Custom Resize Detection**: Özel boyut algılama

**Ne Zaman Kullanılmalı?**
- ✅ Responsive tasarım gerektiğinde
- ✅ Dinamik layout gerektiğinde
- ✅ Element boyut takibi gerektiğinde
- ✅ Performans optimizasyonu gerektiğinde
- ✅ Viewport değişiklik takibi gerektiğinde
- ✅ Container boyut takibi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit boyut takibi gerektiğinde
- ❌ Sadece window resize yeterli olduğunda
- ❌ Performans kritik olmadığında

**Kullanılmazsa Ne Olur?**
- **Layout Issues**: Layout sorunları oluşur
- **Responsive Problems**: Responsive tasarım sorunları
- **Performance Issues**: Performans sorunları
- **User Experience**: Kullanıcı deneyimi kötüleşir
- **Dynamic Content Issues**: Dinamik içerik sorunları

**Modern Alternatifler:**
- **CSS Container Queries**: CSS container sorguları
- **CSS Grid/Flexbox**: Modern CSS layout
- **Intersection Observer**: Görünürlük takibi
- **Custom Hooks**: React custom hooks
- **Third-party Libraries**: Harici layout kütüphaneleri

### Reporting API

**Reporting API Nedir?**
Reporting API, web uygulamalarının tarayıcı tarafından oluşturulan raporları (reports) toplamasını ve işlemesini sağlayan modern bir web API'sidir. Tarayıcı güvenlik ihlalleri, performans sorunları, deprecation uyarıları ve diğer önemli olaylar hakkında raporlar oluşturur. Bu raporlar, geliştiricilerin uygulamalarındaki sorunları tespit etmesini ve düzeltmesini sağlar. CSP (Content Security Policy), Feature Policy, Deprecation Reports ve diğer güvenlik/performans raporlarını toplar.

**Neden Kullanılır?**
- **Security Monitoring**: Güvenlik ihlallerini izleme
- **Performance Tracking**: Performans sorunlarını takip etme
- **Deprecation Warnings**: Eski API kullanım uyarıları
- **Error Reporting**: Hata raporlama sistemi
- **Compliance Monitoring**: Uyumluluk izleme
- **Analytics**: Analitik veri toplama
- **Debugging**: Hata ayıklama ve sorun tespiti

**Temel Kullanım Örnekleri:**

```javascript
// Temel Reporting Observer
const observer = new ReportingObserver(reports => {
  reports.forEach(report => {
    console.log('Rapor türü:', report.type);
    console.log('Rapor içeriği:', report.body);
    console.log('URL:', report.url);
    console.log('Zaman:', report.timestamp);
  });
});

observer.observe();

// Rapor türlerine göre işleme
const observer = new ReportingObserver(reports => {
  reports.forEach(report => {
    switch (report.type) {
      case 'csp-violation':
        handleCSPViolation(report.body);
        break;
      case 'deprecation':
        handleDeprecation(report.body);
        break;
      case 'feature-policy-violation':
        handleFeaturePolicyViolation(report.body);
        break;
      case 'intervention':
        handleIntervention(report.body);
        break;
      default:
        console.log('Bilinmeyen rapor türü:', report.type);
    }
  });
});

observer.observe();

// CSP ihlali işleme
function handleCSPViolation(body) {
  console.error('CSP İhlali:', {
    blockedURL: body.blockedURL,
    violatedDirective: body.violatedDirective,
    originalPolicy: body.originalPolicy,
    sourceFile: body.sourceFile,
    lineNumber: body.lineNumber,
    columnNumber: body.columnNumber
  });
  
  // Güvenlik ihlalini logla
  logSecurityViolation(body);
}

// Deprecation uyarısı işleme
function handleDeprecation(body) {
  console.warn('Deprecated API kullanımı:', {
    id: body.id,
    message: body.message,
    sourceFile: body.sourceFile,
    lineNumber: body.lineNumber,
    columnNumber: body.columnNumber
  });
  
  // Deprecation uyarısını takip et
  trackDeprecationUsage(body);
}

// Feature Policy ihlali işleme
function handleFeaturePolicyViolation(body) {
  console.error('Feature Policy İhlali:', {
    featureId: body.featureId,
    message: body.message,
    sourceFile: body.sourceFile,
    lineNumber: body.lineNumber,
    columnNumber: body.columnNumber
  });
}

// Intervention işleme
function handleIntervention(body) {
  console.warn('Tarayıcı müdahalesi:', {
    id: body.id,
    message: body.message,
    sourceFile: body.sourceFile,
    lineNumber: body.lineNumber,
    columnNumber: body.columnNumber
  });
}

// Rapor gözlemci yöneticisi
class ReportingManager {
  constructor() {
    this.observers = new Map();
    this.reportHandlers = new Map();
    this.isSupported = 'ReportingObserver' in window;
  }
  
  // Rapor gözlemci oluştur
  createObserver(name, options = {}) {
    if (!this.isSupported) {
      console.warn('ReportingObserver desteklenmiyor');
      return null;
    }
    
    const observer = new ReportingObserver((reports) => {
      this.handleReports(name, reports);
    }, options);
    
    this.observers.set(name, observer);
    return observer;
  }
  
  // Rapor işleyici ekle
  addReportHandler(reportType, handler) {
    if (!this.reportHandlers.has(reportType)) {
      this.reportHandlers.set(reportType, []);
    }
    this.reportHandlers.get(reportType).push(handler);
  }
  
  // Raporları işle
  handleReports(observerName, reports) {
    reports.forEach(report => {
      const handlers = this.reportHandlers.get(report.type) || [];
      handlers.forEach(handler => {
        try {
          handler(report, observerName);
        } catch (error) {
          console.error('Rapor işleyici hatası:', error);
        }
      });
    });
  }
  
  // Gözlemciyi başlat
  startObserver(name) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.observe();
      console.log('Rapor gözlemci başlatıldı:', name);
    }
  }
  
  // Gözlemciyi durdur
  stopObserver(name) {
    const observer = this.observers.get(name);
    if (observer) {
      observer.disconnect();
      console.log('Rapor gözlemci durduruldu:', name);
    }
  }
  
  // Tüm gözlemcileri durdur
  stopAllObservers() {
    this.observers.forEach((observer, name) => {
      observer.disconnect();
    });
    console.log('Tüm rapor gözlemcileri durduruldu');
  }
}

// Güvenlik rapor yöneticisi
class SecurityReportingManager {
  constructor() {
    this.reportingManager = new ReportingManager();
    this.violations = [];
    this.setupSecurityObserver();
  }
  
  // Güvenlik gözlemci kurulumu
  setupSecurityObserver() {
    const observer = this.reportingManager.createObserver('security');
    
    // CSP ihlali işleyici
    this.reportingManager.addReportHandler('csp-violation', (report) => {
      this.handleCSPViolation(report);
    });
    
    // Feature Policy ihlali işleyici
    this.reportingManager.addReportHandler('feature-policy-violation', (report) => {
      this.handleFeaturePolicyViolation(report);
    });
    
    // Gözlemciyi başlat
    this.reportingManager.startObserver('security');
  }
  
  // CSP ihlali işle
  handleCSPViolation(report) {
    const violation = {
      type: 'csp-violation',
      timestamp: new Date().toISOString(),
      url: report.url,
      blockedURL: report.body.blockedURL,
      violatedDirective: report.body.violatedDirective,
      originalPolicy: report.body.originalPolicy,
      sourceFile: report.body.sourceFile,
      lineNumber: report.body.lineNumber,
      columnNumber: report.body.columnNumber
    };
    
    this.violations.push(violation);
    this.logViolation(violation);
    this.sendViolationToServer(violation);
  }
  
  // Feature Policy ihlali işle
  handleFeaturePolicyViolation(report) {
    const violation = {
      type: 'feature-policy-violation',
      timestamp: new Date().toISOString(),
      url: report.url,
      featureId: report.body.featureId,
      message: report.body.message,
      sourceFile: report.body.sourceFile,
      lineNumber: report.body.lineNumber,
      columnNumber: report.body.columnNumber
    };
    
    this.violations.push(violation);
    this.logViolation(violation);
    this.sendViolationToServer(violation);
  }
  
  // İhlali logla
  logViolation(violation) {
    console.error('Güvenlik ihlali:', violation);
  }
  
  // İhlali sunucuya gönder
  sendViolationToServer(violation) {
    fetch('/api/security-violations', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(violation)
    }).catch(error => {
      console.error('İhlal raporu gönderilemedi:', error);
    });
  }
  
  // İhlalleri al
  getViolations() {
    return this.violations;
  }
  
  // İhlalleri temizle
  clearViolations() {
    this.violations = [];
  }
}

// Performans rapor yöneticisi
class PerformanceReportingManager {
  constructor() {
    this.reportingManager = new ReportingManager();
    this.deprecations = [];
    this.interventions = [];
    this.setupPerformanceObserver();
  }
  
  // Performans gözlemci kurulumu
  setupPerformanceObserver() {
    const observer = this.reportingManager.createObserver('performance');
    
    // Deprecation işleyici
    this.reportingManager.addReportHandler('deprecation', (report) => {
      this.handleDeprecation(report);
    });
    
    // Intervention işleyici
    this.reportingManager.addReportHandler('intervention', (report) => {
      this.handleIntervention(report);
    });
    
    // Gözlemciyi başlat
    this.reportingManager.startObserver('performance');
  }
  
  // Deprecation işle
  handleDeprecation(report) {
    const deprecation = {
      type: 'deprecation',
      timestamp: new Date().toISOString(),
      url: report.url,
      id: report.body.id,
      message: report.body.message,
      sourceFile: report.body.sourceFile,
      lineNumber: report.body.lineNumber,
      columnNumber: report.body.columnNumber
    };
    
    this.deprecations.push(deprecation);
    this.logDeprecation(deprecation);
    this.sendDeprecationToServer(deprecation);
  }
  
  // Intervention işle
  handleIntervention(report) {
    const intervention = {
      type: 'intervention',
      timestamp: new Date().toISOString(),
      url: report.url,
      id: report.body.id,
      message: report.body.message,
      sourceFile: report.body.sourceFile,
      lineNumber: report.body.lineNumber,
      columnNumber: report.body.columnNumber
    };
    
    this.interventions.push(intervention);
    this.logIntervention(intervention);
    this.sendInterventionToServer(intervention);
  }
  
  // Deprecation logla
  logDeprecation(deprecation) {
    console.warn('Deprecated API kullanımı:', deprecation);
  }
  
  // Intervention logla
  logIntervention(intervention) {
    console.warn('Tarayıcı müdahalesi:', intervention);
  }
  
  // Deprecation'ı sunucuya gönder
  sendDeprecationToServer(deprecation) {
    fetch('/api/deprecations', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(deprecation)
    }).catch(error => {
      console.error('Deprecation raporu gönderilemedi:', error);
    });
  }
  
  // Intervention'ı sunucuya gönder
  sendInterventionToServer(intervention) {
    fetch('/api/interventions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(intervention)
    }).catch(error => {
      console.error('Intervention raporu gönderilemedi:', error);
    });
  }
  
  // Deprecation'ları al
  getDeprecations() {
    return this.deprecations;
  }
  
  // Intervention'ları al
  getInterventions() {
    return this.interventions;
  }
  
  // Raporları temizle
  clearReports() {
    this.deprecations = [];
    this.interventions = [];
  }
}
```

**Alternatifler:**
- **Console API**: Konsol logları
- **Error Tracking**: Hata takip sistemleri
- **Analytics**: Analitik servisleri
- **Custom Error Handlers**: Özel hata işleyicileri
- **Third-party Monitoring**: Harici izleme servisleri

**Ne Zaman Kullanılmalı?**
- ✅ Güvenlik ihlallerini izlemek gerektiğinde
- ✅ Performans sorunlarını takip etmek gerektiğinde
- ✅ Deprecated API kullanımını tespit etmek gerektiğinde
- ✅ Uyumluluk izleme gerektiğinde
- ✅ Hata raporlama sistemi gerektiğinde
- ✅ Analitik veri toplama gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit hata takibi yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ Güvenlik gereksinimleri düşük olduğunda

**Kullanılmazsa Ne Olur?**
- **Security Issues**: Güvenlik sorunları tespit edilemez
- **Performance Problems**: Performans sorunları fark edilmez
- **Deprecation Warnings**: Eski API kullanımı tespit edilmez
- **Compliance Issues**: Uyumluluk sorunları fark edilmez
- **Error Tracking**: Hata takibi yapılamaz

**Modern Alternatifler:**
- **Sentry**: Hata takip servisi
- **Bugsnag**: Hata raporlama servisi
- **LogRocket**: Kullanıcı deneyimi izleme
- **DataDog**: Uygulama performans izleme
- **New Relic**: Uygulama performans yönetimi

---

## S - Bölümü

### Screen Capture API

**Screen Capture API Nedir?**
Screen Capture API, web uygulamalarının kullanıcının ekranını, pencerelerini veya sekmelerini yakalamasını sağlayan modern bir web API'sidir. Kullanıcı izni ile ekran paylaşımı, ekran kaydı, uzaktan eğitim, video konferans ve ekran görüntüsü alma gibi işlevleri destekler. getDisplayMedia() metodu ile ekran, pencere veya sekme yakalama seçenekleri sunar. WebRTC teknolojisi ile entegre çalışır ve gerçek zamanlı ekran paylaşımı sağlar.

**Neden Kullanılır?**
- **Screen Sharing**: Ekran paylaşımı
- **Screen Recording**: Ekran kaydı
- **Remote Education**: Uzaktan eğitim
- **Video Conferencing**: Video konferans
- **Screenshot Capture**: Ekran görüntüsü alma
- **Remote Support**: Uzaktan destek
- **Live Streaming**: Canlı yayın

**Temel Kullanım Örnekleri:**

```javascript
// Temel ekran yakalama
const stream = await navigator.mediaDevices.getDisplayMedia({
  video: {mediaSource: 'screen'}
});

const video = document.getElementById('video');
video.srcObject = stream;

// Gelişmiş ekran yakalama seçenekleri
const stream = await navigator.mediaDevices.getDisplayMedia({
  video: {
    mediaSource: 'screen',
    width: { ideal: 1920 },
    height: { ideal: 1080 },
    frameRate: { ideal: 30 }
  },
  audio: true
});

// Ekran yakalama türleri
const screenStream = await navigator.mediaDevices.getDisplayMedia({
  video: { mediaSource: 'screen' }
});

const windowStream = await navigator.mediaDevices.getDisplayMedia({
  video: { mediaSource: 'window' }
});

const tabStream = await navigator.mediaDevices.getDisplayMedia({
  video: { mediaSource: 'browser' }
});

// Ekran yakalama yöneticisi
class ScreenCaptureManager {
  constructor() {
    this.currentStream = null;
    this.isSupported = 'getDisplayMedia' in navigator.mediaDevices;
  }
  
  // Ekran yakalama başlat
  async startCapture(options = {}) {
    if (!this.isSupported) {
      throw new Error('Screen Capture API desteklenmiyor');
    }
    
    try {
      const defaultOptions = {
        video: {
          mediaSource: 'screen',
          width: { ideal: 1920 },
          height: { ideal: 1080 },
          frameRate: { ideal: 30 }
        },
        audio: false
      };
      
      const captureOptions = { ...defaultOptions, ...options };
      this.currentStream = await navigator.mediaDevices.getDisplayMedia(captureOptions);
      
      // Stream olaylarını dinle
      this.setupStreamEvents();
      
      return this.currentStream;
    } catch (error) {
      console.error('Ekran yakalama hatası:', error);
      throw error;
    }
  }
  
  // Stream olaylarını kur
  setupStreamEvents() {
    if (this.currentStream) {
      this.currentStream.getVideoTracks().forEach(track => {
        track.addEventListener('ended', () => {
          this.stopCapture();
        });
      });
    }
  }
  
  // Ekran yakalama durdur
  stopCapture() {
    if (this.currentStream) {
      this.currentStream.getTracks().forEach(track => {
        track.stop();
      });
      this.currentStream = null;
    }
  }
  
  // Mevcut stream'i al
  getCurrentStream() {
    return this.currentStream;
  }
  
  // Yakalama durumunu kontrol et
  isCapturing() {
    return this.currentStream !== null;
  }
}

// Ekran kayıt yöneticisi
class ScreenRecorder {
  constructor() {
    this.mediaRecorder = null;
    this.recordedChunks = [];
    this.isRecording = false;
  }
  
  // Kayıt başlat
  async startRecording(options = {}) {
    const screenCaptureManager = new ScreenCaptureManager();
    const stream = await screenCaptureManager.startCapture(options);
    
    this.mediaRecorder = new MediaRecorder(stream, {
      mimeType: 'video/webm;codecs=vp9'
    });
    
    this.mediaRecorder.ondataavailable = (event) => {
      if (event.data.size > 0) {
        this.recordedChunks.push(event.data);
      }
    };
    
    this.mediaRecorder.onstop = () => {
      this.createDownloadLink();
    };
    
    this.mediaRecorder.start();
    this.isRecording = true;
    
    return screenCaptureManager;
  }
  
  // Kayıt durdur
  stopRecording() {
    if (this.mediaRecorder && this.isRecording) {
      this.mediaRecorder.stop();
      this.isRecording = false;
    }
  }
  
  // İndirme linki oluştur
  createDownloadLink() {
    const blob = new Blob(this.recordedChunks, { type: 'video/webm' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = `screen-recording-${Date.now()}.webm`;
    a.click();
    
    URL.revokeObjectURL(url);
  }
  
  // Kayıt durumunu kontrol et
  isCurrentlyRecording() {
    return this.isRecording;
  }
}

// Ekran paylaşım yöneticisi
class ScreenShareManager {
  constructor() {
    this.peerConnection = null;
    this.localStream = null;
    this.remoteStreams = new Map();
  }
  
  // Ekran paylaşımı başlat
  async startScreenShare() {
    try {
      this.localStream = await navigator.mediaDevices.getDisplayMedia({
        video: { mediaSource: 'screen' },
        audio: true
      });
      
      // WebRTC bağlantısı kur
      await this.setupPeerConnection();
      
      // Stream'i peer connection'a ekle
      this.localStream.getTracks().forEach(track => {
        this.peerConnection.addTrack(track, this.localStream);
      });
      
      return this.localStream;
    } catch (error) {
      console.error('Ekran paylaşımı hatası:', error);
      throw error;
    }
  }
  
  // Peer connection kur
  async setupPeerConnection() {
    this.peerConnection = new RTCPeerConnection({
      iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
    });
    
    // Remote stream olaylarını dinle
    this.peerConnection.ontrack = (event) => {
      const [remoteStream] = event.streams;
      this.remoteStreams.set(event.track.id, remoteStream);
      this.onRemoteStream(remoteStream);
    };
  }
  
  // Remote stream işle
  onRemoteStream(stream) {
    const video = document.createElement('video');
    video.srcObject = stream;
    video.autoplay = true;
    video.controls = true;
    document.body.appendChild(video);
  }
  
  // Ekran paylaşımı durdur
  stopScreenShare() {
    if (this.localStream) {
      this.localStream.getTracks().forEach(track => {
        track.stop();
      });
      this.localStream = null;
    }
    
    if (this.peerConnection) {
      this.peerConnection.close();
      this.peerConnection = null;
    }
  }
}

// Ekran görüntüsü yöneticisi
class ScreenshotManager {
  constructor() {
    this.canvas = document.createElement('canvas');
    this.ctx = this.canvas.getContext('2d');
  }
  
  // Ekran görüntüsü al
  async takeScreenshot() {
    try {
      const stream = await navigator.mediaDevices.getDisplayMedia({
        video: { mediaSource: 'screen' }
      });
      
      const video = document.createElement('video');
      video.srcObject = stream;
      video.play();
      
      return new Promise((resolve) => {
        video.onloadedmetadata = () => {
          this.canvas.width = video.videoWidth;
          this.canvas.height = video.videoHeight;
          
          this.ctx.drawImage(video, 0, 0);
          
          // Stream'i durdur
          stream.getTracks().forEach(track => track.stop());
          
          // Canvas'ı blob'a çevir
          this.canvas.toBlob((blob) => {
            resolve(blob);
          }, 'image/png');
        };
      });
    } catch (error) {
      console.error('Ekran görüntüsü hatası:', error);
      throw error;
    }
  }
  
  // Ekran görüntüsünü indir
  async downloadScreenshot() {
    const blob = await this.takeScreenshot();
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = `screenshot-${Date.now()}.png`;
    a.click();
    
    URL.revokeObjectURL(url);
  }
  
  // Ekran görüntüsünü base64'e çevir
  async getScreenshotAsBase64() {
    const blob = await this.takeScreenshot();
    return new Promise((resolve) => {
      const reader = new FileReader();
      reader.onload = () => resolve(reader.result);
      reader.readAsDataURL(blob);
    });
  }
}
```

**Alternatifler:**
- **Canvas API**: Canvas ile ekran çizimi
- **WebRTC**: Gerçek zamanlı iletişim
- **MediaStream API**: Medya stream yönetimi
- **File API**: Dosya işlemleri
- **Third-party Libraries**: Harici ekran yakalama kütüphaneleri

**Ne Zaman Kullanılmalı?**
- ✅ Ekran paylaşımı gerektiğinde
- ✅ Ekran kaydı gerektiğinde
- ✅ Uzaktan eğitim gerektiğinde
- ✅ Video konferans gerektiğinde
- ✅ Ekran görüntüsü alma gerektiğinde
- ✅ Uzaktan destek gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik kısıtlamaları olduğunda
- ❌ Kullanıcı izni alınamadığında
- ❌ HTTPS gereksinimleri karşılanmadığında

**Kullanılmazsa Ne Olur?**
- **No Screen Sharing**: Ekran paylaşımı yapılamaz
- **No Screen Recording**: Ekran kaydı yapılamaz
- **Limited Remote Support**: Uzaktan destek sınırlı olur
- **No Live Streaming**: Canlı yayın yapılamaz
- **Poor User Experience**: Kullanıcı deneyimi kötüleşir

**Modern Alternatifler:**
- **WebRTC**: Gerçek zamanlı iletişim
- **MediaStream API**: Medya stream yönetimi
- **Canvas API**: Canvas ile çizim
- **Third-party Services**: Harici ekran paylaşım servisleri
- **Native Apps**: Yerel uygulamalar

### Screen Orientation API

**Screen Orientation API Nedir?**
Screen Orientation API, web uygulamalarının cihazın ekran yönelimini (portrait/landscape) okumasını ve kontrol etmesini sağlayan modern bir web API'sidir. Cihazın döndürülmesi durumunda yönelim değişikliklerini algılar ve uygulamaların bu değişikliklere uyum sağlamasını sağlar. Responsive tasarım, mobil uygulama geliştirme ve kullanıcı deneyimi optimizasyonu için kritik öneme sahiptir. Ekran yönelimini programatik olarak değiştirme, yönelim kilitleme ve yönelim değişiklik olaylarını dinleme özellikleri sunar.

**Neden Kullanılır?**
- **Responsive Design**: Responsive tasarım uyumluluğu
- **Mobile Optimization**: Mobil optimizasyon
- **User Experience**: Kullanıcı deneyimi iyileştirme
- **Layout Adaptation**: Layout uyarlama
- **Orientation Locking**: Yönelim kilitleme
- **Event Handling**: Yönelim değişiklik olayları
- **Device Detection**: Cihaz yönelim tespiti

**Temel Kullanım Örnekleri:**

```javascript
// Temel yönelim kontrolü
if ('orientation' in screen) {
  console.log(`Mevcut yönelim: ${screen.orientation.type}`);
  console.log(`Yönelim açısı: ${screen.orientation.angle}`);
  
  // Yönelim değişiklik olayını dinle
  screen.orientation.addEventListener('change', () => {
    console.log(`Yeni yönelim: ${screen.orientation.type}`);
    console.log(`Yeni açı: ${screen.orientation.angle}`);
  });
}

// Yönelim türlerini kontrol et
function getOrientationType() {
  if ('orientation' in screen) {
    return screen.orientation.type;
  }
  return 'unknown';
}

// Yönelim açısını al
function getOrientationAngle() {
  if ('orientation' in screen) {
    return screen.orientation.angle;
  }
  return 0;
}

// Yönelim değişiklik olayı
screen.orientation.addEventListener('change', (event) => {
  const orientation = event.target;
  console.log('Yönelim değişti:', {
    type: orientation.type,
    angle: orientation.angle
  });
  
  // Yönelim değişikliğine göre layout güncelle
  updateLayoutForOrientation(orientation.type);
});

// Layout güncelleme
function updateLayoutForOrientation(orientationType) {
  const body = document.body;
  
  switch (orientationType) {
    case 'portrait-primary':
    case 'portrait-secondary':
      body.classList.add('portrait');
      body.classList.remove('landscape');
      break;
    case 'landscape-primary':
    case 'landscape-secondary':
      body.classList.add('landscape');
      body.classList.remove('portrait');
      break;
  }
}

// Yönelim kilitleme
async function lockOrientation(orientation) {
  if ('orientation' in screen && screen.orientation.lock) {
    try {
      await screen.orientation.lock(orientation);
      console.log('Yönelim kilitlendi:', orientation);
    } catch (error) {
      console.error('Yönelim kilitleme hatası:', error);
    }
  }
}

// Yönelim kilidini kaldır
function unlockOrientation() {
  if ('orientation' in screen && screen.orientation.unlock) {
    screen.orientation.unlock();
    console.log('Yönelim kilidi kaldırıldı');
  }
}

// Yönelim yöneticisi
class OrientationManager {
  constructor() {
    this.isSupported = 'orientation' in screen;
    this.currentOrientation = null;
    this.listeners = new Map();
    this.setupOrientationListener();
  }
  
  // Yönelim dinleyicisi kur
  setupOrientationListener() {
    if (!this.isSupported) {
      console.warn('Screen Orientation API desteklenmiyor');
      return;
    }
    
    screen.orientation.addEventListener('change', (event) => {
      this.handleOrientationChange(event);
    });
    
    // İlk yönelim değerini al
    this.currentOrientation = {
      type: screen.orientation.type,
      angle: screen.orientation.angle
    };
  }
  
  // Yönelim değişikliğini işle
  handleOrientationChange(event) {
    const orientation = event.target;
    const newOrientation = {
      type: orientation.type,
      angle: orientation.angle
    };
    
    const oldOrientation = this.currentOrientation;
    this.currentOrientation = newOrientation;
    
    // Dinleyicileri bilgilendir
    this.notifyListeners(oldOrientation, newOrientation);
  }
  
  // Dinleyici ekle
  addListener(name, callback) {
    this.listeners.set(name, callback);
  }
  
  // Dinleyici kaldır
  removeListener(name) {
    this.listeners.delete(name);
  }
  
  // Dinleyicileri bilgilendir
  notifyListeners(oldOrientation, newOrientation) {
    this.listeners.forEach((callback, name) => {
      try {
        callback(oldOrientation, newOrientation, name);
      } catch (error) {
        console.error('Yönelim dinleyici hatası:', error);
      }
    });
  }
  
  // Mevcut yönelim
  getCurrentOrientation() {
    return this.currentOrientation;
  }
  
  // Yönelim türü
  getOrientationType() {
    return this.currentOrientation ? this.currentOrientation.type : 'unknown';
  }
  
  // Yönelim açısı
  getOrientationAngle() {
    return this.currentOrientation ? this.currentOrientation.angle : 0;
  }
  
  // Portrait mi kontrol et
  isPortrait() {
    const type = this.getOrientationType();
    return type.includes('portrait');
  }
  
  // Landscape mi kontrol et
  isLandscape() {
    const type = this.getOrientationType();
    return type.includes('landscape');
  }
  
  // Yönelim kilitle
  async lockOrientation(orientation) {
    if (!this.isSupported || !screen.orientation.lock) {
      throw new Error('Yönelim kilitleme desteklenmiyor');
    }
    
    try {
      await screen.orientation.lock(orientation);
      return true;
    } catch (error) {
      console.error('Yönelim kilitleme hatası:', error);
      return false;
    }
  }
  
  // Yönelim kilidini kaldır
  unlockOrientation() {
    if (this.isSupported && screen.orientation.unlock) {
      screen.orientation.unlock();
    }
  }
}

// Responsive yönelim yöneticisi
class ResponsiveOrientationManager {
  constructor() {
    this.orientationManager = new OrientationManager();
    this.breakpoints = {
      mobile: 768,
      tablet: 1024,
      desktop: 1200
    };
    this.setupResponsiveListener();
  }
  
  // Responsive dinleyici kur
  setupResponsiveListener() {
    this.orientationManager.addListener('responsive', (oldOrientation, newOrientation) => {
      this.handleResponsiveChange(oldOrientation, newOrientation);
    });
  }
  
  // Responsive değişiklik işle
  handleResponsiveChange(oldOrientation, newOrientation) {
    const isPortrait = this.orientationManager.isPortrait();
    const isLandscape = this.orientationManager.isLandscape();
    
    // CSS sınıflarını güncelle
    this.updateCSSClasses(isPortrait, isLandscape);
    
    // Layout'u güncelle
    this.updateLayout(isPortrait, isLandscape);
    
    // Event gönder
    this.dispatchOrientationEvent(newOrientation);
  }
  
  // CSS sınıflarını güncelle
  updateCSSClasses(isPortrait, isLandscape) {
    const body = document.body;
    
    if (isPortrait) {
      body.classList.add('portrait');
      body.classList.remove('landscape');
    } else if (isLandscape) {
      body.classList.add('landscape');
      body.classList.remove('portrait');
    }
  }
  
  // Layout güncelle
  updateLayout(isPortrait, isLandscape) {
    const viewport = {
      width: window.innerWidth,
      height: window.innerHeight
    };
    
    if (isPortrait) {
      this.updatePortraitLayout(viewport);
    } else if (isLandscape) {
      this.updateLandscapeLayout(viewport);
    }
  }
  
  // Portrait layout güncelle
  updatePortraitLayout(viewport) {
    // Portrait için özel layout mantığı
    console.log('Portrait layout güncellendi:', viewport);
  }
  
  // Landscape layout güncelle
  updateLandscapeLayout(viewport) {
    // Landscape için özel layout mantığı
    console.log('Landscape layout güncellendi:', viewport);
  }
  
  // Yönelim olayı gönder
  dispatchOrientationEvent(orientation) {
    const event = new CustomEvent('orientationchange', {
      detail: {
        type: orientation.type,
        angle: orientation.angle,
        isPortrait: this.orientationManager.isPortrait(),
        isLandscape: this.orientationManager.isLandscape()
      }
    });
    
    document.dispatchEvent(event);
  }
  
  // Yönelim kilitle
  async lockToPortrait() {
    return await this.orientationManager.lockOrientation('portrait');
  }
  
  // Landscape'e kilitle
  async lockToLandscape() {
    return await this.orientationManager.lockOrientation('landscape');
  }
  
  // Yönelim kilidini kaldır
  unlockOrientation() {
    this.orientationManager.unlockOrientation();
  }
}

// Yönelim animasyon yöneticisi
class OrientationAnimationManager {
  constructor() {
    this.orientationManager = new OrientationManager();
    this.animations = new Map();
    this.setupAnimationListener();
  }
  
  // Animasyon dinleyici kur
  setupAnimationListener() {
    this.orientationManager.addListener('animation', (oldOrientation, newOrientation) => {
      this.handleAnimationChange(oldOrientation, newOrientation);
    });
  }
  
  // Animasyon değişiklik işle
  handleAnimationChange(oldOrientation, newOrientation) {
    const isPortrait = this.orientationManager.isPortrait();
    const isLandscape = this.orientationManager.isLandscape();
    
    // Animasyonları güncelle
    this.updateAnimations(isPortrait, isLandscape);
  }
  
  // Animasyon ekle
  addAnimation(name, portraitAnimation, landscapeAnimation) {
    this.animations.set(name, {
      portrait: portraitAnimation,
      landscape: landscapeAnimation
    });
  }
  
  // Animasyonları güncelle
  updateAnimations(isPortrait, isLandscape) {
    this.animations.forEach((animation, name) => {
      if (isPortrait && animation.portrait) {
        this.playAnimation(name, animation.portrait);
      } else if (isLandscape && animation.landscape) {
        this.playAnimation(name, animation.landscape);
      }
    });
  }
  
  // Animasyon oynat
  playAnimation(name, animation) {
    console.log('Animasyon oynatılıyor:', name, animation);
    // Animasyon mantığı burada implement edilir
  }
}
```

**Alternatifler:**
- **CSS Media Queries**: CSS medya sorguları
- **Window Resize Events**: Pencere boyut değişiklikleri
- **Viewport Meta Tag**: Viewport meta etiketi
- **Device Orientation Events**: Cihaz yönelim olayları
- **Custom Detection**: Özel yönelim algılama

**Ne Zaman Kullanılmalı?**
- ✅ Mobil uygulama geliştirme gerektiğinde
- ✅ Responsive tasarım gerektiğinde
- ✅ Yönelim kilitleme gerektiğinde
- ✅ Yönelim değişiklik takibi gerektiğinde
- ✅ Layout uyarlama gerektiğinde
- ✅ Kullanıcı deneyimi optimizasyonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Sadece CSS yeterli olduğunda
- ❌ Yönelim kontrolü gereksiz olduğunda
- ❌ Performans kritik olduğunda

**Kullanılmazsa Ne Olur?**
- **Layout Issues**: Layout sorunları oluşur
- **Responsive Problems**: Responsive tasarım sorunları
- **User Experience**: Kullanıcı deneyimi kötüleşir
- **Mobile Issues**: Mobil uyumluluk sorunları
- **Orientation Problems**: Yönelim sorunları

**Modern Alternatifler:**
- **CSS Container Queries**: CSS container sorguları
- **CSS Grid/Flexbox**: Modern CSS layout
- **Viewport Units**: Viewport birimleri
- **CSS Custom Properties**: CSS özel özellikleri
- **Progressive Web Apps**: PWA teknolojileri

### Selection API

**Selection API Nedir?**
Selection API, web uygulamalarının kullanıcının seçtiği metinleri okumasını, manipüle etmesini ve seçim olaylarını dinlemesini sağlayan modern bir web API'sidir. Kullanıcı metin seçtiğinde, seçimi değiştirdiğinde veya seçimi kaldırdığında bu olayları yakalar. Metin editörleri, rich text editörler, kopyala-yapıştır işlemleri, metin analizi ve kullanıcı etkileşimi için kritik öneme sahiptir. Seçilen metnin içeriğini, konumunu, başlangıç ve bitiş noktalarını kontrol eder.

**Neden Kullanılır?**
- **Text Selection**: Metin seçimi kontrolü
- **Rich Text Editors**: Zengin metin editörleri
- **Copy-Paste Operations**: Kopyala-yapıştır işlemleri
- **Text Analysis**: Metin analizi
- **User Interaction**: Kullanıcı etkileşimi
- **Content Manipulation**: İçerik manipülasyonu
- **Selection Events**: Seçim olayları

**Temel Kullanım Örnekleri:**

```javascript
// Temel seçim kontrolü
const selection = window.getSelection();
console.log(`Seçilen metin: ${selection.toString()}`);
console.log(`Seçim aralığı: ${selection.rangeCount}`);

// Seçim değişiklik olayı
selection.addEventListener('change', () => {
  console.log('Seçim değişti');
  console.log(`Yeni seçim: ${selection.toString()}`);
});

// Seçim temizleme
selection.removeAllRanges();

// Seçim bilgilerini al
function getSelectionInfo() {
  const selection = window.getSelection();
  return {
    text: selection.toString(),
    rangeCount: selection.rangeCount,
    isCollapsed: selection.isCollapsed,
    anchorNode: selection.anchorNode,
    anchorOffset: selection.anchorOffset,
    focusNode: selection.focusNode,
    focusOffset: selection.focusOffset
  };
}

// Seçim yöneticisi
class SelectionManager {
  constructor() {
    this.selection = window.getSelection();
    this.listeners = new Map();
    this.setupSelectionListener();
  }
  
  // Seçim dinleyicisi kur
  setupSelectionListener() {
    this.selection.addEventListener('change', () => {
      this.handleSelectionChange();
    });
    
    document.addEventListener('mouseup', () => {
      this.handleSelectionComplete();
    });
    
    document.addEventListener('keyup', (event) => {
      if (event.key === 'Escape') {
        this.clearSelection();
      }
    });
  }
  
  // Seçim değişikliğini işle
  handleSelectionChange() {
    const selectionInfo = this.getSelectionInfo();
    this.notifyListeners('change', selectionInfo);
  }
  
  // Seçim tamamlandığında işle
  handleSelectionComplete() {
    const selectionInfo = this.getSelectionInfo();
    if (selectionInfo.text.length > 0) {
      this.notifyListeners('complete', selectionInfo);
    }
  }
  
  // Dinleyici ekle
  addListener(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }
  
  // Dinleyicileri bilgilendir
  notifyListeners(event, data) {
    if (this.listeners.has(event)) {
      this.listeners.get(event).forEach(callback => {
        try {
          callback(data);
        } catch (error) {
          console.error('Seçim dinleyici hatası:', error);
        }
      });
    }
  }
  
  // Seçim bilgilerini al
  getSelectionInfo() {
    return {
      text: this.selection.toString(),
      rangeCount: this.selection.rangeCount,
      isCollapsed: this.selection.isCollapsed,
      anchorNode: this.selection.anchorNode,
      anchorOffset: this.selection.anchorOffset,
      focusNode: this.selection.focusNode,
      focusOffset: this.selection.focusOffset
    };
  }
  
  // Seçimi temizle
  clearSelection() {
    this.selection.removeAllRanges();
    this.notifyListeners('clear', null);
  }
  
  // Metin seç
  selectText(element, startOffset = 0, endOffset = null) {
    const range = document.createRange();
    const textNode = element.firstChild;
    
    if (textNode) {
      range.setStart(textNode, startOffset);
      range.setEnd(textNode, endOffset || textNode.textContent.length);
      
      this.selection.removeAllRanges();
      this.selection.addRange(range);
    }
  }
}

// Metin editör yöneticisi
class TextEditorManager {
  constructor(editorElement) {
    this.editor = editorElement;
    this.selectionManager = new SelectionManager();
    this.setupEditor();
  }
  
  // Editör kurulumu
  setupEditor() {
    this.selectionManager.addListener('change', (selectionInfo) => {
      this.handleSelectionChange(selectionInfo);
    });
    
    this.selectionManager.addListener('complete', (selectionInfo) => {
      this.handleSelectionComplete(selectionInfo);
    });
  }
  
  // Seçim değişikliğini işle
  handleSelectionChange(selectionInfo) {
    this.updateToolbar(selectionInfo);
    this.updateSelectionStyles(selectionInfo);
  }
  
  // Seçim tamamlandığında işle
  handleSelectionComplete(selectionInfo) {
    this.showSelectionMenu(selectionInfo);
    this.analyzeSelection(selectionInfo);
  }
  
  // Araç çubuğunu güncelle
  updateToolbar(selectionInfo) {
    const toolbar = document.getElementById('editor-toolbar');
    if (toolbar) {
      const hasSelection = selectionInfo.text.length > 0;
      toolbar.classList.toggle('has-selection', hasSelection);
    }
  }
  
  // Seçim stillerini güncelle
  updateSelectionStyles(selectionInfo) {
    if (selectionInfo.text.length > 0) {
      this.editor.classList.add('has-selection');
    } else {
      this.editor.classList.remove('has-selection');
    }
  }
  
  // Seçim menüsünü göster
  showSelectionMenu(selectionInfo) {
    console.log('Seçim menüsü gösteriliyor:', selectionInfo);
  }
  
  // Seçim analizi yap
  analyzeSelection(selectionInfo) {
    const text = selectionInfo.text;
    const analysis = {
      length: text.length,
      wordCount: text.split(/\s+/).length,
      hasNumbers: /\d/.test(text),
      hasSpecialChars: /[!@#$%^&*(),.?":{}|<>]/.test(text),
      isAllCaps: text === text.toUpperCase(),
      isAllLower: text === text.toLowerCase()
    };
    console.log('Seçim analizi:', analysis);
  }
  
  // Metin formatla
  formatSelection(format) {
    const selection = window.getSelection();
    const selectedText = selection.toString();
    
    if (selectedText.length === 0) return;
    
    switch (format) {
      case 'bold':
        this.wrapSelection('<strong>', '</strong>');
        break;
      case 'italic':
        this.wrapSelection('<em>', '</em>');
        break;
      case 'underline':
        this.wrapSelection('<u>', '</u>');
        break;
    }
  }
  
  // Seçimi sarmala
  wrapSelection(startTag, endTag) {
    const selection = window.getSelection();
    const range = selection.getRangeAt(0);
    const contents = range.extractContents();
    
    const wrapper = document.createElement('div');
    wrapper.innerHTML = startTag + contents.textContent + endTag;
    
    range.insertNode(wrapper.firstChild);
    selection.removeAllRanges();
  }
  
  // Seçimi kopyala
  copySelection() {
    const selection = window.getSelection();
    const selectedText = selection.toString();
    
    if (selectedText.length > 0) {
      navigator.clipboard.writeText(selectedText).then(() => {
        console.log('Seçim kopyalandı');
      });
    }
  }
}
```

**Alternatifler:**
- **Range API**: Metin aralığı kontrolü
- **Document.execCommand**: Komut çalıştırma
- **Clipboard API**: Pano işlemleri
- **Input Events**: Giriş olayları
- **Custom Selection**: Özel seçim mantığı

**Ne Zaman Kullanılmalı?**
- ✅ Metin editörü geliştirme gerektiğinde
- ✅ Rich text editör gerektiğinde
- ✅ Metin seçimi kontrolü gerektiğinde
- ✅ Kopyala-yapıştır işlemleri gerektiğinde
- ✅ Metin analizi gerektiğinde
- ✅ Kullanıcı etkileşimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit metin işleme yeterli olduğunda
- ❌ Seçim kontrolü gereksiz olduğunda
- ❌ Performans kritik olduğunda

**Kullanılmazsa Ne Olur?**
- **No Text Selection**: Metin seçimi yapılamaz
- **No Rich Text Editing**: Zengin metin editörü oluşturulamaz
- **Limited User Interaction**: Kullanıcı etkileşimi sınırlı olur
- **No Text Analysis**: Metin analizi yapılamaz
- **Poor User Experience**: Kullanıcı deneyimi kötüleşir

**Modern Alternatifler:**
- **ContentEditable**: İçerik düzenlenebilir öğeler
- **Rich Text Editors**: Zengin metin editör kütüphaneleri
- **Markdown Editors**: Markdown editörleri
- **WYSIWYG Editors**: Görsel editörler
- **Third-party Libraries**: Harici editör kütüphaneleri

### Server-Sent Events API

**Server-Sent Events API Nedir?**
Server-Sent Events (SSE) API, sunucudan istemciye tek yönlü gerçek zamanlı veri akışı sağlayan modern bir web API'sidir. Sunucu, istemciye sürekli olarak veri gönderebilir ve istemci bu verileri dinleyebilir. WebSocket'ten farklı olarak tek yönlü iletişim sağlar ve HTTP protokolü üzerinden çalışır. Gerçek zamanlı bildirimler, canlı güncellemeler, chat uygulamaları, dashboard'lar ve monitoring sistemleri için ideal bir çözümdür. Otomatik yeniden bağlanma, olay türleri ve hata yönetimi özellikleri sunar.

**Neden Kullanılır?**
- **Real-time Updates**: Gerçek zamanlı güncellemeler
- **Live Notifications**: Canlı bildirimler
- **Chat Applications**: Chat uygulamaları
- **Dashboard Updates**: Dashboard güncellemeleri
- **Monitoring Systems**: İzleme sistemleri
- **Live Data Streaming**: Canlı veri akışı
- **One-way Communication**: Tek yönlü iletişim

**Temel Kullanım Örnekleri:**

```javascript
// Temel Server-Sent Events
const eventSource = new EventSource('/events');

eventSource.onmessage = event => {
  console.log('Sunucudan gelen veri:', event.data);
};

eventSource.addEventListener('custom-event', event => {
  console.log('Özel olay:', event.data);
});

// Gelişmiş SSE yapılandırması
const eventSource = new EventSource('/events', {
  withCredentials: true
});

// Bağlantı durumu kontrolü
eventSource.onopen = () => {
  console.log('SSE bağlantısı açıldı');
};

eventSource.onerror = (error) => {
  console.error('SSE bağlantı hatası:', error);
  
  if (eventSource.readyState === EventSource.CLOSED) {
    console.log('SSE bağlantısı kapandı');
  }
};

// Bağlantıyı kapat
eventSource.close();

// SSE yöneticisi
class SSEManager {
  constructor(url, options = {}) {
    this.url = url;
    this.options = options;
    this.eventSource = null;
    this.listeners = new Map();
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectInterval = 3000;
    this.isConnected = false;
  }
  
  // SSE bağlantısı başlat
  connect() {
    try {
      this.eventSource = new EventSource(this.url, this.options);
      this.setupEventListeners();
      console.log('SSE bağlantısı başlatıldı:', this.url);
    } catch (error) {
      console.error('SSE bağlantı hatası:', error);
      this.handleReconnect();
    }
  }
  
  // Olay dinleyicilerini kur
  setupEventListeners() {
    // Bağlantı açıldığında
    this.eventSource.onopen = () => {
      this.isConnected = true;
      this.reconnectAttempts = 0;
      this.notifyListeners('open', { connected: true });
      console.log('SSE bağlantısı açıldı');
    };
    
    // Mesaj geldiğinde
    this.eventSource.onmessage = (event) => {
      this.handleMessage(event);
    };
    
    // Hata oluştuğunda
    this.eventSource.onerror = (error) => {
      this.isConnected = false;
      this.notifyListeners('error', { error: error });
      console.error('SSE hatası:', error);
      
      if (this.eventSource.readyState === EventSource.CLOSED) {
        this.handleReconnect();
      }
    };
  }
  
  // Mesaj işle
  handleMessage(event) {
    const data = this.parseEventData(event.data);
    const eventType = event.type || 'message';
    
    this.notifyListeners(eventType, {
      data: data,
      lastEventId: event.lastEventId,
      origin: event.origin
    });
  }
  
  // Olay verisini parse et
  parseEventData(data) {
    try {
      return JSON.parse(data);
    } catch (error) {
      return data;
    }
  }
  
  // Dinleyici ekle
  addEventListener(eventType, callback) {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, []);
    }
    this.listeners.get(eventType).push(callback);
    
    // EventSource'a dinleyici ekle
    if (this.eventSource) {
      this.eventSource.addEventListener(eventType, callback);
    }
  }
  
  // Dinleyici kaldır
  removeEventListener(eventType, callback) {
    if (this.listeners.has(eventType)) {
      const callbacks = this.listeners.get(eventType);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
    
    // EventSource'tan dinleyici kaldır
    if (this.eventSource) {
      this.eventSource.removeEventListener(eventType, callback);
    }
  }
  
  // Dinleyicileri bilgilendir
  notifyListeners(eventType, data) {
    if (this.listeners.has(eventType)) {
      this.listeners.get(eventType).forEach(callback => {
        try {
          callback(data);
        } catch (error) {
          console.error('SSE dinleyici hatası:', error);
        }
      });
    }
  }
  
  // Yeniden bağlanma işle
  handleReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      console.log(`SSE yeniden bağlanma denemesi: ${this.reconnectAttempts}`);
      
      setTimeout(() => {
        this.connect();
      }, this.reconnectInterval);
    } else {
      console.error('SSE maksimum yeniden bağlanma denemesi aşıldı');
      this.notifyListeners('maxReconnectAttempts', { attempts: this.reconnectAttempts });
    }
  }
  
  // Bağlantıyı kapat
  disconnect() {
    if (this.eventSource) {
      this.eventSource.close();
      this.eventSource = null;
      this.isConnected = false;
      console.log('SSE bağlantısı kapatıldı');
    }
  }
  
  // Bağlantı durumunu al
  getConnectionState() {
    if (!this.eventSource) return 'disconnected';
    
    switch (this.eventSource.readyState) {
      case EventSource.CONNECTING:
        return 'connecting';
      case EventSource.OPEN:
        return 'open';
      case EventSource.CLOSED:
        return 'closed';
      default:
        return 'unknown';
    }
  }
  
  // Bağlantı durumu kontrol et
  isConnectedToServer() {
    return this.isConnected && this.eventSource && this.eventSource.readyState === EventSource.OPEN;
  }
}

// Gerçek zamanlı bildirim yöneticisi
class RealTimeNotificationManager {
  constructor(notificationEndpoint) {
    this.sseManager = new SSEManager(notificationEndpoint);
    this.notifications = [];
    this.setupNotificationHandlers();
  }
  
  // Bildirim işleyicilerini kur
  setupNotificationHandlers() {
    this.sseManager.addEventListener('notification', (data) => {
      this.handleNotification(data);
    });
    
    this.sseManager.addEventListener('open', () => {
      console.log('Bildirim bağlantısı açıldı');
    });
    
    this.sseManager.addEventListener('error', (data) => {
      console.error('Bildirim bağlantı hatası:', data.error);
    });
  }
  
  // Bildirim işle
  handleNotification(data) {
    const notification = {
      id: data.data.id || Date.now(),
      title: data.data.title,
      message: data.data.message,
      type: data.data.type || 'info',
      timestamp: new Date(),
      read: false
    };
    
    this.notifications.push(notification);
    this.displayNotification(notification);
    this.updateNotificationBadge();
  }
  
  // Bildirimi göster
  displayNotification(notification) {
    // Tarayıcı bildirimi
    if ('Notification' in window && Notification.permission === 'granted') {
      new Notification(notification.title, {
        body: notification.message,
        icon: '/notification-icon.png',
        tag: notification.id
      });
    }
    
    // UI bildirimi
    this.showUINotification(notification);
  }
  
  // UI bildirimi göster
  showUINotification(notification) {
    const notificationContainer = document.getElementById('notification-container');
    if (notificationContainer) {
      const notificationElement = document.createElement('div');
      notificationElement.className = `notification notification-${notification.type}`;
      notificationElement.innerHTML = `
        <div class="notification-content">
          <h4>${notification.title}</h4>
          <p>${notification.message}</p>
          <span class="notification-time">${notification.timestamp.toLocaleTimeString()}</span>
        </div>
        <button class="notification-close" onclick="this.parentElement.remove()">×</button>
      `;
      
      notificationContainer.appendChild(notificationElement);
      
      // Otomatik kaldırma
      setTimeout(() => {
        if (notificationElement.parentElement) {
          notificationElement.remove();
        }
      }, 5000);
    }
  }
  
  // Bildirim rozetini güncelle
  updateNotificationBadge() {
    const unreadCount = this.notifications.filter(n => !n.read).length;
    const badge = document.getElementById('notification-badge');
    
    if (badge) {
      badge.textContent = unreadCount;
      badge.style.display = unreadCount > 0 ? 'block' : 'none';
    }
  }
  
  // Bildirimleri al
  getNotifications() {
    return this.notifications;
  }
  
  // Okunmamış bildirimleri al
  getUnreadNotifications() {
    return this.notifications.filter(n => !n.read);
  }
  
  // Bildirimi okundu olarak işaretle
  markAsRead(notificationId) {
    const notification = this.notifications.find(n => n.id === notificationId);
    if (notification) {
      notification.read = true;
      this.updateNotificationBadge();
    }
  }
  
  // Tüm bildirimleri okundu olarak işaretle
  markAllAsRead() {
    this.notifications.forEach(notification => {
      notification.read = true;
    });
    this.updateNotificationBadge();
  }
  
  // Bildirim izni iste
  async requestNotificationPermission() {
    if ('Notification' in window) {
      const permission = await Notification.requestPermission();
      return permission === 'granted';
    }
    return false;
  }
  
  // Bağlantıyı başlat
  start() {
    this.sseManager.connect();
  }
  
  // Bağlantıyı durdur
  stop() {
    this.sseManager.disconnect();
  }
}

// Canlı veri akış yöneticisi
class LiveDataStreamManager {
  constructor(dataEndpoint) {
    this.sseManager = new SSEManager(dataEndpoint);
    this.dataHandlers = new Map();
    this.setupDataHandlers();
  }
  
  // Veri işleyicilerini kur
  setupDataHandlers() {
    this.sseManager.addEventListener('data', (data) => {
      this.handleDataUpdate(data);
    });
    
    this.sseManager.addEventListener('error', (data) => {
      console.error('Veri akış hatası:', data.error);
    });
  }
  
  // Veri güncellemesini işle
  handleDataUpdate(data) {
    const updateData = data.data;
    
    // Veri türüne göre işle
    if (updateData.type) {
      const handler = this.dataHandlers.get(updateData.type);
      if (handler) {
        handler(updateData);
      }
    }
    
    // Genel veri güncellemesi
    this.notifyDataUpdate(updateData);
  }
  
  // Veri işleyici ekle
  addDataHandler(dataType, handler) {
    this.dataHandlers.set(dataType, handler);
  }
  
  // Veri işleyici kaldır
  removeDataHandler(dataType) {
    this.dataHandlers.delete(dataType);
  }
  
  // Veri güncellemesini bildir
  notifyDataUpdate(data) {
    // Custom event gönder
    const event = new CustomEvent('dataUpdate', {
      detail: data
    });
    document.dispatchEvent(event);
  }
  
  // Bağlantıyı başlat
  start() {
    this.sseManager.connect();
  }
  
  // Bağlantıyı durdur
  stop() {
    this.sseManager.disconnect();
  }
}
```

**Alternatifler:**
- **WebSocket**: Çift yönlü gerçek zamanlı iletişim
- **Polling**: Periyodik veri sorgulama
- **Long Polling**: Uzun süreli sorgulama
- **WebRTC**: Peer-to-peer iletişim
- **Server Push**: Sunucu itme teknolojileri

**Ne Zaman Kullanılmalı?**
- ✅ Gerçek zamanlı güncellemeler gerektiğinde
- ✅ Canlı bildirimler gerektiğinde
- ✅ Dashboard güncellemeleri gerektiğinde
- ✅ Chat uygulamaları gerektiğinde
- ✅ Monitoring sistemleri gerektiğinde
- ✅ Tek yönlü veri akışı yeterli olduğunda

**Ne Zaman Kullanılmamalı?**
- ❌ Çift yönlü iletişim gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Yüksek performans gerektiğinde
- ❌ Karmaşık veri yapıları gerektiğinde

**Kullanılmazsa Ne Olur?**
- **No Real-time Updates**: Gerçek zamanlı güncellemeler olmaz
- **Poor User Experience**: Kullanıcı deneyimi kötüleşir
- **Manual Refresh Required**: Manuel yenileme gerekir
- **No Live Notifications**: Canlı bildirimler olmaz
- **Outdated Data**: Güncel olmayan veriler

**Modern Alternatifler:**
- **WebSocket**: Çift yönlü gerçek zamanlı iletişim
- **WebRTC**: Peer-to-peer iletişim
- **GraphQL Subscriptions**: GraphQL abonelikleri
- **Server Push**: Sunucu itme teknolojileri
- **Third-party Services**: Harici gerçek zamanlı servisler

### Service Workers API

**Service Workers API Nedir?**
Service Workers API, web uygulamalarının arka planda çalışan JavaScript worker'ları oluşturmasını sağlayan modern bir web API'sidir. Tarayıcı ve ağ arasında bir proxy görevi görür ve offline çalışma, push bildirimleri, arka plan senkronizasyonu ve cache yönetimi gibi özellikler sunar. Progressive Web Apps (PWA) geliştirme, offline-first uygulamalar, performans optimizasyonu ve native app benzeri deneyimler için kritik öneme sahiptir. Tarayıcı kapatıldıktan sonra bile çalışabilir ve network isteklerini intercept edebilir.

**Neden Kullanılır?**
- **Offline Functionality**: Offline çalışma
- **Push Notifications**: Push bildirimleri
- **Background Sync**: Arka plan senkronizasyonu
- **Caching**: Cache yönetimi
- **Performance**: Performans optimizasyonu
- **PWA Development**: PWA geliştirme
- **Native-like Experience**: Native app benzeri deneyim

**Temel Kullanım Örnekleri:**

```javascript
// Service Worker kaydet
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(registration => {
      console.log('Service Worker kaydedildi:', registration);
    })
    .catch(error => {
      console.error('Service Worker kayıt hatası:', error);
    });
}

// Service Worker içinde - sw.js
self.addEventListener('install', event => {
  console.log('Service Worker yükleniyor');
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/script.js'
      ]);
    })
  );
});

self.addEventListener('activate', event => {
  console.log('Service Worker aktif');
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== 'v1') {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        if (response) {
          return response;
        }
        return fetch(event.request);
      })
  );
});

// Service Worker yöneticisi
class ServiceWorkerManager {
  constructor(swPath = '/sw.js') {
    this.swPath = swPath;
    this.registration = null;
    this.isSupported = 'serviceWorker' in navigator;
  }
  
  // Service Worker kaydet
  async register() {
    if (!this.isSupported) {
      throw new Error('Service Worker desteklenmiyor');
    }
    
    try {
      this.registration = await navigator.serviceWorker.register(this.swPath);
      console.log('Service Worker kaydedildi:', this.registration);
      
      // Güncelleme kontrolü
      this.registration.addEventListener('updatefound', () => {
        this.handleUpdate();
      });
      
      return this.registration;
    } catch (error) {
      console.error('Service Worker kayıt hatası:', error);
      throw error;
    }
  }
  
  // Güncelleme işle
  handleUpdate() {
    const newWorker = this.registration.installing;
    
    newWorker.addEventListener('statechange', () => {
      if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
        // Yeni Service Worker yüklendi
        this.showUpdateNotification();
      }
    });
  }
  
  // Güncelleme bildirimi göster
  showUpdateNotification() {
    if (confirm('Yeni sürüm mevcut. Güncellemek ister misiniz?')) {
      this.updateServiceWorker();
    }
  }
  
  // Service Worker güncelle
  updateServiceWorker() {
    if (this.registration && this.registration.waiting) {
      this.registration.waiting.postMessage({ action: 'skipWaiting' });
    }
  }
  
  // Service Worker kaldır
  async unregister() {
    if (this.registration) {
      const result = await this.registration.unregister();
      console.log('Service Worker kaldırıldı:', result);
      return result;
    }
  }
  
  // Service Worker durumunu al
  getServiceWorkerState() {
    if (!this.registration) return 'unregistered';
    
    if (this.registration.installing) return 'installing';
    if (this.registration.waiting) return 'waiting';
    if (this.registration.active) return 'active';
    
    return 'unknown';
  }
  
  // Cache temizle
  async clearCache() {
    const cacheNames = await caches.keys();
    await Promise.all(
      cacheNames.map(cacheName => caches.delete(cacheName))
    );
    console.log('Cache temizlendi');
  }
}

// Push bildirim yöneticisi
class PushNotificationManager {
  constructor() {
    this.registration = null;
    this.isSupported = 'serviceWorker' in navigator && 'PushManager' in window;
  }
  
  // Push bildirim izni iste
  async requestPermission() {
    if (!this.isSupported) {
      throw new Error('Push bildirimler desteklenmiyor');
    }
    
    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }
  
  // Push aboneliği oluştur
  async subscribeToPush() {
    if (!this.isSupported) {
      throw new Error('Push bildirimler desteklenmiyor');
    }
    
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlBase64ToUint8Array('YOUR_VAPID_PUBLIC_KEY')
    });
    
    return subscription;
  }
  
  // Push aboneliğini kaldır
  async unsubscribeFromPush() {
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.getSubscription();
    
    if (subscription) {
      await subscription.unsubscribe();
      console.log('Push aboneliği kaldırıldı');
    }
  }
  
  // VAPID key dönüştürme
  urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, '+')
      .replace(/_/g, '/');
    
    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);
    
    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
  }
  
  // Push bildirimi gönder
  async sendPushNotification(subscription, payload) {
    const response = await fetch('/api/send-push', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        subscription: subscription,
        payload: payload
      })
    });
    
    return response.json();
  }
}

// Cache yöneticisi
class CacheManager {
  constructor(cacheName = 'app-cache') {
    this.cacheName = cacheName;
    this.cache = null;
  }
  
  // Cache aç
  async openCache() {
    this.cache = await caches.open(this.cacheName);
    return this.cache;
  }
  
  // Cache'e ekle
  async addToCache(request, response) {
    if (!this.cache) {
      await this.openCache();
    }
    
    await this.cache.put(request, response);
  }
  
  // Cache'den al
  async getFromCache(request) {
    if (!this.cache) {
      await this.openCache();
    }
    
    return await this.cache.match(request);
  }
  
  // Cache'den sil
  async deleteFromCache(request) {
    if (!this.cache) {
      await this.openCache();
    }
    
    return await this.cache.delete(request);
  }
  
  // Cache'i temizle
  async clearCache() {
    if (!this.cache) {
      await this.openCache();
    }
    
    return await this.cache.clear();
  }
  
  // Cache boyutunu al
  async getCacheSize() {
    if (!this.cache) {
      await this.openCache();
    }
    
    const keys = await this.cache.keys();
    let totalSize = 0;
    
    for (const key of keys) {
      const response = await this.cache.match(key);
      if (response) {
        const blob = await response.blob();
        totalSize += blob.size;
      }
    }
    
    return totalSize;
  }
}

// Background sync yöneticisi
class BackgroundSyncManager {
  constructor() {
    this.isSupported = 'serviceWorker' in navigator && 'sync' in window.ServiceWorkerRegistration.prototype;
  }
  
  // Background sync kaydet
  async registerBackgroundSync(tag, data) {
    if (!this.isSupported) {
      throw new Error('Background sync desteklenmiyor');
    }
    
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register(tag);
    
    // Veriyi IndexedDB'ye kaydet
    await this.storeSyncData(tag, data);
  }
  
  // Sync verisini kaydet
  async storeSyncData(tag, data) {
    const db = await this.openIndexedDB();
    const transaction = db.transaction(['syncData'], 'readwrite');
    const store = transaction.objectStore('syncData');
    
    await store.put({ tag, data, timestamp: Date.now() });
  }
  
  // Sync verisini al
  async getSyncData(tag) {
    const db = await this.openIndexedDB();
    const transaction = db.transaction(['syncData'], 'readonly');
    const store = transaction.objectStore('syncData');
    
    return await store.get(tag);
  }
  
  // Sync verisini sil
  async deleteSyncData(tag) {
    const db = await this.openIndexedDB();
    const transaction = db.transaction(['syncData'], 'readwrite');
    const store = transaction.objectStore('syncData');
    
    await store.delete(tag);
  }
  
  // IndexedDB aç
  openIndexedDB() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('BackgroundSyncDB', 1);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        if (!db.objectStoreNames.contains('syncData')) {
          db.createObjectStore('syncData', { keyPath: 'tag' });
        }
      };
    });
  }
}

// Offline yöneticisi
class OfflineManager {
  constructor() {
    this.isOnline = navigator.onLine;
    this.setupOnlineOfflineListeners();
  }
  
  // Online/offline dinleyicilerini kur
  setupOnlineOfflineListeners() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.handleOnline();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
      this.handleOffline();
    });
  }
  
  // Online durumu işle
  handleOnline() {
    console.log('İnternet bağlantısı sağlandı');
    this.syncOfflineData();
  }
  
  // Offline durumu işle
  handleOffline() {
    console.log('İnternet bağlantısı kesildi');
    this.showOfflineNotification();
  }
  
  // Offline verileri senkronize et
  async syncOfflineData() {
    // Offline sırasında biriken verileri senkronize et
    console.log('Offline veriler senkronize ediliyor');
  }
  
  // Offline bildirimi göster
  showOfflineNotification() {
    // Offline durumu bildirimi
    console.log('Uygulama offline modda çalışıyor');
  }
  
  // Bağlantı durumunu al
  getConnectionStatus() {
    return this.isOnline ? 'online' : 'offline';
  }
}
```

**Alternatifler:**
- **Web Workers**: Arka plan işlemleri
- **Shared Workers**: Paylaşılan worker'lar
- **Dedicated Workers**: Özel worker'lar
- **Web App Manifest**: PWA manifest
- **Cache API**: Cache yönetimi

**Ne Zaman Kullanılmalı?**
- ✅ PWA geliştirme gerektiğinde
- ✅ Offline çalışma gerektiğinde
- ✅ Push bildirimleri gerektiğinde
- ✅ Arka plan senkronizasyonu gerektiğinde
- ✅ Cache yönetimi gerektiğinde
- ✅ Performans optimizasyonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit web uygulamaları için
- ❌ Güvenlik kısıtlamaları olduğunda
- ❌ HTTPS gereksinimleri karşılanmadığında

**Kullanılmazsa Ne Olur?**
- **No Offline Support**: Offline destek olmaz
- **No Push Notifications**: Push bildirimleri olmaz
- **No Background Sync**: Arka plan senkronizasyonu olmaz
- **Poor Performance**: Performans kötüleşir
- **No PWA Features**: PWA özellikleri olmaz

**Modern Alternatifler:**
- **Web App Manifest**: PWA manifest
- **Cache API**: Cache yönetimi
- **Push API**: Push bildirimleri
- **Background Sync API**: Arka plan senkronizasyonu
- **Workbox**: Service Worker kütüphanesi

### Storage API

**Storage API Nedir?**
Storage API, web uygulamalarının tarayıcıda veri depolamasını sağlayan modern bir web API'sidir. Farklı depolama türleri sunar: localStorage (kalıcı depolama), sessionStorage (oturum depolama), IndexedDB (gelişmiş veritabanı), Cache API (HTTP cache) ve WebSQL (deprecated). Veri kalıcılığı, offline çalışma, performans optimizasyonu, kullanıcı tercihleri ve uygulama durumu yönetimi için kritik öneme sahiptir. Her depolama türünün kendine özgü kullanım alanları, sınırları ve performans karakteristikleri vardır.

**Neden Kullanılır?**
- **Data Persistence**: Veri kalıcılığı
- **Offline Storage**: Offline depolama
- **Performance**: Performans optimizasyonu
- **User Preferences**: Kullanıcı tercihleri
- **Application State**: Uygulama durumu
- **Caching**: Cache yönetimi
- **Data Synchronization**: Veri senkronizasyonu

**Temel Kullanım Örnekleri:**

```javascript
// LocalStorage - Kalıcı depolama
localStorage.setItem('user', JSON.stringify({ name: 'John', age: 30 }));
const user = JSON.parse(localStorage.getItem('user'));
localStorage.removeItem('user');
localStorage.clear();

// SessionStorage - Oturum depolama
sessionStorage.setItem('temp', 'geçici veri');
const temp = sessionStorage.getItem('temp');
sessionStorage.removeItem('temp');

// Storage olayları
window.addEventListener('storage', (event) => {
  console.log('Storage değişti:', {
    key: event.key,
    oldValue: event.oldValue,
    newValue: event.newValue,
    url: event.url
  });
});

// Storage yöneticisi
class StorageManager {
  constructor() {
    this.storageTypes = {
      localStorage: window.localStorage,
      sessionStorage: window.sessionStorage
    };
    this.setupStorageListeners();
  }
  
  // Storage dinleyicilerini kur
  setupStorageListeners() {
    window.addEventListener('storage', (event) => {
      this.handleStorageChange(event);
    });
  }
  
  // Storage değişikliğini işle
  handleStorageChange(event) {
    console.log('Storage değişikliği:', {
      key: event.key,
      oldValue: event.oldValue,
      newValue: event.newValue,
      storageArea: event.storageArea
    });
  }
  
  // Veri kaydet
  setItem(key, value, storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    if (storage) {
      const serializedValue = typeof value === 'object' ? JSON.stringify(value) : value;
      storage.setItem(key, serializedValue);
    }
  }
  
  // Veri al
  getItem(key, storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    if (storage) {
      const value = storage.getItem(key);
      try {
        return JSON.parse(value);
      } catch (error) {
        return value;
      }
    }
    return null;
  }
  
  // Veri sil
  removeItem(key, storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    if (storage) {
      storage.removeItem(key);
    }
  }
  
  // Tüm verileri temizle
  clear(storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    if (storage) {
      storage.clear();
    }
  }
  
  // Storage boyutunu al
  getStorageSize(storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    if (storage) {
      let totalSize = 0;
      for (let key in storage) {
        if (storage.hasOwnProperty(key)) {
          totalSize += storage[key].length + key.length;
        }
      }
      return totalSize;
    }
    return 0;
  }
  
  // Storage kullanım yüzdesini al
  getStorageUsage(storageType = 'localStorage') {
    const usedSize = this.getStorageSize(storageType);
    const maxSize = 5 * 1024 * 1024; // 5MB varsayılan limit
    return (usedSize / maxSize) * 100;
  }
  
  // Tüm anahtarları al
  getAllKeys(storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    if (storage) {
      return Object.keys(storage);
    }
    return [];
  }
  
  // Veri var mı kontrol et
  hasItem(key, storageType = 'localStorage') {
    const storage = this.storageTypes[storageType];
    return storage ? storage.getItem(key) !== null : false;
  }
}

// Gelişmiş depolama yöneticisi
class AdvancedStorageManager {
  constructor() {
    this.storageManager = new StorageManager();
    this.indexedDB = null;
    this.cacheStorage = null;
    this.setupAdvancedStorage();
  }
  
  // Gelişmiş depolama kurulumu
  async setupAdvancedStorage() {
    // IndexedDB kurulumu
    if ('indexedDB' in window) {
      this.indexedDB = await this.openIndexedDB();
    }
    
    // Cache Storage kurulumu
    if ('caches' in window) {
      this.cacheStorage = caches;
    }
  }
  
  // IndexedDB aç
  openIndexedDB() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('AdvancedStorageDB', 1);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        // Ana veri deposu
        if (!db.objectStoreNames.contains('mainData')) {
          const store = db.createObjectStore('mainData', { keyPath: 'id', autoIncrement: true });
          store.createIndex('key', 'key', { unique: true });
        }
        
        // Cache veri deposu
        if (!db.objectStoreNames.contains('cacheData')) {
          const store = db.createObjectStore('cacheData', { keyPath: 'url' });
          store.createIndex('timestamp', 'timestamp');
        }
      };
    });
  }
  
  // Veri kaydet (otomatik depolama seçimi)
  async setData(key, value, options = {}) {
    const { 
      storageType = 'auto',
      ttl = null,
      priority = 'normal'
    } = options;
    
    const data = {
      key,
      value,
      timestamp: Date.now(),
      ttl,
      priority
    };
    
    switch (storageType) {
      case 'localStorage':
        this.storageManager.setItem(key, data);
        break;
      case 'sessionStorage':
        this.storageManager.setItem(key, data, 'sessionStorage');
        break;
      case 'indexedDB':
        await this.setIndexedDBData(key, data);
        break;
      case 'cache':
        await this.setCacheData(key, value);
        break;
      case 'auto':
        await this.autoSelectStorage(key, data);
        break;
    }
  }
  
  // Otomatik depolama seçimi
  async autoSelectStorage(key, data) {
    const dataSize = JSON.stringify(data).length;
    
    if (dataSize < 1000) {
      // Küçük veriler için localStorage
      this.storageManager.setItem(key, data);
    } else if (dataSize < 10000) {
      // Orta boyutlu veriler için sessionStorage
      this.storageManager.setItem(key, data, 'sessionStorage');
    } else {
      // Büyük veriler için IndexedDB
      await this.setIndexedDBData(key, data);
    }
  }
  
  // IndexedDB'ye veri kaydet
  async setIndexedDBData(key, data) {
    if (!this.indexedDB) return;
    
    const transaction = this.indexedDB.transaction(['mainData'], 'readwrite');
    const store = transaction.objectStore('mainData');
    
    await store.put({ key, ...data });
  }
  
  // IndexedDB'den veri al
  async getIndexedDBData(key) {
    if (!this.indexedDB) return null;
    
    const transaction = this.indexedDB.transaction(['mainData'], 'readonly');
    const store = transaction.objectStore('mainData');
    const index = store.index('key');
    
    const request = index.get(key);
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Cache'e veri kaydet
  async setCacheData(key, value) {
    if (!this.cacheStorage) return;
    
    const cache = await this.cacheStorage.open('data-cache');
    const response = new Response(JSON.stringify(value));
    await cache.put(key, response);
  }
  
  // Cache'den veri al
  async getCacheData(key) {
    if (!this.cacheStorage) return null;
    
    const cache = await this.cacheStorage.open('data-cache');
    const response = await cache.match(key);
    
    if (response) {
      return await response.json();
    }
    return null;
  }
  
  // Veri al (otomatik depolama arama)
  async getData(key) {
    // Önce localStorage'da ara
    let data = this.storageManager.getItem(key);
    if (data) return data.value;
    
    // Sonra sessionStorage'da ara
    data = this.storageManager.getItem(key, 'sessionStorage');
    if (data) return data.value;
    
    // IndexedDB'de ara
    data = await this.getIndexedDBData(key);
    if (data) return data.value;
    
    // Cache'de ara
    data = await this.getCacheData(key);
    if (data) return data;
    
    return null;
  }
  
  // TTL kontrolü
  async checkTTL() {
    const allKeys = this.storageManager.getAllKeys();
    
    for (const key of allKeys) {
      const data = this.storageManager.getItem(key);
      if (data && data.ttl) {
        const now = Date.now();
        if (now - data.timestamp > data.ttl) {
          this.storageManager.removeItem(key);
        }
      }
    }
  }
  
  // Depolama temizleme
  async cleanup() {
    // TTL kontrolü
    await this.checkTTL();
    
    // Eski cache verilerini temizle
    if (this.cacheStorage) {
      const cacheNames = await this.cacheStorage.keys();
      for (const cacheName of cacheNames) {
        const cache = await this.cacheStorage.open(cacheName);
        const keys = await cache.keys();
        
        for (const request of keys) {
          const response = await cache.match(request);
          if (response) {
            const data = await response.json();
            if (data.timestamp && Date.now() - data.timestamp > 7 * 24 * 60 * 60 * 1000) {
              await cache.delete(request);
            }
          }
        }
      }
    }
  }
}

// Veri senkronizasyon yöneticisi
class DataSyncManager {
  constructor() {
    this.storageManager = new AdvancedStorageManager();
    this.syncQueue = [];
    this.isOnline = navigator.onLine;
    this.setupSyncListeners();
  }
  
  // Senkronizasyon dinleyicilerini kur
  setupSyncListeners() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.processSyncQueue();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }
  
  // Veri senkronize et
  async syncData(key, value, endpoint) {
    const syncItem = {
      key,
      value,
      endpoint,
      timestamp: Date.now(),
      retries: 0
    };
    
    if (this.isOnline) {
      await this.sendToServer(syncItem);
    } else {
      this.syncQueue.push(syncItem);
    }
  }
  
  // Sunucuya gönder
  async sendToServer(syncItem) {
    try {
      const response = await fetch(syncItem.endpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          key: syncItem.key,
          value: syncItem.value,
          timestamp: syncItem.timestamp
        })
      });
      
      if (response.ok) {
        console.log('Veri senkronize edildi:', syncItem.key);
      } else {
        throw new Error('Senkronizasyon hatası');
      }
    } catch (error) {
      console.error('Senkronizasyon hatası:', error);
      syncItem.retries++;
      
      if (syncItem.retries < 3) {
        this.syncQueue.push(syncItem);
      }
    }
  }
  
  // Senkronizasyon kuyruğunu işle
  async processSyncQueue() {
    while (this.syncQueue.length > 0 && this.isOnline) {
      const syncItem = this.syncQueue.shift();
      await this.sendToServer(syncItem);
    }
  }
  
  // Senkronizasyon durumunu al
  getSyncStatus() {
    return {
      isOnline: this.isOnline,
      queueLength: this.syncQueue.length,
      pendingItems: this.syncQueue
    };
  }
}
```

**Alternatifler:**
- **IndexedDB**: Gelişmiş veritabanı
- **WebSQL**: Web SQL veritabanı (deprecated)
- **File System Access API**: Dosya sistemi erişimi
- **Web Storage**: Web depolama
- **Third-party Storage**: Harici depolama servisleri

**Ne Zaman Kullanılmalı?**
- ✅ Veri kalıcılığı gerektiğinde
- ✅ Offline çalışma gerektiğinde
- ✅ Kullanıcı tercihleri saklama gerektiğinde
- ✅ Uygulama durumu yönetimi gerektiğinde
- ✅ Cache yönetimi gerektiğinde
- ✅ Veri senkronizasyonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Hassas veri depolama gerektiğinde
- ❌ Çok büyük veri depolama gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **No Data Persistence**: Veri kalıcılığı olmaz
- **No Offline Support**: Offline destek olmaz
- **Poor User Experience**: Kullanıcı deneyimi kötüleşir
- **No State Management**: Durum yönetimi olmaz
- **No Caching**: Cache yönetimi olmaz

**Modern Alternatifler:**
- **IndexedDB**: Gelişmiş veritabanı
- **Cache API**: HTTP cache yönetimi
- **File System Access API**: Dosya sistemi erişimi
- **Web Locks API**: Web kilitleme
- **Storage Access API**: Depolama erişimi

### Streams API

**Streams API Nedir?**
Streams API, web uygulamalarının büyük veri setlerini parça parça işlemesini sağlayan modern bir web API'sidir. Veri akışlarını oluşturur, manipüle eder ve tüketir. ReadableStream (okunabilir akış), WritableStream (yazılabilir akış), TransformStream (dönüştürülebilir akış) ve ByteStreams (byte akışları) gibi farklı akış türleri sunar. Büyük dosya işleme, veri dönüştürme, network streaming, real-time data processing ve memory-efficient operations için kritik öneme sahiptir. Veriyi bellekte tutmadan işleyebilir ve backpressure (geri basınç) kontrolü sağlar.

**Neden Kullanılır?**
- **Large Data Processing**: Büyük veri işleme
- **Memory Efficiency**: Bellek verimliliği
- **Real-time Streaming**: Gerçek zamanlı akış
- **Data Transformation**: Veri dönüştürme
- **Network Streaming**: Ağ akışı
- **File Processing**: Dosya işleme
- **Backpressure Control**: Geri basınç kontrolü

**Temel Kullanım Örnekleri:**

```javascript
// ReadableStream - Okunabilir akış
const stream = new ReadableStream({
  start(controller) {
    controller.enqueue('Veri 1');
    controller.enqueue('Veri 2');
    controller.close();
  }
});

const reader = stream.getReader();
while (true) {
  const {done, value} = await reader.read();
  if (done) break;
  console.log('Okunan veri:', value);
}

// WritableStream - Yazılabilir akış
const writableStream = new WritableStream({
  write(chunk) {
    console.log('Yazılan veri:', chunk);
  },
  close() {
    console.log('Akış kapatıldı');
  }
});

const writer = writableStream.getWriter();
await writer.write('Veri 1');
await writer.write('Veri 2');
await writer.close();

// TransformStream - Dönüştürülebilir akış
const transformStream = new TransformStream({
  transform(chunk, controller) {
    const transformed = chunk.toUpperCase();
    controller.enqueue(transformed);
  }
});

// Akış yöneticisi
class StreamManager {
  constructor() {
    this.streams = new Map();
    this.isSupported = 'ReadableStream' in window;
  }
  
  // ReadableStream oluştur
  createReadableStream(source, options = {}) {
    if (!this.isSupported) {
      throw new Error('Streams API desteklenmiyor');
    }
    
    const stream = new ReadableStream({
      start(controller) {
        if (options.start) {
          options.start(controller);
        }
      },
      pull(controller) {
        if (options.pull) {
          options.pull(controller);
        }
      },
      cancel(reason) {
        if (options.cancel) {
          options.cancel(reason);
        }
      }
    });
    
    const streamId = this.generateStreamId();
    this.streams.set(streamId, stream);
    
    return { stream, streamId };
  }
  
  // WritableStream oluştur
  createWritableStream(options = {}) {
    if (!this.isSupported) {
      throw new Error('Streams API desteklenmiyor');
    }
    
    const stream = new WritableStream({
      write(chunk, controller) {
        if (options.write) {
          options.write(chunk, controller);
        }
      },
      close() {
        if (options.close) {
          options.close();
        }
      },
      abort(reason) {
        if (options.abort) {
          options.abort(reason);
        }
      }
    });
    
    const streamId = this.generateStreamId();
    this.streams.set(streamId, stream);
    
    return { stream, streamId };
  }
  
  // TransformStream oluştur
  createTransformStream(options = {}) {
    if (!this.isSupported) {
      throw new Error('Streams API desteklenmiyor');
    }
    
    const stream = new TransformStream({
      start(controller) {
        if (options.start) {
          options.start(controller);
        }
      },
      transform(chunk, controller) {
        if (options.transform) {
          options.transform(chunk, controller);
        }
      },
      flush(controller) {
        if (options.flush) {
          options.flush(controller);
        }
      }
    });
    
    const streamId = this.generateStreamId();
    this.streams.set(streamId, stream);
    
    return { stream, streamId };
  }
  
  // Akış ID oluştur
  generateStreamId() {
    return 'stream_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Akışı al
  getStream(streamId) {
    return this.streams.get(streamId);
  }
  
  // Akışı kaldır
  removeStream(streamId) {
    this.streams.delete(streamId);
  }
  
  // Tüm akışları temizle
  clearStreams() {
    this.streams.clear();
  }
}

// Dosya işleme akışı
class FileStreamProcessor {
  constructor() {
    this.streamManager = new StreamManager();
  }
  
  // Dosyayı akış olarak işle
  async processFileAsStream(file, processor) {
    const fileStream = file.stream();
    const reader = fileStream.getReader();
    
    const processedChunks = [];
    
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      
      const processed = await processor(value);
      processedChunks.push(processed);
    }
    
    return processedChunks;
  }
  
  // Büyük dosyayı parça parça işle
  async processLargeFile(file, chunkSize = 1024 * 1024) {
    const { stream } = this.streamManager.createReadableStream({
      start(controller) {
        let offset = 0;
        
        const readChunk = async () => {
          const chunk = file.slice(offset, offset + chunkSize);
          const arrayBuffer = await chunk.arrayBuffer();
          
          if (arrayBuffer.byteLength > 0) {
            controller.enqueue(new Uint8Array(arrayBuffer));
            offset += chunkSize;
            
            if (offset < file.size) {
              readChunk();
            } else {
              controller.close();
            }
          } else {
            controller.close();
          }
        };
        
        readChunk();
      }
    });
    
    return stream;
  }
  
  // Dosya dönüştürme akışı
  createFileTransformStream(transformFunction) {
    const { stream } = this.streamManager.createTransformStream({
      transform(chunk, controller) {
        const transformed = transformFunction(chunk);
        controller.enqueue(transformed);
      }
    });
    
    return stream;
  }
}

// Ağ akış yöneticisi
class NetworkStreamManager {
  constructor() {
    this.streamManager = new StreamManager();
    this.activeStreams = new Map();
  }
  
  // HTTP isteğini akış olarak işle
  async fetchAsStream(url, options = {}) {
    const response = await fetch(url, options);
    
    if (!response.body) {
      throw new Error('Response body desteklenmiyor');
    }
    
    const { stream, streamId } = this.streamManager.createReadableStream({
      start(controller) {
        const reader = response.body.getReader();
        
        const pump = async () => {
          try {
            while (true) {
              const { done, value } = await reader.read();
              
              if (done) {
                controller.close();
                break;
              }
              
              controller.enqueue(value);
            }
          } catch (error) {
            controller.error(error);
          }
        };
        
        pump();
      }
    });
    
    this.activeStreams.set(streamId, { url, startTime: Date.now() });
    
    return { stream, streamId };
  }
  
  // Akış verilerini topla
  async collectStreamData(stream) {
    const reader = stream.getReader();
    const chunks = [];
    
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      chunks.push(value);
    }
    
    return chunks;
  }
  
  // Akış verilerini birleştir
  async combineStreamChunks(chunks) {
    const totalLength = chunks.reduce((sum, chunk) => sum + chunk.length, 0);
    const combined = new Uint8Array(totalLength);
    
    let offset = 0;
    for (const chunk of chunks) {
      combined.set(chunk, offset);
      offset += chunk.length;
    }
    
    return combined;
  }
  
  // Akış durumunu al
  getStreamStatus(streamId) {
    const streamInfo = this.activeStreams.get(streamId);
    if (streamInfo) {
      return {
        ...streamInfo,
        duration: Date.now() - streamInfo.startTime
      };
    }
    return null;
  }
  
  // Aktif akışları temizle
  cleanupStreams() {
    this.activeStreams.clear();
  }
}

// Gerçek zamanlı veri akış yöneticisi
class RealTimeStreamManager {
  constructor() {
    this.streamManager = new StreamManager();
    this.dataStreams = new Map();
    this.subscribers = new Map();
  }
  
  // Veri akışı oluştur
  createDataStream(streamId, options = {}) {
    const { stream } = this.streamManager.createReadableStream({
      start(controller) {
        if (options.start) {
          options.start(controller);
        }
      }
    });
    
    this.dataStreams.set(streamId, {
      stream,
      controller: null,
      subscribers: new Set(),
      options
    });
    
    return stream;
  }
  
  // Veri akışına abone ol
  subscribeToStream(streamId, callback) {
    const streamInfo = this.dataStreams.get(streamId);
    if (!streamInfo) {
      throw new Error('Akış bulunamadı: ' + streamId);
    }
    
    const reader = streamInfo.stream.getReader();
    streamInfo.subscribers.add(reader);
    
    const processData = async () => {
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        try {
          callback(value);
        } catch (error) {
          console.error('Veri işleme hatası:', error);
        }
      }
    };
    
    processData();
    
    return reader;
  }
  
  // Veri akışına veri gönder
  sendToStream(streamId, data) {
    const streamInfo = this.dataStreams.get(streamId);
    if (!streamInfo) {
      throw new Error('Akış bulunamadı: ' + streamId);
    }
    
    // Tüm abonelere veri gönder
    streamInfo.subscribers.forEach(reader => {
      // Bu basit bir örnek, gerçek uygulamada daha karmaşık olabilir
      console.log('Veri gönderiliyor:', data);
    });
  }
  
  // Veri akışını kapat
  closeStream(streamId) {
    const streamInfo = this.dataStreams.get(streamId);
    if (streamInfo) {
      streamInfo.subscribers.forEach(reader => {
        reader.cancel();
      });
      this.dataStreams.delete(streamId);
    }
  }
  
  // Tüm akışları kapat
  closeAllStreams() {
    this.dataStreams.forEach((streamInfo, streamId) => {
      this.closeStream(streamId);
    });
  }
}

// Veri dönüştürme akış yöneticisi
class DataTransformStreamManager {
  constructor() {
    this.streamManager = new StreamManager();
    this.transformers = new Map();
  }
  
  // Veri dönüştürücü oluştur
  createTransformer(transformerId, transformFunction) {
    const { stream } = this.streamManager.createTransformStream({
      transform(chunk, controller) {
        try {
          const transformed = transformFunction(chunk);
          controller.enqueue(transformed);
        } catch (error) {
          controller.error(error);
        }
      }
    });
    
    this.transformers.set(transformerId, {
      stream,
      transformFunction
    });
    
    return stream;
  }
  
  // JSON dönüştürücü
  createJSONTransformer() {
    return this.createTransformer('json', (chunk) => {
      if (typeof chunk === 'string') {
        return JSON.parse(chunk);
      } else {
        return JSON.stringify(chunk);
      }
    });
  }
  
  // Base64 dönüştürücü
  createBase64Transformer() {
    return this.createTransformer('base64', (chunk) => {
      if (typeof chunk === 'string') {
        return btoa(chunk);
      } else {
        return atob(chunk);
      }
    });
  }
  
  // Veri sıkıştırma dönüştürücü
  createCompressionTransformer() {
    return this.createTransformer('compression', async (chunk) => {
      const stream = new CompressionStream('gzip');
      const writer = stream.writable.getWriter();
      const reader = stream.readable.getReader();
      
      await writer.write(chunk);
      await writer.close();
      
      const compressed = [];
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        compressed.push(value);
      }
      
      return new Uint8Array(compressed.reduce((acc, chunk) => acc + chunk.length, 0));
    });
  }
  
  // Veri şifreleme dönüştürücü
  createEncryptionTransformer(key) {
    return this.createTransformer('encryption', async (chunk) => {
      const encoder = new TextEncoder();
      const data = encoder.encode(chunk);
      
      const cryptoKey = await crypto.subtle.importKey(
        'raw',
        encoder.encode(key),
        { name: 'AES-GCM' },
        false,
        ['encrypt']
      );
      
      const iv = crypto.getRandomValues(new Uint8Array(12));
      const encrypted = await crypto.subtle.encrypt(
        { name: 'AES-GCM', iv },
        cryptoKey,
        data
      );
      
      return { encrypted, iv };
    });
  }
  
  // Dönüştürücüyü kaldır
  removeTransformer(transformerId) {
    this.transformers.delete(transformerId);
  }
  
  // Tüm dönüştürücüleri temizle
  clearTransformers() {
    this.transformers.clear();
  }
}
```

**Alternatifler:**
- **File API**: Dosya işleme
- **Blob API**: Blob işleme
- **ArrayBuffer**: Array buffer işleme
- **TypedArray**: Typed array işleme
- **Web Workers**: Arka plan işleme

**Ne Zaman Kullanılmalı?**
- ✅ Büyük veri setleri işleme gerektiğinde
- ✅ Bellek verimliliği gerektiğinde
- ✅ Gerçek zamanlı veri akışı gerektiğinde
- ✅ Dosya işleme gerektiğinde
- ✅ Ağ akışı gerektiğinde
- ✅ Veri dönüştürme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Küçük veri setleri için
- ❌ Basit veri işleme yeterli olduğunda
- ❌ Performans kritik olmadığında

**Kullanılmazsa Ne Olur?**
- **Memory Issues**: Bellek sorunları oluşur
- **Poor Performance**: Performans kötüleşir
- **No Real-time Processing**: Gerçek zamanlı işleme olmaz
- **Limited Data Handling**: Veri işleme sınırlı olur
- **No Backpressure Control**: Geri basınç kontrolü olmaz

**Modern Alternatifler:**
- **Web Streams API**: Web akış API'leri
- **Fetch API**: Fetch API ile akış
- **File System Access API**: Dosya sistemi erişimi
- **Web Workers**: Arka plan işleme
- **WebAssembly**: WebAssembly ile işleme

---

## T - Bölümü

### Touch Events API

**Touch Events API Nedir?**
Touch Events API, web uygulamalarının dokunmatik cihazlardaki dokunma olaylarını algılamasını ve işlemesini sağlayan modern bir web API'sidir. Kullanıcının parmaklarıyla ekrana dokunması, kaydırması, çimdiklemesi ve diğer dokunmatik hareketleri algılar. Mobil uygulama geliştirme, touch-based interactions, gesture recognition, multi-touch support ve responsive design için kritik öneme sahiptir. touchstart, touchmove, touchend ve touchcancel olaylarını destekler ve her dokunma için detaylı bilgi sağlar.

**Neden Kullanılır?**
- **Mobile Development**: Mobil uygulama geliştirme
- **Touch Interactions**: Dokunmatik etkileşimler
- **Gesture Recognition**: Jest tanıma
- **Multi-touch Support**: Çoklu dokunma desteği
- **Responsive Design**: Responsive tasarım
- **User Experience**: Kullanıcı deneyimi
- **Touch-based Games**: Dokunmatik oyunlar

**Temel Kullanım Örnekleri:**

```javascript
// Temel dokunma olayları
element.addEventListener('touchstart', event => {
  console.log('Dokunma başladı');
  console.log(`Dokunma sayısı: ${event.touches.length}`);
});

element.addEventListener('touchmove', event => {
  event.preventDefault(); // Kaydırmayı engelle
  const touch = event.touches[0];
  console.log(`X: ${touch.clientX}, Y: ${touch.clientY}`);
});

element.addEventListener('touchend', event => {
  console.log('Dokunma bitti');
});

// Gelişmiş dokunma işleme
element.addEventListener('touchstart', event => {
  const touches = Array.from(event.touches);
  touches.forEach((touch, index) => {
    console.log(`Dokunma ${index + 1}:`, {
      id: touch.identifier,
      x: touch.clientX,
      y: touch.clientY,
      force: touch.force,
      radiusX: touch.radiusX,
      radiusY: touch.radiusY
    });
  });
});

// Dokunma yöneticisi
class TouchManager {
  constructor(element) {
    this.element = element;
    this.touches = new Map();
    this.gestures = new Map();
    this.setupTouchListeners();
  }
  
  // Dokunma dinleyicilerini kur
  setupTouchListeners() {
    this.element.addEventListener('touchstart', (event) => {
      this.handleTouchStart(event);
    });
    
    this.element.addEventListener('touchmove', (event) => {
      this.handleTouchMove(event);
    });
    
    this.element.addEventListener('touchend', (event) => {
      this.handleTouchEnd(event);
    });
    
    this.element.addEventListener('touchcancel', (event) => {
      this.handleTouchCancel(event);
    });
  }
  
  // Dokunma başlangıcını işle
  handleTouchStart(event) {
    const touches = Array.from(event.touches);
    
    touches.forEach(touch => {
      this.touches.set(touch.identifier, {
        id: touch.identifier,
        startX: touch.clientX,
        startY: touch.clientY,
        currentX: touch.clientX,
        currentY: touch.clientY,
        startTime: Date.now(),
        force: touch.force,
        radiusX: touch.radiusX,
        radiusY: touch.radiusY
      });
    });
    
    this.detectGesture('start', touches);
  }
  
  // Dokunma hareketini işle
  handleTouchMove(event) {
    event.preventDefault();
    
    const touches = Array.from(event.touches);
    
    touches.forEach(touch => {
      const touchData = this.touches.get(touch.identifier);
      if (touchData) {
        touchData.currentX = touch.clientX;
        touchData.currentY = touch.clientY;
        touchData.force = touch.force;
      }
    });
    
    this.detectGesture('move', touches);
  }
  
  // Dokunma bitişini işle
  handleTouchEnd(event) {
    const changedTouches = Array.from(event.changedTouches);
    
    changedTouches.forEach(touch => {
      const touchData = this.touches.get(touch.identifier);
      if (touchData) {
        touchData.endTime = Date.now();
        touchData.duration = touchData.endTime - touchData.startTime;
        touchData.distance = this.calculateDistance(
          touchData.startX, touchData.startY,
          touchData.currentX, touchData.currentY
        );
        
        this.touches.delete(touch.identifier);
      }
    });
    
    this.detectGesture('end', changedTouches);
  }
  
  // Dokunma iptalini işle
  handleTouchCancel(event) {
    const changedTouches = Array.from(event.changedTouches);
    
    changedTouches.forEach(touch => {
      this.touches.delete(touch.identifier);
    });
    
    console.log('Dokunma iptal edildi');
  }
  
  // Mesafe hesapla
  calculateDistance(x1, y1, x2, y2) {
    const dx = x2 - x1;
    const dy = y2 - y1;
    return Math.sqrt(dx * dx + dy * dy);
  }
  
  // Jest algılama
  detectGesture(type, touches) {
    if (touches.length === 1) {
      this.detectSingleTouchGesture(type, touches[0]);
    } else if (touches.length === 2) {
      this.detectMultiTouchGesture(type, touches);
    }
  }
  
  // Tek dokunma jesti
  detectSingleTouchGesture(type, touch) {
    const touchData = this.touches.get(touch.identifier);
    if (!touchData) return;
    
    switch (type) {
      case 'start':
        this.gestures.set('tap', { startTime: Date.now() });
        break;
      case 'move':
        if (touchData.distance > 10) {
          this.gestures.set('swipe', touchData);
        }
        break;
      case 'end':
        if (touchData.duration < 300 && touchData.distance < 10) {
          this.handleTap(touchData);
        } else if (touchData.distance > 50) {
          this.handleSwipe(touchData);
        }
        break;
    }
  }
  
  // Çoklu dokunma jesti
  detectMultiTouchGesture(type, touches) {
    if (touches.length === 2) {
      const touch1 = this.touches.get(touches[0].identifier);
      const touch2 = this.touches.get(touches[1].identifier);
      
      if (touch1 && touch2) {
        const distance = this.calculateDistance(
          touch1.currentX, touch1.currentY,
          touch2.currentX, touch2.currentY
        );
        
        if (type === 'start') {
          this.gestures.set('pinch', { startDistance: distance });
        } else if (type === 'move') {
          const pinchData = this.gestures.get('pinch');
          if (pinchData) {
            const scale = distance / pinchData.startDistance;
            this.handlePinch(scale);
          }
        }
      }
    }
  }
  
  // Tap işle
  handleTap(touchData) {
    console.log('Tap algılandı:', touchData);
    this.element.dispatchEvent(new CustomEvent('tap', {
      detail: touchData
    }));
  }
  
  // Swipe işle
  handleSwipe(touchData) {
    const dx = touchData.currentX - touchData.startX;
    const dy = touchData.currentY - touchData.startY;
    
    let direction = '';
    if (Math.abs(dx) > Math.abs(dy)) {
      direction = dx > 0 ? 'right' : 'left';
    } else {
      direction = dy > 0 ? 'down' : 'up';
    }
    
    console.log('Swipe algılandı:', direction, touchData);
    this.element.dispatchEvent(new CustomEvent('swipe', {
      detail: { ...touchData, direction }
    }));
  }
  
  // Pinch işle
  handlePinch(scale) {
    console.log('Pinch algılandı:', scale);
    this.element.dispatchEvent(new CustomEvent('pinch', {
      detail: { scale }
    }));
  }
  
  // Aktif dokunmaları al
  getActiveTouches() {
    return Array.from(this.touches.values());
  }
  
  // Jest verilerini al
  getGestures() {
    return Array.from(this.gestures.entries());
  }
}

// Dokunmatik çizim yöneticisi
class TouchDrawingManager {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.touchManager = new TouchManager(canvas);
    this.isDrawing = false;
    this.currentPath = [];
    this.setupDrawingListeners();
  }
  
  // Çizim dinleyicilerini kur
  setupDrawingListeners() {
    this.canvas.addEventListener('touchstart', (event) => {
      this.startDrawing(event);
    });
    
    this.canvas.addEventListener('touchmove', (event) => {
      this.continueDrawing(event);
    });
    
    this.canvas.addEventListener('touchend', (event) => {
      this.endDrawing(event);
    });
  }
  
  // Çizimi başlat
  startDrawing(event) {
    event.preventDefault();
    this.isDrawing = true;
    
    const touch = event.touches[0];
    const rect = this.canvas.getBoundingClientRect();
    
    this.currentPath = [{
      x: touch.clientX - rect.left,
      y: touch.clientY - rect.top,
      force: touch.force || 1
    }];
    
    this.ctx.beginPath();
    this.ctx.moveTo(this.currentPath[0].x, this.currentPath[0].y);
  }
  
  // Çizimi devam ettir
  continueDrawing(event) {
    if (!this.isDrawing) return;
    
    event.preventDefault();
    
    const touch = event.touches[0];
    const rect = this.canvas.getBoundingClientRect();
    
    const point = {
      x: touch.clientX - rect.left,
      y: touch.clientY - rect.top,
      force: touch.force || 1
    };
    
    this.currentPath.push(point);
    
    // Çizgi kalınlığını kuvvete göre ayarla
    this.ctx.lineWidth = point.force * 5;
    this.ctx.lineCap = 'round';
    this.ctx.lineJoin = 'round';
    
    this.ctx.lineTo(point.x, point.y);
    this.ctx.stroke();
  }
  
  // Çizimi bitir
  endDrawing(event) {
    if (!this.isDrawing) return;
    
    this.isDrawing = false;
    this.ctx.closePath();
    
    // Çizim tamamlandı olayı
    this.canvas.dispatchEvent(new CustomEvent('drawingComplete', {
      detail: { path: this.currentPath }
    }));
  }
  
  // Çizimi temizle
  clearCanvas() {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
  }
  
  // Çizim rengini ayarla
  setColor(color) {
    this.ctx.strokeStyle = color;
  }
  
  // Çizim kalınlığını ayarla
  setLineWidth(width) {
    this.ctx.lineWidth = width;
  }
}
```

**Alternatifler:**
- **Pointer Events API**: Pointer olayları
- **Mouse Events**: Fare olayları
- **Gesture Events**: Jest olayları
- **Custom Events**: Özel olaylar
- **Third-party Libraries**: Harici dokunma kütüphaneleri

**Ne Zaman Kullanılmalı?**
- ✅ Mobil uygulama geliştirme gerektiğinde
- ✅ Dokunmatik etkileşimler gerektiğinde
- ✅ Jest tanıma gerektiğinde
- ✅ Çoklu dokunma desteği gerektiğinde
- ✅ Dokunmatik oyunlar gerektiğinde
- ✅ Responsive tasarım gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Sadece masaüstü uygulamalar için
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit etkileşimler yeterli olduğunda
- ❌ Performans kritik olmadığında

**Kullanılmazsa Ne Olur?**
- **No Touch Support**: Dokunma desteği olmaz
- **Poor Mobile Experience**: Mobil deneyim kötüleşir
- **No Gesture Recognition**: Jest tanıma olmaz
- **Limited Interactions**: Etkileşimler sınırlı olur
- **No Multi-touch**: Çoklu dokunma olmaz

**Modern Alternatifler:**
- **Pointer Events API**: Pointer olayları
- **Gesture Events**: Jest olayları
- **Touch Action CSS**: CSS dokunma eylemleri
- **Passive Event Listeners**: Pasif olay dinleyicileri
- **Third-party Libraries**: Harici dokunma kütüphaneleri

---

## U - Bölümü

### URL API

**URL API Nedir?**
URL API, web uygulamalarının URL'leri oluşturmasını, ayrıştırmasını ve manipüle etmesini sağlayan modern bir web API'sidir. URL'lerin farklı bileşenlerini (protocol, host, pathname, search, hash) ayrı ayrı işleyebilir ve URLSearchParams ile query parametrelerini yönetebilir. Web geliştirme, routing, API çağrıları, URL validation, query string manipulation ve URL-based navigation için kritik öneme sahiptir. URL'leri güvenli bir şekilde oluşturur ve ayrıştırır, XSS saldırılarına karşı koruma sağlar.

**Neden Kullanılır?**
- **URL Parsing**: URL ayrıştırma
- **URL Construction**: URL oluşturma
- **Query Parameters**: Query parametreleri yönetimi
- **URL Validation**: URL doğrulama
- **Routing**: Yönlendirme
- **API Calls**: API çağrıları
- **Security**: Güvenlik

**Temel Kullanım Örnekleri:**

```javascript
// Temel URL işlemleri
const url = new URL('https://example.com/path?param=value');
console.log(`Host: ${url.host}`);
console.log(`Path: ${url.pathname}`);
console.log(`Param: ${url.searchParams.get('param')}`);

// URL değiştir
url.searchParams.set('newParam', 'newValue');
console.log(`Yeni URL: ${url.href}`);

// URL bileşenlerini al
const urlComponents = {
  protocol: url.protocol,
  hostname: url.hostname,
  port: url.port,
  pathname: url.pathname,
  search: url.search,
  hash: url.hash,
  origin: url.origin
};

// URL yöneticisi
class URLManager {
  constructor() {
    this.baseURL = null;
    this.defaultParams = new Map();
  }
  
  // Base URL ayarla
  setBaseURL(baseURL) {
    this.baseURL = baseURL;
  }
  
  // URL oluştur
  createURL(path, params = {}) {
    let url;
    
    if (this.baseURL) {
      url = new URL(path, this.baseURL);
    } else {
      url = new URL(path);
    }
    
    // Varsayılan parametreleri ekle
    this.defaultParams.forEach((value, key) => {
      if (!url.searchParams.has(key)) {
        url.searchParams.set(key, value);
      }
    });
    
    // Yeni parametreleri ekle
    Object.entries(params).forEach(([key, value]) => {
      url.searchParams.set(key, value);
    });
    
    return url;
  }
  
  // URL ayrıştır
  parseURL(urlString) {
    try {
      const url = new URL(urlString);
      return {
        isValid: true,
        url: url,
        components: {
          protocol: url.protocol,
          hostname: url.hostname,
          port: url.port,
          pathname: url.pathname,
          search: url.search,
          hash: url.hash,
          origin: url.origin
        },
        params: Object.fromEntries(url.searchParams)
      };
    } catch (error) {
      return {
        isValid: false,
        error: error.message
      };
    }
  }
  
  // URL doğrula
  validateURL(urlString) {
    try {
      new URL(urlString);
      return true;
    } catch {
      return false;
    }
  }
  
  // Query parametrelerini yönet
  manageParams(url, params) {
    const urlObj = new URL(url);
    
    Object.entries(params).forEach(([key, value]) => {
      if (value === null || value === undefined) {
        urlObj.searchParams.delete(key);
      } else {
        urlObj.searchParams.set(key, value);
      }
    });
    
    return urlObj.href;
  }
  
  // URL'yi güvenli hale getir
  sanitizeURL(urlString) {
    try {
      const url = new URL(urlString);
      
      // Sadece güvenli protokollere izin ver
      const allowedProtocols = ['http:', 'https:', 'mailto:', 'tel:'];
      if (!allowedProtocols.includes(url.protocol)) {
        throw new Error('Güvenli olmayan protokol');
      }
      
      return url.href;
    } catch (error) {
      throw new Error('Geçersiz URL: ' + error.message);
    }
  }
  
  // Varsayılan parametre ekle
  setDefaultParam(key, value) {
    this.defaultParams.set(key, value);
  }
  
  // Varsayılan parametre kaldır
  removeDefaultParam(key) {
    this.defaultParams.delete(key);
  }
}

// API URL yöneticisi
class APIURLManager {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.endpoints = new Map();
    this.version = 'v1';
  }
  
  // Endpoint kaydet
  registerEndpoint(name, path, method = 'GET') {
    this.endpoints.set(name, { path, method });
  }
  
  // API URL oluştur
  createAPIURL(endpointName, params = {}, queryParams = {}) {
    const endpoint = this.endpoints.get(endpointName);
    if (!endpoint) {
      throw new Error(`Endpoint bulunamadı: ${endpointName}`);
    }
    
    let path = endpoint.path;
    
    // Path parametrelerini değiştir
    Object.entries(params).forEach(([key, value]) => {
      path = path.replace(`:${key}`, encodeURIComponent(value));
    });
    
    const url = new URL(`/api/${this.version}${path}`, this.baseURL);
    
    // Query parametrelerini ekle
    Object.entries(queryParams).forEach(([key, value]) => {
      url.searchParams.set(key, value);
    });
    
    return url.href;
  }
  
  // API çağrısı yap
  async makeAPICall(endpointName, params = {}, queryParams = {}, options = {}) {
    const url = this.createAPIURL(endpointName, params, queryParams);
    const endpoint = this.endpoints.get(endpointName);
    
    const requestOptions = {
      method: endpoint.method,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    };
    
    try {
      const response = await fetch(url, requestOptions);
      return await response.json();
    } catch (error) {
      throw new Error(`API çağrısı başarısız: ${error.message}`);
    }
  }
  
  // API versiyonunu ayarla
  setVersion(version) {
    this.version = version;
  }
}

// URL routing yöneticisi
class URLRouter {
  constructor() {
    this.routes = new Map();
    this.currentRoute = null;
    this.basePath = '';
  }
  
  // Route kaydet
  registerRoute(path, handler, options = {}) {
    const route = {
      path,
      handler,
      params: options.params || {},
      query: options.query || {},
      hash: options.hash || null
    };
    
    this.routes.set(path, route);
  }
  
  // URL'yi route ile eşleştir
  matchRoute(urlString) {
    const url = new URL(urlString);
    const pathname = url.pathname.replace(this.basePath, '');
    
    for (const [routePath, route] of this.routes) {
      const match = this.matchPath(pathname, routePath);
      if (match) {
        return {
          route,
          params: match.params,
          query: Object.fromEntries(url.searchParams),
          hash: url.hash
        };
      }
    }
    
    return null;
  }
  
  // Path eşleştirme
  matchPath(pathname, routePath) {
    const pathSegments = pathname.split('/').filter(Boolean);
    const routeSegments = routePath.split('/').filter(Boolean);
    
    if (pathSegments.length !== routeSegments.length) {
      return null;
    }
    
    const params = {};
    
    for (let i = 0; i < pathSegments.length; i++) {
      const pathSegment = pathSegments[i];
      const routeSegment = routeSegments[i];
      
      if (routeSegment.startsWith(':')) {
        const paramName = routeSegment.slice(1);
        params[paramName] = pathSegment;
      } else if (pathSegment !== routeSegment) {
        return null;
      }
    }
    
    return { params };
  }
  
  // Route'u çalıştır
  navigate(urlString) {
    const match = this.matchRoute(urlString);
    if (match) {
      this.currentRoute = match;
      match.route.handler(match.params, match.query, match.hash);
      return true;
    }
    return false;
  }
  
  // Base path ayarla
  setBasePath(path) {
    this.basePath = path;
  }
  
  // Mevcut route'u al
  getCurrentRoute() {
    return this.currentRoute;
  }
}

// URL utility fonksiyonları
class URLUtils {
  // URL'yi normalize et
  static normalizeURL(urlString) {
    try {
      const url = new URL(urlString);
      return url.href;
    } catch {
      return null;
    }
  }
  
  // URL'yi kısalt
  static shortenURL(urlString, maxLength = 50) {
    if (urlString.length <= maxLength) {
      return urlString;
    }
    
    const url = new URL(urlString);
    const shortPath = url.pathname.length > 20 ? 
      url.pathname.substring(0, 20) + '...' : 
      url.pathname;
    
    return `${url.origin}${shortPath}`;
  }
  
  // URL'den domain al
  static getDomain(urlString) {
    try {
      const url = new URL(urlString);
      return url.hostname;
    } catch {
      return null;
    }
  }
  
  // URL'den path al
  static getPath(urlString) {
    try {
      const url = new URL(urlString);
      return url.pathname;
    } catch {
      return null;
    }
  }
  
  // Query parametrelerini al
  static getQueryParams(urlString) {
    try {
      const url = new URL(urlString);
      return Object.fromEntries(url.searchParams);
    } catch {
      return {};
    }
  }
  
  // URL'yi güvenli hale getir
  static sanitizeURL(urlString) {
    try {
      const url = new URL(urlString);
      
      // Sadece güvenli protokollere izin ver
      const allowedProtocols = ['http:', 'https:'];
      if (!allowedProtocols.includes(url.protocol)) {
        return null;
      }
      
      return url.href;
    } catch {
      return null;
    }
  }
  
  // URL'yi encode et
  static encodeURL(urlString) {
    try {
      const url = new URL(urlString);
      return encodeURIComponent(url.href);
    } catch {
      return null;
    }
  }
  
  // URL'yi decode et
  static decodeURL(encodedURL) {
    try {
      return decodeURIComponent(encodedURL);
    } catch {
      return null;
    }
  }
}
```

**Alternatifler:**
- **Location API**: Browser location
- **History API**: Browser history
- **Custom URL Parsing**: Özel URL ayrıştırma
- **Third-party Libraries**: Harici URL kütüphaneleri
- **Regular Expressions**: Regex ile URL işleme

**Ne Zaman Kullanılmalı?**
- ✅ URL işleme gerektiğinde
- ✅ Query parametreleri yönetimi gerektiğinde
- ✅ URL doğrulama gerektiğinde
- ✅ API çağrıları gerektiğinde
- ✅ Routing gerektiğinde
- ✅ URL güvenliği gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit string işlemleri yeterli olduğunda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Performans kritik olmadığında
- ❌ URL işleme gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No URL Parsing**: URL ayrıştırma olmaz
- **No Query Management**: Query yönetimi olmaz
- **No URL Validation**: URL doğrulama olmaz
- **Security Issues**: Güvenlik sorunları oluşur
- **Poor API Management**: API yönetimi kötüleşir

**Modern Alternatifler:**
- **URL Pattern API**: URL desen eşleştirme
- **URLSearchParams**: Query parametreleri
- **Location API**: Browser location
- **History API**: Browser history
- **Custom URL Libraries**: Özel URL kütüphaneleri

### URL Pattern API

**URL Pattern API Nedir?**
URL Pattern API, web uygulamalarının URL'leri desenlerle eşleştirmesini sağlayan modern bir web API'sidir. URL'lerin belirli desenlere uyup uymadığını kontrol eder ve eşleşen URL'lerden parametreleri çıkarır. Routing, URL validation, pattern matching, parameter extraction ve URL-based navigation için kritik öneme sahiptir. Service Workers, PWA routing, API routing ve URL-based filtering için kullanılır. URL'lerin farklı bileşenlerini (protocol, hostname, pathname, search, hash) ayrı ayrı desenlerle eşleştirebilir.

**Neden Kullanılır?**
- **URL Pattern Matching**: URL desen eşleştirme
- **Parameter Extraction**: Parametre çıkarma
- **Routing**: Yönlendirme
- **URL Validation**: URL doğrulama
- **Service Workers**: Service Worker routing
- **PWA Navigation**: PWA navigasyonu
- **API Routing**: API yönlendirme

**Temel Kullanım Örnekleri:**

```javascript
// Temel URL desen eşleştirme
const pattern = new URLPattern({
  pathname: '/users/:id',
  search: '?sort=:sort'
});

const result = pattern.exec('https://example.com/users/123?sort=name');
if (result) {
  console.log(`Kullanıcı ID: ${result.pathname.groups.id}`);
  console.log(`Sıralama: ${result.search.groups.sort}`);
}

// Gelişmiş desen eşleştirme
const complexPattern = new URLPattern({
  protocol: 'https',
  hostname: 'api.example.com',
  pathname: '/v1/users/:userId/posts/:postId',
  search: '?category=:category&limit=:limit'
});

const complexResult = complexPattern.exec('https://api.example.com/v1/users/456/posts/789?category=tech&limit=10');
if (complexResult) {
  console.log('Eşleşme bulundu:', complexResult.pathname.groups);
  console.log('Query parametreleri:', complexResult.search.groups);
}

// URL desen yöneticisi
class URLPatternManager {
  constructor() {
    this.patterns = new Map();
    this.isSupported = 'URLPattern' in window;
  }
  
  // Desen kaydet
  registerPattern(name, patternConfig, handler) {
    if (!this.isSupported) {
      throw new Error('URLPattern API desteklenmiyor');
    }
    
    const pattern = new URLPattern(patternConfig);
    this.patterns.set(name, {
      pattern,
      config: patternConfig,
      handler
    });
  }
  
  // URL'yi desenlerle eşleştir
  matchURL(urlString) {
    const url = new URL(urlString);
    
    for (const [name, patternInfo] of this.patterns) {
      const result = patternInfo.pattern.exec(url);
      if (result) {
        return {
          patternName: name,
          match: result,
          handler: patternInfo.handler,
          params: this.extractParams(result)
        };
      }
    }
    
    return null;
  }
  
  // Parametreleri çıkar
  extractParams(matchResult) {
    const params = {};
    
    // Pathname parametreleri
    if (matchResult.pathname && matchResult.pathname.groups) {
      Object.assign(params, matchResult.pathname.groups);
    }
    
    // Search parametreleri
    if (matchResult.search && matchResult.search.groups) {
      Object.assign(params, matchResult.search.groups);
    }
    
    // Hash parametreleri
    if (matchResult.hash && matchResult.hash.groups) {
      Object.assign(params, matchResult.hash.groups);
    }
    
    return params;
  }
  
  // Desen test et
  testPattern(patternConfig, urlString) {
    try {
      const pattern = new URLPattern(patternConfig);
      const result = pattern.exec(urlString);
      return {
        matches: !!result,
        result: result
      };
    } catch (error) {
      return {
        matches: false,
        error: error.message
      };
    }
  }
  
  // Desen kaldır
  removePattern(name) {
    this.patterns.delete(name);
  }
  
  // Tüm desenleri temizle
  clearPatterns() {
    this.patterns.clear();
  }
  
  // Desen listesini al
  getPatterns() {
    return Array.from(this.patterns.keys());
  }
}

// Service Worker routing yöneticisi
class ServiceWorkerRouter {
  constructor() {
    this.patternManager = new URLPatternManager();
    this.routes = new Map();
    this.middlewares = [];
  }
  
  // Route kaydet
  registerRoute(patternConfig, handler, options = {}) {
    const routeId = this.generateRouteId();
    
    this.patternManager.registerPattern(routeId, patternConfig, handler);
    this.routes.set(routeId, {
      pattern: patternConfig,
      handler,
      options,
      priority: options.priority || 0
    });
    
    return routeId;
  }
  
  // Middleware ekle
  addMiddleware(middleware) {
    this.middlewares.push(middleware);
  }
  
  // Request'i işle
  async handleRequest(request) {
    const url = new URL(request.url);
    
    // Middleware'leri çalıştır
    for (const middleware of this.middlewares) {
      const result = await middleware(request);
      if (result) {
        return result;
      }
    }
    
    // Route'u bul
    const match = this.patternManager.matchURL(url.href);
    if (match) {
      try {
        return await match.handler(request, match.params);
      } catch (error) {
        console.error('Route handler hatası:', error);
        return new Response('Internal Server Error', { status: 500 });
      }
    }
    
    // 404 döndür
    return new Response('Not Found', { status: 404 });
  }
  
  // Route ID oluştur
  generateRouteId() {
    return 'route_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Route kaldır
  removeRoute(routeId) {
    this.patternManager.removePattern(routeId);
    this.routes.delete(routeId);
  }
  
  // Route listesini al
  getRoutes() {
    return Array.from(this.routes.entries()).map(([id, route]) => ({
      id,
      ...route
    }));
  }
}

// API routing yöneticisi
class APIRouter {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.patternManager = new URLPatternManager();
    this.endpoints = new Map();
  }
  
  // API endpoint kaydet
  registerEndpoint(name, patternConfig, handler, options = {}) {
    this.patternManager.registerPattern(name, patternConfig, handler);
    this.endpoints.set(name, {
      pattern: patternConfig,
      handler,
      options,
      method: options.method || 'GET'
    });
  }
  
  // API çağrısı yap
  async makeAPICall(endpointName, params = {}, options = {}) {
    const endpoint = this.endpoints.get(endpointName);
    if (!endpoint) {
      throw new Error(`Endpoint bulunamadı: ${endpointName}`);
    }
    
    // URL oluştur
    const url = this.buildURL(endpoint.pattern, params);
    
    const requestOptions = {
      method: endpoint.method,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    };
    
    try {
      const response = await fetch(url, requestOptions);
      return await response.json();
    } catch (error) {
      throw new Error(`API çağrısı başarısız: ${error.message}`);
    }
  }
  
  // URL oluştur
  buildURL(patternConfig, params) {
    let url = this.baseURL;
    
    // Pathname'i oluştur
    if (patternConfig.pathname) {
      let pathname = patternConfig.pathname;
      Object.entries(params).forEach(([key, value]) => {
        pathname = pathname.replace(`:${key}`, encodeURIComponent(value));
      });
      url += pathname;
    }
    
    // Query parametrelerini ekle
    const queryParams = new URLSearchParams();
    Object.entries(params).forEach(([key, value]) => {
      if (!patternConfig.pathname || !patternConfig.pathname.includes(`:${key}`)) {
        queryParams.set(key, value);
      }
    });
    
    if (queryParams.toString()) {
      url += '?' + queryParams.toString();
    }
    
    return url;
  }
  
  // Endpoint kaldır
  removeEndpoint(name) {
    this.patternManager.removePattern(name);
    this.endpoints.delete(name);
  }
  
  // Endpoint listesini al
  getEndpoints() {
    return Array.from(this.endpoints.keys());
  }
}

// URL filtreleme yöneticisi
class URLFilterManager {
  constructor() {
    this.patternManager = new URLPatternManager();
    this.filters = new Map();
  }
  
  // Filtre kaydet
  registerFilter(name, patternConfig, filterFunction) {
    this.patternManager.registerPattern(name, patternConfig, filterFunction);
    this.filters.set(name, {
      pattern: patternConfig,
      filter: filterFunction
    });
  }
  
  // URL'yi filtrele
  filterURL(urlString, data) {
    const match = this.patternManager.matchURL(urlString);
    if (match) {
      const filter = this.filters.get(match.patternName);
      if (filter) {
        return filter.filter(data, match.params);
      }
    }
    return data;
  }
  
  // Çoklu filtre uygula
  applyFilters(urlString, data) {
    let filteredData = data;
    
    for (const [name, filterInfo] of this.filters) {
      const match = this.patternManager.matchURL(urlString);
      if (match && match.patternName === name) {
        filteredData = filterInfo.filter(filteredData, match.params);
      }
    }
    
    return filteredData;
  }
  
  // Filtre kaldır
  removeFilter(name) {
    this.patternManager.removePattern(name);
    this.filters.delete(name);
  }
  
  // Filtre listesini al
  getFilters() {
    return Array.from(this.filters.keys());
  }
}

// URL desen utility fonksiyonları
class URLPatternUtils {
  // Desen doğrula
  static validatePattern(patternConfig) {
    try {
      new URLPattern(patternConfig);
      return { valid: true };
    } catch (error) {
      return { valid: false, error: error.message };
    }
  }
  
  // Desen test et
  static testPattern(patternConfig, urlString) {
    try {
      const pattern = new URLPattern(patternConfig);
      const result = pattern.exec(urlString);
      return {
        matches: !!result,
        result: result
      };
    } catch (error) {
      return {
        matches: false,
        error: error.message
      };
    }
  }
  
  // Parametreleri çıkar
  static extractParams(matchResult) {
    const params = {};
    
    if (matchResult.pathname && matchResult.pathname.groups) {
      Object.assign(params, matchResult.pathname.groups);
    }
    
    if (matchResult.search && matchResult.search.groups) {
      Object.assign(params, matchResult.search.groups);
    }
    
    if (matchResult.hash && matchResult.hash.groups) {
      Object.assign(params, matchResult.hash.groups);
    }
    
    return params;
  }
  
  // Desen oluştur
  static createPattern(pathname, options = {}) {
    return new URLPattern({
      pathname,
      ...options
    });
  }
  
  // Desen birleştir
  static combinePatterns(patterns) {
    return patterns.map(config => new URLPattern(config));
  }
}
```

**Alternatifler:**
- **Regular Expressions**: Regex ile URL eşleştirme
- **Custom URL Parsing**: Özel URL ayrıştırma
- **Third-party Libraries**: Harici routing kütüphaneleri
- **URL API**: URL API ile manuel eşleştirme
- **String Matching**: String eşleştirme

**Ne Zaman Kullanılmalı?**
- ✅ URL desen eşleştirme gerektiğinde
- ✅ Parametre çıkarma gerektiğinde
- ✅ Service Worker routing gerektiğinde
- ✅ PWA navigation gerektiğinde
- ✅ API routing gerektiğinde
- ✅ URL filtreleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit URL işleme yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ URL desen eşleştirme gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No Pattern Matching**: Desen eşleştirme olmaz
- **No Parameter Extraction**: Parametre çıkarma olmaz
- **No Advanced Routing**: Gelişmiş routing olmaz
- **No Service Worker Routing**: Service Worker routing olmaz
- **No URL Filtering**: URL filtreleme olmaz

**Modern Alternatifler:**
- **URL API**: URL API ile manuel eşleştirme
- **Regular Expressions**: Regex ile eşleştirme
- **Custom Routing Libraries**: Özel routing kütüphaneleri
- **Service Worker Routing**: Service Worker routing
- **PWA Navigation**: PWA navigasyon kütüphaneleri

---

## V - Bölümü

### Vibration API

**Vibration API Nedir?**
Vibration API, web uygulamalarının mobil cihazların titreşim motorunu kontrol etmesini sağlayan modern bir web API'sidir. Kullanıcıya dokunsal geri bildirim sağlar, bildirimler, uyarılar, oyun etkileşimleri ve kullanıcı deneyimi geliştirme için kullanılır. Tek titreşim, desenli titreşim ve titreşim durdurma işlemlerini destekler. Mobil uygulama geliştirme, PWA, oyun geliştirme ve kullanıcı etkileşimi için kritik öneme sahiptir. Sadece kullanıcı etkileşimi sonrasında çalışır ve gizlilik koruması sağlar.

**Neden Kullanılır?**
- **Tactile Feedback**: Dokunsal geri bildirim
- **Notifications**: Bildirimler
- **User Experience**: Kullanıcı deneyimi
- **Game Interactions**: Oyun etkileşimleri
- **Mobile Apps**: Mobil uygulamalar
- **PWA Features**: PWA özellikleri
- **Accessibility**: Erişilebilirlik

**Temel Kullanım Örnekleri:**

```javascript
// Temel titreşim kontrolü
if ('vibrate' in navigator) {
  // Tek titreşim
  navigator.vibrate(200);
  
  // Desenli titreşim
  navigator.vibrate([200, 100, 200]);
  
  // Titreşimi durdur
  navigator.vibrate(0);
}

// Gelişmiş titreşim desenleri
const vibrationPatterns = {
  short: [100],
  medium: [200],
  long: [500],
  double: [200, 100, 200],
  triple: [150, 100, 150, 100, 150],
  morse: [200, 100, 200, 100, 200, 300, 200, 100, 200]
};

// Titreşim yöneticisi
class VibrationManager {
  constructor() {
    this.isSupported = 'vibrate' in navigator;
    this.isEnabled = true;
    this.patterns = new Map();
    this.currentVibration = null;
    this.setupDefaultPatterns();
  }
  
  // Varsayılan desenleri kur
  setupDefaultPatterns() {
    this.patterns.set('success', [100, 50, 100]);
    this.patterns.set('error', [200, 100, 200, 100, 200]);
    this.patterns.set('warning', [300, 100, 300]);
    this.patterns.set('notification', [200, 100, 200]);
    this.patterns.set('button', [50]);
    this.patterns.set('long', [500]);
    this.patterns.set('double', [200, 100, 200]);
    this.patterns.set('triple', [150, 100, 150, 100, 150]);
  }
  
  // Titreşim gönder
  vibrate(pattern) {
    if (!this.isSupported || !this.isEnabled) {
      return false;
    }
    
    try {
      navigator.vibrate(pattern);
      this.currentVibration = pattern;
      return true;
    } catch (error) {
      console.error('Titreşim hatası:', error);
      return false;
    }
  }
  
  // Desenli titreşim gönder
  vibratePattern(patternName) {
    const pattern = this.patterns.get(patternName);
    if (pattern) {
      return this.vibrate(pattern);
    }
    return false;
  }
  
  // Özel desen kaydet
  registerPattern(name, pattern) {
    this.patterns.set(name, pattern);
  }
  
  // Titreşimi durdur
  stop() {
    if (this.isSupported) {
      navigator.vibrate(0);
      this.currentVibration = null;
    }
  }
  
  // Titreşim durumunu kontrol et
  isVibrating() {
    return this.currentVibration !== null;
  }
  
  // Titreşimi etkinleştir/devre dışı bırak
  setEnabled(enabled) {
    this.isEnabled = enabled;
    if (!enabled) {
      this.stop();
    }
  }
  
  // Desteklenip desteklenmediğini kontrol et
  isVibrationSupported() {
    return this.isSupported;
  }
  
  // Mevcut desenleri al
  getPatterns() {
    return Array.from(this.patterns.keys());
  }
  
  // Desen kaldır
  removePattern(name) {
    this.patterns.delete(name);
  }
}

// Bildirim titreşim yöneticisi
class NotificationVibrationManager {
  constructor() {
    this.vibrationManager = new VibrationManager();
    this.notificationTypes = new Map();
    this.setupNotificationTypes();
  }
  
  // Bildirim türlerini kur
  setupNotificationTypes() {
    this.notificationTypes.set('message', 'notification');
    this.notificationTypes.set('email', 'double');
    this.notificationTypes.set('call', 'long');
    this.notificationTypes.set('alarm', 'triple');
    this.notificationTypes.set('reminder', 'warning');
    this.notificationTypes.set('success', 'success');
    this.notificationTypes.set('error', 'error');
  }
  
  // Bildirim titreşimi gönder
  vibrateForNotification(type, customPattern = null) {
    if (customPattern) {
      return this.vibrationManager.vibrate(customPattern);
    }
    
    const patternName = this.notificationTypes.get(type);
    if (patternName) {
      return this.vibrationManager.vibratePattern(patternName);
    }
    
    return this.vibrationManager.vibratePattern('notification');
  }
  
  // Bildirim türü kaydet
  registerNotificationType(type, patternName) {
    this.notificationTypes.set(type, patternName);
  }
  
  // Bildirim türü kaldır
  removeNotificationType(type) {
    this.notificationTypes.delete(type);
  }
  
  // Bildirim türlerini al
  getNotificationTypes() {
    return Array.from(this.notificationTypes.keys());
  }
}

// Oyun titreşim yöneticisi
class GameVibrationManager {
  constructor() {
    this.vibrationManager = new VibrationManager();
    this.gameEvents = new Map();
    this.setupGameEvents();
  }
  
  // Oyun olaylarını kur
  setupGameEvents() {
    this.gameEvents.set('hit', [50]);
    this.gameEvents.set('miss', [200, 100, 200]);
    this.gameEvents.set('score', [100, 50, 100]);
    this.gameEvents.set('levelup', [200, 100, 200, 100, 200]);
    this.gameEvents.set('gameover', [500, 200, 500]);
    this.gameEvents.set('powerup', [150, 100, 150, 100, 150]);
    this.gameEvents.set('explosion', [300, 100, 300, 100, 300]);
    this.gameEvents.set('collect', [100]);
  }
  
  // Oyun olayı titreşimi
  vibrateForGameEvent(event, intensity = 1) {
    const pattern = this.gameEvents.get(event);
    if (pattern) {
      const adjustedPattern = this.adjustIntensity(pattern, intensity);
      return this.vibrationManager.vibrate(adjustedPattern);
    }
    return false;
  }
  
  // Titreşim yoğunluğunu ayarla
  adjustIntensity(pattern, intensity) {
    return pattern.map(duration => Math.round(duration * intensity));
  }
  
  // Oyun olayı kaydet
  registerGameEvent(event, pattern) {
    this.gameEvents.set(event, pattern);
  }
  
  // Oyun olayı kaldır
  removeGameEvent(event) {
    this.gameEvents.delete(event);
  }
  
  // Oyun olaylarını al
  getGameEvents() {
    return Array.from(this.gameEvents.keys());
  }
}

// Kullanıcı etkileşim titreşim yöneticisi
class InteractionVibrationManager {
  constructor() {
    this.vibrationManager = new VibrationManager();
    this.interactions = new Map();
    this.setupInteractions();
  }
  
  // Etkileşimleri kur
  setupInteractions() {
    this.interactions.set('tap', [50]);
    this.interactions.set('longpress', [100, 50, 100]);
    this.interactions.set('swipe', [75, 25, 75]);
    this.interactions.set('pinch', [100, 50, 100, 50, 100]);
    this.interactions.set('drag', [25]);
    this.interactions.set('drop', [100]);
    this.interactions.set('hover', [30]);
    this.interactions.set('focus', [40]);
  }
  
  // Etkileşim titreşimi
  vibrateForInteraction(interaction, customPattern = null) {
    if (customPattern) {
      return this.vibrationManager.vibrate(customPattern);
    }
    
    const pattern = this.interactions.get(interaction);
    if (pattern) {
      return this.vibrationManager.vibrate(pattern);
    }
    
    return this.vibrationManager.vibratePattern('button');
  }
  
  // Etkileşim kaydet
  registerInteraction(interaction, pattern) {
    this.interactions.set(interaction, pattern);
  }
  
  // Etkileşim kaldır
  removeInteraction(interaction) {
    this.interactions.delete(interaction);
  }
  
  // Etkileşimleri al
  getInteractions() {
    return Array.from(this.interactions.keys());
  }
}

// Titreşim utility fonksiyonları
class VibrationUtils {
  // Titreşim desteğini kontrol et
  static isSupported() {
    return 'vibrate' in navigator;
  }
  
  // Basit titreşim gönder
  static vibrate(duration) {
    if (VibrationUtils.isSupported()) {
      navigator.vibrate(duration);
    }
  }
  
  // Desenli titreşim gönder
  static vibratePattern(pattern) {
    if (VibrationUtils.isSupported()) {
      navigator.vibrate(pattern);
    }
  }
  
  // Titreşimi durdur
  static stop() {
    if (VibrationUtils.isSupported()) {
      navigator.vibrate(0);
    }
  }
  
  // Morse kodu titreşimi
  static vibrateMorse(message) {
    if (!VibrationUtils.isSupported()) return;
    
    const morseCode = {
      'A': '.-', 'B': '-...', 'C': '-.-.', 'D': '-..', 'E': '.',
      'F': '..-.', 'G': '--.', 'H': '....', 'I': '..', 'J': '.---',
      'K': '-.-', 'L': '.-..', 'M': '--', 'N': '-.', 'O': '---',
      'P': '.--.', 'Q': '--.-', 'R': '.-.', 'S': '...', 'T': '-',
      'U': '..-', 'V': '...-', 'W': '.--', 'X': '-..-', 'Y': '-.--',
      'Z': '--..', ' ': ' '
    };
    
    const pattern = [];
    const dotDuration = 100;
    const dashDuration = 300;
    const pauseDuration = 100;
    const letterPause = 300;
    const wordPause = 700;
    
    for (let i = 0; i < message.length; i++) {
      const char = message[i].toUpperCase();
      const code = morseCode[char];
      
      if (code) {
        for (let j = 0; j < code.length; j++) {
          if (code[j] === '.') {
            pattern.push(dotDuration);
          } else if (code[j] === '-') {
            pattern.push(dashDuration);
          }
          
          if (j < code.length - 1) {
            pattern.push(pauseDuration);
          }
        }
        
        if (i < message.length - 1) {
          pattern.push(letterPause);
        }
      } else if (char === ' ') {
        pattern.push(wordPause);
      }
    }
    
    navigator.vibrate(pattern);
  }
  
  // Titreşim deseni oluştur
  static createPattern(durations) {
    return durations;
  }
  
  // Titreşim deseni birleştir
  static combinePatterns(patterns) {
    return patterns.flat();
  }
}
```

**Alternatifler:**
- **Audio Feedback**: Ses geri bildirimi
- **Visual Feedback**: Görsel geri bildirim
- **Haptic Feedback**: Haptic geri bildirim
- **Custom Events**: Özel olaylar
- **Third-party Libraries**: Harici titreşim kütüphaneleri

**Ne Zaman Kullanılmalı?**
- ✅ Mobil uygulama geliştirme gerektiğinde
- ✅ Dokunsal geri bildirim gerektiğinde
- ✅ Bildirimler gerektiğinde
- ✅ Oyun etkileşimleri gerektiğinde
- ✅ Kullanıcı deneyimi geliştirme gerektiğinde
- ✅ PWA özellikleri gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Masaüstü uygulamalar için
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Titreşim desteklenmeyen cihazlarda
- ❌ Kullanıcı etkileşimi olmadan
- ❌ Gizlilik endişeleri olduğunda

**Kullanılmazsa Ne Olur?**
- **No Tactile Feedback**: Dokunsal geri bildirim olmaz
- **Poor Mobile Experience**: Mobil deneyim kötüleşir
- **No Notification Feedback**: Bildirim geri bildirimi olmaz
- **Limited Game Interactions**: Oyun etkileşimleri sınırlı olur
- **Reduced Accessibility**: Erişilebilirlik azalır

**Modern Alternatifler:**
- **Haptic Feedback API**: Haptic geri bildirim
- **Audio Context API**: Ses geri bildirimi
- **Visual Feedback**: Görsel geri bildirim
- **Custom Events**: Özel olaylar
- **Third-party Libraries**: Harici titreşim kütüphaneleri

### Visual Viewport API

**Visual Viewport API Nedir?**
Visual Viewport API, web uygulamalarının görsel görünüm alanının (visual viewport) özelliklerini ve değişikliklerini algılamasını sağlayan modern bir web API'sidir. Tarayıcının görünür alanının boyutunu, ölçeğini, kaydırma pozisyonunu ve klavye gibi UI elementlerinin görünüm alanını nasıl etkilediğini kontrol eder. Responsive design, mobile optimization, keyboard handling, zoom management ve viewport-based interactions için kritik öneme sahiptir. Layout viewport'tan farklı olarak, gerçekte görünen alanı temsil eder.

**Neden Kullanılır?**
- **Responsive Design**: Responsive tasarım
- **Mobile Optimization**: Mobil optimizasyon
- **Keyboard Handling**: Klavye yönetimi
- **Zoom Management**: Zoom yönetimi
- **Viewport Detection**: Görünüm alanı algılama
- **UI Adaptation**: UI adaptasyonu
- **Layout Adjustments**: Layout ayarlamaları

**Temel Kullanım Örnekleri:**

```javascript
// Temel görünüm alanı kontrolü
if ('visualViewport' in window) {
  const viewport = window.visualViewport;
  console.log(`Görünüm boyutu: ${viewport.width}x${viewport.height}`);
  console.log(`Ölçek: ${viewport.scale}`);
  
  viewport.addEventListener('resize', () => {
    console.log('Görünüm alanı değişti');
  });
}

// Gelişmiş görünüm alanı bilgileri
if ('visualViewport' in window) {
  const viewport = window.visualViewport;
  console.log('Görünüm alanı bilgileri:', {
    width: viewport.width,
    height: viewport.height,
    scale: viewport.scale,
    offsetLeft: viewport.offsetLeft,
    offsetTop: viewport.offsetTop,
    pageLeft: viewport.pageLeft,
    pageTop: viewport.pageTop
  });
}

// Görünüm alanı yöneticisi
class VisualViewportManager {
  constructor() {
    this.viewport = window.visualViewport;
    this.isSupported = 'visualViewport' in window;
    this.listeners = new Map();
    this.currentState = null;
    this.setupViewport();
  }
  
  // Görünüm alanını kur
  setupViewport() {
    if (!this.isSupported) {
      console.warn('Visual Viewport API desteklenmiyor');
      return;
    }
    
    this.updateState();
    this.setupEventListeners();
  }
  
  // Olay dinleyicilerini kur
  setupEventListeners() {
    this.viewport.addEventListener('resize', () => {
      this.handleResize();
    });
    
    this.viewport.addEventListener('scroll', () => {
      this.handleScroll();
    });
  }
  
  // Yeniden boyutlandırma işle
  handleResize() {
    this.updateState();
    this.notifyListeners('resize', this.currentState);
  }
  
  // Kaydırma işle
  handleScroll() {
    this.updateState();
    this.notifyListeners('scroll', this.currentState);
  }
  
  // Durumu güncelle
  updateState() {
    if (!this.isSupported) return;
    
    this.currentState = {
      width: this.viewport.width,
      height: this.viewport.height,
      scale: this.viewport.scale,
      offsetLeft: this.viewport.offsetLeft,
      offsetTop: this.viewport.offsetTop,
      pageLeft: this.viewport.pageLeft,
      pageTop: this.viewport.pageTop,
      timestamp: Date.now()
    };
  }
  
  // Dinleyici ekle
  addListener(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }
  
  // Dinleyici kaldır
  removeListener(event, callback) {
    if (this.listeners.has(event)) {
      const callbacks = this.listeners.get(event);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }
  
  // Dinleyicileri bilgilendir
  notifyListeners(event, data) {
    if (this.listeners.has(event)) {
      this.listeners.get(event).forEach(callback => {
        try {
          callback(data);
        } catch (error) {
          console.error('Viewport listener hatası:', error);
        }
      });
    }
  }
  
  // Mevcut durumu al
  getCurrentState() {
    return this.currentState;
  }
  
  // Desteklenip desteklenmediğini kontrol et
  isViewportSupported() {
    return this.isSupported;
  }
  
  // Görünüm alanı boyutunu al
  getViewportSize() {
    if (!this.isSupported) return null;
    
    return {
      width: this.viewport.width,
      height: this.viewport.height
    };
  }
  
  // Ölçeği al
  getScale() {
    return this.isSupported ? this.viewport.scale : 1;
  }
  
  // Kaydırma pozisyonunu al
  getScrollPosition() {
    if (!this.isSupported) return null;
    
    return {
      pageLeft: this.viewport.pageLeft,
      pageTop: this.viewport.pageTop
    };
  }
}

// Responsive layout yöneticisi
class ResponsiveLayoutManager {
  constructor() {
    this.viewportManager = new VisualViewportManager();
    this.breakpoints = new Map();
    this.currentBreakpoint = null;
    this.setupBreakpoints();
    this.setupViewportListeners();
  }
  
  // Breakpoint'leri kur
  setupBreakpoints() {
    this.breakpoints.set('mobile', { maxWidth: 768 });
    this.breakpoints.set('tablet', { minWidth: 769, maxWidth: 1024 });
    this.breakpoints.set('desktop', { minWidth: 1025 });
  }
  
  // Görünüm alanı dinleyicilerini kur
  setupViewportListeners() {
    this.viewportManager.addListener('resize', (state) => {
      this.handleViewportChange(state);
    });
  }
  
  // Görünüm alanı değişikliğini işle
  handleViewportChange(state) {
    const newBreakpoint = this.getCurrentBreakpoint(state.width);
    
    if (newBreakpoint !== this.currentBreakpoint) {
      const oldBreakpoint = this.currentBreakpoint;
      this.currentBreakpoint = newBreakpoint;
      
      this.onBreakpointChange(oldBreakpoint, newBreakpoint, state);
    }
  }
  
  // Mevcut breakpoint'i al
  getCurrentBreakpoint(width) {
    for (const [name, config] of this.breakpoints) {
      if (this.matchesBreakpoint(width, config)) {
        return name;
      }
    }
    return 'desktop';
  }
  
  // Breakpoint eşleşmesini kontrol et
  matchesBreakpoint(width, config) {
    if (config.minWidth && width < config.minWidth) return false;
    if (config.maxWidth && width > config.maxWidth) return false;
    return true;
  }
  
  // Breakpoint değişikliği işle
  onBreakpointChange(oldBreakpoint, newBreakpoint, state) {
    console.log(`Breakpoint değişti: ${oldBreakpoint} -> ${newBreakpoint}`);
    
    // CSS sınıflarını güncelle
    document.body.className = document.body.className
      .replace(/breakpoint-\w+/g, '')
      .trim() + ` breakpoint-${newBreakpoint}`;
    
    // Özel olay gönder
    window.dispatchEvent(new CustomEvent('breakpointChange', {
      detail: {
        oldBreakpoint,
        newBreakpoint,
        viewport: state
      }
    }));
  }
  
  // Breakpoint ekle
  addBreakpoint(name, config) {
    this.breakpoints.set(name, config);
  }
  
  // Breakpoint kaldır
  removeBreakpoint(name) {
    this.breakpoints.delete(name);
  }
  
  // Mevcut breakpoint'i al
  getCurrentBreakpoint() {
    return this.currentBreakpoint;
  }
  
  // Breakpoint'leri al
  getBreakpoints() {
    return Array.from(this.breakpoints.keys());
  }
}

// Klavye yönetimi
class KeyboardViewportManager {
  constructor() {
    this.viewportManager = new VisualViewportManager();
    this.keyboardHeight = 0;
    this.isKeyboardVisible = false;
    this.setupKeyboardDetection();
  }
  
  // Klavye algılamasını kur
  setupKeyboardDetection() {
    this.viewportManager.addListener('resize', (state) => {
      this.detectKeyboard(state);
    });
  }
  
  // Klavye algıla
  detectKeyboard(state) {
    const windowHeight = window.innerHeight;
    const viewportHeight = state.height;
    const heightDifference = windowHeight - viewportHeight;
    
    if (heightDifference > 150) {
      // Klavye görünür
      if (!this.isKeyboardVisible) {
        this.isKeyboardVisible = true;
        this.keyboardHeight = heightDifference;
        this.onKeyboardShow(heightDifference);
      }
    } else {
      // Klavye gizli
      if (this.isKeyboardVisible) {
        this.isKeyboardVisible = false;
        this.onKeyboardHide();
      }
    }
  }
  
  // Klavye gösterildiğinde
  onKeyboardShow(height) {
    console.log('Klavye gösterildi:', height);
    
    // Body'ye sınıf ekle
    document.body.classList.add('keyboard-visible');
    
    // Özel olay gönder
    window.dispatchEvent(new CustomEvent('keyboardShow', {
      detail: { height }
    }));
  }
  
  // Klavye gizlendiğinde
  onKeyboardHide() {
    console.log('Klavye gizlendi');
    
    // Body'den sınıf kaldır
    document.body.classList.remove('keyboard-visible');
    
    // Özel olay gönder
    window.dispatchEvent(new CustomEvent('keyboardHide'));
  }
  
  // Klavye yüksekliğini al
  getKeyboardHeight() {
    return this.keyboardHeight;
  }
  
  // Klavye görünür mü?
  isKeyboardOpen() {
    return this.isKeyboardVisible;
  }
}

// Zoom yönetimi
class ZoomManager {
  constructor() {
    this.viewportManager = new VisualViewportManager();
    this.currentZoom = 1;
    this.zoomThreshold = 0.1;
    this.setupZoomDetection();
  }
  
  // Zoom algılamasını kur
  setupZoomDetection() {
    this.viewportManager.addListener('resize', (state) => {
      this.detectZoom(state);
    });
  }
  
  // Zoom algıla
  detectZoom(state) {
    const newZoom = state.scale;
    const zoomDifference = Math.abs(newZoom - this.currentZoom);
    
    if (zoomDifference > this.zoomThreshold) {
      const oldZoom = this.currentZoom;
      this.currentZoom = newZoom;
      this.onZoomChange(oldZoom, newZoom);
    }
  }
  
  // Zoom değişikliği işle
  onZoomChange(oldZoom, newZoom) {
    console.log(`Zoom değişti: ${oldZoom} -> ${newZoom}`);
    
    // Body'ye zoom sınıfı ekle
    document.body.className = document.body.className
      .replace(/zoom-\d+/g, '')
      .trim() + ` zoom-${Math.round(newZoom * 100)}`;
    
    // Özel olay gönder
    window.dispatchEvent(new CustomEvent('zoomChange', {
      detail: {
        oldZoom,
        newZoom,
        zoomLevel: this.getZoomLevel(newZoom)
      }
    }));
  }
  
  // Zoom seviyesini al
  getZoomLevel(zoom) {
    if (zoom < 0.8) return 'out';
    if (zoom > 1.2) return 'in';
    return 'normal';
  }
  
  // Mevcut zoom'u al
  getCurrentZoom() {
    return this.currentZoom;
  }
  
  // Zoom seviyesini al
  getZoomLevel() {
    return this.getZoomLevel(this.currentZoom);
  }
}

// Görünüm alanı utility fonksiyonları
class ViewportUtils {
  // Görünüm alanı desteğini kontrol et
  static isSupported() {
    return 'visualViewport' in window;
  }
  
  // Görünüm alanı boyutunu al
  static getViewportSize() {
    if (!ViewportUtils.isSupported()) {
      return {
        width: window.innerWidth,
        height: window.innerHeight
      };
    }
    
    return {
      width: window.visualViewport.width,
      height: window.visualViewport.height
    };
  }
  
  // Ölçeği al
  static getScale() {
    return ViewportUtils.isSupported() ? window.visualViewport.scale : 1;
  }
  
  // Kaydırma pozisyonunu al
  static getScrollPosition() {
    if (!ViewportUtils.isSupported()) {
      return {
        pageLeft: window.pageXOffset,
        pageTop: window.pageYOffset
      };
    }
    
    return {
      pageLeft: window.visualViewport.pageLeft,
      pageTop: window.visualViewport.pageTop
    };
  }
  
  // Görünüm alanı bilgilerini al
  static getViewportInfo() {
    if (!ViewportUtils.isSupported()) {
      return {
        width: window.innerWidth,
        height: window.innerHeight,
        scale: 1,
        offsetLeft: 0,
        offsetTop: 0,
        pageLeft: window.pageXOffset,
        pageTop: window.pageYOffset
      };
    }
    
    const viewport = window.visualViewport;
    return {
      width: viewport.width,
      height: viewport.height,
      scale: viewport.scale,
      offsetLeft: viewport.offsetLeft,
      offsetTop: viewport.offsetTop,
      pageLeft: viewport.pageLeft,
      pageTop: viewport.pageTop
    };
  }
  
  // Breakpoint kontrolü
  static getBreakpoint(width = null) {
    const viewportWidth = width || ViewportUtils.getViewportSize().width;
    
    if (viewportWidth < 768) return 'mobile';
    if (viewportWidth < 1024) return 'tablet';
    return 'desktop';
  }
  
  // Klavye yüksekliğini tahmin et
  static estimateKeyboardHeight() {
    if (!ViewportUtils.isSupported()) return 0;
    
    const windowHeight = window.innerHeight;
    const viewportHeight = window.visualViewport.height;
    const difference = windowHeight - viewportHeight;
    
    return difference > 150 ? difference : 0;
  }
}
```

**Alternatifler:**
- **Window Events**: Window olayları
- **ResizeObserver**: Resize Observer
- **Media Queries**: CSS Media Queries
- **Intersection Observer**: Intersection Observer
- **Custom Viewport Detection**: Özel görünüm alanı algılama

**Ne Zaman Kullanılmalı?**
- ✅ Responsive tasarım gerektiğinde
- ✅ Mobil optimizasyon gerektiğinde
- ✅ Klavye yönetimi gerektiğinde
- ✅ Zoom yönetimi gerektiğinde
- ✅ Görünüm alanı algılama gerektiğinde
- ✅ UI adaptasyonu gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit responsive tasarım yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ Görünüm alanı algılama gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No Viewport Detection**: Görünüm alanı algılama olmaz
- **Poor Mobile Experience**: Mobil deneyim kötüleşir
- **No Keyboard Handling**: Klavye yönetimi olmaz
- **No Zoom Management**: Zoom yönetimi olmaz
- **Limited Responsive Design**: Responsive tasarım sınırlı olur

**Modern Alternatifler:**
- **CSS Container Queries**: CSS container sorguları
- **ResizeObserver API**: Resize Observer API
- **Intersection Observer API**: Intersection Observer API
- **Media Queries**: CSS Media Queries
- **Custom Viewport Libraries**: Özel görünüm alanı kütüphaneleri

---

## W - Bölümü

### Web Animations API

**Web Animations API Nedir?**
Web Animations API, web uygulamalarının CSS animasyonları ve geçişlerini JavaScript ile programatik olarak kontrol etmesini sağlayan modern bir web API'sidir. Element.animate() metodu ile animasyonlar oluşturur, Animation objesi ile animasyonları kontrol eder ve performanslı animasyonlar sağlar. CSS animasyonlarından daha esnek, JavaScript'ten daha performanslıdır. Karmaşık animasyon dizileri, zamanlama kontrolü, animasyon durumu yönetimi ve performans optimizasyonu için kritik öneme sahiptir.

**Neden Kullanılır?**
- **Performance**: Performans
- **Control**: Kontrol
- **Flexibility**: Esneklik
- **Timing**: Zamanlama
- **Sequencing**: Sıralama
- **State Management**: Durum yönetimi
- **Complex Animations**: Karmaşık animasyonlar

**Temel Kullanım Örnekleri:**

```javascript
// Temel animasyon
const element = document.getElementById('myElement');
const animation = element.animate([
  { transform: 'translateX(0px)', opacity: 1 },
  { transform: 'translateX(100px)', opacity: 0.5 }
], {
  duration: 1000,
  iterations: Infinity,
  direction: 'alternate'
});

// Animasyonu kontrol et
animation.pause();
animation.play();
animation.reverse();

// Gelişmiş animasyon özellikleri
const advancedAnimation = element.animate([
  { transform: 'scale(1)', backgroundColor: 'red' },
  { transform: 'scale(1.2)', backgroundColor: 'blue', offset: 0.3 },
  { transform: 'scale(1)', backgroundColor: 'green' }
], {
  duration: 2000,
  iterations: 3,
  easing: 'cubic-bezier(0.4, 0, 0.2, 1)',
  fill: 'forwards'
});

// Animasyon yöneticisi
class AnimationManager {
  constructor() {
    this.animations = new Map();
    this.isSupported = 'animate' in Element.prototype;
    this.defaultOptions = {
      duration: 1000,
      easing: 'ease',
      fill: 'auto'
    };
  }
  
  // Animasyon oluştur
  createAnimation(element, keyframes, options = {}) {
    if (!this.isSupported) {
      console.warn('Web Animations API desteklenmiyor');
      return null;
    }
    
    const animationOptions = { ...this.defaultOptions, ...options };
    const animation = element.animate(keyframes, animationOptions);
    
    const animationId = this.generateAnimationId();
    this.animations.set(animationId, {
      animation,
      element,
      keyframes,
      options: animationOptions,
      startTime: Date.now()
    });
    
    return { animation, animationId };
  }
  
  // Animasyon ID oluştur
  generateAnimationId() {
    return 'anim_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Animasyonu oynat
  playAnimation(animationId) {
    const animData = this.animations.get(animationId);
    if (animData) {
      animData.animation.play();
      return true;
    }
    return false;
  }
  
  // Animasyonu duraklat
  pauseAnimation(animationId) {
    const animData = this.animations.get(animationId);
    if (animData) {
      animData.animation.pause();
      return true;
    }
    return false;
  }
  
  // Animasyonu durdur
  stopAnimation(animationId) {
    const animData = this.animations.get(animationId);
    if (animData) {
      animData.animation.cancel();
      this.animations.delete(animationId);
      return true;
    }
    return false;
  }
  
  // Animasyonu tersine çevir
  reverseAnimation(animationId) {
    const animData = this.animations.get(animationId);
    if (animData) {
      animData.animation.reverse();
      return true;
    }
    return false;
  }
  
  // Animasyon durumunu al
  getAnimationState(animationId) {
    const animData = this.animations.get(animationId);
    if (animData) {
      return {
        playState: animData.animation.playState,
        currentTime: animData.animation.currentTime,
        playbackRate: animData.animation.playbackRate,
        startTime: animData.startTime
      };
    }
    return null;
  }
  
  // Tüm animasyonları durdur
  stopAllAnimations() {
    this.animations.forEach((animData, animationId) => {
      animData.animation.cancel();
    });
    this.animations.clear();
  }
  
  // Animasyon listesini al
  getAnimations() {
    return Array.from(this.animations.keys());
  }
  
  // Animasyon kaldır
  removeAnimation(animationId) {
    const animData = this.animations.get(animationId);
    if (animData) {
      animData.animation.cancel();
      this.animations.delete(animationId);
    }
  }
}

// Animasyon dizisi yöneticisi
class AnimationSequenceManager {
  constructor() {
    this.animationManager = new AnimationManager();
    this.sequences = new Map();
    this.currentSequence = null;
  }
  
  // Animasyon dizisi oluştur
  createSequence(sequenceId, animations) {
    const sequence = {
      id: sequenceId,
      animations: animations,
      currentIndex: 0,
      isPlaying: false,
      isPaused: false
    };
    
    this.sequences.set(sequenceId, sequence);
    return sequence;
  }
  
  // Diziyi oynat
  async playSequence(sequenceId) {
    const sequence = this.sequences.get(sequenceId);
    if (!sequence) return false;
    
    sequence.isPlaying = true;
    sequence.isPaused = false;
    sequence.currentIndex = 0;
    
    try {
      await this.playNextAnimation(sequence);
      return true;
    } catch (error) {
      console.error('Dizi oynatma hatası:', error);
      return false;
    }
  }
  
  // Sonraki animasyonu oynat
  async playNextAnimation(sequence) {
    if (sequence.currentIndex >= sequence.animations.length) {
      sequence.isPlaying = false;
      return;
    }
    
    const animationData = sequence.animations[sequence.currentIndex];
    const { animation, animationId } = this.animationManager.createAnimation(
      animationData.element,
      animationData.keyframes,
      animationData.options
    );
    
    // Animasyon tamamlandığında sonraki animasyona geç
    animation.addEventListener('finish', () => {
      sequence.currentIndex++;
      if (sequence.isPlaying && !sequence.isPaused) {
        this.playNextAnimation(sequence);
      }
    });
    
    animation.play();
  }
  
  // Diziyi duraklat
  pauseSequence(sequenceId) {
    const sequence = this.sequences.get(sequenceId);
    if (sequence) {
      sequence.isPaused = true;
      sequence.isPlaying = false;
    }
  }
  
  // Diziyi durdur
  stopSequence(sequenceId) {
    const sequence = this.sequences.get(sequenceId);
    if (sequence) {
      sequence.isPlaying = false;
      sequence.isPaused = false;
      sequence.currentIndex = 0;
    }
  }
  
  // Dizi durumunu al
  getSequenceState(sequenceId) {
    const sequence = this.sequences.get(sequenceId);
    if (sequence) {
      return {
        isPlaying: sequence.isPlaying,
        isPaused: sequence.isPaused,
        currentIndex: sequence.currentIndex,
        totalAnimations: sequence.animations.length
      };
    }
    return null;
  }
}

// Animasyon efektleri yöneticisi
class AnimationEffectsManager {
  constructor() {
    this.animationManager = new AnimationManager();
    this.effects = new Map();
    this.setupDefaultEffects();
  }
  
  // Varsayılan efektleri kur
  setupDefaultEffects() {
    this.effects.set('fadeIn', {
      keyframes: [
        { opacity: 0 },
        { opacity: 1 }
      ],
      options: { duration: 500, fill: 'forwards' }
    });
    
    this.effects.set('fadeOut', {
      keyframes: [
        { opacity: 1 },
        { opacity: 0 }
      ],
      options: { duration: 500, fill: 'forwards' }
    });
    
    this.effects.set('slideInLeft', {
      keyframes: [
        { transform: 'translateX(-100%)' },
        { transform: 'translateX(0)' }
      ],
      options: { duration: 500, fill: 'forwards' }
    });
    
    this.effects.set('slideInRight', {
      keyframes: [
        { transform: 'translateX(100%)' },
        { transform: 'translateX(0)' }
      ],
      options: { duration: 500, fill: 'forwards' }
    });
    
    this.effects.set('bounce', {
      keyframes: [
        { transform: 'scale(1)' },
        { transform: 'scale(1.1)', offset: 0.3 },
        { transform: 'scale(0.9)', offset: 0.6 },
        { transform: 'scale(1.05)', offset: 0.8 },
        { transform: 'scale(1)' }
      ],
      options: { duration: 600, easing: 'ease-out' }
    });
    
    this.effects.set('shake', {
      keyframes: [
        { transform: 'translateX(0)' },
        { transform: 'translateX(-10px)', offset: 0.1 },
        { transform: 'translateX(10px)', offset: 0.2 },
        { transform: 'translateX(-10px)', offset: 0.3 },
        { transform: 'translateX(10px)', offset: 0.4 },
        { transform: 'translateX(-10px)', offset: 0.5 },
        { transform: 'translateX(10px)', offset: 0.6 },
        { transform: 'translateX(-10px)', offset: 0.7 },
        { transform: 'translateX(10px)', offset: 0.8 },
        { transform: 'translateX(-10px)', offset: 0.9 },
        { transform: 'translateX(0)' }
      ],
      options: { duration: 500 }
    });
  }
  
  // Efekt uygula
  applyEffect(element, effectName, customOptions = {}) {
    const effect = this.effects.get(effectName);
    if (!effect) {
      console.warn(`Efekt bulunamadı: ${effectName}`);
      return null;
    }
    
    const options = { ...effect.options, ...customOptions };
    const { animation, animationId } = this.animationManager.createAnimation(
      element,
      effect.keyframes,
      options
    );
    
    return { animation, animationId };
  }
  
  // Özel efekt kaydet
  registerEffect(name, keyframes, options) {
    this.effects.set(name, { keyframes, options });
  }
  
  // Efekt kaldır
  removeEffect(name) {
    this.effects.delete(name);
  }
  
  // Mevcut efektleri al
  getEffects() {
    return Array.from(this.effects.keys());
  }
}

// Animasyon utility fonksiyonları
class AnimationUtils {
  // Animasyon desteğini kontrol et
  static isSupported() {
    return 'animate' in Element.prototype;
  }
  
  // Basit animasyon oluştur
  static animate(element, keyframes, options) {
    if (!AnimationUtils.isSupported()) {
      console.warn('Web Animations API desteklenmiyor');
      return null;
    }
    
    return element.animate(keyframes, options);
  }
  
  // Fade in animasyonu
  static fadeIn(element, duration = 500) {
    return AnimationUtils.animate(element, [
      { opacity: 0 },
      { opacity: 1 }
    ], { duration, fill: 'forwards' });
  }
  
  // Fade out animasyonu
  static fadeOut(element, duration = 500) {
    return AnimationUtils.animate(element, [
      { opacity: 1 },
      { opacity: 0 }
    ], { duration, fill: 'forwards' });
  }
  
  // Slide animasyonu
  static slide(element, direction, distance, duration = 500) {
    const startTransform = `translate${direction}(-${distance})`;
    const endTransform = `translate${direction}(0)`;
    
    return AnimationUtils.animate(element, [
      { transform: startTransform },
      { transform: endTransform }
    ], { duration, fill: 'forwards' });
  }
  
  // Scale animasyonu
  static scale(element, fromScale, toScale, duration = 500) {
    return AnimationUtils.animate(element, [
      { transform: `scale(${fromScale})` },
      { transform: `scale(${toScale})` }
    ], { duration, fill: 'forwards' });
  }
  
  // Rotate animasyonu
  static rotate(element, fromAngle, toAngle, duration = 500) {
    return AnimationUtils.animate(element, [
      { transform: `rotate(${fromAngle}deg)` },
      { transform: `rotate(${toAngle}deg)` }
    ], { duration, fill: 'forwards' });
  }
  
  // Bounce animasyonu
  static bounce(element, duration = 600) {
    return AnimationUtils.animate(element, [
      { transform: 'scale(1)' },
      { transform: 'scale(1.1)', offset: 0.3 },
      { transform: 'scale(0.9)', offset: 0.6 },
      { transform: 'scale(1.05)', offset: 0.8 },
      { transform: 'scale(1)' }
    ], { duration, easing: 'ease-out' });
  }
  
  // Shake animasyonu
  static shake(element, duration = 500) {
    return AnimationUtils.animate(element, [
      { transform: 'translateX(0)' },
      { transform: 'translateX(-10px)', offset: 0.1 },
      { transform: 'translateX(10px)', offset: 0.2 },
      { transform: 'translateX(-10px)', offset: 0.3 },
      { transform: 'translateX(10px)', offset: 0.4 },
      { transform: 'translateX(-10px)', offset: 0.5 },
      { transform: 'translateX(10px)', offset: 0.6 },
      { transform: 'translateX(-10px)', offset: 0.7 },
      { transform: 'translateX(10px)', offset: 0.8 },
      { transform: 'translateX(-10px)', offset: 0.9 },
      { transform: 'translateX(0)' }
    ], { duration });
  }
}
```

**Alternatifler:**
- **CSS Animations**: CSS animasyonları
- **CSS Transitions**: CSS geçişleri
- **JavaScript Animations**: JavaScript animasyonları
- **Third-party Libraries**: Harici animasyon kütüphaneleri
- **SVG Animations**: SVG animasyonları

**Ne Zaman Kullanılmalı?**
- ✅ Performanslı animasyonlar gerektiğinde
- ✅ Karmaşık animasyon kontrolü gerektiğinde
- ✅ Animasyon dizileri gerektiğinde
- ✅ Zamanlama kontrolü gerektiğinde
- ✅ Durum yönetimi gerektiğinde
- ✅ Programatik animasyon gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit CSS animasyonları yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ Animasyon kontrolü gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No Animation Control**: Animasyon kontrolü olmaz
- **Poor Performance**: Performans kötüleşir
- **No Complex Animations**: Karmaşık animasyonlar olmaz
- **No Timing Control**: Zamanlama kontrolü olmaz
- **Limited Flexibility**: Esneklik sınırlı olur

**Modern Alternatifler:**
- **CSS Animations**: CSS animasyonları
- **CSS Transitions**: CSS geçişleri
- **JavaScript Animations**: JavaScript animasyonları
- **Third-party Libraries**: Harici animasyon kütüphaneleri
- **SVG Animations**: SVG animasyonları

### Web Audio API

**Web Audio API Nedir?**
Web Audio API, web uygulamalarının ses işleme, ses sentezi, ses analizi ve ses efektleri oluşturmasını sağlayan güçlü bir web API'sidir. AudioContext ile ses grafiği oluşturur, çeşitli ses düğümleri (oscillator, gain, filter, delay, reverb) ile ses işleme yapar ve gerçek zamanlı ses manipülasyonu sağlar. Müzik uygulamaları, ses editörleri, oyun ses efektleri, ses analizi ve interaktif ses deneyimleri için kritik öneme sahiptir. Yüksek performanslı ses işleme, düşük gecikme ve profesyonel ses kalitesi sunar.

**Neden Kullanılır?**
- **Audio Processing**: Ses işleme
- **Sound Synthesis**: Ses sentezi
- **Audio Analysis**: Ses analizi
- **Real-time Audio**: Gerçek zamanlı ses
- **Music Applications**: Müzik uygulamaları
- **Game Audio**: Oyun sesi
- **Interactive Audio**: İnteraktif ses

**Temel Kullanım Örnekleri:**

```javascript
// Temel ses oluşturma
const audioContext = new AudioContext();
const oscillator = audioContext.createOscillator();
const gainNode = audioContext.createGain();

oscillator.type = 'sine';
oscillator.frequency.setValueAtTime(440, audioContext.currentTime);

gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);

oscillator.connect(gainNode);
gainNode.connect(audioContext.destination);

oscillator.start();
oscillator.stop(audioContext.currentTime + 1);

// Gelişmiş ses işleme
const audioContext = new AudioContext();
const oscillator = audioContext.createOscillator();
const gainNode = audioContext.createGain();
const filterNode = audioContext.createBiquadFilter();
const delayNode = audioContext.createDelay();

// Ses düğümlerini yapılandır
oscillator.type = 'sawtooth';
oscillator.frequency.setValueAtTime(220, audioContext.currentTime);

filterNode.type = 'lowpass';
filterNode.frequency.setValueAtTime(1000, audioContext.currentTime);

delayNode.delayTime.setValueAtTime(0.3, audioContext.currentTime);

gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);

// Ses grafiğini bağla
oscillator.connect(filterNode);
filterNode.connect(delayNode);
delayNode.connect(gainNode);
gainNode.connect(audioContext.destination);

// Ses yöneticisi
class AudioManager {
  constructor() {
    this.audioContext = null;
    this.isSupported = 'AudioContext' in window || 'webkitAudioContext' in window;
    this.nodes = new Map();
    this.sounds = new Map();
    this.isInitialized = false;
  }
  
  // Ses bağlamını başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web Audio API desteklenmiyor');
    }
    
    try {
      this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
      
      // Kullanıcı etkileşimi gerekiyor
      if (this.audioContext.state === 'suspended') {
        await this.audioContext.resume();
      }
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Ses bağlamı başlatma hatası:', error);
      return false;
    }
  }
  
  // Ses düğümü oluştur
  createNode(type, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Ses yöneticisi başlatılmamış');
    }
    
    let node;
    const nodeId = this.generateNodeId();
    
    switch (type) {
      case 'oscillator':
        node = this.audioContext.createOscillator();
        node.type = options.type || 'sine';
        node.frequency.setValueAtTime(options.frequency || 440, this.audioContext.currentTime);
        break;
        
      case 'gain':
        node = this.audioContext.createGain();
        node.gain.setValueAtTime(options.gain || 1, this.audioContext.currentTime);
        break;
        
      case 'filter':
        node = this.audioContext.createBiquadFilter();
        node.type = options.filterType || 'lowpass';
        node.frequency.setValueAtTime(options.frequency || 1000, this.audioContext.currentTime);
        node.Q.setValueAtTime(options.Q || 1, this.audioContext.currentTime);
        break;
        
      case 'delay':
        node = this.audioContext.createDelay();
        node.delayTime.setValueAtTime(options.delayTime || 0.1, this.audioContext.currentTime);
        break;
        
      case 'reverb':
        node = this.audioContext.createConvolver();
        if (options.impulseResponse) {
          node.buffer = options.impulseResponse;
        }
        break;
        
      default:
        throw new Error(`Desteklenmeyen düğüm türü: ${type}`);
    }
    
    this.nodes.set(nodeId, { node, type, options });
    return { node, nodeId };
  }
  
  // Düğüm ID oluştur
  generateNodeId() {
    return 'node_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Düğümleri bağla
  connectNodes(sourceId, destinationId) {
    const source = this.nodes.get(sourceId);
    const destination = this.nodes.get(destinationId);
    
    if (source && destination) {
      source.node.connect(destination.node);
      return true;
    }
    return false;
  }
  
  // Düğümü hedefe bağla
  connectToDestination(nodeId) {
    const nodeData = this.nodes.get(nodeId);
    if (nodeData) {
      nodeData.node.connect(this.audioContext.destination);
      return true;
    }
    return false;
  }
  
  // Düğümü kaldır
  removeNode(nodeId) {
    const nodeData = this.nodes.get(nodeId);
    if (nodeData) {
      nodeData.node.disconnect();
      this.nodes.delete(nodeId);
      return true;
    }
    return false;
  }
  
  // Ses durumunu al
  getAudioState() {
    if (!this.audioContext) return null;
    
    return {
      state: this.audioContext.state,
      currentTime: this.audioContext.currentTime,
      sampleRate: this.audioContext.sampleRate,
      baseLatency: this.audioContext.baseLatency,
      outputLatency: this.audioContext.outputLatency
    };
  }
  
  // Ses bağlamını duraklat
  suspend() {
    if (this.audioContext) {
      return this.audioContext.suspend();
    }
  }
  
  // Ses bağlamını devam ettir
  resume() {
    if (this.audioContext) {
      return this.audioContext.resume();
    }
  }
  
  // Tüm düğümleri temizle
  clearNodes() {
    this.nodes.forEach((nodeData, nodeId) => {
      nodeData.node.disconnect();
    });
    this.nodes.clear();
  }
}

// Ses sentezi yöneticisi
class SoundSynthesizer {
  constructor(audioManager) {
    this.audioManager = audioManager;
    this.oscillators = new Map();
    this.envelopes = new Map();
  }
  
  // Nota çal
  playNote(frequency, duration = 1, type = 'sine', volume = 0.3) {
    const { node: oscillator, nodeId: oscId } = this.audioManager.createNode('oscillator', {
      type,
      frequency
    });
    
    const { node: gainNode, nodeId: gainId } = this.audioManager.createNode('gain', {
      gain: 0
    });
    
    // ADSR envelope uygula
    const now = this.audioManager.audioContext.currentTime;
    const attackTime = 0.1;
    const decayTime = 0.2;
    const sustainLevel = volume * 0.7;
    const releaseTime = 0.3;
    
    // Attack
    gainNode.gain.setValueAtTime(0, now);
    gainNode.gain.linearRampToValueAtTime(volume, now + attackTime);
    
    // Decay
    gainNode.gain.linearRampToValueAtTime(sustainLevel, now + attackTime + decayTime);
    
    // Sustain
    gainNode.gain.setValueAtTime(sustainLevel, now + attackTime + decayTime);
    
    // Release
    gainNode.gain.linearRampToValueAtTime(0, now + duration);
    
    // Bağlantıları kur
    oscillator.connect(gainNode);
    gainNode.connect(this.audioManager.audioContext.destination);
    
    // Notayı çal
    oscillator.start(now);
    oscillator.stop(now + duration);
    
    // Düğümleri kaydet
    this.oscillators.set(oscId, { oscillator, gainNode, startTime: now });
    
    // Temizlik
    setTimeout(() => {
      this.audioManager.removeNode(oscId);
      this.audioManager.removeNode(gainId);
      this.oscillators.delete(oscId);
    }, (duration + releaseTime) * 1000);
    
    return { oscillator, gainNode };
  }
  
  // Akor çal
  playChord(notes, duration = 2, type = 'sine', volume = 0.2) {
    const chord = [];
    
    notes.forEach(note => {
      const sound = this.playNote(note, duration, type, volume);
      chord.push(sound);
    });
    
    return chord;
  }
  
  // Melodi çal
  playMelody(notes, durations, type = 'sine', volume = 0.3) {
    const melody = [];
    let currentTime = 0;
    
    notes.forEach((note, index) => {
      const duration = durations[index] || 0.5;
      
      setTimeout(() => {
        const sound = this.playNote(note, duration, type, volume);
        melody.push(sound);
      }, currentTime * 1000);
      
      currentTime += duration;
    });
    
    return melody;
  }
  
  // Ses efektleri
  createEffect(type, options = {}) {
    switch (type) {
      case 'reverb':
        return this.createReverbEffect(options);
      case 'delay':
        return this.createDelayEffect(options);
      case 'distortion':
        return this.createDistortionEffect(options);
      case 'chorus':
        return this.createChorusEffect(options);
      default:
        throw new Error(`Desteklenmeyen efekt türü: ${type}`);
    }
  }
  
  // Reverb efekti
  createReverbEffect(options = {}) {
    const { node: reverbNode, nodeId } = this.audioManager.createNode('reverb', options);
    return { node: reverbNode, nodeId };
  }
  
  // Delay efekti
  createDelayEffect(options = {}) {
    const { node: delayNode, nodeId } = this.audioManager.createNode('delay', options);
    return { node: delayNode, nodeId };
  }
  
  // Distortion efekti
  createDistortionEffect(options = {}) {
    const { node: gainNode, nodeId } = this.audioManager.createNode('gain', {
      gain: options.gain || 10
    });
    
    // Distortion için gain düğümünü aşırı yükle
    return { node: gainNode, nodeId };
  }
  
  // Chorus efekti
  createChorusEffect(options = {}) {
    const { node: delayNode, nodeId } = this.audioManager.createNode('delay', {
      delayTime: options.delayTime || 0.02
    });
    
    // LFO ile delay time'ı modüle et
    const { node: lfo, nodeId: lfoId } = this.audioManager.createNode('oscillator', {
      type: 'sine',
      frequency: options.lfoFrequency || 0.5
    });
    
    lfo.connect(delayNode.delayTime);
    lfo.start();
    
    return { node: delayNode, nodeId, lfo, lfoId };
  }
}

// Ses analizi yöneticisi
class AudioAnalyzer {
  constructor(audioManager) {
    this.audioManager = audioManager;
    this.analyzers = new Map();
  }
  
  // Ses analizörü oluştur
  createAnalyzer(fftSize = 2048) {
    const analyzer = this.audioManager.audioContext.createAnalyser();
    analyzer.fftSize = fftSize;
    analyzer.smoothingTimeConstant = 0.8;
    
    const analyzerId = this.audioManager.generateNodeId();
    this.analyzers.set(analyzerId, analyzer);
    
    return { analyzer, analyzerId };
  }
  
  // Frekans verilerini al
  getFrequencyData(analyzerId) {
    const analyzer = this.analyzers.get(analyzerId);
    if (!analyzer) return null;
    
    const bufferLength = analyzer.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    analyzer.getByteFrequencyData(dataArray);
    
    return dataArray;
  }
  
  // Zaman verilerini al
  getTimeData(analyzerId) {
    const analyzer = this.analyzers.get(analyzerId);
    if (!analyzer) return null;
    
    const bufferLength = analyzer.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    analyzer.getByteTimeDomainData(dataArray);
    
    return dataArray;
  }
  
  // Ses seviyesini al
  getVolumeLevel(analyzerId) {
    const timeData = this.getTimeData(analyzerId);
    if (!timeData) return 0;
    
    let sum = 0;
    for (let i = 0; i < timeData.length; i++) {
      sum += timeData[i];
    }
    
    return sum / timeData.length / 255;
  }
  
  // Dominant frekansı al
  getDominantFrequency(analyzerId) {
    const frequencyData = this.getFrequencyData(analyzerId);
    if (!frequencyData) return 0;
    
    let maxValue = 0;
    let maxIndex = 0;
    
    for (let i = 0; i < frequencyData.length; i++) {
      if (frequencyData[i] > maxValue) {
        maxValue = frequencyData[i];
        maxIndex = i;
      }
    }
    
    const nyquist = this.audioManager.audioContext.sampleRate / 2;
    return (maxIndex * nyquist) / frequencyData.length;
  }
  
  // Analizörü kaldır
  removeAnalyzer(analyzerId) {
    const analyzer = this.analyzers.get(analyzerId);
    if (analyzer) {
      analyzer.disconnect();
      this.analyzers.delete(analyzerId);
      return true;
    }
    return false;
  }
}

// Ses utility fonksiyonları
class AudioUtils {
  // Frekansı nota adına çevir
  static frequencyToNote(frequency) {
    const A4 = 440;
    const noteNames = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];
    
    const semitone = Math.round(12 * Math.log2(frequency / A4));
    const octave = Math.floor(semitone / 12) + 4;
    const noteIndex = ((semitone % 12) + 12) % 12;
    
    return noteNames[noteIndex] + octave;
  }
  
  // Nota adını frekansa çevir
  static noteToFrequency(note) {
    const A4 = 440;
    const noteNames = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];
    
    const noteName = note.slice(0, -1);
    const octave = parseInt(note.slice(-1));
    const noteIndex = noteNames.indexOf(noteName);
    
    if (noteIndex === -1) return 0;
    
    const semitone = (octave - 4) * 12 + noteIndex;
    return A4 * Math.pow(2, semitone / 12);
  }
  
  // Desibel değerini lineer değere çevir
  static dbToLinear(db) {
    return Math.pow(10, db / 20);
  }
  
  // Lineer değeri desibel değerine çevir
  static linearToDb(linear) {
    return 20 * Math.log10(linear);
  }
  
  // Ses dosyası yükle
  static async loadAudioFile(audioContext, url) {
    try {
      const response = await fetch(url);
      const arrayBuffer = await response.arrayBuffer();
      const audioBuffer = await audioContext.decodeAudioData(arrayBuffer);
      return audioBuffer;
    } catch (error) {
      console.error('Ses dosyası yükleme hatası:', error);
      return null;
    }
  }
  
  // Ses buffer'ını oynat
  static playAudioBuffer(audioContext, audioBuffer, startTime = 0, duration = null) {
    const source = audioContext.createBufferSource();
    source.buffer = audioBuffer;
    source.connect(audioContext.destination);
    
    const actualDuration = duration || audioBuffer.duration;
    source.start(startTime, 0, actualDuration);
    
    return source;
  }
}
```

**Alternatifler:**
- **HTML5 Audio**: HTML5 ses elementi
- **Web Audio API**: Web Audio API
- **Third-party Libraries**: Harici ses kütüphaneleri
- **WebRTC Audio**: WebRTC ses
- **MediaStream API**: MediaStream API

**Ne Zaman Kullanılmalı?**
- ✅ Ses işleme gerektiğinde
- ✅ Ses sentezi gerektiğinde
- ✅ Ses analizi gerektiğinde
- ✅ Gerçek zamanlı ses gerektiğinde
- ✅ Müzik uygulamaları gerektiğinde
- ✅ Oyun sesi gerektiğinde
- ✅ İnteraktif ses gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit ses oynatma yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ Ses işleme gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No Audio Processing**: Ses işleme olmaz
- **No Sound Synthesis**: Ses sentezi olmaz
- **No Audio Analysis**: Ses analizi olmaz
- **No Real-time Audio**: Gerçek zamanlı ses olmaz
- **Limited Audio Features**: Ses özellikleri sınırlı olur

**Modern Alternatifler:**
- **Web Audio API**: Web Audio API
- **HTML5 Audio**: HTML5 ses elementi
- **Third-party Libraries**: Harici ses kütüphaneleri
- **WebRTC Audio**: WebRTC ses
- **MediaStream API**: MediaStream API

### Web Bluetooth API

**Web Bluetooth API Nedir?**
Web Bluetooth API, web uygulamalarının Bluetooth Low Energy (BLE) cihazları ile iletişim kurmasını sağlayan modern bir web API'sidir. Bluetooth cihazlarını keşfetme, bağlanma, servis ve karakteristik okuma/yazma işlemleri yapar. IoT cihazları, akıllı saatler, fitness tracker'lar, tıbbi cihazlar ve diğer Bluetooth cihazları ile etkileşim için kritik öneme sahiptir. Güvenli bağlantı, veri okuma/yazma ve gerçek zamanlı iletişim sağlar.

**Neden Kullanılır?**
- **IoT Devices**: IoT cihazları
- **Wearables**: Giyilebilir cihazlar
- **Medical Devices**: Tıbbi cihazlar
- **Smart Home**: Akıllı ev
- **Fitness Tracking**: Fitness takibi
- **Real-time Data**: Gerçek zamanlı veri
- **Device Control**: Cihaz kontrolü

**Temel Kullanım Örnekleri:**

```javascript
// Temel Bluetooth bağlantısı
if ('bluetooth' in navigator) {
  const device = await navigator.bluetooth.requestDevice({
    filters: [{services: ['battery_service']}]
  });
  
  const server = await device.gatt.connect();
  const service = await server.getPrimaryService('battery_service');
  const characteristic = await service.getCharacteristic('battery_level');
  const value = await characteristic.readValue();
  
  console.log('Battery level:', value.getUint8(0));
}

// Gelişmiş Bluetooth yönetimi
const bluetoothManager = new BluetoothManager();
await bluetoothManager.connectToDevice('heart_rate_service');
const heartRate = await bluetoothManager.readCharacteristic('heart_rate_measurement');

// Bluetooth yöneticisi
class BluetoothManager {
  constructor() {
    this.isSupported = 'bluetooth' in navigator;
    this.connectedDevices = new Map();
    this.services = new Map();
    this.characteristics = new Map();
    this.isScanning = false;
  }
  
  // Cihaz desteğini kontrol et
  isBluetoothSupported() {
    return this.isSupported;
  }
  
  // Cihaz tarama
  async scanForDevices(filters = [], options = {}) {
    if (!this.isSupported) {
      throw new Error('Bluetooth desteklenmiyor');
    }
    
    try {
      this.isScanning = true;
      const device = await navigator.bluetooth.requestDevice({
        filters: filters.length > 0 ? filters : undefined,
        optionalServices: options.optionalServices || [],
        acceptAllDevices: filters.length === 0
      });
      
      this.isScanning = false;
      return device;
    } catch (error) {
      this.isScanning = false;
      if (error.name === 'NotFoundError') {
        throw new Error('Cihaz bulunamadı');
      }
      throw error;
    }
  }
  
  // Cihaza bağlan
  async connectToDevice(device, options = {}) {
    if (!device) {
      throw new Error('Cihaz belirtilmedi');
    }
    
    try {
      const server = await device.gatt.connect();
      const deviceId = this.generateDeviceId(device);
      
      this.connectedDevices.set(deviceId, {
        device,
        server,
        services: new Map(),
        characteristics: new Map(),
        isConnected: true,
        connectTime: Date.now()
      });
      
      // Bağlantı koptuğunda temizlik yap
      device.addEventListener('gattserverdisconnected', () => {
        this.handleDisconnection(deviceId);
      });
      
      return { deviceId, server };
    } catch (error) {
      throw new Error(`Bağlantı hatası: ${error.message}`);
    }
  }
  
  // Servis al
  async getService(deviceId, serviceUuid) {
    const deviceData = this.connectedDevices.get(deviceId);
    if (!deviceData) {
      throw new Error('Cihaz bulunamadı');
    }
    
    try {
      const service = await deviceData.server.getPrimaryService(serviceUuid);
      deviceData.services.set(serviceUuid, service);
      return service;
    } catch (error) {
      throw new Error(`Servis alınamadı: ${error.message}`);
    }
  }
  
  // Karakteristik al
  async getCharacteristic(deviceId, serviceUuid, characteristicUuid) {
    const service = await this.getService(deviceId, serviceUuid);
    
    try {
      const characteristic = await service.getCharacteristic(characteristicUuid);
      const deviceData = this.connectedDevices.get(deviceId);
      deviceData.characteristics.set(characteristicUuid, characteristic);
      return characteristic;
    } catch (error) {
      throw new Error(`Karakteristik alınamadı: ${error.message}`);
    }
  }
  
  // Veri oku
  async readValue(deviceId, serviceUuid, characteristicUuid) {
    const characteristic = await this.getCharacteristic(deviceId, serviceUuid, characteristicUuid);
    
    try {
      const value = await characteristic.readValue();
      return value;
    } catch (error) {
      throw new Error(`Veri okunamadı: ${error.message}`);
    }
  }
  
  // Veri yaz
  async writeValue(deviceId, serviceUuid, characteristicUuid, data) {
    const characteristic = await this.getCharacteristic(deviceId, serviceUuid, characteristicUuid);
    
    try {
      await characteristic.writeValue(data);
      return true;
    } catch (error) {
      throw new Error(`Veri yazılamadı: ${error.message}`);
    }
  }
  
  // Bildirim başlat
  async startNotifications(deviceId, serviceUuid, characteristicUuid, callback) {
    const characteristic = await this.getCharacteristic(deviceId, serviceUuid, characteristicUuid);
    
    try {
      await characteristic.startNotifications();
      characteristic.addEventListener('characteristicvaluechanged', (event) => {
        callback(event.target.value);
      });
      return true;
    } catch (error) {
      throw new Error(`Bildirim başlatılamadı: ${error.message}`);
    }
  }
  
  // Bildirim durdur
  async stopNotifications(deviceId, serviceUuid, characteristicUuid) {
    const characteristic = await this.getCharacteristic(deviceId, serviceUuid, characteristicUuid);
    
    try {
      await characteristic.stopNotifications();
      return true;
    } catch (error) {
      throw new Error(`Bildirim durdurulamadı: ${error.message}`);
    }
  }
  
  // Cihazdan bağlantıyı kes
  async disconnectDevice(deviceId) {
    const deviceData = this.connectedDevices.get(deviceId);
    if (!deviceData) {
      return false;
    }
    
    try {
      if (deviceData.server.connected) {
        deviceData.server.disconnect();
      }
      this.connectedDevices.delete(deviceId);
      return true;
    } catch (error) {
      console.error('Bağlantı kesme hatası:', error);
      return false;
    }
  }
  
  // Bağlantı kopma işlemi
  handleDisconnection(deviceId) {
    const deviceData = this.connectedDevices.get(deviceId);
    if (deviceData) {
      deviceData.isConnected = false;
      deviceData.services.clear();
      deviceData.characteristics.clear();
    }
  }
  
  // Cihaz ID oluştur
  generateDeviceId(device) {
    return device.id || `device_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  // Bağlı cihazları al
  getConnectedDevices() {
    return Array.from(this.connectedDevices.keys());
  }
  
  // Cihaz durumunu al
  getDeviceStatus(deviceId) {
    const deviceData = this.connectedDevices.get(deviceId);
    if (!deviceData) return null;
    
    return {
      isConnected: deviceData.isConnected,
      connectTime: deviceData.connectTime,
      servicesCount: deviceData.services.size,
      characteristicsCount: deviceData.characteristics.size
    };
  }
  
  // Tüm bağlantıları kes
  async disconnectAll() {
    const promises = Array.from(this.connectedDevices.keys()).map(deviceId => 
      this.disconnectDevice(deviceId)
    );
    
    await Promise.all(promises);
    this.connectedDevices.clear();
  }
}

// Bluetooth servis yöneticisi
class BluetoothServiceManager {
  constructor(bluetoothManager) {
    this.bluetoothManager = bluetoothManager;
    this.serviceHandlers = new Map();
    this.setupCommonServices();
  }
  
  // Yaygın servisleri kur
  setupCommonServices() {
    this.serviceHandlers.set('battery_service', {
      uuid: 'battery_service',
      name: 'Battery Service',
      characteristics: {
        'battery_level': {
          uuid: 'battery_level',
          name: 'Battery Level',
          properties: ['read', 'notify']
        }
      }
    });
    
    this.serviceHandlers.set('heart_rate', {
      uuid: 'heart_rate',
      name: 'Heart Rate Service',
      characteristics: {
        'heart_rate_measurement': {
          uuid: 'heart_rate_measurement',
          name: 'Heart Rate Measurement',
          properties: ['notify']
        },
        'body_sensor_location': {
          uuid: 'body_sensor_location',
          name: 'Body Sensor Location',
          properties: ['read']
        }
      }
    });
    
    this.serviceHandlers.set('device_information', {
      uuid: 'device_information',
      name: 'Device Information Service',
      characteristics: {
        'manufacturer_name_string': {
          uuid: 'manufacturer_name_string',
          name: 'Manufacturer Name',
          properties: ['read']
        },
        'model_number_string': {
          uuid: 'model_number_string',
          name: 'Model Number',
          properties: ['read']
        },
        'serial_number_string': {
          uuid: 'serial_number_string',
          name: 'Serial Number',
          properties: ['read']
        }
      }
    });
  }
  
  // Servis bilgilerini al
  getServiceInfo(serviceUuid) {
    return this.serviceHandlers.get(serviceUuid);
  }
  
  // Pil seviyesini oku
  async readBatteryLevel(deviceId) {
    try {
      const value = await this.bluetoothManager.readValue(
        deviceId,
        'battery_service',
        'battery_level'
      );
      return value.getUint8(0);
    } catch (error) {
      throw new Error(`Pil seviyesi okunamadı: ${error.message}`);
    }
  }
  
  // Kalp atış hızını oku
  async readHeartRate(deviceId) {
    try {
      const value = await this.bluetoothManager.readValue(
        deviceId,
        'heart_rate',
        'heart_rate_measurement'
      );
      
      // Kalp atış hızı verisi parse et
      const flags = value.getUint8(0);
      let heartRate;
      
      if (flags & 0x01) {
        // 16-bit değer
        heartRate = value.getUint16(1, true);
      } else {
        // 8-bit değer
        heartRate = value.getUint8(1);
      }
      
      return heartRate;
    } catch (error) {
      throw new Error(`Kalp atış hızı okunamadı: ${error.message}`);
    }
  }
  
  // Cihaz bilgilerini oku
  async readDeviceInfo(deviceId) {
    try {
      const [manufacturer, model, serial] = await Promise.all([
        this.bluetoothManager.readValue(deviceId, 'device_information', 'manufacturer_name_string'),
        this.bluetoothManager.readValue(deviceId, 'device_information', 'model_number_string'),
        this.bluetoothManager.readValue(deviceId, 'device_information', 'serial_number_string')
      ]);
      
      return {
        manufacturer: this.decodeString(manufacturer),
        model: this.decodeString(model),
        serial: this.decodeString(serial)
      };
    } catch (error) {
      throw new Error(`Cihaz bilgileri okunamadı: ${error.message}`);
    }
  }
  
  // String decode et
  decodeString(dataView) {
    const decoder = new TextDecoder('utf-8');
    return decoder.decode(dataView);
  }
  
  // Kalp atış hızı bildirimlerini başlat
  async startHeartRateNotifications(deviceId, callback) {
    try {
      await this.bluetoothManager.startNotifications(
        deviceId,
        'heart_rate',
        'heart_rate_measurement',
        (value) => {
          const flags = value.getUint8(0);
          let heartRate;
          
          if (flags & 0x01) {
            heartRate = value.getUint16(1, true);
          } else {
            heartRate = value.getUint8(1);
          }
          
          callback(heartRate);
        }
      );
      return true;
    } catch (error) {
      throw new Error(`Kalp atış hızı bildirimleri başlatılamadı: ${error.message}`);
    }
  }
  
  // Pil seviyesi bildirimlerini başlat
  async startBatteryNotifications(deviceId, callback) {
    try {
      await this.bluetoothManager.startNotifications(
        deviceId,
        'battery_service',
        'battery_level',
        (value) => {
          const batteryLevel = value.getUint8(0);
          callback(batteryLevel);
        }
      );
      return true;
    } catch (error) {
      throw new Error(`Pil seviyesi bildirimleri başlatılamadı: ${error.message}`);
    }
  }
}

// Bluetooth utility fonksiyonları
class BluetoothUtils {
  // UUID'yi normalize et
  static normalizeUuid(uuid) {
    if (typeof uuid === 'string') {
      return uuid.toLowerCase();
    }
    return uuid;
  }
  
  // Servis UUID'sini kontrol et
  static isValidServiceUuid(uuid) {
    const normalizedUuid = BluetoothUtils.normalizeUuid(uuid);
    const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
    return uuidRegex.test(normalizedUuid);
  }
  
  // Karakteristik UUID'sini kontrol et
  static isValidCharacteristicUuid(uuid) {
    return BluetoothUtils.isValidServiceUuid(uuid);
  }
  
  // Veri tipini kontrol et
  static getDataType(dataView) {
    if (dataView instanceof DataView) {
      return 'DataView';
    } else if (dataView instanceof ArrayBuffer) {
      return 'ArrayBuffer';
    } else if (dataView instanceof Uint8Array) {
      return 'Uint8Array';
    }
    return 'Unknown';
  }
  
  // Veri boyutunu al
  static getDataSize(dataView) {
    if (dataView instanceof DataView) {
      return dataView.byteLength;
    } else if (dataView instanceof ArrayBuffer) {
      return dataView.byteLength;
    } else if (dataView instanceof Uint8Array) {
      return dataView.length;
    }
    return 0;
  }
  
  // Veri hex string'e çevir
  static dataToHex(dataView) {
    const bytes = new Uint8Array(dataView);
    return Array.from(bytes)
      .map(byte => byte.toString(16).padStart(2, '0'))
      .join(' ');
  }
  
  // Hex string'i veriye çevir
  static hexToData(hexString) {
    const bytes = hexString.split(' ')
      .map(hex => parseInt(hex, 16));
    return new Uint8Array(bytes);
  }
  
  // Veri string'e çevir
  static dataToString(dataView, encoding = 'utf-8') {
    const decoder = new TextDecoder(encoding);
    return decoder.decode(dataView);
  }
  
  // String'i veriye çevir
  static stringToData(string, encoding = 'utf-8') {
    const encoder = new TextEncoder();
    return encoder.encode(string);
  }
  
  // Veri integer'a çevir
  static dataToInt(dataView, littleEndian = true) {
    if (dataView.byteLength === 1) {
      return dataView.getUint8(0);
    } else if (dataView.byteLength === 2) {
      return dataView.getUint16(0, littleEndian);
    } else if (dataView.byteLength === 4) {
      return dataView.getUint32(0, littleEndian);
    }
    return 0;
  }
  
  // Integer'ı veriye çevir
  static intToData(value, byteLength = 4, littleEndian = true) {
    const buffer = new ArrayBuffer(byteLength);
    const dataView = new DataView(buffer);
    
    if (byteLength === 1) {
      dataView.setUint8(0, value);
    } else if (byteLength === 2) {
      dataView.setUint16(0, value, littleEndian);
    } else if (byteLength === 4) {
      dataView.setUint32(0, value, littleEndian);
    }
    
    return dataView;
  }
}
```

**Alternatifler:**
- **Web Serial API**: Web Serial API
- **Web USB API**: Web USB API
- **Web NFC API**: Web NFC API
- **WebSocket**: WebSocket
- **HTTP APIs**: HTTP API'leri

**Ne Zaman Kullanılmalı?**
- ✅ IoT cihazları gerektiğinde
- ✅ Giyilebilir cihazlar gerektiğinde
- ✅ Tıbbi cihazlar gerektiğinde
- ✅ Akıllı ev gerektiğinde
- ✅ Fitness takibi gerektiğinde
- ✅ Gerçek zamanlı veri gerektiğinde
- ✅ Cihaz kontrolü gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Bluetooth desteği olmayan cihazlarda
- ❌ Güvenlik kritik uygulamalarda
- ❌ Basit veri transferi yeterli olduğunda

**Kullanılmazsa Ne Olur?**
- **No IoT Integration**: IoT entegrasyonu olmaz
- **No Wearable Support**: Giyilebilir cihaz desteği olmaz
- **No Medical Device Access**: Tıbbi cihaz erişimi olmaz
- **No Smart Home Control**: Akıllı ev kontrolü olmaz
- **Limited Device Communication**: Cihaz iletişimi sınırlı olur

**Modern Alternatifler:**
- **Web Serial API**: Web Serial API
- **Web USB API**: Web USB API
- **Web NFC API**: Web NFC API
- **WebSocket**: WebSocket
- **HTTP APIs**: HTTP API'leri

### Web Crypto API

**Web Crypto API Nedir?**
Web Crypto API, web uygulamalarının güvenli kriptografik işlemler gerçekleştirmesini sağlayan modern bir web API'sidir. Şifreleme, şifre çözme, dijital imzalama, hash oluşturma, anahtar üretimi ve güvenli rastgele sayı üretimi gibi kriptografik işlemler yapar. Güvenli veri transferi, kimlik doğrulama, dijital imza, şifreli depolama ve güvenlik protokolleri için kritik öneme sahiptir. Yüksek performanslı, donanım destekli kriptografik işlemler sağlar.

**Neden Kullanılır?**
- **Data Encryption**: Veri şifreleme
- **Digital Signatures**: Dijital imzalar
- **Authentication**: Kimlik doğrulama
- **Secure Communication**: Güvenli iletişim
- **Password Hashing**: Şifre hash'leme
- **Key Management**: Anahtar yönetimi
- **Random Generation**: Rastgele sayı üretimi

**Temel Kullanım Örnekleri:**

```javascript
// Anahtar çifti oluştur
const keyPair = await crypto.subtle.generateKey(
  {
    name: 'RSA-OAEP',
    modulusLength: 2048,
    publicExponent: new Uint8Array([1, 0, 1]),
    hash: 'SHA-256'
  },
  true,
  ['encrypt', 'decrypt']
);

// Veri şifrele
const data = new TextEncoder().encode('Gizli mesaj');
const encrypted = await crypto.subtle.encrypt(
  { name: 'RSA-OAEP' },
  keyPair.publicKey,
  data
);

// Veri şifresini çöz
const decrypted = await crypto.subtle.decrypt(
  { name: 'RSA-OAEP' },
  keyPair.privateKey,
  encrypted
);

// Hash oluştur
const hash = await crypto.subtle.digest('SHA-256', data);

// Güvenli rastgele sayı
const randomBytes = crypto.getRandomValues(new Uint8Array(16));

// Kriptografi yöneticisi
class CryptoManager {
  constructor() {
    this.isSupported = 'crypto' in window && 'subtle' in crypto;
    this.keys = new Map();
    this.algorithms = {
      rsa: { name: 'RSA-OAEP', hash: 'SHA-256' },
      aes: { name: 'AES-GCM', length: 256 },
      hmac: { name: 'HMAC', hash: 'SHA-256' },
      ecdsa: { name: 'ECDSA', namedCurve: 'P-256' }
    };
  }
  
  // Destek kontrolü
  isCryptoSupported() {
    return this.isSupported;
  }
  
  // RSA anahtar çifti oluştur
  async generateRSAKeyPair(modulusLength = 2048) {
    if (!this.isSupported) {
      throw new Error('Web Crypto API desteklenmiyor');
    }
    
    try {
      const keyPair = await crypto.subtle.generateKey(
        {
          name: 'RSA-OAEP',
          modulusLength,
          publicExponent: new Uint8Array([1, 0, 1]),
          hash: 'SHA-256'
        },
        true,
        ['encrypt', 'decrypt']
      );
      
      const keyId = this.generateKeyId();
      this.keys.set(keyId, {
        type: 'RSA',
        keyPair,
        created: Date.now()
      });
      
      return { keyId, keyPair };
    } catch (error) {
      throw new Error(`RSA anahtar oluşturma hatası: ${error.message}`);
    }
  }
  
  // AES anahtarı oluştur
  async generateAESKey(length = 256) {
    if (!this.isSupported) {
      throw new Error('Web Crypto API desteklenmiyor');
    }
    
    try {
      const key = await crypto.subtle.generateKey(
        {
          name: 'AES-GCM',
          length
        },
        true,
        ['encrypt', 'decrypt']
      );
      
      const keyId = this.generateKeyId();
      this.keys.set(keyId, {
        type: 'AES',
        key,
        created: Date.now()
      });
      
      return { keyId, key };
    } catch (error) {
      throw new Error(`AES anahtar oluşturma hatası: ${error.message}`);
    }
  }
  
  // HMAC anahtarı oluştur
  async generateHMACKey() {
    if (!this.isSupported) {
      throw new Error('Web Crypto API desteklenmiyor');
    }
    
    try {
      const key = await crypto.subtle.generateKey(
        {
          name: 'HMAC',
          hash: 'SHA-256'
        },
        true,
        ['sign', 'verify']
      );
      
      const keyId = this.generateKeyId();
      this.keys.set(keyId, {
        type: 'HMAC',
        key,
        created: Date.now()
      });
      
      return { keyId, key };
    } catch (error) {
      throw new Error(`HMAC anahtar oluşturma hatası: ${error.message}`);
    }
  }
  
  // ECDSA anahtar çifti oluştur
  async generateECDSAKeyPair() {
    if (!this.isSupported) {
      throw new Error('Web Crypto API desteklenmiyor');
    }
    
    try {
      const keyPair = await crypto.subtle.generateKey(
        {
          name: 'ECDSA',
          namedCurve: 'P-256'
        },
        true,
        ['sign', 'verify']
      );
      
      const keyId = this.generateKeyId();
      this.keys.set(keyId, {
        type: 'ECDSA',
        keyPair,
        created: Date.now()
      });
      
      return { keyId, keyPair };
    } catch (error) {
      throw new Error(`ECDSA anahtar oluşturma hatası: ${error.message}`);
    }
  }
  
  // Veri şifrele
  async encryptData(keyId, data, algorithm = 'AES-GCM') {
    const keyData = this.keys.get(keyId);
    if (!keyData) {
      throw new Error('Anahtar bulunamadı');
    }
    
    try {
      let encrypted;
      
      if (algorithm === 'AES-GCM') {
        const iv = crypto.getRandomValues(new Uint8Array(12));
        encrypted = await crypto.subtle.encrypt(
          { name: 'AES-GCM', iv },
          keyData.key,
          data
        );
        
        // IV'yi encrypted data ile birleştir
        const result = new Uint8Array(iv.length + encrypted.byteLength);
        result.set(iv, 0);
        result.set(new Uint8Array(encrypted), iv.length);
        
        return result;
      } else if (algorithm === 'RSA-OAEP') {
        encrypted = await crypto.subtle.encrypt(
          { name: 'RSA-OAEP' },
          keyData.keyPair.publicKey,
          data
        );
        return new Uint8Array(encrypted);
      }
      
      throw new Error(`Desteklenmeyen şifreleme algoritması: ${algorithm}`);
    } catch (error) {
      throw new Error(`Şifreleme hatası: ${error.message}`);
    }
  }
  
  // Veri şifresini çöz
  async decryptData(keyId, encryptedData, algorithm = 'AES-GCM') {
    const keyData = this.keys.get(keyId);
    if (!keyData) {
      throw new Error('Anahtar bulunamadı');
    }
    
    try {
      let decrypted;
      
      if (algorithm === 'AES-GCM') {
        const iv = encryptedData.slice(0, 12);
        const data = encryptedData.slice(12);
        
        decrypted = await crypto.subtle.decrypt(
          { name: 'AES-GCM', iv },
          keyData.key,
          data
        );
      } else if (algorithm === 'RSA-OAEP') {
        decrypted = await crypto.subtle.decrypt(
          { name: 'RSA-OAEP' },
          keyData.keyPair.privateKey,
          encryptedData
        );
      }
      
      return new Uint8Array(decrypted);
    } catch (error) {
      throw new Error(`Şifre çözme hatası: ${error.message}`);
    }
  }
  
  // Dijital imza oluştur
  async signData(keyId, data, algorithm = 'ECDSA') {
    const keyData = this.keys.get(keyId);
    if (!keyData) {
      throw new Error('Anahtar bulunamadı');
    }
    
    try {
      let signature;
      
      if (algorithm === 'ECDSA') {
        signature = await crypto.subtle.sign(
          { name: 'ECDSA', hash: 'SHA-256' },
          keyData.keyPair.privateKey,
          data
        );
      } else if (algorithm === 'HMAC') {
        signature = await crypto.subtle.sign(
          { name: 'HMAC' },
          keyData.key,
          data
        );
      }
      
      return new Uint8Array(signature);
    } catch (error) {
      throw new Error(`İmza oluşturma hatası: ${error.message}`);
    }
  }
  
  // Dijital imzayı doğrula
  async verifySignature(keyId, data, signature, algorithm = 'ECDSA') {
    const keyData = this.keys.get(keyId);
    if (!keyData) {
      throw new Error('Anahtar bulunamadı');
    }
    
    try {
      let isValid;
      
      if (algorithm === 'ECDSA') {
        isValid = await crypto.subtle.verify(
          { name: 'ECDSA', hash: 'SHA-256' },
          keyData.keyPair.publicKey,
          signature,
          data
        );
      } else if (algorithm === 'HMAC') {
        isValid = await crypto.subtle.verify(
          { name: 'HMAC' },
          keyData.key,
          signature,
          data
        );
      }
      
      return isValid;
    } catch (error) {
      throw new Error(`İmza doğrulama hatası: ${error.message}`);
    }
  }
  
  // Hash oluştur
  async createHash(data, algorithm = 'SHA-256') {
    if (!this.isSupported) {
      throw new Error('Web Crypto API desteklenmiyor');
    }
    
    try {
      const hash = await crypto.subtle.digest(algorithm, data);
      return new Uint8Array(hash);
    } catch (error) {
      throw new Error(`Hash oluşturma hatası: ${error.message}`);
    }
  }
  
  // Güvenli rastgele sayı üret
  generateRandomBytes(length) {
    if (!this.isSupported) {
      throw new Error('Web Crypto API desteklenmiyor');
    }
    
    return crypto.getRandomValues(new Uint8Array(length));
  }
  
  // Güvenli rastgele string üret
  generateRandomString(length, charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789') {
    const randomBytes = this.generateRandomBytes(length);
    let result = '';
    
    for (let i = 0; i < length; i++) {
      result += charset[randomBytes[i] % charset.length];
    }
    
    return result;
  }
  
  // Anahtar ID oluştur
  generateKeyId() {
    return 'key_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Anahtar listesini al
  getKeys() {
    return Array.from(this.keys.keys());
  }
  
  // Anahtar bilgilerini al
  getKeyInfo(keyId) {
    const keyData = this.keys.get(keyId);
    if (!keyData) return null;
    
    return {
      type: keyData.type,
      created: keyData.created,
      age: Date.now() - keyData.created
    };
  }
  
  // Anahtarı kaldır
  removeKey(keyId) {
    return this.keys.delete(keyId);
  }
  
  // Tüm anahtarları temizle
  clearKeys() {
    this.keys.clear();
  }
}

// Şifre yöneticisi
class PasswordManager {
  constructor(cryptoManager) {
    this.cryptoManager = cryptoManager;
    this.saltLength = 16;
    this.iterations = 100000;
  }
  
  // Şifre hash'le
  async hashPassword(password, salt = null) {
    if (!salt) {
      salt = this.cryptoManager.generateRandomBytes(this.saltLength);
    }
    
    const encoder = new TextEncoder();
    const passwordData = encoder.encode(password);
    
    // PBKDF2 ile şifre hash'le
    const key = await crypto.subtle.importKey(
      'raw',
      passwordData,
      'PBKDF2',
      false,
      ['deriveBits']
    );
    
    const derivedBits = await crypto.subtle.deriveBits(
      {
        name: 'PBKDF2',
        salt: salt,
        iterations: this.iterations,
        hash: 'SHA-256'
      },
      key,
      256
    );
    
    return {
      hash: new Uint8Array(derivedBits),
      salt: salt
    };
  }
  
  // Şifre doğrula
  async verifyPassword(password, hash, salt) {
    const { hash: computedHash } = await this.hashPassword(password, salt);
    
    // Hash'leri karşılaştır
    if (computedHash.length !== hash.length) {
      return false;
    }
    
    for (let i = 0; i < computedHash.length; i++) {
      if (computedHash[i] !== hash[i]) {
        return false;
      }
    }
    
    return true;
  }
  
  // Güçlü şifre oluştur
  generateStrongPassword(length = 16) {
    const charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*';
    return this.cryptoManager.generateRandomString(length, charset);
  }
  
  // Şifre gücünü kontrol et
  checkPasswordStrength(password) {
    let score = 0;
    const checks = {
      length: password.length >= 8,
      lowercase: /[a-z]/.test(password),
      uppercase: /[A-Z]/.test(password),
      numbers: /\d/.test(password),
      symbols: /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password)
    };
    
    Object.values(checks).forEach(check => {
      if (check) score++;
    });
    
    if (score < 3) return 'weak';
    if (score < 5) return 'medium';
    return 'strong';
  }
}

// Kriptografi utility fonksiyonları
class CryptoUtils {
  // Hex string'e çevir
  static arrayBufferToHex(buffer) {
    const bytes = new Uint8Array(buffer);
    return Array.from(bytes)
      .map(byte => byte.toString(16).padStart(2, '0'))
      .join('');
  }
  
  // Hex string'den array buffer'a çevir
  static hexToArrayBuffer(hex) {
    const bytes = new Uint8Array(hex.length / 2);
    for (let i = 0; i < hex.length; i += 2) {
      bytes[i / 2] = parseInt(hex.substr(i, 2), 16);
    }
    return bytes.buffer;
  }
  
  // Base64'e çevir
  static arrayBufferToBase64(buffer) {
    const bytes = new Uint8Array(buffer);
    let binary = '';
    for (let i = 0; i < bytes.byteLength; i++) {
      binary += String.fromCharCode(bytes[i]);
    }
    return btoa(binary);
  }
  
  // Base64'den array buffer'a çevir
  static base64ToArrayBuffer(base64) {
    const binary = atob(base64);
    const bytes = new Uint8Array(binary.length);
    for (let i = 0; i < binary.length; i++) {
      bytes[i] = binary.charCodeAt(i);
    }
    return bytes.buffer;
  }
  
  // String'i array buffer'a çevir
  static stringToArrayBuffer(string) {
    const encoder = new TextEncoder();
    return encoder.encode(string);
  }
  
  // Array buffer'ı string'e çevir
  static arrayBufferToString(buffer) {
    const decoder = new TextDecoder();
    return decoder.decode(buffer);
  }
  
  // Anahtar export et
  static async exportKey(key, format = 'raw') {
    return await crypto.subtle.exportKey(format, key);
  }
  
  // Anahtar import et
  static async importKey(format, keyData, algorithm, extractable, keyUsages) {
    return await crypto.subtle.importKey(format, keyData, algorithm, extractable, keyUsages);
  }
  
  // Anahtar wrap et
  static async wrapKey(format, key, wrappingKey, wrapAlgorithm) {
    return await crypto.subtle.wrapKey(format, key, wrappingKey, wrapAlgorithm);
  }
  
  // Anahtar unwrap et
  static async unwrapKey(format, wrappedKey, unwrappingKey, unwrapAlgorithm, unwrappedKeyAlgorithm, extractable, keyUsages) {
    return await crypto.subtle.unwrapKey(format, wrappedKey, unwrappingKey, unwrapAlgorithm, unwrappedKeyAlgorithm, extractable, keyUsages);
  }
}
```

**Alternatifler:**
- **Third-party Libraries**: Harici kriptografi kütüphaneleri
- **Server-side Crypto**: Sunucu tarafı kriptografi
- **Hardware Security**: Donanım güvenliği
- **TLS/SSL**: TLS/SSL protokolleri
- **JWT**: JSON Web Tokens

**Ne Zaman Kullanılmalı?**
- ✅ Veri şifreleme gerektiğinde
- ✅ Dijital imza gerektiğinde
- ✅ Kimlik doğrulama gerektiğinde
- ✅ Güvenli iletişim gerektiğinde
- ✅ Şifre hash'leme gerektiğinde
- ✅ Anahtar yönetimi gerektiğinde
- ✅ Rastgele sayı üretimi gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit hash işlemleri yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ Güvenlik gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No Data Encryption**: Veri şifreleme olmaz
- **No Digital Signatures**: Dijital imza olmaz
- **No Authentication**: Kimlik doğrulama olmaz
- **No Secure Communication**: Güvenli iletişim olmaz
- **Limited Security**: Güvenlik sınırlı olur

**Modern Alternatifler:**
- **Web Crypto API**: Web Crypto API
- **Third-party Libraries**: Harici kriptografi kütüphaneleri
- **Server-side Crypto**: Sunucu tarafı kriptografi
- **Hardware Security**: Donanım güvenliği
- **TLS/SSL**: TLS/SSL protokolleri

### WebGL API

**WebGL API Nedir?**
WebGL API, web uygulamalarının OpenGL ES 2.0 tabanlı 3D grafikler oluşturmasını sağlayan güçlü bir web API'sidir. Canvas element üzerinde donanım hızlandırmalı 3D grafikler, animasyonlar, oyunlar ve görselleştirmeler oluşturur. Vertex shader'lar, fragment shader'lar, buffer'lar, texture'lar ve render pipeline'ı ile profesyonel 3D grafik işleme sağlar. Oyun geliştirme, 3D görselleştirme, bilimsel simülasyonlar ve interaktif 3D deneyimler için kritik öneme sahiptir.

**Neden Kullanılır?**
- **3D Graphics**: 3D grafikler
- **Games**: Oyunlar
- **Visualizations**: Görselleştirmeler
- **Animations**: Animasyonlar
- **Simulations**: Simülasyonlar
- **Interactive 3D**: İnteraktif 3D
- **Hardware Acceleration**: Donanım hızlandırması

**Temel Kullanım Örnekleri:**

```javascript
// WebGL bağlamı oluştur
const canvas = document.getElementById('glCanvas');
const gl = canvas.getContext('webgl');

// Basit üçgen çiz
const vertices = [
  0.0, 0.5, 0.0,
  -0.5, -0.5, 0.0,
  0.5, -0.5, 0.0
];

const vertexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

// Vertex shader
const vertexShaderSource = `
  attribute vec4 a_position;
  void main() {
    gl_Position = a_position;
  }
`;

// Fragment shader
const fragmentShaderSource = `
  precision mediump float;
  void main() {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
  }
`;

// Shader'ları derle ve program oluştur
const vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
const fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);
const program = createProgram(gl, vertexShader, fragmentShader);

// Render döngüsü
function render() {
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.useProgram(program);
  
  const positionAttributeLocation = gl.getAttribLocation(program, 'a_position');
  gl.enableVertexAttribArray(positionAttributeLocation);
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  gl.vertexAttribPointer(positionAttributeLocation, 3, gl.FLOAT, false, 0, 0);
  
  gl.drawArrays(gl.TRIANGLES, 0, 3);
  requestAnimationFrame(render);
}

// WebGL yöneticisi
class WebGLManager {
  constructor(canvas) {
    this.canvas = canvas;
    this.gl = null;
    this.programs = new Map();
    this.buffers = new Map();
    this.textures = new Map();
    this.isInitialized = false;
    this.renderLoop = null;
  }
  
  // WebGL bağlamını başlat
  initialize() {
    try {
      this.gl = this.canvas.getContext('webgl') || this.canvas.getContext('experimental-webgl');
      
      if (!this.gl) {
        throw new Error('WebGL desteklenmiyor');
      }
      
      // WebGL ayarları
      this.gl.enable(this.gl.DEPTH_TEST);
      this.gl.enable(this.gl.CULL_FACE);
      this.gl.cullFace(this.gl.BACK);
      this.gl.frontFace(this.gl.CCW);
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('WebGL başlatma hatası:', error);
      return false;
    }
  }
  
  // Shader oluştur
  createShader(type, source) {
    const shader = this.gl.createShader(type);
    this.gl.shaderSource(shader, source);
    this.gl.compileShader(shader);
    
    if (!this.gl.getShaderParameter(shader, this.gl.COMPILE_STATUS)) {
      console.error('Shader derleme hatası:', this.gl.getShaderInfoLog(shader));
      this.gl.deleteShader(shader);
      return null;
    }
    
    return shader;
  }
  
  // Program oluştur
  createProgram(vertexShaderSource, fragmentShaderSource) {
    const vertexShader = this.createShader(this.gl.VERTEX_SHADER, vertexShaderSource);
    const fragmentShader = this.createShader(this.gl.FRAGMENT_SHADER, fragmentShaderSource);
    
    if (!vertexShader || !fragmentShader) {
      return null;
    }
    
    const program = this.gl.createProgram();
    this.gl.attachShader(program, vertexShader);
    this.gl.attachShader(program, fragmentShader);
    this.gl.linkProgram(program);
    
    if (!this.gl.getProgramParameter(program, this.gl.LINK_STATUS)) {
      console.error('Program bağlama hatası:', this.gl.getProgramInfoLog(program));
      this.gl.deleteProgram(program);
      return null;
    }
    
    const programId = this.generateId();
    this.programs.set(programId, program);
    
    return { program, programId };
  }
  
  // Buffer oluştur
  createBuffer(data, usage = this.gl.STATIC_DRAW) {
    const buffer = this.gl.createBuffer();
    this.gl.bindBuffer(this.gl.ARRAY_BUFFER, buffer);
    this.gl.bufferData(this.gl.ARRAY_BUFFER, new Float32Array(data), usage);
    
    const bufferId = this.generateId();
    this.buffers.set(bufferId, buffer);
    
    return { buffer, bufferId };
  }
  
  // Index buffer oluştur
  createIndexBuffer(indices, usage = this.gl.STATIC_DRAW) {
    const buffer = this.gl.createBuffer();
    this.gl.bindBuffer(this.gl.ELEMENT_ARRAY_BUFFER, buffer);
    this.gl.bufferData(this.gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), usage);
    
    const bufferId = this.generateId();
    this.buffers.set(bufferId, buffer);
    
    return { buffer, bufferId };
  }
  
  // Texture oluştur
  createTexture(image = null) {
    const texture = this.gl.createTexture();
    this.gl.bindTexture(this.gl.TEXTURE_2D, texture);
    
    if (image) {
      this.gl.texImage2D(this.gl.TEXTURE_2D, 0, this.gl.RGBA, this.gl.RGBA, this.gl.UNSIGNED_BYTE, image);
    } else {
      // Boş texture
      this.gl.texImage2D(this.gl.TEXTURE_2D, 0, this.gl.RGBA, 1, 1, 0, this.gl.RGBA, this.gl.UNSIGNED_BYTE, new Uint8Array([0, 0, 0, 255]));
    }
    
    // Texture parametreleri
    this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_WRAP_S, this.gl.CLAMP_TO_EDGE);
    this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_WRAP_T, this.gl.CLAMP_TO_EDGE);
    this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_MIN_FILTER, this.gl.LINEAR);
    this.gl.texParameteri(this.gl.TEXTURE_2D, this.gl.TEXTURE_MAG_FILTER, this.gl.LINEAR);
    
    const textureId = this.generateId();
    this.textures.set(textureId, texture);
    
    return { texture, textureId };
  }
  
  // Texture'ı güncelle
  updateTexture(textureId, image) {
    const texture = this.textures.get(textureId);
    if (!texture) return false;
    
    this.gl.bindTexture(this.gl.TEXTURE_2D, texture);
    this.gl.texImage2D(this.gl.TEXTURE_2D, 0, this.gl.RGBA, this.gl.RGBA, this.gl.UNSIGNED_BYTE, image);
    
    return true;
  }
  
  // Render döngüsünü başlat
  startRenderLoop(renderFunction) {
    if (this.renderLoop) {
      this.stopRenderLoop();
    }
    
    const loop = () => {
      renderFunction();
      this.renderLoop = requestAnimationFrame(loop);
    };
    
    this.renderLoop = requestAnimationFrame(loop);
  }
  
  // Render döngüsünü durdur
  stopRenderLoop() {
    if (this.renderLoop) {
      cancelAnimationFrame(this.renderLoop);
      this.renderLoop = null;
    }
  }
  
  // Canvas'ı temizle
  clear(r = 0, g = 0, b = 0, a = 1) {
    this.gl.clearColor(r, g, b, a);
    this.gl.clear(this.gl.COLOR_BUFFER_BIT | this.gl.DEPTH_BUFFER_BIT);
  }
  
  // Viewport ayarla
  setViewport(x = 0, y = 0, width = null, height = null) {
    const w = width || this.canvas.width;
    const h = height || this.canvas.height;
    this.gl.viewport(x, y, w, h);
  }
  
  // Program kullan
  useProgram(programId) {
    const program = this.programs.get(programId);
    if (program) {
      this.gl.useProgram(program);
      return true;
    }
    return false;
  }
  
  // Buffer bağla
  bindBuffer(bufferId, target = this.gl.ARRAY_BUFFER) {
    const buffer = this.buffers.get(bufferId);
    if (buffer) {
      this.gl.bindBuffer(target, buffer);
      return true;
    }
    return false;
  }
  
  // Texture bağla
  bindTexture(textureId, unit = 0) {
    const texture = this.textures.get(textureId);
    if (texture) {
      this.gl.activeTexture(this.gl.TEXTURE0 + unit);
      this.gl.bindTexture(this.gl.TEXTURE_2D, texture);
      return true;
    }
    return false;
  }
  
  // Uniform değer ayarla
  setUniform(programId, name, value, type = 'float') {
    const program = this.programs.get(programId);
    if (!program) return false;
    
    const location = this.gl.getUniformLocation(program, name);
    if (!location) return false;
    
    switch (type) {
      case 'float':
        this.gl.uniform1f(location, value);
        break;
      case 'int':
        this.gl.uniform1i(location, value);
        break;
      case 'vec2':
        this.gl.uniform2f(location, value[0], value[1]);
        break;
      case 'vec3':
        this.gl.uniform3f(location, value[0], value[1], value[2]);
        break;
      case 'vec4':
        this.gl.uniform4f(location, value[0], value[1], value[2], value[3]);
        break;
      case 'mat4':
        this.gl.uniformMatrix4fv(location, false, value);
        break;
    }
    
    return true;
  }
  
  // Attribute ayarla
  setAttribute(programId, name, size, type = this.gl.FLOAT, normalized = false, stride = 0, offset = 0) {
    const program = this.programs.get(programId);
    if (!program) return false;
    
    const location = this.gl.getAttribLocation(program, name);
    if (location === -1) return false;
    
    this.gl.enableVertexAttribArray(location);
    this.gl.vertexAttribPointer(location, size, type, normalized, stride, offset);
    
    return true;
  }
  
  // Draw call
  drawArrays(mode, first, count) {
    this.gl.drawArrays(mode, first, count);
  }
  
  // Indexed draw call
  drawElements(mode, count, type = this.gl.UNSIGNED_SHORT, offset = 0) {
    this.gl.drawElements(mode, count, type, offset);
  }
  
  // ID oluştur
  generateId() {
    return 'webgl_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Kaynakları temizle
  cleanup() {
    this.stopRenderLoop();
    
    // Program'ları temizle
    this.programs.forEach(program => {
      this.gl.deleteProgram(program);
    });
    this.programs.clear();
    
    // Buffer'ları temizle
    this.buffers.forEach(buffer => {
      this.gl.deleteBuffer(buffer);
    });
    this.buffers.clear();
    
    // Texture'ları temizle
    this.textures.forEach(texture => {
      this.gl.deleteTexture(texture);
    });
    this.textures.clear();
  }
  
  // WebGL bilgilerini al
  getInfo() {
    return {
      vendor: this.gl.getParameter(this.gl.VENDOR),
      renderer: this.gl.getParameter(this.gl.RENDERER),
      version: this.gl.getParameter(this.gl.VERSION),
      shadingLanguageVersion: this.gl.getParameter(this.gl.SHADING_LANGUAGE_VERSION),
      maxTextureSize: this.gl.getParameter(this.gl.MAX_TEXTURE_SIZE),
      maxVertexAttribs: this.gl.getParameter(this.gl.MAX_VERTEX_ATTRIBS),
      maxVaryingVectors: this.gl.getParameter(this.gl.MAX_VARYING_VECTORS),
      maxFragmentUniforms: this.gl.getParameter(this.gl.MAX_FRAGMENT_UNIFORM_VECTORS),
      maxVertexUniforms: this.gl.getParameter(this.gl.MAX_VERTEX_UNIFORM_VECTORS)
    };
  }
}

// 3D nesne yöneticisi
class Object3DManager {
  constructor(webglManager) {
    this.webglManager = webglManager;
    this.objects = new Map();
    this.camera = {
      position: [0, 0, 5],
      target: [0, 0, 0],
      up: [0, 1, 0],
      fov: 45,
      aspect: 1,
      near: 0.1,
      far: 1000
    };
    this.lights = [];
  }
  
  // 3D nesne oluştur
  createObject(geometry, material, programId) {
    const object = {
      geometry,
      material,
      programId,
      position: [0, 0, 0],
      rotation: [0, 0, 0],
      scale: [1, 1, 1],
      matrix: this.createIdentityMatrix()
    };
    
    const objectId = this.webglManager.generateId();
    this.objects.set(objectId, object);
    
    return { object, objectId };
  }
  
  // Küp geometrisi oluştur
  createCubeGeometry(size = 1) {
    const s = size / 2;
    const vertices = [
      // Front face
      -s, -s,  s,  s, -s,  s,  s,  s,  s, -s,  s,  s,
      // Back face
      -s, -s, -s, -s,  s, -s,  s,  s, -s,  s, -s, -s,
      // Top face
      -s,  s, -s, -s,  s,  s,  s,  s,  s,  s,  s, -s,
      // Bottom face
      -s, -s, -s,  s, -s, -s,  s, -s,  s, -s, -s,  s,
      // Right face
       s, -s, -s,  s,  s, -s,  s,  s,  s,  s, -s,  s,
      // Left face
      -s, -s, -s, -s, -s,  s, -s,  s,  s, -s,  s, -s
    ];
    
    const indices = [
      0,  1,  2,    0,  2,  3,    // front
      4,  5,  6,    4,  6,  7,    // back
      8,  9, 10,    8, 10, 11,    // top
     12, 13, 14,   12, 14, 15,    // bottom
     16, 17, 18,   16, 18, 19,    // right
     20, 21, 22,   20, 22, 23     // left
    ];
    
    const normals = this.calculateNormals(vertices, indices);
    
    return { vertices, indices, normals };
  }
  
  // Normal hesapla
  calculateNormals(vertices, indices) {
    const normals = new Array(vertices.length).fill(0);
    
    for (let i = 0; i < indices.length; i += 3) {
      const i1 = indices[i] * 3;
      const i2 = indices[i + 1] * 3;
      const i3 = indices[i + 2] * 3;
      
      const v1 = [vertices[i1], vertices[i1 + 1], vertices[i1 + 2]];
      const v2 = [vertices[i2], vertices[i2 + 1], vertices[i2 + 2]];
      const v3 = [vertices[i3], vertices[i3 + 1], vertices[i3 + 2]];
      
      const edge1 = this.subtractVectors(v2, v1);
      const edge2 = this.subtractVectors(v3, v1);
      const normal = this.crossProduct(edge1, edge2);
      this.normalizeVector(normal);
      
      for (let j = 0; j < 3; j++) {
        normals[i1 + j] += normal[j];
        normals[i2 + j] += normal[j];
        normals[i3 + j] += normal[j];
      }
    }
    
    // Normalize et
    for (let i = 0; i < normals.length; i += 3) {
      const normal = [normals[i], normals[i + 1], normals[i + 2]];
      this.normalizeVector(normal);
      normals[i] = normal[0];
      normals[i + 1] = normal[1];
      normals[i + 2] = normal[2];
    }
    
    return normals;
  }
  
  // Vektör çıkarma
  subtractVectors(a, b) {
    return [a[0] - b[0], a[1] - b[1], a[2] - b[2]];
  }
  
  // Çapraz çarpım
  crossProduct(a, b) {
    return [
      a[1] * b[2] - a[2] * b[1],
      a[2] * b[0] - a[0] * b[2],
      a[0] * b[1] - a[1] * b[0]
    ];
  }
  
  // Vektör normalize et
  normalizeVector(v) {
    const length = Math.sqrt(v[0] * v[0] + v[1] * v[1] + v[2] * v[2]);
    if (length > 0) {
      v[0] /= length;
      v[1] /= length;
      v[2] /= length;
    }
  }
  
  // Identity matrix oluştur
  createIdentityMatrix() {
    return [
      1, 0, 0, 0,
      0, 1, 0, 0,
      0, 0, 1, 0,
      0, 0, 0, 1
    ];
  }
  
  // Translation matrix
  createTranslationMatrix(x, y, z) {
    return [
      1, 0, 0, 0,
      0, 1, 0, 0,
      0, 0, 1, 0,
      x, y, z, 1
    ];
  }
  
  // Rotation matrix (X ekseni)
  createRotationXMatrix(angle) {
    const c = Math.cos(angle);
    const s = Math.sin(angle);
    return [
      1, 0, 0, 0,
      0, c, s, 0,
      0, -s, c, 0,
      0, 0, 0, 1
    ];
  }
  
  // Rotation matrix (Y ekseni)
  createRotationYMatrix(angle) {
    const c = Math.cos(angle);
    const s = Math.sin(angle);
    return [
      c, 0, -s, 0,
      0, 1, 0, 0,
      s, 0, c, 0,
      0, 0, 0, 1
    ];
  }
  
  // Rotation matrix (Z ekseni)
  createRotationZMatrix(angle) {
    const c = Math.cos(angle);
    const s = Math.sin(angle);
    return [
      c, s, 0, 0,
      -s, c, 0, 0,
      0, 0, 1, 0,
      0, 0, 0, 1
    ];
  }
  
  // Scale matrix
  createScaleMatrix(x, y, z) {
    return [
      x, 0, 0, 0,
      0, y, 0, 0,
      0, 0, z, 0,
      0, 0, 0, 1
    ];
  }
  
  // Matrix çarpımı
  multiplyMatrices(a, b) {
    const result = new Array(16);
    
    for (let i = 0; i < 4; i++) {
      for (let j = 0; j < 4; j++) {
        result[i * 4 + j] = 0;
        for (let k = 0; k < 4; k++) {
          result[i * 4 + j] += a[i * 4 + k] * b[k * 4 + j];
        }
      }
    }
    
    return result;
  }
  
  // Perspective projection matrix
  createPerspectiveMatrix(fov, aspect, near, far) {
    const f = Math.tan(Math.PI * 0.5 - 0.5 * fov * Math.PI / 180);
    const rangeInv = 1.0 / (near - far);
    
    return [
      f / aspect, 0, 0, 0,
      0, f, 0, 0,
      0, 0, (near + far) * rangeInv, -1,
      0, 0, near * far * rangeInv * 2, 0
    ];
  }
  
  // Look-at matrix
  createLookAtMatrix(eye, target, up) {
    const zAxis = this.normalize(this.subtractVectors(eye, target));
    const xAxis = this.normalize(this.crossProduct(up, zAxis));
    const yAxis = this.crossProduct(zAxis, xAxis);
    
    return [
      xAxis[0], xAxis[1], xAxis[2], 0,
      yAxis[0], yAxis[1], yAxis[2], 0,
      zAxis[0], zAxis[1], zAxis[2], 0,
      eye[0], eye[1], eye[2], 1
    ];
  }
  
  // Vektör normalize et
  normalize(v) {
    const length = Math.sqrt(v[0] * v[0] + v[1] * v[1] + v[2] * v[2]);
    if (length > 0) {
      return [v[0] / length, v[1] / length, v[2] / length];
    }
    return [0, 0, 0];
  }
  
  // Nesne güncelle
  updateObject(objectId, position = null, rotation = null, scale = null) {
    const object = this.objects.get(objectId);
    if (!object) return false;
    
    if (position) object.position = [...position];
    if (rotation) object.rotation = [...rotation];
    if (scale) object.scale = [...scale];
    
    // Transform matrix hesapla
    const translationMatrix = this.createTranslationMatrix(object.position[0], object.position[1], object.position[2]);
    const rotationXMatrix = this.createRotationXMatrix(object.rotation[0]);
    const rotationYMatrix = this.createRotationYMatrix(object.rotation[1]);
    const rotationZMatrix = this.createRotationZMatrix(object.rotation[2]);
    const scaleMatrix = this.createScaleMatrix(object.scale[0], object.scale[1], object.scale[2]);
    
    let matrix = this.multiplyMatrices(scaleMatrix, rotationZMatrix);
    matrix = this.multiplyMatrices(matrix, rotationYMatrix);
    matrix = this.multiplyMatrices(matrix, rotationXMatrix);
    matrix = this.multiplyMatrices(matrix, translationMatrix);
    
    object.matrix = matrix;
    return true;
  }
  
  // Kamera güncelle
  updateCamera(position = null, target = null, up = null) {
    if (position) this.camera.position = [...position];
    if (target) this.camera.target = [...target];
    if (up) this.camera.up = [...up];
  }
  
  // Render
  render() {
    // View matrix hesapla
    const viewMatrix = this.createLookAtMatrix(this.camera.position, this.camera.target, this.camera.up);
    
    // Projection matrix hesapla
    const projectionMatrix = this.createPerspectiveMatrix(
      this.camera.fov,
      this.camera.aspect,
      this.camera.near,
      this.camera.far
    );
    
    // Tüm nesneleri render et
    this.objects.forEach((object, objectId) => {
      this.webglManager.useProgram(object.programId);
      
      // Matrix'leri uniform olarak gönder
      this.webglManager.setUniform(object.programId, 'u_modelMatrix', object.matrix, 'mat4');
      this.webglManager.setUniform(object.programId, 'u_viewMatrix', viewMatrix, 'mat4');
      this.webglManager.setUniform(object.programId, 'u_projectionMatrix', projectionMatrix, 'mat4');
      
      // Geometry render et
      this.renderGeometry(object.geometry);
    });
  }
  
  // Geometry render et
  renderGeometry(geometry) {
    // Vertex buffer'ı bağla
    this.webglManager.bindBuffer(geometry.vertexBufferId);
    this.webglManager.setAttribute(geometry.programId, 'a_position', 3);
    
    // Normal buffer'ı bağla
    if (geometry.normalBufferId) {
      this.webglManager.bindBuffer(geometry.normalBufferId);
      this.webglManager.setAttribute(geometry.programId, 'a_normal', 3);
    }
    
    // Index buffer varsa indexed draw
    if (geometry.indexBufferId) {
      this.webglManager.bindBuffer(geometry.indexBufferId, this.webglManager.gl.ELEMENT_ARRAY_BUFFER);
      this.webglManager.drawElements(this.webglManager.gl.TRIANGLES, geometry.indices.length);
    } else {
      this.webglManager.drawArrays(this.webglManager.gl.TRIANGLES, 0, geometry.vertices.length / 3);
    }
  }
  
  // Nesne kaldır
  removeObject(objectId) {
    return this.objects.delete(objectId);
  }
  
  // Tüm nesneleri temizle
  clearObjects() {
    this.objects.clear();
  }
}
```

**Alternatifler:**
- **WebGL 2.0**: WebGL 2.0
- **WebGPU**: WebGPU
- **Three.js**: Three.js kütüphanesi
- **Babylon.js**: Babylon.js kütüphanesi
- **Canvas 2D**: Canvas 2D API

**Ne Zaman Kullanılmalı?**
- ✅ 3D grafikler gerektiğinde
- ✅ Oyunlar gerektiğinde
- ✅ Görselleştirmeler gerektiğinde
- ✅ Animasyonlar gerektiğinde
- ✅ Simülasyonlar gerektiğinde
- ✅ İnteraktif 3D gerektiğinde
- ✅ Donanım hızlandırması gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit 2D grafikler yeterli olduğunda
- ❌ Performans kritik olmadığında
- ❌ 3D grafik gerektirmediğinde

**Kullanılmazsa Ne Olur?**
- **No 3D Graphics**: 3D grafikler olmaz
- **No Games**: Oyunlar olmaz
- **No Visualizations**: Görselleştirmeler olmaz
- **No Animations**: Animasyonlar olmaz
- **Limited Graphics**: Grafik özellikleri sınırlı olur

**Modern Alternatifler:**
- **WebGL 2.0**: WebGL 2.0
- **WebGPU**: WebGPU
- **Three.js**: Three.js kütüphanesi
- **Babylon.js**: Babylon.js kütüphanesi
- **Canvas 2D**: Canvas 2D API

### WebGPU API

**WebGPU API Nedir?**
WebGPU API, web uygulamalarının modern GPU'ların gücünden yararlanmasını sağlayan yeni nesil bir web API'sidir. Vulkan, Metal ve DirectX 12 gibi modern grafik API'lerine benzer düşük seviye GPU erişimi sağlar. Yüksek performanslı 3D grafikler, GPU hesaplama, paralel işleme ve donanım hızlandırmalı uygulamalar için kritik öneme sahiptir. WebGL'den daha modern, daha performanslı ve daha esnek bir alternatif sunar.

**Neden Kullanılır?**
- **High Performance**: Yüksek performans
- **GPU Computing**: GPU hesaplama
- **Modern Graphics**: Modern grafikler
- **Parallel Processing**: Paralel işleme
- **Hardware Acceleration**: Donanım hızlandırması
- **Cross Platform**: Platform bağımsızlığı
- **Future Proof**: Gelecek güvenli

**Temel Kullanım Örnekleri:**

```javascript
// WebGPU başlatma
if ('gpu' in navigator) {
  const adapter = await navigator.gpu.requestAdapter();
  const device = await adapter.requestDevice();
  
  const shaderModule = device.createShaderModule({
    code: `
      @vertex
      fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
        var pos = array<vec2<f32>, 3>(
          vec2<f32>(0.0, 0.5),
          vec2<f32>(-0.5, -0.5),
          vec2<f32>(0.5, -0.5)
        );
        return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
      }
      
      @fragment
      fn fs_main() -> @location(0) vec4<f32> {
        return vec4<f32>(1.0, 0.0, 0.0, 1.0);
      }
    `
  });
  
  const renderPipeline = device.createRenderPipeline({
    layout: 'auto',
    vertex: {
      module: shaderModule,
      entryPoint: 'vs_main'
    },
    fragment: {
      module: shaderModule,
      entryPoint: 'fs_main',
      targets: [{
        format: 'bgra8unorm'
      }]
    },
    primitive: {
      topology: 'triangle-list'
    }
  });
  
  // Render döngüsü
  function render() {
    const commandEncoder = device.createCommandEncoder();
    const textureView = context.getCurrentTexture().createView();
    
    const renderPass = commandEncoder.beginRenderPass({
      colorAttachments: [{
        view: textureView,
        clearValue: { r: 0.0, g: 0.0, b: 0.0, a: 1.0 },
        loadOp: 'clear',
        storeOp: 'store'
      }]
    });
    
    renderPass.setPipeline(renderPipeline);
    renderPass.draw(3);
    renderPass.end();
    
    device.queue.submit([commandEncoder.finish()]);
    requestAnimationFrame(render);
  }
}

// WebGPU yöneticisi
class WebGPUManager {
  constructor() {
    this.isSupported = 'gpu' in navigator;
    this.adapter = null;
    this.device = null;
    this.context = null;
    this.canvas = null;
    this.isInitialized = false;
    this.pipelines = new Map();
    this.buffers = new Map();
    this.textures = new Map();
    this.bindGroups = new Map();
  }
  
  // WebGPU başlat
  async initialize(canvas) {
    if (!this.isSupported) {
      throw new Error('WebGPU desteklenmiyor');
    }
    
    try {
      this.canvas = canvas;
      
      // Adapter al
      this.adapter = await navigator.gpu.requestAdapter({
        powerPreference: 'high-performance'
      });
      
      if (!this.adapter) {
        throw new Error('WebGPU adapter bulunamadı');
      }
      
      // Device al
      this.device = await this.adapter.requestDevice({
        requiredFeatures: [],
        requiredLimits: {}
      });
      
      // Context oluştur
      this.context = this.canvas.getContext('webgpu');
      if (!this.context) {
        throw new Error('WebGPU context oluşturulamadı');
      }
      
      // Canvas format ayarla
      const canvasFormat = navigator.gpu.getPreferredCanvasFormat();
      this.context.configure({
        device: this.device,
        format: canvasFormat
      });
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('WebGPU başlatma hatası:', error);
      return false;
    }
  }
  
  // Shader modülü oluştur
  createShaderModule(code) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    return this.device.createShaderModule({ code });
  }
  
  // Render pipeline oluştur
  createRenderPipeline(descriptor) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    const pipeline = this.device.createRenderPipeline(descriptor);
    const pipelineId = this.generateId();
    this.pipelines.set(pipelineId, pipeline);
    
    return { pipeline, pipelineId };
  }
  
  // Compute pipeline oluştur
  createComputePipeline(descriptor) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    const pipeline = this.device.createComputePipeline(descriptor);
    const pipelineId = this.generateId();
    this.pipelines.set(pipelineId, pipeline);
    
    return { pipeline, pipelineId };
  }
  
  // Buffer oluştur
  createBuffer(size, usage) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    const buffer = this.device.createBuffer({
      size,
      usage
    });
    
    const bufferId = this.generateId();
    this.buffers.set(bufferId, buffer);
    
    return { buffer, bufferId };
  }
  
  // Vertex buffer oluştur
  createVertexBuffer(data) {
    const buffer = this.createBuffer(data.byteLength, GPUBufferUsage.VERTEX | GPUBufferUsage.COPY_DST);
    
    // Veriyi buffer'a yaz
    this.device.queue.writeBuffer(buffer.buffer, 0, data);
    
    return buffer;
  }
  
  // Index buffer oluştur
  createIndexBuffer(data) {
    const buffer = this.createBuffer(data.byteLength, GPUBufferUsage.INDEX | GPUBufferUsage.COPY_DST);
    
    // Veriyi buffer'a yaz
    this.device.queue.writeBuffer(buffer.buffer, 0, data);
    
    return buffer;
  }
  
  // Uniform buffer oluştur
  createUniformBuffer(size) {
    return this.createBuffer(size, GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST);
  }
  
  // Storage buffer oluştur
  createStorageBuffer(size) {
    return this.createBuffer(size, GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_DST);
  }
  
  // Texture oluştur
  createTexture(descriptor) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    const texture = this.device.createTexture(descriptor);
    const textureId = this.generateId();
    this.textures.set(textureId, texture);
    
    return { texture, textureId };
  }
  
  // 2D texture oluştur
  createTexture2D(width, height, format = 'rgba8unorm', usage = GPUTextureUsage.TEXTURE_BINDING | GPUTextureUsage.COPY_DST) {
    return this.createTexture({
      size: { width, height },
      format,
      usage
    });
  }
  
  // Texture'ı güncelle
  updateTexture(textureId, data, width, height) {
    const texture = this.textures.get(textureId);
    if (!texture) return false;
    
    this.device.queue.writeTexture(
      { texture },
      data,
      { bytesPerRow: width * 4 },
      { width, height }
    );
    
    return true;
  }
  
  // Bind group oluştur
  createBindGroup(descriptor) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    const bindGroup = this.device.createBindGroup(descriptor);
    const bindGroupId = this.generateId();
    this.bindGroups.set(bindGroupId, bindGroup);
    
    return { bindGroup, bindGroupId };
  }
  
  // Command encoder oluştur
  createCommandEncoder() {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    return this.device.createCommandEncoder();
  }
  
  // Render pass oluştur
  beginRenderPass(commandEncoder, colorAttachments, depthStencilAttachment = null) {
    const renderPassDescriptor = {
      colorAttachments
    };
    
    if (depthStencilAttachment) {
      renderPassDescriptor.depthStencilAttachment = depthStencilAttachment;
    }
    
    return commandEncoder.beginRenderPass(renderPassDescriptor);
  }
  
  // Compute pass oluştur
  beginComputePass(commandEncoder) {
    return commandEncoder.beginComputePass();
  }
  
  // Buffer'ı güncelle
  updateBuffer(bufferId, data, offset = 0) {
    const buffer = this.buffers.get(bufferId);
    if (!buffer) return false;
    
    this.device.queue.writeBuffer(buffer, offset, data);
    return true;
  }
  
  // Pipeline kullan
  setPipeline(renderPass, pipelineId) {
    const pipeline = this.pipelines.get(pipelineId);
    if (pipeline) {
      renderPass.setPipeline(pipeline);
      return true;
    }
    return false;
  }
  
  // Bind group ayarla
  setBindGroup(renderPass, index, bindGroupId) {
    const bindGroup = this.bindGroups.get(bindGroupId);
    if (bindGroup) {
      renderPass.setBindGroup(index, bindGroup);
      return true;
    }
    return false;
  }
  
  // Vertex buffer ayarla
  setVertexBuffer(renderPass, slot, bufferId) {
    const buffer = this.buffers.get(bufferId);
    if (buffer) {
      renderPass.setVertexBuffer(slot, buffer);
      return true;
    }
    return false;
  }
  
  // Index buffer ayarla
  setIndexBuffer(renderPass, bufferId) {
    const buffer = this.buffers.get(bufferId);
    if (buffer) {
      renderPass.setIndexBuffer(buffer, 'uint16');
      return true;
    }
    return false;
  }
  
  // Draw call
  draw(renderPass, vertexCount, instanceCount = 1, firstVertex = 0, firstInstance = 0) {
    renderPass.draw(vertexCount, instanceCount, firstVertex, firstInstance);
  }
  
  // Indexed draw call
  drawIndexed(renderPass, indexCount, instanceCount = 1, firstIndex = 0, baseVertex = 0, firstInstance = 0) {
    renderPass.drawIndexed(indexCount, instanceCount, firstIndex, baseVertex, firstInstance);
  }
  
  // Compute dispatch
  dispatch(computePass, workgroupCountX, workgroupCountY = 1, workgroupCountZ = 1) {
    computePass.dispatchWorkgroups(workgroupCountX, workgroupCountY, workgroupCountZ);
  }
  
  // Command'ı submit et
  submit(commandEncoder) {
    if (!this.isInitialized) {
      throw new Error('WebGPU başlatılmamış');
    }
    
    this.device.queue.submit([commandEncoder.finish()]);
  }
  
  // Canvas texture'ını al
  getCurrentTexture() {
    if (!this.context) {
      throw new Error('WebGPU context bulunamadı');
    }
    
    return this.context.getCurrentTexture();
  }
  
  // ID oluştur
  generateId() {
    return 'webgpu_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  // Kaynakları temizle
  cleanup() {
    // Pipeline'ları temizle
    this.pipelines.clear();
    
    // Buffer'ları temizle
    this.buffers.forEach(buffer => {
      buffer.destroy();
    });
    this.buffers.clear();
    
    // Texture'ları temizle
    this.textures.forEach(texture => {
      texture.destroy();
    });
    this.textures.clear();
    
    // Bind group'ları temizle
    this.bindGroups.clear();
  }
  
  // WebGPU bilgilerini al
  getInfo() {
    if (!this.adapter) return null;
    
    return {
      vendor: this.adapter.info?.vendor || 'Unknown',
      architecture: this.adapter.info?.architecture || 'Unknown',
      device: this.adapter.info?.device || 'Unknown',
      description: this.adapter.info?.description || 'Unknown',
      limits: this.adapter.limits
    };
  }
}
```

**Alternatifler:**
- **WebGL**: WebGL API
- **WebGL 2.0**: WebGL 2.0
- **WebAssembly**: WebAssembly
- **Web Workers**: Web Workers
- **Canvas 2D**: Canvas 2D API

**Ne Zaman Kullanılmalı?**
- ✅ Yüksek performans gerektiğinde
- ✅ GPU hesaplama gerektiğinde
- ✅ Modern grafikler gerektiğinde
- ✅ Paralel işleme gerektiğinde
- ✅ Donanım hızlandırması gerektiğinde
- ✅ Platform bağımsızlığı gerektiğinde
- ✅ Gelecek güvenli uygulamalar gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Basit 2D grafikler yeterli olduğunda
- ❌ Düşük güç tüketimi gerektiğinde
- ❌ Basit uygulamalar gerektiğinde

**Kullanılmazsa Ne Olur?**
- **No High Performance**: Yüksek performans olmaz
- **No GPU Computing**: GPU hesaplama olmaz
- **No Modern Graphics**: Modern grafikler olmaz
- **No Parallel Processing**: Paralel işleme olmaz
- **Limited Performance**: Performans sınırlı olur

**Modern Alternatifler:**
- **WebGPU**: WebGPU API
- **WebGL 2.0**: WebGL 2.0
- **WebAssembly**: WebAssembly
- **Web Workers**: Web Workers
- **Canvas 2D**: Canvas 2D API

### Web Locks API

**Web Locks API Nedir?**
Web Locks API, web uygulamalarının kaynakları kilitlemesini ve senkronize etmesini sağlayan bir web API'sidir. Çoklu sekme, çoklu pencere veya çoklu worker ortamlarında aynı kaynağa erişimi kontrol etmek için kullanılır. Race condition'ları önler, veri tutarlılığını sağlar ve kaynak çakışmalarını engeller. Özellikle IndexedDB, localStorage veya diğer paylaşılan kaynaklarla çalışırken kritik öneme sahiptir.

**Neden Kullanılır?**
- **Resource Synchronization**: Kaynak senkronizasyonu
- **Race Condition Prevention**: Race condition önleme
- **Data Consistency**: Veri tutarlılığı
- **Multi-tab Coordination**: Çoklu sekme koordinasyonu
- **Critical Section Protection**: Kritik bölge koruması
- **Concurrent Access Control**: Eşzamanlı erişim kontrolü
- **Deadlock Prevention**: Deadlock önleme

**Temel Kullanım Örnekleri:**

```javascript
// Basit kaynak kilitleme
if ('locks' in navigator) {
  await navigator.locks.request('my-resource', async lock => {
    console.log('Kaynak kilitlendi');
    // Kilitli kaynakla işlem yap
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('Kaynak serbest bırakıldı');
  });
}

// Koşullu kilitleme
await navigator.locks.request('shared-data', { ifAvailable: true }, async lock => {
  if (lock) {
    console.log('Kilit alındı');
    // İşlem yap
  } else {
    console.log('Kilit alınamadı, başka bir işlem devam ediyor');
  }
});

// Timeout ile kilitleme
await navigator.locks.request('timeout-resource', { 
  ifAvailable: false,
  steal: false 
}, async lock => {
  console.log('Kilit alındı');
  // İşlem yap
});

// Kilit durumu sorgulama
const lockInfo = await navigator.locks.query();
console.log('Aktif kilitler:', lockInfo.held);
console.log('Bekleyen kilitler:', lockInfo.pending);

// Web Locks yöneticisi
class WebLocksManager {
  constructor() {
    this.isSupported = 'locks' in navigator;
    this.activeLocks = new Map();
    this.lockQueue = new Map();
    this.lockTimeouts = new Map();
    this.defaultTimeout = 5000; // 5 saniye
    this.maxRetries = 3;
  }
  
  // Kilit al
  async acquireLock(resourceName, options = {}) {
    if (!this.isSupported) {
      throw new Error('Web Locks API desteklenmiyor');
    }
    
    const {
      mode = 'exclusive',
      ifAvailable = false,
      steal = false,
      timeout = this.defaultTimeout,
      retries = this.maxRetries
    } = options;
    
    const lockOptions = {
      mode,
      ifAvailable,
      steal
    };
    
    try {
      const lock = await navigator.locks.request(resourceName, lockOptions, async (acquiredLock) => {
        if (!acquiredLock) {
          throw new Error('Kilit alınamadı');
        }
        
        // Kilit bilgilerini kaydet
        this.activeLocks.set(resourceName, {
          lock: acquiredLock,
          timestamp: Date.now(),
          mode,
          timeout
        });
        
        // Timeout ayarla
        const timeoutId = setTimeout(() => {
          this.releaseLock(resourceName);
        }, timeout);
        
        this.lockTimeouts.set(resourceName, timeoutId);
        
        return acquiredLock;
      });
      
      return lock;
    } catch (error) {
      if (retries > 0) {
        console.log(`Kilit alınamadı, ${retries} deneme kaldı`);
        await this.delay(1000);
        return this.acquireLock(resourceName, { ...options, retries: retries - 1 });
      }
      throw error;
    }
  }
  
  // Paylaşımlı kilit al
  async acquireSharedLock(resourceName, options = {}) {
    return this.acquireLock(resourceName, { ...options, mode: 'shared' });
  }
  
  // Özel kilit al
  async acquireExclusiveLock(resourceName, options = {}) {
    return this.acquireLock(resourceName, { ...options, mode: 'exclusive' });
  }
  
  // Kilit serbest bırak
  async releaseLock(resourceName) {
    const lockInfo = this.activeLocks.get(resourceName);
    if (lockInfo) {
      // Timeout'u temizle
      const timeoutId = this.lockTimeouts.get(resourceName);
      if (timeoutId) {
        clearTimeout(timeoutId);
        this.lockTimeouts.delete(resourceName);
      }
      
      // Kilit bilgilerini temizle
      this.activeLocks.delete(resourceName);
      
      console.log(`Kilit serbest bırakıldı: ${resourceName}`);
    }
  }
  
  // Tüm kilitleri serbest bırak
  async releaseAllLocks() {
    const lockNames = Array.from(this.activeLocks.keys());
    
    for (const resourceName of lockNames) {
      await this.releaseLock(resourceName);
    }
  }
  
  // Kilit durumu kontrol et
  isLocked(resourceName) {
    return this.activeLocks.has(resourceName);
  }
  
  // Kilit bilgilerini al
  getLockInfo(resourceName) {
    return this.activeLocks.get(resourceName);
  }
  
  // Tüm kilit bilgilerini al
  getAllLockInfo() {
    return Array.from(this.activeLocks.entries()).map(([name, info]) => ({
      resourceName: name,
      mode: info.mode,
      timestamp: info.timestamp,
      duration: Date.now() - info.timestamp
    }));
  }
  
  // Kilit durumu sorgula
  async queryLocks() {
    if (!this.isSupported) {
      return { held: [], pending: [] };
    }
    
    return await navigator.locks.query();
  }
  
  // Kilit istatistikleri
  getLockStatistics() {
    const locks = this.getAllLockInfo();
    const now = Date.now();
    
    return {
      totalLocks: locks.length,
      exclusiveLocks: locks.filter(l => l.mode === 'exclusive').length,
      sharedLocks: locks.filter(l => l.mode === 'shared').length,
      averageDuration: locks.reduce((sum, l) => sum + l.duration, 0) / locks.length || 0,
      longestDuration: Math.max(...locks.map(l => l.duration), 0)
    };
  }
  
  // Kilit temizleme
  cleanup() {
    // Tüm timeout'ları temizle
    this.lockTimeouts.forEach(timeoutId => clearTimeout(timeoutId));
    this.lockTimeouts.clear();
    
    // Tüm kilitleri temizle
    this.activeLocks.clear();
    this.lockQueue.clear();
  }
  
  // Gecikme
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

**Alternatifler:**
- **Mutex**: Mutex benzeri yapılar
- **Semaphore**: Semaphore benzeri yapılar
- **Atomic Operations**: Atomik işlemler
- **Database Locks**: Veritabanı kilitleri
- **File System Locks**: Dosya sistemi kilitleri

**Ne Zaman Kullanılmalı?**
- ✅ Çoklu sekme/pencere uygulamalarında
- ✅ Paylaşılan kaynaklarla çalışırken
- ✅ Race condition'ları önlemek için
- ✅ Veri tutarlılığı gerektiğinde
- ✅ Kritik bölge koruması gerektiğinde
- ✅ Eşzamanlı erişim kontrolü gerektiğinde
- ✅ Deadlock önleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Tek sekme uygulamalarında
- ❌ Basit işlemler için
- ❌ Performans kritik uygulamalarda
- ❌ Kısa süreli işlemler için

**Kullanılmazsa Ne Olur?**
- **Race Conditions**: Race condition'lar oluşur
- **Data Inconsistency**: Veri tutarsızlığı olur
- **Resource Conflicts**: Kaynak çakışmaları olur
- **Concurrent Access Issues**: Eşzamanlı erişim sorunları olur
- **Data Corruption**: Veri bozulması olur

**Modern Alternatifler:**
- **Web Locks API**: Web Locks API
- **IndexedDB Transactions**: IndexedDB işlemleri
- **SharedArrayBuffer**: SharedArrayBuffer
- **Web Workers**: Web Workers
- **Service Workers**: Service Workers

### Web Messaging API

**Web Messaging API Nedir?**
Web Messaging API, farklı origin'ler arasında güvenli mesajlaşma sağlayan bir web API'sidir. PostMessage, MessageChannel ve MessagePort gibi mekanizmalar kullanarak iframe'ler, popup'lar, web worker'lar ve farklı pencereler arasında veri alışverişi yapılmasını sağlar. Same-origin policy'yi aşarak güvenli cross-origin iletişim kurar. Modern web uygulamalarında mikro-frontend'ler, widget'lar ve modüler yapılar için kritik öneme sahiptir.

**Neden Kullanılır?**
- **Cross-Origin Communication**: Cross-origin iletişim
- **Secure Messaging**: Güvenli mesajlaşma
- **Iframe Communication**: Iframe iletişimi
- **Worker Communication**: Worker iletişimi
- **Window Communication**: Pencere iletişimi
- **Modular Architecture**: Modüler mimari
- **Micro-Frontend Support**: Mikro-frontend desteği

**Temel Kullanım Örnekleri:**

```javascript
// PostMessage - Iframe iletişimi
const iframe = document.getElementById('myIframe');
iframe.contentWindow.postMessage('Merhaba iframe!', '*');

// PostMessage - Popup iletişimi
const popup = window.open('popup.html', 'popup');
popup.postMessage({ type: 'INIT', data: 'Başlangıç verisi' }, '*');

// PostMessage - Worker iletişimi
const worker = new Worker('worker.js');
worker.postMessage({ command: 'START', data: 'İşlem verisi' });

// MessageChannel - İki yönlü iletişim
const channel = new MessageChannel();
const port1 = channel.port1;
const port2 = channel.port2;

port1.onmessage = event => {
  console.log('Port1\'den gelen:', event.data);
};

port2.onmessage = event => {
  console.log('Port2\'den gelen:', event.data);
};

// Port'ları başlat
port1.start();
port2.start();

// Mesaj gönder
port1.postMessage('Port1\'den Port2\'ye mesaj');
port2.postMessage('Port2\'den Port1\'e mesaj');

// Mesaj dinleme
window.addEventListener('message', event => {
  console.log('Gelen mesaj:', event.data);
  console.log('Origin:', event.origin);
  console.log('Source:', event.source);
});

// Web Messaging yöneticisi
class WebMessagingManager {
  constructor() {
    this.isSupported = 'postMessage' in window;
    this.messageHandlers = new Map();
    this.channels = new Map();
    this.ports = new Map();
    this.messageQueue = [];
    this.isListening = false;
    this.defaultTimeout = 5000;
    this.messageId = 0;
  }
  
  // Mesaj dinleyici başlat
  startListening() {
    if (this.isListening) return;
    
    window.addEventListener('message', this.handleMessage.bind(this));
    this.isListening = true;
  }
  
  // Mesaj dinleyici durdur
  stopListening() {
    if (!this.isListening) return;
    
    window.removeEventListener('message', this.handleMessage.bind(this));
    this.isListening = false;
  }
  
  // Mesaj işleyici
  handleMessage(event) {
    const { data, origin, source } = event;
    
    // Güvenlik kontrolü
    if (!this.isValidOrigin(origin)) {
      console.warn('Geçersiz origin:', origin);
      return;
    }
    
    // Mesaj tipine göre işle
    if (data && typeof data === 'object') {
      if (data.type === 'REQUEST') {
        this.handleRequest(data, origin, source);
      } else if (data.type === 'RESPONSE') {
        this.handleResponse(data, origin, source);
      } else if (data.type === 'EVENT') {
        this.handleEvent(data, origin, source);
      }
    }
    
    // Genel mesaj işleyicileri
    this.messageHandlers.forEach(handler => {
      try {
        handler(data, origin, source);
      } catch (error) {
        console.error('Mesaj işleyici hatası:', error);
      }
    });
  }
  
  // İstek işleyici
  async handleRequest(data, origin, source) {
    const { id, method, params } = data;
    
    try {
      // İstek işleyicisini bul
      const handler = this.messageHandlers.get(method);
      if (!handler) {
        throw new Error(`İstek işleyicisi bulunamadı: ${method}`);
      }
      
      // İsteği işle
      const result = await handler(params, origin, source);
      
      // Yanıt gönder
      this.sendResponse(source, origin, id, result);
    } catch (error) {
      // Hata yanıtı gönder
      this.sendError(source, origin, id, error.message);
    }
  }
  
  // Yanıt işleyici
  handleResponse(data, origin, source) {
    const { id, result, error } = data;
    
    // Bekleyen istekleri bul
    const pendingRequest = this.messageQueue.find(req => req.id === id);
    if (pendingRequest) {
      if (error) {
        pendingRequest.reject(new Error(error));
      } else {
        pendingRequest.resolve(result);
      }
      
      // İsteği kuyruktan çıkar
      const index = this.messageQueue.indexOf(pendingRequest);
      if (index > -1) {
        this.messageQueue.splice(index, 1);
      }
    }
  }
  
  // Olay işleyici
  handleEvent(data, origin, source) {
    const { event, payload } = data;
    
    // Olay işleyicilerini çağır
    const eventHandlers = this.messageHandlers.get(`event:${event}`);
    if (eventHandlers) {
      eventHandlers.forEach(handler => {
        try {
          handler(payload, origin, source);
        } catch (error) {
          console.error('Olay işleyici hatası:', error);
        }
      });
    }
  }
  
  // Mesaj gönder
  sendMessage(target, message, origin = '*') {
    if (!this.isSupported) {
      throw new Error('PostMessage desteklenmiyor');
    }
    
    target.postMessage(message, origin);
  }
  
  // İstek gönder
  async sendRequest(target, method, params = {}, origin = '*', timeout = this.defaultTimeout) {
    return new Promise((resolve, reject) => {
      const id = this.generateMessageId();
      
      // İstek oluştur
      const request = {
        type: 'REQUEST',
        id,
        method,
        params
      };
      
      // Bekleyen isteği kaydet
      const pendingRequest = {
        id,
        resolve,
        reject,
        timeout: setTimeout(() => {
          reject(new Error('İstek zaman aşımına uğradı'));
        }, timeout)
      };
      
      this.messageQueue.push(pendingRequest);
      
      // İsteği gönder
      this.sendMessage(target, request, origin);
    });
  }
  
  // Yanıt gönder
  sendResponse(target, origin, id, result) {
    const response = {
      type: 'RESPONSE',
      id,
      result
    };
    
    this.sendMessage(target, response, origin);
  }
  
  // Hata yanıtı gönder
  sendError(target, origin, id, error) {
    const response = {
      type: 'RESPONSE',
      id,
      error
    };
    
    this.sendMessage(target, response, origin);
  }
  
  // Olay gönder
  sendEvent(target, event, payload, origin = '*') {
    const message = {
      type: 'EVENT',
      event,
      payload
    };
    
    this.sendMessage(target, message, origin);
  }
  
  // Mesaj işleyici kaydet
  registerHandler(method, handler) {
    this.messageHandlers.set(method, handler);
  }
  
  // Olay işleyici kaydet
  registerEventHandler(event, handler) {
    const eventKey = `event:${event}`;
    if (!this.messageHandlers.has(eventKey)) {
      this.messageHandlers.set(eventKey, []);
    }
    this.messageHandlers.get(eventKey).push(handler);
  }
  
  // Mesaj işleyici kaldır
  unregisterHandler(method) {
    this.messageHandlers.delete(method);
  }
  
  // Olay işleyici kaldır
  unregisterEventHandler(event, handler) {
    const eventKey = `event:${event}`;
    const handlers = this.messageHandlers.get(eventKey);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index > -1) {
        handlers.splice(index, 1);
      }
    }
  }
  
  // MessageChannel oluştur
  createChannel(name) {
    const channel = new MessageChannel();
    const port1 = channel.port1;
    const port2 = channel.port2;
    
    // Port'ları başlat
    port1.start();
    port2.start();
    
    // Kanalı kaydet
    this.channels.set(name, {
      channel,
      port1,
      port2
    });
    
    return { port1, port2 };
  }
  
  // Kanal al
  getChannel(name) {
    return this.channels.get(name);
  }
  
  // Kanal kaldır
  removeChannel(name) {
    const channel = this.channels.get(name);
    if (channel) {
      channel.port1.close();
      channel.port2.close();
      this.channels.delete(name);
    }
  }
  
  // Port oluştur
  createPort(name) {
    const port = new MessagePort();
    this.ports.set(name, port);
    return port;
  }
  
  // Port al
  getPort(name) {
    return this.ports.get(name);
  }
  
  // Port kaldır
  removePort(name) {
    const port = this.ports.get(name);
    if (port) {
      port.close();
      this.ports.delete(name);
    }
  }
  
  // Origin doğrulama
  isValidOrigin(origin) {
    // Güvenlik için origin kontrolü
    const allowedOrigins = ['https://example.com', 'https://localhost:3000'];
    return allowedOrigins.includes(origin) || origin === '*';
  }
  
  // Mesaj ID oluştur
  generateMessageId() {
    return `msg_${++this.messageId}_${Date.now()}`;
  }
  
  // Bekleyen istekleri temizle
  clearPendingRequests() {
    this.messageQueue.forEach(request => {
      clearTimeout(request.timeout);
    });
    this.messageQueue = [];
  }
  
  // Kaynakları temizle
  cleanup() {
    this.stopListening();
    this.clearPendingRequests();
    
    // Kanalları temizle
    this.channels.forEach(channel => {
      channel.port1.close();
      channel.port2.close();
    });
    this.channels.clear();
    
    // Port'ları temizle
    this.ports.forEach(port => {
      port.close();
    });
    this.ports.clear();
    
    // İşleyicileri temizle
    this.messageHandlers.clear();
  }
  
  // İstatistikler
  getStatistics() {
    return {
      activeChannels: this.channels.size,
      activePorts: this.ports.size,
      registeredHandlers: this.messageHandlers.size,
      pendingRequests: this.messageQueue.length,
      isListening: this.isListening
    };
  }
}

// Mesajlaşma utility fonksiyonları
class WebMessagingUtils {
  // Güvenli mesaj gönder
  static sendSecureMessage(target, message, allowedOrigins = []) {
    if (!target || !target.postMessage) {
      throw new Error('Geçersiz hedef');
    }
    
    // Origin kontrolü
    const origin = allowedOrigins.length > 0 ? allowedOrigins[0] : '*';
    
    // Mesajı gönder
    target.postMessage(message, origin);
  }
  
  // Mesaj doğrulama
  static validateMessage(message) {
    if (!message || typeof message !== 'object') {
      return { valid: false, error: 'Mesaj geçersiz' };
    }
    
    if (!message.type) {
      return { valid: false, error: 'Mesaj tipi eksik' };
    }
    
    const validTypes = ['REQUEST', 'RESPONSE', 'EVENT'];
    if (!validTypes.includes(message.type)) {
      return { valid: false, error: 'Geçersiz mesaj tipi' };
    }
    
    return { valid: true };
  }
  
  // Origin güvenlik kontrolü
  static isSecureOrigin(origin) {
    if (origin === '*') return false;
    if (origin.startsWith('https://')) return true;
    if (origin.startsWith('http://localhost')) return true;
    if (origin.startsWith('http://127.0.0.1')) return true;
    
    return false;
  }
  
  // Mesaj şifreleme
  static encryptMessage(message, key) {
    // Basit şifreleme (gerçek uygulamada daha güvenli yöntemler kullanın)
    const messageStr = JSON.stringify(message);
    const encrypted = btoa(messageStr);
    return encrypted;
  }
  
  // Mesaj şifre çözme
  static decryptMessage(encryptedMessage, key) {
    try {
      const decrypted = atob(encryptedMessage);
      return JSON.parse(decrypted);
    } catch (error) {
      throw new Error('Mesaj şifre çözülemedi');
    }
  }
  
  // Mesaj boyutu kontrolü
  static checkMessageSize(message) {
    const messageStr = JSON.stringify(message);
    const size = new Blob([messageStr]).size;
    
    return {
      size,
      sizeKB: Math.round(size / 1024 * 100) / 100,
      isTooLarge: size > 1024 * 1024 // 1MB limit
    };
  }
  
  // Mesaj sıkıştırma
  static compressMessage(message) {
    const messageStr = JSON.stringify(message);
    // Basit sıkıştırma (gerçek uygulamada daha gelişmiş yöntemler kullanın)
    return btoa(messageStr);
  }
  
  // Mesaj açma
  static decompressMessage(compressedMessage) {
    try {
      const decompressed = atob(compressedMessage);
      return JSON.parse(decompressed);
    } catch (error) {
      throw new Error('Mesaj açılamadı');
    }
  }
}
```

**Alternatifler:**
- **WebSocket**: WebSocket API
- **Server-Sent Events**: Server-Sent Events
- **Broadcast Channel**: Broadcast Channel API
- **SharedArrayBuffer**: SharedArrayBuffer
- **Web Workers**: Web Workers

**Ne Zaman Kullanılmalı?**
- ✅ Cross-origin iletişim gerektiğinde
- ✅ Iframe iletişimi gerektiğinde
- ✅ Worker iletişimi gerektiğinde
- ✅ Pencere iletişimi gerektiğinde
- ✅ Modüler mimari gerektiğinde
- ✅ Mikro-frontend gerektiğinde
- ✅ Güvenli mesajlaşma gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Aynı origin içinde iletişim gerektiğinde
- ❌ Gerçek zamanlı iletişim gerektiğinde
- ❌ Büyük veri transferi gerektiğinde
- ❌ Basit uygulamalar için

**Kullanılmazsa Ne Olur?**
- **No Cross-Origin Communication**: Cross-origin iletişim olmaz
- **No Secure Messaging**: Güvenli mesajlaşma olmaz
- **No Iframe Communication**: Iframe iletişimi olmaz
- **No Worker Communication**: Worker iletişimi olmaz
- **Limited Modularity**: Modülerlik sınırlı olur

**Modern Alternatifler:**
- **Web Messaging API**: Web Messaging API
- **WebSocket**: WebSocket API
- **Server-Sent Events**: Server-Sent Events
- **Broadcast Channel**: Broadcast Channel API
- **SharedArrayBuffer**: SharedArrayBuffer

### Web MIDI API

**Web MIDI API Nedir?**
Web MIDI API, web uygulamalarının MIDI (Musical Instrument Digital Interface) cihazlarıyla iletişim kurmasını sağlayan bir web API'sidir. Klavye, synthesizer, drum machine ve diğer müzik enstrümanlarını web tarayıcısından kontrol etmeyi mümkün kılar. Gerçek zamanlı müzik üretimi, ses sentezi, müzik eğitimi ve performans uygulamaları için kritik öneme sahiptir. Web Audio API ile birlikte kullanılarak güçlü müzik uygulamaları oluşturulabilir.

**Neden Kullanılır?**
- **Music Production**: Müzik üretimi
- **Real-time Control**: Gerçek zamanlı kontrol
- **Hardware Integration**: Donanım entegrasyonu
- **Music Education**: Müzik eğitimi
- **Performance Applications**: Performans uygulamaları
- **Audio Synthesis**: Ses sentezi
- **Creative Expression**: Yaratıcı ifade

**Temel Kullanım Örnekleri:**

```javascript
// MIDI erişimi
if ('requestMIDIAccess' in navigator) {
  const midiAccess = await navigator.requestMIDIAccess();
  
  // Input cihazları
  for (const input of midiAccess.inputs.values()) {
    input.onmidimessage = event => {
      console.log('MIDI mesajı:', event.data);
      console.log('Cihaz:', input.name);
      console.log('Zaman damgası:', event.timeStamp);
    };
  }
  
  // Output cihazları
  for (const output of midiAccess.outputs.values()) {
    console.log('Output cihazı:', output.name);
    
    // Nota gönder
    output.send([0x90, 60, 127]); // C4 notası, velocity 127
    setTimeout(() => {
      output.send([0x80, 60, 0]); // Notayı durdur
    }, 1000);
  }
}

// MIDI mesaj türleri
const midiMessages = {
  NOTE_ON: 0x90,
  NOTE_OFF: 0x80,
  CONTROL_CHANGE: 0xB0,
  PROGRAM_CHANGE: 0xC0,
  PITCH_BEND: 0xE0
};

// Web MIDI yöneticisi
class WebMIDIManager {
  constructor() {
    this.isSupported = 'requestMIDIAccess' in navigator;
    this.midiAccess = null;
    this.inputs = new Map();
    this.outputs = new Map();
    this.messageHandlers = new Map();
    this.isInitialized = false;
    this.connectionState = 'disconnected';
  }
  
  // MIDI başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web MIDI API desteklenmiyor');
    }
    
    try {
      this.midiAccess = await navigator.requestMIDIAccess();
      this.connectionState = 'connected';
      
      // Input cihazlarını kaydet
      for (const [id, input] of this.midiAccess.inputs) {
        this.inputs.set(id, input);
        this.setupInputHandlers(input);
      }
      
      // Output cihazlarını kaydet
      for (const [id, output] of this.midiAccess.outputs) {
        this.outputs.set(id, output);
      }
      
      // Cihaz değişikliklerini dinle
      this.midiAccess.onstatechange = this.handleStateChange.bind(this);
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('MIDI başlatma hatası:', error);
      return false;
    }
  }
  
  // Input işleyicilerini ayarla
  setupInputHandlers(input) {
    input.onmidimessage = (event) => {
      this.handleMIDIMessage(event, input);
    };
  }
  
  // MIDI mesaj işleyici
  handleMIDIMessage(event, input) {
    const { data, timeStamp } = event;
    const messageType = data[0] & 0xF0;
    const channel = data[0] & 0x0F;
    
    const message = {
      type: messageType,
      channel: channel,
      data: data,
      timeStamp: timeStamp,
      device: input.name,
      deviceId: input.id
    };
    
    // Mesaj tipine göre işle
    switch (messageType) {
      case 0x90: // Note On
        this.handleNoteOn(message);
        break;
      case 0x80: // Note Off
        this.handleNoteOff(message);
        break;
      case 0xB0: // Control Change
        this.handleControlChange(message);
        break;
      case 0xC0: // Program Change
        this.handleProgramChange(message);
        break;
      case 0xE0: // Pitch Bend
        this.handlePitchBend(message);
        break;
    }
    
    // Genel mesaj işleyicileri
    this.messageHandlers.forEach(handler => {
      try {
        handler(message);
      } catch (error) {
        console.error('MIDI mesaj işleyici hatası:', error);
      }
    });
  }
  
  // Note On işleyici
  handleNoteOn(message) {
    const note = message.data[1];
    const velocity = message.data[2];
    
    const noteInfo = {
      note: note,
      velocity: velocity,
      frequency: this.midiToFrequency(note),
      noteName: this.midiToNoteName(note),
      octave: Math.floor(note / 12) - 1
    };
    
    console.log('Note On:', noteInfo);
    
    // Note On işleyicilerini çağır
    const handlers = this.messageHandlers.get('noteOn');
    if (handlers) {
      handlers.forEach(handler => handler(noteInfo, message));
    }
  }
  
  // Note Off işleyici
  handleNoteOff(message) {
    const note = message.data[1];
    const velocity = message.data[2];
    
    const noteInfo = {
      note: note,
      velocity: velocity,
      frequency: this.midiToFrequency(note),
      noteName: this.midiToNoteName(note),
      octave: Math.floor(note / 12) - 1
    };
    
    console.log('Note Off:', noteInfo);
    
    // Note Off işleyicilerini çağır
    const handlers = this.messageHandlers.get('noteOff');
    if (handlers) {
      handlers.forEach(handler => handler(noteInfo, message));
    }
  }
  
  // Control Change işleyici
  handleControlChange(message) {
    const controller = message.data[1];
    const value = message.data[2];
    
    const controlInfo = {
      controller: controller,
      value: value,
      normalizedValue: value / 127
    };
    
    console.log('Control Change:', controlInfo);
    
    // Control Change işleyicilerini çağır
    const handlers = this.messageHandlers.get('controlChange');
    if (handlers) {
      handlers.forEach(handler => handler(controlInfo, message));
    }
  }
  
  // Program Change işleyici
  handleProgramChange(message) {
    const program = message.data[1];
    
    console.log('Program Change:', program);
    
    // Program Change işleyicilerini çağır
    const handlers = this.messageHandlers.get('programChange');
    if (handlers) {
      handlers.forEach(handler => handler(program, message));
    }
  }
  
  // Pitch Bend işleyici
  handlePitchBend(message) {
    const lsb = message.data[1];
    const msb = message.data[2];
    const value = (msb << 7) | lsb;
    const normalizedValue = (value - 8192) / 8192;
    
    const pitchBendInfo = {
      value: value,
      normalizedValue: normalizedValue,
      cents: normalizedValue * 200
    };
    
    console.log('Pitch Bend:', pitchBendInfo);
    
    // Pitch Bend işleyicilerini çağır
    const handlers = this.messageHandlers.get('pitchBend');
    if (handlers) {
      handlers.forEach(handler => handler(pitchBendInfo, message));
    }
  }
  
  // Cihaz durumu değişikliği
  handleStateChange(event) {
    const { port } = event;
    
    if (port.type === 'input') {
      if (port.state === 'connected') {
        this.inputs.set(port.id, port);
        this.setupInputHandlers(port);
        console.log('MIDI input bağlandı:', port.name);
      } else {
        this.inputs.delete(port.id);
        console.log('MIDI input bağlantısı kesildi:', port.name);
      }
    } else if (port.type === 'output') {
      if (port.state === 'connected') {
        this.outputs.set(port.id, port);
        console.log('MIDI output bağlandı:', port.name);
      } else {
        this.outputs.delete(port.id);
        console.log('MIDI output bağlantısı kesildi:', port.name);
      }
    }
  }
  
  // Nota gönder
  sendNote(outputId, note, velocity = 127, duration = 1000) {
    const output = this.outputs.get(outputId);
    if (!output) {
      throw new Error('Output cihazı bulunamadı');
    }
    
    // Note On
    output.send([0x90, note, velocity]);
    
    // Note Off
    setTimeout(() => {
      output.send([0x80, note, 0]);
    }, duration);
  }
  
  // Akor gönder
  sendChord(outputId, notes, velocity = 127, duration = 1000) {
    const output = this.outputs.get(outputId);
    if (!output) {
      throw new Error('Output cihazı bulunamadı');
    }
    
    // Tüm notaları başlat
    notes.forEach(note => {
      output.send([0x90, note, velocity]);
    });
    
    // Tüm notaları durdur
    setTimeout(() => {
      notes.forEach(note => {
        output.send([0x80, note, 0]);
      });
    }, duration);
  }
  
  // Control Change gönder
  sendControlChange(outputId, controller, value) {
    const output = this.outputs.get(outputId);
    if (!output) {
      throw new Error('Output cihazı bulunamadı');
    }
    
    output.send([0xB0, controller, value]);
  }
  
  // Program Change gönder
  sendProgramChange(outputId, program) {
    const output = this.outputs.get(outputId);
    if (!output) {
      throw new Error('Output cihazı bulunamadı');
    }
    
    output.send([0xC0, program]);
  }
  
  // Pitch Bend gönder
  sendPitchBend(outputId, value) {
    const output = this.outputs.get(outputId);
    if (!output) {
      throw new Error('Output cihazı bulunamadı');
    }
    
    const lsb = value & 0x7F;
    const msb = (value >> 7) & 0x7F;
    
    output.send([0xE0, lsb, msb]);
  }
  
  // Mesaj işleyici kaydet
  registerHandler(type, handler) {
    if (!this.messageHandlers.has(type)) {
      this.messageHandlers.set(type, []);
    }
    this.messageHandlers.get(type).push(handler);
  }
  
  // Mesaj işleyici kaldır
  unregisterHandler(type, handler) {
    const handlers = this.messageHandlers.get(type);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index > -1) {
        handlers.splice(index, 1);
      }
    }
  }
  
  // MIDI notayı frekansa çevir
  midiToFrequency(note) {
    return 440 * Math.pow(2, (note - 69) / 12);
  }
  
  // MIDI notayı nota adına çevir
  midiToNoteName(note) {
    const noteNames = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];
    const octave = Math.floor(note / 12) - 1;
    const noteName = noteNames[note % 12];
    return `${noteName}${octave}`;
  }
  
  // Frekansı MIDI notaya çevir
  frequencyToMIDI(frequency) {
    return Math.round(12 * Math.log2(frequency / 440) + 69);
  }
  
  // Cihaz bilgilerini al
  getDeviceInfo() {
    return {
      inputs: Array.from(this.inputs.values()).map(input => ({
        id: input.id,
        name: input.name,
        manufacturer: input.manufacturer,
        state: input.state,
        connection: input.connection
      })),
      outputs: Array.from(this.outputs.values()).map(output => ({
        id: output.id,
        name: output.name,
        manufacturer: output.manufacturer,
        state: output.state,
        connection: output.connection
      }))
    };
  }
  
  // Bağlantı durumu
  getConnectionState() {
    return this.connectionState;
  }
  
  // Kaynakları temizle
  cleanup() {
    this.messageHandlers.clear();
    this.inputs.clear();
    this.outputs.clear();
    this.midiAccess = null;
    this.isInitialized = false;
    this.connectionState = 'disconnected';
  }
}
```

**Alternatifler:**
- **Web Audio API**: Web Audio API
- **Web Speech API**: Web Speech API
- **Canvas API**: Canvas API
- **WebGL API**: WebGL API
- **WebRTC**: WebRTC

**Ne Zaman Kullanılmalı?**
- ✅ Müzik üretimi uygulamalarında
- ✅ Gerçek zamanlı müzik kontrolü gerektiğinde
- ✅ Donanım entegrasyonu gerektiğinde
- ✅ Müzik eğitimi uygulamalarında
- ✅ Performans uygulamalarında
- ✅ Ses sentezi gerektiğinde
- ✅ Yaratıcı ifade gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit ses çalma gerektiğinde
- ❌ Müzik dışı uygulamalarda
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Donanım erişimi olmayan ortamlarda

**Kullanılmazsa Ne Olur?**
- **No Music Production**: Müzik üretimi olmaz
- **No Real-time Control**: Gerçek zamanlı kontrol olmaz
- **No Hardware Integration**: Donanım entegrasyonu olmaz
- **No Music Education**: Müzik eğitimi olmaz
- **Limited Creative Expression**: Yaratıcı ifade sınırlı olur

**Modern Alternatifler:**
- **Web MIDI API**: Web MIDI API
- **Web Audio API**: Web Audio API
- **Web Speech API**: Web Speech API
- **Canvas API**: Canvas API
- **WebGL API**: WebGL API

### Web NFC API

**Web NFC API Nedir?**
Web NFC API, web uygulamalarının NFC (Near Field Communication) teknolojisiyle etkileşim kurmasını sağlayan bir web API'sidir. NFC etiketlerini okuma, yazma ve NFC cihazları arasında veri alışverişi yapma imkanı sunar. Temassız ödeme, akıllı etiketler, hızlı veri transferi ve IoT cihaz yönetimi için kritik öneme sahiptir. Mobil cihazlarda yaygın olarak kullanılan bu teknoloji, web uygulamalarına da entegre edilmiştir.

**Neden Kullanılır?**
- **Contactless Payments**: Temassız ödemeler
- **Smart Tags**: Akıllı etiketler
- **Quick Data Transfer**: Hızlı veri transferi
- **IoT Device Management**: IoT cihaz yönetimi
- **Access Control**: Erişim kontrolü
- **Information Sharing**: Bilgi paylaşımı
- **Mobile Integration**: Mobil entegrasyon

**Temel Kullanım Örnekleri:**

```javascript
// NFC okuma
if ('NDEFReader' in window) {
  const reader = new NDEFReader();
  
  try {
    await reader.scan();
    reader.addEventListener('reading', event => {
      console.log('NFC etiketi okundu:', event.message);
      console.log('Seri numarası:', event.serialNumber);
    });
  } catch (err) {
    console.error('NFC okuma hatası:', err);
  }
}

// NFC yazma
if ('NDEFWriter' in window) {
  const writer = new NDEFWriter();
  
  const message = {
    records: [
      {
        recordType: 'text',
        data: 'Merhaba NFC!'
      }
    ]
  };
  
  try {
    await writer.write(message);
    console.log('NFC etiketine yazıldı');
  } catch (err) {
    console.error('NFC yazma hatası:', err);
  }
}

// NFC mesaj türleri
const nfcMessageTypes = {
  TEXT: 'text',
  URL: 'url',
  MIME: 'mime',
  ABSOLUTE_URL: 'absolute-url',
  SMART_POSTER: 'smart-poster',
  HANDOVER_SELECT: 'handover-select',
  HANDOVER_REQUEST: 'handover-request'
};

// Web NFC yöneticisi
class WebNFCManager {
  constructor() {
    this.isSupported = 'NDEFReader' in window && 'NDEFWriter' in window;
    this.reader = null;
    this.writer = null;
    this.isScanning = false;
    this.messageHandlers = new Map();
    this.isInitialized = false;
    this.permissions = {
      granted: false,
      denied: false,
      prompt: false
    };
  }
  
  // NFC başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web NFC API desteklenmiyor');
    }
    
    try {
      // İzin kontrolü
      const permission = await navigator.permissions.query({ name: 'nfc' });
      this.permissions.granted = permission.state === 'granted';
      this.permissions.denied = permission.state === 'denied';
      this.permissions.prompt = permission.state === 'prompt';
      
      if (permission.state === 'denied') {
        throw new Error('NFC izni reddedildi');
      }
      
      // Reader ve Writer oluştur
      this.reader = new NDEFReader();
      this.writer = new NDEFWriter();
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('NFC başlatma hatası:', error);
      return false;
    }
  }
  
  // NFC tarama başlat
  async startScanning() {
    if (!this.isInitialized) {
      throw new Error('NFC başlatılmamış');
    }
    
    if (this.isScanning) {
      console.log('NFC tarama zaten aktif');
      return;
    }
    
    try {
      await this.reader.scan();
      this.isScanning = true;
      
      // Okuma olayını dinle
      this.reader.addEventListener('reading', this.handleReading.bind(this));
      this.reader.addEventListener('readingerror', this.handleReadingError.bind(this));
      
      console.log('NFC tarama başlatıldı');
    } catch (error) {
      console.error('NFC tarama başlatma hatası:', error);
      throw error;
    }
  }
  
  // NFC tarama durdur
  stopScanning() {
    if (!this.isScanning) {
      return;
    }
    
    this.isScanning = false;
    console.log('NFC tarama durduruldu');
  }
  
  // Okuma olayı işleyici
  handleReading(event) {
    const { message, serialNumber } = event;
    
    const readingData = {
      message: message,
      serialNumber: serialNumber,
      timestamp: Date.now(),
      records: message.records
    };
    
    console.log('NFC okuma:', readingData);
    
    // Mesaj işleyicilerini çağır
    this.messageHandlers.forEach(handler => {
      try {
        handler(readingData);
      } catch (error) {
        console.error('NFC mesaj işleyici hatası:', error);
      }
    });
    
    // Kayıt türüne göre işle
    message.records.forEach(record => {
      this.handleRecord(record, readingData);
    });
  }
  
  // Okuma hatası işleyici
  handleReadingError(event) {
    console.error('NFC okuma hatası:', event);
  }
  
  // Kayıt işleyici
  handleRecord(record, readingData) {
    const recordType = record.recordType;
    
    switch (recordType) {
      case 'text':
        this.handleTextRecord(record, readingData);
        break;
      case 'url':
        this.handleUrlRecord(record, readingData);
        break;
      case 'mime':
        this.handleMimeRecord(record, readingData);
        break;
      case 'absolute-url':
        this.handleAbsoluteUrlRecord(record, readingData);
        break;
      case 'smart-poster':
        this.handleSmartPosterRecord(record, readingData);
        break;
      default:
        console.log('Bilinmeyen kayıt türü:', recordType);
    }
  }
  
  // Metin kaydı işleyici
  handleTextRecord(record, readingData) {
    const textData = this.decodeTextRecord(record);
    console.log('Metin kaydı:', textData);
    
    // Metin işleyicilerini çağır
    const handlers = this.messageHandlers.get('text');
    if (handlers) {
      handlers.forEach(handler => handler(textData, readingData));
    }
  }
  
  // URL kaydı işleyici
  handleUrlRecord(record, readingData) {
    const urlData = this.decodeUrlRecord(record);
    console.log('URL kaydı:', urlData);
    
    // URL işleyicilerini çağır
    const handlers = this.messageHandlers.get('url');
    if (handlers) {
      handlers.forEach(handler => handler(urlData, readingData));
    }
  }
  
  // MIME kaydı işleyici
  handleMimeRecord(record, readingData) {
    const mimeData = this.decodeMimeRecord(record);
    console.log('MIME kaydı:', mimeData);
    
    // MIME işleyicilerini çağır
    const handlers = this.messageHandlers.get('mime');
    if (handlers) {
      handlers.forEach(handler => handler(mimeData, readingData));
    }
  }
  
  // Mutlak URL kaydı işleyici
  handleAbsoluteUrlRecord(record, readingData) {
    const urlData = this.decodeAbsoluteUrlRecord(record);
    console.log('Mutlak URL kaydı:', urlData);
    
    // Mutlak URL işleyicilerini çağır
    const handlers = this.messageHandlers.get('absolute-url');
    if (handlers) {
      handlers.forEach(handler => handler(urlData, readingData));
    }
  }
  
  // Akıllı poster kaydı işleyici
  handleSmartPosterRecord(record, readingData) {
    const posterData = this.decodeSmartPosterRecord(record);
    console.log('Akıllı poster kaydı:', posterData);
    
    // Akıllı poster işleyicilerini çağır
    const handlers = this.messageHandlers.get('smart-poster');
    if (handlers) {
      handlers.forEach(handler => handler(posterData, readingData));
    }
  }
  
  // Metin kaydını çöz
  decodeTextRecord(record) {
    const decoder = new TextDecoder(record.encoding);
    const text = decoder.decode(record.data);
    
    return {
      text: text,
      encoding: record.encoding,
      language: record.lang
    };
  }
  
  // URL kaydını çöz
  decodeUrlRecord(record) {
    const decoder = new TextDecoder();
    const url = decoder.decode(record.data);
    
    return {
      url: url
    };
  }
  
  // MIME kaydını çöz
  decodeMimeRecord(record) {
    const decoder = new TextDecoder();
    const data = decoder.decode(record.data);
    
    return {
      mimeType: record.mediaType,
      data: data
    };
  }
  
  // Mutlak URL kaydını çöz
  decodeAbsoluteUrlRecord(record) {
    const decoder = new TextDecoder();
    const url = decoder.decode(record.data);
    
    return {
      url: url
    };
  }
  
  // Akıllı poster kaydını çöz
  decodeSmartPosterRecord(record) {
    const decoder = new TextDecoder();
    const data = decoder.decode(record.data);
    
    return {
      data: data,
      records: record.records
    };
  }
  
  // NFC yazma
  async writeMessage(message) {
    if (!this.isInitialized) {
      throw new Error('NFC başlatılmamış');
    }
    
    try {
      await this.writer.write(message);
      console.log('NFC mesajı yazıldı');
      return true;
    } catch (error) {
      console.error('NFC yazma hatası:', error);
      throw error;
    }
  }
  
  // Metin yazma
  async writeText(text, language = 'tr') {
    const message = {
      records: [
        {
          recordType: 'text',
          data: text,
          encoding: 'utf-8',
          lang: language
        }
      ]
    };
    
    return this.writeMessage(message);
  }
  
  // URL yazma
  async writeUrl(url) {
    const message = {
      records: [
        {
          recordType: 'url',
          data: url
        }
      ]
    };
    
    return this.writeMessage(message);
  }
  
  // MIME veri yazma
  async writeMimeData(mimeType, data) {
    const message = {
      records: [
        {
          recordType: 'mime',
          mediaType: mimeType,
          data: data
        }
      ]
    };
    
    return this.writeMessage(message);
  }
  
  // Akıllı poster yazma
  async writeSmartPoster(title, url, action = 'default') {
    const message = {
      records: [
        {
          recordType: 'smart-poster',
          data: new TextEncoder().encode(''),
          records: [
            {
              recordType: 'text',
              data: title,
              encoding: 'utf-8',
              lang: 'tr'
            },
            {
              recordType: 'url',
              data: url
            }
          ]
        }
      ]
    };
    
    return this.writeMessage(message);
  }
  
  // Mesaj işleyici kaydet
  registerHandler(type, handler) {
    if (!this.messageHandlers.has(type)) {
      this.messageHandlers.set(type, []);
    }
    this.messageHandlers.get(type).push(handler);
  }
  
  // Mesaj işleyici kaldır
  unregisterHandler(type, handler) {
    const handlers = this.messageHandlers.get(type);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index > -1) {
        handlers.splice(index, 1);
      }
    }
  }
  
  // İzin durumu
  getPermissionState() {
    return this.permissions;
  }
  
  // Tarama durumu
  getScanningState() {
    return this.isScanning;
  }
  
  // Destek durumu
  getSupportState() {
    return this.isSupported;
  }
  
  // Kaynakları temizle
  cleanup() {
    this.stopScanning();
    this.messageHandlers.clear();
    this.reader = null;
    this.writer = null;
    this.isInitialized = false;
  }
}

// NFC utility fonksiyonları
class NFCUtils {
  // Metin kaydı oluştur
  static createTextRecord(text, language = 'tr', encoding = 'utf-8') {
    return {
      recordType: 'text',
      data: text,
      encoding: encoding,
      lang: language
    };
  }
  
  // URL kaydı oluştur
  static createUrlRecord(url) {
    return {
      recordType: 'url',
      data: url
    };
  }
  
  // MIME kaydı oluştur
  static createMimeRecord(mimeType, data) {
    return {
      recordType: 'mime',
      mediaType: mimeType,
      data: data
    };
  }
  
  // Akıllı poster oluştur
  static createSmartPoster(title, url, action = 'default') {
    return {
      recordType: 'smart-poster',
      data: new TextEncoder().encode(''),
      records: [
        this.createTextRecord(title),
        this.createUrlRecord(url)
      ]
    };
  }
  
  // Mesaj doğrulama
  static validateMessage(message) {
    if (!message || !message.records) {
      return { valid: false, error: 'Geçersiz mesaj formatı' };
    }
    
    if (!Array.isArray(message.records)) {
      return { valid: false, error: 'Records dizi olmalı' };
    }
    
    if (message.records.length === 0) {
      return { valid: false, error: 'En az bir kayıt gerekli' };
    }
    
    return { valid: true };
  }
  
  // Kayıt doğrulama
  static validateRecord(record) {
    if (!record || !record.recordType) {
      return { valid: false, error: 'Geçersiz kayıt formatı' };
    }
    
    const validTypes = ['text', 'url', 'mime', 'absolute-url', 'smart-poster'];
    if (!validTypes.includes(record.recordType)) {
      return { valid: false, error: 'Geçersiz kayıt türü' };
    }
    
    return { valid: true };
  }
  
  // Veri boyutu kontrolü
  static checkDataSize(data) {
    const size = new Blob([data]).size;
    
    return {
      size: size,
      sizeKB: Math.round(size / 1024 * 100) / 100,
      isTooLarge: size > 8192 // 8KB limit
    };
  }
  
  // NFC mesajı sıkıştır
  static compressMessage(message) {
    const messageStr = JSON.stringify(message);
    // Basit sıkıştırma (gerçek uygulamada daha gelişmiş yöntemler kullanın)
    return btoa(messageStr);
  }
  
  // NFC mesajı aç
  static decompressMessage(compressedMessage) {
    try {
      const decompressed = atob(compressedMessage);
      return JSON.parse(decompressed);
    } catch (error) {
      throw new Error('Mesaj açılamadı');
    }
  }
}
```

**Alternatifler:**
- **QR Code**: QR kod
- **Bluetooth**: Bluetooth
- **WiFi Direct**: WiFi Direct
- **WebRTC**: WebRTC
- **Web Share API**: Web Share API

**Ne Zaman Kullanılmalı?**
- ✅ Temassız ödeme uygulamalarında
- ✅ Akıllı etiket uygulamalarında
- ✅ Hızlı veri transferi gerektiğinde
- ✅ IoT cihaz yönetimi gerektiğinde
- ✅ Erişim kontrolü gerektiğinde
- ✅ Bilgi paylaşımı gerektiğinde
- ✅ Mobil entegrasyon gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ NFC desteklemeyen cihazlarda
- ❌ Uzun mesafeli iletişim gerektiğinde
- ❌ Büyük veri transferi gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde

**Kullanılmazsa Ne Olur?**
- **No Contactless Payments**: Temassız ödemeler olmaz
- **No Smart Tags**: Akıllı etiketler olmaz
- **No Quick Data Transfer**: Hızlı veri transferi olmaz
- **No IoT Device Management**: IoT cihaz yönetimi olmaz
- **Limited Mobile Integration**: Mobil entegrasyon sınırlı olur

**Modern Alternatifler:**
- **Web NFC API**: Web NFC API
- **QR Code**: QR kod
- **Bluetooth**: Bluetooth
- **WiFi Direct**: WiFi Direct
- **WebRTC**: WebRTC

### Web Notifications API

**Web Notifications API Nedir?**
Web Notifications API, web uygulamalarının kullanıcılara sistem seviyesinde bildirimler göndermesini sağlayan bir web API'sidir. Tarayıcı dışında da görülebilen, ses ve titreşim ile desteklenen bildirimler oluşturur. Kullanıcı deneyimini artırır, önemli olayları bildirir ve uygulama etkileşimini sürdürür. Modern web uygulamalarında kullanıcı katılımını artırmak ve gerçek zamanlı güncellemeler sağlamak için kritik öneme sahiptir.

**Neden Kullanılır?**
- **User Engagement**: Kullanıcı katılımı
- **Real-time Updates**: Gerçek zamanlı güncellemeler
- **Important Alerts**: Önemli uyarılar
- **Background Notifications**: Arka plan bildirimleri
- **Cross-platform**: Platform bağımsızlığı
- **Rich Content**: Zengin içerik
- **User Experience**: Kullanıcı deneyimi

**Temel Kullanım Örnekleri:**

```javascript
// Bildirim izni kontrolü
if ('Notification' in window) {
  const permission = await Notification.requestPermission();
  
  if (permission === 'granted') {
    const notification = new Notification('Başlık', {
      body: 'Bildirim içeriği',
      icon: 'icon.png',
      badge: 'badge.png',
      tag: 'unique-tag',
      requireInteraction: true
    });
    
    // Bildirim olayları
    notification.onclick = () => {
      console.log('Bildirime tıklandı');
      window.focus();
    };
    
    notification.onclose = () => {
      console.log('Bildirim kapatıldı');
    };
    
    notification.onerror = () => {
      console.log('Bildirim hatası');
    };
  }
}

// Bildirim seçenekleri
const notificationOptions = {
  body: 'Detaylı açıklama',
  icon: '/images/icon-192x192.png',
  badge: '/images/badge-72x72.png',
  image: '/images/large-image.png',
  tag: 'notification-tag',
  data: { url: '/page' },
  requireInteraction: true,
  silent: false,
  timestamp: Date.now(),
  vibrate: [200, 100, 200],
  actions: [
    {
      action: 'view',
      title: 'Görüntüle',
      icon: '/images/view-icon.png'
    },
    {
      action: 'dismiss',
      title: 'Kapat',
      icon: '/images/dismiss-icon.png'
    }
  ]
};
```

**Alternatifler:**
- **Push API**: Push API
- **Service Workers**: Service Workers
- **Web Share API**: Web Share API
- **Email**: Email bildirimleri
- **SMS**: SMS bildirimleri

**Ne Zaman Kullanılmalı?**
- ✅ Kullanıcı katılımını artırmak için
- ✅ Gerçek zamanlı güncellemeler için
- ✅ Önemli uyarılar için
- ✅ Arka plan bildirimleri için
- ✅ Platform bağımsızlığı için
- ✅ Zengin içerik için
- ✅ Kullanıcı deneyimi için

**Ne Zaman Kullanılmamalı?**
- ❌ Spam bildirimler için
- ❌ Gereksiz uyarılar için
- ❌ Kullanıcı izni olmadan
- ❌ Eski tarayıcı desteği gerektiğinde

**Kullanılmazsa Ne Olur?**
- **No User Engagement**: Kullanıcı katılımı olmaz
- **No Real-time Updates**: Gerçek zamanlı güncellemeler olmaz
- **No Important Alerts**: Önemli uyarılar olmaz
- **No Background Notifications**: Arka plan bildirimleri olmaz
- **Limited User Experience**: Kullanıcı deneyimi sınırlı olur

**Modern Alternatifler:**
- **Web Notifications API**: Web Notifications API
- **Push API**: Push API
- **Service Workers**: Service Workers
- **Web Share API**: Web Share API
- **Email**: Email bildirimleri

### Web RTC API

**Web RTC API Nedir?**
Web RTC API, web uygulamalarının gerçek zamanlı ses, video ve veri iletişimi yapmasını sağlayan bir web API'sidir. Peer-to-peer (P2P) bağlantılar kurarak tarayıcılar arasında doğrudan iletişim sağlar. Video konferans, sesli arama, dosya paylaşımı ve gerçek zamanlı oyun uygulamaları için kritik öneme sahiptir. STUN/TURN sunucuları kullanarak NAT traversal yapar ve ICE protokolü ile en iyi bağlantı yolunu bulur.

**Neden Kullanılır?**
- **Real-time Communication**: Gerçek zamanlı iletişim
- **Peer-to-Peer**: Peer-to-peer bağlantılar
- **Video Conferencing**: Video konferans
- **Voice Calls**: Sesli aramalar
- **File Sharing**: Dosya paylaşımı
- **Gaming**: Oyun uygulamaları
- **Live Streaming**: Canlı yayın

**Temel Kullanım Örnekleri:**

```javascript
// Peer connection oluştur
const peerConnection = new RTCPeerConnection({
  iceServers: [
    {urls: 'stun:stun.l.google.com:19302'},
    {urls: 'stun:stun1.l.google.com:19302'}
  ]
});

// Local stream al
const localStream = await navigator.mediaDevices.getUserMedia({
  video: true,
  audio: true
});

// Stream'i peer connection'a ekle
localStream.getTracks().forEach(track => {
  peerConnection.addTrack(track, localStream);
});

// Remote stream'i al
peerConnection.ontrack = event => {
  const remoteStream = event.streams[0];
  const remoteVideo = document.getElementById('remoteVideo');
  remoteVideo.srcObject = remoteStream;
};

// ICE candidate'leri işle
peerConnection.onicecandidate = event => {
  if (event.candidate) {
    // Candidate'i diğer peer'a gönder
    sendToPeer('ice-candidate', event.candidate);
  }
};

// Connection state değişikliklerini izle
peerConnection.onconnectionstatechange = () => {
  console.log('Connection state:', peerConnection.connectionState);
};

// Offer oluştur ve gönder
const offer = await peerConnection.createOffer();
await peerConnection.setLocalDescription(offer);
sendToPeer('offer', offer);

// Answer al ve ayarla
peerConnection.setRemoteDescription(answer);
const answer = await peerConnection.createAnswer();
await peerConnection.setLocalDescription(answer);
sendToPeer('answer', answer);

// Web RTC yöneticisi
class WebRTCManager {
  constructor() {
    this.isSupported = 'RTCPeerConnection' in window;
    this.peerConnection = null;
    this.localStream = null;
    this.remoteStream = null;
    this.isInitialized = false;
    this.connectionState = 'new';
    this.iceConnectionState = 'new';
    this.dataChannel = null;
    this.messageHandlers = new Map();
    this.iceServers = [
      {urls: 'stun:stun.l.google.com:19302'},
      {urls: 'stun:stun1.l.google.com:19302'}
    ];
  }
  
  // Web RTC başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web RTC desteklenmiyor');
    }
    
    try {
      // Peer connection oluştur
      this.peerConnection = new RTCPeerConnection({
        iceServers: this.iceServers
      });
      
      // Event listener'ları ayarla
      this.setupEventListeners();
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Web RTC başlatma hatası:', error);
      return false;
    }
  }
  
  // Event listener'ları ayarla
  setupEventListeners() {
    // ICE candidate
    this.peerConnection.onicecandidate = (event) => {
      this.handleIceCandidate(event);
    };
    
    // Connection state
    this.peerConnection.onconnectionstatechange = () => {
      this.handleConnectionStateChange();
    };
    
    // ICE connection state
    this.peerConnection.oniceconnectionstatechange = () => {
      this.handleIceConnectionStateChange();
    };
    
    // Track (remote stream)
    this.peerConnection.ontrack = (event) => {
      this.handleTrack(event);
    };
    
    // Data channel
    this.peerConnection.ondatachannel = (event) => {
      this.handleDataChannel(event);
    };
  }
  
  // ICE candidate işleyici
  handleIceCandidate(event) {
    if (event.candidate) {
      console.log('ICE candidate:', event.candidate);
      
      // Candidate işleyicilerini çağır
      const handlers = this.messageHandlers.get('ice-candidate');
      if (handlers) {
        handlers.forEach(handler => {
          try {
            handler(event.candidate);
          } catch (error) {
            console.error('ICE candidate işleyici hatası:', error);
          }
        });
      }
    }
  }
  
  // Connection state değişikliği
  handleConnectionStateChange() {
    this.connectionState = this.peerConnection.connectionState;
    console.log('Connection state:', this.connectionState);
    
    // Connection state işleyicilerini çağır
    const handlers = this.messageHandlers.get('connection-state');
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(this.connectionState);
        } catch (error) {
          console.error('Connection state işleyici hatası:', error);
        }
      });
    }
  }
  
  // ICE connection state değişikliği
  handleIceConnectionStateChange() {
    this.iceConnectionState = this.peerConnection.iceConnectionState;
    console.log('ICE connection state:', this.iceConnectionState);
    
    // ICE connection state işleyicilerini çağır
    const handlers = this.messageHandlers.get('ice-connection-state');
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(this.iceConnectionState);
        } catch (error) {
          console.error('ICE connection state işleyici hatası:', error);
        }
      });
    }
  }
  
  // Track işleyici
  handleTrack(event) {
    this.remoteStream = event.streams[0];
    console.log('Remote stream alındı:', this.remoteStream);
    
    // Track işleyicilerini çağır
    const handlers = this.messageHandlers.get('track');
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(this.remoteStream);
        } catch (error) {
          console.error('Track işleyici hatası:', error);
        }
      });
    }
  }
  
  // Data channel işleyici
  handleDataChannel(event) {
    this.dataChannel = event.channel;
    console.log('Data channel alındı:', this.dataChannel);
    
    // Data channel event listener'ları
    this.dataChannel.onopen = () => {
      console.log('Data channel açıldı');
    };
    
    this.dataChannel.onmessage = (event) => {
      this.handleDataChannelMessage(event);
    };
    
    this.dataChannel.onclose = () => {
      console.log('Data channel kapandı');
    };
    
    this.dataChannel.onerror = (error) => {
      console.error('Data channel hatası:', error);
    };
  }
  
  // Data channel mesaj işleyici
  handleDataChannelMessage(event) {
    console.log('Data channel mesajı:', event.data);
    
    // Data channel mesaj işleyicilerini çağır
    const handlers = this.messageHandlers.get('data-channel-message');
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(event.data);
        } catch (error) {
          console.error('Data channel mesaj işleyici hatası:', error);
        }
      });
    }
  }
  
  // Local stream al
  async getLocalStream(constraints = {video: true, audio: true}) {
    try {
      this.localStream = await navigator.mediaDevices.getUserMedia(constraints);
      
      // Stream'i peer connection'a ekle
      this.localStream.getTracks().forEach(track => {
        this.peerConnection.addTrack(track, this.localStream);
      });
      
      return this.localStream;
    } catch (error) {
      console.error('Local stream alma hatası:', error);
      throw error;
    }
  }
  
  // Offer oluştur
  async createOffer() {
    if (!this.isInitialized) {
      throw new Error('Web RTC başlatılmamış');
    }
    
    try {
      const offer = await this.peerConnection.createOffer();
      await this.peerConnection.setLocalDescription(offer);
      
      console.log('Offer oluşturuldu:', offer);
      return offer;
    } catch (error) {
      console.error('Offer oluşturma hatası:', error);
      throw error;
    }
  }
  
  // Answer oluştur
  async createAnswer() {
    if (!this.isInitialized) {
      throw new Error('Web RTC başlatılmamış');
    }
    
    try {
      const answer = await this.peerConnection.createAnswer();
      await this.peerConnection.setLocalDescription(answer);
      
      console.log('Answer oluşturuldu:', answer);
      return answer;
    } catch (error) {
      console.error('Answer oluşturma hatası:', error);
      throw error;
    }
  }
  
  // Remote description ayarla
  async setRemoteDescription(description) {
    if (!this.isInitialized) {
      throw new Error('Web RTC başlatılmamış');
    }
    
    try {
      await this.peerConnection.setRemoteDescription(description);
      console.log('Remote description ayarlandı:', description);
    } catch (error) {
      console.error('Remote description ayarlama hatası:', error);
      throw error;
    }
  }
  
  // ICE candidate ekle
  async addIceCandidate(candidate) {
    if (!this.isInitialized) {
      throw new Error('Web RTC başlatılmamış');
    }
    
    try {
      await this.peerConnection.addIceCandidate(candidate);
      console.log('ICE candidate eklendi:', candidate);
    } catch (error) {
      console.error('ICE candidate ekleme hatası:', error);
      throw error;
    }
  }
  
  // Data channel oluştur
  createDataChannel(label, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web RTC başlatılmamış');
    }
    
    try {
      this.dataChannel = this.peerConnection.createDataChannel(label, options);
      
      // Data channel event listener'ları
      this.dataChannel.onopen = () => {
        console.log('Data channel açıldı');
      };
      
      this.dataChannel.onmessage = (event) => {
        this.handleDataChannelMessage(event);
      };
      
      this.dataChannel.onclose = () => {
        console.log('Data channel kapandı');
      };
      
      this.dataChannel.onerror = (error) => {
        console.error('Data channel hatası:', error);
      };
      
      console.log('Data channel oluşturuldu:', this.dataChannel);
      return this.dataChannel;
    } catch (error) {
      console.error('Data channel oluşturma hatası:', error);
      throw error;
    }
  }
  
  // Data channel mesaj gönder
  sendDataChannelMessage(message) {
    if (!this.dataChannel || this.dataChannel.readyState !== 'open') {
      throw new Error('Data channel açık değil');
    }
    
    try {
      this.dataChannel.send(message);
      console.log('Data channel mesajı gönderildi:', message);
    } catch (error) {
      console.error('Data channel mesaj gönderme hatası:', error);
      throw error;
    }
  }
  
  // Mesaj işleyici kaydet
  registerHandler(eventType, handler) {
    if (!this.messageHandlers.has(eventType)) {
      this.messageHandlers.set(eventType, []);
    }
    this.messageHandlers.get(eventType).push(handler);
  }
  
  // Mesaj işleyici kaldır
  unregisterHandler(eventType, handler) {
    const handlers = this.messageHandlers.get(eventType);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index > -1) {
        handlers.splice(index, 1);
      }
    }
  }
  
  // Bağlantı durumu
  getConnectionState() {
    return this.connectionState;
  }
  
  // ICE bağlantı durumu
  getIceConnectionState() {
    return this.iceConnectionState;
  }
  
  // Local stream
  getLocalStream() {
    return this.localStream;
  }
  
  // Remote stream
  getRemoteStream() {
    return this.remoteStream;
  }
  
  // Data channel
  getDataChannel() {
    return this.dataChannel;
  }
  
  // Bağlantıyı kapat
  close() {
    if (this.peerConnection) {
      this.peerConnection.close();
    }
    
    if (this.localStream) {
      this.localStream.getTracks().forEach(track => track.stop());
    }
    
    this.isInitialized = false;
    this.connectionState = 'closed';
    this.iceConnectionState = 'closed';
  }
  
  // Kaynakları temizle
  cleanup() {
    this.close();
    this.messageHandlers.clear();
    this.peerConnection = null;
    this.localStream = null;
    this.remoteStream = null;
    this.dataChannel = null;
  }
}
```

**Alternatifler:**
- **WebSocket**: WebSocket API
- **Server-Sent Events**: Server-Sent Events
- **WebRTC Data Channel**: WebRTC Data Channel
- **Web Messaging API**: Web Messaging API
- **Web Share API**: Web Share API

**Ne Zaman Kullanılmalı?**
- ✅ Gerçek zamanlı iletişim gerektiğinde
- ✅ Video konferans gerektiğinde
- ✅ Sesli arama gerektiğinde
- ✅ Dosya paylaşımı gerektiğinde
- ✅ Oyun uygulamaları gerektiğinde
- ✅ Canlı yayın gerektiğinde
- ✅ Peer-to-peer bağlantı gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Basit mesajlaşma gerektiğinde
- ❌ Tek yönlü iletişim gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Düşük bant genişliği gerektiğinde

**Kullanılmazsa Ne Olur?**
- **No Real-time Communication**: Gerçek zamanlı iletişim olmaz
- **No Peer-to-Peer**: Peer-to-peer bağlantılar olmaz
- **No Video Conferencing**: Video konferans olmaz
- **No Voice Calls**: Sesli aramalar olmaz
- **Limited File Sharing**: Dosya paylaşımı sınırlı olur

**Modern Alternatifler:**
- **Web RTC API**: Web RTC API
- **WebSocket**: WebSocket API
- **Server-Sent Events**: Server-Sent Events
- **WebRTC Data Channel**: WebRTC Data Channel
- **Web Messaging API**: Web Messaging API

### Web Serial API

**Web Serial API Nedir?**
Web Serial API, web uygulamalarının seri port cihazlarıyla (Arduino, mikrodenetleyiciler, sensörler) iletişim kurmasını sağlayan bir web API'sidir. USB seri portlar üzerinden veri gönderme ve alma imkanı sunar. IoT uygulamaları, robotik projeler, donanım prototipleme ve endüstriyel otomasyon için kritik öneme sahiptir. Tarayıcı güvenlik modeli içinde çalışarak kullanıcı izni ile cihazlara erişim sağlar.

**Neden Kullanılır?**
- **IoT Communication**: IoT iletişimi
- **Hardware Prototyping**: Donanım prototipleme
- **Robotics**: Robotik projeler
- **Industrial Automation**: Endüstriyel otomasyon
- **Sensor Integration**: Sensör entegrasyonu
- **Device Control**: Cihaz kontrolü
- **Data Logging**: Veri kaydetme

**Temel Kullanım Örnekleri:**

```javascript
// Seri port bağlantısı
if ('serial' in navigator) {
  const port = await navigator.serial.requestPort();
  await port.open({baudRate: 9600});
  
  // Veri gönderme
  const writer = port.writable.getWriter();
  const data = new TextEncoder().encode('Merhaba Arduino!');
  await writer.write(data);
  writer.releaseLock();
  
  // Veri alma
  const reader = port.readable.getReader();
  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    
    const text = new TextDecoder().decode(value);
    console.log('Gelen veri:', text);
  }
}

// Seri port ayarları
const serialOptions = {
  baudRate: 9600,
  dataBits: 8,
  stopBits: 1,
  parity: 'none',
  flowControl: 'none',
  bufferSize: 255
};

// Web Serial yöneticisi
class WebSerialManager {
  constructor() {
    this.isSupported = 'serial' in navigator;
    this.port = null;
    this.reader = null;
    this.writer = null;
    this.isConnected = false;
    this.messageHandlers = new Map();
    this.isInitialized = false;
    this.defaultOptions = {
      baudRate: 9600,
      dataBits: 8,
      stopBits: 1,
      parity: 'none',
      flowControl: 'none',
      bufferSize: 255
    };
  }
  
  // Seri port başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web Serial API desteklenmiyor');
    }
    
    try {
      // Port seçimi
      this.port = await navigator.serial.requestPort();
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Seri port başlatma hatası:', error);
      return false;
    }
  }
  
  // Port bağlantısı aç
  async connect(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Seri port başlatılmamış');
    }
    
    try {
      const connectionOptions = { ...this.defaultOptions, ...options };
      await this.port.open(connectionOptions);
      
      // Reader ve Writer oluştur
      this.reader = this.port.readable.getReader();
      this.writer = this.port.writable.getWriter();
      
      this.isConnected = true;
      
      // Okuma döngüsünü başlat
      this.startReading();
      
      console.log('Seri port bağlantısı açıldı');
      return true;
    } catch (error) {
      console.error('Seri port bağlantı hatası:', error);
      throw error;
    }
  }
  
  // Port bağlantısı kapat
  async disconnect() {
    if (!this.isConnected) {
      return;
    }
    
    try {
      // Reader'ı serbest bırak
      if (this.reader) {
        await this.reader.cancel();
        this.reader = null;
      }
      
      // Writer'ı serbest bırak
      if (this.writer) {
        await this.writer.close();
        this.writer = null;
      }
      
      // Port'u kapat
      if (this.port) {
        await this.port.close();
      }
      
      this.isConnected = false;
      console.log('Seri port bağlantısı kapatıldı');
    } catch (error) {
      console.error('Seri port kapatma hatası:', error);
    }
  }
  
  // Okuma döngüsü başlat
  async startReading() {
    if (!this.reader) {
      return;
    }
    
    try {
      while (this.isConnected) {
        const { value, done } = await this.reader.read();
        
        if (done) {
          console.log('Okuma tamamlandı');
          break;
        }
        
        // Veriyi işle
        this.handleIncomingData(value);
      }
    } catch (error) {
      console.error('Okuma hatası:', error);
    }
  }
  
  // Gelen veriyi işle
  handleIncomingData(data) {
    try {
      // Veriyi metne çevir
      const text = new TextDecoder().decode(data);
      
      console.log('Gelen veri:', text);
      
      // Mesaj işleyicilerini çağır
      this.messageHandlers.forEach(handler => {
        try {
          handler(text, data);
        } catch (error) {
          console.error('Mesaj işleyici hatası:', error);
        }
      });
    } catch (error) {
      console.error('Veri işleme hatası:', error);
    }
  }
  
  // Veri gönder
  async sendData(data) {
    if (!this.isConnected || !this.writer) {
      throw new Error('Seri port bağlı değil');
    }
    
    try {
      let encodedData;
      
      if (typeof data === 'string') {
        encodedData = new TextEncoder().encode(data);
      } else if (data instanceof Uint8Array) {
        encodedData = data;
      } else {
        throw new Error('Desteklenmeyen veri tipi');
      }
      
      await this.writer.write(encodedData);
      console.log('Veri gönderildi:', data);
    } catch (error) {
      console.error('Veri gönderme hatası:', error);
      throw error;
    }
  }
  
  // Metin gönder
  async sendText(text) {
    return this.sendData(text);
  }
  
  // Komut gönder
  async sendCommand(command, parameters = []) {
    const commandString = `${command} ${parameters.join(' ')}\n`;
    return this.sendData(commandString);
  }
  
  // JSON gönder
  async sendJSON(obj) {
    const jsonString = JSON.stringify(obj) + '\n';
    return this.sendData(jsonString);
  }
  
  // Mesaj işleyici kaydet
  registerHandler(handler) {
    this.messageHandlers.set(handler, handler);
  }
  
  // Mesaj işleyici kaldır
  unregisterHandler(handler) {
    this.messageHandlers.delete(handler);
  }
  
  // Port bilgilerini al
  getPortInfo() {
    if (!this.port) return null;
    
    return {
      usbVendorId: this.port.getInfo().usbVendorId,
      usbProductId: this.port.getInfo().usbProductId,
      isConnected: this.isConnected
    };
  }
  
  // Bağlantı durumu
  getConnectionState() {
    return this.isConnected;
  }
  
  // Destek durumu
  getSupportState() {
    return this.isSupported;
  }
  
  // Kaynakları temizle
  async cleanup() {
    await this.disconnect();
    this.messageHandlers.clear();
    this.port = null;
    this.isInitialized = false;
  }
}

// Seri port utility fonksiyonları
class SerialUtils {
  // Baud rate doğrulama
  static validateBaudRate(baudRate) {
    const validRates = [300, 600, 1200, 2400, 4800, 9600, 14400, 19200, 28800, 38400, 57600, 115200];
    return validRates.includes(baudRate);
  }
  
  // Veri biti doğrulama
  static validateDataBits(dataBits) {
    return [7, 8].includes(dataBits);
  }
  
  // Stop bit doğrulama
  static validateStopBits(stopBits) {
    return [1, 2].includes(stopBits);
  }
  
  // Parity doğrulama
  static validateParity(parity) {
    return ['none', 'even', 'odd'].includes(parity);
  }
  
  // Flow control doğrulama
  static validateFlowControl(flowControl) {
    return ['none', 'hardware'].includes(flowControl);
  }
  
  // Seri port seçenekleri doğrulama
  static validateSerialOptions(options) {
    const errors = [];
    
    if (options.baudRate && !this.validateBaudRate(options.baudRate)) {
      errors.push('Geçersiz baud rate');
    }
    
    if (options.dataBits && !this.validateDataBits(options.dataBits)) {
      errors.push('Geçersiz data bits');
    }
    
    if (options.stopBits && !this.validateStopBits(options.stopBits)) {
      errors.push('Geçersiz stop bits');
    }
    
    if (options.parity && !this.validateParity(options.parity)) {
      errors.push('Geçersiz parity');
    }
    
    if (options.flowControl && !this.validateFlowControl(options.flowControl)) {
      errors.push('Geçersiz flow control');
    }
    
    return {
      valid: errors.length === 0,
      errors: errors
    };
  }
  
  // Veri boyutu hesapla
  static calculateDataSize(data) {
    if (typeof data === 'string') {
      return new TextEncoder().encode(data).length;
    } else if (data instanceof Uint8Array) {
      return data.length;
    }
    return 0;
  }
  
  // Veri sıkıştırma
  static compressData(data) {
    // Basit sıkıştırma (gerçek uygulamada daha gelişmiş yöntemler kullanın)
    if (typeof data === 'string') {
      return new TextEncoder().encode(data);
    }
    return data;
  }
  
  // Veri açma
  static decompressData(data) {
    try {
      return new TextDecoder().decode(data);
    } catch (error) {
      throw new Error('Veri açılamadı');
    }
  }
  
  // Komut oluştur
  static createCommand(command, parameters = []) {
    return `${command} ${parameters.join(' ')}\n`;
  }
  
  // Komut ayrıştır
  static parseCommand(commandString) {
    const parts = commandString.trim().split(' ');
    return {
      command: parts[0],
      parameters: parts.slice(1)
    };
  }
  
  // JSON komut oluştur
  static createJSONCommand(type, data) {
    return JSON.stringify({ type, data, timestamp: Date.now() }) + '\n';
  }
  
  // JSON komut ayrıştır
  static parseJSONCommand(jsonString) {
    try {
      return JSON.parse(jsonString.trim());
    } catch (error) {
      throw new Error('Geçersiz JSON komut');
    }
  }
}
```

**Alternatifler:**
- **Web Bluetooth API**: Web Bluetooth API
- **Web USB API**: Web USB API
- **Web HID API**: Web HID API
- **WebSocket**: WebSocket API
- **WebRTC**: WebRTC API

**Ne Zaman Kullanılmalı?**
- ✅ IoT iletişimi gerektiğinde
- ✅ Donanım prototipleme gerektiğinde
- ✅ Robotik projeler gerektiğinde
- ✅ Endüstriyel otomasyon gerektiğinde
- ✅ Sensör entegrasyonu gerektiğinde
- ✅ Cihaz kontrolü gerektiğinde
- ✅ Veri kaydetme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Yüksek hızlı veri transferi gerektiğinde
- ❌ Kablosuz iletişim gerektiğinde
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Güvenlik kritik uygulamalarda

**Kullanılmazsa Ne Olur?**
- **No IoT Communication**: IoT iletişimi olmaz
- **No Hardware Prototyping**: Donanım prototipleme olmaz
- **No Robotics**: Robotik projeler olmaz
- **No Industrial Automation**: Endüstriyel otomasyon olmaz
- **Limited Device Control**: Cihaz kontrolü sınırlı olur

**Modern Alternatifler:**
- **Web Serial API**: Web Serial API
- **Web Bluetooth API**: Web Bluetooth API
- **Web USB API**: Web USB API
- **Web HID API**: Web HID API
- **WebSocket**: WebSocket API

### Web Share API

**Web Share API Nedir?**
Web Share API, web uygulamalarının yerel paylaşım özelliklerini kullanarak içerik paylaşmasını sağlayan bir web API'sidir. Kullanıcıların cihazlarının yerel paylaşım menüsünü kullanarak metin, URL, dosya ve diğer içerikleri sosyal medya, mesajlaşma uygulamaları, e-posta ve diğer uygulamalarla paylaşmasına olanak tanır. PWA'lar ve web uygulamaları için kritik öneme sahiptir, kullanıcı deneyimini iyileştirir ve native uygulama benzeri işlevsellik sağlar.

**Neden Kullanılır?**
- **Native Sharing**: Yerel paylaşım özellikleri
- **Social Media Integration**: Sosyal medya entegrasyonu
- **Content Distribution**: İçerik dağıtımı
- **User Experience**: Kullanıcı deneyimi
- **PWA Features**: PWA özellikleri
- **Cross-Platform**: Platformlar arası uyumluluk
- **Accessibility**: Erişilebilirlik

**Temel Kullanım Örnekleri:**

```javascript
// Temel paylaşım
if ('share' in navigator) {
  try {
    await navigator.share({
      title: 'Başlık',
      text: 'Paylaşılacak metin',
      url: 'https://example.com'
    });
    console.log('Paylaşım başarılı');
  } catch (err) {
    console.error('Paylaşım hatası:', err);
  }
}

// Dosya paylaşımı
if ('canShare' in navigator && navigator.canShare({ files: [file] })) {
  try {
    await navigator.share({
      title: 'Dosya Paylaşımı',
      text: 'Bu dosyayı paylaş',
      files: [file]
    });
  } catch (err) {
    console.error('Dosya paylaşım hatası:', err);
  }
}

// Web Share yöneticisi
class WebShareManager {
  constructor() {
    this.isSupported = 'share' in navigator;
    this.canShare = 'canShare' in navigator;
    this.isInitialized = false;
    this.shareHandlers = new Map();
    this.defaultOptions = {
      title: '',
      text: '',
      url: ''
    };
  }
  
  // Paylaşım yöneticisini başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web Share API desteklenmiyor');
    }
    
    try {
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Web Share başlatma hatası:', error);
      return false;
    }
  }
  
  // Paylaşım yapabilir mi kontrol et
  canShareData(data) {
    if (!this.canShare) {
      return false;
    }
    
    try {
      return navigator.canShare(data);
    } catch (error) {
      console.error('Paylaşım kontrolü hatası:', error);
      return false;
    }
  }
  
  // Metin paylaşımı
  async shareText(title, text, url = '') {
    if (!this.isInitialized) {
      throw new Error('Web Share başlatılmamış');
    }
    
    const shareData = {
      title: title || this.defaultOptions.title,
      text: text || this.defaultOptions.text,
      url: url || this.defaultOptions.url
    };
    
    // Paylaşım yapabilir mi kontrol et
    if (!this.canShareData(shareData)) {
      throw new Error('Bu veri paylaşılamaz');
    }
    
    try {
      await navigator.share(shareData);
      console.log('Metin paylaşımı başarılı');
      return true;
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('Paylaşım iptal edildi');
      } else {
        console.error('Metin paylaşım hatası:', error);
      }
      throw error;
    }
  }
  
  // URL paylaşımı
  async shareURL(url, title = '', text = '') {
    return this.shareText(title, text, url);
  }
  
  // Dosya paylaşımı
  async shareFiles(files, title = '', text = '') {
    if (!this.isInitialized) {
      throw new Error('Web Share başlatılmamış');
    }
    
    const shareData = {
      title: title || this.defaultOptions.title,
      text: text || this.defaultOptions.text,
      files: files
    };
    
    // Dosya paylaşımı yapabilir mi kontrol et
    if (!this.canShareData(shareData)) {
      throw new Error('Bu dosyalar paylaşılamaz');
    }
    
    try {
      await navigator.share(shareData);
      console.log('Dosya paylaşımı başarılı');
      return true;
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('Dosya paylaşımı iptal edildi');
      } else {
        console.error('Dosya paylaşım hatası:', error);
      }
      throw error;
    }
  }
  
  // Resim paylaşımı
  async shareImage(imageFile, title = '', text = '') {
    return this.shareFiles([imageFile], title, text);
  }
  
  // Video paylaşımı
  async shareVideo(videoFile, title = '', text = '') {
    return this.shareFiles([videoFile], title, text);
  }
  
  // Çoklu dosya paylaşımı
  async shareMultipleFiles(files, title = '', text = '') {
    return this.shareFiles(files, title, text);
  }
  
  // Paylaşım işleyici kaydet
  registerHandler(eventType, handler) {
    this.shareHandlers.set(eventType, handler);
  }
  
  // Paylaşım işleyici kaldır
  unregisterHandler(eventType) {
    this.shareHandlers.delete(eventType);
  }
  
  // Paylaşım işleyicilerini çağır
  callHandlers(eventType, data) {
    const handler = this.shareHandlers.get(eventType);
    if (handler) {
      try {
        handler(data);
      } catch (error) {
        console.error('Paylaşım işleyici hatası:', error);
      }
    }
  }
  
  // Paylaşım durumu
  getShareState() {
    return {
      isSupported: this.isSupported,
      canShare: this.canShare,
      isInitialized: this.isInitialized
    };
  }
  
  // Destek durumu
  getSupportState() {
    return this.isSupported;
  }
  
  // Kaynakları temizle
  cleanup() {
    this.shareHandlers.clear();
    this.isInitialized = false;
  }
}

// Web Share utility fonksiyonları
class WebShareUtils {
  // Paylaşım verisi doğrulama
  static validateShareData(data) {
    const errors = [];
    
    if (!data || typeof data !== 'object') {
      errors.push('Paylaşım verisi geçersiz');
      return { valid: false, errors };
    }
    
    if (data.title && typeof data.title !== 'string') {
      errors.push('Başlık metin olmalı');
    }
    
    if (data.text && typeof data.text !== 'string') {
      errors.push('Metin string olmalı');
    }
    
    if (data.url && typeof data.url !== 'string') {
      errors.push('URL string olmalı');
    }
    
    if (data.files && !Array.isArray(data.files)) {
      errors.push('Dosyalar array olmalı');
    }
    
    return {
      valid: errors.length === 0,
      errors: errors
    };
  }
  
  // URL doğrulama
  static validateURL(url) {
    try {
      new URL(url);
      return true;
    } catch (error) {
      return false;
    }
  }
  
  // Dosya tipi doğrulama
  static validateFileType(file, allowedTypes = []) {
    if (allowedTypes.length === 0) {
      return true;
    }
    
    return allowedTypes.some(type => {
      if (type.startsWith('.')) {
        return file.name.toLowerCase().endsWith(type.toLowerCase());
      } else {
        return file.type.startsWith(type);
      }
    });
  }
  
  // Dosya boyutu doğrulama
  static validateFileSize(file, maxSize = 10 * 1024 * 1024) { // 10MB
    return file.size <= maxSize;
  }
  
  // Paylaşım verisi oluştur
  static createShareData(title, text, url, files = []) {
    const shareData = {};
    
    if (title) shareData.title = title;
    if (text) shareData.text = text;
    if (url) shareData.url = url;
    if (files.length > 0) shareData.files = files;
    
    return shareData;
  }
  
  // Paylaşım verisi temizle
  static sanitizeShareData(data) {
    const sanitized = {};
    
    if (data.title) {
      sanitized.title = data.title.trim().substring(0, 100);
    }
    
    if (data.text) {
      sanitized.text = data.text.trim().substring(0, 500);
    }
    
    if (data.url) {
      sanitized.url = data.url.trim();
    }
    
    if (data.files) {
      sanitized.files = data.files.filter(file => file instanceof File);
    }
    
    return sanitized;
  }
  
  // Paylaşım istatistikleri
  static getShareStats() {
    return {
      totalShares: 0,
      successfulShares: 0,
      failedShares: 0,
      cancelledShares: 0
    };
  }
  
  // Paylaşım geçmişi
  static getShareHistory() {
    return [];
  }
  
  // Paylaşım tercihleri
  static getSharePreferences() {
    return {
      defaultTitle: '',
      defaultText: '',
      defaultURL: '',
      allowedFileTypes: [],
      maxFileSize: 10 * 1024 * 1024
    };
  }
  
  // Paylaşım tercihlerini kaydet
  static saveSharePreferences(preferences) {
    try {
      localStorage.setItem('webSharePreferences', JSON.stringify(preferences));
      return true;
    } catch (error) {
      console.error('Paylaşım tercihleri kaydedilemedi:', error);
      return false;
    }
  }
  
  // Paylaşım tercihlerini yükle
  static loadSharePreferences() {
    try {
      const stored = localStorage.getItem('webSharePreferences');
      return stored ? JSON.parse(stored) : this.getSharePreferences();
    } catch (error) {
      console.error('Paylaşım tercihleri yüklenemedi:', error);
      return this.getSharePreferences();
    }
  }
}
```

**Alternatifler:**
- **Social Media APIs**: Sosyal medya API'leri
- **Email APIs**: E-posta API'leri
- **SMS APIs**: SMS API'leri
- **Clipboard API**: Pano API'si
- **Custom Share Buttons**: Özel paylaşım butonları

**Ne Zaman Kullanılmalı?**
- ✅ Native paylaşım özellikleri gerektiğinde
- ✅ Sosyal medya entegrasyonu gerektiğinde
- ✅ İçerik dağıtımı gerektiğinde
- ✅ Kullanıcı deneyimi iyileştirme gerektiğinde
- ✅ PWA özellikleri gerektiğinde
- ✅ Platformlar arası uyumluluk gerektiğinde
- ✅ Erişilebilirlik gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Özel paylaşım mantığı gerektiğinde
- ❌ Güvenlik kritik uygulamalarda
- ❌ Offline uygulamalarda

**Kullanılmazsa Ne Olur?**
- **No Native Sharing**: Yerel paylaşım özellikleri olmaz
- **No Social Media Integration**: Sosyal medya entegrasyonu olmaz
- **No Content Distribution**: İçerik dağıtımı olmaz
- **Poor User Experience**: Kullanıcı deneyimi kötü olur
- **Limited PWA Features**: PWA özellikleri sınırlı olur

**Modern Alternatifler:**
- **Web Share API**: Web Share API
- **Social Media APIs**: Sosyal medya API'leri
- **Email APIs**: E-posta API'leri
- **SMS APIs**: SMS API'leri
- **Clipboard API**: Pano API'si

### Web Speech API

**Web Speech API Nedir?**
Web Speech API, web uygulamalarında konuşma tanıma (speech recognition) ve konuşma sentezi (speech synthesis) özelliklerini sağlayan bir web API'sidir. Kullanıcıların sesli komutlarla uygulamalarla etkileşim kurmasını, metinleri sesli olarak okutmasını ve erişilebilirlik özelliklerini geliştirmesini sağlar. Sesli asistanlar, dikte uygulamaları, çok dilli uygulamalar ve erişilebilirlik araçları için kritik öneme sahiptir. Tarayıcı tabanlı ses işleme ve doğal dil işleme imkanı sunar.

**Neden Kullanılır?**
- **Voice Commands**: Sesli komutlar
- **Speech Recognition**: Konuşma tanıma
- **Speech Synthesis**: Konuşma sentezi
- **Accessibility**: Erişilebilirlik
- **Multilingual Support**: Çok dilli destek
- **Voice Assistants**: Sesli asistanlar
- **Dictation**: Dikte uygulamaları

**Temel Kullanım Örnekleri:**

```javascript
// Konuşma tanıma
if ('webkitSpeechRecognition' in window) {
  const recognition = new webkitSpeechRecognition();
  recognition.continuous = true;
  recognition.interimResults = true;
  
  recognition.onresult = event => {
    const transcript = event.results[event.results.length - 1][0].transcript;
    console.log('Tanınan metin:', transcript);
  };
  
  recognition.start();
}

// Konuşma sentezi
if ('speechSynthesis' in window) {
  const utterance = new SpeechSynthesisUtterance('Merhaba dünya!');
  utterance.lang = 'tr-TR';
  utterance.rate = 1;
  utterance.pitch = 1;
  utterance.volume = 1;
  
  speechSynthesis.speak(utterance);
}

// Web Speech yöneticisi
class WebSpeechManager {
  constructor() {
    this.isRecognitionSupported = 'webkitSpeechRecognition' in window || 'SpeechRecognition' in window;
    this.isSynthesisSupported = 'speechSynthesis' in window;
    this.recognition = null;
    this.synthesis = window.speechSynthesis;
    this.isInitialized = false;
    this.recognitionHandlers = new Map();
    this.synthesisHandlers = new Map();
    this.defaultOptions = {
      continuous: true,
      interimResults: true,
      lang: 'tr-TR',
      maxAlternatives: 1
    };
  }
  
  // Speech yöneticisini başlat
  async initialize() {
    if (!this.isRecognitionSupported && !this.isSynthesisSupported) {
      throw new Error('Web Speech API desteklenmiyor');
    }
    
    try {
      // Recognition başlat
      if (this.isRecognitionSupported) {
        this.recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
        this.setupRecognition();
      }
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Web Speech başlatma hatası:', error);
      return false;
    }
  }
  
  // Recognition ayarları
  setupRecognition() {
    if (!this.recognition) return;
    
    this.recognition.continuous = this.defaultOptions.continuous;
    this.recognition.interimResults = this.defaultOptions.interimResults;
    this.recognition.lang = this.defaultOptions.lang;
    this.recognition.maxAlternatives = this.defaultOptions.maxAlternatives;
    
    // Event listeners
    this.recognition.onstart = () => {
      console.log('Konuşma tanıma başladı');
      this.callHandlers('start', {});
    };
    
    this.recognition.onresult = (event) => {
      const results = event.results;
      const transcript = results[results.length - 1][0].transcript;
      const confidence = results[results.length - 1][0].confidence;
      
      console.log('Tanınan metin:', transcript);
      this.callHandlers('result', { transcript, confidence, results });
    };
    
    this.recognition.onerror = (event) => {
      console.error('Konuşma tanıma hatası:', event.error);
      this.callHandlers('error', { error: event.error });
    };
    
    this.recognition.onend = () => {
      console.log('Konuşma tanıma bitti');
      this.callHandlers('end', {});
    };
  }
  
  // Konuşma tanıma başlat
  startRecognition(options = {}) {
    if (!this.isInitialized || !this.recognition) {
      throw new Error('Web Speech başlatılmamış');
    }
    
    try {
      // Seçenekleri uygula
      if (options.lang) this.recognition.lang = options.lang;
      if (options.continuous !== undefined) this.recognition.continuous = options.continuous;
      if (options.interimResults !== undefined) this.recognition.interimResults = options.interimResults;
      
      this.recognition.start();
      return true;
    } catch (error) {
      console.error('Konuşma tanıma başlatma hatası:', error);
      throw error;
    }
  }
  
  // Konuşma tanıma durdur
  stopRecognition() {
    if (this.recognition) {
      this.recognition.stop();
    }
  }
  
  // Konuşma tanıma iptal et
  abortRecognition() {
    if (this.recognition) {
      this.recognition.abort();
    }
  }
  
  // Konuşma sentezi
  speak(text, options = {}) {
    if (!this.isSynthesisSupported) {
      throw new Error('Konuşma sentezi desteklenmiyor');
    }
    
    try {
      const utterance = new SpeechSynthesisUtterance(text);
      
      // Seçenekleri uygula
      utterance.lang = options.lang || 'tr-TR';
      utterance.rate = options.rate || 1;
      utterance.pitch = options.pitch || 1;
      utterance.volume = options.volume || 1;
      utterance.voice = options.voice || null;
      
      // Event listeners
      utterance.onstart = () => {
        console.log('Konuşma sentezi başladı');
        this.callHandlers('speakStart', { text, options });
      };
      
      utterance.onend = () => {
        console.log('Konuşma sentezi bitti');
        this.callHandlers('speakEnd', { text, options });
      };
      
      utterance.onerror = (event) => {
        console.error('Konuşma sentezi hatası:', event.error);
        this.callHandlers('speakError', { error: event.error, text, options });
      };
      
      this.synthesis.speak(utterance);
      return utterance;
    } catch (error) {
      console.error('Konuşma sentezi hatası:', error);
      throw error;
    }
  }
  
  // Konuşma sentezi durdur
  stopSpeaking() {
    if (this.synthesis) {
      this.synthesis.cancel();
    }
  }
  
  // Konuşma sentezi duraklat
  pauseSpeaking() {
    if (this.synthesis) {
      this.synthesis.pause();
    }
  }
  
  // Konuşma sentezi devam ettir
  resumeSpeaking() {
    if (this.synthesis) {
      this.synthesis.resume();
    }
  }
  
  // Mevcut sesler
  getVoices() {
    if (!this.synthesis) return [];
    return this.synthesis.getVoices();
  }
  
  // Dil desteği
  getSupportedLanguages() {
    const voices = this.getVoices();
    const languages = new Set();
    
    voices.forEach(voice => {
      if (voice.lang) {
        languages.add(voice.lang);
      }
    });
    
    return Array.from(languages);
  }
  
  // Ses arama
  findVoice(lang, name = '') {
    const voices = this.getVoices();
    return voices.find(voice => 
      voice.lang === lang && 
      (name === '' || voice.name.includes(name))
    );
  }
  
  // Event handler kaydet
  registerHandler(eventType, handler) {
    if (eventType.startsWith('speak')) {
      this.synthesisHandlers.set(eventType, handler);
    } else {
      this.recognitionHandlers.set(eventType, handler);
    }
  }
  
  // Event handler kaldır
  unregisterHandler(eventType) {
    if (eventType.startsWith('speak')) {
      this.synthesisHandlers.delete(eventType);
    } else {
      this.recognitionHandlers.delete(eventType);
    }
  }
  
  // Event handler'ları çağır
  callHandlers(eventType, data) {
    const handlers = eventType.startsWith('speak') ? 
      this.synthesisHandlers : this.recognitionHandlers;
    
    const handler = handlers.get(eventType);
    if (handler) {
      try {
        handler(data);
      } catch (error) {
        console.error('Event handler hatası:', error);
      }
    }
  }
  
  // Durum bilgisi
  getState() {
    return {
      isRecognitionSupported: this.isRecognitionSupported,
      isSynthesisSupported: this.isSynthesisSupported,
      isInitialized: this.isInitialized,
      isRecognizing: this.recognition && this.recognition.isRecognizing,
      isSpeaking: this.synthesis && this.synthesis.speaking,
      isPaused: this.synthesis && this.synthesis.paused
    };
  }
  
  // Destek durumu
  getSupportState() {
    return {
      recognition: this.isRecognitionSupported,
      synthesis: this.isSynthesisSupported
    };
  }
  
  // Kaynakları temizle
  cleanup() {
    if (this.recognition) {
      this.recognition.abort();
      this.recognition = null;
    }
    
    if (this.synthesis) {
      this.synthesis.cancel();
    }
    
    this.recognitionHandlers.clear();
    this.synthesisHandlers.clear();
    this.isInitialized = false;
  }
}

// Web Speech utility fonksiyonları
class WebSpeechUtils {
  // Dil kodu doğrulama
  static validateLanguageCode(lang) {
    const validLanguages = [
      'tr-TR', 'en-US', 'en-GB', 'es-ES', 'fr-FR', 'de-DE', 
      'it-IT', 'pt-PT', 'ru-RU', 'ja-JP', 'ko-KR', 'zh-CN'
    ];
    return validLanguages.includes(lang);
  }
  
  // Ses seçenekleri doğrulama
  static validateVoiceOptions(options) {
    const errors = [];
    
    if (options.rate && (options.rate < 0.1 || options.rate > 10)) {
      errors.push('Rate 0.1-10 arasında olmalı');
    }
    
    if (options.pitch && (options.pitch < 0 || options.pitch > 2)) {
      errors.push('Pitch 0-2 arasında olmalı');
    }
    
    if (options.volume && (options.volume < 0 || options.volume > 1)) {
      errors.push('Volume 0-1 arasında olmalı');
    }
    
    if (options.lang && !this.validateLanguageCode(options.lang)) {
      errors.push('Geçersiz dil kodu');
    }
    
    return {
      valid: errors.length === 0,
      errors: errors
    };
  }
  
  // Metin temizleme
  static sanitizeText(text) {
    if (typeof text !== 'string') {
      return '';
    }
    
    // HTML etiketlerini kaldır
    const cleanText = text.replace(/<[^>]*>/g, '');
    
    // Özel karakterleri temizle
    const sanitized = cleanText.replace(/[^\w\s.,!?;:()-]/g, '');
    
    return sanitized.trim();
  }
  
  // Metin uzunluğu kontrolü
  static validateTextLength(text, maxLength = 1000) {
    return text.length <= maxLength;
  }
  
  // Ses kalitesi ayarları
  static getOptimalVoiceSettings(lang = 'tr-TR') {
    const settings = {
      'tr-TR': { rate: 1, pitch: 1, volume: 1 },
      'en-US': { rate: 1.1, pitch: 1, volume: 1 },
      'en-GB': { rate: 1, pitch: 0.9, volume: 1 },
      'es-ES': { rate: 1.2, pitch: 1, volume: 1 },
      'fr-FR': { rate: 1.1, pitch: 1.1, volume: 1 },
      'de-DE': { rate: 1, pitch: 0.8, volume: 1 }
    };
    
    return settings[lang] || settings['tr-TR'];
  }
  
  // Konuşma tanıma ayarları
  static getOptimalRecognitionSettings(lang = 'tr-TR') {
    return {
      continuous: true,
      interimResults: true,
      lang: lang,
      maxAlternatives: 1
    };
  }
  
  // Ses dosyası oluşturma
  static async createAudioFile(text, options = {}) {
    return new Promise((resolve, reject) => {
      if (!('speechSynthesis' in window)) {
        reject(new Error('Konuşma sentezi desteklenmiyor'));
        return;
      }
      
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = options.lang || 'tr-TR';
      utterance.rate = options.rate || 1;
      utterance.pitch = options.pitch || 1;
      utterance.volume = options.volume || 1;
      
      // Audio context oluştur
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const destination = audioContext.createMediaStreamDestination();
      
      utterance.onend = () => {
        resolve(destination.stream);
      };
      
      utterance.onerror = (event) => {
        reject(new Error('Ses oluşturma hatası: ' + event.error));
      };
      
      speechSynthesis.speak(utterance);
    });
  }
  
  // Konuşma tanıma istatistikleri
  static getRecognitionStats() {
    return {
      totalRecognitions: 0,
      successfulRecognitions: 0,
      failedRecognitions: 0,
      averageConfidence: 0
    };
  }
  
  // Konuşma sentezi istatistikleri
  static getSynthesisStats() {
    return {
      totalSpeeches: 0,
      successfulSpeeches: 0,
      failedSpeeches: 0,
      totalDuration: 0
    };
  }
}
```

**Alternatifler:**
- **Web Audio API**: Web Audio API
- **MediaRecorder API**: MediaRecorder API
- **WebRTC**: WebRTC API
- **Third-party APIs**: Üçüncü parti API'ler
- **Native Apps**: Native uygulamalar

**Ne Zaman Kullanılmalı?**
- ✅ Sesli komutlar gerektiğinde
- ✅ Konuşma tanıma gerektiğinde
- ✅ Konuşma sentezi gerektiğinde
- ✅ Erişilebilirlik gerektiğinde
- ✅ Çok dilli destek gerektiğinde
- ✅ Sesli asistanlar gerektiğinde
- ✅ Dikte uygulamaları gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Yüksek güvenlik gerektiğinde
- ❌ Offline uygulamalarda
- ❌ Düşük bant genişliğinde

**Kullanılmazsa Ne Olur?**
- **No Voice Commands**: Sesli komutlar olmaz
- **No Speech Recognition**: Konuşma tanıma olmaz
- **No Speech Synthesis**: Konuşma sentezi olmaz
- **Poor Accessibility**: Erişilebilirlik kötü olur
- **Limited Multilingual Support**: Çok dilli destek sınırlı olur

**Modern Alternatifler:**
- **Web Speech API**: Web Speech API
- **Web Audio API**: Web Audio API
- **MediaRecorder API**: MediaRecorder API
- **WebRTC**: WebRTC API
- **Third-party APIs**: Üçüncü parti API'ler

### Web Storage API

**Web Storage API Nedir?**
Web Storage API, web uygulamalarının tarayıcıda veri saklamasını sağlayan bir web API'sidir. Local Storage ve Session Storage olmak üzere iki ana depolama türü sunar. Local Storage verileri kalıcı olarak saklar, Session Storage ise tarayıcı sekmesi kapatıldığında verileri siler. Cookie'lerden daha büyük depolama kapasitesi (genellikle 5-10MB) ve daha iyi performans sağlar. Offline uygulamalar, kullanıcı tercihleri, geçici veriler ve cache mekanizmaları için kritik öneme sahiptir.

**Neden Kullanılır?**
- **Data Persistence**: Veri kalıcılığı
- **User Preferences**: Kullanıcı tercihleri
- **Offline Storage**: Offline depolama
- **Performance**: Performans
- **Caching**: Önbellekleme
- **State Management**: Durum yönetimi
- **Cross-Tab Communication**: Sekmeler arası iletişim

**Temel Kullanım Örnekleri:**

```javascript
// Local Storage
localStorage.setItem('key', 'value');
const value = localStorage.getItem('key');
localStorage.removeItem('key');

// Session Storage
sessionStorage.setItem('key', 'value');
const sessionValue = sessionStorage.getItem('key');

// Storage event
window.addEventListener('storage', event => {
  console.log('Depolama değişti:', event.key, event.newValue);
});

// Web Storage yöneticisi
class WebStorageManager {
  constructor() {
    this.isSupported = this.checkSupport();
    this.storageTypes = {
      LOCAL: 'localStorage',
      SESSION: 'sessionStorage'
    };
    this.storageHandlers = new Map();
    this.isInitialized = false;
    this.defaultOptions = {
      type: this.storageTypes.LOCAL,
      prefix: '',
      expiration: null,
      compression: false
    };
  }
  
  // Destek kontrolü
  checkSupport() {
    try {
      const test = 'test';
      localStorage.setItem(test, test);
      localStorage.removeItem(test);
      return true;
    } catch (error) {
      return false;
    }
  }
  
  // Storage yöneticisini başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web Storage API desteklenmiyor');
    }
    
    try {
      // Storage event listener'ı ekle
      window.addEventListener('storage', this.handleStorageEvent.bind(this));
      
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Web Storage başlatma hatası:', error);
      return false;
    }
  }
  
  // Storage event handler
  handleStorageEvent(event) {
    const storageData = {
      key: event.key,
      oldValue: event.oldValue,
      newValue: event.newValue,
      url: event.url,
      storageArea: event.storageArea
    };
    
    console.log('Storage değişikliği:', storageData);
    
    // Event handler'ları çağır
    this.storageHandlers.forEach(handler => {
      try {
        handler(storageData);
      } catch (error) {
        console.error('Storage handler hatası:', error);
      }
    });
  }
  
  // Veri kaydet
  setItem(key, value, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const storage = this.getStorage(options.type);
      const prefixedKey = this.getPrefixedKey(key, options.prefix);
      
      // Veriyi hazırla
      const preparedValue = this.prepareValue(value, options);
      
      // Kaydet
      storage.setItem(prefixedKey, preparedValue);
      
      console.log('Veri kaydedildi:', prefixedKey);
      return true;
    } catch (error) {
      console.error('Veri kaydetme hatası:', error);
      throw error;
    }
  }
  
  // Veri al
  getItem(key, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const storage = this.getStorage(options.type);
      const prefixedKey = this.getPrefixedKey(key, options.prefix);
      
      const value = storage.getItem(prefixedKey);
      
      if (value === null) {
        return null;
      }
      
      // Veriyi işle
      const processedValue = this.processValue(value, options);
      
      return processedValue;
    } catch (error) {
      console.error('Veri alma hatası:', error);
      throw error;
    }
  }
  
  // Veri sil
  removeItem(key, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const storage = this.getStorage(options.type);
      const prefixedKey = this.getPrefixedKey(key, options.prefix);
      
      storage.removeItem(prefixedKey);
      
      console.log('Veri silindi:', prefixedKey);
      return true;
    } catch (error) {
      console.error('Veri silme hatası:', error);
      throw error;
    }
  }
  
  // Tüm verileri temizle
  clear(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const storage = this.getStorage(options.type);
      
      if (options.prefix) {
        // Prefix ile başlayan verileri sil
        const keysToRemove = [];
        for (let i = 0; i < storage.length; i++) {
          const key = storage.key(i);
          if (key && key.startsWith(options.prefix)) {
            keysToRemove.push(key);
          }
        }
        
        keysToRemove.forEach(key => storage.removeItem(key));
      } else {
        // Tüm verileri sil
        storage.clear();
      }
      
      console.log('Storage temizlendi');
      return true;
    } catch (error) {
      console.error('Storage temizleme hatası:', error);
      throw error;
    }
  }
  
  // Tüm anahtarları al
  getAllKeys(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const storage = this.getStorage(options.type);
      const keys = [];
      
      for (let i = 0; i < storage.length; i++) {
        const key = storage.key(i);
        if (key) {
          if (options.prefix) {
            if (key.startsWith(options.prefix)) {
              keys.push(key.replace(options.prefix, ''));
            }
          } else {
            keys.push(key);
          }
        }
      }
      
      return keys;
    } catch (error) {
      console.error('Anahtar alma hatası:', error);
      throw error;
    }
  }
  
  // Tüm verileri al
  getAllItems(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const keys = this.getAllKeys(options);
      const items = {};
      
      keys.forEach(key => {
        items[key] = this.getItem(key, options);
      });
      
      return items;
    } catch (error) {
      console.error('Tüm verileri alma hatası:', error);
      throw error;
    }
  }
  
  // Storage boyutu
  getStorageSize(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Storage başlatılmamış');
    }
    
    try {
      const storage = this.getStorage(options.type);
      let size = 0;
      
      for (let i = 0; i < storage.length; i++) {
        const key = storage.key(i);
        if (key) {
          const value = storage.getItem(key);
          size += key.length + (value ? value.length : 0);
        }
      }
      
      return size;
    } catch (error) {
      console.error('Storage boyutu alma hatası:', error);
      throw error;
    }
  }
  
  // Storage kullanılabilir mi
  isStorageAvailable(options = {}) {
    try {
      const storage = this.getStorage(options.type);
      const test = 'test';
      storage.setItem(test, test);
      storage.removeItem(test);
      return true;
    } catch (error) {
      return false;
    }
  }
  
  // Storage referansı al
  getStorage(type) {
    switch (type) {
      case this.storageTypes.LOCAL:
        return localStorage;
      case this.storageTypes.SESSION:
        return sessionStorage;
      default:
        throw new Error('Geçersiz storage tipi');
    }
  }
  
  // Prefix'li anahtar oluştur
  getPrefixedKey(key, prefix) {
    return prefix ? `${prefix}_${key}` : key;
  }
  
  // Veriyi hazırla
  prepareValue(value, options) {
    let preparedValue = value;
    
    // JSON'a çevir
    if (typeof value === 'object') {
      preparedValue = JSON.stringify(value);
    }
    
    // Sıkıştırma
    if (options.compression) {
      preparedValue = this.compress(preparedValue);
    }
    
    // Expiration ekle
    if (options.expiration) {
      const expirationData = {
        value: preparedValue,
        expiration: Date.now() + options.expiration
      };
      preparedValue = JSON.stringify(expirationData);
    }
    
    return preparedValue;
  }
  
  // Veriyi işle
  processValue(value, options) {
    let processedValue = value;
    
    // Expiration kontrolü
    if (options.expiration) {
      try {
        const expirationData = JSON.parse(processedValue);
        if (expirationData.expiration && Date.now() > expirationData.expiration) {
          return null; // Süresi dolmuş
        }
        processedValue = expirationData.value;
      } catch (error) {
        // Expiration verisi değil, normal veri
      }
    }
    
    // Sıkıştırma açma
    if (options.compression) {
      processedValue = this.decompress(processedValue);
    }
    
    // JSON'dan çevir
    try {
      processedValue = JSON.parse(processedValue);
    } catch (error) {
      // JSON değil, string olarak bırak
    }
    
    return processedValue;
  }
  
  // Basit sıkıştırma
  compress(data) {
    // Gerçek uygulamada daha gelişmiş sıkıştırma kullanın
    return btoa(data);
  }
  
  // Basit sıkıştırma açma
  decompress(data) {
    try {
      return atob(data);
    } catch (error) {
      return data;
    }
  }
  
  // Event handler kaydet
  registerHandler(handler) {
    this.storageHandlers.set(handler, handler);
  }
  
  // Event handler kaldır
  unregisterHandler(handler) {
    this.storageHandlers.delete(handler);
  }
  
  // Durum bilgisi
  getState() {
    return {
      isSupported: this.isSupported,
      isInitialized: this.isInitialized,
      localStorageSize: this.getStorageSize({ type: this.storageTypes.LOCAL }),
      sessionStorageSize: this.getStorageSize({ type: this.storageTypes.SESSION })
    };
  }
  
  // Destek durumu
  getSupportState() {
    return this.isSupported;
  }
  
  // Kaynakları temizle
  cleanup() {
    window.removeEventListener('storage', this.handleStorageEvent.bind(this));
    this.storageHandlers.clear();
    this.isInitialized = false;
  }
}

// Web Storage utility fonksiyonları
class WebStorageUtils {
  // Veri tipi doğrulama
  static validateDataType(value) {
    const validTypes = ['string', 'number', 'boolean', 'object', 'array'];
    return validTypes.includes(typeof value) || Array.isArray(value);
  }
  
  // Anahtar doğrulama
  static validateKey(key) {
    if (typeof key !== 'string') {
      return false;
    }
    
    if (key.length === 0) {
      return false;
    }
    
    if (key.length > 100) {
      return false;
    }
    
    return true;
  }
  
  // Veri boyutu kontrolü
  static validateDataSize(data, maxSize = 5 * 1024 * 1024) { // 5MB
    const dataString = typeof data === 'string' ? data : JSON.stringify(data);
    return dataString.length <= maxSize;
  }
  
  // Expiration doğrulama
  static validateExpiration(expiration) {
    return typeof expiration === 'number' && expiration > 0;
  }
  
  // Storage seçenekleri doğrulama
  static validateStorageOptions(options) {
    const errors = [];
    
    if (options.type && !['localStorage', 'sessionStorage'].includes(options.type)) {
      errors.push('Geçersiz storage tipi');
    }
    
    if (options.prefix && typeof options.prefix !== 'string') {
      errors.push('Prefix string olmalı');
    }
    
    if (options.expiration && !this.validateExpiration(options.expiration)) {
      errors.push('Geçersiz expiration değeri');
    }
    
    return {
      valid: errors.length === 0,
      errors: errors
    };
  }
  
  // Veri şifreleme
  static encrypt(data, key) {
    // Basit şifreleme (gerçek uygulamada daha güvenli yöntemler kullanın)
    try {
      const dataString = typeof data === 'string' ? data : JSON.stringify(data);
      return btoa(dataString + key);
    } catch (error) {
      throw new Error('Şifreleme hatası');
    }
  }
  
  // Veri şifre çözme
  static decrypt(encryptedData, key) {
    try {
      const decrypted = atob(encryptedData);
      return decrypted.replace(key, '');
    } catch (error) {
      throw new Error('Şifre çözme hatası');
    }
  }
  
  // Veri sıkıştırma
  static compress(data) {
    // Gerçek uygulamada daha gelişmiş sıkıştırma kullanın
    return btoa(JSON.stringify(data));
  }
  
  // Veri açma
  static decompress(compressedData) {
    try {
      return JSON.parse(atob(compressedData));
    } catch (error) {
      throw new Error('Veri açma hatası');
    }
  }
  
  // Veri yedekleme
  static backupStorage(type = 'localStorage') {
    const storage = type === 'localStorage' ? localStorage : sessionStorage;
    const backup = {};
    
    for (let i = 0; i < storage.length; i++) {
      const key = storage.key(i);
      if (key) {
        backup[key] = storage.getItem(key);
      }
    }
    
    return backup;
  }
  
  // Veri geri yükleme
  static restoreStorage(backup, type = 'localStorage') {
    const storage = type === 'localStorage' ? localStorage : sessionStorage;
    
    // Mevcut verileri temizle
    storage.clear();
    
    // Yedekten geri yükle
    Object.entries(backup).forEach(([key, value]) => {
      storage.setItem(key, value);
    });
  }
  
  // Storage istatistikleri
  static getStorageStats(type = 'localStorage') {
    const storage = type === 'localStorage' ? localStorage : sessionStorage;
    let totalSize = 0;
    let itemCount = 0;
    
    for (let i = 0; i < storage.length; i++) {
      const key = storage.key(i);
      if (key) {
        const value = storage.getItem(key);
        totalSize += key.length + (value ? value.length : 0);
        itemCount++;
      }
    }
    
    return {
      itemCount,
      totalSize,
      averageSize: itemCount > 0 ? totalSize / itemCount : 0
    };
  }
  
  // Storage temizleme
  static cleanupStorage(type = 'localStorage', maxAge = 30 * 24 * 60 * 60 * 1000) { // 30 gün
    const storage = type === 'localStorage' ? localStorage : sessionStorage;
    const now = Date.now();
    const keysToRemove = [];
    
    for (let i = 0; i < storage.length; i++) {
      const key = storage.key(i);
      if (key) {
        const value = storage.getItem(key);
        try {
          const data = JSON.parse(value);
          if (data.timestamp && (now - data.timestamp) > maxAge) {
            keysToRemove.push(key);
          }
        } catch (error) {
          // JSON değil, timestamp yok
        }
      }
    }
    
    keysToRemove.forEach(key => storage.removeItem(key));
    return keysToRemove.length;
  }
}
```

**Alternatifler:**
- **IndexedDB**: IndexedDB API
- **WebSQL**: WebSQL API
- **Cookies**: Cookie API
- **Cache API**: Cache API
- **File System Access**: File System Access API

**Ne Zaman Kullanılmalı?**
- ✅ Veri kalıcılığı gerektiğinde
- ✅ Kullanıcı tercihleri gerektiğinde
- ✅ Offline depolama gerektiğinde
- ✅ Performans gerektiğinde
- ✅ Önbellekleme gerektiğinde
- ✅ Durum yönetimi gerektiğinde
- ✅ Sekmeler arası iletişim gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Büyük veri setleri gerektiğinde
- ❌ Karmaşık sorgular gerektiğinde
- ❌ Güvenlik kritik veriler için
- ❌ Çok büyük dosyalar için

**Kullanılmazsa Ne Olur?**
- **No Data Persistence**: Veri kalıcılığı olmaz
- **No User Preferences**: Kullanıcı tercihleri olmaz
- **No Offline Storage**: Offline depolama olmaz
- **Poor Performance**: Performans kötü olur
- **No Caching**: Önbellekleme olmaz

**Modern Alternatifler:**
- **Web Storage API**: Web Storage API
- **IndexedDB**: IndexedDB API
- **Cache API**: Cache API
- **File System Access**: File System Access API
- **WebSQL**: WebSQL API

### Web Streams API

**Web Streams API Nedir?**
Web Streams API, web uygulamalarında büyük veri setlerini akış halinde işlemesini sağlayan bir web API'sidir. ReadableStream, WritableStream, TransformStream ve diğer stream türlerini destekler. Verileri parça parça işleyerek bellek kullanımını optimize eder ve büyük dosyaların, ağ isteklerinin ve veri işleme operasyonlarının etkili bir şekilde yönetilmesini sağlar. Asenkron veri işleme, backpressure yönetimi ve pipe işlemleri için kritik öneme sahiptir.

**Neden Kullanılır?**
- **Large Data Processing**: Büyük veri işleme
- **Memory Optimization**: Bellek optimizasyonu
- **Backpressure Management**: Backpressure yönetimi
- **Asynchronous Processing**: Asenkron işleme
- **Data Transformation**: Veri dönüştürme
- **Network Streaming**: Ağ akışı
- **File Processing**: Dosya işleme

**Temel Kullanım Örnekleri:**

```javascript
// ReadableStream
const readable = new ReadableStream({
  start(controller) {
    controller.enqueue('Veri 1');
    controller.enqueue('Veri 2');
    controller.close();
  }
});

// WritableStream
const writable = new WritableStream({
  write(chunk) {
    console.log('Yazılan veri:', chunk);
  }
});

// TransformStream
const transform = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});

// Web Streams yöneticisi
class WebStreamsManager {
  constructor() {
    this.isSupported = this.checkSupport();
    this.streams = new Map();
    this.isInitialized = false;
    this.defaultOptions = {
      highWaterMark: 1,
      size: () => 1,
      autoAllocateChunkSize: undefined
    };
  }
  
  // Destek kontrolü
  checkSupport() {
    return typeof ReadableStream !== 'undefined' && 
           typeof WritableStream !== 'undefined' && 
           typeof TransformStream !== 'undefined';
  }
  
  // Streams yöneticisini başlat
  async initialize() {
    if (!this.isSupported) {
      throw new Error('Web Streams API desteklenmiyor');
    }
    
    try {
      this.isInitialized = true;
      return true;
    } catch (error) {
      console.error('Web Streams başlatma hatası:', error);
      return false;
    }
  }
  
  // ReadableStream oluştur
  createReadableStream(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamOptions = { ...this.defaultOptions, ...options };
      const stream = new ReadableStream({
        start: streamOptions.start || (() => {}),
        pull: streamOptions.pull || (() => {}),
        cancel: streamOptions.cancel || (() => {}),
        type: streamOptions.type || undefined,
        autoAllocateChunkSize: streamOptions.autoAllocateChunkSize
      }, {
        highWaterMark: streamOptions.highWaterMark,
        size: streamOptions.size
      });
      
      const streamId = this.generateStreamId();
      this.streams.set(streamId, { stream, type: 'readable' });
      
      console.log('ReadableStream oluşturuldu:', streamId);
      return { stream, id: streamId };
    } catch (error) {
      console.error('ReadableStream oluşturma hatası:', error);
      throw error;
    }
  }
  
  // WritableStream oluştur
  createWritableStream(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamOptions = { ...this.defaultOptions, ...options };
      const stream = new WritableStream({
        start: streamOptions.start || (() => {}),
        write: streamOptions.write || (() => {}),
        close: streamOptions.close || (() => {}),
        abort: streamOptions.abort || (() => {})
      }, {
        highWaterMark: streamOptions.highWaterMark,
        size: streamOptions.size
      });
      
      const streamId = this.generateStreamId();
      this.streams.set(streamId, { stream, type: 'writable' });
      
      console.log('WritableStream oluşturuldu:', streamId);
      return { stream, id: streamId };
    } catch (error) {
      console.error('WritableStream oluşturma hatası:', error);
      throw error;
    }
  }
  
  // TransformStream oluştur
  createTransformStream(options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamOptions = { ...this.defaultOptions, ...options };
      const stream = new TransformStream({
        start: streamOptions.start || (() => {}),
        transform: streamOptions.transform || ((chunk, controller) => {
          controller.enqueue(chunk);
        }),
        flush: streamOptions.flush || (() => {})
      }, {
        readableType: streamOptions.readableType || undefined,
        writableType: streamOptions.writableType || undefined,
        readableHighWaterMark: streamOptions.readableHighWaterMark || 1,
        writableHighWaterMark: streamOptions.writableHighWaterMark || 1,
        readableSize: streamOptions.readableSize || (() => 1),
        writableSize: streamOptions.writableSize || (() => 1)
      });
      
      const streamId = this.generateStreamId();
      this.streams.set(streamId, { stream, type: 'transform' });
      
      console.log('TransformStream oluşturuldu:', streamId);
      return { stream, id: streamId };
    } catch (error) {
      console.error('TransformStream oluşturma hatası:', error);
      throw error;
    }
  }
  
  // Stream pipe işlemi
  async pipeStreams(sourceId, destinationId, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const sourceStream = this.streams.get(sourceId);
      const destinationStream = this.streams.get(destinationId);
      
      if (!sourceStream || !destinationStream) {
        throw new Error('Stream bulunamadı');
      }
      
      if (sourceStream.type !== 'readable') {
        throw new Error('Kaynak stream readable olmalı');
      }
      
      if (destinationStream.type !== 'writable') {
        throw new Error('Hedef stream writable olmalı');
      }
      
      const pipeOptions = {
        preventClose: options.preventClose || false,
        preventAbort: options.preventAbort || false,
        preventCancel: options.preventCancel || false,
        signal: options.signal || undefined
      };
      
      await sourceStream.stream.pipeTo(destinationStream.stream, pipeOptions);
      
      console.log('Stream pipe işlemi tamamlandı');
      return true;
    } catch (error) {
      console.error('Stream pipe hatası:', error);
      throw error;
    }
  }
  
  // Stream pipeThrough işlemi
  pipeThroughStream(sourceId, transformId, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const sourceStream = this.streams.get(sourceId);
      const transformStream = this.streams.get(transformId);
      
      if (!sourceStream || !transformStream) {
        throw new Error('Stream bulunamadı');
      }
      
      if (sourceStream.type !== 'readable') {
        throw new Error('Kaynak stream readable olmalı');
      }
      
      if (transformStream.type !== 'transform') {
        throw new Error('Transform stream transform olmalı');
      }
      
      const pipeOptions = {
        preventClose: options.preventClose || false,
        preventAbort: options.preventAbort || false,
        preventCancel: options.preventCancel || false,
        signal: options.signal || undefined
      };
      
      const resultStream = sourceStream.stream.pipeThrough(transformStream.stream, pipeOptions);
      
      const resultId = this.generateStreamId();
      this.streams.set(resultId, { stream: resultStream, type: 'readable' });
      
      console.log('Stream pipeThrough işlemi tamamlandı');
      return { stream: resultStream, id: resultId };
    } catch (error) {
      console.error('Stream pipeThrough hatası:', error);
      throw error;
    }
  }
  
  // Stream veri okuma
  async readFromStream(streamId, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamData = this.streams.get(streamId);
      
      if (!streamData || streamData.type !== 'readable') {
        throw new Error('Readable stream bulunamadı');
      }
      
      const reader = streamData.stream.getReader();
      const chunks = [];
      
      try {
        while (true) {
          const { done, value } = await reader.read();
          
          if (done) {
            break;
          }
          
          chunks.push(value);
          
          if (options.maxChunks && chunks.length >= options.maxChunks) {
            break;
          }
        }
      } finally {
        reader.releaseLock();
      }
      
      console.log('Stream veri okuma tamamlandı:', chunks.length, 'chunk');
      return chunks;
    } catch (error) {
      console.error('Stream veri okuma hatası:', error);
      throw error;
    }
  }
  
  // Stream veri yazma
  async writeToStream(streamId, data, options = {}) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamData = this.streams.get(streamId);
      
      if (!streamData || streamData.type !== 'writable') {
        throw new Error('Writable stream bulunamadı');
      }
      
      const writer = streamData.stream.getWriter();
      
      try {
        if (Array.isArray(data)) {
          for (const chunk of data) {
            await writer.write(chunk);
          }
        } else {
          await writer.write(data);
        }
        
        if (options.close) {
          await writer.close();
        }
      } finally {
        writer.releaseLock();
      }
      
      console.log('Stream veri yazma tamamlandı');
      return true;
    } catch (error) {
      console.error('Stream veri yazma hatası:', error);
      throw error;
    }
  }
  
  // Stream iptal et
  async cancelStream(streamId, reason = 'User cancelled') {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamData = this.streams.get(streamId);
      
      if (!streamData) {
        throw new Error('Stream bulunamadı');
      }
      
      if (streamData.type === 'readable') {
        const reader = streamData.stream.getReader();
        await reader.cancel(reason);
        reader.releaseLock();
      } else if (streamData.type === 'writable') {
        const writer = streamData.stream.getWriter();
        await writer.abort(reason);
        writer.releaseLock();
      }
      
      console.log('Stream iptal edildi:', streamId);
      return true;
    } catch (error) {
      console.error('Stream iptal hatası:', error);
      throw error;
    }
  }
  
  // Stream kapat
  async closeStream(streamId) {
    if (!this.isInitialized) {
      throw new Error('Web Streams başlatılmamış');
    }
    
    try {
      const streamData = this.streams.get(streamId);
      
      if (!streamData) {
        throw new Error('Stream bulunamadı');
      }
      
      if (streamData.type === 'writable') {
        const writer = streamData.stream.getWriter();
        await writer.close();
        writer.releaseLock();
      }
      
      console.log('Stream kapatıldı:', streamId);
      return true;
    } catch (error) {
      console.error('Stream kapatma hatası:', error);
      throw error;
    }
  }
  
  // Stream durumu
  getStreamState(streamId) {
    const streamData = this.streams.get(streamId);
    
    if (!streamData) {
      return null;
    }
    
    return {
      id: streamId,
      type: streamData.type,
      locked: streamData.stream.locked || false,
      desiredSize: streamData.stream.desiredSize || null
    };
  }
  
  // Tüm stream'leri listele
  getAllStreams() {
    const streams = [];
    
    this.streams.forEach((streamData, id) => {
      streams.push(this.getStreamState(id));
    });
    
    return streams;
  }
  
  // Stream ID oluştur
  generateStreamId() {
    return `stream_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  // Stream sil
  removeStream(streamId) {
    this.streams.delete(streamId);
    console.log('Stream silindi:', streamId);
  }
  
  // Durum bilgisi
  getState() {
    return {
      isSupported: this.isSupported,
      isInitialized: this.isInitialized,
      streamCount: this.streams.size,
      streams: this.getAllStreams()
    };
  }
  
  // Destek durumu
  getSupportState() {
    return this.isSupported;
  }
  
  // Kaynakları temizle
  cleanup() {
    this.streams.forEach((streamData, id) => {
      this.cancelStream(id);
    });
    this.streams.clear();
    this.isInitialized = false;
  }
}

// Web Streams utility fonksiyonları
class WebStreamsUtils {
  // Stream seçenekleri doğrulama
  static validateStreamOptions(options) {
    const errors = [];
    
    if (options.highWaterMark && (typeof options.highWaterMark !== 'number' || options.highWaterMark < 0)) {
      errors.push('highWaterMark pozitif sayı olmalı');
    }
    
    if (options.size && typeof options.size !== 'function') {
      errors.push('size fonksiyon olmalı');
    }
    
    if (options.autoAllocateChunkSize && typeof options.autoAllocateChunkSize !== 'number') {
      errors.push('autoAllocateChunkSize sayı olmalı');
    }
    
    return {
      valid: errors.length === 0,
      errors: errors
    };
  }
  
  // Veri tipi doğrulama
  static validateDataType(data) {
    return data !== null && data !== undefined;
  }
  
  // Chunk boyutu hesapla
  static calculateChunkSize(chunk) {
    if (typeof chunk === 'string') {
      return chunk.length;
    } else if (chunk instanceof ArrayBuffer) {
      return chunk.byteLength;
    } else if (chunk instanceof Uint8Array) {
      return chunk.length;
    } else if (typeof chunk === 'object') {
      return JSON.stringify(chunk).length;
    }
    return 1;
  }
  
  // Veri dönüştürme
  static transformData(data, transformType) {
    switch (transformType) {
      case 'uppercase':
        return typeof data === 'string' ? data.toUpperCase() : data;
      case 'lowercase':
        return typeof data === 'string' ? data.toLowerCase() : data;
      case 'json':
        return JSON.stringify(data);
      case 'parse':
        try {
          return JSON.parse(data);
        } catch (error) {
          return data;
        }
      default:
        return data;
    }
  }
  
  // Stream buffer oluştur
  static createBuffer(size = 1024) {
    return new ArrayBuffer(size);
  }
  
  // Stream chunk oluştur
  static createChunk(data, type = 'string') {
    switch (type) {
      case 'string':
        return data;
      case 'arraybuffer':
        return new TextEncoder().encode(data).buffer;
      case 'uint8array':
        return new TextEncoder().encode(data);
      case 'json':
        return JSON.stringify(data);
      default:
        return data;
    }
  }
  
  // Stream chunk işle
  static processChunk(chunk, type = 'string') {
    switch (type) {
      case 'string':
        return chunk;
      case 'arraybuffer':
        return new TextDecoder().decode(chunk);
      case 'uint8array':
        return new TextDecoder().decode(chunk);
      case 'json':
        try {
          return JSON.parse(chunk);
        } catch (error) {
          return chunk;
        }
      default:
        return chunk;
    }
  }
  
  // Stream istatistikleri
  static getStreamStats() {
    return {
      totalStreams: 0,
      activeStreams: 0,
      totalBytesProcessed: 0,
      averageChunkSize: 0
    };
  }
  
  // Stream performans ölçümü
  static measureStreamPerformance(streamId, startTime, endTime, bytesProcessed) {
    const duration = endTime - startTime;
    const throughput = bytesProcessed / (duration / 1000); // bytes per second
    
    return {
      streamId,
      duration,
      bytesProcessed,
      throughput,
      averageChunkSize: bytesProcessed / 1000 // varsayılan chunk sayısı
    };
  }
  
  // Stream hata yönetimi
  static handleStreamError(error, streamId) {
    console.error(`Stream ${streamId} hatası:`, error);
    
    return {
      streamId,
      error: error.message,
      timestamp: Date.now(),
      type: error.name || 'UnknownError'
    };
  }
  
  // Stream temizleme
  static cleanupStream(stream) {
    try {
      if (stream.locked) {
        const reader = stream.getReader();
        reader.cancel();
        reader.releaseLock();
      }
    } catch (error) {
      console.error('Stream temizleme hatası:', error);
    }
  }
}
```

**Alternatifler:**
- **Fetch API**: Fetch API
- **XMLHttpRequest**: XMLHttpRequest
- **WebSocket**: WebSocket API
- **Server-Sent Events**: Server-Sent Events
- **File API**: File API

**Ne Zaman Kullanılmalı?**
- ✅ Büyük veri işleme gerektiğinde
- ✅ Bellek optimizasyonu gerektiğinde
- ✅ Backpressure yönetimi gerektiğinde
- ✅ Asenkron işleme gerektiğinde
- ✅ Veri dönüştürme gerektiğinde
- ✅ Ağ akışı gerektiğinde
- ✅ Dosya işleme gerektiğinde

**Ne Zaman Kullanılmamalı?**
- ❌ Küçük veri setleri için
- ❌ Basit veri işleme için
- ❌ Eski tarayıcı desteği gerektiğinde
- ❌ Senkron işleme gerektiğinde

**Kullanılmazsa Ne Olur?**
- **No Large Data Processing**: Büyük veri işleme olmaz
- **No Memory Optimization**: Bellek optimizasyonu olmaz
- **No Backpressure Management**: Backpressure yönetimi olmaz
- **Poor Asynchronous Processing**: Asenkron işleme kötü olur
- **No Data Transformation**: Veri dönüştürme olmaz

**Modern Alternatifler:**
- **Web Streams API**: Web Streams API
- **Fetch API**: Fetch API
- **WebSocket**: WebSocket API
- **Server-Sent Events**: Server-Sent Events
- **File API**: File API

### Web Workers API
Web işçileri için API - Arka planda JavaScript kodu çalıştırma.

## Web Workers Nedir?

Web Workers, JavaScript'in **çok iş parçacıklı (multi-threaded)** programlama yapmasını sağlayan bir API'dir. Normalde JavaScript **tek iş parçacıklı (single-threaded)** çalışır, yani aynı anda sadece bir işlem yapabilir. Web Workers sayesinde ağır işlemleri arka planda çalıştırabilir, ana thread'i (UI thread) bloke etmeden hesaplamalar yapabilirsiniz.

## Neden Kullanılır?

### 1. **Ana Thread'i Bloke Etmemek**
```javascript
// ❌ Kötü: Ana thread'i bloke eder
function heavyCalculation() {
  let result = 0;
  for (let i = 0; i < 1000000000; i++) {
    result += i;
  }
  return result;
}

// Bu işlem sırasında sayfa donar, kullanıcı tıklayamaz
const result = heavyCalculation();
```

```javascript
// ✅ İyi: Web Worker kullanarak
// main.js
const worker = new Worker('calculation-worker.js');
worker.postMessage({ type: 'CALCULATE', data: 1000000000 });
worker.onmessage = (event) => {
  console.log('Sonuç:', event.data.result);
  // UI donmaz, kullanıcı etkileşime devam edebilir
};

// calculation-worker.js
self.onmessage = (event) => {
  const { type, data } = event.data;
  if (type === 'CALCULATE') {
    let result = 0;
    for (let i = 0; i < data; i++) {
      result += i;
    }
    self.postMessage({ result });
  }
};
```

### 2. **Performans Artışı**
```javascript
// Ana thread'de ağır işlem
function processLargeData(data) {
  const start = performance.now();
  
  // Büyük veri setini işle
  const processed = data.map(item => {
    // Karmaşık hesaplamalar
    return complexCalculation(item);
  });
  
  const end = performance.now();
  console.log(`İşlem süresi: ${end - start}ms`);
  return processed;
}

// Web Worker ile
const worker = new Worker('data-processor.js');
worker.postMessage({ data: largeDataSet });
worker.onmessage = (event) => {
  console.log('İşlem tamamlandı:', event.data);
};
```

## Web Workers Türleri

### 1. **Dedicated Workers (Özel İşçiler)**
```javascript
// Ana thread
const worker = new Worker('worker.js');

// Worker ile iletişim
worker.postMessage('Merhaba worker!');
worker.onmessage = (event) => {
  console.log('Worker\'dan gelen:', event.data);
};

// Worker'ı sonlandır
worker.terminate();

// worker.js
self.onmessage = (event) => {
  console.log('Ana thread\'den gelen:', event.data);
  
  // Ağır işlem
  const result = performHeavyTask(event.data);
  
  // Sonucu gönder
  self.postMessage(result);
};

function performHeavyTask(data) {
  // Karmaşık hesaplamalar
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}
```

### 2. **Shared Workers (Paylaşımlı İşçiler)**
```javascript
// Birden fazla tab/window arasında paylaşılabilir
const sharedWorker = new SharedWorker('shared-worker.js');

sharedWorker.port.postMessage('Merhaba shared worker!');
sharedWorker.port.onmessage = (event) => {
  console.log('Shared worker\'dan gelen:', event.data);
};

// shared-worker.js
let connections = 0;

self.onconnect = (event) => {
  const port = event.ports[0];
  connections++;
  
  port.onmessage = (event) => {
    console.log('Bağlantı sayısı:', connections);
    port.postMessage(`Merhaba! Toplam ${connections} bağlantı var.`);
  };
};
```

### 3. **Service Workers (Servis İşçileri)**
```javascript
// PWA ve offline çalışma için
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(registration => {
      console.log('Service Worker kaydedildi:', registration);
    })
    .catch(error => {
      console.log('Service Worker kaydı başarısız:', error);
    });
}

// sw.js
self.addEventListener('install', event => {
  console.log('Service Worker yüklendi');
});

self.addEventListener('fetch', event => {
  // Ağ isteklerini yakala ve cache'den yanıtla
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        return response || fetch(event.request);
      })
  );
});
```

## Pratik Kullanım Örnekleri

### 1. **Görüntü İşleme**
```javascript
// main.js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

const worker = new Worker('image-processor.js');
worker.postMessage({ imageData, filter: 'blur' });

worker.onmessage = (event) => {
  const processedImageData = event.data;
  ctx.putImageData(processedImageData, 0, 0);
};

// image-processor.js
self.onmessage = (event) => {
  const { imageData, filter } = event.data;
  
  // Görüntü filtreleme işlemi
  const processedData = applyFilter(imageData, filter);
  
  self.postMessage(processedData);
};

function applyFilter(imageData, filter) {
  const data = imageData.data;
  
  switch (filter) {
    case 'blur':
      return applyBlurFilter(data);
    case 'grayscale':
      return applyGrayscaleFilter(data);
    case 'invert':
      return applyInvertFilter(data);
    default:
      return imageData;
  }
}
```

### 2. **Veri Analizi**
```javascript
// main.js
const worker = new Worker('data-analyzer.js');

// Büyük veri setini analiz et
const largeDataset = generateLargeDataset();
worker.postMessage({ 
  type: 'ANALYZE', 
  data: largeDataset,
  analysisType: 'statistical'
});

worker.onmessage = (event) => {
  const analysis = event.data;
  displayResults(analysis);
};

// data-analyzer.js
self.onmessage = (event) => {
  const { type, data, analysisType } = event.data;
  
  if (type === 'ANALYZE') {
    const result = performAnalysis(data, analysisType);
    self.postMessage(result);
  }
};

function performAnalysis(data, type) {
  switch (type) {
    case 'statistical':
      return calculateStatistics(data);
    case 'clustering':
      return performClustering(data);
    case 'regression':
      return performRegression(data);
    default:
      return null;
  }
}

function calculateStatistics(data) {
  const mean = data.reduce((sum, val) => sum + val, 0) / data.length;
  const variance = data.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0) / data.length;
  const stdDev = Math.sqrt(variance);
  
  return {
    mean,
    variance,
    standardDeviation: stdDev,
    min: Math.min(...data),
    max: Math.max(...data)
  };
}
```

### 3. **Kriptografi İşlemleri**
```javascript
// main.js
const worker = new Worker('crypto-worker.js');

const password = 'mySecretPassword';
const data = 'Sensitive data to encrypt';

worker.postMessage({
  type: 'ENCRYPT',
  password,
  data
});

worker.onmessage = (event) => {
  const { type, result } = event.data;
  if (type === 'ENCRYPT_COMPLETE') {
    console.log('Şifrelenmiş veri:', result);
  }
};

// crypto-worker.js
self.onmessage = (event) => {
  const { type, password, data } = event.data;
  
  if (type === 'ENCRYPT') {
    // Ağır kriptografi işlemi
    const encrypted = performEncryption(data, password);
    self.postMessage({
      type: 'ENCRYPT_COMPLETE',
      result: encrypted
    });
  }
};

function performEncryption(data, password) {
  // Gerçek şifreleme algoritması burada olacak
  // Bu örnek basit bir XOR şifreleme
  let encrypted = '';
  for (let i = 0; i < data.length; i++) {
    const charCode = data.charCodeAt(i) ^ password.charCodeAt(i % password.length);
    encrypted += String.fromCharCode(charCode);
  }
  return btoa(encrypted); // Base64 encode
}
```

## Alternatifler

### 1. **setTimeout/setInterval (Sınırlı)**
```javascript
// ❌ Sınırlı alternatif
function processInChunks(data, chunkSize = 1000) {
  let index = 0;
  
  function processChunk() {
    const chunk = data.slice(index, index + chunkSize);
    
    // Chunk'ı işle
    processChunkData(chunk);
    
    index += chunkSize;
    
    if (index < data.length) {
      setTimeout(processChunk, 0); // Ana thread'i serbest bırak
    }
  }
  
  processChunk();
}
```

### 2. **requestAnimationFrame (UI İşlemleri için)**
```javascript
// ✅ Animasyonlar için uygun
function animateElement(element) {
  let position = 0;
  
  function animate() {
    position += 1;
    element.style.left = position + 'px';
    
    if (position < 500) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}
```

### 3. **WebAssembly (WASM)**
```javascript
// ✅ Çok yüksek performans için
async function loadWasmModule() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('heavy-computation.wasm')
  );
  
  const result = wasmModule.instance.exports.heavyCalculation(1000000);
  return result;
}
```

## Kullanmalı mıyız?

### ✅ **Kullanın:**
- Büyük veri setlerini işlerken
- Karmaşık matematiksel hesaplamalar yaparken
- Görüntü/video işleme yaparken
- Kriptografi işlemleri yaparken
- Ağ isteklerini arka planda yaparken
- Real-time veri analizi yaparken

### ❌ **Kullanmayın:**
- Basit DOM manipülasyonları için
- Küçük hesaplamalar için
- UI güncellemeleri için
- Çok sık iletişim gereken işlemler için

## Kullanmazsak Ne Olur?

### 1. **UI Donması**
```javascript
// ❌ Ana thread bloke olur
function heavyTask() {
  let result = 0;
  for (let i = 0; i < 100000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

// Bu sırada kullanıcı hiçbir şey yapamaz
const result = heavyTask();
```

### 2. **Kötü Kullanıcı Deneyimi**
- Sayfa donar
- Butonlar çalışmaz
- Scroll yapılamaz
- Animasyonlar durur

### 3. **Performans Sorunları**
- Tarayıcı "Sayfa yanıt vermiyor" uyarısı
- Düşük FPS
- Gecikmeli etkileşimler

## Modern Alternatifler

### 1. **Web Streams API**
```javascript
// Büyük veri akışları için
const stream = new ReadableStream({
  start(controller) {
    // Veri üret
    for (let i = 0; i < 1000000; i++) {
      controller.enqueue(processData(i));
    }
    controller.close();
  }
});
```

### 2. **OffscreenCanvas**
```javascript
// Canvas işlemlerini arka planda yap
const offscreen = new OffscreenCanvas(800, 600);
const ctx = offscreen.getContext('2d');

// Ağır canvas işlemleri
drawComplexGraphics(ctx);

// Sonucu ana thread'e aktar
const imageBitmap = offscreen.transferToImageBitmap();
```

### 3. **WebAssembly**
```javascript
// C/C++/Rust kodunu JavaScript'te çalıştır
const wasmModule = await WebAssembly.instantiateStreaming(
  fetch('computation.wasm')
);

const result = wasmModule.instance.exports.heavyComputation();
```

## Özet

**Web Workers API**, modern web uygulamalarında **kritik öneme sahip** bir teknolojidir. Ağır işlemleri arka planda çalıştırarak:

- ✅ Ana thread'i bloke etmez
- ✅ Kullanıcı deneyimini iyileştirir
- ✅ Performansı artırır
- ✅ Responsive uygulamalar yaratır

**Kullanın** - özellikle veri işleme, görüntü işleme, kriptografi ve büyük hesaplamalar için. **Alternatifleri** de mevcuttur ama Web Workers en yaygın ve güvenilir çözümdür.

```javascript
// Basit Web Worker örneği
const worker = new Worker('worker.js');

worker.postMessage('Merhaba worker!');
worker.onmessage = event => {
  console.log('Worker\'dan gelen:', event.data);
};

// worker.js
self.onmessage = event => {
  console.log('Ana thread\'den gelen:', event.data);
  self.postMessage('Merhaba ana thread!');
};
```

### WebXR API
Sanal ve artırılmış gerçeklik için API.

```javascript
if ('xr' in navigator) {
  const isSupported = await navigator.xr.isSessionSupported('immersive-vr');
  
  if (isSupported) {
    const session = await navigator.xr.requestSession('immersive-vr');
    
    session.addEventListener('end', () => {
      console.log('XR oturumu sona erdi');
    });
  }
}
```

---

## X - Bölümü

### XMLHttpRequest API
HTTP istekleri için API.

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data', true);

xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status === 200) {
    const data = JSON.parse(xhr.responseText);
    console.log('Veri:', data);
  }
};

xhr.send();
```

---

## Tarayıcı Uyumluluğu

Web API'lerinin tarayıcı desteği değişkenlik gösterir. Geliştirme yaparken:

1. **Can I Use** (caniuse.com) sitesini kontrol edin
2. **Feature Detection** kullanın
3. **Polyfill** kütüphanelerini değerlendirin
4. **Progressive Enhancement** yaklaşımını benimseyin

```javascript
// Feature detection örneği
if ('geolocation' in navigator) {
  // Geolocation API destekleniyor
  navigator.geolocation.getCurrentPosition(/* ... */);
} else {
  // Fallback çözüm
  console.log('Geolocation desteklenmiyor');
}
```

## Güvenlik ve İzinler

Çoğu Web API'si güvenlik nedeniyle:

- **HTTPS** gerektirir
- **Kullanıcı izni** ister
- **Same-origin policy** uygular
- **Content Security Policy** ile kontrol edilir

## Performans Optimizasyonu

Web API'lerini kullanırken:

1. **Lazy loading** uygulayın
2. **Debouncing/Throttling** kullanın
3. **Memory leaks** önleyin
4. **Event listeners** temizleyin

```javascript
// Event listener temizleme
const observer = new IntersectionObserver(callback);
observer.observe(element);

// Temizleme
observer.disconnect();
```

## Sonuç

Web API'leri modern web geliştirmenin temel taşlarıdır. Bu API'ler sayesinde:

- **Zengin kullanıcı deneyimleri** oluşturabilirsiniz
- **Performanslı uygulamalar** geliştirebilirsiniz
- **Çoklu platform desteği** sağlayabilirsiniz
- **Modern web standartlarına** uygun çalışabilirsiniz

Her API'nin kendi kullanım alanı ve sınırlamaları vardır. Projenizin ihtiyaçlarına göre doğru API'leri seçmek ve bunları etkili bir şekilde kullanmak başarılı web uygulamaları geliştirmenin anahtarıdır.
