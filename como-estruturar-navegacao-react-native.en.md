---
title: "how to structure navigation in react native apps"
description: "a practical guide on organizing your app's navigation using react navigation, with real examples of stack, tab, and drawer patterns."
date: "2026-04-02"
tags: ["react native", "react navigation", "mobile"]
---

navigation is one of the pillars of any mobile application. a well-thought-out structure from the start prevents rework and makes maintenance easier. in this article i'll show you how i organize navigation in my react native projects.

## why react navigation?

[react navigation](https://reactnavigation.org/) is the most established library for navigation in react native. it supports the main mobile navigation patterns — stack, tab, and drawer — and integrates well with both expo and bare projects.

## installation

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
```

for each navigator type, install the specific package:

```bash
# stack
npm install @react-navigation/native-stack

# tabs
npm install @react-navigation/bottom-tabs

# drawer
npm install @react-navigation/drawer
```

## folder structure

before writing code, i like to separate navigation into dedicated files:

```
src/
  navigation/
    AppNavigator.tsx     # root navigator
    AuthNavigator.tsx    # authentication flow
    MainNavigator.tsx    # main flow (tabs)
```

this separation makes things much easier when the app grows and you need to protect routes or add authentication logic.

## root navigator

the `AppNavigator` decides which flow to display based on the authentication state:

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

## tabs with nested stacks

a very common pattern is having bottom tabs, with each tab having its own stack for deep navigation:

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

## typescript typing

typing your navigation prevents hard-to-trace bugs. create a `types.ts` file centralizing all params:

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

and use them in your components:

```tsx
import { NativeStackScreenProps } from '@react-navigation/native-stack';
import { HomeStackParamList } from '@/navigation/types';

type Props = NativeStackScreenProps<HomeStackParamList, 'Details'>;

export default function DetailsScreen({ route, navigation }: Props) {
  const { id, title } = route.params;
  // ...
}
```

## final tips

- **avoid navigating through global state** — pass navigation via props or use the `useNavigation` hook
- **name routes as constants** — avoids typos and makes refactoring easier
- **use `screenOptions` on the navigator** — centralize header styles instead of repeating them on each screen

good navigation structure is invisible to the user, but makes all the difference for the developer. hope this guide helps in your projects!