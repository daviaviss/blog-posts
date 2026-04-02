---
title: "como estruturar a navegação em apps react native"
description: "um guia prático sobre como organizar a navegação do seu app usando react navigation, com exemplos reais de stack, tab e drawer."
date: "2026-04-02"
tags: ["react native", "react navigation", "mobile"]
---

a navegação é um dos pilares de qualquer aplicativo mobile. uma estrutura bem pensada desde o início evita retrabalho e facilita a manutenção. neste artigo vou mostrar como organizo a navegação nos meus projetos com react native.

## por que react navigation?

o [react navigation](https://reactnavigation.org/) é a biblioteca mais consolidada para navegação em react native. ela suporta os principais padrões de navegação mobile — stack, tab e drawer — e se integra bem tanto com expo quanto com projetos bare.

## instalação

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
```

para cada tipo de navegador, instale o pacote específico:

```bash
# stack
npm install @react-navigation/native-stack

# tabs
npm install @react-navigation/bottom-tabs

# drawer
npm install @react-navigation/drawer
```

## estrutura de pastas

antes de escrever código, gosto de separar a navegação em arquivos dedicados:

```
src/
  navigation/
    AppNavigator.tsx     # navegador raiz
    AuthNavigator.tsx    # fluxo de autenticação
    MainNavigator.tsx    # fluxo principal (tabs)
```

essa separação facilita muito quando o app cresce e você precisa proteger rotas ou adicionar lógica de autenticação.

## navegador raiz

o `AppNavigator` decide qual fluxo exibir com base no estado de autenticação:

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

## tabs com stack dentro

um padrão muito comum é ter tabs na parte inferior, e dentro de cada tab um stack próprio para navegação em profundidade:

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

## tipagem com typescript

tipar a navegação evita bugs difíceis de rastrear. crie um arquivo `types.ts` centralizando todos os params:

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

e use nos seus componentes:

```tsx
import { NativeStackScreenProps } from '@react-navigation/native-stack';
import { HomeStackParamList } from '@/navigation/types';

type Props = NativeStackScreenProps<HomeStackParamList, 'Details'>;

export default function DetailsScreen({ route, navigation }: Props) {
  const { id, title } = route.params;
  // ...
}
```

## dicas finais

- **evite navegar pelo estado global** — passe a navegação via props ou use o hook `useNavigation`
- **nomeie as rotas como constantes** — evita typos e facilita refatorações
- **use `screenOptions` no navigator** — centralize estilos de header em vez de repetir em cada tela

uma boa estrutura de navegação é invisível para o usuário, mas faz toda a diferença para quem desenvolve. espero que esse guia ajude nos seus projetos!