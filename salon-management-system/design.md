# Salon Yönetim Sistemi - Tasarım Dokümanı

## Genel Bakış

Salon Yönetim Sistemi, modern bayan kuaförü salonları için geliştirilecek basit
ve etkili bir randevu yönetim platformudur. Sistem, minimal ama güçlü
teknolojiler kullanarak monolitik mimari ile tasarlanmıştır. NextJS 15 frontend,
NestJS backend ve PostgreSQL veritabanı ile geliştirilecektir.

## Architecture

### High-Level Architecture

```mermaid
graph TB
    subgraph "Frontend Layer"
        WEB[NextJS Web App]
        ADMIN[Admin Dashboard]
    end

    subgraph "API Gateway"
        GATEWAY[NestJS API Gateway]
    end

    subgraph "Microservices"
        AUTH[Authentication Service]
        APPOINTMENT[Appointment Service]
        USER[User Service]
        NOTIFICATION[Notification Service]
        PAYMENT[Payment Service]
        CONTENT[Content Management Service]
    end

    subgraph "Data Layer"
        POSTGRES[(PostgreSQL)]
        REDIS[(Redis Cache)]
        FILES[File Storage]
    end

    subgraph "External Services"
        SMS[SMS Provider]
        EMAIL[Email Service]
        OAUTH[Google OAuth 2.0]
        CDN[CDN/File Storage]
    end

    subgraph "Monitoring & Observability"
        OTEL[OpenTelemetry]
        METRICS[Metrics Collection]
        LOGS[Centralized Logging]
    end

    WEB --> GATEWAY
    ADMIN --> GATEWAY
    GATEWAY --> AUTH
    GATEWAY --> APPOINTMENT
    GATEWAY --> USER
    GATEWAY --> NOTIFICATION
    GATEWAY --> PAYMENT
    GATEWAY --> CONTENT

    AUTH --> POSTGRES
    AUTH --> REDIS
    AUTH --> OAUTH
    AUTH --> OTEL

    APPOINTMENT --> POSTGRES
    APPOINTMENT --> REDIS
    APPOINTMENT --> OTEL

    USER --> POSTGRES
    USER --> OTEL

    NOTIFICATION --> SMS
    NOTIFICATION --> EMAIL
    NOTIFICATION --> OTEL

    PAYMENT --> POSTGRES
    PAYMENT --> OTEL

    CONTENT --> POSTGRES
    CONTENT --> CDN
    CONTENT --> OTEL

    OTEL --> METRICS
    OTEL --> LOGS
```

### Teknoloji Stack'i (2025 Güncel)

- **Frontend**: NextJS 15.0+ (React 19, Server Components, Turbopack stable)
- **UI Framework**: ShadCN UI (Radix UI tabanlı, TailwindCSS v4 uyumlu)
- **Styling**: TailwindCSS v4.0+ (Native container queries, performance
  optimizations)
- **Backend**: NestJS 10.4+ + TypeScript 5.7+ (ES2024 target)
- **ORM**: Prisma 5.22+ (OpenTelemetry tracing, type-safe client)
- **Database**: PostgreSQL 16+ (primary), Redis 7+ (cache/session)
- **Authentication**: JWT + argon2 + Google OAuth 2.0
- **Communication**: REST API + GraphQL + Socket.IO v4+
- **Monitoring**: OpenTelemetry (distributed tracing, metrics)
- **File Storage**: Secure cloud storage with CDN
- **Containerization**: Docker + Docker Compose
- **Monorepo**: Turborepo (build optimization, caching)
- **Date Management**: date-fns v3+ (modern date utilities)

## Components and Interfaces

### 1. Authentication Service (Kimlik Doğrulama Servisi)

**Sorumluluklar:**

- Kullanıcı kimlik doğrulama (email/şifre, Google OAuth 2.0)
- JWT token yönetimi (access/refresh token)
- Şifre sıfırlama ve güvenlik
- Role-based access control (RBAC)
- Session yönetimi (Redis)
- OpenTelemetry tracing

**Ana Sınıflar:**

