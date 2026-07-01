# Firebase Authentication Integration Guide

## Overview
Firebase Authentication provides simple, secure user authentication for web, mobile, and desktop applications. This guide covers best practices for integrating Firebase auth with modern frameworks.

## Core Concepts

### Authentication Methods
- **Email/Password**: Traditional username/password authentication
- **Social Authentication**: Google, Facebook, GitHub, Microsoft, Apple Sign-In
- **Phone Authentication**: SMS-based verification
- **Custom Authentication**: JWT tokens via custom backend
- **Anonymous Authentication**: Temporary user sessions

### Key Components
1. Firebase SDK initialization
2. Auth state management
3. User session handling
4. Security rules configuration
5. Error handling and user feedback

## React/Vite Implementation

### Setup Firebase
```javascript
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

### Authentication State Management
```javascript
import { onAuthStateChanged } from 'firebase/auth';

export function useAuthState() {
  const [user, setUser] = React.useState(null);
  const [loading, setLoading] = React.useState(true);

  React.useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setLoading(false);
    });
    return unsubscribe;
  }, []);

  return { user, loading };
}
```

### Email/Password Authentication
```javascript
import { createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut } from 'firebase/auth';

export async function registerUser(email, password) {
  try {
    const userCredential = await createUserWithEmailAndPassword(auth, email, password);
    return userCredential.user;
  } catch (error) {
    throw new Error(error.message);
  }
}

export async function loginUser(email, password) {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, password);
    return userCredential.user;
  } catch (error) {
    throw new Error(error.message);
  }
}

export async function logoutUser() {
  await signOut(auth);
}
```

### Protected Routes Pattern
```javascript
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const { user, loading } = useAuthState();

  if (loading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" replace />;

  return children;
}
```

## Security Best Practices

### Environment Variables
- Store Firebase config in `.env.local`
- Never commit credentials to version control
- Use VITE_ prefix for Vite environment variables

### Email Verification
```javascript
import { sendEmailVerification } from 'firebase/auth';

export async function verifyEmail() {
  if (auth.currentUser) {
    await sendEmailVerification(auth.currentUser);
  }
}
```

### Password Reset
```javascript
import { sendPasswordResetEmail } from 'firebase/auth';

export async function resetPassword(email) {
  await sendPasswordResetEmail(auth, email);
}
```

### Firebase Security Rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
  }
}
```

## Error Handling

### Common Errors
- `auth/email-already-in-use`: User already registered
- `auth/wrong-password`: Incorrect password
- `auth/user-not-found`: Email not registered
- `auth/weak-password`: Password does not meet requirements
- `auth/too-many-requests`: Too many login attempts

### User-Friendly Error Messages
```javascript
const getErrorMessage = (errorCode) => {
  const messages = {
    'auth/email-already-in-use': 'Email already registered',
    'auth/wrong-password': 'Incorrect password',
    'auth/user-not-found': 'No account found with this email',
    'auth/weak-password': 'Password must be at least 6 characters',
    'auth/too-many-requests': 'Too many failed attempts. Try again later.',
  };
  return messages[errorCode] || 'Authentication failed';
};
```

## Persistence Options

### Local Persistence (Default)
```javascript
import { setPersistence, localPersistence } from 'firebase/auth';

await setPersistence(auth, localPersistence);
```

### Session Persistence
```javascript
import { setPersistence, sessionPersistence } from 'firebase/auth';

await setPersistence(auth, sessionPersistence);
```

## Production Checklist

- [ ] Enable only necessary authentication methods
- [ ] Configure custom email templates
- [ ] Set up password policies
- [ ] Enable email verification requirement
- [ ] Configure allowed redirect URLs
- [ ] Set up OAuth credentials for social login
- [ ] Enable reCAPTCHA for bot protection
- [ ] Monitor authentication logs
- [ ] Test all authentication flows
- [ ] Implement rate limiting
- [ ] Add comprehensive error handling
- [ ] Secure environment variables

## Testing Firebase Authentication

```javascript
// Mock Firebase for testing
jest.mock('firebase/auth', () => ({
  getAuth: jest.fn(),
  onAuthStateChanged: jest.fn((auth, callback) => {
    callback({ uid: 'test-user' });
    return jest.fn(); // unsubscribe
  }),
}));
```

## Resources
- [Firebase Documentation](https://firebase.google.com/docs/auth)
- [Firebase Admin SDK](https://firebase.google.com/docs/admin/setup)
- [Security Best Practices](https://firebase.google.com/docs/auth/where-to-start)
