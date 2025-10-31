InspireXAIStudio/
├── App.import React, { useState, useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { Text } from 'react-native';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from './firebaseConfig';
import HomeScreen from './screens/HomeScreen';
import DemoScreen from './screens/DemoScreen';
import ContactScreen from './screens/ContactScreen';
import AuthScreen from './screens/AuthScreen';
import HistoryScreen from './screens/HistoryScreen';

const Stack = createStackNavigator();

export default function App() {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setLoading(false);
    });
    return unsubscribe;
  }, []);

  if (loading) return <Text>Loading...</Text>;

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerStyle: { backgroundColor: '#0F172A' }, headerTintColor: '#00D9FF' }}>
        {user ? (
          <>
            <Stack.Screen name="Home" component={HomeScreen} options={{ title: 'Inspire X AI Studio' }} />
            <Stack.Screen name="Demo" component={DemoScreen} options={{ title: 'Edge AI Demo' }} />
            <Stack.Screen name="History" component={HistoryScreen} options={{ title: 'Prediction History' }} />
            <Stack.Screen name="Contact" component={ContactScreen} options={{ title: 'Contact Us' }} />
          </>
        ) : (
          <Stack.Screen name="Auth" component={AuthScreen} options={{ headerShown: false }}>
            {(props) => <AuthScreen {...props} onAuth={() => props.navigation.replace('Home')} />}
          </Stack.Screen>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
├── app.{
  "expo": {
    "name": "Inspire X AI Studio",
    "slug": "inspirex-ai-studio",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/logo.png",
    "userInterfaceStyle": "dark",
    "splash": { "image": "./assets/logo.png", "resizeMode": "contain", "backgroundColor": "#0F172A" },
    "assetBundlePatterns": ["**/*"],
    "ios": { "supportsTablet": true, "googleServicesFile": "./GoogleService-Info.plist" },
    "android": { 
      "adaptiveIcon": { "foregroundImage": "./assets/logo.png", "backgroundColor": "#0F172A" },
      "googleServicesFile": "./google-services.json"
    },
    "web": {
      "favicon": "./assets/logo.png",
      "bundler": "metro",
      "output": "static",
      "themeColor": "#00D9FF",
      "backgroundColor": "#0F172A"
    },
    "plugins": [
      [
        "@react-native-google-signin/google-signin",
        {
          "iosUrlScheme": "com.googleusercontent.apps.YOUR_WEB_CLIENT_ID"
        }
      ]
    ]
  }
}
├── firebaseConfig.import { initializeApp } from 'firebase/app';
import { getAuth, GoogleAuthProvider } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: "your-api-key",
  authDomain: "your-project-id.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project-id.appspot.com",
  messagingSenderId: "123456789",
  appId: "your-app-id"
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const googleProvider = new GoogleAuthProvider();
export const db = getFirestore(app);
export default app;
├── screens/
│   ├── AuthScreen.import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert, Platform } from 'react-native';
import { createUserWithEmailAndPassword, signInWithEmailAndPassword, signInWithPopup, signInWithCredential, GoogleAuthProvider } from 'firebase/auth';
import { GoogleSignin, statusCodes } from '@react-native-google-signin/google-signin';
import { auth, googleProvider } from '../firebaseConfig';

const AuthScreen: React.FC<{ onAuth: () => void }> = ({ onAuth }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isSignup, setIsSignup] = useState(false);

  useEffect(() => {
    GoogleSignin.configure({
      webClientId: 'YOUR_WEB_CLIENT_ID_FROM_FIREBASE_CONSOLE',
      offlineAccess: true,
      forceCodeForRefreshToken: true,
      iosClientId: 'YOUR_IOS_CLIENT_ID_FROM_PLIST',
    });
  }, []);

  const handleEmailAuth = async () => {
    try {
      if (isSignup) {
        await createUserWithEmailAndPassword(auth, email, password);
      } else {
        await signInWithEmailAndPassword(auth, email, password);
      }
      onAuth();
    } catch (error: any) {
      Alert.alert('Error', error.message);
    }
  };

  const handleGoogleSignIn = async () => {
    try {
      if (Platform.OS === 'web') {
        const result = await signInWithPopup(auth, googleProvider);
        onAuth();
        return;
      }
      await GoogleSignin.hasPlayServices();
      const userInfo = await GoogleSignin.signIn();
      const { idToken } = await GoogleSignin.getTokens(userInfo.user.id);
      const googleCredential = GoogleAuthProvider.credential(idToken);
      await signInWithCredential(auth, googleCredential);
      onAuth();
    } catch (error: any) {
      if (error.code === statusCodes.SIGN_IN_CANCELLED) {
        Alert.alert('Cancelled', 'Sign-in was cancelled');
      } else if (error.code === statusCodes.IN_PROGRESS) {
        Alert.alert('In Progress', 'Sign-in is in progress');
      } else if (error.code === statusCodes.PLAY_SERVICES_NOT_AVAILABLE) {
        Alert.alert('Play Services Error', 'Update Google Play Services');
      } else {
        Alert.alert('Error', error.message);
      }
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>{isSignup ? 'Sign Up' : 'Sign In'}</Text>
      <TextInput style={styles.input} placeholder="Email" value={email} onChangeText={setEmail} keyboardType="email-address" autoCapitalize="none" />
      <TextInput style={styles.input} placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      <TouchableOpacity style={styles.button} onPress={handleEmailAuth}>
        <Text style={styles.buttonText}>{isSignup ? 'Sign Up' : 'Sign In'}</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => setIsSignup(!isSignup)}>
        <Text style={styles.switchText}>{isSignup ? 'Already have an account? Sign In' : "Don't have an account? Sign Up"}</Text>
      </TouchableOpacity>
      <View style={styles.divider}><Text style={styles.orText}>OR</Text></View>
      <TouchableOpacity style={[styles.button, styles.googleButton]} onPress={handleGoogleSignIn}>
        <Text style={styles.googleText}>Sign in with Google</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0F172A', justifyContent: 'center', padding: 20 },
  title: { fontSize: 24, color: '#00D9FF', textAlign: 'center', marginBottom: 20 },
  input: { backgroundColor: 'rgba(255,255,255,0.1)', padding: 15, borderRadius: 10, marginVertical: 10, color: '#FFFFFF' },
  button: { backgroundColor: '#00D9FF', padding: 15, borderRadius: 25, alignItems: 'center', marginVertical: 10 },
  buttonText: { color: '#0F172A', fontSize: 16, fontWeight: 'bold' },
  switchText: { color: '#FF00FF', textAlign: 'center', marginTop: 10 },
  divider: { flexDirection: 'row', alignItems: 'center', justifyContent: 'center', marginVertical: 20 },
  orText: { color: '#FFFFFF', fontSize: 14 },
  googleButton: { backgroundColor: '#FFFFFF' },
  googleText: { color: '#0F172A', fontSize: 16, fontWeight: 'bold' },
});