```typescript
/**
 * Kimlik doğrulama işlemlerini yöneten ana servis
 */
@Injectable()
export class AuthService {
  constructor(
    private readonly tokenService: TokenService,
    private readonly userService: UserService,
    private readonly redisService: RedisService,
    private readonly telemetryService: TelemetryService
  ) {}

  /**
   * Kullanıcı giriş işlemi - email/şifre ile
   */
  @Trace('auth.login')
  async login(credentials: LoginDto): Promise<AuthResponse> {
    const span = this.telemetryService.startSpan('user.login');
    try {
      // Implementation with tracing
      return await this.performLogin(credentials);
    } finally {
      span.end();
    }
  }

  /**
   * Google OAuth 2.0 ile giriş
   */
  @Trace('auth.google-login')
  async googleLogin(token: string): Promise<AuthResponse>;

  /**
   * Refresh token ile yeni access token alma
   */
  @Trace('auth.refresh-token')
  async refreshToken(refreshToken: string): Promise<TokenResponse>;

  /**
   * Güvenli şifre sıfırlama
   */
  @Trace('auth.reset-password')
  async resetPassword(email: string): Promise<void>;

  /**
   * Kullanıcı çıkış işlemi (token invalidation)
   */
  @Trace('auth.logout')
  async logout(userId: string, tokenId: string): Promise<void>;
}

/**
 * JWT token yönetimi ve güvenlik
 */
@Injectable()
export class TokenService {
  /**
   * Access token oluşturma (kısa süreli)
   */
  generateAccessToken(payload: JwtPayload): string;

  /**
   * Refresh token oluşturma (uzun süreli)
   */
  generateRefreshToken(userId: string): string;

  /**
   * Token doğrulama ve payload çıkarma
   */
  verifyToken(token: string): JwtPayload;

  /**
   * Token blacklist kontrolü (Redis)
   */
  async isTokenBlacklisted(tokenId: string): Promise<boolean>;
}
```

### 2. User Service (Kullanıcı Yönetim Servisi)

**Sorumluluklar:**

- Müşteri profil yönetimi (GDPR uyumlu)
- Çalışan yönetimi ve yetkilendirme
- Kullanıcı arama ve filtreleme (performanslı)
- Avatar ve profil resmi yönetimi
- Kullanıcı aktivite takibi
- KVKK uyumlu veri işleme

**Ana Sınıflar:**

```typescript
/**
 * Kullanıcı yönetim ana servisi
 */
@Injectable()
export class UserService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly fileService: FileService,
    private readonly telemetryService: TelemetryService,
    private readonly cacheService: CacheService
  ) {}

  /**
   * Müşteri profil güncelleme (GDPR uyumlu)
   */
  @Trace('user.update-customer-profile')
  async updateCustomerProfile(
    id: string,
    data: UpdateCustomerDto,
    userId: string
  ): Promise<Customer> {
    // GDPR consent kontrolü
    await this.validateGDPRConsent(id, userId);
    return await this.performProfileUpdate(id, data);
  }

  /**
   * Çalışan oluşturma ve yetki atama
   */
  @Trace('user.create-employee')
  async createEmployee(
    data: CreateEmployeeDto,
    createdBy: string
  ): Promise<Employee> {
    // Role-based authorization kontrolü
    await this.validateEmployeeCreationPermission(createdBy);
    return await this.performEmployeeCreation(data);
  }

  /**
   * Gelişmiş kullanıcı arama (cache'li)
   */
  @Trace('user.search-users')
  async searchUsers(
    query: SearchUserDto,
    requesterId: string
  ): Promise<PaginatedUsers> {
    const cacheKey = `user_search:${JSON.stringify(query)}`;
    const cached = await this.cacheService.get(cacheKey);
    if (cached) return cached;

    const result = await this.performUserSearch(query, requesterId);
    await this.cacheService.set(cacheKey, result, 300); // 5 dakika cache
    return result;
  }

  /**
   * Kullanıcı avatar yükleme
   */
  @Trace('user.upload-avatar')
  async uploadAvatar(
    userId: string,
    file: Express.Multer.File
  ): Promise<AvatarResponse>;

  /**
   * GDPR uyumlu kullanıcı verisi silme
   */
  @Trace('user.gdpr-delete')
  async deleteUserData(userId: string, requesterId: string): Promise<void>;
}

/**
 * Kullanıcı cache yönetimi
 */
@Injectable()
export class UserCacheService {
  /**
   * Kullanıcı bilgilerini cache'e alma
   */
  async cacheUserProfile(userId: string, profile: UserProfile): Promise<void>;

  /**
   * Cache'den kullanıcı bilgisi alma
   */
  async getCachedUserProfile(userId: string): Promise<UserProfile | null>;

  /**
   * Kullanıcı cache'ini temizleme
   */
  async invalidateUserCache(userId: string): Promise<void>;
}
```

### 3. Appointment Service (Randevu Yönetim Servisi)

**Sorumluluklar:**

- Randevu oluşturma ve yönetimi (real-time)
- Randevu durumu takibi ve workflow
- Gelişmiş çakışma kontrolü (multi-employee)
- Takip numarası yönetimi (6 haneli)
- Otomatik randevu hatırlatmaları
- Randevu analitikleri ve raporlama
- Socket.IO ile real-time güncellemeler

**Ana Sınıflar:**

