# Claude Generated Code

**Task**: Objective: Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code  Claude Tasks:  🧩 Feature Implementation Read the existing codebase and integrate new feature modules where appropriate.  Ensure new code follows existing patterns and conventions.  🗂️ Example Features (replace/add specifics as needed): Add user role-based access control  Implement export to PDF  Add multi-language support  Enable dark mode toggle  🧪 Testing & Documentation Add unit/integration tests for new functionality.  Update README or in-code comments if required.

**Generated on**: Sun Jul 27 14:44:11 UTC 2025

## Generated Content

```
Based on the codebase context, I notice this appears to be an Ionic starter templates repository rather than a specific government billing application. However, I'll design a comprehensive government billing/invoicing feature set using Ionic React TypeScript that can be implemented in the React section of this project structure.

Let me implement a government billing/invoicing module with user role-based access control, PDF export, multi-language support, and dark mode toggle.

## FILES_TO_CREATE

### Core Types and Interfaces
```typescript
// react/official/government-billing/src/types/billing.types.ts
// react/official/government-billing/src/types/user.types.ts
// react/official/government-billing/src/types/common.types.ts
```

### Services
```typescript
// react/official/government-billing/src/services/BillingService.ts
// react/official/government-billing/src/services/AuthService.ts
// react/official/government-billing/src/services/PDFService.ts
// react/official/government-billing/src/services/LocalizationService.ts
```

### Context Providers
```typescript
// react/official/government-billing/src/contexts/AuthContext.tsx
// react/official/government-billing/src/contexts/ThemeContext.tsx
// react/official/government-billing/src/contexts/LocalizationContext.tsx
```

### Components
```typescript
// react/official/government-billing/src/components/BillingDashboard.tsx
// react/official/government-billing/src/components/InvoiceForm.tsx
// react/official/government-billing/src/components/InvoiceList.tsx
// react/official/government-billing/src/components/UserManagement.tsx
// react/official/government-billing/src/components/RoleGuard.tsx
// react/official/government-billing/src/components/ThemeToggle.tsx
// react/official/government-billing/src/components/LanguageSelector.tsx
```

### Hooks
```typescript
// react/official/government-billing/src/hooks/useAuth.ts
// react/official/government-billing/src/hooks/useBilling.ts
// react/official/government-billing/src/hooks/useTheme.ts
// react/official/government-billing/src/hooks/useLocalization.ts
```

### Tests
```typescript
// react/official/government-billing/src/__tests__/BillingService.test.ts
// react/official/government-billing/src/__tests__/AuthService.test.ts
// react/official/government-billing/src/__tests__/components/BillingDashboard.test.tsx
```

## FILES_TO_MODIFY

```typescript
// react/official/government-billing/package.json - Add dependencies
// react/official/government-billing/src/App.tsx - Integrate contexts and routing
// react/official/government-billing/ionic.config.json - Update configuration
```

## CODE_CHANGES

### 1. Types and Interfaces

**react/official/government-billing/src/types/user.types.ts**
```typescript
export enum UserRole {
  ADMIN = 'admin',
  BILLING_MANAGER = 'billing_manager',
  BILLING_CLERK = 'billing_clerk',
  VIEWER = 'viewer'
}

export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  department?: string;
  isActive: boolean;
  createdAt: Date;
  lastLogin?: Date;
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}
```

**react/official/government-billing/src/types/billing.types.ts**
```typescript
export interface Invoice {
  id: string;
  invoiceNumber: string;
  citizenId: string;
  citizenName: string;
  citizenEmail: string;
  items: InvoiceItem[];
  totalAmount: number;
  taxAmount: number;
  status: InvoiceStatus;
  dueDate: Date;
  createdAt: Date;
  createdBy: string;
  paidAt?: Date;
  notes?: string;
}

export interface InvoiceItem {
  id: string;
  description: string;
  category: ServiceCategory;
  quantity: number;
  unitPrice: number;
  totalPrice: number;
}

export enum InvoiceStatus {
  DRAFT = 'draft',
  SENT = 'sent',
  PAID = 'paid',
  OVERDUE = 'overdue',
  CANCELLED = 'cancelled'
}

export enum ServiceCategory {
  PERMITS = 'permits',
  LICENSES = 'licenses',
  FINES = 'fines',
  UTILITIES = 'utilities',
  TAXES = 'taxes',
  OTHER = 'other'
}

export interface BillingFilters {
  status?: InvoiceStatus;
  dateRange?: {
    start: Date;
    end: Date;
  };
  category?: ServiceCategory;
  searchTerm?: string;
}
```

**react/official/government-billing/src/types/common.types.ts**
```typescript
export interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
  errors?: string[];
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export type Theme = 'light' | 'dark';

