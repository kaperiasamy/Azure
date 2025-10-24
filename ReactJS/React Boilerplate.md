# Complete React Boilerplate with Best Practices

A comprehensive, production-ready React application showcasing modern patterns, hooks, and architectural best practices.

---

## Project Structure

```
react-boilerplate/
├── public/
│   ├── index.html
│   └── favicon.ico
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Loader.tsx
│   │   │   ├── ErrorBoundary.tsx
│   │   │   └── index.ts
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── MainLayout.tsx
│   │   ├── features/
│   │   │   ├── dashboard/
│   │   │   │   ├── Dashboard.tsx
│   │   │   │   ├── StatCard.tsx
│   │   │   │   └── ChartWidget.tsx
│   │   │   ├── users/
│   │   │   │   ├── UserList.tsx
│   │   │   │   ├── UserCard.tsx
│   │   │   │   ├── UserForm.tsx
│   │   │   │   └── UserDetail.tsx
│   │   │   └── products/
│   │   │       ├── ProductList.tsx
│   │   │       ├── ProductCard.tsx
│   │   │       └── ProductFilter.tsx
│   │   └── forms/
│   │       └── FormExample.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useFetch.ts
│   │   ├── useForm.ts
│   │   ├── useDebounce.ts
│   │   ├── useLocalStorage.ts
│   │   ├── usePrevious.ts
│   │   ├── useAsync.ts
│   │   └── index.ts
│   ├── contexts/
│   │   ├── AuthContext.tsx
│   │   ├── ThemeContext.tsx
│   │   └── NotificationContext.tsx
│   ├── services/
│   │   ├── api/
│   │   │   ├── axiosInstance.ts
│   │   │   ├── userService.ts
│   │   │   ├── productService.ts
│   │   │   └── authService.ts
│   │   └── localStorage.ts
│   ├── types/
│   │   ├── index.ts
│   │   ├── user.types.ts
│   │   ├── product.types.ts
│   │   └── api.types.ts
│   ├── utils/
│   │   ├── validation.ts
│   │   ├── formatters.ts
│   │   ├── constants.ts
│   │   └── errorHandler.ts
│   ├── store/
│   │   ├── store.ts
│   │   ├── slices/
│   │   │   ├── userSlice.ts
│   │   │   ├── productSlice.ts
│   │   │   └── uiSlice.ts
│   │   └── hooks.ts
│   ├── pages/
│   │   ├── HomePage.tsx
│   │   ├── LoginPage.tsx
│   │   ├── DashboardPage.tsx
│   │   ├── UsersPage.tsx
│   │   ├── ProductsPage.tsx
│   │   ├── NotFoundPage.tsx
│   │   └── ErrorPage.tsx
│   ├── App.tsx
│   ├── index.tsx
│   └── styles/
│       ├── globals.css
│       ├── variables.css
│       └── animations.css
├── .env.example
├── .eslintrc.json
├── .prettierrc.json
├── tsconfig.json
├── package.json
└── README.md
```

---

## 1. Package.json

```json
{
  "name": "react-boilerplate",
  "version": "1.0.0",
  "description": "Production-ready React boilerplate with best practices",
  "main": "index.tsx",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css}\"",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:coverage": "vitest --coverage"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "redux": "^4.2.1",
    "@reduxjs/toolkit": "^1.9.7",
    "react-redux": "^8.1.3",
    "axios": "^1.6.2",
    "clsx": "^2.0.0",
    "date-fns": "^2.30.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.37",
    "@types/react-dom": "^18.2.15",
    "@types/node": "^20.10.0",
    "typescript": "^5.3.2",
    "vite": "^5.0.8",
    "@vitejs/plugin-react": "^4.2.1",
    "eslint": "^8.55.0",
    "@typescript-eslint/eslint-plugin": "^6.13.2",
    "@typescript-eslint/parser": "^6.13.2",
    "prettier": "^3.1.0",
    "vitest": "^1.0.4",
    "@vitest/ui": "^1.0.4",
    "@testing-library/react": "^14.1.2",
    "@testing-library/jest-dom": "^6.1.5"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}
```

---

## 2. Types

### types/index.ts

```typescript
// types/index.ts
export * from './user.types';
export * from './product.types';
export * from './api.types';

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest',
}

export enum NotificationType {
  SUCCESS = 'success',
  ERROR = 'error',
  WARNING = 'warning',
  INFO = 'info',
}

export interface Pagination {
  page: number;
  limit: number;
  total: number;
  pages: number;
}

export interface RequestConfig {
  headers?: Record<string, string>;
  params?: Record<string, any>;
  timeout?: number;
}
```

### types/user.types.ts

```typescript
// types/user.types.ts
import { UserRole } from './index';

export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  role: UserRole;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface AuthResponse {
  user: User;
  token: string;
  refreshToken?: string;
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface RegisterData extends LoginCredentials {
  name: string;
  confirmPassword: string;
}

export interface UserListResponse {
  data: User[];
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
}
```

### types/product.types.ts

```typescript
// types/product.types.ts
export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  image: string;
  category: string;
  stock: number;
  rating: number;
  reviews: number;
  createdAt: string;
  updatedAt: string;
}

export interface ProductFilter {
  category?: string;
  priceRange?: [number, number];
  rating?: number;
  inStock?: boolean;
  searchTerm?: string;
}

export interface ProductListResponse {
  data: Product[];
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
}
```

### types/api.types.ts

```typescript
// types/api.types.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
  timestamp: string;
}

export interface ApiError {
  status: number;
  message: string;
  errors?: Record<string, string[]>;
  timestamp: string;
}

export interface RequestState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}
```

---

## 3. Constants & Utils

### utils/constants.ts

```typescript
// utils/constants.ts
export const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3001/api';

export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  REGISTER: '/register',
  DASHBOARD: '/dashboard',
  USERS: '/users',
  USER_DETAIL: '/users/:id',
  PRODUCTS: '/products',
  PRODUCT_DETAIL: '/products/:id',
  NOT_FOUND: '/404',
};

export const USER_ROLES = {
  ADMIN: 'admin',
  USER: 'user',
  GUEST: 'guest',
};

export const PAGINATION = {
  DEFAULT_PAGE: 1,
  DEFAULT_LIMIT: 10,
  PAGE_SIZE_OPTIONS: [10, 20, 50, 100],
};

export const VALIDATION_RULES = {
  EMAIL_REGEX: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  PASSWORD_MIN_LENGTH: 8,
  PASSWORD_REGEX: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
  NAME_MIN_LENGTH: 2,
  NAME_MAX_LENGTH: 50,
  PHONE_REGEX: /^[+]?[(]?[0-9]{3}[)]?[-\s.]?[0-9]{3}[-\s.]?[0-9]{4,6}$/,
};

export const ERROR_MESSAGES = {
  REQUIRED_FIELD: 'This field is required',
  INVALID_EMAIL: 'Please enter a valid email address',
  INVALID_PASSWORD: 'Password must contain uppercase, lowercase, number, and special character',
  PASSWORDS_NOT_MATCH: 'Passwords do not match',
  INVALID_NAME: 'Name must be between 2-50 characters',
  NETWORK_ERROR: 'Network error. Please check your connection.',
  SERVER_ERROR: 'Server error. Please try again later.',
  UNAUTHORIZED: 'Unauthorized. Please login.',
  FORBIDDEN: 'You do not have permission to perform this action.',
  NOT_FOUND: 'Resource not found.',
};

export const SUCCESS_MESSAGES = {
  LOGIN_SUCCESS: 'Login successful!',
  LOGOUT_SUCCESS: 'Logout successful!',
  CREATE_SUCCESS: 'Item created successfully!',
  UPDATE_SUCCESS: 'Item updated successfully!',
  DELETE_SUCCESS: 'Item deleted successfully!',
};

export const API_TIMEOUT = 10000; // 10 seconds
export const DEBOUNCE_DELAY = 300; // 300ms
export const TOAST_DURATION = 5000; // 5 seconds
```

### utils/validation.ts

```typescript
// utils/validation.ts
import { VALIDATION_RULES, ERROR_MESSAGES } from './constants';

export const validateEmail = (email: string): string | null => {
  if (!email) return ERROR_MESSAGES.REQUIRED_FIELD;
  if (!VALIDATION_RULES.EMAIL_REGEX.test(email)) {
    return ERROR_MESSAGES.INVALID_EMAIL;
  }
  return null;
};

export const validatePassword = (password: string): string | null => {
  if (!password) return ERROR_MESSAGES.REQUIRED_FIELD;
  if (password.length < VALIDATION_RULES.PASSWORD_MIN_LENGTH) {
    return `Password must be at least ${VALIDATION_RULES.PASSWORD_MIN_LENGTH} characters`;
  }
  if (!VALIDATION_RULES.PASSWORD_REGEX.test(password)) {
    return ERROR_MESSAGES.INVALID_PASSWORD;
  }
  return null;
};

export const validateName = (name: string): string | null => {
  if (!name) return ERROR_MESSAGES.REQUIRED_FIELD;
  if (
    name.length < VALIDATION_RULES.NAME_MIN_LENGTH ||
    name.length > VALIDATION_RULES.NAME_MAX_LENGTH
  ) {
    return ERROR_MESSAGES.INVALID_NAME;
  }
  return null;
};

export const validatePasswordMatch = (password: string, confirmPassword: string): string | null => {
  if (password !== confirmPassword) {
    return ERROR_MESSAGES.PASSWORDS_NOT_MATCH;
  }
  return null;
};

export const validateRequired = (value: any, fieldName: string): string | null => {
  if (!value || (typeof value === 'string' && !value.trim())) {
    return `${fieldName} is required`;
  }
  return null;
};
```

