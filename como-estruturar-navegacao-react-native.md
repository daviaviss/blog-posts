---
title: "Como Estruturar a Navegação em Apps React Native"
description: "Um guia prático sobre como organizar a navegação do seu app usando React Navigation, com exemplos reais de Stack, Tab e Drawer."
date: "2026-04-02"
tags: ["react native", "react navigation", "mobile"]
---

A navegação é um dos pilares de qualquer aplicativo mobile. Uma estrutura bem pensada desde o início evita retrabalho e facilita a manutenção. Neste artigo vou mostrar como organizo a navegação nos meus projetos com React Native.

## Por que React Navigation?

O [React Navigation](https://reactnavigation.org/) é a biblioteca mais consolidada para navegação em React Native. Ela suporta os principais padrões de navegação mobile — Stack, Tab e Drawer — e se integra bem tanto com Expo quanto com projetos bare.

## Instalação

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
```

Para cada tipo de navegador, instale o pacote específico:

```bash
# Stack
npm install @react-navigation/native-stack

# Tabs
npm install @react-navigation/bottom-tabs

# Drawer
npm install @react-navigation/drawer
```

## Estrutura de pastas

Antes de escrever código, gosto de separar a navegação em arquivos dedicados:

```
src/
  navigation/
    AppNavigator.tsx     # navegador raiz
    AuthNavigator.tsx    # fluxo de autenticação
    MainNavigator.tsx    # fluxo principal (tabs)
```

Essa separação facilita muito quando o app cresce e você precisa proteger rotas ou adicionar lógica de autenticação.

## Navegador raiz

O `AppNavigator` decide qual fluxo exibir com base no estado de autenticação:

```tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { useAuth } from '@/hooks/useAuth';
import AuthNavigator from './AuthNavigator';
import MainNavigator from './MainNavigator';

const Stack = createNativeStackNavigator();

export default function AppNavigator() {
  const { isAuthenticated } = useAuth();

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {isAuthenticated ? (
          <Stack.Screen name="Main" component={MainNavigator} />
        ) : (
          <Stack.Screen name="Auth" component={AuthNavigator} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

## Tabs com Stack dentro

Um padrão muito comum é ter tabs na parte inferior, e dentro de cada tab um Stack próprio para navegação em profundidade:

```tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import HomeScreen from '@/screens/HomeScreen';
import ProfileScreen from '@/screens/ProfileScreen';
import DetailsScreen from '@/screens/DetailsScreen';

const Tab = createBottomTabNavigator();
const HomeStack = createNativeStackNavigator();

function HomeStackNavigator() {
  return (
    <HomeStack.Navigator>
      <HomeStack.Screen name="Home" component={HomeScreen} />
      <HomeStack.Screen name="Details" component={DetailsScreen} />
    </HomeStack.Navigator>
  );
}

export default function MainNavigator() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="HomeTab" component={HomeStackNavigator} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}
```

## Tipagem com TypeScript

Tipar a navegação evita bugs difíceis de rastrear. Crie um arquivo `types.ts` centralizando todos os params:

```ts
export type RootStackParamList = {
  Main: undefined;
  Auth: undefined;
};

export type HomeStackParamList = {
  Home: undefined;
  Details: { id: string; title: string };
};
```

E use nos seus componentes:

```tsx
import { NativeStackScreenProps } from '@react-navigation/native-stack';
import { HomeStackParamList } from '@/navigation/types';

type Props = NativeStackScreenProps<HomeStackParamList, 'Details'>;

export default function DetailsScreen({ route, navigation }: Props) {
  const { id, title } = route.params;
  // ...
}
```

## Dicas finais

- **Evite navegar pelo estado global** — passe a navegação via props ou use o hook `useNavigation`
- **Nomeie as rotas como constantes** — evita typos e facilita refatorações
- **Use `screenOptions` no navigator** — centralize estilos de header em vez de repetir em cada tela

Uma boa estrutura de navegação é invisível para o usuário, mas faz toda a diferença para quem desenvolve. Espero que esse guia ajude nos seus projetos!