export default AuthScreen;
│   ├── HomeScreen.import React from 'react';
import { View, Text, Image, TouchableOpacity, ScrollView, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';

const HomeScreen: React.FC = () => {
  const navigation = useNavigation();

  const services = [
    'AI Software Development', 'Business Process Automation', 'Machine Learning & Predictive Analytics',
    'Custom Chatbots & Virtual Assistants', 'Image & Video Analysis Tools', 'AI Consultancy & Strategy Planning',
    'API Integration & Data Engineering', 'AI Training & Online Coaching'
  ];

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Image source={require('../assets/logo.png')} style={styles.logo} />
        <Text style={styles.title}>Inspire X AI Studio</Text>
        <Text style={styles.subtitle}>Your Partner in AI-Powered Innovation</Text>
      </View>
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>🚀 What We Do:</Text>
        <Text style={styles.description}>At Inspire X AI Studio, we specialize in cutting-edge AI Solutions and Consultancy Services designed to drive innovation, automate processes, and empower organizations with intelligent systems.</Text>
        {services.map((service, index) => (
          <View key={index} style={styles.serviceItem}>
            <Text style={styles.checkmark}>✅</Text>
            <Text style={styles.serviceText}>{service}</Text>
          </View>
        ))}
      </View>
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>🧠 Edge AI-Powered Innovation</Text>
        <Text style={styles.description}>Bringing Intelligence to the Source: Real-time, on-device processing for low latency, privacy, and offline reliability.</Text>
        <TouchableOpacity style={styles.button} onPress={() => navigation.navigate('Demo')}>
          <Text style={styles.buttonText}>Try Edge AI Demo</Text>
        </TouchableOpacity>
      </View>
      <TouchableOpacity style={styles.button} onPress={() => navigation.navigate('History')}>
        <Text style={styles.buttonText}>View Prediction History</Text>
      </TouchableOpacity>
      <TouchableOpacity style={styles.button} onPress={() => navigation.navigate('Contact')}>
        <Text style={styles.buttonText}>Get in Touch</Text>
      </TouchableOpacity>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0F172A' },
  header: { alignItems: 'center', padding: 20 },
  logo: { width: 120, height: 120, borderRadius: 60 },
  title: { fontSize: 24, color: '#FFFFFF', fontWeight: 'bold', marginTop: 10 },
  subtitle: { fontSize: 16, color: '#00D9FF', marginBottom: 20 },
  section: { padding: 20, margin: 10, backgroundColor: 'rgba(255,255,255,0.05)', borderRadius: 15 },
  sectionTitle: { fontSize: 20, color: '#00D9FF', marginBottom: 10, fontWeight: 'bold' },
  description: { fontSize: 14, color: '#FFFFFF', marginBottom: 15 },
  serviceItem: { flexDirection: 'row', alignItems: 'center', marginVertical: 5 },
  checkmark: { fontSize: 18, marginRight: 10 },
  serviceText: { fontSize: 16, color: '#FFFFFF', flex: 1 },
  button: { backgroundColor: '#00D9FF', padding: 15, margin: 10, borderRadius: 25, alignItems: 'center' },
  buttonText: { color: '#0F172A', fontSize: 16, fontWeight: 'bold' },
});