### utils/formatters.ts

```typescript
// utils/formatters.ts
import { format, formatDistanceToNow } from 'date-fns';

export const formatDate = (date: string | Date, formatStr = 'yyyy-MM-dd'): string => {
  try {
    return format(new Date(date), formatStr);
  } catch {
    return '';
  }
};

export const formatTime = (date: string | Date): string => {
  try {
    return format(new Date(date), 'HH:mm:ss');
  } catch {
    return '';
  }
};

export const formatDateTime = (date: string | Date): string => {
  try {
    return format(new Date(date), 'yyyy-MM-dd HH:mm:ss');
  } catch {
    return '';
  }
};

export const formatRelativeTime = (date: string | Date): string => {
  try {
    return formatDistanceToNow(new Date(date), { addSuffix: true });
  } catch {
    return '';
  }
};

export const formatCurrency = (amount: number, currency = 'USD'): string => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount);
};

export const formatNumber = (num: number, decimals = 2): string => {
  return num.toFixed(decimals);
};

export const truncateString = (str: string, maxLength: number): string => {
  if (str.length <= maxLength) return str;
  return `${str.slice(0, maxLength)}...`;
};

export const capitalizeFirstLetter = (str: string): string => {
  return str.charAt(0).toUpperCase() + str.slice(1);
};
```

### utils/errorHandler.ts

```typescript
// utils/errorHandler.ts
import { ApiError } from '../types';
import { ERROR_MESSAGES } from './constants';

export class AppError extends Error {
  constructor(
    public status: number,
    public message: string,
    public errors?: Record<string, string[]>
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export const handleApiError = (error: any): AppError => {
  if (error.response) {
    // Server responded with error status
    const { status, data } = error.response;
    return new AppError(
      status,
      data.message || ERROR_MESSAGES.SERVER_ERROR,
      data.errors
    );
  }

  if (error.request) {
    // Request made but no response
    return new AppError(
      0,
      ERROR_MESSAGES.NETWORK_ERROR
    );
  }

  // Other errors
  return new AppError(
    500,
    error.message || ERROR_MESSAGES.SERVER_ERROR
  );
};

export const getErrorMessage = (error: any): string => {
  if (error instanceof AppError) {
    return error.message;
  }

  if (error?.response?.data?.message) {
    return error.response.data.message;
  }

  if (error?.message) {
    return error.message;
  }

  return ERROR_MESSAGES.SERVER_ERROR;
};

export const getFieldError = (
  errors: Record<string, string[]> | undefined,
  fieldName: string
): string | null => {
  if (!errors || !errors[fieldName]) return null;
  return errors[fieldName][0];
};
```

---

## 4. Services

### services/api/axiosInstance.ts

```typescript
// services/api/axiosInstance.ts
import axios, { AxiosInstance, AxiosError } from 'axios';
import { API_BASE_URL, API_TIMEOUT } from '../../utils/constants';
import { handleApiError } from '../../utils/errorHandler';

const axiosInstance: AxiosInstance = axios.create({
  baseURL: API_BASE_URL,
  timeout: API_TIMEOUT,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
axiosInstance.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('authToken');
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
axiosInstance.interceptors.response.use(
  (response) => response.data,
  (error: AxiosError) => {
    const appError = handleApiError(error);

    // Handle 401 - redirect to login
    if (appError.status === 401) {
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }

    return Promise.reject(appError);
  }
);

export default axiosInstance;
```

### services/api/authService.ts

```typescript
// services/api/authService.ts
import axiosInstance from './axiosInstance';
import { AuthResponse, LoginCredentials, RegisterData, User } from '../../types';

export const authService = {
  login: async (credentials: LoginCredentials): Promise<AuthResponse> => {
    const response = await axiosInstance.post('/auth/login', credentials);
    if (response.token) {
      localStorage.setItem('authToken', response.token);
    }
    return response;
  },

  register: async (data: RegisterData): Promise<AuthResponse> => {
    const response = await axiosInstance.post('/auth/register', data);
    if (response.token) {
      localStorage.setItem('authToken', response.token);
    }
    return response;
  },

  logout: (): void => {
    localStorage.removeItem('authToken');
  },

  getCurrentUser: async (): Promise<User> => {
    return axiosInstance.get('/auth/me');
  },

  refreshToken: async (): Promise<AuthResponse> => {
    return axiosInstance.post('/auth/refresh');
  },

  verifyEmail: async (token: string): Promise<void> => {
    return axiosInstance.post('/auth/verify-email', { token });
  },

  requestPasswordReset: async (email: string): Promise<void> => {
    return axiosInstance.post('/auth/forgot-password', { email });
  },

  resetPassword: async (token: string, newPassword: string): Promise<void> => {
    return axiosInstance.post('/auth/reset-password', { token, newPassword });
  },
};
```

### services/api/userService.ts

```typescript
// services/api/userService.ts
import axiosInstance from './axiosInstance';
import { User, UserListResponse, Pagination } from '../../types';

export const userService = {
  getUsers: async (page = 1, limit = 10): Promise<UserListResponse> => {
    return axiosInstance.get('/users', {
      params: { page, limit },
    });
  },

  getUserById: async (id: string): Promise<User> => {
    return axiosInstance.get(`/users/${id}`);
  },

  createUser: async (userData: Partial<User>): Promise<User> => {
    return axiosInstance.post('/users', userData);
  },

  updateUser: async (id: string, userData: Partial<User>): Promise<User> => {
    return axiosInstance.put(`/users/${id}`, userData);
  },

  deleteUser: async (id: string): Promise<void> => {
    return axiosInstance.delete(`/users/${id}`);
  },

  searchUsers: async (searchTerm: string, page = 1, limit = 10) => {
    return axiosInstance.get('/users/search', {
      params: { q: searchTerm, page, limit },
    });
  },

  bulkDeleteUsers: async (ids: string[]): Promise<void> => {
    return axiosInstance.post('/users/bulk-delete', { ids });
  },
};
```

### services/api/productService.ts

```typescript
// services/api/productService.ts
import axiosInstance from './axiosInstance';
import { Product, ProductListResponse, ProductFilter } from '../../types';

export const productService = {
  getProducts: async (page = 1, limit = 10): Promise<ProductListResponse> => {
    return axiosInstance.get('/products', {
      params: { page, limit },
    });
  },

  getProductById: async (id: string): Promise<Product> => {
    return axiosInstance.get(`/products/${id}`);
  },

  createProduct: async (productData: Partial<Product>): Promise<Product> => {
    return axiosInstance.post('/products', productData);
  },

  updateProduct: async (id: string, productData: Partial<Product>): Promise<Product> => {
    return axiosInstance.put(`/products/${id}`, productData);
  },

  deleteProduct: async (id: string): Promise<void> => {
    return axiosInstance.delete(`/products/${id}`);
  },

  searchProducts: async (searchTerm: string, page = 1, limit = 10) => {
    return axiosInstance.get('/products/search', {
      params: { q: searchTerm, page, limit },
    });
  },

  filterProducts: async (filters: ProductFilter, page = 1, limit = 10) => {
    return axiosInstance.get('/products/filter', {
      params: { ...filters, page, limit },
    });
  },

  getFeaturedProducts: async (): Promise<Product[]> => {
    return axiosInstance.get('/products/featured');
  },
};
```

### services/localStorage.ts

```typescript
// services/localStorage.ts
export const storageService = {
  set: <T>(key: string, value: T): void => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  },

  get: <T>(key: string, defaultValue?: T): T | null => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue ?? null;
    } catch (error) {
      console.error(`Error getting localStorage key "${key}":`, error);
      return defaultValue ?? null;
    }
  },

  remove: (key: string): void => {
    try {
      localStorage.removeItem(key);
    } catch (error) {
      console.error(`Error removing localStorage key "${key}":`, error);
    }
  },

  clear: (): void => {
    try {
      localStorage.clear();
    } catch (error) {
      console.error('Error clearing localStorage:', error);
    }
  },

  getAllKeys: (): string[] => {
    try {
      return Object.keys(localStorage);
    } catch (error) {
      console.error('Error getting localStorage keys:', error);
      return [];
    }
  },
};
```

---

## 5. Custom Hooks

### hooks/useFetch.ts