```typescript
/**
 * Randevu yönetim ana servisi
 */
@Injectable()
export class AppointmentService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly conflictService: ConflictDetectionService,
    private readonly trackingService: TrackingNumberService,
    private readonly notificationService: NotificationService,
    private readonly socketGateway: AppointmentSocketGateway,
    private readonly telemetryService: TelemetryService
  ) {}

  /**
   * Randevu oluşturma (çakışma kontrolü ile)
   */
  @Trace('appointment.create')
  async createAppointment(
    data: CreateAppointmentDto,
    createdBy?: string
  ): Promise<AppointmentResponse> {
    const span = this.telemetryService.startSpan('appointment.create');

    try {
      // 1. Çakışma kontrolü
      await this.conflictService.validateAppointmentSlot(
        data.dateTime,
        data.duration,
        data.employeeId
      );

      // 2. Takip numarası oluştur
      const trackingNumber = await this.trackingService.generateUnique();

      // 3. Randevu oluştur
      const appointment = await this.prisma.appointment.create({
        data: {
          ...data,
          trackingNumber,
          status: AppointmentStatus.PENDING,
        },
        include: this.getAppointmentIncludes(),
      });

      // 4. Real-time bildirim gönder
      await this.socketGateway.notifyAppointmentCreated(appointment);

      // 5. Otomatik bildirim planla
      await this.notificationService.scheduleAppointmentReminders(appointment);

      span.setAttributes({
        'appointment.id': appointment.id,
        'appointment.tracking_number': trackingNumber,
      });

      return this.mapToAppointmentResponse(appointment);
    } finally {
      span.end();
    }
  }

  /**
   * Randevu durumu güncelleme (workflow ile)
   */
  @Trace('appointment.update-status')
  async updateAppointmentStatus(
    id: string,
    status: AppointmentStatus,
    updatedBy: string,
    notes?: string
  ): Promise<AppointmentResponse> {
    // Status transition validation
    await this.validateStatusTransition(id, status);

    const appointment = await this.prisma.appointment.update({
      where: { id },
      data: {
        status,
        notes,
        updatedBy,
        updatedAt: new Date()
      },
      include: this.getAppointmentIncludes(),
    });

    // Real-time güncelleme
    await this.socketGateway.notifyAppointmentUpdated(appointment);

    // Status'a göre otomatik işlemler
    await this.handleStatusChangeActions(appointment, status);

    return this.mapToAppointmentResponse(appointment);
  }

  /**
   * Gelişmiş çakışma kontrolü
   */
  @Trace('appointment.check-conflicts')
  async checkConflicts(
    dateTime: Date,
    duration: number,
    employeeId?: string,
    excludeAppointmentId?: string
  ): Promise<ConflictCheckResult> {
    return await this.conflictService.checkDetailedConflicts({
      dateTime,
      duration,
      employeeId,
      excludeAppointmentId,
    });
  }

  /**
   * Takip numarası ile randevu bulma (cache'li)
   */
  @Trace('appointment.find-by-tracking')
  async findByTrackingNumber(
    trackingNumber: string
  ): Promise<AppointmentResponse | null> {
    const cacheKey = `appointment:tracking:${trackingNumber}`;
    const cached = await this.cacheService.get(cacheKey);
    if (cached) return cached;

    const appointment = await this.prisma.appointment.findUnique({
      where: { trackingNumber },
      include: this.getAppointmentIncludes(),
    });

    if (appointment) {
      const response = this.mapToAppointmentResponse(appointment);
      await this.cacheService.set(cacheKey, response, 600); // 10 dakika
      return response;
    }

    return null;
  }

  /**
   * Randevu listesi (sayfalama ve filtreleme ile)
   */
  @Trace('appointment.list')
  async getAppointments(
    query: AppointmentQueryDto,
    requesterId: string
  ): Promise<PaginatedAppointments>;

  /**
   * Randevu analitikleri
   */
  @Trace('appointment.analytics')
  async getAppointmentAnalytics(
    period: DateRange,
    requesterId: string
  ): Promise<AppointmentAnalytics>;
}

/**
 * Çakışma tespit servisi
 */
@Injectable()
export class ConflictDetectionService {
  /**
   * Detaylı çakışma kontrolü
   */
  async checkDetailedConflicts(
    params: ConflictCheckParams
  ): Promise<ConflictCheckResult> {
    // Multi-employee, service duration, break times kontrolü
  }

  /**
   * Randevu slotu doğrulama
   */
  async validateAppointmentSlot(
    dateTime: Date,
    duration: number,
    employeeId?: string
  ): Promise<void> {
    // Çalışma saatleri, tatil günleri, mevcut randevular kontrolü
  }
}

/**
 * Takip numarası yönetimi
 */
@Injectable()
export class TrackingNumberService {
  /**
   * Benzersiz 6 haneli takip numarası oluşturma
   */
  async generateUnique(): Promise<string> {
    let attempts = 0;
    const maxAttempts = 10;

    while (attempts < maxAttempts) {
      const trackingNumber = this.generateSixDigitNumber();
      const exists = await this.checkIfExists(trackingNumber);

      if (!exists) {
        return trackingNumber;
      }

      attempts++;
    }

    throw new Error('Takip numarası oluşturulamadı');
  }

  /**
   * 6 haneli sayı oluşturma
   */
  private generateSixDigitNumber(): string {
    return Math.floor(100000 + Math.random() * 900000).toString();
  }

  /**
   * Takip numarası varlık kontrolü
   */
  private async checkIfExists(trackingNumber: string): Promise<boolean> {
    const count = await this.prisma.appointment.count({
      where: { trackingNumber },
    });
    return count > 0;
  }
}
```