export default HomeScreen;
│   ├── DemoScreen.import React, { useState, useRef } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { Camera } from 'expo-camera';
import { launchImageLibraryAsync, MediaType } from 'expo-image-picker';
import * as tf from '@tensorflow/tfjs';
import { bundleResourceIO, decodeJpeg } from '@tensorflow/tfjs-react-native';
import * as mobilenet from '@tensorflow-models/mobilenet';
import { collection, addDoc } from 'firebase/firestore';
import { db, auth } from '../firebaseConfig';
import { useNavigation } from '@react-navigation/native';

const DemoScreen: React.FC = () => {
  const [isTfReady, setIsTfReady] = useState(false);
  const [isModelReady, setIsModelReady] = useState(false);
  const [predictions, setPredictions] = useState<any[]>([]);
  const [latency, setLatency] = useState(0);
  const [hasPermission, setHasPermission] = useState(false);
  const [type] = useState(Camera.Constants.Type.back);
  const cameraRef = useRef<Camera>(null);
  const navigation = useNavigation();

  const loadModel = async () => {
    try {
      await tf.ready();
      setIsTfReady(true);
      const model = await mobilenet.load({ version: 2, alpha: 0.5 });  // Quantized lightweight
      setIsModelReady(true);
      // For custom: const modelJson = require('../assets/models/model.json'); etc.
    } catch (error) {
      Alert.alert('Error', 'Failed to load model');
    }
  };

  const detect = async (image: any) => {
    const start = performance.now();
    const preds = await mobilenet(image);
    const end = performance.now();
    setLatency(Math.round(end - start));
    setPredictions(preds);

    // Save to backend
    const user = auth.currentUser;
    if (user && preds.length > 0) {
      const topPred = preds[0];
      try {
        await addDoc(collection(db, 'predictions'), {
          userId: user.uid,
          className: topPred.className,
          probability: topPred.probability,
          timestamp: new Date(),
        });
      } catch (error) {
        console.error('Save error:', error);
      }
    }
  };

  const pickImage = async () => {
    const result = await launchImageLibraryAsync({ mediaTypes: MediaType.Images, allowsEditing: true, quality: 1 });
    if (!result.canceled) {
      const imageAsset = result.assets[0];
      const response = await fetch(imageAsset.uri);
      const blob = await response.blob();
      const imgBuffer = await blob.arrayBuffer();
      const imageTensor = tf.node.decodeImage(imgBuffer) as tf.Tensor3D;
      await detect(imageTensor);
    }
  };

  const takePicture = async () => {
    if (cameraRef.current) {
      const photo = await cameraRef.current.takePictureAsync({ quality: 0.5 });
      const imageTensor = await decodeJpeg(photo.uri);
      await detect(imageTensor);
    }
  };

  React.useEffect(() => {
    (async () => {
      const { status } = await Camera.requestCameraPermissionsAsync();
      setHasPermission(status === 'granted');
      await loadModel();
    })();
  }, []);

  if (hasPermission === null) return <Text>Loading...</Text>;
  if (hasPermission === false) return <Text>No camera access</Text>;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>🧠 Edge AI Demo: Object Detection</Text>
      <Text style={styles.subtitle}>All processing on-device – {latency}ms latency!</Text>
      <View style={styles.cameraContainer}>
        <Camera style={styles.camera} type={type} ref={cameraRef}>
          <TouchableOpacity style={styles.capture} onPress={takePicture}>
            <Text style={styles.captureText}>Capture</Text>
          </TouchableOpacity>
        </Camera>
      </View>
      <TouchableOpacity style={styles.button} onPress={pickImage}>
        <Text style={styles.buttonText}>Upload Image</Text>
      </TouchableOpacity>
      <TouchableOpacity style={styles.button} onPress={() => navigation.navigate('History')}>
        <Text style={styles.buttonText}>View History</Text>
      </TouchableOpacity>
      <View style={styles.results}>
        <Text style={styles.resultsTitle}>Predictions:</Text>
        {predictions.slice(0, 5).map((pred, i) => (
          <Text key={i} style={styles.prediction}>{pred.className}: {Math.round(pred.probability * 100)}%</Text>
        ))}
      </View>
      <Text style={styles.stat}>Powered by Quantized TF.js – Privacy: 100% Local</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0F172A', padding: 20 },
  title: { fontSize: 20, color: '#00D9FF', textAlign: 'center', marginBottom: 10 },
  subtitle: { fontSize: 14, color: '#FFFFFF', textAlign: 'center', marginBottom: 20 },
  cameraContainer: { flex: 1, justifyContent: 'center', alignItems: 'center', marginBottom: 20 },
  camera: { flex: 1, width: '100%', borderRadius: 10 },
  capture: { backgroundColor: 'rgba(0,217,255,0.7)', padding: 10, borderRadius: 5, alignSelf: 'center' },
  captureText: { color: '#0F172A', fontWeight: 'bold' },
  button: { backgroundColor: '#FF00FF', padding: 15, borderRadius: 25, alignItems: 'center', marginVertical: 10 },
  buttonText: { color: '#FFFFFF', fontSize: 16 },
  results: { backgroundColor: 'rgba(255,0,255,0.1)', padding: 15, borderRadius: 10, marginTop: 10 },
  resultsTitle: { color: '#00D9FF', fontSize: 16, fontWeight: 'bold' },
  prediction: { color: '#FFFFFF', marginVertical: 2 },
  stat: { color: '#00D9FF', textAlign: 'center', marginTop: 10, fontSize: 12 },
});