```typescript
// hooks/useFetch.ts
import { useState, useEffect, useCallback, useRef } from 'react';

interface UseFetchOptions<T> {
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  skip?: boolean;
}

interface UseFetchReturn<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
  reset: () => void;
}

/**
 * Custom hook for fetching data
 * @param url - The endpoint URL
 * @param options - Configuration options
 * @returns Data, loading state, error, and refetch function
 */
export const useFetch = <T = any>(
  url: string | null,
  options: UseFetchOptions<T> = {}
): UseFetchReturn<T> => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(!options.skip);
  const [error, setError] = useState<string | null>(null);
  const abortControllerRef = useRef<AbortController | null>(null);

  const fetchData = useCallback(async () => {
    if (!url) return;

    abortControllerRef.current?.abort();
    abortControllerRef.current = new AbortController();

    setLoading(true);
    setError(null);

    try {
      const response = await fetch(url, {
        signal: abortControllerRef.current.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP Error: ${response.status}`);
      }

      const result = (await response.json()) as T;
      setData(result);
      setError(null);
      options.onSuccess?.(result);
    } catch (err) {
      if (err instanceof Error && err.name !== 'AbortError') {
        setError(err.message);
        options.onError?.(err);
      }
    } finally {
      setLoading(false);
    }
  }, [url, options]);

  useEffect(() => {
    if (!options.skip) {
      fetchData();
    }

    return () => {
      abortControllerRef.current?.abort();
    };
  }, [fetchData, options.skip]);

  const reset = useCallback(() => {
    setData(null);
    setError(null);
    setLoading(false);
  }, []);

  return { data, loading, error, refetch: fetchData, reset };
};

export default useFetch;
```

### hooks/useForm.ts

```typescript
// hooks/useForm.ts
import { useState, useCallback, useRef } from 'react';

interface UseFormOptions<T> {
  initialValues: T;
  onSubmit: (values: T) => void | Promise<void>;
  validate?: (values: T) => Record<keyof T, string>;
}

interface UseFormReturn<T> {
  values: T;
  errors: Record<keyof T, string>;
  touched: Record<keyof T, boolean>;
  isSubmitting: boolean;
  isDirty: boolean;
  handleChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleBlur: (e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleSubmit: (e: React.FormEvent<HTMLFormElement>) => Promise<void>;
  resetForm: () => void;
  setFieldValue: (field: keyof T, value: any) => void;
  setFieldError: (field: keyof T, error: string) => void;
}

/**
 * Custom hook for form state management
 */
export const useForm = <T extends Record<string, any>>({
  initialValues,
  onSubmit,
  validate,
}: UseFormOptions<T>): UseFormReturn<T> => {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<keyof T, string>>({} as any);
  const [touched, setTouched] = useState<Record<keyof T, boolean>>({} as any);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const initialValuesRef = useRef(initialValues);

  const isDirty = JSON.stringify(values) !== JSON.stringify(initialValues);

  const handleChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
      const { name, value, type, checked } = e.target as any;
      const newValue = type === 'checkbox' ? checked : value;

      setValues((prev) => ({
        ...prev,
        [name]: newValue,
      }));

      // Clear error on change
      if (errors[name as keyof T]) {
        setErrors((prev) => ({
          ...prev,
          [name]: '',
        }));
      }
    },
    [errors]
  );

  const handleBlur = useCallback(
    (e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
      const { name } = e.target;
      setTouched((prev) => ({
        ...prev,
        [name]: true,
      }));

      // Validate on blur
      if (validate) {
        const fieldErrors = validate(values);
        setErrors((prev) => ({
          ...prev,
          [name]: fieldErrors[name as keyof T] || '',
        }));
      }
    },
    [values, validate]
  );

  const handleSubmit = useCallback(
    async (e: React.FormEvent<HTMLFormElement>) => {
      e.preventDefault();

      // Mark all fields as touched
      const allTouched = Object.keys(values).reduce((acc, key) => {
        acc[key as keyof T] = true;
        return acc;
      }, {} as Record<keyof T, boolean>);
      setTouched(allTouched);

      // Validate
      if (validate) {
        const fieldErrors = validate(values);
        setErrors(fieldErrors);

        // Check if there are any errors
        if (Object.values(fieldErrors).some((err) => err)) {
          return;
        }
      }

      setIsSubmitting(true);
      try {
        await onSubmit(values);
      } finally {
        setIsSubmitting(false);
      }
    },
    [values, validate, onSubmit]
  );

  const resetForm = useCallback(() => {
    setValues(initialValuesRef.current);
    setErrors({} as any);
    setTouched({} as any);
  }, []);

  const setFieldValue = useCallback((field: keyof T, value: any) => {
    setValues((prev) => ({
      ...prev,
      [field]: value,
    }));
  }, []);

  const setFieldError = useCallback((field: keyof T, error: string) => {
    setErrors((prev) => ({
      ...prev,
      [field]: error,
    }));
  }, []);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isDirty,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
    setFieldValue,
    setFieldError,
  };
};

export default useForm;
```

### hooks/useAuth.ts

```typescript
// hooks/useAuth.ts
import { useContext } from 'react';
import { AuthContext } from '../contexts/AuthContext';

/**
 * Custom hook to use auth context
 */
export const useAuth = () => {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }

  return context;
};

export default useAuth;
```

### hooks/useDebounce.ts

```typescript
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

/**
 * Custom hook for debouncing values
 */
export const useDebounce = <T>(value: T, delay = 500): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
};

export default useDebounce;
```

### hooks/useLocalStorage.ts

```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect, useCallback } from 'react';

/**
 * Custom hook for syncing state with localStorage
 */
export const useLocalStorage = <T>(key: string, initialValue: T) => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback(
    (value: T | ((val: T) => T)) => {
      try {
        const valueToStore = value instanceof Function ? value(storedValue) : value;
        setStoredValue(valueToStore);
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      } catch (error) {
        console.error(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key, storedValue]
  );

  const removeValue = useCallback(() => {
    try {
      window.localStorage.removeItem(key);
      setStoredValue(initialValue);
    } catch (error) {
      console.error(`Error removing localStorage key "${key}":`, error);
    }
  }, [key, initialValue]);

  return [storedValue, setValue, removeValue] as const;
};

export default useLocalStorage;
```

### hooks/usePrevious.ts

```typescript
// hooks/usePrevious.ts
import { useEffect, useRef } from 'react';

/**
 * Custom hook to get previous value
 */
export const usePrevious = <T>(value: T): T | undefined => {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
};

export default usePrevious;
```

### hooks/useAsync.ts

```typescript
// hooks/useAsync.ts
import { useState, useEffect, useCallback } from 'react';

interface UseAsyncOptions<T> {
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  immediate?: boolean;
}

interface UseAsyncReturn<T> {
  status: 'idle' | 'pending' | 'success' | 'error';
  data: T | null;
  error: Error | null;
  execute: (...args: any[]) => Promise<T>;
}

/**
 * Custom hook for handling async operations
 */
export const useAsync = <T,>(
  asyncFunction: (...args: any[]) => Promise<T>,
  options: UseAsyncOptions<T> = {}
): UseAsyncReturn<T> => {
  const [status, setStatus] = useState<'idle' | 'pending' | 'success' | 'error'>('idle');
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(
    async (...args: any[]) => {
      setStatus('pending');
      setData(null);
      setError(null);

      try {
        const response = await asyncFunction(...args);
        setData(response);
        setStatus('success');
        options.onSuccess?.(response);
        return response;
      } catch (err) {
        const error = err instanceof Error ? err : new Error(String(err));
        setError(error);
        setStatus('error');
        options.onError?.(error);
        throw error;
      }
    },
    [asyncFunction, options]
  );

  useEffect(() => {
    if (options.immediate) {
      execute();
    }
  }, [execute, options.immediate]);

  return { status, data, error, execute };
};

export default useAsync;
```

### hooks/index.ts

```typescript
// hooks/index.ts
export { useAuth } from './useAuth';
export { useFetch } from './useFetch';
export { useForm } from './useForm';
export { useDebounce } from './useDebounce';
export { useLocalStorage } from './useLocalStorage';
export { usePrevious } from './usePrevious';
export { useAsync } from './useAsync';
```

---

## 6. Contexts

### contexts/AuthContext.tsx

```typescript
// contexts/AuthContext.tsx
import React, { createContext, useState, useCallback, useEffect } from 'react';
import { User, AuthResponse, LoginCredentials, RegisterData } from '../types';
import { authService } from '../services/api/authService';
import { getErrorMessage } from '../utils/errorHandler';

interface AuthContextType {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
  login: (credentials: LoginCredentials) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => void;
  clearError: () => void;
}

export const AuthContext = createContext<AuthContextType | undefined>(undefined);

interface AuthProviderProps {
  children: React.ReactNode;
}

/**
 * AuthProvider component
 */
export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Initialize auth from localStorage
  useEffect(() => {
    const storedToken = localStorage.getItem('authToken');
    if (storedToken) {
      setToken(storedToken);
      // Fetch current user
      authService
        .getCurrentUser()
        .then(setUser)
        .catch((err) => {
          console.error('Failed to load user:', err);
          localStorage.removeItem('authToken');
          setToken(null);
        })
        .finally(() => setIsLoading(false));
    } else {
      setIsLoading(false);
    }
  }, []);

  const login = useCallback(async (credentials: LoginCredentials) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await authService.login(credentials);
      setUser(response.user);
      setToken(response.token);
    } catch (err) {
      const errorMessage = getErrorMessage(err);
      setError(errorMessage);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, []);

  const register = useCallback(async (data: RegisterData) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await authService.register(data);
      setUser(response.user);
      setToken(response.token);
    } catch (err) {
      const errorMessage = getErrorMessage(err);
      setError(errorMessage);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    authService.logout();
    setUser(null);
    setToken(null);
    setError(null);
  }, []);

  const clearError = useCallback(() => {
    setError(null);
  }, []);

  const value: AuthContextType = {
    user,
    token,
    isAuthenticated: !!token && !!user,
    isLoading,
    error,
    login,
    register,
    logout,
    clearError,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export default AuthProvider;
```

### contexts/ThemeContext.tsx

```typescript
// contexts/ThemeContext.tsx
import React, { createContext, useState, useCallback, useEffect } from 'react';

type ThemeType = 'light' | 'dark';

interface ThemeContextType {
  theme: ThemeType;
  toggleTheme: () => void;
  setTheme: (theme: ThemeType) => void;
}

export const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

interface ThemeProviderProps {
  children: React.ReactNode;
}

/**
 * ThemeProvider component
 */
export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [theme, setThemeState] = useState<ThemeType>(() => {
    const storedTheme = localStorage.getItem('theme') as ThemeType | null;
    if (storedTheme) return storedTheme;

    // Check system preference
    if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
      return 'dark';
    }
    return 'light';
  });

  // Apply theme to document
  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  const setTheme = useCallback((newTheme: ThemeType) => {
    setThemeState(newTheme);
  }, []);

  const toggleTheme = useCallback(() => {
    setThemeState((prev) => (prev === 'light' ? 'dark' : 'light'));
  }, []);

  const value: ThemeContextType = {
    theme,
    setTheme,
    toggleTheme,
  };

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};

