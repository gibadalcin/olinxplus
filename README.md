# Olinx Plus Mobile App

<div align="center">

**Aplicativo de Realidade Aumentada com Reconhecimento Visual de Logos**

[![Expo](https://img.shields.io/badge/Expo-54-000020.svg?logo=expo&logoColor=white)](https://expo.dev/)
[![React Native](https://img.shields.io/badge/React%20Native-0.76.5-61DAFB.svg?logo=react&logoColor=white)](https://reactnative.dev/)
[![React](https://img.shields.io/badge/React-19-61DAFB.svg?logo=react&logoColor=white)](https://react.dev/)
[![Three.js](https://img.shields.io/badge/Three.js-R3F-000000.svg?logo=three.js&logoColor=white)](https://threejs.org/)

Experiência AR imersiva com reconhecimento de logos via CLIP e visualização 3D nativa

[Documentação](https://github.com/gibadalcin/olinxplus-docs) • [Backend API](https://github.com/gibadalcin/olinxplus-backend) • [Admin UI](https://github.com/gibadalcin/olinxplus-adminui)

</div>

---

## 📋 Visão Geral

Olinx Plus Mobile App é um aplicativo cross-platform (iOS/Android) que permite aos usuários:

- 📷 **Captura Guiada**: Marcadores visuais (300x250px) para enquadramento preciso de logos
- 🔍 **Reconhecimento Visual IA**: Identificação de marcas usando CLIP + FAISS (~85-90% taxa de sucesso)
- 🌟 **AR Nativo**: Realidade aumentada com Expo GL + React Three Fiber (ARCore/ARKit)
- 📍 **Conteúdo Contextual**: Baseado em localização GPS e marca reconhecida
- 💾 **Modo Offline**: Cache inteligente de logos e conteúdo (AsyncStorage, TTL 30min)
- ⚡ **Performance**: Crop app-side, imagens otimizadas, loading com dicas educativas

## 🚀 Quick Start

### Pré-requisitos

- Node.js 18 ou superior
- npm ou yarn
- Expo CLI: `npm install -g expo-cli`
- Backend OlinxRA rodando
- Conta Expo (para build)

**Para desenvolvimento Android:**
- Android Studio
- Android SDK
- Emulador Android ou dispositivo físico

**Para desenvolvimento iOS:**
- macOS
- Xcode 14+
- iOS Simulator ou dispositivo físico

### Instalação

1. **Clone o repositório**
```bash
git clone https://github.com/gibadalcin/olinxplus.git
cd olinxplus
```

2. **Instale as dependências**
```bash
npm install
```

3. **Configure o Firebase**

Crie o arquivo `firebaseConfig.ts` na raiz:

```typescript
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getStorage } from 'firebase/storage';

const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "seu-projeto.firebaseapp.com",
  projectId: "seu-projeto-id",
  storageBucket: "seu-projeto.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef123456"
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const storage = getStorage(app);
export default app;
```

4. **Configure a API**

Edite `config/api.ts`:

```typescript
export const API_BASE_URL = __DEV__ 
  ? 'http://192.168.1.100:8000'  // Use seu IP local, não localhost
  : 'https://your-backend.ondigitalocean.app'; // Backend Digital Ocean
```

**Importante:** Para testar no dispositivo físico, use o IP da máquina onde o backend está rodando, não `localhost`.

5. **Inicie o Expo**
```bash
npx expo start
```

Opções:
- Pressione `a` para abrir no Android Emulator
- Pressione `i` para abrir no iOS Simulator
- Escaneie QR code com Expo Go (dispositivo físico)
- Pressione `w` para abrir no navegador (web)

**Para dispositivo físico:**
- **iOS**: Abra a câmera nativa e escaneie o QR code
- **Android**: Instale Expo Go e escaneie o QR code

## 📱 Funcionalidades

### 1. Captura e Reconhecimento

#### Câmera Inteligente

- ✅ Captura de foto em alta resolução
- ✅ Detecção automática de logos
- ✅ Feedback visual em tempo real
- ✅ Otimização de imagem antes do envio

<!-- SCREENSHOT: Tela de captura -->

#### Pipeline de Reconhecimento

```
1. Usuário abre câmera
   ↓
2. Tira foto do logo OU seleciona da galeria
   ↓
3. Modal de decisão:
   • Se foto capturada: "Buscar conteúdo" | "Salvar na galeria" | "Cancelar"
   • Se da galeria: "Buscar conteúdo" | "Cancelar"
   ↓
4. Imagem redimensionada (max 800px)
   ↓
5. Enviada para API (Base64)
   ↓
6. Backend: CLIP embedding
   ↓
7. Backend: FAISS busca similar
   ↓
8. Melhor resultado retornado (top-1)
   ↓
9. App carrega conteúdo associado automaticamente
   ↓
10. Exibe conteúdo em tela comum
    ↓
11. Usuário pode visualizar modelos 3D/AR (se dispositivo suportar)
```

### 2. Visualização de Conteúdo

#### Blocos de Conteúdo Suportados

- 📷 **Imagens**: Banner principal, imagens topo
- 🎪 **Carrosséis**: Galeria swipeable de imagens
- 📝 **Textos**: Títulos, subtítulos, parágrafos
- 🔘 **Botões**: Com ações (links externos, navegação)
- 🎭 **Modelos 3D**: Visualização AR (GLB)

<!-- SCREENSHOT: Visualização de conteúdo -->

### 3. Realidade Aumentada

#### AR Viewer

Visualização imersiva de modelos 3D:

- ✅ Carregamento de GLB
- ✅ Rotação 360° com gestos
- ✅ Zoom pinch
- ✅ Iluminação realista
- ✅ Sombras projetadas
- ✅ Performance otimizada

**Tecnologias:**
- Expo GL (WebGL)
- React Three Fiber
- Three.js

<!-- SCREENSHOT: AR Viewer -->

#### Controles AR

```typescript
// Rotação com pan gesture
const onPanGesture = (event) => {
  rotation.y += event.translationX * 0.01;
  rotation.x += event.translationY * 0.01;
};

// Zoom com pinch
const onPinchGesture = (event) => {
  scale *= event.scale;
};
```

### 4. Cache Offline

#### Logo Cache

Cache inteligente para reconhecimento offline:

```typescript
// useLogoCache.ts
const cacheLogos = async (logos: Logo[]) => {
  const cached = await AsyncStorage.getItem('cached_logos');
  const data = cached ? JSON.parse(cached) : [];
  
  // Mesclar novos logos
  const merged = mergeLogs(data, logos);
  
  await AsyncStorage.setItem('cached_logos', JSON.stringify(merged));
};
```

**Estratégia:**
- Cache top-100 logos mais usados
- Atualização incremental
- Expiração após 7 dias
- Fallback para API se não encontrado

#### Content Cache

```typescript
// Salvar conteúdo por marca/região
await AsyncStorage.setItem(
  `content_${marca}_${regiao}`, 
  JSON.stringify(blocos)
);

// Recuperar offline
const cached = await AsyncStorage.getItem(`content_${marca}_${regiao}`);
if (cached) {
  setBlocos(JSON.parse(cached));
}
```

## 🏗️ Arquitetura

```
olinxplus/
├── app/                          # Expo Router (file-based routing)
│   ├── _layout.tsx               # Root layout com GlobalSplashOverlay
│   ├── index.tsx                 # Splash inicial com logo animado
│   └── _tabs/                    # Navegação principal (5 tabs)
│       ├── _layout.tsx
│       ├── index.tsx             # Home (Dashboard)
│       ├── camera.tsx            # Captura guiada com marcadores
│       ├── ar.tsx                # Visualização AR (Expo GL)
│       ├── explore.tsx           # Explorar conteúdos
│       └── profile.tsx           # Perfil do usuário
│
├── components/
│   ├── CustomHeader.tsx          # Header com status WiFi/GPS/Bateria
│   ├── CustomTabBar.tsx          # Tab bar customizada
│   ├── ContentBlocks.tsx         # Renderiza blocos dinâmicos
│   ├── ThemedText.tsx            # Texto com suporte a tema
│   ├── ThemedView.tsx            # View com suporte a tema
│   ├── ar/                       # Componentes AR
│   │   ├── ARCanvas.tsx          # Expo GL + React Three Fiber
│   │   ├── ModelLoader.tsx       # Carrega modelos GLB
│   │   ├── ARSceneControls.tsx   # Controles de zoom/rotação
│   │   └── ImageMarkerTracker.tsx
│   └── ui/
│       ├── GlobalSplashOverlay.tsx  # Loading global com dicas
│       ├── CameraMarkers.tsx        # Guias visuais 300x250px
│       ├── LoadingOverlay.tsx
│       └── ErrorBoundary.tsx
│
├── context/                          # React Context
│   ├── ARPayloadContext.tsx          # Estado AR global (marca, logo, conteúdo)
│   └── CaptureSettingsContext.tsx    # Configurações de captura
│
├── hooks/                            # Custom hooks
│   ├── useARSupport.ts               # Detecta ARCore/ARKit
│   ├── useARContent.ts               # Fetch logo + conteúdo da marca
│   ├── useImageRecognition.ts        # Envio para backend reconhecimento
│   ├── useOfflineCache.ts            # AsyncStorage com TTL 30min
│   └── useColorScheme.ts             # Tema dark/light
│
├── config/
│   └── api.ts                        # API_BASE_URL (Digital Ocean)
│
├── constants/
│   └── Colors.ts                     # Paleta de cores tema
│
├── utils/
│   ├── imageProcessor.ts             # Crop 300x250px, resize
│   ├── cacheManager.ts               # Gerencia AsyncStorage
│   └── networkMonitor.ts             # Status conectividade
│
├── assets/
│   └── images/                       # Imagens estáticas
│
├── firebaseConfig.ts                 # Configuração Firebase
├── app.json                          # Configuração Expo
└── package.json                      # Dependências (Expo 54, React Native 0.76.5)
```

## 🔧 Componentes Principais

### ARCanvas.tsx

Visualizador AR com Expo GL + React Three Fiber:

```typescript
import { Canvas } from '@react-three/fiber/native';
import { useGLTF } from '@react-three/drei/native';

export function ARCanvas({ modelUrl }: { modelUrl: string }) {
  return (
    <Canvas gl={{ physicallyCorrectLights: true }}>
      <ambientLight intensity={0.4} />
      <directionalLight position={[5, 5, 5]} intensity={0.8} castShadow />
      <Model url={modelUrl} />
    </Canvas>
  );
}

function Model({ url }: { url: string }) {
  const { scene } = useGLTF(url);
  return <primitive object={scene} scale={1.5} />;
}
```

### CameraMarkers.tsx

Marcadores visuais para captura guiada (300x250px):

```typescript
export function CameraMarkers() {
  return (
    <View style={styles.overlay}>
      <View style={styles.markerFrame}>
        {/* Cantos do frame guia */}
        <View style={[styles.corner, styles.topLeft]} />
        <View style={[styles.corner, styles.topRight]} />
        <View style={[styles.corner, styles.bottomLeft]} />
        <View style={[styles.corner, styles.bottomRight]} />
        <Text style={styles.hint}>Centralize o logo aqui</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  markerFrame: {
    width: 300,
    height: 250,
    borderWidth: 2,
    borderColor: '#00FF00',
    // ... estilos adicionais
  }
});
```

### ContentBlocks.tsx

Renderizador universal de blocos dinâmicos:

```typescript
export function ContentBlocks({ blocos }: { blocos: Bloco[] }) {
  return (
    <ScrollView>
      {blocos.map((bloco, idx) => {
        switch (bloco.tipo) {
          case 'Imagem topo 1':
            return <ImageBlock key={idx} {...bloco} />;
          
          case 'Carousel 1':
            return <CarouselBlock key={idx} {...bloco} />;
          
          case 'Título 1':
            return <TitleBlock key={idx} {...bloco} />;
          
          case 'modelo_3d':
            return <ARButton key={idx} {...bloco} />;
          
          default:
            return <TextBlock key={idx} {...bloco} />;
        }
      })}
    </ScrollView>
  );
}
```

### useLogoCompare.ts

Hook para comparação visual de logos (retorna melhor match):

```typescript
export function useLogoCompare() {
  const compareLogo = async (capturedImage: string) => {
    try {
      const response = await fetch(`${API_BASE_URL}/logos/find-similar`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ 
          image_data: capturedImage,
          top_k: 1  // Retorna apenas o melhor resultado
        })
      });
      
      const results = await response.json();
      return results.logo; // { marca, score, url }
    } catch (error) {
      console.error('Logo comparison failed:', error);
      return null;
    }
  };
  
  return { compareLogo };
}
```

## 🎯 Fluxo de Dados

### Autenticação

```
1. App inicia
   ↓
2. Verifica auth state (Firebase)
   ↓
3. Se autenticado → Home
   Se não → Login
   ↓
4. Login via email/senha
   ↓
5. Token armazenado
   ↓
6. Usado em requests API
```

### Captura → AR

```
1. Usuário abre câmera no app
   ↓
2. Tira foto do logo OU seleciona da galeria
   ↓
3. Modal de decisão:
   • Se foto capturada: "Buscar conteúdo" | "Salvar na galeria" | "Cancelar"
   • Se da galeria: "Buscar conteúdo" | "Cancelar"
   ↓
4. Usuário escolhe "Buscar conteúdo"
   ↓
5. Redimensiona imagem (max 800px)
   ↓
6. Converte para Base64
   ↓
7. POST /logos/find-similar
   ↓
8. Backend: CLIP embedding + FAISS busca
   ↓
9. Melhor resultado retornado (top-1)
   ↓
10. GET /conteudos/{marca}/{regiao}
    ↓
11. Renderiza blocos de conteúdo
    ↓
12. Exibe em tela comum (imagens, textos, botões, carrosséis)
    ↓
13. Usuário pode visualizar modelos 3D/AR:
    • Verifica suporte AR do dispositivo
    • Se suportado → Abre AR Viewer
    • Visualiza GLB com gestos (rotação, zoom)
```

### Cache Flow

```
1. App busca conteúdo
   ↓
2. Verifica AsyncStorage
   ↓
3. Se cached && !expirado → Usa cache
   ↓
4. Senão → Fetch da API
   ↓
5. Salva no cache
   ↓
6. Renderiza
```

## 🔐 Permissões

### Android (app.json)

```json
{
  "expo": {
    "android": {
      "permissions": [
        "CAMERA",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE",
        "ACCESS_FINE_LOCATION",
        "ACCESS_COARSE_LOCATION"
      ]
    }
  }
}
```

### iOS (app.json)

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "Usamos a câmera para reconhecer logos",
        "NSPhotoLibraryUsageDescription": "Acesso à galeria para salvar fotos",
        "NSLocationWhenInUseUsageDescription": "Localização para conteúdo contextual"
      }
    }
  }
}
```

## 🧪 Testes

### Testar no Dispositivo

**Via Expo Go:**
```bash
npm start
# Escanear QR code
```

**Build Development:**
```bash
# Android
npm run android

# iOS (macOS apenas)
npm run ios
```

### Testar AR

1. Prepare modelo GLB de teste
2. Upload via Admin UI
3. Associe ao carousel
4. Capture logo no app
5. Abra AR viewer
6. Teste gestos (rotação, zoom)

## 📊 Performance

### Otimizações Implementadas

- ✅ **Lazy Loading**: Componentes carregados sob demanda
- ✅ **Image Caching**: Expo Image para cache automático
- ✅ **Memoização**: React.memo em componentes pesados
- ✅ **Debounce**: Inputs de busca
- ✅ **Redução de Bundle**: Tree shaking + code splitting

### Benchmarks

```
Tempo de captura:       ~500ms
Reconhecimento (API):   ~1.5s
Carregamento GLB:       ~2s (modelo médio 5MB)
FPS AR Viewer:          60 FPS (dispositivos modernos)
Uso de Memória:         ~150MB (sem AR)
                        ~300MB (com AR ativo)
```

## 🔨 Build

### Development Build

```bash
# Android
eas build --profile development --platform android

# iOS
eas build --profile development --platform ios
```

### Production Build

```bash
# Android (AAB para Google Play)
eas build --profile production --platform android

# iOS (IPA para App Store)
eas build --profile production --platform ios
```

### Configuração EAS (eas.json)

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "android": {
        "buildType": "app-bundle"
      },
      "ios": {
        "bundleIdentifier": "com.olinxplus.app"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

## 🐛 Troubleshooting

### Problema: "Camera permission denied"
- Android: Verificar `android.permissions` em `app.json`
- iOS: Verificar `NSCameraUsageDescription` em `infoPlist`
- Reinstalar app após adicionar permissões

### Problema: "GL context could not be created"
- Dispositivo não suporta WebGL
- Verificar `useARSupport()` antes de renderizar AR
- Exibir mensagem de erro amigável

### Problema: "GLB model not loading"
- Verificar URL do modelo (signed_url válida)
- Modelo pode estar corrompido
- Testar modelo em https://gltf-viewer.donmccurdy.com/
- Verificar tamanho do arquivo (max ~10MB recomendado)

### Problema: "AsyncStorage quota exceeded"
- Limpar cache antigo: `AsyncStorage.clear()`
- Implementar limpeza automática de cache expirado
- Reduzir número de logos cached

## 📈 Deploy

### Publicar Update (OTA)

```bash
# Publicar update sem rebuild
eas update --branch production --message "Correções de bugs"
```

Usuários receberão update na próxima abertura do app.

### Submeter para Stores

**Google Play:**
```bash
eas submit --platform android
```

**App Store:**
```bash
eas submit --platform ios
```

## 📚 Documentação Adicional

- [Teste de Fluxo AR](TESTE-FLUXO-AR.md)
- [Histórico AR Android](HISTORICO-AR-ANDROID.md)
- [Correção Delay Imagem](CORRECAO-DELAY-IMAGEM.md)
- [Expo Documentation](https://docs.expo.dev/)
- [React Native](https://reactnative.dev/)
- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber)

## 🔗 Repositórios Relacionados

- **Backend API**: [olinxplus-backend](https://github.com/gibadalcin/olinxplus-backend) - FastAPI + CLIP ONNX + FAISS
- **Admin Dashboard**: [olinxplus-adminui](https://github.com/gibadalcin/olinxplus-adminui) - React + MUI
- **Documentação**: [olinxplus-docs](https://github.com/gibadalcin/olinxplus-docs) - Guias técnicos

## 🤝 Contribuindo

Ao contribuir:

1. Teste em iOS **e** Android
2. Verifique permissões necessárias
3. Otimize imagens e assets
4. Mantenha TypeScript types
5. Documente novos hooks/components

## 📄 Licença

Este projeto está sob a licença MIT.

---

<div align="center">
<strong>OlinxPlus Mobile App</strong> | Realidade Aumentada na palma da mão
</div>