### 4. Notification Service

**Sorumluluklar:**

- SMS bildirimleri
- Email bildirimleri
- Socket.IO real-time bildirimleri
- Bildirim şablonları

**Ana Sınıflar:**

```typescript
/**
 * Bildirim yönetim servisi
 */
class NotificationService {
  /**
   * SMS gönderme
   */
  async sendSMS(phone: string, message: string): Promise<void>;

  /**
   * Email gönderme
   */
  async sendEmail(
    email: string,
    subject: string,
    content: string
  ): Promise<void>;

  /**
   * Socket.IO bildirimi
   */
  async sendSocketNotification(
    userId: string,
    data: NotificationData
  ): Promise<void>;
}

/**
 * Bildirim şablonları
 */
class NotificationTemplates {
  /**
   * Randevu onay şablonu
   */
  getAppointmentConfirmationTemplate(
    appointment: Appointment
  ): NotificationTemplate;

  /**
   * Randevu iptal şablonu
   */
  getAppointmentCancellationTemplate(
    appointment: Appointment
  ): NotificationTemplate;
}
```

### 5. Payment Service

**Sorumluluklar:**

- Ödeme işlemleri
- İndirim hesaplamaları
- Finansal raporlama

**Ana Sınıflar:**

```typescript
/**
 * Ödeme yönetim servisi
 */
class PaymentService {
  /**
   * Ödeme işlemi
   */
  async processPayment(data: PaymentDto): Promise<Payment>;

  /**
   * İndirim hesaplama
   */
  async calculateDiscount(
    services: Service[],
    discounts: Discount[]
  ): Promise<number>;

  /**
   * Finansal rapor oluşturma
   */
  async generateFinancialReport(period: DateRange): Promise<FinancialReport>;
}
```

### 6. Content Management Service

**Sorumluluklar:**

- Salon ayarları yönetimi
- Hizmet yönetimi
- Galeri yönetimi
- Dosya yükleme

**Ana Sınıflar:**

```typescript
/**
 * İçerik yönetim servisi
 */
class ContentService {
  /**
   * Salon ayarları güncelleme
   */
  async updateSalonSettings(data: SalonSettingsDto): Promise<SalonSettings>;

  /**
   * Hizmet yönetimi
   */
  async manageServices(data: ServiceDto): Promise<Service>;

  /**
   * Dosya yükleme
   */
  async uploadFile(file: Express.Multer.File): Promise<FileUploadResponse>;
}
```

## Data Models

### Core Entities

```typescript
/**
 * Kullanıcı temel modeli
 */
interface User {
  id: string;
  email?: string;
  phone: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  createdAt: Date;
  updatedAt: Date;
}

/**
 * Müşteri modeli
 */
interface Customer extends User {
  appointments: Appointment[];
  feedbacks: Feedback[];
}

/**
 * Çalışan modeli
 */
interface Employee extends User {
  permissions: Permission[];
  isActive: boolean;
}

/**
 * Randevu modeli
 */
interface Appointment {
  id: string;
  trackingNumber: string;
  customerId?: string;
  customerInfo: CustomerInfo;
  services: Service[];
  dateTime: Date;
  duration: number;
  status: AppointmentStatus;
  notes?: string;
  totalAmount: number;
  createdAt: Date;
  updatedAt: Date;
}

/**
 * Hizmet modeli
 */
interface Service {
  id: string;
  name: string;
  description: string;
  duration: number;
  price: number;
  isActive: boolean;
}

/**
 * Salon ayarları modeli
 */
interface SalonSettings {
  id: string;
  name: string;
  address: string;
  phone: string;
  email: string;
  socialMedia: SocialMediaLinks;
  workingHours: WorkingHours[];
  specialDays: SpecialDay[];
  discounts: Discount[];
}

/**
 * Ödeme modeli
 */
interface Payment {
  id: string;
  appointmentId: string;
  amount: number;
  discountAmount: number;
  finalAmount: number;
  paymentDate: Date;
  services: Service[];
}

/**
 * Geri bildirim modeli
 */
interface Feedback {
  id: string;
  customerId: string;
  appointmentId: string;
  rating: number;
  comment: string;
  createdAt: Date;
}
```

### Enums and Types