export const useTheme = () => {
  const context = React.useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};
```

### contexts/NotificationContext.tsx

```typescript
// contexts/NotificationContext.tsx
import React, { createContext, useState, useCallback } from 'react';
import { NotificationType } from '../types';

interface Notification {
  id: string;
  message: string;
  type: NotificationType;
  duration?: number;
}

interface NotificationContextType {
  notifications: Notification[];
  addNotification: (message: string, type: NotificationType, duration?: number) => string;
  removeNotification: (id: string) => void;
  clearAll: () => void;
}

export const NotificationContext = createContext<NotificationContextType | undefined>(undefined);

interface NotificationProviderProps {
  children: React.ReactNode;
}

/**
 * NotificationProvider component
 */
export const NotificationProvider: React.FC<NotificationProviderProps> = ({ children }) => {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  const addNotification = useCallback(
    (message: string, type: NotificationType, duration = 5000): string => {
      const id = Date.now().toString();

      setNotifications((prev) => [
        ...prev,
        { id, message, type, duration },
      ]);

      if (duration) {
        setTimeout(() => {
          removeNotification(id);
        }, duration);
      }

      return id;
    },
    []
  );

  const removeNotification = useCallback((id: string) => {
    setNotifications((prev) => prev.filter((notif) => notif.id !== id));
  }, []);

  const clearAll = useCallback(() => {
    setNotifications([]);
  }, []);

  const value: NotificationContextType = {
    notifications,
    addNotification,
    removeNotification,
    clearAll,
  };

  return (
    <NotificationContext.Provider value={value}>{children}</NotificationContext.Provider>
  );
};

export const useNotification = () => {
  const context = React.useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotification must be used within NotificationProvider');
  }
  return context;
};
```

---

## 7. Redux Store

### store/store.ts

```typescript
// store/store.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './slices/userSlice';
import productReducer from './slices/productSlice';
import uiReducer from './slices/uiSlice';

export const store = configureStore({
  reducer: {
    users: userReducer,
    products: productReducer,
    ui: uiReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // Ignore these action types for serialization checks
        ignoredActions: ['users/fetchUsers/pending'],
      },
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### store/slices/userSlice.ts

```typescript
// store/slices/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { User, UserListResponse } from '../../types';
import { userService } from '../../services/api/userService';

interface UserState {
  users: User[];
  selectedUser: User | null;
  loading: boolean;
  error: string | null;
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
}

const initialState: UserState = {
  users: [],
  selectedUser: null,
  loading: false,
  error: null,
  pagination: {
    page: 1,
    limit: 10,
    total: 0,
  },
};

// Async thunks
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async ({ page, limit }: { page: number; limit: number }) => {
    const response = await userService.getUsers(page, limit);
    return response;
  }
);

export const fetchUserById = createAsyncThunk(
  'users/fetchUserById',
  async (id: string) => {
    return await userService.getUserById(id);
  }
);

export const createUser = createAsyncThunk(
  'users/createUser',
  async (userData: Partial<User>) => {
    return await userService.createUser(userData);
  }
);

export const updateUser = createAsyncThunk(
  'users/updateUser',
  async ({ id, userData }: { id: string; userData: Partial<User> }) => {
    return await userService.updateUser(id, userData);
  }
);

export const deleteUser = createAsyncThunk(
  'users/deleteUser',
  async (id: string) => {
    await userService.deleteUser(id);
    return id;
  }
);

const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    setSelectedUser: (state, action: PayloadAction<User | null>) => {
      state.selectedUser = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(
        fetchUsers.fulfilled,
        (state, action: PayloadAction<UserListResponse>) => {
          state.loading = false;
          state.users = action.payload.data;
          state.pagination = {
            page: action.payload.pagination.page,
            limit: action.payload.pagination.limit,
            total: action.payload.pagination.total,
          };
        }
      )
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch users';
      })
      .addCase(
        fetchUserById.fulfilled,
        (state, action: PayloadAction<User>) => {
          state.selectedUser = action.payload;
        }
      )
      .addCase(
        createUser.fulfilled,
        (state, action: PayloadAction<User>) => {
          state.users.unshift(action.payload);
        }
      )
      .addCase(
        updateUser.fulfilled,
        (state, action: PayloadAction<User>) => {
          const index = state.users.findIndex((u) => u.id === action.payload.id);
          if (index !== -1) {
            state.users[index] = action.payload;
          }
          if (state.selectedUser?.id === action.payload.id) {
            state.selectedUser = action.payload;
          }
        }
      )
      .addCase(
        deleteUser.fulfilled,
        (state, action: PayloadAction<string>) => {
          state.users = state.users.filter((u) => u.id !== action.payload);
          if (state.selectedUser?.id === action.payload) {
            state.selectedUser = null;
          }
        }
      );
  },
});

export const { setSelectedUser, clearError } = userSlice.actions;
export default userSlice.reducer;
```

### store/slices/productSlice.ts

```typescript
// store/slices/productSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { Product, ProductListResponse, ProductFilter } from '../../types';
import { productService } from '../../services/api/productService';

interface ProductState {
  products: Product[];
  selectedProduct: Product | null;
  featuredProducts: Product[];
  loading: boolean;
  error: string | null;
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
}

const initialState: ProductState = {
  products: [],
  selectedProduct: null,
  featuredProducts: [],
  loading: false,
  error: null,
  pagination: {
    page: 1,
    limit: 10,
    total: 0,
  },
};

// Async thunks
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async ({ page, limit }: { page: number; limit: number }) => {
    return await productService.getProducts(page, limit);
  }
);

export const fetchFeaturedProducts = createAsyncThunk(
  'products/fetchFeaturedProducts',
  async () => {
    return await productService.getFeaturedProducts();
  }
);

export const filterProducts = createAsyncThunk(
  'products/filterProducts',
  async ({
    filters,
    page,
    limit,
  }: {
    filters: ProductFilter;
    page: number;
    limit: number;
  }) => {
    return await productService.filterProducts(filters, page, limit);
  }
);

const productSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {
    setSelectedProduct: (state, action: PayloadAction<Product | null>) => {
      state.selectedProduct = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(
        fetchProducts.fulfilled,
        (state, action: PayloadAction<ProductListResponse>) => {
          state.loading = false;
          state.products = action.payload.data;
          state.pagination = {
            page: action.payload.pagination.page,
            limit: action.payload.pagination.limit,
            total: action.payload.pagination.total,
          };
        }
      )
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch products';
      })
      .addCase(
        fetchFeaturedProducts.fulfilled,
        (state, action: PayloadAction<Product[]>) => {
          state.featuredProducts = action.payload;
        }
      )
      .addCase(
        filterProducts.fulfilled,
        (state, action: PayloadAction<ProductListResponse>) => {
          state.products = action.payload.data;
          state.pagination = {
            page: action.payload.pagination.page,
            limit: action.payload.pagination.limit,
            total: action.payload.pagination.total,
          };
        }
      );
  },
});

export const { setSelectedProduct, clearError } = productSlice.actions;
export default productSlice.reducer;
```

### store/hooks.ts

```typescript
// store/hooks.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

---

## 8. Common Components

### components/common/Button.tsx

```typescript
// components/common/Button.tsx
import React from 'react';
import clsx from 'clsx';
import styles from './Button.module.css';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger' | 'success';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  isFullWidth?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

/**
 * Button Component
 */
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      variant = 'primary',
      size = 'md',
      isLoading = false,
      isFullWidth = false,
      leftIcon,
      rightIcon,
      disabled,
      children,
      className,
      ...props
    },
    ref
  ) => {
    return (
      <button
        ref={ref}
        disabled={disabled || isLoading}
        className={clsx(
          styles.button,
          styles[variant],
          styles[size],
          {
            [styles.fullWidth]: isFullWidth,
            [styles.loading]: isLoading,
          },
          className
        )}
        {...props}
      >
        {leftIcon && <span className={styles.icon}>{leftIcon}</span>}
        {isLoading ? 'Loading...' : children}
        {rightIcon && <span className={styles.icon}>{rightIcon}</span>}
      </button>
    );
  }
);