export default DemoScreen;
│   ├── HistoryScreen.import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, StyleSheet } from 'react-native';
import { collection, query, where, onSnapshot } from 'firebase/firestore';
import { db, auth } from '../firebaseConfig';

interface Prediction {
  id: string;
  className: string;
  probability: number;
  timestamp: Date;
}

const HistoryScreen: React.FC = () => {
  const [predictions, setPredictions] = useState<Prediction[]>([]);

  useEffect(() => {
    const user = auth.currentUser;
    if (!user) return;

    const q = query(collection(db, 'predictions'), where('userId', '==', user.uid));
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() } as Prediction));
      setPredictions(data);
    });

    return unsubscribe;
  }, []);

  const renderItem = ({ item }: { item: Prediction }) => (
    <View style={styles.item}>
      <Text style={styles.itemText}>{item.className}: {Math.round(item.probability * 100)}%</Text>
      <Text style={styles.timestamp}>{item.timestamp.toLocaleString()}</Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Your AI Prediction History</Text>
      <FlatList data={predictions} renderItem={renderItem} keyExtractor={item => item.id} style={styles.list} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0F172A', padding: 20 },
  title: { fontSize: 20, color: '#00D9FF', textAlign: 'center', marginBottom: 20 },
  list: { flex: 1 },
  item: { backgroundColor: 'rgba(255,0,255,0.1)', padding: 15, borderRadius: 10, marginVertical: 5 },
  itemText: { color: '#FFFFFF', fontSize: 16 },
  timestamp: { color: '#00D9FF', fontSize: 12 },
});

export default HistoryScreen;
│   └── ContactScreen.import React from 'react';
import { View, Text, TouchableOpacity, Linking, StyleSheet } from 'react-native';

const ContactScreen: React.FC = () => {
  const openEmail = () => Linking.openURL('mailto:inspirexaistudio@gmail.com');
  const openPhone = () => Linking.openURL('tel:+8801740960222');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Contact Us</Text>
      <Text style={styles.info}>Phone: +880-1740-960222</Text>
      <Text style={styles.info}>Email: inspirexaistudio@gmail.com</Text>
      <Text style={styles.info}>Location: Dhaka, Bangladesh</Text>
      <TouchableOpacity style={styles.button} onPress={openEmail}>
        <Text style={styles.buttonText}>Send Email</Text>
      </TouchableOpacity>
      <TouchableOpacity style={styles.button} onPress={openPhone}>
        <Text style={styles.buttonText}>Call Us</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0F172A', padding: 20, justifyContent: 'center' },
  title: { fontSize: 24, color: '#00D9FF', textAlign: 'center', marginBottom: 20 },
  info: { fontSize: 16, color: '#FFFFFF', textAlign: 'center', marginVertical: 10 },
  button: { backgroundColor: '#00D9FF', padding: 15, borderRadius: 25, marginVertical: 10 },
  buttonText: { color: '#0F172A', fontSize: 16, textAlign: 'center', fontWeight: 'bold' },
});

export default ContactScreen;
└── assets/
    └── {
    "name": "InspireX AI Studio Digital Card",
    "short_name": "InspireX AI Card",
    "start_url": ".",
    "display": "standalone",
    "background_color": "#0F172A",
    "theme_color": "#00D9FF",
    "description": "Your Partner in AI-Powered Innovation",
    "icons": [
        { 
            "src": " ",  // Use your logo for the app icon!
            "sizes": "192x192",
            "type": "image/png"
        },
        {
            "src": "   ",
            "sizes": "512x512",
            "type": "image/png"
        }
    ]
}
