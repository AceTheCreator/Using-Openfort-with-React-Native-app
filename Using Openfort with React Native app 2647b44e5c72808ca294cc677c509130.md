# Using Openfort with React Native app

**Build a cross-platform Web3 game featuring seamless authentication, wallet management, and blockchain integration using React Native and Openfort.**

![banner.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/banner.png)

As mobile technology increasingly influences the adoption of on-chain applications, Openfort offers a foundational framework for developers to easily implement streamlined authentication and wallet integration across various platforms.

In this tutorial, you'll learn how to create a game application called **`LukcyNumber`** from scratch. You'll create this app with React Native and implement secure player authentication and wallet management with Openfort. By the end of this tutorial, you'll have built a fully functional, cross-platform Web3 game app with a smooth native experience on both iOS and Android.

Here’s an overview of the application infrastructure:

![infra.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/infra.png)


## Why Openfort?

When building a complete on-chain application, you need more than just authentication. You need a full-stack solution. Openfort provides a comprehensive infrastructure that consolidates everything in one place, from authentication to transactions and beyond, so you can focus on the user experience.

### End-to-End Control

- **Authentication & Wallets:** Seamlessly manage user identities and wallets. You can offer custodial wallets for ease of use or non-custodial wallets for true ownership.
- **Transaction Management:** Simplify complex blockchain interactions. Openfort handles everything from transaction signing to gas sponsorship, so your users never have to worry about gas fees.
- **Server-Side Control:** Maintain full control over the user experience from your own backend, ensuring a secure and reliable application.

---

### Developer Experience