export type Language = 'en' | 'es' | 'fr';

export interface LocalizedStrings {
  [key: string]: string;
}
```

### 2. Services

**react/official/government-billing/src/services/BillingService.ts**
```typescript
import { Invoice, InvoiceItem, BillingFilters, InvoiceStatus } from '../types/billing.types';
import { ApiResponse, PaginatedResponse } from '../types/common.types';

class BillingService {
  private baseUrl = process.env.REACT_APP_API_BASE_URL || '/api';

  async getInvoices(
    page: number = 1,
    limit: number = 10,
    filters?: BillingFilters
  ): Promise<PaginatedResponse<Invoice>> {
    try {
      const queryParams = new URLSearchParams({
        page: page.toString(),
        limit: limit.toString(),
        ...(filters?.status && { status: filters.status }),
        ...(filters?.category && { category: filters.category }),
        ...(filters?.searchTerm && { search: filters.searchTerm }),
      });

      const response = await fetch(`${this.baseUrl}/invoices?${queryParams}`, {
        headers: this.getAuthHeaders(),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      return await response.json();
    } catch (error) {
      console.error('Error fetching invoices:', error);
      throw error;
    }
  }

  async getInvoiceById(id: string): Promise<Invoice> {
    try {
      const response = await fetch(`${this.baseUrl}/invoices/${id}`, {
        headers: this.getAuthHeaders(),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result: ApiResponse<Invoice> = await response.json();
      return result.data;
    } catch (error) {
      console.error('Error fetching invoice:', error);
      throw error;
    }
  }

  async createInvoice(invoiceData: Partial<Invoice>): Promise<Invoice> {
    try {
      const response = await fetch(`${this.baseUrl}/invoices`, {
        method: 'POST',
        headers: {
          ...this.getAuthHeaders(),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(invoiceData),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result: ApiResponse<Invoice> = await response.json();
      return result.data;
    } catch (error) {
      console.error('Error creating invoice:', error);
      throw error;
    }
  }

  async updateInvoice(id: string, updates: Partial<Invoice>): Promise<Invoice> {
    try {
      const response = await fetch(`${this.baseUrl}/invoices/${id}`, {
        method: 'PUT',
        headers: {
          ...this.getAuthHeaders(),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updates),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result: ApiResponse<Invoice> = await response.json();
      return result.data;
    } catch (error) {
      console.error('Error updating invoice:', error);
      throw error;
    }
  }

  async updateInvoiceStatus(id: string, status: InvoiceStatus): Promise<Invoice> {
    return this.updateInvoice(id, { status });
  }

  async deleteInvoice(id: string): Promise<void> {
    try {
      const response = await fetch(`${this.baseUrl}/invoices/${id}`, {
        method: 'DELETE',
        headers: this.getAuthHeaders(),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
    } catch (error) {
      console.error('Error deleting invoice:', error);
      throw error;
    }
  }

  private getAuthHeaders(): HeadersInit {
    const token = localStorage.getItem('authToken');
    return {
      'Authorization': token ? `Bearer ${token}` : '',
    };
  }

  calculateTotalAmount(items: InvoiceItem[]): number {
    return items.reduce((total, item) => total + item.totalPrice, 0);
  }

  calculateTaxAmount(subtotal: number, taxRate: number = 0.08): number {
    return subtotal * taxRate;
  }
}

export default new BillingService();
```

**react/official/government-billing/src/services/PDFService.ts**
```typescript
import jsPDF from 'jspdf';
import 'jspdf-autotable';
import { Invoice } from '../types/billing.types';

class PDFService {
  async generateInvoicePDF(invoice: Invoice): Promise<Blob> {
    const pdf = new jsPDF();
    
    // Header
    pdf.setFontSize(20);
    pdf.text('Government Invoice', 20, 30);
    
    // Invoice details
    pdf.setFontSize(12);
    pdf.text(`Invoice Number: ${invoice.invoiceNumber}`, 20, 50);
    pdf.text(`Date: ${new Date(invoice.createdAt).toLocaleDateString()}`, 20, 60);
    pdf.text(`Due Date: ${new Date(invoice.dueDate).toLocaleDateString()}`, 20, 70);
    pdf.text(`Status: ${invoice.status.toUpperCase()}`, 20, 80);

    // Citizen information
    pdf.text('Bill To:', 20, 100);
    pdf.text(`${invoice.citizenName}`, 20, 110);
    pdf.text(`ID: ${invoice.citizenId}`, 20, 120);
    pdf.text(`Email: ${invoice.citizenEmail}`, 20, 130);

    // Items table
    const tableData = invoice.items.map(item => [
      item.description,
      item.category,
      item.quantity.toString(),
      `$${item.unitPrice.toFixed(2)}`,
      `$${item.totalPrice.toFixed(2)}`
    ]);

    (pdf as any).autoTable({
      startY: 150,
      head: [['Description', 'Category', 'Qty', 'Unit Price', 'Total']],
      body: tableData,
      theme: 'striped',
      headStyles: { fillColor: [41, 128, 185] },
    });

    // Totals
    const finalY = (pdf as any).lastAutoTable.finalY + 20;
    pdf.text(`Subtotal: $${(invoice.totalAmount - invoice.taxAmount).toFixed(2)}`, 140, finalY);
    pdf.text(`Tax: $${invoice.taxAmount.toFixed(2)}`, 140, finalY + 10);
    pdf.setFontSize(14);
    pdf.text(`Total: $${invoice.totalAmount.toFixed(2)}`, 140, finalY + 25);

    return pdf.output('blob');
  }

  async downloadInvoicePDF(invoice: Invoice): Promise<void> {
    const pdfBlob = await this.generateInvoicePDF(invoice);
    const url = URL.createObjectURL(pdfBlob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `invoice-${invoice.invoiceNumber}.pdf`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }

  async emailInvoicePDF(invoice: Invoice): Promise<void> {
    try {
      const pdfBlob = await this.generateInvoicePDF(invoice);
      const formData = new FormData();
      formData.append('pdf', pdfBlob, `invoice-${invoice.invoiceNumber}.pdf`);
      formData.append('email', invoice.citizenEmail);
      formData.append('invoiceId', invoice.id);

      const response = await fetch('/api/invoices/email', {
        method: 'POST',
        body: formData,
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('authToken')}`,
        },
      });

      if (!response.ok) {
        throw new Error('Failed to email invoice');
      }
    } catch (error) {
      console.error('Error emailing invoice:', error);
      throw error;
    }
  }
}

export default new PDFService();
```

### 3. Context Providers

**react/official/government-billing/src/contexts/AuthContext.tsx**
```typescript
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { User, AuthState, UserRole } from '../types/user.types';

interface AuthContextType extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  hasPermission: (requiredRole: UserRole) => boolean;
  hasAnyPermission: (roles: UserRole[]) => boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'CLEAR_ERROR' };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        isLoading: false,
        isAuthenticated: true,
        user: action.payload,
        error: null,
      };
    case 'LOGIN_FAILURE':
      return {
        ...state,
        isLoading: false,
        isAuthenticated: false,
        user: null,
        error: action.payload,
      };
    case 'LOGOUT':
      return {
        ...state,
        isAuthenticated: false,
        user: null,
        error: null,
      };
    case 'CLEAR_ERROR':
      return { ...state, error: null };
    default:
      return state;
  }
};

const roleHierarchy: Record<UserRole, UserRole[]> = {
  [UserRole.ADMIN]: [UserRole.ADMIN, UserRole.BILLING_MANAGER, UserRole.BILLING_CLERK, UserRole.VIEWER],
  [UserRole.BILLING_MANAGER]: [UserRole.BILLING_MANAGER, UserRole.BILLING_CLERK, UserRole.VIEWER],
  [UserRole.BILLING_CLERK]: [UserRole.BILLING_CLERK, UserRole.VIEWER],
  [UserRole.VIEWER]: [UserRole.VIEWER],
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    isAuthenticated: false,
    isLoading: false,
    error: null,
  });

  useEffect(() => {
    // Check for existing auth token on app load
    const token = localStorage.getItem('authToken');
    const userData = localStorage.getItem('userData');
    
    if (token && userData) {
      try {
        const user = JSON.parse(userData);
        dispatch({ type: 'LOGIN_SUCCESS', payload: user });
      } catch (error) {
        localStorage.removeItem('authToken');
        localStorage.removeItem('userData');
      }
    }
  }, []);

  const login = async (email: string, password: string): Promise<void> => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const { token, user } = await response.json();
      
      localStorage.setItem('authToken', token);
      localStorage.setItem('userData', JSON.stringify(user));
      
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: 'Invalid credentials' });
      throw error;
    }
  };

  const logout = (): void => {
    localStorage.removeItem('authToken');
    localStorage.removeItem('userData');
    dispatch({ type: 'LOGOUT' });
  };

  const hasPermission = (requiredRole: UserRole): boolean => {
    if (!state.user) return false;
    return roleHierarchy[state.user.role].includes(requiredRole);
  };

  const hasAnyPermission = (roles: UserRole[]): boolean => {
    return roles.some(role => hasPermission(role));
  };

  const value: AuthContextType = {
    ...state,
    login,
    logout,
    hasPermission,
    hasAnyPermission,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

**react/official/government-billing/src/contexts/ThemeContext.tsx**
```typescript
import React, { createContext, useContext, useState, useEffect } from 'react';
import { Theme } from '../types/common.types';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setThemeState] = useState<Theme>(() => {
    const savedTheme = localStorage.getItem('app-theme') as Theme;
    return savedTheme || 'light';
  });

  useEffect(() => {
    document.body.classList.toggle('dark', theme === 'dark');
    localStorage.setItem('app-theme', theme);
  }, [theme]);

  const toggleTheme = (): void => {
    setThemeState(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };

  const setTheme = (newTheme: Theme): void => {
    setThemeState(newTheme);
  };

  const value: ThemeContextType = {
    theme,
    toggleTheme,
    setTheme,
  };

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 4. Main Components

**react/official/government-billing/src/components/BillingDashboard.tsx**
```typescript
import React, { useState, useEffect } from 'react';
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonGrid,
  IonRow,
  IonCol,
  IonButton,
  IonIcon,
  IonItem,
  IonLabel,
  IonList,
  IonBadge,
  IonRefresher,
  IonRefresherContent,
  IonSearchbar,
  IonSelect,
  IonSelectOption,
  IonSpinner,
  useIonToast,
} from '@ionic/react';
import { add, download, eye, create } from 'ionicons/icons';
import { Invoice, InvoiceStatus, BillingFilters } from '../types/billing.types';
import { UserRole } from '../types/user.types';
import { useAuth } from '../contexts/AuthContext';
import { useLocalization } from '../contexts/LocalizationContext';
import BillingService from '../services/BillingService';
import PDFService from '../services/PDFService';
import RoleGuard from './RoleGuard';
import InvoiceList from './InvoiceList';

const BillingDashboard: React.FC = () => {
  const { user, hasPermission } = useAuth();
  const { t } = useLocalization();
  const [present] = useIonToast();

  const [invoices, setInvoices] = useState<Invoice[]>([]);
  const [loading, setLoading] = useState(true);
  const [filters, setFilters] = useState<BillingFilters>({});
  const [stats, setStats] = useState({
    total: 0,
    paid: 0,
    pending: 0,
    overdue: 0,
  });

  useEffect(() => {
    loadData();
  }, [filters]);

  const loadData = async () => {
    try {
      setLoading(true);
      const response = await BillingService.getInvoices(1, 50, filters);
      setInvoices(response.data);
      
      // Calculate stats
      const total = response.data.length;
      const paid = response.data.filter(inv => inv.status === InvoiceStatus.PAID).length;
      const pending = response.data.filter(inv => inv.status === InvoiceStatus.SENT).length;
      const overdue = response.data.filter(inv => inv.status === InvoiceStatus.OVERDUE).length;
      
      setStats({ total, paid, pending, overdue });
    } catch (error) {
      present({
        message: t('error.loadingData'),
        duration: 3000,
        color: 'danger',
      });
    } finally {
      setLoading(false);
    }
  };

  const handleRefresh = async (event: CustomEvent) => {
    await loadData();
    event.detail.complete();
  };

  const handleExportPDF = async (invoice: Invoice) => {
    try {
      await PDFService.downloadInvoicePDF(invoice);
      present({
        message: t('success.pdfExported'),
        duration: 2000,
        color: 'success',
      });
    } catch (error) {
      present({
        message: t('error.exportFailed'),
        duration: 3000,
        color: 'danger',
      });
    }
  };

  const handleSearchChange = (event: CustomEvent) => {
    setFilters(prev => ({
      ...prev,
      searchTerm: event.detail.value,
    }));
  };

  const handleStatusFilter = (status?: InvoiceStatus) => {
    setFilters(prev => ({
      ...prev,
      status,
    }));
  };

  if (loading) {
    return (
      <IonPage>
        <IonContent className="ion-padding">
          <div className="ion-text-center">
            <IonSpinner name="crescent" />
          </div>
        </IonContent>
      </IonPage>
    );
  }

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>{t('dashboard.title')}</IonTitle>
          <RoleGuard allowedRoles={[UserRole.ADMIN, UserRole.BILLING_MANAGER, UserRole.BILLING_CLERK]}>
            <IonButton slot="end" routerLink="/invoices/new">
              <IonIcon icon={add} />
            </IonButton>
          </RoleGuard>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <IonRefresher slot="fixed" onIonRefresh={handleRefresh}>
          <IonRefresherContent />
        </IonRefresher>

        {/* Statistics Cards */}
        <IonGrid>
          <IonRow>
            <IonCol size="6" size-md="3">
              <IonCard>
                <IonCardContent>
                  <h2>{stats.total}</h2>
                  <p>{t('dashboard.totalInvoices')}</p>
                </IonCardContent>
              </IonCard>
            </IonCol>
            <IonCol size="6" size-md="3">
              <IonCard>
                <IonCardContent>
                  <h2>{stats.paid}</h2>
                  <p>{t('dashboard.paidInvoices')}</p>
                </IonCardContent>
              </IonCard>
            </IonCol>
            <IonCol size="6" size-md="3">
              <IonCard>
                <IonCardContent>
                  <h2>{stats.pending}</h2>
                  <p>{t('dashboard.pendingInvoices')}</p>
                </IonCardContent>
              </IonCard>
            </IonCol>
            <IonCol size="6" size-md="3">
              <IonCard>
                <IonCardContent>
                  <h2>{stats.overdue}</h2>
                  <p>{t('dashboard.overdueInvoices')}</p>
                </IonCardContent>
              </IonCard>
            </IonCol>
          </IonRow>
        </IonGrid>

        {/* Filters */}
        <IonCard>
          <IonCardHeader>
            <IonCardTitle>{t('dashboard.filters')}</IonCardTitle>
          </IonCardHeader>
          <IonCardContent>
            <IonSearchbar
              placeholder={t('dashboard.searchPlaceholder')}
              onIonInput={handleSearchChange}
              debounce={300}
            />
            <IonItem>
              <IonLabel>{t('dashboard.filterByStatus')}</IonLabel>
              <IonSelect
                placeholder={t('dashboard.allStatuses')}
                onSelectionChange={(e) => handleStatusFilter(e.detail.value)}
              >
                <IonSelectOption value={undefined}>{t('dashboard.allStatuses')}</IonSelectOption>
                <IonSelectOption value={InvoiceStatus.DRAFT}>{t('status.draft')}</IonSelectOption>
                <IonSelectOption value={InvoiceStatus.SENT}>{t('status.sent')}</IonSelectOption>
                <IonSelectOption value={InvoiceStatus.PAID}>{t('status.paid')}</IonSelectOption>
                <IonSelectOption value={InvoiceStatus.OVERDUE}>{t('status.overdue')}</IonSelectOption>
              </IonSelect>
            </IonItem>
          </IonCardContent>
        </IonCard>

        {/* Invoice List */}
        <InvoiceList
          invoices={invoices}
          onExportPDF={handleExportPDF}
          onRefresh={loadData}
        />
      </IonContent>
    </IonPage>
  );
};

export default BillingDashboard;
```

**react/official/government-billing/src/components/RoleGuard.tsx**
```typescript
import React from 'react';
import { UserRole } from '../types/user.types';
import { useAuth } from '../contexts/AuthContext';

interface RoleGuardProps {
  allowedRoles: UserRole[];
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

const RoleGuard: React.FC<RoleGuardProps> = ({ 
  allowedRoles, 
  children, 
  fallback = null 
}) => {
  const { hasAnyPermission } = useAuth();

  if (!hasAnyPermission(allowedRoles)) {
    return <>{fallback}</>;
  }

  return <>{children}</>;
};

export default RoleGuard;
```

**react/official/government-billing/src/components/ThemeToggle.tsx**
```typescript
import React from 'react';
import { IonToggle, IonItem, IonLabel, IonIcon } from '@ionic/react';
import { moon, sunny } from 'ionicons/icons';
import { useTheme } from '../contexts/ThemeContext';
import { useLocalization } from '../contexts/LocalizationContext';

const ThemeToggle: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  const { t } = useLocalization();

  return (
    <IonItem>
      <IonIcon 
        icon={theme === 'dark' ? moon : sunny} 
        slot="start" 
      />
      <IonLabel>{t('settings.darkMode')}</IonLabel>
      <IonToggle
        checked={theme === 'dark'}
        onIonChange={toggleTheme}
        slot="end"
      />
    </IonItem>
  );
};

export default ThemeToggle;
```

### 5. Package.json Updates

**react/official/government-billing/package.json**
```json
{
  "name": "government-billing-app",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "@capacitor/app": "5.0.6",
    "@capacitor/core": "5.5.1",
    "@capacitor/haptics": "5.0.6",
    "@capacitor/keyboard": "5.0
```