```typescript
/**
 * Kullanıcı rolleri
 */
enum UserRole {
  CUSTOMER = 'customer',
  EMPLOYEE = 'employee',
  ADMIN = 'admin',
}

/**
 * Randevu durumları
 */
enum AppointmentStatus {
  PENDING = 'pending',
  CONFIRMED = 'confirmed',
  CANCELLED = 'cancelled',
  COMPLETED = 'completed',
}

/**
 * Müşteri bilgileri (kayıtsız müşteriler için)
 */
interface CustomerInfo {
  firstName: string;
  lastName: string;
  phone: string;
  email?: string;
}

/**
 * Çalışma saatleri
 */
interface WorkingHours {
  dayOfWeek: number;
  startTime: string;
  endTime: string;
  isOpen: boolean;
}

/**
 * Özel günler
 */
interface SpecialDay {
  date: Date;
  name: string;
  workingHours?: WorkingHours;
  isClosed: boolean;
}
```

## Error Handling

### Error Response Structure

```typescript
/**
 * Standart hata yanıt yapısı
 */
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
  };
  timestamp: Date;
}

/**
 * Hata kodları
 */
enum ErrorCodes {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  AUTHENTICATION_ERROR = 'AUTHENTICATION_ERROR',
  AUTHORIZATION_ERROR = 'AUTHORIZATION_ERROR',
  NOT_FOUND = 'NOT_FOUND',
  CONFLICT = 'CONFLICT',
  INTERNAL_ERROR = 'INTERNAL_ERROR',
}
```

### Error Messages (Turkish)

```typescript
/**
 * Türkçe hata mesajları
 */
const ErrorMessages = {
  VALIDATION_ERROR: 'Girilen bilgiler geçersiz',
  AUTHENTICATION_ERROR: 'Giriş bilgileri hatalı',
  AUTHORIZATION_ERROR: 'Bu işlem için yetkiniz bulunmuyor',
  NOT_FOUND: 'Aradığınız kayıt bulunamadı',
  APPOINTMENT_CONFLICT:
    'Seçilen tarih ve saatte başka bir randevu bulunmaktadır',
  INVALID_TRACKING_NUMBER: 'Geçersiz takip numarası',
  APPOINTMENT_ALREADY_CONFIRMED: 'Randevu zaten onaylanmış',
  OUTSIDE_WORKING_HOURS: 'Seçilen saat çalışma saatleri dışında',
  NETWORK_ERROR: 'Bağlantı hatası. Lütfen tekrar deneyin',
  SERVER_ERROR: 'Sistem hatası. Lütfen daha sonra tekrar deneyin',
};
```

## Testing Strategy

### Unit Testing

- Her servis için birim testleri
- Jest framework kullanımı
- Mock veriler ile test senaryoları
- %90+ kod kapsamı hedefi

### Integration Testing

- API endpoint testleri
- Veritabanı entegrasyon testleri
- Mikroservisler arası iletişim testleri

### E2E Testing

- Cypress ile frontend testleri
- Kritik kullanıcı senaryoları
- Randevu oluşturma ve yönetimi akışları

### Performance Testing

- Load testing ile performans ölçümü
- Redis cache etkinliği testleri
- Database query optimizasyonu

## Security Considerations

### Authentication & Authorization

- JWT token tabanlı kimlik doğrulama
- argon2 ile şifre hashleme
- Google OAuth entegrasyonu
- Role-based access control (RBAC)

### Data Protection

- GDPR uyumlu veri işleme
- Kişisel verilerin şifrelenmesi
- Güvenli dosya yükleme
- SQL injection koruması

### API Security

- Rate limiting
- CORS yapılandırması
- Input validation
- Error message sanitization

## Performance Optimization (2025 Güncel)

### Caching Strategy (Gelişmiş)

- **Redis 7+ Multi-tier Caching**:
  - L1: Application memory cache (Node.js)
  - L2: Redis cluster cache (distributed)
  - L3: CDN edge caching
- **Smart Cache Invalidation**: Event-driven cache invalidation
- **Cache Warming**: Proactive cache population
- **Cache Analytics**: Hit/miss ratio monitoring

### Database Optimization (Modern)

- **Prisma 5.22+ Optimizations**:
  - Connection pooling with PgBouncer
  - Query optimization with EXPLAIN ANALYZE
  - Selective field loading (Prisma select)
  - Batch operations for bulk data
- **PostgreSQL 16+ Features**:
  - Parallel query execution
  - Advanced indexing (GIN, GIST)
  - Partitioning for large tables
  - Query plan caching
- **Read Replicas**: Master-slave replication for read scaling

### Frontend Optimization (NextJS 15 + React 19)