Button.displayName = 'Button';

export default Button;
```

### components/common/Button.module.css

```css
/* components/common/Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border: none;
  border-radius: 0.375rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;

  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  &:focus {
    outline: 2px solid transparent;
    outline-offset: 2px;
  }
}

/* Variants */
.primary {
  background-color: var(--color-primary);
  color: white;

  &:hover:not(:disabled) {
    background-color: var(--color-primary-dark);
  }

  &:focus {
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }
}

.secondary {
  background-color: var(--color-secondary);
  color: var(--color-text);
  border: 1px solid var(--color-border);

  &:hover:not(:disabled) {
    background-color: var(--color-secondary-light);
  }
}

.danger {
  background-color: var(--color-danger);
  color: white;

  &:hover:not(:disabled) {
    background-color: var(--color-danger-dark);
  }
}

.success {
  background-color: var(--color-success);
  color: white;

  &:hover:not(:disabled) {
    background-color: var(--color-success-dark);
  }
}

/* Sizes */
.sm {
  padding: 0.375rem 0.75rem;
  font-size: 0.875rem;
}

.md {
  padding: 0.5rem 1rem;
  font-size: 1rem;
}

.lg {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

.fullWidth {
  width: 100%;
}

.loading {
  opacity: 0.8;
}

.icon {
  display: flex;
  align-items: center;
}
```

### components/common/Card.tsx

```typescript
// components/common/Card.tsx
import React from 'react';
import clsx from 'clsx';
import styles from './Card.module.css';

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: 'default' | 'outlined' | 'elevated';
  hoverable?: boolean;
  clickable?: boolean;
}

/**
 * Card Component
 */
const Card = React.forwardRef<HTMLDivElement, CardProps>(
  (
    { variant = 'default', hoverable = false, clickable = false, className, children, ...props },
    ref
  ) => {
    return (
      <div
        ref={ref}
        className={clsx(
          styles.card,
          styles[variant],
          {
            [styles.hoverable]: hoverable,
            [styles.clickable]: clickable,
          },
          className
        )}
        {...props}
      >
        {children}
      </div>
    );
  }
);

Card.displayName = 'Card';

// Compound components
interface CardHeaderProps extends React.HTMLAttributes<HTMLDivElement> {}

const CardHeader: React.FC<CardHeaderProps> = ({ className, ...props }) => (
  <div className={clsx(styles.cardHeader, className)} {...props} />
);

interface CardBodyProps extends React.HTMLAttributes<HTMLDivElement> {}

const CardBody: React.FC<CardBodyProps> = ({ className, ...props }) => (
  <div className={clsx(styles.cardBody, className)} {...props} />
);

interface CardFooterProps extends React.HTMLAttributes<HTMLDivElement> {}

const CardFooter: React.FC<CardFooterProps> = ({ className, ...props }) => (
  <div className={clsx(styles.cardFooter, className)} {...props} />
);

Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

export default Card;
```

### components/common/Card.module.css

```css
/* components/common/Card.module.css */
.card {
  border-radius: 0.5rem;
  background-color: var(--color-background);
  transition: all 0.2s ease;
}

.default {
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
}

.outlined {
  border: 1px solid var(--color-border);
}

.elevated {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

.hoverable:hover {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.15);
}

.clickable {
  cursor: pointer;

  &:active {
    transform: scale(0.98);
  }
}

.cardHeader {
  padding: 1rem;
  border-bottom: 1px solid var(--color-border);

  &:first-child {
    border-radius: 0.5rem 0.5rem 0 0;
  }
}

.cardBody {
  padding: 1rem;
}

.cardFooter {
  padding: 1rem;
  border-top: 1px solid var(--color-border);
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;

  &:last-child {
    border-radius: 0 0 0.5rem 0.5rem;
  }
}
```

### components/common/Input.tsx

```typescript
// components/common/Input.tsx
import React from 'react';
import clsx from 'clsx';
import styles from './Input.module.css';

interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  hint?: string;
  isRequired?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

/**
 * Input Component
 */
const Input = React.forwardRef<HTMLInputElement, InputProps>(
  (
    {
      label,
      error,
      hint,
      isRequired,
      leftIcon,
      rightIcon,
      disabled,
      className,
      ...props
    },
    ref
  ) => {
    return (
      <div className={styles.container}>
        {label && (
          <label className={styles.label}>
            {label}
            {isRequired && <span className={styles.required}>*</span>}
          </label>
        )}

        <div className={styles.inputWrapper}>
          {leftIcon && <span className={styles.leftIcon}>{leftIcon}</span>}

          <input
            ref={ref}
            disabled={disabled}
            className={clsx(
              styles.input,
              {
                [styles.error]: error,
                [styles.disabled]: disabled,
                [styles.withLeftIcon]: leftIcon,
                [styles.withRightIcon]: rightIcon,
              },
              className
            )}
            {...props}
          />

          {rightIcon && <span className={styles.rightIcon}>{rightIcon}</span>}
        </div>

        {error && <p className={styles.errorMessage}>{error}</p>}
        {hint && <p className={styles.hint}>{hint}</p>}
      </div>
    );
  }
);

Input.displayName = 'Input';

export default Input;
```

### components/common/Input.module.css

```css
/* components/common/Input.module.css */
.container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.label {
  font-weight: 500;
  color: var(--color-text);
  font-size: 0.875rem;
}

.required {
  color: var(--color-danger);
}

.inputWrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input {
  width: 100%;
  padding: 0.5rem 0.75rem;
  border: 1px solid var(--color-border);
  border-radius: 0.375rem;
  font-size: 1rem;
  font-family: inherit;
  background-color: var(--color-background);
  color: var(--color-text);
  transition: all 0.2s ease;

  &:focus {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }

  &:disabled {
    background-color: var(--color-secondary-light);
    cursor: not-allowed;
    opacity: 0.6;
  }

  &.error {
    border-color: var(--color-danger);

    &:focus {
      box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.1);
    }
  }

  &.withLeftIcon {
    padding-left: 2.5rem;
  }

  &.withRightIcon {
    padding-right: 2.5rem;
  }
}

.leftIcon,
.rightIcon {
  position: absolute;
  display: flex;
  align-items: center;
  color: var(--color-text-secondary);
  font-size: 1rem;
}

.leftIcon {
  left: 0.75rem;
}

.rightIcon {
  right: 0.75rem;
}

.errorMessage {
  font-size: 0.875rem;
  color: var(--color-danger);
  margin: 0;
}

.hint {
  font-size: 0.875rem;
  color: var(--color-text-secondary);
  margin: 0;
}
```

### components/common/Loader.tsx

```typescript
// components/common/Loader.tsx
import React from 'react';
import clsx from 'clsx';
import styles from './Loader.module.css';

interface LoaderProps {
  size?: 'sm' | 'md' | 'lg';
  fullScreen?: boolean;
  message?: string;
}

/**
 * Loader Component
 */
const Loader: React.FC<LoaderProps> = ({ size = 'md', fullScreen = false, message }) => {
  return (
    <div className={clsx(styles.container, { [styles.fullScreen]: fullScreen })}>
      <div className={clsx(styles.spinner, styles[size])} />
      {message && <p className={styles.message}>{message}</p>}
    </div>
  );
};

export default Loader;
```

### components/common/Loader.module.css

```css
/* components/common/Loader.module.css */
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  gap: 1rem;
}

.fullScreen {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.1);
  z-index: 9999;
}