- **Gaming SDKs:** Onboard every type of gamer by creating user-friendly Web3 games that feel as familiar as traditional ones.
- **Simple APIs:** [**Straightforward REST endpoints**](https://openfort.apidocumentation.com/reference) that make integration easy.
- **Multiple SDKs:** We support a wide range of platforms, including **React, React Native, Unity, and Unreal Engine**.
- **Open Source:** Our **open-source** approach gives you the flexibility to inspect, modify, and contribute to our codebase. This not only builds trust but also allows for a more secure and adaptable infrastructure.

## Building LuckyNumber: step-by-step

Now that you understand why using Openfort for your dApp is such a cool idea, it's time to build our `LuckyNumber` game from scratch.

Before we begin, this tutorial assumes that you're comfortable with building a simple React Native application independently. No prior knowledge of Openfort is required.

If you want to check the final code by yourself, here's the [repository](https://github.com/AceTheCreator/react-native-with-openfort).

### Requirements

Before following along, make sure you have the following setup:

- An Openfort account (sign up at [dashboard.openfort.xyz](https://dashboard.openfort.xyz/))
- [Expo environment setup](https://docs.expo.dev/get-started/installation/#requirements) (Node.js, Git, Watchman)

> **Note:** As you build this project, be sure to check the comments in each code snippet for additional context on relevant code blocks.

### 1. Set up your Openfort Project

1. Start by creating a new project in the [Openfort dashboard](https://dashboard.openfort.io/). Name your project "LuckyN", and then **Create project**:

    ![dashboard1.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/dashboard1.png)

2. Once created, your project will appear on the dashboard's list of projects. As seen below, select your project name:

     ![d2.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d2.png)

3. The next page will show Openfort's onboarding screen, which gives you a quick overview of how to get started.

    ![d3.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d3.png)

4. Next, navigate to the **API Keys** tab in the left sidebar, and click on the **`Create Shield keys`** button to open a modal prompting you to create shield keys. The shield API keys we are creating will allow our players to interact with wallet functionality directly in our app, instead of connecting to MetaMask or other external wallets:

    ![d5.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d5.png)

5.  In the modal, select **Create Shield keys*** to generate shield keys:

    ![d6.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d6.png)

6. Once the Shield key has been generated, copy the **`Shield encryption key`** and save it somewhere. This will be used in the next step:

    ![d7.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d7.png)

7. Finally, copy the **`Publishable keys`** from both the Project keys and Shield key. This will be used in the next step:

    ![d8.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d8.png)

### 2. Project Setup

To get started, we'll set up your React native app using [Expo](https://docs.expo.dev/). This approach will allow us to create a cohesive experience across platforms.

#### Setting Up the React Native Project(IOS Version)

1. **Create React Native App:**
    - Open your terminal and run the following command to create a new React Native project using the latest Expo SDK:
        
        ```bash
        npx create-expo-app@latest luckyN
        cd luckyN
        ```
        
    - Run the development server, and if everything works correctly, you should see the Expo development server running. For this tutorial, I'll be using the iOS version of the app:
        
        ```bash
         npm run ios
        ```
        
   ![simulator.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/simulator.png)

2. **Install Dependencies:**
    - We'll use the [Openfort React Native SDK](https://www.npmjs.com/package/@openfort/react-native) to manage the wallet creation and authentication for our app, and its required peer dependencies to facilitate the foundational cryptographic and platform-specific features that make non-custodial embedded wallets possible in React Native. Install these dependencies by running:
    
    ```bash
    # Install Openfort React Native SDK
    npm install @openfort/react-native
     
    # Install required dependencies
    yarn add expo-apple-authentication expo-application expo-crypto expo-secure-store react native-get-random-values
    ```
    
3. **Configure Metro for React Native Crypto Compatibility:**
    - This Metro configuration fixes [Jose library](https://www.npmjs.com/package/jose) compatibility in React Native by forcing Metro to load the `browser` version instead of the **`Node.js`** version, since React Native's JavaScript environment doesn't support Node.js-specific APIs. Create a new file in your root folder by running the following command:
    
      ```bash
      touch metro.config.js
      ```
    
    - In the newly created **Metro config file**, include the following:
    
      ```jsx
      // metro.config.js
      const { getDefaultConfig } = require("expo/metro-config");
       
      /** @type {import('expo/metro-config').MetroConfig} */
      const config = getDefaultConfig(__dirname);
       
      const resolveRequestWithPackageExports = (context, moduleName, platform) => {
        // Package exports in `jose` are incorrect, so we need to force the browser version
        if (moduleName === "jose") {
          const ctx = {
            ...context,
            unstable_conditionNames: ["browser"],
          };
          return ctx.resolveRequest(ctx, moduleName, platform);
        }
       
        return context.resolveRequest(context, moduleName, platform);
      };
       
      config.resolver.resolveRequest = resolveRequestWithPackageExports;
       
      module.exports = config;
      ```
    
4. **Set up Entry Point:**
   - Create an `entrypoint.js` file in your project root. This file is **crucial** for initializing the Openfort SDK and ensuring proper polyfill loading:
   
     ```jsx
     // entrypoint.js
     
     // Import required polyfills first
     // IMPORTANT: These polyfills must be installed in this order
     
     import "react-native-get-random-values";
     // Then import the expo router
     
     import "expo-router/entry";
     ```
   
   - Finally, update your `package.json` to use this entry point:

     ```json
     {
       "main": "entrypoint.js",
     }
     ```

5. **Adding your Openfort API Keys**
   - To connect your app to Openfort's services, you need to add your API keys to the project configuration. Navigate to your projects `app.json` file and include your Openfort API keys:
   
     ```json
      {
        "expo": {
          "name": "your-app-name",
          "slug": "your-app-slug",
          "extra": {
            "openfortPublishableKey": "your_publishable_key",
            "openfortShieldPublishableKey": "your_sheild_publishable_key",
            "openfortShieldEncryptionKey": "your_shield_encryption_key"
          }
        }
      }
     ```

     > **Security Best Practice:** Always use environment variables for production apps. Never commit API keys directly to version control, especially when using public repositories.
    
    - Finally, extend your application plugin by including the following installed dependencies: 

      ```json
      {
          "expo": {
            "name": "your-app-name",
            "slug": "your-app-slug",
            "plugins": [
              "expo-router",
              "expo-secure-store", // Secure Storage Plugin
              "expo-apple-authentication", // Social Logins Plugin
              [
                "expo-splash-screen",
                {
                  "image": "./assets/images/splash-icon.png",
                  "imageWidth": 200,
                  "resizeMode": "contain",
                  "backgroundColor": "#ffffff"
                }
              ]
            ],
          }
        }
      ```
6. **Run the Application:**
    -  Start the Expo server and test the app on your device or simulator:

       ```shell
       npm run ios
       ```
#### Summary of Project Setup

At this point, you now have a React Native app with Expo configured and ready for development. With these essential foundations in place, we can begin integrating Openfort's Web3 capabilities to add wallet functionality and user authentication to your application.
  

### 3. Adding Player Authentication with Openfort

Now that your project is set up, the next step is to integrate Openfort into your application for a complete Web3 experience.

To use the Openfort SDK, you need to wrap your entire application in the `OpenfortProvider` component, which provides access to all Openfort hooks and functionality throughout your app's component tree.

#### Setting Up Openfort in your Application

   1. **Provider Setup:**
       - Start by updating your `app/_layout.tsx` to wrap your application in the **`OpenfortProvider`**:

         ```jsx
         import {
           DarkTheme,
           DefaultTheme,
           ThemeProvider,
         } from "@react-navigation/native";
         import Constants from "expo-constants";
         import { useFonts } from "expo-font";
         import { Stack } from "expo-router";
         import { StatusBar } from "expo-status-bar";
         import "react-native-reanimated";
         
         import { useColorScheme } from "@/hooks/useColorScheme";
         import { OpenfortProvider } from "@openfort/react-native"; // Import the provider
         
         export default function RootLayout() {
           const colorScheme = useColorScheme();
           const [loaded] = useFonts({
             SpaceMono: require("../assets/fonts/SpaceMono-Regular.ttf"),
           });
         
           // Wait for fonts to load before rendering the app
           if (!loaded) {
             return null;
           }
         
           return (
             <OpenfortProvider
               publishableKey={Constants.expoConfig?.extra?.openfortPublishableKey} // Connect to your Openfort project
             >
               <ThemeProvider value={colorScheme === "dark" ? DarkTheme : DefaultTheme}>
                 <Stack>
                   <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
                   <Stack.Screen name="+not-found" />
                 </Stack>
                 <StatusBar style="auto" />
               </ThemeProvider>
             </OpenfortProvider>
           );
         }
         ```
2. **Adding Wallet Configuration:**
      - To enable wallet creation and management, extend the provider with wallet configuration properties:

        ```jsx
        import { OpenfortProvider, RecoveryMethod } from "@openfort/react-native"; // Add   RecoveryMethod import
        
        export default function RootLayout() {
          const colorScheme = useColorScheme();
          const [loaded] = useFonts({
            SpaceMono: require("../assets/fonts/SpaceMono-Regular.ttf"),
          });
        
          if (!loaded) {
            return null;
          }
        
          return (
            <OpenfortProvider
              publishableKey={Constants.expoConfig?.extra?.openfortPublishableKey}
              walletConfig={{
                // Shield service for secure key management
                
                shieldPublishableKey:
                  Constants.expoConfig?.extra?.openfortShieldPublishableKey,
                
                // Allow users to recover wallets with password
                recoveryMethod: RecoveryMethod.PASSWORD,
                
                // Encryption key for additional security
                shieldEncryptionKey:
                  Constants.expoConfig?.extra?.openfortShieldEncryptionKey,
              }}
            >
              <ThemeProvider value={colorScheme === "dark" ? DarkTheme : DefaultTheme}>
                <Stack>
                  <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
                  <Stack.Screen name="+not-found" />
                </Stack>
                <StatusBar style="auto" />
              </ThemeProvider>
            </OpenfortProvider>
          );
        }
        ```
3. **Adding Blockchain Network Support:**
      - Finally, specify which blockchain networks your app will support by adding the **`supportedChains`** configuration:

        ```jsx
        export default function RootLayout() {
          const colorScheme = useColorScheme();
          const [loaded] = useFonts({
            SpaceMono: require("../assets/fonts/SpaceMono-Regular.ttf"),
          });
        
          if (!loaded) {
            return null;
          }
        
          return (
            <OpenfortProvider
              publishableKey={Constants.expoConfig?.extra?.openfortPublishableKey}
              walletConfig={{
                shieldPublishableKey:
                  Constants.expoConfig?.extra?.openfortShieldPublishableKey,
                recoveryMethod: RecoveryMethod.PASSWORD,
                shieldEncryptionKey:
                  Constants.expoConfig?.extra?.openfortShieldEncryptionKey,
              }}
              supportedChains={[
                {
                  // Avalanche Fuji testnet configuration
                  id: 43113, // Chain ID for Avalanche Fuji
                  name: "Avalanche Fuji",
                  nativeCurrency: {
                    name: "Avalanche", // Display name
                    symbol: "AVAX", // Currency symbol
                    decimals: 18, // Standard ERC-20 decimals
                  },
                  rpcUrls: {
                    default: {
                      // RPC endpoint for connecting to the network
                      http: ["https://api.avax-test.network/ext/bc/C/rpc"],
                    },
                  },
                },
              ]}
            >
              <ThemeProvider value={colorScheme === "dark" ? DarkTheme : DefaultTheme}>
                <Stack>
                  <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
                  <Stack.Screen name="+not-found" />
                </Stack>
                <StatusBar style="auto" />
              </ThemeProvider>
            </OpenfortProvider>
          );
        }
        ```
      
      - With this setup complete, your app now has access to Openfort's infastructure, including wallet management capabilities. The provider makes these features available to any component in your app through Openfort's React hooks.

        > **Note:** We're using [Avalanche Fuji testnet](https://build.avax.network/docs/quick-start/networks/fuji-testnet) `(chain ID 43113)` for development, which provides free AVAX tokens for testing. For production, you would add Avalanche mainnet (chain ID 43114).

  #### Setting Up Protected Routes and Authentication
  
  1. **Create a Protected Area for Authenticated Players:**

       - Create a new folder called `app/(tabs)/protected/`

       - Move all existing files from `app/(tabs)/` into the new `protected/` folder. Your file structure should change as follows:
       
         ```
         # Before
         app/
         └── (tabs)/
             ├── _layout.tsx
             ├── explore.tsx
             └── index.tsx
         
         # After
         app/
         └── (tabs)/
             ├── _layout.tsx          # New: Main authentication router
             ├── index.tsx            # New: Login/authentication page
             └── protected/
                 ├── _layout.tsx      # Moved: Protected area layout
                 ├── explore.tsx      # Moved: Protected screen
                 └── index.tsx        # Moved: Protected home screen
          ```

  2. **Create Authentication Router:**

       - Create a new layout file at `app/(tabs)/_layout.tsx` that handles routing between public and protected areas:

          ```jsx
          import { useOpenfort } from "@openfort/react-native";
          import { Stack } from "expo-router";
          import { useEffect, useState } from "react";
          
          export default function AppLayout() {
            const { user } = useOpenfort(); // Get current user state
            const [isAuth, setIsAuth] = useState<boolean>(false);
          
            // Update authentication state when user changes
            useEffect(() => {
              if (user) {
                setIsAuth(true);
              } else {
                setIsAuth(false); // Reset to false when user logs out
              }
            }, [user]);
          
            return (
              <Stack screenOptions={{ headerShown: false }}>
                {/* Public routes - shown when user is NOT authenticated */}
                <Stack.Protected guard={!user}>
                  <Stack.Screen name="index" /> {/* Login/Auth screen */}
                </Stack.Protected>
          
                {/* Protected routes - shown when user IS authenticated */}
                <Stack.Protected guard={isAuth}>
                  <Stack.Screen name="protected" /> {/* Protected area */}
                </Stack.Protected>
              </Stack>
            );
          }
          ```

  #### Handling Player Authentication
  
  1. **Create Authentication Page:**

       - Create a dedicated styles file for consistent authentication UI across your app.

       - Create `constants/AuthStyles.ts`:

          ```typescript
          import { StyleSheet } from 'react-native';
          
          export const styles = StyleSheet.create({
            // Main container
            container: {
              flex: 1,
              backgroundColor: '#1a1a2e', // Dark theme background
            },
          
            // Content layout
            content: {
              flex: 1,
              justifyContent: "space-between",
              paddingHorizontal: 24,
              paddingVertical: 40,
            },
          
            // Header section
            header: {
              alignItems: "center",
              marginTop: 60,
            },
            logoContainer: {
              marginBottom: 24,
            },
            logo: {
              width: 80,
              height: 80,
              borderRadius: 40,
              backgroundColor: "rgba(255, 255, 255, 0.1)",
              justifyContent: "center",
              alignItems: "center",
              borderWidth: 2,
              borderColor: "rgba(255, 255, 255, 0.2)",
            },
            logoText: {
              fontSize: 40,
            },
            title: {
              fontSize: 24,
              color: '#ffffff',
              marginBottom: 8,
              fontWeight: "300",
            },
            appName: {
              fontSize: 32,
              fontWeight: "bold",
              color: '#ffffff',
              marginBottom: 12,
              textAlign: "center",
            },
            subtitle: {
              fontSize: 16,
              color: "rgba(255, 255, 255, 0.7)", // Semi-transparent white
              textAlign: "center",
              lineHeight: 22,
            },
          
            // Authentication section
            authSection: {
              flex: 1,
              justifyContent: "center",
              paddingVertical: 40,
            },
          
            // Guest login button
            guestButton: {
              borderRadius: 16,
              overflow: "hidden",
              marginBottom: 32,
              // Shadow effects for iOS
              shadowColor: "#ff6b6b",
              shadowOffset: { width: 0, height: 4 },
              shadowOpacity: 0.3,
              shadowRadius: 8,
              // Elevation for Android
              elevation: 8,
            },
            buttonGradient: {
              flexDirection: "row",
              alignItems: "center",
              justifyContent: "center",
              paddingVertical: 18,
              paddingHorizontal: 24,
            },
            guestButtonIcon: {
              fontSize: 20,
              marginRight: 12,
            },
            primaryButtonText: {
              color: '#ffffff',
              fontSize: 18,
              fontWeight: "600",
              letterSpacing: 0.5,
            },
          
            // Divider between auth methods
            dividerContainer: {
              flexDirection: "row",
              alignItems: "center",
              marginVertical: 24,
            },
            dividerLine: {
              flex: 1,
              height: 1,
              backgroundColor: "rgba(255, 255, 255, 0.2)", // Light divider line
            },
            dividerText: {
              color: "rgba(255, 255, 255, 0.6)",
              fontSize: 14,
              fontWeight: "500",
              marginHorizontal: 16,
              letterSpacing: 1,
            },
          
            // OAuth section
            oauthContainer: {
              gap: 16,
            },
            oauthButton: {
              backgroundColor: "rgba(255, 255, 255, 0.95)",
              borderRadius: 12,
              overflow: "hidden",
              // Shadow effects
              shadowColor: "#000",
              shadowOffset: { width: 0, height: 2 },
              shadowOpacity: 0.1,
              shadowRadius: 4,
              elevation: 4,
            },
            oauthButtonContent: {
              flexDirection: "row",
              alignItems: "center",
              justifyContent: "center",
              paddingVertical: 16,
              paddingHorizontal: 24,
            },
            googleIcon: {
              fontSize: 20,
              marginRight: 12,
            },
            oauthButtonText: {
              color: "#333333",
              fontSize: 16,
              fontWeight: "600",
              letterSpacing: 0.3,
            },
          
            // Error display
            errorContainer: {
              backgroundColor: "rgba(255, 59, 48, 0.1)",
              borderRadius: 8,
              padding: 12,
              marginTop: 16,
              borderWidth: 1,
              borderColor: "rgba(255, 59, 48, 0.3)",
            },
            errorText: {
              color: "#ff6b6b",
              fontSize: 14,
              textAlign: "center",
              fontWeight: "500",
            },
          });
          ```

      - Update your `app/(tabs)/index.tsx` to create a comprehensive login screen with both guest and OAuth authentication options:

        ```jsx
        import { OAuthProvider, useGuestAuth, useOAuth } from "@openfort/react-native";
        import { LinearGradient } from "expo-linear-gradient";
        import { styles } from '@/constants/AuthStyles';
        import {
          Alert,
          SafeAreaView,
          StatusBar,
          Text,
          TouchableOpacity,
          View,
        } from "react-native";
        
        export default function AuthScreen() {
          // Openfort authentication hooks
          const { signUpGuest } = useGuestAuth();
          const { initOAuth, error } = useOAuth();
        
          // Handle guest login with error handling
          const handleGuestLogin = async () => {
            try {
              await signUpGuest();
              // User will be automatically redirected to protected routes
            } catch (err) {
              console.error("Guest login error:", err);
              Alert.alert("Login Error", "Failed to login as guest. Please try again.");
            }
          };
        
          // Handle OAuth login with error handling
          const handleOAuthLogin = async (provider: string) => {
            try {
              await initOAuth({ provider: provider as OAuthProvider });
              // User will be automatically redirected to protected routes
            } catch (err) {
              console.error(`OAuth login error with ${provider}:`, err);
              Alert.alert(
                "Login Error",
                `Failed to login with ${provider}. Please try again.`
              );
            }
          };
        
          return (
            <SafeAreaView style={styles.container}>
              {/* Status bar configuration for dark theme */}
              <StatusBar barStyle="light-content" backgroundColor="#1a1a2e" />
        
              <View style={styles.content}>
                {/* App branding header */}
                <View style={styles.header}>
                  <View style={styles.logoContainer}>
                    <View style={styles.logo}>
                      <Text style={styles.logoText}>🎲</Text>
                    </View>
                  </View>
        
                  <Text style={styles.title}>Welcome to</Text>
                  <Text style={styles.appName}>Openfort LuckyN</Text>
                  <Text style={styles.subtitle}>
                    Your Web3 gaming experience starts here
                  </Text>
                </View>
        
                {/* Authentication options */}
                <View style={styles.authSection}>
                  {/* Guest login - fastest way to get started */}
                  <TouchableOpacity
                    style={styles.guestButton}
                    onPress={handleGuestLogin}
                    activeOpacity={0.8}
                  >
                    <LinearGradient
                      colors={["#ff6b6b", "#ee5a52"]} // Red gradient
                      style={styles.buttonGradient}
                    >
                      <Text style={styles.guestButtonIcon}>👤</Text>
                      <Text style={styles.primaryButtonText}>Continue as Guest</Text>
                    </LinearGradient>
                  </TouchableOpacity>
        
                  {/* Divider between auth methods */}
                  <View style={styles.dividerContainer}>
                    <View style={styles.dividerLine} />
                    <Text style={styles.dividerText}>OR</Text>
                    <View style={styles.dividerLine} />
                  </View>
        
                  {/* OAuth login options */}
                  <View style={styles.oauthContainer}>
                    <TouchableOpacity
                      style={styles.oauthButton}
                      onPress={() => handleOAuthLogin("google")}
                      activeOpacity={0.8}
                    >
                      <View style={styles.oauthButtonContent}>
                        <Text style={styles.googleIcon}>📧</Text>
                        <Text style={styles.oauthButtonText}>Continue with Google</Text>
                      </View>
                    </TouchableOpacity>
                  </View>
        
                  {/* Error display - shows OAuth errors */}
                  {error && (
                    <View style={styles.errorContainer}>
                      <Text style={styles.errorText}>⚠️ {error.message}</Text>
                    </View>
                  )}
                </View>
              </View>
            </SafeAreaView>
          );
        }
        ```

      - With your `app/(tabs)/index.tsx` updated, this will ensure that the first screen a player sees when they access the platform is the login page:

 2. **Configure Google OAuth Setup:**

     - To enable Google OAuth authentication, you need to configure credentials in both Google Cloud Console and your Openfort dashboard.

     - **Visit the [Google Cloud Console](https://console.cloud.google.com/apis/credentials)** to configure your OAuth credentials. When creating a new OAuth **client ID**, choose Android or iOS depending on the mobile operating system your app is built for.

     - Navigate to the **Configuration** tab in the left sidebar in your [Openfort Dashboard](https://dashboard.openfort.io), and expand the **Google** authentication section:

       ![d4.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d4.png)

    - Toggle the Enable Sign in with Google button:

      ![d10.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d10.png)

    - Paste your application **`Client IDs`** and **`Client Secret`** from your Google Cloud Project, and save your configuration:

      ![d9.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d9.png)


    - Authentication should be fully configured at this point. Start the development server by running the following command in your terminal:

      ```
      npm run ios
      ```
    - Your application should look like this after executing the above commmand:

        ![auth.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/auth.png)

### 4. Implementing the Game Screen

Our goal in this section is to create the Lucky Number game and implementing wallet-gated access. Only authenticated users with active wallets can play the game, demonstrating how to build engaging Web3 gaming experiences.
 
 ![homescreen.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/homescreen.png)

 #### Setting up Game Page. 
 
   1. **Create the Game Component:**
       
       -  Create **`constants/GameStyles.ts`** and paste the game style:
          ```typescript
          import { StyleSheet, Dimensions } from 'react-native';
          
          const { width } = Dimensions.get('window');
          
          export const styles = StyleSheet.create({
            // Main container
            container: {
              alignItems: "center",
              padding: 20,
              backgroundColor: '#f8f9fa', // Light background
            },
          
            // Game title
            title: {
              fontSize: 24,
              fontWeight: "bold",
              color: "#333333", // Dark text for readability
              marginBottom: 20,
              textAlign: "center",
            },
          
            // Countdown timer
            countdownContainer: {
              width: 60,
              height: 60,
              borderRadius: 30,
              backgroundColor: "#ff6b6b", // Red background for urgency
              justifyContent: "center",
              alignItems: "center",
              marginBottom: 20,
            },
            countdown: {
              fontSize: 28,
              fontWeight: "bold",
              color: "#ffffff",
            },
          
            // Numbers grid layout
            numbersContainer: {
              flexDirection: "row",
              flexWrap: "wrap",
              justifyContent: "center",
              gap: 10,
              marginBottom: 30,
            },
            numberCard: {
              width: (width - 80) / 3, // Responsive width for 3 cards per row
              height: 80,
              backgroundColor: "#f0f0f0",
              borderRadius: 8,
              justifyContent: "center",
              alignItems: "center",
              borderWidth: 2,
              borderColor: "#ddd",
            },
            // Selected target number styling
            targetCard: {
              backgroundColor: "#3CB371", // Green for selected target
              borderColor: "#7CFC00",
            },
            // User's guess selection
            selectedCard: {
              backgroundColor: "#ff6b6b", // Red for selected guess
              borderColor: "#ff6b6b",
            },
            numberText: {
              fontSize: 32,
              fontWeight: "bold",
              color: "#333333",
            },
          
            // Game instructions
            instructionContainer: {
              backgroundColor: "rgba(0, 0, 0, 0.05)", // Subtle background
              padding: 15,
              borderRadius: 8,
              marginHorizontal: 20,
            },
            instructionText: {
              fontSize: 16,
              color: "#333333",
              textAlign: "center",
            },
          
            // Reset/New Game button
            resetButton: {
              marginTop: 20,
              backgroundColor: "rgba(46, 213, 115, 0.2)",
              borderRadius: 12,
              paddingVertical: 12,
              paddingHorizontal: 24,
              borderWidth: 1,
              borderColor: "#2ed573",
            },
            resetButtonText: {
              color: "#2ed573",
              fontSize: 16,
              fontWeight: "600",
              textAlign: "center",
            },
          });
          ```


          - Create **`components/NumberPreview.tsx`** and paste the following code:

             ```jsx
             import React, { useEffect, useState } from "react";
             import {
               Text,
               TouchableOpacity,
               View,
             } from "react-native";
             import { styles } from '../constants/GameStyles';
             
             interface NumberPreviewProps {
               numbers: number[];
               onNumberSelect: (selectedNumber: number, isCorrect: boolean) => void;
               onGameReset?: () => void;
             }
             
             export const NumberPreview: React.FC<NumberPreviewProps> = ({
               numbers,
               onNumberSelect,
               onGameReset,
             }) => {
               // Game state management
               const [countdown, setCountdown] = useState(5);
               const [gamePhase, setGamePhase] = useState<
                 "select" | "ready" | "shuffle" | "guess" | "failed"
               >("select");
               const [selectedTargetNumber, setSelectedTargetNumber] = useState<number | null>(null);
               const [shuffledNumbers, setShuffledNumbers] = useState(numbers);
               const [correctPosition, setCorrectPosition] = useState(0);
               const [selectedCard, setSelectedCard] = useState<number | null> (null);
             
               // Countdown timer effect for ready phase
               useEffect(() => {
                 if (gamePhase === "ready" && countdown > 0) {
                   const timer = setTimeout(() => {
                     setCountdown(countdown - 1);
                   }, 1000);
                   return () => clearTimeout(timer);
                 } else if (countdown === 0 && gamePhase === "ready") {
                   startShufflePhase();
                 }
               }, [countdown, gamePhase]);
             
               // Handle player selecting their target number
               const handleTargetNumberSelect = (number: number) => {
                 if (gamePhase !== "select") return;
             
                 setSelectedTargetNumber(number);
                 setGamePhase("ready");
                 setCountdown(5); // Start 5-second countdown
               };
             
               // Begin the shuffle phase
               const startShufflePhase = () => {
                 setGamePhase("shuffle");
             
                 // Brief delay for visual feedback before shuffle
                 setTimeout(() => {
                   shuffleNumbers();
                 }, 1000);
               };
             
               // Shuffle the numbers and track target position
               const shuffleNumbers = () => {
                 // Fisher-Yates shuffle algorithm
                 const shuffled = [...numbers];
                 for (let i = shuffled.length - 1; i > 0; i--) {
                   const j = Math.floor(Math.random() * (i + 1));
                   [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
                 }
             
                 // Find where the target number ended up
                 const newTargetIndex = shuffled.indexOf(selectedTargetNumber!);
             
                 setShuffledNumbers(shuffled);
                 setCorrectPosition(newTargetIndex);
                 setGamePhase("guess");
               };
             
               // Handle player's guess
               const handleCardPress = (index: number) => {
                 if (gamePhase !== "guess" || selectedCard !== null) return;
             
                 setSelectedCard(index);
                 const isCorrect = index === correctPosition;
                 
                 if (!isCorrect) {
                   setGamePhase("failed");
                 }
             
                 // Delay feedback to show selection
                 setTimeout(() => {
                   onNumberSelect(selectedTargetNumber!, isCorrect);
                 }, 500);
               };
             
               // Reset game to initial state
               const resetGame = () => {
                 setCountdown(5);
                 setGamePhase("select");
                 setSelectedTargetNumber(null);
                 setSelectedCard(null);
                 setShuffledNumbers(numbers);
                 setCorrectPosition(0);
                 
                 // Call optional reset callback
                 if (onGameReset) {
                   onGameReset();
                 }
               };
             
               // Dynamic title based on game phase
               const getPhaseTitle = () => {
                 switch (gamePhase) {
                   case "select":
                     return "Choose Your Lucky Number!";
                   case "ready":
                     return `Get Ready! Your number: ${selectedTargetNumber}`;
                   case "shuffle":
                     return "Shuffling...";
                   case "guess":
                     return `Find: ${selectedTargetNumber}`;
                   case "failed":
                     return "Game Over!";
                   default:
                     return "";
                 }
               };
             
               // Get current number array based on game phase
               const getCurrentNumbers = () => {
                 if (gamePhase === "select" || gamePhase === "ready") {
                   return numbers; // Show original positions
                 }
                 return shuffledNumbers; // Show shuffled positions
               };
             
               // Determine whether to show number or hide it
               const showNumber = (number: number, index: number) => {
                 // Always show during select, ready, and failed phases
                 if (gamePhase === "select" || gamePhase === "ready" ||              gamePhase === "failed") {
                   return true;
                 }
                 // During guess phase, only show if card is selected
                 return selectedCard === index;
               };
             
               return (
                 <View style={styles.container}>
                   {/* Dynamic game title */}
                   <Text style={styles.title}>{getPhaseTitle()}</Text>
             
                   {/* Countdown timer (only during ready phase) */}
                   {gamePhase === "ready" && (
                     <View style={styles.countdownContainer}>
                       <Text style={styles.countdown}>{countdown}</Text>
                     </View>
                   )}
             
                   {/* Numbers grid */}
                   <View style={styles.numbersContainer}>
                     {getCurrentNumbers().map((number, index) => {
                       const isTargetNumber =
                         selectedTargetNumber === number && gamePhase !== "guess";
                       const isSelected = selectedCard === index;
             
                       return (
                         <TouchableOpacity
                           key={`${gamePhase}-${index}`}
                           style={[
                             styles.numberCard,
                             isTargetNumber && styles.targetCard, // Highlight selected target
                             isSelected && styles.selectedCard,   // Highlight player's guess
                           ]}
                           onPress={() => {
                             if (gamePhase === "select") {
                               handleTargetNumberSelect(number);
                             } else if (gamePhase === "guess") {
                               handleCardPress(index);
                             }
                           }}
                           disabled={
                             gamePhase === "ready" ||
                             gamePhase === "shuffle" ||
                             gamePhase === "failed"
                           }
                         >
                           <Text style={styles.numberText}>
                             {showNumber(number, index) ? number : "?"}
                           </Text>
                         </TouchableOpacity>
                       );
                     })}
                   </View>
             
                   {/* Dynamic instructions */}
                   <View style={styles.instructionContainer}>
                     <Text style={styles.instructionText}>
                       {gamePhase === "select" && "Tap a number to begin!"}
                       {gamePhase === "ready" && `Shuffling in ${countdown} seconds...`}
                       {gamePhase === "shuffle" && "Shuffling numbers..."}
                       {gamePhase === "guess" && "Tap where you think your number is!"}
                       {gamePhase === "failed" && "Better luck next time!"}
                     </Text>
                   </View>
             
                   {/* Reset button (show after game ends) */}
                   {(gamePhase === "guess" || gamePhase === "failed") && (
                     <TouchableOpacity style={styles.resetButton} onPress= {resetGame}>
                       <Text style={styles.resetButtonText}>New Game</Text>
                     </TouchableOpacity>
                   )}
                 </View>
               );
             };
             ```
    
  2. **Implement Wallet-Gated Access**
       - Update your protected home screen to integrate the game with wallet requirements. Replace **`app/(tabs)/protected/index.tsx`:** with what's below:

          ```jsx
          import { useOpenfort, useWallets } from "@openfort/react-native";
          import { useState } from "react";
          import { Alert, StyleSheet, Text, View } from "react-native";
          import { SafeAreaView } from "react-native-safe-area-context";
          import { NumberPreview } from "../../../components/ NumberPreview"; // Updated import
          
          export default function HomeScreen() {
            const { user } = useOpenfort(); // Get authenticated user
            const { activeWallet } = useWallets(); // Get active wallet state
            const [gameNumbers] = useState([7, 23, 91, 45, 8]); // Static game numbers
          
            // Handle game result with user feedback
            const handleNumberSelect = (selectedNumber: number, isCorrect: boolean) => {
              if (isCorrect) {
                Alert.alert(
                  "🎉 Congratulations!", 
                  `You found ${selectedNumber} in the right position!`
                );
              } else {
                Alert.alert(
                  "❌ Close!", 
                  `${selectedNumber} was in a different position. Try again!`
                );
              }
            };
          
          
            return (
              <SafeAreaView style={styles.container}>
                <View style={styles.content}>
                  {/* Header section */}
                  <View style={styles.header}>
                    <Text style={styles.title}>Lucky Number Game</Text>
                    <Text style={styles.subtitle}>Welcome back!</Text>
                  </View>
          
                  {/* User info display */}
                  {user && (
                    <View style={styles.userInfo}>
                      <Text style={styles.userLabel}>Player ID:</Text>
                      <Text style={styles.userId}>{user.id}</Text>
                    </View>
                  )}
          
                  {/* Main game area */}
                  <View style={styles.gameContainer}>
                    {activeWallet ? (
                      // Show game when wallet is connected
                      <NumberPreview
                        numbers={gameNumbers}
                        onNumberSelect={handleNumberSelect}
                        onGameReset={handleGameReset}
                      />
                    ) : (
                      // Show wallet prompt when no wallet connected
                      <View style={styles.noWalletContainer}>
                        <View style={styles.noWalletIconContainer}>
                          <Text style={styles.noWalletIcon}>🎯</Text>
                        </View>
                        <Text style={styles.noWalletTitle}>Wallet Required</Text>
                        <Text style={styles.noWalletMessage}>
                          Connect your wallet to start playing the Lucky Number game!
                        </Text>
                        <Text style={styles.noWalletHint}>
                          💡 Go to Settings to connect your wallet
                        </Text>
                      </View>
                    )}
                  </View>
                </View>
              </SafeAreaView>
            );
          }
          
          const styles = StyleSheet.create({
            container: {
              flex: 1,
              backgroundColor: '#f8f9fa',
            },
            content: {
              flex: 1,
              paddingHorizontal: 16,
            },
          
            // Header styles
            header: {
              paddingVertical: 20,
              alignItems: 'center',
            },
            title: {
              fontSize: 28,
              fontWeight: "bold",
              color: '#333',
              marginBottom: 4,
            },
            subtitle: {
              fontSize: 16,
              color: '#666',
            },
          
            // User info display
            userInfo: {
              backgroundColor: 'rgba(0, 0, 0, 0.05)',
              borderRadius: 8,
              padding: 12,
              marginBottom: 20,
              flexDirection: 'row',
              alignItems: 'center',
            },
            userLabel: {
              fontSize: 14,
              fontWeight: '600',
              color: '#666',
              marginRight: 8,
            },
            userId: {
              fontSize: 14,
              fontFamily: 'monospace', // Monospace for ID display
              color: '#333',
              flex: 1,
            },
          
            // Game container
            gameContainer: {
              flex: 1,
              justifyContent: 'center',
            },
          
            // No wallet state styles
            noWalletContainer: {
              backgroundColor: 'rgba(255, 255, 255, 0.9)',
              borderRadius: 16,
              padding: 32,
              alignItems: 'center',
              marginHorizontal: 16,
              borderWidth: 1,
              borderColor: 'rgba(0, 0, 0, 0.1)',
              // Shadow for iOS
              shadowColor: '#000',
              shadowOffset: { width: 0, height: 2 },
              shadowOpacity: 0.1,
              shadowRadius: 8,
              // Elevation for Android
              elevation: 4,
            },
            noWalletIconContainer: {
              width: 80,
              height: 80,
              borderRadius: 40,
              backgroundColor: 'rgba(0, 0, 0, 0.05)',
              justifyContent: 'center',
              alignItems: 'center',
              marginBottom: 20,
            },
            noWalletIcon: {
              fontSize: 40,
            },
            noWalletTitle: {
              fontSize: 24,
              fontWeight: 'bold',
              color: '#333',
              marginBottom: 12,
              textAlign: 'center',
            },
            noWalletMessage: {
              fontSize: 16,
              color: '#666',
              textAlign: 'center',
              lineHeight: 24,
              marginBottom: 16,
            },
            noWalletHint: {
              fontSize: 14,
              color: '#888',
              textAlign: 'center',
              fontStyle: 'italic',
            },
          });
          ```
        
        - Now access the application in your simulator again,and you'd see this when a player logs in:

          ![d11.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d11.png)

        - With the Openfort **`useOpenfort`** hook, we can now retrieve our player's information.

        - Visit the Openfort dashboard to see the list of players that joined the game:
         
          ![d12.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d12.png)

### 4. Implementing the Settings Screen

The last thing you'll build in this demo is the settings screen that provides essential player management functionality, allowing players to create and manage wallets, switch between different wallets, and sign out of the application.Two helper functions form the Openfort SDK will be used: the **`useOpenfort`** hook to get currently logged in user and [**useWallets**](https://www.openfort.io/docs/products/embedded-wallet/react-native/hooks/useWallets) hook to access the function required to manage wallets. 

#### Setting Up the Settings Screen

   1. Create the **`app/(tabs)/protected/settings.tsx`** file and add the following code:
      
         ```jsx
         import { useOpenfort, useWallets } from "@openfort/react-native";
         import { useRouter } from "expo-router";
         import { Alert, StyleSheet, Text, TouchableOpacity, View } from "react-native";
         import { SafeAreaView } from "react-native-safe-area-context";
         
         export default function SettingsScreen() {
           const router = useRouter();
           const { logout } = useOpenfort(); // Get logout function from Openfort
           
           // Wallet management hooks with error handling
           const { wallets, setActiveWallet, createWallet, activeWallet, isCreating } = useWallets({
             onError: (error) => {
               console.error("Wallet Error:", error);
               Alert.alert("Wallet Error", error.message || "An error occurred with wallet operations");
             },
           });
         
           // Temporary password for demo - in production, use secure password management
           const recoveryPassword = "test123";
         
           // Handle user logout with navigation
           const handleLogout = () => {
             Alert.alert(
               "Confirm Logout",
               "Are you sure you want to sign out?",
               [
                 { text: "Cancel", style: "cancel" },
                 {
                   text: "Sign Out",
                   style: "destructive",
                   onPress: () => {
                     logout();
                     router.replace("/(tabs)"); // Navigate back to auth screen
                   },
                 },
               ]
             );
           };
         
           // Create new wallet with error handling
           const handleCreateWallet = async () => {
             try {
               await createWallet({
                 recoveryPassword: recoveryPassword,
               });
               Alert.alert("Success", "New wallet created successfully!");
             } catch (error) {
               console.error("Wallet creation failed:", error);
               Alert.alert("Error", "Failed to create wallet. Please try again.");
             }
           };
         
           // Set active wallet with comprehensive error handling
           const handleSetActiveWallet = async (walletAddress: string) => {
             try {
               await setActiveWallet({
                 address: walletAddress,
                 chainId: 43113, // Avalanche Fuji testnet
                 recoveryPassword: recoveryPassword,
                 onSuccess: () => {
                   Alert.alert("Success", `Wallet ${walletAddress.slice(0, 6)}...${walletAddress.slice(-4)} is now active`);
                 },
                 onError: (error) => {
                   Alert.alert("Error", `Failed to set active wallet: ${error.message}`);
                 },
               });
             } catch (error) {
               console.error("Set active wallet failed:", error);
               Alert.alert("Error", "Failed to activate wallet. Please try again.");
             }
           };
         
           return (
             <SafeAreaView style={styles.container}>
               <View style={styles.content}>
                 {/* Header */}
                 <View style={styles.header}>
                   <Text style={styles.title}>Settings</Text>
                   <Text style={styles.subtitle}>Manage your wallets and account</Text>
                 </View>
         
                 {/* Wallet Management Section */}
                 <View style={styles.walletContainer}>
                   <Text style={styles.walletSectionTitle}>
                     {wallets.length > 0 ? "Your Wallets" : "No Wallets Available"}
                   </Text>
                   
                   {wallets.length > 0 ? (
                     <Text style={styles.walletDescription}>
                       Tap a wallet to make it active. Active wallets can be used for transactions.
                     </Text>
                   ) : (
                     <Text style={styles.walletDescription}>
                       Create your first wallet to start playing games and making transactions.
                     </Text>
                   )}
         
                   {/* Wallet List */}
                   <View style={styles.walletList}>
                     {wallets.map((wallet, index) => {
                       const isActive = activeWallet?.address === wallet.address;
                       const displayAddress = `${wallet.address.slice(0, 8)}...${wallet.address.slice(-6)}`;
         
                       return (
                         <View key={wallet.address + index} style={styles.walletItem}>
                           <TouchableOpacity
                             style={[
                               styles.walletButton,
                               isActive && styles.activeWalletButton,
                             ]}
                             onPress={() => handleSetActiveWallet(wallet.address)}
                             disabled={isActive || wallet.isConnecting}
                           >
                             <View style={styles.walletButtonContent}>
                               <Text style={[
                                 styles.walletAddress,
                                 isActive && styles.activeWalletText,
                               ]}>
                                 {displayAddress}
                               </Text>
                               {isActive && (
                                 <View style={styles.activeBadge}>
                                   <Text style={styles.activeBadgeText}>ACTIVE</Text>
                                 </View>
                               )}
                             </View>
                           </TouchableOpacity>
         
                           {/* Connection status */}
                           {wallet.isConnecting && (
                             <Text style={styles.connectingText}>Connecting..</Text>
                           )}
                         </View>
                       );
                     })}
                   </View>
         
                   {/* Create Wallet Button */}
                   <TouchableOpacity
                     style={[
                       styles.createWalletButton,
                       isCreating && styles.disabledButton,
                     ]}
                     onPress={handleCreateWallet}
                     disabled={isCreating}
                   >
                     <Text style={styles.createWalletButtonText}>
                       {isCreating ? "Creating Wallet..." : "+ Create New Wallet"}
                     </Text>
                   </TouchableOpacity>
                 </View>
         
                 {/* Additional Settings */}
                 <View style={styles.additionalSettings}>
                   <Text style={styles.sectionTitle}>Account</Text>
                   
                   <TouchableOpacity style={styles.settingItem}>
                     <Text style={styles.settingItemText}>Backup & Recovery</Text>
                     <Text style={styles.settingItemSubtext}>Manage wallet backup</Text>
                   </TouchableOpacity>
                   
                   <TouchableOpacity style={styles.settingItem}>
                     <Text style={styles.settingItemText}>Security</Text>
                     <Text style={styles.settingItemSubtext}>Password and security settings</         Text>
                   </TouchableOpacity>
                 </View>
               </View>
         
               {/* Logout Section */}
               <View style={styles.logoutContainer}>
                 <TouchableOpacity style={styles.logoutButton} onPress={handleLogout}>
                   <Text style={styles.logoutText}>Sign Out</Text>
                 </TouchableOpacity>
               </View>
             </SafeAreaView>
           );
         }
         
         const styles = StyleSheet.create({
           container: {
             flex: 1,
             backgroundColor: '#f8f9fa',
           },
           content: {
             flex: 1,
             paddingHorizontal: 16,
           },
         
           // Header
           header: {
             paddingVertical: 24,
           },
           title: {
             fontSize: 28,
             fontWeight: "bold",
             color: '#333',
             marginBottom: 4,
           },
           subtitle: {
             fontSize: 16,
             color: '#666',
           },
         
           // Wallet Section
           walletContainer: {
             marginBottom: 32,
           },
           walletSectionTitle: {
             fontSize: 20,
             fontWeight: "bold",
             color: '#333',
             marginBottom: 8,
           },
           walletDescription: {
             fontSize: 14,
             color: '#666',
             marginBottom: 20,
             lineHeight: 20,
           },
           walletList: {
             marginBottom: 20,
           },
           walletItem: {
             marginBottom: 12,
           },
           walletButton: {
             backgroundColor: '#fff',
             borderRadius: 12,
             padding: 16,
             borderWidth: 1,
             borderColor: '#e0e0e0',
             shadowColor: '#000',
             shadowOffset: { width: 0, height: 1 },
             shadowOpacity: 0.1,
             shadowRadius: 2,
             elevation: 2,
           },
           activeWalletButton: {
             borderColor: '#28a745',
             backgroundColor: '#f8fff9',
           },
           walletButtonContent: {
             flexDirection: 'row',
             alignItems: 'center',
             justifyContent: 'space-between',
           },
           walletAddress: {
             fontSize: 16,
             fontFamily: 'monospace',
             color: '#333',
             fontWeight: '500',
           },
           activeWalletText: {
             color: '#28a745',
           },
           activeBadge: {
             backgroundColor: '#28a745',
             borderRadius: 12,
             paddingHorizontal: 8,
             paddingVertical: 4,
           },
           activeBadgeText: {
             color: '#fff',
             fontSize: 10,
             fontWeight: 'bold',
           },
           connectingText: {
             color: '#666',
             fontSize: 12,
             fontStyle: "italic",
             marginTop: 4,
             textAlign: 'center',
           },
         
           // Create Wallet Button
           createWalletButton: {
             backgroundColor: '#007AFF',
             borderRadius: 12,
             paddingVertical: 16,
             paddingHorizontal: 24,
             alignItems: 'center',
             shadowColor: '#000',
             shadowOffset: { width: 0, height: 2 },
             shadowOpacity: 0.1,
             shadowRadius: 4,
             elevation: 3,
           },
           disabledButton: {
             backgroundColor: '#ccc',
             opacity: 0.6,
           },
           createWalletButtonText: {
             color: '#fff',
             fontSize: 16,
             fontWeight: '600',
           },
         
           // Additional Settings
           additionalSettings: {
             flex: 1,
           },
           sectionTitle: {
             fontSize: 18,
             fontWeight: 'bold',
             color: '#333',
             marginBottom: 16,
           },
           settingItem: {
             backgroundColor: '#fff',
             borderRadius: 12,
             padding: 16,
             marginBottom: 8,
             borderWidth: 1,
             borderColor: '#e0e0e0',
           },
           settingItemText: {
             fontSize: 16,
             fontWeight: '500',
             color: '#333',
             marginBottom: 4,
           },
           settingItemSubtext: {
             fontSize: 14,
             color: '#666',
           },
         
           // Logout
           logoutContainer: {
             paddingVertical: 24,
             paddingHorizontal: 16,
           },
           logoutButton: {
             backgroundColor: '#ff3b30',
             borderRadius: 12,
             paddingVertical: 16,
             alignItems: 'center',
             shadowColor: '#000',
             shadowOffset: { width: 0, height: 2 },
             shadowOpacity: 0.1,
             shadowRadius: 4,
             elevation: 3,
           },
           logoutText: {
             color: '#fff',
             fontSize: 16,
             fontWeight: '600',
           },
         });
         ```

  2. **Configure Tab Navigation:**

     - Update the protected area's tab layout to include the settings screen. **Update `app/(tabs)/protected/_layout.tsx`:**
  
     ```jsx
     import { Tabs } from "expo-router";
     import React from "react";
     import { Platform } from "react-native";
     
     import { HapticTab } from "@/components/HapticTab";
     import { IconSymbol } from "@/components/ui/IconSymbol";
     import TabBarBackground from "@/components/ui/TabBarBackground";
     import { Colors } from "@/constants/Colors";
     import { useColorScheme } from "@/hooks/useColorScheme";
     
     export default function TabLayout() {
       const colorScheme = useColorScheme();
     
       return (
         <Tabs
           screenOptions={{
             tabBarActiveTintColor: Colors[colorScheme ?? "light"].tint,
             headerShown: false,
             tabBarButton: HapticTab,
             tabBarBackground: TabBarBackground,
             tabBarStyle: Platform.select({
               ios: {
                 // Floating tab bar on iOS
                 position: "absolute",
               },
               default: {},
             }),
           }}
         >
           {/* Home Tab */}
           <Tabs.Screen
             name="index"
             options={{
               title: "Home",
               tabBarIcon: ({ color }) => (
                 <IconSymbol size={28} name="house.fill" color={color} />
               ),
             }}
           />
           
           {/* Settings Tab */}
           <Tabs.Screen
             name="settings"
             options={{
               title: "Settings",
               tabBarIcon: ({ color }) => (
                 <IconSymbol size={28} name="gear" color={color} />
               ),
             }}
           />
         </Tabs>
       );
     }
     ```

  - Add the gear icon mapping for consistent cross-platform display. **Update `components/ui/IconSymbol.tsx`:**

    ```tsx
    // Fallback for using MaterialIcons on Android and web.
    import MaterialIcons from "@expo/vector-icons/MaterialIcons";
    import { SymbolViewProps, SymbolWeight } from "expo-symbols";
    import { ComponentProps } from "react";
    import { OpaqueColorValue, type StyleProp, type TextStyle } from "react-native";
    
    type IconMapping = Record<
      SymbolViewProps["name"],
      ComponentProps<typeof MaterialIcons>["name"]
    >;
    type IconSymbolName = keyof typeof MAPPING;
    
    /**
     * SF Symbols to Material Icons mappings
     * - SF Symbols: https://developer.apple.com/sf-symbols/
     * - Material Icons: https://icons.expo.fyi
     */
    const MAPPING = {
      "house.fill": "home",
      "paperplane.fill": "send",
      "chevron.left.forwardslash.chevron.right": "code",
      "chevron.right": "chevron-right",
      gear: "settings", // Added mapping for settings icon
    } as IconMapping;
    
    /**
     * Cross-platform icon component
     * Uses SF Symbols on iOS and Material Icons on Android/web
     */
    export function IconSymbol({
      name,
      size = 24,
      color,
      style,
      weight,
    }: {
      name: IconSymbolName;
      size?: number;
      color: string | OpaqueColorValue;
      style?: StyleProp<TextStyle>;
      weight?: SymbolWeight;
    }) {
      return (
        <MaterialIcons
          color={color}
          size={size}
          name={MAPPING[name]}
          style={style}
        />
      );
    }
    ```

- Delete the **`app/(tabs)/protected/explore.tsx`** screen since it's no longer used:
   ```bash
   rm app/(tabs)/protected/explore.tsx
   ```

- Navigate to the settings page to add new wallets or signout as seen below:

     ![d13.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d13.png)

- Finally, head back home and the homescreen should now be displaying the lukcy number game, as seen below:

     ![d14.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d14.png)


### 5. Wallet Overview

To view wallet details for your application users:

1. **Open the Openfort Dashboard** and navigate to the **Users** tab in the left sidebar.

2. **Select a Player Account** from the list to view their profile:
     
     ![d15.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d15.png)

3. **Expand the "Embedded Wallets" Section** to view wallet details:
 
   ![d16.png](Using%20Openfort%20with%20React%20Native%20app%202647b44e5c72808ca294cc677c509130/d16.png)

The wallet information displays the blockchain network configuration that matches your `OpenfortProvider` setup in `app/_layout.tsx`. This confirms that wallets are created on the correct network (Avalanche Fuji testnet with chain ID 43113).

#### Multi-Chain Support

The `supportedChains` configuration in your provider enables:
- Support for multiple blockchain networks
- User ability to create wallets on different chains
- Network-specific wallet management

> **Note:** Currently, your application supports only Avalanche Fuji testnet. To add additional networks, update the `supportedChains` array in your provider configuration.


### 6. Conclusion

You’ve successfully built a Web3-powered React Native app using Expo and Openfort’s embedded wallet infrastructure. Along the way, you learned how to set up secure authentication, manage wallets with Openfort, and even create a simple game where players guess the position of their lucky number.

By combining React Native with Openfort, you now have a strong foundation for building secure, scalable mobile apps that can grow with your needs. In the next article, we’ll dive deeper into signing messages, minting NFTs, and handling transactions with the Openfort React Native SDK — unlocking even more powerful features for your players.