- **Server Components**: Zero-bundle server-side rendering
- **Streaming SSR**: Progressive page loading
- **Turbopack**: Ultra-fast bundling and HMR
- **Image Optimization**: Next/Image with WebP/AVIF
- **Code Splitting**: Route-based and component-based
- **Bundle Analysis**: Webpack Bundle Analyzer integration
- **Performance Monitoring**: Core Web Vitals tracking

### Real-time Optimization

- **Socket.IO v4+ Optimizations**:
  - Connection pooling
  - Room-based broadcasting
  - Binary data support
  - Compression middleware
- **WebSocket Scaling**: Redis adapter for multi-instance

## Deployment Architecture (2025 Modern)

### Docker Configuration (Multi-stage & Optimized)

```yaml
# docker-compose.yml
version: '3.8'
services:
  # NextJS Frontend
  web:
    build:
      context: .
      dockerfile: apps/web/Dockerfile
      target: production
    ports:
      - '3000:3000'
    environment:
      - NEXT_PUBLIC_API_URL=http://api:4000
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:14268
    depends_on:
      - api
    healthcheck:
      test: ['CMD', 'curl', '-f', 'http://localhost:3000/api/health']
      interval: 30s
      timeout: 10s
      retries: 3

  # NestJS API Gateway
  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile
      target: production
    ports:
      - '4000:4000'
    environment:
      - DATABASE_URL=postgresql://salon_user:salon_pass@postgres:5432/salon_management
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:14268
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ['CMD', 'curl', '-f', 'http://localhost:4000/health']
      interval: 30s
      timeout: 10s
      retries: 3

  # PostgreSQL 16 with optimizations
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: salon_management
      POSTGRES_USER: salon_user
      POSTGRES_PASSWORD: salon_pass
      POSTGRES_INITDB_ARGS: '--encoding=UTF8 --locale=tr_TR.UTF-8'
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d
    ports:
      - '5432:5432'
    command: >
      postgres -c shared_preload_libraries=pg_stat_statements -c
      max_connections=200 -c shared_buffers=256MB -c effective_cache_size=1GB -c
      work_mem=4MB
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U salon_user -d salon_management']
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 7 with persistence
  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 10s
      timeout: 5s
      retries: 3

  # OpenTelemetry Collector
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ['--config=/etc/otel-collector-config.yaml']
    volumes:
      - ./observability/otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - '4317:4317' # OTLP gRPC receiver
      - '4318:4318' # OTLP HTTP receiver
    depends_on:
      - jaeger

  # Jaeger for distributed tracing
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - '16686:16686' # Jaeger UI
      - '14268:14268' # Jaeger collector
    environment:
      - COLLECTOR_OTLP_ENABLED=true

  # Prometheus for metrics
  prometheus:
    image: prom/prometheus:latest
    ports:
      - '9090:9090'
    volumes:
      - ./observability/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  # Grafana for dashboards
  grafana:
    image: grafana/grafana:latest
    ports:
      - '3001:3000'
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./observability/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./observability/grafana/datasources:/etc/grafana/provisioning/datasources

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
  grafana_data:

networks:
  default:
    driver: bridge
```

### Environment Configuration (Secure & Scalable)

```bash
# .env.production
NODE_ENV=production
PORT=4000

# Database
DATABASE_URL=postgresql://salon_user:${DB_PASSWORD}@postgres:5432/salon_management
DATABASE_POOL_SIZE=20

# Redis
REDIS_URL=redis://redis:6379
REDIS_TTL=3600

# Authentication
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d
GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}
GOOGLE_CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}

# File Storage
UPLOAD_MAX_SIZE=10485760  # 10MB
ALLOWED_FILE_TYPES=image/jpeg,image/png,image/webp

# Monitoring
OTEL_SERVICE_NAME=salon-management-api
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
OTEL_RESOURCE_ATTRIBUTES=service.name=salon-management,service.version=1.0.0

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000  # 15 minutes
RATE_LIMIT_MAX_REQUESTS=100

# CORS
CORS_ORIGIN=https://salon.example.com
CORS_CREDENTIALS=true
```

### Health Checks & Monitoring

```typescript
/**
 * Comprehensive health check endpoint
 */
@Controller('health')
export class HealthController {
  constructor(
    private readonly prisma: PrismaService,
    private readonly redis: RedisService
  ) {}

  @Get()
  async getHealth(): Promise<HealthResponse> {
    const checks = await Promise.allSettled([
      this.checkDatabase(),
      this.checkRedis(),
      this.checkExternalServices(),
    ]);

    return {
      status: checks.every(check => check.status === 'fulfilled')
        ? 'healthy'
        : 'unhealthy',
      timestamp: new Date().toISOString(),
      checks: {
        database: checks[0].status === 'fulfilled' ? 'healthy' : 'unhealthy',
        redis: checks[1].status === 'fulfilled' ? 'healthy' : 'unhealthy',
        external: checks[2].status === 'fulfilled' ? 'healthy' : 'unhealthy',
      },
      uptime: process.uptime(),
      memory: process.memoryUsage(),
    };
  }
}
```