.spinner {
  border: 3px solid var(--color-border);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.sm {
  width: 1.5rem;
  height: 1.5rem;
}

.md {
  width: 2.5rem;
  height: 2.5rem;
}

.lg {
  width: 3.5rem;
  height: 3.5rem;
}

.message {
  font-size: 1rem;
  color: var(--color-text-secondary);
  margin: 0;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
```

### components/common/Modal.tsx

```typescript
// components/common/Modal.tsx
import React, { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';
import clsx from 'clsx';
import styles from './Modal.module.css';
import Button from './Button';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
  size?: 'sm' | 'md' | 'lg';
  closeOnEscape?: boolean;
  closeOnBackdropClick?: boolean;
}

/**
 * Modal Component
 */
const Modal: React.FC<ModalProps> = ({
  isOpen,
  onClose,
  title,
  children,
  footer,
  size = 'md',
  closeOnEscape = true,
  closeOnBackdropClick = true,
}) => {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen) return;

    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape' && closeOnEscape) {
        onClose();
      }
    };

    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen, closeOnEscape, onClose]);

  if (!isOpen) return null;

  const handleBackdropClick = (e: React.MouseEvent) => {
    if (closeOnBackdropClick && e.target === e.currentTarget) {
      onClose();
    }
  };

  return createPortal(
    <div className={clsx(styles.backdrop, styles.show)} onClick={handleBackdropClick}>
      <div
        ref={modalRef}
        className={clsx(styles.modal, styles[size])}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
      >
        {title && (
          <div className={styles.header}>
            <h2 id="modal-title" className={styles.title}>
              {title}
            </h2>
            <button
              className={styles.closeButton}
              onClick={onClose}
              aria-label="Close modal"
            >
              ×
            </button>
          </div>
        )}

        <div className={styles.body}>{children}</div>

        {footer && <div className={styles.footer}>{footer}</div>}
      </div>
    </div>,
    document.body
  );
};

export default Modal;
```

### components/common/Modal.module.css

```css
/* components/common/Modal.module.css */
.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  opacity: 0;
  transition: opacity 0.2s ease;
  pointer-events: none;

  &.show {
    opacity: 1;
    pointer-events: auto;
  }
}

.modal {
  background-color: var(--color-background);
  border-radius: 0.5rem;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease;
}

.sm {
  width: 90%;
  max-width: 400px;
}

.md {
  width: 90%;
  max-width: 600px;
}

.lg {
  width: 90%;
  max-width: 800px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid var(--color-border);
}

.title {
  margin: 0;
  font-size: 1.25rem;
  color: var(--color-text);
}

.closeButton {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: var(--color-text-secondary);
  padding: 0;
  width: 2rem;
  height: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;

  &:hover {
    color: var(--color-text);
  }
}

.body {
  padding: 1.5rem;
}

.footer {
  padding: 1.5rem;
  border-top: 1px solid var(--color-border);
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
}

