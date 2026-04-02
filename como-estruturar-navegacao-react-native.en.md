---
title: "How to Structure Navigation in React Native Apps"
description: "A practical guide on organizing your app's navigation using React Navigation, with real examples of Stack, Tab, and Drawer patterns."
date: "2026-04-02"
tags: ["react native", "react navigation", "mobile"]
---

Navigation is one of the pillars of any mobile application. A well-thought-out structure from the start prevents rework and makes maintenance easier. In this article I'll show you how I organize navigation in my React Native projects.

## Why React Navigation?

[React Navigation](https://reactnavigation.org/) is the most established library for navigation in React Native. It supports the main mobile navigation patterns — Stack, Tab, and Drawer — and integrates well with both Expo and bare projects.

## Installation

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
```

For each navigator type, install the specific package:

```bash
# Stack
npm install @react-navigation/native-stack

# Tabs
npm install @react-navigation/bottom-tabs

# Drawer
npm install @react-navigation/drawer
```

## Folder Structure

Before writing code, I like to separate navigation into dedicated files:

```
src/
  navigation/
    AppNavigator.tsx     # root navigator
    AuthNavigator.tsx    # authentication flow
    MainNavigator.tsx    # main flow (tabs)
```

This separation makes things much easier when the app grows and you need to protect routes or add authentication logic.

## Root Navigator

The `AppNavigator` decides which flow to display based on the authentication state:

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

## Tabs with Nested Stacks

A very common pattern is having bottom tabs, with each tab having its own Stack for deep navigation:

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

## TypeScript Typing

Typing your navigation prevents hard-to-trace bugs. Create a `types.ts` file centralizing all params:

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

And use them in your components:

```tsx
import { NativeStackScreenProps } from '@react-navigation/native-stack';
import { HomeStackParamList } from '@/navigation/types';

type Props = NativeStackScreenProps<HomeStackParamList, 'Details'>;

export default function DetailsScreen({ route, navigation }: Props) {
  const { id, title } = route.params;
  // ...
}
```

## Final Tips

- **Avoid navigating through global state** — pass navigation via props or use the `useNavigation` hook
- **Name routes as constants** — avoids typos and makes refactoring easier
- **Use `screenOptions` on the navigator** — centralize header styles instead of repeating them on each screen

Good navigation structure is invisible to the user, but makes all the difference for the developer. Hope this guide helps in your projects!