### Production Deployment (Kubernetes Ready)

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: salon-management-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: salon-management-api
  template:
    metadata:
      labels:
        app: salon-management-api
    spec:
      containers:
        - name: api
          image: salon-management/api:latest
          ports:
            - containerPort: 4000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: salon-secrets
                  key: database-url
          resources:
            requests:
              memory: '256Mi'
              cpu: '250m'
            limits:
              memory: '512Mi'
              cpu: '500m'
          livenessProbe:
            httpGet:
              path: /health
              port: 4000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 4000
            initialDelaySeconds: 5
            periodSeconds: 5
```

Bu tasarım dokümanı, salon yönetim sisteminin teknik mimarisini ve
implementasyon detaylarını kapsamlı şekilde açıklamaktadır. Mikroservis
mimarisi, güvenlik önlemleri ve performans optimizasyonları ile ölçeklenebilir
ve sürdürülebilir bir sistem hedeflenmektedir.

## Modern Frontend Architecture (NextJS 15 + React 19)

### Server Components Strategy

```typescript
/**
 * Server Component - Randevu listesi (SSR)
 */
// app/appointments/page.tsx
export default async function AppointmentsPage({
  searchParams,
}: {
  searchParams: { page?: string; status?: string };
}) {
  // Server-side data fetching
  const appointments = await getAppointments({
    page: parseInt(searchParams.page || "1"),
    status: searchParams.status,
  });

  return (
    <div className="container mx-auto p-6">
      <AppointmentHeader />
      <Suspense fallback={<AppointmentListSkeleton />}>
        <AppointmentList appointments={appointments} />
      </Suspense>
      <AppointmentPagination />
    </div>
  );
}

/**
 * Client Component - Interactive randevu formu
 */
("use client");

import { useState, useTransition } from "react";
import { createAppointment } from "@/actions/appointments";

export function AppointmentForm() {
  const [isPending, startTransition] = useTransition();
  const [formData, setFormData] = useState<AppointmentFormData>({});

  const handleSubmit = async (formData: FormData) => {
    startTransition(async () => {
      const result = await createAppointment(formData);
      if (result.success) {
        toast.success("Randevu başarıyla oluşturuldu");
      } else {
        toast.error(result.error);
      }
    });
  };

  return (
    <form action={handleSubmit} className="space-y-4">
      {/* Form fields */}
      <Button type="submit" disabled={isPending}>
        {isPending ? "Oluşturuluyor..." : "Randevu Oluştur"}
      </Button>
    </form>
  );
}
```

### Real-time Updates (Socket.IO v4)

```typescript
/**
 * Socket.IO Gateway - Real-time randevu güncellemeleri
 */
@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true,
  },
  transports: ['websocket', 'polling'],
})
export class AppointmentSocketGateway {
  @WebSocketServer()
  server: Server;

  constructor(private readonly jwtService: JwtService) {}

  /**
   * Client bağlantısı ve authentication
   */
  async handleConnection(client: Socket) {
    try {
      const token = client.handshake.auth.token;
      const payload = this.jwtService.verify(token);

      client.data.userId = payload.sub;
      client.data.role = payload.role;

      // Role-based room'lara katılım
      if (payload.role === 'employee' || payload.role === 'admin') {
        client.join('staff');
      }
      client.join(`user:${payload.sub}`);

      console.log(`Client connected: ${client.id} (User: ${payload.sub})`);
    } catch (error) {
      client.disconnect();
    }
  }

  /**
   * Randevu oluşturulduğunda bildirim
   */
  async notifyAppointmentCreated(appointment: Appointment) {
    // Staff'a bildirim
    this.server.to('staff').emit('appointment:created', {
      appointment,
      message: 'Yeni randevu oluşturuldu',
    });

    // Müşteriye bildirim (eğer kayıtlı ise)
    if (appointment.customerId) {
      this.server
        .to(`user:${appointment.customerId}`)
        .emit('appointment:created', {
          appointment,
          message: 'Randevunuz başarıyla oluşturuldu',
        });
    }
  }

  /**
   * Randevu durumu güncellendiğinde bildirim
   */
  async notifyAppointmentUpdated(appointment: Appointment) {
    const message = this.getStatusUpdateMessage(appointment.status);

    // Tüm ilgili kullanıcılara bildirim
    this.server.to('staff').emit('appointment:updated', {
      appointment,
      message,
    });

    if (appointment.customerId) {
      this.server
        .to(`user:${appointment.customerId}`)
        .emit('appointment:updated', {
          appointment,
          message,
        });
    }
  }