@keyframes slideUp {
  from {
    transform: translateY(2rem);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
```

### components/common/ErrorBoundary.tsx

```typescript
// components/common/ErrorBoundary.tsx
import React, { Component, ReactNode } from 'react';
import Card from './Card';
import Button from './Button';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, info: React.ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

/**
 * Error Boundary Component
 */
class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('Error caught:', error, info);
    this.props.onError?.(error, info);
  }

  handleReset = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <Card variant="outlined" style={{ margin: '2rem auto', maxWidth: '600px' }}>
          <Card.Body>
            <h2 style={{ color: 'var(--color-danger)', marginTop: 0 }}>
              Oops! Something went wrong
            </h2>
            <details style={{ marginBottom: '1rem', cursor: 'pointer' }}>
              <summary>Error details</summary>
              <pre style={{ fontSize: '0.875rem', overflow: 'auto' }}>
                {this.state.error?.message}
              </pre>
            </details>
            <Button variant="primary" onClick={this.handleReset}>
              Try again
            </Button>
          </Card.Body>
        </Card>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

### components/common/index.ts

```typescript
// components/common/index.ts
export { default as Button } from './Button';
export { default as Card } from './Card';
export { default as Input } from './Input';
export { default as Modal } from './Modal';
export { default as Loader } from './Loader';
export { default as ErrorBoundary } from './ErrorBoundary';
```

---

## 9. Layout Components

### components/layout/Header.tsx

```typescript
// components/layout/Header.tsx
import React, { useCallback } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from '../../hooks/useAuth';
import { useTheme } from '../../contexts/ThemeContext';
import { ROUTES } from '../../utils/constants';
import Button from '../common/Button';
import styles from './Header.module.css';

/**
 * Header Component
 */
const Header: React.FC = () => {
  const { user, isAuthenticated, logout } = useAuth();
  const { theme, toggleTheme } = useTheme();
  const navigate = useNavigate();

  const handleLogout = useCallback(() => {
    logout();
    navigate(ROUTES.LOGIN);
  }, [logout, navigate]);

  return (
    <header className={styles.header}>
      <div className={styles.container}>
        {/* Logo */}
        <Link to={ROUTES.HOME} className={styles.logo}>
          React App
        </Link>

        {/* Navigation */}
        <nav className={styles.nav}>
          <Link to={ROUTES.HOME} className={styles.navLink}>
            Home
          </Link>
          {isAuthenticated && (
            <>
              <Link to={ROUTES.DASHBOARD} className={styles.navLink}>
                Dashboard
              </Link>
              <Link to={ROUTES.USERS} className={styles.navLink}>
                Users
              </Link>
              <Link to={ROUTES.PRODUCTS} className={styles.navLink}>
                Products
              </Link>
            </>
          )}
        </nav>

        {/* Actions */}
        <div className={styles.actions}>
          <button
            className={styles.themeToggle}
            onClick={toggleTheme}
            aria-label="Toggle theme"
          >
            {theme === 'light' ? '🌙' : '☀️'}
          </button>

          {isAuthenticated ? (
            <div className={styles.userMenu}>
              <span className={styles.userName}>{user?.name}</span>
              <Button variant="secondary" size="sm" onClick={handleLogout}>
                Logout
              </Button>
            </div>
          ) : (
            <Button
              variant="primary"
              size="sm"
              onClick={() => navigate(ROUTES.LOGIN)}
            >
              Login
            </Button>
          )}
        </div>
      </div>
    </header>
  );
};

export default Header;
```

### components/layout/Header.module.css

```css
/* components/layout/Header.module.css */
.header {
  background-color: var(--color-background);
  border-bottom: 1px solid var(--color-border);
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  height: 4rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo {
  font-size: 1.25rem;
  font-weight: bold;
  color: var(--color-primary);
  text-decoration: none;
  margin-right: 2rem;

  &:hover {
    color: var(--color-primary-dark);
  }
}

.nav {
  display: flex;
  gap: 1rem;
  flex: 1;
}

.navLink {
  color: var(--color-text);
  text-decoration: none;
  padding: 0.5rem 1rem;
  border-radius: 0.375rem;
  transition: all 0.2s ease;

  &:hover {
    background-color: var(--color-secondary-light);
  }
}

.actions {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.themeToggle {
  background: none;
  border: none;
  font-size: 1.25rem;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 0.375rem;
  transition: all 0.2s ease;

  &:hover {
    background-color: var(--color-secondary-light);
  }
}

.userMenu {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.userName {
  font-weight: 500;
  color: var(--color-text);
}

@media (max-width: 768px) {
  .nav {
    display: none;
  }

  .container {
    padding: 0 0.5rem;
  }
}
```

### components/layout/MainLayout.tsx

```typescript
// components/layout/MainLayout.tsx
import React from 'react';
import Header from './Header';
import Footer from './Footer';
import styles from './MainLayout.module.css';

interface MainLayoutProps {
  children: React.ReactNode;
}

/**
 * Main Layout Component
 */
const MainLayout: React.FC<MainLayoutProps> = ({ children }) => {
  return (
    <div className={styles.layout}>
      <Header />
      <main className={styles.main}>{children}</main>
      <Footer />
    </div>
  );
};

export default MainLayout;
```

### components/layout/MainLayout.module.css

```css
/* components/layout/MainLayout.module.css */
.layout {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  padding: 2rem 1rem;
}

@media (max-width: 768px) {
  .main {
    padding: 1rem 0.5rem;
  }
}
```

### components/layout/Footer.tsx

```typescript
// components/layout/Footer.tsx
import React from 'react';
import styles from './Footer.module.css';

/**
 * Footer Component
 */
const Footer: React.FC = () => {
  const currentYear = new Date().getFullYear();

  return (
    <footer className={styles.footer}>
      <div className={styles.container}>
        <p>&copy; {currentYear} React App. All rights reserved.</p>
        <div className={styles.links}>
          <a href="#privacy">Privacy Policy</a>
          <a href="#terms">Terms of Service</a>
          <a href="#contact">Contact Us</a>
        </div>
      </div>
    </footer>
  );
};

export default Footer;
```

### components/layout/Footer.module.css

```css
/* components/layout/Footer.module.css */
.footer {
  background-color: var(--color-background);
  border-top: 1px solid var(--color-border);
  padding: 2rem 1rem;
  margin-top: auto;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.links {
  display: flex;
  gap: 1.5rem;

  a {
    color: var(--color-primary);
    text-decoration: none;

    &:hover {
      text-decoration: underline;
    }
  }
}

@media (max-width: 768px) {
  .container {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
}
```

---

## 10. Feature Components

### components/features/users/UserList.tsx

```typescript
// components/features/users/UserList.tsx
import React, { useEffect, useState, useCallback } from 'react';
import { useAppDispatch, useAppSelector } from '../../../store/hooks';
import { fetchUsers, deleteUser } from '../../../store/slices/userSlice';
import { User } from '../../../types';
import { Card, Button, Loader } from '../../common';
import UserCard from './UserCard';
import styles from './UserList.module.css';

/**
 * UserList Component
 */
const UserList: React.FC = () => {
  const dispatch = useAppDispatch();
  const { users, loading, error, pagination } = useAppSelector((state) => state.users);
  const [page, setPage] = useState(1);
  const limit = 10;

  useEffect(() => {
    dispatch(fetchUsers({ page, limit }));
  }, [dispatch, page, limit]);

  const handleDelete = useCallback(
    async (id: string) => {
      if (window.confirm('Are you sure you want to delete this user?')) {
        await dispatch(deleteUser(id));
      }
    },
    [dispatch]
  );

  if (loading && users.length === 0) {
    return <Loader message="Loading users..." />;
  }

  return (
    <div className={styles.container}>
      <h2>Users</h2>

      {error && (
        <Card variant="outlined" style={{ marginBottom: '1rem' }}>
          <Card.Body style={{ color: 'var(--color-danger)' }}>
            Error: {error}
          </Card.Body>
        </Card>
      )}

      <div className={styles.grid}>
        {users.map((user: User) => (
          <UserCard
            key={user.id}
            user={user}
            onDelete={() => handleDelete(user.id)}
          />
        ))}
      </div>

      {users.length === 0 && !loading && (
        <Card variant="outlined">
          <Card.Body>No users found</Card.Body>
        </Card>
      )}

      {/* Pagination */}
      <div className={styles.pagination}>
        <Button
          variant="secondary"
          disabled={page === 1}
          onClick={() => setPage(page - 1)}
        >
          Previous
        </Button>

        <span>
          Page {page} of {Math.ceil(pagination.total / limit)}
        </span>

        <Button
          variant="secondary"
          disabled={page >= Math.ceil(pagination.total / limit)}
          onClick={() => setPage(page + 1)}
        >
          Next
        </Button>
      </div>
    </div>
  );
};

export default UserList;
```

### components/features/users/UserCard.tsx

```typescript
// components/features/users/UserCard.tsx
import React from 'react';
import { User } from '../../../types';
import { Card, Button } from '../../common';
import { formatDateTime } from '../../../utils/formatters';
import styles from './UserCard.module.css';

interface UserCardProps {
  user: User;
  onDelete: () => void;
}

/**
 * UserCard Component
 */
const UserCard: React.FC<UserCardProps> = ({ user, onDelete }) => {
  return (
    <Card variant="outlined" hoverable>
      <Card.Body>
        <h3 className={styles.name}>{user.name}</h3>
        <p className={styles.email}>{user.email}</p>

        <div className={styles.meta}>
          <span className={styles.role}>{user.role}</span>
          <span className={styles.status}>
            {user.isActive ? '🟢 Active' : '🔴 Inactive'}
          </span>
        </div>

        <p className={styles.date}>
          Created: {formatDateTime(user.createdAt)}
        </p>
      </Card.Body>

      <Card.Footer>
        <Button variant="secondary" size="sm">
          View Details
        </Button>
        <Button variant="danger" size="sm" onClick={onDelete}>
          Delete
        </Button>
      </Card.Footer>
    </Card>
  );
};

export default UserCard;
```

### components/features/users/UserCard.module.css

```css
/* components/features/users/UserCard.module.css */
.name {
  margin: 0 0 0.5rem 0;
  font-size: 1.125rem;
  color: var(--color-text);
}

.email {
  margin: 0.25rem 0;
  color: var(--color-text-secondary);
  font-size: 0.875rem;
}

.meta {
  display: flex;
  gap: 0.5rem;
  margin: 0.75rem 0;
}

.role {
  background-color: var(--color-primary);
  color: white;
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.75rem;
  font-weight: 500;
}

.status {
  font-size: 0.875rem;
}

.date {
  margin: 0.5rem 0 0 0;
  font-size: 0.75rem;
  color: var(--color-text-secondary);
}
```

### components/features/users/UserList.module.css

```css
/* components/features/users/UserList.module.css */
.container {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1rem;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
}

@media (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr;
  }
}
```

---

## 11. Pages

### pages/LoginPage.tsx

```typescript
// pages/LoginPage.tsx
import React, { useState, useEffect } from 'react';
import { useNavigate, Link } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';
import { useForm } from '../hooks/useForm';
import { useNotification } from '../contexts/NotificationContext';
import { validateEmail, validatePassword } from '../utils/validation';
import { ROUTES, SUCCESS_MESSAGES } from '../utils/constants';
import { Card, Button, Input, Loader } from '../components/common';
import styles from './LoginPage.module.css';

interface LoginFormValues {
  email: string;
  password: string;
}

/**
 * LoginPage Component
 */
const LoginPage: React.FC = () => {
  const navigate = useNavigate();
  const { isAuthenticated, isLoading: authLoading } = useAuth();
  const { addNotification } = useNotification();
  const { login } = useAuth();

  // Redirect if already authenticated
  useEffect(() => {
    if (isAuthenticated) {
      navigate(ROUTES.DASHBOARD);
    }
  }, [isAuthenticated, navigate]);

  const validateForm = (values: LoginFormValues) => {
    const errors: Record<keyof LoginFormValues, string> = {
      email: validateEmail(values.email) || '',
      password: validatePassword(values.password) || '',
    };
    return errors;
  };

  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
  } = useForm<LoginFormValues>(
    { email: '', password: '' },
    async (values) => {
      try {
        await login(values);
        addNotification(SUCCESS_MESSAGES.LOGIN_SUCCESS, 'success');
        navigate(ROUTES.DASHBOARD);
      } catch (error) {
        addNotification('Login failed. Please check your credentials.', 'error');
      }
    },
    validateForm
  );

  if (authLoading) {
    return <Loader message="Checking authentication..." />;
  }

  return (
    <div className={styles.container}>
      <Card className={styles.card}>
        <Card.Body>
          <h1 className={styles.title}>Login</h1>
          <p className={styles.subtitle}>Welcome back!</p>

          <form onSubmit={handleSubmit} className={styles.form}>
            <Input
              label="Email"
              name="email"
              type="email"
              value={values.email}
              onChange={handleChange}
              onBlur={handleBlur}
              error={touched.email ? errors.email : undefined}
              placeholder="Enter your email"
              isRequired
            />

            <Input
              label="Password"
              name="password"
              type="password"
              value={values.password}
              onChange={handleChange}
              onBlur={handleBlur}
              error={touched.password ? errors.password : undefined}
              placeholder="Enter your password"
              isRequired
            />

            <Button
              type="submit"
              variant="primary"
              isFullWidth
              isLoading={isSubmitting}
            >
              Login
            </Button>
          </form>

          <div className={styles.footer}>
            <p>Don't have an account?</p>
            <Link to={ROUTES.REGISTER}>Create one</Link>
          </div>
        </Card.Body>
      </Card>
    </div>
  );
};

export default LoginPage;
```

### pages/LoginPage.module.css

```css
/* pages/LoginPage.module.css */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 1rem;
  background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-primary-dark) 100%);
}

.card {
  width: 100%;
  max-width: 400px;
  background-color: var(--color-background);
}

.title {
  margin-top: 0;
  font-size: 1.875rem;
  color: var(--color-text);
  text-align: center;
}

.subtitle {
  color: var(--color-text-secondary);
  text-align: center;
  margin-bottom: 1.5rem;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.footer {
  margin-top: 1.5rem;
  text-align: center;
  color: var(--color-text-secondary);
  font-size: 0.875rem;

  a {
    color: var(--color-primary);
    text-decoration: none;

    &:hover {
      text-decoration: underline;
    }
  }
}
```

### pages/DashboardPage.tsx

```typescript
// pages/DashboardPage.tsx
import React, { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '../store/hooks';
import { fetchFeaturedProducts } from '../store/slices/productSlice';
import { useAuth } from '../hooks/useAuth';
import { Card } from '../components/common';
import styles from './DashboardPage.module.css';

/**
 * DashboardPage Component
 */
const DashboardPage: React.FC = () => {
  const dispatch = useAppDispatch();
  const { user } = useAuth();
  const { featuredProducts } = useAppSelector((state) => state.products);

  useEffect(() => {
    dispatch(fetchFeaturedProducts());
  }, [dispatch]);

  return (
    <div className={styles.container}>
      <h1>Welcome, {user?.name}!</h1>

      <div className={styles.stats}>
        <Card className={styles.statCard}>
          <Card.Body>
            <h3>Total Users</h3>
            <p className={styles.statValue}>1,234</p>
          </Card.Body>
        </Card>

        <Card className={styles.statCard}>
          <Card.Body>
            <h3>Total Products</h3>
            <p className={styles.statValue}>5,678</p>
          </Card.Body>
        </Card>

        <Card className={styles.statCard}>
          <Card.Body>
            <h3>Total Revenue</h3>
            <p className={styles.statValue}>$123,456</p>
          </Card.Body>
        </Card>
      </div>

      <Card>
        <Card.Header>
          <h2>Featured Products</h2>
        </Card.Header>
        <Card.Body>
          <div className={styles.productGrid}>
            {featuredProducts.map((product) => (
              <div key={product.id} className={styles.productCard}>
                <h4>{product.name}</h4>
                <p>${product.price}</p>
              </div>
            ))}
          </div>
        </Card.Body>
      </Card>
    </div>
  );
};

export default DashboardPage;
```

### pages/DashboardPage.module.css

```css
/* pages/DashboardPage.module.css */
.container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.statCard {
  background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-primary-dark) 100%);
}

.statCard :global(div) {
  color: white;
}

.statValue {
  font-size: 2rem;
  font-weight: bold;
  margin: 0;
  color: white;
}

.productGrid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 1rem;
}

.productCard {
  padding: 1rem;
  background-color: var(--color-secondary-light);
  border-radius: 0.375rem;

  h4 {
    margin-top: 0;
    margin-bottom: 0.5rem;
  }

  p {
    margin: 0;
    font-weight: bold;
    color: var(--color-primary);
  }
}
```

---

## 12. Routing & Main App

### App.tsx

```typescript
// App.tsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import { ErrorBoundary, Loader } from './components/common';
import MainLayout from './components/layout/MainLayout';
import { useAuth } from './hooks/useAuth';
import { ROUTES } from './utils/constants';

// Pages
import HomePage from './pages/HomePage';
import LoginPage from './pages/LoginPage';
import NotFoundPage from './pages/NotFoundPage';

// Lazy loaded pages
const RegisterPage = lazy(() => import('./pages/RegisterPage'));
const DashboardPage = lazy(() => import('./pages/DashboardPage'));
const UsersPage = lazy(() => import('./pages/UsersPage'));
const ProductsPage = lazy(() => import('./pages/ProductsPage'));

/**
 * Protected Route Component
 */
const ProtectedRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <Loader />;
  }

  if (!isAuthenticated) {
    return <Navigate to={ROUTES.LOGIN} replace />;
  }

  return <>{children}</>;
};

/**
 * App Component
 */
const App: React.FC = () => {
  return (
    <ErrorBoundary>
      <Router>
        <Routes>
          {/* Public routes */}
          <Route
            path={ROUTES.HOME}
            element={
              <MainLayout>
                <HomePage />
              </MainLayout>
            }
          />

          <Route
            path={ROUTES.LOGIN}
            element={<LoginPage />}
          />

          <Route
            path={ROUTES.REGISTER}
            element={
              <Suspense fallback={<Loader />}>
                <RegisterPage />
              </Suspense>
            }
          />

          {/* Protected routes */}
          <Route
            path={ROUTES.DASHBOARD}
            element={
              <ProtectedRoute>
                <MainLayout>
                  <Suspense fallback={<Loader />}>
                    <DashboardPage />
                  </Suspense>
                </MainLayout>
              </ProtectedRoute>
            }
          />

          <Route
            path={ROUTES.USERS}
            element={
              <ProtectedRoute>
                <MainLayout>
                  <Suspense fallback={<Loader />}>
                    <UsersPage />
                  </Suspense>
                </MainLayout>
              </ProtectedRoute>
            }
          />

          <Route
            path={ROUTES.PRODUCTS}
            element={
              <ProtectedRoute>
                <MainLayout>
                  <Suspense fallback={<Loader />}>
                    <ProductsPage />
                  </Suspense>
                </MainLayout>
              </ProtectedRoute>
            }
          />

          {/* Error routes */}
          <Route
            path={ROUTES.NOT_FOUND}
            element={
              <MainLayout>
                <NotFoundPage />
              </MainLayout>
            }
          />

          {/* Catch all - 404 */}
          <Route
            path="*"
            element={<Navigate to={ROUTES.NOT_FOUND} replace />}
          />
        </Routes>
      </Router>
    </ErrorBoundary>
  );
};

export default App;
```

### index.tsx

```typescript
// index.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import App from './App';
import { AuthProvider } from './contexts/AuthContext';
import { ThemeProvider } from './contexts/ThemeContext';
import { NotificationProvider } from './contexts/NotificationContext';
import { store } from './store/store';
import './styles/globals.css';
import './styles/variables.css';
import './styles/animations.css';

const root = ReactDOM.createRoot(document.getElementById('root')!);

root.render(
  <React.StrictMode>
    <Provider store={store}>
      <AuthProvider>
        <ThemeProvider>
          <NotificationProvider>
            <App />
          </NotificationProvider>
        </ThemeProvider>
      </AuthProvider>
    </Provider>
  </React.StrictMode>
);
```

---

## 13. Styles

### styles/globals.css

```css
/* styles/globals.css */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  height: 100%;
  width: 100%;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: var(--color-background);
  color: var(--color-text);
  transition: background-color 0.2s ease, color 0.2s ease;
}

#root {
  height: 100%;
  width: 100%;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  line-height: 1.2;
}

a {
  color: inherit;
  text-decoration: none;
}

button,
input,
select,
textarea {
  font-family: inherit;
}

code {
  background-color: var(--color-secondary-light);
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  font-family: 'Courier New', monospace;
}

pre {
  background-color: var(--color-secondary-light);
  padding: 1rem;
  border-radius: 0.375rem;
  overflow-x: auto;
}
```

### styles/variables.css

```css
/* styles/variables.css */
:root {
  --color-primary: #3b82f6;
  --color-primary-dark: #2563eb;
  --color-primary-light: #93c5fd;

  --color-secondary: #f3f4f6;
  --color-secondary-light: #f9fafb;

  --color-success: #10b981;
  --color-success-dark: #059669;

  --color-danger: #ef4444;
  --color-danger-dark: #dc2626;

  --color-warning: #f59e0b;

  --color-border: #e5e7eb;
  --color-background: #ffffff;
  --color-text: #111827;
  --color-text-secondary: #6b7280;

  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);

  --transition-fast: 0.15s ease;
  --transition-normal: 0.25s ease;
  --transition-slow: 0.35s ease;
}

/* Dark theme */
[data-theme='dark'] {
  --color-background: #1f2937;
  --color-text: #f3f4f6;
  --color-text-secondary: #d1d5db;
  --color-secondary: #374151;
  --color-secondary-light: #4b5563;
  --color-border: #4b5563;
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### styles/animations.css

```css
/* styles/animations.css */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideInUp {
  from {
    transform: translateY(1rem);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@keyframes slideInDown {
  from {
    transform: translateY(-1rem);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-fade-in {
  animation: fadeIn 0.3s ease-out;
}

.animate-slide-up {
  animation: slideInUp 0.3s ease-out;
}

.animate-slide-down {
  animation: slideInDown 0.3s ease-out;
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
```

---

## 14. Configuration Files

### .env.example

```
REACT_APP_API_URL=http://localhost:3001/api
REACT_APP_ENV=development
REACT_APP_ENABLE_ANALYTICS=false
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### .eslintrc.json

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true
    }
  },
  "plugins": ["react", "@typescript-eslint", "react-hooks"],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react/prop-types": "off",
    "@typescript-eslint/no-unused-vars": ["warn"],
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

### .prettierrc.json

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always"
}
```

---

## Key Features Covered

✅ **React Hooks**
- useState, useEffect, useContext, useReducer
- useCallback, useMemo, useRef
- useLayoutEffect, useId

✅ **Custom Hooks**
- useFetch, useForm, useAuth
- useDebounce, useLocalStorage, useAsync
- usePrevious

✅ **Context API**
- Authentication context
- Theme context
- Notification context

✅ **Redux Toolkit**
- Slices and reducers
- Async thunks
- Store configuration

✅ **Component Patterns**
- Functional components
- Compound components
- Presentational vs container
- Error boundaries
- Suspense & lazy loading

✅ **Best Practices**
- TypeScript
- Prop validation
- Error handling
- Performance optimization
- Accessibility
- Code organization
- Styling with CSS modules

✅ **Advanced Features**
- Authentication & authorization
- Protected routes
- Form handling & validation
- API integration
- State management
- Theme switching
- Notifications

This boilerplate is production-ready and follows modern React best practices! 🚀

--- 

Generate a boiler-plate React code, following best practices. Use the most React components and React components in that boiler-plate code so that I can use that for understanding React as well as prepare for technical interview.

---