  private getStatusUpdateMessage(status: AppointmentStatus): string {
    const messages = {
      [AppointmentStatus.CONFIRMED]: 'Randevunuz onaylandı',
      [AppointmentStatus.CANCELLED]: 'Randevunuz iptal edildi',
      [AppointmentStatus.COMPLETED]: 'Randevunuz tamamlandı',
    };
    return messages[status] || 'Randevu durumu güncellendi';
  }
}
```

### State Management (Zustand v4)

```typescript
/**
 * Appointment store - Client-side state management
 */
interface AppointmentStore {
  appointments: Appointment[];
  selectedAppointment: Appointment | null;
  filters: AppointmentFilters;
  isLoading: boolean;

  // Actions
  setAppointments: (appointments: Appointment[]) => void;
  addAppointment: (appointment: Appointment) => void;
  updateAppointment: (id: string, updates: Partial<Appointment>) => void;
  setFilters: (filters: Partial<AppointmentFilters>) => void;
  setLoading: (loading: boolean) => void;
}

export const useAppointmentStore = create<AppointmentStore>((set, get) => ({
  appointments: [],
  selectedAppointment: null,
  filters: {},
  isLoading: false,

  setAppointments: appointments => set({ appointments }),

  addAppointment: appointment =>
    set(state => ({
      appointments: [appointment, ...state.appointments],
    })),

  updateAppointment: (id, updates) =>
    set(state => ({
      appointments: state.appointments.map(apt =>
        apt.id === id ? { ...apt, ...updates } : apt
      ),
    })),

  setFilters: filters =>
    set(state => ({
      filters: { ...state.filters, ...filters },
    })),

  setLoading: isLoading => set({ isLoading }),
}));

/**
 * Real-time appointment updates hook
 */
export function useAppointmentRealtime() {
  const { addAppointment, updateAppointment } = useAppointmentStore();
  const { data: session } = useSession();

  useEffect(() => {
    if (!session?.accessToken) return;

    const socket = io(process.env.NEXT_PUBLIC_API_URL!, {
      auth: { token: session.accessToken },
      transports: ['websocket'],
    });

    socket.on('appointment:created', ({ appointment }) => {
      addAppointment(appointment);
      toast.success('Yeni randevu oluşturuldu');
    });

    socket.on('appointment:updated', ({ appointment }) => {
      updateAppointment(appointment.id, appointment);
      toast.info('Randevu güncellendi');
    });

    return () => {
      socket.disconnect();
    };
  }, [session?.accessToken, addAppointment, updateAppointment]);
}
```

## Observability & Monitoring (OpenTelemetry)

### Distributed Tracing

```typescript
/**
 * OpenTelemetry configuration
 */
// tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { PrismaInstrumentation } from '@prisma/instrumentation';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-otlp-http';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-fs': {
        enabled: false, // Disable file system instrumentation
      },
    }),
    new PrismaInstrumentation(),
  ],
});

sdk.start();

/**
 * Custom tracing decorator
 */
export function Trace(operationName: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const tracer = trace.getTracer('salon-management');
      const span = tracer.startSpan(
        `${target.constructor.name}.${operationName}`
      );

      try {
        span.setAttributes({
          'service.name': 'salon-management',
          'operation.name': operationName,
          'method.name': propertyKey,
        });

        const result = await originalMethod.apply(this, args);
        span.setStatus({ code: SpanStatusCode.OK });
        return result;
      } catch (error) {
        span.recordException(error);
        span.setStatus({
          code: SpanStatusCode.ERROR,
          message: error.message,
        });
        throw error;
      } finally {
        span.end();
      }
    };

    return descriptor;
  };
}
```

### Metrics Collection

```typescript
/**
 * Custom metrics for business logic
 */
@Injectable()
export class MetricsService {
  private readonly appointmentCounter = metrics.createCounter(
    'appointments_total',
    {
      description: 'Total number of appointments created',
    }
  );

  private readonly appointmentDuration = metrics.createHistogram(
    'appointment_duration_minutes',
    {
      description: 'Duration of appointments in minutes',
      buckets: [30, 60, 90, 120, 180, 240],
    }
  );

  private readonly activeUsers = metrics.createGauge('active_users_total', {
    description: 'Number of currently active users',
  });

  recordAppointmentCreated(status: AppointmentStatus) {
    this.appointmentCounter.add(1, { status });
  }

  recordAppointmentDuration(duration: number) {
    this.appointmentDuration.record(duration);
  }

  updateActiveUsers(count: number) {
    this.activeUsers.record(count);
  }
}
```

Bu güncellenmiş tasarım dokümanı, 2025 yılının en güncel teknolojileri ve en iyi
pratikleri kullanarak salon yönetim sisteminin modern, ölçeklenebilir ve
sürdürülebilir bir mimarisini sunuyor. NextJS 15, React 19, NestJS 10.4+, Prisma
5.22+, OpenTelemetry monitoring ve Docker containerization ile tam entegre bir
çözüm sağlıyor.
